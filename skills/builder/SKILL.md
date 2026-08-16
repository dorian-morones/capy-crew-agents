---
name: builder
description: Use when delivering one task from an approved task list end to end — spawning a coder to write it, a reviewer to review the uncommitted diff, and looping fix rounds until the review is clean or a developer decision is needed.
---

## Overview

The builder skill delivers one task from an approved task list, reviewed. It does not write code and it does not review code — it spawns a coder subagent to write, spawns a reviewer subagent to review the uncommitted diff, and loops fix rounds until the reviewer returns a clean verdict.

This is why no code reaches a commit unreviewed. The coder is forbidden from committing, so the reviewer reads the working tree — review happens *between* writing the code and committing it, not after. By the time the builder returns to capy, the task has been written, reviewed, and fixed.

The builder is itself a subagent, spawned by capy once per task. It cannot ask the developer anything. When the loop cannot resolve findings on its own, it stops and returns them upward for capy to put in front of the developer.

## When to Use

**Use this skill when:**
- An approved task list exists and one task needs to be delivered, reviewed, and ready to commit
- You want the write → review → fix loop run for a task without supervising each round

**Skip this skill when:**
- No task list exists (run the planner skill first)
- You only want the code written, with no review loop (use the coder skill directly)
- You only want an existing diff reviewed (use the reviewer skill directly)
- The task is unclear or contradicts the spec — stop and report before implementing

## Core Process

### Step 1 — Write

Spawn a coder subagent using the Agent tool:
- `subagent_type`: "capy-crew-agents:coder"
- `description`: "Write Task [N]: [task name]"
- `prompt`: Coder Prompt (see Subagent Prompts)
- `run_in_background`: false

Extract its `## CODER_RESULT` block and note the files it changed.

### Step 2 — Review

Set `review_round` to 1 and spawn a reviewer subagent:
- `subagent_type`: "capy-crew-agents:reviewer"
- `description`: "Review Task [N] (round [R])"
- `prompt`: Reviewer Prompt, with the changed files and round number filled in
- `run_in_background`: false

Extract its `## REVIEW_RESULT` block and read `verdict`:
- `verdict: clean` → go to Step 4
- `verdict: needs_fix` → go to Step 3

### Step 3 — Fix

If `review_round` is 3 or higher, do not fix — go to Step 5.

Otherwise spawn a coder subagent in fix mode:
- `subagent_type`: "capy-crew-agents:coder"
- `description`: "Fix Task [N] findings (round [R])"
- `prompt`: Coder Fix-Mode Prompt, with the reviewer's findings pasted in verbatim
- `run_in_background`: false

Pass only `[blocker]` and `[concern]` findings. Hold `[nit]` and `[question]` findings and return them in your final result — the coder never answers questions and never auto-fixes nits.

When it completes, increment `review_round` and return to Step 2. The coder's claim that findings are addressed is not the end of the loop; only a reviewer round with `verdict: clean` closes the task.

### Step 4 — Return clean

Return to capy with `status: clean`, the files changed, the review round count, and any held `[nit]` and `[question]` findings.

### Step 5 — Return for a developer decision

At the round cap, return `status: needs_developer_decision` with the remaining findings and the reasons the coder could not resolve them. Do not spawn another fix round. Do not fabricate an answer to an open question.

### Never

Do not commit. Do not write or edit code yourself. Do not review the diff yourself. You coordinate.

## Specific Techniques

### Review Round Cap

A task gets at most **two automatic fix rounds**. Track `review_round`, starting at 1.

| Round | Outcome |
|-------|---------|
| Review 1 | `clean` → return. `needs_fix` → fix round 1 |
| Review 2 | `clean` → return. `needs_fix` → fix round 2 |
| Review 3 | `clean` → return. `needs_fix` → **return for a developer decision** |

A finding that survives two fix rounds is usually a spec problem, not a code problem. More loops will not resolve it — a human decision will. Returning is not failure; it is the loop working correctly.

### You Cannot Ask, So Return

You are a subagent. There is no developer on the other end of your output — only capy, which parses your result block. Every question you have must leave as structured output:

- `[question]` findings → `held_questions` in your result
- Findings that conflict with the spec → `unresolved_findings` with the reason
- A task that contradicts the spec → return `status: error` with the contradiction

Never guess at a product decision to keep the loop moving. A guess becomes an unreviewed decision written into the code.

### Verdict Authority

`verdict` from the reviewer is authoritative. Do not compute your own verdict from finding counts, do not re-grade a `[blocker]` as a `[nit]`, and do not decide a finding "looks minor" and return clean anyway. If `verdict: needs_fix`, the loop continues.

### Subagent Failure

If a coder or reviewer subagent returns `status: error`, or returns no result block, do not continue the loop and do not attempt the work yourself. Return `status: error` to capy with which subagent failed, at which round, and any partially changed files. Capy handles retries.

## Subagent Prompts

Each subagent starts with no context. Every prompt must be self-contained.

### Coder Prompt

```
Spec file: specs/<FEATURE_NAME>.md
Task list: specs/<FEATURE_NAME>-tasks.md
Task to implement: Task <N> — <TASK_NAME>

Use the Skill tool to run the coder skill: Skill({ skill: "capy-crew-agents:coder", args: "Implement Task <N>: <TASK_NAME> from specs/<FEATURE_NAME>.md and specs/<FEATURE_NAME>-tasks.md" })

Implement only Task <N>. Read the spec and every file you will modify before editing. Run the build check. Do not commit.

End your response with:

## CODER_RESULT
status: success
task_number: <N>
files_changed:
  - path/to/file.ts — what was added or modified
done_conditions_met: <yes / no / partial>
build_check: <pass | fail>
suggested_commit: <commit message>
notes: <unrelated issues noticed but not changed, or "none">

On failure:

## CODER_RESULT
status: error
task_number: <N>
error: <what went wrong>
partial_files_changed:
  - <any partially modified files>
```

