# Performance Checklist

Pre-ship performance review for actify-web (Core Web Vitals) and actify-api (response times, query efficiency).

---

## actify-web (Frontend)

### Core Web Vitals Targets

| Metric | Target | Measured by |
|--------|--------|------------|
| LCP (Largest Contentful Paint) | < 2.5s | PageSpeed Insights, Vercel Analytics |
| INP (Interaction to Next Paint) | < 200ms | PageSpeed Insights |
| CLS (Cumulative Layout Shift) | < 0.1 | PageSpeed Insights |

### Bundle Size

- [ ] New dependencies checked with `bundlephobia.com` before adding
- [ ] Large libraries imported from their specific subpath (not root)
  ```typescript
  // ✅ Tree-shakeable
  import { format } from "date-fns/format";
  // ❌ Imports entire library
  import { format } from "date-fns";
  ```
- [ ] `next build` output checked — no unexpected large chunks
- [ ] Images use `next/image` with explicit `width` and `height` to prevent CLS

### Client vs. Server Components

- [ ] Heavy data-only components kept as Server Components (no `"use client"`)
- [ ] `"use client"` not added to components that don't need it
- [ ] Dynamic imports (`next/dynamic`) used for heavy client-only components that aren't above the fold:
  ```typescript
  const HeavyChart = dynamic(() => import("@/components/HeavyChart"), {
    loading: () => <Skeleton />,
  });
  ```

### Data Fetching

- [ ] Lists paginated — not fetching all rows (`PAGE_SIZE = 50` default)
- [ ] Loading states shown immediately — don't block render on data
- [ ] No waterfall fetches (multiple sequential awaits in a Server Component)

---

## actify-api (Backend)

### Response Time Targets

| Endpoint type | Target p95 |
|--------------|-----------|
| Simple read | < 200ms |
| Write with validation | < 300ms |
| AI-assisted operation | < 5s |

### Database Query Efficiency

- [ ] `account_id` column indexed on every queried table
- [ ] Composite index on `(account_id, created_at)` for paginated lists
- [ ] No N+1 queries — fetch related data with `.select("*, related_table(*)")` not in a loop
- [ ] Pagination in place for all list endpoints — no unbounded `SELECT *`
- [ ] `count: "exact"` only used when count is needed for UI — it adds overhead

```typescript
// ✅ Efficient — paginated, indexed
const { data, count } = await supabaseClient
  .from("feedback")
  .select("id, content, sentiment, created_at", { count: "exact" })
  .eq("account_id", user.account_id)
  .order("created_at", { ascending: false })
  .range(0, 49);

// ❌ Inefficient — unbounded, selects all columns
const { data } = await supabaseClient
  .from("feedback")
  .select("*")
  .eq("account_id", user.account_id);
```

### Logging Overhead

- [ ] Request/response logging doesn't block the response (async or fire-and-forget)
- [ ] PostHog log batching in place (`BatchLogRecordProcessor`) — not sending one HTTP call per log

### Agent Performance

- [ ] Agent polling intervals set appropriately — not polling faster than data changes
- [ ] Agent background work doesn't block the main request loop

---

## Supabase Query Profiling

To check slow queries:
1. Supabase dashboard → SQL Editor
2. `EXPLAIN ANALYZE SELECT ...` on your query
3. Look for `Seq Scan` on large tables — these need indexes
4. Target: all production queries use `Index Scan` not `Seq Scan`

---

## When to Optimize

Optimize when:
- A measured metric is outside the targets above
- A query shows `Seq Scan` on a table with > 10k rows
- A user reports slow load times

Don't optimize speculatively. Premature optimization adds complexity without measured benefit.
