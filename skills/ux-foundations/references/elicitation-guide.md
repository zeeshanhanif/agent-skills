# Elicitation Guide

The upstream documents already answer a lot. **Read them first and infer
everything you can**: the architecture gives the surfaces, actors, and frontend
constraints; the SRS gives the user classes (§2.3 — **the personas source; confirm
them, never re-elicit them**), the vision/tone (§1.2), the accessibility bar
(§3.3 NFRs), and brand/compliance constraints (§2.5). Only ask what's genuinely
missing or what shapes the design and isn't derivable. A question is worth asking
only if a different answer would lead to a different design.

Two mode-awareness rules: the **brand/visual-direction questions apply only in
research mode** — in extract/ingest/tool modes the direction comes from the
source, so those questions collapse to gap-filling (what the source doesn't
show). And **never re-ask what a source already answered** — an ingested design
file that defines the palette closes the palette question.

Run two kinds of rounds: the **shared-core round** once for the whole product,
and the **per-surface round** once for each surface. Adapt and stop when you can
make confident decisions.

---

## Shared-core round (ask once)

These define the single design language that spans every surface.

- **Brand and visual direction.** Is there an existing brand — logo, colors,
  typography, guidelines? If yes, you're codifying it, not inventing it. If no,
  what feeling should the product convey (e.g., trustworthy and calm, bold and
  energetic, minimal and precise)? One or two adjectives is enough to anchor
  tokens later.
- **Voice and tone.** How should the product talk to users — formal, plain,
  friendly, technical? Does it differ by surface (an admin tool can be terser
  than a consumer site)?
- **Accessibility bar.** Target standard (e.g., WCAG 2.2 AA is a common
  default)? Any legal or sector requirement? This drives contrast, focus, target
  sizes, and semantics throughout — set it now, not later.
- **Existing design system or component library.** Are they already on something
  (a design system, a UI kit, Tailwind, Material, a company library)? If so, the
  core inherits from it rather than starting clean.
- **Internationalization.** Multiple languages, RTL scripts, locale formats? If
  yes, it constrains layout, typography, and component design across all
  surfaces.

If the user has none of this thought through, offer sensible defaults they can
accept in a word — a neutral, accessible token set at WCAG AA — and move on. The
core can be refined; don't block on perfect brand answers.

## Per-surface round (ask once per surface)

For each surface the architecture named (admin portal, website, mobile app, …):

- **Primary users and their context.** Who uses this surface and in what
  situation? Internal staff at a desk all day? First-time visitors on a phone?
  Field workers on the move? The context drives density, target size, and
  navigation.
- **Top jobs.** The 3–7 things a user comes to this surface to accomplish. These
  become the key flows and seed the screen inventory.
- **Device and responsive targets.** Desktop-first, mobile-first, fully
  responsive, or a specific platform (iOS, Android, both)? Native or web? This
  decides breakpoints, interaction model (pointer vs touch), and how much the
  surface diverges from the others.
- **Surface-specific constraints.** Anything unique — must match an existing
  internal tool's look, must work offline, must embed in another product, heavy
  data-entry vs mostly read-only, real-time updates.
- **Volume and complexity of content.** Dense dashboards and large tables, or a
  handful of simple pages? This sizes the component set and the IA.

Infer hard from the architecture and domain. If the architecture says "internal
admin portal for ~20 staff," you already know: desktop-first, density over
hand-holding, efficiency-focused, modest visual flourish — state that and let
them correct it rather than asking every question.

---

## Closing

Play back, before designing: the shared direction (brand feel, voice,
accessibility bar) and, for each surface, its users, primary jobs, and device
target. Confirm the surface list one more time — it's the spine of the document.
A wrong assumption about who a surface is for, or whether it's native or web,
ripples through every later decision, so it's cheap to verify now.
