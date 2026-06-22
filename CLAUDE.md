# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of **Agent Skills** following the open `SKILL.md` standard (see https://agentskills.io). There is no application code, build system, test suite, or linter — the deliverable is prompt/instruction content that Claude (or another agent) loads at runtime. "Correctness" here means the instructions are clear, well-scoped, and trigger reliably, not that anything compiles.

Each skill lives under `skills/` as its own directory containing a `SKILL.md` and (optionally) a `references/` folder of supporting guides. There are currently three skills, designed to run back-to-back as a greenfield design-to-build pipeline:
- `skills/software-architecture/` — interviews the user and produces the architecture document (`docs/architecture.md`).
- `skills/ux-foundations/` — the "architecture of the UI." It **consumes** the architecture document (defaults to reading `docs/architecture.md`) and produces the UX foundations (`docs/ux-foundations.md`).
- `skills/implementation-planning/` — the bridge from design to construction. It **consumes both** prior documents (defaults `docs/architecture.md` + `docs/ux-foundations.md`) and produces an executable build plan (`docs/implementation-plan.md`).

The hand-offs are real couplings, not loose suggestions:
- `ux-foundations` extracts the UI surfaces, actors, and constraints from the architecture doc's containers/context.
- `implementation-planning` slices vertically by joining the architecture's building blocks/endpoints with the ux-foundations' per-surface **screen inventory** — it needs both to form a slice, which is why `ux-foundations` frames its screen inventory as that hand-off.

So changes to what an upstream skill emits (surface naming, container vocabulary, the screen-inventory shape) can ripple downstream into how the next skill ingests it. Keep the vocabulary consistent across the three when editing any one.

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
