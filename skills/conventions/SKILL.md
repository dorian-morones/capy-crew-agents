---
name: conventions
description: Use when setting up the capy pipeline in a codebase for the first time, or when a coder or reviewer needs project conventions that do not exist yet. Inspects the codebase and writes .capy/conventions.md.
---

## Overview

The conventions skill writes `.capy/conventions.md` — the file that tells every other skill in this pipeline what "correct" means *in this codebase*. It is the difference between a reviewer that files real blockers and one that files confident nonsense about a framework the project does not use.

Conventions are not invented. They are read out of the code that already exists. If the project puts data fetching in `src/hooks/`, that is the convention — not because it is best practice, but because it is what this codebase does, and a task that breaks the pattern is a task that costs the next reader time.

This skill runs once per project. After that, the coder and reviewer read the file it produced. Re-run it when the stack changes materially — a new framework, a new auth provider, a new data layer.

## When to Use

**Use this skill when:**
- Starting capy in a codebase that has no `.capy/conventions.md`
- A coder or reviewer reports that project conventions are missing or wrong
- The project adopted a new framework, auth provider, or data layer

**Skip this skill when:**
- `.capy/conventions.md` exists and still matches the codebase — read it instead
- The project is empty (there are no patterns to read yet — write the conventions by hand as decisions, and mark them `[ASSUMED]`)

## Core Process

1. **Identify the stack** — Read `package.json`, `requirements.txt`, `Gemfile`, `go.mod`, `composer.json`, or the equivalent. Record the framework, language, test runner, database client, and auth provider with their major versions. Do not guess from directory names alone.

2. **Find the layer boundaries** — Locate where each kind of code actually lives: routes/controllers, data access, business logic, UI components, hooks/composables, tests, migrations. Record real paths from this repo, not idiomatic ones from the framework's docs.

3. **Read three real examples per layer** — For each layer, open two or three existing files and extract the *repeated* pattern: how errors are raised, how input is validated, how auth context is obtained, how responses are shaped, how files are named. A pattern that appears once is a data point; a pattern that appears three times is a convention.

4. **Find the security invariants** — This is the highest-value section. Determine how the project scopes data to a user or tenant, where that identity comes from (session, JWT claim, header), and which client bypasses those checks (an admin/service-role client, a raw DB connection). Record the rule and the bypass, because the bypass is what reviewers must hunt for.

5. **Find the verification commands** — The exact commands for typecheck, lint, unit tests, E2E tests, and build. Copy them from `package.json` scripts, `Makefile`, or CI config. These are what the coder runs as its build check.

6. **Write `.capy/conventions.md`** — Use the template in `templates/conventions.template.md`. Every rule must be traceable to code you read; cite an example file path for each.

7. **Mark what you could not determine** — Anything you inferred rather than observed gets `[ASSUMED]`. Anything genuinely absent gets `[NONE FOUND]`. Present these to the developer for confirmation. Never write a convention you did not see in the code and did not have confirmed.

8. **Show the developer the result** — Conventions drive every future review. Getting them wrong is expensive and quiet. Ask for an explicit confirmation before the pipeline relies on the file.

## Specific Techniques

### Observed, Not Recommended

Write what the codebase does, not what it should do. If the project throws raw `Error` everywhere, the convention is raw `Error` — record it, and note improving it as a separate concern. A conventions file that describes an aspirational codebase makes every reviewer round file findings against code that was correct.

If a pattern is genuinely inconsistent — half the routes validate input, half do not — record it as `[INCONSISTENT]` with both variants and let the developer choose which becomes the rule. Do not silently pick the one you prefer.

### Cite Your Source

Every rule carries an example:

```markdown
- Errors are raised with `ApiError(status, message, code)`, never raw `Error`
  → see `src/routes/feedback.routes.ts:41`
```

A cited rule can be verified and updated by the next reader. An uncited rule is folklore, and folklore is what makes a reviewer wrong six months later.

### Security Invariants Get Extra Scrutiny

Read at least five files when determining the tenancy rule, and state it as a testable sentence:

> Every query that reads tenant data must filter on `account_id` taken from `user.account_id` (the JWT claim), never from request input. The `supabaseAdmin` client bypasses row-level security and must not appear in any route that has already run the `auth` middleware.

That sentence is what a reviewer converts into a `[blocker]`. Vague phrasing here produces vague findings.

### Keep It Short

Target 100–200 lines. This file is read by every coder and reviewer subagent on every task, so length is a real cost. Rules that apply to one file belong in that file, not here. If a section grows past a page, it is documentation, not conventions.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "It's a Next.js project, I know the conventions" | You know the framework's defaults. You do not know this team's. Read the code. |
| "I'll write the best-practice version and they can adjust it" | Then every reviewer round files findings against correct code. Record what is there. |
| "One example file is enough to establish the pattern" | One is a data point. Three is a convention. |
| "I couldn't find the auth pattern, I'll leave that section out" | Mark it `[NONE FOUND]` and ask. A missing security invariant is the one gap that produces real vulnerabilities. |
| "The developer can read the file later" | They will not. Show it and get explicit confirmation now, while it is cheap to fix. |

## Red Flags

- A rule with no example file path
- Conventions copied from framework documentation rather than read from this repo
- The security invariants section left empty or vague
- Verification commands that were guessed rather than copied from `package.json` or CI config
- Inconsistent patterns silently resolved to the author's preference
- A conventions file over 300 lines
- Writing the file without showing it to the developer for confirmation

## Verification

- [ ] `.capy/conventions.md` exists and follows the template
- [ ] Every rule cites at least one real file path in this repo
- [ ] Stack and versions were read from a manifest file, not inferred
- [ ] The tenancy/authorization rule is stated as one testable sentence, including which client bypasses it
- [ ] Verification commands were copied from the project's scripts or CI config and actually run
- [ ] Everything inferred is marked `[ASSUMED]`; everything absent is marked `[NONE FOUND]`
- [ ] The developer explicitly confirmed the file before the pipeline used it
