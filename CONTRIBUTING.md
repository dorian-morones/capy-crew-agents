# Contributing

## Adding a New Skill

### Directory naming
- Kebab-case, lowercase: `my-new-skill/`
- Single `SKILL.md` file inside (uppercase)
- Supporting files only if content exceeds 100 lines

### Required frontmatter
```yaml
---
name: my-new-skill
description: Use when [specific, observable trigger condition]. [One-sentence outcome].
---
```

The `description` must start with "Use when" — this is how agents discover and activate skills.

### Required sections (in order)

1. **Overview** — What the skill provides and why it matters (2–3 paragraphs)
2. **When to Use** — Specific activation conditions + when to skip
3. **Core Process** — Numbered steps, each actionable and specific
4. **Specific Techniques** — Detailed scenarios with code examples for this stack
5. **Common Rationalizations** — Table of excuses and factual rebuttals
6. **Red Flags** — Observable patterns indicating the skill is being misapplied
7. **Verification** — Checkbox checklist with concrete evidence requirements

### Content standards

- **Specific over vague** — "Hash passwords with bcrypt ≥12 rounds" not "use secure hashing"
- **Stack-aware** — Reference actual tools: Elysia, Supabase, Clerk, Next.js, Playwright
- **Verifiable** — Every step has a clear done condition
- **Anti-rationalizations** — Address the real excuses that cause shortcuts
- **Length** — 400–1500 lines; split into supporting files only if needed

### Voice
- Imperative and direct: "Do X", "Never Y", "Always Z"
- No hedging language ("consider", "you might want to", "perhaps")
- Preserve the existing tone across all skills

## Adding a Reference

References go in `/references/` — not inside skill directories.

- Filename: `domain-checklist.md` or `domain-patterns.md`
- No frontmatter required
- Format: checklist items (`- [ ]`), tables, code blocks
- Length: 150–350 lines
- Include a subtitle: `Quick reference for [domain]. Use alongside the \`[skill-name]\` skill.`

## What Not to Add

- Duplicate content already covered by an existing skill
- Theoretical guidance without actionable steps
- Generic advice not specific to this stack
- Skills under 200 lines (too thin to be useful)
