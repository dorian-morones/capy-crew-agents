---
name: debugging-and-error-recovery
description: Use when a bug, error, or unexpected behavior needs to be diagnosed and fixed systematically.
---

## Overview

Debugging without a process is a random walk. Each bug investigation is a hypothesis-testing loop: form a theory, find the smallest test that confirms or refutes it, narrow the search space, repeat. This skill encodes a structured approach that gets from symptom to root cause faster than intuition alone — and prevents the most common mistake of fixing the symptom while the root cause remains.

## When to Use

**Use this skill when:**
- A test is failing and the cause isn't immediately obvious
- A production error appears in logs
- A feature works locally but not in production
- Behavior is inconsistent or intermittent
- You've been stuck on a bug for more than 20 minutes

**Skip this skill when:**
- The error message and stack trace point directly to the problem — just fix it

## Core Process

1. **Read the full error** — Don't skim. Read the error message, the stack trace, and any request/response logged. The answer is often already there.

2. **Reproduce it** — Can you trigger the error consistently? If not, find the conditions that cause it. A bug you can't reproduce is a bug you can't fix.

3. **State your hypothesis** — What do you think is causing it? Write it down as one sentence: "I think X is happening because Y."

4. **Find the smallest test** — What is the minimal change or input that confirms or refutes the hypothesis? Add a log, write a test, narrow an input, isolate a function.

5. **Confirm or update** — Did the test confirm the hypothesis? If yes, fix it. If no, update the hypothesis and repeat.

6. **Fix the root cause** — Not the symptom. If a null check hides the error, that's not the fix — find out why the value is null.

7. **Verify the fix** — The test that reproduced the bug must now pass. Write a regression test if the fix is non-obvious.

## Specific Techniques

### Reading Errors

Before googling, extract these from every error:

```
Error message: "Cannot read properties of undefined (reading 'account_id')"
Stack trace line: src/middleware/auth.ts:42
Request context: POST /feedback, Authorization: Bearer [token]

→ user is undefined at auth.ts:42
→ Why? JWT parsing failed, or user not in DB, or query returned null
```

### The Hypothesis Template

```
"I think [specific behavior] is happening because [specific cause].
I'll test this by [specific action] and expect [specific outcome]."

Example:
"I think `user` is null in the auth middleware because the JWT is expired.
I'll test this by decoding the token manually and expect to see an exp in the past."
```

### Isolation Strategy

When the bug is in a complex flow, binary-search for it:

```
Full flow: Browser → Next.js → API → Supabase

Step 1: Does the API return the right data?
  curl -H "Authorization: Bearer $TOKEN" https://api.useactify.com/feedback
  → Yes: bug is in the frontend
  → No: continue to Step 2

Step 2: Does the Supabase query return the right data?
  Run the query in Supabase Studio directly
  → Yes: bug is in the route handler
  → No: bug is in the query or RLS policy
```

### Log-Driven Debugging

Add logs before guessing:

```typescript
// ✅ Targeted log at the suspect point
console.log("auth middleware - user:", JSON.stringify(user, null, 2));
console.log("supabaseClient headers:", supabaseClient.headers);

// ❌ Adding logs everywhere at once
console.log("step 1");
console.log("step 2");
// ... produces noise, not signal
```

### Environment Diff

For "works locally, fails in production" bugs, diff the environments systematically:

```
Local                       Production
─────────────────────────── ─────────────────────────────
CLERK_SECRET_KEY=sk_test_*  CLERK_SECRET_KEY=sk_live_*  ← different Clerk instance
SUPABASE_URL=*.supabase.co  SUPABASE_URL=*.supabase.co  ← same
NODE_ENV=development        NODE_ENV=production          ← different
```

Differences in env vars, auth keys, or URLs explain most "works local, fails prod" bugs.

### Regression Test Pattern

After fixing a bug, add a test that would have caught it:

```typescript
it("returns 401 when JWT is from wrong Clerk instance", async () => {
  const testToken = generateTestClerkToken(); // test key
  const res = await app.handle(
    new Request("/feedback", {
      headers: { Authorization: `Bearer ${testToken}` },
    })
  );
  expect(res.status).toBe(401);
});
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll just try a few things and see what sticks" | Random changes create new bugs and obscure the original. Form a hypothesis first. |
| "It works on my machine" | That sentence ends with "so I'll deploy and see." The next sentence is "it broke in prod." Diff your environments. |
| "The logs don't tell me much" | You're probably not logging enough at the right places. Add targeted logs at the trust boundaries. |
| "I'll just add a null check here" | Null checks hide bugs — they don't fix them. Find out why the value is null. |
| "This is probably a framework bug" | It's almost never a framework bug. It's almost always your usage of the framework. Rule out your code first. |

## Red Flags

- Making multiple changes at once while debugging (now you don't know what fixed it)
- Fixing a symptom without understanding the root cause
- Deploying to production to test a theory
- Adding `try/catch` that swallows errors without logging
- Debugging production without checking logs first
- Spending more than 30 minutes without changing the approach

## Verification

- [ ] Full error message and stack trace read — not skimmed
- [ ] Bug reproduced consistently before fixing
- [ ] Hypothesis written as a testable statement
- [ ] Root cause identified — not just the symptom fixed
- [ ] Fix tested against the original reproduction case
- [ ] Regression test added for non-obvious bugs
- [ ] Any temporary debug logs removed before committing
