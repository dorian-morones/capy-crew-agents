---
name: discovery
description: Use when running the full product definition pipeline for a new product, feature, or client engagement — interview, brief, PRD — with a critic loop on each artifact and developer approval gates between phases, ending with a handoff to the technical spec.
---

## Overview

The discovery skill orchestrates the product definition pipeline: **interview → brief → PRD → handoff**. It is the upstream counterpart to `capy`, which runs the build pipeline. Discovery decides *what is worth building and what it must do*; capy turns that into *working code*.

Each artifact goes through a write → critique → revise loop before the developer sees it, so no unreviewed product document reaches a client. The loop is capped: two revision rounds, then the findings go to the developer, because a finding that survives two rounds usually means missing information, and another draft cannot manufacture information.

Discovery does not write briefs or PRDs itself. It spawns subagents, reads their structured result blocks, holds the approval gates, and hands off. Its jobs are: coordinate, summarize, gate, and iterate.

## When to Use

**Use this skill when:**
- Starting a new product, feature, or client engagement where the problem is not yet written down
- A client needs a brief or PRD to approve before implementation is quoted or started
- An existing project has drifted and needs its premise re-established

**Skip this skill when:**
- You only need one artifact (run `brief-writer` or `prd-writer` directly)
- A brief and PRD already exist and are approved — go straight to `capy`
- The work is a single small feature with a clear problem — use `idea-refine`, then `capy`

## Core Process

### Phase 1 — Interview

1. Spawn an interviewer subagent using the Agent tool:
   - `subagent_type`: "capy-crew-agents:interviewer"
   - `description`: "Interview for: [topic]"
   - `prompt`: Interviewer Subagent Prompt
   - `run_in_background`: false

**The interviewer talks to the developer through you.** A subagent cannot hold a conversation with the user, so run this phase differently from all others: the interviewer returns its question list and its research, and *you* ask the questions in the main thread — one at a time, waiting for each answer. Record the answers to `product/interview-<topic>.md` yourself.

This is the one phase where the orchestrator produces an artifact directly, because it is the only phase that requires a live human conversation.

Ask one question per turn. Never present the whole list at once.

2. When the interview is done, summarise:

```
Interview saved to product/interview-<topic>.md.

[N] questions answered · [N] unknowns recorded

Unknowns that will carry into the brief as risks:
  - <unknown> → <risk>

✅ Continue to the brief
✏️  More questions — tell me what else to ask
```

---

### Phase 2 — Brief

3. Spawn a brief-writer subagent:
   - `subagent_type`: "capy-crew-agents:brief-writer"
   - `description`: "Write brief: [topic]"
   - `prompt`: Brief-Writer Subagent Prompt
   - `run_in_background`: false

4. Run the **Critique Loop** (below) on `product/brief.md`.

5. When the loop returns clean, present the gate:

```
Brief saved to product/brief.md — reviewed clean after [R] round(s).

Problem: <one sentence>
Who: <role and context>
Today they: <current workaround>
Success signal: <falsifiable signal>
Out of scope: <list>
Risks: [N] carried from the interview

✅ Approved — continue to the PRD
✏️  Changes needed — describe what to adjust
🛑 Stop — the brief says this is not worth building
```

The third option is real. A brief that concludes the problem is not worth solving is a successful outcome, and the pipeline should stop there without embarrassment.

---

### Phase 3 — PRD

6. Spawn a prd-writer subagent:
   - `subagent_type`: "capy-crew-agents:prd-writer"
   - `description`: "Write PRD: [topic]"
   - `prompt`: PRD-Writer Subagent Prompt
   - `run_in_background`: false

7. Run the **Critique Loop** on `product/prd.md`.

8. Present the gate:

```
PRD saved to product/prd.md — reviewed clean after [R] round(s).

[N] outcomes · [N] requirements ([N] P0, [N] P1, [N] P2)
Success metrics: [N] with baselines
Release criteria: [N]
Open risks: [N]

✅ Approved — hand off to the build pipeline
✏️  Changes needed — describe what to adjust
```

---

### Phase 4 — Handoff

9. Report and stop:

```
Discovery complete.

  product/interview-<topic>.md
  product/brief.md
  product/prd.md

Open risks carried forward:
  - <risk>

Next: run capy to turn the PRD into a spec, architecture, tasks, and code:
  /capy build <feature> from product/prd.md
```

Discovery does not start the build. That is a separate decision and a separate pipeline.

## Specific Techniques

### The Critique Loop

Every artifact runs the same loop before the developer sees it:

1. Set `round` to 1. Spawn a product-critic subagent on the artifact.
2. Read `verdict`:
   - `clean` → present the developer gate
   - `needs_revision` → continue
3. If `round` is 3 or higher, stop and apply the Revision Cap protocol.
4. Otherwise re-spawn the writer subagent in **revision mode** with the `[blocker]` and `[concern]` findings pasted in verbatim.
5. Increment `round`, return to step 2.

Pass only `[blocker]` and `[concern]` findings to the writer. Hold `[nit]` findings for the gate. **`[question]` findings go to the developer immediately** — a question means the critic found something only a human can resolve, and a writer answering it would invent product truth.

The critic's verdict is authoritative. Never compute your own from the finding counts, and never decide a blocker looks minor.

### Revision Cap

Two automatic revision rounds. At the cap:

```
<artifact> still has unresolved findings after 2 revision rounds.

Remaining:
  [blocker] <finding> | Fix: <fix>

Why the writer could not resolve them:
  - <reason>

Options:
  🔁 One more round — with your guidance
  🎤 More interview — the missing information does not exist yet (recommended if
     findings are about unattributed claims or unknown users)
  ⏭️  Accept — you have judged the remaining findings acceptable
  🛑 Stop — end discovery here
```

The interview option is usually the right one. Product findings that survive two rounds are almost never writing problems — they are missing-information problems, and only a human can supply the information.

### Never Invent Product Truth

The rule that governs this entire pipeline: no subagent, and not you, may invent a fact about users, customers, or the market. Claims come from the interview, from cited evidence, or carry an `[ASSUMED]` tag. An unattributed claim about what users want is the single failure mode this pipeline exists to prevent — and it is fluent, confident, and invisible unless someone checks.

### Unknowns Are Carried, Never Closed

An `[UNKNOWN]` from the interview appears in the brief as a risk, and in the PRD as a risk, until a human answers it. If one disappears between artifacts, something guessed. The critic checks this, and so should you when reading a result block.

## Subagent Prompt Templates

Each subagent starts with no context. Every prompt must be self-contained. Every subagent ends with a `## DISCOVERY_RESULT` block.

### Interviewer Subagent Prompt

```
Topic: <TOPIC>
Purpose: this interview feeds product/brief.md

Use the Skill tool to run the interviewer skill: Skill({ skill: "capy-crew-agents:interviewer", args: "<TOPIC>" })

You cannot talk to the developer directly. Instead: research what is already
available (repo, README, specs/, product/), then return the question list for the
orchestrator to ask in the main thread, ordered, one per line.

End your response with:

## DISCOVERY_RESULT
status: success
already_known:
  - <fact read from the repo, with source>
questions:
  1. <question>
  2. <question>
skip_because_discoverable:
  - <question not worth asking, and where the answer already is>
```

### Brief-Writer Subagent Prompt

```
Interview: product/interview-<TOPIC>.md
Topic: <TOPIC>

Use the Skill tool to run the brief-writer skill: Skill({ skill: "capy-crew-agents:brief-writer", args: "Write the brief for <TOPIC> from product/interview-<TOPIC>.md" })

Read the interview in full first. Carry every [UNKNOWN] and [ASSUMED] forward as a
risk — never resolve one yourself. Write product/brief.md. One page.

End your response with:

## DISCOVERY_RESULT
status: success
artifact_path: product/brief.md
problem: <one sentence, no solution in it>
who: <role and context>
today_they: <current workaround>
why_now: <what changed>
success_signal: <falsifiable signal with timeframe>
out_of_scope: <list>
risks_carried: <count and one-line list>

On failure:

## DISCOVERY_RESULT
status: error
error: <what went wrong, and what information was missing>
```

### PRD-Writer Subagent Prompt

```
Brief: product/brief.md
Interview: product/interview-<TOPIC>.md

Use the Skill tool to run the prd-writer skill: Skill({ skill: "capy-crew-agents:prd-writer", args: "Write the PRD from product/brief.md" })

Read the brief and interview in full first. Do not change the brief's problem,
audience, or success signal — if one seems wrong, report it rather than editing it.
No implementation detail. Write product/prd.md.

End your response with:

## DISCOVERY_RESULT
status: success
artifact_path: product/prd.md
outcomes: <count and one-line list>
requirements: <total> (P0: <n>, P1: <n>, P2: <n>)
metrics_with_baselines: <count>
release_criteria: <count>
risks_carried: <count>
brief_conflicts: <anything in the brief that seemed wrong, or "none">

On failure:

## DISCOVERY_RESULT
status: error
error: <what went wrong>
```

### Product-Critic Subagent Prompt

