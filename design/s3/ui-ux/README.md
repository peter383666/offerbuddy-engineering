# OfferBuddy S3 UI Specifications

Status: **APPROVED / FROZEN**

The governed page documents are published under [`pages/`](pages/). Approved visual references are stored under [`assets/`](assets/) and must be read with the applicable page specification.

## Governed Page Specifications

### Focused Application

- [S3-UI-01 — Match Analysis](<pages/S3-UI-01 — Match Analysis.md>)
- [S3-UI-02 — Create Tailored Resume](<pages/S3-UI-02 — Create Tailored Resume.md>)
- [S3-UI-03 — Base Resume Preview](<pages/S3-UI-03 — Base Resume Preview.md>)
- [S3-UI-04 — Tailored Resume Review](<pages/S3-UI-04 — Tailored Resume Review.md>)
- [S3-UI-05 — Focused Cover Letter Review](<pages/S3-UI-05 — Focused Cover Letter Review.md>)

### Candidate Profile

- [S3-UI-06 — Candidate Profile Overview](<pages/S3-UI-06 — Candidate Profile Overview.md>)
- [S3-UI-07 — Resume Import Review](<pages/S3-UI-07 — Resume Import Review.md>)
- [S3-UI-08 — Personal Details](pages/S3-UI-08.md)
- [S3-UI-09 — Professional Summary](<pages/S3-UI-09 — Professional Summary.md>)
- [S3-UI-10 — Skills](<pages/S3-UI-10 — Skills.md>)
- [S3-UI-11 — Experience](<pages/S3-UI-11 — Experience.md>)
- [S3-UI-12 — Education](<pages/S3-UI-12 — Education.md>)
- [S3-UI-13 — Certifications](<pages/S3-UI-13 — Certifications.md>)
- [S3-UI-14 — Languages and Eligibility](<pages/S3-UI-14 — Languages & Eligibility.md>)

### Extension and S2 Integration

- [S3-UI-15 — Extension Floating Assistant](<pages/S3-UI-15 — Extension Floating Assistant.md>)
- [S3-UI-16 — SEEK Cover Letter Assistance](<pages/S3-UI-16 — SEEK Cover Letter Assistance.md>)
- [S3-UI-17 — Application Detail Integration](<pages/S3-UI-17 — Application Detail Integration.md>)

### RuoYi Admin

- [S3-UI-18 — Sponsor Employers Admin](<pages/S3-UI-18 — Sponsor Employers Admin.md>)
- [S3-UI-19 — AI Governance and Runtime Configuration](<pages/S3-UI-19 — AI Governance  Runtime Configuration.md>)
- [S3-UI-20 — AI Monitoring and Operational View](<pages/S3-UI-20 — AI Monitoring  Operational View.md>)

See the [Specification Index](INDEX.md) for dependency order and Figma-frame mapping, and the [Final Coverage Review](coverage-review.md) for freeze evidence.

## Purpose

This directory defines the implementation contract between the frozen S3
product/technical design, Figma UI design, and frontend implementation.

These specifications are written primarily for implementation agents and
engineering review.

They describe behaviour, state, data dependencies, navigation, validation,
reuse boundaries, and implementation constraints.

They do not replace the Phase 3 Technical Design or Figma.

## Source of Truth

Implementation must use the following sources together:

1. Phase 3 Technical Design
   - architecture
   - domain ownership
   - persistence
   - API contracts
   - concurrency
   - async processing
   - security
   - AI governance

2. S3 Page Specifications
   - page behaviour
   - UI state
   - interaction
   - navigation
   - validation
   - frontend/backend mapping

3. Figma
   - visual structure
   - layout
   - hierarchy
   - component presentation

4. Existing S2 implementation
   - reuse baseline
   - existing application workflows
   - existing components and conventions

## Conflict Rule

Do not invent behaviour when the sources appear inconsistent.

Priority:

Technical/domain correctness
→ frozen API contract
→ Page Specification
→ Figma visual representation
→ implementation convenience

Stop and resolve a contradiction before changing a frozen business rule.

## S2 Reuse Rule

S3 is additive.

Existing S2 functionality must be reused wherever practical.

Do not rewrite an existing S2 page, component, API workflow, or domain
capability merely because an S3 specification references it.

S3 modifications must be implemented as explicit deltas.

## Fast Apply Rule

The existing S2 Fast Apply path remains independent of Candidate Profile
and Focused Preparation.

A user must still be able to apply without:

- Match Analysis
- Tailored Resume
- Focused Cover Letter
- Candidate Profile completion

