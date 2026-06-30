# Change Management (Amending a Finalized SRS)

This protocol governs changes to requirements **after** the SRS is finalized. The
hard problem is not editing text — it's keeping five interdependent files
consistent while preserving the IDs that every downstream skill (architecture,
ux-foundations, implementation-planning) references. The whole protocol exists to
make change **ID-stable and additive**, never a free rewrite.

## The cardinal rule: IDs are immutable and never recycled

Once assigned, a requirement ID never changes meaning-preservingly and is never
reused for something else. This is non-negotiable because downstream documents
cite requirements by ID; renumbering silently corrupts the RTM and every document
built on it. Concretely:

- **Add** → new ID. Always safe — nothing references it yet.
- **Modify** → same ID, new statement. The dangerous case: the ID still resolves
  but its meaning shifted, so downstream work built on it may be stale.
- **Remove** → tombstone, don't delete. Mark `Deprecated`/`Removed`, keep the row.
  Deleting and renumbering is what breaks references; a tombstone keeps dangling
  references visible instead of silent.

## Per-operation rules

### Add a requirement
1. Identify the area; find the highest existing ID in that area (the progress
   tracker records ID ranges) and assign the next number. Never fill a gap left
   by a removed requirement.
2. Specify it with the full enumeration discipline (testable "shall" statement,
   priority, rules) — an addition gets the same rigor as original elicitation,
   including proposing related sub-requirements it implies.
3. Append to the SRS, add any use case(s) it needs, add RTM row(s).

### Modify a requirement
1. Keep the ID. Rewrite the statement/rules.
2. Check its dependents via the RTM: the use case(s) that trace to it may need
   updating; flag them.
3. Because the meaning changed, this triggers a cross-skill impact note (below).

### Remove a requirement
1. Do not delete the row or renumber anything. Set its status to
   `Removed`/`Deprecated` with the version it was removed in.
2. Mark use cases that traced solely to it as affected; update the RTM row to
   reflect removal.
3. Triggers a cross-skill impact note.

## Versioning and revision history

Every amendment session bumps the SRS version (e.g., 1.2 → 1.3) and adds a row to
the **Revision History** section at the top of `docs/srs.md`:

```markdown
## Revision History
| Version | Date | Author | Changes |
| :------ | :--- | :----- | :------ |
| 1.3 | 2026-06-25 | <name> | Added FR-NOTIF-009..011 (SMS alerts); modified FR-AUTH-007 (lockout threshold 5→10); removed FR-RPT-004. |
| 1.2 | 2026-06-10 | <name> | … |
```

This is standard for a formal SRS and is the human-readable audit trail that
complements the RTM.

## Propagation checklist (run after every amendment)

A change is not done until all three files are consistent:

1. **`srs.md`** — requirement added/modified/tombstoned; version bumped; revision
   history updated.
2. **`use-cases.md`** — affected use cases updated or flagged; new use cases added
   for new requirements that need them.
3. **`rtm.md`** — rows added/updated; removed requirements marked; traceability
   intact.
4. **Progress tracker** — record the amendment and the new ID ranges.

## Cross-skill impact notes

When a **modify** or **remove** touches a requirement that downstream documents
were built on, this skill cannot edit `architecture.md`, `ux-foundations.md`, or
`implementation-plan.md` — but it must surface the impact so they don't silently
drift. After the amendment, emit a short note, e.g.:

```
Impact: FR-AUTH-007 modified, FR-RPT-004 removed.
Downstream documents may reference these:
 - docs/architecture.md (security NFR mapping) — review
 - docs/implementation-plan.md (slice "Reporting export") — review
Re-run the affected skills if these requirements shaped their decisions.
```

Derive the list by checking which downstream docs exist and searching them for
the affected IDs. This is cheap and is exactly what keeps the SRS and the
downstream documents from diverging — the requirements-side complement to those
skills reading the SRS by ID.

## Mode interaction

If the SRS exists but the interview is **unfinished** (tracker has pending areas),
resume and finish those areas *before* taking amendment requests — amending a
half-specified SRS invites inconsistency. Amendment mode applies only to a
**finalized** SRS.
