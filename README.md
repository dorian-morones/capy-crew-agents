# dmx-agents

A personal library of agent skills, reference checklists, and agent personas built for my development process.

Structured after [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) — the same format, customized for my stack: **Bun/Elysia API**, **Next.js App Router**, **Supabase**, **Clerk**, **Render**, and **Vercel**.

## How It Works

Skills are markdown files with YAML frontmatter. Claude Code (and compatible agents) discover them by reading the `description` field. Each skill encodes a repeatable engineering process — not documentation, but step-by-step executable workflows with verification checkpoints.

## Install

```bash
claude mcp add --transport http dmx-agents https://raw.githubusercontent.com/dorian-morones/dmx-agents/main
```

Or clone locally and reference from your project's `CLAUDE.md`:

```bash
git clone https://github.com/dorian-morones/dmx-agents ~/.dmx-agents
```

## Skill Catalog

### Define
| Skill | Description |
|-------|-------------|
| [idea-refine](./skills/idea-refine/SKILL.md) | Use when an idea is vague or underspecified before writing any code |
| [feature-spec](./skills/feature-spec/SKILL.md) | Use when starting any non-trivial feature or change |

### Plan
| Skill | Description |
|-------|-------------|
| [planning-and-task-breakdown](./skills/planning-and-task-breakdown/SKILL.md) | Use when breaking down a spec into executable tasks |

### Build
| Skill | Description |
|-------|-------------|
| [incremental-implementation](./skills/incremental-implementation/SKILL.md) | Use when implementing any feature to ensure working software at every step |
| [supabase-data-modeling](./skills/supabase-data-modeling/SKILL.md) | Use when adding tables, columns, RLS policies, indexes, or migrations |
| [api-route-design](./skills/api-route-design/SKILL.md) | Use when creating or modifying Elysia API routes |
| [nextjs-component-patterns](./skills/nextjs-component-patterns/SKILL.md) | Use when building Next.js pages, layouts, or components |

### Verify
| Skill | Description |
|-------|-------------|
| [e2e-with-playwright](./skills/e2e-with-playwright/SKILL.md) | Use when writing or updating Playwright E2E tests |
| [debugging-and-error-recovery](./skills/debugging-and-error-recovery/SKILL.md) | Use when stuck on a bug or unexpected behavior for more than 15 minutes |

### Review
| Skill | Description |
|-------|-------------|
| [code-review-and-quality](./skills/code-review-and-quality/SKILL.md) | Use when reviewing a PR or self-reviewing before pushing |
| [security-hardening](./skills/security-hardening/SKILL.md) | Use when touching auth, data access, input handling, or API exposure |

### Ship
| Skill | Description |
|-------|-------------|
| [git-workflow-and-versioning](./skills/git-workflow-and-versioning/SKILL.md) | Use when committing, branching, or preparing a PR |
| [vercel-render-deploy](./skills/vercel-render-deploy/SKILL.md) | Use when deploying the frontend to Vercel or the API to Render |

## References

Quick-reference checklists used alongside skills:

- [supabase-checklist.md](./references/supabase-checklist.md) — Migrations, RLS, query patterns
- [clerk-auth-patterns.md](./references/clerk-auth-patterns.md) — JWT, webhooks, middleware
- [deployment-checklist.md](./references/deployment-checklist.md) — Render + Vercel pre-deploy
- [testing-patterns.md](./references/testing-patterns.md) — Playwright, Jest, Testing Library
- [security-checklist.md](./references/security-checklist.md) — OWASP, headers, CORS
- [performance-checklist.md](./references/performance-checklist.md) — Core Web Vitals, API latency

## Capy-Crew — SDD Agent Pipeline

A set of Claude Code subagents that enforce **Spec-Driven Development**: no code is written until a spec is approved.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Feature Request                              │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
              ┌────────────────────────────────┐
              │             capy               │  ← orchestrator
              │  Coordinates the full pipeline │
              │  with approval gates           │
              └───────────────┬────────────────┘
                              │
               ┌──────────────▼──────────────┐
               │         willy-writer          │
               │  Reads codebase, writes spec │
               │  specs/<feature>.md          │
               └──────────────┬──────────────┘
                              │
                   ✋ Developer approves spec
                              │
               ┌──────────────▼──────────────┐
               │       archy-architect         │
               │  DB schema, API contracts,   │
               │  component tree              │
               └──────────────┬──────────────┘
                              │
               ┌──────────────▼──────────────┐
               │        tupi-planner         │
               │  Ordered atomic task list    │
               │  specs/<feature>-tasks.md    │
               └──────────────┬──────────────┘
                              │
                   ✋ Developer approves tasks
                              │
               ┌──────────────▼──────────────┐
               │         bera-builder         │
               │  Task 1 → commit             │
               │  Task 2 → commit             │
               │  Task N → commit             │
               └─────────────────────────────┘
```

### How to use

**Full pipeline (recommended):**
```
"Use capy to build [feature request]"
```

**Individual agents:**
```
"Use willy-writer to write a spec for [feature]"
"Use archy-architect-architect on specs/[feature].md"
"Use tupi-planner on specs/[feature].md"
"Use bera-builder to implement Task 1 from specs/[feature]-tasks.md"
```

### Agent reference

| Agent | Role | Writes code? |
|-------|------|-------------|
| [capy](./.claude/agents/capy.md) | Pipeline orchestrator | No |
| [willy-writer](./.claude/agents/willy-writer.md) | Feature spec writer | No |
| [archy-architect](./.claude/agents/archy-architect.md) | Technical architect | No |
| [tupi-planner](./.claude/agents/tupi-planner.md) | Task list planner | No |
| [bera-builder](./.claude/agents/bera-builder.md) | Spec-faithful implementer | Yes |

## Agent Personas

Personas for focused review sessions (loaded as context, not subagents):

- [code-reviewer.md](./agents/code-reviewer.md)
- [test-engineer.md](./agents/test-engineer.md)
- [security-auditor.md](./agents/security-auditor.md)

## Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 16 (App Router), TypeScript, Tailwind CSS v4 |
| Backend | Bun, Elysia |
| Database | Supabase (PostgreSQL) |
| Auth | Clerk |
| State | Zustand |
| Testing | Playwright |
| Deploy (FE) | Vercel → app.useactify.com |
| Deploy (API) | Render → api.useactify.com |
| Monitoring | PostHog (events + logs via OpenTelemetry) |

## Docs

- [Getting Started](./docs/getting-started.md)
- [Skill Anatomy](./docs/skill-anatomy.md)
- [Contributing](./CONTRIBUTING.md)
