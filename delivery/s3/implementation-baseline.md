# Sprint 3 Implementation Baseline and Scope Lock

## Purpose

This unit fixes the Sprint 3 baseline before dependency mapping, migration sequencing, vertical-slice planning, or GitHub Issue creation. It is a change-control gate, not a restatement of S3 design.

## Authoritative inputs

| Input | Governs implementation |
| --- | --- |
| [Frozen S3 requirements and final scope review](../../design/s3/requirements/s3-scope.md) | Product scope, acceptance boundary, and exclusions |
| [S3 Phase 2 architecture and final architecture review](../../architecture/s3/architecture-overview.md) | Module ownership, dependency direction, runtime boundaries, and non-goals |
| [S3 Phase 3 technical design](../../design/s3/technical/README.md) | Persistence, consistency, async, security, observability, API contract, and implementation structure |
| [S3 Phase 4 UI/UX](../../design/s3/ui-ux/README.md) and approved `S3-UI-01`–`S3-UI-20` page specifications | Surface behaviour, states, navigation, reuse, and frontend/backend mapping |
| [`engineering-s2`](../../README.md#release-history) and the current `offerbuddy` source repository | Implemented reuse baseline and current behaviour |
| [`operations/documentation-governance.md`](../../operations/documentation-governance.md) | Documentation source-of-truth and reconciliation rules |
| [`quality/definition-of-done.md`](../../quality/definition-of-done.md) and [`operations/development-workflow.md`](../../operations/development-workflow.md) | Verification and delivery conventions |

The frozen S3 requirements, architecture, technical design/API contract, and approved UI/UX specifications are published in this repository. Implementation Issues must link these governed copies rather than the external working records.

## Existing S2 reuse baseline

Inspection of the S2 source at the `engineering-s2` baseline establishes the following implementation posture.

| Area | Classification | S3 implementation rule |
| --- | --- | --- |
| Application create, update, tracking, history, analytics, and Application Detail | Reuse and extend additively | Preserve the Fast Path; add OCC and Preparation integration without replacing existing workflows |
| Spring Boot modular backend, authentication, ownership lookup, validation, error handling, JPA, PostgreSQL, and Flyway | Reuse | Add module-private S3 capabilities under `com.offerbuddy`; do not introduce microservices or alternate persistence infrastructure |
| Flyway `V1`–`V9` | Reuse unchanged | Treat as immutable; all S3 schema changes use new additive migrations |
| PostgreSQL `business_events` claim, lease, retry, diagnostics, and handler registry | Extend additively | Add versioned S3 event contracts and handlers; do not introduce Kafka, RabbitMQ, or a generic task platform |
| Job Intelligence | Integrate and extend | Reuse the persisted semantic analysis pipeline; add source-version semantics and Preparation projections rather than duplicate analysis |
| React authentication, routing, `AppShell`, API client, shared components, and Application pages | Reuse and extend | Add S3 routes and focused surfaces; integrate a secondary Preparation summary into the existing Application Detail page |
| Extension content script, Site Adapter contract, `JobPageContext`, service worker messaging, credential flow, and floating companion | Extend additively | Add LinkedIn and S3 states through existing boundaries; retain SEEK/Indeed and application recording behaviour |
| RuoYi backend/frontend, RBAC, PostgreSQL/Redis connectivity, and separate deployment | Integrate | Add operational S3 modules inside the existing Admin shell; do not create another admin application or bypass product-domain ownership |
| Product Redis | New product integration | Redis exists in deployment but the product backend has no Redis dependency; any S3 use requires explicit product configuration and failure isolation |
| Candidate, Preparation, AI capability routing, generated artefacts, sponsor dataset, and S3 monitoring | New | Implement as S3 modules while consuming the reused platform boundaries above |

## S3 implementation scope

The frozen implementation scope comprises:

- Candidate Profile and user-reviewed Resume Import;
- Preparation anchored to Candidate + Job;
- Job Intelligence S3 versioning and Preparation projection;
- explainable Match Analysis;
- Base Resume support, Tailored Resume, and Focused Cover Letter artefacts;
- LinkedIn adapter, floating-assistant states, sponsor signal, and SEEK cover-letter assistance;
- additive Application Detail integration;
- AI capability routing, runtime configuration, safe telemetry, and failure diagnostics;
- RuoYi sponsor administration, AI configuration, and monitoring surfaces;
- additive persistence, event contracts, security, concurrency, provenance, and observability required by those capabilities.

Detailed behaviour remains governed by the frozen design and page specifications.

## Feature-freeze boundary

S3 implementation must not introduce Auto Apply, crawler-based job ingestion, ATS simulation, hiring prediction, interview coaching, multiple Candidate personas, a full Resume Designer, a generic AI workflow platform, a prompt playground, a secrets manager, a new Admin shell, microservices, a message broker, or a Saved Job workflow. These exclusions are scope controls, not implied backlog commitments.

## Compatibility and preservation rules

1. Application recording remains usable without Candidate Profile, Preparation, Match, generated artefacts, AI, or Admin availability.
2. Preparation remains Candidate + Job owned; it must not become an Application child or create an Application implicitly.
3. Authenticated server identity and resource ownership chains remain authoritative. Client-supplied ownership identifiers are untrusted.
4. Cross-module access uses application-facing contracts and immutable projections; persistence entities and repositories remain private.
5. S3 async work extends `business_events`, uses at-least-once-safe handlers, and keeps external/AI calls outside database transactions.
6. `profileVersion`, Job `contentVersion`, Application `version`, artefact concurrency, generation identity, and idempotency retain distinct meanings.
7. Derived artefact freshness is calculated from recorded source versions; completion order and event order do not redefine recency.
8. Extension remains a lightweight, untrusted assistant. Candidate editing and focused artefact review stay in the Web application.
9. RuoYi remains an operational client with independent Admin identity and no universal Candidate/Application data privilege.
10. Business modules depend on semantic AI capability ports. Provider secrets, raw prompts, raw responses, and Candidate content must not leak through events, logs, monitoring, or Admin views.

## Change control

| Classification | Treatment |
| --- | --- |
| **A. Implementation detail** | Decide within the delivery unit when product, domain, API, security, and approved UI semantics are unchanged. Record only where it affects verification or future maintenance. |
| **B. Compatible implementation delta** | Document the additive adjustment and its codebase evidence. Keep it inside frozen boundaries and include it in the affected Issue/PR acceptance criteria. |
| **C. Design contradiction / blocker** | Stop planning or implementation for the affected area. Record the conflicting sources and obtain an explicit design resolution; do not silently choose one. |

## Baseline readiness

### Compatible implementation deltas

- `applications` has no optimistic concurrency version and Job uses timestamps rather than the frozen `contentVersion`; both require additive schema, API, and client changes.
- Product Redis integration is not implemented even though Redis is deployed for Admin/future use.
- Extension platform support is currently SEEK/Indeed only; LinkedIn is a new adapter within the existing contract.
- S3 business modules and API surfaces do not yet exist, as expected at this baseline.

### Reconciled design findings

- Frozen §3.17 defines Tailored Resume generation, limited editing, revision behaviour, and artefact OCC. Generated revisions remain immutable records; approved editing produces the frozen revision/update outcome rather than mutating historical provenance.
- Sponsor Admin create/edit/delete-or-disable actions are limited operational management of the working/import dataset before validation and publication. They do not bypass the frozen publish lifecycle or provide unrestricted mutation of the published canonical dataset.
- Sponsor lookup uses layered responsibilities: the published backend dataset remains authoritative, Redis holds the active representation, and the Extension refreshes a versioned local snapshot for page-time lookup. The snapshot is not a second source of truth.
- Phase 3 §3.17 is complete and frozen and governs endpoint, DTO, error, OCC, idempotency, and async polling semantics.

The reuse and scope baseline is stable. The earlier Tailored Resume, Sponsor Admin, Sponsor Extension, and API-contract findings are compatible clarifications resolved by the frozen Phase 3 baseline and the [Dependency, Vertical Slice, and Migration Plan](dependency-and-migration-plan.md); no genuine baseline blocker remains.
