# OfferBuddy Sprint 2 GitHub Issues (#38–#53)

> Exported from the private OfferBuddy GitHub repository on 2026-08-19.
> This document preserves the current Issue titles and Markdown descriptions for task redistribution.

## Index

- [#38 — Implement canonical Job and user-owned Application tracking core](https://github.com/peter383666/offerbuddy/issues/38)
- [#39 — Implement Extension pairing and revocable credential lifecycle](https://github.com/peter383666/offerbuddy/issues/39)
- [#40 — Expose authenticated Extension application ingestion](https://github.com/peter383666/offerbuddy/issues/40)
- [#41 — Build Chrome MV3 Extension capture shell and save-state flow](https://github.com/peter383666/offerbuddy/issues/41)
- [#42 — Implement reliable SEEK job capture adapter](https://github.com/peter383666/offerbuddy/issues/42)
- [#43 — Implement reliable Indeed job capture adapter](https://github.com/peter383666/offerbuddy/issues/43)
- [#44 — Resolve and implement the S2 eligibility review contract](https://github.com/peter383666/offerbuddy/issues/44)
- [#45 — Implement durable PostgreSQL Business Event processing](https://github.com/peter383666/offerbuddy/issues/45)
- [#46 — Implement asynchronous Job Intelligence enrichment](https://github.com/peter383666/offerbuddy/issues/46)
- [#47 — Implement basic user-scoped Application Analytics backend](https://github.com/peter383666/offerbuddy/issues/47)
- [#48 — Integrate the S2 Home and Applications web experience](https://github.com/peter383666/offerbuddy/issues/48)
- [#49 — Integrate Application Detail lifecycle and Job Intelligence states](https://github.com/peter383666/offerbuddy/issues/49)
- [#50 — Preserve the secondary manual and URL-prefill New Application flow](https://github.com/peter383666/offerbuddy/issues/50)
- [#51 — Implement the basic Application Analytics web page](https://github.com/peter383666/offerbuddy/issues/51)
- [#52 — Complete Sprint 2 integration, security, and regression verification](https://github.com/peter383666/offerbuddy/issues/52)
- [#53 — Prepare and promote the tested Sprint 2 release candidate](https://github.com/peter383666/offerbuddy/issues/53)

---

## #38 — Implement canonical Job and user-owned Application tracking core

**GitHub:** https://github.com/peter383666/offerbuddy/issues/38

## Background

Sprint 2 needs one Core tracking path that preserves shared canonical Jobs and user-owned Applications across Extension, manual, and URL-prefill ingestion.

## Scope

- [ ] Resolve Jobs by source platform and external identifier; create when absent and selectively refresh reliable source facts without null erasure.
- [ ] Create-or-reuse one Application per user and Job while preserving the deployed `job_applications` name and existing status.
- [ ] Add `WEB` / `EXTENSION` creation source and real status history, including initial `null -> APPLIED` for new records.
- [ ] Persist Core, initial history, and required Business Event records atomically.
- [ ] Add owning forward-only Flyway migrations, reliable backfills, constraints, indexes, and concurrency protection.

## Acceptance Criteria

- [ ] Shared Job identity and user-owned Application uniqueness are enforced in domain logic and PostgreSQL.
- [ ] Repeated/concurrent ingestion converges without resetting status.
- [ ] Selective refresh never erases facts with missing input.
- [ ] Legacy unknown history/timestamps are not fabricated.
- [ ] Core/history/event writes commit or roll back together.
- [ ] Service, repository, concurrency, rollback, and production-shaped migration tests pass.

## Dependencies

- None; foundation for ingestion, async processors, Analytics, and S2 reads.

## Out of Scope

Synchronous AI/Analytics, event sourcing, CQRS, and microservice decomposition.

## Related Documents

- `offerbuddy-engineering/delivery/s2/sprint-plan.md`
- `offerbuddy-engineering/design/sprint-2/backend-service-design.md`
- `offerbuddy-engineering/design/sprint-2/database-design.md`

---

## #39 — Implement Extension pairing and revocable credential lifecycle

**GitHub:** https://github.com/peter383666/offerbuddy/issues/39

## Background

The Browser Extension needs an authenticated identity paired through the existing Web session without exposing Web credentials to page execution.

## Scope

- [ ] Implement pairing create, Web approval, one-time exchange, current-user, and revoke operations under `/api/v1/extension/auth`.
- [ ] Issue finite-lived revocable Extension credentials and derive ownership server-side.
- [ ] Preserve pending, consumed, expired, conflict, authentication-required, and idempotent approval semantics.
- [ ] Add owning persistence migration and protect pairing secrets/credentials.

## Acceptance Criteria

- [ ] Approved paths, statuses, errors, and OpenAPI shapes are implemented.
- [ ] Another user cannot take over a pairing.
- [ ] Exchange is atomic/one-time; credentials expire and revoke.
- [ ] Secrets are not logged, committed, or exposed to page execution.
- [ ] API, persistence, expiry, replay, ownership, and concurrency tests pass.

## Dependencies

- Existing Sprint 1 Web session authentication.

## Out of Scope

New login providers or long-lived non-revocable credentials.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/backend-api-design.md`
- `offerbuddy-engineering/design/sprint-2/backend-service-design.md`
- `offerbuddy-engineering/design/sprint-2/database-design.md`

---

## #40 — Expose authenticated Extension application ingestion

**GitHub:** https://github.com/peter383666/offerbuddy/issues/40

## Background

Reliable Extension facts must be validated and orchestrated by the backend, which owns identity, ownership, duplicates, and business truth.

## Scope

- [ ] Implement `POST /api/v1/extension/applications` with approved validation and stable errors.
- [ ] Authenticate Extension credentials and derive user ownership server-side.
- [ ] Orchestrate canonical Job resolution/refresh and Application create-or-reuse.
- [ ] Return duplicates as `200` with `alreadyTracked=true` and existing status.
- [ ] Keep Core independent of AI, Analytics, Redis, and provider availability.

## Acceptance Criteria

- [ ] Contract and OpenAPI match the approved design.
- [ ] Unsupported platforms/unreliable required facts fail safely.
- [ ] Client-supplied ownership is impossible.
- [ ] Duplicate/concurrent requests converge without status reset.
- [ ] Downstream failures do not affect Core success.
- [ ] Contract, auth, validation, ownership, concurrency, and rollback tests pass.

## Dependencies

- Canonical Job/Application tracking core.
- Extension pairing and credential lifecycle.

## Out of Scope

AI in the request path, match scoring, auto-apply, cover letters, and resume tailoring.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/backend-api-design.md`
- `offerbuddy-engineering/design/sprint-2/backend-service-design.md`

---

## #41 — Build Chrome MV3 Extension capture shell and save-state flow

**GitHub:** https://github.com/peter383666/offerbuddy/issues/41

## Background

The Browser Extension is the primary Sprint 2 workflow and needs a secure deterministic shell before site-specific adapters connect.

## Scope

- [ ] Create MV3 structure, common page representation, content-script/adapter boundary, service-worker communication, and credential isolation.
- [ ] Implement dynamic-context coordination, stale-context clearing, mutation coalescing, and explicit save.
- [ ] Connect pairing/authentication and backend save.
- [ ] Implement approved ready, review, saving, saved, duplicate, auth-required, and failure states from final v2.0/Figma.

## Acceptance Criteria

- [ ] Page execution cannot access credentials.
- [ ] Only explicit action saves the current validated context.
- [ ] Navigation clears stale facts.
- [ ] Approved states preserve context and recovery.
- [ ] Backend outcomes map correctly.
- [ ] State, auth-expiry, stale-context, failure, and isolation tests pass.

## Dependencies

- Extension pairing/credentials.
- Authenticated Extension ingestion.
- Site adapters may develop against fixtures in parallel.

## Out of Scope

LinkedIn, auto-apply, broad platform expansion, and background crawling.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/extension-design.md`
- `offerbuddy-engineering/design/sprint-2/backend-api-design.md`
- `offerbuddy-engineering/design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md`
- `offerbuddy-engineering/decisions/ADR-009-browser-extension-site-adapters.md`

---

## #42 — Implement reliable SEEK job capture adapter

**GitHub:** https://github.com/peter383666/offerbuddy/issues/42

## Background

SEEK is an approved initial platform with full-page, dynamic, and side-panel contexts.

## Scope

- [ ] Implement an isolated SEEK adapter against the common representation.
- [ ] Extract reliable facts including stable platform/external identity.
- [ ] Handle supported SPA/side-panel navigation, missing optional facts, noisy mutations, and stale context.
- [ ] Maintain representative fixtures and connect to the common Extension flow.

## Acceptance Criteria

- [ ] Current supported SEEK contexts yield deterministic validated facts.
- [ ] Unreliable required identity fails safely.
- [ ] Missing optional facts are not fabricated.
- [ ] Navigation replaces stale context.
- [ ] Fixture tests and current-site manual checks cover success/failure variants.

## Dependencies

- Chrome MV3 shell and adapter contract; fixtures can precede backend completion.

## Out of Scope

LinkedIn and generalized scraping infrastructure.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/extension-design.md`
- `offerbuddy-engineering/decisions/ADR-009-browser-extension-site-adapters.md`

---

## #43 — Implement reliable Indeed job capture adapter

**GitHub:** https://github.com/peter383666/offerbuddy/issues/43

## Background

Indeed is an approved initial platform with full-page, dynamic, and side-panel contexts.

## Scope

- [ ] Implement an isolated Indeed adapter against the common representation.
- [ ] Extract reliable facts including stable platform/external identity.
- [ ] Handle supported SPA/side-panel navigation, missing optional facts, noisy mutations, and stale context.
- [ ] Maintain representative fixtures and connect to the common Extension flow.

## Acceptance Criteria

- [ ] Current supported Indeed contexts yield deterministic validated facts.
- [ ] Unreliable required identity fails safely.
- [ ] Missing optional facts are not fabricated.
- [ ] Navigation replaces stale context.
- [ ] Fixture tests and current-site manual checks cover success/failure variants.

## Dependencies

- Chrome MV3 shell and adapter contract; fixtures can precede backend completion.

## Out of Scope

LinkedIn and generalized scraping infrastructure.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/extension-design.md`
- `offerbuddy-engineering/decisions/ADR-009-browser-extension-site-adapters.md`

---

## #44 — Resolve and implement the S2 eligibility review contract

**GitHub:** https://github.com/peter383666/offerbuddy/issues/44

## Background

Requirements cover citizenship, permanent residency, clearance, working rights, and sponsorship, while final UI v2.0 limits Review Detail to citizenship/permanent residency. This is a narrow handoff gap, not authority to reopen Phase 4.

## Scope

- [x] Obtain and record the narrow product/UI clarification for non-citizenship outcomes.
- [x] Implement approved Review Required and Review Detail behavior.
- [x] Preserve no restriction, review required, explicit restriction, and unable-to-determine outcomes.
- [x] Keep language factual and do not equate ambiguous working rights with permanent residency.

## Acceptance Criteria

- [x] Clarification is documented without scope expansion.
- [x] All approved categories have implementation-facing treatment.
- [x] No default eligibility dashboard or absolute eligibility decision is added.
- [x] Outcome, ambiguity, missing-evidence, and UI-state tests pass.

## Dependencies

- Extension shell and captured job content.
- Does not block unrelated Core capture work.

## Out of Scope

Match scoring, broad eligibility advice, or reopening Phase 4.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/eligibility-review-contract.md` (S2 product/UI contract)
- `offerbuddy-engineering/delivery/s2/sprint-plan.md`
- `offerbuddy-engineering/product/sprint-2-requirements.md`
- `offerbuddy-engineering/design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md`

---

## #45 — Implement durable PostgreSQL Business Event processing

**GitHub:** https://github.com/peter383666/offerbuddy/issues/45

## Background

Non-critical S2 work must run asynchronously without distributed infrastructure or loss across restarts.

## Scope

- [ ] Implement PostgreSQL-backed `business_events` and bounded claim/process/outcome phases.
- [ ] Support concurrent claiming, abandoned-claim recovery, bounded retry/exhaustion, visible failure/lag, and idempotency.
- [ ] Keep external calls and long locks outside Core transactions.
- [ ] Add owning migration and operational diagnostics.

## Acceptance Criteria

- [ ] Core atomically persists events without downstream work.
- [ ] Claims are concurrency-safe and interrupted work recovers.
- [ ] Retries are bounded and terminal failure observable.
- [ ] Duplicate delivery is handled idempotently.
- [ ] Processor failure does not block Core success.
- [ ] PostgreSQL integration tests cover concurrency, recovery, retry, exhaustion, duplicates, and rollback.

## Dependencies

- Core event production and `business_events` persistence contract.

## Out of Scope

Redis queues/caches, Kafka, RabbitMQ, exactly-once, CQRS, and event sourcing.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/event-design.md`
- `offerbuddy-engineering/decisions/ADR-010-lightweight-business-events.md`
- `offerbuddy-engineering/design/sprint-2/database-design.md`

---

## #46 — Implement asynchronous Job Intelligence enrichment

**GitHub:** https://github.com/peter383666/offerbuddy/issues/46

## Background

Persisted Jobs need structured understanding, but AI is downstream and never owns Job identity or Application lifecycle success.

## Scope

- [ ] Consume durable Job snapshot events asynchronously.
- [ ] Produce/validate concise factual summary, responsibilities, requirements, and skills.
- [ ] Persist versioned attempts and coherent pending/completed/unavailable/failed results.
- [ ] Bound retries and handle missing description, malformed output, timeout, and provider failure.
- [ ] Add owning migrations and backend-only provider configuration.

## Acceptance Criteria

- [ ] Application creation succeeds during all provider/processor failures.
- [ ] Results derive from persisted Job content and never determine identity/lifecycle.
- [ ] Only coherent validated results become current; handling is idempotent.
- [ ] Failure states are distinguishable.
- [ ] Validation, duplicate, retry, timeout, malformed output, versioning, and isolation tests pass with mocked providers in CI.

## Dependencies

- Durable Business Events.
- Persisted canonical Job snapshots.

## Out of Scope

Match scoring, resume tailoring, cover letters, synchronous AI, or replacing URL-prefill parsing.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/job-intelligence-design.md`
- `offerbuddy-engineering/design/sprint-2/event-design.md`
- `offerbuddy-engineering/decisions/ADR-004-ai-provider-abstraction.md`

---

## #47 — Implement basic user-scoped Application Analytics backend

**GitHub:** https://github.com/peter383666/offerbuddy/issues/47

## Background

Sprint 2 needs a deliberately small eventually consistent view of each user's Application activity and outcomes.

## Scope

- [ ] Add rebuildable one-row-per-Application projection from `job_applications`, real history, and reliable facts.
- [ ] Handle duplicate/out-of-order events idempotently and preserve earliest reliable milestone time.
- [ ] Implement deterministic rebuild/reconciliation.
- [ ] Expose authenticated user-scoped approved range, summary, outcome, and conversion reads.
- [ ] Derive `NO_RESPONSE` without persisting lifecycle or fabricating dates; add owning migrations.

## Acceptance Criteria

- [ ] Arbitrary `userId` selection is impossible.
- [ ] Duplicate/out-of-order events converge without double count.
- [ ] Rebuild equals incremental output.
- [ ] Unknown legacy times remain null and `NO_RESPONSE` is derived.
- [ ] Projection/AI failure does not affect Core.
- [ ] Range, metric, isolation, rebuild, lag, and legacy tests pass.

## Dependencies

- Application lifecycle history/source facts.
- Durable Business Events.

## Out of Scope

Large-scale BI, cohorts, predictive metrics, match scoring, or AI-dependent Analytics.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/analytics-design.md`
- `offerbuddy-engineering/design/sprint-2/database-design.md`

---

## #48 — Integrate the S2 Home and Applications web experience

**GitHub:** https://github.com/peter383666/offerbuddy/issues/48

## Background

Web must present Extension and Web-created Applications coherently while making the Extension the primary S2 workflow.

## Scope

- [ ] Implement final v2.0 shell/navigation/branding.
- [ ] Implement approved Home summary, Extension discovery, and recent Applications.
- [ ] Update Applications search, status/source filters, sort, table, and pagination.
- [ ] Connect APIs with loading, empty, error, populated, responsive, and accessible states.

## Acceptance Criteria

- [ ] Both creation sources appear in one user-scoped experience.
- [ ] Home preserves Extension-first hierarchy.
- [ ] Controls/source/status match final v2.0.
- [ ] Responsive/accessibility states are verified.
- [ ] Frontend integration and final Figma/spec visual comparison pass.

## Dependencies

- Stable S2 Application reads; static states may proceed in parallel.

## Out of Scope

Major Analytics expansion or platforms beyond SEEK/Indeed.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md`
- `offerbuddy-engineering/design/sprint-2/backend-api-design.md`

---

## #49 — Integrate Application Detail lifecycle and Job Intelligence states

**GitHub:** https://github.com/peter383666/offerbuddy/issues/49

## Background

Application Detail must show authoritative lifecycle and non-blocking Job Intelligence in one useful view.

## Scope

- [ ] Extend user-scoped detail reads for creation source, real history, and Intelligence state/result.
- [ ] Implement final v2.0 summary, details, history, source, destructive action, and Intelligence presentation.
- [ ] Handle pending, unavailable, failed, and completed Intelligence.
- [ ] Preserve Sprint 1 update/delete authorization.

## Acceptance Criteria

- [ ] Users access only their own detail/history.
- [ ] History contains real ordered transitions only.
- [ ] Intelligence shows only summary, responsibilities, requirements, and skills.
- [ ] Failure states leave authoritative detail usable.
- [ ] Loading/not-found/error/responsive/accessibility states match final v2.0.
- [ ] Backend/frontend and visual comparison tests pass.

## Dependencies

- Core lifecycle history.
- Asynchronous Job Intelligence.

## Out of Scope

Match scoring, generated materials, or AI ownership of lifecycle.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md`
- `offerbuddy-engineering/design/sprint-2/job-intelligence-design.md`

---

## #50 — Preserve the secondary manual and URL-prefill New Application flow

**GitHub:** https://github.com/peter383666/offerbuddy/issues/50

## Background

The Extension is primary, but manual and AI URL-prefill New Application remain explicit secondary fallbacks.

## Scope

- [ ] Implement final v2.0 New Application hierarchy while retaining manual entry and URL prefill.
- [ ] Route creation through compatible Core semantics with `WEB` source.
- [ ] Keep parsed fields editable and preserve manual recovery from parsing failure/incomplete results.
- [ ] Verify responsive/accessibility and Sprint 1 regression.

## Acceptance Criteria

- [ ] Extension discovery is primary and both fallback paths remain.
- [ ] Parsing only prefills and never auto-creates.
- [ ] Manual creation remains usable after parser failure.
- [ ] Web and Extension Applications unify and follow duplicate rules.
- [ ] UI states and regression/integration tests pass.

## Dependencies

- Canonical Job/Application core and stable Web creation contract.

## Out of Scope

Making URL parsing primary, crawling, auto-apply, or expanding AI parsing responsibility.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md`
- `offerbuddy-engineering/design/sprint-2/backend-service-design.md`

---

## #51 — Implement the basic Application Analytics web page

**GitHub:** https://github.com/peter383666/offerbuddy/issues/51

## Background

Users need the approved compact Analytics view, not a broad reporting platform.

## Scope

- [ ] Implement final v2.0 Analytics navigation, time-range selector, summary, outcomes, and conversions.
- [ ] Connect authenticated Analytics API and apply range consistently.
- [ ] Implement loading, empty, error, populated, responsive, and accessible states.

## Acceptance Criteria

- [ ] Only final v2.0 ranges/metrics appear.
- [ ] Selected range controls all metrics.
- [ ] `NO_RESPONSE` is an Analytics classification, not lifecycle history.
- [ ] Accessibility/responsive behavior is verified.
- [ ] Frontend/API integration and Figma/spec comparison pass.

## Dependencies

- Basic user-scoped Analytics backend.

## Out of Scope

Custom dashboards, exports, cohorts, prediction, match scoring, or persisted `NO_RESPONSE`.

## Related Documents

- `offerbuddy-engineering/design/sprint-2/ui-ux/OfferBuddy S2 — UI UX Page Specification v2.0 — Final.md`
- `offerbuddy-engineering/design/sprint-2/analytics-design.md`

---

## #52 — Complete Sprint 2 integration, security, and regression verification

**GitHub:** https://github.com/peter383666/offerbuddy/issues/52

## Background

Prove integrated S2 flows, security, migrations, async recovery, UI behavior, and Sprint 1 regression on the release branch.

## Scope

- [ ] Run backend, Extension, frontend, contract, migration, async, Analytics, accessibility, and regression verification.
- [ ] Manually test current SEEK/Indeed full-page and dynamic-panel pairing, extraction, save, duplicate, expiry, and recovery.
- [ ] Verify ownership, credential isolation, privacy, downstream failure isolation, restart recovery, and production-shaped migrations.
- [ ] Track discovered defects with the Bug template.

## Acceptance Criteria

- [ ] Feature PRs target `release`, not `main`, with green CI.
- [ ] Supported SEEK/Indeed flows pass and unify with Web creation.
- [ ] Core succeeds during AI/Analytics failure and without Redis.
- [ ] Auth/concurrency/events/Analytics/migration/isolation evidence is recorded.
- [ ] Final v2.0 UI/accessibility states pass.
- [ ] Sprint 1 auth, creation, lists, detail/update/status/delete, builds, and production config remain green.
- [ ] No blocking defect remains.

## Dependencies

- All S2 feature issues integrated.
- Eligibility clarification resolved for affected checks ([eligibility-review-contract.md](../../design/sprint-2/eligibility-review-contract.md)).

## Out of Scope

Feature-to-main merging or unrelated scope.

## Related Documents

- `offerbuddy-engineering/delivery/s2/sprint-plan.md`
- `offerbuddy-engineering/quality/testing-strategy.md`
- `offerbuddy-engineering/operations/development-workflow.md`

---

## #53 — Prepare and promote the tested Sprint 2 release candidate

**GitHub:** https://github.com/peter383666/offerbuddy/issues/53

## Background

Produce an auditable release candidate from `release` and promote only the tested immutable candidate to `main`, tag, and release.

## Scope

- [ ] Maintain `release` as integration/verification branch.
- [ ] Confirm reviewed feature PRs, CI, candidate SHA, migration/config impact, rollback, limitations, and release notes.
- [ ] Deploy/verify the candidate as required.
- [ ] Promote the exact tested candidate to `main`, then tag/release.
- [ ] Update implementation, API, operational, and release documentation.

## Acceptance Criteria

- [ ] No feature branch merges directly to `main`.
- [ ] One immutable green candidate SHA and verification evidence are recorded.
- [ ] Migration/config/rollback/limitations/release notes are current.
- [ ] Tested candidate is promoted without divergence.
- [ ] Tag/release occurs only after promotion/verification.
- [ ] Sprint Review evidence is ready.

## Dependencies

- Complete integration/security/regression verification.
- All accepted feature PRs merged to `release`.

## Out of Scope

Direct feature-to-main merges or unrelated infrastructure redesign.

## Related Documents

- `offerbuddy-engineering/operations/development-workflow.md`
- `offerbuddy-engineering/quality/definition-of-done.md`
- `offerbuddy-engineering/delivery/s2/sprint-plan.md`

---
