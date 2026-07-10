# Changelog

All notable changes to this collection of Agent Skills are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project does not yet publish tagged releases, so entries are grouped by date.

## [Unreleased]

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

### Added
- `ux-foundations` reference guides: `source-modes.md`, `design-md-guide.md`,
  and `design-tool-integrations.md`.

## [2026-07-01]

### Changed
- **`software-architecture`** now consumes the upstream requirements artifacts,
  connecting the pipeline hand-off through `docs/srs.md`:
  - Designs from the SRS as the primary source — ingests requirements/NFRs,
    plays back the drivers, and interviews only for the architecture-specific
    gaps; falls back to the full interview when no SRS exists (still usable
    standalone).
  - Also ingests `docs/use-cases.md` (secondary) to inform the runtime view and
    resilience/security decisions; the RTM is intentionally not consumed.
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
