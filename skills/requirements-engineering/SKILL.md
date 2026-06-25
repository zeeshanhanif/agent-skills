---
name: requirements-engineering
description: >-
  Runs the requirements-engineering phase — the first SDLC step, upstream of
  design. Through a thorough, area-by-area interview it elicits and specifies
  the complete requirements for a new application, proactively enumerating the
  standard sub-requirements for each capability area (for authentication:
  sign-up, sign-in, verification, forgot/reset password, logout, session
  expiry, lockout) plus all non-functional requirements. Produces a structured
  SRS (ISO/IEC/IEEE 29148 lineage) in markdown and Word, a separate use-case
  document (detailed specs plus a use-case diagram) in markdown and Word, and a
  requirements traceability matrix. Use at the very start of a project, before
  architecture and UX, whenever a new system's requirements must be captured.
  Trigger on "gather requirements", "write an SRS", "requirements engineering",
  "specify the requirements", or any project kickoff before design begins.
  Saves progress incrementally so long sessions can resume.
---

# Requirements Engineering

This skill produces the **traditional, structured requirements specification**
that sits at the very front of the SDLC — upstream of architecture and UX. It is
deliberately a *detail phase*: its job is to capture **every** requirement
comprehensively, so that the documents it produces become the single source of
truth the design and planning skills downstream consume.

Two principles define it:

1. **Exhaustive enumeration, not transcription.** When a capability area comes
   up, the skill does not merely record what the user mentions. It proactively
   proposes the full set of standard sub-requirements for that area and has the
   user confirm, extend, or remove them. "User authentication" is never one line
   — it expands into sign-up, sign-in, email verification, forgot password, reset
   password, change password, logout, session expiry, account lockout, rate
   limiting, and so on. The `requirement-catalog.md` reference is the engine for
   this. The same discipline applies to non-functional requirements.

2. **Structured and traditional.** The output is a formal SRS in the
   ISO/IEC/IEEE 29148 (IEEE 830) lineage — numbered, uniquely-identified,
   testable requirements — not an agile backlog. Whatever the SRS calls a unit
   (function / feature) is fine; **epic and feature slicing is deliberately
   deferred to the downstream implementation-planning skill.** This skill owns
   the problem space, comprehensively.

Because a full requirements interview is long, the skill **checkpoints its work
incrementally** and can **resume** after an interruption — see Phase 0 and
`references/checkpointing.md`.

## Inputs

This is the first skill in the pipeline, so it has no upstream document to
consume — it elicits from the user. If the user has an existing brief, notes, or
partial requirements, ingest them first and treat them as raw input to confirm
and expand, not as finished requirements.

## Outputs

Five files (three logical documents). Working files are written to `docs/`.

1. `docs/srs.md` — the comprehensive SRS (markdown; the source of truth
   downstream skills read). Vision & scope, user characteristics, constraints,
   assumptions, glossary, **functional requirements**, and **non-functional
   requirements** all live here as structured sections.
2. `docs/srs.docx` — the same SRS as a formal Word document.
3. `docs/use-cases.md` — detailed textual use-case specifications plus a Mermaid
   use-case diagram (separate document, use-case-centric; no user stories).
4. `docs/use-cases.docx` — the use-case document as Word.
5. `docs/rtm.md` — the Requirements Traceability Matrix (markdown table).

Plus working files that are **not** deliverables: `docs/.requirements-progress.md`
(the progress tracker), and the incrementally-built `srs.md` / `use-cases.md`
drafts themselves (which become deliverables 1 and 3 once finalized).

## Workflow

### Phase 0 — Resume check (always first)

Before anything else, look for `docs/.requirements-progress.md` and an existing
`docs/srs.md`. Read `references/checkpointing.md` for the protocol. If they
exist, read them, summarize for the user what's already captured and what remains,
and continue from the first pending area. If they don't exist, create the
progress tracker and start fresh. **Never silently restart work that's already
been done.**

### Phase 1 — Frame and scope

Establish the foundation: the product's purpose and vision, business goals and
success metrics, the primary stakeholders, the user classes/personas, and the
overall scope (what's in, what's out, MVP boundary). Write these into the SRS's
introduction and overall-description sections immediately, and mark the area done
in the tracker. This section also seeds the user-characteristics content that
ux-foundations will later consume.

### Phase 2 — Functional requirements, area by area (the detail loop)

This is the core. Read `references/elicitation-guide.md` and
`references/requirement-catalog.md`. Work through the application **one capability
area at a time**. For each area:
- Propose the standard sub-requirements from the catalog as a checklist, in plain
  language, and ask the user to confirm / extend / remove.
- Specify each confirmed requirement formally: a unique ID (e.g.,
  `FR-AUTH-001`), a clear testable statement, priority, and any rules/validation.
- Write the finalized area straight into `docs/srs.md`, then update the progress
  tracker before moving to the next area.

Cover every area the application needs — not just the obvious ones. The catalog
lists commonly-forgotten areas (audit logging, account deletion, rate limiting,
admin/back-office) to prompt with.

### Phase 3 — Non-functional requirements

Read the NFR section of `references/requirement-catalog.md` (organized on the ISO
25010 quality model). Walk each relevant category — performance, scalability,
availability/reliability, security, usability/accessibility, compatibility,
maintainability, compliance/legal, localization, observability — and specify
**measurable** NFRs with unique IDs (e.g., `NFR-PERF-001`). Vague NFRs are not
acceptable; "fast" becomes a number. Write into the SRS and checkpoint.

### Phase 4 — External interface requirements

Capture the external interfaces the SRS lineage expects: user interfaces (at the
requirements level), hardware, software/third-party integrations, and
communication interfaces. Write into the SRS and checkpoint.

### Phase 5 — Use cases

Read `references/use-case-guide.md`. Derive use cases from the functional
requirements. For each use case, write a detailed textual specification (ID,
name, actors, preconditions, postconditions, main success scenario, alternate
flows, exception flows, and the FRs it traces to) into `docs/use-cases.md`, and
add the Mermaid use-case diagram(s) grouping actors and use cases by the system
boundary. Checkpoint as you complete each use case.

### Phase 6 — Traceability matrix

Read `references/rtm-guide.md` and build `docs/rtm.md`: a table linking every
requirement ID to its source and to the use case(s) that exercise it, with
placeholder columns for the downstream design and test artifacts. Because every
requirement was given a stable ID during specification, this is largely
assembly.

### Phase 7 — Generate Word documents

Only after the markdown sources are finalized, read
`references/docx-generation.md` and produce `docs/srs.docx` and
`docs/use-cases.docx` from the finalized markdown. These are generated last, so
they never need checkpointing — the markdown is the checkpointed source.

### Phase 8 — Deliver

Confirm all five files exist. Summarize the requirement counts (e.g., "47
functional across 8 areas, 19 non-functional, 12 use cases") and any open
questions or TBDs flagged during elicitation. Note that `docs/srs.md` is the
source of truth the architecture, UX, and planning skills will consume. Offer to
proceed to `software-architecture`.

## Scope boundaries

It deliberately does **not**:
- slice requirements into epics/vertical features — that's
  `implementation-planning`;
- make architectural or UX design decisions — those are the design-phase skills
  (which will *read* this SRS rather than re-eliciting NFRs and personas);
- write user stories — this is use-case-centric by design.

## What good looks like

- Every capability area is enumerated to its standard sub-requirements, not just
  what the user first mentioned.
- Every requirement has a unique, stable ID and is individually testable.
- NFRs are measurable, never vague.
- Work is checkpointed so a stop or network drop never loses captured areas.
- The five outputs are internally consistent and cross-referenced via the RTM.
