# ADR-S2-001: Browser Extension as Primary Ingestion

## Status

Accepted — implemented in Sprint 2.

## Context

Sprint 1 preferred server-side page retrieval plus AI extraction. Supported recruitment sites can block that path even when the user can see the full Job. Copying URLs and waiting for parse adds friction during normal search.

## Decision

Make the Chrome Manifest V3 Browser Extension the preferred Sprint 2 Job capture client for SEEK and Indeed. Site Adapters isolate platform DOM behaviour; the backend remains authoritative for validation, Job resolution, Application rules, ownership, and persistence. Manual and AI URL-prefill New Application remain secondary fallbacks.

## Alternatives

- Continue server-side acquisition as the primary path
- Embed capture only in the Web app without an Extension
- Push business rules into the Extension

## Consequences

- Capture uses facts already visible in the user's browser
- OfferBuddy gains a separate client artifact, store distribution, and adapter maintenance surface
- Cross-sprint detail: [ADR-009](../ADR-009-browser-extension-site-adapters.md)

## Related

- [Browser Extension Architecture](../../architecture/s2/browser-extension-architecture.md)
- [Extension Design](../../design/s2/technical/extension-design.md)
