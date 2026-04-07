---
name: reviewer
description: Use when reviewing a PR or self-reviewing code before merging, focused on finding real problems — correctness, security, and maintainability — not style preferences.
---

## Overview

The reviewer skill conducts systematic code review with a fixed priority order: correctness first, security second, maintainability third, tests fourth, style last. It finds real problems, not opportunities to rewrite working code in a different style.

Every comment is prefixed with its severity so the developer knows what must be fixed before merge and what is optional. The reviewer never leaves a blocker disguised as a suggestion.

## When to Use

**Use this skill when:**
- Reviewing a PR before it is merged
- Self-reviewing your own code before opening a PR
- Looking for bugs, security issues, or maintainability problems in a specific file or route

**Skip this skill when:**
- You want style opinions on code that already works correctly and safely
- The code has not been written yet (use the writer or architect skill instead)

## Core Process

1. **Read the full diff** — Understand what changed and why before commenting on any individual line. Read the spec or PR description if available.

2. **Correctness pass** — Does this code do what it is supposed to do? Check edge cases, null handling, error paths, and race conditions. Can it fail silently?

3. **Security pass** — Check input validation, account scoping, auth middleware, secrets handling, and webhook verification. Any finding here is at least `[concern]` and likely `[blocker]`.

4. **Maintainability pass** — Is the code readable? Is it appropriately scoped? Is it unnecessarily complex for what it does?

5. **Tests pass** — Are the behaviors that matter actually tested? Do the tests assert on visible outcomes or internal state?

6. **Style pass** — Only flag style if it materially affects readability. Do not flag naming preferences or formatting as blockers.

7. **Write the review** — Use the severity tagging format. Group comments by file. State what the problem is, what the impact is, and what the fix is.

## Specific Techniques

### Severity Tags

Every comment gets one prefix:

| Tag | Meaning | Must fix before merge? |
|-----|---------|----------------------|
| `[blocker]` | Will cause a production bug, data leak, or security issue | Yes |
| `[concern]` | Should be discussed before merging — likely to cause problems | Discuss |
| `[nit]` | Optional improvement, low priority | No |
| `[question]` | Needs clarification — answer before deciding severity | Yes, answer it |

Never write a `[blocker]` as a suggestion. If it will break production, say so directly.

### Stack-Specific Checks

**API routes (Elysia):**
- Does every POST/PUT/PATCH have an Elysia body schema?
- Is `account_id` from `user.account_id` (JWT), not `body.account_id`?
- Does every error use `ApiError`, `NotFoundError`, or `UnauthorizedError` — not raw `Error` or `{ error: string }`?
- Is `supabaseAdmin` used in a route that also has `auth` middleware? (blocker — RLS bypass)

**Frontend (Next.js):**
- Is `"use client"` justified? Does the component actually need hooks or browser APIs?
- Is data fetching in a hook under `src/hooks/`, not inline in the component?
- Are toast notifications using `sonner`?
- Are analytics events tracked for the key user action?

**Auth (Clerk):**
- Is a new route that should be protected actually behind the auth middleware?
- Is a new public route (like `/onboarding`) added to `isPublicRoute` in `proxy.ts`?

**Tests:**
- Does the test assert on visible outcomes (text, URL, response status) — not DOM structure?
- Does the test use `getByRole`, `getByLabel`, or `getByText` — not CSS class selectors?
- Does the test clean up its own data?

### Review Comment Format

```
[blocker] src/routes/feedback.routes.ts:34
account_id is sourced from body.account_id instead of user.account_id.
Impact: any authenticated user can read another account's feedback by sending a different account_id.
Fix: replace body.account_id with user.account_id from the JWT context.
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "This is a style issue, not a correctness issue" | Security issues are often described as style first. Apply the security checklist before deciding. |
| "I'll leave it as a concern instead of a blocker to be polite" | If it will break production, it is a blocker. Being polite about a blocker wastes everyone's time. |
| "I should suggest how I would have written it" | Only if the current approach has a correctness, security, or maintainability problem. Otherwise, leave it. |
| "I'll also review the files outside the diff" | Review the diff. Expanding scope creates noise and buries the real findings. |

## Red Flags

- Blocker-severity findings written as `[concern]` or `[nit]` to avoid conflict
- Comments on naming, formatting, or personal style without a correctness justification
- Suggestions to abstract or refactor code that appears only once
- Comments on files outside the diff
- Security findings described as "potentially" or "might be an issue" — be direct

## Verification

- [ ] All `[blocker]` items are resolved before the PR is approved
- [ ] All `[question]` items have been answered
- [ ] Account scoping was explicitly checked for any route that reads or writes user data
- [ ] Auth middleware was checked for every new or modified route
- [ ] Every comment includes what the problem is, what the impact is, and what the fix is
