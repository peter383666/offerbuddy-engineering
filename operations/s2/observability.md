# Sprint 2 Observability

## Purpose

Records what operators can observe for Sprint 2 without claiming a full observability platform.

## Available Signals

- Backend Actuator health (`/actuator/health`)
- Docker Compose service status and backend logs
- Host Nginx access/error logs
- Application structured logs around Extension track, Business Event processing, Job Intelligence, and Analytics where implemented
- Business Event processing states in PostgreSQL (`PENDING` / `PROCESSING` / `RETRY` / `SUCCEEDED` / `FAILED`)

## What Is Not Present

- Centralised APM / metrics platform
- Guaranteed request correlation IDs across all boundaries
- Automated alerting for event lag or AI failure rates

These remain known limitations / follow-ups rather than Sprint 2 deliverables.

## Operator Guidance

When Core save succeeds but Intelligence/Analytics look stale:

1. Confirm backend health and recent logs.
2. Inspect Business Event states for the affected aggregate.
3. Distinguish Core success from downstream lag/failure.
4. Do not roll back Core data solely because AI enrichment failed.

## Related

- [Production Runbook](../production-runbook.md)
- [Event / Async Architecture](../../architecture/s2/event-async-architecture.md)
