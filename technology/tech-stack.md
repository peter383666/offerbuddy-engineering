# Technology Stack

## Purpose

This document records the technology used by the deployed Sprint 1 system. It replaces planning-era options with the implementation that exists.

## Implemented Stack

| Area | Technology |
| --- | --- |
| Frontend | React 19, TypeScript, Vite, React Router |
| Backend | Java 21, Spring Boot, Spring Web MVC |
| Security | Spring Security, Google OAuth 2.0/OpenID Connect, server-side servlet session |
| Data | PostgreSQL 17, Spring Data JPA, Flyway |
| AI parsing | Google Gemini, Google Gen AI SDK, jsoup, Java HTTP client |
| Backend build | Maven and Maven Wrapper |
| Frontend build | npm and package lock |
| Testing | JUnit 5, Mockito, Spring Boot Test, Testcontainers |
| API documentation | springdoc OpenAPI/Swagger UI; disabled in production |
| Local infrastructure | Docker Compose with PostgreSQL and reserved Redis |
| Production | AWS EC2, host Nginx, Docker Compose, HTTPS/Certbot |
| CI/CD | GitHub Actions, GHCR, immutable SHA-tagged images and artifacts |

## Frontend

The browser application is a React single-page application written in TypeScript and built by Vite. It provides login, home, new application, applications list, application detail, and edit routes.

Production runs the compiled static files under Nginx. There is no production Node.js application server.

Frontend CI currently performs:

- deterministic dependency installation with `npm ci`
- ESLint
- TypeScript compilation
- a Vite production build
- upload of the built `dist` directory on `main` and `release` pushes

Sprint 1 does not include Vitest, React Testing Library, or another frontend unit/component test suite.

## Backend

The backend is a Java 21 Spring Boot modular monolith. It exposes versioned REST APIs and contains application, job, job-parsing, authentication, user, OpenAPI, configuration, and shared-error packages.

Spring Data JPA provides persistence access. Transaction boundaries are managed in services. Controllers and DTOs define the external contract; persistence entities are not returned directly.

## Authentication and Security

Google is the only Sprint 1 identity provider. Spring Security handles OAuth 2.0/OpenID Connect, maps the Google subject to a local user, and establishes a server-side session.

The browser receives a `JSESSIONID` cookie. In production it is secure, HTTP-only, and `SameSite=Lax`. Spring Security CSRF protection uses the CSRF cookie/header convention for state-changing requests. API requests without a session receive JSON `401` responses rather than an OAuth redirect.

See [ADR-002](../decisions/ADR-002-google-authentication.md).

## Database and Migrations

PostgreSQL 17 is the primary database. Flyway applies the versioned schema before the application begins normal operation. Hibernate does not create or update the production schema.

Sprint 1 uses:

- `users`
- `jobs`
- `job_applications`

See [Data Model](../architecture/data-model.md) and [ADR-007](../decisions/ADR-007-postgresql.md).

## AI-Assisted Job Parsing

The backend retrieves available job-page content, sends relevant text to Google Gemini, validates the structured response, and returns a draft to the frontend.

Application-owned interfaces separate orchestration and parsing from the provider client. Gemini is the current provider, not a dependency of the application domain.

The flow is synchronous and subject to job-site accessibility and provider latency. AI output is not persisted until the user reviews the form and creates an application.

## Infrastructure

### Local

The frontend and backend normally run on the developer host. Docker Compose runs PostgreSQL and Redis.

Redis is not an application dependency in Sprint 1. It is retained as reserved infrastructure for possible future session or cache capabilities.

### Production

One AWS EC2 host runs:

- host Nginx for TLS termination, SPA static files, and reverse proxying
- the Spring Boot backend container
- PostgreSQL with a persistent Docker volume
- a reserved Redis container

Docker Compose manages the containers. PostgreSQL, Redis, and the backend are not directly internet-facing; the backend binds to host loopback for Nginx.

See [ADR-008](../decisions/ADR-008-single-host-production.md).

## CI/CD

Backend CI runs Maven `clean verify`, builds the Docker image, and publishes full- and short-SHA tags on `main` or `release` pushes.

Frontend CI lints and builds the application, then publishes an immutable SHA-named build artifact on `main` or `release` pushes.

Backend and frontend deployment are separate, manually dispatched workflows. Both can select an explicit full commit SHA. Production should use a verified immutable SHA rather than an unqualified `latest` artifact.

## Intentionally Deferred

The following are not implemented Sprint 1 technologies:

- Kubernetes
- microservices
- Kafka or RabbitMQ
- Elasticsearch
- distributed tracing infrastructure
- managed secret storage
- frontend automated test framework

They should be introduced only in response to a concrete requirement.
