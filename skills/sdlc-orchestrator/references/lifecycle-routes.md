# Lifecycle Routes (feature cycle and maintenance)

The two lifecycle events beyond the loop: a **change request** (new/changed
capability on an existing system) and a **defect** (verified behavior now
broken). Both are routing recipes — every step belongs to its owning skill;
the orchestrator's job is order and completeness, not substance.

## Feature-cycle route (change requests)

"Add X" / "change how Y works" after the system exists. The chain, in order,
skippable only where an impact note says so:

1. **Requirements amendment** — requirements-engineering, amendment mode:
   the new/changed FRs and UCs get IDs (or revisions), the SRS/use-cases/RTM
   rows update, and the amendment's **cross-skill impact notes** are written.
   Nothing downstream moves before this: a FEAT without FRs would break the
   inheritance the whole loop runs on.
2. **Impact pass on architecture and ux-foundations** — read the impact
   notes; amend **only what they implicate**: architecture when the change
   carries NFR pressure, a new entity, or boundary consequences (its
   amendment/ADR-supersession path); ux-foundations when new screens or
   flows are needed (SCR minting, inventory update — and ui-design's
   re-verification mode afterward if design.md itself changed). No implicated
   impact → explicitly record "no architecture/ux impact" rather than
   silently skipping.
3. **Plan amendment** — implementation-planning, amendment mode: mint the
   FEAT (next free number), full touchpoints (the new FR/UC/SCR IDs),
   insert into the build sequence by the same dependency-and-risk logic,
   RTM Plan ref append, coverage check over the delta.
4. **Enter the loop** — the new feature is now ordinary planned work; the
   loop's computed front will reach it per the amended sequence (or the user
   drives it explicitly off-sequence, announced as such).

The orchestrator walks the user through 1–3 (all interactive by design),
verifying each step's artifact landed before proceeding — the chain's value
is that step N+1 always finds step N's IDs already minted.

## Maintenance route (defects)

A bug report: behavior observed violating a requirement that
acceptance-verification already accepted.

1. **Locate the owner.** From the symptom to the FR (the SRS statement it
   violates), from the FR's RTM row to the feature(s) (Plan ref). Ambiguous
   symptom → reproduce first, then locate. If the behavior matches the
   requirement and the *requirement* is what's wrong — that's not a defect,
   it's a change request: reroute to the feature-cycle route.
2. **Record the defect** — append a row to `docs/defects.md` (this ledger is
   the orchestrator's one owned artifact; create on first use):

   | DEF | Reported | FR / Feature | Symptom (one line) | Fixed by | Re-verified |
   | :-- | :------- | :----------- | :----------------- | :------- | :---------- |
   | DEF-001 | 2026-07-18 | FR-ORD-010 / FEAT-009 | duplicate order on retry | _open_ | _open_ |

   DEF IDs: sequential, immutable, never recycled — the pipeline's standard
   ID discipline.
3. **Failing test first.** A test demonstrating the bug, added to the owning
   feature's suite, red against current code — this is legitimate new test
   work (the bug proves the suite had a gap), asserting what the FR/criterion
   actually requires. No fix lands before its failing test exists.
4. **Scoped fix under implementation disciplines.** feature-implementation's
   rules apply verbatim, scoped to the fix: bounded attempts (default 3),
   anti-fake-green (the new test is corrected only *toward the
   requirement*), this-fix-only scope, commit referencing the defect
   (`DEF-001 fix: <summary>`). Diagnosis revealing the *design* was wrong →
   stop; reroute to the design-defect path (detailed-design amendment) — a
   bug report is a symptom, not a classification.
5. **Re-verify.** Re-run acceptance-verification on the affected feature — a
   normal run; the report gains a dated verdict section; on acceptance the
   defect ledger's `Fixed by` (commit) and `Re-verified` (report ref)
   columns fill. The RTM needs no edit: the row's Test ref already points at
   the report, whose new dated section is the record — and computed
   verification stays truthful throughout (a known-open defect on a verified
   FR is exactly what the ledger surfaces that the RTM alone wouldn't).

## What both routes protect

The inheritance chain. Every piece of work in this pipeline traces to IDs
minted upstream of it — that's what keeps N features coherent. Both routes
exist so that lifecycle events *re-enter through the top of their chain*
(requirements for changes, the failing test + owning feature for defects)
instead of being patched in at the bottom where the trace can't see them.
