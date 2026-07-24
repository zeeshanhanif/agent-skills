# Routing Guide (loop driving)

Mechanics for the default job: compute the global position, invoke the right
stage, route the outcome. Nothing here does a stage's work.

## Stage detection (the computed position)

For each feature in the plan's build sequence, in order, its stage is read
off the artifacts:

| Artifacts present in `docs/features/FEAT-NNN-<slug>/` | Stage |
| :--- | :--- |
| no folder, or folder without technical-design.md | **planned** |
| technical-design.md (+ tasks.md), no ui-design.md | **designed** |
| ui-design.md present, tasks.md has unchecked boxes | **ui-designed** |
| tasks.md fully checked, no accepted acceptance-report.md | **developer-done** |
| acceptance-report.md, latest verdict Accepted | **verified** |

Special reads: a tasks.md with a WIP failure note → **blocked** (a stop
state, not a stage); a report whose latest verdict is Rework or Design
defect → routes per the verdict (below); features whose screens list is
empty skip ui-design (the stage table collapses accordingly — detailed-
design's plan entry is the authority on whether screens exist).

**The loop front** = the earliest feature in sequence not yet verified. The
next invocation = that feature's next stage skill. Announce both before
acting; explicit user naming of a different feature is honored as the
off-sequence path (same rule the stage skills follow).

## Invocation contract

Each stage is invoked as itself — its own skill, its own resolution
(which will land on the same feature the orchestrator computed; if it
announces differently, that's a signal to stop and reconcile, not to
override). The orchestrator passes nothing the skill wouldn't compute — at
most the explicit feature name when driving off-sequence. Between stages,
trust disk: read the artifacts the stage produced, not a summary of them.

## Run scopes

- **One stage**: invoke, read the outcome, report, stop.
- **One feature cycle** (default): drive the front feature stage → stage
  until verified or a stop condition. Rework rounds count toward the cycle
  (below).
- **Run-until-blocked**: feature cycles back-to-back until a pause-class
  stop, plan completion, or user stop. Announce each feature's start; keep
  per-feature summaries terse (the artifacts carry the detail).

## Outcome routing

- **Stage completed normally** → next stage (or next feature).
- **Rework verdict** → re-invoke feature-implementation with the report as
  the work order, then re-invoke acceptance-verification. **Bound the
  rounds**: after 2 rework cycles on one feature, pause and surface — the
  auditor and the builder disagreeing repeatedly is a design-quality signal,
  not a grind-through situation.
- **Design-defect verdict** → pause-class. Surface the report's defect
  findings toward detailed-design's amendment path (interactive); after the
  amendment lands, the feature re-enters at the stage the amendment
  invalidated (a changed criterion → re-verify; a changed contract → tasks
  likely changed → detailed-design will have said).
- **Escalation filed mid-stage** (schema→architecture, screen→design-system,
  divergence→detailed-design) → pause-class. Name the escalation, its owning
  skill, and what resumption looks like. Never resolve it inline.
- **Blocked task** → pause-class. Relay the failure note verbatim (it was
  written to be picked up cold). Repeated blocking on one feature → relay
  feature-implementation's meta-signal: recommend the feature return to
  detailed-design.
- **Plan complete** → all features verified: summarize, then point at the
  edges — first-deploy if never run; the standing recommendation to run the
  full suites once more post-final-feature is already covered by the last
  acceptance run.

## Pause discipline

Every pause message contains: what stopped (the condition), where (feature +
stage + artifact), who owns the next move (which skill or the user), and the
resume instruction ("after the amendment, say 'continue the loop' — the
position recomputes"). A pause with those four elements is a feature of the
system, not a failure of it.

## Session and context notes

The loop is fresh-context safe end to end — that was every stage skill's
discipline 2 — so long runs may be split across sessions freely; re-entry
recomputes. When driving many stages in one session, prefer starting each
stage from its documents rather than from the session's accumulated context
(the stage skills instruct this themselves; don't undermine it by
summarizing ahead of them).
