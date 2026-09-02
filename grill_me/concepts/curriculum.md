# Curriculum — mid → senior/staff engineering

**This file is the syllabus. The PR is the example material.** Read it every session, find where this engineer is thin, then use the diff as the anchor for teaching those topics.

Six buckets. Topics are named at the **concept** level with the local instance in parens — the concept travels to the next job, the instance makes the question concrete here.

**State markers**
- *no marker* — untouched. Never asked. The richest source of new questions.
- `[ ]` — in rotation. Asked, came back shaky or missed. Retest until nailed.
- `[x]` — solid, with the date it landed. Uncheck if it resurfaces as a miss.

**Notes stay to one line, ≤15 words.** The moment a topic grows a paragraph it has stopped being a curriculum entry and become a bug report. The prose belongs in the session's grade, not here.

---

## 1. Platform depth

*Mid → senior's biggest single gap: you stop treating the framework, browser, database and HTTP as magic.*

### HTTP
- Methods and idempotency semantics
- Status code semantics (what 409 / 422 / 202 actually promise)
- [x] Caching headers — ETag, Cache-Control, conditional requests *(solid 2026-08-28)*
- Content negotiation and `Accept`
- Cookies, SameSite, and third-party context
- Redirect semantics — 301 vs 302 vs 303 vs 307

### Browser security model
- [x] Same-origin policy — what it actually forbids *(solid 2026-08-28)*
- [ ] `importScripts` is a privilege grant, not a boundary; no SRI *(2026-08-28)*
- CORS and preflight — who enforces it, and on which request
- Content Security Policy and nonces
- XSS surfaces — where user text becomes markup
- CSRF mechanics and why token-in-form works

### Browser runtime
- [ ] Event model — capture, bubble, delegation *(drag-and-drop `dropEffect` missed 2026-08-07)*
- Event loop, tasks, microtasks
- DOM lifecycle and when a node is detached
- [ ] Listener lifetime on long-lived targets (`document` / `window`) *(shaky 2026-07-23)*
- Native form validation and constraint API
- File inputs, the File API, and `accept` as UX not enforcement
- Browser storage — localStorage, sessionStorage, IndexedDB, and their failure modes

### Framework request lifecycle (Rails)
- Middleware stack and where a request can be short-circuited
- Routing, constraints, and shallow vs. nested resources
- Strong params as an allowlist
- Filter and callback ordering
- Rendering, layouts, and partial resolution
- [ ] Turbo frame targeting vs. stream dispatch on Content-Type *(missed 2026-08-13)*
- [x] Turbo frame swaps discard out-of-frame content *(solid 2026-08-13)*
- [ ] `Turbo.renderStreamMessage` appends whatever you hand it *(2026-08-07)*

### ORM mechanics (ActiveRecord)
- Lazy relations and when a query actually fires
- [ ] N+1: preload vs. includes vs. eager_load vs. joins
- [ ] Object materialization cost in read-then-aggregate paths *(missed 3× — 2026-07-13, 07-23, 08-05)*
- Dirty tracking and `*_was` / `saved_change_to_*`
- Validations vs. callbacks — which run when
- [ ] Callback-skipping writes (`update_columns`, `update_all`, `insert_all`) *(2026-08-07)*
- [ ] Nested attributes and `reject_if` on persisted children *(2026-08-04)*
- STI, polymorphism, and delegated types
- Association scoping and `dependent:` semantics

### SQL and the database (Postgres)
- Query planning basics — how to read an EXPLAIN
- Index types and selectivity; when an index is not used
- Composite index column order
- Partial and unique indexes as invariant enforcement
- [ ] NULL semantics — grouping, comparison, and unique indexes *(2026-08-24)*
- [x] Ordering determinism and tiebreakers *(solid 2026-08-07)*
- [ ] Raw-SQL allowlists are adapter-specific (`Arel.sql`) *(2026-08-07)*
- JSON/JSONB columns and when they're the wrong tool

