# Sprint 2 Redis Design

## Document Status

| Item | Value |
| --- | --- |
| Phase | Phase 3 — Technical Design |
| Section | 3.5 — Redis |
| Status | Completed and approved |
| Implementation status | No new Redis-backed Sprint 2 capability is required |

## Purpose

This document records the intentionally limited Sprint 2 Redis decision. Redis is retained as available infrastructure but is used only when a concrete requirement justifies ephemeral state or caching. The confirmed Sprint 2 feature set introduces no such required use.

PostgreSQL remains the durable system of record. Redis is neither forbidden nor promoted into the architecture merely because a container already exists.

## Verified Existing State

Sprint 1 local and production Compose definitions include Redis 8 as reserved, private infrastructure. The Spring Boot application does not currently connect to it for sessions, caching, queues, or business data.

Web authentication currently uses the existing Google OIDC and OfferBuddy-managed Web-session design without Redis session storage. Sprint 2 does not change that fact through Redis Design.

This is a retained infrastructure capability, not an active application dependency.

## Responsibility Boundary

### PostgreSQL owns

- users and authenticated OfferBuddy identity data;
- canonical/shared Jobs and platform identity;
- user-owned Applications and lifecycle history;
- Job Intelligence analysis and structured results;
- durable `business_events` and retry state;
- Application Analytics projections.

### Redis may own

Only explicitly justified ephemeral/cache state for a future concrete requirement with understood invalidation and a safe fallback.

No confirmed Sprint 2 responsibility above moves to Redis.

## Sprint 2 Decisions

| Area | Redis decision |
| --- | --- |
| Extension ingestion | No buffering, capture storage, Extension session domain, or ingestion persistence |
| Job identity and duplicates | PostgreSQL/domain uniqueness remains authoritative |
| Application tracking | Redis is not in the critical path |
| Business Events | PostgreSQL `business_events`, not Streams, Lists, Pub/Sub, or a Redis job queue |
| Job Intelligence | No Redis task queue, result/prompt cache, analysis-state store, or retry store |
| Analytics | PostgreSQL projection with no Redis counters, aggregates, or cache |
| Authentication | No new Redis session, token blacklist, or refresh-token requirement |

## Caching Admission Rule

A Redis cache is added only when all of these are demonstrated:

1. a concrete repeated read path exists;
2. measured PostgreSQL/application performance is insufficient or caching has a clear product benefit;
3. invalidation semantics are understood;
4. stale data is acceptable;
5. Redis failure has a safe source-of-truth fallback.

No currently confirmed Sprint 2 path meets this threshold. The default is therefore no new Redis cache.

## Key, TTL, and Invalidation Policy

Sprint 2 defines no new Redis key space, TTL values, or cache invalidation flows because it defines no new Redis-backed feature.

Hypothetical Job, Application, Analytics, Job Intelligence, Event, or Extension keys would create undocumented consistency responsibilities and are not part of the design. TTL and invalidation must be defined per real ephemeral use case, never globally or speculatively.

## Failure Behaviour

Redis unavailability has no effect on confirmed Sprint 2 core correctness:

- Job and Application persistence continues through PostgreSQL;
- accepted Extension ingestion does not lose durable domain data;
- Application creation/status changes and lifecycle history remain available;
- durable Business Events and retry state remain in PostgreSQL;
- Job Intelligence retains durable event/analysis state;
- Analytics source data and projection remain intact.

If a future optional Redis-backed feature is approved, Redis failure may degrade only that feature and must fall back safely without becoming authoritative.

## Explicit Non-Goals

- Redis Streams, Lists, or Pub/Sub event delivery
- Redis-backed AI queues, retry state, prompt caches, or result caches
- Redis Analytics counters, aggregates, projections, or read cache
- Extension persistence, capture buffering, or duplicate detection
- Distributed locks, Redlock, leader election, or distributed scheduling
- Redis Cluster or Sentinel
- Rate limiting introduced without a separate confirmed requirement
- Write-through/write-behind, multi-level caching, warming, or stampede frameworks

Existing Redis infrastructure is not removed by this decision. Its responsibilities simply do not expand in Sprint 2 without evidence and approval.

## Related Documents

- [Sprint 2 Technical Design Index](README.md)
- [Database Design](database-design.md)
- [Backend / Service Design](backend-service-design.md)
- [Event Design](event-design.md)
- [Job Intelligence Design](job-intelligence-design.md)
- [Analytics Design](analytics-design.md)
- [Sprint 2 Architecture Design](../../architecture/sprint-2-architecture-design.md#postgresql-and-redis-decisions)
- [Technology Stack](../../technology/tech-stack.md)
- [Deployment Strategy](../../operations/deployment-strategy.md)
- [Production Runbook](../../operations/production-runbook.md)
- [ADR-010 — Lightweight Business Events](../../decisions/ADR-010-lightweight-business-events.md)
