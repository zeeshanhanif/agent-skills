# UX-Foundations Document Template

The document has two parts: a **shared core** defined once, then **one section
per surface**. This structure is the whole point — it keeps the single design
language in one place and specializes per interface without duplication.

**Right-size it.** Sections marked *[essential]* always appear. A single-surface
internal tool may need only the core plus one short surface section. A
multi-surface consumer product gets the full core plus a section per surface with
real depth. Favor structure and decisions over exhaustive style documentation.

Use this structure:

```markdown
# UX Foundations: <Product Name>

> Status: Draft | Reviewed · Last updated: <date>
> Sources: docs/srs.md · docs/architecture.md · docs/use-cases.md
> Design system: **docs/design.md + docs/tokens.json** (single source of truth
> for all design values — this document references, never restates)

## 1. Overview  *[essential]*
One paragraph: what the product is, and the list of UI **surfaces** this document
covers (admin portal, website, mobile app, …) with one line on who each is for.
This surface list is the spine of the document. Note the design-source mode used
(research / images / design file / tool).

---

## Part A — Shared Core (summary; authority lives in design.md)
The single design language inherited by every surface. **This part summarizes
direction and links to design.md — it does not restate token values or component
specs.** Keeping value authority in one file is what prevents drift.

### A1. Brand and visual direction  *[essential]*
The visual personality in a few lines, the design-source provenance, and a link
to design.md §1.

### A2. Voice and tone  *[essential]*
The product's voice and how tone flexes by context (full microcopy rules:
design.md §6).

### A3. Design language summary  *[essential]*
One short paragraph characterizing the palette, type, and density — **then defer:
tokens live in design.md §2 / tokens.json; components and states in design.md
§4.** No value tables here.

### A4. Accessibility standard  *[essential]*
The target, **citing the SRS NFR ID(s) it comes from** (e.g., WCAG 2.2 AA per
NFR-USE-001; prose if no SRS). Mechanical rules: design.md §5.

### A5. Cross-surface principles  *[scale]*
Product-wide interaction rules: destructive-action confirmation, update/error
handling, terminology consistency.

---

## Part B — Surfaces
One subsection per surface. Repeat B for each.

### B<n>. <Surface name> — surface code `<CODE>`  *[essential per surface]*
- **Users and primary jobs** — who, in what context, and the top 3–7 jobs,
  **traced to the SRS user classes (§2.3) they come from** when an SRS exists.
- **Information architecture & navigation** — the nav model and a **sitemap
  diagram**.
- **Key user flows** — the critical journeys as **flow diagrams**, **citing the
  UC IDs they realize** when use-cases.md exists (e.g., "Checkout flow
  [UC-011]").
- **Screen / page inventory** — the catalogue of screens, **each with a stable
  `SCR-<CODE>-<NNN>` ID**, purpose, and key states. *(The handoff to
  implementation planning; the per-slice ui-design step is told which screens
  to design by these IDs.)*
- **Surface-specific components** — what this surface adds beyond the core.
- **Token overrides** — deviations from the shared tokens, with rationale (or
  "none", which is the healthy default).

---

## Part C — Cross-Surface Reconciliation  *[scale: include when >1 surface]*
- **Shared vs. surface-specific components** — the final split, and any component
  promoted into the core because multiple surfaces need it.
- **Flows that span surfaces** — journeys that hop between interfaces (act on the
  web, get notified on mobile), and how they stay coherent.
- **Consistency rules** — what must stay identical across surfaces vs. where
  surfaces are deliberately allowed to differ.

## Part D — Open Questions and Risks  *[essential]*
Unresolved design decisions, assumptions to validate with real users, and known
tensions (e.g., a brand requirement that fights the accessibility bar). A short,
honest section.
```

---

## Right-sizing cheat-sheet

- **Single-surface internal tool:** Overview + full Part A (brief) + one Part B +
  Part D. Skip C.
- **Two web surfaces (e.g., admin + site):** full A, two B sections, a light C.
  Expect few token overrides and a largely shared component set.
- **Multi-surface incl. native mobile:** full A, a B per surface with real depth
  on the divergent ones, a substantive C. The native surface earns its own
  component layer and platform-convention notes.

Keep it decision-dense. The screen inventories and the tokens are the parts
downstream skills actually consume — make those complete and precise; keep the
prose tight.
