---
name: planner
description: Use when a spec with approved architecture needs to be broken into an ordered, atomic task list where each task is independently committable and at most two hours of work.
---

## Overview

The planner skill takes a spec with an approved architecture section and produces a precise, ordered task list. One task per layer. Each task independently committable. Each task with a specific file path, a clear done condition, and a suggested commit message.

The planner reads the dependency order from the architecture section and uses it as the task sequence. It does not invent its own order. It does not mix layers in a single task. It does not implement anything — it only plans.

## When to Use

**Use this skill when:**
- A spec has an approved architecture section and is ready for implementation planning
- You need to break a feature into tasks before handing off to the builder skill
- The architecture's dependency order needs to become concrete build steps

**Skip this skill when:**
- The spec does not exist or has not been reviewed
- The architecture section has unresolved `[DECISION]` items
- A task list already exists for the feature

## Core Process

1. **Read the spec** — Find and read `specs/<feature-name>.md` including the full Architecture section. Do not plan tasks for anything not in the spec.

2. **Extract the dependency order** — From the Architecture section's Dependency Order list. This is your task sequence. Do not reorder it.

3. **Slice into atomic tasks** — One task per logical layer change. Rules:
   - One layer per task (DB migration is one task, API route is a separate task — never combined)
   - Each task leaves the codebase in a buildable, committable state when complete
   - Each task is one to two hours maximum. If a task is larger, split it and mark with `[SPLIT]`
   - No task mixes concerns

4. **Write the task list** — Create `specs/<feature-name>-tasks.md` following the template in Specific Techniques.

5. **Report** — Tell the developer the task list has been saved, the total count and rough estimate, any `[SPLIT]` tasks that need further breakdown, and that the next step is the builder skill starting with Task 1.

## Specific Techniques

### Task List Template

```markdown
# Tasks: <Feature Name>

**Spec:** [specs/<feature-name>.md](specs/<feature-name>.md)
**Date:** <today>
**Status:** Draft

---

## Task List

### Task 1: DB — <what this creates or modifies>

**Layer:** Database
**File(s):** `supabase/migrations/<timestamp>_<description>.sql`
**Size:** S (< 1h)

**What to build:**
- Create table `<table_name>` with columns: `id uuid`, `account_id uuid`, `<field> text`, `created_at timestamptz`
- Enable RLS with standard account-scoped policies (SELECT, INSERT, UPDATE, DELETE)
- Add index on `account_id`

**Done when:**
- Migration file exists and applies cleanly
- RLS policies are in place and visible
- No existing tests broken

**Commit:** `feat(db): add <table_name> table with RLS`

---

### Task 2: Types — <what this defines>

**Layer:** TypeScript types
**File(s):** `src/types/<resource>.ts`
**Size:** S (< 1h)

**What to build:**
- Export `interface <Resource>` with fields matching the DB schema from Task 1
- Export `interface Create<Resource>Request` with required input fields

**Done when:**
- Types compile without errors
- Types match the DB schema from Task 1

**Commit:** `feat(types): add <Resource> interfaces`

---

### Task 3: API — <route description>

**Layer:** API route
**File(s):** `src/routes/<resource>.routes.ts`, `src/index.ts`
**Size:** M (1–2h)

**What to build:**
- `GET /<resource>` — returns list scoped to `user.account_id`
- `POST /<resource>` — validates body schema, inserts with `account_id` from JWT
- Register route in `src/index.ts` after `csrfGuard`
- Add Swagger tag

**Done when:**
- `GET /<resource>` returns 200 with list for authenticated user
- `POST /<resource>` returns 201 with created item
- `POST /<resource>` returns 422 for invalid body
- `GET /<resource>` returns 401 for unauthenticated request

**Commit:** `feat(api): add GET and POST /<resource> routes`

---

### Task 4: Hook — use<Resource>

**Layer:** Frontend data hook
**File(s):** `src/hooks/use<Resource>.ts`
**Size:** S (< 1h)

**What to build:**
- `fetchItems()` — GET list, sets `items`, `isLoading`, `error`
- `createItem(data)` — POST, appends to `items` on success, sets `error` on failure
- Return: `{ items, isLoading, error, fetchItems, createItem }`

**Done when:**
- Hook compiles without errors
- `fetchItems()` calls the GET route and handles errors
- `createItem()` calls the POST route and handles errors

**Commit:** `feat(hooks): add use<Resource> hook`

---

### Task 5: UI — <Component or Page name>

**Layer:** Frontend component
**File(s):** `src/app/(app)/<route>/page.tsx`, `src/components/<Component>.tsx`
**Size:** M (1–2h)

**What to build:**
- Page at the route specified in the architecture — `"use client"`, fetches on mount, renders list
- Extracted component if over 50 lines or reused
- Loading state, error state, empty state
- Toast on success/error using `sonner`

**Done when:**
- Page renders list from API
- Loading spinner shows while fetching
- Error message shows on failure
- Success toast shows on create

**Commit:** `feat(ui): add <Route> page with <Component>`

---

### Task 6: Tests — <what is tested>

**Layer:** E2E test
**File(s):** `tests/e2e/<feature>.spec.ts`
**Size:** M (1–2h)

**What to build:**
- Authenticated test using `@clerk/testing`
- Test: user can [main user story action] and see [expected outcome]
- Test is independent (creates its own data, cleans up after)

**Done when:**
- Test passes in isolation
- Test uses `getByRole`/`getByLabel`/`getByText` selectors — no CSS selectors
- Test is independent and does not share state with other tests

**Commit:** `test(e2e): add <feature> flow test`

---

## Summary

| # | Layer | Size | Commit |
|---|-------|------|--------|
| 1 | DB migration | S | `feat(db): ...` |
| 2 | TypeScript types | S | `feat(types): ...` |
| 3 | API routes | M | `feat(api): ...` |
| 4 | Frontend hook | S | `feat(hooks): ...` |
| 5 | UI components | M | `feat(ui): ...` |
| 6 | E2E tests | M | `test(e2e): ...` |

Total estimate: ~Xh
```

