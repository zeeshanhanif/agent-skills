# Decision Guide

How to reason through the recurring architectural forks. For each one, the job is
to map the answers from the interview onto a choice, and to be able to articulate
the trade-off. Every choice here becomes an ADR.

The meta-rule: **start as simple as the requirements allow, and add complexity
only where a quality attribute forces it.** Most teams over-engineer. A modular
monolith on managed infrastructure is the right answer far more often than the
internet implies. Complexity is a cost paid in development speed, operational
burden, and cognitive load; demand that each increment of it buy a real
requirement.

---

## Monolith vs. modular monolith vs. microservices

- **Default to a modular monolith**: one deployable, but with clean internal
  module boundaries (the sub-domains from Round 2) and dependencies pointing
  inward. It gives you most of the organizational benefit of services with none
  of the distributed-systems tax (network failures, eventual consistency,
  deployment orchestration, distributed tracing).
- **Split a service out only when** a part has genuinely different scaling needs,
  a different release cadence that's blocked by the monolith, a different
  team/owner, or a different technology requirement. "We might need to scale
  later" is not a reason to start distributed — a well-modularized monolith can
  be carved up later precisely *because* the boundaries are already clean.
- **Full microservices** earn their place at real organizational scale (many
  teams needing independent deploy autonomy) or when components have wildly
  divergent scaling/availability profiles. The cost is high; pay it deliberately.

The deciding inputs: team size and ops capacity (Round 4), divergent
load/criticality across sub-domains (Round 2/3).

## Synchronous vs. asynchronous communication

- **Synchronous (request/response, REST/gRPC)** when the caller needs the answer
  now and the operation is fast and reliable.
- **Asynchronous (queue/event)** when work can happen in the background, when you
  need to absorb spikes (buffering), when you want to decouple producer and
  consumer lifecycles, or when a downstream dependency is slow/flaky and you
  don't want to block on it. Async is also how you turn "the dependency is down"
  (Round 6) from an outage into a delay.
- Don't make everything async — it adds operational complexity (a broker to run,
  message ordering, idempotency, dead-letter handling). Reach for it where the
  requirement is buffering, decoupling, or resilience.

## Relational (SQL) vs. document vs. other stores

- **Relational by default** when data is relational (Round 5) and you need
  transactions, joins, and strong consistency. Modern Postgres covers an
  enormous range, including JSON columns for semi-structured data.
- **Document store** when data is naturally document-shaped, access is mostly by
  key, the schema is fluid, and you want easy horizontal scaling over complex
  querying.
- **Specialized stores** when a quality attribute demands it: a search engine for
  full-text/faceted search, a time-series DB for metrics, a cache (Redis) for
  hot reads and ephemeral state, a graph DB for deeply connected traversals,
  object storage for large blobs/files.
- Resist polyglot persistence early — each store is another thing to run, back
  up, and reason about. Add a second store when one store genuinely can't serve
  an access pattern, not preemptively.

## Serverless vs. containers vs. VMs

- **Serverless (functions, managed runtimes)** shines for spiky or low/irregular
  traffic, event-driven workloads, and teams that want minimal ops (Round 4).
  Trade-offs: cold starts, execution limits, statelessness, and tighter vendor
  coupling.
- **Containers (managed: Cloud Run / ECS Fargate / Container Apps, or
  Kubernetes)** for steady traffic, long-running processes, more control, and
  portability. Prefer *managed* container platforms over self-run Kubernetes
  unless the team has real K8s capacity and a reason — K8s is a large operational
  commitment.
- **VMs** when you need full control, specific OS/runtime requirements, or are
  lifting-and-shifting.
- Inputs: traffic shape (Round 3), ops capacity and lock-in tolerance (Round 4).

## Vendor-managed vs. portable

This is the trade-off to name out loud, because it's usually made by accident.

- **Lean into a provider's managed services** (their queue, their managed DB,
  their auth, their functions) to build faster and operate less. Cost: coupling
  to that provider; migrating later is real work.
- **Stay portable** (containers, open-source data stores, thin abstraction at the
  edges) to keep your options open. Cost: more upfront effort, and you forgo the
  slick managed conveniences.
- There's no universally correct answer — it's a function of the lock-in
  tolerance you elicited in Round 4 and how much speed matters now. A pragmatic
  middle path: use managed services freely for undifferentiated heavy lifting,
  but keep your core domain logic free of provider-specific dependencies so the
  expensive-to-move part stays movable.

## Cross-cutting concerns (decide these explicitly, don't let them emerge)

- **AuthN/AuthZ**: how users and services prove identity and what they're allowed
  to do. Driven by Round 3 security answers.
- **Resilience**: timeouts, retries with backoff, circuit breakers, idempotency,
  graceful degradation — sized to the dependency behavior from Round 6.
- **Observability**: logs, metrics, traces, alerting — sized to Round 7.
- **Data protection**: encryption in transit and at rest, secrets management,
  network isolation — driven by data sensitivity and compliance (Round 3).
- **Consistency strategy**: where you need transactions, where eventual
  consistency is fine, and how you reconcile (sagas, outbox pattern) when work
  spans boundaries.

Each of these should appear in the "Cross-cutting Concepts" section of the
document, with the level of rigor proportional to the system's stakes.
