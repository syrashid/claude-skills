---
name: explain-codebase-html
description: Use when someone is new to a codebase or a large subsystem and wants a high-level architecture overview / lay-of-the-land: onboarding to an unfamiliar repo, understanding how an app fits together, or producing a shareable architecture explainer. Produces a self-contained HTML field guide. For explaining a specific diff, branch, or PR instead, use explain-diff-html.
---

# Explain Codebase

Produce a rich, self-contained HTML "field guide" that gives someone the lay of the land for an unfamiliar codebase — enough to know what the app is, how it fits together, and where to look first. Aim for **orientation, not an exhaustive spec**: the reader should finish knowing the shape of the system and which files to open, not every detail.

**Budget:** target a page readable in ~15 minutes — roughly 1,500–3,000 words, 6–10 screens. Cap each section at one or two screens; if a section wants to grow past that, you're writing a spec, so cut. This budget is the main defense against the failure this skill exists to prevent: a bloated, unreadable dump that technically "covered everything." "Read these first" is 5–10 files, never the whole tree.

This is the whole-app sibling of `explain-diff-html`. A codebase is too big to explore linearly in one context, so the method is: **fan out parallel subagents to explore, verify their findings against the real code, then synthesize into one page.**

## Process

### 1. Orient yourself first (quick — a few minutes of reading)
Get the shape before dispatching anyone. Read the dependency manifest (Gemfile / package.json / go.mod / …), the route or endpoint map, the schema / data-model layer, a listing of the main source tree, and any in-repo docs (README, `docs/`, ARCHITECTURE.md). Form a **one-sentence thesis** of what the app is and its major subsystems — this is what you'll brief the subagents with.

### 2. Fan out parallel research subagents
**Scale to the repo — don't over-engineer a small one.** For a small or simple codebase, skip the fan-out and explore inline. Use ~2–3 agents for a typical app, and 4–6 only for a large or multi-service codebase. Don't spawn one agent per bullet. Dispatch them in a single message so they run concurrently (via the Task tool; see `superpowers:dispatching-parallel-agents`). If you're already running as a subagent and can't spawn more, walk the subsystems yourself in sequence.

Split by subsystem — a typical split:
- the **data / domain model** (core entities + relationships);
- each major **entry point** (web UI, API, integrations, CLI) — separately if large;
- the **back of house** (jobs, services, external integrations);
- **build / test / deploy** (config, CI, how to run it).

Give each agent your one-sentence thesis, the scopes the *other* agents own (so they don't overlap), a tight scope of its own, and this instruction: return **synthesis-ready** notes — concrete model/file/endpoint names, notable design decisions, and "the N files a new dev should read first" — **not** file dumps. When their notes overlap or conflict, reconcile in the code's favor (step 3).

### 3. Verify — trust the code over the docs
Hand-written docs (README, ARCHITECTURE.md, a RESOURCES.md) are the most dangerous source: authoritative-sounding but prone to drift. Cross-check every load-bearing claim (state machines, who-can-do-what, whether a job is scheduled, framework versions) against the actual code before it goes in the guide. The corrections you find are often the most valuable content. If there are no in-repo docs, build purely from code — and note their absence as a finding.

### 4. Write the HTML
**REQUIRED: load the `artifact-design` skill first.** This is a utilitarian-technical document — the craft goes into typographic hierarchy, information design, and clear diagrams, not a flashy hero.

The section list below **is a menu, not a checklist** — it assumes a request/response web app. Drop what doesn't fit rather than padding it, and adapt for other shapes: a **CLI** has commands/subcommands instead of "faces" (auth often doesn't apply); a **library / SDK** replaces "faces" and UI mockups with its public API surface and integration points; a **data pipeline** leads with sources → transforms → sinks.

1. **Overview** — what the app is, its core thesis, an at-a-glance stat/fact block (models, endpoints, services, jobs, LOC — whatever characterizes it).
2. **Domain / data model** — central entities and how they relate; one strong ER-style diagram.
3. **Entry points / the "faces"** — how each kind of client reaches the system and its auth model.
4. **Key workflows** — the handful of flows that carry the business, with concrete example data.
5. **Back of house** — jobs, services, external integrations.
6. **Build, test & deploy** — how to run it, the test setup, the release path.
7. **Where things live** — an annotated code map + "read these first" files (5–10, never the whole tree).
8. **Gotchas** — the non-obvious traps (load-bearing files, caches to keep in sync, footguns).

### 5. Critique pass — adversarial fact-check
After the file is written, dispatch a fresh subagent (Task tool) to read the finished HTML, extract its factual claims, and verify each against the codebase with file-level evidence — framed adversarially ("find a claim that's wrong or misleading"). Fix everything it surfaces. Also self-review the rendering: balanced tags, no cascade collisions, `<pre>` whitespace, both light and dark themes. Only publish after these fixes land.

## Format
- A single self-contained HTML file: inline CSS + JS, one long page, section headers, a table of contents (don't use tabs for the top-level structure). Responsive and theme-aware (light/dark). Fully self-contained — the Artifact CSP blocks external fonts / CDNs / images, so inline everything. The source filename starts with today's date in `YYYY-MM-DD-` form, kept outside the code repo (for artifact output this local file is just the tool's input).
- Diagrams are HTML/CSS, **never ASCII**. Reuse a small number of diagram families — system / data-flow diagrams that include **example data**, and simplified UI mockups. Code blocks go in `<pre>` (CSS `white-space: pre` or `pre-wrap`, or the browser collapses newlines). Use callouts for definitions, key decisions, and gotchas.
- Write with the clarity and flow of Martin Kleppmann — technical, concise, plain; every sentence earns its place. Let diagrams do the heavy lifting, and keep it a lay of the land, not a data dump.

## Ship it
Publish with the Artifact tool (it requires a `favicon` emoji, and takes a `title` and a one-sentence `description`). Artifacts start private. If you fixed anything in the step-5 critique, re-publish the same file path to update the artifact in place (same URL) before sharing. Give the user the link and a one-line summary of what it covers.
