S3-UI-17 — Application Detail Integration

# S3-UI-17 — Application Detail Integration

```yaml
spec_id: S3-UI-17
title: Application Detail Integration
surface: web
status: frozen
figma_page: S3-03 Application Detail Integration
depends_on:
  - s2-application-detail
  - application
  - job
  - preparation
  - match
  - tailored-resume
  - focused-cover-letter
s2_reuse: true
integration_type: additive
```

## 1. Purpose

`S3-UI-17` defines how the new S3 Focused Preparation capability integrates into the **existing, latest S2 Application Detail page**.

This is deliberately an **integration specification**, not a redesign specification.

The core rule is:

> **S2 Application Detail remains the authoritative Application page. S3 adds Preparation context to it without replacing its existing structure or responsibilities.**

Conceptually:

```text
S2 Application Detail
├── existing Job/Application information
├── existing application status
├── existing S2 actions
├── existing notes / timeline / metadata
│
└── S3 additive integration
    └── Focused Preparation
```

------

# 2. Product Context

There are two valid routes to an Application.

### Fast Apply

```text
Recruitment Website
        ↓
Apply normally
        ↓
S2 Application capture
        ↓
Application (APPLIED, focused_path=false)
```

No Preparation is required.
Application Detail **must not** show the Focused Preparation card.
There is **no** “Prepare with OfferBuddy” CTA on Fast Apply Application Detail.

### Focused Prepare (Prepare with OfferBuddy)

```text
Recruitment Website
        ↓
Prepare with OfferBuddy
        ↓
Application (PREPARING, focused_path=true)  ← appears in Application list
        +
Preparation (Candidate + Job)
        +
async Match request (when prerequisites allow)
        ↓
Application Detail → Open preparation → Match / Resume / Cover Letter
        ↓
External application submission
        ↓
Application status promotes PREPARING → APPLIED (focused_path stays true)
```

Focused path **creates the Application immediately** in `PREPARING` so the user can track it in the list and open AI analysis from Application Detail.

There is still **not** a separate “Focused Application” aggregate — it is the same Application resource with `focused_path=true` and an initial `PREPARING` status.

Therefore:

```text
Fast Apply Application
  → Detail without Focused Preparation card

Focused Prepare Application
  → Detail with Focused Preparation card (“Only shown for focused applications”)
```

------

# 3. Scope

### In Scope

- preserve latest S2 Application Detail;
- add Focused Preparation section/card;
- show whether Preparation exists;
- provide entry/re-entry into Preparation;
- surface high-level Preparation state;
- connect Application's Job to Preparation;
- handle stale Preparation/artifact indication;
- preserve existing Application status management;
- support Applications created through Fast Apply;
- support Applications that previously used Focused Preparation.

### Out of Scope

- redesigning Application Detail;
- moving Match into Application aggregate;
- embedding full Match Analysis;
- embedding full Tailored Resume editor;
- embedding full Cover Letter editor;
- replacing S2 Application status;
- creating another Application lifecycle;
- creating Saved Job;
- Auto Apply;
- rebuilding S2 notes/timeline/history;
- changing S2 Application capture semantics.

------

# 4. Figma Reference

**Figma Page**

```
S3-03 Application Detail Integration
```

This page was intentionally created separately from the S2 Application Detail design.

Reason:

```text
S2 page
= frozen baseline

S3 page
= additive integration reference
```

The Agent must inspect:

1. latest S2 Application Detail implementation/design;
2. `S3-03 Application Detail Integration`;
3. S3 annotation frames.

The Agent must **not implement an old Application Detail mockup** merely because an earlier Figma page still exists.

------

# 5. Baseline Rule

Before implementing this specification, the Agent must inspect the actual current S2 Application Detail code.

The baseline is:

> **the latest implemented S2 Application Detail**, not an earlier design snapshot.

S3 should then apply the smallest required delta.

Correct:

