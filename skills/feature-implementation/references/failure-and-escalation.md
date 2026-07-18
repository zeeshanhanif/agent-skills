# Failure and Escalation Guide

What happens when things don't go green. Two distinct situations with two
distinct protocols — a **failure** (the design is right, the code isn't yet)
and a **divergence** (reality contradicts the design). Misclassifying the
second as the first is how designs rot; handle them separately.

## The bounded fix-loop (failures)

At most **3 attempts per task** — or the project's stated override: a
"fix attempts: N" line in the agent-instructions file (set it deliberately;
higher isn't better — a task failing 5 times is telling you something).

Each attempt is a real attempt, not a flail:
1. **Diagnose first**: read the actual error, locate the cause, form a
   hypothesis. An attempt without a changed hypothesis is a wasted attempt.
2. **Fix the cause**, not the symptom — and never the test (see below).
3. **Re-run the done-when.** Green → continue the task cycle. Red → next
   attempt with what was learned.

**Exhausted → stop honestly:**
- Leave the checkbox **unchecked**.
- Write the failure note in tasks.md under the task: what was tried (each
  hypothesis), the current error verbatim, the best remaining hypothesis, and
  any state a resumer must know.
- **WIP-commit**: `FEAT-NNN T<k>: WIP — blocked, see tasks.md note`. Durable
  failure state beats a lost session; the explicit WIP marker keeps red code
  from masquerading as done.
- Surface it in the session summary and stop the loop — do not skip ahead to
  later tasks (order is load-bearing; a blocked T3 usually poisons T4).

## The anti-fake-green rule (absolute)

Tests are never weakened, skipped, deleted, or edited to make them pass. The
forbidden moves, named: loosening an assertion, widening a tolerance,
commenting out a case, marking skip/todo, deleting the test, mocking away the
behavior under test, catching-and-ignoring the failure. A red test has
exactly two legitimate resolutions: **the code is wrong → fix the code**, or
**the design (and thus the test derived from it) is wrong → that's a
divergence, escalate it**. There is no third path. If a test itself has a
genuine bug (asserts something the design never said), fixing it *toward the
design* is legitimate — record it in the commit message; fixing it *toward
the code* is the forbidden move wearing a costume.

## Divergences (reality beats the design)

Detected when implementation finds the design can't be built as written: the
library's actual API contradicts the contract, the schema misses a column the
logic needs, a screen spec doesn't survive contact with real data shapes.

**Two paths, chosen by blast radius:**
- **Small and design-consistent** — the design's *intent* is implementable
  with a trivially different mechanism (a renamed method, an equivalent
  type): implement the intent, record the deviation in the task's commit
  message and a one-liner in technical-design §8 territory via the
  escalations note in the delivery summary. No amendment needed.
- **Design-changing** — anything that alters a **contract shape, schema,
  acceptance criterion, or screen spec**: **stop the task; escalate.** File
  the issue toward the owning skill's amendment path — technical matters to
  detailed-design (which owns technical-design.md and may in turn escalate to
  architecture per its own schema rule), screen matters to ui-design (which
  may escalate to the design system). State precisely: what the design says,
  what reality is, the proposed change, which ACs/tasks it touches. Then
  either wait, or — with the user's explicit go — proceed on the stated
  assumption the amendment will land, marked in the failure-note style.
  **Never implement the divergent version silently**: the design documents
  are what acceptance verification will audit against; code that drifted from
  them fails honestly later at twice the cost.

## Tech debt discovered en route

Pre-existing problems the tasks didn't cause (a smelly module the feature
touches, a missing index elsewhere): **record, don't act.** One line each in
the delivery summary's recorded-debt list, concrete enough to become a plan
item. Acting on them violates scope discipline and makes the diff
unreviewable — the plan is where debt gets scheduled, not the middle of a
feature.

## Repeated blocking (the meta-signal)

The same task blocking across multiple sessions, or several tasks of one
feature exhausting their budgets, is not bad luck — it's evidence the
feature's design has a systematic problem. Say so explicitly in the summary
and recommend the feature go back to detailed-design rather than burning more
attempt budgets.
