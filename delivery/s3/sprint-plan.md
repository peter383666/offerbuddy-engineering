# Sprint 3 – Candidate-Aware Application Preparation

# Sprint Information

| Item | Value |
| --- | --- |
| Sprint | Sprint 3 |
| Status | Implementation planning complete; ready for final review and Issue creation |
| Primary capability | Candidate-aware Focused Application Preparation while preserving the Sprint 2 Fast Path |
| Planning branch | `docs/sprint-3` |

---

# Overview

Sprint 3 adds a deliberate Preparation workflow around a Candidate and a captured Job. It delivers an explainable Match Analysis, Tailored Resume, and Focused Cover Letter; expands supported Extension behaviour to LinkedIn and sponsor signals; and adds limited Sponsor and AI operational surfaces to the existing RuoYi Admin application.

The sprint extends the current Spring Boot modular monolith, React Web app, Chrome Extension, PostgreSQL/Flyway persistence, durable `business_events`, and separate RuoYi Admin deployment. The frozen requirements, architecture, Phase 3 technical design/API contract, and approved Phase 4 page specifications remain authoritative. This plan defines implementation order and delivery boundaries only.

---

# Sprint Goal

Enable a user to maintain an owned Candidate Profile, prepare against a captured Job, understand an evidence-grounded Match, and produce reviewable application artefacts without weakening the existing low-friction Application-recording path. Deliver the supporting AI routing, Sponsor publication, Extension, security, provenance, concurrency, and operational controls as bounded additions to the existing system.

# Scope

## In Scope

- Candidate Profile and reviewed Resume Import.
- Candidate + Job Preparation context and source-version-aware Job Intelligence.
- Explainable Match Analysis.
- Base Resume assets, Tailored Resume generation and approved limited editing, Focused Cover Letter, rendering, preview, and download.
- Additive Preparation summary on the existing Application Detail page.
- LinkedIn Extension adapter, floating-assistant Preparation hand-off, Sponsor local signal, and SEEK deterministic cover-letter assistance.
- Semantic AI capability ports, routing, runtime configuration, safe execution metadata, and operational monitoring.
- Sponsor working/import dataset management, validation, publication, active backend/Redis representation, and versioned Extension snapshot refresh.
- Forward-only V10–V18 Flyway migrations, automated verification, regression, deployment rehearsal, and release evidence.

## Out of Scope

The exclusions locked in the [Implementation Baseline](implementation-baseline.md) remain controlling. Sprint 3 does not introduce Auto Apply, crawler ingestion, ATS simulation, hiring prediction, interview coaching, multiple Candidate personas, a full Resume Designer, a generic AI workflow platform, a prompt playground, a secrets manager, a new Admin shell, microservices, a message broker, or a Saved Job workflow.

# Delivery Workstreams

| Workstream | Implementation outcome | Primary planning source |
| --- | --- | --- |
| Shared foundation | V10 revisions, common errors/OCC, correlation propagation, event extensions, semantic AI runtime foundation | [Backend, API, and Async Plan](backend-api-async-plan.md) |
| Candidate and import | Owned Candidate aggregate, Profile Web states, async import draft, explicit review/merge | [Client and Admin Integration Plan](client-admin-integration-plan.md) |
| Job and Preparation | Job `contentVersion`, Intelligence provenance, Preparation readiness/context and Application summary | [Dependency and Migration Plan](dependency-and-migration-plan.md) |
| Match and artefacts | Grounded Match, Base Resume storage/rendering, Tailored Resume revisions/OCC, Focused Cover Letter | [Backend, API, and Async Plan](backend-api-async-plan.md) |
| Extension integration | LinkedIn adapter, stable page lifecycle, Preparation hand-off, Sponsor snapshot/local lookup, SEEK assistance | [Client and Admin Integration Plan](client-admin-integration-plan.md) |
| Sponsor operations | Working-dataset correction/validation, explicit Publish, immutable published version, Redis active projection | [Dependency and Migration Plan](dependency-and-migration-plan.md) |
| AI operations | Capability reference bootstrap, runtime registration/configuration, RuoYi RBAC/OCC, privacy-safe monitoring | [Backend, API, and Async Plan](backend-api-async-plan.md) |
| Integration and release | Focused/Fast paths, migration upgrade, privacy/security, failure recovery, operational and documentation evidence | [Verification and Release Readiness](verification-release-readiness.md) |

