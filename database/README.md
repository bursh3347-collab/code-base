# 🗄️ Database Best Practices

> Extracted database patterns, ORM configurations, and schema designs from high-star projects.

## 📦 What's Inside

| ORM/Tool | Source Project | Type | Best For |
|----------|---------------|------|----------|
| Drizzle ORM | *TBD* | Type-safe SQL | Lightweight, Next.js projects |
| Prisma | *TBD* | Full ORM | Rapid prototyping, complex relations |
| Supabase | *TBD* | BaaS + Postgres | Full-stack with auth + realtime |
| Raw SQL | *TBD* | Direct queries | Performance-critical paths |

## 🎯 Selection Guide

```
Project type?
├── SaaS MVP (speed matters) → Supabase (auth + DB + realtime in one)
├── Next.js app (type safety) → Drizzle ORM (lighter than Prisma)
├── Complex data model → Prisma (best migration tooling)
└── Performance critical → Raw SQL with connection pooling
```

### ⭐ Recommended Default: Drizzle ORM + Postgres

**Why**: Type-safe, lightweight, great DX with Next.js. Replaces Prisma with less overhead. Used by next-saas-starter (4.2k⭐).

## 📁 Directory Structure

```
database/
├── README.md          ← You are here
├── drizzle/           ← Drizzle ORM patterns + schema examples
├── prisma/            ← Prisma patterns + migration strategies
├── supabase/          ← Supabase client patterns + RLS policies
└── migrations/        ← Migration best practices (cross-ORM)
```

## 🔗 Cross-References

- Auth user tables → [../auth/](../auth/)
- API data fetching → [../api/](../api/)

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
