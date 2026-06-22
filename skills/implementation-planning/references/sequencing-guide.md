# Sequencing Guide

A backlog of slices is not a plan until it's ordered. Sequencing decides what
gets built first, and the goal is to make the system real and de-risk the
unknowns as early as possible — not to pick off the easy wins.

## The walking skeleton comes first

Before any feature, define the **walking skeleton**: the thinnest possible
end-to-end path that exercises every major container and proves the system runs
*and deploys*. For a typical app that means a trivial request travelling UI →
API → domain → database and back, deployed to the target environment through the
real pipeline, with a minimal **UI shell** (layout, navigation, and the design
system/tokens wired in) so feature UIs have somewhere to land.

Specify it concretely:
- **What's real:** the wiring, the deployment path, the shell, one trivial
  end-to-end operation.
- **What's stubbed:** business logic, most screens, auth beyond a placeholder,
  external integrations.
- **Done when:** a request flows end-to-end in a deployed environment and the
  pipeline is green.

The skeleton turns the architecture from hypotheses into a running system at the
moment when changing it is cheapest, and it gives every later slice a place to
attach.

## Order by dependency, then risk

Within the remaining slices, two forces set the order:

- **Dependency** — foundational capabilities before the things that need them.
  Auth and the identity model, the core data layer, and shared domain primitives
  usually come early because many slices sit on top of them. Read the
  architecture's building-block dependencies to get this right.
- **Risk** — pull the most uncertain work forward. The ADRs the architecture
  flagged as close calls or assumptions, the hardest external integrations, the
  highest-coupling components, and anything with unproven performance or
  consistency characteristics. The principle is *fail early*: if a risky decision
  is going to break, you want it to break in week two, not month four.

When dependency and risk disagree, let risk win for anything that could force a
re-architecture, and dependency win for ordinary build order. Make the reasoning
visible so the sequence isn't arbitrary.

## The first vertical slice

Right after the skeleton, pick a first slice that is narrow, genuinely
demonstrable, exercises a real user flow end-to-end, and — ideally — touches a
risky area so it doubles as validation. A good first slice proves the
architecture and the design system work together on something real, however
small. Avoid making the first slice the most complex feature; avoid making it so
trivial it proves nothing.

## Phases and milestones

For larger systems, group the ordered slices into phases or milestones — e.g., a
walking-skeleton phase, an MVP phase (the smallest set that delivers the core
value), then iteration phases. Each milestone should be a coherent, demonstrable
increment, not a layer. Keep it light; the point is a build order, not a
Gantt-chart fantasy of dates.

## Progressive elaboration

Detail the near slices; keep the far ones coarse. As construction proceeds and
you learn from shipped slices, re-sequence the remainder. The plan is a living
order, not a contract — say so in the document so no one treats the later phases
as fixed commitments.
