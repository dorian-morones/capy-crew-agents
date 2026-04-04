# Clerk Auth Patterns

Reference for Clerk authentication in the actify stack — JWT validation, middleware, webhooks, and key management.

---

## Key Management

Clerk has separate test and live environments. They are completely isolated — tokens from one environment are invalid in the other.

| Environment | Frontend key | Backend key |
|-------------|-------------|-------------|
| Test (dev) | `pk_test_*` | `sk_test_*` |
| Live (prod) | `pk_live_*` | `sk_live_*` |

**Rule:** Frontend and backend must always use the same tier. A `pk_live_*` frontend paired with a `sk_test_*` backend produces `Invalid or expired token` on every request.

---

## Next.js Middleware (actify-web)

Clerk middleware lives in `src/proxy.ts` (not `middleware.ts` — using both at once causes a build error).

```typescript
// src/proxy.ts
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server";

const isPublicRoute = createRouteMatcher([
  "/",
  "/login(.*)",
  "/signup(.*)",
  "/onboarding(.*)",   // ← public: new users haven't completed onboarding yet
  "/auth(.*)",
  "/api/webhooks(.*)",
  "/ingest(.*)",       // ← public: PostHog proxy must be reachable without auth
]);

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) {
    await auth.protect();
  }
});
```

**Critical:** `/onboarding` must be a public route. New users have a valid Clerk session but no account membership yet. If `/onboarding` is protected, they get redirected to `/login` in a loop.

---

## API Auth Middleware (actify-api)

Two middleware options:

| Middleware | Use case |
|-----------|---------|
| `auth` | Standard protected route — verifies JWT and account membership |
| `authLite` | Onboarding only — verifies JWT, but user may not have account yet |

```typescript
// Standard protected route
app.group("/feedback", (app) =>
  app.use(auth).get("/", async ({ user, supabaseClient }) => {
    // user.account_id is available and verified
  })
);

// Onboarding route
app.group("/onboarding", (app) =>
  app.use(authLite).post("/complete", async ({ user }) => {
    // user.clerk_id is available, but user may not have account_id yet
  })
);
```

---

## Supabase JWT Integration

Supabase uses the Clerk JWT to enforce RLS. The JWT must include `account_id` in its claims.

In Clerk Dashboard → JWT Templates → Supabase:
```json
{
  "account_id": "{{user.public_metadata.account_id}}"
}
```

In RLS policies, access this via:
```sql
(auth.jwt() ->> 'account_id')::uuid
```

The per-request Supabase client in `auth` middleware passes the user's JWT:
```typescript
const supabaseClient = createClient(url, anonKey, {
  global: {
    headers: { Authorization: `Bearer ${clerkToken}` },
  },
});
```

---

## Webhook Verification

Clerk sends webhooks via Svix. Always verify the signature before processing.

```typescript
import { Webhook } from "svix";

const webhook = new Webhook(process.env.CLERK_WEBHOOK_SECRET!);

try {
  const event = webhook.verify(rawBody, {
    "svix-id": headers["svix-id"],
    "svix-timestamp": headers["svix-timestamp"],
    "svix-signature": headers["svix-signature"],
  }) as ClerkWebhookEvent;

  // Process event.type: "user.created" | "user.updated" | "user.deleted"
} catch {
  throw new UnauthorizedError("Invalid webhook signature");
}
```

Webhook routes must be registered **before** `csrfGuard` in `src/index.ts` — external services don't send an `Origin` header.

---

## E2E Testing with Clerk

Never use production Clerk credentials in tests. Use the Clerk test instance (`pk_test_*` / `sk_test_*`) with `@clerk/testing`.

```typescript
import { clerk, setupClerkTestingToken } from "@clerk/testing/playwright";

test.beforeEach(async ({ page }) => {
  await setupClerkTestingToken({ page });
  await clerk.signIn({
    page,
    signInParams: {
      strategy: "password",
      identifier: process.env.E2E_USER_EMAIL!,
      password: process.env.E2E_USER_PASSWORD!,
    },
  });
});
```

Required env vars for E2E:
```
E2E_USER_EMAIL=test@example.com
E2E_USER_PASSWORD=testpassword123
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
```

---

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Test and live keys mixed | `Invalid or expired token` on all API calls | Match key tiers across frontend and backend |
| `/onboarding` as protected route | New users redirect to login after sign-up | Add `/onboarding(.*)` to `isPublicRoute` |
| Clerk token not in Supabase client | RLS rejects all queries | Pass `Authorization: Bearer ${token}` in Supabase client headers |
| `middleware.ts` and `proxy.ts` both exist | Vercel build error: duplicate middleware | Delete `middleware.ts`, use only `proxy.ts` |
| Webhook route after csrfGuard | Webhook calls silently blocked | Move webhook routes before `.use(csrfGuard)` in index.ts |
