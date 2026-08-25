# Sprint 3 Security, Privacy, and AI Data Boundary Design

## Status and Scope

This document publishes the frozen Phase 3 security/privacy design for Candidate data, Preparation, generated artefacts, AI processing, Extension, RuoYi Admin, file/object storage, events and diagnostics. Existing authentication remains; Sprint 3 adds ownership chains and least-privilege controls for new resources.

## Trust Boundaries

| Actor/system | Trust rule |
| --- | --- |
| Customer Web client | Authenticated client, but identifiers, versions and content remain untrusted input |
| Browser Extension | Lightweight untrusted client with finite-lived revocable credentials; page context is hostile/untrusted |
| OfferBuddy backend | Authoritative customer identity, ownership, domain validation and policy enforcement |
| Worker | Trusted application process but must re-check resource state/policy and minimise loaded data |
| AI provider | External processor receiving only approved minimal context; never trusted as truth |
| Object storage/rendering | Infrastructure behind authorised application ports; object keys are not access control |
| RuoYi Admin | Separately authenticated operational client with explicit RBAC, not universal domain privilege |
| Redis/Extension snapshot | Rebuildable projection/cache, not authority |

## Ownership Chains

Server-derived authenticated identity is the root for all customer operations. Typical chains are:

```text
user → Candidate Profile → Candidate child/import draft
user → Preparation → Match/generation/artefact revision → rendered asset
user → Application → submitted-material association
```

Job is shared canonical data, but a shared Job ID never grants access to another user's Candidate, Preparation, Match, artefact, or Application. Every query/command checks the complete owning chain in the backend. Client-supplied `userId`, object key, Preparation ID, or related-resource ID cannot override it.

Internal module calls use authorised, minimised contracts; “internal” does not mean unscoped access.

## RuoYi RBAC

RuoYi permissions are capability/action specific: Sponsor read/import/correct/validate/publish and AI configuration/monitoring permissions remain distinct. Admin identity and actor metadata come from the RuoYi security context.

Admin cannot:

- browse arbitrary Candidate, Resume, Cover Letter, Match, Preparation or Application content;
- retrieve AI prompts, raw responses or secrets;
- bypass Sponsor publication by mutating active published rows;
- call generic provider operations;
- obtain customer ownership through shared-database access.

Privileged operations are validated, audited and protected by OCC where frozen.

## Data Classification and Minimisation

Candidate/Profile facts, imported resume text/files, generated Resume/Cover Letter content, user notes, Application materials, prompts and provider responses are private content. Credentials and provider keys are secrets.

Private content is stored/transmitted only where required by the owning capability and retention policy. AI requests use capability-specific snapshots rather than complete entities. Events carry identities, not content. Monitoring and audit carry safe metadata, not business documents.

HTTP errors, URLs, correlation headers, logs, metrics, traces and Admin screens must not disclose private content or secrets. Safe resource IDs may be logged only where approved and access remains enforced.

## File Upload and Object Access

Resume uploads require authenticated ownership, byte-size limits, allow-listed format/media verification, safe filename handling, parser isolation, malware/security checks supported by the repository, and controlled retention/cleanup.

Object identities are server generated. Buckets/objects are private. Preview/download is authorised through application metadata and returns a short-lived signed URL or controlled stream under the frozen design. Public object URLs and guessable-path access are prohibited.

Failed, abandoned, expired and superseded temporary objects are cleaned without deleting accepted historical/Application materials. Rendered output is treated as private even when generated from already accepted content.

## AI Safety Boundary

Job descriptions, resumes, Candidate/user notes and provider output are untrusted data. Source text cannot instruct the system to reveal secrets, change provider/model, call tools, ignore policy, access another user, or alter system behaviour.

Controls include:

- capability-scoped system/developer instructions owned by the business module;
- explicit separation of instruction and source data;
- minimum necessary source fields and evidence IDs;
- structured output/schema validation;
- domain truthfulness and evidence validation after provider execution;
- output length/content constraints and safe error mapping;
- no provider-controlled persistence/entity updates;
- mandatory user review for generated artefacts.

No control makes AI output canonical Candidate truth automatically.

## Secrets and Configuration

Secrets remain in the approved deployment secret boundary. Runtime database configuration references safe provider/credential identities but does not store plaintext keys. APIs/Admin UI never return existing key values. Rotation, missing-secret state and access are observable without disclosing values.

Repository files, migrations, fixtures, test output and documentation must not contain real credentials or private user documents.

## Extension Security

Extension credentials remain isolated in the service-worker/auth boundary and are never injected into page execution. Host permissions and externally accessible resources are minimal. Messages validate sender/context and payload shape.

Extension storage is limited to approved authentication state and versioned Sponsor/config data; it excludes Candidate/resume/Cover Letter content and provider data. Deep links carry stable identifiers only through approved origins/routes.

## Audit and Correlation

Audit captures privileged actor/action/target/outcome and safe before/after metadata. Correlation connects HTTP, event, worker, retry and AI execution for diagnosis. Neither audit nor correlation is authorisation, business idempotency, or a place for source content.

Security-relevant denial/failure is diagnosable without revealing whether an inaccessible resource belongs to another user beyond the frozen error contract.

## Retention and Deletion

Retention follows owning-domain and historical Application requirements. Temporary import data and failed/unaccepted assets expire/clean up under frozen policy. Accepted Application materials and audit records are retained according to their explicit lifecycle.

Candidate deletion/change must not rewrite another aggregate's immutable historical record. Deletion operations remain user scoped, auditable where required, and do not cascade through raw cross-module repository access.

## Verification Boundary

Required evidence includes:

- cross-user tests for every Candidate/Preparation/artefact/Application route and object access;
- Admin permission matrix and denied operations;
- upload format/size/name/parser failure and object cleanup/access tests;
- prompt-injection/adversarial source fixtures and structured/evidence validation;
- absence of private content/secrets in events, logs, errors, URLs, audit and monitoring;
- Extension credential/message/storage/permission checks;
- secret create/rotate/reference responses without read-back;
- disabled/provider/storage/Redis/Admin failure without weakened authorisation or Fast Path regression.

Exact error/status behaviour is defined by the frozen API contract. Operational projections are defined by observability design.
