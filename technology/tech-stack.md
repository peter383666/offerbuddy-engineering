# Technology Stack

## Purpose

This document defines the initial technology stack for the OfferBuddy MVP.

The selected technologies support the following goals:

* Deliver the MVP quickly
* Keep the architecture simple
* Minimise infrastructure cost
* Support secure user authentication
* Maintain clear separation between frontend and backend
* Demonstrate production-relevant Java and React engineering skills
* Allow the system to evolve without introducing premature complexity

This document describes the current technical direction. Significant changes should be recorded through Architecture Decision Records.

---

# Technology Overview

| Area                     | Selected Technology                                     |
| ------------------------ | ------------------------------------------------------- |
| Frontend                 | React, TypeScript, Vite                                 |
| UI Components            | To be confirmed                                         |
| Backend                  | Java 21, Spring Boot                                    |
| API Style                | REST API                                                |
| Authentication           | Google OAuth 2.0 / OpenID Connect                       |
| Security                 | Spring Security                                         |
| Database                 | PostgreSQL                                              |
| Database Migration       | Flyway                                                  |
| Data Access              | Spring Data JPA                                         |
| Backend Build Tool       | Maven                                                   |
| Frontend Package Manager | npm                                                     |
| Local Development        | Docker Compose                                          |
| Containerisation         | Docker                                                  |
| Source Control           | Git and GitHub                                          |
| Continuous Integration   | GitHub Actions                                          |
| Testing                  | JUnit 5, Mockito, Testcontainers, React Testing Library |
| API Documentation        | OpenAPI / Swagger                                       |
| Deployment               | Docker-based cloud deployment                           |
| Monitoring               | Application logs initially; extended monitoring later   |
| AI Integration           | External large language model API with provider abstraction |

---

# Frontend

## Selected Technologies

* React
* TypeScript
* Vite

## Decision

OfferBuddy will use React with TypeScript as the frontend framework.

Vite will be used for frontend development, local startup, and production builds.

## Rationale

React was selected because:

* It has a large ecosystem and strong industry adoption.
* It is commonly used in Australian software engineering roles.
* It works well with a separate Spring Boot REST API.
* It supports reusable component-based development.
* It is suitable for a single-page application.
* It allows the frontend and backend to evolve independently.

TypeScript was selected because:

* It provides compile-time type checking.
* It improves maintainability as the application grows.
* It reduces common runtime errors.
* It provides better IDE support and code navigation.
* It is widely used in production React applications.

Vite was selected because:

* It provides a simple project structure.
* It offers fast local development startup.
* It supports modern React and TypeScript projects.
* It introduces less framework complexity than Next.js.
* OfferBuddy does not currently require server-side rendering.

## Alternatives Considered

### Next.js

Next.js was not selected for the MVP because:

* The main OfferBuddy application is an authenticated web application.
* Search engine optimisation is not a major requirement for authenticated pages.
* Server-side rendering is not required for the first release.
* It would introduce additional framework concepts that are not necessary for the MVP.

A separate public marketing website may be introduced later if SEO becomes important.

### Vue

Vue is a capable frontend framework, but React was selected because it better aligns with the intended portfolio and employment goals.

### Angular

Angular provides a complete application framework but introduces more structure and complexity than required for the MVP.

---

# Backend

## Selected Technologies

* Java 21
* Spring Boot
* Spring Web
* Spring Security
* Spring Data JPA
* Maven

## Decision

OfferBuddy will use Java 21 and Spring Boot for the backend application.

The backend will expose REST APIs to the React frontend.

## Rationale

Java 21 was selected because:

* It is a current long-term support Java version.
* It provides modern Java language capabilities.
* It aligns with the target backend engineering roles.
* It is suitable for building maintainable business applications.

Spring Boot was selected because:

* It provides mature support for REST APIs.
* It integrates well with Spring Security.
* It provides strong database and transaction support.
* It supports validation, testing, observability, and configuration.
* It is widely used in enterprise Java development.
* It allows the project to begin as a modular monolith and evolve later if necessary.

## API Style

The initial backend will expose RESTful HTTP APIs.

Example resource areas may include:

