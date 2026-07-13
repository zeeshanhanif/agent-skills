---
name: implementation-planning
description: >-
  Turns finalized requirements and design documents into an executable,
  sequenced build plan. Reads the SRS, use cases, architecture document, and
  UX-foundations document, then decomposes the system into epics and vertical
  feature slices — each tracing the FR IDs it implements, the UC IDs it
  realizes, and the SCR screen IDs it touches — defines the walking skeleton,
  orders the work by dependency and risk, verifies Must-requirement coverage,
  specifies the first vertical slice, and lists the engineering foundations to
  stand up. Use in the planning phase — after software-architecture and
  ux-foundations, before scaffolding or coding — whenever a designed
  application needs to become a backlog and a build order. Trigger on "plan the
  implementation", "break this into epics and features", "what do we build
  first", "create the build plan", "turn the architecture into a roadmap", or
  "sequence the work". It stops at the plan; per-slice detailed design and
  ui design are separate downstream steps.
---

# Implementation Planning

This skill is the bridge between design and construction. It takes the
requirements and design-phase documents and produces a plan a team (or an
agent) can execute: what to build, broken into thin vertical slices, in what
order, starting where.

Two principles govern it:

1. **Slice vertically, never horizontally.** A unit of work is a thin slice
   that cuts through every layer — UI, API, domain, data — and delivers
   something demonstrable. "Build all the tables" or "build all the screens"
   are horizontal slices: they deliver no value and integrate late. Every
   feature here spans backend and UI together.
2. **Make the architecture executable before making it complete.** The first
   thing built is the *walking skeleton* — the minimal end-to-end path that
   proves the system runs and deploys — not the easiest feature. Sequence the
   rest by dependency and risk, de-risking the most uncertain decisions early.

It plans the whole application's breadth but only details what's next.
Producing implementation-ready detail for the entire backlog upfront is the
waterfall trap; this skill gives full breadth and depth-on-demand.

## Inputs

Four upstream documents, each read from its default path unless the user
provides another. Each degrades independently — a missing document reduces
fidelity, it doesn't block (though the plan is materially weaker without the
architecture and ux-foundations, which carry the building blocks and screens
that make slices vertical).

- **SRS** — `docs/srs.md`. The source of *what a feature is*: functional
  requirements with IDs and priorities (MoSCoW feeds sequencing and the
  coverage check), plus constraints (team, timeline). **Skip tombstoned
  requirements** (status `Removed`/`Deprecated`) — they must not spawn slices.
- **Use cases** — `docs/use-cases.md`. Slices often realize use cases; their
  flows are prime slice candidates.
- **Architecture** — `docs/architecture.md`. Building blocks and their
  dependencies, risky ADRs, runtime scenarios, deployment view, cross-cutting
  concepts.
- **UX foundations** — `docs/ux-foundations.md`. Surfaces, per-surface screen
  inventories with **SCR IDs**, key user flows. (Skip tombstoned screens.)

**Source-gated traceability** (the pipeline rule): cite an ID — FR/NFR, UC,
SCR — only when the document defining it is present; otherwise describe in
prose. **Never fabricate an ID.**

## Workflow

### Phase 1 — Ingest and frame

Read all four documents (per Inputs). Build a combined picture: the functional
requirements by area and priority, the building blocks and their dependencies,
the ADRs flagged risky, the surfaces with their screen inventories, and the
key user flows. Note constraints (team size, timeline, priorities) so the plan
is realistic. Tell the user which documents were found and which are missing.

### Phase 2 — Confirm priorities (brief)

Most of what you need is in the documents. Ask only what changes the plan and
isn't already answered: what "first" should optimize for (de-risk the scariest
part, reach a demoable milestone, or unblock the most downstream work), any
fixed deadlines or an MVP cut line, and team capacity if the SRS/architecture
didn't say. Keep it short; infer and offer defaults.

### Phase 3 — Decompose

Read `references/slicing-guide.md`. Break the system into **epics** (aligned to
the architecture's building blocks / bounded contexts) and **features** (thin
vertical slices within each epic; "feature" and "slice" are synonyms in this
pipeline). **Mint a stable ID for every feature: `FEAT-NNN`**, sequential in
definition order (not build order — build order can change; IDs never do).
Same immutability rules as FR/SCR IDs: never renumbered or recycled;
amendments take the next free number; removed features are tombstoned, not
deleted. The epic association lives in the table, not the ID — features may
migrate between epics without renumbering. For every feature, record its
traced touchpoints:
- **FRs implemented** (by ID, e.g., FR-AUTH-001..003)
- **UCs realized** (by ID, e.g., UC-004)
- **Screens touched** (by SCR ID and surface, e.g., SCR-WEB-004, SCR-ADM-002)
- **Building blocks / endpoints** (from the architecture)
- **Data** (entities read/written)

