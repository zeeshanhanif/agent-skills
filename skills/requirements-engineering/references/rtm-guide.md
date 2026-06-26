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
> Design and Test columns are placeholders, filled in downstream phases.

| Req ID | Requirement (short) | Priority | Status | Source | Use case(s) | Design ref | Test ref |
| :----- | :------------------ | :------- | :----- | :----- | :---------- | :--------- | :------- |
| FR-AUTH-001 | Register with email/password | Must | Active | Stakeholder interview | UC-001 | _TBD_ | _TBD_ |
| FR-AUTH-002 | Verification email on registration | Must | Active | Stakeholder interview | UC-001, UC-002 | _TBD_ | _TBD_ |
| NFR-SEC-001 | Encryption in transit & at rest | Must | Active | Compliance (GDPR) | — | _TBD_ | _TBD_ |
| … | … | … | … | … | … | … | … |
```

## Rules

- **Every requirement ID appears exactly once** as a row. If a functional
  requirement has no use case, flag it — either it needs one, or it's a system
  requirement (many NFRs legitimately have no use case; mark "—").
- **Status** is `Active` by default; amendments set removed requirements to
  `Removed`/`Deprecated` here too (the row stays, matching the SRS) so
  traceability survives removal — see `change-management.md`.
- **Keep the short description short** — the SRS holds the full statement; the RTM
  is an index, not a copy.
- **Design and Test columns stay as `_TBD_`** at this stage. They exist so the
  architecture/detailed-design and testing phases can fill them, completing the
  trace from requirement → design → test.
- **Source** records where the requirement came from (a stakeholder, a regulation,
  a business goal) — useful when a requirement is later questioned.

## Generation

Build the RTM **after** the SRS and use cases are finalized, by walking the
requirement IDs in `docs/srs.md` and cross-referencing the **Traces to** fields in
`docs/use-cases.md`. Keep it markdown (per the chosen output format) so it stays
diffable and lives beside the other source-of-truth files.