```
Artifact to review: <ARTIFACT_PATH>
Artifact type: <brief | prd>
Upstream artifact: <UPSTREAM_PATH>
Review round: <R>

Use the Skill tool to run the product-critic skill: Skill({ skill: "capy-crew-agents:product-critic", args: "Review <ARTIFACT_PATH> as a <TYPE>" })

Read the upstream artifact first. Run the Universal Rubric and the type-specific
rubric in full. Do not rewrite the artifact.

On round 2 or later, state whether each previous finding is resolved, and raise no
new [nit] findings.

End your response with:

## DISCOVERY_RESULT
status: success
artifact_path: <ARTIFACT_PATH>
review_round: <R>
verdict: <clean | needs_revision>
blockers: <N>
concerns: <N>
nits: <N>
questions: <N>
findings:
  - [blocker] <location> — <problem> | Why: <why it matters> | Fix: <fix>
unknowns_dropped: <upstream [UNKNOWN]s that vanished, or "none">
previously_raised_resolved: <or "n/a">

`verdict: clean` requires blockers: 0 and concerns: 0.

On failure:

## DISCOVERY_RESULT
status: error
review_round: <R>
error: <what went wrong>
```

### Writer Revision-Mode Prompt

Used for both brief-writer and prd-writer during the critique loop.

```
You are running in REVISION MODE. The artifact already exists at <ARTIFACT_PATH>.
Apply the critic's findings to it — do not rewrite it from scratch.

Artifact: <ARTIFACT_PATH>
Upstream: <UPSTREAM_PATH>
Revision round: <R>

Critic findings to address (blockers and concerns only):
<FINDINGS_VERBATIM>

Use the Skill tool to run the <brief-writer | prd-writer> skill.

- Read the current artifact and its upstream source before editing
- Address only the findings listed; do not restructure sections nobody flagged
- Every finding must end this round addressed or explicitly unaddressed with a reason
- Never invent a fact about users to satisfy a finding — if the information does not
  exist, report the finding as unaddressed and say what interview question would get it
- Do not answer [question] findings

End your response with:

## DISCOVERY_RESULT
status: success
artifact_path: <ARTIFACT_PATH>
revision_round: <R>
findings_addressed:
  - [blocker] <finding> — <what changed>
findings_not_addressed:
  - [concern] <finding> — <why, and what interview question would resolve it>
notes: <or "none">
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll ask all the interview questions at once to save time" | One question per turn. A batch gets skimmed and half-answered. |
| "The brief looks good, I'll skip the critic" | Every artifact is critiqued before the developer sees it. No exceptions. |
| "The writer said it addressed everything, so it's clean" | Only a critic round with `verdict: clean` closes an artifact. |
| "I know what users want here, I'll fill that in" | You may not invent product truth. Nothing in this pipeline may. |
| "This blocker is really a nit" | The critic sets severity. You do not re-grade findings. |
| "Round 3 still has blockers, one more round will fix it" | Two rounds is the cap. Findings that survive it need information, not prose. |
| "The unknown got resolved somewhere along the way" | Then a subagent guessed. Check who answered it. |
| "The PRD is approved, I'll start building" | Discovery hands off. Starting the build is a separate decision. |

## Red Flags

- Presenting more than one interview question in a turn
- Writing brief or PRD content yourself instead of spawning a subagent
- Presenting an artifact at a gate without a critic round returning `verdict: clean`
- Passing `[question]` or `[nit]` findings into a revision prompt
- Answering a `[question]` finding on the developer's behalf
- Re-grading the critic's severity tags or computing your own verdict
- Spawning a third revision round instead of applying the Revision Cap
- An `[UNKNOWN]` disappearing between artifacts without a human answering it
- Any claim about users appearing without attribution or an `[ASSUMED]` tag
- Continuing into implementation after the PRD gate

## Verification

- [ ] Interview questions were asked one per turn in the main thread
- [ ] The interview transcript records answers verbatim, with unknowns marked
- [ ] Every artifact was critiqued by a product-critic subagent before its gate
- [ ] Every artifact reaching a gate returned `verdict: clean`, or the developer explicitly accepted the findings
- [ ] No artifact ran more than two revision rounds without the developer being asked
- [ ] Revision prompts carried only `[blocker]` and `[concern]` findings
- [ ] `[question]` findings went to the developer unanswered
- [ ] Every `[UNKNOWN]` from the interview still appears as a risk in the brief and PRD
- [ ] No claim about users appears without attribution or an `[ASSUMED]` tag
- [ ] The pipeline stopped at the handoff and did not start the build
