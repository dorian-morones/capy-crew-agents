---
name: capy
description: Use when orchestrating the full Spec-Driven Development pipeline for a feature end to end, coordinating writer, architect, planner, builder, and reviewer skills in sequence with developer approval gates between each phase.
---

## Overview

The capy skill orchestrates the full SDD pipeline. It coordinates the specialist skills — writer, architect, planner, builder — in the correct order, presents summaries to the developer after each phase, and enforces approval gates before proceeding.

The pipeline runs: **conventions → spec → clarify → architecture → tasks → builder per task**. Capy spawns four kinds of subagent, one per phase. The builder is itself an orchestrator: inside it, a **coder** writes the task and a **reviewer** reviews the uncommitted diff, looping fix rounds until the review is clean. Capy never spawns a coder or a reviewer directly.

That nesting is what guarantees no code is committed unreviewed. The coder cannot commit, so the reviewer reads the working tree — review lands between writing and committing. A task comes back to capy already written, reviewed, and fixed.

Capy does not write specs, architecture, or code directly. It delegates every task to the appropriate skill and manages the pipeline flow. Its only jobs are: coordinate, summarize, gate, and iterate.

Each SDD phase runs in a dedicated subagent spawned via the Agent tool. Subagents receive an explicit prompt that instructs them to invoke the relevant skill (writer, architect, planner, or builder) via the Skill tool and return a structured result block. The subagent does the work in isolation; capy reads the result block and manages all approval gates, summaries, and pipeline decisions in the main conversation. Never perform phase work in the main thread — always spawn a subagent.

When the capy skill is active, adopt the orchestrator role for the entire session. Do not exit this role until the feature is complete or the developer explicitly stops the pipeline.

## When to Use

**Use this skill when:**
- Starting a new feature from scratch and want to run the full pipeline hands-off
- You want the complete spec → clarify → architecture → tasks → build/review/fix cycle managed for you
- You want approval gates enforced between each phase automatically
- You want every task reviewed and its findings fixed before you are asked to commit it

**Skip this skill when:**
- You only need one phase (e.g., just run the writer skill to write a spec)
- You have an existing spec and want to start from architect or planner
- You are delivering a single task and the spec and task list already exist (use builder directly)
- You only want code written, with no review loop (use coder directly)

## Core Process

### Phase 0 — Conventions

Before the first task of a feature, check whether `.capy/conventions.md` exists in the project.

If it does not, tell the developer and offer to create it:

```
No .capy/conventions.md found in this project.

The coder and reviewer use it to know what "correct" means here — stack, layout,
security invariants, verification commands. Without it, reviews are guesses.

✅ Create it — I will inspect the codebase and write it for your review (~2 min)
⏭️  Skip — the pipeline will infer patterns from neighbouring code instead
```

If they choose to create it, spawn a subagent with `subagent_type: "general-purpose"` instructing it to run `Skill({ skill: "capy-crew-agents:conventions" })`, then show the developer the result for confirmation before continuing.

This phase runs once per project, not once per feature. If the file already exists, say so in one line and move on.

---

### Phase 1 — Spec

1. Receive the feature request from the developer.
2. Spawn a writer subagent using the Agent tool:
   - `subagent_type`: "capy-crew-agents:writer"
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

### Phase 1b — Clarification

The spec is not approved while open questions remain. If `open_questions_count` is greater than zero, do not accept approval — surface the questions first:

```
The spec has [N] open questions that must be answered before architecture:

[Q1]: <question from the spec>
[Q2]: <question from the spec>

Answer each and I will fold them into the spec.
```

Once answered, re-spawn the writer subagent with the answers appended to the feature request so the spec is updated with resolved questions, then re-present the Phase 1 gate. A spec reaches architecture with `open_questions_count: 0` or it does not reach architecture at all.

Never answer an open question yourself, and never treat "use your judgment" as an answer to a question about product behavior — ask for the specific decision.

---

### Phase 2 — Architecture

5. Once the spec is approved, spawn an architect subagent using the Agent tool:
   - `subagent_type`: "capy-crew-agents:architect"
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
   - `subagent_type`: "capy-crew-agents:planner"
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

### Phase 4 — Implementation Loop

11. Once the task list is approved, iterate through each task. Capy spawns **one builder subagent per task**. The builder runs the write → review → fix loop internally and returns only when the task is reviewed clean, or when it needs a decision only the developer can make.

**Step 4a — Hand off the task**

Spawn a builder subagent using the Agent tool:
- `subagent_type`: "capy-crew-agents:builder"
- `description`: "Deliver Task [N]: [task name]"
- `prompt`: Builder Subagent Prompt (see Subagent Prompt Templates), with task number and name filled in
- `run_in_background`: false

Inside that subagent, the builder spawns a coder to write the code and a reviewer to review the uncommitted diff, and loops fix rounds between them. Capy does not see those rounds — it sees one result.

