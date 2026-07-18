---
name: acceptance-verification
description: >-
  The independent auditor that closes the per-feature loop. Runs after
  feature-implementation declares a feature developer-done, and answers
  without the implementer's investment: does this feature actually satisfy
  its requirements? Audits the acceptance tests against the criteria
  (coverage per criterion, assertions that assert what the criterion says —
  rejecting and correcting weak tests), re-runs all suites fresh from the
  repo state, verifies requirements directly beyond the tests (FR behavior
  observed, binding NFRs measured), and reviews the feature's test diff for
  weakened or skipped tests. Delivers a verdict — accepted, rework, or design
  defect (each routed to its owner) — writes the feature's acceptance report,
  and on acceptance appends the RTM's Test ref, completing each requirement's
  lifecycle. Never fixes production code; corrects only test artifacts.
  Trigger on "verify the feature", "acceptance verification", "audit FEAT-N",
  "is the feature really done", or "run acceptance".
---

# Acceptance Verification

The independent check that turns *developer-done* into **verified** — the
per-feature loop's real definition of done. Feature-implementation built and
self-gated the feature; this skill audits it without the builder's
investment. Auditor and mechanic stay separate: findings **route** (back to
implementation as rework, or to the design's amendment path as defects) —
this skill never fixes production code. The one thing it may correct is the
**measurement**: test artifacts that fail its audit.

**The fresh-eyes rule.** Independence is this skill's entire value, and it is
enforced through *derivation*, not session hygiene alone: every check is
re-derived from the authoritative documents — technical-design §6's criteria,
the FR/UC statements, the manifest — **never from tasks.md's checkboxes, the
implementation's delivery summary, or any session's claims of green.** Those
are the artifacts under audit, not evidence. Running in a fresh session is
strongly recommended (and is how an orchestrator should invoke it), but the
skill is not refused in-session — the derivation rule does the real work.

## Inputs

Defaults; user paths win; source-gated citation throughout.

- **The feature folder** — technical-design.md (**§6 acceptance criteria: the
  audit standard**, §3 contracts, §8 escalations), ui-design.md, tasks.md
  (read only to confirm developer-done state — never as evidence of
  correctness).
- **The sources behind the criteria** — the SRS (verbatim FR statements and
  binding NFRs for this feature's trace) and use-cases.md (the flows,
  including alternates/exceptions). The criteria are audited *against these*
  too: a criterion that misencodes its FR is a finding, not an authority.
- **The design manifest** — the feature's screens: states coverage and
  conformance claims to spot-check.
- **The live repo** — the code, the test suites, the harness and its
  commands (scaffold-notes / agent-instructions), the git history
  (`FEAT-NNN T*` commits and the feature's test-file diff).
- **The RTM** — `docs/rtm.md` (if present): Test ref is this skill's owned
  column, written on acceptance.

## Workflow

### Phase 1 — Locate the feature

The user names one (the only off-sequence path), or **compute the default:
walk the plan's build sequence; the first feature that is developer-done
(tasks.md fully checked) but has no accepted acceptance report is it.**
Announce the resolution; never a menu. Not developer-done → say so and stop
(implementation finishes first); already accepted → ask what's wanted
(re-verification after changes is legitimate — say so).

### Phase 2 — Re-derive the standard

Build the audit standard fresh from documents: the feature's FR IDs with
their verbatim SRS statements and priorities, its UC flows
(main/alternate/exception), binding NFR constraints, and technical-design
§6's criteria. Cross-check the criteria against their sources — every FR
covered by at least one criterion, every consequential UC path represented,
criteria faithful to the statements they encode. A gap or misencoding here
is already a finding (design defect class).

### Phase 3 — Audit the tests

Read `references/audit-guide.md`. Criterion by criterion: does a test cover
it; does the test **assert what the criterion says** — not a weakened proxy,
not a reinterpretation that fits the code; are the exception/alternate paths
actually exercised? Then the independent anti-fake-green review: the
feature's test-file diff inspected for loosened assertions, skips, deleted
cases, mocked-away behavior — the implementer self-checked this; the
self-check of an anti-self-deception rule is what deserves second eyes.
**Rejected tests are corrected by this skill** — rewritten to assert the
criterion, committed as `FEAT-NNN acceptance: correct test for AC-k` — the
one artifact class it may change.

### Phase 4 — Execute independently

Run everything fresh from the actual repo state, with the harness's own
commands: the feature suite, the whole-repo suite, the E2E suite including
the path this feature owed, migrations against a fresh local store. No
reported green is trusted; only observed green counts — including the tests
Phase 3 corrected.

### Phase 5 — Verify requirements directly

Beyond the tests: each FR's behavior observed where a test alone doesn't
suffice (an audit-log FR's entries inspected; a data-retention FR's effect
checked), and **binding NFRs measured where cheaply measurable** (a
response-time bound timed against the local stack — recorded as indicative,
environment-caveated; a hard perf NFR needing real infrastructure is
recorded as *pending environment*, never silently skipped). Screens
spot-checked against their manifest states and conformance claims.

### Phase 6 — Verdict, report, and routing

Read `references/verdict-and-report.md`. One of three verdicts, with the
report written to the feature folder as `acceptance-report.md`:
- **Accepted** — criteria hold, suites green, requirements verified. **Append
  the RTM Test ref** for every implemented FR (append-only, Test ref column
  only — completing Plan ref → Design ref → Test ref). The feature is
  *verified* — the loop's true done.
- **Rework** — specific failures, each cited by AC/FR with the observed
  evidence, routed back to feature-implementation as new work. No RTM write.
- **Design defect** — the criteria/design themselves are wrong or
  unsatisfiable; routed to detailed-design's amendment path (which may
  escalate further per its own rules). No RTM write.
Mixed findings take the most severe verdict; partial acceptance is not a
verdict — a feature is verified whole or not yet.

## Scope boundaries

Does **not**: fix production code (rework routes back); change designs or
criteria (defects route to their owners); write any RTM column but Test ref;
re-run feature-implementation's fix-loops (a red here is a finding, not a
debugging session); verify features that aren't developer-done.

## What good looks like

- Every criterion traced source → criterion → test → observed result — the
  audit chain is complete and cited.
- Zero trust in claims: every green in the report was produced in this run.
- Corrected tests assert criteria verbatim-faithfully; the correction commits
  say why.
- The verdict is unambiguous, and rework/defect findings are specific enough
  to act on without re-deriving context.
- On acceptance, the RTM rows tell the whole story: planned, designed,
  tested — the requirement lifecycle closed.
