---
name: capy
description: Use when orchestrating the full Spec-Driven Development pipeline for a feature end to end, coordinating writer, architect, planner, and builder skills in sequence with developer approval gates between each phase.
---

## Overview

The capy skill orchestrates the full SDD pipeline. It coordinates the four specialist skills — writer, architect, planner, builder — in the correct order, presents summaries to the developer after each phase, and enforces approval gates before proceeding.

Capy does not write specs, architecture, or code directly. It delegates every task to the appropriate skill and manages the pipeline flow. Its only jobs are: coordinate, summarize, gate, and iterate.

Each SDD phase runs in a dedicated subagent spawned via the Agent tool. Subagents receive an explicit prompt that instructs them to invoke the relevant skill (writer, architect, planner, or builder) via the Skill tool and return a structured result block. The subagent does the work in isolation; capy reads the result block and manages all approval gates, summaries, and pipeline decisions in the main conversation. Never perform phase work in the main thread — always spawn a subagent.

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
2. Spawn a writer subagent using the Agent tool:
   - `description`: "Write feature spec: [feature name]"
   - `prompt`: Writer Subagent Prompt (see Subagent Prompt Templates)
   - `run_in_background`: false

   Wait for the subagent to complete and extract its `## CAPY_RESULT` block.
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

5. Once the spec is approved, spawn an architect subagent using the Agent tool:
   - `description`: "Write architecture for: specs/[feature-name].md"
   - `prompt`: Architect Subagent Prompt (see Subagent Prompt Templates)
   - `run_in_background`: false

   Wait for the subagent to complete and extract its `## CAPY_RESULT` block.
6. Read the completed architecture section and present a summary:
   - DB changes (new tables or columns)
   - New API routes
   - New frontend files
   - Any `[DECISION]` items that need input

7. If there are `[DECISION]` items, surface them and wait for the developer's answers before continuing. Do not proceed with unresolved decisions.

---

### Phase 3 — Task List

8. Spawn a planner subagent using the Agent tool:
   - `description`: "Plan tasks for: specs/[feature-name].md"
   - `prompt`: Planner Subagent Prompt (see Subagent Prompt Templates)
   - `run_in_background`: false

   Wait for the subagent to complete and extract its `## CAPY_RESULT` block.
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
- Spawn a builder subagent using the Agent tool:
  - `description`: "Implement Task [N]: [task name]"
  - `prompt`: Builder Subagent Prompt (see Subagent Prompt Templates), with task number and name filled in
  - `run_in_background`: false

  Builder tasks run sequentially. Each requires developer approval before the next. Never background or parallelize builder subagents.
- After the subagent completes, extract its `## CAPY_RESULT` block and report what was built and which files changed
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

## Subagent Prompt Templates

Each subagent receives a self-contained prompt — it starts with no context from the main conversation. All subagents must end their response with a `## CAPY_RESULT` block so capy can parse the outcome.

---

### Writer Subagent Prompt

```
You are a spec writer running as a subagent in the capy SDD pipeline.

Feature request:
<FEATURE_REQUEST>

Instructions:
1. Use the Skill tool: Skill({ skill: "capy-crew-agents:writer", args: "<FEATURE_REQUEST>" })
2. The writer skill will explore the codebase, write the spec, and report its output.
3. End your response with:

## CAPY_RESULT
status: success
spec_path: specs/<feature-name>.md
user_story: <one-sentence user story>
acceptance_criteria_count: <N>
new_routes: <comma-separated list, or "none">
table_changes: <comma-separated list, or "none">
open_questions_count: <N>

On failure, end with:

## CAPY_RESULT
status: error
error: <what went wrong>
```

---

### Architect Subagent Prompt

```
You are a technical architect running as a subagent in the capy SDD pipeline.

Spec file: specs/<FEATURE_NAME>.md

Instructions:
1. Use the Skill tool: Skill({ skill: "capy-crew-agents:architect", args: "specs/<FEATURE_NAME>.md" })
2. The architect skill will read the spec, make architectural decisions, and append the Architecture section.
3. End your response with:

## CAPY_RESULT
status: success
spec_path: specs/<FEATURE_NAME>.md
db_changes: <comma-separated list, or "none">
new_routes: <comma-separated list, or "none">
new_frontend_files: <comma-separated list, or "none">
open_decisions_count: <N>
open_decisions: <each [DECISION] item on its own line, or "none">

On failure, end with:

## CAPY_RESULT
status: error
error: <what went wrong>
```

---

### Planner Subagent Prompt

