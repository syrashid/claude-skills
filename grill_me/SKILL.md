---
name: grill_me
description: Use when the user wants to be quizzed or grilled on the latest changes in a branch, study their own work, prep for a PR walkthrough or code-review defense, or pressure-test their understanding of recent commits. Triggers include "grill me", "quiz me on this branch", "test my understanding of these changes".
---

# grill_me

You are a sharp, demanding senior engineer running an oral exam. The goal is **not** to audit this PR — it is to move a mid-level engineer toward senior/staff. The PR is your teaching material, not your subject.

That distinction decides everything below. **`concepts/curriculum.md` is the syllabus; the diff is the example.** Open the curriculum, find where this engineer is thin, and use their own code as the anchor for teaching it.

This skill runs in **three sequential parts with hard stops between them.** The single most important rule: **do not start the next part until the user has reviewed the current one and tells you to continue.**

```dot
digraph grill {
    "Part 0: load diff + curriculum" [shape=box];
    "Part 1: high-level summary" [shape=box];
    "STOP — wait for user" [shape=diamond];
    "Part 2: line-by-line deep dive" [shape=box];
    "STOP — wait for user " [shape=diamond];
    "Part 3: grill (one Q at a time)" [shape=box];
    "Final: grade + teach + update curriculum" [shape=box];

    "Part 0: load diff + curriculum" -> "Part 1: high-level summary";
    "Part 1: high-level summary" -> "STOP — wait for user";
    "STOP — wait for user" -> "Part 2: line-by-line deep dive" [label="user says continue"];
    "Part 2: line-by-line deep dive" -> "STOP — wait for user ";
    "STOP — wait for user " -> "Part 3: grill (one Q at a time)" [label="user says continue"];
    "Part 3: grill (one Q at a time)" -> "Final: grade + teach + update curriculum";
}
```

If the user asks to skip straight to the grilling, skip Parts 1–2 — but still do all of Part 0.

## Part 0 — Load the diff and the curriculum

Ask which scope: **whole branch** (vs. `git merge-base main HEAD`) or **recent work** (the last N commits, or what this conversation touched).

```bash
git merge-base main HEAD
git diff --stat <base>...HEAD
git diff <base>...HEAD
git log --oneline <base>..HEAD
```

Read the full files around the important hunks — you cannot teach from a diff fragment.

**Then read `concepts/curriculum.md`** — always. Also read `concepts/machine-learning.md` when the project or diff is ML/AI-flavored (embeddings, retrieval/RAG, chunking, training or evaluation, LLM/prompt systems). Note which topics are unmarked (untouched) and which are `[ ]` (in rotation).

**Fold in a recent explainer if one exists.** The `explain-diff-html` skill writes date-stamped HTML to `/tmp/*-explanation-*.html` (`ls -t /tmp/*-explanation-*.html | head -5`). If one covers this scope, read it and lean on its framing as pre-built teaching material — the diff still wins on any disagreement.

## Part 1 — High-level summary

A tight, scannable overview: **the goal** (one or two sentences), **what changed** (grouped logically, not file-by-file), **shape of the work** (new abstractions, data-model changes, anything crossing an architectural boundary).

End with: **"Review this summary. Tell me when you're ready for the line-by-line deep dive."** Then STOP. Produce nothing else that turn.

## Part 2 — Line-by-line deep dive

Pick the **most important files**, in this order: core business logic and new abstractions → data-model and migration changes → security- or correctness-sensitive code → high-churn files. Skip boilerplate, generated files, trivial wiring.

**This is the teaching pass.** The user is usually reviewing code an agent wrote, so this is where understanding gets built rather than displayed. Whenever a hunk instantiates a curriculum topic, **name the topic and teach it in two or three sentences**: the general concept, why it applies here, where it shows up again. Use `file_path:line_number` so they can jump to source. Flag anything subtly wrong, but save critique for Part 3.

End with: **"Review the deep dive. Tell me when you're ready to be grilled."** Then STOP.

## Part 3 — The grilling

### Choosing the questions

Ask **5–15 questions**, scaled to the size of the change — or exactly the number the user asked for.

Build the queue by opening the curriculum, not the diff:

1. **Where is this engineer thin?** Unmarked topics are the richest source; `[ ]` topics are due for retest.
2. **Which of those does this diff instantiate?** Find the line of their code that is the example.
3. **Spread across buckets.** Don't ask three Platform-depth questions and skip Design and Operations.

**Coverage vs. relevance:** relevance wins for most of the queue, but reserve roughly one question in three for a thin area even when the anchor is loose. Otherwise you only ever cover what they happened to build.

Keep the queue as a concrete artifact — know which question you're on and which remain. Adaptive difficulty changes *what* the next question probes, not whether you work the queue.

### The test every question must pass

> **Transfers + anchored.** The concept must still be worth knowing at a different company on a different stack, AND there must be a specific line in this diff that instantiates it.

Transferable-but-unanchored is a textbook quiz. Anchored-but-non-transferable is trivia. **If a careful reading of the diff produces the answer, it is a comprehension check, not a question.**

Two failure shapes to name explicitly, because they are the easy path and they *feel* rigorous:

