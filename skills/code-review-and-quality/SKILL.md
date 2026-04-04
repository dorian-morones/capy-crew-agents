---
name: code-review-and-quality
description: Use when reviewing a PR or self-reviewing code before merging to catch correctness, security, and maintainability issues.
---

## Overview

Code review is the last gate before bad code enters the main branch. A good review catches correctness bugs, security issues, and design problems — in that priority order. It's not about style preferences or proving you read the code. This skill encodes a five-dimension review framework that finds real problems without wasting time on noise.

## When to Use

**Use this skill when:**
- Reviewing a teammate's PR
- Self-reviewing your own code before requesting review
- Doing a pre-merge check on a long-running branch
- Reviewing AI-generated code before committing

**Skip this skill when:**
- Applying a trivial fix (typo, config value) — just ship it

## Core Process

1. **Read the PR description first** — Understand the intent before reading the code. Code that looks wrong might be intentional. Code that looks right might be solving the wrong problem.

2. **Check correctness** — Does the code do what the spec says? Edge cases? Error paths?

3. **Check security** — Does this code introduce a vulnerability? (See security-hardening skill)

4. **Check maintainability** — Will the next developer understand this? Is it over-engineered for the problem?

5. **Check tests** — Is the critical behavior tested? Is the test testing the implementation or the behavior?

6. **Leave actionable comments** — Prefix by severity. Ask questions before assuming bugs.

## Specific Techniques

### Five-Dimension Review Framework

Review in this order — stop when you find a blocker:

```
1. Correctness     — Does it solve the right problem? Are edge cases handled?
2. Security        — Input validation, auth scoping, no secret leaks
3. Maintainability — Readable, appropriately sized, no premature abstraction
4. Tests           — Covers the behavior, not just the happy path
5. Style           — Only flag if it's a real readability issue, not preference
```

Never bikeshed on style (1) if there's a correctness bug (5). Priority order matters.

### Comment Severity Prefixes

```
[blocker]  — Must fix before merge. Correctness bug or security issue.
[concern]  — Should discuss. Design decision that may cause problems.
[nit]      — Optional improvement. Low priority.
[question] — I don't understand this — explain or clarify.
```

Example comments:
```
[blocker] account_id is taken from body.account_id instead of user.account_id — 
this allows any authenticated user to read any account's data.

[concern] This function is 180 lines. The three branches inside feel like 
separate responsibilities — worth splitting?

[nit] Variable name `d` doesn't communicate intent. `feedbackItem` would be clearer.

[question] Why does this need to fetch the full list here instead of using 
the already-fetched items from the hook?
```

### Self-Review Checklist

Before requesting review, read your own diff as if you'd never seen it:

```markdown
- [ ] Does the implementation match the spec/ticket?
- [ ] Are all error paths handled?
- [ ] Is input validated at the API boundary?
- [ ] Are Supabase queries scoped to user.account_id?
- [ ] Is there a test for the critical behavior?
- [ ] Would I understand this code in 3 months?
- [ ] Are there any hardcoded values that should be config?
- [ ] Does this change affect any shared code (types, utils, middleware)?
```

### Common Review Targets in This Stack

| Area | What to look for |
|------|-----------------|
| API routes | Input schema, account_id from JWT not body, ApiError not raw Error |
| Supabase queries | Account scoping, no supabaseAdmin in user routes |
| React hooks | Missing deps in useEffect, loading/error states handled |
| Auth | Route in correct matcher (public vs. protected), correct middleware |
| Tests | Tests outcome not implementation, no CSS selectors, data cleaned up |

### PR Size

| Size | Lines changed | Approach |
|------|--------------|---------|
| Small | < 200 | Review all at once |
| Medium | 200-500 | Review file by file, check integration points |
| Large | > 500 | Request split unless it's a rename/move. If must review, focus on interfaces and data flow |

Large PRs obscure bugs. If a PR is too big to review properly, say so.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll just approve it, they know what they're doing" | Even good engineers make mistakes. A 5-minute review catches issues a 2-hour debugging session would find later. |
| "I don't want to be picky" | Leaving correctness bugs unmentioned to be polite causes production incidents. Be specific, not personal. |
| "The tests pass so it must be fine" | Tests pass when what was written is correct. They don't verify that the right thing was written. |
| "I'll add a comment but not block" | A non-blocking correctness bug gets merged. If it's a real issue, mark it `[blocker]`. |
| "Style issues don't matter" | Unreadable code accumulates. Mention them as `[nit]` — acknowledge them without blocking. |

## Red Flags

- PR description that doesn't explain the "why" — only the "what"
- Missing or incomplete error handling on external calls
- Supabase query without `.eq("account_id", user.account_id)`
- Test that asserts on a component's internal implementation
- New route that doesn't have Swagger detail or input schema
- Multiple unrelated changes in one PR (scope creep)
- "WIP" or "TODO" comments in code that's being merged to main

## Verification

- [ ] PR description read and intent understood before reviewing code
- [ ] Correctness checked — spec match, edge cases, error paths
- [ ] Security checked — input validation, auth scoping, no secrets
- [ ] Maintainability checked — readable, not over-engineered
- [ ] Tests present and testing behavior not implementation
- [ ] All `[blocker]` comments resolved before approving
- [ ] Comments are specific and actionable — not vague or preference-based
