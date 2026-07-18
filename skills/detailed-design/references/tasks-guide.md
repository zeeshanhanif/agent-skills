# Tasks Guide (tasks.md)

The feature's design decomposed into an **ordered, executable task list** — the
pipeline's homolog of spec-driven development's `tasks` step, with one
structural difference that is the whole point: these tasks inherit their
*what* from fixed FR/UC IDs and their *how* from technical-design.md, so a fresh session
(or a loop agent iterating with disk as memory) can execute them
top-to-bottom without re-deriving or reinterpreting anything.

## tasks.md structure

```markdown
# Tasks: FEAT-NNN — <Feature name>

> Executes: docs/features/FEAT-NNN-<slug>/technical-design.md
> Status: pending | in-progress | done per task · Last updated: <date>

- [ ] T1 — Migration: add <tables/columns> (design §4)
      Done when: migration applies clean up and down against current schema.
- [ ] T2 — Domain: <module/service> logic for <behavior> (design §5; FR-…)
      Done when: unit tests for AC-1..3 pass.
- [ ] T3 — Contract: implement <METHOD path> incl. error responses (design §3; UC-… flows)
      Done when: contract tests pass for success + designed errors.
- [ ] T4 — Wire: replace skeleton stub <name>; connect route → domain → store
      Done when: end-to-end path exercises the real implementation.
- [ ] T5 — UI integration point: consume contract in SCR-… screens
      (screens themselves: ui-design's manifest — this task is the wiring only)
- [ ] T6 — E2E: extend the suite for <flow> [UC-…] to cover the path this
      feature completes (only when the feature touches an architecture-named
      critical flow)
      Done when: the flow's E2E spec passes against the local stack.
- [ ] T7 — Verify: all acceptance criteria (design §6) demonstrably pass;
      boundary/lint rules green; feature test suite green in CI config.
```

Adapt the shape to the feature; the *rules* below are what's fixed.

## Rules

- **Ordering is the build order**: schema → domain → contract → wiring →
  UI-integration → E2E (when owed) → verification. Each task assumes only its predecessors.
  Dependencies between tasks beyond simple order are stated explicitly.
- **One session per task**: sized so a focused session completes and verifies
  it. Too big → split; trivially small → merge. (For loop-agent execution,
  this granularity *is* the iteration size.)
- **Every task has a "done when"** — checkable, not aspirational, usually
  pointing at tests or a demonstrable behavior. A task without a done-when is
  a wish.
- **Write each done-when as the right kind for the task's nature** — the
  classification is made here, at design time, so implementation executes it
  rather than improvising per task. **Behavioral tasks** (logic, decisions,
  transformations, contract semantics, validation) get test-artifact
  done-whens: "unit tests for AC-1..3 pass", "contract tests pass for success
  + designed errors" — runnable, rerunnable, tracing to criteria.
  **Structural/realization tasks** (schema objects, wiring, realizing a
  screen spec, config) get demonstration done-whens: "migration applies clean
  up and down", "screen renders and conforms to its spec" — no ceremony test
  asserting mere existence; the structure's behavior is covered by the logic
  tasks that consume it. Verification is universal; test artifacts attach to
  behavior.
- **Every task points at its design section** and, where it delivers a
  requirement directly, its FR/UC ID. Traceability doesn't stop at the design
  document.
- **The E2E obligation is computed, flow-aware, per feature.** When composing
  the list, intersect this feature's UC IDs with the **critical flows named in
  the architecture's cross-cutting Testing entry** (skip gracefully when the
  architecture is silent on testing). Non-empty → mint an explicit task before
  the final verification task: *"extend the E2E suite for <flow> [UC-xxx] to
  cover the path this feature completes"* — done when the E2E spec for that
  path passes against the local stack. Flow-aware means: extend coverage to
  what the flow can now demonstrably do end-to-end, not a full-flow test of a
  flow that's one-third built (testing against stubs is churn without value);
  the feature completing a flow's last segment carries the task that makes the
  whole flow's E2E real. This is how the architecture's thin-E2E strategy
  stays real instead of silently becoming no-E2E.
- **The last task is always verification** — the acceptance criteria pass,
  demonstrated. A feature whose tasks are done but whose criteria weren't run is
  not done.
- **UI tasks are integration-only here**: the screens belong to ui-design's
  output (its manifest joins on SCR IDs); tasks.md covers consuming the
  contract in them, not designing them.
- **Status is updated in place** as implementation proceeds — tasks.md doubles
  as the feature's progress tracker across sessions. Implementation sessions
  check boxes; they don't restructure the list (a needed restructure means the
  design changed — revisit technical-design.md first).

## What tasks are not

Not tickets (the plan's optional export owns that shape), not a Gantt, not
per-function todo noise. The list is the feature's *execution program*: complete
enough that nothing is left to memory, small enough to stay legible.