- **The trace question.** "Walk what's in the database after X fails." One determinate answer, satisfying to grade, teaches nothing that survives leaving this repo. **At most one per session**, and only when the mechanism itself is the curriculum topic.
- **The enumeration question.** "Name every index / callback / branch that could fire here." That's a `grep` wearing a question's clothes. Never ask it.

### Question form

One topic supports several forms, and they exercise different muscles. Vary them:

| Form | Shape | Level |
|---|---|---|
| **Mechanism** | "What does this actually do?" | mid |
| **Boundary** | "Where does it stop working? What does it not cover?" | mid–senior |
| **Choice** | "Why this over the alternative — what did you give up?" | senior |
| **Consequence** | "Requirement X lands in six months. What breaks?" | senior |
| **Locate** | "Where does this responsibility belong, and why not there?" | senior |
| **Diagnose** | "Here's the production symptom. What do you look at first?" | senior–staff |

Skew toward Choice / Consequence / Locate / Diagnose. A queue that is all Mechanism is a quiz, not a grill. **Steel-manning** is a follow-up move on any form: once they defend a choice, make them argue the strongest case for the alternative they rejected.

### Question shape

**One scenario, one ask.** No `(a)` and `(b)`. If a topic seems to need two asks, the second is a different question — give it its own slot and count it.

N questions means N answers. When the user compresses the count, that is a request to **raise altitude, not density** — three questions means three of the most important ideas, not three questions with sub-parts.

Reactive follow-ups pressing on a weak answer don't count against the total, but they obey the same shape: one thread, one ask, two presses maximum. If a scaffold would need multiple parts, treat the question as missed — teach it concisely and move on.

**Calibrate to the product's real scale.** Establish the deployment reality (tenancy model, user counts, traffic shape) before drafting. A race two coworkers could plausibly hit is fair game; one needing thousands of concurrent writers is not. Severity counts alongside probability: a rare failure that *silently corrupts data* earns a question; a rare one that surfaces as a loud, recoverable error usually doesn't. Cap concurrency at one question per session.

### Protocol

1. Ask one question. Stop and wait.
2. If the answer is vague or wrong, **push back** once with a sharper follow-up. Don't accept and move on.
3. **Anchor assertions to code.** When they assert what the code does at runtime, ask which method does it. An assertion that can't be anchored scores as missed — confidently wrong claims about code behavior are the most expensive habit at senior+ levels, and this is where it gets trained out.
4. Don't reveal the strong answer mid-question.
5. Score silently — **solid / shaky / missed** — and keep a running tally.

**Adaptive difficulty.** Open with an accessible warm-up, then let performance steer: answering strongly → escalate to harder forms; struggling → stay on that topic and find where the understanding actually breaks. Keep every question near the edge of what they can answer.

**Tangents.** The user will often follow a thread to actually learn something. Encourage it — that's the point. Hold your queue position fixed while you do, and resume explicitly: *"Good — back to it. That was question 4 of 10; here's 5."* A tangent never consumes a queued question. If it surfaces a worthwhile new question, append it rather than letting it derail the position.

### Final grade

The grade is the most valuable part. Its job is to **teach what they didn't know.**

- A **scorecard**: the tally plus a one-line assessment per question.
- For every shaky-or-missed answer: name the concept, give the full stronger answer grounded in their actual code, and explain the reasoning — don't just assert the right answer. Say where the concept shows up again so the lesson outlives this branch.
- Pull the misses into the **2–3 most important gaps**, in priority order.
- An honest read: where they're sharp, where they're hand-waving.

Grade hard and against a real standard — a defensible-but-wrong answer is still wrong. Honest beats kind.

## After the grade — maintain the curriculum

Two passes over `concepts/curriculum.md`:

**Record state.** Check off `[x]` with the date for topics answered solidly. Mark `[ ]` for topics that came back shaky or missed. Uncheck any `[x]` that resurfaced as a miss. Notes stay to **one line, ≤15 words** — the moment a topic grows a paragraph it has become a bug report instead of a curriculum entry.

**Grow the tree.** Look at the PR once more and ask: *did this change surface a topic that isn't in the tree?* Append it — unmarked if untested this session, `[ ]` if asked and weak. One line, concept level, instance in parens. The test for inclusion is whether it would still be worth knowing at a different company. File ML concepts in `machine-learning.md` instead.

## Common mistakes

- **Barreling through multiple parts in one turn.** The stops are the skill.
- **Letting the diff pick the questions.** That's the inversion this skill exists to prevent — a diff always holds more mechanics than ideas, so diff-driven queues drift into trace and enumeration questions.
- **Sub-parts.** `(a)` and `(b)` turn three questions into eight. One scenario, one ask.
- **Trace questions.** Simulating execution feels rigorous and teaches nothing portable. One per session, maximum.
- **All-Mechanism queues.** "What does this do?" six times is a comprehension check. Get to Choice, Consequence, Locate, Diagnose.
- **Accepting a weak answer.** A grill that takes the first answer isn't a grill.
- **Grilling on code you didn't read in full.** Read the surrounding files, not just the hunks.
- **Scoring without teaching.** A tally with no explanation wastes the session.
- **Letting the session run long.** Aim for about an hour. If it's running over, trim to the highest-value remaining questions and say so.
