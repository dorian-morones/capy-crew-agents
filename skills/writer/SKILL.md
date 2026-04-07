---
name: writer
description: Use when writing a formal spec for a new feature or fix before any code is touched. Produces a complete spec document covering user story, acceptance criteria, API surface, data model, UI changes, and open questions.
---

## Overview

The writer skill turns a feature request into an unambiguous spec document — the single source of truth for everything built after it. Without a spec, requirements live in someone's head and change mid-implementation. With a spec, every decision is made once, in writing, before any code is written.

The writer explores the existing codebase before writing anything. It reads existing routes, migrations, pages, and hooks to understand what already exists — then writes a spec that fits the project without duplicating or contradicting it.

The writer's output is always a spec document, never source code. If asked to suggest implementation details, the writer declines and notes that the architect skill handles technical decisions.

## When to Use

**Use this skill when:**
- Starting any non-trivial feature (more than one file or more than 30 minutes)
- Fixing an ambiguous bug where the correct behavior is unclear
- The feature request is a user story or outcome without technical direction
- A product decision needs to be locked down before implementation begins

**Skip this skill when:**
- The change is a typo fix or single-line correction
- The spec already exists and is approved
- You are implementing from an existing approved spec

## Core Process

1. **Read the feature request** — Understand the user story and the desired outcome. Do not start exploring or writing until you have a clear picture of what is being asked.

2. **Explore the existing codebase** — Before writing the spec, read the parts of the project that the feature will touch:
   - Existing routes (e.g., `src/routes/`) to understand what already exists
   - Existing DB tables (e.g., `supabase/migrations/`) to understand the current schema
   - Existing pages and hooks (e.g., `src/app/`, `src/hooks/`) to understand the UI layer
   - Use Glob and Grep to find related code by domain keyword

3. **Write the spec** — Create `specs/<feature-name>.md` following the template in Specific Techniques. Use kebab-case for the filename. Every section is required — write "N/A" only if a section genuinely does not apply.

4. **Flag open questions** — Anything that requires a product or technical decision that you cannot resolve by reading the codebase goes in Open Questions as a checkbox. Do not assume answers. Surface them.

5. **Report** — Tell the developer where the spec was saved, how many open questions need resolution, and that the next step is the architect skill.

## Specific Techniques

### Spec Document Template

```markdown
# Spec: <Feature Name>

**Status:** Draft
**Date:** <today>

---

## User Story

As a [user type], I want to [action] so that [outcome].

## Acceptance Criteria

- [ ] [Observable, testable outcome — visible to a user or verifiable in the API]
- [ ] [Each criterion is independent and binary: done or not done]

## API Surface

### New / Modified Routes

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST   | /resource | auth | Creates a resource |

### Request / Response Shapes

**POST /resource**

Request body:
```json
{
  "field": "string"
}
```

Response (201):
```json
{
  "data": { "id": "uuid", "field": "string", "created_at": "iso8601" }
}
```

## Data Model

### New Tables

| Table | Columns | Notes |
|-------|---------|-------|
| `table_name` | `id uuid`, `account_id uuid`, `field text`, `created_at timestamptz` | RLS required |

### Modified Tables

| Table | Change | Reason |
|-------|--------|--------|
| `existing_table` | Add `new_column text` | Required for X |

## UI Changes

### New Pages / Components

| Path / Component | Type | Description |
|-----------------|------|-------------|
| `/new-page` | page | Shows X to the user |
| `NewComponent` | component | Renders Y |

### Modified Pages / Components

| Component | Change |
|-----------|--------|
| `ExistingPage` | Add button to trigger new flow |

## Out of Scope

- [Thing explicitly NOT included in this feature]
- [Related feature that will be a separate spec]

## Open Questions

- [ ] [Decision needed — e.g. "Should X be paginated or return all records?"]
- [ ] [Dependency — e.g. "Does this require a migration before the route can be built?"]
```

### Writing Testable Acceptance Criteria

Vague criteria fail at review time. Every criterion must be binary (done or not) and observable by someone other than the author.

| Vague | Testable |
|-------|----------|
| "User can manage resources" | "User can create a resource and see it appear in the list without refreshing" |
| "API returns the data" | "GET /resource returns 200 with a JSON array scoped to the authenticated account" |
| "Error handling works" | "POST /resource with missing required fields returns 422 with a field-level error message" |

### Scoping the Out of Scope Section

Explicitly ruling things out prevents scope creep mid-implementation. Write Out of Scope items as:
- Features related to this domain that are NOT included
- Edge cases that will be handled in a later spec
- Integrations that are not part of this feature

If you cannot think of anything that is out of scope, the feature is probably not scoped tightly enough.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "This is simple, I don't need a spec" | Simple features grow. A spec takes 20 minutes. Rework takes days. |
| "I know what to build" | The spec is for the reviewer, the architect, and your future self — not for you right now. |
| "The acceptance criteria are implied" | If they're implied, they'll be interpreted differently by the person reviewing. Write them. |
| "I'll add the API shapes later" | The architect needs them. The planner needs them. Write them now. |

## Red Flags

- Acceptance criteria that describe implementation ("The function calls X") instead of outcome ("The user sees Y")
- Request or response shapes left as "TBD" or with ellipsis
- Open questions answered inline instead of surfaced as checkboxes
- Out of Scope section is empty on a feature with more than 2 acceptance criteria
- Spec written without exploring the existing codebase first

## Verification

- [ ] Every acceptance criterion is observable by someone other than the author
- [ ] Every new or modified route has a complete request body and response shape defined
- [ ] The data model is specific enough that a migration SQL could be written from it
- [ ] Out of scope items are explicit (not empty)
- [ ] All open questions are surfaced as checkboxes, not assumed
- [ ] The spec was saved to `specs/<feature-name>.md`
