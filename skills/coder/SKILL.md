---
name: coder
description: Use when writing the code for a single task from an approved task list, or when applying reviewer findings to that task in fix mode. Reads the spec and every file to be modified before writing any code.
---

## Overview

The coder skill writes the code for one specific task from an approved task list — no more, no less. The spec is the source of truth. The coder does not add features, refactor unrelated code, or deviate from what the task describes. If something unrelated looks wrong, the coder notes it in the completion report and leaves it alone.

Before writing any code, the coder reads the spec, reads the task list, reads every file it will modify, and finds one or two similar existing files to match their patterns. Only after those four steps does it write code.

The coder has a second mode. In **fix mode** the task is already implemented and a reviewer has filed findings against it; the coder applies those findings and nothing else. Both modes end the same way: a report, a passing build, and no commit.

The coder is spawned by the builder, which owns the build → review → fix loop. The coder never decides whether its own work is finished — the reviewer does.

## When to Use

**Use this skill when:**
- An approved task list exists and a specific task needs to be implemented
- A reviewer has filed findings against a task and they need to be applied (fix mode)
- You need to implement exactly one layer (DB migration, types, route, hook, component, or test)

**Skip this skill when:**
- No task list exists (run the planner skill first)
- The task is unclear or contradicts the spec — stop and ask before implementing
- You want to implement multiple tasks at once — implement one, report, then run this skill again
- You need the full loop coordinated for a task (use the builder skill, which spawns this one)

## Core Process

0. **Read `.capy/conventions.md`** — If it exists, it defines what correct looks like in this codebase. If it does not, derive conventions from neighbouring code in step 4 and say so in your report.

1. **Read the spec** — Find and read `specs/<feature-name>.md` in full including the architecture section.

2. **Read the task list** — Find `specs/<feature-name>-tasks.md` and identify the specific task to implement. Read its "What to build" and "Done when" conditions carefully.

3. **Read every file you will modify** — No exceptions. Every existing file that will be changed must be read before it is edited. This is not optional.

4. **Find existing patterns** — Before creating a new file, find one or two similar existing files in the codebase and read them. Match their patterns exactly — naming, imports, error handling, structure.

5. **Implement the task** — Build exactly what the task specifies. If you notice something unrelated that could be improved, note it in the report and do not change it.

6. **Run the build check** — Run the verification commands from `.capy/conventions.md` (typecheck and build at minimum). If there is no conventions file, take them from the project's `package.json` scripts, `Makefile`, or CI config.

7. **Report** — State exactly what was built, what files changed, whether the done conditions are met, the suggested commit message, and what the next task is.

8. **Do not commit** — The developer reviews and commits. The coder only implements and reports.

## Specific Techniques

### Fix Mode

A prompt that opens with `You are running in FIX MODE` means the task is already implemented in the working tree and a reviewer has filed findings against it. Do not build it again — apply the findings.

In fix mode, the review is the task list. Each `[blocker]` and `[concern]` is one item:

1. **Read the findings** — severity, `file:line`, the problem, the stated fix. A finding without a concrete fix is not actionable; flag it rather than guessing.
2. **Read the spec and the task's "Done when" conditions** — fix mode does not skip step 1 of the core process.
3. **Read the current state of every flagged file, in full** — the working tree has changed since the reviewer read it, especially on a second fix round. Never edit from the findings alone.
4. **Apply the fixes in severity order** — blockers first, then concerns. Apply the stated fix unless it is wrong; if it is wrong, apply the correct one and record the deviation.
5. **Re-verify the done conditions** — a fix that resolves a finding but breaks a done condition has traded one blocker for another.
6. **Run the build check** — the project's own verification commands. A fix round that leaves the build broken is worse than no fix at all.
7. **Report the disposition of every finding.**
8. **Do not commit.**

**Finding disposition.** Every finding ends the round in exactly one state:

| Disposition | When | What to report |
|-------------|------|----------------|
| Addressed | The fix was applied and the build passes | File, line, what changed |
| Addressed with deviation | The stated fix was wrong; a different fix was applied | What was applied instead, and why |
| Not addressed | The fix requires a spec or architecture change, or two findings conflict | The exact reason, so the developer can decide |

A finding that is silently dropped will be raised again on the next review round, and the loop will not converge. There is no "partially addressed" — say which part remains, and that part is unaddressed.

**Scope in fix mode is tighter than in a build round.** Do not modify a file that is not named in a finding. While fixing `src/routes/feedback.routes.ts:34` you will see other things worth improving in that file — leave them. The reviewer reviewed a specific diff; widening it means the next round reviews changes nobody asked for, and the loop grows instead of converging. The only exception: a `[nit]` that is a one-line change inside a block you are already editing.

**What you never fix:** `[question]` findings. A question means the reviewer could not decide severity without a product or architecture decision. That decision is the developer's. Never answer one, and never change the spec to make a finding go away — if a finding conflicts with the spec, report it unaddressed with the conflict spelled out.

