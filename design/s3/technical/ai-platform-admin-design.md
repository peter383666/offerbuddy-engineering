# Sprint 3 AI Platform and Admin Governance Technical Design

## Status and Scope

This document publishes the frozen Phase 3 AI platform, provider routing, runtime configuration, governance, Sponsor Admin, and RuoYi integration design. The platform is operational infrastructure for named business capabilities, not a generic AI product.

## Capability Boundary

Business modules depend on semantic ports owned with their use case, for example Candidate Resume extraction, Match analysis, Tailored Resume generation, and Cover Letter generation. They do not call provider SDKs or submit generic prompts through a public platform API.

Each business capability owns:

- semantic input and structured output contracts;
- prompt/instruction content and version;
- minimal context assembly;
- evidence/truthfulness validation;
- interpretation of success, failure, and domain result;
- its durable business operation/generation record.

The AI platform owns:

- capability registration/reference metadata;
- runtime policy lookup;
- provider/model routing and supported settings;
- provider adapters and credential resolution;
- timeout, bounded provider retry/fallback policy;
- safe execution metadata, token/cost capture and diagnostics;
- operational configuration audit.

Provider adapters cannot write Candidate, Job, Preparation, Match, Resume, Cover Letter, Sponsor, or Application state.

## Capability Registry and Bootstrap

The frozen capability identifiers are stable reference identities shared by business ports, runtime configuration, monitoring, and Admin UI. Persistent capability-shell/reference rows are Flyway-managed in V17 because they are database reference data required before Admin configuration can address a capability.

Runtime handler/adapter registration remains application startup responsibility. Flyway must not register Java implementations, inspect provider SDKs, or choose environment credentials. Startup validates that required semantic handlers and configured providers are available and reports safe readiness/diagnostics.

Reference bootstrap is idempotent through migration history and contains no secret, environment-specific credential, or mutable operational choice.

## Runtime Configuration

Configuration is per approved capability and includes only frozen fields such as enablement, provider/model routing policy, supported limits/settings, optimistic version, and audit metadata. The exact schema/DTO comes from database/API design.

Resolution occurs when an execution attempt starts:

```text
semantic capability request
→ load current enabled runtime policy
→ resolve provider/model and credential reference
→ execute provider adapter
→ record effective safe metadata
```

The business request/event carries semantic capability and source identity, not a provider/model decision. A later configuration change does not rewrite or automatically recalculate history. Every completed attempt records the effective capability definition/prompt and provider/model metadata required for provenance.

Admin updates use RBAC, validation, OCC, audit actor, and safe secret references. No configuration change can bypass the owning backend policy or create a generic arbitrary model-call endpoint.

## Provider Router

The router receives a named semantic execution and an already-authorised/minimised payload. It applies:

- capability enabled/disabled state;
- configured provider/model and supported parameters;
- timeout and bounded retry/fallback rules;
- credential resolution from the approved secret boundary;
- request correlation and safe usage metadata;
- provider result/error mapping.

Fallback, where configured, occurs within one business attempt. It does not publish another requested business event or create another accepted user action. The final attempt records which provider/model produced the accepted result.

Business validation remains after provider execution. A syntactically successful provider response may still fail domain/schema/evidence validation.

## Secrets

Provider API keys and credentials are not stored as plaintext capability settings and are never returned by API/Admin UI. Runtime configuration stores only an approved secret reference or provider configuration identity where required. Logs, audit entries, errors, exports, and monitoring redact credential values.

The Admin UI may show safe presence/status metadata but cannot retrieve an existing secret. Rotation follows the environment/secret-store deployment boundary and is audited without recording the secret.

## Execution Metadata and Monitoring

Safe execution records/aggregates support per-capability operations using:

- correlation/execution identity;
- capability and operation outcome;
- provider/model identifiers;
- start/completion time and latency;
- retry/fallback count/classification;
- token usage where supplied;
- estimated cost where practical;
- safe error code/category and bounded diagnostic detail;
- effective configuration/prompt version references.

They must not expose prompt text, raw provider request/response, Candidate/resume/Cover Letter content, credentials, or arbitrary provider exceptions. Domain content stays in its owning module.

