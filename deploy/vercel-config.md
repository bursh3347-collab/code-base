# Vercel Configuration

> Production-ready vercel.json config: environment variables, Cron Jobs, security headers, and redirects.

**Source**: [Vercel Documentation](https://vercel.com/docs) + [Next.js SaaS Starter](https://github.com/leerob/next-saas-starter) (8.2k⭐, MIT)
**Stack**: Next.js 14+ on Vercel

---

## vercel.json Configuration

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": "nextjs",
  "regions": ["iad1"],
  "crons": [
    {
      "path": "/api/cron/daily-digest",
      "schedule": "0 9 * * *"
    },
    {
      "path": "/api/cron/cleanup",
      "schedule": "0 0 * * 0"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" }
      ]
    }
  ]
}
```

---

## Cron Job API Route

```typescript
// app/api/cron/daily-digest/route.ts
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  // Verify the request is from Vercel Cron
  const authHeader = request.headers.get("authorization");
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  try {
    // Your cron logic here
    console.log("Running daily digest...");

    return NextResponse.json({ success: true, timestamp: new Date().toISOString() });
  } catch (error) {
    console.error("Cron job failed:", error);
    return NextResponse.json({ error: "Internal error" }, { status: 500 });
  }
}
```

---

## Environment Variables Management

```typescript
// lib/env.ts — Type-safe environment variables with Zod
import { z } from "zod";

const envSchema = z.object({
  // Database
  DATABASE_URL: z.string().url(),
  // Auth
  AUTH_SECRET: z.string().min(32),
  // Stripe
  STRIPE_SECRET_KEY: z.string().startsWith("sk_"),
  STRIPE_WEBHOOK_SECRET: z.string().startsWith("whsec_"),
  NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY: z.string().startsWith("pk_"),
  // OpenAI
  OPENAI_API_KEY: z.string().startsWith("sk-"),
  // App
  NEXT_PUBLIC_APP_URL: z.string().url(),
  CRON_SECRET: z.string().min(16),
});

export const env = envSchema.parse(process.env);

// Usage: import { env } from "@/lib/env";
// env.DATABASE_URL — fully typed, validated at startup
```

```env
# .env.local (never commit this)
DATABASE_URL="postgresql://..."
AUTH_SECRET="your-secret-min-32-chars-long-here"
STRIPE_SECRET_KEY="sk_test_..."
STRIPE_WEBHOOK_SECRET="whsec_..."
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_test_..."
OPENAI_API_KEY="sk-..."
NEXT_PUBLIC_APP_URL="http://localhost:3000"
CRON_SECRET="your-cron-secret-here"
```

---

## Security Headers (next.config.ts)

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: [
          {
            key: "Content-Security-Policy",
            value: [
              "default-src 'self'",
              "script-src 'self' 'unsafe-eval' 'unsafe-inline' https://js.stripe.com",
              "style-src 'self' 'unsafe-inline'",
              "img-src 'self' blob: data: https:",
              "font-src 'self'",
              "connect-src 'self' https://api.stripe.com https://api.openai.com",
              "frame-src https://js.stripe.com",
            ].join("; "),
          },
          { key: "Strict-Transport-Security", value: "max-age=31536000; includeSubDomains" },
        ],
      },
    ];
  },
};

export default nextConfig;
```

---

**Why this is good**: Type-safe env vars catch misconfigurations at deploy time, not runtime. Cron auth prevents unauthorized triggers. CSP headers block XSS attacks.
