# Verification (Self-Check Before Delivery)

Run this after the plan is drafted and before delivering (Phase 8). This is the
skill checking **its own contract** — mechanical conformance of its outputs —
not the pipeline-wide cross-document verification (a separate, later concern;
when that exists, this phase becomes its per-skill layer).

Everything here is checkable, not judgment. Fix failures before delivery;
anything genuinely unfixable is flagged in the delivery summary, never silently
shipped.

## 1. ID integrity

- Every FR, UC, and SCR ID cited anywhere in the plan **exists in its source
  document**. Grep the plan's IDs; diff against the IDs defined in srs.md,
  use-cases.md, and ux-foundations.md. No dangling references, no invented IDs.
- **FEAT IDs are sound**: every feature has exactly one `FEAT-NNN` ID; no
  duplicates or gaps filled by reuse; the build sequence and dependency graph
  reference features by ID; RTM Plan ref entries use the ID.
- **No tombstoned item spawned work**: no slice traces to an FR or screen whose
  status is `Removed`/`Deprecated` in its source.
- Citations respect source-gating: if a source document is absent, the plan
  uses prose there, not IDs.

## 2. Coverage completeness

- Every non-tombstoned **Must-priority FR** appears in exactly one of: a
  slice's "FRs implemented", the engineering foundations (with its ID), or the
  Risks section as an explicit exclusion/open gap. None missing from all three.
- Every non-tombstoned **SCR ID** in the inventory is touched by at least one
  slice, or listed as an explicit open gap.
- The escalation ladder ran: open gaps in Risks show they were surfaced (not
  discovered post-hoc).

## 3. Structural soundness

- The **dependency graph** is acyclic, every node is a defined slice or the
  walking skeleton, and every edge's endpoints exist.
- The **first slice** named in section 5 is at the front of the build sequence
  (immediately after the skeleton), and appears in the dependency graph.
- The **walking skeleton** states what's real, what's stubbed, and its
  done-when condition.
- All **Mermaid blocks parse** (mentally trace the syntax rules from
  diagram-guide.md: quoted labels, no bare `end`; where a renderer is
  available, actually parse them).
- Every epic maps to an architecture building block; every slice belongs to an
  epic.

## 4. Handoff completeness

The first-slice spec contains, explicitly:
- the user flow it exercises and why chosen;
- acceptance criteria;
- **screens by SCR ID, named as the input to ui-design**;
- **endpoints/data needs, named as the input to detailed-design**.

Both downstream targets are named. A first slice that doesn't say who consumes
it isn't a handoff.

## 5. Document conformance

- All essential sections of the document template are present and non-empty.
- The plan states which upstream documents were found/missing (fidelity note).
- Priorities used in sequencing match the SRS's (no silently reassigned
  MoSCoW).
- **RTM write-back done** (when `docs/rtm.md` exists): every FR a slice
  implements or the foundations absorbed has its Plan ref filled; no other
  column was touched.

## Reporting

Close the verify phase with a one-line result in the delivery summary: either
"verification clean" or the list of flagged items ("2 open coverage gaps
[FR-RPT-004, SCR-ADM-011] documented in Risks"). The user should never discover
a gap the skill knew about.
