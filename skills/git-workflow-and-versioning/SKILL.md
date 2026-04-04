---
name: git-workflow-and-versioning
description: Use when committing, branching, or merging to keep a clean, readable git history that supports safe rollbacks.
---

## Overview

Git history is the primary record of why the codebase is the way it is. A well-maintained history makes rollbacks safe, code review fast, and blame useful. This skill encodes the workflow conventions for the actify repos — trunk-based development, conventional commits, and atomic commit discipline — and explains why each convention exists.

## When to Use

**Use this skill when:**
- Starting a new feature branch
- Writing a commit message
- Opening a PR
- Deciding how to structure a set of changes into commits
- Performing a rollback or bisect

**Skip this skill when:**
- Making a one-liner fix directly on main for a critical hotfix (just commit it)

## Core Process

1. **Branch from main** — Create a short-lived feature branch. Name it `type/short-description`.

2. **Commit atomically** — One logical change per commit. Each commit should leave the codebase working.

3. **Use conventional commits** — `type: subject` format. No ambiguity about what a commit does.

4. **Keep branches short-lived** — Feature branches older than 2-3 days accumulate merge conflicts. Merge often.

5. **Open a PR** — Link to the ticket or spec. PR description explains the "why," not just the "what."

6. **Merge** — Squash merge for small features. Merge commit for large features with meaningful intermediate history.

7. **Delete the branch** — After merge, delete the branch. Keep the remote clean.

## Specific Techniques

### Branch Naming

```
type/short-description

feat/feedback-csv-import
fix/auth-token-expiry
chore/update-dependencies
refactor/feedback-hook-cleanup
docs/api-readme-update
```

Keep branch names lowercase, hyphenated, under 50 characters.

### Conventional Commit Format

```
<type>(<scope>): <subject>

type    → feat | fix | chore | docs | refactor | test | perf
scope   → optional, the area of the codebase (auth, feedback, api, etc.)
subject → imperative mood, lowercase, no period, under 72 chars

Examples:
feat(feedback): add CSV import endpoint
fix(auth): handle expired Clerk tokens gracefully
chore: update @opentelemetry packages
refactor(hooks): extract useFeedback from FeedbackPage
test(e2e): add feedback submission smoke test
docs: update API README with Render deployment steps
```

### Atomic Commit Checklist

Before committing, verify:
- [ ] This commit does exactly one logical thing
- [ ] The codebase compiles and tests pass with this commit applied
- [ ] The commit message describes what changed and (if not obvious) why
- [ ] No debug logs, commented-out code, or TODO comments in this commit

### What Goes in the PR Description

```markdown
## What
Brief description of the change.

## Why
What problem does this solve? Link to ticket or spec if applicable.

## How
Any non-obvious implementation decisions.

## Test plan
- [ ] Smoke tested locally
- [ ] E2E test added / existing tests pass
- [ ] Deployed to staging and verified (if applicable)
```

The "Why" is the most important section. The diff shows the what. The description explains the why.

### Commit Sequence for a Feature

```bash
# 1. DB layer
git commit -m "feat(db): add source_type column to feedback table"

# 2. API layer
git commit -m "feat(api): add POST /feedback with source_type validation"

# 3. Hook layer
git commit -m "feat(hooks): add createFeedback to useFeedback hook"

# 4. UI layer
git commit -m "feat(ui): add FeedbackForm with source type selector"

# 5. Test
git commit -m "test(e2e): add feedback creation flow smoke test"
```

Each commit is a save point. If the UI breaks, you can revert just the UI commit without touching the API.

### Rollback Strategy

```bash
# Revert a specific commit (creates a new revert commit — safe)
git revert <commit-sha>

# Find the last working commit (if tests are failing after a merge)
git bisect start
git bisect bad HEAD
git bisect good <last-known-good-sha>
# → runs binary search to find the first bad commit
```

Never `git reset --hard` on a branch that's been pushed — it rewrites shared history.

### Hotfix Pattern

For urgent production fixes:

```bash
# Fix directly on main (if it's a one-liner)
git commit -m "fix(auth): handle null user in auth middleware"
git push origin main

# OR: branch from the production tag if main has diverged
git checkout -b hotfix/auth-null-user v1.2.0
# fix, commit, PR, merge, tag
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll clean up the commits before merging" | You won't. Commit cleanly from the start. |
| "The commit message doesn't matter much" | Commit messages are the searchable history of why code exists. Bad messages mean `git log` is useless for debugging. |
| "I'll put everything in one PR — it's all related" | Related changes that affect different layers should still be sequenced commits. A 1000-line PR is a review burden. |
| "Force-push to clean up history on my branch" | Fine on your local branch before it's reviewed. Never force-push after someone else has looked at it. |
| "I'll merge main into my branch weekly" | Daily or every commit is better. Merge conflicts grow quadratically with time. |

## Red Flags

- Commit message: "fix", "wip", "asdf", "changes", "updates"
- Single commit containing DB migration + API route + frontend + tests
- Branch open for more than a week
- `git add .` without reviewing what's being staged
- Secrets accidentally committed (check with `git diff --staged` before committing)
- Merge conflicts larger than the feature itself

## Verification

- [ ] Branch name follows `type/description` convention
- [ ] Each commit is atomic — one logical change, codebase working after each
- [ ] Commit messages use conventional commit format
- [ ] `git diff --staged` reviewed before every commit — no secrets, no debug logs
- [ ] PR description explains the "why", not just the "what"
- [ ] PR linked to ticket or spec
- [ ] Branch deleted after merge
