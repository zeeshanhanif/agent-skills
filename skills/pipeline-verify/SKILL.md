---
name: pipeline-verify
description: >-
  The cross-document seam checker for the SDLC pipeline. Each stage skill
  verifies its own outputs; this verifies the seams between them —
  mechanical, read-only, fresh-session. Checks that every cited ID
  (FR, NFR, UC, SCR, FEAT, ADR, DEF) resolves in its defining document;
  detects orphans (requirements no design or feature touches, screens no
  feature renders); audits RTM integrity (rows match the SRS, column
  ownership respected, computed verification consistent with partial
  markers); confirms design-manifest locators resolve and the manifest
  matches the screen inventory; enforces tombstone discipline (no active
  work referencing removed items); and spot-checks value authority
  (tokens.json vs design.md agreement). Renders the computed status
  dashboard — per-requirement lifecycle, loop position, coverage — as a
  derived report, never stored state. Trigger on "verify the pipeline",
  "check the documents", "check traceability", "find orphan requirements",
  "project health check", or "are the docs consistent".
---

# Pipeline Verify

The second tier of the pipeline's verification design. **Tier 1** is built
into every stage skill — each verifies its own contract at delivery.
**Tier 2 is this skill**: the seams *between* documents that no single stage
can see — a citation written correctly by one skill against a document
another skill later amended, an FR that every stage individually handled but
no stage ever covered, a manifest locator that rotted when a heading moved.

Three principles govern it:

1. **Mechanical only.** Every check here is decidable by reading and
   computing — IDs resolve or don't, sets cover or don't, values match or
   don't. Judgment quality (is the architecture good, are the tests
   meaningful) belongs to the stage skills and their auditors; a check that
   needs taste doesn't belong in this skill.
2. **Read-only, derived-output-only.** This skill writes nothing into any
   pipeline artifact — no RTM column, no document fix, no "helpful"
   correction. Its single output is the report, an explicitly **derived
   view**: safe to overwrite, safe to delete, regenerable at any time.
   Findings route to their owning skills; fixing is theirs.
3. **Fresh eyes, actual commands.** Run it cold — it needs no session
   context, only the repo. And extract mechanically: grep/parse the IDs,
   diff the sets, resolve the locators with real commands — never eyeball a
   700-row matrix. The checks are specified so a script could run them;
   run them like a script would.

## When to run

After any system-level phase completes (catch seam breaks while the phase's
author-skill session is still warm); before handing the loop to the
orchestrator for a long run; after amendment chains (the highest-risk moment
for dangling references); periodically on mature projects. Full sweep by
default; targeted scope on request ("check the RTM", "check FEAT-007's
seams").

## Inputs

Everything, read-only: `docs/srs.md`, `use-cases.md`, `rtm.md`,
`architecture.md`, `ux-foundations.md`, `design.md`, `tokens.json`,
`implementation-plan.md`, `design-manifest.json`, `docs/features/**`,
`docs/defects.md`, scaffold/deploy notes where present. Absent documents
scope their checks out (source-gating applies to verification too — a
missing use-cases.md means UC checks are skipped and *reported as skipped*,
never failed).

## Workflow

### Phase 1 — Inventory the namespaces

Parse every defining document for the IDs it mints: SRS → FR/NFR; use-cases
→ UC; ux-foundations → SCR (+ surface codes); architecture → ADR (+ CON
constraints if present); plan → FEAT; defects ledger → DEF. Record each ID's
status (active/tombstoned). This inventory is the ground truth every later
check diffs against. Then parse every document for the IDs it *cites*.

### Phase 2 — Run the seam checks

Read `references/checks-guide.md` and execute the catalog: resolution
(every citation lands on a defined, correctly-namespaced ID), tombstone
discipline (no active work cites a removed item), coverage/orphans (the
bidirectional set diffs), RTM integrity (rows, ownership, computed
verification), manifest and folder conventions (locators resolve, names
match), **decision → realization conformance** (the architecture's named
test frameworks and coverage stance actually realized in CI and the repo),
and value authority (tokens agreement). Classify every finding:
**error** (a broken seam — something downstream will trip on), **warning**
(suspicious but survivable — e.g., an FR designed but unplanned on a young
project), **info** (notable, no action implied).

### Phase 3 — Compute the dashboard

The read-only status view, computed fresh (see `references/report-guide.md`):
per-FR lifecycle (planned → designed → verified, honoring partial markers
and the Plan-ref⨝Test-ref completeness rule), per-feature stage (the same
artifact table the orchestrator uses), coverage counts, open
blocks/escalations/defects. Stored nowhere — the report *is* the rendering.

### Phase 4 — Report and route

Write `docs/pipeline-verify-report.md` (overwriting the previous — it's
derived) and summarize in chat: the verdict line (clean / N errors, M
warnings), findings grouped by **owning skill** (an unresolved SCR citation
in the plan routes to implementation-planning's amendment; a stale manifest
locator to ui-design; an RTM row mismatch to requirements-engineering) —
each finding specific enough to fix without re-deriving, each naming its
owner. This skill fixes nothing.

## Scope boundaries

Does **not**: fix anything (findings route to owners); judge quality (stage
skills and auditors); write any pipeline artifact (the derived report only);
replace the stage skills' tier-1 verification (both tiers run; neither
substitutes).

## What good looks like

- Zero eyeballing: every set comparison was computed; the report shows the
  counts (IDs defined, cited, diffed) so the sweep itself is auditable.
- Findings are actionable verbatim: ID, location, what's broken, who owns
  it.
- Skipped checks are reported as skipped with their missing input — absence
  of evidence is never reported as cleanliness.
- The dashboard agrees with what the orchestrator would compute — same
  artifacts, same rules, independently derived.
- Running it twice in a row without changes yields the identical report.