```text
Current S2 implementation
        +
Focused Preparation integration
        ↓
S3 Application Detail
```

Incorrect:

```text
Old Figma Application Detail
        ↓
rebuild page
        ↓
lose newer S2 changes
```

------

# 6. Existing S2 Responsibilities

Application Detail continues to own/display the existing S2 Application concerns.

Examples include whatever is present in the final S2 implementation:

- company;
- role;
- application status;
- applied date;
- Job information;
- source/platform;
- Job URL;
- notes;
- existing actions;
- existing status history/timeline;
- existing layout/navigation.

S3 must not duplicate these fields inside Preparation.

------

# 7. S3 Additive Section

S3 adds one conceptual area:

> **Focused Preparation**

Its purpose is to answer:

> Did I prepare specifically for this Job, and can I open that preparation?

Conceptually:

```text
Focused Preparation

[ state / summary ]

Match analysis
Tailored resume
Focused cover letter

[ Open preparation ]
```

Exact visible content follows frozen Figma.

This is a **summary/entry point**, not the full Preparation workspace.

------

# 8. Preparation Relationship

Preparation remains anchored to:

```text
Candidate + Job
```

not:

```text
Application
```

This is critical.

Application Detail can locate Preparation because the Application references the relevant Job and belongs to the authorised user context.

Conceptually:

```text
Application
    ↓
   Job
    ↑
Candidate + Job
    ↓
Preparation
```

Do not remodel this as:

```text
Application
    ↓
Preparation
```

ownership.

The UI relationship does not redefine the domain relationship.

------

# 9. Why Preparation Is Linked Through Job — and Application Still Appears Early on Focused Path

Focused Preparation remains anchored to Candidate + Job.

On the **Focused Prepare** path, OfferBuddy also creates an Application in `PREPARING` immediately so:

1. the Application list shows an in-progress focused item;
2. Application Detail can host the Focused Preparation entry and AI analysis entry points;
3. later external submission promotes the same Application to `APPLIED`.

Preparation is still not nested under Application as an owned child aggregate — Application Detail locates Preparation via the shared Job (+ authorised user).

Example:

```text
10:00
User finds Job

10:01
Prepare with OfferBuddy

10:01
Application created (PREPARING, focused_path=true)
Preparation created
Match requested asynchronously when ready

10:03
User opens Application Detail → Open preparation

10:07
User submits SEEK application

10:08
Same Application becomes Applied (focused_path remains true)
```

------

# 10. Application Detail After Focused Apply

After submission, Application Detail provides a convenient way back to the earlier Preparation.

Example:

```text
Application Detail

Software Engineer
Company A
Applied

...

Focused Preparation
────────────────────────

Preparation completed

Match analysis
Tailored resume
Focused cover letter

[ Open preparation ]
```

The user does not need to reconstruct the preparation from scratch.

------

# 11. Application Detail After Fast Apply

A Fast Apply Application has `focused_path=false` and no Focused Preparation card.

Example:

```text
Application Detail

Software Engineer
Company B
Applied

...

(no Focused preparation sidebar card)
```

This is normal.

It does not indicate an error or incomplete Application.

Fast Apply is a first-class valid workflow and **must not** offer Prepare-from-Detail.

------

# 12. Post-Application Preparation

Post-submission “start Focused Prepare from a Fast Apply Application Detail” is **out of product scope**.

- Fast Apply Detail: no Focused Preparation card / no Prepare CTA.
- Focused Prepare starts only from Extension (or an explicit Focused entry that creates `PREPARING` + `focused_path=true`).

Domain technically still allows Candidate + Job Preparation independently, but Application Detail must follow the frozen UI:

> Only shown for focused applications.

Do not invent a post-Fast-Apply prepare workflow on Application Detail.

------

# 13. Preparation Summary

Application Detail should show only enough information to help the user decide whether to open Preparation.

Possible summary concepts:

