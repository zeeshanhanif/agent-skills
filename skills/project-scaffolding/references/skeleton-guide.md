# Skeleton Guide

Mechanics for Phases 5–6: wiring the walking skeleton and realizing the
foundations. The plan's skeleton spec is the contract; this guide is how it
becomes running code. The skeleton's purpose is to prove the architecture
*runs* — every major container exercised by one trivial end-to-end operation,
with stubs exactly where the plan said stubs.

## The wiring contract, per unit type

- **Frontend surface(s)** — the **UI shell**: layout, navigation frame, and
  the design system wired in (below). One page/screen that calls the API and
  renders the result. No feature screens — the shell is the landing place for
  per-slice UI work later.
- **API unit** — one trivial endpoint (e.g., a health-or-echo route that
  round-trips the database): request in → domain stub → store → response out.
  Error handling conventions in place (the cross-cutting concepts' shape, not
  their full depth).
- **Worker/async unit** (if the architecture has one) — one no-op job through
  the real queue/trigger mechanism locally, proving the async path exists.
- **Database/store** — running locally (docker-compose or the ecosystem's
  local equivalent), one table/collection the skeleton reads and writes, plus
  the migration mechanism initialized (the *mechanism*, with migration 001 —
  not the schema; schemas are per-slice detailed-design work).
- **The end-to-end test** — one automated test driving the whole path:
  UI-or-API entry → store → back, asserting the round trip. This test *is*
  the done-when condition's local half, encoded.

Stubs are honest stubs: clearly named, returning fixed shapes, with a comment
pointing at the slice that will replace them. The plan's real-vs-stubbed list
is authoritative — don't gold-plate a stub into a feature.

## Wiring the design system

From ux-foundations' outputs:

- **tokens.json → the stack's token mechanism.** DTCG JSON transforms to
  whatever the frontend uses — CSS custom properties, a theme config, a
  Tailwind config's values. Wire it so the shell *consumes* the tokens (the
  verification check is literal: delete tokens.json's values and the shell
  visibly breaks). Don't hand-copy values into components — the wiring is the
  point.
- **design.md referenced from the agent-instructions file**, so every future
  UI session builds inside the system. The shell's own minimal styling (layout,
  nav) uses the tokens, demonstrating the pattern per-slice work will follow.
- Multi-surface projects: each frontend unit gets the wiring; surface-specific
  token overrides (from ux-foundations Part B) apply per surface.

## Foundations (Phase 6)

The plan's engineering-foundations checklist made real — sized to the
checklist, not beyond it:

- **CI pipeline**: lint + test + build for every unit, on the target the plan
  or user named. It must be written to pass — a red pipeline at delivery is a
  verification failure, not a TODO.
- **Environments as config**: dev/staging/prod configuration files or
  parameterization per the architecture's deployment view — *written, not
  provisioned*. Secrets handled by the ecosystem's standard mechanism with
  placeholder documentation, never committed values.
- **Observability hooks**: structured logging in every unit at minimum;
  metrics/tracing scaffolded only if the architecture's cross-cutting concepts
  demanded them at skeleton stage.
- **Test harness**: the ecosystem-standard runner per unit, wired into CI,
  containing at least the end-to-end skeleton test (and whatever unit-test
  scaffolding the generator provided, kept).
- **Deployment config written**: for the architecture's stated target —
  Dockerfiles, service config, IaC skeleton as appropriate — syntactically
  valid, checked into the repo, **not executed**. The delivery summary states
  plainly: first deploy is the user's step, and the done-when's deployed half
  is pending until then.
- **Cross-cutting FRs placed in foundations by the plan** (e.g., audit-logging
  groundwork) get their hooks here, with their FR IDs in a comment — the
  traceability carries into code.

## What the skeleton is not

Not an MVP, not slice 1, not a demo. It's the proof that the architecture
runs and deploys, and the substrate every slice attaches to. Resist making it
impressive; make it *green*.
