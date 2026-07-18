# Verification (Developer-Done Gate)

Phase 4's contract — run when tasks.md's final verification task executes.
Everything here is demonstrated by running, never asserted. This gate makes
the feature **developer-done**; acceptance verification independently audits
it afterward — this skill never grades its own homework alone, which is
exactly why the honest execution of this checklist matters: manufactured
green here fails louder there.

## 1. Task completeness

- Every task box in tasks.md is checked — including the E2E task when the
  feature owed one.
- No WIP markers remain: no WIP commits unresolved at the head of history for
  this feature, no failure notes still describing open blocks, no `WIP`/`TODO
  (blocked)` markers in the feature's code.
- The git log contains one commit per task (`FEAT-NNN T<k>: …`), in task
  order — the readable execution log.

## 2. Acceptance criteria — demonstrably

- Every criterion in technical-design §6 has an automated check (per-task
  tests or the acceptance-level tests authored in the final task), traceable
  by AC/FR reference, and **it ran green in this session** — not "should
  pass," ran.
- No criterion was reinterpreted to fit the code: the check asserts what the
  criterion says. Doubt → divergence path, not a softer assertion.

## 3. Suites and rules — the full gate

- The **feature's tests** green. The **whole-repo suite** green — this
  feature broke nothing (a red elsewhere is this feature's problem until
  proven otherwise).
- **Boundary/lint rules** pass repo-wide.
- The **E2E suite** green, including the flow path this feature extended
  (when owed).
- Migrations: the feature's migrations apply cleanly to a fresh local store
  (up), and down where the mechanism supports it.

## 4. Design conformance

- Contracts as implemented match technical-design §3 — spot-check request/
  response/error shapes against the document; any deviation is a recorded
  small-divergence or a filed escalation, never silent.
- UI: screens match their manifest sources; **no raw values where tokens
  exist** (grep the feature's UI diff for hex/px literals that design.md
  tokenizes); state supplements implemented.
- Scope: the diff is the feature — `git diff` against the pre-feature state
  touches only files the tasks required (plus their tests). Anything else is
  either reverted or explained in the summary.

## 5. Anti-fake-green audit (self-check with teeth)

- No test was weakened, skipped, or deleted this feature: review the diff of
  test files for loosened assertions, new skips, removed cases, or
  mocked-away behavior under test. Any legitimate test fix moved *toward the
  design* and says so in its commit message.

## Reporting

Close with the honest line: "developer-done — all criteria demonstrated,
suites green" plus escalations filed and debt recorded — or the blocked
state: which task, attempts used, current hypothesis, WIP commit ref. The
user (and the next skill) never discovers a state this skill knew about and
didn't report.
