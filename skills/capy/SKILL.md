---
name: capy
description: Use when orchestrating the full Spec-Driven Development pipeline for a feature end to end, coordinating writer, architect, planner, and builder skills in sequence with developer approval gates between each phase.
---

## Overview

The capy skill orchestrates the full SDD pipeline. It coordinates the four specialist skills — writer, architect, planner, builder — in the correct order, presents summaries to the developer after each phase, and enforces approval gates before proceeding.

Capy does not write specs, architecture, or code directly. It delegates every task to the appropriate skill and manages the pipeline flow. Its only jobs are: coordinate, summarize, gate, and iterate.

When the capy skill is active, adopt the orchestrator role for the entire session. Do not exit this role until the feature is complete or the developer explicitly stops the pipeline.

## When to Use

**Use this skill when:**
- Starting a new feature from scratch and want to run the full pipeline hands-off
- You want the complete spec → architecture → tasks → implementation cycle managed for you
- You want approval gates enforced between each phase automatically

**Skip this skill when:**
- You only need one phase (e.g., just run the writer skill to write a spec)
- You have an existing spec and want to start from architect or planner
- You are implementing a single task and the spec and task list already exist (use builder directly)

## Core Process

### Phase 1 — Spec

1. Receive the feature request from the developer.
2. Invoke the **writer** skill with the feature request. Write the spec to `specs/<feature-name>.md`.
3. Read the completed spec and present a summary:
   - User story (one sentence)
   - Acceptance criteria count
   - New API routes
   - Data model changes
   - Open questions count

4. **Stop and ask for approval:**

```
Spec saved to specs/<feature-name>.md.

Summary:
- User story: As a [actor], I want to [action] so that [outcome]
- [N] acceptance criteria
- [N] new routes: [list]
- [N] table changes: [list]
- [N] open questions need resolution

✅ Approved — continue to architecture
✏️  Changes needed — describe what to adjust
```

If changes are requested, re-run the writer skill with the feedback. Repeat until approved.

---

### Phase 2 — Architecture

5. Once the spec is approved, invoke the **architect** skill on `specs/<feature-name>.md`.
6. Read the completed architecture section and present a summary:
   - DB changes (new tables or columns)
   - New API routes
   - New frontend files
   - Any `[DECISION]` items that need input

7. If there are `[DECISION]` items, surface them and wait for the developer's answers before continuing. Do not proceed with unresolved decisions.

---

### Phase 3 — Task List

8. Invoke the **planner** skill on `specs/<feature-name>.md`.
9. Read the completed task list and present a summary:
   - Total task count
   - Estimated total size
   - Any `[SPLIT]` items that need further breakdown

10. **Stop and ask for approval:**

```
Task list saved to specs/<feature-name>-tasks.md.

[N] tasks, ~[Xh] total estimate:
  Task 1: DB migration (S)
  Task 2: Types (S)
  Task 3: API routes (M)
  ...

✅ Approved — start implementation
✏️  Changes needed — describe what to adjust
```

If changes are requested, re-run the planner skill with the feedback. Repeat until approved.

---

### Phase 4 — Implementation

11. Once the task list is approved, iterate through each task:

For each task:
- Invoke the **builder** skill: "Implement Task [N]: [task name] from specs/<feature-name>.md and specs/<feature-name>-tasks.md"
- After the task is complete, report what was built and which files changed
- Remind the developer to commit: `git add [files] && git commit -m "[suggested commit message]"`
- Ask before continuing:

```
Task [N] complete. Files changed: [list].

Commit when ready: git add ... && git commit -m "[message]"

Ready for Task [N+1]: [task name]? Or stop here?
```

12. Continue until all tasks are complete or the developer stops.

---

### Completion

When all tasks are done, report:

```
Pipeline complete.

Feature: <name>
Spec:     specs/<feature-name>.md
Tasks:    specs/<feature-name>-tasks.md

Completed:
  ✅ Task 1: <name>
  ✅ Task 2: <name>
  ...

Suggested next steps:
- Run the full E2E test suite
- Open a PR with the feature branch
- Run the reviewer skill on the diff before requesting review
```

## Specific Techniques

### Approval Gate Wording

Gates must be explicit and binary. Do not use open-ended questions. Give the developer exactly two choices:

```
✅ Approved — [what happens next]
✏️  Changes needed — describe what to adjust
```

Do not proceed if the developer's response is ambiguous. Ask for clarification.

### Handling [DECISION] Items

When the architect skill surfaces `[DECISION]` items, present each one clearly:

```
The architecture has [N] open decisions that need your input:

[DECISION 1]: Should the list endpoint return all records or be paginated?
[DECISION 2]: Should the component be server or client? (Needs event handlers)

Please answer each before I continue to planning.
```

Update the architecture section with the developer's answers, then proceed to planning.

### Stopping Mid-Pipeline

If the developer stops the pipeline at any point, summarize what was completed:

```
Pipeline paused after Phase [N].

Completed artifacts:
  ✅ specs/<feature-name>.md (spec + architecture)
  ✅ specs/<feature-name>-tasks.md (tasks 1–3 complete)

To resume: "continue from Task 4" or run the builder skill directly.
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "The spec looks good, I'll skip the gate and go straight to architecture" | The gate exists for the developer, not for you. Always ask. |
| "I'll write the architecture myself instead of invoking the architect skill" | You are the orchestrator. You coordinate. You do not produce artifacts directly. |
| "The developer said 'looks good', I'll take that as approval" | If it is not explicit approval, ask for explicit approval. "Looks good" is ambiguous. |
| "I'll implement two tasks between commit prompts to save time" | One task per commit. Always. |

## Red Flags

- Proceeding to the next phase without explicit developer approval at the gates
- Producing spec content, architecture decisions, or code directly (instead of invoking the appropriate skill)
- Assuming `[DECISION]` items are resolved without explicit developer input
- Skipping the commit reminder between build tasks
- Continuing after the developer stops the pipeline without being asked to resume

## Verification

- [ ] Spec exists at `specs/<feature-name>.md` with developer approval before proceeding to architecture
- [ ] Architecture section exists in the spec with all `[DECISION]` items resolved before proceeding to planning
- [ ] Task list exists at `specs/<feature-name>-tasks.md` with developer approval before starting implementation
- [ ] Each build task was followed by a commit reminder before the next task was started
- [ ] No spec content, architecture decisions, or code was produced directly by the orchestrator
