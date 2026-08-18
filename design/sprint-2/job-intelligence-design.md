# Sprint 2 Job Intelligence Design

## Document Status

| Item | Value |
| --- | --- |
| Phase | Phase 3 — Technical Design |
| Section | 3.7 — AI Job Intelligence |
| Status | Completed and approved |
| Implementation status | Target design; Sprint 1 synchronous URL parsing remains the current implementation |

## Purpose

This document defines the downstream AI Job Intelligence workflow, contracts, validation, persistence, retry, and failure isolation. It does not publish production prompts, provider credentials, concrete classes, or provider-specific configuration.

## Responsibility Boundary

Job Intelligence owns semantic understanding of a persisted Job description. Sprint 2 outputs are:

- responsibilities;
- requirements;
- skills;
- a concise factual Job summary.

It does not own Job identity, browser extraction, authentication, Application ownership/lifecycle, Analytics, match scoring, resume tailoring, or Cover Letter generation.

Job Intelligence is Job-owned and shared. It does not receive `user_id` ownership merely because one user caused the Job to be captured.

Eligibility screening and business warnings remain deterministic Extension/Backend responsibilities where reliable facts or text rules support them. Job Intelligence may expose relevant JD content, but AI is not authoritative for citizenship, permanent residency, security-clearance, working-rights, or sponsorship eligibility.

## Processing Flow

```mermaid
flowchart LR
    Core["Core Job/Application commit"] --> Event["Durable Business Event"]
    Event --> Processor["Job Intelligence processor"]
    Processor --> Snapshot["Load persisted Job snapshot"]
    Snapshot --> AI["Backend-owned AI provider abstraction"]
    AI --> Validate["Validate and normalise structured output"]
    Validate --> Persist["Persist analysis attempt and semantic results"]
```

The persisted Job description is the analysis source snapshot. Normal processing does not re-browse or re-scrape the external Job page.

## Input Contract

The processor loads the smallest sufficient persisted Job snapshot, primarily:

- Job identity/reference;
- title;
- company;
- description text;
- analysis/version context needed to prevent stale or duplicate work.

Missing or unusable description produces a controlled non-success outcome; it does not corrupt Core Job/Application state.

## Output Contract and Validation

Provider output is mapped into an OfferBuddy-owned structured model. Before persistence, processing verifies:

- the response is structurally valid;
- required collections/fields have supported shapes and bounds;
- blank, duplicate, or malformed items are normalised or rejected consistently;
- factual summary content remains concise and grounded in the supplied snapshot;
- responsibilities, requirements, and skills are supported by the supplied description rather than invented to fill missing content;
- provider-specific DTOs do not leak into domain persistence.

The public repository describes this contract but does not contain the full production prompt.

## Attempt and Version Semantics

Before an analysis root exists, the Job is conceptually **not yet analysed**. Each created processing attempt then has a durable `job_analysis` root with a state such as `PENDING`, `PROCESSING`, `SUCCEEDED`, or `FAILED`.

Event retry metadata distinguishes a transient failed attempt that remains retry-eligible from a terminal failure whose bounded policy is exhausted. These concepts do not become Application statuses and do not change Application lifecycle state.

Retry/re-analysis creates a new attempt/version instead of overwriting historical attempts. Only a successfully validated attempt publishes usable structured responsibilities, requirements, skills, and summary.

The exact rule for selecting the current successful analysis and detecting stale Job snapshots is fixed with persistence implementation, without deleting earlier evidence.

## Transaction Boundary

The external provider call occurs after the Core transaction and without holding that transaction open.

```text
claim analysis work
  -> load snapshot
  -> call provider outside core transaction
  -> validate result
  -> short result-persistence transaction
```

Application tracking succeeds even when AI is unavailable, slow, malformed, rate-limited, or exhausted after retry.

## Retry, Idempotency, and Failure

- Retry is bounded and reserved for failures classified as transient, such as provider unavailability, timeout, rate limiting, or transient network failure.
- Invalid structured output, malformed semantic content, and deterministic validation failures are recorded as failed attempts rather than partially persisted as success or retried forever.
- Repeated delivery of the same logical analysis request converges on the same attempt/outcome and does not append duplicate responsibilities, requirements, or skills.
- An explicit later re-analysis is a new attempt/version; it is distinct from retrying or redelivering the same logical request.
- Successfully validated child results are persisted as one coherent attempt result using replacement/upsert or equivalent uniqueness semantics selected during implementation.
- A process restart can resume/retry from durable Business Event and analysis state.
- Provider failure does not change Application status or roll back a Job/Application commit.

Exact retry counts, delays, timeouts, provider error mappings, and concurrency controls remain implementation/configuration details.

The design provides practical at-least-once tolerance at the consumer boundary. It does not claim a global exactly-once AI guarantee.

## Sprint 1 Compatibility

Sprint 1 implements synchronous server-side URL acquisition followed by AI parsing through backend-owned abstractions. That remains a secondary New Application fallback in Sprint 2.

The new Job Intelligence path reuses the architectural provider-abstraction principle, but it is a separate downstream workflow over persisted Job content. Existing synchronous parsing must not be described as if it already implements durable events, versioned analysis, or asynchronous persistence.

Sprint 1 `jobs.responsibilities` and `jobs.requirements` remain legacy compatibility fields until a safe migration. Sprint 2 structured semantic results live in Job Intelligence persistence rather than being silently written back as the new authoritative model.

## Security and Privacy

- Provider credentials remain backend-only.
- No secrets or full production prompts are published.
- Only required Job content is sent to the provider.
- User/session/Extension credentials are not part of the semantic-analysis input.
- Logs avoid provider credentials and unnecessary full Job descriptions or responses.

## Provider and Prompt Boundary

Job Intelligence depends on an OfferBuddy-owned provider abstraction rather than provider SDK types in the domain/service boundary. The abstraction exists to isolate provider configuration, transport, and error mapping; Sprint 2 does not create a general plugin ecosystem or a separate AI microservice.

Public documentation may describe prompt purpose, structured-output contract, grounding, and validation expectations. Full production prompt text, API keys, environment values, credentials, and sensitive provider configuration remain private implementation/operations material. Business correctness relies on validated output and domain rules, not prompt wording alone.

## Section 3.7 Non-Goals

- AI-based deterministic page extraction or independent browsing
- Synchronous AI dependency in Extension Track
- Match scoring, resume tailoring, Cover Letter generation, or auto-apply
- Analytics ownership or user-specific Job Intelligence copies
- Provider lock-in, published production prompts, or invented provider configuration
- AI-owned citizenship, residency, clearance, working-rights, or sponsorship decisions
- Candidate embeddings, vector search, suitability ranking, recommendations, or match percentages
- Kafka, RabbitMQ, Redis queues, workflow engines, or a separate AI microservice
- Unbounded retry or AI calls from Flyway

## Related Documents

- [Sprint 2 Design Index](README.md)
- [Database Design](database-design.md)
- [Event Design](event-design.md)
- [Extension Design](extension-design.md)
- [ADR-003 — AI-Assisted Job Extraction](../../decisions/ADR-003-ai-assisted-job-extraction.md)
- [ADR-004 — AI Provider Abstraction](../../decisions/ADR-004-ai-provider-abstraction.md)
