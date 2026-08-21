# Checks Guide (the seam catalog)

Phase 2's catalog, organized by seam. Every check states what it computes and
its finding class. Execute mechanically — extract with grep/parse, diff as
sets, resolve with real commands. Where a defining document is absent, the
seam's checks are **skipped-and-reported**, never failed.

ID patterns (the namespaces): `FR-<AREA>-NNN`, `NFR-<CAT>-NNN`, `UC-NNN`,
`SCR-<CODE>-NNN`, `FEAT-NNN`, `ADR-NNN`, `DEF-NNN` (plus `CON-` constraints
where the architecture minted them). Extraction is regex over all pipeline
documents; keep the extraction counts for the report.

## 1. Resolution (every citation lands)

For every cited ID anywhere: it exists in its defining document. Diff
cited-set minus defined-set per namespace; every element of the difference
is an **error**, reported with its citing location(s). Also **namespace
sanity**: an ID cited in an impossible place (a DEF in the SRS, a FEAT in
use-cases) is a **warning** — legal-looking, structurally suspicious.

## 2. Tombstone discipline

For every ID whose defining row/entry is tombstoned (Removed/Deprecated):
no *active* artifact cites it as live work — no active plan feature traces a
removed FR, no manifest entry serves a removed SCR (its entry should read
`status: removed`), no un-superseded ADR addresses only removed
requirements (**warning** — candidate for supersession). Historical
citations (revision histories, tombstoned rows' own refs, acceptance
reports' past sections) are exempt — the check targets *active* columns and
current-status entries only. Violations: **error**.

## 3. Coverage and orphans (the bidirectional diffs)

- **FR → plan**: every active Must FR appears in some feature's touchpoints,
  the foundations, or a documented exclusion (the plan's own coverage check
  should have ensured this; absence here is an **error** — the seam broke
  after planning, usually via amendment). Should/Could uncovered: **info**.
- **FR → design**: every active FR's RTM Design ref non-empty *once its
  feature is designed* (stage-aware: an FR whose features are all still
  planned is **info**, not a finding).
- **UC → realization**: every active UC cited by ≥1 feature or key flow.
  Orphan UC: **warning** (specified behavior nothing delivers).
- **SCR → features**: every active inventory screen touched by ≥1 feature or
  an explicit gap. Orphan screen: **warning** (dead UI or missed feature).
- **ADR → requirements**: ADRs whose "Requirements addressed" is empty:
  **info** (legal — purely internal decisions — but listed). ADRs citing
  only tombstoned IDs: caught by check 2.
- **Reverse orphans**: features whose FR lists are empty (**error** — a
  FEAT without requirements breaks the inheritance chain), manifest entries
  for SCR IDs no inventory defines (**error**).

## 4. RTM integrity

- **Row set**: RTM rows ⇔ SRS-defined FR/NFR set — missing rows or extra
  rows are **errors**. Row fields (priority, status) match the SRS:
  mismatches **error** (two truths about one requirement).
- **Ownership heuristics**: requirement columns changed relative to SRS
  content (see above); Design/Plan/Test ref cells containing values of the
  wrong kind (a FEAT in Design ref, a design path in Plan ref): **error**.
- **Computed verification consistency**: per FR, apply the rule — fully
  verified iff every Plan-ref feature has an accepted report in Test ref;
  `(partial)` markers consistent with the plan's partial notations
  (a Test ref marked partial whose plan touchpoint never said partial:
  **warning**, and vice versa). A Test ref citing a report whose latest
  verdict is *not* Accepted: **error** (the append should only happen on
  acceptance).
- **Plan ref ⇔ plan**: every Plan ref FEAT exists in the plan and lists that
  FR in its touchpoints (bidirectional): mismatches **error**.

## 5. Manifest and conventions

- Every manifest `source` locator **resolves**: spec paths+anchors exist in
  the named ui-design.md/anchor-screens.md (parse the headings), tool
  locators are well-formed (network resolution only when tools are
  connected — otherwise **info**: unverifiable this run).
- Manifest ⇔ inventory: SCR keys ⊆ active inventory (+`removed` entries for
  tombstoned); per-feature ui-design.md screen sections ⇔ manifest entries
  claiming those specs.
- **Folder conventions**: every `docs/features/FEAT-*` folder's ID exists in
  the plan; slug mismatch with the plan's feature name: **warning** (rename
  drift); folders for tombstoned FEATs: **warning**.
- tasks.md structural sanity: final task is verification; an E2E task exists
  when the feature's UCs intersect the architecture's critical flows
  (**warning** when absent — the obligation may predate the feature's
  design).

## 6. Decision → realization conformance (testing)

The seam between what the architecture decided and what scaffolding built —
it breaks most often *after* an amendment, when the decision moved and the
repo didn't.

- Architecture §8 names an **enforced** coverage stance → the CI config
  contains a coverage gate at the stated threshold and scope: absent gate, or
  a threshold below the stated one, is an **error** (a stance nobody enforces
  is a stance in name only). A threshold *above* the stated one: **info**.
- Stance **report-only** but no coverage configuration exists at all:
  **warning**. Stance **none** but a gate exists: **warning** (unintended
  strictness — likely a stale realization).
- `scaffold-notes.md`'s recorded threshold or named frameworks disagreeing
  with architecture §8: **warning** (decision/record drift — one of them is
  stale).
- Architecture §8 names unit runners / an E2E framework that the repo's test
  configuration doesn't use: **warning** (realization drift; the fallback
  case is legitimate only when §8 is silent, and then scaffold-notes should
  say so).

## 7. Value authority (spot checks)

- tokens.json parses as valid DTCG; design.md's CSS block values ⊆
  tokens.json values (a value in the CSS block absent from tokens.json:
  **error** — the derivation drifted).
- ux-foundations.md restating concrete token values (hex/px tables outside
  design.md): **warning** — Part A is contracted to summarize and link.
- Optional deeper checks when trivially scriptable: claimed contrast pairs
  in design.md §5 recomputed from the token values (mismatch: **warning**).

## 8. Defect ledger (when present)

DEF rows with `Fixed by` filled but `Re-verified` open beyond the fix
commit's presence: **warning** (fix landed, audit pending). Re-verified refs
resolve to reports whose latest verdict is Accepted: else **error**.

## Counting discipline

The report records, per namespace: defined, cited, resolved, orphaned — the
raw counts that make the sweep auditable. A check that examined zero items
says so ("0 DEF rows — ledger absent, checks skipped").
