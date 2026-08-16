---
description: Review the uncommitted diff or a PR for correctness, security, and maintainability
---

Review: $ARGUMENTS

Invoke the `capy-crew-agents:reviewer` skill. Default to the uncommitted diff (`git diff` plus untracked files) when no target is given. Read `.capy/conventions.md` first — review against this project's conventions, not framework defaults. Tag every finding `[blocker]`, `[concern]`, `[nit]`, or `[question]`.
