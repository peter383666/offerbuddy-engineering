# Sprint 3 Verification and Release Readiness

## Purpose

This unit defines the evidence required to accept Sprint 3 implementation and promote a release candidate. It applies the frozen requirements, architecture, technical design, API contract, and page specifications; it does not restate their detailed acceptance criteria.

## Verification principles

- Verify observable state and contract behaviour, not implementation structure alone.
- Test each delivery unit at its owning boundary, then repeat critical paths after integration.
- Treat migration, security, privacy, idempotency, optimistic concurrency control (OCC), and stale-data handling as release evidence.
- Preserve the Sprint 2 Fast Path while adding the Sprint 3 Focused Preparation path.
- Use deterministic fakes for automated AI-path tests; reserve provider smoke tests for controlled environments.
- Record commands, environment, fixture version, and relevant correlation IDs in PR evidence.

## Delivery-unit evidence matrix

| Area | Required focused evidence | Integration evidence |
| --- | --- | --- |
| Shared revisions and diagnostics | Aggregate version changes, common error mapping, `correlation_id` persistence and propagation | One correlation trace across HTTP, business event, worker retry, and AI execution |
| Candidate Profile | Ownership, validation, `profileVersion` OCC, authorised read/update | Candidate update immediately changes Preparation readiness without leaking another user's data |
| Resume Import | File validation, async state transitions, extraction failure/retry, draft isolation, explicit merge | Accepted import updates Candidate once; rejected/unaccepted draft does not mutate Candidate |
| Job and Intelligence delta | `contentVersion`, source-version comparison, stale/refresh behaviour | Re-captured job invalidates or refreshes dependent Intelligence and Preparation state correctly |
| Preparation and Match | Readiness rules, generation idempotency, grounded result provenance, failure states | Candidate + Job produces Match; stale source versions remain visible and do not silently overwrite current output |
| Tailored Resume | Authorised assets, generation, limited edit, artefact OCC, render isolation, preview/download | Edit creates the frozen revision behaviour and re-rendered output remains tied to the correct source versions |
| Focused Cover Letter | Generation/edit persistence, stale and failure states, render/download | Focused output remains separate from deterministic Default/Fast assistance |
| Application Detail | Existing page regression and additive Preparation summary states | Existing application actions still work with incomplete, ready, and generated Preparation states |
| Extension | LinkedIn extraction fixtures, lifecycle cleanup, snapshot versioning, local lookup, hand-off states | Repeated supported-page navigation does not duplicate jobs or listeners; Preparation opens the correct job |
| Sponsor | Working-dataset correction permissions, validation/publish transition, immutable published version, Redis projection | Extension refreshes a published snapshot and performs page-time lookup locally without a second source of truth |
| Default/SEEK assistance | Deterministic field mapping, eligibility, source priority and missing-field handling | Fast Path remains non-AI and uses a fresh approved Focused letter only when allowed by the frozen rules |
| AI platform and Admin | Capability registration, routing, timeout/retry, redaction, RBAC, OCC, safe monitoring projection | Runtime config change is auditable; failure detail exposes metadata but no prompt, response, resume, or candidate content |

The delivery-unit PR owns the focused evidence. `S3-I01` owns the integration column and must not compensate for missing focused tests.

## Database and migration verification

Verify the committed V10–V18 sequence from the [Dependency, Vertical Slice, and Migration Plan](dependency-and-migration-plan.md) using both paths:

1. **Clean build:** create an empty PostgreSQL database and migrate from V1 through the final S3 migration.
2. **Upgrade build:** restore a representative V9 Sprint 2 database, migrate through V10–V18, and run compatibility checks against retained Sprint 2 records.

Required evidence:

- Flyway validation succeeds and every migration is applied once in the reserved order;
- existing users, jobs, applications, Job Intelligence, and business events remain readable;
- new non-null constraints have safe backfill or compatible defaults;
- version columns and uniqueness rules enforce the frozen concurrency/idempotency semantics;
- V10 `business_events.correlation_id` works for new events without invalidating historical rows;
- V17 bootstrap is repeatable reference data, not runtime provider registration;
- V18 Sponsor working and published data respect the publish lifecycle;
- production-like startup succeeds after migration without manual schema repair.

Merged migrations are immutable. A defect found after merge is corrected by a new forward migration; rollback rehearsal restores the database and object assets from the agreed backup point rather than editing migration history.

## Contract and cross-surface checks

Backend contract tests remain authoritative for status codes, error envelopes, version fields, idempotency keys, async resources, and authorisation. Generated or shared client types must be refreshed when the frozen API representation requires it.

For each Web, Extension, or RuoYi integration:

- exercise loading, empty, ready, stale, validation, forbidden, conflict, failed, and retryable states that the approved page specification assigns to that surface;
- confirm clients do not derive domain truth that belongs to the backend;
- confirm retries reuse the appropriate idempotency/request identity;
- confirm an OCC conflict preserves the user's input and presents the approved recovery action;
- confirm links and hand-offs carry identifiers only through approved routes and do not expose private content in URLs or logs.

