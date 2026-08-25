# Sprint 3 Client and Admin Integration Plan

## Purpose

This unit defines implementation responsibilities across the customer Web app, Browser Extension, and RuoYi Admin. Approved `S3-UI-01`–`S3-UI-20` specifications govern behaviour and Figma governs presentation. This plan records reuse, integration ownership, and verification boundaries only.

## Surface allocation

| Surface | Page specifications | Implementation responsibility |
| --- | --- | --- |
| Focused Apply Web | `S3-UI-01`–`S3-UI-05` | Match decision, Base Resume selection/preview, Tailored Resume review, Focused Cover Letter review |
| Candidate Profile Web | `S3-UI-06`–`S3-UI-14` | Profile overview/first-time state, Resume Import review, aggregate editors, Default Cover Letter entry |
| Extension / Fast Apply | `S3-UI-15`–`S3-UI-16` | Floating assistant states, Prepared hand-off, sponsor signal, SEEK cover-letter fill |
| Existing Application Detail | `S3-UI-17` | Add a secondary Preparation summary/entry to the existing S2 page |
| RuoYi Admin | `S3-UI-18`–`S3-UI-20` | Sponsor working/publish workflow, AI runtime configuration, metadata-only AI monitoring |

A Figma frame is a visual state, not automatically a route or component. Annotation nodes remain design documentation and are never rendered.

## Customer Web implementation

### Reuse baseline

Reuse the existing React Router/auth guards, `AppShell`, navigation, API client, form/button/status patterns, loading/error conventions, and Application pages. New S3 routes live inside the authenticated shell. Do not create a second design system, client state platform, or duplicate Application Detail page.

### Web delivery boundaries

| Boundary | Backend dependency | Web responsibility | Verification |
| --- | --- | --- | --- |
| Candidate Profile | Frozen Candidate aggregate API | Overview, first-time state, section editing, import entry; one `profileVersion` across meaningful edits | Loading/empty/error, validation, stale-write preservation, keyboard/forms, no Fast Path gate |
| Resume Import review | Import resource/status/accept contracts | Poll business resource, display proposals, collect explicit acceptance/merge decisions | Processing/ready/failed, no auto-accept, 409 preserves user review state |
| Preparation workspace | Preparation context/readiness | Compose Job context and available actions without inventing workflow state | Insufficient/current/stale states; command preconditions still enforced by backend |
| Match | Match resource/generation | Explainable result, strengths/gaps, regenerate stale result, continue/exit | No score-only UI, no Candidate mutation, processing/failure recovery |
| Resume | Resume generation/read/update/assets | Select/preview Base Resume, generate, limited edit/save, download/review | Selection does not generate; artefact OCC differs from provenance staleness |
| Focused Cover Letter | Cover Letter generation/read/update | Review/edit/save/regenerate and return to recruitment site | Default/Focused resources remain separate; no Application status mutation |
| Application Detail | Preparation summary read | Add one secondary section to existing page and deep-link to Preparation | Existing S2 states and updates remain unchanged when Preparation is absent/unavailable |

Client polling follows resource state from frozen §3.17. It must stop at terminal state, avoid aggressive intervals, survive navigation where required by the approved specification, and never expose event IDs, provider retries, prompts, or raw responses.

## Browser Extension implementation

### Reuse baseline

Extend the existing TypeScript `SiteAdapter`, `JobPageContext`, content-script lifecycle, service-worker messaging, credential store, API client, Shadow DOM companion, pending context, and application tracking core. SEEK/Indeed extraction and confirmed/uncertain submission behaviour remain regression baselines.

### Extension delivery boundaries

| Boundary | Change | Isolation rule | Verification |
| --- | --- | --- | --- |
| LinkedIn adapter | Add platform identity, reliable fact extraction, lifecycle/navigation handling, and partial capability flags | LinkedIn-specific DOM logic stays inside its adapter | Fixture tests for supported/partial/unsupported states; SEEK/Indeed unchanged |
| Floating assistant | Extend one presentation/state resolver for collapsed/default/sponsor/requirement/ready/applied states | Do not create separate assistants or embed Match/Resume editors | State precedence, hover/drag/accessibility, backend failure-neutral behaviour |
| Prepare hand-off | Capture/reuse Job and open authenticated Web Preparation | Must not create Application or wait synchronously for Job Intelligence | Auth pairing, duplicate capture, route/context preservation, unavailable backend |
| Sponsor snapshot | Refresh/version the backend-published projection and perform page-time local lookup | PostgreSQL/backend remains authority; no second dataset or per-page mandatory network call | Atomic cache replacement, invalidation/fallback, cheap negative lookup, safe copy |
| SEEK cover-letter assistance | Resolve fresh Focused Cover Letter, otherwise Default Cover Letter; user-triggered fill | Deterministic Default fill uses no AI; never overwrite silently or submit automatically | Source priority, field detection, existing content protection, no Applied status mutation |
| Application recording | Reuse S2 ingestion/tracking | Sponsor/Preparation/AI failures cannot block recording | Existing extension regression suite plus LinkedIn coverage |