### Transactions and locking
- [x] Savepoints, nested transactions, and partial rollback *(solid 2026-08-25)*
- Isolation levels and what each actually prevents
- MVCC — readers don't block writers
- Row-level locking and what it does *not* block (`SELECT FOR UPDATE`)
- Advisory locks and when they beat row locks
- [ ] What a transaction does NOT give you — read-compute-write is not serialized *(partial 2026-08-13)*

### Caching
- HTTP cache vs. application cache
- Fragment and Russian-doll caching; key derivation
- Memoization scope and lifetime (`||=`, `defined?`, per-request vs. per-process)
- Invalidation strategies and why they're the hard part
- [x] Name- vs. content-addressed assets; `immutable` kills revalidation *(solid 2026-08-28)*

### Background work
- Queue semantics and delivery guarantees
- At-least-once and what it demands of the consumer
- Retry, backoff, and poison messages
- [ ] Stranded work is a state, not an exception — needs a sweep *(2026-07-11)*

### Files and blobs
- [ ] Direct-upload topology — metadata-first, bytes second *(half credit 2026-08-25)*
- [x] Content-type sniffing vs. declared type *(solid 2026-08-28)*
- Streaming vs. buffering and memory cost
- [x] Signed URLs as bearer tokens — issuance is not access *(solid 2026-08-28)*

### Auth mechanics
- Sessions vs. tokens — where state lives
- Signing vs. encryption — what each guarantees
- [ ] Signed values in URLs — encoding and serializer pinning *(missed 2026-08-05)*
- Credential storage and rotation

### Encoding and serialization
- JSON vs. Marshal and why Marshal is a deserialization hazard
- Timezones — storage, display, and the boundary between them
- [x] Numeric parsing: prefix-parse vs. strict (`to_i` vs. `Integer(..., exception: false)`) *(solid 2026-08-13)*
- Money and precision — why floats lose

### Language mechanics (Ruby)
- [ ] Closures, `def` scope, and DSL lambda evaluation *(2026-07-16)*
- [ ] Proc vs. lambda arity and callback signature widening *(2026-08-04)*
- Method lookup, modules, and `prepend`
- [ ] Class-level answers to instance-level questions *(missed 2026-07-23)*
- Freezing, mutation, and shared state

---

## 2. Design and boundaries

*Where code goes, what couples to what, and when not to abstract.*

- [ ] Abstraction choice by input/output signature — helper vs. PORO vs. component vs. concern vs. service *(2026-08-13, never tested)*
- Coupling and dependency direction
- When to extract vs. inline; the cost of premature abstraction
- Naming as design — when a bad name is a design smell
- Interface design: what a caller is forced to know
- Composition vs. inheritance; STI vs. delegation
- Where validation belongs — model, form object, controller, or DB
- [x] Where authorization belongs, and at what granularity *(solid 2026-08-28)*
- View-layer boundaries — what a component or presenter may know
- State machines vs. boolean flags vs. derived predicates
- [x] Guard clauses accumulate responsibility as a function grows *(solid 2026-08-05)*
- Form objects and the transaction-boundary question
- Feature slicing — how to cut work so each slice ships

---

## 3. Data modeling and contracts

*Schemas and interfaces that survive requirements changes.*

- [x] Stored vs. computed state; snapshot vs. drifting cache *(solid 2026-08-13)*
- Normalization vs. denormalization and the read/write tradeoff
- Which invariants live in the DB, which in the app, and which live nowhere
- Soft delete and its blast radius across every read path
- Ownership: who owns a shared record when two features reference it
- Evidentiary vs. convenience references — does this record need to be immutable?
- Temporal data — snapshots, effective dating, history tables
- [ ] Enum and vocabulary design; predicates that conflate two questions *(2026-08-13)*
- Multi-tenancy patterns and where isolation leaks
- API and contract versioning; consumer-driven contracts
- One-way doors — which schema decisions ossify once customers have data
- Money modeling: currency, rounding, and stored totals
- Counters, sequences, and gapless numbering