Builder tasks run sequentially. Never background or parallelize them: each task's reviewer must see a working tree containing only that task's changes.

**Step 4b — Read the outcome**

Extract the `## CAPY_RESULT` block and branch on `status`:

| `status` | What it means | Go to |
|----------|---------------|-------|
| `clean` | Written, reviewed, and clean | Step 4c |
| `needs_developer_decision` | Findings survived two fix rounds | Step 4d |
| `error` | A coder or reviewer subagent failed | Subagent Failure Protocol |

**Step 4c — Developer gate**

Report the outcome and ask before continuing:

```
Task [N] complete and reviewed clean (after [R] review round(s)).

Files changed: [list]

Review summary:
  [blocker] 0 remaining
  [concern] 0 remaining
  [nit] [N] — [list, or "none"]
  [question] [N] — [list, or "none"]

Commit when ready: git add ... && git commit -m "[message]"

Ready for Task [N+1]: [task name]? Or stop here?
```

Answer any `[question]` findings with the developer before moving on. If the developer wants a held `[nit]` fixed, re-spawn the builder for that task with the nit as the finding to address.

**Step 4d — Developer decision**

When the builder returns `status: needs_developer_decision`, apply the Review Round Cap protocol below. Do not re-spawn the builder for another round on your own initiative — the developer chooses.

12. Continue until all tasks are complete or the developer stops.

---

### Completion

When all tasks are done, report:

```
Pipeline complete.

Feature: <name>
Spec:     specs/<feature-name>.md
Tasks:    specs/<feature-name>-tasks.md

Completed (each reviewed before commit):
  ✅ Task 1: <name> — clean after <R> review round(s)
  ✅ Task 2: <name> — clean after <R> review round(s)
  ...

Findings the developer accepted without a fix:
  - <finding, or "none">

Suggested next steps:
- Run the full E2E test suite
- Open a PR with the feature branch
```

## Subagent Prompt Templates

Each subagent receives a self-contained prompt — it starts with no context from the main conversation. All subagents must end their response with a `## CAPY_RESULT` block so capy can parse the outcome.

---

### Writer Subagent Prompt

Spawn with `subagent_type: "capy-crew-agents:writer"`.

```
Feature request:
<FEATURE_REQUEST>

Use the Skill tool to run the writer skill: Skill({ skill: "capy-crew-agents:writer", args: "<FEATURE_REQUEST>" })

After the spec is complete, end your response with:

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

Spawn with `subagent_type: "capy-crew-agents:architect"`.

```
Spec file: specs/<FEATURE_NAME>.md

Use the Skill tool to run the architect skill: Skill({ skill: "capy-crew-agents:architect", args: "specs/<FEATURE_NAME>.md" })

After the architecture section is appended, end your response with:

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

Spawn with `subagent_type: "capy-crew-agents:planner"`.

```
Spec file: specs/<FEATURE_NAME>.md

Use the Skill tool to run the planner skill: Skill({ skill: "capy-crew-agents:planner", args: "specs/<FEATURE_NAME>.md" })

After the task list is saved, end your response with:

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

Spawn with `subagent_type: "capy-crew-agents:builder"`.

The builder is an orchestrator, not an implementer. It spawns a coder to write the task and a reviewer to review the uncommitted diff, and loops fix rounds between them. Capy never spawns coder or reviewer subagents directly — that is the builder's job.

```
Spec file: specs/<FEATURE_NAME>.md
Task list: specs/<FEATURE_NAME>-tasks.md
Task to deliver: Task <N> — <TASK_NAME>

Use the Skill tool to run the builder skill: Skill({ skill: "capy-crew-agents:builder", args: "Deliver Task <N>: <TASK_NAME> from specs/<FEATURE_NAME>.md and specs/<FEATURE_NAME>-tasks.md" })

Deliver only Task <N>. Spawn a coder to write it and a reviewer to review the uncommitted
diff, and loop fix rounds until the review is clean — at most two fix rounds. Do not write
or review code yourself. Do not commit. Do not answer [question] findings; return them.

End your response with one of:

## CAPY_RESULT
status: clean
task_number: <N>
task_name: <TASK_NAME>
review_rounds: <how many review rounds it took>
files_changed:
  - path/to/file.ts — what was added or modified
done_conditions_met: <yes / no>
build_check: <pass | fail>
suggested_commit: <commit message>
held_nits: <[nit] findings not auto-fixed, or "none">
held_questions: <[question] findings for the developer, or "none">
next_task: <Task N+1: name, or "none — this was the last task">
notes: <unrelated issues noticed but not changed, or "none">

## CAPY_RESULT
status: needs_developer_decision
task_number: <N>
task_name: <TASK_NAME>
review_rounds: 3
unresolved_findings:
  - [blocker] path/to/file.ts:34 — <problem> | Fix: <fix>
