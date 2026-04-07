---
name: architect
description: Use when an approved spec needs a technical architecture document covering DB schema, API route contracts, TypeScript types, component structure, and dependency order. No code written — only decisions documented.
---

## Overview

The architect skill takes an approved spec and produces every technical decision needed before implementation begins. It explores the existing codebase, matches its patterns, and documents one decisive choice per area — not options for the developer to pick from.

The architect's output is an `## Architecture` section appended to the spec file. It does not write source code. When a decision requires developer input that cannot be resolved by reading the codebase, it marks the item `[DECISION]` with a clear question and stops — it does not assume an answer.

## When to Use

**Use this skill when:**
- A spec has been reviewed and approved and is ready for technical design
- You need complete DB schema, route contracts, and TypeScript types before planning tasks
- The codebase patterns need to be understood before making architectural decisions

**Skip this skill when:**
- No spec exists yet (run the writer skill first)
- The spec has unresolved open questions (resolve them first)
- The architecture section already exists in the spec

## Core Process

1. **Read the spec** — Find and read `specs/<feature-name>.md` in full, including any open questions. Do not start exploring until you understand the full scope.

2. **Explore existing patterns** — Before making any decision, read the patterns that already exist in the areas the spec touches:
   - Existing routes (e.g., `src/routes/`) — file structure, auth middleware usage, error handling, body schema patterns
   - Existing migrations (e.g., `supabase/migrations/`) — naming conventions, RLS policy patterns, index conventions
   - Existing hooks (e.g., `src/hooks/`) — hook pattern: state, loading, error, callbacks
   - Existing components (e.g., `src/app/`, `src/components/`) — page structure, component organization
   - Auth middleware (e.g., `src/middleware/auth.ts`) — how `auth` vs `authLite` works
   - Error utilities (e.g., `src/utils/errors.ts`) — `ApiError`, `NotFoundError`, `UnauthorizedError`

3. **Make decisions** — For each area the spec touches, make one decisive architectural choice. No "option A or B" — pick one and state why it fits the existing patterns.

4. **Document the architecture** — Append an `## Architecture` section to `specs/<feature-name>.md` using the template in Specific Techniques.

5. **Flag decisions** — Mark any item that requires developer input with `[DECISION]:` and a clear question. Do not answer it yourself.

6. **Report** — Tell the developer the architecture has been appended, list any `[DECISION]` items that must be resolved before implementation, and note that the next step is the planner skill.

## Specific Techniques

### Architecture Section Template

```markdown
---

## Architecture

**Date:** <today>
**Status:** Draft

### Database Schema

#### New Tables

```sql
CREATE TABLE <table_name> (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
  <columns>,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ
);

ALTER TABLE <table_name> ENABLE ROW LEVEL SECURITY;

CREATE POLICY "account_select" ON <table_name>
  FOR SELECT USING ((auth.jwt() ->> 'account_id')::uuid = account_id);

CREATE POLICY "account_insert" ON <table_name>
  FOR INSERT WITH CHECK ((auth.jwt() ->> 'account_id')::uuid = account_id);

CREATE POLICY "account_update" ON <table_name>
  FOR UPDATE USING ((auth.jwt() ->> 'account_id')::uuid = account_id);

CREATE POLICY "account_delete" ON <table_name>
  FOR DELETE USING ((auth.jwt() ->> 'account_id')::uuid = account_id);

CREATE INDEX idx_<table>_account_id ON <table_name>(account_id);
```

#### Modified Tables

| Table | Migration | Reason |
|-------|-----------|--------|
| `existing_table` | `ADD COLUMN field TEXT` | Required by spec |

### API Routes

#### New Routes

| Method | Path | File | Auth | Body Schema | Response |
|--------|------|------|------|-------------|----------|
| POST | /resource | `src/routes/resource.routes.ts` | `auth` | `{ field: t.String() }` | `{ data: Resource }` |
| GET | /resource | `src/routes/resource.routes.ts` | `auth` | — | `{ data: Resource[] }` |

#### TypeScript Types

```typescript
// src/types/resource.ts
export interface Resource {
  id: string;
  account_id: string;
  field: string;
  created_at: string;
}

export interface CreateResourceRequest {
  field: string;
}
```

### Frontend

#### New Files

| File | Type | Reason |
|------|------|--------|
| `src/app/(app)/resource/page.tsx` | Client Component | Needs hooks and event handlers |
| `src/hooks/useResource.ts` | Hook | Data fetching for resource domain |
| `src/components/ResourceForm.tsx` | Component | Extracted — more than 50 lines, reusable |

#### Server vs. Client Decisions

| Component | Decision | Reason |
|-----------|----------|--------|
| `ResourcePage` | `"use client"` | Needs `useState`, event handlers |
| `ResourceList` | Server Component | Pure render, no interactivity |

### Dependency Order

Each item depends on the one above it:

1. DB migration (new table + RLS)
2. TypeScript types
3. API routes
4. Frontend hook
5. Frontend page and components
6. E2E test

### Open Architectural Decisions

- [ ] [DECISION]: <Question that requires developer input — be specific about what is being decided and what the options are>
```

### Auth Middleware Decision

| Route type | Middleware | Why |
|-----------|-----------|-----|
| User-facing protected route | `auth` | Verifies JWT + account membership |
| Onboarding route | `authLite` | Verifies JWT only, no account check |
| Webhook route | none | Verifies its own signature instead |
| Public route | none | No auth required |

### RLS Policy Pattern

Every new table follows the same four-policy pattern: SELECT, INSERT, UPDATE, DELETE — all scoped to `(auth.jwt() ->> 'account_id')::uuid = account_id`. Never skip a policy. Never use a single permissive policy.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll figure out the DB schema during implementation" | The planner needs it. The builder needs it. Decide it now. |
| "Option A or B both work" | Pick one. Document why. Undecided architecture produces inconsistent implementation. |
| "The dependency order is obvious" | Write it anyway. The planner uses it to generate the task sequence. |
| "I don't need to read existing migrations" | You will miss the naming convention, the RLS pattern, or the index convention. Read them. |

## Red Flags

- Architecture section says "option A or B" anywhere
- A new table is documented without RLS policies
- A route is documented without its body schema and response shape
- `[DECISION]` markers are answered inline instead of surfaced
- Dependency order is missing or out of sequence
- Existing patterns were not read before decisions were made

## Verification

- [ ] The DB schema is complete enough to write the migration SQL from it
- [ ] Every new route has a complete body schema and response shape
- [ ] Server vs. client is justified for every new component
- [ ] The dependency order matches the actual build sequence with no circular dependencies
- [ ] All `[DECISION]` items are explicit questions, not assumed answers
- [ ] Architecture section was appended to the spec file (not written to a separate file)
