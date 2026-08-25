# Sprint 2 Release Quality Gate

## Purpose

Defines when a Sprint 2 documentation/code candidate is ready for engineering release tagging after merge to `main`.

## Required

- [x] Implementation Status and Reconciliation reviewed
- [x] Living architecture synced with implemented boundaries
- [x] Backend CI green on the candidate application SHA
- [x] Frontend CI green on the candidate application SHA
- [x] Extension `npm run verify` executed for the candidate extension revision
- [x] Regression checklist completed for S1 + S2 critical paths
- [x] Migration impact understood (Flyway V3–V9 already in S2 baseline; later migrations called out)
- [x] No secrets committed in application or engineering repos
- [x] Known limitations and deferred items recorded
- [x] Engineering docs PR `docs/sprint-2` → `main` reviewed and merged
- [x] Tag `engineering-s2` created on merged `main` (documentation snapshot)

Post-tag additions (not blocking the original tag):

- [x] Chrome Web Store listing recorded
- [x] Sprint-scoped `ADR-S2-*` extracted
- [ ] Extension CI + Frontend Vitest gate merged to application `main`

## Explicitly Not Sufficient Alone

- A single green CI job
- Homepage HTTPS reachability
- Design documents without implementation reconciliation

## Related

- [Regression Checklist](regression-checklist.md)
- [Documentation Governance](../../operations/documentation-governance.md)
- [Definition of Done](../definition-of-done.md)
