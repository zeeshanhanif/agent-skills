# Scaffolding Guide

Mechanics for Phases 1–4: extracting the build contract, discovering tooling,
generating, and structuring. The philosophy throughout: **the skill encodes
process, not stack structures** — the executing agent's knowledge plus live
verification plus official generators supply the per-ecosystem specifics.

## Extracting the build contract (Phase 1)

From the architecture: each **container** in the container diagram becomes a
deployable unit. For each, pull: language/framework (from the ADR that chose
it — cite it), the store(s) it binds to, and the module boundaries inside it
(the sub-domain seams the ADRs promised). From the plan: the walking-skeleton
spec verbatim (what's real, what's stubbed, done-when) and the foundations
checklist. From ux-foundations/design.md/tokens.json: which units are UI
surfaces and therefore get the token wiring.

The **scaffold plan playback** is one tight block: units → stacks (with ADR
refs), repo shape, local data story (e.g., docker-compose Postgres), CI
target, done-when. One confirmation, then execute.

## Repo shape

**Monorepo is the default** — one repo, one history, agent-friendly, right for
a solo developer or single team, and the natural home for pipeline `docs/`.
Ask only when signals point otherwise: the deployment view implies units
released on independent cadences, separate team ownership per container, or an
explicit user/org convention. Multi-unit monorepos get a workspace tool
appropriate to the dominant ecosystem (elicited in Phase 2 if the architecture
didn't say).

## Tooling discovery (Phase 3)

For each unit:

1. Identify the ecosystem's **current official generator/initializer** — the
   framework's own CLI, not a community starter kit, unless the architecture
   mandated a specific starter.
2. **Verify against live documentation when uncertain** — generator names,
   flags, and defaults change between versions; memory of a CLI's interface is
   the least reliable knowledge there is. Live reality always wins over both
   memory and any written note.
3. Record into `docs/scaffold-notes.md` *before running*: generator, version,
   the exact flags chosen and why (interactive prompts answered how, and on
   what basis — architecture constraint, user answer, or default).
4. **No generator exists** (some backends, workers, libraries): plan a manual
   structure from the ecosystem's documented conventions — the official docs'
   recommended layout, not folklore. Note the sources in scaffold-notes.

Preflight comes first: required runtimes and tools at the versions the
generators need, Docker if the local data story needs it. Anything missing is
a blocking ask with exact install guidance — never silently substituted.

## Generation (Phase 4, first half)

Run generators **for real**, one unit at a time, checkpointing after each
(see checkpointing.md — a failed generator mid-run must not orphan the units
already built). Prefer non-interactive flags where the generator supports
them; where it insists on prompts, answer from the build contract and record
the answers. Never pipe blind defaults into prompts that encode architecture
decisions.

## Structure (Phase 4, second half — the pipeline's layer)

What generators don't do and this skill owns:

- **Monorepo arrangement**: units under a conventional layout (e.g.,
  `apps/<unit>` + `packages/<shared>` or the workspace tool's convention),
  workspace config at the root.
- **Module boundaries made enforceable**: the architecture's internal seams
  become folder structure *plus* import rules — lint constraints, dependency-
  check config, or the ecosystem's equivalent — so "clean seams" is checked by
  tooling, not discipline. A boundary that isn't enforced will erode.
- **Strip conflicting boilerplate**: generators ship demo pages, example
  routes, and default styling that contradict the architecture or design
  system. Remove what conflicts; keep what's neutral. Every removal noted in
  scaffold-notes (future sessions shouldn't wonder where the demo page went).
- **Carry `docs/` into the repo** — the pipeline documents move in (or are
  confirmed already in place if scaffolding runs inside the existing project
  folder), so the repo is self-describing.
- **Agent-instructions file** (CLAUDE.md or equivalent): points at the
  pipeline docs (srs, architecture, plan), **design.md and tokens.json** (so
  every UI-building session inherits the design system), the repo's run/test
  commands, and the boundary rules. Keep it short and pointer-heavy — it's an
  index, not a copy.

## Scaffold-notes discipline

`docs/scaffold-notes.md` is the project's own record, written as you go, never
retrofitted: environment preflight results; per-unit generator + version +
flags + prompt answers; deviations (where live docs disagreed with
expectation, what was done); boilerplate removed; decisions made in Phase 2
gaps. **Durable project facts, not chatter.** It lives in the project because
skills are read-only programs and projects are where state lives — if the
skill's author later wants to promote a durable ecosystem lesson into the
skill itself, that's a manual authoring act on the skills repo, not this
skill's job.
