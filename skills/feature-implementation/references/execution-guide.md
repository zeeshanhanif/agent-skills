# Execution Guide (the task loop)

Mechanics for Phase 3 — one task at a time, in tasks.md order, each executed
the same way. The loop's invariant: **after every task, the repo is in a
state a fresh session can pick up cold** — checkboxes current, work
committed, nothing important only in memory.

## Per-task cycle

1. **Read the task**: its done-when, the design sections it points at
   (technical-design §N / ui-design section), and the FR/UC it serves. If the
   task references a design section that doesn't answer what implementation
   needs, that's a divergence (see failure-and-escalation.md), not a license
   to invent.
2. **Survey the touchpoint**: the files this task changes, the conventions in
   force there (technical-design §2 recorded them; the code confirms them),
   the stubs to replace (skeleton stubs carry comments naming their
   replacing feature).
3. **Implement** the smallest change that satisfies the design. Follow the
   established envelope/error/naming conventions exactly — a second
   convention is a defect even if internally clean.
4. **Satisfy the done-when in its stated kind.** tasks.md writes each
   done-when as the right kind for the task's nature; execute the
   classification, don't improvise it:
   - **Behavioral tasks** (logic, decisions, transformations, contract
     semantics, validation) — the done-when names test artifacts: author them
     with the task, in the inherited frameworks (from the harness — never
     introduce a different runner), tracing to the acceptance criteria (name
     the AC/FR in the test description where natural). They are the done-when
     made runnable and rerunnable.
   - **Structural/realization tasks** (schema objects, wiring, realizing a
     screen spec, config) — the done-when is a demonstration: it applied
     cleanly, it builds, it renders and conforms. Zero new test files is
     legitimate here; the behavior gets covered by the tasks that build logic
     on the structure. Never manufacture ceremony tests — and never skip the
     demonstration either. Verification is universal; test artifacts are not.
5. **Run the done-when** — actually run it: the test command from
   scaffold-notes/agent-instructions, the migration against the local store,
   the rendered screen against its spec. Green → proceed; red → the fix-loop.
   **Verification timing is three-tier**: a task's own done-when is all that
   runs per task (task-scoped, fast — never the E2E suite, which mid-feature
   has no complete path to traverse); the feature's E2E extension runs at its
   own pre-minted task, ordered after wiring/UI so the path it tests exists;
   the **full suites** — whole-repo tests, complete E2E including prior
   features' flows — run once, at the final verification task's gate.
6. **Task hygiene, then commit**: boundary/lint rules pass; no debug
   leftovers; tasks.md box checked; commit
   `FEAT-NNN T<k>: <imperative summary>`. One task, one commit — don't batch.

## UI tasks specifically

- Screens come from the manifest entry: **registered/generated** → realize
  from the source locator, conformance rules still binding (the tool design
  is the layout authority; the design system is the values authority);
  **code-native** → build from the spec section, then the **screenshot loop**:
  render, screenshot, compare against the spec and design.md, fix, re-render —
  bounded by the same attempt rule as everything else.
- **Tokens, never raw values.** design.md's Do/Don'ts are binding. A needed
  value with no token is a design-system gap → escalation path, not a
  hard-coded hex.
- State supplements in the manifest (empty/loading/error specs) are part of
  the screen — not optional polish.

## The E2E task (when pre-minted)

tasks.md carries it only when the feature touches an architecture-named
critical flow. Extend the existing suite in the E2E workspace — same
framework, same conventions as the skeleton seed — to cover the path this
feature completes. It runs against the real local stack; a red E2E is a real
integration failure (diagnose which side), never a reason to loosen the spec.

## The final verification task

Always last, always present — and **it executes under the same per-task
cycle and bounded fix-loop as every other task**: a failing criterion or a
red suite here triggers diagnose → fix → re-run, up to the attempt bound
(each fix committed against the responsible earlier work), and exhaustion
stops the feature short of developer-done with the honest blocked state.
Before spending attempts, classify the red: code wrong → fix it (the normal
case); the acceptance test misencodes a criterion → fix the test *toward the
design* (legitimate, doesn't burn the budget the same way — record it in the
commit); the criterion itself unsatisfiable as designed → **divergence, stop
immediately and escalate** — no attempts are owed to a broken design.

Author the acceptance-level tests here if the
per-task tests don't already cover every criterion in technical-design §6 —
each criterion demonstrably checked, cited by AC/FR ID. Then the full gate:
feature suite green, whole-repo suite green (this feature broke nothing),
boundaries clean, E2E (when owed) green, no WIP markers in tasks.md or code.
Only then is the feature developer-done.

## Session discipline

- **One task per iteration** is the unit of progress; a session may run many
  iterations, but each completes (or cleanly blocks) its task before the next
  begins. Never two tasks half-done.
- **Resume protocol**: on entry, the first unchecked box is the position; if
  it carries a WIP note, that note is the starting context — verify the
  described state against the actual code (the note may predate a human's
  intervention), then continue inside the remaining attempt budget.
- tasks.md is updated **in place, boxes only** — never restructured. A task
  that proves wrong-sized or wrongly ordered means the design changed:
  escalate, don't edit the program mid-run.