**Conflicting findings.** When two findings cannot both be satisfied, resolve the higher-severity one, leave the other unaddressed, and report the conflict. Do not invent a compromise that satisfies neither.

### Project Conventions

Correctness is defined by this codebase, not by framework documentation. Before writing code:

1. **Read `.capy/conventions.md`** if it exists. It is the authority on stack, layout, security invariants, naming, and the verification commands to run. Follow it over your own preferences and over any general best practice.
2. **If it does not exist**, derive conventions from the code itself: open two or three files in the same layer as the one you are about to touch and match the repeated pattern — error handling, input validation, how auth context is obtained, naming, imports, file structure. Note in your report that no conventions file was found, so the developer can run the `conventions` skill.

Never introduce a pattern the codebase does not already use. A task is not an opportunity to upgrade the project's idioms — if the existing pattern is wrong, note it and leave it.

### Security Invariants Are Non-Negotiable

Whatever `.capy/conventions.md` records under Security Invariants, treat as a hard constraint — most often a tenancy rule (which identity field scopes queries, and where it comes from) and a set of bypasses (admin or service-role clients that skip access control).

Two rules hold in every codebase regardless of stack:

- **Identity comes from the verified session or token, never from request input.** A tenant/account/user identifier read out of a request body or query parameter is an access-control hole even when the endpoint is authenticated.
- **Never hardcode secrets, and never log credentials, tokens, or personal data.**

If the conventions file is missing its security section, do not guess the tenancy rule from one example. Implement what the task specifies, and flag the gap in your report.

### Verification

Run the verification commands from `.capy/conventions.md`. If there is no conventions file, find them in `package.json` scripts, `Makefile`, or CI config — typecheck and build at minimum. Do not report success on a broken build, and do not invent a command that is not in the project.

### Build Report Template

```
## Task Complete: <Task Name>

### Files changed
- `path/to/file.ts` — what was added or modified

### Done conditions
- [ ] <condition from task>

### Build check
- [ ] Compiles without errors

### Suggested commit
`<commit message from task list>`

### Notes
- <any unrelated issues noticed but not changed>
```

### Fix Report Template

```
## Fix Round <R> Complete: <Task Name>

### Findings addressed
- [blocker] path/to/file.ts:34 — <what changed>

### Findings not addressed
- [concern] path/to/other.ts:12 — <why: spec conflict, needs developer decision>

### Files changed
- `path/to/file.ts` — <what was modified>

### Done conditions
- [ ] <condition from task — still holds>

### Build check
- [ ] Compiles without errors

### Notes
- <deviations from stated fixes, conflicts between findings, or "none">
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll just fix this unrelated thing while I'm here" | No. Note it. Do not change it. One task per commit. |
| "I don't need to read the existing file first" | You will break the pattern. Read it first, every time. |
| "The task is clear, I don't need to re-read the spec" | The spec is the source of truth. Read it before every task. |
| "I'll implement two small tasks together" | One task per commit. Implement one, report, and let the builder run the review loop. |
| "The done conditions are obvious" | Read them. If a done condition is not met, the task is not complete. |
| "In fix mode I already know this file — I wrote it" | You are a fresh subagent. The working tree has changed. Read it again. |
| "This concern is minor, I'll skip it silently" | Skipped silently means raised again next round. Report it as unaddressed with a reason. |
| "The reviewer's suggested fix is wrong, so I'll ignore the finding" | The finding is still valid. Apply the correct fix and record the deviation. |
| "I'll adjust the spec so the finding no longer applies" | The spec is the source of truth. Escalate the conflict instead. |
| "My fixes are correct, so the task is done" | The reviewer decides that, not you. Report and let the loop re-review. |

## Red Flags

- Adding features or refactoring code not mentioned in the task
- Taking a tenant, account, or user identifier from request input instead of the verified session or token
- Using an admin or service-role client in a path that already enforces user-scoped access
- Introducing a library, pattern, or idiom the codebase does not already use
- Ignoring `.capy/conventions.md` in favour of framework defaults or personal preference
- Hardcoding secrets or API keys
- Logging sensitive data (tokens, passwords, PII)
- Committing instead of reporting
- Skipping the build check after implementing
- In fix mode: re-implementing the task instead of applying the findings
- In fix mode: changing a file not named in any finding
- In fix mode: a finding that appears in neither the addressed nor the unaddressed list
- In fix mode: answering a `[question]` finding, or editing the spec to dissolve a finding

## Verification

- [ ] The spec was read in full before writing any code
- [ ] Every file modified was read before being edited
- [ ] `.capy/conventions.md` was read and followed, or its absence was reported
- [ ] Existing patterns were matched (naming, imports, error handling)
- [ ] Only what the task describes was implemented — nothing more
- [ ] The project's own verification commands were run and passed
- [ ] The report lists every file changed and all done conditions
- [ ] Commit was not made
- [ ] Fix mode only: every `[blocker]` and `[concern]` is addressed or listed as unaddressed with a concrete reason
- [ ] Fix mode only: no file was modified that was not named in a finding
- [ ] Fix mode only: no `[question]` finding was answered on the developer's behalf
