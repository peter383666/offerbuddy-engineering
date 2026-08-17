# Definition of Done

## Purpose

This document defines the minimum completion standard for OfferBuddy changes. A passing CI job is necessary where applicable but is not the whole release decision.

## Change Completion

A change is done when:

- acceptance criteria are satisfied
- implementation and documentation agree
- relevant backend tests pass
- frontend lint and production build pass when frontend code changes
- database changes use a new Flyway migration
- security, ownership, and error behaviour are considered
- no secrets or production credentials are committed
- the pull request is reviewed and merged through the intended branch
- unresolved limitations are recorded as technical debt or backlog work

## Release-Candidate Completion

A release candidate is ready for promotion when:

- required frontend/backend CI checks pass on `release`
- immutable frontend and backend artifacts exist for the candidate SHA
- integration behaviour is verified
- production configuration and migration impact are understood
- the candidate SHA is explicitly identified
- rollback to a known-good SHA is available
- documentation required by the increment is current

Code on `release` is a candidate under verification, not automatically production-ready.

## Production Completion

A deployment is complete when:

- the intended explicit SHA/version was deployed
- backend health is `UP`
- the production homepage is reachable over HTTPS
- critical authentication and application smoke checks pass
- Nginx and application logs show no new critical error
- persistence behaviour is intact
- the deployed version is recorded
- rollback remains possible

Application rollback and database rollback are assessed separately.

## Exceptions

If a criterion does not apply, the pull request or release record should state why. A criterion is not silently ignored merely because CI is green.
