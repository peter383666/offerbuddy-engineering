# Sprint 2 – Lower-Friction Job Capture

# Sprint Information

| Item | Value |
| --- | --- |
| Sprint | Sprint 2 |
| Status | Planning complete; implementation not started |
| Primary capability | Browser Extension for SEEK and Indeed |

---

# Overview

Sprint 2 makes OfferBuddy more practical during the user's normal job-search workflow. The Browser Extension becomes the preferred way to record an Application while the existing manual and URL-prefill New Application flow remains available as a secondary fallback.

The sprint extends the existing Spring Boot modular monolith and PostgreSQL foundation. It does not redesign the product or introduce distributed infrastructure.

---

# Sprint Goal

Make OfferBuddy practical in the user's daily job-application workflow by delivering a Chrome Manifest V3 Browser Extension that reliably captures SEEK and Indeed jobs and records the authenticated user's Application with minimal friction. Core Application creation must complete independently of asynchronous Job Intelligence and Analytics, while the existing manual and URL-prefill workflow remains available as a secondary fallback.

# Scope

## In Scope

- Browser Extension capture for SEEK and Indeed using independent Site Adapters, dynamic-page awareness, explicit user-initiated saving, and the approved ready, review, saving, saved, duplicate, authentication-required, and failure states.
- Web-to-Extension pairing, finite-lived revocable Extension credentials, and authenticated Extension API operations.
- Backend validation and orchestration for canonical Job resolution/selective refresh and create-if-absent, user-owned Application tracking.
- Application lifecycle history and `WEB`/`EXTENSION` creation-source support, while preserving one Application per user and Job.
- PostgreSQL-backed durable Business Events with bounded retry, restart recovery, observable terminal failure, and idempotent downstream handling.
- Asynchronous, Job-owned Intelligence producing a concise factual summary, responsibilities, requirements, and skills from persisted Job content.
- Small, user-scoped Application Analytics covering the approved time ranges, totals, outcomes, and conversion measures.
- Web UI integration for the final S2 Home, Applications, Application Detail, New Application, and Analytics specifications, including loading, empty, error, and populated states.
- Forward-only Flyway migrations, automated verification, supported-platform manual testing, documentation, and release preparation.

## Out of Scope

