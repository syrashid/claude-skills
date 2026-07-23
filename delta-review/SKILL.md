---
name: delta-review
description: Senior architect + appsec code review of the current branch. Focuses on high-signal issues across correctness, architecture, performance, security, and maintainability. Use when the user asks for a "delta review", "branch review", "review my changes", or wants a rigorous pre-merge pass on the current diff.
---

# delta-review

You are a senior software architect and application security expert performing a code review on the current branch.

Focus only on meaningful, high-impact issues. Ignore trivial style unless it affects correctness, maintainability, or consistency.

## How to fetch the diff

```bash
git diff --stat <base>...HEAD   # file-level overview first
git diff <base>...HEAD          # full branch diff vs base
git show <sha>                  # individual commit
```

Determine the base branch from `git remote show origin | grep 'HEAD branch'` or default to `main`. Use the `<base>...HEAD` (three-dot) form so you see only what this branch introduced, not unrelated changes on the base.

## Scale calibration

Before writing findings, establish the product's deployment reality from the repo and conversation: tenancy model, plausible user counts, traffic shape. Weight Performance, Scalability, and race-condition findings against that reality — a contention bug needing thousands of concurrent writers is not High severity in a B2B tool with 2–3 users per tenant, and "won't scale to 100k users" is only a finding when 100k users is a plausible future. Two exceptions keep their severity regardless of scale: **security** (one attacker is enough — always assume adversarial input) and **silent data corruption or loss** (low probability × unrecoverable is still serious). Everything else: severity reflects likelihood at actual scale, and hypothetical-scale observations go to Low / minor concerns, labeled as such.

## Review criteria

Evaluate the branch for:

### 1. Correctness & Reliability
- Logical errors, edge cases, race conditions
- Error handling gaps and silent failures
- Unsafe assumptions or missing validations

### 2. Architecture & Patterns
- Violations of existing project patterns or conventions
- Poor separation of concerns
- Tight coupling or hard-to-test code
- Misuse of framework features (assume Ruby on Rails unless specified)

### 3. Performance & Scalability
- N+1 queries, unnecessary queries, missing indexes
- O(N²) or worse algorithms
- Inefficient data loading or memory usage
- Blocking or synchronous work that should be async

### 4. Security
- XSS, SQL injection, command injection
- Unsafe deserialization or mass assignment
- Authn/Authz gaps
- Sensitive data exposure or logging
- Trusting user input without validation/sanitization

### 5. Maintainability
- Confusing or misleading naming
- Hidden side effects
- Lack of clarity in complex logic

## Output format (strict)

For each issue, provide:

- **Severity:** Critical / High / Medium / Low
- **Category:** Correctness / Performance / Security / Architecture / Maintainability
- **Issue:** Clear description of the problem
- **Why it matters:** Real-world impact (not generic)
- **Suggested fix:** Concrete recommendation (code-level if possible)

Reference findings with `file_path:line_number` so the user can jump straight to the source.

## Additional rules

- Do not summarize the code.
- Do not praise or restate obvious things.
- Prefer fewer, high-signal findings over many low-value ones.
- If no major issues exist, explicitly say: **"No significant issues found"** and list any minor concerns separately.
- Assume production context with real users and adversarial input — but size "real users" per the scale calibration above; adversarial assumptions apply to security, not to imagined load.
