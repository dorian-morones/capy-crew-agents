---
description: Run the pre-deploy checklist before shipping to production
---

Pre-deploy check for: $ARGUMENTS

Invoke the `capy-crew-agents:deploy-checklist` skill. Verify environment variables, key alignment between services, build success, and smoke tests. Take real domains, services, and env var names from `.capy/conventions.md` or the project's deploy config — never assume the placeholders in the skill.
