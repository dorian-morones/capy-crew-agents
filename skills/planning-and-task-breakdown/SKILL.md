---
name: planning-and-task-breakdown
description: Use when starting any feature or project to decompose work into independently shippable tasks before writing code.
---

## Overview

Planning before coding prevents the two most common failure modes: building the wrong thing and building the right thing in the wrong order. A good task breakdown produces a sequence of independently shippable increments — each one adds real value and leaves the codebase in a working state. This skill encodes the process for turning a feature spec into a concrete task list that can be executed without ambiguity.

## When to Use

**Use this skill when:**
- Starting a new feature from a spec or ticket
- Breaking down a large refactor
- Estimating scope before committing to a deadline
- A task feels undefined or "too big to start"

**Skip this skill when:**
- The task is a single, well-understood bug fix
- A change is purely cosmetic with no logic involved

## Core Process

1. **Read the spec or goal** — What is the user story? What is the acceptance criteria? If neither exists, write them before proceeding.

2. **Identify the layers** — Which layers does this feature touch? DB schema → API route → frontend hook → UI component → tests

3. **Find the dependencies** — Which tasks block which? DB migration must precede routes. Routes must precede hooks. Hooks must precede UI.

4. **Slice vertically, not horizontally** — Each task should be one thin end-to-end slice, not "all the DB work" then "all the API work."

5. **Write the task list** — Ordered, atomic tasks. Each task has: what to build, what layer it touches, how to verify it's done.

6. **Identify risks early** — Flag any task that requires a decision (data model choice, API shape, auth behavior) and make that decision before coding starts.

7. **Timebox** — Each task should be completable in ~2 hours. If not, split it.

## Specific Techniques

### Task List Format

```markdown
## Feature: [Name]

### Tasks

- [ ] DB: Add `source_type` column to `feedback` table (migration + RLS check)
- [ ] API: POST /feedback — validate body, insert with account_id from JWT
- [ ] API: GET /feedback — list with pagination, scoped to account
- [ ] Hook: useFeedback — fetch, create, error states
- [ ] UI: FeedbackList component — render items, loading, empty state
- [ ] UI: FeedbackForm component — controlled form, submit, toast on success
- [ ] E2E: User can submit feedback and see it in the list
```

### Dependency Graph

Before writing tasks, sketch the dependency chain:

```
DB schema
  └── API routes
        └── API client types
              └── useFeedback hook
                    └── FeedbackList page
                          └── E2E test
```

Tasks flow top-to-bottom. Never code a downstream task until the upstream one is done and committed.

### The "What Does Done Look Like?" Test

For every task, answer: "How do I know this task is complete?"

```markdown
❌ Vague: "Add feedback API"
✅ Specific: "POST /feedback returns 201 with created item. GET /feedback returns paginated list scoped to account. Both verified via Swagger UI."
```

### Risk Flagging

Mark tasks with `[DECISION]` when a choice must be made before implementation:

```markdown
- [DECISION] What is the pagination strategy — cursor or offset? → Choose before API task
- [ ] API: GET /feedback — implement chosen pagination strategy
```

Resolve all `[DECISION]` items before starting the coding sequence.

### Estimating Scope

| Signals | Likely scope |
|---------|-------------|
| 1-2 layers touched, familiar code | 1-3 tasks, half a day |
| 3-4 layers, one new pattern | 4-7 tasks, 1-2 days |
| New domain, data model changes | 8+ tasks, plan carefully |

If you can't enumerate the tasks, you don't understand the feature well enough to start coding.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll figure it out as I go" | Unplanned work compounds mid-execution. You'll make worse decisions under pressure. |
| "The spec is clear enough, I don't need to break it down" | A spec describes what to build. A task breakdown describes the order to build it in. They serve different purposes. |
| "Breaking it down takes time I don't have" | Not breaking it down costs more time later — rework, blocked teammates, bugs from wrong order. |
| "I'll just do the DB and API first, then the UI" | Horizontal slicing means nothing is shippable until everything is done. Vertical slices mean each task ships. |
| "This is just a small change" | "Small" changes that span multiple files and layers are medium changes. Treat them accordingly. |

## Red Flags

- Starting to code before identifying all affected layers
- Tasks like "add all the API routes" or "build the whole settings page"
- No acceptance criteria — no way to know when a task is done
- Tasks that can't be committed independently
- Unresolved data model or API shape decisions when coding begins
- "I'll clean this up later" on work that affects shared interfaces

## Verification

- [ ] User story and acceptance criteria written before task list
- [ ] All affected layers identified: DB, API, hooks, UI, tests
- [ ] Tasks ordered by dependency — no task starts before its upstream is done
- [ ] Each task is independently committable and verifiable
- [ ] Each task is estimable in ~2 hours or less
- [ ] All `[DECISION]` items resolved before coding begins
- [ ] Task list reviewed against the spec — nothing missed, nothing extra
