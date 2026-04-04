# Security Checklist

Pre-ship security review for API routes, frontend code, and infrastructure in the actify stack.

---

## API Route Security

- [ ] Every POST/PUT/PATCH has an Elysia body schema (`t.Object(...)`)
- [ ] Every route with params has a params schema (`t.Object({ id: t.String({ format: "uuid" }) })`)
- [ ] `account_id` comes from `user.account_id` (JWT), never from `body.account_id`
- [ ] No `supabaseAdmin` used in routes that have `auth` middleware
- [ ] All database errors caught — never return raw Supabase errors to client
- [ ] Route returns minimal data — no extra fields that leak schema or PII

---

## Authentication

- [ ] Protected routes use `auth` middleware (not just any middleware)
- [ ] `authLite` only used for onboarding routes
- [ ] Webhook routes validate their own signature — never use `auth` middleware for webhooks
- [ ] New public routes explicitly added to `isPublicRoute` in `proxy.ts`
- [ ] Clerk test and live keys are never mixed between environments

---

## Webhook Security

- [ ] Signature verified before payload is processed (not after)
- [ ] Raw body (not parsed JSON) used for signature verification
- [ ] Webhook route registered before `csrfGuard` in `src/index.ts`
- [ ] `CLERK_WEBHOOK_SECRET` and `LINEAR_WEBHOOK_SECRET` in env vars, not source code

---

## Secrets and Environment

- [ ] No API keys, tokens, or passwords in source code
- [ ] No `.env` files with real values committed to git
- [ ] `.env.example` exists with placeholder values only
- [ ] All required secrets are in platform env vars (Render, Vercel)
- [ ] `NEXT_PUBLIC_*` variables contain no secrets (they're visible in the browser bundle)

---

## CORS

- [ ] `CORS_ORIGIN` explicitly lists allowed origins — no wildcards
- [ ] Production origin (`https://app.useactify.com`) is in `CORS_ORIGIN`
- [ ] `origin: true` or `origin: *` never used in production

---

## Frontend Security (Next.js)

- [ ] No `dangerouslySetInnerHTML` with user-provided content
- [ ] No `eval()` with user-provided content
- [ ] External URLs validated before redirect (prevent open redirect)
- [ ] PostHog proxy in place — `NEXT_PUBLIC_POSTHOG_HOST` uses `/ingest` path
- [ ] CSP headers in `next.config.ts` — new third-party domains added to appropriate directive

---

## Data Access (Supabase RLS)

- [ ] RLS enabled on every table: `ENABLE ROW LEVEL SECURITY`
- [ ] SELECT policy uses `USING ((auth.jwt() ->> 'account_id')::uuid = account_id)`
- [ ] INSERT policy uses `WITH CHECK` — not just `USING`
- [ ] Service role key only used in background jobs and webhook handlers
- [ ] Tested RLS policies in Supabase Studio with a real JWT

---

## Logging

- [ ] No JWT tokens in log output
- [ ] No passwords or secrets in log output
- [ ] No full user PII (email, name) in logs unless explicitly needed for debugging
- [ ] Error logs include method, path, and error code — not raw error objects

---

## OWASP Top 10 Quick Check

| Risk | Check |
|------|-------|
| Injection | Supabase client uses parameterized queries — no raw SQL with user input |
| Broken Auth | Clerk JWT verified on every protected request |
| Sensitive Data Exposure | No secrets in logs, responses, or source code |
| Security Misconfiguration | CORS explicit, no wildcard, CSP headers set |
| XSS | No `dangerouslySetInnerHTML` with user content |
| Broken Access Control | All queries scoped to `user.account_id` from JWT |
| CSRF | `csrfGuard` middleware in place on all non-webhook routes |
| Insecure Components | Dependencies audited with `pnpm audit` |
