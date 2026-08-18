# Architecture Decision Records

ADRs record decisions with meaningful alternatives and lasting architectural consequences. Implementation details that do not meet that threshold remain in the relevant architecture or operations documentation.

| ADR | Decision | Status |
| --- | --- | --- |
| [ADR-001](ADR-001-modular-monolith.md) | Use a modular monolith for the MVP | Accepted, implemented |
| [ADR-002](ADR-002-google-authentication.md) | Use Google OIDC with a server-side OfferBuddy session | Accepted, implemented |
| [ADR-003](ADR-003-ai-assisted-job-extraction.md) | Use AI-assisted extraction for job capture | Accepted, implemented |
| [ADR-004](ADR-004-ai-provider-abstraction.md) | Abstract the AI provider behind application-owned interfaces | Accepted, implemented |
| [ADR-005](ADR-005-docker-compose.md) | Use Docker Compose for local infrastructure | Accepted, implemented |
| [ADR-006](ADR-006-flyway.md) | Use Flyway for database migrations | Accepted, implemented |
| [ADR-007](ADR-007-postgresql.md) | Use PostgreSQL as the primary database | Accepted, implemented |
| [ADR-008](ADR-008-single-host-production.md) | Use single-host EC2 deployment with immutable SHA artifacts | Accepted, implemented |
| [ADR-009](ADR-009-browser-extension-site-adapters.md) | Use Browser Extension Site Adapters for primary Sprint 2 Job Capture | Accepted for Sprint 2 architecture |
| [ADR-010](ADR-010-lightweight-business-events.md) | Use lightweight Business Events without a message broker | Accepted for Sprint 2 architecture |

Release branch responsibilities are documented as a delivery/operations decision rather than an ADR. Redis retention is documented as current infrastructure state, not as an active application architecture decision.
