# 📚 SOURCES — All Code Origins & Licenses

> Every code snippet and pattern in this repository traces back to a specific open-source project.
> This file documents all sources for transparency and license compliance.

---

## Auth Module (`auth/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| jwt-auth.md | [next-saas-starter](https://github.com/leerob/next-saas-starter) | 8.2k | MIT | JWT session management with Jose |
| supabase-auth.md | [Supabase Auth Helpers](https://github.com/supabase/auth-helpers) | 1.2k | MIT | Supabase SSR auth + middleware |
| middleware-protection.md | [next-saas-starter](https://github.com/leerob/next-saas-starter) | 8.2k | MIT | Route protection middleware |

## API Module (`api/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| rest-patterns.md | [next-saas-starter](https://github.com/leerob/next-saas-starter) | 8.2k | MIT | API route patterns + validation |
| rate-limiting.md | [Upstash Ratelimit](https://github.com/upstash/ratelimit) | 1.8k | MIT | Sliding window rate limiter |
| webhook-handling.md | [Stripe Node.js](https://github.com/stripe/stripe-node) | 4k | MIT | Stripe webhook verification |

## Database Module (`database/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| drizzle-schema.md | [Drizzle ORM](https://github.com/drizzle-team/drizzle-orm) | 28k | Apache-2.0 | Schema definition + relations |
| migrations.md | [Drizzle Kit](https://github.com/drizzle-team/drizzle-orm) | 28k | Apache-2.0 | Migration workflow |
| supabase-rls.md | [Supabase Docs](https://supabase.com/docs) | — | CC-BY | Row Level Security policies |

## UI Module (`ui/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| shadcn-patterns.md | [shadcn/ui](https://github.com/shadcn-ui/ui) | 112k | MIT | cn() utility, Form integration, themes |
| tailwind-utils.md | [Tailwind CSS](https://github.com/tailwindlabs/tailwindcss) | 94k | MIT | Responsive patterns, dark mode |
| form-validation.md | [Bulletproof React](https://github.com/alan2207/bulletproof-react) | 29k | MIT | Zod + react-hook-form patterns |

## Deploy Module (`deploy/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| vercel-config.md | [Vercel Docs](https://vercel.com/docs) + [next-saas-starter](https://github.com/leerob/next-saas-starter) | 8.2k | MIT | vercel.json, env validation, security headers |
| docker-compose.md | [FastAPI Template](https://github.com/tiangolo/full-stack-fastapi-template) | 42.6k | MIT | Multi-stage Docker, dev + prod compose (adapted for Next.js) |
| github-actions-ci.md | Community best practices | — | — | CI/CD pipeline with caching |

## AI Integration Module (`ai-integration/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| openai-streaming.md | [Vercel AI SDK](https://github.com/vercel/ai) | 12k | Apache-2.0 | streamText + useChat hook |
| rag-basic.md | [LlamaIndex.TS](https://github.com/run-llama/LlamaIndexTS) + [Supabase AI Docs](https://supabase.com/docs/guides/ai) | 3.5k | MIT | RAG pipeline with pgvector |
| embedding-search.md | [Supabase AI Docs](https://supabase.com/docs/guides/ai) + [OpenAI Cookbook](https://cookbook.openai.com) | — | MIT/CC | Vector + hybrid search |

## Testing Module (`testing/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| vitest-setup.md | [Vitest](https://github.com/vitest-dev/vitest) | 14k | MIT | Config + unit/component test patterns |
| playwright-e2e.md | [Playwright](https://github.com/microsoft/playwright) | 70k | Apache-2.0 | E2E config + auth flow tests |

## Error Handling Module (`error-handling/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| global-error-boundary.md | [Next.js Docs](https://nextjs.org/docs) | — | MIT | error.tsx + global-error.tsx patterns |
| api-error-handling.md | [Bulletproof React](https://github.com/alan2207/bulletproof-react) | 29k | MIT | ApiError class + withErrorHandling wrapper |

## Monitoring Module (`monitoring/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| health-check.md | [12-Factor App](https://12factor.net) | — | CC | Basic + deep health endpoints |

## Scheduling Module (`scheduling/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| cron-patterns.md | Vercel/Inngest/trigger.dev official docs | — | — | Platform comparison + code examples |

## Logging Module (`logging/`)

| File | Source Project | Stars | License | What We Extracted |
|------|---------------|-------|---------|------------------|
| structured-logging.md | [Pino](https://github.com/pinojs/pino) | 14k | MIT | Structured JSON logging + middleware |

---

## License Summary

All source projects use permissive licenses (MIT or Apache-2.0). Key obligations:

| License | Obligation |
|---------|------------|
| **MIT** | Include copyright notice + license text |
| **Apache-2.0** | Include copyright notice + license text + state changes |
| **CC-BY** | Give credit to original author |

This repository's code is original work inspired by and adapted from the above sources. Each file includes attribution to its source project.

---

*Last updated: 2026-04-16*
