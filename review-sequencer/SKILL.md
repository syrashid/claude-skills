---
name: review-sequencer
description: Use when executing an implementation plan that Sy will review afterward, or when Sy asks to review built work task by task, walk through commits, or "start the review".
---

# Review Sequencer

Build with subagents, then walk Sy through every task, one at a time, and finish with an end-to-end view. Sy reviews the code to learn it, so every review is plain English and shows all of the code.

## 1. Build

**REQUIRED SUB-SKILL:** superpowers:subagent-driven-development.

- Every task stays under 100 changed lines, not counting tests. Split a bigger plan task at a natural seam (one model, one controller, one set of views) before dispatching it.
- One commit (or more) per task. Don't start reviewing when the build finishes; hand back and wait for "let's review".

## 2. Ledger

Keep `docs/superpowers/plans/<plan-name>-review.md` beside the plan. It is the source of truth for where the review is.

```markdown
| # | Task | Commits | Status |
|---|------|---------|--------|
| 1 | ProjectStage model | a1b2c3d | reviewed |
| 2 | Stages controller | e4f5a6b, 9c8d7e6 (fix: upcase in view) | in review |
| 3 | Reorder endpoint | 1a2b3c4 | pending |
```

Update it after every commit and every piece of feedback. After a context reset, read it and resume at the first task not marked reviewed. When the work ships, fold anything worth keeping into the spec and delete the ledger.

## 3. Review one task per message

Each review message has these parts, in order:

1. **Tally:** one line per task, with its status.
2. **What and why:** two to four plain sentences. What this task adds, and why it's built this way.
3. **The code:** every changed file in full fenced blocks, in the order a request travels through them. Show all of it. No `...`, no excerpts. A sentence or two under each file, for the decisions that aren't obvious from the code.
4. **Stop.** End with "Questions, or on to task N?"

Write for a person, not an agent: short sentences, everyday words, and a technical term only with a plain explanation.

## 4. Feedback

- Make the change as a **new follow-up commit**. Never amend, fixup, squash, or rebase. Rewriting history is not worth the trouble.
- Run the affected tests. Show the new commit's code, and note it in the ledger against the task.
- Pick up exactly where you left off: the same task, unless Sy says to move on.

## 5. End-to-end review

After the last task is reviewed, send one short walkthrough of the whole system. Follow one real user action from start to finish (for example, "dragging a card to a new stage") through the pieces that were built, and name which task each piece came from. No code dumps; Sy has already seen the code.
