---
name: supabase-data-modeling
description: Use when adding new tables, columns, indexes, RLS policies, or writing migrations in Supabase.
---

## Overview

Database changes are the highest-risk changes in the stack. Unlike application code, bad migrations can corrupt data, silently break RLS policies, or cause downtime during deployment. Supabase adds additional surface area through Row Level Security, realtime subscriptions, and the generated TypeScript types that downstream code depends on.

This skill encodes a safe, repeatable process for making database changes — from design through migration through type regeneration.

## When to Use

**Use this skill when:**
- Adding a new table
- Adding or removing columns from an existing table
- Writing or modifying RLS policies
- Adding indexes
- Changing foreign key relationships
- Any SQL that runs against production data

**Skip this skill when:**
- The change is purely in application code with no schema modification
- Writing a one-off query for analysis (not a migration)

## Core Process

1. **Design the schema first** — Write the table definition in SQL before touching the Supabase dashboard or migration files. Define columns, types, constraints, and foreign keys on paper.

2. **Identify RLS requirements** — Every table that stores user or account data needs RLS. Determine: who can SELECT, INSERT, UPDATE, DELETE? What column links the row to an account or user?

3. **Identify index requirements** — Every foreign key column should be indexed. Every column used in a WHERE clause in a hot query path should be indexed.

4. **Write the migration file** — Name it sequentially: `NNN_description.sql`. Never modify an existing migration that has been applied.

5. **Test locally** — Apply the migration to your local Supabase instance. Verify the schema looks correct in the dashboard.

6. **Verify RLS policies** — Test as an authenticated user that policies allow correct access and deny incorrect access.

7. **Regenerate TypeScript types** — Run `supabase gen types typescript` after any schema change. Update imports in application code.

8. **Apply to production** — Run migration via Supabase dashboard or CLI against production.

## Specific Techniques

### Table Definition Template

```sql
-- NNN_table_name.sql

create table if not exists public.table_name (
  id          uuid primary key default gen_random_uuid(),
  account_id  uuid not null references public.accounts(id) on delete cascade,
  -- domain columns
  name        text not null,
  status      text not null check (status in ('active', 'inactive')),
  metadata    jsonb default '{}',
  created_at  timestamptz not null default now(),
  updated_at  timestamptz not null default now()
);

-- Indexes
create index on public.table_name (account_id);
create index on public.table_name (status) where status = 'active';

-- Updated at trigger
create trigger set_updated_at
  before update on public.table_name
  for each row execute function public.set_updated_at();

-- RLS
alter table public.table_name enable row level security;

create policy "accounts can read own rows"
  on public.table_name for select
  using (account_id = (auth.jwt() ->> 'account_id')::uuid);

create policy "accounts can insert own rows"
  on public.table_name for insert
  with check (account_id = (auth.jwt() ->> 'account_id')::uuid);

create policy "accounts can update own rows"
  on public.table_name for update
  using (account_id = (auth.jwt() ->> 'account_id')::uuid);

create policy "accounts can delete own rows"
  on public.table_name for delete
  using (account_id = (auth.jwt() ->> 'account_id')::uuid);
```

### RLS Pattern for This Stack

The JWT claim used is `account_id` — set via Clerk's session claims and passed through Supabase's `auth.jwt()`. Always use:

```sql
account_id = (auth.jwt() ->> 'account_id')::uuid
```

For tables that belong to a user (not an account):
```sql
user_id = (auth.jwt() ->> 'sub')::text
```

### Service Role Bypass

The `supabaseAdmin` client (initialized with `SUPABASE_SERVICE_ROLE_KEY`) bypasses RLS. Use it only in:
- Webhook handlers (no user context)
- Background jobs
- Admin operations

Never expose the service role key to the frontend.

### Index Decision Rules

| Situation | Add index? |
|-----------|-----------|
| Foreign key column | Always |
| Column in WHERE clause in hot path | Yes |
| Column used only in admin queries | No |
| UUID primary key | Automatic (PK constraint) |
| `status` column with few distinct values | Partial index if filtering on one value |
| `created_at` for time-range queries | Yes, if queried frequently |

### Migration Numbering

```
001_initial_schema.sql
002_add_sources.sql
003_add_feedback.sql
...
031_feedback_imports.sql   ← next available number
```

Never edit an already-applied migration. If you need to fix a mistake, write a new migration.

### TypeScript Type Regeneration

After any schema change:
```bash
supabase gen types typescript --project-id <project-id> > src/types/supabase.ts
```

Then update any affected type imports in the codebase.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll add RLS later" | RLS added later requires auditing every existing query. Data may already be leaking. Do it in the same migration. |
| "I don't need an index on that column" | Missing indexes on FK columns cause full table scans in production. Add them at creation time. |
| "I'll just modify the existing migration" | Existing migrations that have been applied cannot be safely modified. Write a new one. |
| "I'll update the types later" | Stale types cause silent type errors that only surface at runtime. Regenerate immediately. |
| "The dashboard is faster than writing SQL" | Dashboard changes aren't version controlled. Always write migration files. |

## Red Flags

- A new table with no RLS policies
- Foreign key column with no index
- Migration that modifies an already-applied file
- TypeScript types not regenerated after schema change
- `account_id` column missing from a table that stores user data
- Service role key used in a frontend environment variable
- `metadata jsonb` used as a catch-all instead of proper columns for queryable data

## Verification

- [ ] Table definition written in SQL with correct types and constraints
- [ ] RLS enabled on the table
- [ ] SELECT, INSERT, UPDATE, DELETE policies written and tested
- [ ] Index added for every FK column
- [ ] Index added for every column used in a WHERE clause on a hot path
- [ ] Migration file named sequentially (`NNN_description.sql`)
- [ ] Migration applied locally and tested
- [ ] TypeScript types regenerated
- [ ] No existing migration files modified
