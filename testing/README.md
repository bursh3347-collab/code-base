# 🧪 Testing Patterns

> Extracted testing strategies, configurations, and patterns from high-star projects.

## 📦 What's Inside

| Framework | Type | Best For |
|-----------|------|----------|
| Vitest | Unit + Integration | Fast, Vite-compatible, TypeScript-first |
| Playwright | E2E | Cross-browser, reliable, CI-friendly |
| Testing Library | Component | React component testing |

## 🎯 Selection Guide

### ⭐ Recommended Default: Vitest + Playwright

**Why**: Vitest for unit/integration (fast, modern), Playwright for E2E (reliable, cross-browser). Both work great with Next.js.

## 📁 Directory Structure

```
testing/
├── README.md          ← You are here
├── vitest/            ← Vitest config + patterns
├── playwright/        ← Playwright setup + page objects
└── strategies/        ← Testing strategies (what to test, what not to)
```

---

*Status: 🟡 Scaffolded — Patterns will be populated as projects are analyzed.*
