# ⏰ Scheduling Patterns

> Cron jobs and background task scheduling for Next.js.

## Modules

| Module | What it does | Source |
|--------|-------------|--------|
| [cron-patterns.md](./cron-patterns.md) | Vercel Cron vs Inngest vs trigger.dev comparison + code examples | Official docs |

## Quick Decision

- **Simple crons (< 5)**: Use Vercel Cron → zero dependencies
- **Multi-step workflows**: Use Inngest → step functions with per-step retries
- **Long-running jobs**: Use trigger.dev → runs outside serverless limits
