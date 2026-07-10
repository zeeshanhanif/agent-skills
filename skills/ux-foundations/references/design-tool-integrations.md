# Design-Tool Integrations

Per-tool guidance for Mode 4 (connected design tool). Structure: a **generic
protocol that always works standalone**, then per-tool sections that refine it.
A missing tool section never blocks the mode — apply the generic protocol.
Sections here record *what to fetch and how it maps to our token/component
structure*, not API minutiae (those rot; the MCP tool descriptions are the
authority for call shapes at run time).

## Generic protocol (any tool)

1. **Discover** what the connected tool exposes (list its MCP tools; look for
   variables/tokens, styles, components, and screen/frame access).
2. **Fetch and map**:
   - variables / design tokens → our token set (color, spacing, radius, size)
   - text styles → type scale (family, sizes, weights, line-heights)
   - color / effect styles → palette and elevation scale
   - published components (+ variants) → component inventory with states
   - representative frames/screens → layout & grid rules; early screen hints
3. **Normalize** into semantic role-based tokens (the tool may use raw names;
   assign roles from usage, confirm with the user).
4. **Record fidelity**: values pulled from a tool are exact — say so in
   Provenance; anything the tool doesn't expose goes to Known Gaps.
5. **Play back and confirm** before codifying.

If the tool exposes more than we model (prototypes, auto-layout rules, motion),
note its existence in Known Gaps rather than force-fitting it.

---

## Figma

Connected via the Figma MCP. The high-value fetches, mapped:

- **Variables** (Figma variables incl. modes) → tokens. Figma variable *modes*
  (e.g., light/dark) map to token themes — capture both if present.
- **get_variable_defs** on key nodes → the raw token values behind a design.
- **Styles** (text styles, color styles, effect styles) → type scale, palette,
  elevation.
- **Published library components** (and their variant properties) → component
  inventory: each Figma variant axis (size, state, kind) maps to our
  variant/state spec.
- **Screenshots / design context of key frames** → layout & grid rules; use
  sparingly, values from variables/styles beat pixel inspection.

Practical notes: prefer *published library* styles/components over ad-hoc local
ones (they represent the intended system); if the file has no variables and
only raw fills, treat it closer to Mode 2 fidelity and say so in Provenance.

## Claude Design

Stub — refine as usage patterns emerge. Expect design-system-aware output;
map its generated systems/screens via the generic protocol. If it emits a
design spec document, that is Mode 3 input of high quality.

## Figma Make

Stub — refine as usage patterns emerge. Generated artifacts typically carry an
implicit system; extract via the generic protocol, expect Known Gaps around
states.

## Open Design

Stub — refine as usage patterns emerge. Apply the generic protocol.

## Unlisted tools

Apply the generic protocol. If a tool proves recurrent, add a section above:
what it exposes, the mapping, and its fidelity quirks — nothing more.
