# Sprint 3 Backend, API, and Async Implementation Plan

## Purpose

This unit translates frozen Phase 3 backend contracts into implementation boundaries. It uses the [scope lock](implementation-baseline.md) and [dependency/migration plan](dependency-and-migration-plan.md); frozen §3.17 remains authoritative for HTTP semantics and §3.18 for package/layer boundaries.

## Shared backend foundations

These foundations precede capability implementation because several slices consume them.

| Foundation | Reuse / change | Completion boundary |
| --- | --- | --- |
| Security context and ownership lookup | Reuse Spring Security, Google session identity, Extension credentials, and server-owned user resolution | Candidate-, Job-, Preparation-, artefact-, and Admin-scoped services can resolve ownership without accepting client ownership claims |
| Common API errors | Extend existing `ApiExceptionHandler`/`ErrorResponse` to frozen §3.17 codes and `X-Request-ID` semantics | MVC tests cover validation, inaccessible resources, OCC, idempotency, and safe internal failures |
| Aggregate concurrency | Add Application `version`, Job `contentVersion`, Candidate `profileVersion`, mutable artefact version, and AI config version according to their distinct contracts | Repository updates use expected versions and surface deterministic 409 responses |
| Correlation | Add request context/MDC and durable `business_events.correlation_id` propagation | HTTP → event → retry → worker → AI retains one correlation chain without logging sensitive payloads |
| Event contracts | Extend the existing V7 writer, claim/lease, handler registry, retry, and diagnostics | S3 event types are versioned, payload-minimised, and safe under duplicate delivery |
| AI runtime | Put semantic capability ports in front of the existing Gemini integrations; add provider router, runtime config, safe execution metadata, and bounded fallback | Business modules have no provider SDK/config dependency and can be tested with fake capability implementations |
| Generation coordination | Add Preparation-domain operations and artefact heads without a generic task system | Duplicate commands and out-of-order completions converge to one valid current result |
| File/asset boundary | Add secure file validation, object-storage abstraction, rendering boundary, and authorised download flow | Database stores metadata only; AI, rendering, storage, and download failures remain separable |

Shared-foundation changes should be narrow. Existing S2 Application events, Job Intelligence handlers, analytics projection, health endpoints, and Fast Path transactions remain operational throughout incremental delivery.

## Backend capability units

| Unit | Prerequisites | Backend/API responsibility | Persistence / async | Verification boundary |
| --- | --- | --- | --- | --- |
| Candidate Profile | V10–V11; security/OCC | Candidate aggregate commands/queries and frozen `/api/candidate/profile` contract | Candidate root/children; short local transactions | Ownership, partial profile, aggregate replacement, stale-write 409, no Fast Path dependency |
| Resume Import | Candidate; V12; AI/event foundations; file boundary | Multipart import, status/draft read, explicit accept command | Import/draft state; async extraction event; atomic Candidate acceptance | PDF/DOCX validation, retry, duplicate accept, profile-version conflict, no unreviewed fact mutation |
| Job revision delta | V10; existing Job service and V8 intelligence | Prepared-before-Application Job lifecycle, controlled update, current intelligence commands/reads | Existing Job/analysis tables extended with source versions | Existing Web/Extension create remains compatible; semantic changes increment only `contentVersion` |
| Preparation context | Candidate and Job projections; V13 | Candidate + Job create/get and composed readiness/status read | One Preparation per Candidate + Job; no large workflow state machine | No Application creation, capability availability, ownership, stale summary calculation |
| Match Analysis | Preparation; AI/generation foundations; V13 | Generate/read/regenerate through frozen Preparation sub-resource contract | Match artefact/children, evidence, generation identity, head selection | Grounded references, score validation, duplicate command, stale sources, retry/failure |
| Base/Tailored Resume | Candidate/Match; V14; AI/file foundations | Base Resume selection/read, async generation, limited artefact edit, render/download | Documents, sections/evidence, assets, provenance, generation and OCC versions | Fact grounding, multiple variants, edit conflict, out-of-order completion, render/storage isolation |
| Focused Cover Letter | Match; V15; AI/file foundations | Generate/read/edit/re-render focused artefact | Cover Letter/evidence/assets with provenance and OCC | Candidate fact boundary, edit conflict, stale display, no Default Cover Letter mutation |
| Application integration | Preparation summary; Application OCC | Preserve existing `/api/applications/**`; expose only the additive summary needed by existing detail UI | Existing Application schema plus `version`; no Preparation FK ownership | Existing create/update/history/analytics tests plus no/in-progress/ready/stale Preparation states |
| Sponsor reference data | V18; RuoYi/Admin integration; Redis configuration | Working/import data service, validate/publish command, published snapshot/refresh and lookup contracts | PostgreSQL authority; active Redis representation; versioned client projection | Failed import/publish preserves active version; cheap local negative lookup; no second source of truth |
| AI operations | V16–V17; AI runtime; Admin boundary | Capability configuration, routing validation, safe aggregate monitoring/diagnostics | Config OCC, execution metadata, safe audit; no raw content | Disabled capability, invalid route, missing secret, safe telemetry, Admin RBAC |