This is whole-app breadth — every feature, coarse where far off.

Then run the **coverage check** (see `references/slicing-guide.md`, Coverage):
every non-tombstoned Must-priority FR, and every inventory screen, must be
covered by a slice, placed in the foundations, or explicitly excluded. Apply
the escalation ladder: **self-resolve → ask the user (batched, with your best
guess per item) → record unresolved items as explicit open gaps in Risks.**
Should/Could gaps go straight to the report.

### Phase 4 — Define the walking skeleton

Read `references/sequencing-guide.md`. Specify the walking skeleton: the
minimal end-to-end path through the major containers (plus a minimal UI shell
with the design system wired in — design.md/tokens.json from ux-foundations)
that proves the system runs and deploys. State explicitly what's real and
what's stubbed.

### Phase 5 — Sequence

Order the work by **dependency** (foundational capabilities before what needs
them) and **risk** (uncertain ADRs, hardest integrations, highest-coupling
components first — fail early while it's cheap). SRS priorities (Must before
Should before Could) shape ordering within dependency constraints. Group into
phases/milestones if it helps. Produce a dependency graph (see
`references/diagram-guide.md`).

### Phase 6 — Specify the first vertical slice

Define the first slice after the skeleton: narrow, demonstrable, exercising a
real user flow, ideally touching a risky area. Give it acceptance criteria
(derived from its FRs' statements and UC flows), the screens it renders (SCR
IDs), and the endpoints/data it needs — enough to scope and assign, **not**
full detailed design. **Name the handoffs explicitly:** the screens list (SCR
IDs) is the input to the downstream **ui-design** step; the endpoints/data/
contract needs are the input to the downstream **detailed-design** step.

### Phase 7 — Engineering foundations

List the foundations to stand up alongside the skeleton, derived from the
architecture's cross-cutting concepts (repo conventions, CI/CD, environments,
observability, test strategy) and ux-foundations (accessibility bar,
design-system/token setup). Cross-cutting FRs placed here by the coverage
check (e.g., audit logging) appear with their FR IDs.

### Phase 8 — Verify

Read `references/verification.md` and run the self-check before delivering:
ID integrity (every cited FR/UC/SCR exists in its source; no tombstoned item
spawned a slice), coverage completeness (every Must FR and every screen is
sliced, placed, or documented as an open gap), structural soundness (acyclic
dependency graph over defined slices; first slice at the sequence front;
Mermaid parses), and handoff completeness (first-slice spec names both
downstream targets with required fields). Fix what fails; flag in the summary
anything unfixable.

### Phase 9 — Document and deliver

Read `references/document-template.md` and write the plan to
`docs/implementation-plan.md`.

**RTM write-back** (if `docs/rtm.md` exists; skip silently otherwise): fill the
**Plan ref** column for every requirement a feature implements or the
foundations absorb — the **feature ID** (e.g., `FEAT-012`) or `Foundations`.
The FEAT ID is the stable join key downstream skills use to reference this
feature, and the key it shares with the RTM. **Append
into cells, never overwrite**, and touch only Plan ref: rows and requirement
columns belong to requirements-engineering, Design ref to
architecture/detailed-design, Test ref to testing. Scheduling is not design —
that's why Plan ref is its own column.

Summarize the build order, the first slice, and any open coverage gaps. Note
the handoffs: first-slice screens → **ui-design**; first-slice contracts/data →
**detailed-design**. Offer next steps: scaffold the walking skeleton, or run
detailed design on the first slice.

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
- produce per-feature detailed design (API contracts, data schemas) — that's
  the downstream **detailed-design** step, run per slice;
- design screens — that's the downstream **ui-design** step, run per slice;
- generate scaffolding or code.

Keeping planning separate from per-slice design is what enables
depth-on-demand: slices are elaborated as they reach the front of the queue.

## What good looks like

- Every feature is a vertical slice traced to the FR IDs it implements, the UC
  IDs it realizes, and the SCR IDs it touches (source-gated — prose when a
  document is absent, never fabricated).
- Every Must-priority FR and every inventory screen is sliced, placed in
  foundations, or an explicit documented gap — nothing silently dropped.
- The first thing built is the walking skeleton; the sequence front-loads risk
  and dependencies; priorities shape order within those constraints.
- The first-slice spec is a complete handoff to ui-design and detailed-design.
- Full breadth across the app; full depth only on the first/near slices.
