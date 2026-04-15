# 📊 Monitoring & Observability

> Extracted monitoring setups, alerting patterns, and observability configurations from high-star projects.

## 📦 What's Inside

| Tool | Type | Best For |
|------|------|----------|
| Sentry | Error tracking | Crash reporting, performance |
| Vercel Analytics | Web vitals | Next.js performance monitoring |
| PostHog | Product analytics | User behavior, feature flags |
| Uptime monitors | Availability | SLA tracking |

## 🎯 Selection Guide

### ⭐ Recommended Default: Sentry + Vercel Analytics

**Why**: Sentry catches errors (free tier covers Micro SaaS), Vercel Analytics tracks performance (built-in for Vercel deploys).

## 📁 Directory Structure

```
monitoring/
├── README.md          ← You are here
├── sentry/            ← Sentry setup patterns
├── analytics/         ← Product analytics patterns
└── uptime/            ← Uptime monitoring patterns
```

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
