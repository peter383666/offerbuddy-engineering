# Changelog

All notable changes to the OfferBuddy engineering project will be documented in this file.

## [Unreleased] — Sprint 1 closure

### Added

- Google OAuth 2.0/OpenID Connect authentication and server-managed sessions
- Application creation, list, search, filtering, sorting, pagination, detail, update, status, and deletion
- Home page and AI-assisted job parsing with manual entry fallback
- PostgreSQL production persistence and Flyway schema history
- AWS EC2, Nginx, HTTPS, and Docker Compose production deployment documentation
- Independent frontend/backend CI/CD with immutable SHA artifacts
- PostgreSQL backup and restore-verification procedures
- Production restart, recovery, logging, and rollback runbooks
- Sprint 1 Review, Retrospective, and reconciled technical-debt register
- ADRs for PostgreSQL and the single-host immutable-artifact deployment model

### Changed

- Reconciled product, architecture, API, testing, and operations documentation with the implemented Sprint 1 system
- Documented Redis as retained infrastructure that is inactive in Sprint 1 business logic
- Clarified `release`, `main`, tag, and explicit-SHA production responsibilities
- Separated delivered Sprint 1 scope from future Browser Extension, Analytics, resume, cover-letter, and interview capabilities

### Known Limitations

- No frontend unit/component or browser end-to-end test suite
- AI parsing remains synchronous and depends on server-side job-page acquisition
- Production uses one EC2 host and an EC2-local `.env`
- Backups are not automatically copied off-host
- Observability is limited to health endpoints and local service logs
- Final Sprint 1 release/tag is intentionally pending documentation approval and sprint closure

## [0.1.0] - 2026-08-05

### Added

- Established the initial monorepo project structure
- Initialised the React and TypeScript frontend
- Initialised the Java 21 and Spring Boot backend
- Added Docker Compose local infrastructure
- Added PostgreSQL and Redis
- Integrated Spring Data JPA with PostgreSQL
- Added Flyway database migrations
- Added GitHub Actions continuous integration
- Added product, architecture, quality, operations, and delivery documentation
- Added Sprint 0 review and retrospective records

### Changed

- Refined the Sprint 0 scope to separate future AI extraction testing from engineering foundation work

### Known Limitations

- Business functionality has not yet been implemented
- Backend and frontend automated test foundations remain planned work
- Deployment automation is not yet configured
