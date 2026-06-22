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
  the domain logic, and the data. This is the unit you sequence and assign.
- A feature is too big if it can't be demonstrated in one go; split it. It's too
  small if it delivers nothing observable on its own; merge it.

## Why vertical, not horizontal

Horizontal slices — "build all the tables", "build every screen", "wire all the
endpoints" — feel orderly but deliver no working software until the very end,
when everything integrates at once and the risks all surface together. Vertical
slices each produce something runnable and demonstrable, integrate continuously,
and let you learn from real behavior before committing to the next slice.

## How to find the slices

Walk the **key user flows** from ux-foundations — each flow, or each meaningful
step in a flow, is a candidate slice. Cross-check against the architecture's
**runtime scenarios**; a scenario usually corresponds to one or more slices.
Then, for each candidate, record what it touches:

- **Surfaces & screens** — which screens from the ux-foundations inventory it
  renders or modifies (and which surface, if the product has several).
- **Building blocks & endpoints** — which containers/components from the
  architecture it exercises, and the API operations it needs.
- **Data** — the entities it reads or writes.
- **Flow served** — the user flow and step it advances.

This mapping is what makes a slice *vertical and concrete* rather than a vague
title — and it's the trace that lets the downstream detailed-design step pick a
slice up without re-deriving context.

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
