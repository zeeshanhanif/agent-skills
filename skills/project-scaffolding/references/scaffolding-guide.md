# Scaffolding Guide

Mechanics for Phases 1–4: extracting the build contract, discovering tooling,
generating, and structuring. The philosophy throughout: **the skill encodes
process, not stack structures** — the executing agent's knowledge plus live
verification plus official generators supply the per-ecosystem specifics.

## Installation discipline

Anything the scaffolding process installs goes **project-local first**:
dev dependencies in the unit's manifest (pinned by the lockfile), or a
one-off runner (`npx`, `uvx`, and equivalents) for tools used once. Reason:
a globally installed CLI at version X on this machine and version Y on CI
produces different output from the same repo — project-local tooling is
reproducible everywhere and doesn't mutate a machine the skill doesn't own.

If something genuinely cannot be local, **ask first**: name the tool, the
exact install command, and why a local install won't serve. Never install
globally on the skill's own initiative. Prerequisites that are ambient by
nature — the language runtime, the package manager, Docker — aren't this
rule's concern; they're preflight checks, and missing ones are blocking asks
with install guidance (above).

## Extracting the build contract (Phase 1)

From the architecture: each **container** in the container diagram becomes a
deployable unit. For each, pull: language/framework (from the ADR that chose
it — cite it), the store(s) it binds to, and the module boundaries inside it
(the sub-domain seams the ADRs promised). From the architecture's
cross-cutting **Testing** entry: the named unit/integration runner per unit,
the E2E framework, and the coverage stance (with their ADR ref if any) —
these are realized in Phase 6, never re-decided. From the plan: the walking-skeleton
spec verbatim (what's real, what's stubbed, done-when) and the foundations
checklist. From ux-foundations/design.md/tokens.json: which units are UI
surfaces and therefore get the token wiring.

The **scaffold plan playback** is one tight block: units → stacks (with ADR
refs), repo shape, **the local stack** (the store's compose service and
pinned version — or the recorded deviation for emulator/embedded stores —
plus which non-store dependencies go in compose versus expected on the
machine), CI target, done-when. One confirmation, then execute.

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
2. **Verify against live documentation before first use — always, not only
   when uncertain.** Generator names, flags, and defaults change between
   versions; memory of a CLI's interface is the least reliable knowledge
   there is, and confidence in it is exactly the signal that can't be
   trusted. One lookup removes the judgment call. Use the **current** release
   unless the architecture pinned a version; live reality always wins over
   both memory and any written note.
3. Record into `docs/scaffold-notes.md` *before running*: generator, version,
   the exact flags chosen and why (interactive prompts answered how, and on
   what basis — architecture constraint, user answer, or default).
4. **No generator exists** (some backends, workers, libraries): plan a manual
   structure from the ecosystem's documented conventions — the official docs'
   recommended layout, not folklore. Note the sources in scaffold-notes.

Preflight comes first: required runtimes and tools at the versions the
generators need, and **Docker when the confirmed plan uses compose**.
Anything missing is
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
- **READMEs — replace generator boilerplate, everywhere it appears.** Every
  generator leaves a starter README describing *a template*, not this system;
  it is misleading from the moment it lands. **Root README (always)**: what
  the system is (from the SRS/architecture), the units and where each lives,
  prerequisites, how to bring up the local stack, how to run the tests, and
  where the pipeline docs are. **Per-unit README (only where the unit has
  something of its own to say** — deployable apps usually do, thin internal
  packages usually don't; no empty ceremony files): that unit's purpose in a
  line, its own run/test/build commands, its config template. Unit READMEs
  **link up to the root** for anything system-wide rather than restating it —
  duplicated content is what drifts. A single-unit project at the root
  collapses to one README, no duplication to manage.
- **Carry `docs/` into the repo** — the pipeline documents move in (or are
  confirmed already in place if scaffolding runs inside the existing project
  folder), so the repo is self-describing.
- **Agent instructions — `AGENTS.md` at the root, with `CLAUDE.md` as a
  one-line pointer to it** (`@AGENTS.md`). AGENTS.md holds all the substance;
  CLAUDE.md holds nothing but the reference. Reason: AGENTS.md is the
  cross-tool convention several coding agents read, so one file serves every
  tool and there is exactly one source of truth — two files with overlapping
  content inevitably diverge. AGENTS.md points at the
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
