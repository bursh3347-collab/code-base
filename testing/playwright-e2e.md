# Playwright E2E Testing

> End-to-end testing: Playwright config, page object pattern, auth flow tests, visual regression.

**Source**: [Playwright](https://github.com/microsoft/playwright) (70k⭐, Apache-2.0)
**Stack**: TypeScript + Playwright + Next.js

---

## Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ["html", { open: "never" }],
    ["list"],
  ],
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
  },
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "mobile",
      use: { ...devices["iPhone 14"] },
    },
  ],
  webServer: {
    command: "npm run dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});
```

---

## Auth Flow Test

```typescript
// e2e/auth.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Authentication", () => {
  test("login page renders correctly", async ({ page }) => {
    await page.goto("/login");
    await expect(page.getByRole("heading", { name: /sign in/i })).toBeVisible();
    await expect(page.getByLabel(/email/i)).toBeVisible();
    await expect(page.getByLabel(/password/i)).toBeVisible();
  });

  test("shows error for invalid credentials", async ({ page }) => {
    await page.goto("/login");
    await page.getByLabel(/email/i).fill("wrong@example.com");
    await page.getByLabel(/password/i).fill("wrongpassword");
    await page.getByRole("button", { name: /sign in/i }).click();

    await expect(page.getByText(/invalid/i)).toBeVisible();
  });

  test("successful login redirects to dashboard", async ({ page }) => {
    await page.goto("/login");
    await page.getByLabel(/email/i).fill("test@example.com");
    await page.getByLabel(/password/i).fill("password123");
    await page.getByRole("button", { name: /sign in/i }).click();

    await expect(page).toHaveURL("/dashboard");
    await expect(page.getByText(/welcome/i)).toBeVisible();
  });

  test("unauthenticated user is redirected to login", async ({ page }) => {
    await page.goto("/dashboard");
    await expect(page).toHaveURL(/\/login/);
  });
});
```

---

## Landing Page Test

```typescript
// e2e/landing.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Landing Page", () => {
  test("has correct title and CTA", async ({ page }) => {
    await page.goto("/");
    await expect(page).toHaveTitle(/your app name/i);
    await expect(page.getByRole("link", { name: /get started/i })).toBeVisible();
  });

  test("pricing section shows plans", async ({ page }) => {
    await page.goto("/#pricing");
    await expect(page.getByText(/free/i)).toBeVisible();
    await expect(page.getByText(/pro/i)).toBeVisible();
  });

  test("responsive navigation works on mobile", async ({ page }) => {
    // Only runs in mobile project
    await page.goto("/");
    const menuButton = page.getByRole("button", { name: /menu/i });
    if (await menuButton.isVisible()) {
      await menuButton.click();
      await expect(page.getByRole("navigation")).toBeVisible();
    }
  });
});
```

---

## Authenticated Test Setup

```typescript
// e2e/fixtures.ts
import { test as base } from "@playwright/test";

// Extend test with authenticated page
export const test = base.extend<{ authenticatedPage: any }>({
  authenticatedPage: async ({ page }, use) => {
    // Login before test
    await page.goto("/login");
    await page.getByLabel(/email/i).fill("test@example.com");
    await page.getByLabel(/password/i).fill("password123");
    await page.getByRole("button", { name: /sign in/i }).click();
    await page.waitForURL("/dashboard");

    await use(page);
  },
});

export { expect } from "@playwright/test";

// Usage:
// import { test, expect } from "./fixtures";
// test("dashboard loads", async ({ authenticatedPage }) => { ... });
```

---

**Dependencies**:

```bash
npm install -D @playwright/test
npx playwright install
```

**Package.json**:

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:headed": "playwright test --headed"
  }
}
```
