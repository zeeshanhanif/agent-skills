# Document Templates (ui-design.md and anchor-screens.md)

The two prose documents this skill writes. They share one per-screen shape —
the skill executes the same structure in both modes — with mode-specific
framing. The manifest stays the machine index; these carry the substance: the
code-native specs, the composition reasoning, the decisions, and (anchor mode)
the verdict.

**Anchors are load-bearing.** The manifest's `source` locators for
code-native screens and state supplements point at these documents' headings
(`ui-design.md#scr-web-004--sign-in`, `#empty`). So: headings follow the fixed
formats below, and are **never renamed after registration** — a necessary
rename is a manifest-entry update too, same discipline as everything else.

---

## ui-design.md (per-feature mode; beside technical-design.md)

```markdown
# UI Design: FEAT-NNN — <Feature name>

> Feature from: docs/implementation-plan.md · Pairs with: technical-design.md
> Screens: SCR-… (list) · Mode: per-feature · Status: Draft | Verified · Date: <date>

## SCR-XXX-NNN — <Screen name>
*(one section per screen; heading format is fixed — manifest locators anchor here)*

- **Strategy & source:** registered (tool + locator) | generated (tool +
  locator) | code-native (this section is the design).
- **Composition:** which design.md components, arranged how — layout per the
  system's grid/layout rules; the surface's nav/shell context.
- **Content & data mapping:** which contract fields appear where, citing
  technical-design.md's endpoints (e.g., "list rows ← GET /orders items[]").
- **Conformance:** pass | corrected (what) | escalation filed (what, why).

### default
The primary state, specified buildably (code-native) or referenced (tool-backed).
### empty
### loading
### error
*(a subsection per state the inventory lists; state supplements' locators
anchor here. States that simply follow design.md's state conventions say so
in one line rather than restating them.)*

- **Responsive notes:** behavior at the surface's breakpoints, where
  non-obvious.
- **Decisions:** screen-local forks, mini-ADR form (decision, driver,
  rejected alternative), only where a real fork existed.

## Cross-screen decisions
Choices spanning this feature's screens (shared patterns, consistent
treatments) — kept here so per-screen sections stay local.

## Escalations & open items
Design-system amendments filed toward ux-foundations (what, why, which SCR
IDs need it), TBDs. Empty is the happy case; silent is never acceptable.
```

## anchor-screens.md (anchor mode; at docs/anchor-screens.md)

Same per-screen structure, three differences: the verdict leads, no contract
bindings exist yet, and the header carries system provenance instead of a
FEAT reference.

```markdown
# Anchor Screens

> Design system: docs/design.md (provenance mode: <mode>) · Status: … · Date: <date>
> Purpose: validate that the design system composes into real screens
> before features build on it.

## Verdict: does the system compose?
**Composes cleanly** | **Composes with amendments** (listed below, filed
toward ux-foundations, with the reasoning) | **Does not compose** (back to
ux-foundations before features proceed). This section is the document's
point — it leads, with the *why*, never a bare flag.

## SCR-XXX-NNN — <Screen name>
*(same per-screen shape as ui-design.md, with:)*
- **Content:** representative placeholder data shapes — no contracts exist
  yet; note the shapes assumed so the eventual feature can check them.
- *(no contract-bindings field)*

## Escalations & open items
The amendments the verdict references, in full: what the system lacks, which
screens exposed it, the proposed addition.
```

## Right-sizing

A simple screen's section can be a few lines; states that are trivially
standard reference the system's conventions in one line. What's never
compressed: the fixed heading formats (locators depend on them), honest
conformance results, and — anchor mode — the verdict with its reasoning.
