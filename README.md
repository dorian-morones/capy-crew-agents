# Capy Crew Agents

![capy-crew-agents](./banner.png)

Most AI-assisted features fail not because the code is wrong, but because the requirements were never pinned down. **Capy Crew Agents** is a Claude Code plugin that enforces Spec-Driven Development (SDD): a four-phase pipeline where every feature begins with a written, approved spec before a single line of code is generated. It installs four slash commands, eight agent personas, thirteen auto-loading skills, and six reference checklists into every Claude Code session.

---

## What This Gives You

| Type | Count | How it works |
|------|-------|--------------|
| **Slash commands** | 4 | Drive the four phases of the SDD pipeline — you run one per phase |
| **Agent personas** | 8 | Named roles you invoke by natural language for focused sessions |
| **Auto-loading skills** | 13 | Activate automatically when the conversation context matches their trigger — no invocation needed |
| **Reference checklists** | 6 | Static quick-lookup docs for Supabase, Clerk, deployment, testing, security, and performance |

Skills deserve a specific note: they are not commands you run. Each skill has a trigger description and Claude loads it silently when that trigger is detected. If you're touching auth code, the `security-hardening` skill loads. If you're stuck on a bug, `debugging-and-error-recovery` loads. You never call them by name.

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

After this, the commands, agents, and skills are active in every Claude Code session — no project-level `CLAUDE.md` changes required.

**3. Update when new versions ship:**

```bash
claude plugin marketplace update dorian-morones/capy-crew-agents
claude plugin update capy-crew-agents
```

---

## Quick Start: Your First Feature

Here is the full pipeline for a concrete feature: "add CSV export for the feedback table."

**Step 1 — Write the spec**

```
/capy-crew-agents:spec add CSV export for feedback table
```

`willy-writter` (Haiku) explores your codebase and writes `specs/csv-export.md` covering the user story, acceptance criteria, API surface, data model, and open questions. Review it. Edit it if needed. When it looks right, continue.

**Step 2 — Design the architecture**

```
/capy-crew-agents:architect
```

`archy-architect` (Opus) reads `specs/csv-export.md` and appends an `## Architecture` section documenting the DB schema, API routes, TypeScript types, and frontend file decisions. Look for `[DECISION]` markers — those are items that need your input before implementation starts.

**Step 3 — Break it into tasks**

```
/capy-crew-agents:plan
```

`tupi-planner` (Opus) reads the spec and architecture and writes `specs/csv-export-tasks.md`. Each task covers one layer (DB → Types → API → Hook → UI → Tests), takes at most two hours, and leaves the codebase in a working state when committed. Review the task list. You control the gate.

**Step 4 — Build one task at a time**

```
/capy-crew-agents:build Task 1
# review the diff, then: git add ... && git commit -m "..."

/capy-crew-agents:build Task 2
# review + commit
```

`bera-builder` (Sonnet) reads the spec and task list before writing any code, matches your existing patterns, and reports exactly what it changed with a suggested commit message. It never commits — you always review and commit each task before moving to the next.

---

## The SDD Pipeline

Reference for all command argument patterns, inputs, and outputs.

### Phase 1 — Spec

```bash
/capy-crew-agents:spec <feature description>

# Examples:
/capy-crew-agents:spec add CSV export for feedback table
/capy-crew-agents:spec fix pagination bug on the members list
/capy-crew-agents:spec add Stripe webhook handler for subscription events
```

| | |
|--|--|
| **Input** | Free-form description — anything you'd say to a colleague |
| **Output** | `specs/<feature-name>.md` |
| **Agent** | willy-writter |
| **Model** | Haiku (fast codebase exploration) |
| **Approval gate** | Review the spec before running Phase 2 |

The spec file follows a fixed structure: User Story, Acceptance Criteria, API Surface, Request/Response Shapes, Data Model, UI Changes, Out of Scope, and Open Questions. Open questions are surfaced explicitly — never assumed.

---

### Phase 2 — Architecture

```bash
/capy-crew-agents:architect                        # reads most recent file in specs/
/capy-crew-agents:architect specs/csv-export.md   # reads specified spec
```

