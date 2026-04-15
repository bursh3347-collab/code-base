# 🔌 API — API Design Patterns

> Type-safe, validated, and rate-limited API patterns for Next.js 14+ App Router. Production-ready TypeScript.

## Quick Selection Guide

| Pattern | Best For | Complexity | Type Safety | Source |
|---------|----------|------------|-------------|--------|
| **[tRPC Setup](./trpc-setup.md)** | Full-stack TS apps, internal APIs | Medium | ✅ End-to-end | tRPC 40k⭐ |
| **[REST Patterns](./rest-patterns.md)** | Public APIs, multi-client | Low | ✅ With Zod | NestJS 70k⭐ |
| **[Rate Limiting](./rate-limiting.md)** | All APIs (add to either above) | Low | N/A | Hono 30k⭐ |

## ⭐ Recommended

**If you're building a SaaS with only a web frontend, start with tRPC.**

Reasons:
- Zero API boilerplate — define a function on the server, call it on the client with full autocomplete
- Type errors are caught at build time, not runtime
- Built-in React Query integration for caching and mutations

**If you need a public API (mobile app, third-party integrations), use REST Patterns.**

## When to Use What

```
┌─ Only web frontend (Next.js)? → tRPC
│
├─ Need public API / mobile clients? → REST Patterns
│
├─ Both? → tRPC for internal + REST for public
│
└─ Always add Rate Limiting on top of either
```

## Combining Patterns

- **tRPC + Rate Limiting**: Apply rate limit in tRPC middleware
- **REST + Rate Limiting**: Apply in Next.js middleware or per-route
- **REST + tRPC**: Use REST for public endpoints, tRPC for dashboard/internal

## Stack Compatibility

| Component | tRPC | REST | Rate Limiting |
|-----------|------|------|---------------|
| Next.js App Router | ✅ | ✅ | ✅ |
| Vercel Edge | ✅ | ✅ | ✅ (Upstash) |
| React Server Components | ✅ (server caller) | ✅ | N/A |
| Zod Validation | ✅ (built-in) | ✅ (manual) | N/A |
| Drizzle ORM | ✅ | ✅ | N/A |
