# capy-crew-agents — Claude Instructions

This repository contains personal agent skills for use with Claude Code.

## Project Conventions

Skills are stack-agnostic. What "correct" means in a given codebase lives in that project's `.capy/conventions.md`, written once by the `conventions` skill and read by every coder and reviewer subagent. Never hardcode a stack assumption into a skill — put it in the conventions template instead.

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
| `/capy` | Run the full pipeline end to end |
| `/conventions` | Write `.capy/conventions.md` for this project |
| `/spec` | Write a feature spec (writer skill) |
| `/plan` | Break a spec into tasks (planner skill) |
| `/build` | Deliver one task, written + reviewed (builder skill) |
| `/review` | Review the uncommitted diff (reviewer skill) |
| `/ship` | Run the pre-deploy checklist |

## Repository Structure

```
skills/          17 skill files — executable engineering processes
agents/          8 agent personas — the subagents the pipeline spawns
commands/        7 slash commands — entry points to the skills
references/      6 reference checklists — quick lookups
templates/       conventions template, filled in per project
specs/           generated specs and task lists (per project)
```
