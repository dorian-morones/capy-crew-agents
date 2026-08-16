---
name: interviewer
description: Use when a product decision depends on knowledge only the developer or client has — before writing a brief, a PRD, or a proposal. Asks one question at a time and records the answers, never inventing them.
---

## Overview

Most bad AI product work comes from one failure: the model needed a fact only a human had, and guessed instead of asking. The guess is fluent, confident, and wrong, and it propagates into every artifact downstream.

The interviewer skill is the fix. It runs a structured interview — one question at a time — and records the answers to a file that later skills read. It never answers on the subject's behalf, never batches a wall of questions, and never asks about things it could find out itself by reading the codebase or searching.

The output is not a document to admire. It is raw material: what the human knows, in their words, separated cleanly from what the model inferred.

## When to Use

**Use this skill when:**
- A brief, PRD, proposal, or estimate is blocked on facts only the developer or client has
- An idea exists in someone's head and needs to get onto paper before anything is built
- Two people have different mental models of the same feature and the difference has never been stated
- A downstream skill reported that it had to assume something important

**Skip this skill when:**
- The answer is discoverable from the codebase, the repo history, or a search — go find it instead
- The decision is reversible and cheap, and asking costs more than being wrong
- You are mid-implementation and the question is technical, not product (ask directly, do not run a whole interview)

## Core Process

1. **State the goal and the cost** — Open with what the interview is for and roughly how many questions it will take. "I need about 8 questions to write the brief" respects the subject's time and sets an end point.

2. **Research first** — Before asking anything, read what is already available: the repo, existing specs in `specs/`, prior artifacts in `product/`, the README. Never spend a human question on something you could have read. Say what you already found, so they correct you rather than repeat you.

3. **Ask one question at a time** — Wait for the answer before asking the next. A batch of ten questions gets three answered badly. One question gets one answered well, and the answer changes what you ask next.

4. **Follow the answer, not the script** — The question list is a starting point. When an answer reveals something unexpected, follow it. An interview that ignores what it hears is a form.

5. **Play back what you heard** — After a substantive answer, restate it in one sentence and let them correct it. Misunderstandings are cheapest to fix in the moment.

6. **Record verbatim** — Write answers to `product/interview-<topic>.md` in the subject's own words. Do not tidy their phrasing into your own. The specific words a client uses about their problem are evidence, and paraphrasing destroys it.

7. **Mark what stayed unknown** — Every question they could not answer gets recorded as `[UNKNOWN]` with the reason, and every inference you made gets `[ASSUMED]`. These become risks in the brief, not silent gaps.

8. **Stop at diminishing returns** — When answers start repeating or turn vague, stop. Say what you got, what is still open, and what you will do with it.

## Specific Techniques

### One Question, One Turn

The single rule that makes this skill work. Not "here are 8 questions, answer what you can" — that is a form, and it produces form-quality answers. Ask, listen, follow up, then move on.

If the subject volunteers answers to later questions, take them, acknowledge it, and skip ahead. Never re-ask something already answered.

### Ask What Only They Know

Sort every candidate question before asking it:

| Kind | Example | Action |
|------|---------|--------|
| Discoverable | "What framework is this?" | Read `package.json`. Never ask. |
| Inferable | "Should the list paginate?" | Propose a default, ask them to confirm |
| Human-only | "Which of these users actually pays you?" | **Ask.** Nothing else can answer it. |

Human-only questions are the whole point: who the customer is, what they have tried before, what the real constraint is, what happens if this is never built, what "done" would mean to them. Spend the interview budget there.

### Make Answering Easy

An open question gets a vague answer. Offer concrete options and let them correct you:

> ❌ "What are your success metrics?"
> ✅ "How would you know in a month that this worked? Some options: more signups convert, support tickets about X drop, you personally stop doing Y by hand. Or something else?"

Offering options is not answering for them. Deciding which option is right without asking is.

### "I Don't Know" Is Data

When someone cannot answer, do not push and do not fill it in. Record it:

```markdown
### How many users hit this today?
[UNKNOWN] — no analytics on this flow yet. Owner said "probably a handful."
→ Risk: the whole feature may be sized for a problem nobody has.
```

An honest `[UNKNOWN]` is more useful than a confident invention, because the brief can carry it forward as a risk and the critic can flag it.

### The Five Questions Worth Asking About Any Product Idea

When you have no better starting point:

1. Who specifically has this problem today, and how do you know?
2. What do they do instead right now?
3. What happens if this never gets built?
4. What would make you say, in a month, that this worked?
5. What are you deliberately not doing?

Question 2 is the most under-asked and most revealing: if the answer is "nothing, they live with it," the problem may not be painful enough to pay for.

### Output Format

```markdown
# Interview: <topic>
Date: <YYYY-MM-DD> · Subject: <who> · Purpose: <what this feeds>

## Already known before asking
- <what you read from the repo/specs, and where>

## Q&A
### <question as asked>
<answer, in their words>
→ <one-line implication, marked as yours>

## Unknowns
- [UNKNOWN] <question> — <why unanswered> → <risk it creates>

## Assumptions I made
- [ASSUMED] <inference> — <what it rests on>
```

## Common Rationalizations

| Rationalization | Reality |
|----------------|---------|
| "I'll send all the questions at once to save their time" | It costs them more. Batches get skimmed and half-answered. |
| "I can infer the answer from context" | Then say so and ask them to confirm. Inference presented as fact is the failure this skill exists to prevent. |
| "They said 'I don't know', I'll put something reasonable" | Record `[UNKNOWN]`. A reasonable invention is still an invention. |
| "I'll clean up their phrasing for the document" | Their words are evidence. Record verbatim, interpret separately. |
| "I should ask about the tech stack too" | Read it. Never spend a human question on something in `package.json`. |
| "They're busy — I'll ask fewer questions and assume the rest" | Ask fewer questions and *record the rest as unknowns*. Assuming is the one option not available. |

## Red Flags

- More than one question in a turn
- Any question answerable by reading the repo
- An answer recorded in your words rather than theirs
- A gap filled with a plausible invention instead of `[UNKNOWN]`
- The interview following a script after an answer contradicted it
- No stated end point, so the subject cannot see how long this will take
- Interpretation blended into the transcript instead of marked as yours

## Verification

- [ ] Each question was asked in its own turn, after the previous answer
- [ ] The repo, specs, and prior artifacts were read before any question was asked
- [ ] Answers are recorded verbatim, with interpretation marked separately
- [ ] Every unanswered question is recorded as `[UNKNOWN]` with the risk it creates
- [ ] Every inference is marked `[ASSUMED]` with what it rests on
- [ ] Nothing the subject did not say is presented as something they said
- [ ] The transcript is saved to `product/interview-<topic>.md`
