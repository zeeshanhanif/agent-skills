# Verification (Demonstrated Live)

Phase 9's contract. Everything checked here was **observed against the
deployed environment in this run** — provider status pages, live probes,
real pipeline executions. A claim without an observation is unverified. Fix
what fails; anything genuinely unfixable is flagged in the delivery summary
with its cause — never silently shipped.

## 1. Provisioned reality matches the plan

- Every resource the confirmed deployment plan named exists and reports healthy
  by the provider's own status; nothing material exists that the plan (plus
  recorded deviations) doesn't account for.
- Deviations from the deployment view are each recorded with a reason — or
  escalated as architecture amendments. The tracker
  (`docs/.deployment-progress.md`) is complete and matches provider reality.

## 2. The pipeline ships

- A **real CD run** was observed: commit/tag → pipeline → deployed. The
  promotion gates match the architecture's cadence (or the recorded manual
  default).
- Rollback is demonstrated or concretely documented: the exact command/steps
  to return to the previous artifact, recorded in deployment-notes — and where
  the platform makes it cheap (revision pinning), actually exercised once.

## 3. The skeleton's done-when — deployed half closed

- The end-to-end exercise ran **against the deployed environment** and the
  skeleton path passed; the result is stated against the plan's done-when
  wording; deployment-notes and the scaffold-notes pending item both updated.
  This is the run's headline claim — it gets its own line in the summary.
- **Feature E2E paths** (when the suite has any) also ran against the
  deployed environment; results recorded. Reds here are deployment findings
  with their routing stated (environment/config issue → fixed here; genuine
  feature failure → re-run acceptance-verification), never folded into the
  skeleton verdict and never left unreported.

## 4. Secrets hygiene

- Services read every secret from the store (no plaintext env values in
  configs/artifacts); the canary flow was exercised; **no secret value
  appears in anything this skill wrote** — deployment-notes, configs, commit
  history, chat. Grep the diff for high-entropy strings as a final check.

## 5. The operations floor holds

- Uptime probe live and its alert **was received once** (test-fire).
- Error alert **was received once** (controlled event).
- Logs reachable via the recorded commands.
- **Restore was performed** — scratch target, data round-trip verified,
  scratch destroyed. The procedure is in deployment-notes verbatim.
- TLS/domain per the architecture (or the recorded platform default);
  HTTPS enforced.
- Billing alert set at the user's threshold.

## 6. Pending-environment measurements

- Each acceptance-report pending-environment item was either measured (result
  + environment context in deployment-notes) or explicitly recorded as still
  pending with what it awaits. The recommendation to re-run
  acceptance-verification for formal verdicts is in the summary — this
  skill measured; the auditor rules.

## 7. Notes completeness

deployment-notes lets a cold session operate the deployment: environment URLs,
deploy + rollback commands, secret-store locations (never values), restore
procedure, alert channels, observed/expected costs, verified CLI command
forms, deviations log.

## Reporting

The delivery summary leads with the headline ("deployed — skeleton
end-to-end demonstrated live; CD shipping on merge"), then: URLs, cost
expectation, alerts confirmed, restore proven, pending items (with owners),
and the acceptance-verification re-run recommendation when measurements
were taken.
