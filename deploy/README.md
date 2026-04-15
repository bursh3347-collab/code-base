# 🚀 Deployment Patterns

> Production deployment configurations for Next.js applications.

## Modules

| Module | What it does | Source |
|--------|-------------|--------|
| [vercel-config.md](./vercel-config.md) | vercel.json, env vars (Zod), Cron Jobs, security headers | Vercel Docs + next-saas-starter 8.2k⭐ |
| [docker-compose.md](./docker-compose.md) | Multi-service Docker (app+db+redis), dev hot reload, prod multi-stage | FastAPI Template 42.6k⭐ (adapted) |
| [github-actions-ci.md](./github-actions-ci.md) | CI/CD pipeline (lint+typecheck+test+build), caching, Vercel deploy | Community best practices |

## Recommended Selection

⭐ **For most Micro SaaS projects**:
- Deploy on **Vercel** (zero-config, free tier generous)
- Use **GitHub Actions** for CI (lint + test gate)
- Use **Docker Compose** only for local development with Postgres/Redis

**When to use Docker in production**:
- Self-hosting (VPS, DigitalOcean, Railway)
- Need Postgres + Redis co-located
- Cost optimization at scale (Vercel gets expensive)

## Quick Start

```bash
# Vercel (recommended)
vercel deploy --prod

# Docker (local dev)
docker compose up

# Docker (production)
docker compose -f docker-compose.prod.yml up -d --build
```
