---
name: api-route-design
description: Use when creating or modifying Elysia API routes, middleware, or authentication patterns in actify-api.
---

## Overview

API routes are the contract between the frontend and backend. A poorly designed route is hard to test, easy to break, and difficult to secure. This skill encodes patterns specific to the Elysia/Bun stack used in actify-api — including auth middleware, CSRF handling, error responses, and the conventions that keep routes consistent across the codebase.

## When to Use

**Use this skill when:**
- Adding a new route or route group
- Modifying an existing route's request/response shape
- Adding or changing auth requirements on a route
- Handling a new webhook
- Adding new middleware

**Skip this skill when:**
- Making changes inside a route's business logic without touching the route signature
- Modifying non-route files (services, jobs, config)

## Core Process

1. **Define the contract** — Method, path, request shape (body/params/query), response shape, auth requirement, error cases.

2. **Choose the right auth middleware** — `auth` for routes requiring account membership, `authLite` for routes that only need a valid Clerk user (e.g. onboarding).

3. **Create the route file** — Follow the existing pattern: export a named function that takes an Elysia app and returns it with `.group()`.

4. **Register the route** — Add `.use(myRoutes)` in `src/index.ts`. Webhook routes go before `csrfGuard`.

5. **Validate inputs** — Use Elysia's schema validation (`t.Object`, `t.String`, etc.) on body, params, and query. Never trust unvalidated input.

6. **Return consistent responses** — Successful responses return data directly. Errors throw `ApiError` from `src/utils/errors.ts`.

7. **Test the route** — Hit it with curl or the Swagger UI at `/swagger`. Verify auth, success, and error cases.

## Specific Techniques

### Route File Template

```typescript
// src/routes/my-resource.routes.ts
import { Elysia, t } from "elysia";
import { auth } from "../middleware/auth";
import { ApiError } from "../utils/errors";

export const myResourceRoutes = (app: Elysia) =>
  app.group("/my-resource", (app) =>
    app
      .use(auth)
      .get(
        "/",
        async ({ user, supabaseClient }) => {
          const { data, error } = await supabaseClient
            .from("my_table")
            .select("*")
            .eq("account_id", user.account_id);

          if (error) throw new ApiError(500, error.message, "DB_ERROR");
          return { data };
        },
        {
          detail: { tags: ["MyResource"], summary: "List resources" },
        }
      )
      .post(
        "/",
        async ({ user, supabaseClient, body }) => {
          const { data, error } = await supabaseClient
            .from("my_table")
            .insert({ ...body, account_id: user.account_id })
            .select()
            .single();

          if (error) throw new ApiError(500, error.message, "DB_ERROR");
          return { data };
        },
        {
          body: t.Object({
            name: t.String(),
            status: t.Optional(t.String()),
          }),
          detail: { tags: ["MyResource"], summary: "Create resource" },
        }
      )
  );
```

### Auth Middleware Choice

| Situation | Middleware |
|-----------|-----------|
| Standard protected route (needs account_id) | `auth` |
| Onboarding (user exists but no membership yet) | `authLite` |
| Webhook (external caller, own signature validation) | None — validate webhook secret manually |
| Public route | None |

### Error Handling

Always use `ApiError` — never throw raw errors or return `{ error: string }` manually:

```typescript
import { ApiError, NotFoundError, UnauthorizedError } from "../utils/errors";

// Specific errors
throw new NotFoundError("Feedback item not found");
throw new UnauthorizedError("Not a member of this account");

// Generic with code
throw new ApiError(400, "Invalid source type", "INVALID_SOURCE_TYPE");
```

### Webhook Route Pattern

Webhook routes go **before** `csrfGuard` in `src/index.ts` — external callers don't send an `Origin` header:

```typescript
// In index.ts — order matters
.use(linearWebhookRoutes)   // ← before csrfGuard
.use(clerkWebhookRoutes)    // ← before csrfGuard
.use(csrfGuard)             // ← all routes below this are CSRF-protected
.use(feedbackRoutes)
```

Webhook route validates its own signature:
```typescript
const signature = request.headers.get("svix-signature");
if (!verifyWebhookSignature(body, signature)) {
  throw new UnauthorizedError("Invalid webhook signature");
}
```

### Route Registration Checklist

```typescript
// src/index.ts
import { myResourceRoutes } from "./routes/my-resource.routes";

const app = new Elysia()
  // ...existing routes...
  .use(myResourceRoutes)  // ← add here, after csrfGuard
```

Also add the tag to the Swagger config in `index.ts`:
```typescript
{ name: "MyResource", description: "Description of the resource" }
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll add input validation later" | Unvalidated input is an open door. Elysia makes validation trivial — do it at route definition. |
| "I'll use `supabaseAdmin` for simplicity" | `supabaseAdmin` bypasses RLS. Using it in user-facing routes exposes all accounts' data. Use the per-request client. |
| "The frontend validates this already" | Frontend validation is UX. Backend validation is security. They serve different purposes. |
| "I'll add Swagger tags later" | Tags take 5 seconds. Without them, the Swagger UI is unusable for testing. |
| "I'll put the webhook route after csrfGuard" | Webhooks from external services have no Origin header. CSRF guard will block them silently. |

## Red Flags

- Route using `supabaseAdmin` for a user-facing operation
- No input schema on a POST/PUT/PATCH route
- Returning raw Supabase error objects to the client (leaks schema details)
- Webhook route registered after `csrfGuard`
- Route with no Swagger `detail` tag
- Throwing raw `Error` instead of `ApiError`
- Not checking `user.account_id` before querying — cross-account data leak risk

## Verification

- [ ] Route contract documented (method, path, request, response, errors)
- [ ] Correct auth middleware chosen and applied
- [ ] Input validation schema defined on body/params/query
- [ ] Only `ApiError` (or subclasses) thrown for error cases
- [ ] Supabase queries scoped to `user.account_id` (not using supabaseAdmin for user data)
- [ ] Webhook routes registered before `csrfGuard` in index.ts
- [ ] Swagger tag added to route detail
- [ ] Route registered in `src/index.ts`
- [ ] Tested manually via Swagger UI or curl — success + auth failure + validation failure
