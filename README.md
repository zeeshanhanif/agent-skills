# Claude Skills

A collection of [Agent Skills](https://agentskills.io) for Claude Code (and other
agents that follow the open `SKILL.md` standard).

## Skills

| Skill | What it does |
| :---- | :----------- |
| [`software-architecture`](./skills/software-architecture) | Interviews you about a new application, then produces a right-sized architecture document with C4 diagrams (Mermaid) and Architecture Decision Records. |

## Install

**Method 1 — the `skills` CLI (recommended).** One command, and it works across
Claude Code, Cursor, GitHub Copilot, and other agents the
[`skills`](https://www.skills.sh) CLI supports:

```bash
npx skills add https://github.com/zeeshanhanif/agent-skills --skill software-architecture
```

The CLI finds the skill because it lives at `skills/software-architecture/SKILL.md`
and the `--skill` value matches the `name:` in its frontmatter.

**Method 2 — manual copy (Claude Code).** Prefer this only if you'd rather not use
the CLI; see [manual installation](#install-claude-code) below.

Once installed, [use it](#use) by describing what you're building or running
`/software-architecture`.

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

## License

[MIT](./LICENSE) — use it, fork it, adapt it.
