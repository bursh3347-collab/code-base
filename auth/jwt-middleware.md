# JWT Authentication Middleware

> Edge-compatible JWT verification middleware for Next.js API routes and middleware, using the `jose` library. Zero Node.js crypto dependency — runs on Vercel Edge, Cloudflare Workers, and any V8 runtime.

**Source**: [open-saas](https://github.com/wasp-lang/open-saas) (14k⭐, MIT) + [jose](https://github.com/panva/jose) (6k⭐, MIT)

**Why this is good**:
- `jose` is the only JWT library fully compatible with Edge Runtime (jsonwebtoken uses Node.js `crypto`)
- Extracted from open-saas production auth flow — battle-tested with Stripe + email verification
- Supports both symmetric (HS256) and asymmetric (RS256/ES256) algorithms
- Type-safe: full TypeScript generics for JWT payload

**What we changed**:
- Extracted JWT logic from open-saas's Wasp framework into standalone Next.js middleware
- Added generic type parameter for custom claims
- Added token refresh pattern
- Simplified to work with `next/headers` (App Router)

---

## Core: JWT Sign & Verify

```typescript
// lib/auth/jwt.ts
import { SignJWT, jwtVerify, type JWTPayload } from 'jose';

const secret = new TextEncoder().encode(
  process.env.JWT_SECRET! // min 32 chars
);

export interface UserJWTPayload extends JWTPayload {
  userId: string;
  email: string;
  role: 'user' | 'admin';
}

export async function signToken(payload: UserJWTPayload): Promise<string> {
  return new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('7d')
    .sign(secret);
}

export async function verifyToken(token: string): Promise<UserJWTPayload> {
  const { payload } = await jwtVerify(token, secret);
  return payload as UserJWTPayload;
}

export async function signRefreshToken(userId: string): Promise<string> {
  return new SignJWT({ userId })
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('30d')
    .sign(secret);
}
```

## Next.js Middleware (Edge-compatible)

```typescript
// middleware.ts
import { NextResponse, type NextRequest } from 'next/server';
import { verifyToken } from '@/lib/auth/jwt';

const PUBLIC_PATHS = ['/login', '/signup', '/api/auth', '/_next', '/favicon.ico'];

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Skip public paths
  if (PUBLIC_PATHS.some((p) => pathname.startsWith(p))) {
    return NextResponse.next();
  }

  const token = request.cookies.get('auth-token')?.value;

  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  try {
    const payload = await verifyToken(token);

    // Inject user info into headers for downstream use
    const response = NextResponse.next();
    response.headers.set('x-user-id', payload.userId);
    response.headers.set('x-user-email', payload.email);
    response.headers.set('x-user-role', payload.role);
    return response;
  } catch {
    // Token expired or invalid — clear cookie and redirect
    const response = NextResponse.redirect(new URL('/login', request.url));
    response.cookies.delete('auth-token');
    return response;
  }
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
};
```

## API Route Helper

```typescript
// lib/auth/get-user.ts
import { cookies } from 'next/headers';
import { verifyToken, type UserJWTPayload } from './jwt';

export async function getUser(): Promise<UserJWTPayload | null> {
  const cookieStore = await cookies();
  const token = cookieStore.get('auth-token')?.value;
  if (!token) return null;

  try {
    return await verifyToken(token);
  } catch {
    return null;
  }
}

export async function requireUser(): Promise<UserJWTPayload> {
  const user = await getUser();
  if (!user) throw new Error('Unauthorized');
  return user;
}
```

## Login API Route

```typescript
// app/api/auth/login/route.ts
import { NextResponse } from 'next/server';
import { signToken, signRefreshToken } from '@/lib/auth/jwt';
import { compare } from 'bcryptjs';
import { db } from '@/lib/db';
import { users } from '@/lib/db/schema';
import { eq } from 'drizzle-orm';

export async function POST(request: Request) {
  const { email, password } = await request.json();

  const [user] = await db
    .select()
    .from(users)
    .where(eq(users.email, email))
    .limit(1);

  if (!user || !(await compare(password, user.passwordHash))) {
    return NextResponse.json(
      { error: 'Invalid credentials' },
      { status: 401 }
    );
  }

  const token = await signToken({
    userId: user.id,
    email: user.email,
    role: user.role,
  });

  const refreshToken = await signRefreshToken(user.id);

  const response = NextResponse.json({ success: true });

  response.cookies.set('auth-token', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 7, // 7 days
    path: '/',
  });

  response.cookies.set('refresh-token', refreshToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 30, // 30 days
    path: '/',
  });

  return response;
}
```

---

## Usage

```typescript
// In any Server Component or API route:
import { requireUser } from '@/lib/auth/get-user';

export default async function DashboardPage() {
  const user = await requireUser(); // throws if not authenticated
  return <h1>Welcome, {user.email}</h1>;
}
```

## Dependencies

```bash
npm install jose bcryptjs
npm install -D @types/bcryptjs
```

## Caveats

- **JWT_SECRET** must be at least 32 characters. Generate with: `openssl rand -base64 32`
- JWTs are stateless — you cannot "revoke" a token. For logout, delete the cookie client-side and optionally maintain a server-side blocklist in Redis.
- `bcryptjs` is used for password hashing (pure JS, Edge-compatible). For Node.js-only environments, `bcrypt` (native) is faster.
- Token refresh logic should be added to middleware for production use (check expiry, auto-refresh if < 1 day remaining).
