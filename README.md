# Capy Crew Agents

![capy-crew-agents](./banner.png)

A Claude Code plugin that enforces **Spec-Driven Development** — no code gets written before a spec is approved.

It bundles four slash commands (one per phase), a library of auto-loading skills, and a set of agent personas for focused work sessions.

---

## Install

**1. Add the marketplace (one-time per machine):**

```bash
claude plugin marketplace add dorian-morones/capy-crew-agents
```

**2. Install the plugin:**

```bash
claude plugin install capy-crew-agents
```

**3. Update when new versions ship:**

```bash
claude plugin marketplace update dorian-morones/capy-crew-agents
claude plugin update capy-crew-agents
```

---

## The SDD Pipeline

Run each phase in order. You control the gates — move to the next phase only when you're satisfied with the output.

### Phase 1 — Spec

```
/capy-crew-agents:spec add CSV import for feedback
```

Runs as **willy-writter** (Haiku). Explores the codebase and writes a formal spec to `specs/<feature>.md` covering user story, acceptance criteria, API surface, data model, UI changes, and open questions.

### Phase 2 — Architecture

```
/capy-crew-agents:architect
```

Runs as **archy-architect** (Opus). Reads the spec, explores existing patterns, and appends an `## Architecture` section with DB schema, API routes, TypeScript types, frontend file decisions, and dependency order.

### Phase 3 — Task List

```
/capy-crew-agents:plan
```

Runs as **tupi-planner** (Opus). Reads the spec + architecture and writes an ordered task list to `specs/<feature>-tasks.md`. One task per layer, each ≤2 hours and independently committable.

### Phase 4 — Implementation

```
/capy-crew-agents:build Task 1
/capy-crew-agents:build Task 2
```

Runs as **bera-builder** (Sonnet). Implements exactly the specified task — reads the spec and task list first, matches existing patterns, reports what changed, and suggests a commit message. Does not commit.

---

## Agents

The crew members are also available as explicit personas for one-off sessions:

```
"use willy-writter to write a spec for [feature]"
"use archy-architect on specs/[feature].md"
"use tupi-planner on specs/[feature].md"
"use bera-builder to implement Task 1 from specs/[feature]-tasks.md"
```

| Agent | Model | Role | Writes code? |
|-------|-------|------|-------------|
| [willy-writter](./agents/willy-writter.md) | Haiku | Feature spec writer | No |
| [archy-architect](./agents/archy-architect.md) | Opus | Technical architect | No |
| [tupi-planner](./agents/tupi-planner.md) | Opus | Task list planner | No |
| [bera-builder](./agents/bera-builder.md) | Sonnet | Spec-faithful implementer | Yes |

---

## Skill Catalog

Skills load automatically when their trigger condition matches. No invocation needed.

### Define
| Skill | Trigger |
|-------|---------|
| [idea-refine](./skills/idea-refine/SKILL.md) | Idea is vague or underspecified |
| [feature-spec](./skills/feature-spec/SKILL.md) | Starting any non-trivial feature |

### Plan
| Skill | Trigger |
|-------|---------|
| [planning-and-task-breakdown](./skills/planning-and-task-breakdown/SKILL.md) | Breaking a spec into executable tasks |

### Build
| Skill | Trigger |
|-------|---------|
| [supabase-data-modeling](./skills/supabase-data-modeling/SKILL.md) | Adding tables, RLS policies, migrations |
| [api-route-design](./skills/api-route-design/SKILL.md) | Creating or modifying Elysia routes |
| [nextjs-component-patterns](./skills/nextjs-component-patterns/SKILL.md) | Building Next.js pages or components |
| [incremental-implementation](./skills/incremental-implementation/SKILL.md) | Shipping in vertical slices |

### Verify
| Skill | Trigger |
|-------|---------|
| [e2e-with-playwright](./skills/e2e-with-playwright/SKILL.md) | Writing Playwright E2E tests |
| [debugging-and-error-recovery](./skills/debugging-and-error-recovery/SKILL.md) | Stuck on a bug for more than 15 minutes |

### Review
| Skill | Trigger |
|-------|---------|
| [code-review-and-quality](./skills/code-review-and-quality/SKILL.md) | Reviewing a PR or self-reviewing |
| [security-hardening](./skills/security-hardening/SKILL.md) | Touching auth, data access, or API exposure |

### Ship
| Skill | Trigger |
|-------|---------|
| [git-workflow-and-versioning](./skills/git-workflow-and-versioning/SKILL.md) | Committing or preparing a PR |
| [vercel-render-deploy](./skills/vercel-render-deploy/SKILL.md) | Deploying to Vercel or Render |

---

## References

Quick-reference checklists used alongside skills:

- [supabase-checklist.md](./references/supabase-checklist.md) — Migrations, RLS, query patterns
- [clerk-auth-patterns.md](./references/clerk-auth-patterns.md) — JWT, webhooks, middleware
- [deployment-checklist.md](./references/deployment-checklist.md) — Render + Vercel pre-deploy
- [testing-patterns.md](./references/testing-patterns.md) — Playwright, Bun test patterns
- [security-checklist.md](./references/security-checklist.md) — OWASP, CORS, input validation
- [performance-checklist.md](./references/performance-checklist.md) — Core Web Vitals, API latency

---

## Review Personas

For focused review sessions — invoke explicitly:

```
"use code-reviewer to review this PR"
"use security-auditor on src/routes/payments.ts"
```

- [code-reviewer.md](./agents/code-reviewer.md) — correctness, security, maintainability
- [test-engineer.md](./agents/test-engineer.md) — behavior coverage, Playwright patterns
- [security-auditor.md](./agents/security-auditor.md) — auth, RLS, injection, secrets

---

## Contributing

### Adding a skill

1. Create `skills/my-skill/SKILL.md`
2. Required frontmatter:
   ```yaml
   ---
   name: my-skill
   description: Use when [specific trigger]. [One-sentence outcome].
   ---
   ```
3. Required sections: Overview → When to Use → Core Process → Techniques → Red Flags → Verification
4. Add to the Skill Catalog above

### Adding an agent

1. Add a `.md` file to `agents/`
2. Required frontmatter:
   ```yaml
   ---
   name: agent-name
   description: One sentence describing what it does and when to use it
   tools: Glob, Grep, Read, Write, Edit, Bash, LS
   model: sonnet
   color: blue
   ---
   ```

### Standards

- Skill descriptions start with `"Use when"` — this is how Claude decides whether to load the skill
- Agent instructions are direct and imperative — no hedging
- Verification sections use checkboxes — every item must be binary
- No skill should exceed 1500 lines

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full guide.
