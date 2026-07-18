# Changelog

All notable changes to this collection of Agent Skills are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project does not yet publish tagged releases, so entries are grouped by date.

## [Unreleased]

### Changed
- **README** and **CLAUDE.md** updated for the seven-skill pipeline (linear
  requirements-to-skeleton pass plus the two-skill per-feature construction
  loop), **`ui-design`** as detailed-design's presentation sibling,
  **`FEAT-NNN` IDs** as the construction-loop join key, and extended RTM
  Design-ref write-back from both loop skills.

## [2026-07-17]

### Added
- **`ui-design`** skill — the **presentation half** of the per-feature
  construction loop, run sequentially after `detailed-design`. Three modes:
  *anchor* (validate the design system composes after ux-foundations),
  *per-feature* (design a slice's SCR screens against its
  `technical-design.md` contracts), and *re-verification* (after a
  `design.md` amendment). Resolves strategy **per screen** — register
  existing tool designs (Figma, Claude Design, etc. via MCP), generate, or
  code-native spec — and emits a uniform **`docs/design-manifest.json`**
  (SCR-keyed, single-writer registry) plus `ui-design.md` or
  `anchor-screens.md`. Screens conform to `design.md` + `tokens.json` or
  escalate to a ux-foundations amendment; appends Design ref to the RTM.
  Reference guides: `strategy-guide.md`, `design-tool-integrations.md`,
  `manifest-guide.md`, `document-templates.md`, `verification.md`.
- **`design_provenance` block** in `docs/design.md` — a structured JSON
  object written in **every** ux-foundations source mode (`mode` and
  `fidelity` always; `source` with tool + locator only in tool mode).
  Drives ui-design's anchor recommendation gradient and tool-sourced screen
  lookup (supersedes the earlier mode-4-only `design_source` block).

### Changed
- **`detailed-design`** resolves the next feature **deterministically** from
  plan build order (first slice without a design folder — no menu); announces
  the resolution so the user can redirect before work starts. Matches
  ui-design's per-feature selection rule (first folder with
  `technical-design.md` but no `ui-design.md`).

## [2026-07-13]

### Changed
- **`implementation-planning`** mints a stable **`FEAT-NNN` ID** for every
  feature slice (sequential in definition order, never renumbered/recycled,
  tombstoned on removal — same discipline as FR/SCR IDs). RTM Plan-ref
  write-back now keys on the FEAT ID; the ID is the join key for
  detailed-design, ui-design, and the RTM.

## [2026-07-12]

### Changed
- **README** and **CLAUDE.md** updated for the six-skill pipeline (linear
  requirements-to-skeleton pass plus the per-feature construction loop),
  `detailed-design` as the first loop skill, and extended RTM Design-ref
  write-back.

### Added
- **`detailed-design`** skill — the first **per-feature loop skill**, run once
  for each vertical slice as it reaches the front of the plan. Reads the plan,
  SRS, use cases, architecture, and the **live codebase** (mandatory), then
  produces per-feature `technical-design.md` (API contracts, schema migrations,
  component design, acceptance criteria) and `tasks.md` under
  `docs/features/FEAT-NNN-<slug>/`. Designs the *how* with the *what* fixed by
  FR/UC IDs; escalates new entities to an architecture amendment; appends
  Design ref to the RTM. Hands contracts to ui-design and tasks to
  implementation.

## [2026-07-11]

### Added
- **`project-scaffolding`** skill — the first skill whose output is a running
  system, not a document. Reads the architecture and implementation plan (plus
  ux-foundations/design/tokens for the UI shell), runs each ecosystem's official
  project generator per container, wires the walking skeleton end-to-end, stands
  up engineering foundations deploy-ready but not deployed, and verifies
  empirically by building and running the skeleton test. Checkpoints progress
  and resumes safely — never regenerates over a partial scaffold.

### Changed
- **README** and **CLAUDE.md** updated for the five-skill pipeline
  (requirements → architecture → ux-foundations → planning → scaffolding),
  the RTM multi-writer contract, implementation-planning's full-pipeline
  consumption, stable SCR IDs, and project-scaffolding.
- **`implementation-planning`** now consumes the **full pipeline** — SRS, use
  cases, architecture, and ux-foundations (each degrading independently):
  - Every slice traces **FR IDs** implemented, **UC IDs** realized, and **SCR
    IDs** touched; tombstoned requirements and screens are skipped.
  - A **Must-requirement coverage check** ensures every Must-priority FR and
    inventory screen is sliced, placed in foundations, or explicitly excluded.
  - SRS **MoSCoW priorities** break sequencing ties after dependency/risk
    ordering.
  - A mechanical **verification** pass (`verification.md`) before delivery.
  - The first-slice spec names downstream handoffs to **ui-design** (screens by
    SCR ID) and **detailed-design** (contracts/data).

