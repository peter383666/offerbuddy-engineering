# Sprint 2 Release Quality Gate

## Purpose

Defines when a Sprint 2 documentation/code candidate is ready for engineering release tagging after merge to `main`.

## Required

- [ ] Implementation Status and Reconciliation reviewed
- [ ] Living architecture synced with implemented boundaries
- [ ] Backend CI green on the candidate application SHA
- [ ] Frontend CI green on the candidate application SHA
- [ ] Extension `npm run verify` executed for the candidate extension revision
- [ ] Regression checklist completed for S1 + S2 critical paths
- [ ] Migration impact understood (Flyway V3–V9 already in S2 baseline; later migrations called out)
- [ ] No secrets committed in application or engineering repos
- [ ] Known limitations and deferred items recorded
- [ ] Engineering docs PR `docs/sprint-2` → `main` reviewed and merged
- [ ] Tag `engineering-s2` created on merged `main` (documentation snapshot)

## Explicitly Not Sufficient Alone

- A single green CI job
- Homepage HTTPS reachability
- Design documents without implementation reconciliation

## Related

- [Regression Checklist](regression-checklist.md)
- [Documentation Governance](../../operations/documentation-governance.md)
- [Definition of Done](../definition-of-done.md)