why_unresolved:
  - <reason the coder could not resolve it — spec conflict, missing decision, etc.>
files_changed:
  - path/to/file.ts — what was changed across all rounds
held_questions: <[question] findings, or "none">

## CAPY_RESULT
status: error
task_number: <N>
task_name: <TASK_NAME>
failed_subagent: <coder | reviewer>
failed_round: <R>
error: <what went wrong>
partial_files_changed:
  - <any partially modified files>
```

The coder and reviewer prompt templates live in the builder skill, not here. Capy does not need them.

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

### Review Round Cap

The builder enforces the cap of **two automatic fix rounds** internally — capy does not count rounds. When the builder returns `status: needs_developer_decision`, the cap has been reached and the decision is the developer's.

Do not re-spawn the builder for another round on your own initiative. Report:

```
Task [N] still has unresolved findings after 2 fix rounds.

Remaining:
  [blocker] path/to/file.ts:34 — <problem> | Fix: <fix>
  [concern] path/to/file.ts:88 — <problem> | Fix: <fix>

Why the coder could not resolve them:
  - <reason from why_unresolved>

Options:
  🔁 One more round — I will re-spawn the builder with your guidance
  ✏️  Adjust the spec — the finding conflicts with what the spec says
  ⏭️  Accept and commit — you have judged the remaining findings acceptable
  🛑 Stop — end the pipeline here
```

The cap exists because a finding surviving two rounds is usually a spec problem, not a code problem. Repeating the loop will not resolve it — a human decision will.

### Held Findings

The builder returns `held_questions` and `held_nits` rather than acting on them.

`[question]` findings are the developer's. A question means the reviewer could not determine severity without a product or architecture decision — surface each one at the gate and never answer it yourself.

`[nit]` findings are surfaced but not auto-fixed. Fixing nits inflates the diff and makes review harder to trust. If the developer wants one fixed, re-spawn the builder with that nit as the finding to address.

### Reading Builder Results

`status` is authoritative. Do not infer a different outcome from the surrounding prose, and do not present a task at the commit gate unless the builder returned `status: clean` — that status means a reviewer round returned `verdict: clean` inside the loop.

Do not re-grade the severity of returned findings, and do not decide a blocker "looks minor" and pass it through.

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
| "I'll spawn the coder directly and skip the builder — one less hop" | The builder is what pairs the coder with a reviewer. Skipping it ships unreviewed code. |
| "I'll spawn a reviewer myself to double-check the builder" | The review already happened inside the loop. Capy does not review. |
| "The builder returned `needs_developer_decision` but the fix looks easy — I'll just do it" | You never edit code. Give the developer the four options. |
| "This blocker is really more of a nit, I'll pass it through to the gate" | The reviewer set the severity inside the loop. You do not re-grade findings. |
| "The spec's open questions can be answered during implementation" | An unanswered question becomes a guess in the code. Resolve it in Phase 1b. |
| "A `[question]` came back held — the answer is obvious, I'll fill it in" | Questions are the developer's. That is what makes them questions. |
| "The builder hit the cap, one more round will get it" | Two rounds is the cap. A finding that survives it needs a human decision, not another round. |

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
- Presenting a task at the commit gate on anything other than `status: clean`
- Proceeding to architecture while `open_questions_count` is greater than zero
- Answering a spec open question or a held `[question]` finding on the developer's behalf
- Spawning coder or reviewer subagents directly — that is the builder's job, not capy's
- Re-grading the severity of findings returned by the builder
- Re-spawning the builder after `needs_developer_decision` without the developer choosing it
- Editing code directly to resolve a finding rather than re-spawning the builder

## Verification

- [ ] `.capy/conventions.md` exists, or the developer explicitly chose to skip it
- [ ] Spec exists at `specs/<feature-name>.md` with developer approval before proceeding to architecture
- [ ] All spec open questions were answered by the developer and folded into the spec before architecture began
- [ ] Architecture section exists in the spec with all `[DECISION]` items resolved before proceeding to planning
- [ ] Task list exists at `specs/<feature-name>-tasks.md` with developer approval before starting implementation
- [ ] Every task was delivered by a builder subagent that ran its own coder + reviewer loop
- [ ] Every task reaching the commit gate returned `status: clean`, or the developer explicitly accepted the remaining findings
- [ ] Held `[question]` findings were put to the developer, never answered by capy
- [ ] No coder or reviewer subagent was spawned directly by capy
- [ ] Each build task was followed by a commit reminder before the next task was started
- [ ] No spec content, architecture decisions, or code was produced directly by the orchestrator
- [ ] Each phase was executed by a dedicated subagent via the Agent tool, not in the main thread
- [ ] Each subagent returned a `## CAPY_RESULT` block with a success status before the approval gate was shown
- [ ] Builder subagents were spawned sequentially, not in parallel or in background
