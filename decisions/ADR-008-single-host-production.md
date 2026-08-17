# ADR-008: Use Single-Host EC2 Deployment with Immutable SHA Artifacts

## Status

Accepted and implemented.

## Date

17 August 2026.

## Context

Sprint 1 needed a real HTTPS production environment with low cost, understandable operations, repeatable deployment, and a practical rollback path.

The system consists of one static React frontend, one Spring Boot backend, one PostgreSQL database, and reserved Redis infrastructure. Its scale does not justify Kubernetes or independently managed application services.

Deploying mutable `latest` artifacts or building directly on the server would make version identity and rollback less reliable.

## Decision

Run the Sprint 1 production system on one AWS EC2 host:

- host Nginx terminates TLS, redirects to the canonical host, serves the React build, and proxies API/OAuth traffic
- Docker Compose runs the Spring Boot backend, PostgreSQL, and reserved Redis containers
- PostgreSQL uses a named persistent volume
- the backend binds to host loopback rather than the public interface
- PostgreSQL and Redis remain private to the Compose network

Use build-once, immutable delivery:

- backend CI publishes a GHCR image tagged with the commit SHA
- frontend CI publishes a `dist` artifact named for the commit SHA
- separate manual deployment workflows deploy an explicit verified SHA
- EC2 pulls/copies the artifact and does not compile production code
- health checks and smoke tests verify deployment
- rollback redeploys a known-good previous SHA

The delivery path is feature work to `release` integration verification, promotion to `main`, immutable release tag, and explicit production version. A branch-scoped “latest successful CI” option exists for operator convenience, but an explicit SHA is the production identity.

## Consequences

### Positive

- Low infrastructure cost and few operational components.
- Nginx provides one HTTPS origin for SPA, API, OAuth callback, cookies, and CSRF.
- Immutable artifacts make deployed code identity and rollback clear.
- Frontend and backend can be deployed independently.
- Docker Compose makes restart and recovery procedures explicit.
- Production does not require Maven or Node builds.

### Negative

- The EC2 instance is a single host and availability boundary.
- Backend, PostgreSQL, Nginx, and static assets share that host.
- Deployments and rollback require operator action.
- Application rollback does not automatically reverse database migrations.
- The EC2-local `.env` and on-host database/backups require further hardening.
- Horizontal scaling would require changes to session, database, and routing architecture.

## Alternatives Considered

### Platform-as-a-service or managed containers

Not selected for Sprint 1 because the single host was sufficient and provided direct operational experience at lower complexity.

### Kubernetes

Rejected because orchestration, cluster operations, and cost are disproportionate to one backend and one host.

### Build on the EC2 host

Rejected because it weakens build provenance, requires production toolchains, increases deployment time, and can produce artifacts different from CI.

### Mutable `latest` deployment

Rejected as the normal production identity because it is ambiguous and makes deterministic rollback harder.

### Managed database

Deferred. PostgreSQL remains on the EC2 host for Sprint 1; application and database rollback remain separate concerns.

## Implementation Outcome

The application is live at `https://offerbuddy.io`. Nginx/HTTPS, frontend and backend deployment, health checks, Docker restart, EC2 reboot recovery, PostgreSQL persistence, backup/restore verification, and known-good-SHA rollback capability have been verified.

See [Container Design](../architecture/container-design.md) and [Deployment Strategy](../operations/deployment-strategy.md).
