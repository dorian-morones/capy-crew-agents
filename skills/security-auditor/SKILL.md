---
name: security-auditor
description: Use when auditing API routes, auth flows, or data access patterns for security vulnerabilities in a multi-tenant SaaS application.
---

## Overview

The security-auditor skill audits code against the threat model that actually applies to a multi-tenant SaaS: auth bypass, cross-account data access, injection, secrets exposure, and webhook forgery. It does not flag theoretical vulnerabilities that cannot be exploited in this architecture.

Every finding includes a severity level, the exact file and line, what the vulnerability is, what an attacker can do if it is exploited, and the specific change needed to fix it.

## When to Use

**Use this skill when:**
- Touching authentication flows, JWT handling, or middleware
- Adding or modifying API routes that read or write user data
- Adding webhook routes
- Reviewing data access patterns in Supabase queries
- Any code that handles external input (request bodies, headers, query params)

**Skip this skill when:**
- Reviewing purely frontend rendering code with no data access
- The code is already under active security review by a dedicated security team

## Core Process

1. **Identify the threat surface** — What does this code do? Which of the five threat categories does it touch? List them before starting the audit.

2. **Check account scoping** — Every Supabase query that returns or modifies user data must be scoped to `user.account_id` from the verified JWT. Never from `body.account_id`.

3. **Check input validation** — Every POST/PUT/PATCH route must have a body schema that validates and strips input before the handler runs.

4. **Check auth middleware** — Is the right middleware applied to each route? Protected routes use `auth`, onboarding uses `authLite`, webhooks verify their own signature, public routes have none.

5. **Check supabaseAdmin usage** — `supabaseAdmin` bypasses RLS. Audit every usage. It should only appear in webhook handlers and background jobs — never in routes with `auth` middleware.

6. **Check webhook verification** — Every webhook route must verify the incoming signature before processing the payload. An unverified webhook accepts arbitrary payloads from anyone.

7. **Check secret handling** — Secrets must be in environment variables, not in source code. Secrets must not appear in logs or response bodies.

8. **Write findings** — Use the findings format in Specific Techniques. One finding per issue. Severity, location, vulnerability, impact, fix.

## Specific Techniques

### Threat Model

| # | Threat | How it happens |
|---|--------|----------------|
| 1 | Cross-account data access | Authenticated user A reads or modifies user B's data |
| 2 | Auth bypass | Unauthenticated caller accesses protected data |
| 3 | Injection | Unsanitized input reaches the database, shell, or rendering layer |
| 4 | Secrets exposure | Keys, tokens, or PII appear in logs, responses, or source code |
| 5 | Webhook forgery | External attacker sends fake webhook payloads |

### Account Scoping Check

```typescript
// Safe — account_id from verified JWT
.eq("account_id", user.account_id)

// Dangerous — account_id from request body (spoofable)
.eq("account_id", body.account_id)
```

Any query that uses `body.account_id`, `params.account_id`, or `query.account_id` to scope data access is a blocker.

### Input Validation Check

Every mutating route needs a body schema. Elysia validates and strips unknown fields before the handler runs:

```typescript
// Required on every POST/PUT/PATCH
body: t.Object({
  field: t.String({ minLength: 1 }),
  count: t.Number({ minimum: 0 }),
})
```

A route with no body schema accepts any payload. The handler must not be trusted to validate inputs manually.

### Auth Middleware Check

| Route type | Required middleware | Red flag |
|-----------|-------------------|----------|
| User data access | `auth` | Using `authLite` or none |
| Onboarding | `authLite` | Using `auth` or none |
| Webhook | Signature verification | Using `auth` or `authLite` |
| Public | None | Accidentally protected |

### supabaseAdmin Audit

Search for every usage of `supabaseAdmin`:

```typescript
// Allowed: webhook handler (system actor, not user actor)
app.post("/webhooks/clerk", async ({ body }) => {
  webhook.verify(rawBody, headers);
  await supabaseAdmin.from("accounts").insert(...);
});

// Not allowed: route with auth middleware (RLS bypass)
app.post("/resource", { beforeHandle: [auth] }, async ({ user, body }) => {
  await supabaseAdmin.from("resources").insert(...);  // RLS bypassed
});
```

If `supabaseAdmin` appears in a route with `auth` middleware, that is a blocker — the user's account isolation via RLS is completely bypassed.

### Webhook Verification Check

```typescript
// Clerk webhooks — Svix
const webhook = new Webhook(process.env.CLERK_WEBHOOK_SECRET!);
webhook.verify(rawBody, {
  "svix-id": headers["svix-id"],
  "svix-timestamp": headers["svix-timestamp"],
  "svix-signature": headers["svix-signature"],
});

// Linear webhooks
const sig = request.headers.get("linear-signature");
verifyLinearSignature(rawBody, sig, process.env.LINEAR_WEBHOOK_SECRET!);
```

A webhook handler that does not verify its signature accepts arbitrary payloads from any caller on the internet.

### Findings Report Format

```
[Severity] path/to/file.ts:<line>
Vulnerability: <what the issue is>
Impact: <what an attacker can do if this is exploited>
Fix: <the specific change needed>
```

Example:
```
[High] src/routes/feedback.routes.ts:34
Vulnerability: account_id sourced from body.account_id instead of user.account_id
Impact: any authenticated user can read or modify another account's feedback by providing a different account_id in the request body
Fix: replace body.account_id with user.account_id from the Elysia context
```

### Severity Levels

| Level | Meaning |
|-------|---------|
| Critical | Exploitable without authentication or by any user |
| High | Exploitable by any authenticated user against other accounts |
| Medium | Requires specific conditions to exploit, limited impact |
| Low | Defense-in-depth issue, no direct exploitability |

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "RLS will catch this anyway" | RLS only applies when using the anon or service role with RLS enabled. supabaseAdmin bypasses it entirely. |
| "The account_id in the body is validated by the frontend" | The API must validate independently of the client. Any client can send any payload. |
| "The webhook is internal" | Internal webhooks are still reachable from the internet unless network-isolated. Verify signatures. |
| "The secret is only in the test environment" | Test credentials leak into CI logs, git history, and developer machines. Keep them in env vars. |

## Red Flags

- `body.account_id` used to scope any Supabase query
- `supabaseAdmin` in a route that also has `auth` middleware
- A webhook handler that processes the payload before verifying the signature
- Secrets interpolated directly into source code (even in test files)
- Error responses that include Supabase error messages or stack traces
- A route that returns all records from a table without account scoping

## Verification

- [ ] Every finding includes severity, file path, vulnerability, impact, and fix
- [ ] All Supabase queries that return user data were checked for account scoping
- [ ] All mutating routes were checked for body schema validation
- [ ] All new routes were checked for correct middleware
- [ ] All `supabaseAdmin` usages were audited for context appropriateness
- [ ] All webhook routes were checked for signature verification
- [ ] No secrets appear in source code or logs
