# Requirement Catalog

This is the enumeration engine. For each common capability area it lists the
standard sub-requirements to **propose** to the user, including the ones people
routinely forget. It is a prompting checklist, not a mandate — the user confirms,
extends, and removes per their product. Domain-specific requirements the catalog
can't know are added during elicitation.

Treat every area as: "does your product have this area? if so, here are the
sub-requirements it normally includes — which apply?"

---

## Functional areas

### Authentication & account management
Sign-up / registration; email or phone **verification**; sign-in; **logout**;
forgot password; **reset password**; change password; session management (session
**expiry**/timeout, refresh, concurrent-session handling); "remember me";
**account lockout** after failed attempts; **rate limiting / brute-force
protection**; password strength/policy rules. Often also: multi-factor auth
(MFA/2FA); social / SSO login (Google, etc.); passwordless/magic-link; account
**deletion / deactivation**; re-authentication for sensitive actions.

### Authorization & access control
Roles and permissions model; role assignment; **RBAC** (or ABAC) enforcement;
resource ownership rules; admin vs. standard user separation; permission checks
on every protected action; least-privilege defaults; elevation/approval flows.

### User profile & settings
View/edit profile; avatar/image upload; contact details; notification
preferences; privacy settings; language/locale; linked accounts; security
settings (active sessions, password, MFA management).

### Onboarding
First-run experience; guided setup/wizard; sample/seed data; invite flow;
team/organization creation; progress tracking.

### Core domain functionality
*(The product-specific heart — elicit directly, but still enumerate the CRUD
lifecycle for each main entity:* create, read/view, list, update, delete,
restore/soft-delete, bulk actions, ownership, validation rules, state/status
transitions.*)*

### Search, filtering & sorting
Keyword search; filters; sort; pagination / infinite scroll; saved searches;
faceted search; result relevance/ranking; empty-result handling.

### Notifications & messaging
In-app notifications; **email** notifications; push (mobile/web); SMS;
notification preferences and opt-out; digests; templates; delivery/read status.

### File & media handling
Upload (size/type limits, validation); storage; download; preview/thumbnails;
versioning; virus scanning; access control on files.

### Payments & billing *(if applicable)*
Plans/pricing; checkout; payment-method management; subscriptions
(upgrade/downgrade/cancel); invoices/receipts; refunds; dunning/failed-payment
handling; tax; webhooks from the processor.

### Reporting & analytics
Dashboards; metrics; export (CSV/PDF/Excel); scheduled reports; date-range
filtering; charts; per-role report visibility.

### Data import / export
Import (formats, validation, error reporting, mapping); export; bulk operations;
templates; background processing for large jobs.

### Integrations & APIs
Third-party integrations; outbound webhooks; public/partner API; API keys/tokens;
rate limits; API versioning.

### Administration / back-office
Admin console; user management; content/config management; feature flags;
impersonation (with audit); system settings; moderation.

### Audit & compliance *(commonly forgotten)*
**Audit logging** of security-relevant and data-changing actions; activity
history; data-retention rules; consent management; **GDPR/CCPA** data export and
**right-to-be-forgotten**; legal/terms acceptance tracking.

### Help & support
Help/docs; contact/support form; feedback; in-app guidance; error reporting.

### Commonly-forgotten cross-area items — always raise these
Account deletion & data export (privacy law); audit logging; rate limiting &
abuse prevention; admin/back-office; empty/error/loading states *as requirements*;
concurrency/conflict handling; soft-delete vs. hard-delete; pagination on every
list; internationalization touchpoints; accessibility as a requirement;
migration/seed for launch.

---

## Non-functional requirements (ISO/IEC 25010 quality model)

Walk each category that applies; specify **measurable** NFRs with IDs
(`NFR-<CAT>-<NNN>`). Replace adjectives with numbers and conditions.

- **Performance efficiency** — response-time targets (e.g., p95 < 300 ms for key
  endpoints), throughput, resource utilization, batch/job completion times.
- **Scalability / capacity** — expected and peak concurrent users, data volume
  and growth, requests/sec, horizontal-scale expectations.
- **Availability / reliability** — uptime target (e.g., 99.9%), fault tolerance,
  recoverability (**RPO/RTO**), backup frequency, graceful degradation,
  maintenance windows.
- **Security** — authentication strength, authorization model, encryption in
  transit and at rest, secrets management, audit logging, **OWASP** coverage,
  vulnerability/patch policy, penetration-test expectations, data
  classification.
- **Usability & accessibility** — accessibility standard (e.g., **WCAG 2.2 AA**),
  supported browsers/devices, task-success/learnability targets, error
  prevention.
- **Compatibility / interoperability** — platforms, browsers, OS/versions,
  standards and formats to interoperate with, third-party compatibility.
- **Maintainability** — code quality/test-coverage expectations, modularity,
  documentation, configurability.
- **Portability** — deployment environments, cloud portability, containerization
  expectations, data-migration needs.
- **Compliance / legal / regulatory** — GDPR, HIPAA, PCI-DSS, SOC 2, sector and
  local regulations, **data residency**, accessibility law, audit requirements.
- **Localization / internationalization** — languages, RTL, locale formats,
  currency, time zones, translation workflow.
- **Observability / operability** — logging, metrics, tracing, alerting,
  monitoring, health checks, SLAs for incident response.

For each NFR, state the **metric, the target value, and the condition** under
which it's measured. "The system shall be fast" is rejected; "95% of catalog
searches shall return within 500 ms at 1,000 concurrent users" is accepted.