### Coder Fix-Mode Prompt

```
You are running in FIX MODE. Do not implement the task from scratch — it is already
implemented in the working tree. Your job is to apply the reviewer's findings to it.

Spec file: specs/<FEATURE_NAME>.md
Task list: specs/<FEATURE_NAME>-tasks.md
Task being fixed: Task <N> — <TASK_NAME>
Fix round: <R>

Reviewer findings to address (blockers and concerns only):
<FINDINGS_VERBATIM>

Use the Skill tool to run the coder skill: Skill({ skill: "capy-crew-agents:coder", args: "Fix mode: apply the reviewer findings for Task <N>: <TASK_NAME> from specs/<FEATURE_NAME>.md" })

Follow the coder skill's Fix Mode section:
- Read the current state of every file named in a finding before editing it
- Fix only what is listed above; do not modify files not named in a finding
- Every finding must end this round either addressed or explicitly reported as unaddressed
  with a reason — never silently dropped
- Do not answer [question] findings and do not change the spec to dissolve a finding
- Run the build check, re-verify the task's "Done when" conditions, and do not commit

End your response with:

## CODER_RESULT
status: success
task_number: <N>
fix_round: <R>
findings_addressed:
  - [blocker] path/to/file.ts:34 — <what changed>
findings_not_addressed:
  - [concern] path/to/file.ts:12 — <why not, and what decision it needs>
files_changed:
  - path/to/file.ts — <what was modified>
build_check: <pass | fail>
notes: <deviations from stated fixes, conflicts between findings, or "none">

On failure:

## CODER_RESULT
status: error
task_number: <N>
fix_round: <R>
error: <what went wrong>
partial_files_changed:
  - <any partially modified files>
```

### Reviewer Prompt

```
Spec file: specs/<FEATURE_NAME>.md
Task list: specs/<FEATURE_NAME>-tasks.md
Task under review: Task <N> — <TASK_NAME>
Review round: <R>
Files changed:
<FILES_CHANGED>

Review the uncommitted diff only (`git diff` plus untracked files from `git status`). Do not review files outside this task's changes.

Use the Skill tool to run the reviewer skill: Skill({ skill: "capy-crew-agents:reviewer", args: "Review the uncommitted diff for Task <N>: <TASK_NAME> against specs/<FEATURE_NAME>.md" })

Check the task's "Done when" conditions as part of the review — an unmet done condition is a [blocker].

On round 2 or later, state for each previously raised finding whether it is now resolved, and do not raise new [nit] findings.

End your response with:

## REVIEW_RESULT
status: success
task_number: <N>
review_round: <R>
verdict: <clean | needs_fix>
blockers: <N>
concerns: <N>
nits: <N>
questions: <N>
done_conditions_met: <yes / no>
findings:
  - [blocker] path/to/file.ts:34 — <problem> | Impact: <impact> | Fix: <fix>
previously_raised_resolved: <findings from earlier rounds now resolved, or "n/a">

`verdict: clean` requires blockers: 0 and concerns: 0. Nits and questions do not block.

On failure:

## REVIEW_RESULT
status: error
task_number: <N>
review_round: <R>
error: <what went wrong>
```

## Result Block

End your own response to capy with exactly one of these.

**Clean:**

```
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
```

**Needs a developer decision:**

```
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
```

**Error:**

```
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

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "The change is small, I'll just write it myself instead of spawning a coder" | You coordinate. The isolation is the point — a builder that writes code also reviews its own work. |
| "I can see the diff is fine, I'll skip the review round" | Every task gets reviewed before it goes back to capy. No exceptions. |
| "The coder said it addressed everything, so the task is clean" | Only a reviewer round with `verdict: clean` closes a task. |
| "This blocker is really a nit, I'll return clean" | The reviewer sets severity. You do not re-grade findings. |
| "Round 3 still has blockers, one more round will get it" | Two rounds is the cap. Return it. |
| "I'll answer the `[question]` finding so the loop can finish" | You cannot ask the developer, which is exactly why you must not answer for them. Return it. |
| "The reviewer subagent failed, I'll just review it myself" | Return `status: error`. Capy handles retries. |

## Red Flags

- Writing or editing code yourself instead of spawning a coder
- Reviewing the diff yourself instead of spawning a reviewer
- Returning `status: clean` without a reviewer round returning `verdict: clean`
- Passing `[question]` or `[nit]` findings into a fix-mode prompt
- Re-grading a reviewer's severity tags, or computing your own verdict from finding counts
- Spawning a third fix round instead of returning for a developer decision
- Sending a fix-mode prompt without the `You are running in FIX MODE` preamble, causing the coder to re-implement the task
- Committing
- Continuing the loop after a subagent returns `status: error` or no result block
- Running coder and reviewer subagents in parallel or in background — the reviewer must see the coder's finished work

## Verification

- [ ] The code was written by a coder subagent, not by this skill
- [ ] The diff was reviewed by a reviewer subagent on the uncommitted working tree
- [ ] `status: clean` was returned only after a reviewer round with `verdict: clean`
- [ ] Fix-mode prompts carried only `[blocker]` and `[concern]` findings
- [ ] No more than two fix rounds ran before returning for a developer decision
- [ ] `[question]` findings were returned to capy unanswered
- [ ] No commit was made
