---
name: feature-implementation
description: >-
  Executes a feature's tasks.md against its designs, producing working,
  tested, committed code — the construction step of the per-feature loop. Runs
  after detailed-design and ui-design, autonomously: one task at a time in
  order, each task's done-when demonstrated by actually running it (tests
  pass, migrations apply), committed per task, until every task including the
  final verification task is checked. Tests are written per task in the
  project's established frameworks and are never weakened, skipped, or edited
  to pass. Diverging reality escalates to the design's amendment path — the
  skill implements the design; it never quietly redesigns. Bounded fix
  attempts per task (default 3); a blocked task stops with an honest WIP
  state, never a fake green. Use per feature, after its technical-design.md
  and ui-design.md exist. Trigger on "implement this feature", "implement the
  next feature", "execute the tasks", "build FEAT-N", or "continue the
  implementation". Scope: this feature only — no drive-by refactors.
---

# Feature Implementation

The construction step of the per-feature loop: documents become code.
**tasks.md is the program; this skill is the interpreter.** It executes the
feature's tasks in order against `technical-design.md` (contracts, schema,
components), `ui-design.md` + the design manifest (screens), and the live
codebase — producing working, tested, committed code that replaces the
skeleton's stubs, ending developer-done: every task checked, including the
final verification task. The independent check is downstream
(acceptance verification); this skill never grades its own homework alone.

Seven disciplines make it rock-solid. Each targets a named failure mode of
agentic implementation:

1. **The program is fixed; improvisation is escalation.** Implement what
   tasks.md says, per the design documents. Reality will sometimes beat the
   design — that has two paths, never a third: small and design-consistent →
   fix and record; anything changing a contract, schema, or acceptance
   criterion → **escalate to the design's amendment path** (see
   `references/failure-and-escalation.md`). The design documents stay true,
   or they change through their owner. *(Kills: quietly designing around
   divergence.)*
2. **Fresh-context safety — disk is the memory.** Every iteration is
   executable in a brand-new session: read tasks.md (checkbox state = exact
   position), read the designs, read the code, do **one task**, update state
   on disk, commit. No iteration depends on chat history. *(Kills: context
   loss mid-feature; enables loop orchestration.)*
3. **Done-when is demonstrated, never asserted.** A checkbox flips only after
   the task's done-when actually ran and passed — the migration applied clean,
   the tests passed, the flow's E2E spec passed. And the **anti-fake-green
   rule**: tests are never weakened, skipped, deleted, or edited to make them
   pass; a failing test means the code or the design is wrong, and each has
   its path (fix, or escalate). *(Kills: the single most corrosive agent
   failure — manufacturing green.)*
4. **Bounded fix-loops — never thrash.** Diagnose, fix, re-run — at most
   **3 attempts per task** (per-project override: see Inputs). Exhausted →
   stop, leave the box unchecked, record the failure state honestly, WIP-
   commit, surface it. *(Kills: infinite loops and desperation hacks.)*
5. **Scope discipline — this feature only.** Touch what the tasks require.
   No drive-by refactors, no "while I'm here" improvements, no reformatting
   neighboring code. Genuine tech-debt discoveries are **recorded** (the
   feature's escalations section), never acted on. *(Kills: scope creep and
   unreviewable diffs.)*
6. **Convention conformance by construction.** Code follows the conventions
   technical-design §2 surveyed. UI realizes screens from ui-design.md/the
   manifest **through the design system** — tokens, never raw values;
   design.md's do/don'ts are binding — with the screenshot loop for
   code-native screens (bounded, per discipline 4). Boundary/lint rules run
   as part of every task's verification, not at the end. *(Kills: convention
   drift and design-system forks.)*
7. **Git is the checkpoint mechanism.** Commit per completed task —
   `FEAT-004 T3: implement POST /auth/login contract` — so every task is a
   recovery point, history is a readable execution log, and an orchestrator
   resumes from durable state. A blocked task WIP-commits, explicitly marked
   (`FEAT-004 T4: WIP — blocked, see tasks.md note`); a failing state is
   never committed as if green. *(Kills: lost work and unrecoverable
   sessions.)*

## Inputs

Defaults; user paths win; source-gated citation throughout.

- **The feature folder** — `docs/features/FEAT-NNN-<slug>/`: **tasks.md** (the
  program; checkbox state is the position), **technical-design.md** (contracts,
  schema, components, acceptance criteria), **ui-design.md** (screen specs).
