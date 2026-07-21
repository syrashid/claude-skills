---
name: explain-diff-html
description: Use when the user asks for a rich explanation of a code change, diff, branch, or PR. Produces a self-contained HTML explainer, then hands off to grill_me for testing understanding.
---

# Explain Diff

Please make me a rich, interactive explanation of the specified code change.

It should have these sections:

- Background: Explain the existing system relevant to this change. (You should broadly explore surrounding code for this.) Write for a teammate who knows the language and framework but not this corner of the codebase: cover our domain concepts and design decisions, skip framework/tooling basics, and don't get bogged down in obscure quirks. Up front, answer the questions a reviewer would ask first: what approach was taken and the key implementation decisions behind it.
- Intuition: Explain the core intuition for the code change. The focus here is to explain the essence, not the full details. Use concrete examples with toy data. Use figures and diagrams liberally.
- Code: Do a high-level walkthrough of the changes to the code. Group/order the changes in an understandable way.

Format:

- Output a single self-contained HTML file which includes CSS and JavaScript. Make the whole thing one long page with section headers and a table of contents. Don't use tabs for the top-level structure. Basic responsive styling so you can view it on a phone is nice too. Put the file in a global place on my computer outside of the code repo, and make sure the filename always starts with today's date in `YYYY-MM-DD-` format, because it helps keep the files time-sorted and out of version control. For example: /tmp/2026-01-12-explanation-<slug>.html
- Please write with the clarity and flow of Martin Kleppmann — technical but concise; every sentence should earn its place. Transitions between sections should be smooth. Say things plainly: no invented shorthand or metaphor labels (e.g. "no new permission axis", "takes one more hop") where a direct technical statement works.
- Some tips on diagrams. Ideally, you should pick a small number of diagram families that can be reused throughout the explanation to explain various cases. Some useful kinds of diagrams:
  - A very simplified version of the UI that the user sees in the app, to explain UI changes.
  - A system diagram showing data flow or communication between components. Make sure to include example data here!
- Don't use ASCII diagrams. Always use simple HTML designs for your diagrams, HTML lists for lists of things, etc.
  - For code blocks, always use `<pre>` tags. If you use a custom styled div instead, it **must** have
    `white-space: pre-wrap` in its CSS, or the browser will collapse all newlines into a single line.
    Before saving the file, scan each code block in the HTML source and confirm its CSS includes
    `white-space: pre` or `pre-wrap`.
- Use callouts for key concepts or definitions, important edge cases, etc.

## After you produce the file — hand off to grill_me

Reading an explanation proves you *followed* it, not that you *understand* it. So don't end with a quiz baked into the page — a passive multiple-choice click-through is weak signal and easy to breeze past. Instead, once the file is written:

1. Tell the user where the file is (the full path) and give a one-line summary of what it covers.
2. Prompt them to pressure-test their understanding with the **grill_me** skill, e.g.:

   > The explainer is at `/tmp/2026-01-12-explanation-<slug>.html`. When you've read it, say **"grill me"** and I'll run an oral exam on these changes — grill_me will pick up this explainer as context and grill you on the concepts it introduced.

Leave the file path visible so grill_me can discover it (grill_me looks for the most recent `/tmp/*-explanation-*.html`). Do **not** start grilling in this same turn — let the user read first and invoke grill_me when ready.