## API implementation rules

- Implement the frozen §3.17 surface additively; do not introduce a generic operation endpoint, generic PATCH framework, public Job catalogue, or provider-selecting business request.
- Keep DTOs separate from JPA entities and module snapshots. Candidate/Profile aggregate DTOs do not become generic cross-module objects.
- User-facing endpoints derive the user from authenticated context. RuoYi operational APIs use their independent RBAC and do not reuse end-user session authority.
- Async commands persist business state and event intent in one short transaction, normally return `202`, and expose the business resource for polling.
- Ordinary edits and acceptance commands remain synchronous short transactions.
- Preserve the S2 Application client workflow. Internal Job canonicalisation reuse must not force new client choreography.
- Treat error codes, version fields, idempotency semantics, and resource statuses as contract tests shared with Web, Extension, and Admin clients.

## Async implementation model

```text
HTTP command
  → authorise and validate source versions
  → short transaction: business operation + business_event
  → commit / 202 business resource
  → existing event claim and lease
  → reload authoritative state and re-check admissibility
  → external/AI execution outside transaction
  → short transaction: result/failure + artefact head
  → poll business resource
```

Handler implementation must preserve V7 at-least-once delivery. `eventId`, `generationId`, `correlationId`, claim token, and artefact version have separate meanings. Claim-token fencing and unique generation bindings protect database effects; they do not promise exactly-once provider invocation.

No AI/provider, object-storage, HTTP, document parsing, or rendering operation may run inside a database transaction. Technical retry reuses the same generation identity; user-requested regeneration creates a new generation identity and reserved artefact version.

## Package and ownership boundaries

New backend code follows existing `com.offerbuddy` module conventions while applying frozen §3.18 layering:

```text
candidate          authoritative facts and resume import
job                canonical Job and S3 revision delta
jobintelligence    reused semantic analysis
preparation        context, Match, Resume, Cover Letter
ai                 capability ports, routing, config, usage/governance
sponsor            reference dataset, publication, lookup
events             reused generic delivery infrastructure
application        existing lifecycle plus OCC/integration only
```

Cross-module calls target application-facing reader/command ports and immutable snapshots. Candidate, Job, Application, Preparation, AI, and Sponsor repositories remain private. Avoid moving domain behaviour into `shared`; shared code is limited to genuinely generic HTTP/security/diagnostic primitives.

## Implementation and integration checkpoints

| Checkpoint | Entry condition | Exit evidence |
| --- | --- | --- |
| Backend foundation | V10 plan accepted; §3.17/§3.18 frozen | Common error/OCC/correlation/event/AI port tests pass without S3 feature UI |
| Candidate foundation | V11 applied | Candidate API and repository integration tests pass |
| Independent ingestion/job branches | Candidate and shared contracts stable | Resume Import and Job revision/Intelligence tests pass independently |
| Preparation/Match | Candidate + Job projections stable; V13 applied | Preparation and Match resource lifecycles pass service/MVC/integration tests |
| Artefacts | Match/generation/storage contracts stable | Resume and Cover Letter generation/edit/render tests pass independently |
| Operational branches | AI runtime and Sponsor publication contracts stable | RuoYi-facing service contracts and published Extension projection pass |
| Application convergence | Preparation summary stable | Full S2 Application regression plus additive summary integration passes |

These checkpoints define integration readiness, not final Sprint order or GitHub Issue numbering.

## Backend readiness and risks

Backend implementation is ready to decompose against this plan. Material watch items are shared migrations, common error DTOs, event classes, and generation tables: concurrent branches should not edit them without a single owner or pre-agreed contract. V8 Job Intelligence compatibility requires the V10 legacy-row policy already recorded in the migration plan. Object storage and product Redis are new product-backend integrations and must degrade without breaking the Application Fast Path.