- **The design manifest** — `docs/design-manifest.json`: each screen's
  strategy and source; tool-backed screens are realized from their locators,
  code-native screens from their specs.
- **The design system** — `docs/design.md` + `docs/tokens.json`: binding for
  all UI code (tokens, never raw values).
- **The live codebase** — including the harness scaffolding installed (the
  test frameworks are **inherited from it**, never chosen here) and the
  boundary/lint rules.
- **Conventions & commands** — the agent-instructions file and
  `docs/scaffold-notes.md`: run/test commands, conventions, and any
  **per-project fix-attempt override** (a stated "fix attempts: N" in the
  agent-instructions file; default 3 when absent).
- **No RTM writes.** Implementation owns no RTM column — Design ref was
  written upstream; Test ref belongs to acceptance verification. Stated so
  nobody "helpfully" writes one.

## Workflow

### Phase 1 — Locate the feature and the position

The user names a feature (explicit naming is the only off-sequence path), or
**compute the default: walk the plan's build sequence in order; the first
feature whose tasks.md exists and has unchecked tasks is it.** Announce the
resolution ("FEAT-004 is next per the plan's build sequence; 3 of 7 tasks
remain") — never a menu. Within the feature, the position is the first
unchecked task. If the folder or its design docs are missing, stop and say
what's missing (the design pair runs first); never improvise a design.

### Phase 2 — Load the context (fresh every session)

Read tasks.md, both design documents, the manifest entries for the feature's
screens, and survey the code the next task touches — including any WIP note a
previous session left. Read the run/test commands from the agent-instructions/
scaffold-notes. Trust disk over memory; a WIP note beats any recollection.

### Phase 3 — Execute the task loop (autonomous)

Read `references/execution-guide.md`. For each task, in order, until done or
stopped:

1. **Implement** per the design and the codebase conventions.
2. **Satisfy the done-when in its stated kind** — author test artifacts where
   the done-when names them (behavioral tasks), demonstrate where it doesn't
   (structural/realization tasks); never skip verification either way.
   Test artifacts use the inherited frameworks; acceptance-level tests belong
   to the final verification task, the E2E extension to its pre-minted task.
3. **Demonstrate the done-when** — run it. Pass → check the box, run the
   boundary/lint rules, **commit** (`FEAT-NNN T<k>: <summary>`).
4. Fail → the bounded fix-loop (`references/failure-and-escalation.md`):
   diagnose, fix, re-run, at most the attempt bound; exhausted or
   design-divergent → stop/escalate per the guide.

Stop conditions (the only ones): all tasks checked; a task blocked after
exhausted attempts; an escalation that changes the design; the user said
stop. On any stop, state is already durable (checkboxes + commits + notes).

### Phase 4 — Feature-done verification

The final task in tasks.md *is* this phase: every acceptance criterion in
technical-design §6 demonstrably passes (the acceptance-level tests exist and
run green), the feature's suite is green, boundary/lint rules pass, the E2E
task (when owed) passed, no WIP markers remain. Read
`references/verification.md` and run its checklist. This is **developer-done**
— the independent audit is acceptance verification's job, next in the loop.

### Phase 5 — Deliver

Summarize: tasks completed (with the commit trail), tests added, escalations
filed or recorded debt, and the live-computed progress line ("FEAT-004
implemented — 4 of 14 features developer-done"; computed, never stored).
Handoff: **acceptance verification** independently audits this feature next.

## Scope boundaries

Does **not**: redesign anything (contracts/schema/criteria change only via
detailed-design's amendment path; screens via ui-design/the design system's);
restructure tasks.md (a needed restructure means the design changed — go back
one step); touch other features' code beyond what its tasks require; write
the RTM; deploy.

## What good looks like

- Every checked box has a passing done-when behind it and a commit that shows
  it — the git log reads as the execution log of tasks.md.
- Zero manufactured green: no test was weakened, skipped, or deleted to pass.
- Blocked states are honest and durable: unchecked box, WIP commit, a note a
  fresh session (or a human) can pick up cold.
- The diff is the feature — nothing else changed.
- UI code contains tokens, not raw values, and screens match their manifest
  sources.
- Escalations were filed the moment reality beat the design — never worked
  around, never batched to the end.