| | |
|--|--|
| **Input** | Spec file |
| **Output** | `## Architecture` section appended to the spec |
| **Agent** | archy-architect |
| **Model** | Opus (architectural rigor) |
| **Approval gate** | Resolve `[DECISION]` items before running Phase 3 |

The architecture section documents complete `CREATE TABLE` statements with RLS policies, route contracts with method/path/auth/body/response, TypeScript interfaces, frontend file decisions (server vs. client with justification), and a dependency order list.

---

### Phase 3 — Task List

```bash
/capy-crew-agents:plan                             # reads most recent spec in specs/
/capy-crew-agents:plan specs/csv-export.md        # reads specified spec
```

| | |
|--|--|
| **Input** | Spec + architecture |
| **Output** | `specs/<feature-name>-tasks.md` |
| **Agent** | tupi-planner |
| **Model** | Opus |
| **Approval gate** | Review the task list before running Phase 4 |

Each task is one layer, at most two hours, with observable done conditions and a suggested conventional commit message. The codebase must build and be committable after every task.

---

### Phase 4 — Implementation

```bash
/capy-crew-agents:build Task 1
/capy-crew-agents:build Task 3: add useResource hook           # name speeds up lookup
/capy-crew-agents:build Task 5 from specs/csv-export-tasks.md  # explicit file for multi-spec sessions
```

| | |
|--|--|
| **Input** | Task reference (number, optional name, optional file path) |
| **Output** | Code changes + completion report + suggested commit |
| **Agent** | bera-builder |
| **Model** | Sonnet |
| **Gate between tasks** | Review diff, commit, then run the next task |

bera-builder reads the full spec and task list before writing any code, finds one or two existing similar files to match patterns, implements exactly the task and nothing more, and reports which files changed. It never commits.

---

## Agents

### Core SDD Agents

These run automatically when you use the slash commands. You can also invoke them directly for one-off sessions — for example, to re-run architecture after changing a spec, or to ask willy-writter to revise a section.

| Agent | Model | Role | Writes code? |
|-------|-------|------|-------------|
| [willy-writter](./agents/willy-writter.md) | Haiku | Feature spec writer | No |
| [archy-architect](./agents/archy-architect.md) | Opus | Technical architect | No |
| [tupi-planner](./agents/tupi-planner.md) | Opus | Task list planner | No |
| [bera-builder](./agents/bera-builder.md) | Sonnet | Spec-faithful implementer | Yes |

**Direct invocation:**

```
"use willy-writter to write a spec for adding Stripe webhooks"
"use archy-architect on specs/stripe-webhooks.md"
"use tupi-planner on specs/stripe-webhooks.md"
"use bera-builder to implement Task 3 from specs/stripe-webhooks-tasks.md"
```

---

### Specialist Reviewers

These are never invoked by the pipeline. Call them explicitly when you want focused review.

| Agent | Role | When to use |
|-------|------|------------|
| [code-reviewer](./agents/code-reviewer.md) | Correctness, security, maintainability | PR review or self-review before merging |
| [test-engineer](./agents/test-engineer.md) | Behavior coverage, Playwright patterns | Before merging a feature |
| [security-auditor](./agents/security-auditor.md) | Auth, RLS, injection, secrets | Any time you touch auth or data access |

**Direct invocation:**

```
"use code-reviewer to review this PR"
"use test-engineer to write E2E tests for the CSV export feature"
"use security-auditor on src/routes/payments.ts"
```

---

### Capy — Full Pipeline Orchestrator

[capy](./agents/capy.md) coordinates the entire four-phase pipeline, spawning each sub-agent in turn and enforcing approval gates. Use this when you want to hand off a feature entirely and just respond to prompts.

```
"use capy to build the CSV export feature end to end"
```

Capy presents spec summaries, asks for approval at gates, iterates through build tasks, and prompts you to commit after each one. You control every gate; capy handles all the coordination.

---

## Skills

Skills are engineering process guides — decision frameworks, checklists, and code patterns encoded in Markdown. Each skill has a `description` frontmatter field starting with "Use when...". When Claude Code detects the trigger condition in the conversation, the skill's content loads automatically. You never invoke a skill by name.

