# Elicitation Guide

The goal of the interview is to learn the things that *change the architecture*.
A question is only worth asking if a different answer would lead to a different
design. Everything below is organized that way.

Run the rounds in order, but **adapt**: infer what you can, skip what's
irrelevant to this system, and stop once you can make confident decisions. For a
small internal tool you may only need rounds 1–3. For a multi-tenant SaaS or
anything handling money, health, or PII, work through all of them.

A note on tone: you are a senior architect having a working conversation, not a
requirements robot. Explain *why* a question matters when it isn't obvious —
"I'm asking about peak concurrent users because that's what decides whether we
need horizontal scaling and a load balancer, or whether a single instance is
fine."

---

## Round 1 — Context and purpose

The frame for everything else. Usually answerable in one batch.

- **What is the system, in one or two sentences?** What problem does it solve?
- **Who uses it?** Internal staff, consumers, other businesses, machines/other
  services? Roughly how many, and where (one region, or global)?
- **What does success look like?** Ship fast and iterate? Rock-solid and
  compliant? Cheap to run? This sets the priority order when trade-offs collide.
- **Is this greenfield, or does it have to live alongside existing systems?**

## Round 2 — Functional scope and domains

You don't need every feature — you need the *shape* of the system.

- **What are the core capabilities?** List the 3–7 things the system must do.
- **Are there natural sub-domains or modules?** (e.g., "auth, catalog, orders,
  payments, notifications"). These become your candidate building blocks and
  later your service boundaries if you split.
- **Any part with very different load, criticality, or release cadence from the
  rest?** This is the main legitimate reason to split a service out early.

## Round 3 — Quality attributes (the real architecture drivers)

Spend the most care here. These determine more of the architecture than the tech
stack does. Don't ask all of them mechanically — ask the ones that matter for
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

## Round 4 — Constraints

Constraints quietly decide as much as requirements do.

- **Team.** How many engineers, and what are they strong in? (Architecting a
  Kafka/Kubernetes microservices estate for a two-person team that knows
  Next.js is malpractice — the team's skills are a first-class constraint.)
- **Timeline and budget.** MVP in three weeks vs. a year-long build changes
  everything. Cost ceiling on infrastructure?
- **Existing commitments.** A cloud they're already on, a language mandated by
  the org, systems that must be integrated, licenses already paid for.
- **Operational capacity.** Is there a team to run this 24/7, or does it need to
  be as hands-off as possible? (Strongly pushes toward managed/serverless.)
- **Lock-in tolerance.** How much do they care about staying portable across
  clouds vs. moving fast on a single provider's managed services? Name this
  explicitly — it's a real, conscious trade-off, not an accident.

## Round 5 — Data

Data shape drives store choice and a lot of the component design.

- **What are the main entities and how do they relate?** Highly relational, or
  mostly independent documents? Graph-like? Time-series? Heavy blobs/files?
- **Volume and growth.** Thousands of rows or billions? Read-heavy or
  write-heavy?
- **Access patterns.** Mostly simple key lookups, or complex queries, reporting,
  search, analytics?
- **Retention and compliance.** How long is data kept, and are there deletion or
  residency rules?

## Round 6 — Integration and external dependencies

- **Third-party services** the system must call (payment processors, email/SMS,
  maps, auth providers, LLM/AI APIs, etc.).
- **Inbound integrations** — who calls *this* system, and how (REST, webhooks,
  events, file drops)?
- **What happens when a dependency is down?** Degrade, queue and retry, fail
  hard? (Decides resilience patterns: timeouts, circuit breakers, queues.)

## Round 7 — Deployment and operations

Often partly inferable from earlier answers.

- **Where does it run?** Specific cloud (AWS / Azure / GCP), on-prem, edge,
  hybrid? Any preference or mandate?
- **Release cadence and process.** CI/CD expectations, how often they deploy.
- **Observability.** What do they need to see — logs, metrics, traces, alerting?
  Any existing tooling?
- **Environments.** Just prod, or dev/staging/prod?

---

## Closing the interview

Before you start designing, **play it back**: a short summary of the system, the
top 2–3 quality attributes you'll optimize for, and the hard constraints. Make
the prioritization explicit ("I'm reading this as: ship fast and keep ops
minimal are the top priorities, strong consistency matters for orders but not
elsewhere, and you want to stay on AWS managed services — is that right?").
Confirmation here is cheap; a wrong assumption baked into the whole document is
expensive.
