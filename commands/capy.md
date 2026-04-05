---
description: Run the full Spec-Driven Development pipeline with the capy-crew — spec, architecture, task breakdown, and implementation with approval gates at each phase.
argument-hint: Feature description (e.g. "add CSV import for feedback")
---

# Capy Crew — SDD Pipeline

You are kicking off the capy-crew Spec-Driven Development pipeline for the following feature request:

**$ARGUMENTS**

Launch the `capy` agent to orchestrate the full pipeline:

> "Use capy to run the full SDD pipeline for this feature: $ARGUMENTS"

Capy will coordinate the crew in order:
1. **willy-writter** — writes the spec (`specs/<feature>.md`)
2. **archy-architect** — designs the technical architecture
3. **tupi-planner** — produces the ordered task list (`specs/<feature>-tasks.md`)
4. **bera-builder** — implements each task, one commit at a time

Developer approval is required before the spec is accepted and before implementation begins.
