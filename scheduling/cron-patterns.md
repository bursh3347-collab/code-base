# Cron & Scheduling Patterns

> Comparison of scheduling solutions: Vercel Cron vs Inngest vs trigger.dev for Next.js.

**Source**: Official docs of each platform
**Stack**: TypeScript + Next.js 14

---

## Option Comparison

| Feature | Vercel Cron | Inngest | trigger.dev |
|---------|------------|---------|-------------|
| **Setup** | vercel.json + API route | npm install + dashboard | npm install + dashboard |
| **Pricing** | Free (2 crons on Hobby) | Free tier (generous) | Free tier (generous) |
| **Min interval** | 1 minute | Seconds | Seconds |
| **Retries** | Manual | Built-in (configurable) | Built-in |
| **Queuing** | No | Yes (concurrency control) | Yes |
| **Fan-out** | Manual | Built-in (step functions) | Built-in |
| **Observability** | Vercel logs only | Full dashboard | Full dashboard |
| **Long-running** | 10s-300s (plan dependent) | Up to hours | Up to hours |
| **Best for** | Simple scheduled tasks | Event-driven workflows | Background jobs |

---

## 1. Vercel Cron (Simplest)

```json
// vercel.json
{
  "crons": [
    { "path": "/api/cron/daily-report", "schedule": "0 9 * * *" },
    { "path": "/api/cron/weekly-cleanup", "schedule": "0 0 * * 0" }
  ]
}
```

```typescript
// app/api/cron/daily-report/route.ts
import { NextResponse } from "next/server";

export async function GET(request: Request) {
  // Auth: verify Vercel Cron secret
  if (
    request.headers.get("authorization") !== `Bearer ${process.env.CRON_SECRET}`
  ) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  // Your logic
  await generateDailyReport();

  return NextResponse.json({ success: true });
}
```

**✅ Use when**: Simple scheduled tasks, < 5 crons, no retries needed.

---

## 2. Inngest (Event-Driven Workflows)

```typescript
// inngest/client.ts
import { Inngest } from "inngest";
export const inngest = new Inngest({ id: "my-app" });
```

```typescript
// inngest/functions/daily-report.ts
import { inngest } from "../client";

export const dailyReport = inngest.createFunction(
  {
    id: "daily-report",
    retries: 3,
  },
  { cron: "0 9 * * *" }, // Every day at 9 AM UTC
  async ({ step }) => {
    // Step 1: Gather data (automatically retried on failure)
    const data = await step.run("gather-data", async () => {
      return await fetchAnalytics();
    });

    // Step 2: Generate report
    const report = await step.run("generate-report", async () => {
      return await generateReport(data);
    });

    // Step 3: Send email
    await step.run("send-email", async () => {
      await sendEmail({ to: "admin@example.com", body: report });
    });

    return { success: true };
  }
);
```

```typescript
// app/api/inngest/route.ts
import { serve } from "inngest/next";
import { inngest } from "@/inngest/client";
import { dailyReport } from "@/inngest/functions/daily-report";

export const { GET, POST, PUT } = serve({
  client: inngest,
  functions: [dailyReport],
});
```

**✅ Use when**: Multi-step workflows, need retries per step, event-driven architecture, fan-out jobs.

---

## 3. trigger.dev (Background Jobs)

```typescript
// trigger/daily-report.ts
import { schedules } from "@trigger.dev/sdk/v3";

export const dailyReportTask = schedules.task({
  id: "daily-report",
  // Cron schedule
  cron: "0 9 * * *",
  run: async (payload) => {
    const data = await fetchAnalytics();
    const report = await generateReport(data);
    await sendEmail({ to: "admin@example.com", body: report });

    return { success: true, reportId: report.id };
  },
});
```

**✅ Use when**: Long-running background jobs (> 5 min), need full observability, complex retry policies.

---

## ⭐ Recommendation

**Start with Vercel Cron** (zero extra dependencies). Migrate to Inngest when you need:
- Retries per step
- Multi-step workflows
- Event-driven triggers (not just time-based)
- Concurrency control

---

**Dependencies**:

```bash
# Inngest
npm install inngest

# trigger.dev
npm install @trigger.dev/sdk
```
