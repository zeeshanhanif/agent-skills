# Elicitation Guide

The goal of the interview is to learn the things that *change the architecture*.
A question is only worth asking if a different answer would lead to a different
design. Everything below is organized that way.

## When an SRS exists, read first and ask little

If `docs/srs.md` is present, **most of these rounds are already answered** — read
them from the SRS rather than asking. Each round below is tagged:

- **[from SRS — confirm]** — the SRS answers this; extract it and play it back,
  don't re-interview. Round 1 ← SRS §1.2/§2.1/§2.3; Round 2 ← §2.2/§3.1; Round 3
  ← §3.3 (the NFRs — the architecture drivers, already measurable); Round 6 ←
  §3.2.
- **[partial]** — the SRS gives part; elicit the rest. Round 4 (constraints: SRS
  has business/regulatory; ask team skills + lock-in tolerance); Round 5 (data:
  SRS has entities/volume; ask store-relevant access patterns).
- **[elicit fresh]** — architecture decisions the SRS doesn't make. Round 7
  (deployment/cloud) and technology-stack preferences. Read any *mandated* tech
  from SRS §2.5, but the stack is otherwise yours to decide.

So with an SRS, the live interview is essentially: confirm the drivers, then ask
about tech stack, lock-in tolerance, team skills, data access patterns, and
deployment. **Without an SRS, run all rounds in full** — this is the standalone
fallback.

If `docs/use-cases.md` is also present, it's a secondary source: its
exception/alternate flows often pre-answer Round 6's "what happens when a
dependency is down?", and its actors inform trust boundaries. Read it for those
rather than asking.

Either way, **adapt**: infer what you can, skip what's irrelevant, stop once you
can decide confidently.

A note on tone: you are a senior architect having a working conversation, not a
requirements robot. Explain *why* a question matters when it isn't obvious —
"I'm asking about peak concurrent users because that's what decides whether we
need horizontal scaling and a load balancer, or whether a single instance is
fine."

---

## Round 1 — Context and purpose  *[from SRS — confirm]*

The frame for everything else. From SRS §1.2 (vision & scope), §2.1 (product
perspective), §2.3 (user classes). Confirm rather than ask when the SRS exists.

- **What is the system, in one or two sentences?** What problem does it solve?
- **Who uses it?** Internal staff, consumers, other businesses, machines/other
  services? Roughly how many, and where (one region, or global)?
- **What does success look like?** Ship fast and iterate? Rock-solid and
  compliant? Cheap to run? This sets the priority order when trade-offs collide.
- **Is this greenfield, or does it have to live alongside existing systems?**

## Round 2 — Functional scope and domains  *[from SRS — confirm]*

You don't need every feature — you need the *shape* of the system. From SRS §2.2
(product functions) and §3.1 (functional requirements); the capability areas are
your candidate building blocks.

