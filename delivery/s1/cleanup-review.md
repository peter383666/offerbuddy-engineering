# OfferBuddy Sprint 1 — Cleanup & Fixes Review

| Item | Value |
| --- | --- |
| Status | Review only — no repository files were modified |
| Source repo | `offerbuddy` |
| Branch reviewed | `review/sprint-1` |
| Date | 2026-08-17 |
| Scope | Repository hygiene, leftover debug artifacts, backend/frontend cleanup, configuration, Docker/ops, CI/CD, tests, lightweight security |

This is **not** a refactoring sprint. Sprint 2 features, UI redesign, and major architecture changes are out of scope.

**Summary**

| Group | Count |
| --- | --- |
| A. Fix before Sprint 1 close | 12 |
| B. Technical debt | 8 |
| C. No action required | 9 |
| D. Potential security / production issues | 4 |

No committed production secrets were found. `.env` files in Git only contain non-secret origins (`VITE_BACKEND_ORIGIN`). Production passwords, OAuth secrets, and API keys are referenced from environment variables. Local YAML placeholders are rejected by `ProductionConfigurationVerifier` on the `prod` profile.

After approval, implement cleanup in small groups:

1. Repository / configuration
2. Backend
3. Frontend
4. Tests (only if in the approved set)
5. Production / security (D1–D3 only if included in this close)

---

## A. Fix Before Sprint 1 Close

Small, clear problems that should be fixed now. Risk is of the cleanup itself, not of leaving the issue.

### A1 — Root `.gitignore` does not ignore dumps, backups, or PEM keys

| | |
| --- | --- |
| Path | `.gitignore` |
| Issue | Root ignore rules cover `.env` and IDE files, but not `*.dump`, `backups/`, `*.pem`, or `.DS_Store` at the repo root. |
| Why it matters | A local `pg_dump` or SSH key dropped in the repo root could be committed accidentally. |
| Proposed fix | Add `*.dump`, `backups/`, `*.pem`, and `.DS_Store`. |
| Risk | Low |

### A2 — Backup script filename has a leading space

| | |
| --- | --- |
| Path | `scripts/operations/ backup-postgres.sh` |
| Issue | Git tracks this file with a space after `operations/`. |
| Why it matters | Awkward to invoke; easy to miss in ops docs and cron. |
| Proposed fix | Rename to `scripts/operations/backup-postgres.sh`. |
| Risk | Low |

### A3 — Unused UI leftovers still tracked

| | |
| --- | --- |
| Path | `frontend/src/components/Card.tsx`, `frontend/src/components/OfferBuddyLogo.tsx`, `frontend/public/logo-mark.png`, `frontend/public/icons.svg` |
| Issue | Components and public assets are not referenced by the running UI. |
| Why it matters | Dead code and unused raster assets in the Sprint 1 baseline. |
| Proposed fix | Delete the unused components and unreferenced public files. |
| Risk | Low |

### A4 — Unused Qodana starter file

| | |
| --- | --- |
| Path | `qodana.yaml` |
| Issue | JetBrains starter file is not wired to any GitHub Actions workflow. |
| Why it matters | Looks like an IDE leftover in a production-ready repo. |
| Proposed fix | Delete, or document why it stays. |
| Risk | Low |

### A5 — Root README still says Sprint 0 and MySQL

| | |
| --- | --- |
| Path | `README.md` |
| Issue | Status is “Sprint 0”; infrastructure lists MySQL. |
| Why it matters | Misleading for a PostgreSQL production app that has completed Sprint 1 delivery. |
| Proposed fix | Update stack and status to Sprint 1 / PostgreSQL. |
| Risk | Low |

### A6 — Backend README has a broken markdown fence and stale Flyway names

| | |
| --- | --- |
| Path | `backend/README.md` |
| Issue | The “run locally” fenced block never closes. Flyway examples mention `V2__add_company_table.sql` / `V3__add_job_application_table.sql`, which are not the shipped migrations. |
| Why it matters | Docs do not match the real schema history (`V1`, `V2`). |
| Proposed fix | Close the fence; cite the V1/V2 files that actually exist. |
| Risk | Low |

