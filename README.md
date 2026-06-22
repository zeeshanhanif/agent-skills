# Claude Skills

A collection of [Agent Skills](https://agentskills.io) for Claude Code (and other
agents that follow the open `SKILL.md` standard).

## Skills

| Skill | What it does |
| :---- | :----------- |
| [`software-architecture`](./skills/software-architecture) | Interviews you about a new application, then produces a right-sized architecture document with C4 diagrams (Mermaid) and Architecture Decision Records. |
| [`ux-foundations`](./skills/ux-foundations) | Reads your architecture document, then produces the UX foundations — a shared design core (brand, tokens, accessibility, voice) plus a per-surface profile (IA, navigation, key flows, screen inventory) for each UI surface. |
| [`implementation-planning`](./skills/implementation-planning) | Reads the architecture and UX-foundations docs and produces a sequenced build plan — epics and vertical feature slices, the walking skeleton, a dependency/risk-ordered sequence, and the first-slice spec. Stops at the plan; detailed design and code are downstream. |

Run back-to-back, they form a greenfield design-to-build pipeline:
`software-architecture` → `ux-foundations` → `implementation-planning`.

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

## `software-architecture`

Turns a vague "I want to build X" into a grounded architecture. Instead of jumping
straight to a tech stack, it **elicits the quality attributes and constraints that
actually drive architectural decisions** — scale, availability, consistency,
security, team, timeline, lock-in tolerance — and *derives* the design from your
answers. Then it writes the document, draws the diagrams, and records each
significant decision (with the options it weighed and why) as an ADR.

**What you get**
- A structured interview, asked in logical batches (it infers what it can and
  offers defaults you can accept with one word).
- A Markdown architecture document based on the **arc42** template, right-sized to
  the system — a CRUD app gets a tight doc, a payment platform gets the full
  treatment.
- **C4-model diagrams** as embedded Mermaid (System Context + Container by default,
  plus sequence/deployment diagrams when warranted), so the whole thing is one
  portable, diff-able artifact that renders on GitHub.
- **Architecture Decision Records** capturing the *why*, not just the *what*.

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

In a Claude Code session, either let Claude trigger it automatically:
```text
I'm building a multi-tenant inventory app for small retailers — help me architect it.
```
or invoke it directly:
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
`software-architecture`. It reads your finalized architecture document, then
**derives the structure and design language of the UI** instead of jumping to
pixel-perfect screens. The governing idea: **one shared core, defined once, plus
a per-surface layer for each interface**, so a multi-surface product (admin
portal, website, mobile app) feels like one thing without flattening the
differences between surfaces.

**What you get**
- A short interview for what the architecture doesn't cover (brand, voice,
  accessibility bar, visual direction, per-surface users and device targets) —
  inferred aggressively from the architecture, with defaults you can accept in a
  word.
- A **shared design core** defined once: brand and voice, **design tokens**
  (color, typography, spacing, radius, iconography), the accessibility standard,
  and cross-surface interaction principles.
- A **per-surface profile** for each UI surface: its users and jobs, information
  architecture and navigation, key user flows, a screen/page inventory, and the
  surface-specific components and token overrides it needs.
- **Sitemaps and user flows** embedded as Mermaid, so the whole thing is one
  portable artifact that renders on GitHub.

**Outputs** default to a single `docs/ux-foundations.md`; on request it can also
emit a real design-tokens file (`tokens.css` / `tokens.json`) or one or two
anchor screen mockups to settle the visual direction.

> Install with `npx skills add ... --skill ux-foundations`, or copy it in by hand
> following [Manual install (Claude Code)](#install-claude-code) above (swap
> `software-architecture` for `ux-foundations`).

### Use

Run it after the architecture is settled — let Claude trigger it automatically:
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
├── SKILL.md                        # 7-phase workflow + triggering
└── references/
    ├── elicitation-guide.md        # UX interview, infer from architecture first
    ├── design-system-guide.md      # the shared core: tokens, a11y, voice
    ├── surface-profile-guide.md    # per-surface layer: IA, flows, components
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
