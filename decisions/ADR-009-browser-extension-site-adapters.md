# ADR-009: Use Browser Extension Site Adapters for Primary Sprint 2 Job Capture

## Status

Accepted for Sprint 2 architecture.

## Date

18 August 2026.

## Context

Sprint 1 uses server-side page retrieval followed by AI extraction. Production use showed that recruitment sites may block or limit server-side acquisition even when the user can see the complete Job page. Copying URLs, switching applications, waiting for parsing, and reconfirming visible facts also add friction to normal job searching.

Sprint 2 needs reliable capture from initially supported SEEK and Indeed pages without moving Application rules, ownership, or semantic analysis into a browser client. Platform DOM and navigation behaviour can change independently and must not spread across the Extension.

## Decision

Use the OfferBuddy Browser Extension as the preferred Sprint 2 Job Capture client.

The Extension will:

- capture reliable page facts visible on supported SEEK and Indeed pages;
- isolate platform-specific DOM, external identifier, and SPA/page-change behaviour behind Site Adapters;
- normalise adapter output into a common conceptual ingestion shape;
- send captured input through an authenticated OfferBuddy Backend ingestion boundary;
- display safe success, duplicate, authentication, invalid-capture, and failure outcomes.

The OfferBuddy Backend remains authoritative for validation, Job resolution and reuse, Application business rules, authenticated ownership, duplicate behaviour, and persistence.

AI remains a backend-only semantic Job Intelligence capability. It is not responsible for basic facts the Extension captures reliably.

AI URL parsing remains available as a secondary fallback rather than the preferred acquisition strategy.

## Boundaries

- The Browser Extension is a new OfferBuddy client boundary, not a new business domain or source of truth.
- Initial Site Adapters cover SEEK and Indeed only.
- LinkedIn and broad multi-platform support are outside Sprint 2.
- Exact DOM selectors, Extension structure, browser APIs, ingestion contracts, and credential mechanisms are deferred to Technical Design.
- Extension input is untrusted even though the Extension is a recognised OfferBuddy client.

## Consequences

### Positive

- Capture can use Job content already visible in the user's browser.
- Basic page facts no longer depend on AI inference.
- Site Adapters localise SEEK/Indeed changes.
- Core ownership and Application rules remain server-side.
- The common adapter output creates a controlled path for later platforms without designing them now.

### Negative

- OfferBuddy gains another client artifact, compatibility surface, and release lifecycle.
- SEEK and Indeed changes require adapter maintenance.
- Extension authentication and least-privilege browser access require separate Technical Design.
- Browser-store distribution and update behaviour add operational work.

## Alternatives Considered

### Continue server-side AI URL capture as the primary path

Rejected for Sprint 2 because it preserves the acquisition limitations and synchronous friction observed in Sprint 1. It remains a fallback.

### Put platform conditions throughout the Extension

Rejected because scattered SEEK/Indeed logic would couple page changes to unrelated Extension behaviour and make additional adapters harder to review.

### Use AI to infer all page facts

Rejected because reliable deterministic facts should not depend on probabilistic semantic interpretation.

### Support every recruitment platform in Sprint 2

Rejected because it would broaden scope and reduce the ability to validate reliable behaviour for SEEK and Indeed.

## Relationship to ADR-003

[ADR-003](ADR-003-ai-assisted-job-extraction.md) remains the accurate Sprint 1 decision and implementation history. This ADR evolves the preferred acquisition strategy for Sprint 2; it does not remove the Sprint 1 fallback.

See [Sprint 2 Architecture Design](../architecture/sprint-2-architecture-design.md).