Monitoring is operationally useful but does not become billing, experimentation, prompt A/B testing, or a raw user-data browser.

## RuoYi Integration Boundary

RuoYi remains a separate Admin application with its own authenticated identity, RBAC, menu/permission model, frontend patterns, and deployment. It adds only the approved Sponsor and AI operational surfaces.

| Surface | Permitted responsibility | Prohibited responsibility |
| --- | --- | --- |
| AI runtime configuration | View/update validated capability settings with RBAC, OCC and audit | Read secrets, edit arbitrary prompts, invoke generic AI calls |
| AI monitoring | View privacy-safe aggregates and bounded failure metadata | Browse private inputs/prompts/responses or Candidate/Application content |
| Sponsor Admin | Import and maintain working data, validate, Publish, view versions/status | Unrestricted mutation of published data or generic employer-domain CRUD |

Complex business transitions execute through OfferBuddy-owned services/contracts. Sharing PostgreSQL does not grant RuoYi permission to write another module's tables directly. Any explicitly approved simple configuration persistence still obeys one owner, RBAC, OCC, validation, and audit.

## Sponsor Working and Publication Model

The CRUD-style approved Admin UI is operational management of the working/import dataset:

```text
import/source
→ working rows
→ limited create/edit/delete-or-disable correction
→ validation
→ explicit Publish
→ immutable published dataset version
→ active projection/cache/snapshot
```

Create/edit/delete permissions do not turn Sponsor Employers into a general product. Publish is a distinct authorised command. Publication validates the complete candidate dataset and atomically advances the active version. Failure leaves the last known good published version active.

Published data is the backend authority. Redis is a rebuildable active representation; Extension snapshots are versioned published copies for local lookup. No Admin or Extension path introduces another source of truth.

## Feature Disablement and In-flight Work

Capability enablement is checked at the frozen request/execution boundaries. A disabled capability rejects new execution with the contract state while existing accepted results remain readable.

For already-pending work, the worker follows the frozen policy: resolve current runtime policy, transition safely to disabled/failed where execution is no longer allowed, and avoid infinite retry. Disablement is not provider failure and must be distinguishable in API/monitoring.

Configuration changes do not mutate queued event payloads, historical outcomes, or current accepted artefacts. They affect later execution resolution according to the attempt lifecycle.

## Audit

Audit records capture actor, action, target identity/version, timestamp, outcome, and safe before/after configuration or publication metadata. They exclude secrets and private AI/domain content.

Audited operations include capability configuration/enablement, Sponsor import/working correction/validation/publication, and other frozen privileged actions. Correlation links an Admin request to resulting domain/event execution without making business events the audit store.

## Failure Behaviour

- Provider unavailable/timeout: bounded retry/fallback; safe terminal capability failure; no impact on Fast Application recording.
- Invalid provider output: domain validation failure; no accepted derived artefact/result.
- Configuration conflict: OCC response; no last-write-wins overwrite.
- Missing/invalid secret: safe disabled/misconfigured diagnostic without value disclosure.
- Monitoring persistence/read failure: business outcome remains authoritative and usable.
- Sponsor import/validation/publish/activation failure: last known good published version remains active.
- RuoYi unavailable: customer Fast Path and existing accepted artefacts remain usable.

## Verification Boundary

Implementation evidence must cover:

- business-module dependence on semantic ports and no provider SDK leakage;
- V17 persistent reference bootstrap versus runtime registration separation;
- per-capability routing, disablement, timeout, bounded fallback and structured validation;
- runtime configuration RBAC, validation, OCC, audit and secret redaction;
- safe execution metadata/aggregates and prohibited-content tests;
- RuoYi permissions and denial of universal customer-data access;
- Sponsor working corrections, full validation, explicit Publish, immutable active version and last-known-good behaviour;
- configuration change/in-flight operation semantics;
- provider/Admin/Redis failure isolation from the Sprint 2 Fast Path.

Exact database structures and API representations are controlled by the frozen database and API documents after publication. Approved Admin states are controlled by `S3-UI-18`–`S3-UI-20`.
