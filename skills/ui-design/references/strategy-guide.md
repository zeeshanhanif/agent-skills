# Strategy Guide (per-screen resolution and the three strategies)

The adapter's core. Strategy is resolved **per screen**, always in this order,
and every strategy converges on the same finish: conformance verified, states
covered, manifest entry written.

## Resolution order (per SCR ID)

1. **Manifest first.** If the screen already has a manifest entry (an anchor
   pass or an earlier feature designed it), reuse it — screens are shared
   across features, and the manifest is the single registry. In per-feature
   mode, re-check its **contract bindings** for *this* feature (the screen may
   now also display a new endpoint's data) and update the entry if bindings
   grew. Re-design an existing entry only when the user says so or
   re-verification flagged it.
2. **Source lookup.** Query connected design tools for the screen. **First
   stop: `design_provenance.source` in design.md §9** (populated when
   ux-foundations built the system from a tool): it names the tool
   and file/project where designs live, so look there before generic
   discovery. Then match by SCR-ID naming convention in the tool (recommend
   designers name frames with SCR IDs; record the mapping on first
   registration so later lookups are exact) or by prior partial
   registrations. Note the converse: modes 1–3 projects can still have
   screens in a tool (a designer started a file after ux-foundations), so
   a null `design_provenance.source` never skips this lookup when tools are
   connected. Found → **Strategy A**.
3. **Fallback per the routing policy.** Not found → the project's recorded
   policy: **Strategy C (code-native, the default)** or **Strategy B
   (generate)**. The policy is asked once on the first run that needs it —
   "when a screen isn't in your design source: spec it for code (default) or
   generate it?" — written into the manifest header, applied thereafter,
   overridable per screen on request.

Why code-native is the default fallback: generation is the flakiest path
(preview-grade tooling, conformance-checking overhead), and screens nobody
hand-designed are usually the long tail — forms, settings, empty states —
which design.md + the component layer renders well straight from code. Users
whose design file must remain the complete source of truth flip the policy to
generate.

## Strategy A — Register an existing design

The screen exists in a connected design tool. The skill does not design; it **fetches,
checks, fills, and registers**:

1. Fetch the screen's design data via the tool (see design-tool-integrations.md).
2. **Conformance check** against design.md/tokens.json: colors, type,
   spacing, components. Off-system elements → conform-or-escalate (below).
3. **Contract check** (per-feature mode): the screen presents what the bound
   endpoints return; UC alternate/exception flows have UI states (an error
   response the screen can't display is a gap).
4. **Fill missing states**: designers rarely draw empty/loading/error — spec
   the missing states code-natively (composition from design.md's state
   conventions) rather than demanding tool round-trips; record them as
   code-native *state supplements* on the same entry.
5. Write the manifest entry: `strategy: registered`, source locator, states,
   bindings, conformance result.

## Strategy B — Generate via a design tool

A generation-capable design tool produces the screen from the system:

1. Compose the generation input from authority: the relevant design.md
   sections (tokens, components, layout rules), the screen's inventory entry
   (purpose, states), and — per-feature mode — the contract shapes it
   displays.
2. Generate; then run the **same conformance check as Strategy A** — generated
   output gets zero trust advantage; a generated off-system color is corrected
   or escalated exactly like a fetched one. This is what stops generation from
   quietly forking the system.
3. Iterate within reason (bounded attempts); persistent non-conformance →
   fall back to Strategy C for that screen and note it.
4. Register: `strategy: generated`, tool + artifact locator, states,
   bindings, conformance.

## Strategy C — Code-native spec

No tool involvement; the screen is specified precisely enough to build:

1. Write the structural spec in the feature's ui-design.md: layout (from
   design.md's grid/layout rules), composition (which components, arranged
   how), content/data mapping (which contract fields appear where), all
   states (default/empty/loading/error per the system's state conventions),
   responsive behavior (the surface's breakpoints), accessibility notes where
   the screen has specifics beyond the system rules.
2. Register: `strategy: code-native`, source = the spec's path+section,
   states, bindings.
3. Realization happens during feature implementation. Recommend (not
   require) a **screenshot loop** at build time: render, screenshot, compare
   against the spec and design.md, iterate — the code-native analog of
   design review.

## Conform-or-escalate (the authority rule, all strategies)

design.md + tokens.json are the authority. When a screen contains an
off-system element:

- **Correctable** (a raw hex that should be a token, a near-miss spacing) →
  correct it to the system; note the correction in the entry.
- **Genuine gap** (the screen legitimately needs a component/pattern/token
  the system lacks — e.g., the first data-dense table when the system never
  defined one) → **escalate**: file a proposed design-system amendment toward
  ux-foundations (what's needed, which screens need it, cited by SCR ID),
  record `escalation` on the manifest entry, and either wait or proceed with
  an explicitly-noted provisional treatment (user's call). The system absorbs
  the pattern once, properly — instead of screens accreting private forks.

The judgment line mirrors detailed-design's schema rule: composing existing
components in a new arrangement — yours. A new *kind* of thing the system's
vocabulary lacks — escalate.

## Anchor mode specifics

Anchor screens pick the **hardest compositions**, not the easiest — a dense
data screen, a primary-flow screen — because the mode's purpose is
falsification: does the system compose into real screens? No contracts exist
yet, so bindings are empty and content uses representative placeholder shapes.
The deliverable's core is the **verdict**: composes cleanly / composes with
these amendments (filed) / doesn't compose (back to ux-foundations before
features proceed). Anchor entries carry `feature: anchor` and are first-class
manifest citizens — later features will find and reuse them via the Phase 2
manifest lookup.
