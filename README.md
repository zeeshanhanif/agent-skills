# Claude Skills

A collection of [Agent Skills](https://agentskills.io) for Claude Code (and other
agents that follow the open `SKILL.md` standard).

## Skills

| Skill | What it does |
| :---- | :----------- |
| [`requirements-engineering`](./skills/requirements-engineering) | The first SDLC step. Through an exhaustive, area-by-area interview it specifies the complete requirements, then produces a structured SRS (IEEE 29148 lineage, functional requirements in EARS syntax or classic "shall" statements — your choice), a use-case document, and a traceability matrix (markdown). Checkpoints so long sessions can resume, and later amends a finalized SRS with stable, never-recycled IDs. |
| [`software-architecture`](./skills/software-architecture) | Interviews you about a new application, then produces a right-sized architecture document with C4 diagrams (Mermaid) and Architecture Decision Records. |
| [`ux-foundations`](./skills/ux-foundations) | Reads your SRS + architecture, then acquires a visual direction (research, reference images, an existing design file, or a connected tool like Figma) and produces the UX foundations (personas, IA, navigation, flows, screen inventory per surface), an agent-ready `design.md`, and canonical `tokens.json` (W3C DTCG). |
| [`implementation-planning`](./skills/implementation-planning) | Reads the full pipeline (SRS, use cases, architecture, UX-foundations) and produces a sequenced build plan — epics and vertical feature slices (each tracing its FR/UC/screen IDs), the walking skeleton, a dependency/risk-ordered sequence, a Must-requirement coverage check, and the first-slice spec. Stops at the plan; detailed design and code are downstream. |
| [`project-scaffolding`](./skills/project-scaffolding) | Turns the design docs into a **running system**: runs each ecosystem's official generator, wires the walking skeleton end-to-end (UI shell with your tokens → API → domain → local DB), stands up CI/test/deploy config, and verifies by building and running it. Deploy-ready, not deployed. |
| [`detailed-design`](./skills/detailed-design) | A **per-feature** skill in the construction loop — the *system* half. For one vertical slice at a time, it reads the plan + requirements + **the live codebase** and produces the feature's technical design (API contracts, schema migrations, component design, acceptance criteria) and an ordered `tasks.md`. Hands contracts to UI design and tasks to feature-implementation. |
| [`ui-design`](./skills/ui-design) | The **presentation** half of each feature's low-level design, and detailed-design's loop sibling. Resolves a strategy **per screen** — pick up screens that already exist in a connected design tool (Figma, Claude Design, …) or fall back to code-native specs / generation — and emits one uniform, SCR-keyed `design-manifest.json` that downstream implementation reads regardless of tool. Screens conform to your design system or escalate — never fork it. |
| [`feature-implementation`](./skills/feature-implementation) | The **construction step** of the loop: executes a feature's `tasks.md` against its designs, autonomously — one task at a time in order, each done-when demonstrated by actually running it, one commit per task, tests never weakened or skipped to pass. Bounded fix attempts (default 3), honest WIP stops, escalation instead of quiet redesign. Ends **developer-done**; the independent audit is acceptance-verification's job. |
| [`acceptance-verification`](./skills/acceptance-verification) | The **independent auditor** that closes the per-feature loop. After feature-implementation declares a feature developer-done, it re-derives the audit standard from the documents (never from checkboxes or claimed greens), audits and corrects weak tests, re-runs every suite fresh, verifies requirements directly, and delivers a verdict — **accepted / rework / design defect** — writing the feature's acceptance report and, on acceptance, the RTM's Test ref. Never fixes production code. |

The first five run once, back-to-back, as a greenfield requirements-to-running-skeleton pipeline; the last four then run **once per feature** in a construction loop, in this order:
`requirements-engineering` → `software-architecture` → `ux-foundations` → `implementation-planning` → `project-scaffolding` → **`detailed-design` → `ui-design` → `feature-implementation` → `acceptance-verification` (per feature) → …**

## Install

