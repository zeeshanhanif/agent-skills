# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of **Agent Skills** following the open `SKILL.md` standard (see https://agentskills.io). There is no application code, build system, test suite, or linter — the deliverable is prompt/instruction content that Claude (or another agent) loads at runtime. "Correctness" here means the instructions are clear, well-scoped, and trigger reliably, not that anything compiles.

Each skill lives under `skills/` as its own directory containing a `SKILL.md` and (optionally) a `references/` folder of supporting guides. There are currently four skills, designed to run back-to-back as a greenfield requirements-to-build pipeline:
- `skills/requirements-engineering/` — the first SDLC step, upstream of design. Elicits the complete requirements and produces three **markdown** files: the SRS (`docs/srs.md`), a use-case document (`docs/use-cases.md`), and a traceability matrix (`docs/rtm.md`). It has **no upstream document** — it elicits from the user. It also **amends** a finalized SRS (add/modify/remove with stable, never-recycled IDs), so it owns the spec across its whole lifecycle, not just first authoring.
- `skills/software-architecture/` — **consumes the SRS (primary) and use-case document (secondary)** — defaults `docs/srs.md` + `docs/use-cases.md` — and produces the architecture document (`docs/architecture.md`). Designs against the SRS's requirements/NFRs (the *what*), uses the use cases to inform the runtime view and resilience/security decisions, and interviews only for the gaps; falls back gracefully to a full interview when no SRS exists, so it still works standalone. (It ignores `rtm.md` — that just re-indexes the other two.)
- `skills/ux-foundations/` — the "architecture of the UI." It **consumes the SRS + architecture (+ use cases)** — defaults `docs/srs.md` + `docs/architecture.md` (+ `docs/use-cases.md`) — and acquires the visual direction through one of **four user-chosen source modes** (research / extract from images / ingest a design file / connect a design tool like Figma via MCP). It produces **three outputs**: `docs/ux-foundations.md` (plan-time: personas, IA, navigation, flows, screen inventory), `docs/design.md` (render-time, agent-ready design system), and `docs/tokens.json` (canonical W3C DTCG tokens).
- `skills/implementation-planning/` — the bridge from design to construction. It **consumes both** prior design documents (defaults `docs/architecture.md` + `docs/ux-foundations.md`) and produces an executable build plan (`docs/implementation-plan.md`).

The hand-offs are real couplings, not loose suggestions:
- `requirements-engineering` emits **two primary documents**, not one: `docs/srs.md` (the requirement source of truth — NFRs, personas/user classes, constraints) **and** `docs/use-cases.md` (detailed use-case specs + a Mermaid use-case diagram, traced back to the FRs), plus the `docs/rtm.md` traceability matrix linking them. The SRS is the requirement spine; the use-case document carries the actor/flow detail that maps naturally onto ux-foundations' key flows and implementation-planning's vertical slices.
- `software-architecture` now **ingests both `docs/srs.md` and `docs/use-cases.md`**: it designs against the SRS requirements/NFRs, draws its runtime scenarios from the significant use cases, and mines the use cases' exception/alternate flows for resilience needs and their actors for trust boundaries. Traceability is **source-gated** — it cites SRS requirement IDs and use-case IDs *only when those documents exist*, and states drivers in plain prose (never a fabricated ID) when they don't.
- `ux-foundations` reads the **SRS + architecture + use cases**: surfaces and actors from the architecture's containers/context, **personas from SRS §2.3** (don't re-elicit), the **accessibility bar from the SRS usability/accessibility NFRs** (non-negotiable — it overrides any ingested design), and key flows citing the UC IDs they realize. Same **source-gated** traceability rule as the architecture skill.
- **Remaining gap:** `implementation-planning` is now the only skill that doesn't read `srs.md`/`use-cases.md` — it still defaults to its own upstream design docs (`architecture.md` + `ux-foundations.md`). Wiring it to also consume the requirements/use cases is the open follow-up; keep it in mind when editing it. (`software-architecture` and `ux-foundations` both consume the requirements now.)
- `implementation-planning` slices vertically by joining the architecture's building blocks/endpoints with the ux-foundations' per-surface **screen inventory** — it needs both to form a slice, which is why `ux-foundations` frames its screen inventory as that hand-off.

So changes to what an upstream skill emits (requirement IDs/vocabulary, surface naming, container vocabulary, the screen-inventory shape) can ripple downstream into how the next skill ingests it. Keep the vocabulary consistent across the four when editing any one.

## Skill anatomy and conventions

The `skills/` directory location matters: it's the flat layout (`skills/<name>/SKILL.md`) the [`skills` CLI](https://www.skills.sh) discovers, which is what makes `npx skills add <repo> --skill <name>` work. Keep skills under `skills/` — moving one to the repo root breaks installability.

