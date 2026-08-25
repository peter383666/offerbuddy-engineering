# Sprint 2 Implementation Reconciliation

## Purpose

Records meaningful differences between Sprint 2 planned documentation and implemented behaviour.

Closeout rule:

> Implementation can correct design documentation, but documentation must not invent implementation.

Difference classes:

| Class | Meaning |
| --- | --- |
| None | Plan and implementation agree |
| Implementation detail | Naming, thresholds, or local shape differ; no product/architecture change required beyond doc sync |
| Design delta | Technical or UI contract should be updated to match reality (or code should be changed deliberately) |
| Architectural delta | Structural responsibility or system boundary changed |

Source-of-truth order used here:

```text
1. Implemented behaviour (application repository)
2. Merged Sprint 2 Issues / PRs
3. Final UI/UX Specification v2.0
4. Phase 3 technical design
5. Architecture overview
6. Requirements (s2-scope)
7. Older / conflicting drafts (ui-ux/v1, early notes)
```

## Summary

Sprint 2's Core vertical is implemented: Extension capture for SEEK and Indeed, pairing, Track API, Job/Application core, Business Events, Job Intelligence, Analytics, and Web integration.

Tracking model closeout: Final UI/UX v2.0 and Extension Design emphasise explicit **Save to OfferBuddy** as the primary user action. Confirmation-based automatic tracking also shipped and is an **accepted secondary client path**. Both paths use the authenticated Track API. Private Extension README wording may still emphasise explicit Save only until optionally aligned.

## Reconciliation Matrix

| Area | Planned | Implemented | Difference | Documentation action |
| --- | --- | --- | --- | --- |
| Extension platforms | SEEK + Indeed only | SEEK + Indeed adapters; LinkedIn rejected | None | Confirm; refresh “not started” banners |
| Explicit Save | UI v2 / extension-design: explicit Save | Popup Save → Track API | None (for this path) | Keep as primary documented user action |
| Automatic tracking | Tracking spec: confirm submission → auto track; uncertain → ask user | Lifecycle adapters + confirmed ingest + companion confirmation | Design delta (accepted) | Dual-path accepted for S2: explicit Save primary; confirmation ingest secondary; both use Track API |
| Eligibility findings | Requirements list citizenship, PR, clearance, working rights, sponsorship | UI and extractors surface citizenship/PR only; tests assert other findings are not surfaced | Implementation detail vs requirements; None vs UI v2 handoff | Closed: S2 surfaces citizenship/PR only; broader findings deferred |
| Pairing / credentials | Create → approve → exchange; revocable credential | Implemented end-to-end | None | Remove “mechanism deferred” language |
| Track API | `alreadyTracked`; no sync AI/Analytics | Matches | None | Confirm |
| Job identity / create-or-reuse / creation_source | Shared Job; WEB/EXTENSION | Matches (V3+) | None | Confirm |
| Status history | Real transitions; initial null→APPLIED | Implemented | None | Confirm |
| Persisted `NO_RESPONSE` status | Target model prefers derived Analytics classification; legacy handling controlled | Enum and persisted status still exist; Analytics also derives no-response | Design delta (partial) | Document legacy retention + derived Analytics rule; defer status migration if not done |
| Business Events | PostgreSQL, brokerless, at-least-once | Implemented (V7 + processors) | None | Update “no infrastructure yet” banners |
| Job Intelligence | Async; summary / responsibilities / requirements / skills | Implemented; includes `UNAVAILABLE` when description missing | Implementation detail | Document `UNAVAILABLE` |
| Analytics | Projection; ranges All time / 30 / 90 / This year; derived no-response | Matches ranges; threshold default 14 days | Implementation detail | Record threshold in analytics design |
| Redis | Not required for S2 features | Compose Redis present; unused by application code | None | Optional ops note only |
| Home Extension discovery | Show when helpful; hide when known installed | Discovery card when `not_installed` | None | Confirm |
| Application Detail intelligence | Pending / unavailable / failed / populated | Implemented including NOT_ANALYSED | None | Confirm naming in UI docs if needed |
| New Application | Secondary manual + URL prefill | Retained | None | Confirm |
| Admin | Not an S2 product deliverable | Separate admin stack exists for ops | None (scope) | Label Admin as non-S2 / ops |
| CI completeness | Plan expects broad automated verification | Backend verify strong; Extension Publish on main; Extension CI + Frontend Vitest gate on follow-up branch | Design delta (closing) | Record follow-up until CI branch merges |
| Living architecture status | Several docs still say S2 not implemented | Code implements S2 vertical | Doc action | Synced during closeout |
| `ui-ux/v1/` | Earlier labeled final | Conflicts with v2.0 (Save model, Analytics ranges, NO_RESPONSE) | Superseded | Keep as historical; v2.0 remains UI authority |
| Chrome Web Store | Release/ops activity | Listing live; Publish workflow on main | None | Recorded in release notes + extension-publishing |

## Confirmed Completed Capabilities

- Chrome MV3 Extension with SEEK and Indeed Site Adapters
- Explicit Save to OfferBuddy and authenticated Track API
- Submission-confirmation ingest path and uncertain-confirmation companion UX
- Web↔Extension pairing and revocable credentials
- Shared Job + user-owned Application create-or-reuse
- Creation source and status history
- PostgreSQL Business Events with bounded retry
- Async Job Intelligence enrichment
- Rebuildable Application Analytics with approved ranges
- Web Home, Applications, Detail, New Application, Analytics, and Extension Connect integration
- Forward Flyway migrations V3–V9

## Explicitly Not Included

Matches requirements exclusions:

- LinkedIn and broad multi-platform capture
- Auto Apply / automatic form submission
- Cover Letter, resume tailoring, match scoring
- Kafka / RabbitMQ / microservices / Kubernetes / exactly-once broker platforms
- Large Analytics/BI expansion
- Redis-backed S2 application behaviour

## Closeout Decisions (Resolved)

1. **Tracking model authority** — Dual-path accepted for S2: explicit **Save to OfferBuddy** is the primary documented user action; confirmation-based ingest is an accepted secondary client path. Both use the authenticated Track API.
2. **Eligibility contract** — S2 surfaces citizenship / permanent-residency findings only; broader eligibility concepts remain deferred.
3. **Legacy `NO_RESPONSE` status** — Retain persisted enum value for now; Analytics also derives no-response. Migration remains an engineering follow-up.
4. **CI gaps** — Extension Publish is on application `main`. Extension CI + Frontend Vitest gate are on `review/s2-ci-followups` pending merge.
5. **ADR extraction** — Sprint-scoped records live under [`decisions/s2/`](../../decisions/s2/).

## Related

- [Implementation Status](implementation-status.md)
- [Sprint Plan](sprint-plan.md)
- [Extension Design](../../design/s2/technical/extension-design.md)
- [Extension Application Tracking](../../design/s2/technical/extension-application-tracking.md)
- [S2 Scope](../../design/s2/requirements/s2-scope.md)
