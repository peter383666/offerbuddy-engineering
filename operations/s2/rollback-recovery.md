# Sprint 2 Rollback and Recovery

## Purpose

Extends the Sprint 1 rollback mindset for Sprint 2 artifacts and data.

## Application Artifact Rollback

Frontend and backend still roll back by redeploying a known-good SHA artifact. See [Production Runbook](../production-runbook.md) and [Deployment Strategy](../deployment-strategy.md).

## Database Reality

- Flyway migrations are forward-only.
- Rolling back a backend SHA does not undo V3–V9 schema.
- Assess whether the older application revision is compatible with the current schema before rolling back.

## Extension Rollback

- Users may remain on a previously installed unpacked/store build until they update.
- Backend Track/pairing contracts should remain backward compatible within the Sprint 2 release line whenever practical.
- If a breaking Extension/API mismatch occurs, prefer fixing forward or coordinating Extension + backend publish order.

## Downstream Failure Recovery

- Core Application rows remain valid if Job Intelligence or Analytics fails.
- Event retry/terminal failure is inspected operationally; rebuild Analytics from authoritative Core/history when required.
- Do not delete successful Applications to “fix” enrichment failures.

## Related

- [Database Migrations](database-migrations.md)
- [PostgreSQL Backup and Restore](../postgresql-backup-and-restore.md)
