# ADR-S2-003: Asynchronous Job Intelligence

## Status

Accepted — implemented in Sprint 2.

## Context

Semantic enrichment (summary, responsibilities, requirements, skills) is useful but slow and failure-prone relative to Application creation. Core capture must not wait on an external AI provider.

## Decision

Run Job Intelligence asynchronously after Core Application/Job persistence. Publish Business Events from the Core write path; Intelligence reacts downstream. Successful tracking remains valid if Intelligence lags, fails, or returns an unavailable outcome when description content is missing. AI stays backend-only and does not determine page facts or ownership.

## Alternatives

- Synchronous Intelligence on Track / save
- Client-side AI enrichment in the Extension
- Defer all Intelligence past Sprint 2

## Consequences

- Core write path stays small and reliable
- Detail UI must expose pending / populated / failed / unavailable states
- Provider cost and latency stay off the capture critical path

## Related

- [Job Intelligence Architecture](../../architecture/s2/job-intelligence-architecture.md)
- [Job Intelligence Design](../../design/s2/technical/job-intelligence-design.md)
- [ADR-S2-004](ADR-S2-004-business-events-without-broker.md)
