# Architecture Document Template

Based on **arc42** (the widely used architecture-documentation template),
trimmed and annotated for practical use. **Right-size it**: sections marked
*[essential]* always appear; sections marked *[scale]* appear only when the
system's complexity or stakes warrant them. For a small internal tool, the
essential sections plus a context and container diagram may be the whole
document. For a payment platform, include everything.

Write in clear prose. Explain decisions, don't just assert them. Use the
diagrams from `diagram-guide.md` inline where indicated.

---

Use this structure:

```markdown
# Architecture: <System Name>

> Status: Draft | Reviewed | Approved · Last updated: <date> · Author: <name>

## 1. Introduction and Goals  *[essential]*
One paragraph on what the system is and the problem it solves. Then:
- **Top quality goals** (ranked): the 2–3 quality attributes this architecture
  optimizes for, in priority order, each with one line on why. This ranking is
  the north star — every later decision should be traceable to it.
- **Key stakeholders** and what each cares about (optional for small systems).

## 2. Constraints  *[essential]*
The fixed constraints the design must respect: team and skills, timeline/budget,
mandated technologies, existing systems, compliance/regulatory, lock-in
tolerance. Brief — a table or bullet list is fine. These explain *why the
architecture isn't something else*.

## 3. Context and Scope  *[essential]*
What's inside the system boundary and what's outside it (users, external
systems). Include the **System Context diagram** (C4 Level 1) here. Note the key
external interfaces — who the system talks to and how.

## 4. Solution Strategy  *[essential]*
The architecture in a nutshell: the handful of fundamental decisions that shape
everything (e.g., "modular monolith on managed containers, Postgres as the
system of record, async queue for notifications, AWS managed services"). One
short paragraph or a tight list. Each item links to a fuller ADR in section 9.

## 5. Building Block View  *[essential]*
The static structure. Include the **Container diagram** (C4 Level 2) — the
deployable/runnable units and how they connect. Then describe each building
block briefly: its responsibility and its key dependencies. For complex blocks,
drill into a Component diagram (C4 Level 3). Keep boundaries aligned to the
sub-domains from the interview.

## 6. Runtime View  *[scale]*
How the building blocks collaborate for the important scenarios. Pick the 1–3
flows that best reveal how the system works (e.g., "user places an order,"
"webhook from payment processor"). Use **sequence diagrams**. Skip for trivial
CRUD systems where the runtime behavior is obvious from the structure.

## 7. Deployment View  *[scale]*
The mapping of software onto infrastructure: environments, regions/zones,
networking, scaling. Include a **deployment diagram** when the topology is
non-trivial. For a single managed service this can be a sentence.

## 8. Cross-cutting Concepts  *[essential, but sized to stakes]*
The decisions that span the whole system. Cover the ones that matter here:
- **Security**: authn/authz model, encryption, secrets, network isolation,
  audit logging.
- **Data**: store choice and why, schema/consistency approach, migrations,
  backup/recovery (RPO/RTO).
- **Resilience and error handling**: timeouts, retries, circuit breakers,
  idempotency, graceful degradation, what happens when each dependency fails.
- **Observability**: logging, metrics, tracing, alerting.
- **Performance and scaling**: caching strategy, statelessness, how it scales
  under the load from the interview.
For a small system, a few sentences each. For a high-stakes system, a subsection
each.

## 9. Architecture Decisions  *[essential]*
The ADRs — one per significant decision (see adr-template.md). This is the most
valuable section: it records the *why*. At minimum, an ADR for each item in the
Solution Strategy.

## 10. Quality Requirements  *[scale]*
Concrete, testable quality scenarios where they matter ("under 500 concurrent
users, p95 response time stays under 300ms"; "the system tolerates the loss of
one availability zone with no data loss"). Turns vague goals into things you can
verify.

## 11. Risks and Technical Debt  *[essential]*
Honest list of the architecture's known weak spots, assumptions that might not
hold, and deliberate shortcuts taken for speed (with a note on when they'd need
revisiting). A short, candid section here builds a lot of trust.

## 12. Glossary  *[scale]*
Domain and technical terms, if the system has jargon a new reader wouldn't know.
```

---

## Right-sizing cheat-sheet

- **Small internal tool / MVP**: sections 1–5, 8 (brief), 9 (a few ADRs), 11.
  One context diagram, one container diagram. Aim for something a reader
  absorbs in a few minutes.
- **Production consumer/B2B app**: add 6, 7, 10. Fuller cross-cutting section.
- **High-stakes / regulated / distributed**: everything, with real rigor in 8,
  10, and 11, and ADRs for every meaningful fork.

When in doubt, shorter and decision-dense beats long and padded. Nobody has ever
complained that an architecture doc was too easy to read.