**Method 1 — the `skills` CLI (recommended).** One command, and it works across
Claude Code, Cursor, GitHub Copilot, and other agents the
[`skills`](https://www.skills.sh) CLI supports:

```bash
npx skills add https://github.com/zeeshanhanif/agent-skills --skill software-architecture
```

The CLI finds the skill because it lives at `skills/software-architecture/SKILL.md`
and the `--skill` value matches the `name:` in its frontmatter. Install any other
skill the same way by swapping the `--skill` value (e.g. `--skill ux-foundations`).

**Method 2 — manual copy (Claude Code).** Prefer this only if you'd rather not use
the CLI; see [manual installation](#install-claude-code) below.

Once installed, [use it](#use) by describing what you're building or running the
skill's slash command (e.g. `/software-architecture`).

---

## `requirements-engineering`

The very front of the SDLC — it runs **before** architecture and UX, and owns the
problem space comprehensively. Instead of transcribing what you mention, it
**proactively enumerates the standard sub-requirements** for each capability area
and has you confirm, extend, or trim them: "user authentication" is never one line
— it expands into sign-up, sign-in, email verification, forgot/reset password,
logout, session expiry, lockout, rate limiting, and so on. The output is a formal,
traditional specification (numbered, uniquely-identified, testable requirements),
not an agile backlog.

**What you get**
- An exhaustive, area-by-area interview driven by a requirement catalog that also
  prompts the commonly-forgotten areas (audit logging, account deletion, rate
  limiting, admin/back-office) and walks the ISO 25010 quality model for
  **measurable** non-functional requirements.
- A structured **SRS** (ISO/IEC/IEEE 29148 / IEEE 830 lineage) as `docs/srs.md` —
  the single source of truth the downstream skills read. Functional requirements
  are written in **EARS** (five constrained sentence patterns — event-driven,
  state-driven, optional-feature, unwanted-behavior, ubiquitous) or classic
  free-form "shall" statements; you pick once and it holds across the spec and
  every amendment.
- A separate **use-case document** (detailed textual specs + a Mermaid use-case
  diagram) as `docs/use-cases.md`.
- A **requirements traceability matrix** (`docs/rtm.md`) linking every requirement
  ID to its source and the use cases that exercise it.

All three outputs are markdown (convert to Word/PDF yourself if you need it).

It **checkpoints incrementally** (`docs/.requirements-progress.md`) and resumes
after an interruption, because a full requirements interview is long. Epic/feature
slicing is deliberately deferred to `implementation-planning`; design decisions to
the design-phase skills.

**It also amends a finalized SRS.** Beyond initial authoring, the skill detects on
startup whether to start fresh, resume an interrupted interview, or **amend** an
existing finalized spec — no flag needed, inferred from what's on disk plus your
plain-language intent ("add a requirement", "update FR-AUTH-007", "remove…"), and
it confirms before touching a finalized SRS rather than guessing. Amendments follow
one cardinal rule — **requirement IDs are immutable and never recycled**: adds take
the next free ID, modifies keep the ID, and removals are *tombstoned*
(`Deprecated`/`Removed`, kept in place) instead of deleted or renumbered, so
traceability holds. Each change bumps the SRS version with a revision-history entry,
propagates to the affected use cases and the RTM, and emits a **cross-skill impact
note** flagging which downstream documents (architecture, UX, plan) referenced the
changed IDs.

> Install with `npx skills add ... --skill requirements-engineering`, or copy it in
> by hand following [Manual install (Claude Code)](#install-claude-code) below
> (swap `software-architecture` for `requirements-engineering`).

### Use

Run it at project kickoff, before any design — let Claude trigger it automatically:
```text
I'm kicking off a new project — help me gather and write up the requirements.
```
or invoke it directly:
```text
/requirements-engineering
```
Later, to change an existing spec, just say what you want — it picks up the
finalized SRS and switches to amendment mode:
```text
Add a "sign in with Google" requirement and remove FR-AUTH-009.
```

### What's inside

```text
skills/requirements-engineering/
├── SKILL.md                        # authoring + amendment workflow, checkpointed
└── references/
    ├── elicitation-guide.md        # the area-by-area interview
    ├── requirement-catalog.md      # the enumeration engine (FR areas + ISO 25010 NFRs)
    ├── ears-guide.md               # EARS syntax — the five FR sentence patterns
    ├── use-case-guide.md           # deriving + specifying use cases
    ├── rtm-guide.md                # building the traceability matrix
    ├── srs-template.md             # IEEE 29148-lineage SRS structure
    ├── checkpointing.md            # incremental save, resume + mode detection
    └── change-management.md        # amending a finalized SRS (stable IDs, tombstones)
```

---

## `software-architecture`

Turns a vague "I want to build X" into a grounded architecture. Instead of jumping
straight to a tech stack, it **elicits the quality attributes and constraints that
actually drive architectural decisions** — scale, availability, consistency,
security, team, timeline, lock-in tolerance — and *derives* the design from your
answers. Then it writes the document, draws the diagrams, and records each
significant decision (with the options it weighed and why) as an ADR.

**It designs from the requirements docs when they exist.** If `docs/srs.md` is
present (from `requirements-engineering`), it reads the requirements and NFRs as
the primary source, plays back the architecture drivers, and interviews only for
the gaps the SRS doesn't cover (tech/stack preferences, lock-in tolerance, team
skills, deployment target) — no re-interview. If `docs/use-cases.md` is present it
also mines them for the **runtime view** (each significant use case becomes a
sequence diagram) and for **resilience and security** decisions (exception flows
reveal failure handling; actors reveal trust boundaries). With no SRS it runs the
full interview and works standalone. The SRS states *what*; architecture still
decides *how* (stack, stores, deployment).

**What you get**
- A structured interview — full when there's no SRS, otherwise just the gaps —
  asked in logical batches (it infers what it can and offers defaults you can
  accept with one word).
- A Markdown architecture document based on the **arc42** template, right-sized to
  the system — a CRUD app gets a tight doc, a payment platform gets the full
  treatment.
- **C4-model diagrams** as embedded Mermaid (System Context + Container by default,
  plus sequence/deployment diagrams when warranted), so the whole thing is one
  portable, diff-able artifact that renders on GitHub.
- **Architecture Decision Records** capturing the *why*, not just the *what*. When
  the requirements docs are present, each ADR cites the **SRS requirement IDs**
  (and any use-case IDs) it addresses so traceability runs both ways; when they're
  absent it names the driver in prose instead — it never invents an ID.

**Outputs** default to a single `docs/architecture.md`; it can also export a `.docx`
for stakeholders, or split ADRs into a `docs/adr/` log on request.

<a id="install-claude-code"></a>
### Manual install (Claude Code)

Prefer `npx skills add` (see [Install](#install) above). To copy the skill in by
hand instead:

**Personal — available in all your projects:**
```bash
mkdir -p ~/.claude/skills
git clone https://github.com/zeeshanhanif/agent-skills.git /tmp/claude-skills
cp -R /tmp/claude-skills/skills/software-architecture ~/.claude/skills/
# verify: you should see SKILL.md directly inside the folder
ls ~/.claude/skills/software-architecture/
```

**Project-scoped — committed to a specific repo so teammates get it too:**
```bash
mkdir -p .claude/skills
cp -R /path/to/skills/software-architecture .claude/skills/
git add .claude/skills/software-architecture && git commit -m "Add software-architecture skill"
```

> **Watch the nesting.** The path must be
> `~/.claude/skills/software-architecture/SKILL.md` — not one level deeper. If your
> copy created `software-architecture/software-architecture/SKILL.md`, flatten it.

If `~/.claude/skills/` already existed when your session started, the new skill is
picked up live. If you just created that directory, restart Claude Code once so it
starts watching it.

<a id="use"></a>
### Use

Two ways in, depending on whether you ran `requirements-engineering` first.

**In the pipeline** — with a `docs/srs.md` already in the repo, it designs from
those requirements and only asks about the gaps:
```text
The requirements are in docs/srs.md — now design the architecture.
```

**Standalone** — no SRS yet, so it runs the full interview:
```text
I'm building a multi-tenant inventory app for small retailers — help me architect it.
```

Either way you can also invoke it directly — it auto-detects `docs/srs.md` (and
`docs/use-cases.md`) if present, and falls back to the full interview if not:
```text
/software-architecture
```

Confirm it loaded with `/skills` or by asking "what skills are available?".

### What's inside

```text
skills/software-architecture/
├── SKILL.md                        # workflow + triggering
└── references/
    ├── elicitation-guide.md        # the interview framework
    ├── decision-guide.md           # reasoning through the recurring forks
    ├── document-template.md        # arc42-based, right-sizing rules
    ├── diagram-guide.md            # C4 + Mermaid, validated examples
    └── adr-template.md             # decision-record format + example
```

---

## `ux-foundations`

The "architecture of the UI" — the design-phase sibling to
`software-architecture`. It reads your SRS and architecture, then **derives the
structure and design language of the UI** instead of jumping to pixel-perfect
screens. The governing idea: **one shared core, defined once, plus a per-surface
layer for each interface**, so a multi-surface product (admin portal, website,
mobile app) feels like one thing without flattening the differences between
surfaces.

It draws personas straight from the SRS's user classes and treats the SRS's
accessibility NFRs as a **non-negotiable bar** that overrides any imported
design. The visual direction comes from one of **four source modes** you pick — it
always asks, never assumes:

- **Research** a new direction (comparable products, current conventions, your
  audience) when there's no existing design;
- **Extract from reference images** you drop in `docs/design-refs/`;
- **Ingest an existing design file** or brand book and map it onto the structure;
- **Connect a design tool** (e.g. Figma via MCP) and pull tokens/styles/components.

**What you get** — three outputs with a strict source-of-truth split:
- `docs/ux-foundations.md` — the **plan-time** document: personas, per-surface
  information architecture and navigation, key user flows, screen inventories
  (each screen with a stable **SCR ID**), and cross-surface reconciliation, with
  **sitemaps and flows as Mermaid**. This is what implementation-planning consumes
  (slices trace those SCR IDs); it *references* the design system rather than
  restating it.
- `docs/design.md` — the **render-time**, agent-ready design system a coding
  agent loads when building UI: tokens (with inline CSS variables), component
  specs with states and variants, layout/accessibility/usage rules, and
  provenance. Reference it from your `CLAUDE.md` so every UI session inherits it.
- `docs/tokens.json` — the canonical machine-readable tokens in **W3C DTCG
  format**, the single source of truth that `design.md`'s CSS is derived from.

> Install with `npx skills add ... --skill ux-foundations`, or copy it in by hand
> following [Manual install (Claude Code)](#install-claude-code) above (swap
> `software-architecture` for `ux-foundations`).

### Use

Run it after the architecture is settled — let Claude trigger it automatically
(it reads `docs/srs.md` + `docs/architecture.md` and asks how you want the design
sourced):
```text
The architecture's done — help me set up the UX foundations and design system.
```
or invoke it directly:
```text
/ux-foundations
```

### What's inside

```text
skills/ux-foundations/
├── SKILL.md                        # 8-phase workflow + triggering
└── references/
    ├── elicitation-guide.md        # UX interview; personas from the SRS
    ├── source-modes.md             # the 4 design-source acquisition modes
    ├── design-tool-integrations.md # Mode 4: pulling from Figma etc. via MCP
    ├── design-system-guide.md      # the shared core: tokens, a11y, voice
    ├── surface-profile-guide.md    # per-surface layer: IA, flows, components
    ├── design-md-guide.md          # the render-time, agent-ready design.md
    ├── document-template.md        # shared core + one section per surface
    └── diagram-guide.md            # sitemaps + user flows in Mermaid
```

---

## `implementation-planning`

The bridge from design to construction. It reads the **full pipeline** — the SRS,
use cases, architecture, and UX foundations — and **turns them into a sequenced,
executable build plan** instead of a wish-list. Two principles govern it: **slice
vertically** (every unit of work cuts through UI, API, domain, and data to deliver
something demonstrable — never "build all the tables"), and **make the
architecture executable before complete** (build the *walking skeleton* first, not
the easiest feature, then sequence the rest by dependency and risk).

**What you get**
- A brief priorities check for what isn't already in the documents (what "first"
  should optimize for, any deadline or MVP cut line, team capacity).
- An **epic + feature breakdown** — thin vertical slices, each with a stable
  **`FEAT-NNN` ID** (the join key the construction-loop skills and the RTM
  reference) and tracing the **FR IDs** it implements, the **UC IDs** it realizes,
  and the **screen (SCR) IDs** it touches, plus the building blocks/endpoints it
  needs. Tombstoned (removed) requirements and screens are skipped.
- A **walking-skeleton** definition: the minimal end-to-end path that proves the
  system runs and deploys, with what's real vs. stubbed called out.
- A **risk- and dependency-ordered sequence** with a Mermaid dependency graph,
  the **first vertical slice** specified (acceptance criteria, screens,
  endpoints), and an **engineering-foundations** checklist.
- A **Must-requirement coverage check** — every Must-priority requirement lands in
  a slice or is consciously deferred, so nothing silently falls through.

It plans the whole app's breadth but only details what's next — full depth on the
first/near slices, coarse further out (depth-on-demand, not the waterfall trap).

**Outputs** default to a single `docs/implementation-plan.md`; on request the same
features can additionally be emitted as paste/import-ready issues (GitHub / Linear
/ Jira). If a `docs/rtm.md` exists, it also fills in that matrix's **Plan** column
(which slice schedules each requirement).

> Install with `npx skills add ... --skill implementation-planning`, or copy it in
> by hand following [Manual install (Claude Code)](#install-claude-code) above
> (swap `software-architecture` for `implementation-planning`).

### Use

Run it after the architecture and UX foundations are settled — let Claude trigger
it automatically:
```text
Architecture and UX foundations are done — turn them into a build plan.
```
or invoke it directly:
```text
/implementation-planning
```

### What's inside

```text
skills/implementation-planning/
├── SKILL.md                        # 9-phase workflow + triggering
└── references/
    ├── slicing-guide.md            # epics + thin vertical feature slices
    ├── sequencing-guide.md         # walking skeleton, dependency/risk order
    ├── verification.md             # Must-requirement coverage + RTM write-back
    ├── document-template.md        # plan structure, right-sizing rules
    └── diagram-guide.md            # dependency graph in Mermaid
```

---

## `project-scaffolding`

The first skill whose output is a **running system, not a document**. It reads the
architecture and the implementation plan and turns the walking skeleton into a real
repo: generated structure, wired end-to-end, foundations in place, verified by
execution. It ends where deployment begins — everything is deploy-*ready*; the
first actual deploy is your step. Three principles govern it: **the stack is an
input, never a decision** (the architecture's ADRs already chose it — gaps go back
as candidate amendments); **official generators first** (it discovers each
ecosystem's current official initializer, verifies its flags against live docs, and
runs it for real, so the skill stays self-updating as frameworks change); and **it
owns the stack-independent layer** (the wiring between units, module boundaries, the
design system in the UI shell, and empirical verification).

**What you get**
- A **real repo** (monorepo by default) — one deployable unit per architecture
  container, module boundaries enforced with folder + import/lint rules, generator
  boilerplate reconciled with the architecture, and `docs/` carried in.
- A **wired walking skeleton**: UI shell (your `tokens.json` wired into each
  frontend, `design.md` referenced) → API → domain stub → local database → back,
  stubbed exactly where the plan said.
- **Engineering foundations** stood up: CI (lint/test/build), environment configs,
  observability hooks, a test harness with one end-to-end skeleton test, and
  deployment config *written but not executed*.
- **Empirical verification** — a clean install builds, the skeleton test passes
  locally, boundary rules hold, and tokens actually render; anything unfixable is
  flagged, never silently shipped.
- An **agent-instructions file** (CLAUDE.md or equivalent) pointing at the pipeline
  docs and `design.md`, plus `docs/scaffold-notes.md` recording the generators,
  versions, and flags actually used.

> Install with `npx skills add ... --skill project-scaffolding`, or copy it in by
> hand following [Manual install (Claude Code)](#install-claude-code) above (swap
> `software-architecture` for `project-scaffolding`).

### Use

Run it after the plan is settled — let Claude trigger it automatically:
```text
The plan's ready — scaffold the repo and stand up the walking skeleton.
```
or invoke it directly:
```text
/project-scaffolding
```

### What's inside

```text
skills/project-scaffolding/
├── SKILL.md                        # 9-phase workflow (checkpointed) + triggering
└── references/
    ├── checkpointing.md            # resume safely; never regenerate over a partial scaffold
    ├── scaffolding-guide.md        # official-generator discovery + repo structure
    ├── skeleton-guide.md           # wiring the skeleton + engineering foundations
    └── verification.md             # empirical checks: build it, run it, prove it
```

---

## `detailed-design`

The first **loop skill** — where the linear pipeline gives way to a per-feature
construction loop. It runs **once for each vertical slice** as that slice reaches
the front of the plan, turning the feature's planned intent into a buildable
technical design (the backend/system half; UI design is the presentation half and
consumes these contracts). Three principles govern it: **the *what* is fixed
upstream** (the FR/UC IDs were set in requirements and scoped by the plan — this
skill designs the *how*, never reinterprets requirements); **the live codebase is
an input, not an obstacle** (feature N is designed months after feature 1, against
code that has evolved past the skeleton — reality wins over the documents where
they diverge); and **depth here, and only here** (full concrete detail for *this*
feature, nothing for features that haven't reached the front — the pipeline's
depth-on-demand promise, kept).

**What you get** — per feature, written into `docs/features/FEAT-NNN-<slug>/`
(keyed on the feature's stable ID from the plan):
- **`technical-design.md`** — API contracts (endpoints, request/response shapes,
  error codes), physical **schema migrations** (within the entities the
  architecture already owns — a new entity escalates to an architecture amendment,
  never invented locally), component-level design, and testable acceptance
  criteria derived from the FR statements and UC flows.
- **`tasks.md`** — an ordered, implementable task breakdown (schema → domain →
  contract → wiring → an E2E-extension task when the feature touches an
  architecture-named critical flow → tests green), each task pointing at the
  design sections and acceptance criteria it serves — the program
  `feature-implementation` executes, ready for a fresh session or a loop agent.
- Two write-backs: the feature's design ref appended to the **RTM** Design column
  for every FR it implements, and any **architecture-amendment escalation** it had
  to file.

It picks the next feature automatically (plan build-order minus the feature folders
already designed) — execution status lives in the artifacts, never edited back into
the plan.

> Install with `npx skills add ... --skill detailed-design`, or copy it in by hand
> following [Manual install (Claude Code)](#install-claude-code) above (swap
> `software-architecture` for `detailed-design`).

### Use

Run it per feature, after the repo is scaffolded — let Claude trigger it
automatically:
```text
Design the next feature from the plan.
```
or name a specific one / invoke it directly:
```text
/detailed-design  →  design the sign-in feature (FEAT-004)
```

### What's inside

```text
skills/detailed-design/
├── SKILL.md                        # 6-phase per-feature workflow + triggering
└── references/
    ├── design-guide.md             # contracts, schema, components, acceptance criteria
    ├── tasks-guide.md              # decomposing the design into an ordered tasks.md
    └── verification.md             # self-check: coverage, ID resolution, no code collisions
```

---

## `ui-design`

The **presentation half** of each feature's low-level design — `detailed-design`'s
loop sibling, but sequential: in per-feature mode it consumes that feature's
`technical-design.md`, because a screen displays what an endpoint returns. Its
defining trait is that it's an **adapter**: how screens get designed varies wildly
by project (an existing Figma file, a generation tool, straight to code), so it
routes across strategies while emitting **one uniform contract** — the design
manifest — that everything downstream reads, whatever produced each screen. All
design-tool variance is absorbed here; no downstream skill ever learns what a Figma
node is.

Three principles govern it: **strategy is resolved per screen, not per project**
(the manifest lookup by SCR ID is always the first move — register screens that
already exist, fall back per policy for the rest); **screens conform to the design
system or escalate — never fork it** (`design.md` + `tokens.json` are the
authority; an off-system screen is corrected or the *system* is amended through
ux-foundations); and **the manifest is the only registry** (no tool ever holds the
complete picture — `docs/design-manifest.json` does, and this skill is its single
writer).

It runs in **three modes**:
- **Anchor** — right after ux-foundations, design 2–3 key screens to validate the
  design system *composes* before features build on it (recommended, not mandatory
  — the strength of the recommendation follows how your `design.md` was sourced).
- **Per-feature** — the construction loop: after `detailed-design`, design that
  feature's screens against its contracts.
- **Re-verification** — after a `design.md` amendment, re-check the screens the
  change affects.

**What you get**
- **`docs/design-manifest.json`** — the global, SCR-keyed screen registry: per
  screen, its strategy, source locator, states coverage, contract bindings,
  conformance result, and status. The single source downstream implementation
  reads — no matter which tool (if any) produced the screen.
- **`docs/features/FEAT-NNN-<slug>/ui-design.md`** (per-feature) or
  **`docs/anchor-screens.md`** (anchor mode, opening with the *does the system
  compose?* verdict) — the human-readable screen specs and decisions.
- An **RTM Design-ref append** for the FRs whose screens it designs, and any
  **design-system amendment escalations** filed toward ux-foundations.

> Install with `npx skills add ... --skill ui-design`, or copy it in by hand
> following [Manual install (Claude Code)](#install-claude-code) above (swap
> `software-architecture` for `ui-design`).

### Use

Either validate the system with anchor screens right after ux-foundations, or
design a feature's screens in the loop — let Claude trigger it automatically:
```text
Design anchor screens to check the design system holds up.
Design the next feature's screens.
```
or invoke it directly:
```text
/ui-design
```

### What's inside

```text
skills/ui-design/
├── SKILL.md                        # 3-mode, per-screen-routing workflow + triggering
└── references/
    ├── strategy-guide.md           # per-screen strategy: register / generate / code-native
    ├── design-tool-integrations.md # design tools as interchangeable capability-first engines
    ├── manifest-guide.md           # the SCR-keyed design-manifest.json schema
    ├── document-templates.md       # ui-design.md + anchor-screens.md (load-bearing headings)
    └── verification.md             # self-check: entries, conformance, states, RTM append
```

---

## `feature-implementation`

The **construction step** of the per-feature loop — where documents become code.
**`tasks.md` is the program; this skill is the interpreter.** After both design
halves exist (`technical-design.md` from detailed-design, `ui-design.md` + the
manifest from ui-design), it executes the feature's tasks autonomously, one at a
time in order, against the live codebase — producing working, tested, committed
code that replaces the skeleton's stubs and ends **developer-done**: every task
checked, including the final verification task. The independent audit is
downstream (`acceptance-verification`) — this skill never grades its own homework
alone.

Seven disciplines, each targeting a named failure mode of agentic implementation:
- **The program is fixed; improvisation is escalation** — when reality beats the
  design, small design-consistent fixes are recorded; anything that changes a
  contract, schema, acceptance criterion, or screen spec escalates to the
  design's amendment path. It implements the design; it never quietly redesigns.
- **Fresh-context safety** — disk is the memory: the checkbox state is the exact
  position, so any iteration can run in a brand-new session (or a loop agent).
- **Done-when demonstrated, never asserted** — a box flips only after the task's
  done-when actually ran and passed, plus the **anti-fake-green rule**: tests are
  never weakened, skipped, deleted, or edited to pass.
- **Bounded fix-loops** — at most 3 attempts per task (per-project override),
  then an honest stop: unchecked box, failure note, explicit WIP commit a fresh
  session can pick up cold.
- **Scope discipline** — this feature only; no drive-by refactors; tech-debt
  discoveries are recorded, never acted on.
- **Convention conformance** — test frameworks inherited from the scaffolded
  harness, UI built through the design system (tokens, never raw values), with a
  bounded screenshot loop for code-native screens.
- **Git as the checkpoint** — one commit per completed task
  (`FEAT-004 T3: implement POST /auth/login contract`); the log reads as the
  execution log of `tasks.md`.

It picks the next feature deterministically (the first in the plan's build order
whose `tasks.md` has unchecked tasks), refuses to start when the design documents
are missing (the design pair runs first), and writes **no RTM column** — Design
ref was written upstream; Test ref belongs to acceptance-verification.

> Install with `npx skills add ... --skill feature-implementation`, or copy it in
> by hand following [Manual install (Claude Code)](#install-claude-code) above
> (swap `software-architecture` for `feature-implementation`).

### Use

Run it per feature, after detailed-design and ui-design — let Claude trigger it
automatically:
```text
Implement the next feature from the plan.
```
or name one / invoke it directly:
```text
/feature-implementation  →  build FEAT-004
```

### What's inside

```text
skills/feature-implementation/
├── SKILL.md                        # the 7 disciplines + 5-phase autonomous task loop
└── references/
    ├── execution-guide.md          # the per-task cycle: implement, verify, commit
    ├── failure-and-escalation.md   # bounded fix-loops, anti-fake-green, divergence routing
    └── verification.md             # the developer-done gate, demonstrated not asserted
```

---

## `acceptance-verification`

The **independent auditor** that closes the per-feature loop — it turns
*developer-done* into **verified**. It runs after `feature-implementation`
finishes a feature (every `tasks.md` box checked) and answers, without the
implementer's investment: does this feature actually satisfy its requirements?
Auditor and mechanic stay separate — findings **route** (back to
feature-implementation as rework, or to the design's amendment path as defects)
and the skill **never fixes production code**. The one artifact class it may correct is the *measurement*: weak tests
that fail its audit.

Its core discipline is the **fresh-eyes rule**: every check is re-derived from the
authoritative documents — the technical design's acceptance criteria, the verbatim
FR/UC statements, the design manifest — never from `tasks.md` checkboxes, delivery
summaries, or any session's claims of green. Claims are exhibits; observations are
evidence.

**What you get**
- A **criterion-by-criterion test audit**: does a test cover each criterion, does
  it assert what the criterion actually says (weakened proxies rejected), is it
  really exercised — plus an anti-fake-green review of the feature's test diff
  (loosened assertions, new skips, mocked-away behavior). Rejected tests are
  **corrected toward the criterion**, never toward the code, in separate commits.
- **Independent execution** — every suite re-run fresh from the repo state with
  the harness's own commands (feature, whole-repo, E2E, migrations); no reported
  green is trusted, only observed green counts.
- **Direct requirement verification** beyond the tests: side-effect FRs observed
  (the audit log's entries inspected, not just a 200), binding NFRs measured where
  cheaply measurable (environment-caveated; infrastructure-needing NFRs recorded
  as *pending environment*, never silently skipped), and screens spot-checked
  against the design manifest.
- A **verdict** — **accepted** / **rework** / **design defect** — written to the
  feature folder as `acceptance-report.md` with the full source → criterion →
  test → observation chain; rework routes back to feature-implementation,
  defects to the design's amendment path.
- On acceptance, the **RTM Test-ref append** (with a permanent `(partial)` marker
  when the feature implements an FR partially) — completing each requirement's
  Plan ref → Design ref → Test ref lifecycle.

It picks the next feature deterministically (the first in the plan's build order
that is developer-done but has no accepted report), and re-verification after
rework or an amendment is a normal run — the report gains a new dated verdict,
prior verdicts preserved.

> Install with `npx skills add ... --skill acceptance-verification`, or copy it in
> by hand following [Manual install (Claude Code)](#install-claude-code) above
> (swap `software-architecture` for `acceptance-verification`).

### Use

Run it per feature, once implementation says it's done — let Claude trigger it
automatically:
```text
FEAT-004 is implemented — verify it's really done.
```
or invoke it directly:
```text
/acceptance-verification
```

### What's inside

```text
skills/acceptance-verification/
├── SKILL.md                        # 6-phase audit workflow + triggering
└── references/
    ├── audit-guide.md              # standard re-derivation, test audit, anti-fake-green, execution
    └── verdict-and-report.md       # three verdicts, acceptance-report.md, RTM Test ref rules
```

---

## License

[MIT](./LICENSE) — use it, fork it, adapt it.
