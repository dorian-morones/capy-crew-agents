---
name: security-hardening
description: Use when touching authentication, data access patterns, API exposure, input handling, or any code that processes external data.
---

## Overview

Security issues in this stack cluster around three areas: Clerk JWT validation, Supabase RLS bypass, and unvalidated input from external sources. This skill encodes the specific patterns, boundaries, and checks that prevent the most common vulnerabilities in the actify-api/actify-web stack.

Use alongside the [security-checklist](../../references/security-checklist.md) reference.

## When to Use

**Use this skill when:**
- Adding or modifying an API route
- Adding or modifying RLS policies
- Handling webhook payloads
- Adding a new integration (OAuth, third-party API)
- Storing or processing user-submitted data
- Modifying CORS or CSP configuration

**Always Do (no exceptions):**
- Validate all external input at the API boundary using Elysia's schema validation
- Scope all Supabase queries to `user.account_id` — never return cross-account data
- Verify webhook signatures before processing webhook payloads
- Use `CLERK_SECRET_KEY` (not the publishable key) for server-side JWT verification
- Set security headers via `next.config.ts` for all frontend responses

**Ask Before Doing:**
- Adding a new OAuth integration
- Storing a new category of sensitive data
- Modifying CORS origins
- Adding file upload handling
- Changing Clerk webhook endpoints

**Never Do:**
- Commit secrets, API keys, or `.env` files to git
- Use `supabaseAdmin` in user-facing routes (bypasses RLS)
- Trust client-sent `account_id` values — always use the value from the verified JWT
- Log sensitive data (tokens, passwords, PII)
- Use `eval()` or `innerHTML` with untrusted content

## Core Process

1. **Identify the trust boundary** — What is the source of this data? (User input, webhook, third-party API, internal service)

2. **Validate at the boundary** — Every external input gets validated before use. No exceptions.

3. **Check account scoping** — Any query against user data must be scoped to `user.account_id` from the verified JWT.

4. **Check secrets handling** — No new secrets in code. All secrets in environment variables.

5. **Check webhook verification** — Any new webhook handler must verify the signature.

6. **Review CORS/CSP impact** — Does this change require updating `CORS_ORIGIN` or CSP headers?

## Specific Techniques

### Input Validation with Elysia Schema

All POST/PUT/PATCH routes must define a body schema:

```typescript
.post("/feedback", async ({ body, user }) => {
  // body is typed and validated — unknown fields stripped
}, {
  body: t.Object({
    content: t.String({ minLength: 1, maxLength: 10000 }),
    source_id: t.String({ format: "uuid" }),
    sentiment: t.Optional(t.Union([
      t.Literal("positive"),
      t.Literal("negative"),
      t.Literal("neutral"),
    ])),
  }),
})
```

### Account Scoping

Always use the JWT-verified `user.account_id`, never a client-provided value:

```typescript
// ✅ Correct — account_id from verified JWT
const { data } = await supabaseClient
  .from("feedback")
  .select("*")
  .eq("account_id", user.account_id);

// ❌ Wrong — account_id from request body (can be spoofed)
const { data } = await supabaseClient
  .from("feedback")
  .select("*")
  .eq("account_id", body.account_id);
```

### Webhook Signature Verification

```typescript
import { Webhook } from "svix";

const webhook = new Webhook(process.env.CLERK_WEBHOOK_SECRET!);
try {
  const event = webhook.verify(rawBody, {
    "svix-id": headers["svix-id"],
    "svix-timestamp": headers["svix-timestamp"],
    "svix-signature": headers["svix-signature"],
  });
} catch {
  throw new UnauthorizedError("Invalid webhook signature");
}
```

### CORS Configuration

Only allowed origins can make state-changing requests. In Render environment variables:

```
CORS_ORIGIN=https://app.useactify.com,http://localhost:3000
```

Never use `origin: true` or wildcard CORS in production.

### CSP Headers

The Content Security Policy in `next.config.ts` controls what the frontend can load and connect to. When adding a new third-party service, add its domain to the appropriate directive:

```typescript
`connect-src 'self' ${API_URL} https://new-service.com`
```

### Environment Variable Security

```bash
# ✅ Never in code
const key = process.env.SOME_SECRET_KEY;

# ❌ Never committed
SOME_SECRET_KEY=actual_value_here
```

Keep `.env` files in `.gitignore`. Provide `.env.example` with placeholder values only.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "RLS on the table is enough" | RLS protects direct DB access. If you use `supabaseAdmin` in a route, RLS doesn't apply. Scope queries explicitly. |
| "This is an internal route, no one will find it" | Security through obscurity is not security. All routes are public if the server is public. |
| "The frontend already validates this" | Frontend validation is for UX. It can be bypassed with curl. Backend validation is the security boundary. |
| "We'll add the webhook signature check later" | An unverified webhook endpoint accepts arbitrary payloads from anyone. Add verification at creation time. |
| "It's just a test key, I'll replace it before shipping" | Test keys committed to git stay in git history. Use environment variables from day one. |

## Red Flags

- `supabaseAdmin` used in a route that has `auth` middleware (use the per-request client instead)
- Body, query, or param values used without Elysia schema validation
- `account_id` taken from request body instead of `user.account_id`
- New webhook route with no signature verification
- Secret value hardcoded in source code
- CORS wildcard (`origin: true`) in any environment
- `console.log` containing token, password, or user PII

## Verification

- [ ] All external input validated with Elysia schema on body/params/query
- [ ] All DB queries scoped to `user.account_id` from JWT — not from request
- [ ] No `supabaseAdmin` used in user-facing routes
- [ ] Webhook signature verified before payload is processed
- [ ] No secrets in source code — all in environment variables
- [ ] CORS origins explicitly listed — no wildcards
- [ ] CSP headers updated if new third-party domains added
- [ ] No sensitive data in logs