```text
Preparation status
Match available
Tailored Resume available
Focused Cover Letter available
Stale / review required
```

Do not duplicate detailed content.

Application Detail should **not** contain:

- Match factor breakdown;
- full Resume preview;
- Resume editing;
- full Cover Letter body;
- Cover Letter editing.

Those remain in their dedicated S3 pages.

------

# 14. No Match Score Requirement

Unlike the Extension, Web Application Detail *may* technically display high-level Match information if the frozen Figma explicitly includes it.

However, the integration does not require duplicating Match score.

Preferred boundary:

```text
Application Detail
→ preparation state / availability

Preparation Web
→ actual Match analysis
```

Do not add Match score just because the data is available.

The Agent should follow frozen Figma.

------

# 15. Preparation States

Application Detail needs to handle at least the conceptual states:

```text
NONE
IN_PROGRESS
READY
STALE
FAILED / ATTENTION
```

Exact backend status enums must come from the frozen contracts.

Do not create frontend-only domain statuses if the backend already provides the necessary state.

------

# 16. State — No Focused Preparation Card (Fast Apply)

This represents an Application created through Fast Apply (`focused_path=false`).

UI must **omit** the Focused Preparation sidebar card entirely.

It must not communicate:

> Incomplete application.

Fast Apply without preparation is a complete, valid Application.

------

# 17. State — Preparation In Progress

If Preparation exists but some asynchronous work is still running:

```text
Preparation
    ↓
Match / Resume / CL operation pending
```

Application Detail may show a high-level:

> Preparation in progress

and allow navigation to the Preparation workspace.

Do not reproduce detailed polling UI for every artifact inside Application Detail.

------

# 18. State — Preparation Ready

When usable preparation exists:

> Focused preparation ready

Primary action:

> Open preparation

The user can then inspect:

- Match;
- Tailored Resume;
- Focused Cover Letter.

Application Detail remains compact.

------

# 19. State — Stale

Preparation-derived artifacts carry provenance.

Example:

```text
Match generated from:
Profile v20
Job content v7

Current:
Profile v21
Job content v7
```

The Preparation may now require review/regeneration.

Application Detail can surface:

> Preparation needs review

or the exact frozen Figma copy.

It must not independently decide how to regenerate everything.

The user opens Preparation.

------

# 20. Staleness Source

Application Detail should consume backend-provided/frozen provenance semantics.

It should not reproduce complex rules such as:

```text
if profileVersion != sourceProfileVersion
and ...
```

independently for every artifact if the API already provides usable staleness information.

Backend remains authoritative for business-resource state.

------

# 21. State — Failed / Attention

If an asynchronous preparation operation failed:

Application Detail may show a high-level attention state.

For example:

> Preparation needs attention

Action:

> Open preparation

Detailed failure/retry behaviour belongs to the corresponding Preparation capability.

Do not place AI provider errors or raw failure payloads directly into Application Detail.

------

# 22. Application Status Remains Independent

Application status continues to belong to S2 Application.

Examples:

```text
APPLIED
INTERVIEW
OFFER
REJECTED
WITHDRAWN
NO_RESPONSE
```

Preparation state does not modify these.

Incorrect:

```text
Preparation Ready
→ Application status = APPLIED
```

Correct:

```text
Preparation Ready
→ Application remains whatever its actual lifecycle says
```

------

# 23. Preparation State Remains Independent

Likewise:

```text
Application = REJECTED
```

does not automatically delete Preparation.

The Preparation remains historical/reusable evidence of what was prepared for that Job, subject to normal retention rules.

Do not couple lifecycle transitions unnecessarily.

------

# 24. Application Editing

Existing S2 Application editing/status behaviour remains unchanged.

If S2 Application uses:

```text
applications.version
```

for optimistic concurrency, continue using it.

Do not replace it with:

```text
candidateProfile.profileVersion
```

or Job `contentVersion`.