### A7 — Production README still describes Nginx as planned

| | |
| --- | --- |
| Path | `infrastructure/production/README.md` |
| Issue | Text still treats host Nginx / TLS / frontend CD as future work. |
| Why it matters | Nginx, HTTPS, and frontend CD are already in production. |
| Proposed fix | Mark Nginx as in place; drop obsolete next-steps. |
| Risk | Low |

### A8 — Terms and Privacy are dead `#` links

| | |
| --- | --- |
| Path | `frontend/src/pages/LoginPage.tsx` |
| Issue | Login legal copy uses `<a href="#">`. |
| Why it matters | Looks unfinished; assistive technology treats them as links to nowhere. |
| Proposed fix | Render as plain text until real pages exist. |
| Risk | Low |

### A9 — Search placeholder says company or position, API only sends `companyName`

| | |
| --- | --- |
| Path | `frontend/src/pages/ApplicationsPage.tsx` |
| Issue | Placeholder: “Search company or position...”. Request only sets `companyName`. |
| Why it matters | Position search returns empty pages. Product spec includes job title. |
| Proposed fix | Send `jobTitle` as well, or change the placeholder to match current behaviour. |
| Risk | Low |

### A10 — Search debounce always calls `setPage(0)`

| | |
| --- | --- |
| Path | `frontend/src/pages/ApplicationsPage.tsx` |
| Issue | After 300ms the effect always does `setDebouncedSearch(...)` and `setPage(0)`, even when trimmed search did not change. |
| Why it matters | Typing then deleting jumps the user from a later page back to page 1. |
| Proposed fix | Reset page only when trimmed search actually changes. |
| Risk | Low |

### A11 — Auth bootstrap hangs on non-401 errors

| | |
| --- | --- |
| Path | `frontend/src/auth/AuthProvider.tsx` |
| Issue | `fetchCurrentUser` rethrows anything that is not HTTP 401. The mount effect has no try/catch, so status stays `loading`. |
| Why it matters | Network, CORS, or 500 leaves the app on “Loading...” forever. |
| Proposed fix | Catch non-401 and treat as unauthenticated, or show a retry state. |
| Risk | Low |

### A12 — Deploy `latest` is inconsistent across frontend and backend

| | |
| --- | --- |
| Path | `.github/workflows/backend-ci.yml`, `backend-deploy.yml`, `frontend-ci.yml`, `frontend-deploy.yml` |
| Issue | Frontend `latest` = last successful Frontend CI on `main`. Backend `latest` = newest full-SHA tag on GHCR, which can be a `release` image not yet merged to `main`. |
| Why it matters | Intended model is: verify on `release`, then `main` is production-ready, then deploy an explicit SHA. Backend `latest` can skip that. |
| Proposed fix | Publish/select backend images the same way as main-verified SHAs. Keep explicit SHA as the production path. Consider not defaulting `workflow_dispatch` to `latest`. |
| Risk | Medium |

---

## B. Technical Debt

Valid engineering improvements that should **not** block Sprint 1 closure.

### B1 — Redis runs but the application never uses it

| | |
| --- | --- |
| Path / subsystem | `infrastructure/docker/docker-compose.yml`, `infrastructure/production/docker-compose.yml` |
| Issue | Redis is started locally and in production. Spring Boot does not use it for sessions, cache, or queues. |
| Why it can wait | Isolated on the Docker network; already described as reserved. |
| Recommended future action | Remove until session/cache work, or document as reserved-only. |

### B2 — No frontend unit/component tests

| | |
| --- | --- |
| Path / subsystem | `frontend/package.json` |
| Issue | Scripts are `dev` / `build` / `lint` / `preview` only. CI is lint + production build. |
| Why it can wait | Manual production verification already happened. |
| Recommended future action | Add Vitest + Testing Library for auth, CSRF, and list/create forms. |

### B3 — `ErrorResponse.requestId` is always null

| | |
| --- | --- |
| Path / subsystem | `backend/src/main/java/com/offerbuddy/shared/error/ErrorResponse.java` |
| Issue | Factory methods pass `null` for `requestId`. |
| Why it can wait | Errors still return code, message, and path. |
| Recommended future action | Correlation id filter + MDC in Sprint 2+. |

