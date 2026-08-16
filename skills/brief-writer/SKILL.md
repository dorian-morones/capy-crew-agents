---
name: brief-writer
description: Use when deciding whether a product or feature is worth building at all — turning an idea into a one-page brief covering the problem, who has it, why now, and what success would look like. Produces product/brief.md, no requirements and no solution design.
---

## Overview

A brief answers one question: is this worth building? Not how to build it, not what the screens look like — whether the problem is real, whose it is, and how you would know if solving it worked.

It is one page. That constraint is the point: an idea that cannot be stated in a page is an idea that has not been thought through, and the missing thinking will surface later as scope churn.

The brief comes before the PRD, which comes before the spec. Each answers a different question — *worth building* → *what it must do* → *how it is built*. Collapsing them is how projects end up with beautiful requirements for something nobody needed.

## When to Use

**Use this skill when:**
- A new product, feature, or client engagement is being considered and nobody has written down why
- A stakeholder is describing a solution and the underlying problem has never been stated
- You need something short to align a client before quoting or building
- A project is drifting and you need to re-establish what it was for

**Skip this skill when:**
- The problem is already written down and agreed — go to `prd-writer`
- It is a bug fix, a small feature, or a technical task — use `idea-refine`, then `writer`
- You have not talked to anyone yet and know almost nothing — run `interviewer` first

## Core Process

1. **Gather what exists** — Read any interview transcripts in `product/`, prior briefs, the repo README, and existing specs. Never start a brief from a one-line prompt when there is material to read.

2. **Find the gaps that block a brief** — You cannot write an honest brief without: who has the problem, what they do today instead, and what success would look like. If any is missing, run the `interviewer` skill rather than inventing them.

3. **State the problem without naming a solution** — Write the problem in one or two sentences with no product in it. "Freelancers lose billable hours writing client updates by hand" is a problem. "We need an update generator" is a solution wearing a problem's clothes.

4. **Name who specifically has it** — Not "users." A describable person: their role, their context, how often they hit this. If the answer is "everyone," the brief is not ready.

5. **Record what they do today** — The current workaround is the real competition, and it is usually "nothing" or "a spreadsheet." If the workaround is tolerable, say so — that is a finding, not a failure.

6. **Answer why now** — What changed that makes this worth doing today rather than last year or next year? A brief with no "why now" is usually a brief for something that can wait.

7. **Define the success signal** — One observable thing that would be different in a month if this worked. It must be falsifiable. "Users are happier" is not; "the client stops sending me the Monday email" is.

8. **List what this is not** — Two to five things deliberately out of scope. This section prevents more rework than the rest of the brief combined.

9. **Carry unknowns forward** — Every `[UNKNOWN]` and `[ASSUMED]` from the interview becomes a listed risk. Never quietly resolve one while writing.

10. **Write `product/brief.md`** — Use `templates/brief.template.md`. One page. Then hand to `product-critic` before anyone acts on it.

## Specific Techniques

### Problem, Not Solution

The most common failure in a brief is a solution restated as a problem. A test: if you removed every product noun from the problem statement, would anything remain?

> "Users need a dashboard" → remove "dashboard" → nothing remains. Not a problem.
> "Account managers cannot tell which clients are at risk until renewal week" → survives. That is a problem.

Write the problem so that at least three different solutions could address it. If only one solution fits, the problem is written too narrowly.

### The Falsifiable Success Signal

The success signal must be something that could turn out false. Write it as a sentence someone could check in a month without arguing about interpretation:

| Weak | Falsifiable |
|------|-------------|
| Better user engagement | Weekly active projects per user goes from 1.2 to 2+ |
| Improved client satisfaction | The client stops asking for status by email |
| Faster workflow | Producing a client update takes under 5 minutes, down from ~40 |

If no falsifiable signal exists, say so explicitly and mark it a risk. That is a legitimate outcome — an unmeasurable goal is worth knowing about before building, not after.

### The Do-Nothing Baseline

Always state what happens if this is never built. It is the cheapest sanity check available, and it occasionally ends the project on page one — which is the most valuable outcome a brief can have.

### One Page Means One Page

If the brief runs long, the extra material is almost always requirements or design, which belong in the PRD. Move them; do not lengthen the brief. A brief nobody finishes reading aligns nobody.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "The problem is obvious, I'll start with the solution" | Then it costs one sentence to write down. Obvious problems are the ones people disagree about silently. |
| "Users is specific enough for who" | It is not. Name a role and a context, or admit you do not know yet. |
| "I'll fill in the success metric later" | Later is after it is built, when it can no longer change anything. |
| "The client said what they want, so the problem is settled" | They stated a solution. Ask what it would fix. |
| "I don't need the out-of-scope list, we all know" | The out-of-scope list is where scope creep goes to die. Write it. |
| "I'll resolve that unknown myself, it's minor" | Carry it forward as a risk. Minor unknowns compound into major rework. |

## Red Flags

- A problem statement containing a product noun, a feature name, or a technology
- "Users," "customers," or "people" where a specific role belongs
- A success signal that cannot turn out false
- No "what they do today" section, or "nothing" recorded without comment
- Missing out-of-scope list
- Unknowns from the interview silently resolved in the brief
- A brief longer than a page
- Requirements, screens, or API design appearing anywhere in it

## Verification

- [ ] The problem statement contains no solution and admits more than one possible fix
- [ ] The affected person is named by role and context, not as "users"
- [ ] What they do today is recorded, including if the answer is "nothing"
- [ ] "Why now" is answered with something that actually changed
- [ ] The success signal is falsifiable and checkable within a stated timeframe
- [ ] At least two out-of-scope items are listed
- [ ] Every `[UNKNOWN]` and `[ASSUMED]` from the interview appears as a risk
- [ ] The brief fits on one page and contains no requirements or design
- [ ] Saved to `product/brief.md`
