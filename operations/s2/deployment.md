# Sprint 2 Deployment

## Purpose

Sprint 2 does not change the single-host immutable-SHA deployment model. This note records what S2 adds to operator attention.

## Unchanged Baseline

- Frontend and backend remain independently deployable SHA artifacts
- Host Nginx serves the React build and proxies API/OAuth
- Backend/PostgreSQL/Redis run under Docker Compose on EC2
- Manual deploy workflows select an explicit verified SHA

See [Deployment Strategy](../deployment-strategy.md) and [Production Runbook](../production-runbook.md).

## Sprint 2 Operator Additions

Before/after backend deploy:

1. Confirm Flyway migrations expected for the candidate SHA (S2 baseline includes V3–V9).
2. Confirm `GOOGLE_API_KEY` is present when Job Intelligence / AI URL parsing should run in that environment.
3. Smoke Extension Track path after backend deploy (pairing + save against production origin).
4. Confirm Analytics and Application Detail intelligence pages load for a signed-in user.
5. Confirm Core create still succeeds if AI is unavailable.

Frontend deploy:

1. Confirm Extension connect route and Analytics navigation are present in the built app.
2. Confirm Home extension discovery behaviour on a browser without the Extension installed.

## Admin

Admin frontend/backend have separate CI/CD and are outside the Sprint 2 product release gate.

## Related

- [Database Migrations](database-migrations.md)
- [Rollback / Recovery](rollback-recovery.md)
- [Release Quality Gate](../../quality/s2/release-quality-gate.md)
