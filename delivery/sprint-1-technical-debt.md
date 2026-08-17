# Sprint 1 Technical Debt Register

## Purpose

This document records **developer-facing** technical debt, contract drift, and accepted design decisions identified during the Sprint 1 implementation and code review (`review/sprint-1` on the OfferBuddy source repository).

Use this register to:

- Plan Sprint 2 and later hardening work
- Avoid reopening accepted Sprint 1 design decisions by accident
- Keep engineering documentation and runtime behaviour aligned

This is **not** a product backlog of new features. Feature ideas remain in [Product Backlog](product-backlog.md). Items here should be refined into GitHub Issues before implementation.

## Source

| Item | Value |
| --- | --- |
| Review branch | `review/sprint-1` |
| Review scope | Package structure through production readiness (Reviews 1–20) |
| Captured | 2026-08-14 |
| Codebase | [offerbuddy](https://github.com/peter383666/offerbuddy) |

## Active Debt Summary

The following items are the current debt candidates after Sprint 1 cleanup and documentation reconciliation. Likely sprint is guidance, not committed scope.

### Frontend automated tests

| Field | Detail |
| --- | --- |
| Current limitation | Frontend CI runs lint, TypeScript compilation, and production build, but no unit, component, or end-to-end tests. |
| Why accepted | Sprint 1 prioritised the complete production workflow and used manual browser/production verification. |
| Impact | Session bootstrap, CSRF wiring, list/filter behaviour, and forms have weaker automated regression protection. |
| Revisit trigger | First substantial Sprint 2 frontend workflow or recurring UI regression. |
| Likely sprint | Sprint 2 candidate |

### Request and correlation IDs

| Field | Detail |
| --- | --- |
| Current limitation | `ErrorResponse.requestId` is present but never populated; logs cannot be linked reliably to a user-visible error. |
| Why accepted | Error codes, messages, paths, and local logs were sufficient for the initial user/traffic level. |
| Impact | Production incident diagnosis becomes slower as traffic and concurrent requests increase. |
| Revisit trigger | Multiple users, centralised logs, or first incident where a request cannot be traced. |
| Likely sprint | Sprint 2 candidate |

### Secrets-management hardening

| Field | Detail |
| --- | --- |
| Current limitation | Runtime secrets are stored in an EC2-local `.env`; deployment credentials use GitHub Environment secrets. |
| Why accepted | This is workable for one host and one operator, and production rejects missing/local-placeholder secrets. |
| Impact | Rotation, auditability, host recovery, and least-privilege secret access remain manual. |
| Revisit trigger | Additional operators/hosts, formal rotation needs, or infrastructure automation. |
| Likely sprint | Sprint 2+ hardening |

### Automated off-host backups

| Field | Detail |
| --- | --- |
| Current limitation | Backup and restore verification exist, but dumps remain on the EC2 host unless copied manually; scheduling/retention are not automated. |
| Why accepted | Sprint 1 established and exercised a recoverable PostgreSQL dump before adding storage automation. |
| Impact | Complete host/storage loss could remove both the database and local backups; recovery point is operator-dependent. |
| Revisit trigger | Production data value increases, recovery objectives are defined, or before broader user adoption. |
| Likely sprint | Sprint 2 candidate |

### Staging environment

| Field | Detail |
| --- | --- |
| Current limitation | There is no persistent production-like staging environment; `release` verification relies on CI and controlled deployment/smoke testing. |
| Why accepted | A second environment would add cost and configuration overhead for one engineer and a small application. |
| Impact | OAuth, Nginx, deployment, and migration issues may be found later than code-level issues. |
| Revisit trigger | More contributors, riskier migrations, more users, or production verification becomes disruptive. |
| Likely sprint | Sprint 2+ decision |

### ApplicationService growth

| Field | Detail |
| --- | --- |
| Current limitation | One service coordinates creation, list validation, detail, updates, status, and deletion. |
| Why accepted | Keeping related Sprint 1 use cases together was understandable and avoided premature abstractions. |
| Impact | Additional analytics, capture, or lifecycle rules could reduce cohesion and increase change risk. |
| Revisit trigger | The service gains distinct workflows/dependencies or tests become difficult to isolate. |
| Likely sprint | Revisit during affected feature work |

### Redis activation and security

| Field | Detail |
| --- | --- |
| Current limitation | Redis is retained in local/production Compose without authentication and is not used by application logic. It is private to the Compose network. |
| Why accepted | It is intentionally reserved for possible future session/cache-related capabilities and is not publicly exposed. |
| Impact | Small inactive operational/security surface; no current product benefit. |
| Revisit trigger | Before the application first connects to Redis or the network/deployment topology changes. |
| Likely sprint | When Redis becomes an active dependency |

### Observability

| Field | Detail |
| --- | --- |
| Current limitation | Operations rely on local service logs and health endpoints; there is no aggregation, alerting, APM, or tracing. |
| Why accepted | One host, low traffic, and direct operator access made local logs proportionate for Sprint 1. |
| Impact | Failures may be detected late and historical/cross-service diagnosis is limited. |
| Revisit trigger | More users, unattended operation, performance issues, or formal availability expectations. |
| Likely sprint | Sprint 2+ hardening |

### Job identity and duplicate rules

| Field | Detail |
| --- | --- |
| Current limitation | Source identity is only partly database-constrained; URL and company/title matching remain application-level and repeat applications are rejected. |
| Why accepted | Sprint 1 needed practical duplicate protection before a broader product rule for job/reapplication identity existed. |
| Impact | Concurrent/manual entries may create near-duplicate jobs, and legitimate later reapplications cannot be represented. |
| Revisit trigger | Browser extension ingestion, analytics joins, or a requirement to reapply to the same job. |
| Likely sprint | Product decision during Sprint 2+ |

## Classification

| Class | Meaning |
| --- | --- |
| **Sprint 2+ debt** | Known gap or drift. Safe for Sprint 1 release if documented; schedule follow-up. |
| **Accepted design decision** | Intentional Sprint 1 choice. Do not “fix” without a new product/architecture decision. |
| **Docs sync** | Runtime behaviour is intentional; engineering docs or OpenAPI must catch up. |

Severity:

| Severity | Meaning |
| --- | --- |
| High | Contract mismatch or integrity risk that will confuse clients or operators soon |
| Medium | Maintainability, observability, or correctness edge cases |
| Low | Cleanup, unused assets, or non-blocking polish |

---

## 1. API contract and OpenAPI

### TD-S1-001 — Job text field names diverge from engineering OpenAPI

**Status: Resolved in Sprint 1 documentation.** The checked-in contract now uses `responsibilities` and `requirements` to match the shipped DTOs.

| | |
| --- | --- |
| Class | Docs sync / Sprint 2+ debt |
| Severity | High |
| Area | API / OpenAPI |

**Current behaviour (implementation)**  
Application and job JSON use:

- `responsibilities`
- `requirements`

**Engineering contract**  
[`architecture/openapi.yaml`](../architecture/openapi.yaml) and examples in [`architecture/api-design.md`](../architecture/api-design.md) still describe:

- `whatYoullDo`
- `whatWereLookingFor`

**Why it matters**  
Clients generated or coded against the engineering OpenAPI will not match the live API.

**Recommended action**  
Prefer updating engineering OpenAPI and api-design to match the shipped field names (lower churn than renaming a live API). If product prefers the original names, add a coordinated rename in backend DTOs + frontend + docs.

**Likely files**

- `architecture/openapi.yaml`
- `architecture/api-design.md`
- `backend/.../JobInputRequest.java`, `JobResponse.java`, mappers
- Frontend create/detail forms and types

---

### TD-S1-002 — Duplicate application is a hard `409` (docs say otherwise)

**Status: Resolved in Sprint 1 documentation.** API and data-model documents now record `409 DUPLICATE_APPLICATION`.

| | |
| --- | --- |
| Class | Docs sync |
| Severity | High |
| Area | Application create / error contract |

**Current behaviour (implementation)**  
Creating a second application for the same `(user_id, job_id)` fails with:

- HTTP `409 Conflict`
- code `DUPLICATE_APPLICATION`
- enforced by unique constraint `uk_job_applications_user_job` and service handling of concurrent UK violations

**Engineering contract**  
[`architecture/api-design.md`](../architecture/api-design.md) §11.7 states Sprint 1 does **not** reject possible duplicates with `409`, and reserves soft warnings for a future contract.

**Why it matters**  
Frontend and API consumers following the design doc will not handle the real failure mode.

**Recommended action**  
Update api-design and OpenAPI to document intentional hard-reject behaviour (preferred). Only change runtime if product reopens “allow re-apply to same job” for Sprint 2.

**Likely files**

- `architecture/api-design.md`
- `architecture/openapi.yaml`
- `DuplicateApplicationException` / `ApiExceptionHandler`
- Frontend create-application error handling

---

### TD-S1-003 — OpenAPI error responses incompletely documented

| | |
| --- | --- |
| Class | Sprint 2+ debt |
| Severity | Medium |
| Area | OpenAPI / Swagger |

**Current behaviour**  
Controllers expose `@Tag` / `@Operation`. Runtime returns a unified `ErrorResponse`. Generated OpenAPI largely lacks `@ApiResponse` coverage for:

- `401 AUTHENTICATION_REQUIRED`
- `403` access denied
- `404 APPLICATION_NOT_FOUND`
- `409 DUPLICATE_APPLICATION`
- `400 VALIDATION_FAILED` (+ `fieldErrors`)
- `500 INTERNAL_ERROR`
- Job-parsing business codes

CSRF is documented as a scheme + description (Sprint 1 review fix); mutating Try-it-out still depends on cookie + header behaviour.

**Recommended action**  
Add a shared error schema component and annotate key operations. Keep production springdoc disabled.

**Likely files**

- Controllers under `com.offerbuddy.*`
- `OpenApiConfiguration`
- `ErrorResponse`

---

### TD-S1-004 — `requestId` is never populated

| | |
| --- | --- |
| Class | Sprint 2+ debt |
| Severity | Medium |
| Area | Errors / observability |

**Current behaviour**  
`ErrorResponse` includes optional `requestId`, always `null` in factory methods. Engineering api-design examples show values such as `req_01JXYZ789`.

**Recommended action**  
Introduce a request correlation id (filter / MDC), set it on `ErrorResponse`, and log it once per failure path.

**Likely files**

- `ErrorResponse`, `ApiExceptionHandler`
- New servlet filter or Spring observation hook
- Logging pattern docs

---

### TD-S1-005 — Status history mentioned in API design, not implemented

**Status: Resolved as documentation alignment.** Sprint 1 documentation now states that only the current status is stored. Status history remains a future product decision, not committed debt.

| | |
| --- | --- |
| Class | Sprint 2+ debt (or docs sync) |
| Severity | Medium |
| Area | Application lifecycle |

**Current behaviour**  
Application status is a single column on `job_applications`. No status-history table or API.

**Engineering contract**  
api-design create flow text refers to recording an initial status-history entry.

**Recommended action**  
Either remove status-history language from Sprint 1 docs, or schedule a real history model/API for Sprint 2+.

---

## 2. Data model and persistence

### TD-S1-006 — Soft UUID references without foreign keys

| | |
| --- | --- |
| Class | **Accepted design decision** |
| Severity | — (documented) |
| Area | Schema / JPA |

**Decision**  
`job_applications.user_id` and `job_applications.job_id` store public UUIDs without database FKs and without JPA `@ManyToOne`. Ownership and existence are enforced in application services/repositories.

**Rationale (Sprint 1)**  
Keep public identifiers and module boundaries simple; avoid coupling application rows to internal surrogate keys.

**Future trigger**  
Revisit if orphan rows, analytics joins, or referential integrity become operational problems. Prefer a **new Flyway migration**; do not edit deployed V1/V2 casually.

---

### TD-S1-007 — Job dedupe is application-level, not fully constrained in DB

| | |
| --- | --- |
| Class | Sprint 2+ debt |
| Severity | Medium |
| Area | Jobs |

**Current behaviour**  
`JobService.findOrCreate` dedupes by:

1. `(source_platform, external_job_id)` when both present (also DB unique)
2. else `source_url` when present (indexed, **not** unique)

No uniqueness on `(company_name, job_title)`.

**Why it matters**  
Manual entries without stable source identity can create near-duplicate job rows. Concurrent creates with the same URL can race without a unique constraint.

**Recommended action**  
Product decision first (when is a job “the same”?). Then add migration(s) and align `findOrCreate`.

---

### TD-S1-008 — Stale comment inside Flyway V1

| | |
| --- | --- |
| Class | Sprint 2+ debt (docs-only) |
| Severity | Low |
| Area | Migrations |

**Current behaviour**  
`V1__create_initial_schema.sql` comments that `uk_job_applications_user_job` may block legitimate repeat applications. Runtime intentionally treats that as a conflict (`409`).

**Recommended action**  
**Do not edit V1.** Capture the intended rule in data-model / api-design (see TD-S1-002). Optional clarifying comment only via a later non-checksum-affecting process if the team ever rebases unused environments (generally avoid).

---

## 3. Infrastructure and operations

### TD-S1-009 — Redis runs locally but is unused by the application

**Status: Accepted reserved infrastructure.** Redis remains intentionally present but inactive. Do not remove it as generic cleanup; revisit authentication and production hardening before first application use.

| | |
| --- | --- |
| Class | Sprint 2+ debt |
| Severity | Low |
| Area | Docker Compose / ops |

**Current behaviour**  
`infrastructure/docker/docker-compose.yml` starts Redis. The Spring Boot app does not use Redis for sessions, cache, or queues (session is servlet/JDBC-free in-memory/server session for Sprint 1).

**Recommended action**  
Keep its inactive status explicit. Before first session/cache use, define the requirement, configure appropriate authentication/security, test failure behaviour, and update operations documentation.

**Likely files**

- `infrastructure/docker/docker-compose.yml`
- `infrastructure/docker/README.md`
- `operations/deployment-strategy.md`
- ADR-005 follow-up note if behaviour changes

---

### TD-S1-010 — Empty Maven POM metadata

| | |
| --- | --- |
| Class | Sprint 2+ debt |
| Severity | Low |
| Area | Backend build |

**Current behaviour**  
`backend/pom.xml` still contains empty Initializr stubs (`url`, `license`, `developers`, `scm`).

**Recommended action**  
Fill or delete the empty elements when next touching build metadata. No functional impact.

---

## 4. Frontend quality

### TD-S1-011 — No frontend unit/component test suite

| | |
| --- | --- |
| Class | Sprint 2+ debt |
| Severity | Medium |
| Area | Frontend / CI |

**Current behaviour**  
`frontend/package.json` has `dev` / `build` / `lint` / `preview` only. CI runs lint + build. No Vitest/Jest/Testing Library scripts or `*.test.*` files.

**Recommended action**  
Add Vitest + Testing Library for auth/session helpers, CSRF header wiring, and critical list/create forms. Wire `npm test` into CI.

---

### TD-S1-012 — Unused frontend components

**Status: Resolved during Sprint 1 cleanup.** The unused components/assets identified by the cleanup review were removed.

| | |
| --- | --- |
| Class | Sprint 2+ debt |
| Severity | Low |
| Area | Frontend |

**Current behaviour**  
Unused (or effectively unused) UI leftovers:

- `frontend/src/components/Card.tsx`
- `frontend/src/components/OfferBuddyLogo.tsx` (+ related CSS rules in `index.css`)

**Recommended action**  
Delete during a focused frontend cleanup PR, or adopt them if the design system needs them.

---

### TD-S1-013 — Frontend dependency audit finding

| | |
| --- | --- |
| Class | Sprint 2+ debt |
| Severity | Medium (verify) |
| Area | Frontend supply chain |

**Current behaviour**  
As of Sprint 1 verification, `npm audit` reported at least one high-severity advisory in the frontend tree.

**Recommended action**  
Triage the advisory (direct vs transitive), apply a minimal fix or override, and re-run lint/build. Do not mix broad dependency upgrades with feature work.

---

## 5. Testing and CI (residual)

### TD-S1-014 — Product backlog AI extraction test checklist still open

**Status: Resolved as backlog reconciliation.** The stale checklist was removed; existing parsing tests are documented in the testing strategy.

| | |
| --- | --- |
| Class | Sprint 2+ debt / backlog hygiene |
| Severity | Medium |
| Area | Quality |

**Current behaviour**  
[Product Backlog](product-backlog.md) still lists AI extraction test tasks. Backend now has substantial job-parsing unit/integration coverage; the backlog checklist has not been reconciled.

**Recommended action**  
Audit each checklist item against existing tests under `com.offerbuddy.jobparsing`, tick completed items, and leave only true gaps.

---

## 6. Engineering documentation drift (process)

These are documentation/process debts around Sprint 1 delivery records, not runtime defects.

| ID | Item | Recommended action |
| --- | --- | --- |
| TD-S1-015 | `delivery/sprint-1.md` showed backend/frontend as “Not started” | Resolved during Sprint 1 documentation closure |
| TD-S1-016 | Root `README.md` / `delivery/roadmap.md` lagged implementation | Resolved during Sprint 1 documentation closure |
| TD-S1-017 | Engineering OpenAPI vs generated springdoc remain dual sources of truth | Pick a sync process: regenerate or manually align on each contract change |

---

## Accepted Sprint 1 design decisions (do not treat as bugs)

Recorded so future reviews do not reopen them casually.

| ID | Decision | Notes |
| --- | --- | --- |
| AD-S1-01 | Job domain extracted to `com.offerbuddy.job`; dependency direction `application → job` only | Review 1–2 |
| AD-S1-02 | Keep a single `ApplicationService` for Sprint 1 use cases | Review 3 |
| AD-S1-03 | Store-on-Repository persistence style accepted for Sprint 1 | Review 4 |
| AD-S1-04 | Soft UUID references; no FKs / no `@ManyToOne` on applications | Review 8; see TD-S1-006 |
| AD-S1-05 | Ownership via user-scoped queries; other users’ rows appear as `404` | Review 11 |
| AD-S1-06 | CSRF cookie + `X-XSRF-TOKEN` required for mutating API calls | Review 11 |
| AD-S1-07 | Swagger/OpenAPI public only when `permit-api-docs=true`; disabled in `prod` | Reviews 14–15 |
| AD-S1-08 | Class-level `@Transactional(readOnly=true)` with explicit writes; `JobService.findOrCreate` is `MANDATORY` | Review 12 |
| AD-S1-09 | Production profile requires externalised secrets and rejects local placeholders | Review 14 |
| AD-S1-10 | Flyway V1/V2 are immutable after use; schema changes = new migrations | Review 16 |

---

## Suggested Sprint 2 intake order

Recommended order for turning debt into issues (not committed scope):

1. **TD-S1-011** — Frontend test foundation  
2. **TD-S1-004** — Request correlation ID  
3. **Backup/secrets hardening** — off-host backups and managed-secret decision  
4. **TD-S1-003 / TD-S1-017** — generated/checked-in OpenAPI maintenance and error coverage  
5. **TD-S1-007** — job identity and repeat-application product decision  
6. **Observability/staging** — prioritise only when the active-summary triggers are met  
7. **TD-S1-010 / TD-S1-013** — low-risk metadata and dependency follow-up  

---

## Out of scope for this register

- New product features (analytics, interview tools, multi-provider auth, etc.) — see MVP exclusions and product backlog  
- Broad dependency upgrades unrelated to a concrete defect  
- Cosmetic refactors with no maintenance benefit  
- Editing already-deployed Flyway scripts in place  

---

## Related documents

- [Sprint 1 Planning](sprint-1.md)
- [Product Backlog](product-backlog.md)
- [API Design](../architecture/api-design.md)
- [Data Model](../architecture/data-model.md)
- [Testing Strategy](../quality/testing-strategy.md)
- [Definition of Done](../quality/definition-of-done.md)
