# Deployment Strategy

## Purpose

This document defines how OfferBuddy is run and verified across local development, continuous integration, and future production environments.

The current implementation provides a reproducible local infrastructure environment and an automated CI pipeline. Automated production deployment has not yet been implemented.

## Environment Overview

| Environment | Current Approach | Status |
|---|---|---|
| Local Development | Applications run locally with infrastructure provided by Docker Compose | Available |
| Continuous Integration | GitHub Actions verifies backend and frontend changes | Available |
| Production | Provider and deployment workflow not yet selected | Planned |

## Local Development with Docker

Docker Compose is the standard mechanism for running OfferBuddy infrastructure during local development.

The current Compose environment includes:

- PostgreSQL 17
- Redis 8
- Named volumes for persistent local data
- A dedicated Docker network
- Environment-based service credentials and host ports

The React frontend and Spring Boot backend may run directly from the developer environment. This preserves fast feedback and IDE debugging while keeping infrastructure setup reproducible.

PostgreSQL is a confirmed application dependency. Redis is present in the local Compose environment but is not currently used by the application; its continued inclusion must be justified before it is treated as part of the runtime architecture.

The local environment must follow these principles:

- Secrets and developer-specific values remain outside source control.
- An example environment file documents required variables without containing real credentials.
- Infrastructure versions are explicit rather than floating implicitly.
- Persistent data is removed only through an intentional destructive operation.
- Local infrastructure should remain smaller than, but operationally representative of, production.

The rationale for Docker Compose is recorded in `decisions/ADR-005-docker-compose.md`.

## Continuous Integration with GitHub Actions

GitHub Actions provides the current continuous integration pipeline.

The workflow runs for:

- Pushes to `main`
- Pushes to `feature/**` branches
- Pull requests targeting `main`

### Backend Verification

The backend job:

- Uses Java 21
- Starts a PostgreSQL 17 service container
- Configures database access through environment variables
- Grants Linux execution permission to the Maven Wrapper
- Runs Maven `clean verify`
- Allows Flyway migrations and backend tests to run against PostgreSQL

### Frontend Verification

The frontend job:

- Uses Node.js 22
- Installs locked dependencies with `npm ci`
- Runs ESLint
- Produces the frontend build

Frontend automated tests are not currently configured. They must not be represented as an active CI gate until the frontend testing foundation is implemented.

### Merge Gate

The intended repository policy is that all required CI jobs pass before a pull request is merged into `main`.

Branch protection is a planned repository-standardisation action and must be configured before this policy can be considered technically enforced.

## Database Migration During Deployment

Flyway is the authoritative mechanism for database schema evolution.

Each environment applies the same version-controlled migration history. Hibernate automatic schema updates are not used as the production migration mechanism.

Before a production deployment process is introduced, the project must define:

- When migrations run
- Which deployment identity applies migrations
- How migration failure blocks application rollout
- How backward compatibility is maintained during deployment
- How destructive or long-running migrations are reviewed
- How database recovery is performed

The migration decision is recorded in `decisions/ADR-006-flyway.md`.

## Production Direction

The initial production architecture is expected to use:

- Static hosting for the React frontend
- A containerised Spring Boot backend
- A managed PostgreSQL database
- HTTPS for all public traffic
- Environment-based configuration and managed secrets
- Platform health checks and application logs

The hosting provider has not been selected. Kubernetes is not required for the MVP.

Redis will not be included in production unless a confirmed caching, session, rate-limiting, or other runtime requirement justifies it.

## Continuous Delivery

GitHub Actions currently provides CI only. It does not deploy OfferBuddy automatically.

Future continuous delivery should be introduced only after the production provider and release process are selected. A deployment workflow should:

1. Run all required CI checks.
2. Build immutable frontend and backend artifacts.
3. Identify the source commit and release version.
4. Apply the approved database migration strategy.
5. Deploy to the target environment.
6. Verify application health.
7. Record the deployment result.

Production deployment should require an explicit environment approval until the release process is proven reliable.

## Release and Rollback Principles

- Releases must be traceable to a Git commit and version tag.
- The `main` branch should remain releasable.
- Deployment configuration must be version controlled.
- Secrets must never be committed to the repository.
- Failed health verification must stop or reverse the rollout where supported.
- Application rollback must consider database compatibility.
- Database recovery must rely on tested backups or forward corrective migrations, not edited migration history.

## Open Decisions

- Production hosting provider
- Frontend hosting provider
- Managed PostgreSQL provider
- Secret management solution
- Artifact and container registry
- Production deployment trigger
- Environment approval policy
- Database backup and recovery approach
- Health verification and rollback mechanism
- Whether Redis remains in local development

## Current Status

**Status:** Local development and CI established; production deployment planned

**Date:** 5 August 2026