These versions serve different resources.

------

# 25. Three Independent Versions

The Agent must keep these concepts separate:

```text
Candidate Profile
→ profileVersion

Job canonical semantic content
→ contentVersion

Application
→ version
```

Derived preparation artifacts may reference:

```text
sourceProfileVersion
sourceJobContentVersion
```

Application Detail may display their resulting state, but it must not collapse these into one generic `version`.

------

# 26. Fast Apply Compatibility

S2 Fast Path must remain fully operational:

```text
Job
 ↓
Apply externally
 ↓
Extension / S2 capture
 ↓
Application
 ↓
Application Detail
```

No Candidate Profile is required merely to view/manage the Application.

No Preparation is required.

No Match is required.

No Resume generation is required.

No Focused Cover Letter is required.

This is one of the most important acceptance requirements.

------

# 27. Focused Apply Compatibility

Focused path:

```text
Job
 ↓
Prepare with OfferBuddy
 ↓
Preparation
 ↓
optional Match / Resume / CL work
 ↓
External application
 ↓
S2 Application capture
 ↓
Application Detail
 ↓
Preparation summary available
```

S3 extends S2 rather than replacing it.

------

# 28. Job Identity

Application Detail must use the canonical Job relationship already established by S2/S3.

Do not try to find Preparation using:

```text
companyName + jobTitle
```

if a canonical `jobId` relationship exists.

Correct conceptual lookup:

```text
authorised Application
       ↓
jobId
       ↓
Candidate + Job Preparation
```

This avoids accidental collisions between similarly titled roles.

------

# 29. Job Content Changes

Canonical Job may receive a meaningful semantic content revision:

```text
contentVersion n
      ↓
contentVersion n+1
```

Existing Preparation artifacts may become stale.

Application itself does not need to become a different Application solely because Job content changed.

Keep:

```text
Application lifecycle
```

separate from:

```text
Preparation artifact freshness
```

------

# 30. Navigation — Open Preparation

`Open preparation` should navigate to the existing Preparation workspace for:

```text
current authorised Candidate
+
Application Job
```

If no Preparation exists and creation is allowed by the frozen UI state:

> use the normal Preparation creation/resolution workflow.

Do not create a special Application-owned preparation endpoint.

------

# 31. Navigation — Artifact Deep Links

If frozen Figma provides direct links such as:

```text
View Match
View Resume
View Cover Letter
```

they should navigate to the corresponding Web preparation artifact.

They are navigation shortcuts.

They do not embed those editors into Application Detail.

If the frozen UI only provides:

> Open preparation

do not invent extra links.

------

# 32. Loading Behaviour

Application Detail should preserve existing S2 loading behaviour.

S3 Preparation summary should not unnecessarily block the entire page.

Prefer:

```text
Application Detail loads
        +
Preparation summary resolves
```

rather than:

```text
wait for Application
+
Preparation
+
Match
+
Resume
+
CL
before rendering anything
```

The S3 integration is secondary to the Application Detail's core S2 purpose.

------

# 33. Preparation Lookup Failure

If Preparation summary cannot load:

- Application Detail remains usable;
- existing S2 Application data remains visible;
- status changes remain available;
- notes/actions remain available;
- S3 section can show a small unavailable/retry state.

Do not fail the entire Application Detail page because S3 Preparation lookup failed.

------

# 34. Async Operations

Application Detail should not introduce generic async operation IDs or an independent task dashboard.

If Preparation work is asynchronous, continue using the frozen business-resource polling model.

Conceptually:

```text
POST command
→ 202
→ Preparation / artifact business resource
→ poll its state
```

Application Detail only needs the resulting high-level summary.

------

# 35. Application Deletion / Removal

If S2 supports deleting/removing an Application, do not automatically assume the Candidate + Job Preparation should also be deleted.

They are independent resources with different lifecycle semantics.

Follow frozen backend retention/deletion rules.

