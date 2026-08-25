# Sprint 3 Concurrency, Idempotency, and Consistency Design

## Status and Scope

This document publishes the frozen Phase 3 rules that make duplicate requests, concurrent edits, asynchronous completion, retry, regeneration and source changes converge without lost work or false freshness.

## Distinct Identities

Do not reuse one mechanism for different problems:

| Mechanism | Protects |
| --- | --- |
| Aggregate version/OCC | Concurrent accepted writes to Candidate, Application, configuration or artefact head/content |
| Idempotency key | Repeated submission of one logical client/business command |
| Domain operation/generation ID | Durable lifecycle of one async requested outcome |
| Event ID | Transport delivery/claim identity |
| Source-version tuple | Provenance and freshness of derived output |
| Correlation ID | Diagnostics across process boundaries |
| Database uniqueness | Final convergence invariant under races |

An idempotency key does not authorise access. A correlation ID does not deduplicate. A timestamp does not replace an aggregate/source version.

## Optimistic Concurrency Control

Mutable aggregates use their frozen expected version. The backend checks ownership and version in the same accepted write boundary. On mismatch it returns the frozen conflict and does not merge or overwrite silently.

OCC applies independently to:

- Candidate `profileVersion`;
- Application `version` where added by V10;
- AI runtime configuration version;
- Sponsor working/publication control state where frozen;
- Role/Base Resume or artefact limited-edit version/revision head.

Generated revisions remain historical immutable records. Limited editing advances the frozen artefact revision/current-head outcome; it does not mutate accepted history. Clients preserve local input on conflict and use the approved reload/reconcile action.

## Request Idempotency

Idempotency scope includes authenticated owner, operation/capability, target resource and key. The stored request fingerprint prevents the same key from being reused with materially different input.

Concurrent first submissions converge through a uniqueness constraint. A repeated accepted request returns the existing operation/outcome under the API contract. Terminal failure/retry/regeneration semantics remain explicit; the same idempotency key cannot be used to disguise a new deliberate generation.

Import upload, draft apply, Preparation create/reuse, Match generation, artefact generation, rendering and Sponsor Publish each use their frozen scope rather than a generic universal interceptor.

## Async Consumer Idempotency

Every handler assumes duplicate delivery and restart after any boundary. It checks the domain attempt/state and uses expected-state updates/uniqueness before external or accepted side effects.

The critical recovery case is:

```text
domain outcome committed
→ event completion marking interrupted
→ event redelivered
→ handler recognises durable outcome
→ no duplicate business result
```

Where an external provider/storage API supports idempotency, use the stable attempt/render identity. Database state remains the final authority.

## Source Consistency and Freshness

Derived Match/artefact attempts record the exact Candidate, Job, Intelligence, Role/Base Resume and relevant Match versions they consumed. Current source is read through owning-module projections before execution and rechecked at the frozen commit/head boundary where required.

Rules:

- event arrival or completion order does not define freshness;
- later completion for older sources cannot replace a newer/current head;
- source change does not rewrite or delete old output;
- stale output remains historical/visible where approved;
- regeneration is explicit and creates a new attempt;
- Candidate change does not cascade automatic Match/resume/letter generation;
- provider/model/prompt changes do not recalculate history automatically;
- user edits are protected from late AI completion.

## Create-or-reuse Races

Preparation and existing Job/Application flows use service-level lookup plus database uniqueness as final protection. When two transactions race, the loser resolves the winning row and returns the frozen already-existing/reused outcome instead of surfacing an internal constraint error or resetting existing state.

Sponsor Publish uses a single authorised transition/lock or expected-version condition so two publish attempts cannot create ambiguous active versions. Activation points to one complete immutable published dataset.

## Transaction Boundaries

Transactions cover one coherent owning-domain state transition and its required event/outcome metadata. They do not span AI calls, file parsing, rendering, object storage, Redis refresh or browser interaction.

Cross-module orchestration obtains immutable projections before long work and records their versions. It must not hold locks across module/external calls. Accepted completion uses short conditional updates to prevent stale/duplicate head advancement.

## Failure and Retry

Automatic retry keeps the same operation identity and source tuple. A new user request/regeneration uses a new identity. Permanent validation/ownership/conflict failures do not retry. Bounded transient retry cannot cause multiple accepted outcomes.

Failed newer work does not remove the last accepted result. Partial object/render/provider success is reconciled through durable state and cleanup rules rather than reported as an accepted artefact prematurely.

## Verification Boundary

Tests must cover concurrent Profile/config/edit/Publish updates, duplicate create/apply/generate/render requests, key fingerprint mismatch, duplicate/out-of-order event delivery, crash between outcome and event marking, source changes during work, late older completion, user edit versus worker completion, create-or-reuse races, and last-known-good preservation.

Exact status/error/idempotency response semantics are controlled by the frozen API contract; database constraints are controlled by database design.
