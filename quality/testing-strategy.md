# Testing Strategy

## Purpose

This document describes the testing and automated validation that exists for the current OfferBuddy system after Sprint 2. Test quantity alone is not treated as evidence of quality; coverage is evaluated against risks and behaviours.

Sprint 2-specific checklists and gates live under [`quality/s2/`](s2/README.md).

## Backend Test Layers

Backend CI runs:

```text
./mvnw -B --no-transfer-progress clean verify
```

### Service Tests

Service tests use JUnit 5, Mockito, and focused in-memory stores to exercise business behaviour without HTTP or a real database.

Covered behaviour includes:

- application creation, validation, initial `APPLIED` status, create-or-reuse, and duplicate behaviour
- application listing, pagination input, filtering, sorting, detail, update, status update, and deletion
- job find-or-create / selective refresh behaviour
- user creation and lookup
- parsing orchestration and failure mapping
- Extension request validation and ingestion orchestration where unit-tested
- Analytics derivation helpers such as no-response classification

### Controller and API Tests

MockMvc tests exercise the HTTP boundary for:

- current user
- job parsing
- application create/list/detail/update/status/delete/history/intelligence
- Extension pairing/track endpoints where covered
- Analytics dashboard reads
- request validation
- pagination and query parameters
- HTTP response status and JSON shape
- consistent business-error responses

These tests use authenticated principals where appropriate and verify that controllers remain separate from persistence entities.

### Security Tests

Security-specific tests cover:

- Google OAuth/OIDC success handling
- local user mapping during login
- unauthenticated API `401` behaviour
- protected endpoints
- CSRF requirements on state-changing Web requests
- logout and session invalidation
- Extension credential authentication paths
- authorised current-user access

Ownership is also tested through user-scoped application and analytics behaviour.

### Repository, Events, and Transaction Integration Tests

Testcontainers starts PostgreSQL 17 for persistence tests. These verify behaviour that an in-memory substitute would not represent reliably:

- Flyway schema application including Sprint 2 migrations
- user repository persistence and identity uniqueness
- application persistence, history, and list queries
- filters, ordering, and pagination against PostgreSQL
- duplicate constraints
- job/application creation with Business Event persistence
- Business Event claim/process/retry behaviour
- Job Intelligence and Analytics persistence/projection paths
- rollback without leaving orphan Core/event state

### OpenAPI Tests

The generated springdoc contract is checked for implemented routes and the session-cookie and CSRF security schemes. Swagger UI availability is tested in the non-production configuration while protected application APIs remain authenticated.

Production disables springdoc API docs and Swagger UI.

### AI and Content-Acquisition Tests

Automated offline tests cover:

- job-page HTTP status and content-type handling
- HTML text extraction and size/validation behaviour
- parsing orchestration
- valid, missing, malformed, and provider-error JSON
- Gemini client configuration, timeout, and error mapping
- Job Intelligence structured-output validation
- controller responses for parsing failures

A live Gemini integration test exists but is conditional. It runs only when `GOOGLE_API_KEY` is supplied and is skipped in normal offline CI. CI passing therefore does not prove current external-provider availability or model behaviour.

### Production Configuration Tests

The production configuration verifier is tested to ensure required database and Google settings are present and local placeholder secrets are rejected.

## Frontend Validation

Frontend CI currently runs:

```text
npm ci
npm run lint
npm run build
```

The build includes TypeScript compilation and Vite's production build. On `main` and `release` pushes, CI uploads the resulting immutable `dist` artifact.

Frontend Vitest component/page tests exist for key Sprint 2 surfaces (Home extension discovery, Analytics, Job Intelligence section, and related flows). They run locally / via precheck; Frontend CI adds `npm test` on the application CI follow-up branch (`review/s2-ci-followups`) until merged to `main`.

Critical journeys still require manual verification for OAuth, Nginx, and live Extension/site behaviour.

## Browser Extension Validation

The Extension package provides:

```text
npm test
npm run verify   # tests + production build
```

There is a dedicated **Extension CI** workflow in the application repository (path-filtered `npm run verify`) on the CI follow-up branch pending merge to `main`. See [Extension Validation](s2/extension-validation.md). **Extension Publish** is separate and manual.

## Local Precheck

Repository scripts `scripts/precheck.ps1` / `scripts/precheck.sh` mirror the main local verify path:

- frontend lint + build
- extension verify
- backend `mvn clean verify`

## CI Versus Integration Verification

A passing CI run confirms that the source at that commit passed the configured automated checks and produced the expected artifact. It does not by itself prove:

- Google production OAuth configuration
- EC2/Nginx routing and HTTPS
- current SEEK/Indeed page compatibility
- Gemini availability
- deployment success
- production data persistence and recovery
- complete browser Extension behaviour

Release-candidate verification and production smoke testing remain separate delivery steps.

## Known Gaps

- Frontend Vitest gate and Extension CI pending merge of application `review/s2-ci-followups`
- no browser end-to-end automation against live SEEK/Indeed
- live Gemini test skipped without an API key
- incomplete generated OpenAPI error-response annotations in places
- no automated performance/load test
- no automated EC2 recovery or database restore schedule
- limited request/correlation ID coverage

## Sprint 2 Verification Map

| Area | Proof |
| --- | --- |
| Extension adapters / lifecycle / boundaries | Extension Vitest + manual current-site checks |
| Pairing / Track / ownership | Backend tests + manual pairing/save |
| Business Events | Backend integration tests |
| Job Intelligence | Backend tests + Detail UI states |
| Analytics | Backend + frontend tests + manual smoke |
| Release readiness | [Release Quality Gate](s2/release-quality-gate.md) |
