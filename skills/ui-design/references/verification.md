# Verification (Self-Check Before Delivery)

Phase 5's contract — mechanical conformance of this run's output, checked
before delivery. Fix failures; flag the unfixable in the summary. (Per-skill
layer of the pipeline's verification approach.)

## 1. Coverage

- Every SCR ID in this run's scope (the feature's screens / the anchor set /
  the amendment-affected set) has exactly one manifest entry — none missing,
  none duplicated. SCR ID is a unique key across the whole manifest.
- Per-feature mode: the scope matches the feature's plan entry / technical-
  design.md screen list exactly. No tombstoned SCR was designed.

## 2. Entry integrity

- Every entry's **source locator resolves** — the Figma node opens, the
  artifact exists, the spec path+section is present in ui-design.md. A
  locator pointing at nothing is a failure, not a TODO.
- `strategy` matches reality (a "registered" entry has a tool locator; a
  "code-native" entry has a spec).
- States coverage is honest: covered/gaps/supplements sum to the screen's
  inventory states; supplements' spec sections exist.

## 3. Conformance and authority

- The conformance check ran for every screen — including generated ones (zero
  trust advantage).
- Every off-system element was **corrected (recorded) or escalated (filed
  toward ux-foundations and marked on the entry)** — zero silent forks. Grep
  the specs/corrections for raw values that should be tokens.
- Escalation-pending screens carry `status: escalation-pending`, and the
  delivery summary names them.

## 4. Contract bindings (per-feature mode)

- Every binding cites an endpoint that exists in this feature's
  technical-design.md.
- Every screen's bindings suffice for its purpose: the data it displays has a
  source; UC alternate/exception flows the screens serve have UI states.
- Screens reused from earlier features had their bindings re-checked and
  updated for this feature.

## 5. Documents and write-backs

- Per-feature: `ui-design.md` exists in the feature folder with a section per
  screen; anchor: `anchor-screens.md` exists **with the composition verdict**
  (composes / composes-with-amendments / doesn't — never omitted).
- RTM Design ref appended (when rtm.md exists) for the FRs whose screens this
  run designed — append-only, Design ref only.
- Manifest is valid JSON, sorted keys, policy header present (if any fallback
  was ever needed).
- All cited IDs (SCR/FEAT/FR/UC) resolve in their source documents;
  source-gating respected.

## Reporting

One line in the delivery summary: "verification clean" or the flagged list
("SCR-ADM-007 escalation pending: data-dense table pattern proposed to
design system"). The user never discovers a gap the skill knew about.
