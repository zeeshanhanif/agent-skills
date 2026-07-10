# Verification (Empirical — Run It)

Phase 7's contract. Unlike the document skills, verification here means
**executing things and observing results** — a claim about the skeleton that
wasn't demonstrated by a run is not verified. Fix-loop on failures; anything
genuinely unfixable is flagged in the delivery summary with its cause — never
silently shipped.

Run these in order (each depends on the previous):

## 1. Clean install

From a clean state (fresh clone semantics — at minimum, wiped dependency
directories): the documented install command(s) complete without errors on
this machine. Record the exact commands in scaffold-notes — they become the
README truth.

## 2. Everything builds

Every unit's build command succeeds. Type checks pass where the stack has
them.

## 3. The skeleton test passes

The end-to-end test (skeleton-guide) runs green locally, with the local data
store up. This **is** the plan's done-when condition, local half. State the
result against the done-when's wording explicitly. The deployed half is
**pending first deploy** — recorded as such in scaffold-notes and the delivery
summary, never claimed.

## 4. Boundaries are enforced, not decorative

The lint/import-boundary rules run and pass — and prove they *work*:
temporarily introduce one forbidden cross-boundary import, confirm the tooling
rejects it, remove it. An enforcement rule that was never seen to fail is
unverified.

## 5. Design-system wiring is real

The shell renders using token-derived values. Spot-check: a token value from
tokens.json appears in the rendered shell via the wiring (not hand-copied).
The agent-instructions file references design.md and tokens.json at paths that
resolve.

## 6. CI config is valid

The pipeline config parses/validates by the CI system's own checker where
available locally; jobs reference commands that exist. (Actually running CI
happens on first push — note it as pending alongside first deploy if the repo
hasn't been pushed.)

## 7. Deployment config is coherent

Deployment artifacts (Dockerfiles, service configs, IaC skeleton) are
syntactically valid by their own tools' check/validate commands where
available locally. Not executed, not provisioned — validity only.

## 8. Repo hygiene

`docs/` present with the pipeline documents; scaffold-notes complete
(preflight, generators+versions+flags, deviations, removals, decisions);
progress tracker marked complete; no committed secrets (scan for obvious
patterns); agent-instructions paths all resolve; stubs are marked and point at
their replacing slices.

## Reporting

Close with one line in the delivery summary: "verification clean — skeleton
green locally; first deploy and first CI run pending" or the flagged list with
causes. The user should never discover a red skeleton the skill knew about.
