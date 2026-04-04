# Getting Started with dmx-agents

This guide covers how to install dmx-agents and use skills and agent personas in your Claude Code sessions.

---

## Installation

### 1. Add to Claude Code

In your Claude Code settings, add this repo as a skill source. Skills are loaded by pointing Claude to the skills directory.

Add to your `~/.claude/CLAUDE.md` or per-project `CLAUDE.md`:

```markdown
## Skills

Skills are in `/Users/yourname/Documents/GitHub/dmx-agents/skills/`.
Load the relevant skill for each task:
- feature-spec: before writing any spec
- supabase-data-modeling: before touching the DB
- api-route-design: before adding API routes
- nextjs-component-patterns: before building components
- security-hardening: before any auth or data access code
- vercel-render-deploy: before deploying
```

### 2. Reference Checklists

Reference files are standalone checklists — link to them directly from CLAUDE.md or use them in prompts:

```markdown
## References
- Security: /Users/yourname/Documents/GitHub/dmx-agents/references/security-checklist.md
- Deployment: /Users/yourname/Documents/GitHub/dmx-agents/references/deployment-checklist.md
- Supabase: /Users/yourname/Documents/GitHub/dmx-agents/references/supabase-checklist.md
```

### 3. Agent Personas

Load an agent persona when you want Claude to take on a specific role:

```
"Take on the code-reviewer persona from /path/to/dmx-agents/agents/code-reviewer.md and review this PR."
```

---

## Using Skills

Skills are most effective when loaded before starting a task. Reference a skill by name in your prompt:

```
"Using the api-route-design skill, help me add a new route for roadmap items."
"Apply the security-hardening skill to review this new webhook handler."
"Follow the incremental-implementation skill as we build the CSV import feature."
```

---

## Skill Index by Phase

### Define
- [`feature-spec`](../skills/feature-spec/SKILL.md) — Write the spec before touching code
- [`idea-refine`](../skills/idea-refine/SKILL.md) — Turn a vague idea into a concrete spec

### Plan
- [`planning-and-task-breakdown`](../skills/planning-and-task-breakdown/SKILL.md) — Decompose features into tasks
- [`supabase-data-modeling`](../skills/supabase-data-modeling/SKILL.md) — Design the DB schema first

### Build
- [`api-route-design`](../skills/api-route-design/SKILL.md) — Add Elysia routes correctly
- [`nextjs-component-patterns`](../skills/nextjs-component-patterns/SKILL.md) — Build Next.js components
- [`incremental-implementation`](../skills/incremental-implementation/SKILL.md) — Ship in vertical slices

### Verify
- [`e2e-with-playwright`](../skills/e2e-with-playwright/SKILL.md) — Write Playwright tests
- [`debugging-and-error-recovery`](../skills/debugging-and-error-recovery/SKILL.md) — Debug systematically

### Review
- [`code-review-and-quality`](../skills/code-review-and-quality/SKILL.md) — Review PRs and self-review
- [`security-hardening`](../skills/security-hardening/SKILL.md) — Security audit before shipping

### Ship
- [`vercel-render-deploy`](../skills/vercel-render-deploy/SKILL.md) — Deploy checklist
- [`git-workflow-and-versioning`](../skills/git-workflow-and-versioning/SKILL.md) — Clean git history

---

## Updating Skills

Skills are living documents. Update them when:
- A pattern that's in a skill causes a bug in practice
- A new tool or package replaces an existing one
- A "Common Rationalization" turns out to be correct

See [CONTRIBUTING.md](../CONTRIBUTING.md) for the update process.
