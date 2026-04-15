# 🗄️ Database — ORM & Data Patterns

> Production-ready database patterns for SaaS applications. TypeScript-first, serverless-optimized.

## Quick Selection Guide

| Pattern | Best For | Complexity | Edge Compatible | Source |
|---------|----------|------------|-----------------|--------|
| **[Drizzle Setup](./drizzle-setup.md)** | New projects, serverless | Medium | ✅ Yes | Drizzle 30k⭐ |
| **[Supabase Patterns](./supabase-patterns.md)** | Full-stack BaaS, rapid MVP | Low | ✅ Yes | Supabase 78k⭐ |
| **[Migration Patterns](./migration-patterns.md)** | Choosing/switching ORMs | Reference | N/A | Prisma + Drizzle |

## ⭐ Recommended

**For most SaaS projects, use Supabase + Drizzle together.**

```
Supabase = Hosted Postgres + Auth + Realtime + Storage
Drizzle  = Type-safe query builder for that Postgres
```

This gives you:
- Supabase's managed infrastructure (backups, connection pooling, dashboard)
- Drizzle's type-safe queries and lightweight runtime
- RLS for multi-tenant data isolation
- Realtime subscriptions out of the box

## Architecture Decision

```
┌─ Need BaaS (Auth + Storage + Realtime)?
│  YES → Supabase Patterns (all-in-one)
│  NO  ↓
│
├─ Just need a database + ORM?
│  → Drizzle Setup (connect to any Postgres)
│
└─ Migrating from Prisma?
   → Migration Patterns (step-by-step guide)
```

## Stack Compatibility

| Component | Drizzle | Supabase | Prisma |
|-----------|---------|----------|--------|
| Vercel Edge | ✅ | ✅ | ⚠️ Accelerate |
| Connection Pooling | postgres.js | Built-in (PgBouncer) | Prisma Accelerate |
| Auto Migrations | ✅ drizzle-kit | ✅ SQL migrations | ✅ prisma migrate |
| Visual Browser | ✅ Drizzle Studio | ✅ Supabase Dashboard | ✅ Prisma Studio |
| Bundle Size | ~50KB | ~100KB (client) | ~2MB (engine) |
| RLS Support | Manual SQL | ✅ Native | Manual SQL |

## Connection String Cheatsheet

```env
# Supabase (use pooler for serverless)
DATABASE_URL=postgresql://postgres.[ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres

# Neon (serverless Postgres)
DATABASE_URL=postgresql://[user]:[password]@[endpoint].neon.tech/[dbname]?sslmode=require

# Local development
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp
```

## Security Checklist

- [ ] RLS enabled on every table with user data
- [ ] Service role key is server-side only (never in `NEXT_PUBLIC_*`)
- [ ] Connection string uses pooler endpoint for serverless
- [ ] Migrations are version-controlled (never use `push` in production)
- [ ] Sensitive columns are encrypted at application level if needed
- [ ] Database backups are configured (Supabase Pro: daily automatic)
