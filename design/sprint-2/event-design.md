# Sprint 2 Event Design

## Document Status

| Item | Value |
| --- | --- |
| Phase | Phase 3 — Technical Design |
| Section | 3.6 — Events |
| Status | Completed and approved |
| Implementation status | Target design; no event infrastructure exists yet |
| Architecture decision | [ADR-010 — Lightweight Business Events](../../decisions/ADR-010-lightweight-business-events.md) |

## Purpose

This document defines durable Business Event persistence, dispatch, retry, idempotency, and failure boundaries for Sprint 2. Exact event classes, payload schemas, polling SQL, schedules, and framework configuration remain implementation-owned.

## Reliability Model

Sprint 2 uses PostgreSQL-backed `business_events` as a lightweight transactional-outbox-style mechanism inside the modular monolith.

```text
domain mutation + durable event persistence
  -> same short PostgreSQL transaction
  -> commit
  -> asynchronous dispatch
  -> domain-specific handler
```

This provides atomic recording of the business fact and its delivery intent. It does not claim exactly-once processing.

## Producer Ownership

The module that owns a fact produces its event intent:

- Job owns Job-created and Job-source-facts-updated facts;
- Application owns Application-created and Application-status-changed facts;
- Job Intelligence owns analysis-completed or analysis-failed facts when downstream use requires them.

The ingestion workflow coordinates Job and Application operations but does not become the owner of their facts.

## Core Transaction Boundary

For a lifecycle change, the consistency unit is:

```text
BEGIN
  update job_applications
  insert application_status_history
  insert business_events
COMMIT
```

For a new tracked Application, Job resolution/refresh, Application creation, initial history, and required event records commit together. An event must not become visible as dispatchable before its associated domain mutation commits.

No AI provider, Analytics calculation, email, notification, or other network call executes within this transaction.

## Event Record Intent

Each durable event conceptually carries:

- a stable event ID;
- event type and version;
- owning aggregate/business reference;
- occurrence time;
- JSON payload sufficient for routing and safe handling;
- processing state;
- attempt/retry metadata;
- processing timestamps and failure diagnostics appropriate for operations.

Payloads are delivery contracts, not query models or copies of every aggregate field. Final names, fields, size limits, and compatibility rules are fixed with Section 3.6 implementation, not invented here.

## Conceptual Event Lifecycle

The durable record must distinguish these conceptual conditions:

| Condition | Meaning |
| --- | --- |
| Pending | Committed and eligible for an initial processing attempt |
| Processing/claimed | Temporarily owned by one worker attempt |
| Succeeded | Required handling completed successfully |
| Retryable | A transient failure occurred and a later attempt is eligible after delay/backoff |
| Failed | Retry policy is exhausted or the failure is terminal |

These labels describe required behaviour, not mandatory persisted enum names. Exact states, claim/lease fields, and transition SQL belong to Database/implementation work. Processing state must remain observable and abandoned claims must be recoverable after interruption.

## Dispatcher Behaviour

After commit, a dispatcher:

1. claims eligible pending/retryable records safely;
2. routes each record to the relevant handler;
3. records success, retry eligibility, or terminal failure;
4. releases/recovers abandoned work after process interruption;
5. limits each dispatch batch so core database use remains healthy.

Multiple backend executions or overlapping polls must not cause unsafe concurrent ownership of one attempt. The exact PostgreSQL claiming and scheduling strategy is an implementation choice validated under concurrency.

Processing separates three phases:

1. **Claim:** in a short PostgreSQL transaction, select and claim a bounded batch of eligible records so workers do not blindly process the same attempt concurrently.
2. **Process:** after the claim transaction and database lock are released, perform downstream work. External AI/network latency never holds the originating Core transaction or a long-lived event-row lock open.
3. **Record outcome:** in another short transaction, mark success, schedule a bounded retry, or record terminal failure.

The claim must remain safe across overlapping pollers and process restarts. Exact locking clauses, lease duration, batch size, scheduler, and backoff values are deliberately deferred to implementation and measured operations.

## Delivery and Ordering Semantics

Delivery is at-least-once in effect: a handler may see the same event again after failure or ambiguous completion. Handlers must tolerate duplicate delivery.

Sprint 2 does not claim global ordering or exactly-once execution. Where lifecycle ordering matters, the handler uses authoritative aggregate/history data, occurrence/version information, or state-based projection rules rather than trusting arrival order alone.

## Retry and Failure

Retry is bounded. Retryable transient failures use delayed attempts with persisted attempt metadata. Permanent validation failures and exhausted retries become visible terminal failures requiring operational review or explicit recovery.

Sprint 2 does not require a separate dead-letter-queue platform. Terminal state in PostgreSQL provides durable evidence without introducing another broker.

A failed Job Intelligence or Analytics handler never rolls back an already-committed Job/Application mutation. Recovery resumes from durable event state or rebuilds a derived projection from authoritative Core data.

## Handler Idempotency

Handlers prefer state/upsert semantics over blind increments or append-on-every-delivery behaviour.

- Job Intelligence prevents one event retry from creating uncontrolled duplicate attempts.
- Analytics converges one Application projection to authoritative lifecycle facts.
- Successfully applied event identity may be recorded where natural state convergence alone is insufficient.

Idempotency is verified per handler; the dispatcher does not make an exactly-once guarantee.

## Downstream Event Context

### Job Intelligence

A Job Intelligence trigger identifies the persisted Job and includes only stable routing/processing context. It does not copy the full Job description into `business_events`. The handler loads the required persisted Job snapshot, then calls the AI provider outside event-claim and Core transactions.

The event does not include resume tailoring, match scoring, Cover Letter generation, or other capabilities outside Sprint 2.

### Analytics

Application lifecycle facts identify the affected Application and include only context needed for stable handling. The Analytics handler may reload authoritative Application/status-history data and converge the PostgreSQL projection idempotently.

Job Intelligence and Analytics share durable event infrastructure, not a separate Analytics bus and not shared domain ownership.

## Processor Separation

Job Intelligence and Analytics may reuse shared dispatch infrastructure, but remain separate domain processors:

- Job Intelligence loads a persisted Job snapshot and performs semantic analysis;
- Analytics updates/rebuilds user-scoped Application projections.

Sharing event delivery does not merge their ownership, data models, failure policies, or business logic.

## Observability and Recovery

Operations must be able to determine:

- pending, processing, retryable, and failed event counts;
- event age and processing lag;
- attempt count and last failure category;
- whether Job Intelligence or Analytics is degraded;
- whether core writes continue successfully.

Recovery operations must be bounded and auditable. Exact metrics, log fields, administrative commands, and alert thresholds remain implementation/operations decisions.

## Section 3.6 Non-Goals

- Kafka, RabbitMQ, Redis queues, or distributed streaming
- Event sourcing or a CQRS framework
- Exactly-once claims or global ordering
- Infinite retry or mandatory external DLQ infrastructure
- External work inside core transactions
- Concrete event classes, payloads, polling SQL, or schedules

## Related Documents

- [Sprint 2 Design Index](README.md)
- [Backend / Service Design](backend-service-design.md)
- [Database Design](database-design.md)
- [Redis Design](redis-design.md)
- [Job Intelligence Design](job-intelligence-design.md)
- [Analytics Design](analytics-design.md)
- [Sprint 2 Architecture Design](../../architecture/sprint-2-architecture-design.md)
