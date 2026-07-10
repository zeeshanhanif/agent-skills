---
name: project-scaffolding
description: >-
  Turns the pipeline's design documents into a running system. Reads the
  architecture document and implementation plan, executes each ecosystem's
  current official project generator to create the real repo structure (one
  deployable unit per architecture container, monorepo by default), wires the
  walking skeleton end-to-end — UI shell with the design system's tokens.json,
  API, domain stub, local database — stands up the engineering foundations
  (CI, environments, test harness, deployment config written but not
  executed), and verifies empirically by building and running the skeleton
  test. Deploy-ready, not deployed. Use after implementation-planning and
  before per-slice work, when the plan's walking skeleton needs to become a
  real repo. Trigger on "scaffold the project", "set up the repo", "create
  the project structure", "build the walking skeleton", "initialize the
  codebase", or "stand up the skeleton". Checkpoints progress and resumes
  safely — never regenerates over a partial scaffold.
---

# Project Scaffolding

The first skill in the pipeline whose output is a **running system**, not a
document. It takes the architecture and the plan and produces the walking
skeleton as a real repo: generated structure, wired end-to-end, foundations in
place, verified by execution. It ends where deployment begins — everything is
deploy-*ready*; the first actual deploy is the user's step.

Three principles govern it:

1. **The stack is an input, never a decision.** The architecture's ADRs already
   chose the technology. This skill reads the stack and refuses to relitigate
   it. Gaps the generators need answered (package manager, monorepo tool) are
   elicited — and flagged as candidate architecture amendments.
2. **Official generators first, manual structure second.** Every serious
   ecosystem ships an official project initializer that encodes *current*
   best-practice structure. Discover it, verify it against live docs (never
   trust memory for CLI names/flags — live reality always wins), run it for
   real, then customize. Hand-build only where no generator exists, following
   that ecosystem's documented conventions. This keeps the skill self-updating:
   framework conventions change, generators change with them, the skill
   inherits it.
3. **What this skill owns is the stack-independent layer**: the walking-skeleton
   wiring between units, module-boundary enforcement, the design system wired
   into the UI shell, pipeline conventions carried into the repo, and empirical
   verification. Generators create isolated apps; this skill makes them a
   system.

## Inputs

Defaults below; user-provided paths win. Each degrades independently.

- **Architecture** — `docs/architecture.md` (primary). The stack per container
  (from ADRs), the containers themselves (each becomes a deployable unit),
  module boundaries, deployment target, cross-cutting concepts. Without it the
  skill can still run by eliciting the stack directly, but say plainly that
  fidelity suffers — this skill is designed to realize an architecture.
- **Implementation plan** — `docs/implementation-plan.md`. The walking-skeleton
  spec (what's real, what's stubbed, done-when) and the engineering-foundations
  checklist. If missing, derive a minimal skeleton from the architecture and
  confirm it with the user.
- **UX foundations** — `docs/ux-foundations.md`, `docs/design.md`,
  `docs/tokens.json`. The UI-shell contract: tokens wired into each frontend
  surface's token mechanism, design.md referenced from the agent-instructions
  file.
- **SRS** — `docs/srs.md`, light touch: constraints (§2.5) affecting tooling.

**Source-gated traceability**: scaffold decisions cite ADR and container names
when the documents exist; prose otherwise; never fabricate a reference.

**RTM: no write-back.** Scaffolding realizes containers, not requirements —
nothing here maps to a requirement row, and per the RTM column-ownership
contract this skill does not touch `docs/rtm.md`.

## Outputs

1. **The repo** — generated per-unit structure (monorepo by default), module
   boundaries enforced (folders + import/lint rules), skeleton wired
   end-to-end, one passing end-to-end test, CI config, environment configs,
   deployment config written (not executed), `docs/` carried in.
