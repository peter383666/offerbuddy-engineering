# Data Architecture

## Ownership

| Data | Owner |
| --- | --- |
| Users | Identity / auth |
| Jobs | Shared Job domain |
| Applications + status history | Application domain |
| Business events | Delivery infrastructure |
| Job analysis results | Job Intelligence |
| Application analytics rows | Analytics projection |

Extension auth tables store credential/pairing material only. They are not Job/Application aggregates.

## Principles

- PostgreSQL is the durable system of record.
- Shared Job identity uses `source_platform + external_job_id` when present.
- One Application per authenticated user and Job.
- Analytics projections are rebuildable and non-authoritative.
- Redis is not part of the S2 data path.

## Related

- [Living Data Model](../data-model.md)
- [Database Design](../../design/s2/technical/database-design.md)
- [ADR-007](../../decisions/ADR-007-postgresql.md)
