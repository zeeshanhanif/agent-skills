# Checkpointing and Resume Protocol

A full requirements interview is long. This protocol ensures that a stop, a
closed session, or a network error never loses captured work. The core idea:
**the deliverable is the checkpoint.** Finalized content is written straight into
the real output files as it is produced, and a small tracker records progress —
so partial work is always real content on disk, never a throwaway scratchpad.

## The progress tracker

Maintain `docs/.requirements-progress.md`. It lists every area of the interview
and its status. Update it at each **natural boundary** — after a whole capability
area, after an NFR category, after a use case — not after every message (that
would be noise). Suggested format:

```markdown
# Requirements Progress

> Session source of truth. Statuses: pending | in-progress | done
> Last updated: <timestamp>

## Phase 1 — Frame & scope
- [done] Purpose & vision
- [done] Goals & success metrics
- [done] Stakeholders & user classes
- [in-progress] Scope & MVP boundary

## Phase 2 — Functional areas
- [done] Authentication & account management  (FR-AUTH-001..018)
- [pending] Authorization & roles
- [pending] User profile & settings
- [pending] Notifications
- [pending] ...

## Phase 3 — Non-functional
- [pending] Performance, Security, Availability, ...

## Phase 4 — External interfaces
- [pending]

## Phase 5 — Use cases
- [pending]

## Phase 6 — RTM
- [pending]

## Phase 7 — Word docs
- [pending]
```

Recording the ID range next to a completed area (e.g., `FR-AUTH-001..018`) makes
it easy to resume without re-reading the whole SRS, and keeps ID numbering
collision-free across sessions.

## Writing incrementally

As each area is finalized, append its formal requirements directly into
`docs/srs.md` (and each completed use case into `docs/use-cases.md`). Do not hold
a whole interview's worth of content in memory to write at the end — write area
by area. This means that at any interruption point, `docs/srs.md` already
contains every completed area.

Keep a short, stable section skeleton in `docs/srs.md` from the start (the
section headings from `srs-template.md`), and fill sections in place as areas
complete. That way the document is always well-formed, just partially populated.

## Resuming (Phase 0)

On every invocation, before doing anything else:

1. Check for `docs/.requirements-progress.md` and `docs/srs.md`.
2. **If neither exists:** this is a fresh start. Create the progress tracker and
   the SRS skeleton, then begin Phase 1.
3. **If they exist:** read both. Summarize for the user concisely — what's
   captured (areas + counts) and what's pending — and confirm before continuing.
   Resume at the first `pending` or `in-progress` area. Re-read the relevant
   finalized content only as needed (the tracker's ID ranges tell you where
   numbering left off). **Do not restart completed areas or renumber existing
   requirements.**

## Honest limits

This is instruction-driven incremental saving, not OS-level autosave. It persists
at each completed boundary, so a crash *mid-area* loses only that in-progress
area, not the ones already written. It depends on a persistent working directory
(present in Claude Code and Cowork). The progress and draft files are working
files; only the five finalized outputs are deliverables. If the user wants a
clean repo, `docs/.requirements-progress.md` can be deleted (or git-ignored)
after delivery.