The exclusions in the [Sprint 2 Requirements](../../product/sprint-2-requirements.md#explicitly-out-of-scope) remain authoritative. In particular, Sprint 2 does not include LinkedIn or broad platform support, Auto Apply or automatic form submission, resume tailoring, Cover Letter generation, match scoring, large-scale Analytics/BI, microservices, Kafka/RabbitMQ, Redis queues/caches, Kubernetes, exactly-once guarantees, or unnecessary distributed infrastructure.

# Delivery Workstreams

## Browser Extension

- **Objective:** Provide the primary low-friction capture workflow on supported SEEK and Indeed job contexts.
- **Outcomes:** Chrome MV3 extension; isolated SEEK and Indeed adapters; deterministic fact extraction; SPA, side-panel, and stale-context handling; explicit save action; compact approved UI states; credential isolation from page execution; safe failure when extraction is unreliable.
- **Dependencies:** Agreed Extension API contracts and final UI assets/specification. Adapter work can begin against fixtures while backend ingestion is built.
- **Sources:** [Extension Design](../../design/sprint-2/extension-design.md), [Backend API Design](../../design/sprint-2/backend-api-design.md), [Final UI/UX Specification](<../../design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md>), [ADR-009](../../decisions/ADR-009-browser-extension-site-adapters.md).

## Extension Authentication and Backend Ingestion

- **Objective:** Safely connect the Extension to an OfferBuddy user and turn validated captures into an idempotent tracking outcome.
- **Outcomes:** Pairing create/approve/exchange, current-user and revoke operations; finite-lived revocable Extension credentials; `POST /api/v1/extension/applications`; stable validation/error responses; server-derived ownership; duplicate response through `alreadyTracked`; no synchronous AI or Analytics dependency.
- **Dependencies:** Existing Web session authentication, Job/Application service boundaries, and database uniqueness/concurrency protection.
- **Sources:** [Backend API Design](../../design/sprint-2/backend-api-design.md), [Backend / Service Design](../../design/sprint-2/backend-service-design.md), [Sprint 2 Architecture](../../architecture/sprint-2-architecture-design.md).

## Job and Application Core

- **Objective:** Preserve canonical shared Jobs and user-owned Applications across Extension, manual, and URL-prefill ingestion paths.
- **Outcomes:** Job lookup by source platform plus external identifier; selective source-fact refresh without null erasure; Application create-or-reuse without resetting an existing status; initial `null -> APPLIED` and later lifecycle history; clear service/repository boundaries; atomic core writes and event persistence.
- **Dependencies:** Forward migrations and authenticated ingestion identity. This work is the foundation for downstream processing and UI reads.
- **Sources:** [Backend / Service Design](../../design/sprint-2/backend-service-design.md), [Database Design](../../design/sprint-2/database-design.md), [Sprint 1 Data Model](../../architecture/data-model.md).

## Database and Migrations

- **Objective:** Extend the deployed Sprint 1 PostgreSQL schema without rewriting existing migrations or fabricating historical facts.
- **Outcomes:** Forward-only Flyway migrations for creation source, status history, Job Intelligence, durable events, and Analytics projection; reliable backfills; retained legacy compatibility fields; database uniqueness as final concurrency protection; production-shaped migration verification.
- **Dependencies:** Implemented persistence contracts for Core, events, Intelligence, and Analytics. Migration slices should land with the vertical capability that uses them where practical.
- **Sources:** [Database Design](../../design/sprint-2/database-design.md), [Redis Design](../../design/sprint-2/redis-design.md), [ADR-006](../../decisions/ADR-006-flyway.md), [ADR-007](../../decisions/ADR-007-postgresql.md).

## Events and Asynchronous Processing

- **Objective:** Decouple non-critical work from Core using lightweight, durable processing inside the modular monolith.
- **Outcomes:** Transactional persistence of Business Events; bounded claim/process/outcome phases; at-least-once-tolerant handlers; bounded retry; abandoned-claim recovery; visible failure and lag; no external calls or long-held locks in Core transactions.
- **Dependencies:** Core event production and `business_events` persistence. Job Intelligence and Analytics processors can then proceed independently.
- **Sources:** [Event Design](../../design/sprint-2/event-design.md), [ADR-010](../../decisions/ADR-010-lightweight-business-events.md), [Sprint 2 Architecture](../../architecture/sprint-2-architecture-design.md).

## Job Intelligence

- **Objective:** Enrich persisted Jobs asynchronously without affecting Application creation.
- **Outcomes:** Job-snapshot processing through the existing provider-abstraction principle; validated concise summary, responsibilities, requirements, and skills; versioned analysis attempts and coherent results; bounded transient retry; controlled missing-description, malformed-output, provider-failure, and terminal-failure states.
- **Dependencies:** Durable event processing, Job snapshots, analysis persistence, and backend-only provider configuration. It is independent of Analytics and the Core response after event production exists.
- **Sources:** [Job Intelligence Design](../../design/sprint-2/job-intelligence-design.md), [Event Design](../../design/sprint-2/event-design.md), [ADR-003](../../decisions/ADR-003-ai-assisted-job-extraction.md), [ADR-004](../../decisions/ADR-004-ai-provider-abstraction.md).

## Application Analytics

- **Objective:** Provide the approved small, eventually consistent view of each user's Application activity and outcomes.
- **Outcomes:** Idempotent one-row-per-Application projection; rebuild from authoritative Core/history data; server-scoped Analytics reads; approved time ranges, totals, outcomes, and conversion measures; derived `NO_RESPONSE` without creating a lifecycle transition or fabricating dates.
- **Dependencies:** Application history, source facts, Business Events, Analytics persistence, and authenticated query APIs. Projection work can proceed in parallel with Job Intelligence.
- **Sources:** [Analytics Design](../../design/sprint-2/analytics-design.md), [Database Design](../../design/sprint-2/database-design.md), [Final UI/UX Specification](<../../design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md>).

## Web Frontend and Product Integration

- **Objective:** Integrate S2 capabilities into one coherent Web experience while keeping the Extension primary and New Application secondary.
- **Outcomes:** Shared navigation/branding; revised Home and Applications surfaces; Application Detail with lifecycle history and pending/unavailable/failed Intelligence states; secondary manual/URL-prefill New Application; compact Analytics page; responsive and accessible loading, empty, error, and populated states matching final Figma/specification.
- **Dependencies:** Implemented API contracts and representative UI states. Static shell/state work can proceed in parallel; final integration follows stable APIs.
- **Sources:** [Final UI/UX Specification v2.0](<../../design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md>), [Backend API Design](../../design/sprint-2/backend-api-design.md). The conflicting `ui-ux/v1` material is a watch item below, not an implementation authority for this plan.

## Quality, Integration, and Release

- **Objective:** Prove the S2 vertical flows, preserve Sprint 1 behaviour, and leave an auditable release candidate.
- **Outcomes:** Risk-focused automated tests; OpenAPI/contract verification; manual SEEK/Indeed and Web end-to-end checks; current documentation; passing CI; reviewed feature PRs; verified candidate SHA, migration impact, rollback considerations, and release notes.
- **Dependencies:** Incremental delivery from every workstream; release closure follows integrated verification.
- **Sources:** [Testing Strategy](../../quality/testing-strategy.md), [Definition of Done](../../quality/definition-of-done.md), [Development Workflow](../../operations/development-workflow.md).

# Implementation Order

1. **Establish contracts and migration foundations.** Confirm implementation-owned details permitted by the designs, then add the first forward migrations and domain persistence for lifecycle history, creation source, and durable events. Preserve Sprint 1 migrations and compatibility.
2. **Deliver the Core tracking vertical.** Implement Job resolve/refresh, Application create-or-reuse, initial history, event persistence, and authenticated ingestion orchestration with concurrency and ownership tests. Expose the approved Extension Track contract without waiting for AI or Analytics.
3. **Integrate Extension authentication early.** Implement pairing, credential verification/revocation, and the Extension client connection flow so real authenticated ingestion can be exercised before the UI is polished.
4. **Build supported-site capture vertically.** Implement common capture state plus SEEK and Indeed adapters using fixtures, then connect explicit saving to the real backend. The two adapters can proceed in parallel once the common representation and adapter boundary are stable.
5. **Add durable dispatch and downstream processors.** Complete claim, retry, recovery, and observability behaviour. Job Intelligence and Analytics can then be implemented in parallel because they share delivery infrastructure but retain separate ownership, data, and failure policies.
6. **Integrate the Web experience.** Build approved Web shell and static states alongside backend work; connect Application history/Intelligence and Analytics when contracts are available. Preserve manual creation and URL prefill as the secondary flow.
7. **Harden and release.** Run full migration, API, async, UI, platform, security, and Sprint 1 regression verification; resolve the eligibility-warning handoff item below; update operational/release documentation; promote only an identified, reviewed candidate with passing CI.

Each step should produce a testable vertical increment. UI shell work, adapter fixtures, and API test scaffolding can begin independently; downstream processing must not be placed on the Core write path.

# Testing and Verification

- **Unit/service tests:** Job identity and selective refresh, Application create-or-reuse/status preservation, lifecycle history, validation, ownership, pairing state, retry classification, Intelligence validation, Analytics derivation, and UI state logic.
- **Backend integration tests:** Authenticated/unauthenticated APIs, server-derived ownership, duplicate outcomes, concurrent ingestion, atomic Core/history/event writes, rollback, and PostgreSQL repository behaviour.
- **Extension tests:** Independent SEEK/Indeed fixtures; missing optional/required facts; SPA and side-panel navigation; noisy mutation coalescing; stale-context clearing; explicit action boundary; authentication expiry; credential isolation; all approved popup outcomes.
- **Contract tests:** Approved `/api/v1/extension` paths, payload constraints, status/error codes, duplicate `200` response, OpenAPI shape, and the absence of synchronous Intelligence/Analytics fields.
- **Migration tests:** Apply real forward Flyway migrations to production-shaped PostgreSQL; verify backfills, legacy compatibility, constraints, uniqueness, and that no history/timestamps/events are fabricated.
- **Events and Intelligence tests:** Atomic event persistence, safe concurrent claiming, restart recovery, bounded retry/exhaustion, duplicate delivery, provider timeout/error/malformed output, coherent versioned results, and Core success during downstream failure.
- **Analytics tests:** Duplicate/out-of-order convergence, rebuild equivalence, selected time-range metrics, null legacy milestone times, derived `NO_RESPONSE`, user scoping, projection lag/failure isolation, and no dependency on successful AI.
- **Frontend integration tests:** Navigation and shared states; filters/sort/pagination; manual and URL-prefill fallback; Application Detail history and Intelligence states; Analytics range consistency; responsive/accessibility checks; visual comparison with final Figma.
- **Manual end-to-end verification:** Current SEEK and Indeed full-page/dynamic-panel flows through extraction, pairing, save, duplicate, and failure recovery; then confirm the Application is unified with Web-created Applications.
- **Regression:** Existing Google login/logout, manual and URL-prefill Application creation, list/search/filter/pagination, detail/update/status/delete, frontend build, backend verification, and production configuration checks remain green. Live external-provider availability is verified separately from normal offline CI when credentials are available.

# Definition of Done

Sprint 2 is done when:

- all accepted Sprint 2 implementation and UI behaviour matches the approved requirements, technical designs, and final Figma/specification, with deliberate deviations documented;
- SEEK and Indeed capture, authenticated save, duplicate, authentication-required, extraction-failure, and save-failure flows are verified against current supported pages;
- Core Application creation is atomic, server-owned, duplicate-safe, and demonstrably independent of AI, Analytics, and Redis availability;
- forward-only migrations preserve valid Sprint 1 data and pass PostgreSQL integration verification;
- relevant backend, Extension, frontend, contract, migration, async, Analytics, and regression checks pass, and required CI is green on the release candidate;
- security, ownership, credential isolation, privacy, and error behaviour are reviewed, with no secrets committed;
- implementation, API/operational/release documentation, and known limitations are current;
- focused feature branches have been reviewed and merged through the repository workflow, an immutable candidate SHA and rollback considerations are recorded, and the increment is ready for Sprint Review and release/tag closure.

# Risks and Watch Items

- SEEK and Indeed DOM/SPA variation may invalidate extraction assumptions; maintain isolated fixtures, current-page manual checks, stale-context clearing, and safe failure.
- Pairing and Extension credential handling must preserve the current Job context without exposing credentials to page execution or weakening server-derived ownership.
- Concurrent or repeated ingestion must converge through domain logic plus database uniqueness, preserve an existing Application status, and return an understandable duplicate outcome.
- At-least-once delivery, process interruption, and AI provider latency/failure require idempotency, bounded retry, visible terminal state, and proof that Core remains successful.
- Web and Extension behaviour must stay aligned with the final Figma/specification, especially the Extension-first hierarchy and non-blocking Intelligence states.
- **Unresolved design handoff:** the approved requirements identify citizenship, permanent residency, security clearance, working rights, and sponsorship as relevant eligibility-screening information, while the final UI/UX specification limits Extension Review Detail specifically to citizenship/permanent-residency findings. Implementation must not invent how the other confirmed screening outcomes are surfaced; resolve and document this product/UI contract before closing the affected eligibility issue. This does not block unrelated Core capture work.
- **Conflicting UI/UX documents:** files under `design/sprint-2/ui-ux/v1/` are still labeled final but conflict with the newer Final v2.0 specification. Notable differences include an Apply-intent/pending-confirmation workflow versus v2.0's explicit `Save to OfferBuddy` creation action, different Analytics time-range options/defaults, and persisted versus derived `NO_RESPONSE` semantics. This plan follows v2.0 as the newer explicitly named final handoff and the Phase 3 designs for data semantics; the repository should mark v1 superseded or reconcile it before creating the affected Extension and Analytics implementation Issues.

# Sprint Backlog and Delivery Handoff

This plan is the implementation-level input to the delivery workflow:

```text
Sprint 2 Milestone
  -> GitHub Issues with acceptance criteria and design links
  -> focused feature branches
  -> implementation and local verification
  -> pull request, CI, and review
  -> release-branch integration and Sprint Review
  -> Retrospective
  -> accepted immutable release/tag
```

Issues should decompose the workstreams into reviewable vertical outcomes and carry forward the dependencies, tests, and unresolved eligibility handoff above. This document does not create or commit those Issues.

---

# Expected Outcome

At the end of Sprint 2, a user can reliably record a SEEK or Indeed Application from the Browser Extension, see the same Application in the Web application regardless of ingestion path, receive Job Intelligence asynchronously, and view the approved basic Application Analytics without optional processing affecting the Core save operation.

---

# Related Documents

- [Sprint 2 Requirements](../../product/sprint-2-requirements.md)
- [Sprint 2 Architecture Design](../../architecture/sprint-2-architecture-design.md)
- [Sprint 2 Technical Design Index](../../design/sprint-2/README.md)
- [Final UI/UX Specification v2.0](<../../design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md>)
- [Delivery Roadmap](../roadmap.md)
- [Product Backlog](../product-backlog.md)
- [Testing Strategy](../../quality/testing-strategy.md)
- [Definition of Done](../../quality/definition-of-done.md)