A skill directory follows this shape:

```
skills/<skill-name>/
├── SKILL.md            # required: frontmatter + the workflow the agent follows
└── references/         # optional: deep-dive guides SKILL.md tells the agent to read on demand
```

- **`SKILL.md` frontmatter** has two keys: `name` (must match the directory name) and `description`. The `description` is the *only* thing the agent sees when deciding whether to invoke the skill, so it carries the trigger phrases and the "use this when…" intent. Treat it as load-bearing, not boilerplate — edits here change when the skill fires.
- **Progressive disclosure is the core pattern.** `SKILL.md` stays a lean workflow (phases, rules) and explicitly instructs the agent to *read a specific `references/*.md` file* at the moment it's needed, rather than inlining all the detail. When adding depth, add or extend a reference file and point to it from the relevant phase — don't bloat `SKILL.md`.
- **The README's "What's inside" tree and Skills table must stay in sync** with the actual files. Adding/removing a reference file or a skill means updating `README.md`.

## How people install these skills

Two supported paths, both documented in `README.md` — keep both working when you change layout:
- **`skills` CLI (primary):** `npx skills add https://github.com/zeeshanhanif/agent-skills --skill <name>`. This resolves against the **pushed GitHub repo**, not the local tree, so layout/frontmatter changes only take effect for users after a push. It depends entirely on the `skills/<name>/SKILL.md` location and the frontmatter `name` matching the `--skill` argument.
- **Manual copy (Claude Code):** users copy `skills/<name>/` into `~/.claude/skills/` (personal) or `.claude/skills/` (project). The skill must land at `~/.claude/skills/<name>/SKILL.md` — one level too deep silently fails to load.

Any rename of a skill directory or its frontmatter `name`, or a change to the `skills/` location, changes the install command — update the README's Install section and the command examples in lockstep.

## When editing the `requirements-engineering` skill

Its two governing principles (stated in `SKILL.md`): **exhaustive enumeration, not transcription** — when a capability area comes up, propose the full set of standard sub-requirements (auth → sign-up, sign-in, verification, forgot/reset, logout, session expiry, lockout, rate limiting…) and have the user confirm/extend/trim, rather than recording only what they mention — and **structured and traditional** — a formal SRS in the ISO/IEC/IEEE 29148 (IEEE 830) lineage with numbered, uniquely-identified, testable requirements, *not* an agile backlog/user stories. Epic/feature slicing is deliberately deferred downstream to `implementation-planning`; this skill owns the problem space.

