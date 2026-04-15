# GitHub Actions CI/CD

> Complete CI/CD workflow: lint + type-check + test + build with caching strategy.

**Source**: Community best practices + [Next.js CI examples](https://github.com/vercel/next.js/tree/canary/examples)
**Stack**: GitHub Actions + Next.js + Vitest + Playwright

---

## Complete CI Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: $ github.workflow -$ github.ref 
  cancel-in-progress: true

jobs:
  lint-and-typecheck:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"

      - run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type Check
        run: npx tsc --noEmit

  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"

      - run: npm ci

      - name: Run Tests
        run: npm run test -- --coverage

      - name: Upload Coverage
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: [lint-and-typecheck]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"

      - run: npm ci

      - name: Install Playwright Browsers
        run: npx playwright install --with-deps chromium

      - name: Build
        run: npm run build
        env:
          DATABASE_URL: "postgresql://test:test@localhost:5432/test"

      - name: Run E2E Tests
        run: npx playwright test
        env:
          DATABASE_URL: "postgresql://test:test@localhost:5432/test"

      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint-and-typecheck, test]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "npm"

      - run: npm ci

      - name: Build
        run: npm run build
        env:
          DATABASE_URL: "postgresql://placeholder:placeholder@localhost:5432/placeholder"

      - name: Upload Build
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: .next/
          retention-days: 7
```

---

## Deployment Workflow (Vercel)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Vercel

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy Production
    runs-on: ubuntu-latest
    needs: []
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: $ secrets.VERCEL_TOKEN 
          vercel-org-id: $ secrets.VERCEL_ORG_ID 
          vercel-project-id: $ secrets.VERCEL_PROJECT_ID 
          vercel-args: "--prod"
```

---

## Package.json Scripts

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "vitest",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "typecheck": "tsc --noEmit"
  }
}
```

---

**Caching strategy**:
- `actions/setup-node` with `cache: "npm"` caches `node_modules` automatically
- `concurrency` cancels stale workflow runs on the same branch
- Jobs run in parallel where possible (lint + test), sequential where needed (build after test)

**Why this is good**: Catches errors before they hit production. Parallel jobs keep CI fast (~3-5 min). Artifact uploads enable debugging failed E2E tests.