Skills are not commands or agents. Think of them as internal playbooks that activate silently when relevant and stay out of the way when not.

### Define

| Skill | Trigger | When active |
|-------|---------|-------------|
| [idea-refine](./skills/idea-refine/SKILL.md) | Idea is vague or underspecified | Asks structured clarifying questions to turn a vague need into a specific technical direction |
| [feature-spec](./skills/feature-spec/SKILL.md) | Starting any non-trivial feature | Enforces formal spec structure — user story, testable acceptance criteria, API surface, data model |

### Plan

| Skill | Trigger | When active |
|-------|---------|-------------|
| [planning-and-task-breakdown](./skills/planning-and-task-breakdown/SKILL.md) | Breaking a spec into executable tasks | Enforces vertical slice decomposition — one task per layer, each independently committable |

### Build

| Skill | Trigger | When active |
|-------|---------|-------------|
| [supabase-data-modeling](./skills/supabase-data-modeling/SKILL.md) | Adding tables, RLS policies, or writing migrations | Enforces migration naming, RLS policy templates, index conventions, and account-scoped queries |
| [api-route-design](./skills/api-route-design/SKILL.md) | Creating or modifying Elysia routes | Enforces body schema validation, auth middleware patterns, error types, and Swagger documentation |
| [nextjs-component-patterns](./skills/nextjs-component-patterns/SKILL.md) | Building Next.js pages or components | Guides server vs. client component decisions, data fetching patterns, and Tailwind conventions |
| [incremental-implementation](./skills/incremental-implementation/SKILL.md) | Implementing any feature | Enforces thin vertical slices — each slice is tested, committed, and leaves the codebase working |

### Verify

| Skill | Trigger | When active |
|-------|---------|-------------|
| [e2e-with-playwright](./skills/e2e-with-playwright/SKILL.md) | Writing Playwright E2E tests | Guides authenticated test setup, semantic selectors, and behavior-focused assertions |
| [debugging-and-error-recovery](./skills/debugging-and-error-recovery/SKILL.md) | Stuck on a bug or unexpected behavior | Runs a hypothesis loop: form theory → find minimal reproduction → confirm or refute → fix root cause |

### Review

| Skill | Trigger | When active |
|-------|---------|-------------|
| [code-review-and-quality](./skills/code-review-and-quality/SKILL.md) | Reviewing a PR or self-reviewing | Runs a five-dimension review: correctness → security → maintainability → tests → style |
| [security-hardening](./skills/security-hardening/SKILL.md) | Touching auth, data access, or API exposure | Checks account scoping, JWT validation, RLS enforcement, webhook signatures, and secret handling |

### Ship

| Skill | Trigger | When active |
|-------|---------|-------------|
| [git-workflow-and-versioning](./skills/git-workflow-and-versioning/SKILL.md) | Committing, branching, or preparing a PR | Enforces conventional commits, atomic discipline, and trunk-based branching |
| [vercel-render-deploy](./skills/vercel-render-deploy/SKILL.md) | Deploying to Vercel or Render | Runs a pre-deploy checklist: env vars, build verification, auth key alignment, smoke tests |

---

## References

Static quick-reference checklists used alongside skills. References are for lookup; skills are for process. When a skill is active, it may point to a reference for additional detail.

- [supabase-checklist.md](./references/supabase-checklist.md) — Migrations, RLS, query patterns
- [clerk-auth-patterns.md](./references/clerk-auth-patterns.md) — JWT, webhooks, middleware
- [deployment-checklist.md](./references/deployment-checklist.md) — Render + Vercel pre-deploy
- [testing-patterns.md](./references/testing-patterns.md) — Playwright, Bun test patterns
- [security-checklist.md](./references/security-checklist.md) — OWASP, CORS, input validation
- [performance-checklist.md](./references/performance-checklist.md) — Core Web Vitals, API latency

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full guide.

Key rules:
- Skill descriptions must start with `"Use when"` — this is how Claude decides when to load them
- Agent instructions are imperative — no hedging, no "you might want to"
- No skill exceeds 1500 lines
