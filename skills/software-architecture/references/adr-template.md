# Architecture Decision Record (ADR) Template

An ADR records a single significant decision: the context that forced it, the
options considered, the choice made, and the consequences accepted. ADRs are
what make an architecture *legible six months later* — they answer "why is it
like this?" so nobody has to reverse-engineer the reasoning or relitigate a
settled question.

Write one ADR per significant decision. A decision is "significant" if it's
costly to reverse, shapes other decisions, or someone might later wonder why it
went the way it did. Tech-stack picks, monolith-vs-services, store choice,
sync-vs-async, vendor-managed-vs-portable, auth model — all ADR-worthy. Naming a
variable is not.

## Format

Keep each ADR short — half a page is plenty. Use this structure:

```markdown
### ADR-NNN: <Short decision title>

**Status:** Proposed | Accepted | Superseded by ADR-XXX
**Date:** <date>

**Context**
What's the situation and the forces at play? Which requirements and constraints
(reference them from the interview) make this decision necessary? What makes it
non-trivial? State the quality attributes in tension.

**Decision**
What did we decide, stated plainly and actively. "We will use X."

**Options considered**
- **Option A (chosen)** — one line on what it is, then its pros and cons.
- **Option B** — pros and cons, and the specific reason it lost.
- **Option C** — (if relevant) pros, cons, why not.
Showing the rejected options is the point: it proves the decision was reasoned,
and it tells a future reader what's already been weighed.

**Consequences**
What becomes easier and what becomes harder as a result — including the
downsides we're knowingly accepting. Note any new constraints this imposes on
later decisions, and what would make us revisit it.
```

## Example

```markdown
### ADR-003: Use a modular monolith rather than microservices

**Status:** Accepted
**Date:** 2026-06-17

**Context**
The system has five sub-domains (auth, catalog, orders, payments,
notifications). The team is two engineers strong in TypeScript/Next.js with no
prior Kubernetes or distributed-systems experience, and the MVP is due in six
weeks. Top quality goals are speed-to-ship and low operational burden; no
sub-domain has a materially different load profile yet. The tension is between
future scaling flexibility and present-day simplicity and delivery speed.

**Decision**
We will build a modular monolith: one deployable application with strict
internal module boundaries aligned to the five sub-domains, dependencies
pointing inward, and no shared mutable state across modules.

**Options considered**
- **Modular monolith (chosen)** — one deployable, clean internal seams. Pro:
  fast to build, trivial to operate, no network/consistency tax, and the clean
  boundaries make a later split cheap. Con: the whole app scales and deploys as
  one unit.
- **Microservices** — separate services per sub-domain. Pro: independent
  scaling and deployment. Con: imposes distributed-systems complexity (network
  failures, eventual consistency, orchestration, distributed tracing) that a
  two-person team can't absorb in six weeks, for scaling needs we don't yet
  have. Lost on team capacity and timeline.

**Consequences**
We ship faster and operate one thing instead of five. If a sub-domain later
needs independent scaling or ownership, the enforced module boundaries let us
extract it into a service without a rewrite — we've kept that option open rather
than foreclosed it. The cost we accept: the app scales as a unit for now, and we
must hold module discipline (enforced via lint rules / dependency checks) to
keep the seams from eroding. Revisit if a sub-domain's load diverges sharply or
the team grows enough to need independent deploy autonomy.
```

## Where ADRs live

By default, embed ADRs in the "Architecture Decisions" section of the main
document. If the user expects the decision log to grow over the project's life,
offer to put each ADR in its own file under `docs/adr/` (e.g.,
`docs/adr/003-modular-monolith.md`) with a sequential number — this is the
conventional long-lived ADR layout and plays well with version control and PR
review.
