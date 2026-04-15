# 🧪 Testing Patterns

> Unit testing with Vitest + E2E testing with Playwright for Next.js applications.

## Modules

| Module | What it does | Source | Stars |
|--------|-------------|--------|-------|
| [vitest-setup.md](./vitest-setup.md) | Vitest config, unit tests, component tests, API route tests | [Vitest](https://github.com/vitest-dev/vitest) | 14k |
| [playwright-e2e.md](./playwright-e2e.md) | Playwright config, auth flows, landing page tests, fixtures | [Playwright](https://github.com/microsoft/playwright) | 70k |

## Testing Strategy

⭐ **Recommended approach for Micro SaaS**:

| Layer | Tool | What to test | Time investment |
|-------|------|-------------|----------------|
| **Unit** | Vitest | Validation schemas, utility functions, business logic | Low |
| **Component** | Vitest + RTL | Critical UI components (forms, buttons) | Medium |
| **E2E** | Playwright | Auth flow, payment flow, core user journey | Medium |

**Don't over-test**: For a solo dev, focus on tests that catch real bugs:
1. Auth flow (login/signup/logout)
2. Payment flow (subscribe/cancel)
3. Core feature happy path

## Quick Start

```bash
# Unit tests
npm install -D vitest @vitejs/plugin-react @testing-library/react @testing-library/jest-dom jsdom
npm run test

# E2E tests
npm install -D @playwright/test
npx playwright install
npm run test:e2e
```
