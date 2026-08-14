# Development Workflow

## Purpose

This document describes how OfferBuddy development work moves from an issue to a production-verified release.

## Delivery flow

```text
Issue
  ↓
feature/*  (or chore/*, review/*)
  ↓
Pull Request → release
  ↓
Path-filtered CI (backend-ci and/or frontend-ci)
  ↓
Merge to release
  ↓
Deploy selected commit (frontend and/or backend independently)
  ↓
Production smoke test
  ↓
Merge release → main
  ↓
Tag release (for example v0.2.0)
  ↓
Sprint review / retrospective / documentation update
```

## Branch meanings

| Branch pattern | Role |
| --- | --- |
| `feature/*` | Product or engineering feature work |
| `chore/*` | Tooling, CI, dependency hygiene |
| `review/*` | Time-boxed review / hardening branches |
| `release` | Production candidate under verification |
| `main` | Production-verified stable history |

## Pull requests

- Prefer one issue → one branch → one pull request.
- Target `release` for work intended for the next production deployment.
- Target `main` only for hotfixes or for merging an already-verified `release`.
- CI must pass for the paths touched by the change.

## CI expectations

- Backend changes run `backend-ci` (`mvn clean verify`).
- Frontend changes run `frontend-ci` (`npm ci`, lint, build).
- Unrelated paths must not trigger the other pipeline.
- CI never deploys.

Details: [Deployment Strategy](deployment-strategy.md) and the source repository `.github/workflows/README.md`.

## Local development

1. Start infrastructure with Docker Compose (Postgres; Redis if needed locally).
2. Run backend and frontend on the host for day-to-day development.
3. Keep secrets out of Git; use environment variables or local untracked files.
4. Do not edit applied Flyway migrations; add a new migration when the schema must change.

## Definition of Done

Work is not done until it meets the project [Definition of Done](../quality/definition-of-done.md), including CI and documentation updates when behaviour or operations change.
