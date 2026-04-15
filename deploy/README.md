# 🚀 Deployment Patterns

> Extracted deployment configurations, CI/CD pipelines, and infrastructure setups from high-star projects.

## 📦 What's Inside

| Platform | Source Project | Complexity | Best For |
|----------|---------------|------------|----------|
| Vercel | *TBD* | Low | Next.js apps, zero-config |
| Docker | *TBD* | Medium | Self-hosted, multi-service |
| GitHub Actions CI/CD | *TBD* | Medium | Automated testing + deploy |

## 🎯 Selection Guide

```
Next.js app?
├── Yes → Vercel (zero config, best DX)
└── No
    ├── Multiple services? → Docker Compose
    ├── Need CI/CD? → GitHub Actions
    └── Self-hosted? → Docker + Nginx
```

### ⭐ Recommended Default: Vercel

**Why**: Zero-config for Next.js, automatic previews, edge functions, great free tier. The standard for Micro SaaS.

## 📁 Directory Structure

```
deploy/
├── README.md          ← You are here
├── vercel/            ← Vercel config patterns
├── docker/            ← Dockerfile + docker-compose patterns
└── ci-cd/             ← GitHub Actions workflow templates
```

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
