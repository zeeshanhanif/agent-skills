---
name: ux-foundations
description: >-
  Establishes the UX and design foundations for an application before feature
  work begins — the "architecture of the UI." Reads a finalized architecture
  document, then produces a shared design core (brand, design tokens,
  accessibility, voice) plus a per-surface profile (users, information
  architecture, navigation, key user flows, screen inventory, surface-specific
  components) for each UI surface such as an admin portal, public website, or
  mobile app. Use this in the design phase, as a sibling to software
  architecture and before implementation planning, whenever a new application
  with one or more user interfaces is being designed. Trigger on phrases like
  "design the UX foundations", "set up the design system", "what should the UI
  look like", "plan the screens and navigation", "create the design language",
  or any point after the architecture is settled and the UI structure needs
  defining.
---

# UX Foundations

This skill defines the **structure and design language of the UI** before any
feature is built — the counterpart to a software-architecture document, but for
the experience layer. It does not produce pixel-perfect screen designs; it
produces the *system* those screens are later assembled from.

The governing idea, especially when an app has several interfaces: **one shared
core, defined once, plus a per-surface layer for each interface.** A single
design language (brand, tokens, accessibility, voice) spans every surface so the
product feels like one thing; each surface (admin portal, website, mobile app)
then gets its own profile because its users, navigation, components, and
interaction context genuinely differ. Not one monolith that flattens the
differences, and not independent design systems that drift apart.

## Input

This skill consumes the architecture document produced upstream.

- If the user provides a path, use it.
- **Otherwise default to `docs/architecture.md`.**
- If no architecture document is found, tell the user, and offer to either point
  to the right path or proceed in a reduced mode where you elicit the surfaces
  and users directly (the output will be weaker without the architecture's
  context, so prefer locating the document).

From the architecture, extract: the **UI surfaces** (they appear as building
blocks / containers — e.g., "Admin Portal", "Web App", "Mobile App"), the
**user types / actors** (from the context diagram), UX-relevant **quality
attributes** (accessibility, latency/performance that shape perceived
experience), **constraints** (existing brand, mandated frontend stack), and the
**domain and features** that the UI must express.

## Workflow

### Phase 1 — Ingest and frame

1. Read the architecture document (per **Input** above) and pull out the surface
   list, actors, constraints, and domain.
2. Play back the surfaces you found and confirm the set ("I see three UI
   surfaces: an admin portal, a public website, and a mobile app — is that the
   full list?"). Surfaces are the spine of everything that follows.

### Phase 2 — Elicit the UX specifics

Read `references/elicitation-guide.md` and run a short interview for what the
architecture doesn't cover. Same philosophy as the architecture skill: ask in
batches, infer aggressively from the architecture and state your inferences,
offer defaults the user can accept in a word, and let them dump a brief if they
have one. Cover the shared-core questions once (brand, voice, accessibility bar,
visual direction, existing design system) and the per-surface questions for each
surface (who uses it, their primary jobs, device/responsive targets).

### Phase 3 — Establish the shared core (once)

Read `references/design-system-guide.md` and define the parts that span every
surface: brand and voice, **design tokens** (color, typography scale, spacing,
radius, iconography), the **accessibility standard**, and the cross-surface
interaction principles. This is the single source of truth that makes the
surfaces feel like one product.

### Phase 4 — Profile each surface (loop)

Read `references/surface-profile-guide.md`. For **each** surface, produce a
profile: its users and primary jobs, its **information architecture and
navigation** (a sitemap), its **key user flows**, its **screen/page inventory**,
and the **surface-specific components and token overrides** it needs on top of
the shared core. Scale the depth to how much the surface actually diverges — two
web surfaces may share a component library and differ mostly in IA, while a
native mobile app warrants its own component layer and platform-convention notes.

### Phase 5 — Reconcile across surfaces

Resolve the cross-cutting questions: which components are **truly shared** (live
in the core) versus **surface-specific**, which **flows span surfaces** (e.g.,
act on the web, get notified on mobile), and the consistency rules that keep the
surfaces coherent without forcing a desktop pattern onto mobile.

### Phase 6 — Document and diagram

Read `references/document-template.md` and write the UX-foundations document.
Read `references/diagram-guide.md` and embed the **sitemaps** and **key user
flows** as Mermaid, so the document is one portable artifact. Right-size it: a
single-surface tool gets a tight doc; a multi-surface product gets the shared
core plus a section per surface.

### Phase 7 — Deliver

Save to `docs/ux-foundations.md` by default (or where the user wants). Summarize
the design direction and the surface set in a few sentences, and point out that
the **screen inventory is the handoff to implementation planning** — it's what
lets planning slice features vertically across backend and UI. Offer concrete
next steps: emit a real design-tokens file (CSS variables / JSON), produce one
or two anchor screen mockups to lock the visual direction, or proceed to
implementation planning.

## Output format

Default to a single Markdown file (`docs/ux-foundations.md`) with embedded
Mermaid sitemaps and flow diagrams, and tokens/component inventories as
structured tables. Optional, on request:
- **A design-tokens file** — `tokens.css` (CSS custom properties) or
  `tokens.json`, as a concrete artifact the frontend build can consume.
- **Anchor mockups** — one or two key screens rendered to settle visual
  direction, when the product is design-led enough to justify it early.

## Scope boundaries

This skill defines the system and the structure. It deliberately does **not**:
- produce pixel-perfect, per-feature screen designs — those happen per slice in
  the detailed-design step, assembled from this system;
- generate frontend component code — that's implementation.

Staying at the system/spec level is what lets the shared core span platforms:
the same tokens compile to web and native even though components are built
differently on each.

## What good looks like

- One coherent shared core, with per-surface layers that diverge only where the
  context truly demands it.
- Every design choice traces back to a user, a job, or a constraint — never
  decoration for its own sake.
- A complete, per-surface screen inventory that implementation planning can
  slice against.
- Diagrams render on the first try (see the diagram guide's reliability notes).
- Right-sized: structural and decision-dense, not a 60-page style bible.
