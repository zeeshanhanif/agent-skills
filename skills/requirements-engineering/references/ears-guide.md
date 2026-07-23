# EARS Guide — Functional Requirement Syntax

This guide governs functional requirement statements **when the SRS's recorded
syntax is EARS** (the choice is made once, at the end of Phase 1, and recorded
in the SRS header — see SKILL.md). **EARS** (Easy Approach to Requirements
Syntax) is constrained natural language built from five sentence
patterns. EARS keeps requirements readable English while forcing every
statement to make its trigger, state, or condition explicit — which is exactly
the ambiguity that otherwise leaks into design and planning. The statement
lives in the same SRS table column it always did; EARS changes the sentence's
shape, not the table's schema or the ID/priority/status contract.

**Scope:** EARS applies to **functional requirements only**. Non-functional
requirements keep their metric/target/condition form (see
`requirement-catalog.md`) — forcing an NFR into an EARS pattern adds words, not
precision.

## The generic structure

Every EARS requirement is an ordered assembly of optional clauses around one
mandatory core:

```
While <precondition/state>, when <trigger>, the <system> shall <response>
```

Rules: zero or many preconditions, zero or one trigger, exactly one system
name, one or many responses — always in that clause order. The five patterns
below are just the meaningful combinations.

## The five patterns

### 1. Ubiquitous — always active, no trigger or state

```
The <system> shall <response>
```

> The system shall store all timestamps in UTC.

Use only when the requirement genuinely holds at all times. **This is the
pattern to be suspicious of**: most "shall" statements written without thought
land here by default, hiding a real trigger or state. If you can ask "when
does this happen?" and get a specific answer, it is not ubiquitous.

### 2. Event-driven — a response to a trigger

```
When <trigger>, the <system> shall <response>
```

> When a visitor completes registration, the system shall send a verification
> email to the submitted address.

The workhorse pattern. Most interactive functionality is event-driven.

### 3. State-driven — active throughout a state

```
While <state>, the <system> shall <response>
```

> While a user's account is locked, the system shall reject sign-in attempts
> without evaluating the submitted password.

Distinguish carefully from event-driven: *entering* a state is an event
("When the account becomes locked…"); behavior *during* the state is
state-driven.

### 4. Optional feature — applies only when a feature is present

```
Where <feature is included>, the <system> shall <response>
```

> Where multi-factor authentication is enabled for an account, the system
> shall require a second factor after successful password verification.

Use for product variants, plan tiers, and feature flags — configuration that
makes whole requirements applicable or not.

### 5. Unwanted behavior — handling failures and errors

```
If <undesired trigger/condition>, then the <system> shall <response>
```

> If a password-reset link is used after its expiry, then the system shall
> reject the request and offer to send a fresh link.

This pattern is the completeness engine for error handling. During
elicitation, for every event-driven requirement ask what the unwanted
counterpart is (invalid input, timeout, failure, abuse) — many belong in the
SRS as their own `If/then` requirements with their own IDs.

### Complex (combined clauses)

Patterns combine, keeping clause order (While → When/If → shall):

> While a user is signed in, when the session exceeds the idle timeout, the
> system shall sign the user out and redirect to the sign-in screen.

Prefer a single pattern; combine only when the state genuinely bounds the
trigger.

## Choosing the pattern

Ask in order:

1. Is this about a failure, error, or unwanted situation? → **If / then**
2. Does it apply only when an optional feature/tier is present? → **Where**
3. Is it active throughout a state or mode? → **While**
4. Does a specific event kick it off? → **When**
5. None of the above, truly always active? → **Ubiquitous**

## EARS and the existing rules

- **One requirement per statement** (already a template rule): EARS enforces
  this naturally — one trigger, one statement. A compound "shall X and send Y"
  splits into separate IDs. Multiple responses to the *same* trigger may share
  a statement only when they are inseparable parts of one behavior; when in
  doubt, split.
- **Testability**: the trigger/state clause *is* the test setup; the response
  *is* the assertion. If an acceptance test isn't obvious from the statement,
  the clause is too vague ("when appropriate" is not a trigger).
- **What, not how** still applies: "When a user submits the form, the system
  shall validate the email address" — not "…shall call the validation
  service."
- **Amendments** (`change-management.md`): follow the SRS's recorded syntax.
  In EARS mode a modified statement is rewritten in EARS form; statements
  authored before EARS adoption may remain in legacy form until next touched —
  rewriting is not itself a change requiring a version bump unless meaning
  changes. The syntax choice itself is not re-asked at amendment time; switching
  an existing SRS's syntax is a deliberate, versioned decision by the user.

## Common mistakes

- **Everything ubiquitous.** The classic failure. Interrogate every trigger-less
  statement.
- **Vague triggers.** "When needed", "when appropriate", "when possible" —
  name the actual event.
- **Hidden requirements in Notes.** If the Notes column says "also locks the
  account after 5 failures", that is an unwanted-behavior requirement missing
  its own ID.
- **Passive voice hiding the system.** "When X occurs, an email shall be
  sent" — sent by what? Name the system as the subject.
- **`Where` for conditions.** `Where` is for optional *features*, not runtime
  conditions. "Where the input is invalid" is wrong — that's `If/then`.