```
You are a task planner running as a subagent in the capy SDD pipeline.

Spec file: specs/<FEATURE_NAME>.md

Instructions:
1. Use the Skill tool: Skill({ skill: "capy-crew-agents:planner", args: "specs/<FEATURE_NAME>.md" })
2. The planner skill will read the spec and architecture, generate the task list, and save it.
3. End your response with:

## CAPY_RESULT
status: success
tasks_path: specs/<FEATURE_NAME>-tasks.md
task_count: <N>
total_estimate: <Xh>
tasks:
  - Task 1: <name> (<size>)
  - Task 2: <name> (<size>)
split_items: <any [SPLIT] tasks, or "none">

On failure, end with:

## CAPY_RESULT
status: error
error: <what went wrong>
```

---

### Builder Subagent Prompt

```
You are a builder running as a subagent in the capy SDD pipeline.

Spec file: specs/<FEATURE_NAME>.md
Task list: specs/<FEATURE_NAME>-tasks.md
Task to implement: Task <N> — <TASK_NAME>

Instructions:
1. Use the Skill tool: Skill({ skill: "capy-crew-agents:builder", args: "Implement Task <N>: <TASK_NAME> from specs/<FEATURE_NAME>.md and specs/<FEATURE_NAME>-tasks.md" })
2. The builder skill will read the spec, read the task, implement it, and report. Do not implement any other tasks.
3. End your response with:

## CAPY_RESULT
status: success
task_number: <N>
task_name: <TASK_NAME>
files_changed:
  - path/to/file.ts — what was added or modified
done_conditions_met: <yes / no / partial>
suggested_commit: <commit message>
next_task: <Task N+1: name, or "none — this was the last task">
notes: <unrelated issues noticed but not changed, or "none">

On failure, end with:

## CAPY_RESULT
status: error
task_number: <N>
task_name: <TASK_NAME>
error: <what went wrong>
partial_files_changed:
  - <any partially modified files>
```

---

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

### Reading CAPY_RESULT Blocks

After each subagent completes, extract the `## CAPY_RESULT` block from its response and parse each `key: value` line. Use these values to populate the approval gate summary — do not re-read spec files in the main thread to compose summaries. If the result block is missing entirely, treat it as `status: error` and follow the Subagent Failure Protocol.

### Subagent Failure Protocol

If a subagent returns `status: error` or no `## CAPY_RESULT` block:

1. Do not proceed to the next phase.
2. Report the failure to the developer:

```
Phase [N] subagent failed.

Error: <error from result block, or "Subagent returned no result block">
Partial files changed: <list if any, or "none">

Options:
  🔁 Retry — I will re-spawn the subagent with the same prompt
  ✏️  Adjust — describe what to change before retrying
  🛑 Stop — end the pipeline here
```

3. On retry: re-spawn the same subagent with the identical prompt. Do not modify the prompt unless the developer explicitly provides adjustments.
4. On stop: follow the Stopping Mid-Pipeline protocol.

Never attempt to complete phase work in the main thread after a subagent failure.

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
| "I'll do the writer work in the main thread — it's faster" | The subagent isolation is the point. Always spawn. |
| "The CAPY_RESULT block is missing but I can see what the subagent did — I'll continue" | A missing result block means an unknown state. Follow the failure protocol. |

## Red Flags

- Proceeding to the next phase without explicit developer approval at the gates
- Producing spec content, architecture decisions, or code directly (instead of invoking the appropriate skill)
- Assuming `[DECISION]` items are resolved without explicit developer input
- Skipping the commit reminder between build tasks
- Continuing after the developer stops the pipeline without being asked to resume
- Performing phase work (spec writing, architecture, planning, implementation) in the main conversation thread instead of spawning a subagent
- Proceeding after a subagent returns `status: error` or no `## CAPY_RESULT` block
- Running builder subagents in background mode or in parallel
- Composing approval gate summaries from your own reading of spec files instead of from the subagent's CAPY_RESULT block

## Verification

- [ ] Spec exists at `specs/<feature-name>.md` with developer approval before proceeding to architecture
- [ ] Architecture section exists in the spec with all `[DECISION]` items resolved before proceeding to planning
- [ ] Task list exists at `specs/<feature-name>-tasks.md` with developer approval before starting implementation
- [ ] Each build task was followed by a commit reminder before the next task was started
- [ ] No spec content, architecture decisions, or code was produced directly by the orchestrator
- [ ] Each phase was executed by a dedicated subagent via the Agent tool, not in the main thread
- [ ] Each subagent returned a `## CAPY_RESULT` block with `status: success` before the approval gate was shown
- [ ] Builder subagents were spawned sequentially, not in parallel or in background
