---
name: e2e-with-playwright
description: Use when writing or updating Playwright E2E tests for actify-web, including authenticated flows and critical user paths.
---

## Overview

E2E tests are the only tests that verify the full stack works together — from browser interaction through the API to the database and back. In actify-web, Playwright handles E2E testing with Clerk authentication support via `@clerk/testing`.

E2E tests are expensive to write and maintain, so they should cover critical user paths only — not every edge case (that's for unit tests). The goal is confidence that the most important flows work before shipping.

## When to Use

**Use this skill when:**
- Implementing or modifying a critical user flow (auth, onboarding, core CRUD operations)
- A bug was found in production that a test would have caught
- Adding a new page or feature that represents a primary user journey

**Skip this skill when:**
- Testing purely visual changes
- Testing isolated component logic (use unit tests)
- Testing API endpoints in isolation (use API integration tests or curl)

## Core Process

1. **Identify the user story** — What flow are you testing? Write it as: "As a user, I [action], and I expect [outcome]."

2. **Identify auth requirements** — Does the test need an authenticated user? Use Clerk's testing helpers.

3. **Write the test file** — Place in the appropriate test directory. Follow the naming convention: `feature-name.spec.ts`.

4. **Set up test data** — Create the state needed before the test runs. Clean up after.

5. **Write assertions on outcomes, not implementation** — Assert on visible text, URL, network responses — not on internal state or DOM structure.

6. **Run the test** — `pnpm test:e2e` or `pnpm test:e2e:ui` for interactive mode.

7. **Add to smoke suite if critical** — Critical paths run on every deploy via `pnpm test:e2e:smoke`.

## Specific Techniques

### Authenticated Test Setup

Using `@clerk/testing` for Playwright:

```typescript
import { test, expect } from "@playwright/test";
import { clerk, setupClerkTestingToken } from "@clerk/testing/playwright";

test.describe("Feedback flow", () => {
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

  test("user can view feedback list", async ({ page }) => {
    await page.goto("/feedback");
    await expect(page.getByRole("heading", { name: "Feedback" })).toBeVisible();
  });
});
```

### Page Object Pattern

For complex pages, use page objects to avoid selector duplication:

```typescript
// tests/pages/feedback.page.ts
import { Page, Locator } from "@playwright/test";

export class FeedbackPage {
  readonly page: Page;
  readonly importButton: Locator;
  readonly feedbackTable: Locator;

  constructor(page: Page) {
    this.page = page;
    this.importButton = page.getByRole("button", { name: "Import CSV" });
    this.feedbackTable = page.getByRole("table");
  }

  async goto() {
    await this.page.goto("/feedback");
  }

  async importCSV(filePath: string) {
    await this.importButton.click();
    await this.page.getByLabel("Upload CSV").setInputFiles(filePath);
    await this.page.getByRole("button", { name: "Confirm Import" }).click();
  }
}
```

### Assertion Best Practices

```typescript
// ✅ Assert on visible outcomes
await expect(page.getByText("3 items imported")).toBeVisible();
await expect(page).toHaveURL("/dashboard");
await expect(page.getByRole("alert")).toContainText("Error");

// ❌ Don't assert on DOM structure
await expect(page.locator(".feedback-list > div:first-child")).toBeVisible();

// ✅ Wait for network to settle before asserting
await page.waitForResponse((res) => res.url().includes("/api/feedback"));
await expect(page.getByTestId("feedback-count")).toHaveText("5");
```

### Environment Variables for E2E

```env
# .env.local
E2E_USER_EMAIL=test@example.com
E2E_USER_PASSWORD=testpassword123
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...  # test key, not live
```

Never run E2E tests against the production Clerk instance or production database.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "This is covered by unit tests" | Unit tests don't test the browser, the API, or the network. E2E tests catch integration failures that unit tests miss. |
| "E2E tests are too slow" | Slow tests for critical paths are worth it. Use the smoke suite for speed-sensitive CI. |
| "I'll write the test after the feature ships" | Tests written after the fact test what was built, not what should work. Write them with the feature. |
| "This flow is too complex to automate" | Complex flows are exactly the ones most likely to break silently. They're worth the effort. |

## Red Flags

- Selectors using class names or CSS paths instead of roles and labels
- Tests that depend on execution order or shared state between tests
- Tests that never clean up their test data
- Assertions on internal component state instead of visible UI outcomes
- E2E tests running against the live Clerk instance or production database

## Verification

- [ ] Test covers a real user story, not an implementation detail
- [ ] Auth handled via `@clerk/testing` helpers
- [ ] Selectors use `getByRole`, `getByLabel`, `getByText` — not CSS selectors
- [ ] Each test independent — no shared state between tests
- [ ] Test data created and cleaned up within the test
- [ ] `pnpm test:e2e` passes locally
- [ ] Critical path tests added to smoke suite
