# Sprint 3 Architecture Overview

## Status and Authority

This document publishes the frozen Sprint 3 Phase 2 architecture baseline. It consolidates the final architecture review and the accepted Candidate, Match, artefact, Extension, AI, Admin, event, data-ownership, security, and deployment decisions. Phase 3 may select classes, schemas, endpoints, algorithms, and libraries within these boundaries but must not silently reopen them.

## Architectural Goal

Sprint 3 extends the existing OfferBuddy modular monolith with an optional candidate-aware Preparation path while preserving Sprint 2 Application recording and tracking.

```text
Fast Path
  Job site → user applies → Extension records → Application → tracking

Focused Path
  Candidate + captured Job → Preparation → Match
  → optional Tailored Resume / Cover Letter
  → user applies → existing Application tracking
```

The Focused Path enhances the product; it does not replace or become a prerequisite for the Fast Path.

## System Shape

Sprint 3 retains:

- one Spring Boot modular-monolith backend;
- one customer React Web application;
- the Chrome Extension as a lightweight client with isolated Site Adapters;
- PostgreSQL as authoritative relational persistence;
- Flyway for forward-only schema evolution;
- the existing database-backed `business_events` processor for durable asynchronous work;
- the separately deployed RuoYi application as the operational Admin subsystem;
- Redis only for approved runtime/cache/reference projections, never as canonical truth.

Sprint 3 does not introduce microservices, a broker, a workflow engine, an agent framework, or a second customer-domain backend.

## Domain Ownership

| Module/domain area | Owns | Must not own |
| --- | --- | --- |
| Identity | authenticated OfferBuddy identity and user resolution | Candidate or Application business data |
| Candidate | Candidate Profile, reviewed import drafts, factual candidate evidence and profile versions | Job-specific analysis or generated application artefacts |
| Job | canonical shared Job facts and Job Intelligence | Candidate truth, Match decisions, or Applications |
| Application | user-owned application lifecycle, submitted-material references/snapshots, tracking integration | Preparation lifecycle or generated-content authoring |
| Preparation | Candidate + Job preparation context and sibling Match, Resume, and Cover Letter capabilities | Candidate/Job canonical facts or implicit Application creation |
| AI platform | semantic capability execution, provider routing, runtime policy, execution metadata and safe diagnostics | business prompts, domain truth, or provider choice exposed to callers |
| Event infrastructure | durable event transport, claiming, retry, recovery and dispatch | domain outcome decisions |
| Analytics | derived Application tracking views | canonical Application state |
| Sponsor/reference data | working import, validation, publication and published projection | Candidate or Application ownership |
| RuoYi Admin | authorised operational UI and Admin identity/RBAC | universal customer-data access or direct ownership of OfferBuddy business rules |

Persistence entities and repositories remain private to their owning module. Cross-module access uses application-facing contracts, immutable snapshots/projections, commands, or domain-owned events.

## Core Architectural Decisions

### Preparation Is Not an Application Child

Preparation starts before the user applies and is anchored by Candidate + Job. It may later be linked from an Application, but Application is not its aggregate root and the Preparation flow must not create an Application implicitly.

### Candidate Is the Factual Source of Truth

Candidate Profile owns reusable candidate facts and evidence. Resume Import produces a reviewable draft; extraction never writes accepted Candidate truth directly. Resumes and Cover Letters are derived/user-reviewed artefacts and must not create an independent factual universe.

### Preparation Contains Sibling Capabilities

Match, Resume, and Cover Letter live inside one Preparation domain area with explicit internal boundaries. They can share a Preparation context and immutable source projections, but none owns the others. Match may inform artefact generation; it is not canonical Candidate or Job truth.

### Match Is Explainable Analysis

Match combines Candidate evidence and Job Intelligence into strong, partial, gap, evidence, and important-requirement outcomes. A numeric score is secondary and must not claim hiring probability or ATS certainty.

### Generated Artefacts Are Versioned Outputs

Role/Base Resumes are reusable inputs. Job-specific Resume and Cover Letter outputs retain generation identity and source-version provenance. Approved limited editing follows the frozen revision model and optimistic concurrency control; historical accepted revisions are not silently mutated or regenerated.

