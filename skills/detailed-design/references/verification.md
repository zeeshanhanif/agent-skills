# Verification (Self-Check Before Delivery)

Phase 5's contract: mechanical conformance of this feature's design and tasks,
checked before delivery. Fix failures; flag the unfixable in the summary —
never ship silently defective design. (Per-skill layer of the pipeline's
verification approach; the future pipeline-wide checker composes with this.)

## 1. Requirement coverage

- Every FR the plan assigned to this feature is covered by at least one
  acceptance criterion **and** at least one task. None missing from either.
- Every UC flow the feature realizes (main + consequential alternates/exceptions)
  maps to contract behavior (§3) and acceptance criteria (§6).
- Binding NFRs appear as concrete contract/design properties, cited by ID.
- **E2E obligation honored**: if this feature's UCs intersect the
  architecture's named critical flows, tasks.md contains the E2E-extension
  task (flow-aware, before final verification); if the architecture is silent
  on testing, the skip is legitimate.
- Nothing in the design lacks a requirement or code-reality driver behind it
  (gold-plating check).

## 2. ID integrity

- Every cited FR/UC/SCR/NFR/ADR ID exists in its source document; no
  tombstoned requirement or screen is served. Source-gating respected where
  documents are absent.
- The feature's folder name matches its plan-sequence position and name.

## 3. Schema conformance

- Every schema change maps to an entity the architecture's conceptual model
  owns. Any new entity / boundary / ownership change has a filed escalation in
  §8 — not a local workaround.
- Migrations are expressed against the *current* schema state (the migration
  history read in Phase 2), not against the skeleton or an assumed state.

## 4. Codebase consistency

- Contracts follow the surveyed conventions (envelope, naming, errors,
  pagination). Any deliberate divergence is a §7 decision with a driver.
- No collision: new endpoints/tables don't conflict with existing ones or
  with prior feature designs in `docs/features/`.
- Breaking changes to prior features' contracts/schema are recorded as decisions
  with a compat/migration story.

## 5. Tasks soundness

- Order respects dependencies (schema → domain → contract → wiring → UI
  integration → E2E when owed → verify); every task has a checkable done-when; every task
  points at its design section; the final task is acceptance verification.
- Executing tasks.md top-to-bottom would produce the design — no design
  element lacks a task, no task lacks a design basis.

## 6. Write-backs and hygiene

- RTM Design ref appended (when rtm.md exists) for every FR the feature
  implements — append-only, Design ref column only.
- Any Mermaid in technical-design.md parses (quoted labels, no bare `end`).
- §8 lists every escalation/open item raised during design — nothing raised in
  conversation is missing from the document.

## Reporting

One line in the delivery summary: "verification clean" or the flagged list
("escalation pending: new entity Invoice — architecture amendment proposed").
The user never discovers a gap the skill knew about.
