# Structured Logging with Pino

> Production JSON logging: Pino setup + request logging middleware + log levels.

**Source**: [Pino](https://github.com/pinojs/pino) (14k⭐, MIT)
**Stack**: TypeScript + Next.js 14 + Pino

---

## Logger Setup

```typescript
// lib/logger.ts
import pino from "pino";

export const logger = pino({
  level: process.env.LOG_LEVEL || (process.env.NODE_ENV === "production" ? "info" : "debug"),
  // Pretty print in development
  transport:
    process.env.NODE_ENV !== "production"
      ? { target: "pino-pretty", options: { colorize: true } }
      : undefined,
  // Base fields included in every log
  base: {
    env: process.env.NODE_ENV,
    version: process.env.npm_package_version,
  },
  // Redact sensitive fields
  redact: {
    paths: ["req.headers.authorization", "req.headers.cookie", "password", "token", "secret"],
    censor: "[REDACTED]",
  },
  // Custom timestamp format
  timestamp: pino.stdTimeFunctions.isoTime,
});

// Create child loggers for different modules
export const dbLogger = logger.child({ module: "database" });
export const authLogger = logger.child({ module: "auth" });
export const aiLogger = logger.child({ module: "ai" });
export const stripeLogger = logger.child({ module: "stripe" });
```

---

## Request Logging Middleware

```typescript
// middleware.ts (or wrap in API routes)
import { NextResponse, type NextRequest } from "next/server";
import { logger } from "@/lib/logger";

export function middleware(request: NextRequest) {
  const start = Date.now();
  const requestId = crypto.randomUUID();

  // Add request ID header
  const response = NextResponse.next();
  response.headers.set("x-request-id", requestId);

  // Log request
  logger.info({
    type: "request",
    requestId,
    method: request.method,
    url: request.nextUrl.pathname,
    userAgent: request.headers.get("user-agent")?.substring(0, 100),
    ip: request.headers.get("x-forwarded-for") || "unknown",
    latencyMs: Date.now() - start,
  });

  return response;
}

export const config = {
  matcher: ["/api/:path*"],
};
```

---

## Usage in API Routes

```typescript
// app/api/projects/route.ts
import { logger } from "@/lib/logger";

export async function POST(req: Request) {
  const reqLogger = logger.child({ route: "POST /api/projects" });

  try {
    const body = await req.json();
    reqLogger.info({ action: "create_project", name: body.name });

    const project = await createProject(body);
    reqLogger.info({ action: "project_created", projectId: project.id });

    return Response.json(project, { status: 201 });
  } catch (error) {
    reqLogger.error({ action: "create_project_failed", error: (error as Error).message });
    return Response.json({ error: "Failed to create project" }, { status: 500 });
  }
}
```

---

## Usage in Business Logic

```typescript
// lib/stripe-webhook.ts
import { stripeLogger } from "@/lib/logger";

export async function handleWebhook(event: Stripe.Event) {
  const log = stripeLogger.child({ eventId: event.id, type: event.type });

  switch (event.type) {
    case "checkout.session.completed":
      log.info({ action: "checkout_completed", customerId: event.data.object.customer });
      await activateSubscription(event.data.object);
      break;

    case "customer.subscription.deleted":
      log.warn({ action: "subscription_cancelled", customerId: event.data.object.customer });
      await deactivateSubscription(event.data.object);
      break;

    default:
      log.debug({ action: "unhandled_event" });
  }
}
```

---

## Log Output Examples

```json
// Production (JSON — parseable by log aggregators)
{"level":30,"time":"2026-04-16T01:00:00.000Z","env":"production","module":"stripe","eventId":"evt_123","type":"checkout.session.completed","action":"checkout_completed","customerId":"cus_456"}

// Development (pretty-printed)
// [01:00:00] INFO (stripe): checkout_completed
//   eventId: "evt_123"
//   customerId: "cus_456"
```

---

**Dependencies**:

```bash
npm install pino
npm install -D pino-pretty  # dev only
```

**Why Pino**: 5x faster than Winston. JSON output works with every log aggregator (Datadog, Logtail, Axiom). Child loggers add context without repetition. Redaction prevents accidental secret logging.
