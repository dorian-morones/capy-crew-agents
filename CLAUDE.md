# capy-crew-agents — Claude Instructions

This repository contains personal agent skills for use with Claude Code.

## Project Conventions

Skills are stack-agnostic. What "correct" means in a given codebase lives in that project's `.capy/conventions.md`, written once by the `conventions` skill and read by every coder and reviewer subagent. Never hardcode a stack assumption into a skill — put it in the conventions template instead.

## Two Pipelines

`discovery` defines what is worth building (interview → brief → PRD). `capy` builds it (spec → architecture → tasks → code). Discovery hands off at the PRD; starting the build is a separate decision.

Both follow the same shape: an orchestrator spawns one subagent per phase, each phase's artifact is critiqued by a separate agent before the developer sees it, and no agent ever answers a question that belongs to a human.

## How Skills Load

Skills are discovered via their `description` frontmatter field. Claude reads descriptions injected into the system prompt and activates the relevant skill when the trigger condition matches.

Each skill lives at `skills/<name>/SKILL.md` and follows this structure:
- YAML frontmatter with `name` and `description: Use when...`
- Overview, When to Use, Core Process, Techniques, Common Rationalizations, Red Flags, Verification

## Using Skills in a Project

Add this to the project's `CLAUDE.md`:

```markdown
## Agent Skills
Skills are available from ~/.capy-crew-agents. Load the relevant skill when the trigger condition matches.
```

Or reference specific skills explicitly:

```markdown
When building Elysia routes, use the api-route-design skill from ~/.capy-crew-agents/skills/api-route-design/SKILL.md
```

## Slash Commands

Available commands (when this repo is active):

| Command | Trigger |
|---------|---------|
| `/discovery` | Run the product definition pipeline (interview → brief → PRD) |
| `/interview-me` | Extract product knowledge only you have |
| `/brief` | Write a one-page product brief |
| `/prd` | Turn an approved brief into product requirements |
| `/critique` | Review a brief or PRD against the product rubric |
| `/capy` | Run the build pipeline end to end |
| `/conventions` | Write `.capy/conventions.md` for this project |
| `/spec` | Write a feature spec (writer skill) |
| `/plan` | Break a spec into tasks (planner skill) |
| `/build` | Deliver one task, written + reviewed (builder skill) |
| `/review` | Review the uncommitted diff (reviewer skill) |
| `/ship` | Run the pre-deploy checklist |

## Repository Structure

```
skills/          22 skill files — executable engineering processes
agents/          12 agent personas — the subagents the pipeline spawns
commands/        12 slash commands — entry points to the skills
references/      6 reference checklists — quick lookups
templates/       conventions, brief, and PRD templates
product/         generated briefs, PRDs, interviews (per project)
specs/           generated specs and task lists (per project)
```