### Task Size Guidelines

| Size | Duration | When to use |
|------|----------|-------------|
| S | < 1h | Single file, clear pattern to follow (migration, types, hook) |
| M | 1–2h | Multiple related files, some decision-making (routes, components) |
| L | > 2h | **Not allowed** — mark `[SPLIT]` and break into two tasks |

### [SPLIT] Marker

When a task exceeds two hours, add `[SPLIT]` to the task name and describe how to break it:

```markdown
### Task 5: UI — Resource page [SPLIT]

This task should be split into:
- Task 5a: ResourceForm component
- Task 5b: ResourcePage that uses ResourceForm
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "DB + types can be one task" | They are different layers and different files. Keep them separate. |
| "Done when is obvious" | If it's obvious, it takes 30 seconds to write. If it's not written, it will be interpreted differently. |
| "I don't need the architecture section to plan" | You do. The dependency order comes from there. Do not invent it. |
| "This task is only 3 hours, close enough" | Split it. Three-hour tasks don't get finished in one session. |

## Red Flags

- Tasks that mention more than one layer (e.g., "add migration and API route")
- "Done when" conditions that aren't observable from outside the code (e.g., "function is written")
- Task order that doesn't match the architecture's dependency order
- Tasks that require a previous task to be partially complete (not fully committed) before starting
- No `[SPLIT]` marker on tasks estimated over two hours

## Verification

- [ ] Every task references specific file paths (not "add a route")
- [ ] Every "Done when" condition is observable and verifiable by someone else
- [ ] Task order matches the dependency order from the architecture section
- [ ] Each task is independently committable — no task requires another to be partially done
- [ ] Tasks over two hours are marked `[SPLIT]`
- [ ] No task mixes layers
- [ ] Task list saved to `specs/<feature-name>-tasks.md`
