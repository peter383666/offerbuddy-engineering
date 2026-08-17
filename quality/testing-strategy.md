# Testing Strategy

## Purpose

This document describes the testing and automated validation that exists at the end of Sprint 1. Test quantity alone is not treated as evidence of quality; coverage is evaluated against risks and behaviours.

## Backend Test Layers

Backend CI runs:

```text
./mvnw -B --no-transfer-progress clean verify
```

### Service Tests

Service tests use JUnit 5, Mockito, and focused in-memory stores to exercise business behaviour without HTTP or a real database.

Covered behaviour includes:

- application creation, validation, initial `APPLIED` status, and duplicate rejection
- application listing, pagination input, filtering, sorting, detail, update, status update, and deletion
- job find-or-create behaviour used by application creation
- user creation and lookup
- parsing orchestration and failure mapping

### Controller and API Tests

MockMvc tests exercise the HTTP boundary for:

- current user
- job parsing
- application create/list/detail/update/status/delete
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
- CSRF requirements on state-changing requests
- logout and session invalidation
- authorised current-user access

Ownership is also tested through user-scoped application behaviour.

### Repository and Transaction Integration Tests

Testcontainers starts PostgreSQL 17 for persistence tests. These verify behaviour that an in-memory substitute would not represent reliably:

- Flyway schema application
- user repository persistence and identity uniqueness
- application persistence and list queries
- filters, ordering, and pagination against PostgreSQL
- duplicate constraints
- job/application creation in one transaction
- rollback without leaving an orphan job

### OpenAPI Tests

The generated springdoc contract is checked for the implemented routes and the session-cookie and CSRF security schemes. Swagger UI availability is tested in the non-production configuration while protected application APIs remain authenticated.

Production disables springdoc API docs and Swagger UI.

### AI and Content-Acquisition Tests

Automated offline tests cover:

- job-page HTTP status and content-type handling
- HTML text extraction and size/validation behaviour
- parsing orchestration
- valid, missing, malformed, and provider-error JSON
- Gemini client configuration, timeout, and error mapping
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

Sprint 1 does not have frontend unit, component, or end-to-end automated tests. ESLint, type checking, and a successful build are useful controls but do not prove runtime UI behaviour.

Critical user journeys were verified manually in production:

- Google login/logout
- application capture and creation
- list/search/filter/pagination
- detail and update
- frontend/backend deployment and smoke checks

Frontend automated testing remains technical debt.

## CI Versus Integration Verification

A passing CI run confirms that the source at that commit passed the configured automated checks and produced the expected artifact. It does not by itself prove:

- Google production OAuth configuration
- EC2/Nginx routing and HTTPS
- job-site accessibility
- Gemini availability
- deployment success
- production data persistence and recovery
- complete browser behaviour

Release-candidate verification and production smoke testing remain separate delivery steps.

## Known Gaps

- no frontend unit/component suite
- no browser end-to-end automation
- live Gemini test skipped without an API key
- incomplete generated OpenAPI error-response annotations
- no automated performance/load test
- no automated EC2 recovery or database restore schedule
- no request/correlation IDs in error responses