Implementation-sized boundaries and safe parallel pairings are defined in [Delivery Units and Two-Agent Coordination](delivery-coordination-plan.md). Their IDs are planning references until GitHub Issues are created.

# Implementation Order

1. **Land shared foundations.** Deliver `S3-F01` before capability schema/API work: V10 aggregate versions and provenance support, durable correlation, common errors/OCC, and event contract extensions. Establish `S3-F02` semantic AI ports, provider-adapter reuse, routing, and deterministic test providers without business prompts or UI.
2. **Establish Candidate and Job inputs.** Deliver Candidate backend/API and Web, then reviewed Resume Import. Add Job `contentVersion`, Intelligence source provenance, and stale/refresh behaviour while preserving existing Job capture and Application recording.
3. **Create the Preparation spine.** Implement Candidate + Job Preparation context, readiness, summary contracts, and additive Application Detail integration. No Preparation action creates an Application implicitly.
4. **Deliver Match vertically.** Connect the accepted Preparation context to asynchronous, grounded Match generation and the approved review states. Prove idempotency, source-version provenance, stale output, retries, and terminal failure before artefact generation depends on it.
5. **Deliver artefact foundations and slices.** Establish authorised Base Resume/object storage/rendering, then Tailored Resume generation, frozen limited editing/revision behaviour and artefact OCC. Deliver Focused Cover Letter as a separate slice using the same approved AI/runtime boundaries.
6. **Extend Extension capabilities.** Build LinkedIn extraction against isolated fixtures, then floating-assistant hand-off. Integrate Sponsor snapshot refresh/local lookup and deterministic SEEK cover-letter assistance without placing AI or mandatory network access on the Fast Path.
7. **Deliver operational surfaces.** Implement Sponsor persistence/publication and active projection before Sponsor Admin/Extension integration. Implement V17 capability reference bootstrap plus runtime AI configuration and privacy-safe monitoring through existing RuoYi RBAC boundaries.
8. **Integrate and harden.** Merge accepted prerequisites into `release/sprint-3`, run clean/V9 migration paths, complete Focused/Fast/Sponsor end-to-end verification, exercise failure recovery, reconcile implementation documentation, and promote only a recorded release candidate.

Parallel work is permitted only where the [coordination plan](delivery-coordination-plan.md) identifies separate ownership. Migration numbers, shared API/security/event contracts, client entry points, and deployment configuration retain a single owner at each checkpoint.

# Testing and Verification

