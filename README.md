# Capy Crew

![Capy Crew](./banner.png)

**A crew of specialist AI agents that define what to build, then build it — and never guess on your behalf.**

Two pipelines for Claude Code. One turns a vague idea into a client-ready PRD. The other turns that PRD into reviewed, committed code. Every artifact is critiqued by a different agent than the one that wrote it, and every question only you can answer comes back to you instead of being quietly invented.

Built for freelance and solo devs who ship other people's software and carry the cost when the requirements were wrong.

---

## The Two Pipelines

```
  ╭──────────────────────────── /discovery ───────────────────────────╮
  │                                                                   │
  │   interview  ──►   brief   ──►   PRD                              │   what & why
  │   what only        is this       what it                          │
  │   you know         worth it?     must do                          │
  ╰────────────────────────────────┬──────────────────────────────────╯
                                   │  you approve the handoff
  ╭────────────────────────────────▼───────── /capy ───────────────────╮
  │                                                                    │
  │   spec  ──►  architecture  ──►  tasks  ──►  build each task        │   how
  │                                              ↳ code → review → fix │
  ╰────────────────────────────────────────────────────────────────────╯
```

**`/discovery` decides what is worth building.** It interviews you one question at a time, writes a one-page brief arguing whether the problem is real, then a PRD your client can approve. A critic reviews each artifact against a fixed rubric before you see it.

**`/capy` builds it.** Spec, architecture, ordered task list, then one builder per task. Each builder spawns a coder to write the code and a reviewer to review the *uncommitted* diff, looping fixes until it's clean.

They're deliberately separate. Discovery hands off at the PRD; starting the build is always your decision.

---

## Why This Instead of Just Prompting

| | What it means in practice |
|---|---|
| **Nothing is committed unreviewed** | The coder is structurally forbidden from committing, so review happens *between* writing and committing — not as a follow-up commit after a bug already shipped |
| **No agent invents facts about you** | Claims about users cite an interview or carry an `[ASSUMED]` tag. Unknowns propagate as risks through every artifact, and the critic blocks if one silently disappears |
| **Conventions come from your repo** | `/conventions` reads your actual codebase once. Reviews then judge against *your* patterns, not a framework's defaults — the difference between a useful reviewer and a merely confident one |
| **Writers never grade themselves** | The agent that writes and the agent that judges are always different agents with separate context |
| **Loops terminate** | Two fix rounds maximum, then it stops and hands you the findings. A problem surviving two rounds is usually missing information, and more rounds can't manufacture information |
| **You hold every gate** | Spec open questions, `[DECISION]` items, `[question]` findings, and the commit itself all stop and wait for you |

---

## Install

```bash
# one-time per machine
claude plugin marketplace add dorian-morones/capy-crew-agents

# install
claude plugin install capy-crew-agents
```

All 22 skills and 12 commands are then active in every Claude Code session — no project-level `CLAUDE.md` changes needed.

```bash
# update later
claude plugin marketplace update dorian-morones/capy-crew-agents
claude plugin update capy-crew-agents
```

> The plugin installs under the id `capy-crew-agents` — that's the package name, and it's also the prefix on every skill (`capy-crew-agents:writer`). Capy Crew is the crew; `capy-crew-agents` is how you install it.

---

## Quickstart

**1. Teach it the codebase — once per project**

```
/conventions
```

Reads the stack, layout, security invariants, and build commands out of your code and writes `.capy/conventions.md`, citing a real file for every rule. Every coder and reviewer afterwards judges against that file.

Skip it and the agents infer patterns from neighbouring files — workable, but reviews get vaguer.

**2. Define the work**

```
/discovery a tool that generates client status updates from git history
```

You answer about six questions, one at a time, then approve a brief and a PRD.

**3. Build it**

```
/capy build from product/prd.md
```

Approve the spec, resolve any architecture decisions, approve the task list. Each task then arrives already written and reviewed, waiting only for your commit.

