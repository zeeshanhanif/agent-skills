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
  new. If an SRS (docs/srs.md) exists, it designs from those requirements and
  NFRs, interviewing only for gaps. Prefer this skill over answering from memory
  whenever real architectural trade-offs are in play.
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

## Input

This skill consumes the requirements artifacts produced upstream, when they
exist.

- **SRS (primary)** — if the user provides a path, use it; **otherwise default to
  `docs/srs.md`.** When found, it is the primary source: read the requirements and
  NFRs and design against them, shrinking the interview to confirmation plus the
  architecture-specific gaps (see Phase 1).
- **Use cases (secondary)** — default `docs/use-cases.md`. When found, read it to
  inform the **runtime view** (each significant use case becomes a runtime
  scenario / sequence diagram) and the **resilience and security** decisions (the
  exception/alternate flows reveal failure-handling needs; the actor-to-use-case
  mapping reveals trust boundaries). It refines *how* the design realizes the
  requirements; it does not replace the SRS as the driver of *what*.
- **RTM** — not consumed. The traceability matrix re-indexes the SRS and use
  cases without adding input information; this skill ignores it.
- **If no SRS is found**, fall back gracefully to the full elicitation interview
  (all seven rounds in `references/elicitation-guide.md`) — the skill stays fully
  usable standalone, for quick architecture work without a prior requirements
  phase.

**Source-gated traceability (one rule, applied per source).** Cite an ID only
when the document that defines it is present: SRS requirement IDs when `srs.md`
exists, use-case IDs when `use-cases.md` exists. When a source is absent, state
the driver in plain prose instead — **never fabricate an ID.** In the common
cases both documents are present (full pipeline) or neither is (standalone
fallback, prose drivers throughout); the rule handles any mix automatically.

The boundary: the SRS states *what* (requirements); architecture still decides
*how* (technology stack, data stores, deployment). Read any *mandated* technology
from the SRS constraints section, but the stack is otherwise an architecture
decision.

## Workflow

Follow these phases in order. Do not skip straight to a document — designing from
unconfirmed assumptions is the failure mode this skill exists to prevent.

### Phase 1 — Ingest the requirements, then elicit the gaps

1. **Ingest.** Read `docs/srs.md` (per **Input**). Extract the architecture-
   relevant content: purpose and scope (§1.2), product perspective and system
   boundary (§2.1), user classes (§2.3), the functional requirements and their
   capability areas (§3.1, which become candidate building blocks), the external
   interfaces (§3.2), the constraints (§2.5), and — most importantly — the
   **non-functional requirements (§3.3), which are the architecture drivers**.
   Note each NFR's ID; you'll cite them in decisions and ADRs.

   Then read `docs/use-cases.md` if present: note the significant use cases (they
   become runtime scenarios), and scan the **exception/alternate flows** for
   resilience requirements (e.g., "processor unavailable → queue and retry"
   implies an async boundary) and the **actors** for trust/authorization
   boundaries. Keep each use case's ID for the runtime view.

   If no SRS exists, skip to running the full interview in
   `references/elicitation-guide.md`, then continue at Phase 2.

2. **Play back the drivers.** Summarize the top quality attributes (by NFR ID),
   the hard constraints, and the capability areas you'll design toward, and get a
   quick nod. Trust the SRS — this is a confirmation, not a re-interview. Flag any
   architecture-driving NFR that's missing or vague (e.g., no concrete
   scalability target); ask about the few that would change the design, and
   assume-and-note the rest.

3. **Interview only the gaps.** Read `references/elicitation-guide.md` and ask
   only what the SRS doesn't answer — technology preferences and any mandated
   stack, lock-in tolerance, team skills, data-store-relevant access patterns,
   and the deployment/cloud target. Same technique throughout: batch related
   questions, infer aggressively and state inferences, offer one-word-acceptable
   defaults. Keep it short — most of the picture is already in the SRS.

### Phase 2 — Decide

Derive the architecture from what you learned. Read
`references/decision-guide.md` for how to reason about the recurring forks
(monolith vs services, sync vs async, SQL vs NoSQL, serverless vs containers,
vendor-managed vs portable). For each significant fork, **record what drives it,
source-gated**: cite SRS requirement IDs when `srs.md` exists (e.g., a scaling
fork driven by NFR-SCAL-001), cite use-case IDs where an exception flow or actor
boundary drove a resilience/security fork (e.g., UC-007), and when neither source
document exists, state the driver in plain prose ("driven by: peak load ~5k
concurrent users") — never invent an ID. This is what makes the decision
traceable and lets a later requirements change point back to the architecture it
affects. You will write an ADR per fork in Phase 4, so note the options you
weighed and *why* you chose. Resist the urge to over-engineer: the right
architecture for most new apps is simpler than the architecture people reach for.

### Phase 3 — Document and diagram

1. Read `references/document-template.md` and write the architecture document.
   **Right-size it** — the template marks which sections are essential and which
   scale with complexity. A small app gets a tight doc; a distributed system
   gets the full treatment.

2. Read `references/diagram-guide.md` and produce the diagrams as **Mermaid
   embedded directly in the document**, so the whole thing is one portable
   artifact. At minimum produce a **System Context** diagram and a **Container**
   diagram (the two C4 levels that earn their keep almost every time). Add a
   sequence diagram for the most important runtime flow — when `use-cases.md`
   exists, the significant use cases are the natural choice for these runtime
   scenarios; cite their UC IDs — and a deployment diagram when the
   infrastructure is non-trivial.

### Phase 4 — Capture decisions

Read `references/adr-template.md` and write one ADR per significant decision
from Phase 2. ADRs are what separate a real architecture from a tech-stack list:
they record the context, the options, the choice, and the consequences, so that
six months later nobody has to reverse-engineer why the system is the way it is.
Each ADR's **"Requirements addressed"** field is source-gated: cite SRS IDs (and
any use-case IDs) the decision addresses when those documents exist, otherwise
state the driver in prose — never fabricate an ID. With a real SRS this gives
requirement → decision traceability both ways. Embed the ADRs in the document's
"Architecture Decisions" section (or as separate files under `docs/adr/` if the
user prefers — ask).

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
