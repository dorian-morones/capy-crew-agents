# Supabase Checklist

Reference for Supabase migrations, RLS policies, query patterns, and indexes in the actify stack.

---

## Migrations

- [ ] Migration file named with timestamp: `YYYYMMDDHHMMSS_description.sql`
- [ ] Migration is idempotent where possible (`IF NOT EXISTS`, `IF EXISTS`)
- [ ] New columns have a default value or are nullable (to avoid locking on large tables)
- [ ] Foreign keys have explicit `ON DELETE` behavior defined
- [ ] Enum types defined before tables that use them
- [ ] Run migration in Supabase Studio or via CLI before deploying code that depends on it

```sql
-- Example: safe column addition
ALTER TABLE feedback
  ADD COLUMN IF NOT EXISTS source_type TEXT NOT NULL DEFAULT 'manual';
```

---

## RLS Policies

Every user-facing table must have RLS enabled and policies that scope rows to the account.

```sql
-- Enable RLS
ALTER TABLE feedback ENABLE ROW LEVEL SECURITY;

-- Standard account-scoped policies
CREATE POLICY "Users can read their account's feedback"
  ON feedback FOR SELECT
  USING ((auth.jwt() ->> 'account_id')::uuid = account_id);

CREATE POLICY "Users can insert feedback for their account"
  ON feedback FOR INSERT
  WITH CHECK ((auth.jwt() ->> 'account_id')::uuid = account_id);

CREATE POLICY "Users can update their account's feedback"
  ON feedback FOR UPDATE
  USING ((auth.jwt() ->> 'account_id')::uuid = account_id);

CREATE POLICY "Users can delete their account's feedback"
  ON feedback FOR DELETE
  USING ((auth.jwt() ->> 'account_id')::uuid = account_id);
```

**Checklist:**
- [ ] RLS enabled on every new table: `ALTER TABLE ... ENABLE ROW LEVEL SECURITY`
- [ ] SELECT policy uses `USING` clause
- [ ] INSERT policy uses `WITH CHECK` clause
- [ ] UPDATE policy uses both `USING` and `WITH CHECK` if applicable
- [ ] `auth.jwt() ->> 'account_id'` cast to `uuid` — not compared as text
- [ ] Tested in Supabase Studio with a real JWT to verify policies work
- [ ] Service role key (used in `supabaseAdmin`) bypasses RLS — only use it for system operations

---

## Query Patterns

### Always scope to account_id

```typescript
// ✅ Correct
const { data } = await supabaseClient
  .from("feedback")
  .select("*")
  .eq("account_id", user.account_id);

// ❌ Wrong — bypasses RLS with admin client
const { data } = await supabaseAdmin
  .from("feedback")
  .select("*")
  .eq("account_id", user.account_id);
```

### Per-request client (with user JWT)

```typescript
// auth middleware creates this — use it for user operations
const { supabaseClient } = context;
```

### Admin client (system operations only)

```typescript
// Use only in webhooks and background jobs
import { supabaseAdmin } from "../config/supabase";
```

### Pagination

```typescript
const PAGE_SIZE = 50;

const { data, count } = await supabaseClient
  .from("feedback")
  .select("*", { count: "exact" })
  .eq("account_id", user.account_id)
  .order("created_at", { ascending: false })
  .range(offset, offset + PAGE_SIZE - 1);
```

---

## Indexes

Add indexes for columns that appear in `.eq()`, `.order()`, or `.in()` on large tables:

```sql
-- Account scoping (most queries filter by this)
CREATE INDEX IF NOT EXISTS idx_feedback_account_id ON feedback(account_id);

-- Sorting by created_at
CREATE INDEX IF NOT EXISTS idx_feedback_account_created
  ON feedback(account_id, created_at DESC);

-- Foreign key lookups
CREATE INDEX IF NOT EXISTS idx_feedback_source_id ON feedback(source_id);
```

**Checklist:**
- [ ] `account_id` indexed on every table with account-scoped queries
- [ ] Composite index on `(account_id, created_at)` if sorting by date within account
- [ ] Foreign key columns indexed
- [ ] No index on low-cardinality boolean columns (not worth it)

---

## Schema Conventions

| Convention | Example |
|-----------|---------|
| Table names | `snake_case`, plural (`feedback`, `roadmap_items`) |
| Column names | `snake_case` |
| Primary key | `id UUID DEFAULT gen_random_uuid()` |
| Timestamps | `created_at TIMESTAMPTZ DEFAULT NOW()`, `updated_at TIMESTAMPTZ` |
| Soft delete | `deleted_at TIMESTAMPTZ` (if used) |
| Account reference | `account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE` |

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `supabaseAdmin` in a user-facing route | Use `supabaseClient` from context |
| Comparing `account_id` as text in RLS | Cast: `(auth.jwt() ->> 'account_id')::uuid` |
| Missing `WITH CHECK` on INSERT policy | Add it — `USING` alone doesn't cover INSERT |
| No index on `account_id` | Add `CREATE INDEX ... ON table(account_id)` |
| Migration adds NOT NULL column without default | All existing rows fail — add a default or migrate in two steps |
