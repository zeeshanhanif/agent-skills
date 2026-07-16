---
name: ui-design
description: >-
  Designs the screens — the presentation half of each feature's low-level
  design, sibling to detailed-design. Runs in three modes: anchor (right after
  ux-foundations, designing 2-3 key screens to validate the design system
  before features build on it), per-feature (the construction loop — after
  detailed-design, designing that feature's SCR screens against its
  contracts), and re-verification (after a design.md amendment). Resolves
  strategy per screen: picks up screens that already exist in a connected
  design tool (Figma, Claude Design, Figma Make, Open Design, or any design tool),
  otherwise falls back per project policy to code-native specs or generation.
  Emits one uniform contract — docs/design-manifest.json keyed on SCR IDs —
  that downstream implementation reads regardless of which tool or strategy
  produced each screen. Trigger on "design the screens", "design anchor
  screens", "design this feature's UI", "ui design for", or "register the
  Figma designs". Screens conform to design.md or escalate — never fork it.
---

# UI Design

The **presentation half of the feature's low-level design** —
detailed-design's conceptual sibling, but sequential in the loop: in
per-feature mode this skill **consumes** the feature's technical-design.md
(screens bind to its contracts — a screen displays what an endpoint returns),
which is why detailed-design runs first. Only anchor mode, which precedes any
feature, has no such dependency. It answers, per screen: what does it look
like, what does it compose
from, what states does it cover, where does its design authority live. Its
defining characteristic is that it's an **adapter**: how screens get designed
varies by project (an existing design file, a generation tool, straight to
code), so this skill routes across strategies while emitting **one uniform
output contract** — the design manifest — that everything downstream reads,
whatever produced each screen. All design-tool variance is absorbed here;
no downstream skill ever learns what a Figma node is.

Three principles govern it:

1. **Strategy is resolved per screen, not per project.** A project with a
   partial design file is "register" for the screens that exist and the
   project's fallback policy for the rest. The manifest lookup by SCR ID is
   always the first move.
2. **Screens conform to the design system or escalate — never fork it.**
   `docs/design.md` + `tokens.json` are the authority. A fetched or generated
   screen introducing an off-system color, component, or pattern is either
   corrected to the system, or the *system* is amended through ux-foundations'
   amendment path (the presentation analog of detailed-design's schema
   escalation). Local work never silently mutates upstream authority.
3. **The manifest is the only registry.** With per-screen routing, no design
   tool ever holds the complete picture — only `docs/design-manifest.json`
   does. This skill is its **single writer**.

## Inputs

Defaults below; user paths win; source-gated citation throughout (SCR/FEAT/
FR/UC/NFR IDs only from documents that exist; never fabricate).

- **Design system** — `docs/design.md` + `docs/tokens.json` (from
  ux-foundations). The authority every screen conforms to. Also read
  design.md's structured `design_provenance` block (mode and fidelity drive the
  anchor recommendation) and ux-foundations.md's screen inventory (SCR IDs,
  purposes, states) and per-surface profiles.
- **Feature context** (per-feature mode) — the feature's
  `technical-design.md` (contracts the screens bind to: a screen displays
  what an endpoint returns) and its plan entry (SCR IDs, FR/UC trace).
- **The manifest** — `docs/design-manifest.json`, if it exists: what's already
  designed, by which strategy, and the project's routing policy.
- **Connected design tools** — whatever design-tool MCPs are connected. Treated
  capability-first (see `references/design-tool-integrations.md`): tools are
  interchangeable engines behind two capabilities — *enumerate/export
  existing designs* and *generate from a system spec*.

## Outputs

1. **`docs/design-manifest.json`** — the global, SCR-keyed screen registry:
   one entry per screen with strategy, source locator, states coverage,
   contract bindings, conformance result, status. Single-writer; entries
   appended/updated by this skill only. Schema:
   `references/manifest-guide.md`.
2. **`docs/features/FEAT-NNN-<slug>/ui-design.md`** (per-feature mode) — the
   human-readable screen specs and decisions, beside technical-design.md.
   Anchor mode writes `docs/anchor-screens.md` instead (no feature folder
   exists yet).
3. **RTM append** (`docs/rtm.md`, if present; silent skip otherwise): the
   feature's ui-design ref appended into **Design ref** for the FRs whose
   screens it designs — append-only, that column only.
4. **Escalations** (when the authority rule trips): a proposed design-system
   amendment filed toward ux-foundations, recorded in the manifest entry —
   never a silent local fork.

## Workflow

### Phase 0 — Mode detection

Determine the mode from context and intent; confirm when ambiguous:
- **Anchor** — ux-foundations just completed, no features designed, or the
  user asks for anchor/key screens. Purpose: validate that the design system
  *composes* before features build on it.
- **Per-feature** — a feature's technical-design.md exists (or the user names
  a FEAT); the loop mode.
- **Re-verification** — design.md was amended; re-check the manifest entries
  whose screens the amendment affects (changed tokens/components), update
  conformance results, flag screens needing rework.

**Anchor recommendation gradient** (state it, let the user decide — anchors
are recommended, never mandatory): *strongly suggested* when design.md's
provenance is research mode or non-technical stakeholders must sign off;
*suggested* for image-extraction or ingestion provenance (the translation was
never visually validated); *skip* for tool-sourced systems (mode 4 — the key
screens already exist) and framework-constrained projects. Cost framing: an
anchor pass is hours; a system amendment discovered at feature 5 is not.

