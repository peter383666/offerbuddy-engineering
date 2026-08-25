# Sprint 3 Preparation, Job Intelligence, and Match Technical Design

## Status and Scope

This document publishes the frozen Phase 3 design for the Sprint 3 Job Intelligence delta, persistent Preparation context, and explainable Match Analysis. Candidate, Job, Preparation, Match, Application, and AI ownership remain as defined by the [Sprint 3 Architecture](../../../architecture/s3/architecture-overview.md).

## Job Intelligence Delta

Sprint 3 does not change the core responsibility of Sprint 2 Job Intelligence. It remains Job-owned asynchronous semantic enrichment of persisted Job content and continues to produce summary, responsibilities, requirements, and skills.

Sprint 3 adds source identity and projection requirements:

- Job has a monotonic `contentVersion` distinct from timestamps and Intelligence versions;
- each Intelligence attempt/result records the Job content version it analysed;
- Preparation consumes a stable Job/Intelligence projection, not Job persistence entities;
- a result for an older Job content version may remain available as stale history but cannot be presented as current silently;
- refreshing Intelligence does not mutate Candidate, Match, artefacts, or Application directly;
- existing S2 Job capture and Application recording do not wait for S3 analysis.

Job content changes only when accepted canonical source facts change. Repeated capture with equivalent content must not advance `contentVersion` merely because it was observed again.

## Preparation Ownership

Preparation is a user-owned context anchored by Candidate + Job. It composes current source projections and links derived capabilities; it does not own Candidate facts or canonical Job facts.

A user has at most the frozen logical Preparation identity for a Candidate/Job pair. Repeating **Prepare with OfferBuddy** returns/reuses that context rather than generating another workspace or creating an Application.

Preparation records or exposes:

- stable Preparation, Candidate, and Job identities;
- owning user identity derived by the backend;
- the source versions used by derived outcomes;
- readiness/missing-input information;
- current accepted Match and artefact heads/references;
- operation state required for refresh-safe UI;
- optional association visible from an Application after the user applies.

Preparation is not a checklist aggregate, completion percentage, Saved Job, or AI task container. Match, Resume, and Cover Letter remain sibling capabilities with their own lifecycle records.

## Preparation Composition

The Preparation query service composes authorised immutable projections:

```text
Candidate query contract
        +
Job / Job Intelligence query contract
        +
Match and artefact heads
        ↓
Preparation read model/API response
```

It never joins another module by reaching into its repository. Missing or stale downstream data is represented explicitly. The page may load useful persisted context while optional operations are unavailable or in progress.

Readiness is capability-specific. For example, Match may require an accepted Candidate and usable Job Intelligence; artefact generation additionally requires the approved selected Role/Base Resume and source context. The backend returns actionable reasons instead of one universal readiness percentage.

## Match Inputs and Identity

A Match generation request is identified by the Preparation and frozen idempotency/generation identity. Its source tuple includes at least:

- Candidate identity and `profileVersion`;
- Job identity and `contentVersion`;
- Job Intelligence source/result version;
- Match capability/prompt/runtime definition version required for provenance.

Provider/model selection is resolved by the AI runtime at execution time and recorded on the attempt/result. It is not supplied by the user or embedded as business intent in the requested event.

Two requests with the same accepted idempotency identity converge. A deliberate regeneration creates a new generation attempt/result under the frozen rules; it does not mutate an older accepted result.

## Explainable Match Model

Match is structured derived analysis, not free-form advice. The frozen result separates:

- overall explanation and optional secondary score/category;
- strong matches with Candidate evidence and Job requirement references;
- partial matches with evidence and limitations;
- gaps/unsupported requirements without invented capability;
- critical or important requirements;
- eligibility observations based on explicit facts;
- source/provenance and freshness metadata.

Every positive Candidate claim must trace to the supplied Candidate evidence. Every Job claim must trace to the supplied Job/Intelligence context. “No evidence” is represented as unknown/gap and cannot be upgraded to a claim by provider fluency.

Match does not predict hiring, guarantee ATS passage, decide whether the user applies, modify Candidate, or write Job Intelligence.

## Eligibility Semantics

Deterministic eligibility facts and sponsorship reference signals are distinct from AI Match analysis:

- Candidate work eligibility comes from explicit Candidate data;
- Job requirements come from persisted Job/Intelligence evidence;
- Sponsor signal means only that the employer appears in the active published reference dataset;
- Sponsor signal does not guarantee sponsorship for the Job;
- Match may explain aligned, unknown, or conflicting evidence but cannot infer citizenship, residency, clearance, or working rights.

Critical deterministic signals are presented separately and with higher authority than probabilistic AI explanation where the approved UI specifies it.

## Async Lifecycle

Match uses a durable domain generation/operation record plus the existing `business_events` transport:

```text
authorised request transaction
→ create/reuse Match attempt and requested event
→ worker claims event
→ load immutable source projections
→ execute semantic AI port outside transaction
→ validate and persist result/state
→ publish completion event where required
```

The user-visible attempt state is durable and refresh-safe. Event state is transport state; it is not the Match API resource. Workers check capability enablement and current source identity at the frozen points, use bounded retry, and never hold database transactions across provider calls.

Transient provider/infrastructure failure may retry. Invalid structured output, unsupported/missing inputs, or exhausted retries become safe terminal states. A failed new attempt does not delete the last accepted Match.

## Freshness and Regeneration

Match freshness compares its recorded source tuple with current Candidate/Job/Intelligence versions. Later completion time does not make an older-source result current.

When source data changes:

- the existing result may be shown as potentially stale with the approved action;
- the system does not cascade automatic regeneration;
- the user may deliberately generate a new result;
- concurrent completions cannot move the current head backwards to an older source tuple;
- downstream artefacts retain the exact Match/source identity they consumed.

## Application Integration

Application Detail reads an additive Preparation summary through an application-facing contract. Application does not query Preparation repositories or own Match/artefact state.

Creating or updating Preparation does not create/update Application status. When a user later applies, existing Application tracking remains authoritative. Submitted-material references/snapshots may be associated according to the frozen Application contract without making Preparation the Application aggregate root.

## Verification Boundary

Implementation evidence must cover:

- equivalent Job recapture versus genuine `contentVersion` change;
- Job Intelligence result/source-version provenance and stale behaviour;
- server-derived ownership and Preparation create-or-reuse concurrency;
- capability-specific readiness and partial page availability;
- Match input minimisation, evidence grounding, structured validation and safe errors;
- request idempotency, deliberate regeneration and current-head ordering;
- worker retry/restart and no external call inside a database transaction;
- Candidate/Job changes producing visible staleness without automatic regeneration;
- failed attempts preserving the last accepted result;
- Application recording and the Sprint 2 Fast Path succeeding independently.

Exact endpoints, DTOs, error codes, polling fields, and idempotency semantics are defined by the frozen `api-contract.md` after publication.
