# Audit Guide

Mechanics for Phases 2–5. The posture throughout: **claims are exhibits,
observations are evidence.** Anything the implementation session asserted —
checkboxes, summaries, reported greens — is what's under audit; only what
this run derives from documents and observes by execution counts.

## Re-deriving the standard (Phase 2)

Build the audit table fresh: one row per acceptance criterion in
technical-design §6, joined to the FR statement(s) it encodes (verbatim from
the SRS), the UC path(s) it represents, and any binding NFR. Then audit the
*standard itself* before auditing against it:

- **Coverage of sources**: every FR in the feature's trace has ≥1 criterion;
  every consequential UC alternate/exception path is represented; binding
  NFRs appear as concrete criteria or constraints. **For an FR the feature
  implements partially** (the plan's touchpoint says "partial"), the standard
  scopes to the part this feature delivers per the plan and design — neither
  demand the whole FR from a partial feature, nor let the partial pass read
  as the whole (the Test ref append will carry the `(partial)` marker). A
  missing criterion is a **design-defect finding** (the standard was
  incomplete), not a silent patch-up.
- **Fidelity**: each criterion says what its FR says. A criterion that
  quietly narrows "the system shall prevent X" into "shows a warning about X"
  is a misencoding — design-defect finding, cited both ways.

Tombstoned FRs/screens in the trace are findings too (stale design), routed
accordingly.

## Auditing the tests (Phase 3)

Per criterion, three questions in order:

1. **Covered?** A test (per-task or acceptance-level) exists whose subject is
   this criterion. Trace by the AC/FR references the tests carry; where
   references are missing, trace by behavior — and note the missing
   traceability as a minor finding.
2. **Faithful?** The assertion asserts the criterion's actual claim — the
   status code *and* the state change, the error *and* its shape, the
   boundary value *at* the boundary. The named weakening patterns to hunt:
   asserting only that "it didn't throw"; tolerances wider than the
   criterion; testing the happy fragment of a criterion that names an
   exception path; asserting against a mock of the behavior under test.
3. **Exercised?** The test actually runs in the suite (not skipped, not
   excluded from the runner's config) and fails when it should — for suspect
   tests, the **mutation check**: temporarily break the behavior, confirm the
   test goes red, restore. A test never seen to fail is unverified
   verification.

**Rejection and correction.** A test failing 2 or 3 is rejected. This skill
corrects it — rewritten to assert the criterion faithfully, committed
separately (`FEAT-NNN acceptance: correct test for AC-k`, message stating
what was weak and why the correction is faithful). Corrections move the test
**toward the criterion**, never toward the code: if the faithful test then
fails against the code, that's a rework finding, which is the system working.

## The independent anti-fake-green review (Phase 3, second half)

Diff the feature's test files across its commit range (`FEAT-NNN T1` … head):
loosened assertions, widened tolerances, new skips/todos, deleted cases,
behavior-under-test newly mocked. Implementation's own gate checked this;
this review is the second pair of eyes on the one rule most worth
double-checking. Any hit is examined against its commit message — a declared
fix-toward-the-design with a sound rationale passes; anything else is a
finding (severity by effect: a weakened acceptance assertion is rework-class
at least).

## Independent execution (Phase 4)

- From the actual repo head, with the harness's own commands
  (scaffold-notes/agent-instructions — never improvised invocations).
- The order that localizes failures: feature suite → whole-repo suite → E2E
  (including this feature's owed path) → migrations up (and down where
  supported) against a fresh local store.
- Every run's observed result goes in the report — including flaky behavior
  (a test passing on retry is a finding of test quality, recorded, not
  laundered into green).

## Direct requirement verification (Phase 5)

Tests verify what tests can see. Verify the rest by observation:

- **Behavioral FRs with side effects**: inspect the effect itself — the audit
  log's entries, the sent-message stub's capture, the retention job's
  deletion — not just the API's 200.
- **Binding NFRs, measured where cheap**: time the response bound against the
  local stack (indicative, environment-caveated in the report); check the
  payload-size cap, the pagination limit. NFRs needing real infrastructure
  (load, availability) are recorded **pending environment** with what would
  verify them — never silently skipped, never fake-verified locally.
- **Screens**: spot-check against manifest entries — the states claimed
  covered actually render (empty/loading/error), conformance claims hold on
  inspection (tokens, not raw values, in the rendered result), locators still
  resolve.
- **Contracts**: request/response/error shapes as implemented match
  technical-design §3 — divergences either carry a recorded
  deviation/escalation from implementation (fine) or are findings (not fine).

## Severity discipline

Every finding is one of: **rework** (code fails a faithful check),
**design defect** (the standard itself is wrong/incomplete/unsatisfiable), or
**minor** (traceability gaps, test-quality notes — recorded, non-blocking,
but accumulating minors on one feature is itself a rework-class signal).
Classify at discovery; the verdict aggregates (see verdict-and-report.md).
