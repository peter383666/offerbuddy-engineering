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

The highest-priority documentation conflict is the **tracking model**: Final UI/UX v2.0 and Extension Design emphasise explicit **Save to OfferBuddy**, while `extension-application-tracking.md` and the shipped extension also implement confirmation-based automatic tracking. Code contains both paths.

## Reconciliation Matrix

| Area | Planned | Implemented | Difference | Documentation action |
| --- | --- | --- | --- | --- |
| Extension platforms | SEEK + Indeed only | SEEK + Indeed adapters; LinkedIn rejected | None | Confirm; refresh “not started” banners |
| Explicit Save | UI v2 / extension-design: explicit Save | Popup Save → Track API | None (for this path) | Keep as primary documented user action |
| Automatic tracking | Tracking spec: confirm submission → auto track; uncertain → ask user | Lifecycle adapters + confirmed ingest + companion confirmation | Design delta | Choose authority: dual-path product, or explicit-only. Update extension-design, UI v2 notes, tracking spec, and extension README together |
| Eligibility findings | Requirements list citizenship, PR, clearance, working rights, sponsorship | UI and extractors surface citizenship/PR only; tests assert other findings are not surfaced | Implementation detail vs requirements; None vs UI v2 handoff | Close handoff: S2 surfaces citizenship/PR only; mark broader findings deferred |
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
| CI completeness | Plan expects broad automated verification | Backend verify strong; frontend CI lint+build only; no extension workflow | Design delta | Record as known limitation / follow-up |
| Living architecture status | Several docs still say S2 not implemented | Code implements S2 vertical | Doc action | Sync living architecture during Agent 3 |
| `ui-ux/v1/` | Earlier labeled final | Conflicts with v2.0 (Save model, Analytics ranges, NO_RESPONSE) | Superseded | Keep as historical; v2.0 remains UI authority |

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

## Open Decisions for Later Closeout Steps

These are documentation or follow-up decisions, not invitations to invent new S3 scope:

1. **Tracking model authority** — document dual-path as accepted S2 behaviour, or treat auto-ingest as design drift to reconcile.
2. **Eligibility contract** — formalise citizenship/PR-only for S2.
3. **Legacy `NO_RESPONSE` status** — retain with derived Analytics, or plan migration.
4. **CI gaps** — extension workflow and frontend test gate.
5. **Living docs sync** — architecture, quality, operations, and ADR extraction (Agents 3–5).

## Related

- [Implementation Status](implementation-status.md)
- [Sprint Plan](sprint-plan.md)
- [Extension Design](../../design/s2/technical/extension-design.md)
- [Extension Application Tracking](../../design/s2/technical/extension-application-tracking.md)
- [S2 Scope](../../design/s2/requirements/s2-scope.md)