Manifest permissions remain minimal and platform-scoped. Candidate Profile, Resume content, Match results, and provider data are not retained in extension storage. Page content and Extension messages remain untrusted inputs.

## RuoYi Admin implementation

Reuse the existing Admin backend/frontend, menus, Vue/TypeScript conventions, table/form/dialog/upload patterns, pagination, JWT/RBAC, shared PostgreSQL/Redis connectivity, audit facilities, and independent deployment. Do not create a new shell or reuse end-user Google/session authority.

| Capability | Admin responsibility | Product/backend responsibility | Verification |
| --- | --- | --- | --- |
| Sponsor working dataset | Import, constrained corrections, validation feedback, unpublished-change state, explicit Publish | Validate and persist working data; atomically activate version; rebuild Redis/published projection | Import errors preserve active version; publish RBAC/audit; no direct published mutation |
| AI runtime configuration | Enabled state, provider/model route, approved parameters, versioned save, safe secret-reference status | Validate capability compatibility, enforce config OCC, refresh runtime cache | Missing secret, invalid route, conflict, previous valid config retained |
| AI monitoring | Filters, summary metrics, provider/capability breakdown, safe failure detail | Supply metadata-only aggregates and sanitised diagnostics | Permission, partial/missing token/cost data, no user-content drill-down |

Admin must not expose plaintext credentials, raw prompt/response content, Candidate/Profile/Resume/Job Description content, or universal user-domain CRUD. Operational mutations are auditable using identifiers and safe before/after metadata.

## Cross-client contract integration

| Contract | Web | Extension | Admin |
| --- | --- | --- | --- |
| Authentication | End-user session | Paired Extension credential | Independent RuoYi JWT/RBAC |
| Request diagnostics | Display/retain `X-Request-ID` for support | Preserve safe request ID in failure feedback | Use safe request ID in operational diagnostics |
| OCC | Profile, Job where exposed, Application, Resume/CL artefacts | Application/Job command contract only | AI config and working-data concurrency |
| Async | Poll Candidate/Preparation business resources | Hand off long work to Web; no AI streaming | Show operational state only where frozen contract supports it |
| Sensitive data | Render only owned capability data | Store minimum transient context | Metadata-first; no Candidate content |

Client API modules should be organised by capability rather than one expanding global types file. Contract DTOs may be shared inside one client application but must not be copied between Web, Extension, and Admin without an explicit generated/shared-contract decision.

## Integration order and conflict controls

1. Freeze shared HTTP contract names, statuses, versions, and errors before client implementation.
2. Build Candidate Web and LinkedIn fixture extraction in parallel because their code paths do not overlap.
3. Integrate Preparation/Match Web after Candidate and Job contracts are executable.
4. Add Resume/Cover Letter surfaces after generation/edit contracts and storage responses stabilise.
5. Implement Sponsor Admin and Extension cache in parallel only after one published version/payload contract exists.
6. Implement AI Admin pages against frozen operational contracts; integrate after AI runtime services exist.
7. Converge on existing Application Detail and Extension recording last enough to minimise repeated S2 regression churn.

High-conflict files include Web routing/navigation/API base types, Extension messages/adapter exports/companion presentation, and RuoYi menu/permission seeds. A later delivery unit must allocate these files deliberately rather than allowing two branches to edit them concurrently.

## Client/Admin readiness

The client and Admin surfaces are ready for delivery decomposition. Visual implementation must load the relevant Page Specification and final Figma frame together; backend behaviour comes from frozen §3.17, not inferred from mockups. No client may compensate for unavailable backend semantics by inventing local ownership, staleness, idempotency, or AI-task rules.
