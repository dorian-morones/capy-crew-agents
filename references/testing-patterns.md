# Testing Patterns

Reference for Playwright E2E tests, Bun unit tests, and testing conventions in the actify stack.

---

## Test Levels

| Level | Tool | Location | Run command |
|-------|------|----------|-------------|
| E2E | Playwright + @clerk/testing | `tests/e2e/` | `pnpm test:e2e` |
| Component | — | — | (not currently used) |
| Unit | Bun test | `src/**/*.test.ts` | `bun test` |

---

## Playwright E2E Tests

### Setup

```typescript
// tests/e2e/[feature].spec.ts
import { test, expect } from "@playwright/test";
import { clerk, setupClerkTestingToken } from "@clerk/testing/playwright";

test.describe("[Feature]", () => {
  test.beforeEach(async ({ page }) => {
    await setupClerkTestingToken({ page });
    await clerk.signIn({
      page,
      signInParams: {
        strategy: "password",
        identifier: process.env.E2E_USER_EMAIL!,
        password: process.env.E2E_USER_PASSWORD!,
      },
    });
  });
});
```

### Selector Priority (best to worst)

```typescript
// ✅ Best — semantic and stable
page.getByRole("button", { name: "Import CSV" })
page.getByLabel("Email address")
page.getByText("3 items imported")
page.getByTestId("feedback-count")  // use sparingly

// ❌ Avoid — fragile, breaks on refactor
page.locator(".feedback-list > div:first-child")
page.locator("div[class*='button']")
```

### Assertion Patterns

```typescript
// Wait for network before asserting on data
await page.waitForResponse((res) => res.url().includes("/api/feedback"));
await expect(page.getByTestId("feedback-count")).toHaveText("5");

// Assert on visible outcomes
await expect(page.getByText("Feedback submitted")).toBeVisible();
await expect(page).toHaveURL("/dashboard");
await expect(page.getByRole("alert")).toContainText("Error");

// Wait for navigation
await page.click("text=Submit");
await page.waitForURL("/dashboard");
```

### Test Data Management

```typescript
test("user can delete feedback", async ({ page }) => {
  // Create test data via API before the test
  const res = await page.request.post("/api/feedback", {
    data: { content: "Test feedback for deletion" },
    headers: { Authorization: `Bearer ${token}` },
  });
  const { data: item } = await res.json();

  // Run test
  await page.goto("/feedback");
  await page.getByRole("button", { name: "Delete" }).first().click();
  await expect(page.getByText("Test feedback for deletion")).not.toBeVisible();

  // Cleanup is implicit — test data is in the test Clerk/Supabase instance
});
```

### Page Object Pattern (for complex pages)

```typescript
// tests/pages/feedback.page.ts
import { Page, Locator } from "@playwright/test";

export class FeedbackPage {
  readonly importButton: Locator;
  readonly feedbackTable: Locator;

  constructor(private page: Page) {
    this.importButton = page.getByRole("button", { name: "Import CSV" });
    this.feedbackTable = page.getByRole("table");
  }

  async goto() {
    await this.page.goto("/feedback");
  }
}
```

Use page objects only for pages with many interactions. Don't create them speculatively.

### Smoke Suite

Critical paths that run on every deploy:

```typescript
// playwright.config.ts
projects: [
  {
    name: "smoke",
    testMatch: "**/*.smoke.spec.ts",
    use: { baseURL: process.env.SMOKE_URL },
  },
]
```

Tag smoke tests explicitly: `auth.smoke.spec.ts`, `feedback.smoke.spec.ts`

---

## Environment Variables for E2E

```env
# .env.local (never commit actual values)
E2E_USER_EMAIL=test@example.com
E2E_USER_PASSWORD=testpassword123
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
```

Never run E2E tests against live Clerk or the production database.

---

## Bun Unit Tests

For pure functions and data transformations:

```typescript
// src/utils/format.test.ts
import { describe, it, expect } from "bun:test";
import { formatSentiment } from "./format";

describe("formatSentiment", () => {
  it("returns 'Positive' for positive", () => {
    expect(formatSentiment("positive")).toBe("Positive");
  });

  it("returns null for unknown values", () => {
    expect(formatSentiment("unknown")).toBeNull();
  });
});
```

---

## What to Test

| Worth testing | Not worth testing |
|--------------|------------------|
| Auth flows (sign in, onboarding) | Framework internals (Clerk auth itself) |
| Core CRUD (create/read/delete) | Visual-only changes |
| Import flows (CSV, sync) | Obvious UI (a button that links to a page) |
| Data transformation logic | Generated code (types, migrations) |
| Edge cases with null/empty values | One-liner utility functions |

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| CSS selector as locator | Use `getByRole`, `getByLabel`, `getByText` |
| Test depends on another test | Each test must set up and clean up its own state |
| Asserting on DOM structure | Assert on visible text, URL, or role |
| E2E test against live Clerk | Use test Clerk instance with `pk_test_*` |
| No `waitForResponse` before data assertion | API call may not have resolved yet |
