# Checkpointing and Resume Protocol

Scaffolding is long, stateful, and — uniquely in this pipeline — **its partial
output is a half-built repo, and regenerating over it is destructive.** The
protocol exists so a stopped or failed run resumes safely instead of
clobbering work.

## The progress tracker

Maintain `docs/.scaffold-progress.md`, updated at every natural boundary —
after preflight, after each unit's generation, after each wiring step, after
each foundations item, after verification. Suggested shape:

```markdown
# Scaffold Progress

> Statuses: pending | in-progress | done
> Last updated: <timestamp>

## Phase 1–2 — Contract & gaps
- [done] Scaffold plan confirmed (units: web, api, worker · monorepo · pnpm)

## Phase 3 — Preflight & tooling
- [done] Preflight (node 22, pnpm 9, docker ok)
- [done] Generators identified & recorded (see scaffold-notes)

## Phase 4 — Generation
- [done] web  (apps/web)
- [in-progress] api  (apps/api)
- [pending] worker

## Phase 5 — Skeleton wiring
- [pending] tokens wiring · shell page · api endpoint · db + migration 001 · e2e test

## Phase 6 — Foundations
- [pending] CI · envs · logging · deploy config

## Phase 7 — Verification
- [pending]
```

The confirmed scaffold plan is recorded in the tracker (or scaffold-notes) so
a resumed session executes the *same* plan — resume never re-opens Phase 1
decisions unless the user asks.

## Resuming (Phase 0)

On every invocation, before anything else:

1. Look for `docs/.scaffold-progress.md` **and** physical signs of a partial
   scaffold (generated unit directories, workspace config).
2. **Neither exists** → fresh start: create the tracker, proceed to Phase 1.
3. **Tracker exists with pending/in-progress items** → resume: summarize
   done/pending, confirm, continue at the first pending item. For an
   `in-progress` generation step, inspect what the generator left behind —
   complete it or cleanly remove *that unit only* and regenerate it; **never
   delete or regenerate completed units.**
4. **Tracker complete** → the scaffold is done; the user likely wants
   something else (a new unit? a fix?) — ask rather than assume.
5. **Physical scaffold present but no tracker** (someone else's repo, or the
   tracker was deleted) → do not touch anything; report what was found and ask
   how to proceed. This skill never assumes ownership of structure it can't
   account for.

## The cardinal rule

**Never silently regenerate over existing work.** Generators are not
idempotent; a re-run into a non-empty directory can overwrite customizations,
duplicate config, or fail halfway and corrupt both attempts. Any regeneration
is scoped to a single named unit, announced, and confirmed.

## Honest limits

Checkpoint granularity is the natural boundary — a crash mid-generator loses
that unit's partial output (which is why step 3 inspects before continuing).
The tracker is a working file, not a deliverable; it can be git-ignored or
deleted after delivery, though keeping it until first deploy is harmless and
useful.
