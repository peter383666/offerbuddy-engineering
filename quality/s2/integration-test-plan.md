# Sprint 2 Integration Test Plan

## Purpose

Records the integration behaviours Sprint 2 relies on and where they are proven.

## Backend Integration (automated)

Proven through Maven `verify` + Testcontainers where applicable:

1. Extension pairing create → Web approve → exchange → credential use
2. Track create and already-tracked duplicate outcome
3. Concurrent / repeated ingestion converging on uniqueness rules
4. Job + Application + history + Business Event atomic commit / rollback
5. Event claim, restart recovery, bounded retry, terminal failure visibility
6. Job Intelligence processing from job events without blocking Core responses
7. Analytics incremental projection and rebuild equivalence for reliable facts
8. Authenticated Analytics reads scoped to the server principal

## Extension + Backend (mixed)

| Path | Automated | Manual |
| --- | --- | --- |
| Adapter fixtures / lifecycle state machine | Yes | Current-site smoke |
| Pairing against local/prod Web | Partial (API) | Required |
| Save / confirmed ingest → Track API | Partial | Required |
| Companion eligibility attention | Partial | Required |

## Frontend + Backend (mixed)

| Path | Automated | Manual |
| --- | --- | --- |
| Extension connect page approve | Component-level | Required end-to-end |
| Analytics dashboard ranges | Page tests + API tests | Smoke |
| Application Detail intelligence states | Component tests + API | Smoke |
| Home discovery card | Component tests | Smoke |

## Not Automated

- Live SEEK/Indeed DOM drift against production pages
- Chrome Web Store packaged-extension install path
- Production Google OAuth / cookie / Nginx behaviour
- Live Gemini availability (conditional offline skip remains)

## Related

- [Test Strategy](test-strategy.md)
- [Extension Validation](extension-validation.md)
