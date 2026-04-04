---
name: incremental-implementation
description: Use when implementing any feature to ensure working software at every step and enable safe, atomic rollbacks.
---

## Overview

Incremental implementation means building software in vertical slices — thin, end-to-end pieces that work at every step — rather than building all layers before testing any of them. The goal is that the codebase is always in a shippable state, every increment is independently revertable, and you never spend more than ~2 hours without a working checkpoint.

This is the antidote to "almost done" — the state where 90% of a feature is built but 0% is shippable.

## When to Use

**Use this skill whenever implementing a feature from a spec.** There is no task too small to benefit from this.

## Core Process

1. **Identify the thinnest possible vertical slice** — What is the smallest end-to-end piece that has a testable outcome? (DB → API → UI, or just one layer if others already exist)

2. **Build it** — Implement only that slice. Nothing more.

3. **Test it** — Does it work? Run tests. Manually verify. Check the logs.

4. **Commit it** — One atomic commit. Message: `feat: [what this slice does]`

5. **Repeat** — Pick the next slice and go back to step 1.

## Specific Techniques

### The Save Point Pattern

Every successful increment gets a commit. If the next increment breaks something, you have a clean revert target:

```bash
git add src/routes/feedback.routes.ts
git commit -m "feat: add GET /feedback endpoint with account scoping"
# → working save point

# Now add the next slice
git add src/routes/feedback.routes.ts
git commit -m "feat: add POST /feedback endpoint with validation"
# → another save point
```

### Slice Sizing

| Size | Description | Max time |
|------|-------------|---------|
| Too small | A single variable rename | Not worth a commit |
| Right | One route, one component, one migration | ~1-2 hours |
| Too large | "The entire feedback feature" | Split it |

### Change One Thing Per Increment

Don't mix concerns in a single increment:
- ❌ Add migration + route + frontend component in one commit
- ✅ Migration commit → route commit → component commit

Each commit should be independently understandable and revertable.

### Feature Flags for Incomplete Work

If an increment changes shared code but the feature isn't ready to show:

```typescript
// In env config
featureFlags: {
  csvImport: process.env.FEATURE_CSV_IMPORT === "true",
}

// In route
if (!config.featureFlags.csvImport) {
  throw new ApiError(404, "Not found", "NOT_FOUND");
}
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll commit when it's all done" | "All done" often takes days. Large uncommitted changes are a risk. Commit working increments. |
| "Let me just quickly add…" | Every "quickly" extends the scope. Finish the current slice before starting a new one. |
| "I'll test at the end" | Bugs found at the end require tracing through every change since the last working state. Test each increment. |
| "These changes are related so they should be one commit" | Related doesn't mean same. If they're independently revertable, they should be separate commits. |

## Red Flags

- More than ~100 lines changed without a commit
- Multiple features or concerns changed in a single commit
- Uncommitted changes sitting for more than a day
- A "WIP" commit that bundles 10 different changes
- Starting a new slice before the current one is tested and committed

## Verification

- [ ] Feature broken into slices of ~1-2 hours each
- [ ] Each slice committed independently with a descriptive message
- [ ] Each commit leaves the codebase in a working state
- [ ] Tests run after each increment
- [ ] No uncommitted changes older than a few hours
