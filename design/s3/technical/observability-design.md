# Sprint 3 Observability, Audit, and Operational Diagnostics Design

## Status and Scope

This document publishes frozen §3.16. Observability explains runtime behaviour without becoming a second domain store or exposing Candidate/job/application material, prompts, responses or secrets.

## Diagnostic Model

Sprint 3 separates:

| Concern | Authority |
| --- | --- |
| Domain/business state | Owning aggregate, operation, generation, artefact or published dataset record |
| Event transport state | `business_events` claim/retry/outcome fields |
| Privileged actor history | Audit records |
| Runtime operational view | Metrics, safe execution metadata, logs and Admin projections |
| Request causality | Correlation ID propagated across boundaries |

Metrics/logs do not determine business success. A durable domain record is the user-visible authority.

## Correlation

Every inbound request accepts or creates the frozen valid `X-Request-ID`/correlation value and returns it according to the API contract. Application services propagate it to created domain operation/execution metadata and new business events.

V10 adds `business_events.correlation_id` because the actual V7 schema lacks it. Workers restore correlation when claiming/retrying and pass it through semantic AI execution and rendering/storage diagnostics. Retries keep causal correlation while claim/attempt identifiers remain distinct.

Correlation is safe metadata only. It is not user identity, authorisation, idempotency, event identity or source version.

## Structured Logging

Logs use stable event/action names and safe fields such as:

- correlation, event, operation/generation and safe resource identifiers;
- module/capability and state transition;
- duration, retry number, error category and outcome;
- provider/model identifiers and token counts where permitted;
- Sponsor published/snapshot version and projection outcome.

Logs must not contain Candidate/resume/Cover Letter text, prompts, provider responses, uploaded file contents, credentials, signed URLs, tokens/cookies, or raw exceptions with private payloads. Error mapping records a safe category/code and retains detailed sensitive diagnostics only in an explicitly approved protected boundary, if any.

## Metrics

Metrics use bounded dimensions. Required families include:

- HTTP request duration/count by route template, method and safe outcome;
- event pending/claimed age, throughput, retry, terminal failure and recovery;
- async capability operation success/failure/disabled/stale and duration;
- AI execution by capability/provider/model, token usage and estimated cost where available;
- render/storage success/failure and latency;
- Sponsor import/validation/publish/projection outcomes and active version age;
- Extension snapshot version/refresh diagnostics through approved backend signals;
- required/optional dependency health.

Never use user ID, Candidate content, Job title/company, raw path, prompt, error message or document identifier as an unbounded metric label.

## AI Operational Records

Safe AI execution metadata records correlation/execution identity, capability, effective provider/model, timing, outcome, retry/fallback, token/cost data, safe failure code and configuration/prompt version references.

Admin monitoring may show aggregates and bounded failure detail from this metadata. It cannot browse raw business inputs/outputs or secrets. Domain result content remains in Candidate/Preparation/artefact ownership and requires customer authorisation, not Admin monitoring permission.

## Audit

Audit is required for privileged configuration and publication operations. It records authenticated Admin actor, action, target/version, timestamp, safe before/after metadata, correlation and outcome.

Audit events include AI capability enablement/configuration changes and Sponsor import/correction/validation/Publish as frozen. Audit does not duplicate ordinary customer edits or transport events indiscriminately and never stores plaintext secrets/private AI content.

## Health and Readiness

Health distinguishes required dependencies from degradable capabilities:

- PostgreSQL/backend/authentication required for core readiness;
- AI providers, Redis acceleration/projection, rendering/object storage for affected actions, and RuoYi are reported without making the Sprint 2 Fast Path unavailable unnecessarily;
- misconfigured/disabled AI capabilities are visible per capability;
- worker lag/claim recovery is observable separately from HTTP health;
- last known good Sponsor publication remains visible during refresh failure.

## Operational Failure Diagnosis

An operator must be able to answer without private content:

- which user-visible operation failed and its current durable state;
- which event/worker attempt handled it and whether retry remains;
- which capability/provider/model and safe configuration version was effective;
- whether source versions made the outcome stale/superseded;
- whether rendering/storage failed after accepted content;
- which Sponsor dataset/snapshot version is active;
- which authorised Admin action changed configuration/publication state.

## Alerts and Retention

Alerting targets actionable sustained conditions: event lag/terminal failures, capability failure/latency, provider misconfiguration, render/storage failure, Sponsor publication/refresh failure, and required dependency unavailability. Threshold values remain an operational implementation detail.

Retention is purpose limited. High-cardinality request logs, safe AI execution metadata, audit and business records have distinct policies. Deleting/retaining telemetry never changes domain state; audit retention follows its governance requirement.

## Verification Boundary

Evidence must prove correlation from HTTP through event/worker retry/AI, historical events without correlation remain compatible, bounded metric cardinality, prohibited-data redaction, safe Admin monitoring, auditable RBAC actions, required/optional readiness behaviour, and diagnosis of injected provider/render/storage/Sponsor failures without corrupting accepted outcomes.

Exact headers/error envelopes and Admin response representations are governed by the frozen API contract and approved `S3-UI-20` specification.
