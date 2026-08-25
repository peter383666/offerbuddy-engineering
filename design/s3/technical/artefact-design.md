# Sprint 3 Resume and Cover Letter Artefact Technical Design

## Status and Scope

This document publishes the frozen Phase 3 design for reusable Role/Base Resumes, Job-specific Tailored Resumes, Focused Cover Letters, rendering, storage, revision, freshness, and submitted-material integration. It does not introduce a general document designer.

## Artefact Model

The model distinguishes:

| Concept | Responsibility |
| --- | --- |
| Candidate Profile | Canonical reusable candidate facts and evidence |
| Role/Base Resume | Long-lived user-controlled resume content/structure for a target role direction |
| Tailored Resume | Job-specific derived artefact created from a selected Role/Base Resume and Preparation sources |
| Focused Cover Letter | Job-specific derived/user-reviewed letter for the Preparation |
| Rendered asset | PDF/DOCX or preview output derived from an accepted artefact revision |
| Application material snapshot/reference | What the user records as actually used for an Application |

Content identity, generation attempt, accepted artefact revision, rendered file, and Application snapshot are distinct. Re-rendering the same accepted revision does not create new factual content; regenerating content does.

## Role/Base Resume

A user may maintain multiple Role/Base Resumes backed by one Candidate Profile. Each has stable identity, ownership, title/target-role metadata, accepted structured content, audit data, and the frozen concurrency version.

The Role/Base Resume selects, orders, emphasises, rewrites, condenses, or omits Candidate-supported information. It must not invent candidate facts. It is stable across Jobs and changes only through explicit user action.

Candidate changes do not rewrite a Role/Base Resume automatically. The system may calculate and present staleness guidance from recorded Candidate source version/evidence, but the user decides whether to revise or regenerate.

## Tailored Resume Generation

Tailored generation requires an authorised Preparation, selected Role/Base Resume, usable Candidate/Job context, and any Match context required/available by the frozen contract.

The immutable generation input identity includes:

- Candidate identity and `profileVersion`;
- Job identity and `contentVersion`;
- selected Role/Base Resume identity and revision/version;
- Match identity/version when consumed;
- capability/prompt/runtime definition provenance;
- explicit user notes/options allowed by the contract.

The domain builds a minimised semantic request through the Tailored Resume capability port. The AI platform chooses the configured provider/model. Returned structured content is validated for schema, section policy, evidence boundaries, length/format constraints, and unsafe output before it can become an accepted artefact revision.

Generation is always explicitly triggered. Viewing Preparation, generating Match, updating Candidate, changing AI configuration, or recording an Application must not start Tailored Resume generation implicitly.

## Focused Cover Letter Generation

Focused Cover Letter generation is a separate capability and lifecycle. It uses authorised Candidate + Job context and may use relevant accepted Match/Resume context according to the frozen input contract.

It must:

- remain evidence grounded and avoid copying the Resume mechanically;
- handle strong, partial, gap, motivation, and optional user context honestly;
- avoid unsupported company claims or fabricated motivation;
- use Australian professional conventions where specified;
- remain optional and explicitly triggered;
- preserve the last accepted letter when a later attempt fails.

Focused generation is distinct from deterministic Default/SEEK cover-letter assistance. The Fast Path must not invoke AI, and it may reuse a fresh approved Focused source only under the frozen priority rules.

## Generation Lifecycle

Each generation uses a durable domain attempt and the existing event transport:

```text
authorised explicit request
→ create/reuse generation attempt + requested event
→ worker loads frozen source tuple
→ semantic AI capability outside transaction
→ structured validation
→ persist accepted revision or terminal failure
→ render separately
```

The operation state is refresh-safe and supports the frozen accepted/running/succeeded/failed/stale semantics. Duplicate requests converge through idempotency. A deliberate regeneration creates a new attempt; it does not overwrite an earlier accepted attempt or revision.

Provider fallback is internal to one execution attempt and does not create a new business generation request. Safe provider/model/token/cost metadata is recorded without prompt, response, or private document content.

## Immutable Revisions and Limited Editing

The apparent tension between immutable generated artefacts and approved editing is resolved by the frozen revision model:

