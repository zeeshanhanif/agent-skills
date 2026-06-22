---
name: implementation-planning
description: >-
  Turns a finalized architecture and UX-foundations into an executable,
  sequenced build plan. Reads the architecture document and the UX-foundations
  document, then decomposes the system into epics and vertical feature slices
  (each mapped to the screens it touches and the building blocks/endpoints it
  needs), defines the walking skeleton, orders the work by dependency and risk,
  specifies the first vertical slice, and lists the engineering foundations to
  stand up. Use this in the planning phase — after software-architecture and
  ux-foundations, before scaffolding or coding — whenever a designed
  application needs to become a backlog and a build order. Trigger on phrases
  like "plan the implementation", "break this into epics and features", "what
  do we build first", "create the build plan", "turn the architecture into a
  roadmap", or "sequence the work". It stops at the plan; per-feature detailed
  design and code are separate downstream steps.
---

# Implementation Planning

This skill is the bridge between design and construction. It takes the two
design-phase documents and produces a plan a team (or an agent) can execute: what
to build, broken into thin vertical slices, in what order, starting where.

Two principles govern it:

1. **Slice vertically, never horizontally.** A unit of work is a thin slice that
   cuts through every layer — UI, API, domain, data — and delivers something
   demonstrable. "Build all the tables" or "build all the screens" are horizontal
   slices: they deliver no value and integrate late, which is where projects go
   to die. Every feature here spans backend and UI together.
2. **Make the architecture executable before making it complete.** The first
   thing built is the *walking skeleton* — the minimal end-to-end path that
   proves the system runs and deploys — not the easiest feature. Sequence the
   rest by dependency and risk, de-risking the most uncertain decisions early.

It plans the whole application's breadth but only details what's next. Producing
implementation-ready detail for the entire backlog upfront is the waterfall trap;
this skill gives full breadth and depth-on-demand.

## Inputs

This skill consumes the two design-phase documents.

- **Architecture** — use a provided path, otherwise default to
  `docs/architecture.md`. Source of the building blocks/containers, the ADRs and
  their risk, the runtime scenarios, constraints (team, timeline), and the
  deployment view.
- **UX foundations** — use a provided path, otherwise default to
  `docs/ux-foundations.md`. Source of the surfaces, the per-surface screen
  inventory, and the key user flows.

If either is missing, tell the user and offer to point to the right path or
proceed in a reduced mode (the plan is materially weaker without both — the
architecture gives the backend slices, ux-foundations gives the UI slices, and
you need both to slice vertically). Prefer locating the documents.

## Workflow

### Phase 1 — Ingest and frame

Read both documents. Build a combined picture: the building blocks and their
dependencies, the ADRs flagged as risky or uncertain, the surfaces and their
screen inventories, and the key user flows. Note the stated constraints (team
size, timeline, priorities) from the architecture so the plan is realistic.

### Phase 2 — Confirm priorities (brief)

Most of what you need is in the documents. Ask only what changes the plan and
isn't already answered: what "first" should optimize for (de-risk the scariest
part, reach a demoable milestone, or unblock the most downstream work), any fixed
deadlines or an MVP cut line, and team capacity if the architecture didn't say.
Keep this short; infer from the documents and offer defaults.

### Phase 3 — Decompose

Read `references/slicing-guide.md`. Break the system into **epics** (aligned to
the architecture's building blocks / bounded contexts) and **features** (thin
vertical slices within each epic). For every feature, record what it touches:
the screens (from the ux-foundations inventory), the building blocks and
endpoints (from the architecture), and the user flow it serves. This is the
whole-app breadth — every feature, coarse where it's far off.

### Phase 4 — Define the walking skeleton

Read `references/sequencing-guide.md`. Specify the walking skeleton: the minimal
end-to-end path through the major containers (plus a minimal UI shell with the
design system wired in) that proves the system runs and deploys. State explicitly
what's real and what's stubbed.

### Phase 5 — Sequence

Order the work by **dependency** (foundational capabilities before what needs
them) and **risk** (the most uncertain ADRs, hardest integrations, and
highest-coupling components first — fail early while it's cheap). Group into
phases/milestones if it helps. Produce a dependency graph (see
`references/diagram-guide.md`).

### Phase 6 — Specify the first vertical slice

Define the first slice to build after the skeleton: narrow, demonstrable,
exercising a real user flow and ideally touching a risky area to validate it.
Give it acceptance criteria, the screens it renders, and the endpoints/data it
needs — enough to scope and assign it, **but not full detailed design** (that's
the downstream per-slice step).

### Phase 7 — Engineering foundations

List the foundations to stand up alongside the skeleton, derived from the
architecture's cross-cutting concepts (repo conventions, CI/CD, environments,
observability, test strategy) and the ux-foundations (accessibility bar,
design-system/tokens setup). A checklist is fine.

### Phase 8 — Document and deliver

Read `references/document-template.md` and write the plan to
`docs/implementation-plan.md` by default. Summarize the build order and the first
slice in a few sentences. Note that the **first slice (and any near slices) are
the handoff to the detailed-design step**. Offer next steps: scaffold the walking
skeleton, or run detailed design on the first slice.

## Output format

Default to a single Markdown file (`docs/implementation-plan.md`) with the
walking-skeleton definition, the epic/feature breakdown, a Mermaid dependency
graph, the first-slice spec, and the foundations checklist. The epic/feature
breakdown is always part of the plan itself — it is a default section, not an
optional add-on. On request, the *same* features can additionally be emitted as
paste/import-ready issue items (GitHub / Linear / Jira); this export is off by
default because it would otherwise duplicate the breakdown in a more verbose
form. The export reshapes the existing breakdown into discrete tickets (title,
body, labels, acceptance criteria) — it adds format, not new information.

## Scope boundaries

It deliberately does **not**:
- produce per-feature detailed design (API contracts, data schemas, per-screen
  designs) — that's the downstream `detailed-design` step, run per slice;
- generate scaffolding or code — that's `project-scaffolding` and build.

Keeping planning separate from detailed design is what enables depth-on-demand:
you elaborate slices as they reach the front of the queue, not all upfront.

## What good looks like

- Every feature is a vertical slice mapped to real screens and real building
  blocks — traceable to both input documents.
- The first thing to build is the walking skeleton, not the easiest feature.
- The sequence front-loads the riskiest and most-depended-on work.
- Full breadth across the app; full depth only on the first/near slices.
- Right-sized: a tight plan for a small app, a phased roadmap for a large one.
