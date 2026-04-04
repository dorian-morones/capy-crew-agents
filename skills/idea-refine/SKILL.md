---
name: idea-refine
description: Use when an idea is vague, underspecified, or stated as a user need without a clear technical direction.
---

## Overview

Raw ideas are valuable but unusable. "We should let users export their data" is an idea. "A CSV export endpoint that streams paginated feedback rows filtered by date range, authenticated via Clerk, with a 10k row limit" is a spec. This skill bridges that gap.

Idea refinement is the process of interrogating a vague concept until it becomes specific enough to write a feature spec. It surfaces hidden assumptions, forces scope decisions, and identifies what you actually need to build versus what sounds good.

## When to Use

**Use this skill when:**
- Someone (including you) describes a feature as a user outcome without a technical shape
- A feature request arrives from outside engineering
- You have an idea but aren't sure where to start
- Two people on the same team have different mental models of what "X" means

**Skip this skill when:**
- The idea is already specific enough to write a feature spec (go straight to `feature-spec`)
- It's a pure technical task with no ambiguity (e.g. "add an index to this column")

## Core Process

1. **Restate the idea in one sentence** — If you can't, it's not one idea. Split it.

2. **Identify the actor** — Who is doing this? (End user, admin, background job, webhook)

3. **Identify the trigger** — What causes this to happen? (User action, scheduled job, external event)

4. **Identify the outcome** — What is different after this happens? (Data written, notification sent, UI updated)

5. **Ask the five hard questions:**
   - What does failure look like?
   - What's the simplest version that delivers value?
   - What are we explicitly NOT building?
   - What does this depend on that doesn't exist yet?
   - How will we know it's working?

6. **Write a one-paragraph description** — Actor + trigger + outcome + constraints

7. **Hand off to `feature-spec`** — The refined idea becomes the input to spec writing

## Specific Techniques

### The Simplest Version Test

For any idea, ask: "What's the version of this that could ship this week?"

If the answer is "nothing", the idea is too vague. If the answer is a different feature entirely, you've found scope creep.

Example:
- Idea: "AI-powered insights dashboard"
- Simplest version: "A page showing the top 5 themes from the last 30 days of feedback, sorted by frequency"
- What we're NOT building: Real-time updates, customizable date ranges, export — yet

### Constraint Surfacing

Ask these for every idea:
- Does this require a new DB table? New migration?
- Does this require a new API route or modifying an existing one?
- Does this require a new Clerk permission or webhook?
- Does this require a new env var or third-party service?
- Does this need a feature flag?

### Out-of-Scope List

Write an explicit "not in this version" list. It prevents scope creep and sets expectations:

```
In scope:
- CSV export of feedback filtered by source and date range
- Max 10,000 rows per export

Out of scope (v1):
- Excel format
- Scheduled/recurring exports
- Export of roadmap items or themes
- Email delivery of exports
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "Everyone knows what this means" | They don't. Ask three people to describe the same feature and get three different answers. |
| "We'll figure out the details during implementation" | Implementation details become architecture. Deciding them under pressure leads to shortcuts that ship. |
| "We don't want to over-engineer it" | Refinement isn't over-engineering. It's deciding what you're building before you build it. |
| "The idea might change anyway" | A refined idea can change. An unrefined idea changes in every conversation. |

## Red Flags

- An idea described entirely in terms of UI ("a button that…") without stating what changes in the system
- No answer to "how will we know it's working?"
- An idea that requires 3+ new DB tables, 5+ new routes, and 2 new integrations (that's a project, not a feature)
- Scope that grows every time someone asks a clarifying question
- No explicit out-of-scope list

## Verification

- [ ] Idea restated in one sentence with actor, trigger, and outcome
- [ ] Simplest shippable version identified and written down
- [ ] Out-of-scope list written
- [ ] Five hard questions answered
- [ ] Dependencies identified
- [ ] Ready to hand off to `feature-spec`