- an accepted revision is an immutable historical/provenance record;
- approved limited editing produces the frozen new/current revision outcome;
- the current artefact head may advance to a later accepted revision;
- editing does not rewrite Candidate Profile automatically;
- editing is constrained to the approved fields/sections and is not a general layout designer;
- `artefactVersion`/revision OCC protects concurrent edits independently from generation identity.

An update supplies the expected artefact version. On conflict, the backend preserves the winning revision and returns the frozen conflict response; the client preserves local input and offers the approved reload/reconcile action. Mutable working state or extra OCC columns must not be invented beyond the frozen persistence model.

## Freshness and Head Selection

Every generated revision records the source tuple it consumed. Freshness compares that tuple with current Candidate, Job, Role/Base Resume, and relevant Match versions.

Rules:

- source change may mark an artefact potentially stale but does not delete it;
- stale does not mean invalid or automatically regenerated;
- user edits have their own revision identity and must not be silently replaced by a late worker;
- current-head advancement obeys the frozen source/attempt ordering, not completion timestamp alone;
- a failed or older-source completion cannot displace a newer accepted/user-edited head;
- historical Application material remains tied to what was used then.

## Structured Content and Rendering

Domain content is stored in the frozen structured representation. Rendering is an infrastructure capability behind a port, not embedded in controllers or AI adapters.

The rendering boundary accepts an authorised accepted artefact revision and produces a versioned asset record. PDF is the required core output; DOCX follows the frozen priority. Preview may use an authorised rendered representation but cannot expose a public bucket/object path.

Rendering/storage failures are local. They may transition a render operation to failed without corrupting content or deleting the previous successful asset. Retry reuses the content revision and creates/reuses the appropriate render identity rather than regenerating AI content.

## Object Storage and Access

Source resumes and rendered artefacts use server-generated object identities through the approved storage port. Metadata in PostgreSQL remains authoritative for ownership, content/revision association, media type, checksum/size where required, lifecycle, and audit.

Download/preview access checks authenticated ownership through the complete resource chain and then returns/streams approved expiring access. Object keys, filenames, Preparation IDs, or guessed URLs are not authorisation.

Private content, object URLs, prompts, and rendered bytes are excluded from events, ordinary logs, metrics, and Admin monitoring. Cleanup must distinguish temporary failed/unaccepted assets from retained accepted/Application materials.

## Application Material Integration

Application records what the user actually used. Preparation completion alone does not infer submission.

The frozen Application command associates the accepted Resume revision/rendered asset and optional Cover Letter revision that the user confirms. Historical material association remains stable even if Candidate, Job, Role/Base Resume, or current artefact heads later change.

Application owns the submitted-material association/snapshot semantics. Resume/Cover Letter modules expose authorised immutable references/projections and do not update Application lifecycle directly.

## Failure and Degraded Behaviour

- AI disabled/unavailable: existing accepted artefacts remain reviewable/downloadable; new generation returns the frozen disabled/failed state.
- Match unavailable: generation follows only the frozen fallback eligibility and never fabricates Match context.
- rendering unavailable: accepted structured content remains safe and retryable.
- object storage failure: transaction does not publish a successful asset reference.
- stale source: existing artefact remains visible with approved guidance; regeneration is explicit.
- concurrent edit/regeneration: OCC and current-head rules prevent lost user work.
- malformed provider output: reject before accepting a revision; do not persist it as user-visible content.

None of these failures may block Sprint 2 Application recording.

## Verification Boundary

Implementation evidence must cover:

- user ownership across Role/Base Resume, Preparation, revisions and assets;
- evidence-grounded generation and structured-output validation;
- explicit trigger, idempotency, retry/restart and deliberate regeneration;
- immutable historical revisions plus approved limited editing/OCC;
- Candidate/Job/Base Resume/Match changes and source-version freshness;
- late/failed completion not replacing a newer or user-edited head;
- render retry without another AI generation;
- authorised preview/download and guessed/cross-user object denial;
- Application material association remaining historical;
- Focused versus deterministic Fast assistance isolation;
- absence of private content in events, logs, monitoring and URLs.

Exact HTTP contracts are defined by the frozen `api-contract.md` after publication. Approved surface behaviour is defined by `S3-UI-02`–`S3-UI-05` and the Application Detail specification.
