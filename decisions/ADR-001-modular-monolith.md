# ADR-001: Use a Modular Monolith for the MVP

## Status

Accepted and implemented.

## Date

3 August 2026; implementation outcome updated after Sprint 1.

## Context

OfferBuddy needed one backend for authentication, job parsing, application tracking, and persistence. One engineer was delivering the first production increment, and the product had no independent scaling, deployment, or team-boundary requirement that justified distributed services.

A completely unstructured monolith would make future change harder, while microservices would introduce deployment, networking, consistency, and observability costs without solving a Sprint 1 problem.

## Decision

Implement the backend as one Spring Boot deployable with domain-oriented package boundaries.

The implemented top-level areas are:

- `auth` — Google OIDC, session security, CSRF, and logout
- `user` — local OfferBuddy users
- `job` — persisted job information and find-or-create behaviour
- `jobparsing` — page acquisition, AI extraction, and validation
- `application` — application creation, query, update, deletion, and ownership
- `openapi` — generated API documentation configuration
- `config` — production configuration verification
- `shared.error` — cross-cutting API error representation

The application and job packages are separate business boundaries within one process and database. Sprint 1 does not implement separate Company, Dashboard, Analytics, status-history, or integration modules.

## Boundaries

- Controllers expose DTOs rather than persistence entities.
- Application services own transactions and business rules.
- The application package depends on the job package for job persistence/retrieval.
- Authentication supplies the current user; clients do not submit ownership identifiers.
- Provider-specific AI code remains behind application-owned interfaces.
- Cross-package sharing is kept deliberate rather than creating a general utility layer.

## Consequences

### Positive

- One artifact and process are straightforward to test and deploy.
- Transactions can cover job and application creation without distributed coordination.
- Package boundaries still make responsibilities visible.
- Operational cost remains proportional to the product scale.

### Negative

- All backend capabilities release and scale together.
- Package boundaries rely on engineering discipline rather than process isolation.
- `ApplicationService` may grow as later use cases are added.
- A single backend failure affects all API capabilities.

## Alternatives Considered

### Traditional layered monolith

Rejected because grouping all controllers, services, and repositories by technical layer would make domain ownership less clear.

### Microservices

Rejected because Sprint 1 had no independent scaling or organisational requirement, and the additional operational complexity would delay delivery.

### Serverless functions

Rejected because the application uses cohesive Spring Security/session behaviour, transactional relational data, and a conventional API runtime.

## Implementation Outcome

The decision supported Sprint 1 delivery and the single-container production backend. No evidence from Sprint 1 requires decomposition into independently deployed services.

Future extraction should be considered only when a concrete ownership, scaling, reliability, or release-independence requirement appears.
