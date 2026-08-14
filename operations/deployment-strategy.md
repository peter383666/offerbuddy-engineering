# Deployment Strategy

## Purpose

This document defines how OfferBuddy is run and verified across local development, continuous integration, continuous delivery, and production.

Sprint 1 production targets a single low-cost EC2 host. Frontend and backend keep **independent CI/CD lifecycles** inside one monorepo.

## Environment Overview

| Environment | Approach | Status |
| --- | --- | --- |
| Local Development | Apps on the host; Postgres (+ Redis) via Docker Compose | Available |
| Continuous Integration | Path-filtered GitHub Actions (`backend-ci`, `frontend-ci`) | Available |
| Continuous Delivery | Independent frontend/backend deploy workflows | Planned |
| Production | Single AWS EC2 + host Nginx + Docker Compose | Planned |

## Branch and release model

| Branch | Meaning |
| --- | --- |
| `feature/*`, `chore/*`, `review/*` | Development work |
| `release` | Production candidate — deploy and smoke-test from here |
| `main` | Production-verified stable line |

Flow:

```text
feature/*  →  PR (+ CI)  →  release  →  deploy exact commit  →  smoke test  →  merge to main  →  tag
```

Rules:

- **Branches** manage code flow.
- **Commit SHA / tag** define the artifact that was actually deployed.
- Frontend and backend may deploy different commits independently.
- Prefer `workflow_dispatch` for production deploys so the operator selects branch / tag / SHA deliberately.

Do not treat “latest `release` tip” as the only deploy identity after further pushes have landed.

## Local Development with Docker

Docker Compose remains the standard mechanism for local infrastructure:

- PostgreSQL 17
- Redis 8 (present locally; not required by the Sprint 1 application runtime yet)
- Named volumes and a private Docker network
- Credentials via environment files that are not committed

React and Spring Boot normally run on the host for fast feedback. See `decisions/ADR-005-docker-compose.md`.

## Continuous Integration

CI **verifies** code. It does **not** deploy.

Workflows live in the source repository:

| Workflow | Paths | What it runs |
| --- | --- | --- |
| `backend-ci.yml` | `backend/**` | Java 21, `./mvnw clean verify` (Testcontainers Postgres) |
| `frontend-ci.yml` | `frontend/**` | Node 22, `npm ci`, lint, production build |

Triggers:

- Push to `main`, `release`, `feature/**`, `review/**`, `chore/**` (path-filtered)
- Pull requests targeting `main` or `release` (path-filtered)
- `workflow_dispatch` for manual re-runs

Principles:

- Changing only frontend must not run Maven.
- Changing only backend must not run npm.
- Frontend unit tests are not yet a CI gate (see Sprint 1 technical debt).
- Required status checks / branch protection should account for path-filtered workflows (a job that does not run must not block unrelated PRs incorrectly).

## Production architecture (Sprint 1 MVP)

Selected intentionally for cost control and portfolio clarity. **Not** using RDS, ElastiCache, ECS/EKS, ALB, or CloudFront in the first release.

```text
Internet
   │
   ▼
offerbuddy.io  (HTTPS / Let's Encrypt)
   │
   ▼
AWS Elastic IP
   │
   ▼
EC2
├── Host Nginx
│     ├── /        → React static files (dist/)
│     └── /api/**  → Spring Boot container :8080
└── Docker Compose (private network only)
      ├── backend
      ├── postgres   (volume-backed)
      └── redis      (reserved; unused by app until justified)
```

Public ports only: **22**, **80**, **443**. Postgres and Redis must not be reachable from the internet.

Same-origin benefits:

- No browser CORS for the SPA API calls
- Simpler Google OAuth redirect (`https://offerbuddy.io/login/oauth2/code/google`)
- Session cookie + CSRF behaviour stays on one site

### Frontend runtime

Production frontend is **static files**, not a long-running Node process:

```text
npm ci → npm run build → dist/ → Nginx
```

EC2 does not need Node for serving production traffic.

### Backend runtime

```text
mvn package → Docker image → GHCR (planned) → EC2 docker pull → backend container
```

EC2 should not compile the backend with Maven in production.

### Configuration

| Concern | Local / default | Production (`SPRING_PROFILES_ACTIVE=prod`) |
| --- | --- | --- |
| Database | Local / Compose Postgres | Compose Postgres on EC2 private network |
| Swagger | Enabled when permitted | Disabled |
| Session cookie | Dev-friendly | `Secure` + HTTPS |
| Secrets | Local placeholders or env | Required env vars only — never commit |

Sensitive values (examples): `DB_PASSWORD`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_API_KEY`, `FRONTEND_BASE_URL`, `SPRING_PROFILES_ACTIVE`.

## Continuous Delivery (planned)

Separate deploy workflows:

| Workflow | Responsibility |
| --- | --- |
| `backend-deploy.yml` | Build/push backend image for a selected ref; update backend container on EC2 |
| `frontend-deploy.yml` | Build `dist` for a selected ref; replace Nginx static files on EC2 |

Principles:

1. **Build once, deploy the artifact** — CI/CD builds the image or `dist`; EC2 only pulls/copies runtime artifacts.
2. Frontend deploy must not restart Postgres/Redis/backend by default.
3. Backend deploy must not rebuild React by default.
4. Deploy from `release` (or an explicit SHA/tag), smoke-test, then merge to `main` and tag.
5. Independent rollback of frontend or backend versions.

Suggested delivery sequence remaining:

1. ~~Backend CI~~ / ~~Frontend CI~~
2. Production Docker layout (`backend` image + Compose)
3. Backend CD
4. Frontend CD
5. Host Nginx + TLS
6. EC2 + Elastic IP + DNS
7. Google OAuth production redirect URIs
8. Production smoke test
9. Backup (`pg_dump`) and basic monitoring

## Database Migration During Deployment

Flyway remains authoritative. Hibernate DDL auto-update is not used for production schema management.

Before automated CD ships, define:

- When migrations run (typically backend container start)
- How migration failure blocks rollout
- Backup/restore expectations (`pg_dump` first; S3 later)
- Compatibility rules for rollback

See `decisions/ADR-006-flyway.md`.

## Release and Rollback Principles

- Releases must be traceable to a Git commit and version tag.
- `main` represents production-verified code, not merely “merged but unproven”.
- Secrets must never be committed.
- Failed health checks stop the rollout.
- Application rollback must consider Flyway compatibility.
- Do not edit applied migration files in place.

## Open Decisions

- Exact EC2 instance size / region
- Whether Redis remains in production Compose while unused by the app
- GHCR image naming and retention
- SSH / deploy key vs OIDC-based deploy auth
- Automated vs manual `workflow_dispatch`-only production deploys
- Backup schedule and off-host storage (S3)
- Branch-protection required-check configuration for path-filtered CI

## Current Status

| Area | Status |
| --- | --- |
| Local Compose | Available |
| Split frontend/backend CI | Available |
| Production Docker / Nginx / EC2 | Planned |
| Frontend / backend CD | Planned |

**Updated:** 14 August 2026
