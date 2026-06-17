---
name: software-architecture
description: >-
  Designs the software architecture for a new application, service, or major
  feature through a structured interview, then produces an architecture
  document (arc42-style, with C4 diagrams in Mermaid and Architecture Decision
  Records). Use this whenever the user is starting a greenfield system or a
  significant new feature and would benefit from thinking through structure,
  quality attributes, tech stack, and trade-offs before writing code. Trigger
  on phrases like "design the architecture for", "I'm building a new app",
  "help me architect", "set up a new project/service", "what should the
  architecture be", "create an architecture doc/diagram", or any greenfield
  system-design discussion — even when the user does not say the word
  "architecture" explicitly but is clearly about to start building something
  new. Prefer this skill over answering from memory whenever real architectural
  trade-offs are in play.
---

# Software Architecture

This skill turns a vague "I want to build X" into a grounded architecture: it
**elicits the things that actually drive architectural decisions**, then
produces a right-sized architecture document with C4 diagrams and decision
records that capture the *why*, not just the *what*.

The single most important idea: **architecture is driven by quality attributes
and constraints, not by technology.** A junior approach jumps straight to "use
Next.js + Postgres on AWS." A senior approach first asks "how many users, how
much can it be down, how consistent must the data be, what's the team, what's
the lock-in tolerance" — and *derives* the stack from the answers. This skill
enforces that order: **understand → decide → document.**

## Workflow

Follow these phases in order. Do not skip the interview — generating a document
from assumptions is the failure mode this skill exists to prevent.

### Phase 1 — Frame and elicit

1. Open with one or two sentences confirming what's being built, then explain
   you'll ask a few rounds of questions so the architecture fits the real
   requirements. Keep it light; this is a conversation, not a form.

2. Read `references/elicitation-guide.md` and run the interview. Key rules:
   - **Ask in thematic batches, not one-at-a-time and not all-at-once.** Group
     related questions so the user can answer a coherent chunk, then move on.
   - **Infer aggressively and state your inferences.** If the user said "an
     internal admin dashboard for 20 staff," you already know scale is small,
     availability needs are modest, and you don't need a global CDN — say so and
     let them correct you rather than asking.
   - **Offer defaults the user can accept with one word.** "I'll assume a
     relational store unless your data is more document-shaped — sound right?"
   - **Let them dump.** If the user pastes a big brief, extract every answer you
     can from it first and only ask about genuine gaps.
   - **Stop when you have enough to decide, not when you've asked everything.**
     A small app needs far fewer answers than a multi-tenant SaaS.

3. Before moving on, play back a short summary of what you heard and the key
   constraints/quality attributes you'll be designing toward. Get a confirable
   nod. This catches misunderstandings before they're baked into a document.

### Phase 2 — Decide

Derive the architecture from what you learned. Read
`references/decision-guide.md` for how to reason about the recurring forks
(monolith vs services, sync vs async, SQL vs NoSQL, serverless vs containers,
vendor-managed vs portable). For each significant fork, you will write an ADR in
Phase 4, so note the options you weighed and *why* you chose as you did. Resist
the urge to over-engineer: the right architecture for most new apps is simpler
than the architecture people reach for.

### Phase 3 — Document and diagram

1. Read `references/document-template.md` and write the architecture document.
   **Right-size it** — the template marks which sections are essential and which
   scale with complexity. A small app gets a tight doc; a distributed system
   gets the full treatment.

2. Read `references/diagram-guide.md` and produce the diagrams as **Mermaid
   embedded directly in the document**, so the whole thing is one portable
   artifact. At minimum produce a **System Context** diagram and a **Container**
   diagram (the two C4 levels that earn their keep almost every time). Add a
   sequence diagram for the most important runtime flow, and a deployment
   diagram when the infrastructure is non-trivial.

### Phase 4 — Capture decisions

Read `references/adr-template.md` and write one ADR per significant decision
from Phase 2. ADRs are what separate a real architecture from a tech-stack list:
they record the context, the options, the choice, and the consequences, so that
six months later nobody has to reverse-engineer why the system is the way it is.
Embed them in the document's "Architecture Decisions" section (or as separate
files under `docs/adr/` if the user prefers — ask).

### Phase 5 — Deliver

Save the document to the repo (default `docs/architecture.md`, or wherever the
user wants). Summarize the key decisions and the main risks in a few sentences
in chat, and point to the diagrams. Offer concrete next steps: a `.docx` export
for stakeholders, a deeper dive on any one decision, or scaffolding the initial
project structure to match the architecture.

## Output format

Default to a single Markdown file with embedded Mermaid diagrams and inline
ADRs. This is repo-native, diff-able, and renders on GitHub. Switch formats only
when asked:
- **Word document** (`.docx`) — when the deliverable is for non-technical
  stakeholders. Use the `docx` skill for this.
- **Separate ADR files** — when the user wants a growing `docs/adr/` log over
  the life of the project.

## What good looks like

- Every major technology choice traces back to a requirement or constraint the
  user actually stated — never "because it's popular."
- The document is right-sized: no 40-page treatise for a CRUD app, no one-pager
  for a payment platform.
- Diagrams render on the first try (see the reliability notes in the diagram
  guide — prefer `flowchart`/`graph` with subgraphs over experimental C4 syntax
  when portability matters).
- Trade-offs are named honestly, including the downsides of the chosen path.
- A new engineer could read it and understand both how the system is shaped and
  why.
