# Sprint 1 Non-Functional Requirements

## Purpose

These requirements record the quality characteristics implemented or explicitly accepted at the end of Sprint 1. They are not claims of enterprise-scale availability.

## Security

Implemented:

- production traffic uses HTTPS through Nginx
- Google OAuth 2.0/OpenID Connect is the only authentication method
- Spring Security maintains a server-side session
- production `JSESSIONID` is HTTP-only, secure, and `SameSite=Lax`
- CSRF protection remains enabled for state-changing requests
- APIs derive ownership from the authenticated principal
- unauthenticated API requests return JSON `401`
- PostgreSQL and Redis are not exposed publicly
- the backend is bound to EC2 loopback for Nginx
- production error responses suppress stack traces and internal exception details
- Swagger/OpenAPI endpoints are disabled in production
- production startup rejects missing or local-placeholder database/Google credentials

Known gaps:

- production secrets are stored in an EC2-local `.env`, not AWS SSM or Secrets Manager
- Redis has no authentication but is private and inactive
- the public Nginx configuration proxies limited-detail Actuator endpoints
- request/correlation IDs are not populated

## Reliability and Recovery

Implemented and verified during Sprint 1:

- Docker services use `restart: unless-stopped`
- PostgreSQL has a Compose health check
- backend startup waits for healthy PostgreSQL
- backend has a Docker health check
- Nginx serves the frontend and proxies the backend after host restart
- PostgreSQL data uses a named persistent volume
- Docker restart and EC2 reboot recovery were exercised
- database persistence and backend/Nginx recovery were checked after restart
- known-good immutable SHAs can be redeployed

The deployment uses one EC2 host. It has no multi-zone redundancy, automatic failover, or uptime SLA.

## Data Durability

- PostgreSQL is the system of record.
- Production data is stored in a Docker named volume.
- `backup-postgres.sh` creates a PostgreSQL custom-format dump on the EC2 host.
- `verify-postgres-backup.sh` restores the latest dump into a temporary database and checks that public tables exist.
- A backup is not considered fully verified until restoration has been tested.

Known gaps:

- backups remain on the same host unless copied manually
- backup scheduling and retention are not automated in the repository
- restore verification checks restored tables, not application-level row counts or business invariants
- database rollback is separate from application rollback

## Performance

- Application lists are server-paginated with a maximum page size.
- User/date and job-search-related indexes support current list/search behaviour.
- Backend HTTP and Gemini requests have bounded timeouts.
- Job-page input size is bounded before AI processing.

Known limitations:

- AI parsing is synchronous and may take up to the configured provider timeout
- page acquisition and AI latency are combined in the user request
- no load, soak, or formal response-time testing has been implemented
- the single EC2 host provides finite shared CPU, memory, and storage

## Maintainability

- backend code uses domain-oriented modular-monolith packages
- REST DTOs are separate from JPA entities
- Flyway migrations are version-controlled and immutable after deployment
- backend CI runs build and automated tests
- frontend CI runs lint, type compilation, and production build
- deployment artifacts are identified by commit SHA
- production and architecture documentation link to operational runbooks

Known gaps include frontend automated tests, growing application-service responsibilities, incomplete error OpenAPI annotations, and limited observability.

## Observability

Current controls:

- Spring Boot application logs at INFO
- Spring Security and SQL log levels are reduced in production
- Docker Compose exposes service logs to operators
- Nginx access/error logs and systemd service status are available on EC2
- Actuator health returns limited information
- deployment workflows perform health/homepage checks

Not implemented:

- centralised log aggregation
- alerting
- application performance monitoring
- distributed tracing
- request/correlation IDs
- product analytics

## Scalability

Sprint 1 is intentionally single-host and single-backend-instance. No claim is made for horizontal scalability.

Triggers for revisiting the architecture include:

- multiple backend instances requiring shared or affinity-based sessions
- database capacity or availability exceeding the EC2-host model
- recovery objectives requiring managed/off-host services
- measured traffic exceeding the current host
