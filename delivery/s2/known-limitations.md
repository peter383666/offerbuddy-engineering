# Sprint 2 Known Limitations

## Purpose

Records limitations that remain true after Sprint 2 delivery. These are not hidden failures; they are accepted constraints or incomplete automation.

## Product / UX

- Extension tracking has two client paths (explicit Save and confirmation-based ingest). Messaging must stay clear so users understand when an Application is created.
- Eligibility screening surfaces citizenship / permanent-residency findings only. Clearance, working-rights, and sponsorship concepts from requirements are not fully surfaced.
- SEEK and Indeed page structure can change without notice; capture may fail safely until adapters are updated.
- New Application AI URL parsing remains dependent on server-side page retrieval and provider latency.

## Architecture / Data

- Legacy persisted `NO_RESPONSE` Application status remains; Analytics also derives a no-response classification for aged `APPLIED` applications.
- Redis is still present in Compose as reserved infrastructure and is unused by application logic.
- Business Events provide at-least-once delivery, not exactly-once broker semantics.
- Analytics is eventually consistent; projection lag is possible after Core writes.

## Quality / CI

- Extension CI and Frontend Vitest as a Frontend CI gate are follow-ups on application `review/s2-ci-followups` until merged to `main`.
- No browser end-to-end automation against live SEEK/Indeed.
- Live Gemini verification remains conditional on credentials and is skipped in normal offline CI.

## Operations

- Single-host EC2 deployment has no automatic failover.
- Backups are host-local unless copied elsewhere.
- Observability is primarily health endpoints, Compose/Nginx logs, and event-state inspection.
- Application artifact rollback does not reverse Flyway migrations.

## Related

- [Deferred Items](deferred-items.md)
- [Implementation Reconciliation](implementation-reconciliation.md)
- [Quality S2](../../quality/s2/README.md)
- [Operations S2](../../operations/s2/README.md)