Two things make this skill structurally different from the other three:
- **It checkpoints and resumes.** A full requirements interview is long, so it writes finalized areas straight into `docs/srs.md` and tracks progress in `docs/.requirements-progress.md` (a working file, not a deliverable). Preserve this if you touch the phases. (Output is markdown-only — three files, `srs.md` / `use-cases.md` / `rtm.md`; Word generation was removed to drop the Pandoc/python-docx dependency, so don't reintroduce a docx phase.)
- **It edits its own output after finalization (amendment mode).** It's not a one-shot generator — a finalized SRS can be changed, so the skill is both author and maintainer of the spec. This is what the next bullet's mode detection and Phase A exist for.

**Phase 0 is three-way mode detection, not just a resume check** — keep this intact if you touch the phases. From disk state plus the user's plain-language intent (no flag), it picks **fresh** (no SRS/tracker), **resume** (tracker has pending areas), or **amendment** (a *finalized* SRS exists and the user wants to change requirements). Two firm rules baked in: **resume beats amend** (finish an interrupted interview before taking new change requests), and **confirm-before-acting** when an SRS exists and intent is ambiguous — the expensive failure is clobbering or duplicating a finalized SRS. Finalization is recorded in the tracker (Phase 8); that flag is what later distinguishes amendment from resume.

**Phase A (amendment) has one cardinal rule: requirement IDs are immutable and never recycled** — everything downstream references them. Adds take the next free ID, modifies keep the ID, and removals are **tombstoned** (`Deprecated`/`Removed`, row kept) rather than deleted/renumbered. Every amendment bumps the SRS version + revision-history entry, propagates to the affected use cases and the RTM, and emits a **cross-skill impact note** naming which downstream docs referenced the changed IDs. That impact note is the skill's answer to the open gap above: it can't edit `architecture.md`/`ux-foundations.md`/`implementation-plan.md`, but it flags them for review/re-run.

The references support the SKILL's phases (more references than the other skills — keep them mutually consistent when changing one):
- `elicitation-guide.md` — the area-by-area functional interview (Phase 2).
- `requirement-catalog.md` — the enumeration engine: standard FR sub-requirements per area (incl. commonly-forgotten areas) and the ISO 25010-organized NFR catalog for **measurable** non-functional requirements (Phases 2–3).
- `use-case-guide.md` — deriving/specifying use cases + the Mermaid use-case diagram (Phase 5).
- `rtm-guide.md` — assembling the traceability matrix from the stable requirement IDs; carries a Status column so tombstoned requirements stay traceable (Phase 6).
- `srs-template.md` — IEEE 29148-lineage SRS structure (the output spine); includes a Revision History section and per-requirement Status/Finalized state.
- `checkpointing.md` — incremental-save/resume **and** the three-mode detection + `FINALIZED` status logic (Phase 0).
- `change-management.md` — the amendment protocol: ID-immutability cardinal rule, add/modify/remove handling, propagation, cross-skill impact note (Phase A).

## When editing the `software-architecture` skill

Its governing principle (stated in `SKILL.md`): **architecture is driven by quality attributes and constraints, not by technology** — the workflow enforces *understand → decide → document*, and every tech choice must trace back to a stated requirement. Keep edits aligned with this; don't add guidance that jumps to a stack before eliciting needs.

**It consumes the requirements artifacts (Input section + Phase 1).** Two inputs, by role: **SRS (primary)** — default `docs/srs.md`; when present it's the source of *what*, and Phase 1 becomes *ingest → play back the drivers (top NFRs by ID, constraints, capability areas) → interview only the gaps* (tech/stack preferences, lock-in tolerance, team skills, access patterns, deployment target), not a full re-interview. **Use cases (secondary)** — default `docs/use-cases.md`; when present they inform the **runtime view** (each significant use case → a runtime scenario / sequence diagram, cited by UC ID) and the **resilience/security** decisions (exception/alternate flows reveal failure-handling; actor→use-case mapping reveals trust boundaries). The **RTM is deliberately not consumed** (it only re-indexes the other two). When no SRS exists it falls back to the full seven-round elicitation and stays fully usable standalone. The boundary: the SRS states *what*; architecture decides *how* (stack, stores, deployment) — read only *mandated* tech from the SRS constraints. Preserve this graceful fallback if you touch the phases.

**Traceability is source-gated — this is the load-bearing rule, apply it everywhere.** Cite an ID only when the document that defines it is present: SRS requirement IDs when `srs.md` exists, use-case IDs when `use-cases.md` exists; when a source is absent, state the driver in plain prose and **never fabricate an ID**. This governs Phase-2 decisions (record what drives each fork), the ADR **"Requirements addressed"** field, and the document template's Introduction (top quality goals), Runtime View (UC IDs), Architecture Decisions, and Quality Requirements sections (§1 also summarizes/links `docs/srs.md` rather than restating it). Keep the gating intact when editing the templates — the common cases are both docs present (full pipeline, cite IDs) or neither (standalone, prose throughout), and the rule handles any mix.

The references form a pipeline matching the SKILL's five phases — keep them mutually consistent when changing one:
- `elicitation-guide.md` — the interview rounds (Phase 1); now run **gap-only** when an SRS exists, full when it doesn't. Each question must be one whose answer would change the design.
- `decision-guide.md` — how to reason through the recurring forks (Phase 2). Meta-rule: start as simple as requirements allow.
- `document-template.md` — arc42-based output, with sections tagged `[essential]` vs `[scale]` for right-sizing (Phase 3); source-gated ID citations (SRS NFR IDs, use-case IDs in the Runtime View) and links `docs/srs.md` rather than restating requirements.
- `diagram-guide.md` — C4 model framing, rendered as inline Mermaid; **prefer plain `flowchart`/`graph` syntax over experimental Mermaid C4 syntax** for portability (Phase 3).
- `adr-template.md` — one ADR per significant (costly-to-reverse) decision (Phase 4); its **"Requirements addressed"** field is source-gated (SRS IDs and/or use-case IDs when those docs exist, prose otherwise — never a fabricated ID).

## When editing the `ux-foundations` skill

Its governing principle (stated in `SKILL.md`): **one shared core, defined once, plus a per-surface layer for each interface** — a single design language (brand, tokens, accessibility, voice) spans every UI surface so the product feels like one thing, while each surface (admin portal, website, mobile app) gets its own profile because its users, navigation, and components genuinely differ. Keep edits aligned with this; don't collapse it into a single monolith or let surfaces drift into independent design systems. It stays at the **system/spec level** — it deliberately does *not* produce pixel-perfect screens or frontend component code.

Three structural things to preserve when editing (all added in the design-system overhaul):

- **Two input classes with opposite consumption rules.** *Pipeline documents* (SRS/architecture/use-cases in their contracted `docs/` locations) are **read silently — never ask**; consuming them is the pipeline working. The *design source* is **always asked, never assumed** — detection tells you what's *possible*, only the user says what's *wanted* (a `design.md` may be stale, images may be leftovers, they may want a fresh direction anyway). Don't collapse these into one "just detect and go" behavior.
- **Four design source modes** (Phase 1 asks, Phase 3 acquires): **research / extract from images / ingest a design file / connect a design tool** (Figma etc. via MCP), plus folded variants (brand book → ingest; existing codebase CSS/Tailwind → extract; mandated framework like Material/shadcn → constrains the component layer). Every mode ends by **playing back a proposed direction for confirmation** — extraction/ingestion/research are approximations, never asserted. Images are only ever read from `docs/design-refs/` — never scan the wider project.
- **Three outputs with a strict authority split** (Phase 7): `docs/tokens.json` (canonical machine-readable tokens, **W3C DTCG** format) is the single source of truth for values; `docs/design.md` (the **render-time**, agent-ready design system — its CSS block is *derived* from tokens.json) is the source of truth for anything design-system; `docs/ux-foundations.md` (the **plan-time** doc: personas, IA, flows, screen inventory) **references** design.md and never restates token values. Keep value authority in tokens.json/design.md so the three never drift.

**The SRS accessibility/usability NFRs override every source mode's output** (e.g., an ingested palette that fails the contrast bar gets adjusted, with the change recorded in design.md's provenance). Preserve that override — it's non-negotiable by design.

