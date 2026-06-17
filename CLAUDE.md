# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of **Agent Skills** following the open `SKILL.md` standard (see https://agentskills.io). There is no application code, build system, test suite, or linter — the deliverable is prompt/instruction content that Claude (or another agent) loads at runtime. "Correctness" here means the instructions are clear, well-scoped, and trigger reliably, not that anything compiles.

Each skill lives under `skills/` as its own directory containing a `SKILL.md` and (optionally) a `references/` folder of supporting guides. Currently the only skill is `skills/software-architecture/`.

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

## When editing the `software-architecture` skill

Its governing principle (stated in `SKILL.md`): **architecture is driven by quality attributes and constraints, not by technology** — the workflow enforces *understand → decide → document*, and every tech choice must trace back to a stated requirement. Keep edits aligned with this; don't add guidance that jumps to a stack before eliciting needs.

The references form a pipeline matching the SKILL's five phases — keep them mutually consistent when changing one:
- `elicitation-guide.md` — the interview rounds (Phase 1). Each question must be one whose answer would change the design.
- `decision-guide.md` — how to reason through the recurring forks (Phase 2). Meta-rule: start as simple as requirements allow.
- `document-template.md` — arc42-based output, with sections tagged `[essential]` vs `[scale]` for right-sizing (Phase 3).
- `diagram-guide.md` — C4 model framing, rendered as inline Mermaid; **prefer plain `flowchart`/`graph` syntax over experimental Mermaid C4 syntax** for portability (Phase 3).
- `adr-template.md` — one ADR per significant (costly-to-reverse) decision (Phase 4).

## Validating changes

There's nothing to build or run. To sanity-check a skill:
- Confirm `SKILL.md` frontmatter `name` matches the directory name.
- Verify every `references/...` path mentioned in `SKILL.md` actually exists.
- Render any Mermaid you add (GitHub preview or a Mermaid live editor) — the diagram guide's reliability notes exist because broken diagrams are the common failure.
