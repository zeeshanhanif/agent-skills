# Slicing Guide

The job here is to turn two design documents into a backlog of work units that
are worth building one at a time. The whole quality of the plan rests on slicing
*vertically* — and on slicing against the real screens and building blocks the
design defined, not invented ones.

## Epics, features, slices

- **Epic** — a large body of work, aligned to an architecture **building block /
  bounded context** (Auth, Catalog, Orders, Notifications). Epics are the map's
  regions; they organize the backlog but aren't built directly.
- **Feature (vertical slice)** — a thin unit *within* an epic that delivers
  user-visible value by cutting through every layer it needs: a screen or flow
  step (from ux-foundations), the endpoint(s) behind it (from the architecture),
  the domain logic, and the data. This is the unit you sequence and assign —
  and the unit the construction loop iterates over. **Every feature gets a
  stable `FEAT-NNN` ID** at definition time, sequential in definition order,
  with the pipeline's immutability rules: never renumbered or recycled,
  amendments take the next free number, removals tombstone. The ID — not the
  name — is what the build sequence, the dependency graph, and the RTM's Plan
  ref key on, and downstream construction skills rely on this ID being stable;
  names can change, IDs can't.
- A feature is too big if it can't be demonstrated in one go; split it. It's too
  small if it delivers nothing observable on its own; merge it.

## Why vertical, not horizontal

Horizontal slices — "build all the tables", "build every screen", "wire all the
endpoints" — feel orderly but deliver no working software until the very end,
when everything integrates at once and the risks all surface together. Vertical
slices each produce something runnable and demonstrable, integrate continuously,
and let you learn from real behavior before committing to the next slice.

## How to find the slices

Work from two directions and reconcile. **Flow-first:** walk the key user flows
from ux-foundations (and the use cases they realize) — each flow, or each
meaningful step, is a candidate slice; cross-check the architecture's runtime
scenarios. **Requirements-first:** walk the SRS functional areas — every
non-tombstoned FR must land somewhere, and FRs that sit on no flow
(cross-cutting behaviors, background jobs, negative requirements) surface here
rather than in flow-walking. The reconciliation *is* the coverage check (below).

For each candidate slice, record its **traced touchpoints** (source-gated —
cite IDs only from documents that exist, prose otherwise, never fabricate):

- **FRs implemented** — the SRS requirement IDs this slice delivers
  (FR-AUTH-001..003). One FR may span slices; note "partial" when so.
- **UCs realized** — the use case(s) this slice advances (UC-004).
- **Screens touched** — the SCR IDs from the ux-foundations inventory, with
  surface visible in the ID itself (SCR-WEB-004, SCR-ADM-002).
- **Building blocks & endpoints** — which containers/components from the
  architecture it exercises, and the API operations it needs.
- **Data** — the entities it reads or writes.

This mapping is what makes a slice *vertical, concrete, and verifiable* — and
it's the trace that lets the downstream detailed-design and ui-design steps
pick a slice up without re-deriving context, and lets a later requirements
amendment find the slices it affects.

## Coverage (run after decomposition)

With FR and SCR IDs traced per slice, completeness is arithmetic, not vibes.
Check two ledgers:

- **Every non-tombstoned Must-priority FR** is either implemented by a slice,
  **placed** in the engineering foundations (legitimate for cross-cutting FRs —
  audit logging is foundations work, not a slice; record the FR ID there), or
  explicitly excluded with a reason.
- **Every non-tombstoned inventory screen (SCR ID)** is touched by some slice —
  an untouched screen is usually dead UI or a missed feature.

Uncovered items go through the **escalation ladder**:
1. **Self-resolve** — attach to an existing slice, mint a small new slice, or
   place in foundations. "Resolved" can mean *placed*, not only *sliced*.
2. **Ask the user** — batch what you can't confidently place, with your best
   guess per item ("FR-AUD-003 looks like foundations — agree, or a slice?").
3. **Report and proceed** — anything still unresolved is recorded in the plan's
   Risks section as an explicit open coverage gap with its IDs. A documented
   gap is honest; a silent one is defective.

Should/Could-priority FR gaps skip step 2 and go straight to the report.

## Quality bar for a slice (INVEST)

A good slice is **I**ndependent (minimal ordering entanglement), **N**egotiable
(scope can flex), **V**aluable (a user or the system can see the result),
**E**stimable (clear enough to size), **S**mall (buildable in a short cycle),
**T**estable (you can state when it's done). If a slice fails several of these,
reshape it.

## Cross-surface slices

When a flow spans surfaces (act on the web, get notified on mobile), you can
either keep it as one slice that touches both surfaces or split per surface with
an explicit dependency between them. Prefer splitting when the surfaces are on
different platforms/teams; keep it together when it's small and tightly coupled.
Note the cross-surface relationship either way.

## Breadth now, depth later

Decompose the **whole application** into epics and features so everyone has the
full map. But keep features that are far off coarse — a title plus the touchpoints
above is enough. Only the walking skeleton and the first (and near) slices get
fleshed out now; the rest are elaborated just-in-time as they reach the front of
the sequence. Resisting the urge to fully specify the entire backlog upfront is
the difference between a plan and a doomed waterfall spec.