Do not introduce cascade deletion from UI assumptions.

------

# 36. Candidate Profile Changes

Suppose:

```text
Application exists
Preparation exists
Tailored Resume exists
```

Candidate changes Experience.

Then:

```text
profileVersion increments
```

Application itself remains unchanged.

Preparation artifacts may become stale.

Application Detail should therefore conceptually show:

```text
Application
→ still Applied

Preparation
→ needs review
```

This separation is essential.

------

# 37. Cover Letter Relationship

A Focused Cover Letter belongs to Preparation.

A Default Cover Letter belongs to Fast Apply configuration.

Application Detail should not confuse them.

If the Application was Fast Applied using Default Cover Letter:

> that does not create a Focused Cover Letter artifact.

If a Focused Cover Letter exists:

> Application Detail may expose it through Preparation.

------

# 38. Tailored Resume Relationship

Similarly:

```text
Base Resume
Tailored Resume
```

belong to Candidate/Preparation Resume workflows.

Application Detail does not become the Resume storage owner.

It may provide navigation to the relevant Tailored Resume when available.

------

# 39. Historical Integrity

Application Detail represents what happened in the application lifecycle.

Preparation represents what OfferBuddy prepared.

They should not rewrite each other's history.

Example:

```text
Application submitted using Resume A

Later:
User regenerates Tailored Resume B
```

OfferBuddy must not automatically imply:

> Resume B was the resume actually submitted

unless submission-artifact tracking explicitly establishes that fact.

Do not infer historical submission facts from current Preparation state.

------

# 40. UI Structure

Conceptually (Focused Prepare applications only):

```text
┌─────────────────────────────┬──────────────────────┐
│ Existing S2 Application     │ Status History       │
│ Detail (main)               │                      │
│                             │ Focused preparation  │
│ Job Intelligence (AI)       │  summary + CTA       │
│ Application details / notes │  “Only shown for     │
│                             │   focused apps”      │
│                             │ Delete application   │
└─────────────────────────────┴──────────────────────┘
```

Fast Apply applications use the same layout **without** the Focused preparation card.

The actual position/layout follows frozen Figma `S3 Application Detail / Focused Preparation`.

Do not move existing S2 components simply to make S3 visually dominant.

------

# 41. Visual Hierarchy

Application Detail remains about:

> **the Application**

Therefore:

```text
Application information
        >
Preparation integration
```

Preparation is useful but secondary.

Avoid turning the page into:

```text
Match dashboard
Resume dashboard
Cover Letter dashboard
+
tiny Application status
```

That would invert the domain/product hierarchy.

------

# 42. UI States

S3 integration must support:

### Loading

Preparation summary resolving independently.

### None

No focused Preparation.

### In Progress

Preparation exists and work remains underway.

### Ready

Preparation is usable.

### Stale / Review Required

One or more relevant artifacts need review.

### Attention / Failure

Preparation requires user attention.

### Unavailable

Summary lookup failed, while Application Detail remains usable.

------

# 43. Business Rules

1. Latest S2 Application Detail is the baseline.
2. S3 integration is additive.
3. Do not rebuild Application Detail from an old design.
4. Application remains independently user-owned.
5. Preparation remains anchored to Candidate + Job.
6. Preparation is not Application-owned.
7. Focused Prepare creates Application immediately in `PREPARING` (`focused_path=true`).
8. Fast Apply does not require Preparation and must not show the Focused Preparation card.
9. Focused Prepare ultimately uses the same Application model.
10. No separate Focused Application aggregate exists.
11. Application status and Preparation status are independent.
12. Preparation Ready does not mean Applied.
13. Applied does not mean Preparation exists.
14. Match remains a Preparation-derived capability.
15. Tailored Resume remains a Preparation artifact.
16. Focused Cover Letter remains a Preparation artifact.
17. Default Cover Letter is not Focused Cover Letter.
18. Application Detail should not duplicate full artifact editors.
19. Candidate Profile changes may stale Preparation without changing Application.
20. Job semantic changes may stale Preparation without changing Application lifecycle.
21. Application OCC uses `applications.version`.
22. Candidate Profile OCC uses `profileVersion`.
23. Job semantic revision uses `contentVersion`.
24. These versions must not be conflated.
25. Preparation lookup failure must not break S2 Application Detail.
26. S2 Application capture remains intact.
27. S2 Fast Path remains intact.
28. S3 must not infer which Resume/Cover Letter was historically submitted without explicit evidence.

