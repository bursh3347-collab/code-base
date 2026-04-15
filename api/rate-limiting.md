# Rate Limiting Patterns

> Protect your API from abuse with in-memory (dev) and Redis-based (production) rate limiting. Sliding window algorithm with configurable limits per route.

**Source**: [Hono Rate Limiter](https://github.com/honojs/hono) (30k⭐, MIT) + [Upstash Ratelimit](https://github.com/upstash/ratelimit) (2k⭐, MIT)

**Why this is good**:
- Two implementations: in-memory for development, Upstash Redis for production
- Upstash works on Vercel Edge (serverless Redis, no connection pooling needed)
- Sliding window algorithm is more accurate than fixed window
- Returns standard `RateLimit-*` headers for client transparency

**What we changed**:
- Extracted Hono's rate limit pattern into standalone Next.js utility
- Added Upstash Redis production implementation
- Added per-route configuration (different limits for auth vs API vs webhooks)
- Added helper to extract IP address on Vercel

---

## In-Memory Rate Limiter (Development)

```typescript
// lib/api/rate-limit-memory.ts
interface RateLimitEntry {
  count: number;
  resetAt: number;
}

const store = new Map<string, RateLimitEntry>();

// Clean up expired entries every 60s
setInterval(() => {
  const now = Date.now();
  for (const [key, entry] of store) {
    if (entry.resetAt < now) store.delete(key);
  }
}, 60_000);

export function memoryRateLimit(
  key: string,
  limit: number,
  windowMs: number
): { success: boolean; remaining: number; reset: number } {
  const now = Date.now();
  const entry = store.get(key);

  if (!entry || entry.resetAt < now) {
    store.set(key, { count: 1, resetAt: now + windowMs });
    return { success: true, remaining: limit - 1, reset: now + windowMs };
  }

  if (entry.count >= limit) {
    return { success: false, remaining: 0, reset: entry.resetAt };
  }

  entry.count++;
  return { success: true, remaining: limit - entry.count, reset: entry.resetAt };
}
```

## Upstash Redis Rate Limiter (Production)

```typescript
// lib/api/rate-limit-redis.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
});

// Different limiters for different routes
export const rateLimiters = {
  // Auth routes: 5 requests per minute
  auth: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(5, '1 m'),
    prefix: 'rl:auth',
  }),

  // API routes: 60 requests per minute
  api: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(60, '1 m'),
    prefix: 'rl:api',
  }),

  // Webhook endpoints: 10 requests per second
  webhook: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(10, '1 s'),
    prefix: 'rl:webhook',
  }),
};

export type RateLimitType = keyof typeof rateLimiters;
```

## Unified Rate Limit Helper

```typescript
// lib/api/rate-limit.ts
import { NextResponse, type NextRequest } from 'next/server';
import { memoryRateLimit } from './rate-limit-memory';
import { rateLimiters, type RateLimitType } from './rate-limit-redis';

const IS_DEV = process.env.NODE_ENV === 'development';

function getIP(request: NextRequest): string {
  return (
    request.headers.get('x-forwarded-for')?.split(',')[0]?.trim() ??
    request.headers.get('x-real-ip') ??
    '127.0.0.1'
  );
}

const LIMITS: Record<RateLimitType, { limit: number; windowMs: number }> = {
  auth: { limit: 5, windowMs: 60_000 },
  api: { limit: 60, windowMs: 60_000 },
  webhook: { limit: 10, windowMs: 1_000 },
};

export async function rateLimit(
  request: NextRequest,
  type: RateLimitType = 'api'
): Promise<NextResponse | null> {
  const ip = getIP(request);
  const key = `${type}:${ip}`;

  if (IS_DEV) {
    const { limit, windowMs } = LIMITS[type];
    const result = memoryRateLimit(key, limit, windowMs);

    if (!result.success) {
      return NextResponse.json(
        { data: null, error: { code: 'RATE_LIMITED', message: 'Too many requests' } },
        {
          status: 429,
          headers: {
            'RateLimit-Limit': String(limit),
            'RateLimit-Remaining': '0',
            'RateLimit-Reset': String(Math.ceil(result.reset / 1000)),
            'Retry-After': String(Math.ceil((result.reset - Date.now()) / 1000)),
          },
        }
      );
    }
    return null; // Allowed
  }

  // Production: Upstash Redis
  const limiter = rateLimiters[type];
  const { success, limit, remaining, reset } = await limiter.limit(key);

  if (!success) {
    return NextResponse.json(
      { data: null, error: { code: 'RATE_LIMITED', message: 'Too many requests' } },
      {
        status: 429,
        headers: {
          'RateLimit-Limit': String(limit),
          'RateLimit-Remaining': String(remaining),
          'RateLimit-Reset': String(reset),
          'Retry-After': String(Math.ceil((reset - Date.now()) / 1000)),
        },
      }
    );
  }

  return null; // Allowed
}
```

## Usage in API Route

```typescript
// app/api/auth/login/route.ts
import { type NextRequest } from 'next/server';
import { rateLimit } from '@/lib/api/rate-limit';
import { ok, err } from '@/lib/api/response';

export async function POST(request: NextRequest) {
  // Rate limit: 5 attempts per minute for auth
  const limited = await rateLimit(request, 'auth');
  if (limited) return limited;

  // ... your login logic
  return ok({ success: true });
}
```

## Usage in Middleware (Global)

```typescript
// middleware.ts — apply to all API routes
import { type NextRequest, NextResponse } from 'next/server';
import { rateLimit } from '@/lib/api/rate-limit';

export async function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/api/')) {
    const type = request.nextUrl.pathname.startsWith('/api/auth/')
      ? 'auth'
      : 'api';

    const limited = await rateLimit(request, type);
    if (limited) return limited;
  }

  return NextResponse.next();
}
```

---

## Dependencies

```bash
# Development (in-memory) — no deps needed

# Production (Upstash Redis)
npm install @upstash/ratelimit @upstash/redis
```

## Environment Variables (Production)

```env
UPSTASH_REDIS_REST_URL=https://xxx.upstash.io
UPSTASH_REDIS_REST_TOKEN=AXxx...
```

## Caveats

- In-memory rate limiter resets on every serverless cold start. Only use for development.
- Upstash free tier: 10,000 requests/day. Sufficient for side projects, upgrade for production.
- `x-forwarded-for` can be spoofed if not behind a trusted proxy. On Vercel, this header is set by the platform and is trustworthy.
- Rate limiting by IP affects all users behind the same NAT. For authenticated routes, consider rate limiting by user ID instead.
- Sliding window is slightly more expensive than fixed window but prevents burst attacks at window boundaries.
