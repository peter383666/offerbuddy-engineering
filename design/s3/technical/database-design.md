# Sprint 3 Database Technical Design

## Status and Scope

This document publishes the frozen Sprint 3 PostgreSQL/Flyway persistence baseline. It is additive to the actual V1–V9 schema. Exact DDL is implemented in forward migrations; this document fixes ownership, identity, constraints, versioning, provenance, and migration grouping.

## Principles

- PostgreSQL is authoritative; Redis, Extension snapshots, files and read projections are not alternate truth.
- V1–V9 remain immutable. Sprint 3 starts at V10 and merged migrations are never edited.
- Tables have one owning business module even in a shared schema/deployment.
- Foreign keys, unique constraints, check constraints and versions protect final invariants; service checks provide usable errors.
- UUID/business identities and timestamps do not substitute for aggregate versions, idempotency keys, generation identities, or source provenance.
- Private content is normalised/stored only where required and is excluded from event payloads and operational telemetry.
- Large source/rendered files live behind object storage; PostgreSQL stores authoritative metadata and associations.

## Version and Provenance Model

| Version/identity | Meaning |
| --- | --- |
| Candidate `profileVersion` | Accepted Candidate aggregate revision |
| Job `contentVersion` | Accepted canonical source-content revision |
| Application `version` | Application aggregate OCC revision |
| Import draft identity/version | Resume Import lifecycle and apply protection |
| Generation/attempt identity | One accepted async business execution request |
| Artefact revision/version | Accepted content/edit revision and OCC |
| Render identity | Render of one accepted artefact revision/format |
| Published Sponsor version | Immutable complete published reference dataset |
| Idempotency key | Client/business request convergence within its frozen scope |
| Correlation ID | Diagnostic causal link; never uniqueness or authorisation |

Derived Match/artefact records persist the exact Candidate/Job/Intelligence/Base Resume/Match source identities needed to calculate freshness. Current-head references are advanced only under frozen consistency rules and do not erase history.

## Migration Sequence

| Migration | Ownership and outcome |
| --- | --- |
| V10 | Shared revisions/diagnostics: Application version, Job `contentVersion`, Job Intelligence provenance, durable `business_events.correlation_id` and compatible shared constraints |
| V11 | Candidate Profile aggregate and child structures |
| V12 | Candidate Resume Import file/draft lifecycle |
| V13 | Preparation, Match generation/results and generation/head foundations |
| V14 | Role/Base Resume, Tailored Resume revisions, assets/renders and associations |
| V15 | Focused Cover Letter attempts/revisions/assets and associations |
| V16 | AI runtime configuration, safe execution metadata/audit structures |
| V17 | Persistent AI capability-shell/reference bootstrap data |
| V18 | Sponsor working/import, immutable publication/version data and active-version metadata |

Runtime Java handler/provider registration is not a V17 database responsibility. Sponsor tables are required by the frozen publication model; V18 does not invent a generic employer CRUD domain.

## Shared Revisions and Events

V10 provides additive versions/backfills compatible with existing S2 data. Historical rows use safe initial versions without fabricating business history. Existing APIs remain compatible until their S3 contract path/representation is enabled.

The actual V7 `business_events` schema does not contain `correlation_id`. Frozen observability requires durable correlation across HTTP → event → worker → retry → AI, so V10 adds the nullable/backward-compatible column and required index/access pattern. This is an implementation delta, not an unresolved observability decision. Historical events may have no correlation ID; new S3 event creation propagates one.

## Candidate and Import Persistence

Candidate structures enforce one Profile per user and child ownership through the Profile aggregate. Child identities support deterministic updates without a generic shared-domain table. `profileVersion` protects aggregate writes.

Import persistence separates source-file metadata, async draft state, structured proposals/evidence, Candidate baseline version, idempotency, expiry and applied outcome. Raw provider responses do not become the persistence contract. Applying a draft updates Candidate and draft outcome atomically.

## Preparation, Match and Artefact Persistence

Preparation uniqueness converges repeated Candidate + Job preparation for the user. Match/Resume/Cover Letter attempts preserve requested source identities and terminal state independently from accepted result/revision tables.

Generation heads/current accepted references are explicit and protected against late older-source completion. Artefact revision tables preserve immutable history and approved limited edits through new/frozen revision semantics plus OCC. Render/asset metadata associates format/status/object identity with one accepted revision; bytes remain in object storage.

Application material association retains what was used historically and does not follow mutable current heads.

## AI Platform Persistence

AI persistence stores stable capability references, versioned runtime configuration with OCC, safe audit metadata, and privacy-safe execution/usage/failure metadata. It does not store plaintext secrets, raw prompts/responses, resume/Candidate content, or arbitrary provider payloads.

V17 inserts only stable reference identities required by configuration/monitoring. Environment/provider credentials and runtime implementation availability are resolved outside Flyway.

## Sponsor Persistence

Sponsor persistence represents the frozen lifecycle:

```text
import/job and working rows
→ validation outcome
→ immutable published dataset/version and entries
→ active published-version pointer/metadata
```

Working rows may be corrected by authorised Admin operations. Publish validates a complete working candidate, creates an immutable version, and atomically changes the active reference. Published rows are not edited/deleted in place. Redis and Extension snapshots derive from the active published version and can be rebuilt.

## Migration Safety

Every migration must pass:

1. clean PostgreSQL V1→V18 application;
2. representative Sprint 2 V9→V18 upgrade;
3. Flyway validation and single application;
4. existing S2 Job/Application/Intelligence/event readability;
5. safe backfills/defaults before new non-null constraints;
6. ownership, uniqueness, OCC, idempotency and publication constraint tests;
7. application startup against the migrated schema.

A discovered post-merge defect is repaired with a new forward migration. Recovery uses the agreed backup/restore or forward-fix decision; it never rewrites applied history.

## Verification Boundary

Implementation evidence must additionally cover concurrent create/update/head/publish operations, duplicate requests/events, old-source completion, historical null correlation, cross-user joins/queries, object metadata ownership, Redis rebuild, and last-known-good Sponsor activation.

Exact table/column/index names remain controlled by the frozen Phase 3 database record and implemented DDL. Exact API exposure is controlled by the API contract.
