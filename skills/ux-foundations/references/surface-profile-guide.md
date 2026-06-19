# Surface-Profile Guide (the Per-Surface Layer)

Run this once per surface. A profile takes the shared core and specializes it for
one interface's users, navigation, and context. The depth scales with how much
the surface diverges: two web surfaces sharing a component library may need only
a light profile (mostly IA and a few components), while a native mobile app needs
a fuller one (its own component layer, platform conventions, touch model).

Produce these six things for each surface.

## 1. Users and primary jobs

A tight restatement, specialized to this surface: who uses it, in what context,
and the 3–7 jobs they come to do. Pulled from the architecture's actors plus the
per-surface elicitation. This frames every other decision in the profile — an
internal admin used all day optimizes for speed and density; a public site for a
first-time visitor optimizes for clarity and guidance.

## 2. Information architecture and navigation

The structure of the surface: its top-level sections, how they're organized, and
the **navigation model** appropriate to the surface and its device target —
e.g., a persistent sidebar for a data-dense admin portal, a top nav for a
marketing site, a tab bar for a mobile app. Capture this as a **sitemap** (a
tree of sections and key pages; see the diagram guide). The navigation model is a
real design decision, not a default — choose it from the surface's content
volume and device context.

## 3. Key user flows

For the most important jobs, the end-to-end path the user takes across screens —
e.g., "admin reviews and approves a refund," "visitor signs up and onboards,"
"user places an order." Capture each as a **flow diagram** (see the diagram
guide). Flows reveal the screens and states the surface actually needs and expose
gaps the IA alone won't. Cover the primary happy paths plus the consequential
edge cases (empty states, errors, permission boundaries) for critical flows.

## 4. Screen / page inventory

The catalog of screens for this surface — the **most important handoff in the
whole document**, because implementation planning slices features against it.
List each screen with a one-line purpose and, where useful, its key states
(empty / loading / error / populated) and the core or surface-specific components
it composes. Organize by the IA sections. Aim for completeness of breadth (every
screen a user can reach) even though each screen's detailed design comes later,
per slice.

## 5. Surface-specific components

The components this surface needs **beyond** the shared core, because its context
demands them. Examples: an admin portal needs data tables with sorting / filtering
/ bulk actions, dashboards, and detail/edit panes; a marketing site needs hero
sections, feature blocks, pricing tables, and CTAs; a mobile app needs native
navigation patterns, pull-to-refresh, and bottom sheets. List them with a one-line
spec, the same way as the core inventory. Be honest about what's genuinely
surface-specific versus what should be promoted into the shared core because more
than one surface needs it.

## 6. Token overrides

Where this surface deviates from the shared tokens, and why — always as
*overrides of named core tokens*, never a fresh palette. Common, legitimate
overrides: larger touch targets and spacing on mobile; a denser type and spacing
scale on a data-heavy admin portal; a more expressive type scale on a marketing
site. If a surface needs no overrides, say so — that's the healthy default and
means the core is doing its job.

---

## After the per-surface loop

The per-surface profiles feed directly into the cross-surface reconciliation
(shared vs. surface-specific components, cross-surface flows, consistency rules)
covered in the main workflow. Keep a running note, as you profile each surface,
of any component you've now seen on two or more surfaces — those are candidates
to promote into the shared core rather than maintain in parallel.