### B4 — Empty Maven POM metadata stubs

| | |
| --- | --- |
| Path / subsystem | `backend/pom.xml` |
| Issue | Empty `url`, `license`, `developers`, `scm` from Initializr. |
| Why it can wait | No runtime effect. |
| Recommended future action | Fill or delete on the next POM touch. |

### B5 — Duplicated `InMemoryUserAccountRepository` in tests

| | |
| --- | --- |
| Path / subsystem | `backend/src/test/java/com/offerbuddy/auth`, `backend/src/test/java/com/offerbuddy/user` |
| Issue | The same test double is copied in three test classes. |
| Why it can wait | Tests pass; duplication is local to tests. |
| Recommended future action | Extract one shared test double. |

### B6 — Backend image rebuilds Maven inside Docker after CI already verified

| | |
| --- | --- |
| Path / subsystem | `backend/Dockerfile`, `.github/workflows/backend-ci.yml` |
| Issue | CI runs `mvn verify`, then the image build compiles again. |
| Why it can wait | Correct and reproducible; only costs CI time. |
| Recommended future action | Copy the CI jar into a runtime image. |

### B7 — Applications pagination is local state only; status edits do not refetch totals

| | |
| --- | --- |
| Path / subsystem | `frontend/src/pages/ApplicationsPage.tsx` |
| Issue | `page` is not in the URL. Inline status change updates current rows only, so `totalElements` / `totalPages` can go stale. |
| Why it can wait | Daily volume is small; refresh returns to page 1. |
| Recommended future action | Sync page to search params; refetch after filtered status edits. |

### B8 — No GitHub Release / tag workflow yet

| | |
| --- | --- |
| Path / subsystem | `.github/workflows` |
| Issue | Close-out still expects a tag (for example `v0.2.0`). Deploy is SHA-based and works without tags. |
| Why it can wait | SHA deploy already works; tagging is a process step at close. |
| Recommended future action | Add a tag/release step after merge to `main`. |

---

## C. No Action Required

Things that may look unusual but are intentional or acceptable at the current scale.

| Item | Why it is acceptable |
| --- | --- |
| Local DB/OAuth placeholders in `application.yaml` | Dev convenience. The `prod` profile refuses empty values and the local defaults (`offerbuddy123`, `local-dev-client-id`, `local-dev-client-secret`). |
| Tracked `frontend/.env.development` and `frontend/.env.production` | Only public origins. Not secrets. CI also sets `VITE_BACKEND_ORIGIN` at build time. |
| Swagger/OpenAPI in local, disabled in prod | `application-prod.yaml` sets `springdoc` enabled to false. |
| `ApplicationService` ~220 lines | CRUD + list validation in one service is an accepted Sprint 1 decision. Do not split now. |
| CSRF cookie is not HttpOnly | Required so the SPA can send `X-XSRF-TOKEN`. The session cookie remains HttpOnly. |
| Local Compose publishes Postgres 5432 and Redis 6379 | Host-only development. Production Compose does not publish those ports. |
| `GeminiAiCompletionClientIntegrationTest` skips without `GOOGLE_API_KEY` | Intentional so CI stays hermetic. |
| CI does not run on `feature/*` / `review/*` / `chore/*` pushes | Documented: those branches are verified by PRs targeting `release` or `main`. |
| `prompt=select_account` on Google OAuth | Intentional after the silent-auth review. Consent is not forced on every login. |

---

## D. Potential Security / Production Issues

No secret values are included below. None of these require an architecture redesign. Confirm before changing production Nginx.

### D1 — Nginx proxies `/actuator/` to the public site

| | |
| --- | --- |
| Path | `infrastructure/production/nginx/offerbuddy.conf` |
| Issue | `location /actuator/` is proxied on 443. Health and info are unauthenticated. `show-details` is `never`. |
| Why it matters | The endpoint is internet-visible even if it only confirms liveness. |
| Proposed fix | Stop proxying `/actuator/` from 443, or restrict by IP. Keep the Docker healthcheck on loopback. |
| Risk | Low |

