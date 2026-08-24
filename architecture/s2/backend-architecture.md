# Backend Architecture

## Shape

OfferBuddy remains a Spring Boot **modular monolith** with one deployable API container.

Logical Sprint 2 responsibilities inside the monolith:

- Web authentication / session
- Extension pairing and credential verification
- Extension Track ingestion
- Job and Application core
- Business Event write and dispatch
- Job Intelligence processing
- Application Analytics projection and query
- AI URL parsing fallback

## Core Write Path

```text
Authenticated request
  -> validate / resolve Job
  -> create-or-reuse Application
  -> write status history when required
  -> persist Business Event intent
  -> commit
  -> return Core result
```

Job Intelligence and Analytics run after commit. Their failure does not roll back Core success.

## Related

- [Architecture Overview](architecture-overview.md)
- [Backend Service Design](../../design/s2/technical/backend-service-design.md)
- [API Design](../../design/s2/technical/api-design.md)
- [ADR-001](../../decisions/ADR-001-modular-monolith.md)
