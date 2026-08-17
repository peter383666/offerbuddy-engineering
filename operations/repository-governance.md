# Repository Governance

## Purpose

This document defines the branch, review, and release responsibilities used at the end of Sprint 1.

## Branches

| Branch | Responsibility |
| --- | --- |
| `feature/*` | Focused development work |
| `release` | Integration and release-candidate verification |
| `main` | Verified stable baseline |

Direct production deployment is not inferred solely from a push to one of these branches.

## Pull Requests

- Feature work normally targets `release`.
- Promotion from `release` to `main` occurs after integration verification.
- Pull requests should remain focused and reviewable.
- Production code, documentation, migration, and configuration effects must be visible.
- Unrelated cleanup should not be hidden inside feature or documentation work.

## Required Checks

Path-filtered backend and frontend CI validate relevant pull requests to `release` and `main`.

Branch protection should account for path filtering so an intentionally skipped workflow does not block an unrelated change. Required checks should reflect the files and risks changed.

A green check confirms the configured automation only; reviewers still assess architecture, security, migrations, operations, and documentation.

## Releases

The final release commit is promoted to `main` and marked with an immutable tag after release verification and sprint closure.

The tag:

- identifies the released source
- must not move
- should correspond to documented release notes
- does not automatically prove that production deployed the intended artifact

Production deployment records the explicit frontend/backend SHA or version actually deployed.

## Deployment Authority

Production deployment is manual through GitHub Actions and may be protected by the GitHub `production` Environment.

Operators should:

- select a verified full SHA
- confirm the matching immutable artifact exists
- observe deployment health checks
- perform the required smoke test
- retain a known-good rollback SHA

## Documentation Repositories

The source repository is authoritative for running code and production configuration. This engineering repository explains product, architecture, quality, operations, and delivery decisions.

When they conflict, current production-relevant source/configuration wins and the documentation must be corrected.
