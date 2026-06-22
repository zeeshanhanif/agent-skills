# Implementation-Plan Document Template

The plan reads as: here's the whole map, here's what we build first to make it
real, and here's the order for the rest. **Right-size it** — sections marked
*[essential]* always appear; a small app gets a tight plan, a large or
distributed system gets phases and a fuller breakdown. Keep it decision-dense;
the breakdown and the first-slice spec are what downstream steps consume.

Use this structure:

```markdown
# Implementation Plan: <System Name>

> Status: Draft | Reviewed · Last updated: <date>
> Inputs: docs/architecture.md · docs/ux-foundations.md

## 1. Overview  *[essential]*
One paragraph on what's being built and the plan's shape. State the top priority
that ordered the work (de-risk / fastest demo / unblock downstream) and any MVP
cut line or fixed deadline.

## 2. Walking Skeleton  *[essential]*
The minimal end-to-end path to build first. What's real, what's stubbed, and the
"done when" condition (a request flows end-to-end in a deployed environment, the
pipeline is green, the UI shell with the design system is live).

## 3. Epic & Feature Breakdown  *[essential]*
The whole-app map. For each **epic** (aligned to an architecture building block),
list its **features** (vertical slices). For each feature, a one-line value
statement plus its touchpoints:
- **Screens** (from the ux-foundations inventory; note the surface)
- **Building blocks / endpoints** (from the architecture)
- **Data** (entities read/written)
- **Flow served**
Near features are detailed; far ones are a title plus touchpoints. A table per
epic works well.

## 4. Build Sequence  *[essential]*
The ordered slices, with the **dependency + risk** reasoning that sets the order.
Include a **dependency graph** (Mermaid; see diagram-guide.md). For larger
systems, group into phases/milestones, each a demonstrable increment.

## 5. First Vertical Slice  *[essential]*
The next thing to build after the skeleton, specified enough to scope and assign:
- the user flow it exercises and why it was chosen (value / risk)
- acceptance criteria (what "done" means)
- the screens it renders and the endpoints/data it needs
*Not* full detailed design — that's the downstream per-slice step. This section
is the handoff to it.

## 6. Engineering Foundations  *[essential]*
A checklist of what to stand up alongside the skeleton, derived from the inputs:
repo conventions, CI/CD, environments, observability, test strategy (from the
architecture's cross-cutting concepts); accessibility bar and design-system/token
setup (from ux-foundations).

## 7. Risks & Assumptions  *[essential]*
Sequencing risks, assumptions the plan rests on, and what would force a
re-order. Note explicitly that phases beyond the near term are a living order,
not fixed commitments.
```

---

## Right-sizing cheat-sheet

- **Small app / MVP:** sections 1–3 (tight), 4 (a simple ordered list + small
  dependency graph), 5, 6, 7. Often no phases — one straight sequence.
- **Production multi-surface app:** full breakdown with per-epic tables, phased
  build sequence, fuller foundations.
- **Large / distributed:** add milestone grouping with a demonstrable increment
  per milestone, and call out the cross-team or cross-surface dependencies
  explicitly in the sequence.

The two parts downstream steps actually consume are the **feature breakdown**
(with touchpoints) and the **first-slice spec** — make those complete and
precise; keep prose tight.