Each delivery-unit PR must attach focused evidence for its authoritative acceptance criteria and the applicable row in the [verification matrix](verification-release-readiness.md#delivery-unit-evidence-matrix). Integration acceptance additionally requires:

- clean V1–V18 and representative V9–V18 migration paths;
- Candidate ownership, RuoYi RBAC, authorised asset access, upload safety, secret handling, and privacy-safe telemetry;
- OCC, idempotency, stale-source behaviour, bounded retry, worker restart, and correlation across HTTP → event → worker → retry → AI;
- approved Web, Extension, and Admin loading/empty/error/conflict/stale states against frozen §3.17;
- the complete Sprint 3 Focused Preparation path;
- Sprint 2 Fast Path regression with AI unavailable;
- Sponsor working data → Publish → Redis/backend representation → Extension snapshot → local lookup;
- production-like configuration, backup/restore decision points, smoke checks, and release notes.

# Definition of Done

Sprint 3 is done when:

- all accepted capability behaviour matches the frozen S3 requirements, architecture, technical design/API contract, and approved page specifications, with any compatible implementation delta recorded;
- delivery units are implemented through focused branches/PRs in dependency order, reviewed, and integrated without unresolved shared-file or migration conflicts;
- V10–V18 migrations pass clean and V9 upgrade verification while V1–V9 remain immutable;
- Candidate, Preparation, Match, artefact, Extension, Sponsor, and AI operational boundaries enforce ownership, RBAC, provenance, concurrency, idempotency, privacy, and failure isolation;
- the Focused Path and retained Fast Path pass automated and production-shaped end-to-end checks;
- required CI, security/privacy review, observability, deployment rehearsal, and rollback/forward-fix evidence pass for an identified candidate SHA;
- implementation changes are reconciled into current-system and Sprint documentation, known limitations are explicit, and the increment is ready for Sprint Review and release/tag closure.

# Risks and Watch Items

- AI latency, malformed output, or provider unavailability must remain outside core transactions and must not block Application recording or deterministic Fast assistance.
- Candidate/Job changes can make Match and artefacts stale; distinct version meanings and explicit regeneration must prevent silent replacement.
- Resume Import and generated assets contain sensitive data; upload, storage, provider context, logging, monitoring, and download boundaries require targeted review.
- LinkedIn/SEEK DOM and SPA changes can invalidate extraction assumptions; isolated adapters, fixtures, lifecycle cleanup, current-page checks, and safe failure are required.
- Sponsor freshness spans publication, Redis activation, snapshot refresh, and local lookup; version visibility and invalidation must prevent a second source of truth.
- V10–V18 is a dense migration sequence; numbering reservation, clean/upgrade tests, and immutable merged scripts are mandatory.
- Shared Web, Extension, Admin, event, security, and configuration files are conflict-prone; use the single-owner checkpoints in the coordination plan.
- Implementation Issues must link the governed S3 requirements, architecture, technical design/API contract, and UI/UX copies now published in this repository; external working records are not implementation authorities.

# Sprint Backlog and Delivery Handoff

```text
Frozen S3 baseline
→ Phase 5 delivery-unit IDs
→ GitHub Issues with acceptance criteria and authoritative links
→ focused feature branches and Agent checkpoints
→ PR verification, CI, and review
→ release/sprint-3 integration
→ release-candidate gates and Sprint Review
→ Retrospective
→ merge to main and tag engineering-s3
```

Issue creation is a later delivery action. Each Issue should map to one delivery unit (or a justified smaller split), state its prerequisites, link frozen acceptance criteria, select verification gates, and name any shared-file owner. It must not copy entire Phase 3 or Phase 4 specifications.

The proposed GitHub backlog for two-agent parallel delivery is [github-issue-backlog.md](github-issue-backlog.md).

The Engineering documentation lifecycle follows the established convention: work on `docs/sprint-3`, complete final documentation review, merge to `main`, then tag the merged commit as `engineering-s3`. Tagging before review/merge is not part of the release process.

---

# Expected Outcome

At the end of Sprint 3, a user can maintain a Candidate Profile, capture a supported Job, assess a grounded Match, and prepare reviewable resume and cover-letter artefacts through the Focused Path. The existing Fast Path remains available and independent of optional AI processing. Sponsor and AI operations are controlled through the existing backend/Admin boundaries with traceable, privacy-safe behaviour.

---

# Related Documents

- [Implementation Baseline and Scope Lock](implementation-baseline.md)
- [Dependency, Vertical Slice, and Migration Plan](dependency-and-migration-plan.md)
- [Backend, API, and Async Implementation Plan](backend-api-async-plan.md)
- [Client and Admin Integration Plan](client-admin-integration-plan.md)
- [Delivery Units and Two-Agent Coordination](delivery-coordination-plan.md)
- [Verification and Release Readiness](verification-release-readiness.md)
- [Product Backlog](../product-backlog.md)
- [Testing Strategy](../../quality/testing-strategy.md)
- [Definition of Done](../../quality/definition-of-done.md)
- [Development Workflow](../../operations/development-workflow.md)
- [Documentation Governance](../../operations/documentation-governance.md)