### D2 — `www` and apex are both served; CORS allows only `FRONTEND_BASE_URL`

| | |
| --- | --- |
| Path | `infrastructure/production/nginx/offerbuddy.conf`, `backend/.../CorsConfig.java`, `frontend/.env.production` |
| Issue | Nginx `server_name` includes `www.offerbuddy.io` and `offerbuddy.io`. CORS allows only `FRONTEND_BASE_URL` (apex). The frontend bundle calls `https://offerbuddy.io`. |
| Why it matters | `https://www.offerbuddy.io` would load the SPA then fail API/CORS against the apex origin. |
| Proposed fix | 301 `www` → apex (preferred), or add `www` to CORS and cookie domain. |
| Risk | Medium |

### D3 — Logout clears `JSESSIONID` without `Secure` / `SameSite`

| | |
| --- | --- |
| Path | `backend/src/main/java/com/offerbuddy/auth/LogoutController.java` |
| Issue | Production session cookie is `Secure` + `SameSite=Lax`. Logout sets `Max-Age=0` with `HttpOnly` and `Path=/` only. |
| Why it matters | Some browsers treat cookies with different `Secure` flags as distinct, so the real session cookie may not be cleared. |
| Proposed fix | Match `Path`, `Secure`, `HttpOnly`, and `SameSite` when expiring the cookie. |
| Risk | Medium |

### D4 — Production Redis has no password and is unused

| | |
| --- | --- |
| Path | `infrastructure/production/docker-compose.yml` |
| Issue | Redis is started with default (no) authentication. It is not published to the host. |
| Why it matters | Residual attack surface on the Compose network for an unused service. |
| Proposed fix | Remove from production Compose until needed, or require a password. |
| Risk | Low |

---

## Configuration vs secrets

### Non-sensitive (examples / Git OK)

- `FRONTEND_BASE_URL`
- `VITE_BACKEND_ORIGIN`
- `POSTGRES_DB`
- `POSTGRES_USER`
- `POSTGRES_PORT`
- `SPRING_PROFILES_ACTIVE`
- `BACKEND_IMAGE` name/tag
- Nginx `server_name`

### Secrets (env / GitHub Environment only)

- `POSTGRES_PASSWORD` / `DB_PASSWORD`
- `GOOGLE_CLIENT_SECRET`
- `GOOGLE_API_KEY`
- `EC2_SSH_KEY`
- `GHCR_READ_TOKEN`

`GOOGLE_CLIENT_ID` is not a password, but production must not use the local default.

`.env.example` files list the required keys with placeholders. They do not contain live secrets. `GOOGLE_API_KEY` is correctly documented as optional.

---

## CI/CD vs intended branch model

Intended:

```text
feature/*  →  release  →  verify  →  main  →  tag  →  deploy SHA
```

Current workflows already deploy by immutable SHA and never deploy from a branch tip automatically.

Gaps:

1. Backend CI also publishes images from `release`, so workflow_dispatch `latest` can select code that is not yet on `main` (see A12).
2. There is still no tag/release workflow (see B8).
3. `review/*` pushes do not run CI until a PR targets `release` or `main` (intentional; see C).

---

## Suggested commit boundaries (after approval)

1. **Repository / configuration** — A1, A2, A4, A5, A6, A7
2. **Backend** — D3 (if approved), plus any tiny comment/README leftovers in backend
3. **Frontend** — A3, A8, A9, A10, A11
4. **CI / docs** — A12
5. **Production / security** — D1, D2, D4 only if explicitly included

After each group: run the relevant tests and build, then report exactly what changed.

---

## Definition of Done (cleanup)

Sprint 1 cleanup is complete when:

- no obvious debug artifacts remain
- no production secrets are committed
- `.gitignore` is appropriate
- `.env.example` reflects required configuration safely
- no clearly obsolete files remain
- backend and frontend builds pass
- existing meaningful tests pass
- CI configuration remains valid
- production configuration remains valid
- larger improvements are recorded as technical debt instead of expanding Sprint 1
- no Sprint 2 functionality has been introduced