### Staleness Is Source-version Based

Candidate Profile, Job content, generation attempts, and artefact revisions have distinct versions. Derived output freshness is determined from recorded source versions, not timestamps or completion order alone. A stale result remains visible where approved and regeneration requires a deliberate action.

### AI Is Infrastructure Behind Semantic Domain Ports

Business modules depend on capability-specific interfaces such as Match or artefact generation, not provider SDKs or a generic prompt API. Business modules own prompts, validation, and domain result interpretation. The AI platform owns provider routing, runtime configuration, timeout/retry policy, usage metadata, and adapter integration.

### RuoYi Remains a Sideline Admin Subsystem

RuoYi provides Sponsor working-data operations, AI runtime configuration, and privacy-safe monitoring through frozen RBAC boundaries. Simple operational reads and configuration writes use approved contracts/persistence boundaries; complex business behaviour is executed by OfferBuddy domain services. RuoYi is not part of the OfferBuddy module graph and cannot bypass user ownership.

## Dependency Rules

Permitted interaction patterns:

- synchronous query through an owning module's application-facing query contract;
- synchronous command through the owning module when an immediate business outcome is required;
- durable event for completed facts and decoupled downstream work;
- infrastructure adapters behind domain/application ports.

Forbidden dependencies:

- Application domain depending on Preparation internals;
- Candidate or Job Core depending on Preparation;
- direct cross-module repository or persistence-entity access;
- business modules depending on AI provider SDKs;
- provider adapters owning business prompts or business validation;
- Extension or Web clients deriving authoritative ownership, freshness, or Match truth;
- RuoYi becoming a peer domain module inside OfferBuddy.

Orchestration belongs to the application service closest to the user intent. Transactions remain inside the smallest owning boundary and do not include AI/provider, rendering, object-storage, or other long external calls.

## Data and Persistence Architecture

PostgreSQL remains authoritative. Each table and write path has one owning module even where OfferBuddy and RuoYi share a PostgreSQL deployment. Database constraints provide final protection for identity, uniqueness, idempotency, version, and publication invariants.

Redis may hold active Sponsor lookup representations, approved runtime/cache data, and versioned projections that can be rebuilt from authoritative data. Redis loss or unavailability must not corrupt canonical state. Extension snapshots are refreshable published-data copies for page-time lookup, not a second source of truth.

All Sprint 3 schema changes are additive forward Flyway migrations after V9. Existing migrations remain immutable.

## Event and Long-running Operation Architecture

Sprint 3 extends the existing PostgreSQL-backed `business_events` mechanism. The model remains at-least-once and requires:

- atomic persistence of a domain write and its event where applicable;
- bounded claim leases, retry and abandoned-claim recovery;
- idempotent handlers and stable business identities;
- durable visible operation/generation state;
- external calls outside database transactions;
- correlation across HTTP, event, worker, retry and AI execution;
- terminal failure that is observable and recoverable without corrupting the last accepted state.

AI generation attempts are domain operation records, not replacements for transport events. Events notify or schedule work; the owning domain record represents the user-visible lifecycle and accepted outcome.

## Extension Architecture

The Extension remains an untrusted, lightweight assistant. It reuses the Site Adapter boundary and common extracted Job representation, adding LinkedIn as a best-effort adapter without coupling SEEK/Indeed implementations.

It may detect supported page facts, show approved signals, record/reuse a Job or Application, open contextual Web Preparation, use a versioned Sponsor snapshot locally, and provide deterministic SEEK assistance. It must not own Candidate editing, Match generation, artefact editors, canonical Sponsor data, provider selection, or domain authorisation. Dynamic-page lifecycle handling must clear stale context and avoid duplicate observers/listeners.

## Sponsor Publication Architecture

```text
official import/source
→ working dataset
→ authorised correction and validation
→ explicit Publish
→ immutable published version
→ backend/Redis active representation
→ Extension snapshot refresh/cache
→ page-time local lookup
```

The backend/published dataset remains authoritative. Admin CRUD-style screens operate only on the working/import dataset and cannot mutate the active published version in place. Failed import, validation, publication, activation, or refresh leaves the last known good version usable.

