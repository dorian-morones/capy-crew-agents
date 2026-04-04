---
name: vercel-render-deploy
description: Use when deploying actify-web to Vercel or actify-api to Render to run the pre-deploy checklist and avoid production incidents.
---

## Overview

Deployments fail in predictable ways: wrong environment variables, missing secrets, mismatched keys between services, or a build that works locally but breaks in CI. This skill encodes the specific pre-deploy checklist for the actify stack — Next.js frontend on Vercel and Bun/Elysia API on Render — including the common failure modes that have caused production incidents before.

## When to Use

**Use this skill when:**
- Deploying either app for the first time to a new environment
- Adding a new environment variable to either app
- Changing any auth, CORS, or API URL configuration
- After a branch merge that touches infra, config, or env vars
- Investigating a deployment failure

**Skip this skill when:**
- Deploying a purely frontend visual change with no config changes

## Core Process

1. **Check env vars first** — The majority of deployment failures are missing or wrong environment variables. Verify all required vars are set in the target platform before deploying.

2. **Verify service-to-service config** — Does the frontend point to the right API URL? Do the Clerk keys match between services (test vs. live)?

3. **Run the build locally** — `pnpm build` (frontend) or `bun build` (API) before pushing. CI is slower feedback than local.

4. **Deploy** — Push to trigger CI/CD or deploy via dashboard.

5. **Smoke test immediately after deploy** — Hit the critical paths: sign in, load the main page, make an API call.

6. **Check logs** — Vercel Function logs and Render logs after deploy. Look for errors in the first 5 minutes.

## Specific Techniques

### actify-web (Vercel) Pre-Deploy Checklist

```markdown
Environment variables (set in Vercel dashboard per environment):

Production:
- [ ] NEXT_PUBLIC_API_URL=https://api.useactify.com        ← must have https://
- [ ] NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_live_...         ← live key, not test
- [ ] CLERK_SECRET_KEY=sk_live_...                          ← live key, not test
- [ ] NEXT_PUBLIC_POSTHOG_KEY=phc_...
- [ ] NEXT_PUBLIC_POSTHOG_HOST=https://app.useactify.com/ingest  ← proxy, not direct

Build settings:
- [ ] Framework: Next.js (auto-detected)
- [ ] Build command: pnpm build
- [ ] Output directory: .next (default)
- [ ] Install command: pnpm install

Domain:
- [ ] app.useactify.com → Production deployment
- [ ] CNAME or A record pointing to Vercel
```

### actify-api (Render) Pre-Deploy Checklist

```markdown
Environment variables (set in Render dashboard):

- [ ] CLERK_SECRET_KEY=sk_live_...              ← must match frontend's pk_live_ key
- [ ] SUPABASE_URL=https://[project].supabase.co
- [ ] SUPABASE_SERVICE_ROLE_KEY=...
- [ ] DATABASE_URL=postgres://...
- [ ] CORS_ORIGIN=http://localhost:3000,https://app.useactify.com
- [ ] POSTHOG_API_KEY=phc_...
- [ ] PORT=3001 (or whatever Render assigns)

Service settings:
- [ ] Runtime: Docker (Dockerfile at repo root)
- [ ] Branch: main
- [ ] Auto-deploy: on push to main
- [ ] Plan: Paid tier (required for agent setInterval loops)

Domain:
- [ ] api.useactify.com → Render service
- [ ] Custom domain configured in Render dashboard
```

### Clerk Key Matching

The most common production incident: frontend and backend using different Clerk instances.

```
Frontend (Vercel)         Backend (Render)
────────────────────────  ──────────────────────────
pk_test_* → sk_test_*     ← Test environment (local dev)
pk_live_* → sk_live_*     ← Production
```

**Never mix test and live keys.** A frontend with `pk_live_*` talking to a backend with `sk_test_*` produces `Invalid or expired token` errors on every API call.

### CORS Configuration

```
CORS_ORIGIN must include every origin that talks to the API:

Development: http://localhost:3000
Production:  https://app.useactify.com

CORS_ORIGIN=http://localhost:3000,https://app.useactify.com
```

If you add a new frontend domain (staging, preview), add it to `CORS_ORIGIN` in Render before it goes live.

### PostHog Proxy Setup (Next.js)

PostHog requests must go through the Next.js proxy to avoid adblocker blocking:

```typescript
// next.config.ts — already configured
rewrites: async () => [
  { source: "/ingest/static/:path*", destination: "https://us-assets.i.posthog.com/static/:path*" },
  { source: "/ingest/:path*", destination: "https://us.i.posthog.com/:path*" },
]

// .env — must use proxy, not direct PostHog URL
NEXT_PUBLIC_POSTHOG_HOST=https://app.useactify.com/ingest   ✅
NEXT_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com           ❌ blocked by adblockers
```

And `/ingest(.*)` must be in `isPublicRoute` in `src/proxy.ts`.

### Smoke Test After Deploy

Run these checks within 5 minutes of any production deploy:

```markdown
- [ ] GET https://api.useactify.com/ → 200 OK
- [ ] GET https://app.useactify.com/ → renders login page
- [ ] Sign in with a real account → lands on dashboard
- [ ] Main data loads (feedback list, roadmap, etc.)
- [ ] No 401/500 errors in Render logs
- [ ] No hydration errors in browser console
```

### Render Agent Tier Requirement

Agents use `setInterval` for polling loops. Render's free tier spins down after inactivity, killing the interval. **Paid tier is required** for the process agent to function.

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "It worked in staging, it'll work in prod" | Staging and prod often have different env vars. Check each environment independently. |
| "I'll fix the env var after deploy" | The deploy is already broken for real users. Set env vars before deploying. |
| "The build passed so it must be fine" | Build success only confirms the code compiles. It doesn't test runtime env var usage. |
| "I'll add CORS for that domain later" | The frontend is already blocked right now. CORS blocks the whole app, not just one feature. |
| "The free Render tier is fine for now" | The agent's setInterval dies when the dyno spins down. Free tier = broken agent. |

## Red Flags

- `NEXT_PUBLIC_API_URL` missing `https://` prefix
- Frontend `pk_live_*` key paired with backend `sk_test_*` key (or vice versa)
- `CORS_ORIGIN` not including the production frontend domain
- `NEXT_PUBLIC_POSTHOG_HOST` pointing directly to PostHog instead of the proxy
- `/onboarding` or `/ingest` missing from `isPublicRoute` in proxy.ts
- Deploying without running `pnpm build` locally first
- Not checking Render/Vercel logs after deploy

## Verification

**actify-web (Vercel):**
- [ ] All env vars set for the target environment in Vercel dashboard
- [ ] `NEXT_PUBLIC_API_URL` starts with `https://`
- [ ] Clerk keys are the correct tier (test vs. live) and match the backend
- [ ] `NEXT_PUBLIC_POSTHOG_HOST` uses the proxy URL
- [ ] `pnpm build` passes locally before pushing
- [ ] Smoke test passes after deploy

**actify-api (Render):**
- [ ] All env vars set in Render dashboard
- [ ] `CLERK_SECRET_KEY` matches the frontend's publishable key tier
- [ ] `CORS_ORIGIN` includes all frontend origins
- [ ] Service running on paid tier if agents are in use
- [ ] Smoke test (curl to `/`) returns 200 after deploy
- [ ] Render logs checked for errors in first 5 minutes