## Focused Apply Rule

Focused Apply is optional.

The normal journey is:

Recruitment Website
→ Extension
→ Prepare with OfferBuddy
→ OfferBuddy Web
→ Match
→ Resume
→ Cover Letter
→ Return to Recruitment Website
→ Submit
→ Application

## Candidate Fact Rule

Candidate Profile is the authoritative source of candidate facts.

Generated content must never silently establish new Candidate facts.

Resume Import may propose facts, but user acceptance is required.

Match, Tailored Resume and Focused Cover Letter are derived artifacts.

## Extension Boundary

The Browser Extension remains lightweight.

It may:

- detect job context
- surface relevant hints
- open Focused Preparation
- assist SEEK cover-letter filling
- reflect already-applied state

It must not become a second OfferBuddy Web application.

In particular, Match Analysis is displayed in the Web application, not
inside the Extension.

## RuoYi Boundary

OfferBuddy uses the existing RuoYi Admin framework.

Do not implement a new Admin shell.

Reuse RuoYi:

- navigation
- authentication / authorization
- forms
- tables
- pagination
- dialogs
- standard CRUD interaction

S3 specifications describe only OfferBuddy-specific Admin content.

## Figma Annotation Rule

Figma nodes labelled `ANNOTATION — ...` are design documentation.

They are NOT application UI and must not be rendered by the implementation.

## State Rule

A Figma frame represents a visual state, not necessarily a separate route
or React page.

The Page Specification determines the implementation boundary.

## API Rule

Do not create new endpoints from UI requirements when an S3 API contract
already exists.

Use the frozen Phase 3 API contract.

If the required behaviour cannot be implemented using the frozen contract,
treat this as a design contradiction.

## Concurrency Rule

Do not silently overwrite stale data.

Respect the relevant optimistic concurrency model:

- Candidate Profile → profileVersion
- Job semantic content → contentVersion
- Application → version

Handle stale writes explicitly.

## AI / Async Rule

AI operations may be asynchronous.

The frontend must represent the business resource state defined by the API
contract rather than inventing generic client-side task semantics.

Do not expose:

- raw prompts
- raw provider responses
- secrets
- internal provider credentials

## Agent Rule

Implementation agents must not:

- invent product features
- redesign frozen UI
- broaden Extension responsibilities
- duplicate existing S2 functionality
- bypass ownership/security rules
- silently merge concurrency conflicts
- convert AI-derived content into Candidate facts
- redesign RuoYi





# OfferBuddy S3 UI Specifications

## Purpose

This directory defines the implementation contract between the frozen S3
technical design, Figma UI design, existing S2 implementation, and the
frontend/browser-extension/admin implementation work.

These documents are written primarily for implementation agents and
engineering review.

They describe:

- UI behaviour
- state transitions
- data dependencies
- navigation
- validation
- concurrency
- staleness
- reuse requirements
- security/privacy boundaries
- implementation constraints

They do not replace the Phase 3 Technical Design or Figma.

---

## Source of Truth

Implementation must use the following sources together:

1. Phase 3 Technical Design
   - architecture
   - domain ownership
   - persistence
   - API contracts
   - concurrency
   - async processing
   - security/privacy
   - AI governance
   - observability

2. S3 Page Specifications
   - page behaviour
   - UI state
   - interaction
   - navigation
   - validation
   - frontend/backend mapping

3. Figma
   - layout
   - hierarchy
   - visual presentation
   - final approved UI states

4. Existing S2 implementation
   - reuse baseline
   - application workflow
   - existing components
   - extension integration
   - coding conventions

---

## Conflict Rule

Do not invent behaviour when sources appear inconsistent.

Use this priority:

Technical/domain correctness
→ frozen API contract
→ Page Specification
→ frozen Figma
→ implementation convenience

If a contradiction remains, stop and resolve it before changing a frozen
business rule.

---

## S2 Reuse Rule

S3 is additive.

Existing S2 functionality must be reused wherever practical.

Do not rewrite an existing S2 page, component, API workflow, Extension flow,
or domain capability merely because S3 adds new functionality.

S3 changes should be explicit deltas.

---

## Fast Apply Rule

The existing S2 Fast Apply path remains independent of Candidate Profile
and Focused Preparation.

A user must still be able to apply without:

- Match Analysis
- Tailored Resume
- Focused Cover Letter
- Candidate Profile completion

S3 must not make normal job applications slower.

---

## Focused Apply Rule

Focused Apply is optional.

Primary journey:

