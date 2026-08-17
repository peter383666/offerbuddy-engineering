# PostgreSQL Backup and Restore Verification

## Purpose

Sprint 1 uses script-backed PostgreSQL custom-format backups and validates them by restoring into a temporary database.

Source scripts:

- [`backup-postgres.sh`](https://github.com/peter383666/offerbuddy/blob/main/scripts/operations/backup-postgres.sh)
- [`verify-postgres-backup.sh`](https://github.com/peter383666/offerbuddy/blob/main/scripts/operations/verify-postgres-backup.sh)

The scripts contain no production passwords. They load database identifiers from the EC2-local `/opt/offerbuddy/.env`.

## Backup Location and Format

The backup script writes to:

```text
/opt/offerbuddy/backups/postgres/offerbuddy_YYYYMMDD_HHMMSS.dump
```

It:

1. verifies that `/opt/offerbuddy/.env` exists
2. loads `POSTGRES_DB` and `POSTGRES_USER`
3. creates the backup directory with restrictive permissions
4. runs `pg_dump -Fc` inside the PostgreSQL container
5. fails if the output file is empty

The dump is stored on the EC2 host. Off-host replication, automated scheduling, retention, and encryption management are not implemented by these scripts.

## Creating a Backup

Ensure the reviewed scripts are present on the host, then run the backup script using the production operator account:

```bash
cd /opt/offerbuddy
./scripts/operations/backup-postgres.sh
```

The exact host placement may differ if operations scripts are installed separately; the scripts themselves use the fixed production project path `/opt/offerbuddy`.

Successful output identifies a non-empty timestamped dump. Do not print or transfer the `.env` file while checking the backup.

## Restore Verification

**A backup is not considered fully verified until restoration has been tested.**

The verification flow is:

```text
latest backup dump
  -> drop stale temporary restore database if present
  -> create offerbuddy_restore_test
  -> pg_restore into temporary database
  -> count and list public tables
  -> fail if no public tables exist
  -> drop temporary restore database
```

Run:

```bash
cd /opt/offerbuddy
./scripts/operations/verify-postgres-backup.sh
```

The script selects the newest file matching `offerbuddy_*.dump` in the backup directory. It restores with `--no-owner` and `--no-privileges`, so verification is not tied to dump ownership metadata.

## What Verification Proves

The current script proves that:

- the selected dump can be read by `pg_restore`
- PostgreSQL can create a temporary database
- the restore completes without a command failure
- at least one public table exists after restoration
- the restored public tables can be listed

It does not currently prove:

- expected application row counts
- application-level business invariants
- freshness against a recovery-point objective
- off-host recoverability after complete EC2 loss
- automated scheduling or retention

Those remain operational hardening opportunities.

## Cleanup and Failure Handling

On success, the temporary `offerbuddy_restore_test` database is removed. If verification exits early, inspect whether that temporary database remains before retrying.

Verification never targets the live production database. Do not change the restore target to the production database for a routine test.

## Actual Recovery

An actual production restore is a separate, higher-risk operation. Before proceeding:

1. identify the incident and recovery point
2. preserve the current state where possible
3. select a restore-tested backup
4. plan downtime and application compatibility
5. restore under an explicitly reviewed recovery procedure
6. verify schema, representative data, authentication, and application behaviour

The Sprint 1 verification script is evidence that a dump is restorable; it is not an automated production-disaster-recovery workflow.
