# Deployment Checklist

Pre-deploy and post-deploy checklist for actify-web (Vercel) and actify-api (Render).

---

## actify-web → Vercel

### Pre-Deploy

**Environment variables (Vercel dashboard → Settings → Environment Variables):**

| Variable | Production value | Notes |
|----------|-----------------|-------|
| `NEXT_PUBLIC_API_URL` | `https://api.useactify.com` | Must start with `https://` |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | `pk_live_*` | Live key, not test |
| `CLERK_SECRET_KEY` | `sk_live_*` | Live key, must match publishable key tier |
| `NEXT_PUBLIC_POSTHOG_KEY` | `phc_*` | From PostHog dashboard |
| `NEXT_PUBLIC_POSTHOG_HOST` | `https://app.useactify.com/ingest` | Proxy URL, not direct PostHog |

**Code checks:**
- [ ] `pnpm build` passes locally
- [ ] No TypeScript errors (`pnpm tsc --noEmit`)
- [ ] `/onboarding(.*)` in `isPublicRoute` in `src/proxy.ts`
- [ ] `/ingest(.*)` in `isPublicRoute` in `src/proxy.ts`
- [ ] PostHog rewrites in `next.config.ts`
- [ ] No `middleware.ts` file (conflicts with `proxy.ts`)

**Domain:**
- [ ] `app.useactify.com` → Production deployment in Vercel
- [ ] SSL certificate active

### Post-Deploy

- [ ] `https://app.useactify.com` loads without error
- [ ] Sign in works (Clerk redirects properly)
- [ ] Dashboard loads data from API
- [ ] No 401 or 500 errors in Vercel Function logs
- [ ] No console errors in browser

---

## actify-api → Render

### Pre-Deploy

**Environment variables (Render dashboard → Environment):**

| Variable | Production value | Notes |
|----------|-----------------|-------|
| `CLERK_SECRET_KEY` | `sk_live_*` | Must match frontend's `pk_live_*` |
| `SUPABASE_URL` | `https://[project].supabase.co` | From Supabase dashboard |
| `SUPABASE_SERVICE_ROLE_KEY` | `eyJ*` | Service role key, not anon key |
| `DATABASE_URL` | `postgres://...` | From Supabase → Settings → Database |
| `CORS_ORIGIN` | `http://localhost:3000,https://app.useactify.com` | Comma-separated, no trailing slash |
| `POSTHOG_API_KEY` | `phc_*` | For OpenTelemetry log shipping |
| `PORT` | (set by Render) | Don't set manually |

**Code checks:**
- [ ] `bun run src/index.ts` starts without errors locally
- [ ] Dockerfile builds successfully
- [ ] Webhook routes registered before `csrfGuard` in `src/index.ts`
- [ ] No hardcoded secrets in source code

**Service settings:**
- [ ] Runtime: Docker
- [ ] Branch: `main`
- [ ] Plan: Paid tier (required for agent setInterval loops)
- [ ] Health check path: `/`

**Domain:**
- [ ] `api.useactify.com` → Render service custom domain
- [ ] SSL certificate active

### Post-Deploy

- [ ] `GET https://api.useactify.com/` returns 200
- [ ] `GET https://api.useactify.com/swagger` loads API docs
- [ ] Auth flow works: sign in on frontend, API calls succeed
- [ ] No errors in Render logs (check first 5 minutes)
- [ ] PostHog receiving logs (check PostHog → Logs)

---

## Clerk Key Matching Verification

Before any deploy that touches auth:

```
Frontend (Vercel env):     NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
Backend (Render env):      CLERK_SECRET_KEY

Both must be the same tier:
  pk_test_* ↔ sk_test_*   (development/staging)
  pk_live_* ↔ sk_live_*   (production)
```

Mixing tiers causes `Invalid or expired token` on every API call.

---

## Rollback Procedure

**Vercel (frontend):**
1. Vercel dashboard → Deployments
2. Find the last known good deployment
3. Click "Promote to Production"

**Render (API):**
1. Render dashboard → Deploys
2. Find the last known good deploy
3. Click "Rollback to this deploy"

Or via git:
```bash
git revert HEAD  # creates a new revert commit
git push origin main  # triggers auto-deploy
```

---

## Environment-Specific Notes

| Setting | Local | Production |
|---------|-------|------------|
| Clerk keys | `pk_test_*` / `sk_test_*` | `pk_live_*` / `sk_live_*` |
| API URL | `http://localhost:3001` | `https://api.useactify.com` |
| PostHog host | Direct or skip | `https://app.useactify.com/ingest` |
| CORS origin | `http://localhost:3000` | `https://app.useactify.com` |
| Render plan | N/A | Paid (for agents) |
