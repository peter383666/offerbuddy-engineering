# OfferBuddy S2 — Final UI/UX Page Specification v2.0

**Status:** Final / Approved
**Phase:** S2 Phase 4 — UI/UX Design
**Purpose:** Frontend & Browser Extension implementation handoff

## 1. Source of Truth

S2 implementation must follow this hierarchy:

**Figma → Visual Source of Truth**
**This Specification → Behaviour / Interaction Source of Truth**
**API & Technical Design → Data / Technical Source of Truth**

If these sources conflict, the implementation Agent must report the conflict rather than silently redesigning or inventing behaviour.

Final Figma:

[OfferBuddy S2 Product Design](https://www.figma.com/design/9otw4DIbkS1J5nG0stMkuW/application?node-id=132-2&utm_source=chatgpt.com)

---

# 2. Global Application Shell

## 2.1 Branding

All S2 product surfaces use the same official **OfferBuddy O + Check logo**.

This includes:

* Login
* Home
* Applications
* Application Detail
* New Application
* Analytics
* Browser Extension

Do not recreate different logo variants for individual surfaces.

The canonical frontend asset should preferably be SVG.

---

## 2.2 Desktop Navigation

Primary navigation:

**Home | Applications | Analytics**

The active navigation item uses the approved **green underline**.

Examples:

| Screen             | Active Navigation |
| ------------------ | ----------------- |
| Home               | Home              |
| Applications       | Applications      |
| Application Detail | Applications      |
| New Application    | Applications      |
| Analytics          | Analytics         |

Application Detail and New Application are part of the Applications workflow and therefore do not become separate top-level navigation items.

---

# 3. Browser Extension

The Browser Extension is the primary S2 job capture workflow.

Supported initial platforms:

* SEEK
* Indeed

The Extension captures reliable job-page facts.

Backend owns business meaning and persistence.

AI Job Intelligence is downstream enrichment and must not block saving an Application.

---

## 3.1 Extension — Default

**Figma node:** `140:2`

### Purpose

Provide the smallest useful interaction for the normal case.

Most jobs should reach this state.

### Display

Show:

* OfferBuddy logo
* OfferBuddy branding
* detected job title
* company
* location where available
* source/platform where appropriate
* `Save to OfferBuddy`

Do **not** display an eligibility dashboard by default.

Do not show:

* Working rights: OK
* Citizenship: OK
* Security clearance: OK
* Sponsorship: OK

when there is nothing actionable.

### Size

Reference:

**400 × 300**

The implementation should size naturally to content rather than forcing a fixed 760px popup.

---

# 4. Extension — Review Required

**Figma node:** `133:3`

### Trigger

Only when OfferBuddy detects relevant **Citizenship / PR requirement language** requiring the user's attention.

### Display

Keep the normal job information visible.

Add a lightweight warning indicating:

**Potential eligibility requirement detected**

Provide:

**Review**

Do not immediately expose the full analysis.

### Behaviour

`Review`

→ opens **Review Detail**

The user decides whether to inspect the finding.

### Reference size

**400 × 390**

---

# 5. Extension — Review Detail

**Figma node:** `142:34`

### S2 Scope

Review Detail is specifically focused on relevant:

**Citizenship / Permanent Residency requirements**

It is not a general eligibility dashboard.

### Display

Show:

* Citizenship / PR requirement
* concise explanation
* detected language / evidence from the job description
* Back
* Save to OfferBuddy

Example conceptual structure:

**Citizenship / PR required**

This job appears to require Australian citizenship or permanent residency.

**Detected language**

> Relevant extracted JD wording.

Do not populate the screen with unrelated "passed" checks.

### Reference size

**400 × 600**

---

# 6. Extension — Saving

**Figma node:** `139:2`

### Trigger

User selects:

**Save to OfferBuddy**

### Meaning

This is the actual Application creation operation.

`Save to OfferBuddy` must not mean merely bookmarking or saving a job for later.

The user has indicated they are recording this job as an Application in OfferBuddy.

### Behaviour

During request:

* prevent accidental repeated submission
* show clear progress
* retain job context
* do not require AI Job Intelligence to finish

### Reference size

**400 × 330**

---

# 7. Extension — Saved

**Figma node:** `139:44`

### Trigger

Core Application persistence succeeds.

### Display

Clearly communicate that the Application was saved successfully.

AI enrichment may continue asynchronously.

The Extension must not wait for Job Intelligence before reporting successful Application creation.

### Reference size

**400 × 350**

---

# 8. Extension — Duplicate

**Figma node:** `139:86`

### Trigger

Backend identifies an existing Application for the same user/job according to the confirmed duplicate rules.

### Display

Explain that this job already exists in the user's Applications.

Provide access to the existing Application where supported.

Do not create another Application silently.

### Reference size

**400 × 400**

---

# 9. Extension — Authentication Required

**Figma node:** `139:128`

### Trigger

The Extension cannot perform the authenticated save operation.

### Display

Explain that the user needs to sign in to OfferBuddy.

Provide the confirmed authentication action.

Do not lose the currently detected job context unnecessarily.

### Reference size

**400 × 430**

---

# 10. Extension — Failed

**Figma node:** `139:170`

### Trigger

Save fails for a recoverable or unexpected reason.

### Display

Provide:

* clear failure message
* retry/recovery action where appropriate
* retained job context

Do not report a failed save as successful.

AI failure alone must not cause this state if the core Application was successfully persisted.

### Reference size

**400 × 430**

---

# 11. Home

**Figma node:** `151:2`

## Purpose

Provide a lightweight overview of the user's job search and quick access to recent activity.

Home is not an analytics dashboard.

---

## 11.1 Header

Use the shared application shell:

**OfferBuddy | Home | Applications | Analytics**

Home uses the green active underline.

---

## 11.2 Welcome Area

Display:

**Welcome back, {displayName} 👋**

Supporting text should remain lightweight.

---

## 11.3 Summary Cards

Display the approved high-level job-search snapshot:

* Applications
* Interviews
* Offers

These are quick overview metrics rather than a replacement for Analytics.

Cards should follow the final Figma visual treatment.

---

## 11.4 Browser Extension Discovery

Provide the lightweight:

**Get the Extension**

discovery card.

Use the official O+Check logo.

Purpose:

Explain that users can save jobs directly while browsing SEEK / Indeed.

This is a discovery/onboarding element, not the primary permanent Home workflow.

Once Extension adoption can be reliably known, this element may be hidden or adapted rather than permanently occupying Home.

---

## 11.5 Recent Applications

Display recent Applications using the final table design.

Relevant information includes:

* Position
* Company
* Applied date
* Status
* row navigation affordance

Selecting a row/navigation affordance opens Application Detail.

Status uses the shared status badge system.

---

## 11.6 Home Exclusions

Do not restore the old S1 Home emphasis on:

**Paste a job link → Start your next application**

S2's primary capture workflow is the Browser Extension.

Do not turn Home into a large analytics dashboard.

---

# 12. Applications

**Figma node:** `157:2`

## Purpose

Provide the primary workspace for browsing and locating tracked Applications.

---

## 12.1 Navigation

Applications active:

**green underline**

---

## 12.2 Page Header

Display:

**Applications**

Supporting copy:

**Track every application in one place.**

Do not make a large `New Application` CTA dominate this page.

---

## 12.3 Controls

Provide:

### Search

Search relevant Application information such as company/position according to supported backend behaviour.

### Status Filter

Filter by Application status.

### Source Filter

Support relevant sources such as:

* SEEK
* Indeed
* Manual

### Sort

Default/current design:

**Newest first**

---

## 12.4 Application Table

Display the final S2 table structure including relevant:

* Position / Job
* Company
* Applied date
* Source
* Status
* navigation affordance

Status badges follow the shared colour system.

Selecting an Application opens Application Detail.

---

## 12.5 Pagination

Provide pagination according to the existing backend/API behaviour.

Do not silently replace server-side pagination with unlimited client-side loading.

---

# 13. Application Detail

**Figma node:** `152:2`

## Purpose

Provide the complete view of one tracked Application and its associated Job information.

---

## 13.1 Navigation

Applications remains active.

---

## 13.2 Job/Application Header

Display relevant:

* Position
* Company
* Location
* work arrangement where available
* current Application status
* applied date
* source/platform

Provide the approved Application editing action.

---

# 14. Application Summary

Provide a compact summary area for relevant high-value metadata such as:

* Status
* Applied
* Source
* Location
* original job posting access where available

Avoid duplicating large amounts of information already displayed elsewhere.

---

# 15. Job Intelligence

Display downstream AI-derived Job Intelligence when available.

S2 Job Intelligence may include:

* concise factual summary
* responsibilities
* requirements
* skills

The UI should remain useful while Job Intelligence is:

* pending
* unavailable
* failed

Job Intelligence must not control the Application lifecycle.

### Explicit exclusions

Do not introduce:

* match score
* resume tailoring
* cover letter generation
* candidate ranking
* application recommendation

into S2 Application Detail.

---

# 16. Application Details

Display relevant persisted Application-specific information.

Keep Job information and Application information conceptually distinct.

---

# 17. Status History

Display the Application status progression/history using the approved visual treatment.

History is informational.

It reflects persisted Application lifecycle changes.

---

# 18. Job Source

Where source information is available, provide appropriate access to the original posting.

Do not treat external source availability as required for viewing the Application.

---

# 19. Destructive Action

`Delete application`

must remain visually separated from normal primary actions.

Deletion behaviour must follow the confirmed product/backend contract rather than being inferred from the Figma alone.

---

# 20. New Application

**Figma node:** `153:2`

## Purpose

Provide the secondary/manual Application creation workflow.

Browser Extension is the primary S2 capture capability.

New Application remains available for unsupported/manual situations.

---

## 20.1 Navigation

Applications remains active.

---

## 20.2 Job Link Prefill

Provide the lightweight optional:

**Prefill from job link**

workflow.

This is not the dominant page experience.

Flow:

**Paste URL → attempt extraction/parsing → populate form → user reviews/edits → Save**

Parsing failure must not prevent manual Application creation.

Existing AI URL parsing therefore remains a secondary/fallback capability rather than being removed.

---

# 21. Manual Application Form

Provide confirmed fields as supported by the S2 contract, including relevant:

* Company
* Position
* Location
* Job URL
* Applied date
* Status
* Source
* Salary
* Notes

Use the final Figma form hierarchy.

Primary action:

**Save application**

Secondary action:

**Cancel**

---

# 22. Analytics

**Figma node:** `153:61`

## Purpose

Provide small, useful Application tracking analytics.

S2 Analytics is intentionally limited in scope.

---

## 22.1 Navigation

Analytics active:

**green underline**

---

# 23. Analytics Time Range

A time-range selector is required.

Default:

**All time**

Approved UX options:

* All time
* Last 30 days
* Last 90 days
* This year

Changing the range updates the metrics represented by the Analytics page.

The selected time range applies consistently across the displayed Analytics metrics.

---

# 24. Analytics Summary Metrics

Display the approved summary cards:

* Applications
* Interviews
* Offers
* Interview rate

Metrics must reflect the selected time range.

---

# 25. Application Outcomes

Provide a simple breakdown of Application outcomes/statuses.

Keep the visualisation lightweight.

The goal is immediate comprehension, not BI-style exploration.

---

# 26. Conversion Summary

Display the approved simple conversion metrics:

**Applications → Interview**

**Interview → Offer**

**Applications → Offer**

These are descriptive Application tracking metrics.

They are not predictive AI metrics.

---

# 27. Analytics Explicit Exclusions

S2 Analytics should not introduce:

* complex BI dashboards
* predictive analytics
* AI career recommendations
* recruiter/company scoring
* salary analytics platform
* extensive historical trend analysis
* custom dashboard building

Keep S2 Analytics intentionally small.

---

# 28. Shared Visual Rules

Final S2 Web visual language:

* white/light background
* generous whitespace
* restrained black/grey typography
* green used primarily as OfferBuddy semantic/accent colour
* green active-navigation underline
* lightweight borders
* rounded cards
* subtle semantic status colours
* consistent O+Check branding
* clean tables
* restrained shadows/effects
* Inter typography

The implementation must follow Figma for exact visual values rather than treating these descriptions as pixel specifications.

---

# 29. Shared Status Badges

Application statuses should use a consistent badge treatment across:

* Home
* Applications
* Application Detail
* Analytics where relevant

Supported lifecycle statuses remain aligned with the backend/domain contract.

Do not introduce different status terminology solely for UI aesthetics.

---

# 30. Responsive Behaviour

Desktop Figma reference:

**1440 × 1024**

This is a design reference, not a fixed application width.

Implementation should use responsive containers.

Preserve:

* hierarchy
* whitespace
* readable forms
* table usability
* navigation consistency
* card relationships

Do not hard-code the application to exactly 1440px.

Extension uses its separate compact popup sizing rules.

---

# 31. Loading / Empty / Error Behaviour

Static Figma frames do not override behavioural requirements.

Pages consuming backend data must account for:

* loading
* empty
* error
* populated

states.

Agents should reuse the established visual system rather than inventing unrelated designs for these states.

---

# 32. Final Figma Mapping

| Surface                     |      Node |
| --------------------------- | --------: |
| Extension — Default         |   `140:2` |
| Extension — Review Required |   `133:3` |
| Extension — Review Detail   |  `142:34` |
| Extension — Saving          |   `139:2` |
| Extension — Saved           |  `139:44` |
| Extension — Duplicate       |  `139:86` |
| Extension — Auth Required   | `139:128` |
| Extension — Failed          | `139:170` |
| Home                        |   `151:2` |
| Applications                |   `157:2` |
| Application Detail          |   `152:2` |
| New Application             |   `153:2` |
| Analytics                   |  `153:61` |

---

# 33. Implementation Definition of Done

A UI feature is not complete merely because API integration works.

For each screen/state, the implementation Agent must verify:

**Functionality**

* required behaviour works
* correct API integration
* loading/error/success behaviour
* navigation works

**Visual**

* layout matches Figma
* typography matches
* spacing matches
* card treatment matches
* colours match
* status badges match
* O+Check branding matches
* green active underline matches
* icons are consistent

**Product**

* no new S2 scope invented
* no confirmed workflow removed
* Extension remains primary capture path
* manual New Application remains available
* AI does not block core Application creation
* Analytics remains intentionally small

Any deliberate deviation from Figma must be documented rather than silently introduced.

---
