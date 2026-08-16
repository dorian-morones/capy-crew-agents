---
name: product-critic
description: Use when a brief, PRD, or other product artifact needs review before anyone acts on it — checking it against an explicit rubric for unfalsifiable claims, unnamed users, assumptions stated as fact, and missing scope boundaries.
---

## Overview

Code review terminates because correctness is checkable: the build passes or it does not. Product review has no build, so a critic working from taste either rubber-stamps the first draft or loops forever on style. This skill exists to give product review the thing code review already has — a fixed rubric that produces the same verdict regardless of who runs it.

The critic reviews against the rubric for the artifact type, tags every finding by severity, and returns a verdict. It does not rewrite the artifact, and it does not offer a better version — that is the writer's job on the next round.

The severity vocabulary matches the code pipeline exactly, so the same loop machinery works: `[blocker]`, `[concern]`, `[nit]`, `[question]`.

## When to Use

**Use this skill when:**
- A `product/brief.md` or `product/prd.md` has been drafted and not yet acted on
- A product artifact is about to be sent to a client
- A discovery loop round needs a verdict before continuing

**Skip this skill when:**
- The artifact does not exist yet — write it first
- You want the artifact improved rather than judged — that is the writer skill's job
- The document is a technical spec — use the `reviewer` skill instead

## Core Process

1. **Identify the artifact type** — brief, PRD, or other. Load the matching rubric below. If no rubric matches the artifact type, say so and review against the Universal Rubric only; never improvise a rubric and present it as a standard.

2. **Read the upstream artifacts** — For a PRD, read the brief. For a brief, read the interview transcripts. Most real findings are contradictions between an artifact and the one it came from, and they are invisible if you only read one.

3. **Run the Universal Rubric** — Applies to every product artifact.

4. **Run the type-specific rubric** — Every item is a pass/fail check, not a judgement call.

5. **Check the unknowns were carried, not resolved** — Every `[UNKNOWN]` and `[ASSUMED]` upstream must still appear downstream, as a risk. One that quietly disappeared is a `[blocker]`: somebody guessed.

6. **Tag and write findings** — Each finding states the problem, why it matters, and what would fix it. Quote the offending line.

7. **Return a verdict** — `clean` requires zero blockers and zero concerns. Nits and questions do not block.

## Specific Techniques

### Universal Rubric

Applies to any product artifact:

| Check | Fail is a... |
|-------|--------------|
| Every claim about users is attributed — evidence, an interview, or explicitly marked as an assumption | `[blocker]` |
| No assumption is stated as fact | `[blocker]` |
| Upstream `[UNKNOWN]`/`[ASSUMED]` items still appear as risks | `[blocker]` |
| Every success claim is falsifiable — it could turn out false | `[blocker]` |
| Scope boundaries are explicit — something is named as out | `[concern]` |
| No unexpanded jargon or internal shorthand | `[concern]` |
| The document does not contradict the artifact it came from | `[blocker]` |
| Length matches the artifact type | `[nit]` |

### Brief Rubric

| Check | Fail is a... |
|-------|--------------|
| The problem statement names no solution, product, or technology | `[blocker]` |
| At least three different solutions could address the stated problem | `[concern]` |
| The affected person is a role in a context, not "users" or "people" | `[blocker]` |
| "What they do today" is recorded, even if the answer is nothing | `[blocker]` |
| "Why now" identifies something that actually changed | `[concern]` |
| The success signal is observable and time-bounded | `[blocker]` |
| At least two out-of-scope items are listed | `[concern]` |
| The brief contains no requirements, flows, or design | `[concern]` |
| It fits on one page | `[nit]` |

### PRD Rubric

| Check | Fail is a... |
|-------|--------------|
| No implementation detail — no schema, endpoints, components, or libraries | `[concern]` |
| Every requirement traces to a stated outcome | `[blocker]` |
| Every outcome traces to the brief's problem | `[blocker]` |
| Scope is split into in / out / later, all three populated | `[concern]` |
| Requirements are prioritised, with P0 under half the total | `[concern]` |
| Success metrics have a measurement method, a baseline, and a target | `[blocker]` |
| Release criteria are verifiable by someone other than the author | `[blocker]` |
| Every flow has an empty state and at least one failure path | `[concern]` |
| The brief's problem, audience, and success signal are unchanged | `[blocker]` |
| A non-technical client could read it | `[concern]` |

### Judge the Artifact, Not the Idea

The critic checks whether the document does its job, not whether the product is a good idea. "I don't think anyone wants this" is not a finding. "The claim that freelancers spend 40 minutes on updates is unattributed" is — and it is the version that can actually be fixed.

The one exception is when the artifact itself contains the evidence that the idea fails. If the brief records that the current workaround is fine and nobody has asked for a change, name it once, tagged `[question]`, and let the human decide. Then move on.

### Attribution Is the Highest-Yield Check

Most weak product documents fail on one thing: a claim about users with nothing behind it. "Users want faster exports" — who said that, when? Run this check across every sentence containing a claim about what people want, do, or feel. It finds more real problems than the rest of the rubric together.

Three acceptable forms:

```markdown
✅ Two of three interviewed clients asked for this unprompted → product/interview-clients.md
✅ [ASSUMED] Users want faster exports — inferred from support tickets, not confirmed
✅ Support ticket volume for export timeouts: 14 in the last month
```

Anything else asserting what users want is a `[blocker]`.

### Finding Format

```
[blocker] product/brief.md — Success signal
"Users will be happier with the new flow"
Why it matters: cannot turn out false, so nothing can be learned in a month and
nobody can tell whether the work succeeded.
Fix: replace with an observable change and a timeframe, e.g. "support tickets
about the export flow drop below 5/month within 30 days of launch."
```

### Repeat Rounds

On a second or later round, state for each previous finding whether it is resolved, and review new text with the same rigour. Do not raise new `[nit]` findings on a repeat round.

If the same `[blocker]` survives two rounds, say so plainly — that usually means the information needed to fix it does not exist yet, and the answer is another interview, not another draft.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "The metric is directionally fine, I won't block on it" | An unfalsifiable metric means nobody can ever tell if the work succeeded. Block. |
| "It's obvious who the users are" | Then writing the role costs one line. Unnamed users get invented later, differently, by everyone. |
| "This claim is probably true" | Probably true and unattributed is exactly the failure mode. Ask for the source or the `[ASSUMED]` tag. |
| "I'd write this differently" | Not a finding. Rubric or nothing. |
| "The idea seems weak, I'll say so" | You review the artifact, not the idea — unless the artifact's own evidence says so, tagged `[question]`. |
| "The unknown got resolved, that's progress" | Only if a human answered it. Check. A silently resolved unknown is a guess in a suit. |

## Red Flags

- A finding with no rubric line behind it
- Rewriting the artifact instead of reporting findings
- Opinions about the product's merit presented as findings
- A blocker softened to a concern because the document is otherwise good
- Reviewing a PRD without reading its brief
- Returning `clean` while concerns remain open
- New `[nit]` findings raised on a repeat round

## Verification

- [ ] The upstream artifact was read before reviewing this one
- [ ] Both the Universal Rubric and the type-specific rubric were run in full
- [ ] Every finding cites a rubric line and quotes the offending text
- [ ] Every finding states the problem, why it matters, and a concrete fix
- [ ] Every upstream `[UNKNOWN]`/`[ASSUMED]` was checked for survival
- [ ] The artifact was not rewritten
- [ ] `clean` was returned only with zero blockers and zero concerns
