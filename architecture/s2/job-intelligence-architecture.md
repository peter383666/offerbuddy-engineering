# Job Intelligence Architecture

## Responsibility

Job Intelligence owns asynchronous semantic understanding of a persisted Job description.

Outputs:

- concise factual summary
- responsibilities
- requirements
- skills

It does not own page capture, ownership, Application lifecycle, or Analytics.

## Boundary

```text
Core Job/Application commit
  -> Business Event
  -> Job Intelligence processor
  -> validated structured persistence
```

Missing description or provider failure yields controlled non-success states (including unavailable/failed) without invalidating Core data.

## Related

- [Job Intelligence Design](../../design/s2/technical/job-intelligence-design.md)
- [ADR-003](../../decisions/ADR-003-ai-assisted-job-extraction.md)
- [ADR-004](../../decisions/ADR-004-ai-provider-abstraction.md)