---

## 4. Correctness and failure

*The common failure modes, at this product's real scale — not exotica.*

- [ ] Idempotency — keys, natural idempotency, and target-state proxies *(2026-07-07, 2026-08-24)*
- The dual-write problem and reconciliation
- Retries and deduplication
- Check-then-act / TOCTOU and its remedies
- Input validation and trust boundaries
- [ ] Error taxonomy — expected vs. exceptional, what to rescue and at what scope *(missed 2026-08-25)*
- [ ] Failing loud vs. failing plausible; the cost of a defensive default *(2026-07-11, 07-16, 08-28)*
- [ ] Boot-time vs. call-time config validation — static config fails at boot *(missed 2026-08-28)*
- Partial failure and transaction boundary placement
- Concurrency at real scale — lost updates, and which races actually matter *(cap: 1 question/session)*
- Graceful degradation when a dependency is down
- Data repair — how you fix it after it has already gone wrong

---

## 5. Testing and verification

*What to test, at what level, and how to know a claim is true.*

- Test level selection — unit vs. integration vs. system, and what each buys
- [x] What a test actually asserts vs. what you think it asserts *(solid 2026-08-28)*
- [x] Contract fixtures must come from the counterpart *(solid 2026-08-13)*
- [ ] Recorded fixtures freeze — cassettes go stale silently *(2026-08-13)*
- [x] Pinning converts vendor drift into a scheduled review *(solid 2026-08-28)*
- Determinism and the common sources of flake
- Coverage as a signal vs. as a target
- Testing the failure path, not just the happy path
- Regression tests as executable bug reports
- Testing what markup actually renders vs. what you wrote

---

## 6. Operations and change

*What you can see in production, and what you can take back.*

- Observability: logs, metrics, traces — and choosing between them
- [ ] Client-side failures need instrumentation, not debugging *(2026-08-25; console not reporting, 08-28)*
- Alert design — what fires, and what a page at 2am should tell you
- [x] Schema-reversible is not data-reversible *(solid 2026-08-07)*
- [x] A `down` that succeeds and corrupts *(solid 2026-08-24)*
- [ ] Expand-contract sequencing; the error window is zero, not small *(2026-08-24)*
- [ ] Expand-contract for enum and vocabulary changes *(2026-07-13)*
- Backfills — batching, locking, and resumability
- Feature flags; rollback vs. roll-forward
- [x] Blast radius — console-configured infra is unrecorded *(solid 2026-08-28)*
- [x] Environment assertions belong in the deploy pipeline, not the suite *(solid 2026-08-28)*
- [ ] Debugging in production: what you look at first *(largely untested)*
- Capacity and cost reasoning at the product's real scale
- [ ] An unrecorded promise is unauditable *(2026-08-13)*
- [x] Read the trigger, not the interval *(solid 2026-08-05)*

---

## Standing habits

Not topics — behaviors, retested opportunistically inside other questions.

- [ ] **Anchor claims to code.** Before asserting what a method does at runtime, read its guard clauses and its rescue chain. To predict what a *user* sees, find which handler the failure lands in — not what the happy path does. *(Scope: our code. A dependency's source only for ordering, ambiguous input, or failure behavior.)*
- [ ] **Check the cheap thing first.** Schema claims are one `grep` of `schema.rb`. Config claims are one file read. Never assert from a plausible mental model when the answer is one command away.

---

## Growing this file

At the end of every session, look at the PR again and ask: **did this change surface a topic that isn't in the tree?** If so, append it — unmarked if it went untested, `[ ]` if it was asked and came back weak. That's how the curriculum stays ahead of the stack instead of fossilizing around whatever was being built the month it was written.

Add topics at the concept level, one line, instance in parens. Resist adding a topic that only makes sense inside this codebase — the test is whether it would still be worth knowing at a different company.

Prune when a bucket bloats: merge near-duplicates, and delete `[x]` entries older than a year that have not resurfaced. A curriculum is a map, not a transcript.
