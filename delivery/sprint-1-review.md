# Sprint 1 Review

## Sprint Goal

Deliver a usable production MVP that can replace the core spreadsheet-based job application tracking workflow.

The intended increment was an authenticated application in which a user could capture a job, create an application, find it later, and maintain its current state.

## Delivered

### Authentication

- Google OAuth 2.0/OpenID Connect login
- local OfferBuddy user creation and lookup
- Spring Security server-side session
- current-user and logout APIs
- protected frontend routes and API endpoints
- user-scoped application access

### Application Tracking

- manual application creation
- AI-assisted job parsing from a submitted URL
- editable parsed draft before creation
- application list with server pagination
- company-name and job-title search
- status filtering and supported sorting
- application detail
- status, applied-date, and notes update
- direct status update
- application deletion
- home page with capture entry points and recent applications

### Production

- live HTTPS application at [offerbuddy.io](https://offerbuddy.io)
- host Nginx serving the React build and proxying API/OAuth routes
- Spring Boot, PostgreSQL, and reserved Redis under Docker Compose on AWS EC2
- PostgreSQL persistent volume and Flyway migrations
- independent frontend and backend CI/CD
- immutable SHA backend images and frontend artifacts
- manual explicit-SHA deployment
- health checks and production smoke verification

## Engineering Delivered

- Java 21/Spring Boot modular-monolith backend
- React/TypeScript/Vite frontend
- versioned REST API and OpenAPI documentation
- Google OIDC, session-cookie, CSRF, and ownership security
- PostgreSQL integration with schema migrations
- service and controller tests
- security tests
- PostgreSQL/Testcontainers repository and transaction tests
- OpenAPI tests
- job-content acquisition, parsing, and Gemini-client tests
- production configuration validation
- frontend lint, type compilation, and production-build validation
- PostgreSQL backup script and restore-verification script
- Docker restart and EC2 reboot/recovery verification
- known-good SHA rollback capability

Test count is not used as the quality conclusion. The meaningful result is that business rules, security boundaries, PostgreSQL behaviour, API boundaries, and operational recovery paths received targeted verification.

## Scope Changes

### Added or Expanded

- AI-assisted job parsing was integrated into the delivered capture path; implementation exposed page-acquisition and latency limitations that were less visible in the original plan.
- Production work expanded beyond deployment to include Nginx/HTTPS, independent CI/CD, backup and restore verification, restart/reboot recovery, log review, and deterministic rollback.
- Application list behaviour included server pagination and explicit filtering/sorting validation.

### Changed

- The delivered home page provides capture and recent applications; it is not a statistics dashboard.
- New applications always begin as `APPLIED`.
- The delivered status set is `APPLIED`, `NO_RESPONSE`, `INTERVIEW`, `OFFER`, `REJECTED`, and `WITHDRAWN`.
- Sprint 1 stores current status only rather than a status-change history.
- Persisted job details are read-only after creation; application status, applied date, and notes remain editable.
- Duplicate matching results in `409 DUPLICATE_APPLICATION` rather than a soft warning.

### Not Delivered

- analytics or application statistics
- browser extension
- persisted parsing-progress workflow
- application status history
- resume or cover-letter generation
- interview tooling
- email status detection
- frontend unit/component or end-to-end automated tests
- managed AWS secret storage
- staging environment

## Production Result

OfferBuddy is running at [offerbuddy.io](https://offerbuddy.io) on the single-host Sprint 1 architecture.

The production verification covered HTTPS/Nginx routing, frontend/backend health, Google authentication configuration, PostgreSQL persistence, container restart, EC2 reboot recovery, backup restoration, logs, and SHA-based redeployment capability.

## Acceptance Criteria Result

| Area | Result | Notes |
| --- | --- | --- |
| Google sign-in | Met | Production OIDC flow implemented |
| Create/view/update/delete applications | Met | Includes detail and current-status tracking |
| Search/filter/pagination | Met | Implemented through user-scoped API queries |
| PostgreSQL persistence | Met | Flyway V1/V2 and persistent production volume |
| Frontend/backend integration | Met | Running through same-origin Nginx deployment |
| Automated backend testing | Met | Multiple risk-focused test layers |
| Frontend automated testing | Partially met | Lint/build present; unit/component tests absent |
| Production deployment | Met | HTTPS EC2 deployment operational |
| Documentation updated | In progress at review time | Completed through Sprint 1 documentation groups |
| GitHub release/tag | Pending | Intentionally follows documentation and sprint closure |
| Replace daily spreadsheet workflow | Product validation required | The delivered product supports the workflow; repository evidence cannot prove the user's long-term habit change |

## Known Limitations

- Server-side page retrieval cannot access every job site reliably.
- AI parsing is synchronous and can be slow or incomplete.
- Manual application entry still contains capture friction.
- There is no frontend automated test suite.
- Production uses one EC2 host with no automatic failover.
- Backups are local to the host unless copied separately.
- Production secrets use an EC2-local `.env`.
- Observability is limited to health endpoints and local service logs.
- Application and database rollback remain separate concerns.

## Technical Debt

The active debt is maintained in the [Sprint 1 Technical Debt Register](sprint-1-technical-debt.md). Highest-value follow-ups include frontend automated tests, request/correlation IDs, secrets and backup hardening, staging/integration environment decisions, observability, job identity rules, and managing `ApplicationService` growth.

Redis is intentionally retained but is not used by Sprint 1 application logic. Authentication/security hardening must be revisited before Redis becomes active.

## Final Sprint Result

Sprint 1 delivered the intended production job application tracker and the supporting engineering/operations foundation. The core product and production goals were met. Final release tagging remains a separate closure action after documentation, Review, and Retrospective approval.
