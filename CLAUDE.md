# dmx-agents — Claude Instructions

This repository contains personal agent skills for use with Claude Code.

## How Skills Load

Skills are discovered via their `description` frontmatter field. Claude reads descriptions injected into the system prompt and activates the relevant skill when the trigger condition matches.

Each skill lives at `skills/<name>/SKILL.md` and follows this structure:
- YAML frontmatter with `name` and `description: Use when...`
- Overview, When to Use, Core Process, Techniques, Common Rationalizations, Red Flags, Verification

## Using Skills in a Project

Add this to the project's `CLAUDE.md`:

```markdown
## Agent Skills
Skills are available from ~/.dmx-agents. Load the relevant skill when the trigger condition matches.
```

Or reference specific skills explicitly:

```markdown
When building Elysia routes, use the api-route-design skill from ~/.dmx-agents/skills/api-route-design/SKILL.md
```

## Slash Commands

Available commands (when this repo is active):

| Command | Trigger |
|---------|---------|
| `/spec` | Start feature-spec skill |
| `/plan` | Start planning-and-task-breakdown skill |
| `/review` | Start code-review-and-quality skill |
| `/ship` | Start vercel-render-deploy skill |

## Repository Structure

```
skills/          13 skill files — executable engineering processes
agents/          3 agent personas — specialized roles for sessions
references/      6 reference checklists — quick lookups
docs/            Getting started and skill anatomy guides
hooks/           Session lifecycle hooks (future use)
```