------

# 44. API Mapping

Use the frozen §3.17 contracts.

### Existing Application Detail

Continue using the S2-compatible Application API.

### Preparation Summary

Resolve through the frozen Preparation API using authorised Candidate + Job context.

Do not invent:

```text
GET /api/applications/{id}/preparation
```

if this implies Application ownership and no such frozen contract exists.

Prefer the already frozen Preparation capability contract.

### Application Updates

Continue using:

```text
applications.version
```

where required by the frozen S2 compatibility contract.

### Preparation Commands

Use Preparation business-resource commands.

Do not send Application version as Preparation OCC.

------

# 45. Authorization

Application and Preparation have related but distinct authorization paths.

### Application

Server verifies:

```text
Application belongs to authenticated user
```

### Preparation

Server verifies through:

```text
Candidate ownership
+
authorised Job relationship
```

Application Detail being able to link to Preparation does not bypass Candidate authorization.

Do not trust:

```text
candidateId
applicationId
jobId
```

sent by frontend as proof of ownership.

------

# 46. Privacy

Application Detail already contains user-owned Job/Application information.

The S3 section must not unnecessarily fetch:

- full Candidate Profile;
- full Match payload;
- Resume body;
- Cover Letter body;

just to render a small Preparation summary.

Prefer narrow summary contracts.

------

# 47. Observability

Useful operational metadata:

- requestId;
- applicationId;
- jobId;
- preparationId where present;
- high-level preparation state;
- error category.

Do not log:

- Resume body;
- Cover Letter body;
- Candidate Profile content;
- Match explanation;
- raw Job description.

------

# 48. Performance

Application Detail should remain responsive.

The Agent should avoid:

```text
Application Detail request
   ↓
load Profile
   ↓
load Match
   ↓
load Resume
   ↓
load Cover Letter
   ↓
render
```

Prefer:

```text
Application Detail
   ↓
render core S2 page

Preparation summary
   ↓
small independent read
```

Detailed artifacts load only after the user opens Preparation.

------

# 49. S2 Reuse

### Reuse Completely

- Application Detail shell;
- Application information;
- status controls;
- notes;
- timeline/history;
- existing Job metadata;
- existing navigation;
- existing Application APIs;
- existing Application OCC;
- existing loading/error conventions.

### S3 Add

Only the frozen Focused Preparation integration.

### Do Not

- fork Application Detail into S2/S3 versions in code;
- create `FocusedApplicationDetail`;
- replace existing Application APIs;
- migrate S2 status management into Preparation.

The separate Figma page exists for design/specification clarity, **not because the implementation should create two production Application Detail pages**.

------

# 50. Component Reuse

Possible implementation responsibility:

```text
ApplicationDetailPage        ← existing S2
        │
        └── PreparationSummaryCard   ← S3 additive
```

Possible supporting responsibilities:

```text
PreparationSummaryCard
PreparationStatusBadge
PreparationSummaryClient
```

Names are illustrative.

Avoid:

```text
S3ApplicationDetailPage
```

unless there is an actual architectural reason beyond design-page separation.

------

# 51. Accessibility / UX Requirements

- Preparation state must not depend solely on colour.
- CTA clearly states what will open.
- Stale/attention state should explain that review is needed.
- S3 loading must not block core Application controls.
- Failure in S3 section must not disable S2 actions.
- Keyboard navigation follows existing Application Detail patterns.
- Existing visual hierarchy must remain recognisable.

