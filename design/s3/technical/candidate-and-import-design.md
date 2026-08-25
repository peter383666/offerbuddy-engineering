# Sprint 3 Candidate Profile and Resume Import Technical Design

## Status and Scope

This document publishes the frozen Phase 3 baseline for module boundaries, Candidate Profile, and reviewed Resume Import. It defines ownership and write semantics; the frozen API contract defines exact HTTP representations.

## Module Baseline

Sprint 3 extends the existing `com.offerbuddy` modular backend. Each business module owns its entities, repositories, application services, and domain invariants. Cross-module consumers use explicit query/command contracts and immutable projections rather than repositories or persistence entities.

Candidate is an independent domain module. Preparation consumes Candidate snapshots but cannot write Candidate state. AI provider adapters cannot write Candidate state. Resume Import belongs to Candidate because its only accepted business outcome is a reviewed Candidate change.

```text
candidate
├─ api/application contracts
├─ Candidate Profile aggregate
├─ Candidate import draft lifecycle
├─ persistence adapters
└─ AI extraction port adapter boundary

preparation ──query──> candidate snapshot
AI provider ──returns structured extraction──> candidate application service
```

## Candidate Aggregate

There is one Candidate Profile per OfferBuddy user in Sprint 3. The stable Profile identity is separate from the authenticated user identifier, while the owning user identity is enforced by the backend. Multiple personas are out of scope.

The aggregate owns:

- personal and contact details required by the approved profile model;
- professional summary;
- skills;
- work experience with responsibilities and achievements;
- projects where included by the frozen contract;
- education;
- certifications;
- languages;
- work eligibility;
- audit timestamps and `profileVersion`.

Candidate contains reusable factual claims. It does not contain Job-specific Match results, generated Resume/Cover Letter state, Application lifecycle, AI execution records, or derived completeness as canonical stored truth unless the frozen persistence design explicitly requires a projection.

## Creation and Lifecycle

A Profile may be created manually or by accepting a Resume Import draft. Absence of a Profile is a valid state and must not block Sprint 2 Application recording.

Manual creation and accepted import converge on the same aggregate rules. Import extraction alone never creates or updates a Profile.

Profile completion is derived from the fields required for a particular action or surface. Sprint 3 does not establish one universal completion percentage or mandatory checklist. APIs/UI expose actionable missing information according to the frozen capability rules.

## Data Semantics

### Null, Unknown, and Empty

`null`/absent means not supplied or unknown according to the field contract. An empty collection means the user has an accepted collection with no entries. Clients and imports must not convert absence into a factual negative.

Work eligibility values are explicit structured facts, not inferred from nationality, location, employer, or AI prose. Sponsor signals belong to the Job/opportunity context and do not alter Candidate eligibility.

### Evidence and Truthfulness

Candidate facts may carry approved source/evidence metadata. AI-specific snapshots contain only the fields required for the capability and preserve stable evidence identifiers where the frozen design requires grounding.

AI extraction may propose normalised structure but cannot invent employers, dates, skills, qualifications, achievements, eligibility, or other claims. Confidence is review guidance, not truth.

### Update Semantics

Profile writes are explicit aggregate operations. Child collection updates follow the frozen replacement/identity rules; they are not generic JSON Patch. The backend validates cross-field rules such as date ranges, current-role semantics, ordering, duplicates, and required identifiers.

Every accepted Candidate change advances `profileVersion`. Updates require the expected frozen version and fail with the contract OCC response when another accepted write has won. Clients preserve user input and reload/reconcile rather than silently overwriting.

Profile version is a Candidate source revision. It is not an Application version, Job content version, import version, artefact revision, or generation attempt.

## Candidate Snapshots

The Candidate module exposes immutable application-facing projections for authorised internal use. A projection contains the stable Candidate identity, `profileVersion`, and only the facts required by the consumer.

Preparation and Match record the source `profileVersion` they consumed. AI capabilities receive a minimised AI-specific projection rather than a persistence entity or unrestricted Profile JSON. A later Profile change may make derived output stale but never triggers cascading regeneration automatically.

## Authorisation and Deletion

All Candidate access derives ownership from authenticated server identity. Client-supplied user IDs are ignored or rejected. Cross-module internal access still carries the owning context; an internal caller does not gain universal Candidate access.

Deletion/clear behaviour follows the frozen aggregate contract and must account for retained historical Application artefacts. Removing current Profile state must not rewrite historical submitted-material snapshots or make another user's data accessible.

## Resume Import Boundary

Resume Import is a controlled pipeline:

```text
authorised upload
→ file validation and private storage
→ durable async extraction
→ CandidateImportDraft
→ user review/correction
→ explicit apply/merge
→ Candidate transaction with OCC
```

The source file, extraction operation, import draft, and Candidate aggregate are distinct records with distinct lifecycles.

## Upload and Storage

The backend validates allowed format, media signature/type, filename handling, and the frozen size limit before accepting work. Files use non-public object storage through an application port. Object keys are server generated and are never trusted as ownership proof.

Parsing occurs outside the Candidate transaction. Unsupported, malformed, encrypted, oversized, or extraction-failed files produce controlled draft/operation failure and do not change Candidate state. Temporary/source-file retention and cleanup follow the frozen privacy design.

OCR is not introduced unless required by the frozen supported-format boundary. Deterministic text extraction precedes AI structuring. Normalised text is treated as sensitive Candidate data and is not written to ordinary logs or events.

## Import Draft Lifecycle

`CandidateImportDraft` is user-owned, temporary, and reviewable. Its lifecycle distinguishes at least accepted processing, processing, review-ready, failed, applied, and expired/otherwise terminal states as defined by the API contract.

A draft records:

- owning Candidate/user context;
- source file reference and safe metadata;
- processing state and bounded failure information;
- structured proposed fields/entries;
- source evidence required for review;
- Candidate `profileVersion` observed when merge planning was created;
- timestamps, expiry, and idempotency identity.

Raw provider response is not the draft contract and is not exposed to clients.

## Extraction Contract

The Candidate module owns the semantic Resume-extraction port and structured output schema. The AI platform/provider adapter executes the request but does not decide Candidate merge behaviour.

Extraction must:

- return schema-valid proposed facts and source evidence;
- retain unknown/ambiguous values instead of guessing;
- avoid destructive normalisation;
- distinguish no evidence from a negative fact;
- reject malformed or unsafe output;
- remain retryable without creating duplicate Candidate outcomes.

Provider/model/prompt metadata belongs to safe execution/provenance records and is not used as Candidate truth.

## Review and Merge Semantics

The user reviews and may correct proposed fields before apply. Apply is an explicit command, not a side effect of viewing the draft.

Merge follows field-specific frozen rules:

- scalar details are applied only when explicitly accepted;
- skills use normalised duplicate detection without fabricating proficiency;
- contact data does not overwrite accepted values merely because extraction differs;
- summary replacement is explicit;
- experience matching uses approved stable/business keys and user choice rather than a generic diff engine;
- absence from the imported resume never deletes existing Candidate data;
- ambiguous matches require review instead of automatic destructive merge.

The apply transaction locks/checks the current Candidate version, validates the reviewed command, applies one coherent aggregate change, advances `profileVersion`, marks the draft applied, and publishes any required event atomically. If the Candidate changed since the draft baseline, apply returns the frozen conflict rather than guessing a merge.

## Idempotency and Duplicate Upload

Upload/request idempotency and apply idempotency are separate:

- repeating an accepted upload request returns/reuses the same logical draft according to the API contract;
- reprocessing a failed draft does not create multiple accepted draft outcomes;
- repeating apply for an already-applied draft returns the existing result or frozen idempotent response;
- two concurrent apply attempts cannot advance Candidate twice.

Matching file bytes alone is not sufficient authority to reuse another user's draft. Duplicate detection remains user scoped and privacy safe.

## Async and Failure Behaviour

Extraction uses the existing durable event/worker architecture. The event carries stable draft/source identity and correlation metadata, not resume text. The handler claims work, performs file parsing/provider calls outside long database transactions, then commits a validated state transition with idempotency protection.

Transient infrastructure/provider failures use bounded retry. Permanent validation/format failures become visible terminal states. Retry processing reuses the draft/source identity; uploading a new file creates a new draft. Refresh/navigation cannot lose a durable accepted operation.

Failure of Resume Import is local: the existing Candidate remains usable, the Fast Path remains usable, and no partial extracted data becomes Candidate truth.

## Verification Boundary

Implementation evidence must cover:

- one Profile per user and server-derived ownership;
- manual create/update and `profileVersion` OCC;
- child validation, null/unknown semantics, and snapshot minimisation;
- supported/unsupported upload, private object access, and cleanup;
- extraction success, malformed output, timeout, retry, restart, and terminal failure;
- review corrections and field-specific non-destructive merge;
- Candidate-changed-since-draft conflict;
- duplicate upload/apply convergence;
- no Candidate mutation before explicit apply;
- no private content in events, logs, monitoring, URLs, or cross-user responses.

Exact endpoints, DTOs, status codes, error codes, idempotency headers, and polling representations are defined only by the frozen `api-contract.md` after its publication.
