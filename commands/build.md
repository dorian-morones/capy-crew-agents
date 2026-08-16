---
description: Deliver one task written, reviewed, and fixed
---

Deliver this task: $ARGUMENTS

Invoke the `capy-crew-agents:builder` skill. It spawns a coder to write the task and a reviewer to review the uncommitted diff, looping fix rounds until the review is clean (at most two). Do not commit — report the result and let the developer commit.
