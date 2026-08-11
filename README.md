# OfferBuddy Engineering

> Engineering documentation for an AI-assisted job application tracking platform.

OfferBuddy helps job seekers capture opportunities, review AI-extracted job information, track applications, and understand application activity.

This repository contains the product, architecture, quality, operations, and delivery records for OfferBuddy. Application source code is maintained in the separate [OfferBuddy source repository](https://github.com/peter383666/offerbuddy).

## Current Status

| Area | Status          |
|---|-----------------|
| Product and MVP Definition | ✅ Established  |
| MVP Architecture | ✅ Accepted     |
| Sprint 0 — Engineering Foundation | ✅ Completed    |
| Local Development and CI | ✅ Established  |
| Sprint 1 | 🟡 in progress  |
| Production Deployment | ⏳ Not selected |

The current milestone is Sprint 0 Closing. Repository standardisation and the engineering documentation update precede the first engineering-foundation release and Sprint 1 Planning.

## MVP Workflow

```text
Paste job advertisement URL
        ↓
Retrieve available page content
        ↓
AI extracts structured job information
        ↓
Backend validates the result
        ↓
User reviews and corrects the draft
        ↓
Save and track the application
```

AI output is treated as untrusted draft data. Manual entry remains available when page retrieval or AI extraction is unavailable.

## Engineering Approach

OfferBuddy is developed through small, reviewable increments.

The project follows these principles:

- Keep the MVP focused on validated user value.
- Prefer simple, explicit architecture over premature distribution.
- Use one issue, one feature branch, and one pull request for delivery work.
- Require continuous integration checks before merge.
- Keep the main branch releasable.
- Evolve engineering documentation with implementation.
- End each sprint with a Review, Retrospective, documentation update, and release decision.

Detailed workflow and quality requirements are maintained in the Operations and Quality documents below.

## Documentation Index

The directories organise documentation by subject. The numbered sections provide the recommended reading order without coupling that order to folder names.

### 01 Product

| Document | Purpose |
|---|---|
| [Product Vision](product/product-vision.md) | Product goals, users, principles, and long-term direction |
| [MVP Scope](product/mvp-scope.md) | Capabilities included in and excluded from the MVP |
| [User Stories](product/user-stories.md) | User-centred functional requirements and acceptance criteria |

### 02 Technology

| Document | Purpose |
|---|---|
| [Technology Stack](technology/tech-stack.md) | Selected technologies, rationale, deferred options, and open decisions |

### 03 Architecture

Read these documents from the system boundary inward.

| Document | Purpose |
|---|---|
| [System Context](architecture/system-context.md) | Users, external systems, trust boundaries, and core information flows |
| [Container Design](architecture/container-design.md) | Runtime containers, integrations, and backend module boundaries |
| [Data Model](architecture/data-model.md) | Domain entities, relationships, ownership, and business rules |

### 04 Architecture Decision Records

| Document | Decision |
|---|---|
| [ADR Index](decisions/README.md) | Status and catalogue of architecture decisions |
| [ADR-001](decisions/ADR-001-modular-monolith.md) | Use a modular monolith for the MVP |
| [ADR-002](decisions/ADR-002-google-authentication.md) | Use Google as the initial authentication provider |
| [ADR-003](decisions/ADR-003-ai-assisted-job-extraction.md) | Use AI-assisted extraction for MVP job capture |
| [ADR-004](decisions/ADR-004-ai-provider-abstraction.md) | Abstract the AI provider behind an application-owned interface |
| [ADR-005](decisions/ADR-005-docker-compose.md) | Use Docker Compose for local infrastructure |
| [ADR-006](decisions/ADR-006-flyway.md) | Use Flyway for database migrations |

### 05 Quality

| Document | Purpose |
|---|---|
| [Testing Strategy](quality/testing-strategy.md) | Test levels, testing foundations, and AI extraction scenarios |
| [Non-Functional Requirements](quality/non-functional-requirements.md) | Security, performance, reliability, and operational quality requirements |
| [Definition of Done](quality/definition-of-done.md) | Minimum completion standard for project work |

### 06 Operations

| Document | Purpose |
|---|---|
| [Development Workflow](operations/development-workflow.md) | OfferBuddy development and change-integration workflow |
| [Deployment Strategy](operations/deployment-strategy.md) | Local Docker, CI, database migration, and production deployment direction |

### 07 Delivery

| Document | Purpose |
|---|---|
| [Delivery Roadmap](delivery/roadmap.md) | Sprint sequence and delivery direction |
| [Product Backlog](delivery/product-backlog.md) | Identified work not committed to the current sprint |
| [Sprint 0 Planning](delivery/sprint-0.md) | Original Sprint 0 goal, plan, task results, and outcome |
| [Sprint 0 Review](delivery/sprint-0-review.md) | Delivered increment, evidence, demonstration, and metrics |
| [Sprint 0 Retrospective](delivery/sprint-0-retrospective.md) | Lessons learned and engineering improvement actions |

## Release History

| Version | Description |
|---|---|
| `engineering-v0.1` | Product and initial MVP architecture documentation established |

Future releases will use the versioning and release process agreed during repository standardisation.

## License

This repository contains engineering documentation for the OfferBuddy project. Application source code is maintained separately.
        