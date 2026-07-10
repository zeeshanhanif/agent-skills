# Claude Skills

A collection of [Agent Skills](https://agentskills.io) for Claude Code (and other
agents that follow the open `SKILL.md` standard).

## Skills

| Skill | What it does |
| :---- | :----------- |
| [`requirements-engineering`](./skills/requirements-engineering) | The first SDLC step. Through an exhaustive, area-by-area interview it specifies the complete requirements, then produces a structured SRS (IEEE 29148 lineage), a use-case document, and a traceability matrix (markdown). Checkpoints so long sessions can resume, and later amends a finalized SRS with stable, never-recycled IDs. |
| [`software-architecture`](./skills/software-architecture) | Interviews you about a new application, then produces a right-sized architecture document with C4 diagrams (Mermaid) and Architecture Decision Records. |
| [`ux-foundations`](./skills/ux-foundations) | Reads your SRS + architecture, then acquires a visual direction (research, reference images, an existing design file, or a connected tool like Figma) and produces the UX foundations (personas, IA, navigation, flows, screen inventory per surface), an agent-ready `design.md`, and canonical `tokens.json` (W3C DTCG). |
| [`implementation-planning`](./skills/implementation-planning) | Reads the architecture and UX-foundations docs and produces a sequenced build plan — epics and vertical feature slices, the walking skeleton, a dependency/risk-ordered sequence, and the first-slice spec. Stops at the plan; detailed design and code are downstream. |

Run back-to-back, they form a greenfield requirements-to-build pipeline:
`requirements-engineering` → `software-architecture` → `ux-foundations` → `implementation-planning`.

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
  the single source of truth the downstream skills read.
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
  information architecture and navigation, key user flows, screen inventories,
  and cross-surface reconciliation, with **sitemaps and flows as Mermaid**. This
  is what implementation-planning consumes; it *references* the design system
  rather than restating it.
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

The bridge from design to construction. It reads the two design-phase documents —
the architecture and the UX foundations — and **turns them into a sequenced,
executable build plan** instead of a wish-list. Two principles govern it: **slice
vertically** (every unit of work cuts through UI, API, domain, and data to deliver
something demonstrable — never "build all the tables"), and **make the
architecture executable before complete** (build the *walking skeleton* first, not
the easiest feature, then sequence the rest by dependency and risk).

**What you get**
- A brief priorities check for what isn't already in the documents (what "first"
  should optimize for, any deadline or MVP cut line, team capacity).
- An **epic + feature breakdown** — thin vertical slices, each mapped to the
  screens it touches (from ux-foundations) and the building blocks/endpoints it
  needs (from the architecture).
- A **walking-skeleton** definition: the minimal end-to-end path that proves the
  system runs and deploys, with what's real vs. stubbed called out.
- A **risk- and dependency-ordered sequence** with a Mermaid dependency graph,
  the **first vertical slice** specified (acceptance criteria, screens,
  endpoints), and an **engineering-foundations** checklist.

It plans the whole app's breadth but only details what's next — full depth on the
first/near slices, coarse further out (depth-on-demand, not the waterfall trap).

**Outputs** default to a single `docs/implementation-plan.md`; on request the same
features can additionally be emitted as paste/import-ready issues (GitHub / Linear
/ Jira).

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
├── SKILL.md                        # 8-phase workflow + triggering
└── references/
    ├── slicing-guide.md            # epics + thin vertical feature slices
    ├── sequencing-guide.md         # walking skeleton, dependency/risk order
    ├── document-template.md        # plan structure, right-sizing rules
    └── diagram-guide.md            # dependency graph in Mermaid
```

---

## License

[MIT](./LICENSE) — use it, fork it, adapt it.