### Added
- **RTM multi-writer column ownership** — the traceability matrix is now a
  living ledger each phase fills at delivery:
  - `requirements-engineering` owns rows and requirement columns; initializes
    Design/Plan/Test refs as `_TBD_`.
  - `software-architecture` writes **Design ref** (ADR refs per a layer rule:
    NFR rows → ADR; FR rows → ADR only when genuinely cited).
  - `implementation-planning` writes **Plan ref** (the scheduling slice).
  - Every writer **appends, never overwrites**; skips silently when no RTM
    exists (standalone runs).

## [2026-07-10]

### Changed
- **`ux-foundations`** substantially reworked along three axes:
  - **Inputs** — now consumes the SRS (`docs/srs.md`) and use-case document
    (`docs/use-cases.md`) alongside the architecture: personas come from SRS
    §2.3 user classes, the accessibility bar from §3.3 NFRs (non-negotiable,
    overrides ingested designs), and brand/compliance from §2.5. ID citations
    are source-gated (cite SRS/UC IDs only when those documents exist, never
    fabricated), with graceful standalone fallback.
  - **Design source** — Phase 1 detects candidate design sources and always asks
    the user which of four modes to use: research a new direction, extract from
    reference images (`docs/design-refs/`), ingest an existing design file, or
    connect a design tool via MCP (e.g., Figma).
  - **Outputs** — split into three files with a strict authority split:
    `docs/ux-foundations.md` (plan-time; references the design system, never
    restates values), `docs/design.md` (render-time, agent-ready design system),
    and `docs/tokens.json` (canonical tokens in W3C DTCG format).
- **`ux-foundations`** screen inventory now assigns every screen a stable
  **`SCR-<CODE>-<NNN>` ID** per surface (same never-renumber/tombstone rules as
  requirement IDs), so downstream slices and ui-design reference screens
  precisely.

### Added
- `ux-foundations` reference guides: `source-modes.md`, `design-md-guide.md`,
  and `design-tool-integrations.md`.
- `CHANGELOG.md` — project changelog (Keep a Changelog format).

## [2026-07-01]

### Changed
- **`software-architecture`** now consumes the upstream requirements artifacts,
  connecting the pipeline hand-off through `docs/srs.md`:
  - Designs from the SRS as the primary source — ingests requirements/NFRs,
    plays back the drivers, and interviews only for the architecture-specific
    gaps; falls back to the full interview when no SRS exists (still usable
    standalone).
  - Also ingests `docs/use-cases.md` (secondary) to inform the runtime view and
    resilience/security decisions; the RTM is not consumed as input.
  - Adds two-way requirement → decision traceability via a source-gated
    "Requirements addressed" field on ADRs (cite SRS/UC IDs only when those
    documents exist, prose otherwise — never fabricate an ID).
- Documented the pipeline-vs-standalone usage modes in the README.

## [2026-06-30]

### Removed
- **`requirements-engineering`** no longer emits Word (`.docx`) output. Markdown
  is the sole deliverable (SRS, use-case document, RTM), dropping the
  Pandoc/`python-docx` dependency; the `docx-generation.md` reference was
  removed.

## [2026-06-26]

### Added
- **`requirements-engineering`** amendment support: a finalized SRS can be
  changed (add/modify/remove) with stable, never-recycled requirement IDs.
  Phase 0 became three-way mode detection (fresh / resume / amendment) and a new
  Phase A defines the amendment protocol (tombstoned removals, version +
  revision-history logging, propagation to use cases and the RTM, cross-skill
  impact note).

## [2026-06-25]

### Added
- **`requirements-engineering`** skill — the first SDLC step. Through an
  exhaustive, area-by-area interview it specifies the complete requirements and
  produces a structured SRS (ISO/IEC/IEEE 29148 lineage), a use-case document,
  and a requirements traceability matrix. Checkpoints incrementally so long
  interview sessions can resume.

## [2026-06-22]

### Added
- **`implementation-planning`** skill — reads the architecture and
  UX-foundations documents and produces a sequenced build plan: epics and thin
  vertical feature slices, the walking skeleton, a dependency/risk-ordered
  sequence, the first-slice spec, and an engineering-foundations checklist.

## [2026-06-19]

### Added
- **`ux-foundations`** skill — the "architecture of the UI." Reads the
  architecture document and produces a shared design core plus a per-surface
  profile (IA, navigation, key flows, screen inventory) for each UI surface.

## [2026-06-17]

### Added
- Initial repository and the **`software-architecture`** skill — interviews the
  user about a new application and produces a right-sized architecture document
  with C4 diagrams (Mermaid) and Architecture Decision Records.
- `skills` CLI installability: skills live under `skills/<name>/`, installable
  via `npx skills add`, with manual Claude Code install documented.
