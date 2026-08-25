# Sprint 2 Release Notes

## Summary

Sprint 2 delivers lower-friction job capture for OfferBuddy.

Users can record SEEK and Indeed Applications from a Chrome Manifest V3 Browser Extension, keep Core tracking independent of AI enrichment, and review Job Intelligence plus basic Application Analytics in the Web app.

## Highlights

- Browser Extension capture for SEEK and Indeed
- Web↔Extension pairing and authenticated Track API
- Explicit Save plus confirmation-aware tracking assistance
- Citizenship/PR eligibility review signalling
- Application creation source and status history
- Asynchronous Job Intelligence
- Basic Application Analytics
- Forward schema migrations V3–V9 on PostgreSQL

## Unchanged

- Google Web sign-in and server sessions
- Manual and AI URL-prefill New Application as secondary paths
- Single-host EC2 / Nginx / Docker Compose production model
- Modular monolith backend

## Not Included

- LinkedIn / broad platform support
- Auto Apply
- Resume / Cover Letter / match scoring
- Message brokers, microservices, Kubernetes
- Redis-backed application features

## Operator Notes

- Deploy frontend and backend by explicit verified SHA as before
- Ensure `GOOGLE_API_KEY` is configured where Intelligence / AI parsing should run
- Extension users install from the Chrome Web Store listing (or load a production `extension/dist` build for validation)
- Core Application writes remain valid if Intelligence or Analytics lag
- Extension store uploads go through application GitHub Actions **Extension Publish** (`upload` or `publish`); bump `manifest.json` version before each upload

## Chrome Web Store

| Item | Value |
| --- | --- |
| Listing | https://chromewebstore.google.com/detail/offerbuddy/ihdknldekiocanohajkgebmnhnneoeka |
| Item ID | `ihdknldekiocanohajkgebmnhnneoeka` |
| Published version at S2 app release | `0.1.24` |
| Application release tag | `v0.3.0` |

## Documentation Snapshot

Engineering documentation closeout is merged to `main` and tagged `engineering-s2`. Post-tag sync covers store publication outcome, Extension Publish workflow, and Sprint-scoped ADR extraction (`decisions/s2/`).

## Related

- [Sprint Review](sprint-review.md)
- [Implementation Status](implementation-status.md)
- [Known Limitations](known-limitations.md)
- [Deferred Items](deferred-items.md)
