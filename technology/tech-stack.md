# Technology Stack

## Purpose

This document records the technology used by the current OfferBuddy system after Sprint 2. It replaces planning-era options with the implementation that exists.

## Implemented Stack

| Area | Technology |
| --- | --- |
| Frontend | React 19, TypeScript, Vite, React Router, Vitest |
| Browser Extension | Chrome Manifest V3, TypeScript, Vite, Vitest |
| Backend | Java 21, Spring Boot, Spring Web MVC |
| Security | Spring Security, Google OAuth 2.0/OpenID Connect, server-side servlet session, Extension credentials |
| Data | PostgreSQL 17, Spring Data JPA, Flyway |
| AI | Google Gemini, Google Gen AI SDK, jsoup, Java HTTP client |
| Backend build | Maven and Maven Wrapper |
| Frontend / Extension build | npm and package lock |
| Testing | JUnit 5, Mockito, Spring Boot Test, Testcontainers, Vitest |
| API documentation | springdoc OpenAPI/Swagger UI; disabled in production |
| Local infrastructure | Docker Compose with PostgreSQL and reserved Redis |
| Production | AWS EC2, host Nginx, Docker Compose, HTTPS/Certbot |
| CI/CD | GitHub Actions, GHCR, immutable SHA-tagged images and artifacts |

## Frontend

The browser application is a React single-page application written in TypeScript and built by Vite. Routes include login, home, applications, application detail/edit, new application, analytics, and extension connect.

Production runs the compiled static files under Nginx. There is no production Node.js application server.

Frontend CI currently performs:

- deterministic dependency installation with `npm ci`
- ESLint
- TypeScript compilation
- a Vite production build
- upload of the built `dist` directory on `main` and `release` pushes

Frontend Vitest component/page tests exist locally, via precheck, and as a Frontend CI gate on the application CI follow-up branch (`review/s2-ci-followups`) until merged to `main`.

## Browser Extension

Chrome Manifest V3 Extension packaged with Vite. Local verification uses `npm test` and `npm run verify`. Application repo workflows: **Extension Publish** (Chrome Web Store upload/submit) on `main`; **Extension CI** (`npm run verify`) on the CI follow-up branch pending merge.

## Backend

The backend is a Java 21 Spring Boot modular monolith. It exposes versioned REST APIs and contains application, job, job-parsing, job-intelligence, analytics, events, extension, authentication, user, OpenAPI, configuration, and shared-error responsibilities.

Spring Data JPA provides persistence access. Transaction boundaries are managed in services. Controllers and DTOs define the external contract; persistence entities are not returned directly.

## Authentication and Security

Google is the Web identity provider. Spring Security handles OAuth 2.0/OpenID Connect, maps the Google subject to a local user, and establishes a server-side session.

The browser receives a `JSESSIONID` cookie. In production it is secure, HTTP-only, and `SameSite=Lax`. Spring Security CSRF protection uses the CSRF cookie/header convention for state-changing Web requests.

The Browser Extension authenticates with a separately paired, revocable Extension credential. Ownership remains server-derived.

## Data and Async Processing

PostgreSQL is the system of record, evolved by Flyway through Sprint 2 migrations V3–V9 (history, extension auth, business events, intelligence, analytics).

Redis remains reserved infrastructure and is not used by application logic for Sprint 2 features.

Business Events provide brokerless asynchronous delivery inside the monolith for Job Intelligence and Analytics.

## Related

- [System Context](../architecture/system-context.md)
- [Testing Strategy](../quality/testing-strategy.md)
- [Deployment Strategy](../operations/deployment-strategy.md)
- [Sprint 2 Operations](../operations/s2/README.md)
