# Design Guide (per-feature technical design)

How to produce `technical-design.md` for a feature. The document's reader is the
implementer (human or agent) and ui-design; every section should be concrete
enough to build from without re-deriving context, and every element traces to
the requirement or flow that demands it.

## technical-design.md structure

```markdown
# Technical Design: FEAT-NNN — <Feature name>

> Feature from: docs/implementation-plan.md · Epic: <epic>
> Implements: FR-… (list) · Realizes: UC-… · Screens: SCR-… (designed by ui-design)
> Status: Draft | Verified · Date: <date>

## 1. Intent
Two or three sentences: what this feature delivers, for whom, and why it's next
(from the plan's value statement and sequencing rationale).

## 2. Codebase context
What the design conforms to, from the Phase 2 survey: the contract/error
conventions in force, the current schema state (last migration), the module
this lands in, utilities reused, prior features depended on. Note any
document-vs-code divergence found (reality wins; divergence recorded).

## 3. API contracts
Per endpoint: method, path, auth requirement, request shape (with validation
rules), response shape(s), and error responses — each error traced to the UC
exception/alternate flow or validation rule that demands it. Follow the
codebase's envelope and naming conventions exactly. State idempotency and
side-effects where relevant.

## 4. Schema changes
The migration(s), concretely: tables/columns/indexes/constraints, with types
and defaults — a diff against the *current* schema, not the skeleton's.
Include the down/rollback story if the project's migration mechanism has one.
Map each change to the conceptual-model entity it belongs to.

## 5. Component design
The internals: modules/services/functions with responsibilities and
interactions (a small Mermaid sequence or component sketch where it clarifies;
follow the pipeline's diagram reliability rules — quoted labels, no bare
`end`). Name the skeleton stubs this feature replaces (the stub comments point
back here).

## 6. Acceptance criteria
Testable statements, each citing its FR/UC: "Given/When/Then" or equally
checkable form. Cover the main flow, the consequential alternates/exceptions,
and the NFR constraints that bind this feature (cite NFR IDs — e.g., the
response-time bound that applies to the new endpoint).

## 7. Decisions
Feature-local forks, mini-ADR form: decision, driver (cite the ID or the code
reality), alternative rejected, consequence. One short block each.

## 8. Escalations & open items
Architecture amendments filed (and why), plan corrections suggested, TBDs.
Empty is the happy case; silent is never acceptable.
```

## The schema ownership rule (hard boundary)

The architecture owns the **conceptual** domain model: which entities exist,
who owns them, the consistency boundaries between them. This skill owns the
**physical** realization per feature: columns, types, indexes, constraints,
migrations — *within* those entities.

**Escalation, not invention:** if the feature needs an entity the conceptual
model doesn't have, or needs to move data across a consistency boundary, or
changes entity ownership — stop. File it as a proposed architecture amendment
(state what's needed and why, citing the feature's FRs), and either wait for the
amendment or proceed on the explicitly-noted assumption it will be accepted
(user's call). A "small local table" that's really a new domain entity is how
conceptual models rot.

Judgment line: a join table, a column addition, an index — physical, yours. A
new noun the domain vocabulary doesn't have — conceptual, escalate.

## Contracts discipline

- **Convention inheritance is mandatory**: read how existing endpoints shape
  envelopes, name fields, paginate, and report errors — and match it. A feature
  that introduces a second convention is a defect even if internally clean.
- **Errors are designed, not defaulted**: every UC exception flow and every
  validation rule maps to a specific error response. The generic 500 is what's
  left *after* design, not instead of it.
- **NFRs bind here**: the SRS's relevant NFRs (auth strength, rate limits,
  response-time bounds, audit events) appear as concrete contract properties,
  cited by ID.

## Cross-feature dependencies

When this feature consumes another feature's contract or schema, cite that feature's
technical-design.md section rather than restating it. When it *changes* something a
prior feature established (a contract field, a column), that's a breaking-change
decision: record it in §7 with the migration/compat story, and note it in the
delivery summary — downstream code and the prior feature's tests are affected.
