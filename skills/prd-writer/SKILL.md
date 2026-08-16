---
name: prd-writer
description: Use when an approved brief needs to become product requirements — outcomes, user flows, scope boundaries, success metrics, and release criteria. Produces product/prd.md, the artifact handed to a client before any technical spec is written.
---

## Overview

The PRD sits between the brief and the technical spec. The brief argued that a problem is worth solving; the PRD says what the product must do about it, in terms a client can approve and an engineer can build from.

It describes outcomes and behaviour, not implementation. No database tables, no endpoints, no component names — those are the `architect` skill's job, downstream. If a reader could implement the PRD three different ways and all three would be acceptable, the PRD is at the right altitude.

This is usually the artifact a freelancer sends a client for sign-off. It has to be readable by someone non-technical and precise enough that "done" is not a matter of opinion.

## When to Use

**Use this skill when:**
- An approved `product/brief.md` exists and the work needs defining before quoting or building
- A client needs something concrete to approve before implementation starts
- A feature is large enough that "we'll figure out the details as we go" would be expensive

**Skip this skill when:**
- No brief exists — run `brief-writer` first, or you will document requirements for an unvalidated problem
- The work is a single small feature — go straight to the `writer` skill for a technical spec
- The problem is still unclear — run `interviewer`

## Core Process

1. **Read the brief and the interviews** — Read `product/brief.md` in full, plus any transcripts in `product/`. The PRD inherits the brief's problem, audience, and success signal. If you find yourself changing them, stop: that is a brief revision, and it needs its own approval.

2. **List the outcomes** — What must be true for a user when this ships? Write each as an observable capability: "a freelancer can produce a client-ready update for any week without opening the repo." Outcomes are the backbone; everything else supports them.

3. **Write the user flows** — For each outcome, the path a person takes: entry point, steps, what they see, what happens on success. Prose or numbered steps, not wireframes. Include the empty state and at least one failure path — those are where under-specified products break.

4. **Set the scope boundary** — Explicitly in, explicitly out, and explicitly later. Three lists. The "later" list is what keeps a client conversation from turning into a "no."

5. **Define success metrics** — Inherit the brief's signal and make it measurable: the metric, how it is measured, the current baseline, and the target. If there is no baseline, say so — an unmeasured baseline makes the target unfalsifiable.

6. **Write the release criteria** — The checklist that decides shipped vs not shipped. Each item must be verifiable by someone other than the author. This is the section that ends "is it done?" arguments.

7. **Record constraints and dependencies** — Deadlines, budget, compliance, third-party services, anything outside your control. A dependency discovered mid-build is a schedule slip; one recorded here is a plan.

8. **Carry risks and unknowns forward** — Everything still `[UNKNOWN]` from the brief, plus new risks the requirements surfaced. Never silently resolve one.

9. **Write `product/prd.md`** — Use `templates/prd.template.md`. Then hand to `product-critic` before sending it to anyone.

## Specific Techniques

### Stay Above Implementation

The line is: *what must be true* versus *how it is achieved*.

| Too low (belongs in the spec) | Right altitude |
|---|---|
| A `client_updates` table with an `account_id` FK | Updates are private to the account that created them |
| `POST /updates` returns 201 with the update ID | Creating an update returns it immediately, no page reload |
| A React modal with a Zustand store | The user can review and edit before sending |

When you catch yourself naming a technology, ask what user-visible outcome it was serving, and write that instead.

### Every Requirement Gets an Owner Outcome

Each requirement must trace to an outcome, which traces to the brief's problem. A requirement that traces to nothing is scope that arrived from somewhere else — cut it, or find the outcome it belongs to. Feature lists that grew this way are the single biggest source of freelance overruns.

### Release Criteria Are Not Acceptance Criteria

Acceptance criteria live per-requirement and say whether one behaviour works. Release criteria say whether the whole thing ships:

```markdown
## Release criteria
- [ ] All P0 requirements pass their acceptance criteria
- [ ] A new user can complete the primary flow without help
- [ ] Update generation completes in under 10s for a repo with 500 commits
- [ ] Client has reviewed one real generated update and approved the format
```

The last one matters most for freelancers: a criterion the *client* has to sign, agreed in advance, is what turns "I don't love it" into a defined conversation.

### Priority Is a Cut List, Not a Ranking

Mark each requirement P0 / P1 / P2, where the meaning is about cutting, not importance:

- **P0** — ship is meaningless without it
- **P1** — ship is worse without it, but still ship
- **P2** — genuinely fine in a later release

If more than about half the requirements are P0, the priorities have not been decided yet. Push back — this is exactly the conversation the PRD exists to force, and having it now costs one email instead of a renegotiation.

### Write for the Client, Not for Engineering

The PRD gets sent to someone who may not be technical. Plain sentences, no jargon, no internal shorthand, and every acronym expanded once. If a paragraph needs the reader to know what a webhook is, rewrite it — the technical detail has a home, and this is not it.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll sketch the data model here so engineering has it" | That is the architect's job, and it locks in decisions before they are informed. |
| "Everything is P0, the client wants it all" | Then nothing is prioritised and the first delay becomes a crisis. Force the cut now. |
| "The success metric is in the brief, no need to repeat it" | The brief's signal is directional. The PRD's metric needs a baseline and a target. |
| "We can define release criteria when we're closer to shipping" | By then whoever wanted more has already asked for more. Agree the bar while it is cheap. |
| "The happy path is enough for now" | Empty states and failure paths are most of the real work. Undocumented, they get invented mid-build. |
| "The brief's problem statement is a bit off, I'll fix it here" | That is a brief revision needing its own approval. Do not silently rewrite the premise. |

## Red Flags

- Table names, endpoints, component names, or library choices anywhere in the document
- A requirement that traces to no outcome
- More than half the requirements marked P0
- Success metrics with a target but no baseline
- No empty state or failure path in any flow
- Release criteria only the author could verify
- Jargon a non-technical client would not follow
- The brief's problem, audience, or success signal quietly changed

## Verification

- [ ] Every requirement traces to a stated outcome, and every outcome to the brief's problem
- [ ] No implementation detail appears — no schema, endpoints, components, or libraries
- [ ] Scope is split into in / out / later, all three populated
- [ ] Each requirement carries P0, P1, or P2, and P0s are under half the total
- [ ] Success metrics have a measurement method, a baseline, and a target
- [ ] Release criteria are verifiable by someone other than the author
- [ ] Every flow includes its empty state and at least one failure path
- [ ] Risks, constraints, dependencies, and unresolved `[UNKNOWN]`s are listed
- [ ] Readable by a non-technical client
- [ ] Saved to `product/prd.md`
