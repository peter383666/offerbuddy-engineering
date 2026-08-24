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
- Extension users need the Sprint 2 Extension build loaded or published for capture features
- Core Application writes remain valid if Intelligence or Analytics lag

## Documentation Snapshot

Engineering documentation closeout lives on branch `docs/sprint-2` and is intended to merge to `main`, then tag `engineering-s2`.

## Related

- [Sprint Review](sprint-review.md)
- [Implementation Status](implementation-status.md)
- [Known Limitations](known-limitations.md)
- [Deferred Items](deferred-items.md)