**First-feature-as-implicit-anchor:** in per-feature mode, if no anchor ever
ran and the manifest is empty, treat this feature's screens with anchor-level
scrutiny — the system's first compositional test is happening now; watch for
system amendments, not just screen correctness.

### Phase 1 — Gather the screen set

Anchor mode: pick 2–3 screens with the user — the most compositionally
demanding and stakeholder-visible from the inventory (a dense data screen, a
key flow screen), not the easiest.

Per-feature mode — first resolve **which feature, deterministically**: if the
user named one (by FEAT ID or name), use it — explicit naming is the only
legitimate off-sequence path. Otherwise **compute the default: walk the
plan's build sequence in order; the first feature whose folder has
`technical-design.md` but no `ui-design.md` is next.** Multiple eligible
folders is not ambiguity — the sequence resolves it (earliest in build order
wins); never present a menu, which invites accidental violation of the plan's
dependency-and-risk ordering. **Announce the resolution** ("FEAT-004 is next
per the plan's build sequence; designing its screens") so the user can
redirect before work starts. Then gather: the feature's SCR IDs
from its plan entry/technical-design.md; pull each screen's inventory entry
(purpose, states) and the contracts it binds to. Re-verification: the
affected entries computed from the amendment's changed tokens/components.

### Phase 2 — Resolve strategy per screen

Read `references/strategy-guide.md`. For each SCR ID:
1. **Manifest lookup** — already designed (by an anchor or an earlier
   feature)? Reuse; verify its contract bindings still hold for this feature.
2. **Source lookup** — does the screen exist in a connected design tool (matched
   by SCR-ID naming or prior registration)? → **Strategy A: register**
   (fetch, verify conformance, fill missing states, register).
3. **Not found** → the project's **routing policy**: **Strategy C:
   code-native spec** (default) or **Strategy B: generate** via a connected
   design tool. The policy is asked once on first run, recorded in the manifest
   header, applied thereafter, overridable per screen.

**When no design tool is connected but one is wanted** (the user asked to
register or generate, or the policy is "generate"): never silently downgrade
to code-native — **say so plainly**, name what's needed (a design-tool MCP —
e.g., Figma's official server), and point at the runtime's own connection
mechanism, **verifying the current steps against live docs rather than
reciting remembered commands** (connection mechanics are runtime-specific and
drift — the same live-reality rule as scaffolding's generators). Then offer
both paths: connect now and proceed tool-backed, or proceed code-native and
register tool designs later (re-verification mode exists precisely for
after-the-fact registration). A deferred integration beats a blocked
pipeline; a silent downgrade beats neither.

### Phase 3 — Execute strategies

Per strategy protocols in `references/strategy-guide.md` (tool specifics in
`references/design-tool-integrations.md`). Every path ends the same way: the screen
verified against design.md/tokens.json (conform-or-escalate), all key states
covered (empty/loading/error — fill or record as gaps), contract bindings
checked (per-feature mode: the screen shows what the endpoints return; UC
alternate/exception flows have UI states), and the manifest entry written.

### Phase 4 — Write the feature document

Read `references/document-templates.md` and follow its structures — the
heading formats are load-bearing (manifest locators anchor on them).
Per-feature mode: `ui-design.md` in the feature folder — per screen: the spec
or source reference, composition (which design.md components), states,
contract bindings, decisions, escalations. Anchor mode: `anchor-screens.md`
**opening with the verdict** that is anchor mode's whole point — **does the
system compose?** — with any system amendments proposed. Keep prose here; the
manifest stays the machine index.

### Phase 5 — Verify

Read `references/verification.md`: every screen in the set has a manifest
entry; every entry's source resolves; conformance run per screen with
escalations filed not forked; states coverage recorded honestly; contract
bindings cited from technical-design.md; RTM appended; IDs source-gated. Fix
failures; flag the unfixable.

### Phase 6 — Deliver

Summarize per screen: strategy, source, states, escalations. Anchor mode
additionally states plainly: "anchor designs are registered in
design-manifest.json; the design system itself is unchanged" — or, when
escalations were filed, "amendments proposed toward ux-foundations: <list>" —
so it's explicit that no ux-foundations document was written by this skill.
Live-computed progress ("FEAT-004 screens designed — manifest now covers 9 of 27 inventory
screens"; computed, never stored). Handoff: implementation consumes
tasks.md + the manifest (+ ui-design.md for the specs); code-native screens
are realized during feature implementation, optionally with a screenshot
loop against the spec.

## Scope boundaries

Does **not**: implement screens (implementation's job — including realizing
code-native specs); design contracts/schema (detailed-design); amend the
design system itself (files escalations to ux-foundations); write any RTM
column but Design ref (append-only); let any downstream artifact depend on a
tool's API — downstream reads the manifest only.

## What good looks like

- Every screen in scope has exactly one manifest entry; SCR ID is a unique
  key; the manifest — not any tool — is the complete registry.
- Strategy was resolved per screen; existing designs were picked up, not
  regenerated; fallbacks followed the recorded policy.
- Zero silent forks: every off-system element was corrected or escalated.
- States coverage is honest — gaps recorded, not glossed.
- A fresh implementation session can build any screen from the manifest +
  specs without knowing which tool (if any) was involved.
