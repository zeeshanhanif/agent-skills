# SRS Template

The SRS follows the **ISO/IEC/IEEE 29148** (IEEE 830 lineage) structure. It
absorbs the related documents the user chose to merge: vision & scope, glossary,
assumptions, and constraints all live here as sections, alongside the full
functional and non-functional requirements.

Create this section skeleton in `docs/srs.md` at the start (so the document is
always well-formed), then fill sections in place as elicitation areas complete.
This skill targets **comprehensive** coverage; it is a detail phase, so prefer
completeness over brevity — but keep each requirement statement tight and
testable.

Use this structure:

```markdown
# Software Requirements Specification: <System Name>

> Version: <n> · Status: Draft | Reviewed | Approved | Finalized · Last updated: <date>

## Revision History
*(One row per version. Seeded at finalization; appended on every amendment — see
`change-management.md`.)*

| Version | Date | Author | Changes |
| :------ | :--- | :----- | :------ |
| 1.0 | <date> | <name> | Initial finalized specification. |

## 1. Introduction
### 1.1 Purpose
What this document specifies and who it's for.
### 1.2 Product vision & scope
The problem, the vision, business objectives, in-scope vs. out-of-scope, the MVP
boundary. (The absorbed Vision & Scope content.)
### 1.3 Goals & success metrics
Measurable definitions of success.
### 1.4 Definitions, acronyms & abbreviations
(Or reference the Glossary appendix.)
### 1.5 References
Source documents, standards, related systems.

## 2. Overall Description
### 2.1 Product perspective
Context and system boundary; relationship to existing/external systems. A
requirements-level context description (external actors and systems).
### 2.2 Product functions (summary)
A high-level list of the major capability areas (detailed in section 3).
### 2.3 User classes & characteristics
The user types/personas, their context and needs. *(ux-foundations consumes
this.)*
### 2.4 Operating environment
Where and how the system runs, at the requirements level.
### 2.5 Constraints
Regulatory, technical, business, and resource constraints that bound the
solution.
### 2.6 Assumptions & dependencies
What is assumed true, and external dependencies the project relies on.

## 3. Specific Requirements
### 3.1 Functional requirements
Organized by capability area. Each area is a subsection; each requirement has a
unique ID, a testable "shall" statement, a priority, and supporting rules. Use a
table per area:

| ID | Requirement | Priority | Status | Notes / rules |
| :-- | :---------- | :------- | :----- | :------------ |
| FR-AUTH-001 | The system shall allow a visitor to register with email and password. | Must | Active | Email must be unique; password policy per NFR-SEC-003. |
| FR-AUTH-002 | The system shall send a verification email on registration. | Must | Active | Link expires in 24h. |
| … | … | … | … | … |

Repeat a subsection (3.1.1, 3.1.2, …) per area: Authentication, Authorization,
Profile, Notifications, etc. The **Status** column is `Active` by default;
amendments set it to `Removed`/`Deprecated` (with the version) rather than
deleting the row — see `change-management.md`.

### 3.2 External interface requirements
User interfaces (requirements level), hardware interfaces, software/third-party
interfaces, communication interfaces.

### 3.3 Non-functional requirements
Organized by ISO 25010 category, each measurable with a unique ID:

| ID | Category | Requirement (metric, target, condition) | Priority |
| :-- | :------- | :-------------------------------------- | :------- |
| NFR-PERF-001 | Performance | 95% of key API calls complete within 300 ms at 500 concurrent users. | Must |
| NFR-SEC-001 | Security | All data encrypted in transit (TLS 1.2+) and at rest (AES-256). | Must |
| … | … | … | … |

### 3.4 Other requirements
Compliance, legal, localization, data retention — anything not covered above.

## Appendix A — Glossary
Domain and technical terms.

## Appendix B — Open questions / TBD
Every unresolved item flagged during elicitation, so nothing is silently
assumed.
```

---

## Rules

- **One requirement per statement.** Split compound requirements so each has its
  own ID and is independently testable.
- **Stable IDs.** Once assigned, an ID never changes — the RTM and all downstream
  skills reference it.
- **Testable language.** If you can't write an acceptance test for it, sharpen it.
- **Requirements state *what*, not *how*.** No architecture or detailed UI/API
  design here — that's downstream. (A requirement may reference an NFR for a
  constraint, e.g., "per NFR-SEC-003".)
- **Right-sizing.** Coverage is exhaustive, but a small product still yields a
  smaller SRS than a large one — the number of areas and requirements scales with
  the product, not padding.
