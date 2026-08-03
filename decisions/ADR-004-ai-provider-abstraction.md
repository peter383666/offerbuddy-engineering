# ADR-004: Abstract the AI Provider Behind an Application-Owned Interface

## Status

Accepted

## Date

3 August 2026

## Context

OfferBuddy requires an external AI provider for structured job information extraction.

Provider APIs, models, pricing, structured-output behaviour, and availability may change. Directly placing provider-specific code inside the job or application modules would make the core workflow difficult to test and expensive to change.

## Decision

OfferBuddy will define an application-owned interface for job information extraction.

Provider-specific implementations will be placed in the integration layer. The parsing workflow will depend on the application interface rather than directly on a provider SDK.

## Example

```java
public interface JobInformationExtractor {

    JobExtractionResult extract(JobContent content);
}
```

Possible implementations may include:

```text
OpenAiJobInformationExtractor
GeminiJobInformationExtractor
StubJobInformationExtractor
```

## Consequences

### Positive

- AI providers can be changed more easily.
- Core business logic is not coupled to one SDK.
- Tests can use a stub or fake implementation.
- Provider-specific failures remain isolated.
- Future fallback providers are easier to introduce.

### Negative

- Introduces an additional abstraction.
- Provider capabilities may not map perfectly to one common interface.
- The internal extraction result schema must be maintained carefully.

## Implementation Guidance

- Provider DTOs must not leak into the business modules.
- Provider responses must be mapped into internal models.
- Provider-specific configuration must remain in the integration layer.
- Retry, timeout, and rate-limit handling should remain close to the provider implementation.
- Business validation should remain in the parsing workflow.
