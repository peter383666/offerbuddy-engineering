# Data Model

## Purpose

This document describes the **current** OfferBuddy persistence model after Sprint 2. Flyway migrations and mapped entities in the application repository remain the executable source of truth.

Historical Sprint 1-only schema is preserved under [`delivery/s1/architecture/data-model.md`](../delivery/s1/architecture/data-model.md).

## Overview

```text
users
  └── job_applications ─── jobs
        │                    │
        ├── application_status_history
        └── application_analytics (projection)

jobs
  └── job_analysis (+ responsibilities / requirements / skills)

business_events          (durable async delivery)
extension_credentials    (auth material; not a business domain)
extension_pairings       (auth handshake; not a business domain)
```

## Core Domain

### users

Local OfferBuddy identity linked to Google OIDC. Unchanged conceptual role from Sprint 1.

### jobs

Shared canonical job facts. When platform identity exists, uniqueness is:

```text
source_platform + external_job_id
```

Selective refresh updates non-empty source facts without null erasure. Sprint 1 `responsibilities` / `requirements` columns remain legacy compatibility fields; structured semantic results live under Job Intelligence tables.

### job_applications

User-owned Application current state. Sprint 2 adds:

- `creation_source` (`WEB` | `EXTENSION`)
- unique `(user_id, job_id)` retained as final duplicate protection

### application_status_history

Real lifecycle transitions. New Applications record an initial `null → APPLIED` history row.

### Application status values

Persisted status enum currently includes:

- `APPLIED`
- `NO_RESPONSE` (legacy Sprint 1 value; retained)
- `INTERVIEW`
- `OFFER`
- `REJECTED`
- `WITHDRAWN`

Analytics may also **derive** a no-response classification from aged `APPLIED` applications without requiring a new lifecycle transition. See [Implementation Reconciliation](../delivery/s2/implementation-reconciliation.md).

## Extension Authentication Tables

`extension_credentials` and `extension_pairings` support Extension authentication. They are not Extension business aggregates and do not replace Job/Application ownership rules.

## Business Events

`business_events` stores durable delivery intent for downstream processors. It is not the authoritative Application/Job history.

## Job Intelligence

Versioned analysis attempts and structured results:

- `job_analysis`
- `job_responsibilities`
- `job_requirements`
- `job_skills`

Job-owned and shared; not user-owned merely because one user triggered capture.

## Application Analytics

`application_analytics` is a rebuildable read projection (one row per Application). It is eventually consistent and not the source of ownership or lifecycle truth.

## Flyway Migrations

| Migration | Effect |
| --- | --- |
| V1–V2 | Sprint 1 baseline |
| V3 | Creation source + status history |
| V4–V6 | Extension credentials and pairings |
| V7 | Business events |
| V8 | Job intelligence |
| V9 | Application analytics projection |

Deployed migrations are immutable. Future changes require a new ordered migration.

## Related

- [Database Design (S2)](../design/s2/technical/database-design.md)
- [ADR-006](../decisions/ADR-006-flyway.md)
- [ADR-007](../decisions/ADR-007-postgresql.md)
- [API Design](api-design.md)
