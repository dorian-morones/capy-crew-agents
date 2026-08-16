# Project Conventions

> Written by the `conventions` skill. Read by every coder and reviewer subagent.
> Every rule must cite a real file in this repo. Mark inferences `[ASSUMED]`,
> gaps `[NONE FOUND]`, and unresolved variation `[INCONSISTENT]`.

## Stack

| Layer | Choice | Version |
|-------|--------|---------|
| Language | | |
| Backend framework | | |
| Frontend framework | | |
| Database / client | | |
| Auth provider | | |
| Test runner | | |
| Package manager | | |

## Layout

Where each kind of code lives in this repo:

| Kind | Path | Example |
|------|------|---------|
| Routes / controllers | | |
| Data access | | |
| Business logic | | |
| UI components | | |
| Data fetching | | |
| Types | | |
| Migrations | | |
| Tests | | |

## Security Invariants

**The tenancy rule** — state as one testable sentence, including where identity comes from:

> _e.g. Every query reading tenant data filters on `account_id` from `user.account_id`
> (the JWT claim), never from request input._

**Bypasses** — clients or code paths that skip the above, and where they are permitted:

> _e.g. `supabaseAdmin` bypasses row-level security; permitted only in webhook handlers
> registered before the auth middleware. → `src/webhooks/stripe.ts:12`_

**Input validation** — what validates external input, and where it is required:

**Secrets** — how they are loaded, and what must never be logged:

## Backend Conventions

- _Rule_ → `path/to/example.ts:LINE`

## Frontend Conventions

- _Rule_ → `path/to/example.tsx:LINE`

## Data / Migration Conventions

- _Rule_ → `path/to/migration.sql`

## Testing Conventions

- _Rule_ → `path/to/example.test.ts:LINE`

## Naming

| Item | Convention | Example |
|------|-----------|---------|
| Files | | |
| Components | | |
| Migrations | | |
| Branches | | |
| Commits | | |

## Verification Commands

Copied from `package.json` / `Makefile` / CI config — these are what the coder runs as its build check.

| Check | Command |
|-------|---------|
| Typecheck | |
| Lint | |
| Unit tests | |
| E2E tests | |
| Build | |

## Open Items

- `[ASSUMED]` _inference the developer should confirm_
- `[NONE FOUND]` _pattern that could not be located_
- `[INCONSISTENT]` _competing patterns; developer picks the rule_
