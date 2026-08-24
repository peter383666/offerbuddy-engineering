# Event / Async Architecture

## Decision

Sprint 2 uses lightweight PostgreSQL-backed Business Events inside the modular monolith. No message broker is introduced.

```text
domain mutation + event intent
  -> same short transaction
  -> commit
  -> asynchronous claim/process
  -> domain handler
```

## Guarantees

- At-least-once delivery with idempotent handlers
- Bounded retry and visible terminal failure
- Core success independent of downstream outcome

Not provided: exactly-once broker semantics, Kafka/RabbitMQ, Redis queues.

## Related

- [Event / Async Design](../../design/s2/technical/event-async-design.md)
- [ADR-010](../../decisions/ADR-010-lightweight-business-events.md)
