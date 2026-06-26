# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of **Agent Skills** following the open `SKILL.md` standard (see https://agentskills.io). There is no application code, build system, test suite, or linter — the deliverable is prompt/instruction content that Claude (or another agent) loads at runtime. "Correctness" here means the instructions are clear, well-scoped, and trigger reliably, not that anything compiles.

Each skill lives under `skills/` as its own directory containing a `SKILL.md` and (optionally) a `references/` folder of supporting guides. There are currently four skills, designed to run back-to-back as a greenfield requirements-to-build pipeline:
- `skills/requirements-engineering/` — the first SDLC step, upstream of design. Elicits the complete requirements and produces the SRS (`docs/srs.md` + `docs/srs.docx`), a use-case document (`docs/use-cases.md` + `.docx`), and a traceability matrix (`docs/rtm.md`). It has **no upstream document** — it elicits from the user. It also **amends** a finalized SRS (add/modify/remove with stable, never-recycled IDs), so it owns the spec across its whole lifecycle, not just first authoring.
- `skills/software-architecture/` — interviews the user and produces the architecture document (`docs/architecture.md`).
- `skills/ux-foundations/` — the "architecture of the UI." It **consumes** the architecture document (defaults to reading `docs/architecture.md`) and produces the UX foundations (`docs/ux-foundations.md`).
- `skills/implementation-planning/` — the bridge from design to construction. It **consumes both** prior design documents (defaults `docs/architecture.md` + `docs/ux-foundations.md`) and produces an executable build plan (`docs/implementation-plan.md`).

The hand-offs are real couplings, not loose suggestions:
- `requirements-engineering` emits **two primary documents**, not one: `docs/srs.md` (the requirement source of truth — NFRs, personas/user classes, constraints) **and** `docs/use-cases.md` (detailed use-case specs + a Mermaid use-case diagram, traced back to the FRs), plus the `docs/rtm.md` traceability matrix linking them. The SRS is the requirement spine; the use-case document carries the actor/flow detail that maps naturally onto ux-foundations' key flows and implementation-planning's vertical slices — treat both as downstream inputs, not srs.md alone. It defers epic/feature slicing to `implementation-planning` and design decisions to the design skills. **Note an existing gap:** the three downstream skills were written first and do *not* yet read `srs.md` or `use-cases.md` — they default to their own inputs (`architecture.md`, `ux-foundations.md`) and still elicit via interview. Wiring them to ingest the SRS and use cases is an open follow-up; keep this in mind when editing any of them.
- `ux-foundations` extracts the UI surfaces, actors, and constraints from the architecture doc's containers/context.
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

Three things make this skill structurally different from the other three:
- **It checkpoints and resumes.** A full requirements interview is long, so it writes finalized areas straight into `docs/srs.md` and tracks progress in `docs/.requirements-progress.md` (a working file, not a deliverable). Preserve this if you touch the phases.
- **It emits Word, not just markdown.** Markdown sources (`srs.md`, `use-cases.md`) are the checkpointed source of truth; the `.docx` files are generated **last** (Phase 7) from the finalized markdown, so they never need checkpointing.
- **It edits its own output after finalization (amendment mode).** It's not a one-shot generator — a finalized SRS can be changed, so the skill is both author and maintainer of the spec. This is what the next bullet's mode detection and Phase A exist for.

**Phase 0 is three-way mode detection, not just a resume check** — keep this intact if you touch the phases. From disk state plus the user's plain-language intent (no flag), it picks **fresh** (no SRS/tracker), **resume** (tracker has pending areas), or **amendment** (a *finalized* SRS exists and the user wants to change requirements). Two firm rules baked in: **resume beats amend** (finish an interrupted interview before taking new change requests), and **confirm-before-acting** when an SRS exists and intent is ambiguous — the expensive failure is clobbering or duplicating a finalized SRS. Finalization is recorded in the tracker (Phase 8); that flag is what later distinguishes amendment from resume.

