# Sprint 2 Review

## Sprint Goal

Make OfferBuddy practical in the user's daily job-application workflow by delivering a Chrome Manifest V3 Browser Extension that reliably captures SEEK and Indeed jobs and records the authenticated user's Application with minimal friction. Core Application creation must complete independently of asynchronous Job Intelligence and Analytics, while the existing manual and URL-prefill workflow remains available as a secondary fallback.

## Delivered

### Browser Extension

- Chrome Manifest V3 Extension for SEEK and Indeed
- Isolated Site Adapters with dynamic / SPA / side-panel awareness
- Explicit **Save to OfferBuddy** popup flow
- Submission-confirmation ingest and uncertain-confirmation companion UX
- Citizenship / permanent-residency eligibility review signalling
- Floating Companion on supported job-search surfaces
- Pairing with the authenticated Web user and privileged credential storage
- Understandable ready, review, saving, saved, duplicate, auth-required, pairing, and failure outcomes

### Backend Ingestion and Core

- Extension pairing create / approve / exchange
- Revocable Extension credentials
- Authenticated Track API with server-derived ownership
- Job resolve/refresh and Application create-or-reuse
- `creation_source` (`WEB` / `EXTENSION`)
- Application status history with initial `null → APPLIED`
- Duplicate outcome via `alreadyTracked` without sync AI/Analytics dependency

### Asynchronous Enrichment

- PostgreSQL-backed Business Events inside the modular monolith
- Bounded claim / retry / recovery processing
- Async Job Intelligence (summary, responsibilities, requirements, skills)
- Rebuildable Application Analytics projection with approved time ranges
- Derived no-response classification alongside retained legacy status handling

### Web Integration

- Home summary cards and Extension discovery when not installed
- Application Detail history and Job Intelligence states
- Analytics page
- Extension connect / pairing approval page
- New Application retained as secondary manual / URL-prefill path

### Database and Delivery Foundations

- Forward Flyway migrations V3–V9
- Single-host production model retained (Nginx, Compose, immutable SHA artifacts)
- Local precheck scripts mirroring primary verify paths

## Engineering Delivered

- Extension Vitest coverage for adapters, lifecycle, companion, pairing/save, and boundaries
- Backend service, API, security, events, intelligence, and analytics tests including Testcontainers
- Frontend Vitest coverage for key S2 surfaces (not yet gated in Frontend CI)
- Backend CI `mvn verify` and Frontend CI lint/build retained
- Documentation closeout information architecture under `docs/sprint-2`

Test count is not the quality conclusion. The meaningful result is that capture ownership, duplicate safety, Core/async separation, and platform adapter risks received targeted verification.

## Scope Changes

### Added or Expanded

- Confirmation-based automatic tracking and uncertain-confirmation UX shipped in addition to explicit Save.
- Floating Companion became part of the Extension experience.
- Frontend automated tests landed for important S2 pages/components even though Frontend CI still gates on lint/build only.

### Changed

- Eligibility surfacing for Sprint 2 closed to citizenship/PR findings; broader requirement concepts remain deferred.
- Analytics treats no-response as a derived classification with a configurable threshold; legacy persisted `NO_RESPONSE` status remains in the enum.
- Job Intelligence exposes an `UNAVAILABLE` style outcome when description content is missing.

### Not Delivered

- LinkedIn or additional recruitment platforms
- Auto Apply / automatic form submission
- Cover Letter, resume tailoring, or match scoring
- Kafka / RabbitMQ / microservices / Kubernetes
- Large Analytics/BI expansion
- Redis-backed application behaviour
- Dedicated Extension GitHub Actions workflow
- Frontend Vitest as a CI gate
- Browser end-to-end automation against live SEEK/Indeed

## Acceptance Criteria Result

| Area | Result | Notes |
| --- | --- | --- |
| Extension SEEK/Indeed capture | Met | Adapters + manual current-site validation still required |
| Authenticated Extension save | Met | Pairing + Track API |
| Duplicate / auth / failure feedback | Met | Approved Extension outcomes implemented |
| Core independent of AI/Analytics | Met | Business Events after Core commit |
| Job Intelligence async enrichment | Met | Non-blocking; controlled failure/unavailable states |
| Basic Application Analytics | Met | Projection + approved ranges |
| Web integration | Met | Home, Detail, Analytics, Connect, secondary New Application |
| Forward migrations | Met | V3–V9 |
| Automated verification | Partially met | Strong backend/extension tests; CI gaps remain |
| Documentation closeout | Met on branch | `docs/sprint-2` pending merge to `main` and `engineering-s2` tag |
| Engineering tag `engineering-s2` | Pending | After merge of documentation PR to `main` |

## Known Limitations

See [Known Limitations](known-limitations.md). Highest-visibility items:

- dual Extension tracking paths require clear product messaging
- SEEK/Indeed DOM drift remains a live operational risk
- Extension CI and Frontend test CI gates are incomplete
- Redis remains reserved/unused
- observability remains health/logs oriented

## Final Sprint Result

Sprint 2 delivered the intended lower-friction capture increment on the existing modular-monolith foundation. The Browser Extension is the preferred capture path; Core save is independent of AI and Analytics; Web surfaces expose history, intelligence, and basic analytics.

Final engineering documentation release tagging (`engineering-s2`) remains a separate closure action after the `docs/sprint-2` PR merges to `main`.

## Related

- [Implementation Status](implementation-status.md)
- [Implementation Reconciliation](implementation-reconciliation.md)
- [Sprint Plan](sprint-plan.md)
- [Retrospective](retrospective.md)
- [Release Notes](release-notes.md)
