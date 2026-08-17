# ADR-005: Use Docker Compose for Local Infrastructure

## Status

Accepted

## Date

5 August 2026

## Context

OfferBuddy depends on infrastructure services such as PostgreSQL during local development.

Installing and configuring these services directly on each developer machine would create environment differences, increase setup time, and make failures harder to reproduce. The project needs a lightweight way to provide consistent local infrastructure without introducing production orchestration complexity.

The solution should:

- Provide repeatable service versions and settings
- Minimise machine-specific setup
- Support isolated startup and shutdown of infrastructure
- Remain understandable and maintainable for a small team
- Avoid coupling local development to a specific cloud provider
- Support future integration testing where appropriate

## Decision

OfferBuddy will use Docker Compose to define and run local infrastructure dependencies.

Application processes may run directly from the developer environment for fast feedback, while infrastructure services run as containers. Only services with a confirmed development or testing purpose should remain in the Compose environment.

This ADR originally selected Docker Compose for local infrastructure only. Sprint 1 later also selected Docker Compose for its single-host production runtime; that separate production decision is recorded in [ADR-008](ADR-008-single-host-production.md).

## Rationale

Docker Compose was selected because it provides a version-controlled, reproducible environment with substantially less operational complexity than a container orchestration platform.

It allows contributors and CI-compatible workflows to use consistent infrastructure versions while preserving normal IDE-based frontend and backend development.

The project does not currently require service discovery, automated scheduling, multi-node orchestration, or independent service scaling.

## Consequences

### Positive

- Local infrastructure can be reproduced consistently.
- Service versions and dependencies are visible in source control.
- Developers avoid installing databases and supporting services directly.
- Local environments can be started and stopped independently of application code.
- Additional infrastructure can be evaluated without redesigning the application.

### Negative

- Developers must install and run a compatible container runtime.
- Container networking and persistent volumes add concepts to local development.
- Local resource usage increases while containers are running.
- A Compose environment can drift from production if it is not maintained deliberately.
- Adding an unused service can create unnecessary complexity and maintenance cost.

## Alternatives Considered

### Install Infrastructure Directly

Rejected because machine-specific installation and configuration would reduce reproducibility and increase onboarding effort.

### Run the Entire Application in Containers During Development

Not selected as the default because running the frontend and backend directly provides faster feedback and simpler IDE debugging. Full containerisation may still be used for deployment or integration verification.

### Kubernetes

Rejected because OfferBuddy does not require orchestration at its current scale. Kubernetes would add operational complexity without improving MVP delivery.

### Cloud-Hosted Development Dependencies

Rejected as the default because it would introduce network dependency, shared-environment coordination, and ongoing cost into normal local development.

## Related Documents

- `technology/tech-stack.md`
- `architecture/container-design.md`
- `operations/development-workflow.md`
- `operations/deployment-strategy.md`

## Outcome

Docker Compose is the standard mechanism for reproducible local infrastructure in OfferBuddy. PostgreSQL and reserved Redis run locally while application processes normally run on the host.

Redis being present in Compose does not make it an active application dependency. Sprint 1 does not use Redis for sessions, caching, queues, or business data.
