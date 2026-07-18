# Verdict and Report Guide

Phase 6: aggregate the findings into one verdict, write the report, route the
consequences. The rules that keep verdicts honest:

- **A feature is verified whole or not yet** — partial acceptance is not a
  verdict. Mixed findings take the most severe class present.
- **Verdicts route; they never fix.** Rework goes back to
  feature-implementation; design defects go to detailed-design's amendment
  path (which may escalate further per its own rules — schema to
  architecture, screens to the design system). This skill's only writes:
  the report, corrected tests (Phase 3), and the RTM Test ref on acceptance.
- **Re-verification after rework or amendment is a normal run** — same
  standard re-derived (amended documents produce an updated standard), full
  execution again; the report gains a new dated verdict section, prior
  verdicts preserved.

## The three verdicts

- **Accepted** — no rework or design-defect findings; suites observed green;
  direct verification passed (pending-environment NFRs listed, not blocking
  by default — flag prominently if the user should treat them as blocking).
  Consequences: RTM Test ref appended; the feature is *verified*, the loop's
  true done.
- **Rework** — the code fails faithful checks. Consequences: no RTM write;
  each finding cited (AC/FR, the faithful expectation, the observed result,
  where it lives) so implementation acts without re-deriving context.
- **Design defect** — the standard is wrong: missing/misencoded criteria,
  unsatisfiable-as-designed behavior, stale references. Consequences: no RTM
  write; the defect stated both ways (what the source requires vs. what the
  design says) toward the amendment path.

## acceptance-report.md (in the feature folder)

```markdown
# Acceptance Report: FEAT-NNN — <Feature name>

> Verdict: Accepted | Rework | Design defect · Date: <date>
> Standard: technical-design.md §6 @ <commit> · Sources: srs.md, use-cases.md
> Repo state audited: <commit>

## Verdict summary
Three or four sentences: the verdict, the decisive findings (or their
absence), and what happens next.

## Audit table
| AC | Encodes | Test(s) | Audit | Observed |
| :- | :------ | :------ | :---- | :------- |
| AC-1 | FR-AUTH-002, UC-002 main | auth.login.spec | faithful | green |
| AC-3 | FR-AUTH-004, UC-002 exc-2 | corrected (see below) | was weak → corrected | green |
*(one row per criterion — the source → criterion → test → observation chain)*

## Corrected tests
Each Phase-3 correction: what was weak, the faithful assertion now made, the
commit ref. "None" is the happy case.

## Independent execution
Suites run, commands used, observed results (incl. flakiness), migration
check, E2E path result.

## Direct verification
Side-effect FR observations; NFR measurements (values, environment caveat);
**pending environment** list with what would verify each; screen/contract
spot-check results.

## Findings
By severity — rework / design defect / minor — each specific enough to act
on. Empty sections stay present, marked "none".

## RTM
Accepted: the FR rows whose Test ref was appended (and the ref written).
Otherwise: "not written — verdict <class>".
```

## The RTM Test ref (on acceptance only)

Append into the **Test ref** cell of every FR the feature implements: the
report path (`features/FEAT-NNN-<slug>/acceptance-report.md`) — the report is
the verification record; the audit table inside it names the tests. **When the
feature implements the FR partially** (the plan's touchpoint says "partial"),
the append carries the marker: `…/acceptance-report.md (partial)` — an
accurate, permanent description of *that report's scope*, never edited or
upgraded later. **An FR's full verification is computed, never stored**: it is
fully verified when every feature in its Plan ref has an accepted report in
its Test ref — the completing feature's own append *is* the transition; no
prior entry changes. (This also handles amendments correctly: a re-opened FR
becomes partially-verified again by the computation alone.) Rules as
established: append never overwrite; Test ref column only; silent skip when
no rtm.md; tombstoned rows untouched. This completes the row's lifecycle —
Plan ref → Design ref → Test ref — a requirement traceable from birth to
verified.

## Delivery summary

The verdict line, the findings count by class, the RTM action taken, and the
live-computed loop position ("FEAT-004 verified — 3 of 14 features verified";
computed from accepted reports vs. the plan, never stored). Handoff:
accepted → the loop's next feature (or, when all features are verified, the
plan is delivered); rework/defect → the named owner with the report as the
work order.