**Or run a single phase** — `/brief`, `/prd`, `/spec`, `/plan`, `/build`, `/review`. Nothing requires the full pipeline.

---

## The Crew

Twelve personas. Each runs as an isolated subagent with no memory of your conversation, so nobody inherits anyone else's assumptions.

### Discovery pipeline

| Agent | Job | Never does |
|-------|-----|-----------|
| [interviewer](./agents/interviewer.md) | Extracts what only you or your client know — one question per turn, answers recorded verbatim | Asks what the repo could answer; fills a gap with a plausible invention |
| [brief-writer](./agents/brief-writer.md) | One page: is this worth building? Problem, who has it, why now, a falsifiable success signal | Names a solution inside the problem; writes "users" where a role belongs |
| [prd-writer](./agents/prd-writer.md) | Outcomes, flows, prioritised requirements, metrics with baselines, release criteria | Names a schema, endpoint, or library; changes the brief's premise |
| [product-critic](./agents/product-critic.md) | Judges briefs and PRDs against a fixed rubric and returns a verdict | Rewrites the artifact; offers opinions on the idea's merit |

### Build pipeline

| Agent | Job | Never does |
|-------|-----|-----------|
| [writer](./agents/writer.md) | Turns a request into a spec unambiguous enough that two engineers build the same thing | Answers an open question by assumption |
| [architect](./agents/architect.md) | Every technical decision before implementation — schema, contracts, types, dependency order | Writes code; offers "option A or B" instead of deciding |
| [planner](./agents/planner.md) | An ordered task list, each task ≤2h and independently committable | Mixes concerns into one task |
| [builder](./agents/builder.md) | Runs the write → review → fix loop for one task | Writes code or reviews a diff itself — it only coordinates |
| [coder](./agents/coder.md) | Writes exactly one task, matching your codebase's patterns | Commits; touches a file the task didn't name |
| [reviewer](./agents/reviewer.md) | Reviews the uncommitted diff against your conventions and the task's done conditions | Edits code; softens a blocker to stay polite |

### Specialists

| Agent | Job |
|-------|-----|
| [security-auditor](./agents/security-auditor.md) | Audits auth flows, data access, and API exposure against your recorded security invariants |
| [test-engineer](./agents/test-engineer.md) | Writes tests for behaviour that would catch a real bug, not tests that mirror the implementation |

---

## Skills

Skills load automatically when the conversation matches their trigger. You can also name one directly, or use its slash command.

### Product — define what to build

| Skill | Command | What it does |
|-------|---------|--------------|
| [discovery](./skills/discovery/SKILL.md) | `/discovery` | Orchestrates interview → brief → PRD, with a critic loop on each artifact and approval gates between phases |
| [interviewer](./skills/interviewer/SKILL.md) | `/interview-me` | Researches the repo first, then asks one question per turn; marks gaps `[UNKNOWN]` rather than inventing them |
| [brief-writer](./skills/brief-writer/SKILL.md) | `/brief` | Writes `product/brief.md` — one page on whether this is worth building |
| [prd-writer](./skills/prd-writer/SKILL.md) | `/prd` | Writes `product/prd.md` — client-readable requirements with no implementation detail |
| [product-critic](./skills/product-critic/SKILL.md) | `/critique` | Runs a fixed rubric: unattributed user claims, unfalsifiable metrics, dropped unknowns, broken traceability |
| [idea-refine](./skills/idea-refine/SKILL.md) | — | For a feature already agreed worth building: turns it into actor, trigger, and outcome |

### Build — turn it into code