The references form a pipeline matching the SKILL's eight phases — keep them mutually consistent when changing one:
- `elicitation-guide.md` — the UX interview (Phase 2): only what upstream doesn't answer (device/responsive targets, brand feel, voice, i18n); personas come from SRS §2.3, confirmed not re-elicited.
- `source-modes.md` — the four design-source acquisition protocols (Phases 1 & 3); every mode ends in a confirmed direction, and SRS accessibility NFRs override all of them.
- `design-tool-integrations.md` — per-tool guidance for Mode 4 (connect a design tool): a generic MCP protocol that always works standalone, refined by optional per-tool sections.
- `design-system-guide.md` — the shared core (Phase 4): semantic role-based tokens, core component inventory with states, voice, accessibility standard cited by SRS NFR ID.
- `surface-profile-guide.md` — the per-surface layer (Phase 5): users/jobs (from SRS user classes), IA + navigation, key flows (cite UC IDs), screen inventory, surface-specific components + token overrides.
- `design-md-guide.md` — how to write the render-time `docs/design.md` (Phase 7): concrete values, component states/variants, layout/accessibility/usage rules, provenance + known gaps; reference it from CLAUDE.md so UI-building sessions inherit it.
- `document-template.md` — the plan-time `docs/ux-foundations.md` structure (Phase 7): shared core plus one section per surface, right-sized, referencing design.md.
- `diagram-guide.md` — sitemaps + user flows as inline Mermaid; **prefer plain `flowchart`/`graph` syntax** for portability, same reliability rationale as the architecture skill's diagram guide.

Two hand-offs to preserve: the **screen inventory** (in `ux-foundations.md`) feeds implementation-planning's vertical slicing, and **design.md + tokens.json** feed every downstream UI-building session (the skill suggests referencing design.md from CLAUDE.md).

## When editing the `implementation-planning` skill

Its two governing principles (stated in `SKILL.md`): **slice vertically, never horizontally** — every unit of work cuts through UI, API, domain, and data to deliver something demonstrable, never "build all the tables" — and **make the architecture executable before complete** — build the *walking skeleton* (minimal end-to-end path that proves the system runs and deploys) first, not the easiest feature, then sequence the rest by dependency and risk. It plans the whole app's breadth but only details the first/near slices (**depth-on-demand**, not waterfall); don't add guidance that elaborates the entire backlog upfront. It **stops at the plan** — per-feature detailed design (API contracts, schemas, per-screen design) and code/scaffolding are separate downstream steps.

The references form a pipeline matching the SKILL's eight phases — keep them mutually consistent when changing one:
- `slicing-guide.md` — decomposition (Phase 3): epics from building blocks, thin vertical feature slices, each mapped to screens + endpoints + flow.
- `sequencing-guide.md` — walking skeleton (Phase 4) and risk/dependency ordering (Phase 5).
- `document-template.md` — output structure (Phase 8): the epic/feature breakdown is a **default** section, not an optional add-on; the issue-tracker export is off by default.
- `diagram-guide.md` — dependency graph as inline Mermaid; **prefer plain `flowchart`/`graph` syntax** for portability, same reliability rationale as the other two skills' diagram guides.

## Validating changes

There's nothing to build or run. To sanity-check a skill:
- Confirm `SKILL.md` frontmatter `name` matches the directory name.
- Verify every `references/...` path mentioned in `SKILL.md` actually exists.
- Render any Mermaid you add (GitHub preview or a Mermaid live editor) — the diagram guide's reliability notes exist because broken diagrams are the common failure.
