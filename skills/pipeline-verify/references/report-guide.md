# Report Guide

Phase 3–4: the dashboard computation and the report. The report is a
**derived view** — overwritten each run, regenerable, deletable; the
artifacts remain the only truth.

## Severities (assigned at the check, aggregated here)

- **Error** — a broken seam something downstream will trip on: dangling
  citation, RTM/SRS contradiction, unresolvable locator, FEAT without FRs,
  Test ref without acceptance. Errors mean: fix before relying on the
  documents (and especially before an orchestrated run).
- **Warning** — survivable but suspicious: orphan UCs/screens, partial-marker
  inconsistencies, rename drift, missing E2E tasks. Warnings mean: an owner
  should look; the pipeline runs meanwhile.
- **Info** — notable, no action implied: uncovered Should/Coulds, ADRs
  without requirement citations, checks unverifiable this run.

The verdict line: **clean** (zero errors, zero warnings), **clean with
warnings**, or **N errors** — never softened.

## The dashboard (computed, stored nowhere)

- **Per-FR lifecycle table**: FR → priority → Plan ref features → design
  state → verification state (fully / partially / pending, by the
  Plan-ref⨝Test-ref rule with partial markers) → open DEFs. This is the
  row-level truth the RTM's cells alone don't shout.
- **Per-feature stage table**: the same artifact-derived stages the
  orchestrator computes (planned / designed / ui-designed / developer-done /
  verified / blocked) — independently derived; a disagreement with what the
  orchestrator announces is itself a finding.
- **Counts**: features verified/total, Must FRs fully verified/total,
  screens designed/inventory, open escalations (from features' §8s and
  ui-design escalation entries), open defects.

## docs/pipeline-verify-report.md

```markdown
# Pipeline Verification Report

> Generated: <date-time> · Scope: full | <targeted> · Verdict: <verdict line>
> Derived view — regenerable; the pipeline artifacts are the truth.

## Verdict and counts
The verdict line; then the namespace table: defined / cited / resolved /
orphaned per namespace; checks run / skipped (with reasons).

## Findings
Grouped by owning skill, errors first within each group:
### requirements-engineering
- [ERROR] RTM row FR-ORD-011 priority "Should" contradicts SRS "Must" (srs.md §3.1.4)
### implementation-planning
- [WARNING] SCR-ADM-006 in no feature's touchpoints and no documented gap
…each finding: class, the IDs, both locations, one line of what's broken.
"None" sections stay present.

## Dashboard
The per-FR lifecycle table, the per-feature stage table, the counts.

## Skipped checks
Each skipped seam with its missing input.
```

## Routing map (finding → owner)

RTM rows/fields, SRS/UC definitions → **requirements-engineering** ·
ADR citations, architecture Testing entry (including a coverage stance the
repo can't satisfy) → **software-architecture** ·
CI gate, harness realization, scaffold-notes drift → **project-scaffolding** ·
inventory, SCR, design.md/tokens.json authority → **ux-foundations** ·
plan touchpoints, FEAT set, Plan ref, coverage → **implementation-planning** ·
technical-design/tasks structure, Design ref appends → **detailed-design** ·
manifest, ui-design.md anchors, screen escalations → **ui-design** ·
task execution states, WIP hygiene → **feature-implementation** ·
Test ref appends, report verdicts → **acceptance-verification** ·
defect ledger → **sdlc-orchestrator**.

Route by the *artifact that's wrong*, not the skill that most recently ran —
an amendment often breaks a seam in a document its own checks passed.

## Chat summary

Lead with the verdict line and the error count; the top findings (up to a
handful) with owners; the headline dashboard numbers; and the pointer to the
report file. Full detail lives in the report — the chat summary is for
deciding what to do next, not for re-reading the sweep.