2. **Agent-instructions file** (CLAUDE.md or the project's equivalent) —
   pointing at the pipeline documents, design.md, and the repo's conventions,
   so every future coding session inherits the context.
3. **`docs/scaffold-notes.md`** — the project-owned record: generators +
   versions + flags actually used, deviations from expectation, decisions
   made, what's stubbed where. Consumers: future sessions on this project, the
   build phases, and the skill author if they ever hand-promote a durable
   lesson into the skill. Notes live in the project, never in the skill folder.

## Workflow

### Phase 0 — Resume check (always first)

Read `references/checkpointing.md`. Look for `docs/.scaffold-progress.md` and
signs of a partial scaffold. If found: **never regenerate over it** —
regeneration over a half-built repo is destructive. Summarize what exists,
what's pending, confirm, and resume at the first pending unit/step. Fresh
start: create the tracker and proceed.

### Phase 1 — Ingest and confirm the scaffold plan (the one gate)

Read the input documents. Extract the build contract: per container — language/
framework (cite the ADR), store bindings, boundaries; plus the skeleton spec
and foundations checklist from the plan. Then play back a short **scaffold
plan** — units and their stacks, repo shape, local data story, CI target, the
done-when condition — and get explicit confirmation. **This is the single gate
before real files are created**; after the nod, the skill executes without
further ceremony.

Repo shape: **monorepo by default**; ask only when signals point otherwise
(e.g., the deployment view implies independently released units or separate
ownership).

### Phase 2 — Gap elicitation (brief)

Ask only what the documents don't answer and the generators require: package
manager, monorepo tool (if multi-unit), repo name/location, versions where it
matters. Batch, offer defaults, keep it short. Anything the *architecture*
should have answered gets flagged as a candidate architecture amendment.

### Phase 3 — Environment preflight and tooling discovery

Read `references/scaffolding-guide.md` (Tooling discovery). Preflight the
environment: required runtimes, package managers, Docker (if the local data
story needs it). Missing prerequisites are a **blocking ask** — surfaced to
the user, never silently worked around. Then, per unit: identify the
ecosystem's current official generator, verify name/flags against live docs
when uncertain, and record generator + version + flags into scaffold-notes.
No generator → plan manual structure from documented conventions.

### Phase 4 — Generate and structure

Run the generators **for real**, per unit, checkpointing after each. Then apply
the pipeline's layer (scaffolding-guide, Structure): monorepo arrangement,
module boundaries as folders + import/lint rules (the architecture's seams made
enforceable), strip generator boilerplate that conflicts with the architecture,
carry `docs/` into the repo, create the agent-instructions file.

### Phase 5 — Wire the walking skeleton

Read `references/skeleton-guide.md`. One trivial end-to-end path: UI shell
(tokens.json wired into each frontend's token mechanism; design.md referenced)
→ API endpoint → domain stub → local database → back. Stubs exactly where the
plan said stubs. Checkpoint per wiring step.

### Phase 6 — Engineering foundations

The plan's checklist made real (skeleton-guide, Foundations): CI pipeline
(lint, test, build) that would run green; environment configs as config, not
provisioned; observability hooks (structured logging at minimum); the test
harness with the one end-to-end skeleton test; deployment config written for
the architecture's target. **Deploy-readiness is the boundary** — nothing is
provisioned or deployed.

### Phase 7 — Empirical verification

Read `references/verification.md`. Verification here means **running things**:
clean install, every unit builds, the skeleton test passes locally (the plan's
done-when — local half verified; deployed half recorded as pending first
deploy), lint/boundary rules pass, CI config valid, tokens actually render in
the shell, agent-instructions paths resolve. Fix-loop failures; flag anything
unfixable in the delivery summary — never silently ship a broken skeleton.

### Phase 8 — Deliver

Summarize: what exists, the exact commands to run it, what's stubbed, the
pending-first-deploy note, and the handoff — the skeleton is ready for slice 1
via **detailed-design** (contracts/data) and **ui-design** (screens by SCR ID).
Finalize scaffold-notes and mark the tracker complete.

## Scope boundaries

Does **not**: choose or change the stack (architecture's job — gaps go back as
amendments); deploy or provision infrastructure (deploy-ready only); implement
features or design slices (per-slice detailed-design/ui-design, then build);
write to the RTM.

## What good looks like

- Every deployable unit traces to an architecture container; every stack choice
  cites its ADR (source-gated).
- Generators were discovered and verified live, run for real, and recorded —
  name, version, flags — in scaffold-notes.
- The skeleton test passes on a clean local run; the done-when condition's
  local half is *demonstrated*, not asserted.
- Module boundaries are enforceable (lint/import rules), not aspirational.
- tokens.json is actually wired — delete it and the shell visibly breaks.
- A stopped run resumes without regenerating anything already built.