* `/api/applications`
* `/api/companies`
* `/api/jobs`
* `/api/users`
* `/api/auth`

The detailed API contract will be defined during implementation.

## Alternatives Considered

### Node.js

Node.js could support full-stack TypeScript development, but Java and Spring Boot were selected because they better represent the intended backend engineering portfolio.

### .NET

.NET is a strong backend platform, but it does not align as closely with the current Java experience and project goals.

### Microservices

Microservices were not selected for the MVP.

The application will begin as a modular monolith because:

* The project is being developed by one engineer.
* The MVP has a limited number of business domains.
* Independent service scaling is not currently required.
* Distributed system complexity would slow down delivery.
* Operational cost should remain low.

The modular monolith decision will be documented separately in ADR-001.

---

# Database

## Selected Technology

* PostgreSQL

## Decision

PostgreSQL will be used as the primary relational database.

## Rationale

PostgreSQL was selected because:

* OfferBuddy contains strongly related business data.
* Application records require transactional consistency.
* It supports relational constraints and indexing.
* It is open source.
* It is widely supported by cloud platforms.
* It works well with Spring Boot and JPA.
* It supports future reporting and analytics requirements.

## Initial Data Areas

The initial database is expected to store:

* Users
* External authentication identities
* Companies
* Jobs
* Applications
* Application status history
* User notes

The final MVP data model will be documented in `architecture/data-model.md`.

## Alternatives Considered

### MySQL

MySQL would also meet the MVP requirements.

PostgreSQL was selected because of its strong relational features, standards support, and suitability for future reporting and search capabilities.

### MongoDB

MongoDB was not selected because OfferBuddy has clear relationships between users, jobs, companies, and applications.

A relational database provides a more natural data model for the current requirements.

---

# Database Migration

## Selected Technology

* Flyway

## Decision

All database schema changes will be managed through versioned Flyway migration scripts.

## Rationale

Flyway was selected because:

* Database changes can be reviewed through Git.
* Local, test, and production environments can use the same schema history.
* Manual production database changes can be avoided.
* It integrates directly with Spring Boot.
* It provides a clear record of schema evolution.

Example migration files:

```text
V1__create_users.sql
V2__create_companies.sql
V3__create_jobs.sql
V4__create_applications.sql
```

---

# Data Access

## Selected Technology

* Spring Data JPA

## Decision

Spring Data JPA will be used for the initial persistence layer.

## Rationale

Spring Data JPA was selected because:

* It reduces repetitive CRUD implementation.
* It supports entity relationships.
* It integrates with Spring transaction management.
* It is sufficient for the expected MVP query complexity.
* It enables rapid delivery of the first version.

Complex reporting or performance-sensitive queries may later use:

* JPQL
* Native SQL
* Projections
* JDBC-based queries

The project should avoid forcing every query through a complex JPA object graph.

---

# Authentication and Security

## Selected Technologies

* Google OAuth 2.0
* OpenID Connect
* Spring Security

## Decision

The MVP will support Google Sign-In as the initial authentication method.

Email and password authentication will not be implemented in the first release.

## Rationale

Google authentication was selected because:

* Users do not need to create another password.
* The MVP does not need password reset functionality.
* The MVP does not need email verification functionality.
* Authentication scope is reduced.
* The first login experience is faster.
* The initial target users are expected to have Google accounts.

## Initial Security Requirements

* Protected APIs must require authentication.
* Users must access only their own application records.
* Authentication tokens must not appear in URLs.
* Sensitive credentials must not be committed to Git.
* Environment-specific secrets must be stored outside the source code.
* Production traffic must use HTTPS.
* Backend authorization must not rely only on frontend checks.
* User ownership must be validated when reading or modifying records.

The final session and token handling approach will be documented during the authentication design.

---

# AI-Assisted Job Parsing

## Selected Direction

OfferBuddy will integrate with an external large language model provider to extract structured information from job advertisement content.

The exact provider and model remain an implementation decision. Possible providers may include OpenAI, Google Gemini, Anthropic, or another provider capable of reliable structured output.

The application should avoid coupling core business logic directly to one provider-specific SDK.

## MVP Responsibility

