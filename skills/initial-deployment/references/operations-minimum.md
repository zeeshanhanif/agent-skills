# Operations Minimum (the day-1 floor)

Phase 7's contract. Full day-2 operations (dashboards, SLOs, runbooks, cost
optimization) is a later concern; this floor is not — **a deployment without
eyes is negligent.** Everything here is small, provider-standard, and
verified by demonstration in this run.

## Uptime

An external availability check on each public surface (the provider's own
uptime/health-check service or the simplest standard equivalent), probing a
real health endpoint (the skeleton exposes one; if it doesn't, adding it is
part of this phase), alerting to a channel the user actually reads (email at
minimum — ask which). **Verify by demonstration**: cause one controlled
failing probe (or use the service's test-fire) and confirm the alert reached
the human.

## Errors

Error signal wired: the platform's error reporting or a log-based alert on
error-level entries, routed to the same real channel. Threshold sane (alert
on presence/spike, not every entry). Demonstrate once: emit a controlled
error-level event from the deployed service, observe the alert.

## Logs

The services' structured logs (scaffolding wired them) reachable in the
provider's log surface, retention at the provider default unless an SRS NFR
says otherwise (cite it), and the *how to read them* commands recorded in
deployment-notes.

## Backups — and the restore proof

Automated backups on for every provisioned store, schedule per provider
default unless an NFR states RPO (cite it). Then the part that's usually
skipped and here is not: **perform one restore** — to a scratch
instance/database, verify the skeleton's data round-trips against it, tear
the scratch down. A backup never restored is a hope, not a capability.
Record the restore procedure verbatim in deployment-notes — during a real
incident nobody wants to derive it.

## TLS and domain

Per the architecture: custom domain wired where specified (DNS steps are
user-driven at their registrar — hand over the exact records; verify
propagation), TLS via the platform's managed certificates, HTTP→HTTPS
enforced. Architecture silent → platform-provided URL with its default TLS
is acceptable; record it as the deliberate default.

## Cost guardrail

A billing alert at a user-chosen monthly threshold (ask; suggest one from
the Phase 1 cost class), via the provider's budget mechanism. This is the
difference between a surprise and an email.

## The line to day-2

Explicitly *not* here: dashboards, tracing, SLOs/error budgets, paging
policies, load testing, multi-region drills, cost optimization. When the
project needs them, that's the fuller operations concern — record in
deployment-notes anything from this run that obviously wants promoting there
(e.g., "error volume suggests a dashboard early").
