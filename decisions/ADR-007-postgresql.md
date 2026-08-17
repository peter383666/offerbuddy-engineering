# ADR-007: Use PostgreSQL as the Primary Database

## Status

Accepted and implemented.

## Date

17 August 2026.

## Context

OfferBuddy stores users, jobs, and user-owned applications with uniqueness rules, transactional creation, filtering, sorting, and future reporting potential. Sprint 1 required persistent relational data in local development, automated integration tests, and production.

Meaningful alternatives included MySQL, a document database, or a managed/no-database prototype.

## Decision

Use PostgreSQL as the primary relational database.

- PostgreSQL 17 runs in Docker Compose locally and on the production EC2 host.
- Spring Data JPA provides application persistence access.
- Flyway owns versioned schema creation and evolution.
- Testcontainers supplies PostgreSQL for persistence integration tests.
- A Docker named volume persists production database data.
- Backups use PostgreSQL custom-format dumps and are restore-verified.

The schema uses internal BIGINT identity keys and public UUID domain identifiers. Sprint 1 application-to-user and application-to-job UUID references are enforced by application behaviour rather than database foreign keys; that trade-off is documented separately as technical debt/accepted design.

## Consequences

### Positive

- Relational constraints and transactions fit the current domain.
- Production and integration tests use the same database engine.
- PostgreSQL supports the implemented query, indexing, and future reporting needs.
- Flyway provides a reviewable, repeatable schema history.
- Standard PostgreSQL backup and restore tooling is available.

### Negative

- The project operates and backs up its own database on the single EC2 host.
- Database availability is coupled to that host.
- Schema and query behaviour remain PostgreSQL-specific where SQL features are used.
- Off-host automated backup storage is not yet implemented.

## Alternatives Considered

### MySQL

Capable of the Sprint 1 workload, but not selected. PostgreSQL was already the implemented and tested relational platform and provides no disadvantage that justifies changing engines.

### Document database

Rejected because users, jobs, applications, uniqueness, and transactions form a relational model.

### In-memory or file persistence

Rejected because it would not provide production durability, concurrency behaviour, or representative integration testing.

### Managed PostgreSQL

Not implemented in Sprint 1. It may reduce operational responsibility later, but would add cost and migration work without being required for the current scale.

## Implementation Outcome

PostgreSQL stores the live Sprint 1 data through Flyway migrations V1 and V2. Backup creation and restoration into a temporary verification database have been exercised. See [Data Model](../architecture/data-model.md).
