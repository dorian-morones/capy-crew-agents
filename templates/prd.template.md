# PRD: <name>

Date: <YYYY-MM-DD> · Brief: `product/brief.md` · Status: <draft | approved>

## Summary

<Two or three sentences a non-technical client can read. What this is, who it is for,
what changes when it ships.>

## Outcomes

What must be true for a user when this ships:

1. <observable capability>
2. <observable capability>

## Flows

### <Flow name> → Outcome <N>

**Entry:** <where the user starts>

1. <step — what they do, what they see>
2. <step>

**Success:** <what is true at the end>
**Empty state:** <what they see with no data>
**Failure:** <what happens when it goes wrong, and what they see>

## Requirements

| ID | Requirement | Outcome | Priority |
|----|-------------|---------|----------|
| R1 | <what the product must do> | 1 | P0 |
| R2 | | | P1 |

*P0 = ship is meaningless without it · P1 = ship is worse without it, but still ship ·
P2 = fine in a later release. If over half are P0, priorities are not decided yet.*

## Scope

**In:** <what this release covers>
**Out:** <explicitly not doing, ever or for now>
**Later:** <deferred to a named future release>

## Success metrics

| Metric | How measured | Baseline | Target | By when |
|--------|--------------|----------|--------|---------|
| | | | | |

*No baseline means the target is unfalsifiable. If a baseline is unknown, say so here.*

## Release criteria

- [ ] <verifiable by someone other than the author>
- [ ] <at least one criterion the client signs off>

## Constraints and dependencies

- <deadline, budget, compliance, third-party service, anything outside your control>

## Risks and open questions

- `[UNKNOWN]` <carried from the brief> → <risk>
- `[ASSUMED]` <carried from the brief> → <what it rests on>

---
*No schemas, endpoints, components, or libraries. Those belong in the technical spec.*
