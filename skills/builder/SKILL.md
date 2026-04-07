---
name: builder
description: Use when implementing a single task from an approved task list, strictly following the spec. Reads the spec and all files to be modified before writing any code, and builds only what the task describes.
---

## Overview

The builder skill implements one specific task from an approved task list — no more, no less. The spec is the source of truth. The builder does not add features, refactor unrelated code, or deviate from what the task describes. If something unrelated looks wrong, the builder notes it in the completion report and leaves it alone.

Before writing any code, the builder reads the spec, reads the task list, reads every file it will modify, and finds one or two similar existing files to match their patterns. Only after those four steps does it write code.

## When to Use

**Use this skill when:**
- An approved task list exists and you are ready to implement a specific task
- You need to implement exactly one layer (DB migration, types, route, hook, component, or test)
- The spec has an approved architecture section

**Skip this skill when:**
- No task list exists (run the planner skill first)
- The task is unclear or contradicts the spec — stop and ask before implementing
- You want to implement multiple tasks at once — implement one, commit, then run this skill again

## Core Process

1. **Read the spec** — Find and read `specs/<feature-name>.md` in full including the architecture section.

2. **Read the task list** — Find `specs/<feature-name>-tasks.md` and identify the specific task to implement. Read its "What to build" and "Done when" conditions carefully.

3. **Read every file you will modify** — No exceptions. Every existing file that will be changed must be read before it is edited. This is not optional.

4. **Find existing patterns** — Before creating a new file, find one or two similar existing files in the codebase and read them. Match their patterns exactly — naming, imports, error handling, structure.

5. **Implement the task** — Build exactly what the task specifies. If you notice something unrelated that could be improved, note it in the report and do not change it.

6. **Run a build check** — After implementing:
   - API: run `bun run src/index.ts` or equivalent to check for TypeScript/runtime errors
   - Frontend: run `pnpm tsc --noEmit` or `pnpm build` for type errors

7. **Report** — Tell the developer exactly what was built, what files changed, whether the done conditions are met, the suggested commit message, and what the next task is.

8. **Do not commit** — The developer reviews and commits. The builder only implements and reports.

## Specific Techniques

### Stack Conventions (Non-Negotiable)

**API Routes (Elysia):**
- Every POST/PUT/PATCH must have an Elysia body schema: `body: t.Object({ field: t.String() })`
- `account_id` always comes from `user.account_id` (from the JWT) — never from `body.account_id`
- All errors use `ApiError`, `NotFoundError`, or `UnauthorizedError` — never throw a raw `Error`
- New routes registered in `src/index.ts` after `csrfGuard`
- Webhook routes go before `csrfGuard`
- Every route has Swagger detail: `detail: { tags: ["TagName"], summary: "..." }`

**Database (Supabase):**
- Migrations named: `YYYYMMDDHHMMSS_description.sql`
- Every new table: `ALTER TABLE <name> ENABLE ROW LEVEL SECURITY`
- RLS policies use: `(auth.jwt() ->> 'account_id')::uuid = account_id`
- INSERT policies use `WITH CHECK`, SELECT/UPDATE/DELETE use `USING`
- Index on `account_id` for every new table

**Frontend (Next.js):**
- `"use client"` only when the component needs hooks, event handlers, or browser APIs
- Data fetching in hooks under `src/hooks/` — never inline in components
- All styles via Tailwind utility classes — no inline styles, no CSS modules
- Toast notifications: `import { toast } from "sonner"`
- Icons: `lucide-react` first choice
- Components over 50 lines or used in more than one place extracted to `src/components/`

**Auth (Clerk):**
- New public routes added to `isPublicRoute` in `src/proxy.ts`
- Protected routes use `auth` middleware
- Onboarding routes use `authLite`

### Completion Report Template

```
## Task Complete: <Task Name>

### Files changed
- `path/to/file.ts` — what was added or modified

### Done conditions
- [ ] <condition from task>
- [ ] <condition from task>

### Build check
- [ ] Compiles without errors

### Suggested commit
`<commit message from task list>`

### Next task
Task <N+1>: <name from task list>

### Notes
- <any unrelated issues noticed but not changed>
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll just fix this unrelated thing while I'm here" | No. Note it. Do not change it. One task per commit. |
| "I don't need to read the existing file first" | You will break the pattern. Read it first, every time. |
| "The task is clear, I don't need to re-read the spec" | The spec is the source of truth. Read it before every task. |
| "I'll implement two small tasks together" | One task per commit. Implement one, report, let the developer commit, then continue. |
| "The done conditions are obvious" | Read them. If a done condition is not met, the task is not complete. |

## Red Flags

- Adding features or refactoring code not mentioned in the task
- Using `body.account_id` instead of `user.account_id`
- Using `supabaseAdmin` in a route that has `auth` middleware
- Adding `"use client"` without a specific reason (hooks, event handlers, browser APIs)
- Using inline styles or non-Tailwind CSS
- Hardcoding secrets or API keys
- Logging sensitive data (tokens, passwords, PII)
- Committing instead of reporting and letting the developer commit
- Skipping the build check after implementing

## Verification

- [ ] The spec was read in full before writing any code
- [ ] Every file modified was read before being edited
- [ ] Existing patterns were matched (naming, imports, error handling)
- [ ] Only what the task describes was implemented — nothing more
- [ ] Build check passed with no TypeScript or runtime errors
- [ ] Completion report lists every file changed and all done conditions
- [ ] Commit was not made — the developer reviews first
