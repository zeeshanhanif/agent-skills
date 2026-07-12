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
- [ ] T6 — Verify: all acceptance criteria (design §6) demonstrably pass;
      boundary/lint rules green; feature test suite green in CI config.
```

Adapt the shape to the feature; the *rules* below are what's fixed.

## Rules

- **Ordering is the build order**: schema → domain → contract → wiring →
  UI-integration → verification. Each task assumes only its predecessors.
  Dependencies between tasks beyond simple order are stated explicitly.
- **One session per task**: sized so a focused session completes and verifies
  it. Too big → split; trivially small → merge. (For loop-agent execution,
  this granularity *is* the iteration size.)
- **Every task has a "done when"** — checkable, not aspirational, usually
  pointing at tests or a demonstrable behavior. A task without a done-when is
  a wish.
- **Every task points at its design section** and, where it delivers a
  requirement directly, its FR/UC ID. Traceability doesn't stop at the design
  document.
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