The AI capability is limited to job information extraction. The AI provider may receive cleaned job advertisement content and return structured fields such as job title, company name, location, employment type, salary information, job description, required skills, and source platform.

## AI Processing Flow

```text
Submitted job URL
        ↓
Backend retrieves page content
        ↓
Backend removes unnecessary page content
        ↓
Backend sends relevant text to AI provider
        ↓
AI returns structured output
        ↓
Backend validates and normalises the result
        ↓
Frontend displays an editable draft
        ↓
User confirms and saves
```

## Design Requirements

- AI output must be treated as untrusted input.
- The backend must validate the response schema.
- Missing fields must be allowed.
- Invalid responses must not be saved automatically.
- The AI provider must not write directly to the database.
- The user must review the result before saving.
- A provider failure must not block manual entry.
- Provider credentials must remain outside source control.
- Requests should use timeouts.
- Input size should be limited.
- Usage and cost should be controlled.
- Sensitive information should not be sent unnecessarily.

## Provider Abstraction

The backend should define an application-owned interface such as:

```java
public interface JobInformationExtractor {

    JobExtractionResult extract(JobContent content);
}
```

A provider-specific implementation may then be added separately:

```text
AiJobInformationExtractor
OpenAiJobInformationExtractor
GeminiJobInformationExtractor
```

This allows the model or provider to change without rewriting the application workflow.

## Structured Output

The preferred AI response format is structured JSON.

```json
{
  "jobTitle": "Software Engineer",
  "companyName": "Example Company",
  "location": "Sydney NSW",
  "employmentType": "Full-time",
  "salaryText": null,
  "description": "Role description...",
  "skills": ["Java", "Spring Boot", "React"],
  "sourcePlatform": "SEEK"
}
```

The backend must validate and map this response into an internal data transfer object.

## Deferred AI Capabilities

- Resume optimisation
- Resume-to-job scoring
- Cover letter generation
- Interview question generation
- Mock interviews
- Career recommendations
- Automated job applications

---

# Local Development

## Selected Technologies

* Docker
* Docker Compose

## Decision

Docker Compose will be used to run local infrastructure dependencies.

The frontend and backend may initially run directly from the developer environment for faster development, while PostgreSQL runs in Docker.

An initial local environment may contain:

```text
React frontend
Spring Boot backend
PostgreSQL container
```

## Rationale

Docker Compose was selected because:

* It provides a repeatable local environment.
* It reduces machine-specific database configuration.
* It supports future addition of infrastructure services.
* It is simpler and cheaper than Kubernetes.
* It can also support integration testing.

## Services Not Included Initially

The following services will not be added until there is a demonstrated requirement:

* Redis
* RabbitMQ
* Kafka
* Elasticsearch
* Kubernetes
* Service discovery
* Distributed tracing infrastructure

---

# Build Tools and Package Management

## Backend

* Maven
* Maven Wrapper

The repository should include the Maven Wrapper so the backend can be built without requiring every developer or CI environment to install the same Maven version manually.

Example command:

```bash
./mvnw clean verify
```

On Windows:

```powershell
.\mvnw.cmd clean verify
```

## Frontend

* npm
* `package-lock.json`

The lock file must be committed to ensure consistent dependency versions.

Example commands:

```bash
npm ci
npm run build
npm run test
```

---

# Testing

## Backend Testing

Selected tools:

* JUnit 5
* Mockito
* Spring Boot Test
* Testcontainers

Testing levels may include:

* Unit tests for business logic
* Repository integration tests
* API integration tests
* Security and authorization tests

Testcontainers will be used where database behaviour should be tested against a real PostgreSQL instance.

## Frontend Testing

Selected tools:

* Vitest
* React Testing Library

Testing will focus on:

* Component behaviour
* Form validation
* User interactions
* API state handling
* Authentication-related rendering

## End-to-End Testing

End-to-end testing is not required during the initial project setup.

A tool such as Playwright may be introduced when the main application workflow is operational.

---

# API Documentation

## Selected Technology

* OpenAPI
* Swagger UI

## Decision

The backend should generate an OpenAPI specification for its REST APIs.

## Rationale

OpenAPI provides:

* A clear API contract
* Easier frontend integration
* Interactive API exploration
* Better documentation for recruiters and contributors
* A foundation for future client generation

Swagger UI should be restricted or disabled in production if necessary.

---

# Continuous Integration

## Selected Technology

* GitHub Actions

## Initial CI Responsibilities

The initial workflow should:

* Build the backend
* Run backend tests
* Build the frontend
* Run frontend tests
* Fail when compilation or tests fail

Later workflows may add:

* Docker image builds
* Dependency scanning
* Static analysis
* Deployment
* Release creation

## Rationale

GitHub Actions was selected because:

* The repositories are hosted on GitHub.
* It requires minimal additional infrastructure.
* It supports pull request validation.
* It provides visible evidence of automated engineering quality.

---

# Deployment

## Initial Direction

OfferBuddy will use Docker-based deployment on a managed cloud platform or virtual server.

The final hosting provider has not yet been selected.

The initial production environment is expected to include:

* React static frontend
* Spring Boot backend
* Managed PostgreSQL database
* HTTPS
* Environment-based configuration
* Automated deployment where practical

## Not Selected

Kubernetes will not be used for the MVP because:

* The system does not require container orchestration.
* There is only one backend application.
* Operational complexity would be unnecessary.
* Infrastructure cost should remain low.
* It previously introduced avoidable cost and complexity in earlier portfolio work.

---

# Observability

## Initial Scope

The MVP will begin with:

* Structured application logs
* Clear error messages
* Request correlation identifiers where practical
* Basic health endpoints
* Cloud platform metrics where available

## Future Scope

The following may be introduced later:

* Centralised log storage
* Error tracking
* Performance monitoring
* Distributed tracing
* Product analytics
* Alerting

Observability infrastructure should be introduced according to actual operational needs rather than for architectural appearance.

---

# Technologies Deferred Until Needed

The following technologies are intentionally deferred:

| Technology                 | Reason for Deferral                                            |
| -------------------------- | -------------------------------------------------------------- |
| Redis                      | No confirmed caching or distributed session requirement        |
| RabbitMQ                   | No asynchronous workflow is required for the first MVP         |
| Kafka                      | Event streaming complexity is not justified                    |
| Elasticsearch              | PostgreSQL search is sufficient initially                      |
| Kubernetes                 | Deployment scale does not justify orchestration                |
| Microservices              | Modular monolith is simpler for the current scope              |
| GraphQL                    | REST is sufficient for the expected API needs                  |
| Next.js                    | SSR and SEO are not required for the authenticated application |
| Native mobile applications | Responsive web delivery has higher priority                    |
| Advanced AI career features | Job extraction is included, but resume and interview features are deferred |

A deferred technology should be introduced only when there is a clear functional, performance, operational, or business requirement.

---

# Engineering Principles

## Keep the first release simple

Technology must support product delivery rather than delay it.

## Prefer proven technologies

The MVP should use stable, well-supported tools with strong documentation and community support.

## Avoid resume-driven architecture

Technologies should not be introduced only to make the architecture appear more complex.

## Maintain clear boundaries

The frontend, backend, business modules, and persistence layer should have clear responsibilities.

## Design for change, not speculation

The system should be easy to extend, but it should not implement future complexity before it is required.

## Automate repeatable checks

Builds, tests, migrations, and deployment steps should become automated as the project develops.

---

# Open Decisions

The following decisions still require confirmation:

* React UI component library
* HTTP client and server-state library
* Frontend routing library
* Backend package and module structure
* Authentication session implementation
* Cloud hosting provider
* Production PostgreSQL provider
* Secret management approach
* Error tracking platform
* End-to-end testing tool
* Initial AI provider
* Initial AI model
* Structured-output implementation
* Maximum job content size
* AI request timeout
* AI usage limits
* Retry policy
* Cost monitoring approach
* Job page content extraction library

These decisions should be made before or during the relevant implementation work rather than all at once.

---

# Current Status

**Status:** Accepted for MVP foundation

**Date:** 3 August 2026

The stack may evolve as implementation constraints and product requirements become clearer. Significant changes should be documented through ADRs and reflected in this document.
