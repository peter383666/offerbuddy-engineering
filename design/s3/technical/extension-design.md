# Sprint 3 Browser Extension Integration Technical Design

## Status and Scope

This document publishes the frozen Sprint 3 Extension delta. It reuses the Sprint 2 Manifest V3 architecture, authentication/pairing, background service-worker boundary, content-script boundary, floating companion, and Site Adapter contract. It does not redesign the Extension or move Web Preparation features into it.

## Reuse Boundary

The existing common `JobPageContext`/capture representation remains the boundary between a site-specific adapter and Extension orchestration. SEEK and Indeed behaviour must remain compatible. LinkedIn is added as an isolated best-effort adapter.

```text
page DOM
→ site adapter
→ normalised page context
→ companion state resolver
→ background/authenticated API client
→ OfferBuddy backend or contextual Web route
```

No adapter calls backend APIs, owns authentication, renders business UI, or imports another site's selectors.

## LinkedIn Adapter

The LinkedIn adapter detects the frozen supported Job-detail contexts and extracts only deterministic page facts permitted by the common representation. It must:

- keep LinkedIn selectors/heuristics isolated;
- distinguish stable required identity from optional facts;
- support relevant SPA navigation and panel/full-page transitions;
- coalesce noisy DOM mutations;
- clear stale Job context when navigation no longer represents the detected Job;
- fail as unsupported/unreliable rather than fabricating an identifier or mixing two page contexts;
- use versioned fixtures and current-page manual verification.

LinkedIn Easy Apply automation, form completion, and submission are out of scope. Best-effort platform support must not delay the core Preparation workflow or regress SEEK/Indeed.

## Floating Assistant State

The companion is a small contextual surface. State resolution is deterministic and follows the priority frozen by `S3-UI-15`, including unsupported/extraction failure, authentication, existing Application, Preparation availability/readiness, Sponsor/eligibility signals, and actionable hand-off.

Only one active context/listener set exists per tab/frame lifecycle. Repeated mutations/navigation must not duplicate buttons, observers, requests, or actions. Async responses are associated with the page-context identity that initiated them; a late response for an old Job cannot update the current companion.

The Extension does not render Candidate editors, Match details, Resume/Cover Letter editors, AI retry dashboards, or Admin configuration.

## Preparation Hand-off

**Prepare with OfferBuddy** uses the authenticated backend contract to resolve/capture the Job and create/reuse the user's Preparation where required by the frozen API. It then opens the approved Web route with stable identifiers, not private content or untrusted destination URLs.

Repeating the action converges on the same logical Preparation. The Web application remains responsible for Candidate readiness, Match, artefact generation, review, and errors. Failure to prepare does not record an Application or alter the page's apply action.

## Sponsor Snapshot and Local Lookup

Sponsor responsibilities are layered:

```text
published canonical backend data
→ backend/Redis active representation
→ authenticated/versioned Extension snapshot refresh
→ local page-time employer lookup
```

The Extension stores only the approved normalised snapshot/version/refresh metadata. Page-time lookup uses the local snapshot where available and does not require a network request for every Job. Backend calls are for acquisition, refresh, invalidation/version checks, or frozen fallback.

The snapshot is not editable and is never a second source of truth. A refresh is atomic: validate the complete new snapshot, then switch active version. Failed/partial refresh retains the last known good snapshot. A match indicates only a published sponsorship-related employer signal, never a guarantee for the Job.

Employer-name normalisation follows the frozen backend/snapshot contract. The Extension must not invent fuzzy/business matching semantics independently.

## SEEK Cover Letter Assistance

`S3-UI-16` extends the SEEK adapter/companion with the frozen deterministic assistance flow. It remains on the Fast Path and must not invoke AI automatically.

Source priority follows the frozen contract: an eligible fresh user-approved Focused Cover Letter may be offered where allowed; otherwise the approved deterministic Candidate/default content is used. Missing Candidate data, stale Focused content, unsupported field shape, or unsafe page state produces the specified review/fallback state rather than silent form mutation.

The user remains the final reviewer and submitter. Sprint 3 does not fill an entire application, answer screening questions, or submit.

## Authentication and Security

The Extension reuses finite-lived revocable credentials and background-mediated authenticated calls. Page execution never receives credentials. All backend ownership is derived from the authenticated principal; Job/Preparation/Application IDs from the page or URL are untrusted.

Manifest host permissions and content-script matches remain minimal. Snapshot and companion storage must not contain Candidate Profile, resume text, Cover Letter content, prompts, responses, provider credentials, or backend session secrets. Logs/errors avoid page private content and authentication material.

## Failure and Offline Behaviour

- Unsupported or unreliable extraction: show the approved safe state and send no fabricated capture.
- Authentication expired/revoked: preserve page context and direct the user through the existing reconnection flow.
- Backend unavailable: local Sponsor lookup may use a valid cached snapshot; Preparation/recording actions fail visibly and retry only on user action/frozen policy.
- Snapshot refresh failure: retain last known good version and expose freshness only as approved.
- AI disabled/failed: Extension Fast Path and deterministic assistance remain usable.
- SPA navigation during request: ignore the stale response.

## Verification Boundary

Implementation evidence must cover:

- LinkedIn isolated fixtures, required/optional facts, SPA/panel transitions and stale clearing;
- unchanged SEEK/Indeed capture and Application regression;
- one observer/listener/action per active context and late-response rejection;
- authenticated Preparation create/reuse and safe contextual deep link;
- snapshot version acquisition, atomic refresh, invalidation, local positive/negative lookup and last-known-good fallback;
- no per-page mandatory Sponsor network request;
- deterministic SEEK assistance source priority and no automatic submission;
- credential isolation, minimal permissions, safe storage and cross-user/backend authorisation;
- approved `S3-UI-15`/`16` states and accessibility behaviour.

Exact request/response shapes and errors remain governed by the frozen `api-contract.md` after publication.
