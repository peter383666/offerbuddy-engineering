# Sprint 2 Database Migrations

## Purpose

Records Sprint 2 schema evolution expectations for operators and closeout readers.

## Forward-Only Rule

Deployed Flyway migrations are immutable. Sprint 2 added ordered migrations; it did not rewrite Sprint 1 V1/V2.

## Sprint 2 Migrations

| Migration | Purpose |
| --- | --- |
| V3 | `creation_source`, `application_status_history` |
| V4 | `extension_credentials` |
| V5 | `extension_pairings` |
| V6 | Pairing consumption marker |
| V7 | `business_events` |
| V8 | Job intelligence tables |
| V9 | `application_analytics` projection |

## Operator Notes

- Backend startup applies pending migrations through Flyway.
- Application rollback to an older SHA does **not** automatically reverse migrations or data.
- Assess schema compatibility separately from artifact rollback.
- Analytics projection can be rebuilt from authoritative Core/history data if required by operations procedures in the application repo.

## Related

- [Living Data Model](../../architecture/data-model.md)
- [Database Design](../../design/s2/technical/database-design.md)
- [PostgreSQL Backup and Restore](../postgresql-backup-and-restore.md)
