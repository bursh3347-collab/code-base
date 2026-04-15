# 📊 Monitoring Patterns

> Health checks and observability for production Next.js applications.

## Modules

| Module | What it does | Source |
|--------|-------------|--------|
| [health-check.md](./health-check.md) | Basic + deep health endpoints, dependency checks (DB, OpenAI, Stripe) | 12-Factor App |

## Recommended Setup

⭐ **For Micro SaaS**:
1. Deploy the basic `/api/health` endpoint
2. Connect to [Better Uptime](https://betteruptime.com) or [UptimeRobot](https://uptimerobot.com) (free tier)
3. Add `/api/health/deep` for debugging production issues

## Future additions
- Sentry error tracking integration
- Custom metrics with Vercel Analytics
- Request logging middleware
