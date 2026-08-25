# ADR-S2-004: Business Events without a Message Broker

## Status

Accepted — implemented in Sprint 2.

## Context

Job Intelligence and Analytics need decoupling from Core writes without introducing Kafka, RabbitMQ, or microservices for a single-host modular monolith.

## Decision

Use lightweight PostgreSQL-backed Business Events inside the modular monolith. Domain mutation and durable event intent share one short transaction; claim / retry / recovery run after commit with at-least-once semantics. Downstream processors own idempotency.

## Alternatives

- Synchronous downstream calls from Application services
- Introduce an external message broker
- Outbox plus separate worker fleet / new deployables

## Consequences

- Proportional to current operations model
- Exactly-once broker guarantees are explicitly out of scope
- Cross-sprint detail: [ADR-010](../ADR-010-lightweight-business-events.md)

## Related

- [Event / Async Architecture](../../architecture/s2/event-async-architecture.md)
- [Event Design](../../design/s2/technical/event-async-design.md)