- **What are the core capabilities?** List the 3–7 things the system must do.
- **Are there natural sub-domains or modules?** (e.g., "auth, catalog, orders,
  payments, notifications"). These become your candidate building blocks and
  later your service boundaries if you split.
- **Any part with very different load, criticality, or release cadence from the
  rest?** This is the main legitimate reason to split a service out early.

## Round 3 — Quality attributes (the real architecture drivers)  *[from SRS — confirm]*

Spend the most care here. These determine more of the architecture than the tech
stack does. **When an SRS exists, these come straight from §3.3 (NFRs), already
enumerated and measurable — read them, note their IDs, and design to them.** Only
ask when a driving NFR is missing or vague. Don't ask all of them mechanically —
the ones that matter for
*this* system, and offer a sensible default for the rest.

- **Scale / load.** Expected users and, more importantly, *peak concurrent*
  load. Steady or spiky? Growth expectation over the next year? (Decides
  scaling strategy, statelessness, caching, queueing.)
- **Availability.** How bad is downtime? "A few minutes at 3am is fine" vs.
  "every minute costs money/lives." (Decides redundancy, multi-AZ/region,
  failover, health-checking.)
- **Latency.** Is there a response-time target users will feel? Any real-time
  needs? (Decides edge/CDN, caching, sync vs. async, data locality.)
- **Consistency.** When data changes, must everyone see it immediately, or is
  eventual consistency acceptable? (This is the big distributed-systems fork —
  decides transactional boundaries, whether you can split data stores, CQRS,
  sagas.)
- **Security and data sensitivity.** PII, payments, health, financial records?
  Any regulatory regime (GDPR, HIPAA, PCI-DSS, SOC 2, local data-residency)?
  (Decides encryption, network isolation, auth model, audit logging, where data
  may physically live.)
- **Durability.** What's the cost of losing data? Backup/recovery expectations
  (RPO/RTO)?

If the user is non-technical about these, translate: instead of "what's your
consistency model," ask "if two people act at the same moment, is it OK for one
of them to see slightly stale data for a second, or must it always be exact?"

## Round 4 — Constraints  *[partial]*

Constraints quietly decide as much as requirements do. The SRS (§2.5) usually
covers business, regulatory, timeline/budget, and mandated-technology
constraints — **read those**. Team skills, operational capacity, and lock-in
tolerance are typically *not* in the SRS — **ask those**.

- **Team.** *(ask)* How many engineers, and what are they strong in? (Architecting
  a Kafka/Kubernetes microservices estate for a two-person team that knows
  Next.js is malpractice — the team's skills are a first-class constraint.)
- **Timeline and budget.** *(read from SRS if present, else ask)* MVP in three
  weeks vs. a year-long build changes everything. Cost ceiling on infrastructure?
- **Existing commitments.** *(read from SRS §2.5)* A cloud they're already on, a
  language mandated by the org, systems that must be integrated, licenses already
  paid for.
- **Operational capacity.** *(ask)* Is there a team to run this 24/7, or does it
  need to be as hands-off as possible? (Strongly pushes toward managed/serverless.)
- **Lock-in tolerance.** *(ask)* How much do they care about staying portable
  across clouds vs. moving fast on a single provider's managed services? Name this
  explicitly — it's a real, conscious trade-off, not an accident.

## Round 5 — Data  *[partial]*

Data shape drives store choice and a lot of the component design. The SRS gives
the entities (§3.1) and volume/growth (often in §3.3 scalability) — **read
those**; the store-relevant access patterns usually need a couple of questions.

- **What are the main entities and how do they relate?** *(read from SRS §3.1)*
  Highly relational, or mostly independent documents? Graph-like? Time-series?
  Heavy blobs/files?
- **Volume and growth.** *(read from SRS NFRs if present, else ask)* Thousands of
  rows or billions? Read-heavy or write-heavy?
- **Access patterns.** *(ask)* Mostly simple key lookups, or complex queries,
  reporting, search, analytics?
- **Retention and compliance.** *(read from SRS §3.4/§2.5)* How long is data kept,
  and are there deletion or residency rules?

## Round 6 — Integration and external dependencies  *[from SRS — confirm]*

From SRS §3.2 (external interface requirements). Confirm rather than ask when the
SRS exists; the failure-handling question is often the one gap to elicit.

- **Third-party services** the system must call (payment processors, email/SMS,
  maps, auth providers, LLM/AI APIs, etc.).
- **Inbound integrations** — who calls *this* system, and how (REST, webhooks,
  events, file drops)?
- **What happens when a dependency is down?** *(often the gap to ask)* Degrade,
  queue and retry, fail hard? (Decides resilience patterns: timeouts, circuit
  breakers, queues.)

## Round 7 — Deployment and operations  *[elicit fresh]*

An architecture decision the SRS doesn't make (beyond the operating-environment
hint in §2.4). Elicit this.

- **Where does it run?** Specific cloud (AWS / Azure / GCP), on-prem, edge,
  hybrid? Any preference or mandate?
- **Release cadence and process.** CI/CD expectations, how often they deploy.
- **Observability.** What do they need to see — logs, metrics, traces, alerting?
  Any existing tooling?
- **Environments.** Just prod, or dev/staging/prod?

---

## Closing the interview

Before you start designing, **play it back**: a short summary of the system, the
top 2–3 quality attributes you'll optimize for (by SRS NFR ID when an SRS
exists), and the hard constraints. Make the prioritization explicit ("I'm reading
this as: ship fast and keep ops minimal are the top priorities, strong
consistency matters for orders [NFR-CON-002] but not elsewhere, and you want to
stay on AWS managed services — is that right?"). Confirmation here is cheap; a
wrong assumption baked into the whole document is expensive.
