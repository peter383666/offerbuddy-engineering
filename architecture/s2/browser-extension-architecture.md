# Browser Extension Architecture

## Responsibility

The Browser Extension is an OfferBuddy **client**, not a system of record.

It:

- detects supported SEEK / Indeed job contexts through Site Adapters;
- captures reliable visible page facts;
- maintains current job context across dynamic navigation;
- surfaces citizenship/PR eligibility findings where reliable;
- authenticates through pairing + revocable Extension credentials;
- submits tracking requests to the Backend Track API.

It does not:

- own Application/Job business truth;
- decide duplicates or ownership;
- run Job Intelligence or Analytics;
- expose credentials to page JavaScript.

## Tracking Paths

Sprint 2 ships two client-side tracking paths that share the same Backend Track API:

1. **Explicit Save** — user confirms **Save to OfferBuddy** in the popup (UI v2 primary action).
2. **Submission-confirmation ingest** — when a Site Adapter can reliably confirm successful submission, the Extension may track automatically; when uncertain, it asks the user.

This dual-path behaviour is an accepted implementation outcome for closeout and is recorded in [Implementation Reconciliation](../../delivery/s2/implementation-reconciliation.md). Backend ownership, create-or-reuse, and `alreadyTracked` semantics remain unchanged.

## Related

- [Architecture Overview](architecture-overview.md)
- [Extension Design](../../design/s2/technical/extension-design.md)
- [Extension Application Tracking](../../design/s2/technical/extension-application-tracking.md)
- [ADR-009](../../decisions/ADR-009-browser-extension-site-adapters.md)
