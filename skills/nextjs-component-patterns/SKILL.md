---
name: nextjs-component-patterns
description: Use when building Next.js pages, layouts, components, or data fetching patterns in actify-web.
---

## Overview

Next.js App Router has strong opinions about where code runs (server vs. client), how data is fetched, and how state is managed. Ignoring these opinions leads to waterfall fetches, unnecessary client bundles, and hydration errors. This skill encodes the patterns specific to actify-web — App Router, Clerk auth, Zustand state, Tailwind v4, and the API client.

## When to Use

**Use this skill when:**
- Building a new page or layout under `src/app/(app)/` or `src/app/(auth)/`
- Creating a new reusable component
- Adding data fetching to a page
- Deciding whether a component should be a Server Component or Client Component

**Skip this skill when:**
- Editing purely visual/style changes with no logic changes
- Modifying existing components with no structural changes

## Core Process

1. **Decide server vs. client** — Default to Server Component. Add `"use client"` only when you need browser APIs, event handlers, or React hooks.

2. **Place the file** — Pages go in `src/app/(app)/[route]/page.tsx`. Shared components go in `src/components/`. Domain hooks go in `src/hooks/`.

3. **Fetch data at the right level** — Fetch in Server Components where possible. In Client Components, use domain hooks (`useFeedback`, `useRoadmap`, etc.) from `src/hooks/`.

4. **Wire auth** — Protected pages are covered by `src/proxy.ts`. Inside components, use `useUser()` or `useAuth()` from `@clerk/nextjs` for client-side access.

5. **Follow Tailwind v4 conventions** — Utility classes only. No CSS modules. No inline styles.

6. **Add analytics** — Track meaningful user actions with `useAnalytics()`. Event names follow `viewname_feature__action` format.

## Specific Techniques

### Server vs. Client Decision Tree

```
Does the component need:
  - onClick, onChange, or other event handlers?     → "use client"
  - useState or useEffect?                          → "use client"
  - Browser APIs (window, document)?                → "use client"
  - useUser(), useAnalytics(), or other hooks?      → "use client"
  - Real-time Supabase subscription?                → "use client"

Otherwise → Server Component (no directive needed)
```

### Page Template (Client Component — most app pages)

```tsx
"use client";

import { useEffect, useState } from "react";
import { useFeedback } from "@/hooks/useFeedback";
import { useAnalytics } from "@/hooks/useAnalytics";

export default function FeedbackPage() {
  const { items, isLoading, error, fetchItems } = useFeedback();
  const { track } = useAnalytics();

  useEffect(() => {
    fetchItems();
  }, []);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="p-6">
      {items.map((item) => (
        <div key={item.id}>{item.content}</div>
      ))}
    </div>
  );
}
```

### Domain Hook Pattern

All data fetching belongs in `src/hooks/`. One hook per domain:

```typescript
// src/hooks/useMyResource.ts
import { useState, useCallback } from "react";
import { api } from "@/api/client";
import { APIError } from "@/api/client";

export function useMyResource() {
  const [items, setItems] = useState<MyResource[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchItems = useCallback(async () => {
    setIsLoading(true);
    setError(null);
    try {
      const res = await api.get<MyResource[]>("/my-resource");
      setItems(res.data);
    } catch (err) {
      setError(err instanceof APIError ? err.message : "Failed to load");
    } finally {
      setIsLoading(false);
    }
  }, []);

  return { items, isLoading, error, fetchItems };
}
```

### Analytics Event Naming

Format: `viewname_feature__action`

```typescript
const { track } = useAnalytics();

track("feedback_filters__cleared");
track("roadmap_item__created", { priority: item.priority });
track("onboarding_step_completed", { step: 2, step_name: "Role" });
```

### Toast Notifications

Use `sonner` for all toast notifications:

```typescript
import { toast } from "sonner";

toast.success("Feedback imported successfully");
toast.error("Failed to connect integration");
toast.loading("Syncing data...");
```

### Route Protection

Routes under `src/app/(app)/` are automatically protected by the Clerk middleware in `src/proxy.ts`. For `/onboarding` and auth routes, they are public — no additional protection needed.

If you add a new public route, add it to `isPublicRoute` in `src/proxy.ts`.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll just add `use client` to everything" | Client components increase bundle size and lose streaming/SSR benefits. Start server, add client only when needed. |
| "I'll fetch data directly in the component" | Data fetching in components is hard to reuse and test. Put it in a hook in `src/hooks/`. |
| "I'll use inline styles for this one thing" | One inline style becomes ten. Use Tailwind utility classes — they're already in the bundle. |
| "I'll add the analytics event later" | Tracking added after the fact is often never added. Do it in the same PR as the feature. |
| "I'll put this component in the page file" | Components in page files aren't reusable. If it's >50 lines, extract to `src/components/`. |

## Red Flags

- `"use client"` on a component with no hooks, events, or browser APIs
- `fetch()` called directly inside a component (use the API client and a hook)
- Inline styles on any element
- Analytics event with a non-standard name format
- New public route not added to `isPublicRoute` in `src/proxy.ts`
- Component file exceeding 300 lines without extracting sub-components
- `useEffect` with missing or incorrect dependencies

## Verification

- [ ] Server vs. client decision made explicitly — not `"use client"` by default
- [ ] Data fetching in a hook under `src/hooks/`, not inline in the component
- [ ] All styles via Tailwind utility classes
- [ ] Analytics events tracked for key user actions with correct name format
- [ ] Toast notifications using `sonner`
- [ ] New public routes added to `isPublicRoute` in `src/proxy.ts`
- [ ] Component extracted to `src/components/` if reused or >50 lines
