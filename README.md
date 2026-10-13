# claude-skills

Personal [Claude Code](https://docs.claude.com/en/docs/claude-code) skills.

Skills live under `~/.claude/skills/`.

## Setup on a new computer

These skills must end up at `~/.claude/skills/` for Claude Code to discover them.

**If `~/.claude/skills/` does not exist yet** (fresh machine), clone directly into it:

```bash
git clone https://github.com/syrashid/claude-skills.git ~/.claude/skills
```

**If `~/.claude/skills/` already exists** (e.g. Claude Code created it), `git clone`
into a non-empty directory will fail. Initialize in place instead:

```bash
cd ~/.claude/skills
git init
git remote add origin https://github.com/syrashid/claude-skills.git
git fetch origin
git checkout -t origin/main   # adjust if the default branch differs
```

Then confirm the skills are picked up by running `/delta-review` or `/grill_me`
in Claude Code (restart the session if they don't appear immediately).

To pull future updates: `git -C ~/.claude/skills pull`.

## Skills

- **delta-review** — Senior architect + appsec code review of the current branch.
  High-signal findings across correctness, architecture, performance, security,
  and maintainability.
- **grill_me** — Runs an oral exam on your own recent branch work in three parts:
  high-level summary, line-by-line deep dive, then an adaptive, tangent-safe
  grilling that grades hard and teaches the gaps. Automatically folds in a recent
  `explain-diff-html` explainer as context when one exists.
- **explain-diff-html** — Produces a rich, self-contained HTML explanation of a code
  change (background, intuition, code walkthrough with diagrams), then hands off to
  `grill_me` to test your understanding.
- **use-build-and-review-sequence** — Builds a plan with subagents in tasks under 100 lines (tests
  excluded), tracks them in a ledger, then reviews each task one at a time (what and
  why, then all the code), fixes feedback as follow-up commits, and ends with an
  end-to-end walkthrough.
