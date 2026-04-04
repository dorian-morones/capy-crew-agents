---
name: feature-spec
description: Use when starting any non-trivial feature, fixing an ambiguous bug, or making changes that will touch more than one file or take more than 30 minutes.
---

## Overview

Writing a spec before writing code is the single highest-leverage habit in software development. A spec forces you to think through the problem before you're buried in implementation details, surfaces hidden complexity early, and creates a shared contract between you and your future self (or collaborators).

This skill encodes the process of turning a vague idea or ticket into a written specification that is unambiguous, testable, and scoped — before a single line of production code is written.

Without a spec, you write code to discover what you're building. With a spec, you build what you've already decided.

## When to Use

**Use this skill when:**
- Starting a new feature, endpoint, page, or database migration
- A bug fix requires changing more than one file
- The requirements are stated as outcomes ("users should be able to…") rather than implementation
- You find yourself asking "what exactly should this do?"
- The change will take more than 30 minutes

**Skip this skill when:**
- Fixing a single-line typo or copy change
- The change is completely self-contained and unambiguous (e.g. updating a config value)
- You are spiking/prototyping and plan to discard the code

## Core Process

1. **State the problem** — Write one paragraph: what user need or system gap does this address? Why now?

2. **Define done** — Write 3–5 testable success criteria in the form "Given X, when Y, then Z." These become your acceptance tests.

3. **Map the scope** — List every file, table, route, or component that will change. If you can't list them, you don't understand the scope yet.

4. **Identify dependencies** — What must exist before this can ship? (Auth, other endpoints, migrations, env vars)

5. **Define the API surface** — For routes: method, path, request shape, response shape, error cases. For components: props, events emitted, state owned. For DB: table name, columns, indexes, RLS policies.

6. **List edge cases** — What happens when the happy path fails? Empty states, auth failures, network errors, concurrent writes.

7. **Write the task list** — Break into discrete units, each completable in under 2 hours. Each task should have a single, verifiable output.

8. **Start implementation** — Only after steps 1–7 are written down.

## Specific Techniques

### Testable Success Criteria

Vague: "Users can upload a CSV"
Testable:
- Given a valid CSV with headers matching the schema, when uploaded, then all rows are imported and a count is returned
- Given a CSV with invalid headers, when uploaded, then a 400 error is returned with the list of missing fields
- Given an empty CSV, when uploaded, then a 400 error is returned

### API Surface Template

```
POST /feedback/import

Request:
  Content-Type: multipart/form-data
  Body: { file: File, source_id: string }

Response 200:
  { imported: number, skipped: number, errors: string[] }

Response 400:
  { error: "INVALID_HEADERS", missing: string[] }

Auth: Required (Bearer token via Clerk)
```

### DB Change Template

```
Table: feedback_imports
Columns:
  id          uuid PK default gen_random_uuid()
  account_id  uuid FK → accounts.id
  source_id   uuid FK → sources.id
  file_name   text
  row_count   int
  status      text CHECK IN ('pending','processing','done','failed')
  created_at  timestamptz default now()

Indexes: (account_id), (source_id)
RLS: SELECT WHERE account_id = auth.jwt()->>'account_id'
Migration file: 031_feedback_imports.sql
```

### Scope Mapping Checklist

Before writing code, confirm you've identified:
- [ ] New files to create
- [ ] Existing files to modify
- [ ] New DB tables or columns (migration needed)
- [ ] New env vars required
- [ ] New API routes
- [ ] New frontend routes or components
- [ ] Clerk webhook handlers affected
- [ ] Supabase RLS policies needed

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "It's simple, I don't need a spec" | Simple-looking changes have caused the worst regressions. The spec takes 10 minutes; the debugging takes hours. |
| "I'll figure it out as I go" | You will write the spec anyway — in bug reports and Slack messages after it ships broken. |
| "The ticket description is enough" | Ticket descriptions describe intent, not implementation. They don't map scope, surface errors cases, or define the API contract. |
| "I know this codebase well enough" | Familiarity creates blind spots. The spec surfaces what you're assuming, not what you know. |
| "We're moving fast" | Unspecced work moves fast until it doesn't. Re-work, missed edge cases, and scope creep are slower than 30 minutes of upfront writing. |

## Red Flags

- Opening an editor before finishing the spec
- A task list with items like "implement the feature" (not atomic enough)
- Success criteria that can't be turned into a test
- Scope that includes "and also while I'm in there…"
- A spec with no error cases listed
- Starting the next task before the current one is verified

## Verification

- [ ] Problem statement written in one paragraph — not a list of features
- [ ] 3+ testable success criteria written in Given/When/Then form
- [ ] All affected files listed
- [ ] API surface (request/response shape) documented for every new endpoint
- [ ] DB changes listed with migration file name
- [ ] Edge cases and error states listed
- [ ] Task list has items estimable under 2 hours each
- [ ] No production code written before this checklist is complete
