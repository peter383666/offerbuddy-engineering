# Deployment Strategy

## Purpose

This document describes how the deployed Sprint 1 system moves from source control to production. Operational commands and incident checks are in the [Production Runbook](production-runbook.md); database procedures are in [PostgreSQL Backup and Restore](postgresql-backup-and-restore.md).

## Delivery Flow

```text
feature/*
  -> pull request to release
  -> CI and integration verification
  -> merge release to main
  -> final immutable tag
  -> deploy explicit verified SHA/version
```

The Sprint 1 tag is created after documentation and sprint closure.

## Branch Responsibilities

| Branch/Marker | Responsibility |
| --- | --- |
| `feature/*` | Isolated development and reviewable change |
| `release` | Integration branch and production candidate under verification |
| `main` | Verified stable baseline |
| tag | Immutable release marker |
| deployed SHA | Exact frontend/backend artifact selected for production |

Code on `release` is not automatically production-ready. A passing CI run is not the same as release integration or production verification.

## Continuous Integration

Frontend and backend use independent, path-filtered workflows.

### Backend CI

Triggers on relevant pull requests and pushes to `main` or `release`, plus manual dispatch.

```text
Java 21 setup
  -> Maven clean verify
  -> Docker image build
  -> push full-SHA and short-SHA tags on main/release push
```

Pull-request and manual CI runs validate the Docker build without publishing it. Published images use GHCR and include the commit revision.

### Frontend CI

Triggers on relevant pull requests and pushes to `main` or `release`, plus manual dispatch.

```text
Node 22
  -> npm ci
  -> ESLint
  -> TypeScript/Vite production build
  -> upload frontend-dist-<full-sha> on main/release push
```

The frontend has no automated unit/component test stage in Sprint 1.

## Continuous Delivery

Deployment is manually dispatched and separate from CI.

### Backend Deployment

The backend workflow:

1. resolves an explicit full SHA, or the latest successful backend CI SHA on an explicitly selected `release`/`main` branch
2. verifies that the SHA-tagged GHCR image exists
3. copies the production Compose file and deployment script to EC2
4. connects over SSH
5. pulls the existing image
6. replaces only the backend container
7. waits for `/actuator/health` to report `UP`

The EC2 host does not compile or build the backend.

### Frontend Deployment

The frontend workflow:

1. resolves an explicit full SHA, or the latest successful frontend CI SHA on an explicitly selected branch
2. downloads the matching immutable CI artifact
3. uploads it to an EC2 staging directory
4. verifies that `index.html` exists
5. replaces the Nginx document-root contents
6. validates and reloads Nginx
7. verifies that `https://offerbuddy.io` is reachable

The EC2 host does not run `npm build`.

### Why Explicit SHA Is Preferred

A mutable name such as `latest` does not uniquely identify code and can select a different artifact after another successful build. Production should normally deploy the full SHA that passed the intended verification.

The workflow's “latest” option is branch-scoped operator convenience, not the release identity.

## Production Architecture

```text
Internet
  -> HTTPS / Nginx on AWS EC2
       |-> React static files
       |-> /api and OAuth proxy to 127.0.0.1:8080
            -> Spring Boot backend container
                 -> PostgreSQL container + named volume
            -> Redis container (reserved; no Sprint 1 application use)
```

Public host ports are limited to SSH and HTTP/HTTPS. PostgreSQL and Redis are not published. The backend is bound only to host loopback.

Nginx preserves the `/api` prefix, forwards proxy headers, redirects HTTP to HTTPS, and redirects `www` to the canonical apex host.

## Production Configuration and Secrets

Production uses `SPRING_PROFILES_ACTIVE=prod` and an EC2-local `/opt/offerbuddy/.env`. The file is not committed.

### Non-sensitive configuration

Examples:

- `POSTGRES_DB`
- `POSTGRES_USER`
- `FRONTEND_BASE_URL`
- `SPRING_PROFILES_ACTIVE`
- backend image repository and SHA tag
- Nginx hostname
- frontend backend origin
- Google client ID, while still requiring correct environment-specific configuration

### Secrets

Examples:

- PostgreSQL/DB password
- Google client secret
- Gemini API key
- EC2 SSH private key
- GHCR read token or other access token

GitHub deployment credentials are stored as GitHub Environment secrets. The backend production verifier rejects missing required database/Google settings and known local placeholder values.

AWS SSM Parameter Store, AWS Secrets Manager, and IAM-role-based secret retrieval are future hardening options. They are not implemented.

## Database Migration

Flyway runs the versioned migrations when the backend starts. Hibernate does not update the schema.

A deployment must consider migration compatibility before replacing the application. Redeploying an earlier application image does not undo a database migration. If a migration is not backward compatible, application rollback may require a separately planned database recovery or forward fix.

## Health and Verification

Current layers are:

- PostgreSQL Compose health check using `pg_isready`
- backend dependency on healthy PostgreSQL
- backend Docker health check against loopback Actuator
- deploy-script polling for JSON health status `UP`
- frontend deployment check against the production homepage
- manual authentication and application smoke testing
- log inspection after deployment

Health confirms availability of the checked endpoint; it does not prove every external integration or user journey.

## Rollback

```text
identify known-good SHA
  -> redeploy its immutable backend image and/or frontend artifact
  -> verify backend health
  -> verify HTTPS homepage
  -> run authentication/application smoke test
  -> inspect logs
```

Frontend and backend can be rolled back independently when their contracts remain compatible.

Application rollback changes deployed code. Database rollback concerns schema/data and is handled separately, normally through recovery or a new forward migration rather than editing deployed Flyway files.

## Current Status

| Area | Status |
| --- | --- |
| Backend CI and SHA image publication | Implemented |
| Frontend CI and SHA artifact publication | Implemented |
| Manual backend deployment | Implemented |
| Manual frontend deployment | Implemented |
| AWS EC2 / Nginx / HTTPS | Implemented |
| PostgreSQL backup and restore verification | Implemented and exercised |
| Docker restart and EC2 reboot recovery | Exercised |
| Automatic tag/GitHub Release workflow | Not implemented |
| Managed secret storage | Not implemented |
| Staging environment | Not implemented |