Contract fixtures should be versioned with the consuming client unit. Temporary mock responses must be removed or proven equivalent before integration acceptance.

## Regression paths

### Sprint 2 Fast Path

The release candidate must repeat the existing critical path:

```text
supported SEEK page
→ job detection/capture
→ application workflow
→ deterministic Default cover-letter assistance where eligible
→ existing application tracking
```

It must remain usable when AI providers are unavailable. Sprint 3 must not make Candidate completion, Match generation, or Focused artefacts mandatory for the Fast Path.

### Sprint 3 Focused Path

Verify at least one complete path:

```text
Candidate Profile or accepted Resume Import
→ supported job capture
→ Preparation readiness
→ Match Analysis
→ Tailored Resume and Focused Cover Letter
→ authorised preview/download
→ Application Detail summary
```

Repeat with a changed Candidate version and a changed Job content version to verify visible staleness and deliberate regeneration rather than silent replacement.

### Sponsor and Admin path

Verify a working Sponsor import, authorised correction, validation, publish, backend/Redis activation, Extension snapshot refresh, and local lookup. Also verify that an unauthorised Admin cannot publish and that deleting/correcting a working row cannot mutate the active published version.

## Security and privacy gate

The release candidate is blocked unless automated tests and a targeted review confirm:

- Candidate, resume, cover-letter, job, application, and preparation resources enforce user ownership;
- RuoYi Sponsor and AI operations enforce their frozen permissions and audit actor identity;
- object preview/download uses authorised, expiring access and rejects guessed object identifiers;
- upload limits, media validation, filename handling, and failed-file cleanup are applied;
- logs, business events, monitoring views, metrics, and errors exclude resume text, cover-letter content, prompts, responses, credentials, and unnecessary personal data;
- AI provider calls use the approved minimal context and configured secret boundary;
- Extension storage contains only the approved snapshot/configuration data and clears obsolete versions safely;
- CORS, CSP/manifest permissions, and production endpoints remain restricted to the deployed surfaces.

Use repository security scans already present in CI. Any new dependency or permission requires an explicit PR explanation and appropriate automated check; Phase 5 does not prescribe a replacement scanner.

## Async, reliability, and observability gate

For every async capability, verify accepted, running, succeeded, failed, stale, and retry behaviour where applicable. Duplicate delivery or client retry must not create duplicate accepted business outcomes. Worker restart must resume or safely reclaim work under the frozen claim model.

Failure injection must cover at least provider timeout, rendering/storage failure, worker interruption, and stale source version. Evidence must show bounded retry, terminal failure visibility, and safe user recovery.

Operational evidence includes:

- a request-visible correlation ID and matching durable event/AI execution metadata;
- queue/claim age, success/failure/retry counts, and capability/provider dimensions without private content;
- a diagnosable render/storage failure that does not corrupt the last accepted artefact;
- Sponsor active-version and Extension snapshot-version visibility;
- health/readiness behaviour when optional AI capability providers are unavailable.

## Environment and deployment readiness

Before release-candidate acceptance, reconcile repository configuration for PostgreSQL, Redis, object storage/rendering, AI providers, Web, Extension, and RuoYi. Document required variables by name and purpose without committing secrets.

The deployment rehearsal must prove:

1. database backup and V9-to-S3 migration;
2. backend/worker startup with production-like dependencies;
3. Web and Admin compatibility with the deployed API;
4. Extension package/configuration against the release environment;
5. focused smoke paths and observability checks;
6. forward-fix and restore decision points.

Release notes identify schema changes, new services/configuration, Extension update requirements, known limitations, and the retained Fast Path. Runtime AI capability registration/configuration is verified here and in the AI implementation unit; persistent V17 reference bootstrap remains a Flyway concern.

## Release gates

| Gate | Evidence owner | Pass condition |
| --- | --- | --- |
| Unit acceptance | Delivery-unit owner | Focused tests, diff review, frozen references and limitations recorded |
| Migration acceptance | Backend/database owner | Clean and V9 upgrade paths pass; forward recovery is rehearsed |
| Client contract acceptance | Surface owner + backend reviewer | Frozen contract and approved UI states verified without client-owned domain rules |
| Security/privacy acceptance | Backend and surface reviewers | Ownership/RBAC, asset access, upload, secret and redaction checks pass |
| Integration acceptance | `S3-I01` owner | Focused, Fast, Sponsor/Admin and failure paths pass on the integration branch |
| Operational acceptance | Release owner | Deployment/configuration, telemetry, backup/restore and smoke evidence is complete |
| Documentation acceptance | Engineering documentation owner | Implementation changes are reconciled with current-system and Sprint records |

No unit is complete solely because code is merged. Failed required evidence either returns the unit for correction or is recorded as an explicitly accepted release blocker; it is not silently deferred.

## Implementation readiness

Verification planning is implementation-ready when each GitHub Issue created later selects the applicable rows and gates from this document, adds exact frozen acceptance links, and names its focused commands/fixtures. The release candidate is ready only when all required gates have recorded evidence and no unresolved frozen-design contradiction remains.
