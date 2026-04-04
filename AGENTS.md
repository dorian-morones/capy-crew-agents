# Agent Contribution Guide

## SKILL.md Template

```markdown
---
name: skill-name
description: Use when [specific trigger]. [One-sentence outcome].
---

## Overview

[2–3 paragraphs explaining what this skill provides, why it matters, and what problem it solves.]

## When to Use

**Use this skill when:**
- [Observable trigger condition 1]
- [Observable trigger condition 2]

**Skip this skill when:**
- [Condition where this is overkill]
- [Condition handled by a different skill]

## Core Process

1. **Step name** — [Specific action with clear done condition]
2. **Step name** — [Specific action with clear done condition]
3. **Step name** — [Specific action with clear done condition]

## Specific Techniques

### Technique Name

[Detailed explanation with code examples relevant to the stack]

\`\`\`typescript
// Example code
\`\`\`

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "This is too simple to need a spec" | Simple-looking changes have caused the worst regressions |
| "We'll fix it later" | Later never arrives; shortcuts ship |

## Red Flags

- [Observable pattern indicating misapplication]
- [Observable pattern indicating misapplication]
- [Observable pattern indicating misapplication]

## Verification

- [ ] [Concrete checkpoint with evidence requirement]
- [ ] [Concrete checkpoint with evidence requirement]
- [ ] [Concrete checkpoint with evidence requirement]
```

## Naming Standards

| Item | Convention | Example |
|------|-----------|---------|
| Skill directory | kebab-case | `api-route-design/` |
| Skill file | uppercase | `SKILL.md` |
| Reference files | kebab-case | `supabase-checklist.md` |
| Agent files | kebab-case | `code-reviewer.md` |
| Script files | kebab-case + .sh | `run-audit.sh` |

## Context Optimization

- Keep SKILL.md under 1500 lines
- Move large code examples to supporting files
- Reference supporting files from SKILL.md: `See script.sh for full implementation`
- Cross-reference related skills rather than duplicating content
