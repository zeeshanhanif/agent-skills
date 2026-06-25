# Elicitation Guide

The interview's defining quality is **completeness through proactive enumeration**.
A weak requirements process records what the user thinks to say; this one drives
the user through the full landscape of what a system like theirs normally needs,
so nothing is missed by omission. The `requirement-catalog.md` reference is the
content engine; this guide is the technique for using it.

## The enumeration discipline (the core technique)

For every capability area, do **not** ask an open "what do you need for X?" and
record the answer. Instead:

1. **Propose the standard set.** Pull the area's standard sub-requirements from
   the catalog and present them as a plain-language checklist. For authentication:
   "A login system normally includes: sign-up, email/phone verification, sign-in,
   logout, forgot-password, reset-password, change-password, session expiry,
   remember-me, account lockout after failed attempts, and rate limiting. Often
   also: multi-factor auth, social/SSO login, account deletion. Which of these
   does your product need, and is anything missing?"
2. **Let the user confirm, extend, or remove.** They tick what applies, add the
   domain-specific ones the catalog can't know, and drop what doesn't fit.
3. **Specify each confirmed item formally** (see below) and write it to the SRS.
4. **Surface the commonly-forgotten ones explicitly.** The catalog flags items
   people routinely overlook (audit logging, rate limiting, account
   deletion/GDPR, admin/back-office, error/empty states at the requirement
   level). Raise them even if the user didn't — that's the value of the skill.

This turns elicitation from stenography into a guided expert review.

## Running the interview

- **Work one area at a time, and checkpoint between areas.** Finish
  authentication completely — propose, confirm, specify, write to SRS, update
  tracker — before opening the next area. This is what makes the work resumable.
- **Batch within an area.** Present an area's checklist as one batch rather than
  one question at a time; it respects the user's time and reads as a coherent
  review.
- **Infer from earlier answers and state inferences.** If the product is "a B2B
  admin tool for internal staff," infer that public sign-up may not apply and
  SSO probably does — propose accordingly rather than asking generically.
- **Keep a running TBD list.** When the user doesn't know an answer, record it as
  an explicit open question in the SRS rather than guessing or stalling, and move
  on. Flag all TBDs at delivery.

## Specifying a requirement formally

Each confirmed requirement gets:

- **A unique, stable ID** — `FR-<AREA>-<NNN>` (e.g., `FR-AUTH-007`). IDs never
  change once assigned; they are what the RTM and downstream skills reference.
- **A single, testable statement** — "The system shall …", one requirement per
  statement (split compound requirements). If you can't write a test for it, it's
  not specific enough.
- **A priority** — e.g., Must / Should / Could (MoSCoW), or High/Med/Low.
- **Supporting detail where it matters** — validation rules, business rules,
  data constraints, error conditions. (Detailed UI and API design is *not* here;
  that's downstream. Requirements state *what*, not *how*.)

## Determining which areas exist

Derive the area list from the product's purpose and scope (Phase 1) plus the
catalog's standard areas. Walk the catalog's list and, for each area, decide with
the user whether it applies. Don't assume the application only needs the areas the
user first described — the point of the catalog is to prompt for the rest.

## Closing functional elicitation

When all areas are covered, play back the area list with requirement counts and
ask whether any capability is still missing. Only then move to non-functional
requirements (Phase 3).
