# Design-System Guide (the Shared Core)

The shared core is defined **once** and is inherited by every surface. It is what
makes an admin portal, a website, and a mobile app read as one product. Keep it
at the level of a *system* — decisions and tokens, not finished screens.

The meta-rule mirrors architecture: **establish a small, coherent core and let
surfaces extend it; don't duplicate, and don't over-specify.** A token set and a
component inventory that fit on a few pages beat an exhaustive style bible nobody
reads.

---

## Design tokens

Tokens are the named, reusable values every surface and component draws from.
Define them once; surfaces may override specific tokens (see the surface-profile
guide), but the names stay stable.

- **Color.** A small semantic palette, not a paint catalog. Define by *role*:
  `primary`, `secondary`, `surface`/`background`, `text` (and muted text),
  `border`, plus state colors `success` / `warning` / `danger` / `info`. Give
  each the value(s) needed to hit the accessibility contrast bar against its
  intended background. Note dark mode if in scope — define it as an alternate set
  of the same role tokens, not a separate palette.
- **Typography.** One or two type families (often a UI/sans family, sometimes a
  display or mono companion), and a **modular type scale** (e.g., a small set of
  steps from caption to display) with line-heights and weights. Density can vary
  per surface via overrides; the scale itself is shared.
- **Spacing.** A single spacing scale (e.g., a 4- or 8-px-based step set) used
  for padding, margins, and gaps everywhere. Consistent spacing is most of what
  makes a UI feel coherent.
- **Radius, elevation, borders.** Corner radii, shadow/elevation levels, and
  border weights — small fixed sets.
- **Iconography.** Which icon set/style, stroke weight, default sizes.
- **Motion.** Standard durations and easing for transitions, if motion matters.

Express tokens as a table in the document. If the user wants a concrete artifact,
emit them as CSS custom properties (`tokens.css`) and/or `tokens.json` — these
can compile to both web and native, which is what lets one core span platforms.

## Core component inventory

List the components that are genuinely **shared** across surfaces, with a one-line
responsibility and the key states each must handle. Don't design them
pixel-by-pixel; specify them. Typical core set:

- **Actions:** button (primary/secondary/tertiary/danger), icon button, link.
- **Forms:** text input, textarea, select, checkbox, radio, toggle, date picker,
  field label + help + error; the form validation and error-display convention.
- **Feedback:** toast/notification, inline alert, modal/dialog, confirmation,
  loading indicator, skeleton.
- **Structure:** card, list, table (if shared), tabs, accordion, breadcrumb,
  pagination.
- **Navigation primitives:** the building blocks surfaces compose into their own
  nav (nav item, menu, dropdown).

For every interactive component, define the **state conventions** once: default,
hover, focus (visible focus is an accessibility requirement), active, disabled,
loading, error. And define the **empty / loading / error** patterns for data
views once — these are the states teams most often forget, so making them a core
convention pays off everywhere.

## Voice and tone

A short, usable definition: the product's voice (the consistent personality) and
how tone flexes by context (an error message vs. a success vs. marketing copy).
Two or three example do/don't pairs are worth more than paragraphs of theory.
Note where tone differs by surface (an admin tool can be terser and more direct
than a consumer site).

## Accessibility standard

State the target (e.g., WCAG 2.2 AA) and make it concrete as cross-cutting rules
the whole system follows: minimum contrast ratios (and that color tokens are
chosen to meet them), always-visible focus states, minimum touch/click target
sizes, full keyboard operability, semantic structure and labels, and respect for
reduced-motion preferences. Setting this in the core means every component and
surface inherits it rather than bolting it on later.

## Cross-surface interaction principles

A few product-wide principles that keep behavior predictable everywhere — for
example, how destructive actions are confirmed, how the product handles
optimistic vs. confirmed updates, how it communicates errors and recovery, and
the consistency rules for terminology and iconography. These are the UX analog of
the architecture's cross-cutting concepts.
