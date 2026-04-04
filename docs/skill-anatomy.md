# Skill Anatomy

This document explains the structure of a SKILL.md file — what each section is for, what makes it good, and how to write one.

---

## File Location

```
skills/
└── skill-name/
    └── SKILL.md
```

One skill per directory. The directory name is the skill's identifier.

---

## Frontmatter

```yaml
---
name: skill-name
description: Use when [specific trigger]. [One-sentence outcome].
---
```

**Rules:**
- `name` matches the directory name exactly
- `description` starts with "Use when" — this is what Claude reads to decide if the skill is relevant
- The trigger is specific — not "Use when building things" but "Use when adding a new Elysia route"
- One sentence for the trigger, one for the outcome

**Good:**
```yaml
description: Use when creating or modifying Elysia API routes, middleware, or authentication patterns in actify-api.
```

**Bad:**
```yaml
description: Helps with API development.
```

---

## Overview

One paragraph. Explains:
1. What problem this skill solves
2. Why this problem keeps appearing
3. What the skill provides (process, patterns, checklists)

Don't explain what the technology is. The reader already knows what an API route is — explain why this specific pattern matters in this specific stack.

---

## When to Use

Two lists:

**"Use this skill when:"** — specific triggering conditions. Be concrete enough that there's no ambiguity about whether the skill applies.

**"Skip this skill when:"** — equally important. Keeps the skill from being loaded for every task.

---

## Core Process

A numbered list of 4-8 steps. This is the step-by-step procedure to follow. Steps should be:
- In the order they should be performed
- Independent enough that each can be explained in one sentence
- Actionable — "Do X" not "Think about X"

---

## Specific Techniques

The technical meat. Code examples, patterns, templates. Each technique should be:
- Named (a header)
- Shown with working code
- Explained with ✅ (correct) and ❌ (wrong) examples where the distinction isn't obvious

This is the section that makes a skill specific to your stack rather than generic advice.

---

## Common Rationalizations

A two-column table:

| Rationalization | Reality |
|----------------|---------|

These are the specific arguments people make to skip a step in the Core Process, paired with the direct rebuttal. Good rationalizations come from real situations — things that actually happen, not theoretical excuses.

This section exists because knowing what to do isn't enough if you know how to talk yourself out of it.

---

## Red Flags

A bullet list of observable signals that the skill's core rules are being violated. Each flag should be:
- Specific and detectable (not "bad code")
- Something you can grep for or see in a diff

Examples:
- `supabaseAdmin` used in a route that has `auth` middleware
- `"use client"` on a component with no hooks or event handlers

---

## Verification

A checkbox list that serves as the final checklist before the work is done. Every item should be:
- Binary (done or not done)
- Testable (someone else could verify it)
- Directly tied to one of the Core Process steps

---

## Voice and Style

- Write in second person imperative: "Validate input at the boundary." Not "You should validate input."
- No hedging: "Never" and "Always" — not "typically" or "usually"
- Short sentences. One idea per sentence.
- Code examples over prose explanations whenever possible.
- If you find yourself writing more than 3 sentences to explain something, consider whether a code example would say it better.
