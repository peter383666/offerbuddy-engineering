# OfferBuddy Engineering

OfferBuddy is a job application tracking web application. It helps a signed-in user capture a job opportunity, review AI-extracted job information, create an application, and track that application through its current status.

The project is a real production application and a portfolio engineering project. It demonstrates delivery and operation of a focused product; it is not presented as a mature SaaS platform.

Application source code is maintained in the separate [OfferBuddy source repository](https://github.com/peter383666/offerbuddy).

## Current Status

Sprint 1 is complete and deployed at [offerbuddy.io](https://offerbuddy.io).

The final Sprint 1 release tag is intentionally created after the documentation and sprint-closing work is complete.

## Sprint 1 Capabilities

- Google sign-in using OAuth 2.0/OpenID Connect
- Server-managed authenticated sessions and user-level data isolation
- Manual application creation
- AI-assisted extraction of job details from a submitted URL
- Review and correction of extracted data before saving
- Application list with search, status filtering, sorting, and pagination
- Application detail, edit, status update, and delete flows
- Home page with application capture and recent applications
- PostgreSQL persistence managed by Flyway migrations
- HTTPS production deployment with independent frontend and backend CI/CD

Analytics, browser extension capture, resume generation, cover-letter generation, and interview tooling are not Sprint 1 capabilities.

## Technology Stack

| Area | Implemented Technology |
| --- | --- |
| Frontend | React, TypeScript, Vite |
| Backend | Java 21, Spring Boot, Spring Web, Spring Security, Spring Data JPA |
| Database | PostgreSQL 17, Flyway |
| Authentication | Google OAuth 2.0/OpenID Connect, Spring Security server session |
| AI integration | Google Gemini behind application-owned parsing interfaces |
| Infrastructure | AWS EC2, Docker Compose, host Nginx, HTTPS/Certbot |
| CI/CD | GitHub Actions, GHCR backend images, immutable frontend build artifacts |
| Backend testing | JUnit 5, Spring Boot Test, Mockito, Testcontainers |
| Frontend validation | ESLint, TypeScript compilation, production build |

Redis is present in the local and production Compose definitions but is not used by Sprint 1 application logic, session storage, or caching. It is retained for possible later session/cache-related needs.

## Architecture Overview

```text
Browser
  |
  | HTTPS
  v
Host Nginx on AWS EC2
  |-- serves the React build
  |-- proxies /api and OAuth routes
  v
Spring Boot API container
  |-- Google OIDC
  |-- Gemini job parsing
  v
PostgreSQL container and persistent volume
```

Nginx, the backend, PostgreSQL, and the reserved Redis container run on one EC2 host. Docker Compose manages the application containers; Nginx runs on the host and terminates TLS.

See [System Context](architecture/system-context.md), [Container Design](architecture/container-design.md), and [Data Model](architecture/data-model.md) for the detailed view.

## Engineering Highlights

- Versioned REST API with an implementation-aligned OpenAPI contract
- Modular-monolith backend with explicit application, job, parsing, authentication, and user boundaries
- Automated backend service, controller, security, persistence, OpenAPI, and integration testing
- Path-filtered frontend and backend CI workflows
- Build-once deployment of immutable SHA-identified images and frontend artifacts
- Manual production deployment of an explicit verified SHA
- PostgreSQL backup plus restore verification
- Docker restart and EC2 reboot/recovery verification
- Health checks, production smoke tests, and known-good-SHA rollback capability

## Documentation Map

| Area | Start Here |
| --- | --- |
| Product | [Product Vision](product/product-vision.md), [Sprint 1 Scope](product/mvp-scope.md), [Sprint 1 User Stories](product/user-stories.md), [Sprint 2 Requirements](product/sprint-2-requirements.md) |
| Technology | [Technology Stack](technology/tech-stack.md) |
| Architecture | [Sprint 2 Architecture Design](architecture/sprint-2-architecture-design.md), [Sprint 1 System Context](architecture/system-context.md), [Sprint 1 Container Design](architecture/container-design.md), [Data Model](architecture/data-model.md), [API Design](architecture/api-design.md) |
| Design | [Sprint 2 Design Index](design/sprint-2/README.md), [Extension Design](design/sprint-2/extension-design.md) |
| Decisions | [ADR Index](decisions/README.md) |
| Quality | [Testing Strategy](quality/testing-strategy.md), [Non-Functional Requirements](quality/non-functional-requirements.md), [Definition of Done](quality/definition-of-done.md) |
| Operations | [Development Workflow](operations/development-workflow.md), [Deployment Strategy](operations/deployment-strategy.md), [Production Runbook](operations/production-runbook.md), [PostgreSQL Backup and Restore](operations/postgresql-backup-and-restore.md) |
| Delivery | [Roadmap](delivery/roadmap.md), [Sprint 1 Plan](delivery/sprint-1.md), [Sprint 1 Review](delivery/sprint-1-review.md), [Sprint 1 Retrospective](delivery/sprint-1-retrospective.md), [Sprint 1 Technical Debt](delivery/sprint-1-technical-debt.md) |

## Roadmap

Sprint 2 Phase 1 Requirement Analysis and Phase 2 Architecture Design are complete and approved; Sprint 2 functionality is not yet represented as delivered. The Browser Extension is the primary feature, initially supporting SEEK and Indeed, while basic Application Analytics is secondary. The approved architecture retains the Spring Boot modular monolith and PostgreSQL, separates page-fact capture from backend Job Intelligence, and uses lightweight Business Events so downstream AI and Analytics do not block core Application operations.

See the authoritative [Sprint 2 Requirements](product/sprint-2-requirements.md), [Sprint 2 Architecture Design](architecture/sprint-2-architecture-design.md), [Delivery Roadmap](delivery/roadmap.md), and [Product Backlog](delivery/product-backlog.md).

## Release History

| Version | Description |
| --- | --- |
| `engineering-v0.1` | Product and initial MVP architecture documentation established |
| Sprint 1 tag pending | Created only after documentation and sprint closure |

## License

This repository contains engineering documentation for OfferBuddy. Application source code is maintained separately.
