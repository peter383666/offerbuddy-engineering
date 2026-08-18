# ADR-010: Use Lightweight Business Events Without a Message Broker

## Status

Accepted for Sprint 2 architecture.

## Date

18 August 2026.

## Context

Sprint 2 adds downstream Job Intelligence and Analytics. Neither capability may extend the critical Application write path or make successful core operations depend on AI, Analytics projections, Redis, or external asynchronous consumers.

The architecture needs an explicit dependency direction that lets upstream business modules announce completed facts and downstream modules react without introducing distributed infrastructure disproportionate to the current modular monolith and single-host deployment.

## Decision

Use lightweight Business Events within the Spring Boot modular monolith to decouple downstream Job Intelligence and Analytics from core Application operations.

The dependency principle is:

> Upstream business modules publish facts; downstream modules react to facts.

Conceptual facts include Application creation, Application status change, and completed Job analysis. Exact event names and representations are not defined by this ADR.

The core business operation commits independently. A downstream handler failure does not roll back an already-successful Application operation. Analytics is eventually consistent, and Job Intelligence is non-critical to Application tracking.

Sprint 2 does not introduce Kafka, RabbitMQ, microservices, event sourcing, a CQRS framework, distributed streaming, mandatory DLQ infrastructure, or exactly-once distributed messaging.

## Boundaries

- Business Events describe completed business facts; they do not transfer ownership of upstream business rules.
- Application remains responsible for Application lifecycle and core writes.
- Job Intelligence owns semantic Job analysis.
- Analytics owns read-oriented derived views.
- PostgreSQL remains the primary business-data and Analytics storage/query foundation.
- Event representation, dispatch, persistence, ordering, durability, retry, recovery, and observability belong to Technical Design.

## Consequences

### Positive

- The Application core path stays small.
- AI and Analytics failures are isolated from core writes.
- Downstream modules can react without direct synchronous coupling from Application.
- The design remains compatible with one backend deployable and current operations.
- Future downstream capabilities can consume established facts without being placed inside Application logic.

### Negative

- Eventual consistency becomes visible for downstream views.
- Phase 3 must define and verify dispatch, durability, retry, and restart behaviour.
- In-process modular boundaries require discipline because process isolation does not enforce them.
- Operators need enough observability to distinguish core success from downstream lag or failure.

## Alternatives Considered

### Synchronous direct calls from Application

Rejected because AI latency or downstream failure would enlarge and weaken the critical write path and couple Application to downstream concerns.

### External message broker or distributed streaming platform

Rejected for Sprint 2 because the current scale and single deployable do not justify Kafka, RabbitMQ, or equivalent operational complexity.

### Event sourcing or a CQRS framework

Rejected because Sprint 2 needs lightweight downstream decoupling, not a replacement persistence model or application-wide architectural rewrite.

### Redis as the event or Analytics foundation

Rejected because Redis availability is not a requirement for core writes or S2 Analytics, and Redis must not become a source of truth merely because it already exists in Compose.

See [Sprint 2 Architecture Design](../architecture/sprint-2-architecture-design.md).
