# ⏰ Scheduling & Cron Patterns

> Extracted scheduling patterns, cron job architectures, and task queue designs from high-star projects.

## 📦 What's Inside

| Pattern | Source | Type | Best For |
|---------|--------|------|----------|
| Cron Scheduler | Hermes Agent (86k⭐) | Natural language cron | Scheduled recurring tasks |
| Vercel Cron | Next.js ecosystem | Serverless cron | Vercel-deployed apps |
| BullMQ | Node.js | Job queue | Complex task pipelines |

## 🎯 Selection Guide

```
Hosting on Vercel?
├── Yes → Vercel Cron Jobs (zero setup, built-in)
└── No
    ├── Simple recurring tasks? → node-cron or cron library
    ├── Complex job pipelines? → BullMQ + Redis
    └── Natural language scheduling? → Hermes-style cron parser
```

### ⭐ Recommended Default: Vercel Cron Jobs

**Why**: Zero infrastructure, configured in vercel.json, perfect for Micro SaaS.

## 📁 Directory Structure

```
scheduling/
├── README.md          ← You are here
├── vercel-cron/       ← Vercel Cron Job patterns
├── cron-parser/       ← Cron expression patterns (from Hermes Agent)
└── job-queue/         ← BullMQ job queue patterns
```

## 🏷️ Origin Note

The cron parser pattern was identified during analysis of [NousResearch/hermes-agent](https://github.com/nousresearch/hermes-agent) (86k⭐, MIT). The natural language → cron expression conversion is a cross-domain reusable pattern.

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
