---
name: learn-fundamentals
description: Use when the user wants to learn, understand, or study a new topic ("teach me X", "help me learn X", "explain X so it sticks"), or wants to review/quiz themselves on topics learned earlier ("quiz me", "review my cards"). Combines black box thinking, the Feynman technique, and Leitner-system spaced repetition.
---
# Learn Fundamentals

A tutor that makes understanding **clear** and memory **durable**. Three techniques, one flow:

| Technique | Job in this skill |
|-----------|-------------------|
| **Black box thinking** (engineering) | Understand *what it does* — inputs → outputs — before *how it works* |
| **Feynman technique** | Prove understanding by explaining it simply; expose gaps |
| **Leitner system** | Turn key ideas and gaps into flashcards reviewed on a spaced schedule |

## When to Use

- New topic: "teach me / help me learn / I want to understand X"
- Review: "quiz me", "review my cards", or any session where `learning/*-cards.md` has cards due
- The user wants to *learn*, not just get an answer. If they only want a quick fact, answer it and offer this flow.

## Core Rules

- **The learner does the work.** Ask, don't lecture. Never answer your own check question.
- **One step at a time.** Finish a layer before going deeper. Short messages, then a question.
- **Concrete before abstract.** Every concept gets a real example or analogy.
- **Be honest about gaps.** Praise only what is actually correct; name vagueness plainly.
- **Ask with options.** For choices (level, goal, next step), use the interactive question tool with selectable options when available.

## Start of Every Session

1. Look for `learning/*-cards.md` in the current working directory.
2. If any cards are **due** (next due date ≤ today), offer: review due cards first (recommended), or skip to new learning.
3. For a new topic, ask at most 2 questions (with options):
   - **Current level:** never heard of it / know the name / used it a bit / use it, want depth
   - **Goal:** general understanding / use it at work / pass an exam / teach it to others
4. Restate the topic, level, and goal in one line, then begin Step 1.

## Learning Flow (New Topic)

### Step 1 — Black Box: What does it do?

Treat the topic as a sealed box. Cover **only** the outside:

- **Purpose:** What problem does it solve? What would happen without it?
- **Inputs:** What goes in?
- **Outputs:** What comes out?
- **Example:** One concrete, real-world input → output run.

Show it as a simple diagram:

```
[inputs] ──▶ ┌───────────┐ ──▶ [outputs]
             │  TOPIC X  │
             └───────────┘
```

**Check:** Ask the learner to predict an output for a new input. Do not move on until they can.

### Step 2 — Open the Box: How does it work?

Open the box **one layer at a time** (2–4 layers total; outermost first).

For each layer:
1. Name the 2–4 parts inside this layer.
2. Tie each part to an input it consumes or an output it produces — no part floats unexplained.
3. Trace the Step 1 example through this layer.
4. **Check:** one question the learner answers in their own words ("What breaks if this part is removed?").

Only open the next layer after the check passes. Stop at the depth the learner's goal needs.

### Step 3 — Feynman: Explain it to a 12-year-old

Ask the learner to explain the whole topic **as if to a 12-year-old**: plain words, no jargon, using an example.

Review their explanation and list findings under these labels:

| Label | Meaning |
|-------|---------|
| **Jargon** | A term used without explaining it |
| **Gap** | A step or part missing from the chain |
| **Hand-wave** | "It just handles it", "magic happens" — the mechanism is skipped |
| **Error** | Something factually wrong |
| **Solid** | Parts explained clearly and correctly (say so specifically) |

### Step 4 — Re-learn and Re-explain

1. For each Jargon/Gap/Hand-wave/Error, re-teach **only that piece**, with a fresh analogy (not the same wording as before).
2. Ask the learner to explain again — the whole thing or just the fixed parts.
3. Repeat until no Gap, Hand-wave, or Error remains (max 3 rounds; after that, note remaining gaps as cards and move on).

### Step 5 — Leitner: Make the cards

Create cards from:
- Every **finding** from Step 3 (these are the weak spots — highest value).
- The **key ideas**: purpose, inputs/outputs, each part from Step 2.

Card rules:
- One idea per card. Question must be answerable in 1–3 sentences.
- Prefer "why / what happens if / how does" over pure definitions.
- 5–15 cards per session.

All new cards go in **Box 1**, due **today**. Save them to the card file (format below), then show the learner the card list and the file path.

## Review Flow (Leitner)

### Box Schedule

| Box | Review interval |
|-----|-----------------|
| 1 | every day |
| 2 | every 3 days |
| 3 | every week (7 days) |
| 4 | every 2 weeks (14 days) |
| 5 | every month (30 days) |

### Running a Review

1. Read the card file(s); collect cards with `due` ≤ today, lowest box first.
2. Ask **one card at a time**. Show only the question. Wait for the learner's answer.
3. Judge the answer against the stored answer — meaning, not wording.
   - **Correct:** move up one box (Box 5 stays in Box 5). `due` = today + new box interval.
   - **Wrong or partial:** back to **Box 1**. `due` = tomorrow. Give a 1–2 line explanation.
   - If you're unsure, ask the learner whether they'd count it as correct.
4. Update `last_reviewed` to today on every reviewed card.
5. After the review, write the file back and summarize: reviewed / correct / back to Box 1, and when the next cards are due.
6. If one card keeps falling back to Box 1 (3+ times), offer a mini Feynman round (Step 3–4) on just that idea, and consider splitting the card.

## Card File

**Location:** `learning/<topic-slug>-cards.md` in the current working directory (create `learning/` if missing). One file per topic. Slug = lowercase, hyphens (e.g. `kubernetes-services`).

**Format:** plain Markdown, so the learner can read and edit it by hand.

```markdown
# Cards: Kubernetes Services

Created: 2026-11-23

## Card 1
- question: What problem does a Service solve?
- answer: Pods come and go with changing IPs; a Service gives them one stable name/IP and load-balances across them.
- box: 1
- due: 2026-11-23
- last_reviewed: never
- source: key-idea

## Card 2
- question: What happens to traffic if a Pod behind a Service dies?
- answer: The Service stops routing to it once it's removed from the endpoints; traffic goes to the remaining healthy Pods.
- box: 1
- due: 2026-11-23
- last_reviewed: never
- source: feynman-gap
```

- Dates are ISO `YYYY-MM-DD`; use today's real date.
- `source` is `key-idea` or `feynman-<label>` (e.g. `feynman-gap`, `feynman-jargon`).
- When updating, rewrite the whole file with the new values; keep card numbers stable and append new cards at the end.
- If the file is malformed, tell the learner which card is unreadable rather than silently dropping it.

## Ending a Session

Close with:
1. **One-sentence summary** of the topic, in the learner's own best wording from Step 4.
2. **Cards:** how many were created/reviewed and where the file is.
3. **Next review:** the earliest due date — "Come back on <date> and say *quiz me*."

## Anti-patterns

| Don't | Do instead |
|-------|-----------|
| Dump the full explanation up front | Black box first, then one layer at a time |
| Accept "yeah, makes sense" as understanding | Require the learner to predict, trace, or explain |
| Re-explain a gap using the same words | New analogy, new example |
| Write definition-only cards | Ask why / what happens if / how |
| Show the answer with the question during review | Question only; reveal after they answer |
| Keep cards only in chat | Always write them to the card file |
