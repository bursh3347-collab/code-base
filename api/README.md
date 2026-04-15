# 🌐 API Design Patterns

> Extracted API architectures, middleware patterns, and endpoint designs from high-star projects.

## 📦 What's Inside

| Approach | Source Project | Complexity | Best For |
|----------|---------------|------------|----------|
| REST (Next.js API Routes) | *TBD* | Low | Simple CRUD, most SaaS apps |
| tRPC | *TBD* | Medium | Full-stack TypeScript, type-safe |
| GraphQL | *TBD* | High | Complex data graphs, mobile apps |

## 🎯 Selection Guide

```
Full-stack TypeScript monorepo?
├── Yes → tRPC (end-to-end type safety, zero boilerplate)
└── No
    ├── Complex nested data? → GraphQL
    ├── Public API needed? → REST (universal compatibility)
    └── Simple SaaS? → REST with Next.js API Routes
```

### ⭐ Recommended Default: REST (Next.js API Routes)

**Why**: Simplest, most universal, zero additional dependencies. Sufficient for 90% of Micro SaaS use cases.

## 📁 Directory Structure

```
api/
├── README.md          ← You are here
├── rest/              ← REST API patterns + middleware
├── trpc/              ← tRPC setup + router patterns
├── graphql/           ← GraphQL schema + resolver patterns
└── shared/            ← Cross-approach patterns (rate limiting, validation)
```

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