------

# 52. Acceptance Criteria

-  Latest implemented S2 Application Detail is used as baseline.
-  Existing S2 layout/functionality is preserved.
-  S3 changes are additive.
-  Focused Preparation section is visible according to frozen Figma.
-  Fast Apply Application with no Preparation remains completely valid.
-  No Preparation state is represented neutrally.
-  Existing Preparation can be reopened.
-  Preparation Ready does not alter Application status.
-  Applied status does not imply Preparation exists.
-  Preparation remains Candidate + Job anchored.
-  No Application-owned Preparation aggregate is introduced.
-  No separate Focused Application model is introduced.
-  Match is not moved into Application.
-  Tailored Resume is not moved into Application.
-  Focused Cover Letter is not moved into Application.
-  Full Match is not unnecessarily duplicated on Application Detail.
-  Full Resume editor is not embedded.
-  Full Cover Letter editor is not embedded.
-  Candidate Profile changes can surface stale Preparation state.
-  Job content changes can surface stale Preparation state.
-  Application itself remains independently versioned.
-  `applications.version`, `profileVersion`, and `contentVersion` remain distinct.
-  Preparation lookup failure does not break core Application Detail.
-  S2 status editing remains functional.
-  S2 Application capture remains unchanged.
-  Fast Apply remains independent of Candidate Profile/Preparation.
-  Focused Apply converges onto the same S2 Application lifecycle.
-  Historical submission artifacts are not inferred from current Preparation state.
-  Server-side ownership is enforced independently for Application and Preparation.

------

# 53. Agent Implementation Notes

1. **Open the current S2 Application Detail implementation first.**
2. Treat current code as baseline, not old Figma.
3. Inspect `S3-03 Application Detail Integration`.
4. Identify the smallest S3 UI delta.
5. Add Preparation integration rather than rebuilding the page.
6. Keep Application and Preparation domain ownership separate.
7. Resolve Preparation using frozen Candidate + Job semantics.
8. Do not introduce Application → Preparation ownership.
9. Preserve S2 Application APIs and `applications.version`.
10. Do not use `profileVersion` for Application writes.
11. Do not use Application version for Preparation writes.
12. Load Preparation summary independently where practical.
13. Do not fetch full Match/Resume/CL merely to render the summary.
14. Reuse Preparation workspace for detailed review.
15. Preserve Fast Apply with zero Preparation.
16. Do not mark Application Applied when Preparation becomes ready.
17. Do not infer submitted Resume/CL from current artifacts.
18. Do not create a second production Application Detail page.
19. Keep S3 integration visually secondary to Application information.
20. Preserve S2 loading/error/status behaviour.
21. Ignore Figma annotations.
22. Do not redesign frozen UI.

```
`S3-UI-17` 可以冻结。

这里最重要的是把之前我们反复纠正过的关系彻底锁死：

​```text
                 Candidate
                     │
                     │
                     ▼
Job ─────────── Preparation
 │                   │
 │              ┌────┼────┐
 │              ▼    ▼    ▼
 │            Match Resume CL
 │
 ▼
Application
 │
 ▼
Application Detail
 │
 └──── 只是找到并展示 Preparation 入口
```

**不是：**

```text
Application
    ↓
Preparation
    ↓
Match / Resume / CL
```

同时，Figma 里我们专门新开 `S3-03 Application Detail Integration` Page，是为了让 **S2 baseline 和 S3 增量设计分开 review**；真正实现的时候不能因此写两个 Application Detail。

下一份可以进入 **`S3-UI-18 — Sponsor Employers Admin / CRUD`**。这会把我们已经设计好的 RuoYi 原生页面、CRUD、批量导入、published snapshot/version，以及 Extension 只消费发布数据这一整条链冻结成 Agent 可实现 specification。