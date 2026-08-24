# Analytics Architecture

## Responsibility

Application Analytics is a secondary, user-scoped, eventually consistent read capability.

```text
job_applications + status history (+ job facts)
  = authoritative source

business_events
  = incremental delivery

application_analytics
  = rebuildable projection
```

## Principles

- One projection row per Application
- Approved time ranges and conversion-oriented measures
- Derived no-response classification may be computed without inventing a new lifecycle transition
- Legacy persisted `NO_RESPONSE` status may still exist; see reconciliation
- No Redis counters or warehouse platform in Sprint 2

## Related

- [Analytics Design](../../design/s2/technical/analytics-design.md)
- [Implementation Reconciliation](../../delivery/s2/implementation-reconciliation.md)