## Security, Privacy, and AI Safety

Security follows server-derived identity and resource ownership chains. Candidate, Job-specific Preparation, artefacts, and Applications are never authorised from client-supplied owner IDs. RuoYi uses its own authenticated Admin identity and least-privilege permissions.

Sensitive Candidate/resume/Cover Letter content is minimised across provider requests and excluded from URLs, ordinary logs, metrics, events, Admin monitoring, and unsafe error details. Provider secrets remain in the approved secret boundary and are never returned to Admin clients.

Uploaded files require size/media validation, safe naming, controlled parsing, isolated failure, and cleanup. Artefact preview/download requires authorised, expiring access rather than public or guessable object paths.

Job text and user-provided content are untrusted AI input. Prompt-injection defence relies on capability-scoped prompts, explicit data/instruction separation, minimal context, structured output validation, truthfulness checks, and refusal to let source content select tools, providers, secrets, or system behaviour.

## Runtime and Failure Isolation

Core dependencies are the backend, PostgreSQL, and existing authentication/application path. Degradable or optional dependencies include AI providers, rendering/object storage for relevant actions, Redis-backed acceleration/projection, and Admin availability.

Failure of an optional dependency must remain local:

- AI failure must not block Application recording;
- one generation failure must not remove the last accepted artefact;
- Sponsor refresh failure must preserve the last known good published dataset/snapshot;
- monitoring failure must not change domain outcomes;
- Extension unsupported/extraction failure must fail safely without fabricating data.

Deployment remains compatible with the current small-scale topology. New runtime configuration is explicit, secrets are externalised, health/readiness distinguishes required from optional dependencies, and no decision assumes distributed exactly-once processing.

## Final Data Flows

### Fast Application

```text
supported Job page
→ user applies
→ Extension/backend records or reuses Job and Application
→ durable downstream events
→ existing tracking and Analytics
```

### Prepared Application

```text
Candidate projection + Job projection
→ persistent Preparation
→ optional Match operation
→ selected Role/Base Resume
→ optional Tailored Resume and Cover Letter operations
→ user review and authorised export
→ user applies on original platform
→ existing Application tracking + submitted-material references
```

### Candidate Resume Import

```text
authorised upload
→ isolated storage and async extraction
→ CandidateImportDraft
→ user review/correction
→ explicit accept/merge with Candidate OCC
→ Candidate Profile new version
```

### AI Governance

```text
business capability request
→ semantic capability port
→ runtime policy/router
→ provider adapter
→ validated domain result

Admin configuration/audit and safe execution metadata
remain outside provider secrets and private content
```

## Accepted Risks

- Sharing PostgreSQL with RuoYi increases boundary-discipline requirements; ownership, permissions and code-level access remain explicit.
- In-process async processing has limited horizontal sophistication; durable claims, leases, idempotency and recovery are sufficient at the current scale.
- External AI providers introduce latency, availability, cost and factual-error risks; capability isolation, validation, provenance and evidence boundaries mitigate them.
- Browser DOM integration changes over time; isolated adapters, fixtures, monitoring and safe failure are required.
- Generated language can still be imperfect; user review remains mandatory and AI never becomes Candidate truth.

These risks are accepted for Sprint 3 and do not justify premature distributed infrastructure.

## Frozen Decisions

Phase 3 and implementation must not reopen:

- the modular monolith and current deployment shape;
- Candidate as factual source of truth;
- Preparation anchored by Candidate + Job and separate from Application ownership;
- Match as explainable derived analysis;
- versioned/provenanced generated artefacts and deliberate regeneration;
- semantic AI ports with provider adapters behind the domain;
- RuoYi as a bounded operational Admin subsystem;
- PostgreSQL/Flyway and the extended `business_events` model;
- Extension as a lightweight client with isolated Site Adapters;
- optional/degradable AI, Redis, rendering, and Admin dependencies;
- preservation of the Sprint 2 Fast Path.

Implementation freedom remains for class/package details, indexes, exact DTO mapping, provider libraries, polling intervals, rendering libraries, and other choices explicitly delegated by the frozen Phase 3 technical design.
