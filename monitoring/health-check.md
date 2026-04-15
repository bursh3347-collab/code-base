# Health Check Endpoint

> Production health check: status + uptime + version + dependency checks.

**Source**: Community best practices + [12-Factor App](https://12factor.net)
**Stack**: TypeScript + Next.js 14 App Router

---

## Basic Health Check

```typescript
// app/api/health/route.ts
import { NextResponse } from "next/server";

const startTime = Date.now();

export async function GET() {
  return NextResponse.json({
    status: "ok",
    timestamp: new Date().toISOString(),
    uptime: Math.floor((Date.now() - startTime) / 1000),
    version: process.env.npm_package_version || "unknown",
    environment: process.env.NODE_ENV,
  });
}
```

---

## Deep Health Check (with dependency checks)

```typescript
// app/api/health/deep/route.ts
import { NextResponse } from "next/server";
import { db } from "@/lib/db";
import { sql } from "drizzle-orm";

const startTime = Date.now();

interface HealthCheck {
  name: string;
  status: "ok" | "error";
  latencyMs: number;
  message?: string;
}

async function checkDatabase(): Promise<HealthCheck> {
  const start = Date.now();
  try {
    await db.execute(sql`SELECT 1`);
    return {
      name: "database",
      status: "ok",
      latencyMs: Date.now() - start,
    };
  } catch (error) {
    return {
      name: "database",
      status: "error",
      latencyMs: Date.now() - start,
      message: (error as Error).message,
    };
  }
}

async function checkOpenAI(): Promise<HealthCheck> {
  const start = Date.now();
  try {
    const res = await fetch("https://api.openai.com/v1/models", {
      headers: { Authorization: `Bearer ${process.env.OPENAI_API_KEY}` },
      signal: AbortSignal.timeout(5000),
    });
    return {
      name: "openai",
      status: res.ok ? "ok" : "error",
      latencyMs: Date.now() - start,
      message: res.ok ? undefined : `HTTP ${res.status}`,
    };
  } catch (error) {
    return {
      name: "openai",
      status: "error",
      latencyMs: Date.now() - start,
      message: (error as Error).message,
    };
  }
}

async function checkStripe(): Promise<HealthCheck> {
  const start = Date.now();
  try {
    const res = await fetch("https://api.stripe.com/v1/balance", {
      headers: { Authorization: `Bearer ${process.env.STRIPE_SECRET_KEY}` },
      signal: AbortSignal.timeout(5000),
    });
    return {
      name: "stripe",
      status: res.ok ? "ok" : "error",
      latencyMs: Date.now() - start,
    };
  } catch (error) {
    return {
      name: "stripe",
      status: "error",
      latencyMs: Date.now() - start,
      message: (error as Error).message,
    };
  }
}

export async function GET(request: Request) {
  // Protect deep health check
  const authHeader = request.headers.get("authorization");
  if (authHeader !== `Bearer ${process.env.HEALTH_CHECK_SECRET}`) {
    // Return basic health for unauthenticated requests
    return NextResponse.json({ status: "ok" });
  }

  const checks = await Promise.all([
    checkDatabase(),
    checkOpenAI(),
    checkStripe(),
  ]);

  const allOk = checks.every((c) => c.status === "ok");

  return NextResponse.json(
    {
      status: allOk ? "ok" : "degraded",
      timestamp: new Date().toISOString(),
      uptime: Math.floor((Date.now() - startTime) / 1000),
      version: process.env.npm_package_version || "unknown",
      checks,
    },
    { status: allOk ? 200 : 503 }
  );
}
```

---

**Usage**:

```bash
# Basic health check (public)
curl https://yourapp.com/api/health

# Deep health check (authenticated)
curl -H "Authorization: Bearer your-secret" https://yourapp.com/api/health/deep
```

**Why this is good**: Basic endpoint for uptime monitors (UptimeRobot, Better Uptime). Deep endpoint for debugging — shows which dependency is failing and how slow it is. Auth-protected to prevent information leakage.