Recruitment Website
→ Extension
→ Prepare with OfferBuddy
→ OfferBuddy Web
→ Match
→ Tailored Resume
→ Focused Cover Letter
→ Return to Recruitment Website
→ Submit externally
→ Application lifecycle

OfferBuddy prepares the application.

The recruitment website remains the actual submission surface.

---

## Candidate Fact Rule

Candidate Profile is the authoritative source of Candidate facts.

Generated content must never silently establish new Candidate facts.

Resume Import may propose facts, but explicit user review/acceptance is
required before those facts enter Candidate Profile.

Derived artifacts include:

- Match
- Tailored Resume
- Focused Cover Letter

These do not become Candidate facts automatically.

---

## Candidate Profile Version Rule

Candidate Profile uses aggregate-level:

`profileVersion`

Meaningful Candidate fact changes increment the version.

Do not introduce child-section versions such as:

- skillVersion
- experienceVersion
- educationVersion

Stale writes fail explicitly.

Do not silently merge them.

---

## Job Version Rule

Canonical Job semantic content uses:

`contentVersion`

Only meaningful semantic Job changes should affect this version.

Derived S3 artifacts may become stale when the source Job contentVersion
changes.

---

## Application Version Rule

Application remains independently user-owned.

Application optimistic concurrency uses:

`version`

Do not confuse:

- Application version
- Candidate profileVersion
- Job contentVersion

They have different meanings and ownership.

---

## Async Rule

AI/external work may be asynchronous.

Use the frozen business-resource model:

command
→ HTTP 202 when appropriate
→ business resource PROCESSING
→ read/poll business resource
→ READY / FAILED

Do not create a generic frontend AI-task abstraction unless the frozen API
contract explicitly defines one.

---

## Extension Boundary

The Browser Extension is a lightweight, untrusted assistant.

It may:

- detect Job context
- surface useful signals
- open Focused Preparation
- assist SEEK Cover Letter filling
- reflect already-applied state

It must not become a second OfferBuddy Web application.

Do not place inside the Extension:

- Match result
- Tailored Resume review/editor
- Focused Cover Letter editor
- Candidate Profile editor

---

## Sponsor Dataset Rule

Sponsor Employer data is maintained in RuoYi Admin.

Flow:

Admin working dataset
→ explicit Publish
→ versioned published snapshot
→ Extension sync
→ local cache
→ local employer lookup

A negative local lookup must not trigger a backend sponsor lookup for every
Job page.

A sponsor employer match is a signal, not a guarantee that the current role
offers sponsorship.

---

## Cover Letter Rule

There are two separate concepts:

### Default Cover Letter
Used for Fast Apply / SEEK Quick Apply.

### Focused Cover Letter
Job-specific derived artifact created through Focused Preparation.

For the same Job:

fresh Focused Cover Letter
> Default Cover Letter

Do not merge the two resources.

---

## RuoYi Rule

OfferBuddy uses the existing RuoYi Admin framework.

Do not implement a new Admin shell.

Reuse RuoYi:

- navigation
- authentication/authorization
- forms
- tables
- pagination
- dialogs
- uploads
- permissions
- status components

S3 specifications describe only OfferBuddy-specific Admin content.

---

## AI Governance Rule

Business modules depend on semantic AI capability ports, not provider SDKs.

Runtime Admin may control:

- capability enablement
- provider routing
- model selection
- approved runtime parameters

It must not expose:

- plaintext secrets
- raw prompts
- raw provider responses

Admin is operational, not a universal user-data bypass.

---

## Monitoring Rule

AI Monitoring is metadata-first.

Allowed examples:

- capability
- provider
- model
- request count
- success/failure
- latency
- tokens
- estimated cost
- safe failure category
- requestId

Do not expose:

- Candidate Profile content
- Resume content
- Cover Letter content
- raw Job description
- raw prompts
- raw responses
- secrets

---

## Figma Annotation Rule

Figma nodes labelled:

`ANNOTATION — ...`

are design documentation.

They are not application UI and must never be rendered by implementation.

---

## State Rule

A Figma frame represents a visual state, not necessarily a separate React
page or route.

The Page Specification defines the engineering implementation boundary.

---

## Agent Rule

Implementation agents must not:

- invent product features
- redesign frozen UI
- broaden Extension responsibilities
- rewrite S2 without necessity
- create new API contracts when frozen ones exist
- silently merge concurrency conflicts
- convert AI-derived content into Candidate facts
- infer Candidate Eligibility from weak evidence
- redesign RuoYi
- expose secrets/prompts/responses
- build Auto Apply
- build Saved Job workflow
- create additional Candidate personas
