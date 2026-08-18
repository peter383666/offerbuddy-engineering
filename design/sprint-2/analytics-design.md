# Sprint 2 Analytics Design

## Document Status

| Item | Value |
| --- | --- |
| Phase | Phase 3 — Technical Design |
| Section | 3.8 — Analytics |
| Status | Completed and approved |
| Implementation status | Target design; no Analytics module or projection exists yet |

## Purpose

This document defines the small Sprint 2 Application Analytics projection, lifecycle semantics, incremental processing, rebuild behaviour, query boundary, and failure isolation.

Analytics is a secondary capability that gives an authenticated user useful visibility into their own Application activity and outcomes. The Browser Extension remains the primary Sprint 2 capability; this design does not turn Sprint 2 into an Analytics or data-platform programme.

## Ownership and Source of Truth

Analytics is a downstream, user-scoped read capability. It does not own Application lifecycle facts.

```text
job_applications + application_status_history + relevant jobs facts
  = authoritative source

business_events
  = incremental delivery

application_analytics
  = derived, rebuildable read projection
```

Business Events are not the only historical source and the projection is not authoritative.

The persistence responsibilities remain distinct:

- `job_applications` owns current Application state;
- `application_status_history` owns real lifecycle-transition history;
- `application_analytics` owns derived, replaceable read state.

Status history is not Event Sourcing, and Analytics never reconstructs ownership or business truth from projection data alone.

## Projection Model

One `application_analytics` row conceptually represents one Application and includes enough derived state for small lifecycle/conversion queries:

- Application and owning user identity;
- source platform and applied date;
- whether interview, offer, and rejection milestones were reached;
- first known date/time for each milestone when reliable.

Boolean milestone evidence and nullable first-milestone time are intentionally distinct. Legacy data can establish that a milestone was reached without proving when. Missing historical timestamps remain null rather than being fabricated.

## NO_RESPONSE Semantics

`NO_RESPONSE` is a derived Analytics classification in the Sprint 2 target model, not a persisted lifecycle transition.

Conceptually, it is derived from elapsed time since application and the absence of meaningful lifecycle progress. The threshold is an Analytics business rule/configuration decision, not a database lifecycle constraint. This design does not invent its numeric value.

Legacy Sprint 1 `NO_RESPONSE` status requires controlled migration toward `APPLIED` semantics. Migration does not manufacture history. Analytics may use only reliable legacy evidence.

## Incremental Processing

Application-created and Application-status-changed Business Events trigger projection convergence after Core commit.

Handlers use state/upsert semantics:

- reread authoritative Application/history where necessary;
- set milestone state based on observed lifecycle facts;
- keep the earliest reliable milestone time;
- tolerate duplicate and out-of-order delivery;
- never use blind `count++` updates that retry can double-count.

Analytics failure leaves Core state unchanged and may temporarily increase projection lag.

## Rebuild

The projection can be rebuilt from authoritative Core rows and status history without relying on old event delivery.

A rebuild:

1. reads user-owned Applications and related Job/source facts;
2. derives milestones from real status history;
3. preserves unknown historical times as null;
4. upserts deterministic projection state;
5. can be rerun safely;
6. does not hold or block the Core mutation path for long-running work.

Exact batching, checkpoints, operational commands, and reconciliation schedule remain implementation details.

## Read API Boundary

Sprint 2 provides authenticated summary and trend-oriented Analytics reads within the existing `/api/v1` family. Exact endpoint paths and response DTOs are fixed with the implemented API contract rather than fabricated here.

The backend derives ownership from the authenticated OfferBuddy user. Clients cannot supply an arbitrary `userId` to read another user's Analytics.

Analytics is strictly user-scoped Application Analytics. It does not aggregate cross-user product telemetry, browser behaviour, or company-wide benchmarks. Extension activity affects Analytics only after normal authenticated Backend/Application operations produce authoritative facts.

Approved small metrics include:

- total Applications;
- response/no-response-oriented insight;
- interview, offer, and rejection conversion measures;
- time to lifecycle milestones where reliable;
- simple time trends.

Current operational status counts may query Core Application data directly where that is simpler and correct. Lifecycle/conversion semantics use history/projection evidence rather than current status alone.

Detailed HTTP paths, request/response DTOs, and OpenAPI artifacts belong to the Backend API contract. This design defines read-model responsibility and does not invent additional convenience endpoints.

## Job Intelligence Boundary

Basic Application Analytics depends on Application, status-history, and relevant persisted Job facts—not successful AI Job Intelligence. AI analysis failure or delay does not prevent totals, lifecycle conversion, activity trends, or other confirmed basic metrics from being processed or read.

Sprint 2 does not add AI-generated Analytics, recommendations, predictions, suitability ranking, or match scoring.

## Consistency and Availability

Analytics is eventually consistent. API responses may lag a newly committed lifecycle change while its event is pending or retrying.

Core Application creation/status updates do not call Analytics synchronously and do not fail because Analytics processing is unavailable. Operational monitoring distinguishes Core success from projection lag.

## Storage and Performance

PostgreSQL is the S2 Analytics storage and query foundation. The projection and supporting queries are indexed according to verified access patterns during implementation.

Sprint 2 introduces no Redis Analytics cache, warehouse, distributed query engine, or hierarchy of daily/weekly/monthly aggregate tables. Scale is addressed first with bounded projection updates, suitable PostgreSQL indexes, and measured queries.

## Idempotency Tests and Reconciliation

Testing must verify:

- duplicate delivery produces the same projection state;
- out-of-order lifecycle delivery converges correctly;
- rebuild matches incremental projection results;
- legacy milestone booleans do not fabricate timestamps;
- NO_RESPONSE derivation does not create a lifecycle transition;
- user ownership is resolved server-side;
- Analytics failure does not affect Core mutations.

## Section 3.8 Non-Goals

- Job Market or skill-market Analytics
- Company-wide BI, cross-user benchmarking, or recruiter-market Analytics
- Browser/product telemetry or behavioural tracking
- Predictive success, ML Analytics, recommendations, or AI-generated insights
- Redis caching
- Kafka, streaming Analytics, CQRS infrastructure, or a BI/OLAP warehouse
- Elasticsearch, ClickHouse, BigQuery, Snowflake, or Redshift
- Large aggregate-table hierarchies
- Persisting NO_RESPONSE as a new lifecycle transition
- Fabricating historical dates
- Client-selected ownership
- Resume Match or recommendation Analytics

## Related Documents

- [Sprint 2 Design Index](README.md)
- [Database Design](database-design.md)
- [Redis Design](redis-design.md)
- [Event Design](event-design.md)
- [Job Intelligence Design](job-intelligence-design.md)
- [Backend / Service Design](backend-service-design.md)
- [Sprint 2 Architecture Design](../../architecture/sprint-2-architecture-design.md)
- [ADR-007 — PostgreSQL](../../decisions/ADR-007-postgresql.md)
- [ADR-010 — Lightweight Business Events](../../decisions/ADR-010-lightweight-business-events.md)
