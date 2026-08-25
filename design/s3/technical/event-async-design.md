# Sprint 3 Event and Asynchronous Processing Technical Design

## Status and Scope

Sprint 3 extends the existing PostgreSQL-backed `business_events` mechanism. It does not add a broker, distributed workflow platform, exactly-once claim, or generic AI task service.

## Event Boundary

A business event records a completed fact or a durable request for decoupled work. It is not:

- a replacement for synchronous validation/commands;
- the user-visible Match/import/generation resource;
- a container for private source content;
- an ordering authority for domain correctness;
- an AI provider job definition.

The module that owns the fact/request owns the event type, versioned payload contract and producer semantics. Event infrastructure owns persistence, claim/lease, dispatch, retry and recovery. Consumers own idempotent handling and domain outcomes.

## Publication and Transaction Boundary

Where a domain write requires downstream work, its state change and event row are committed in the same short PostgreSQL transaction. No provider, parser, renderer, object-store, network, or long computation occurs inside that transaction.

After commit, the processor claims eligible rows using the existing bounded lease model. Publishing success means the event is durable, not that downstream processing has completed.

Payloads contain stable resource/source identities, event schema version and minimum routing facts. They exclude resume text, Candidate content, Cover Letter content, prompts, responses, credentials and rendered files.

## Domain Operation Versus Transport State

Resume Import, Match, Resume generation, Cover Letter generation, rendering and Sponsor publication/projection maintain domain-owned operation/state records. The API polls those resources, not `business_events`.

```text
domain attempt PENDING
  + requested business event
→ event claimed/retried/completed
→ domain attempt SUCCEEDED or FAILED
```

Event terminal state and domain terminal state must converge safely but remain distinct. A completion event, where needed, is produced only after the accepted domain outcome is durable.

## Claim and Processing Model

The existing processor continues to use:

- eligible/pending selection;
- atomic bounded claim and owner/lease metadata;
- work outside the claim transaction;
- short outcome transaction;
- abandoned-claim recovery;
- bounded retry and visible terminal failure;
- multiple-worker-safe claiming without long locks.

A worker may receive the same event more than once. Correctness comes from stable domain identity, database constraints, idempotent state transitions and source/version checks—not an exactly-once assumption.

## Event Catalogue Rules

Sprint 3 event groups include only those required by frozen capability flows:

- Candidate import processing requested/completed where downstream notification is required;
- Job/Job Intelligence source-version-aware processing;
- Match generation requested/completed;
- Tailored Resume generation requested/completed;
- Focused Cover Letter generation requested/completed;
- rendering requested/completed where asynchronous;
- Sponsor publication/active-projection refresh where decoupling is required;
- retained S2 Application/Analytics events without changed Fast Path semantics.

Candidate field updates do not publish private field values. AI provider fallback, token/cost telemetry, and every runtime configuration edit do not create business events by default. They are execution metadata/audit concerns unless a frozen downstream business reaction explicitly requires an event.

Application creation does not automatically start Preparation. Preparation completion does not update Application status or create an Application-prepared event merely for convenience.

## Requested-event Source Identity

Every requested event fixes the business source identity needed for safe execution, such as Candidate `profileVersion`, Job `contentVersion`, selected Resume revision and generation/attempt ID. A worker must not silently substitute newer mutable source state and claim it fulfilled the older request.

If the frozen capability permits cancellation/supersession, the worker transitions the old attempt accordingly. Otherwise it executes against the recorded source or fails safely. Late completion cannot advance a current head contrary to consistency rules.

Provider/model is resolved from runtime policy when execution starts and recorded on the attempt; it is not a business source identity embedded into requested events.

## Ordering and Idempotency

Consumers may observe duplicates and different event types out of order. Therefore:

- handlers check the domain attempt/current state before side effects;
- accepted transitions use expected state/version and uniqueness constraints;
- completion order never determines source freshness;
- events cannot overwrite newer accepted/user-edited heads;
- external side effects use stable idempotency identities where supported;
- replay after restart converges on the durable domain outcome;
- consumers do not create unbounded event cascades.

Success is established by the committed domain state. If work committed but event completion marking failed, redelivery recognises the accepted domain result and completes without repeating the business outcome.

## Retry Classification

Retry is bounded and classified:

| Class | Behaviour |
| --- | --- |
| transient provider/network/storage condition | retry with backoff within frozen attempt limits |
| worker interruption/expired claim | reclaim and resume/converge from durable state |
| invalid input or structured output | terminal safe failure unless a new explicit request changes input |
| capability disabled/misconfigured | distinct disabled/configuration outcome; no infinite retry |
| stale/superseded source | frozen superseded/stale transition or safe terminal outcome |
| authorisation/ownership failure | terminal denial; never retry into access |

Retry does not mean regenerate with a new business identity. A user-requested regeneration creates a new attempt under the API/idempotency contract.

## Correlation and Diagnostics

New S3 events persist the request correlation ID introduced by V10. Workers restore/propagate safe diagnostic context through retry and AI execution. A retry keeps the same causal correlation while using distinct attempt/claim metadata where required.

Logs and monitoring expose event type/version, resource/attempt identity, safe state, claim age, retry count, duration and error category. They exclude event private content and raw exceptions that reveal source data or secrets.

## Failure Isolation

- downstream failure never rolls back already-committed Core Application/Candidate/Preparation acceptance;
- one capability's failure does not block unrelated handlers;
- exhausted work remains visible and recoverable through the frozen operational path;
- the last accepted Match/artefact/published Sponsor dataset remains usable;
- processor unavailability delays optional outcomes but does not corrupt source state;
- event-table growth/lag is observable and operational retention follows frozen policy.

## Verification Boundary

Tests must prove atomic source/event persistence, safe concurrent claiming, lease expiry/recovery, duplicate delivery, out-of-order events, outcome-commit/event-mark failure, bounded retry/exhaustion, capability disablement, old-source completion, correlation propagation, private-payload exclusion, and preserved Sprint 2 Application/Analytics behaviour.

The database design controls persistence changes; the API contract controls domain-operation polling; observability design controls safe operational projections.
