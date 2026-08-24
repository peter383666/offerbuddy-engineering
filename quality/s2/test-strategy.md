# Sprint 2 Test Strategy

## Purpose

Describes how Sprint 2 capabilities are verified. Living baseline remains [`quality/testing-strategy.md`](../testing-strategy.md).

## Risk Focus

Highest-value risks for Sprint 2:

- Extension capture correctness on SEEK/Indeed dynamic pages
- Credential isolation from page JavaScript
- Server-owned Track create-or-reuse and duplicate outcomes
- Core success independent of Job Intelligence / Analytics
- Business Event claim, retry, and idempotent handlers
- Analytics projection convergence and user scoping

## Automated Layers That Exist

| Layer | Where | Command / CI |
| --- | --- | --- |
| Backend unit/service/API/security/integration | `backend/` | `./mvnw clean verify` via Backend CI |
| Extension Vitest | `extension/` | `npm test` / `npm run verify` (local; **no dedicated GitHub Actions workflow**) |
| Frontend Vitest | `frontend/` | `npm test` locally; Frontend CI runs **lint + build only** |
| Local precheck | `scripts/precheck.*` | Mirrors frontend lint/build, extension verify, backend verify |

## Backend Coverage (S2-relevant)

Automated tests exist for:

- Extension pairing, credentials, Track validation/ingestion
- Application create-or-reuse, history, ownership
- Business Event persistence, claim, processing, core atomicity
- Job Intelligence validation/persistence/analysis paths
- Analytics projection, rebuild, query, derived no-response classification

Testcontainers PostgreSQL is used for persistence and migration-shaped integration behaviour.

## Extension Coverage

Automated tests exist for adapters, lifecycle/tracking, companion behaviour, pairing/save orchestration, popup states, and privileged-boundary rules.

Manual checks remain required for current SEEK/Indeed page variants (see [Extension Validation](extension-validation.md)).

## Frontend Coverage

Component/page tests exist for Home extension discovery, Analytics, Job Intelligence section, and related application surfaces. They are not gated by Frontend CI yet.

## Explicit Non-Goals for S2 Closeout

- Inventing missing CI workflows as if they already exist
- Claiming browser end-to-end automation that is not present
- Treating CI green as production OAuth / live site / Gemini proof

## Related

- [Integration Test Plan](integration-test-plan.md)
- [Regression Checklist](regression-checklist.md)
- [Release Quality Gate](release-quality-gate.md)
- [Implementation Reconciliation](../../delivery/s2/implementation-reconciliation.md)