| Skill | Command | What it does |
|-------|---------|--------------|
| [capy](./skills/capy/SKILL.md) | `/capy` | Orchestrates conventions → spec → architecture → tasks → builder-per-task, gating on you between phases |
| [conventions](./skills/conventions/SKILL.md) | `/conventions` | Reads your codebase and writes `.capy/conventions.md` — the file every other skill judges against |
| [writer](./skills/writer/SKILL.md) | `/spec` | Writes `specs/<feature>.md` — user story, acceptance criteria, API surface, data model, open questions |
| [architect](./skills/architect/SKILL.md) | — | Appends `## Architecture` — schema, route contracts, types, component decisions, dependency order |
| [planner](./skills/planner/SKILL.md) | `/plan` | Writes `specs/<feature>-tasks.md` — one task per layer, each independently committable |
| [builder](./skills/builder/SKILL.md) | `/build` | Spawns coder + reviewer and loops fix rounds until the review is clean (max 2) |
| [coder](./skills/coder/SKILL.md) | — | Implements exactly one task, or applies review findings in fix mode. Never commits |
| [reviewer](./skills/reviewer/SKILL.md) | `/review` | Five-pass review — correctness → security → maintainability → tests → style — with severity tagging |

### Quality

| Skill | What it does |
|-------|--------------|
| [security-auditor](./skills/security-auditor/SKILL.md) | Account scoping, input validation, auth middleware, admin-client misuse, webhook verification, secrets |
| [test-engineer](./skills/test-engineer/SKILL.md) | Independent tests using semantic selectors, asserting on visible behaviour |
| [debugging-and-error-recovery](./skills/debugging-and-error-recovery/SKILL.md) | Hypothesis loop: form theory → minimal reproduction → confirm or refute → fix root cause |

### Stack-specific

Opt-in — these trigger on the technology rather than loading everywhere.

| Skill | What it does |
|-------|--------------|
| [supabase-data-modeling](./skills/supabase-data-modeling/SKILL.md) | Migration naming, RLS policy templates, index conventions, account-scoped queries |
| [api-route-design](./skills/api-route-design/SKILL.md) | Elysia body schema validation, auth middleware, error types, Swagger docs |
| [nextjs-component-patterns](./skills/nextjs-component-patterns/SKILL.md) | Server vs. client component decisions, data fetching, Tailwind conventions |

### Ship

| Skill | Command | What it does |
|-------|---------|--------------|
| [git-workflow-and-versioning](./skills/git-workflow-and-versioning/SKILL.md) | — | Conventional commits, atomic discipline, trunk-based branching |
| [deploy-checklist](./skills/deploy-checklist/SKILL.md) | `/ship` | Pre-deploy checklist: env vars, key alignment between services, build verification, smoke tests |

---

## What Gets Written Where

```
.capy/conventions.md          your project's rules — written once, read by every agent
product/interview-<topic>.md  what you told the interviewer, verbatim
product/brief.md              is this worth building
product/prd.md                what it must do — the client-facing artifact
specs/<feature>.md            the technical spec, with its architecture section
specs/<feature>-tasks.md      the ordered task list
```

---

## References & Templates

Static lookups used alongside skills. References are for looking things up; skills are for process.

- [supabase-checklist.md](./references/supabase-checklist.md) — Migrations, RLS, query patterns
- [clerk-auth-patterns.md](./references/clerk-auth-patterns.md) — JWT, webhooks, middleware
- [deployment-checklist.md](./references/deployment-checklist.md) — Pre-deploy verification
- [testing-patterns.md](./references/testing-patterns.md) — Playwright and unit test patterns
- [security-checklist.md](./references/security-checklist.md) — OWASP, CORS, input validation
- [performance-checklist.md](./references/performance-checklist.md) — Core Web Vitals, API latency

Templates filled in per project: [conventions](./templates/conventions.template.md) · [brief](./templates/brief.template.md) · [PRD](./templates/prd.template.md)

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md).

Key rules:
- Skill descriptions start with `"Use when"` — that string is how Claude decides to load them
- Descriptions must not overlap; two skills that could fire on the same prompt make the choice a coin flip
- Agent instructions are imperative — no hedging, no "you might want to"
- Nothing stack-specific goes in a pipeline skill; it belongs in the conventions template

---

MIT © Dorian Morones
