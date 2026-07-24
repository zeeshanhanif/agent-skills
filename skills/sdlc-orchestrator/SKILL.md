---
name: sdlc-orchestrator
description: >-
  Drives the pipeline's per-feature construction loop and routes lifecycle
  events — deliberately thin: zero design or build logic; it computes
  position, invokes the right stage skill, and routes outcomes. Loop driving:
  walks the plan's build sequence, determines each feature's stage from
  artifacts (designed / ui-designed / developer-done / verified), invokes the
  next stage, and continues — one feature cycle by default, run-until-blocked
  on request. Routes verdicts and blocks: rework back to implementation,
  design defects to the amendment path, blocked tasks and escalations
  surfaced to the user. Feature-cycle routing: a new feature request on an
  existing system walks the amendment chain — requirements, architecture/ux
  impact, plan — then enters the loop. Maintenance routing: a reported bug
  becomes a failing test, a scoped fix, and a re-verification. Trigger on
  "run the loop", "run the next feature", "continue the pipeline", "project
  status", "add a feature to the system", or "fix this bug".
---

# SDLC Orchestrator

The loop driver and lifecycle router — **deliberately thin**. Every stage
skill already resolves its own position deterministically, keeps its state on
disk, and announces rather than asks; this skill adds only what no single
stage owns: the *global* position, the invocation of the right next stage,
and the routing of outcomes between stages. If logic here starts looking like
design or build logic, it belongs in a stage skill instead.

Three principles govern it:

1. **Compute, never store.** The global position is derived fresh every time
   from the artifacts — the plan's sequence joined to each feature's folder
   state (technical-design.md → designed; ui-design.md → ui-designed;
   tasks.md fully checked → developer-done; accepted acceptance-report.md →
   verified). No orchestrator state file, no stored pointers; a brand-new
   session computes the same position the last one saw.
2. **Skills stay sovereign.** The orchestrator invokes stages and reads their
   outcomes; it never reaches into a stage's job — never edits tasks.md,
   never writes an RTM column, never resolves an escalation itself. Human
   judgment points (amendments, exhausted blocks, design defects) **pause the
   loop and surface** — autonomy applies to the mechanical cycle, not to the
   decisions the pipeline reserved for people.
3. **Every stop is a resumable state.** The loop halts only on the defined
   stop conditions, each leaving disk truthful (that's the stage skills'
   discipline, inherited); resuming is just re-running — the computation
   lands on the same spot.

## The loop (default job)

The per-feature cycle, driven in plan-sequence order:

**detailed-design → ui-design → feature-implementation →
acceptance-verification** → next feature.

Run scopes, chosen by the user's ask: **one stage** ("run the next step"),
**one feature cycle** (the default for "run the next feature" — carry the
current front feature from its present stage to verified), or
**run-until-blocked** ("run the loop" — feature after feature until a stop
condition or the plan completes). Each invocation announces the computed
position before acting ("FEAT-006 is at ui-designed; invoking
feature-implementation").

Stop conditions and their routing — read `references/routing-guide.md`:
- **Rework verdict** → re-invoke feature-implementation with the report's
  findings; then re-verify. (Mechanical — loop continues through it, with a
  bounded number of rework rounds before surfacing.)
- **Design-defect verdict / filed escalation** → pause; surface toward the
  owning amendment path (these are interactive by design).
- **Blocked task (WIP after exhausted attempts)** → pause; surface the
  failure note; on repeated blocking, relay the meta-signal (the feature
  likely needs redesign).
- **Plan complete** (every feature verified) → close the loop; summarize;
  point at what's next (deploy if not yet; the delivered plan).
- **User stop** — always honored, always resumable.

## Feature-cycle routing (change requests on an existing system)

"Add X to the product" after v1 is not a loop entry — it's an amendment
chain, walked in order with the user (upstream skills are interactive by
design). Read `references/lifecycle-routes.md`: requirements-engineering
amendment (new/changed FR-UC, impact notes) → impact check on architecture
and ux-foundations (amend only where the impact notes say so — new NFR
pressure, new screens needing SCR minting) → implementation-planning
amendment (mint the FEAT, touchpoints, sequence insertion) → the new feature
is now simply *next work in the plan*, and the loop picks it up normally.
The orchestrator's job is walking the chain in order and not letting a step
be skipped — each amendment itself belongs to its owning skill.

## Maintenance routing (bugs on verified behavior)

A bug — observed behavior violating an already-verified requirement — fits
neither the loop nor an amendment. Read `references/lifecycle-routes.md`
(Maintenance): locate the owning FR and feature (RTM → Plan ref), record the
defect in the **defect ledger** (`docs/defects.md`, append-only DEF-NNN
rows), then the fix protocol: **a failing test first** (demonstrating the
bug in the owning feature's suite — legitimate new test work), a scoped fix
under feature-implementation's disciplines (bounded attempts, anti-fake-
green, commit hygiene, this-fix-only scope), then **re-run
acceptance-verification** on the affected feature (re-verification is a
normal run; the report gains a dated verdict). If diagnosis shows the
*design* was wrong rather than the code, reroute to the design-defect path
instead — a bug report is a symptom, not a classification.

## Status (on request)

"Project status" renders the computed view, stored nowhere: per feature, its
stage; the loop front; open blocks/escalations/defects; verified count vs.
plan. The RTM and artifacts remain the truth — this is a reading of them.

## Inputs

The plan (sequence), the feature folders (stage detection), acceptance
reports (verdicts), the RTM (maintenance routing lookups), the defect ledger
(if present). All read-only for this skill except `docs/defects.md`, which
it owns.

## Scope boundaries

Does **not**: perform any stage's work; resolve escalations or amendments
(walks the user to the owning skill); override a stage's computed resolution;
write any pipeline artifact except the defect ledger; deploy (first-deploy's
job — but it will point there when the plan completes undeployed).

## What good looks like

- Announced position before every invocation; the user can always redirect.
- The loop's history is readable entirely from the artifacts and git — the
  orchestrator left no parallel state to reconcile.
- Every pause names its reason, its owner, and the resume path.
- Rework rounds are counted and bounded-in-practice: repeated cycles on one
  feature get surfaced as a signal, not silently ground through.
- Amendment chains ran in order — no FEAT minted without its FRs, no loop
  entry without its plan row.
- Defects trace: DEF row → failing test → fix commits → re-verification
  verdict.
