# Requirements Traceability Matrix (RTM) Guide

The RTM (`docs/rtm.md`) links every requirement to its origin and to the
artifacts that realize and verify it. At the requirements stage it connects each
requirement ID to its **source** and to the **use case(s)** that exercise it, and
holds **placeholder columns** for the downstream design and test artifacts that
later phases fill in. Because every requirement received a stable ID during
specification, building the RTM is mostly assembly, not new thinking.

## Purpose

Traceability answers, for any requirement: where did it come from, where is it
realized, and how is it verified? It catches orphan requirements (no use case,
no test) and orphan work (design/tests with no requirement behind them). It's a
hallmark of the traditional, structured requirements process.

## Format

A markdown table, one row per requirement (functional and non-functional):

```markdown
# Requirements Traceability Matrix

> Source: docs/srs.md, docs/use-cases.md · Last updated: <date>
> Design, Plan, and Test columns are filled by downstream phases as they
> produce their artifacts (see Column ownership below).

| Req ID | Requirement (short) | Priority | Status | Source | Use case(s) | Design ref | Plan ref | Test ref |
| :----- | :------------------ | :------- | :----- | :----- | :---------- | :--------- | :------- | :------- |
| FR-AUTH-001 | Register with email/password | Must | Active | Stakeholder interview | UC-001 | _TBD_ | _TBD_ | _TBD_ |
| FR-AUTH-002 | Verification email on registration | Must | Active | Stakeholder interview | UC-001, UC-002 | _TBD_ | _TBD_ | _TBD_ |
| NFR-SEC-001 | Encryption in transit & at rest | Must | Active | Compliance (GDPR) | — | _TBD_ | _TBD_ | _TBD_ |
| … | … | … | … | … | … | … | … | … |
```

## Column ownership (the multi-writer contract)

The RTM is a living ledger with **exclusive column ownership** — each phase
writes only its own column(s), at the moment its artifact is finalized:

- **Rows and all requirement columns** (Req ID, description, Priority, Status,
  Source, Use case(s)) — owned by **requirements-engineering** alone (creation,
  amendment, tombstoning). No downstream skill edits these.
- **Design ref** — written by **software-architecture** and, later, the
  per-slice **detailed-design / ui-design** steps, per the layer rule:
  - *NFR rows*: the ADR(s) addressing them are usually the right and final
    design ref — NFRs are realized structurally (e.g., `ADR-003`).
  - *FR rows*: architecture writes an ADR ref **only** when that ADR's
    "Requirements addressed" field cites the FR (a genuine design response, not
    a mention). The concrete per-slice design refs (contracts, schemas, screen
    designs) are appended later by detailed-design/ui-design. A row can
    legitimately hold both layers: `ADR-002; slice-3-design.md §2`.
  - An FR row whose Design ref holds only an ADR (or `_TBD_`) is *not yet
    concretely designed* — useful pending-work signal, not an error.
- **Plan ref** — written by **implementation-planning**: the slice (and epic)
  that schedules each requirement (e.g., `Slice: Sign-in`). Scheduling is not
  design; that's why this is its own column.
- **Test ref** — written by the future **testing** phase.

Write rules for every downstream writer: **append, never overwrite** another
entry (comma/semicolon-append into the cell); write at delivery time of your
own phase; if `docs/rtm.md` doesn't exist (standalone run), skip silently.
Tombstoned rows keep their downstream refs visible — those dangling refs are
exactly what amendment impact notes surface.

## Rules

- **Every requirement ID appears exactly once** as a row. If a functional
  requirement has no use case, flag it — either it needs one, or it's a system
  requirement (many NFRs legitimately have no use case; mark "—").
- **Status** is `Active` by default; amendments set removed requirements to
  `Removed`/`Deprecated` here too (the row stays, matching the SRS) so
  traceability survives removal — see `change-management.md`.
- **Keep the short description short** — the SRS holds the full statement; the RTM
  is an index, not a copy.
- **Design, Plan, and Test columns are initialized as `_TBD_`** by this skill
  and filled by their downstream owners per the Column ownership contract above
  — architecture and detailed-design/ui-design (Design ref), planning (Plan
  ref), testing (Test ref) — completing the trace from requirement → plan →
  design → test.
- **Source** records where the requirement came from (a stakeholder, a regulation,
  a business goal) — useful when a requirement is later questioned.

## Generation

Build the RTM **after** the SRS and use cases are finalized, by walking the
requirement IDs in `docs/srs.md` and cross-referencing the **Traces to** fields in
`docs/use-cases.md`. Keep it markdown (per the chosen output format) so it stays
diffable and lives beside the other source-of-truth files.