**Phase A (amendment) has one cardinal rule: requirement IDs are immutable and never recycled** — everything downstream references them. Adds take the next free ID, modifies keep the ID, and removals are **tombstoned** (`Deprecated`/`Removed`, row kept) rather than deleted/renumbered. Every amendment bumps the SRS version + revision-history entry, propagates to the affected use cases and the RTM, regenerates the `.docx` files, and emits a **cross-skill impact note** naming which downstream docs referenced the changed IDs. That impact note is the skill's answer to the open gap above: it can't edit `architecture.md`/`ux-foundations.md`/`implementation-plan.md`, but it flags them for review/re-run.

The references support the SKILL's phases (more references than the other skills — keep them mutually consistent when changing one):
- `elicitation-guide.md` — the area-by-area functional interview (Phase 2).
- `requirement-catalog.md` — the enumeration engine: standard FR sub-requirements per area (incl. commonly-forgotten areas) and the ISO 25010-organized NFR catalog for **measurable** non-functional requirements (Phases 2–3).
- `use-case-guide.md` — deriving/specifying use cases + the Mermaid use-case diagram (Phase 5).
- `rtm-guide.md` — assembling the traceability matrix from the stable requirement IDs; carries a Status column so tombstoned requirements stay traceable (Phase 6).
- `srs-template.md` — IEEE 29148-lineage SRS structure (the output spine); includes a Revision History section and per-requirement Status/Finalized state.
- `checkpointing.md` — incremental-save/resume **and** the three-mode detection + `FINALIZED` status logic (Phase 0).
- `change-management.md` — the amendment protocol: ID-immutability cardinal rule, add/modify/remove handling, propagation, cross-skill impact note (Phase A).
- `docx-generation.md` — rendering finalized markdown to Word (Phase 7) via Pandoc (primary), with a `python-docx` fallback.

## When editing the `software-architecture` skill

Its governing principle (stated in `SKILL.md`): **architecture is driven by quality attributes and constraints, not by technology** — the workflow enforces *understand → decide → document*, and every tech choice must trace back to a stated requirement. Keep edits aligned with this; don't add guidance that jumps to a stack before eliciting needs.

The references form a pipeline matching the SKILL's five phases — keep them mutually consistent when changing one:
- `elicitation-guide.md` — the interview rounds (Phase 1). Each question must be one whose answer would change the design.
- `decision-guide.md` — how to reason through the recurring forks (Phase 2). Meta-rule: start as simple as requirements allow.
- `document-template.md` — arc42-based output, with sections tagged `[essential]` vs `[scale]` for right-sizing (Phase 3).
- `diagram-guide.md` — C4 model framing, rendered as inline Mermaid; **prefer plain `flowchart`/`graph` syntax over experimental Mermaid C4 syntax** for portability (Phase 3).
- `adr-template.md` — one ADR per significant (costly-to-reverse) decision (Phase 4).

## When editing the `ux-foundations` skill

Its governing principle (stated in `SKILL.md`): **one shared core, defined once, plus a per-surface layer for each interface** — a single design language (brand, tokens, accessibility, voice) spans every UI surface so the product feels like one thing, while each surface (admin portal, website, mobile app) gets its own profile because its users, navigation, and components genuinely differ. Keep edits aligned with this; don't collapse it into a single monolith or let surfaces drift into independent design systems. It stays at the **system/spec level** — it deliberately does *not* produce pixel-perfect screens or frontend component code.

The references form a pipeline matching the SKILL's seven phases — keep them mutually consistent when changing one:
- `elicitation-guide.md` — the UX interview (Phase 2): only what the architecture doesn't already cover; infer from the architecture first.
- `design-system-guide.md` — the shared core (Phase 3): design tokens, accessibility standard, voice, cross-surface interaction principles.
- `surface-profile-guide.md` — the per-surface layer (Phase 4): users/jobs, IA + navigation, key flows, screen inventory, surface-specific components.
- `document-template.md` — output structure (Phase 6): shared core plus one section per surface, right-sized.
- `diagram-guide.md` — sitemaps + user flows as inline Mermaid; **prefer plain `flowchart`/`graph` syntax** for portability, same reliability rationale as the architecture skill's diagram guide.

Note the screen inventory is framed as the hand-off to implementation planning — preserve that framing if you touch the surface-profile or delivery phases.

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
