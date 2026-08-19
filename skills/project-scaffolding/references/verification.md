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

The end-to-end test (skeleton-guide) runs green locally against a
**cold-started** local stack — brought up from the committed compose file in
this run (`down` then `up`), not against a store that happened to already be
running; for a recorded deviation (emulator or embedded store), the
equivalent cold start. The up/down/reset commands are recorded in
scaffold-notes. The test runs **in the E2E workspace, using the
architecture-named E2E framework** (and the unit harnesses match the architecture's named runners;
any silent-architecture fallback is noted in scaffold-notes). **The coverage
configuration matches the architecture's stance**: enforced → the gate is
present at the stated threshold and was seen executing in the skeleton's own
CI-config run; report-only → the report generates; none → no coverage
tooling exists. **The skeleton ran on config-sourced values** — every unit
read its configuration from the environment, not from hardcoded constants
(spot-check the skeleton path for baked-in hosts/ports/URLs). This **is** the
plan's done-when condition, local half. State the result against the
done-when's wording explicitly. The deployed half is
**pending initial deployment** — recorded as such in scaffold-notes and the delivery
summary, never claimed.

**Leave the machine as it was found.** Processes this run started to prove
the skeleton — dev servers, workers, the local stack — are stopped at
delivery, or the summary states plainly what is still running and the command
to stop it. Stop only what this run started, by its own handle; never kill by
port scan or name pattern (that port may be the user's editor or another
project). This is the exit-side counterpart to the cold-start rule above.

## 4. Boundaries are enforced, not decorative

The lint/import-boundary rules run and pass — and prove they *work*:
temporarily introduce one forbidden cross-boundary import, confirm the tooling
rejects it, remove it. An enforcement rule that was never seen to fail is
unverified.

## 5. Design-system wiring is real

The shell renders using token-derived values. Spot-check: a token value from
tokens.json appears in the rendered shell via the wiring (not hand-copied).
AGENTS.md references design.md and tokens.json at paths that
resolve.

## 6. CI config is valid

The pipeline config parses/validates by the CI system's own checker where
available locally; jobs reference commands that exist. (Actually running CI
happens on first push — note it as pending alongside the initial deployment if the repo
hasn't been pushed.)

## 7. Deployment config is coherent

Deployment artifacts (Dockerfiles, service configs, IaC skeleton) are
syntactically valid by their own tools' check/validate commands where
available locally. Not executed, not provisioned — validity only.

## 8. Repo hygiene

**No generator boilerplate README survives** — the root README describes
this system (units, prerequisites, local stack, tests, docs), and each unit
README that exists is unit-scoped and links up rather than duplicating;
**`AGENTS.md` at the root carries the substance and `CLAUDE.md` contains only
the `@AGENTS.md` pointer**; no tool was installed globally without an
explicit ask.
`docs/` present with the pipeline documents; scaffold-notes complete
(preflight, generators+versions+flags, **local stack: up/down/reset commands
and any store deviation with its reason**, deviations, removals, decisions);
progress tracker marked complete; **a config template per deployable unit,
placeholders only — no working values and no secrets, with the local config
file generated and gitignored**; no committed secrets (scan for obvious
patterns, and confirm the local config is not tracked);
AGENTS.md paths all resolve; stubs are marked and point at
their replacing slices.

## Reporting

Close with one line in the delivery summary: "verification clean — skeleton
green locally; initial deployment and first CI run pending" or the flagged list with
causes. The user should never discover a red skeleton the skill knew about.
