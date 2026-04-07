---
name: code-test
description: Use when writing Playwright E2E tests or unit tests, focused on behavior coverage over line coverage — tests that would catch real bugs, not tests that mirror the implementation.
---

## Overview

The code-test skill writes tests that give real confidence. A test is valuable if it would catch a real bug. A test is worthless if it only passes because it mirrors the implementation. Before writing any test, the question is: "What behavior would break if this test didn't exist?"

The skill defaults to the highest test level that gives meaningful confidence for the behavior being tested. E2E for critical user flows. Integration for route and DB logic. Unit for pure functions and edge cases.

## When to Use

**Use this skill when:**
- Writing Playwright E2E tests for a user flow that needs authentication
- Writing integration tests for an API route or DB query
- Writing unit tests for data transformation functions, validation logic, or edge cases
- Reviewing existing tests that use CSS selectors, share state, or assert on internal implementation

**Skip this skill when:**
- The behavior has no observable outcome (pure internal state with no external effect)
- Testing framework behavior itself (Clerk, Supabase, Playwright internals)
- UI that is purely visual with no logic

## Core Process

1. **Choose the test level** — Use the test levels table to pick the right level. Default to the highest level that gives meaningful confidence for the behavior.

2. **Identify the behavior** — What observable outcome does the user or API caller experience? Write the test from that perspective, not from the implementation's perspective.

3. **Write a complete test** — Do not sketch an outline and leave blanks. Write the full test including setup, action, assertion, and cleanup.

4. **Verify independence** — The test must pass when run in isolation. It must not depend on another test having run first, and it must not leave state that breaks other tests.

5. **Use semantic selectors** — In E2E tests: `getByRole`, `getByLabel`, `getByText` in that priority order. Only use `data-testid` when semantic selectors are not possible. Never use CSS class selectors.

## Specific Techniques

### Test Levels

| Level | Tool | Use when |
|-------|------|----------|
| E2E | Playwright + @clerk/testing | Critical user flows end-to-end (sign in, core CRUD, import flows) |
| Integration | Bun test + real Supabase | API routes, DB queries, auth middleware behavior |
| Unit | Bun test | Pure functions, data transformations, edge cases with null/empty/boundary values |

Default to E2E for anything a user does. Default to unit for anything a function computes.

### E2E Test Template

```typescript
import { test, expect } from "@playwright/test";
import { clerk, setupClerkTestingToken } from "@clerk/testing/playwright";

test.describe("[Feature] flow", () => {
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

  test("user can [action] and see [expected outcome]", async ({ page }) => {
    // Navigate
    await page.goto("/[route]");

    // Action
    await page.getByRole("button", { name: "Create" }).click();
    await page.getByLabel("Field name").fill("value");
    await page.getByRole("button", { name: "Save" }).click();

    // Assert on visible outcome
    await expect(page.getByText("value")).toBeVisible();
    await expect(page).toHaveURL("/[expected-route]");
  });
});
```

### Selector Priority

Use selectors in this order. Stop at the first one that works.

1. `getByRole("button", { name: "Save" })` — semantic, matches what screen readers see
2. `getByLabel("Email address")` — matches form labels
3. `getByText("Success")` — matches visible text
4. `getByTestId("submit-btn")` — only when 1–3 are not possible

Never use: `.querySelector(".btn-primary")`, `$('.submit')`, or any CSS class selector.

### What to Test

**E2E coverage required:**
- Sign in and sign up flows
- Onboarding completion
- Core CRUD operations for the main domain object
- Import and sync flows

**Unit coverage required:**
- Data transformation functions
- Validation logic
- Edge cases: null inputs, empty arrays, boundary values, malformed data

**Do not test:**
- Framework behavior (Clerk auth itself, Supabase internals, Playwright mechanics)
- UI that is purely visual with no conditional logic
- API routes that are a direct passthrough to the DB with no logic

### Independence Checklist

- Test creates its own data in setup
- Test cleans up its data in teardown
- Test does not read state written by another test
- Test passes when run alone with `--grep "[test name]"`

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "CSS selectors are fine, they're stable in our codebase" | A class rename or refactor breaks the test with no behavior change. Use semantic selectors. |
| "I'll seed the database once and reuse across tests" | Shared state causes flaky tests. Each test creates its own data. |
| "This is covered by the unit test, E2E is redundant" | Unit tests miss integration failures. If it's a critical user flow, write the E2E. |
| "I'll outline the test and fill it in later" | Write the complete test now. Outlines sit unfinished indefinitely. |

## Red Flags

- Selectors using CSS class names, IDs, or `data-*` attributes that are not `data-testid`
- Tests that depend on another test having run first (shared state, ordered execution)
- Tests that hit the production Clerk instance or the live Supabase project
- Assertions on internal component state instead of visible output
- Tests that never clean up their data
- `test.only` or `test.skip` committed to the repository

## Verification

- [ ] Test passes in isolation (`--grep "[test name]"`)
- [ ] Selectors use `getByRole`, `getByLabel`, or `getByText` — no CSS class selectors
- [ ] Assertions are on visible, observable outcomes — not internal state
- [ ] Test creates its own data and cleans it up
- [ ] Test does not depend on another test having run first
- [ ] E2E test uses `@clerk/testing` for authentication — no hardcoded tokens
