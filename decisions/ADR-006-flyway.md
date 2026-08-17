# ADR-006: Use Flyway for Database Migrations

## Status

Accepted

## Date

5 August 2026

## Context

OfferBuddy uses PostgreSQL as its primary relational database. As the product evolves, schema changes must remain consistent across local development, continuous integration, test environments, and production.

Relying on manual database changes or automatic ORM schema mutation would make changes difficult to review, reproduce, and audit. The project needs an explicit, version-controlled migration strategy that integrates with the Spring Boot application and supports incremental delivery.

The solution should:

- Keep schema changes in source control
- Apply changes in a deterministic order
- Provide a shared schema history across environments
- Fail visibly when migrations cannot be applied safely
- Support PostgreSQL-specific migration behaviour
- Remain simple enough for a modular monolith and small engineering team

## Decision

OfferBuddy will use Flyway to manage relational database schema migrations.

Database changes will be represented by ordered, immutable migration files reviewed with the application change that requires them. Hibernate will validate or use the resulting schema but will not create or update the production schema automatically.

Applied migrations must not be edited. Corrections are made through a new migration so that every environment follows the same forward history.

## Rationale

Flyway provides a small and explicit migration model that integrates well with Spring Boot and PostgreSQL.

SQL migrations keep database changes visible to reviewers, make the resulting schema behaviour clear, and avoid hiding production-impacting changes behind ORM configuration.

Its sequential migration model is sufficient for the OfferBuddy modular monolith and does not require a more elaborate database change-management system.

## Consequences

### Positive

- Schema history is versioned with application code.
- Local, CI, and production environments can apply the same changes.
- Database changes become reviewable through pull requests.
- Manual production schema changes are reduced.
- Migration failures are visible during startup or verification.
- PostgreSQL-specific constraints and indexes can be expressed directly.

### Negative

- Engineers must design forward-compatible migrations carefully.
- Applied migration files cannot be casually changed or reordered.
- Destructive or long-running migrations require additional operational planning.
- Rollback is not automatic and may require a compensating migration or recovery procedure.
- Application and schema compatibility must be considered during deployment.

## Alternatives Considered

### Hibernate Automatic Schema Updates

Rejected because generated schema mutation is less explicit, less reviewable, and unsafe as the primary production migration mechanism.

### Manual SQL Changes

Rejected because manual execution creates environment drift and weakens repeatability and auditability.

### Liquibase

Liquibase would meet the core requirements, but Flyway was selected for its simpler migration model and direct fit with the project's SQL-first needs.

### Recreate the Database for Every Change

Rejected because it does not support persistent development data or controlled evolution toward production.

## Related Documents

- `technology/tech-stack.md`
- `architecture/data-model.md`
- `architecture/container-design.md`
- `quality/testing-strategy.md`
- `operations/deployment-strategy.md`

## Outcome

Flyway is the authoritative mechanism for OfferBuddy database schema evolution. Schema changes are delivered as version-controlled migrations and applied consistently across environments.

Sprint 1 production uses `V1__create_initial_schema.sql` and `V2__align_jobs_and_applications_for_create.sql`. Deployed migration files are treated as immutable; later schema changes require a new migration.
