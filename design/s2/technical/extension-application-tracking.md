# Extension Application Tracking — Functional & Interaction Specification

**Sprint:** S2  
**Status:** Implementation-facing specification (accepted for Sprint 2 closeout)  
**Scope:** Browser Extension application tracking workflow  
**Supported platforms:** SEEK, Indeed  
**Out of scope:** LinkedIn, cover-letter generation, resume tailoring, job-match scoring, ATS autofill, auto-apply

### Closeout Authority

This document describes the confirmation-based / companion tracking model that shipped alongside the explicit popup **Save to OfferBuddy** path.

Both paths use the same Backend Track API. For the authoritative planned-vs-implemented summary, see [Implementation Reconciliation](../../../delivery/s2/implementation-reconciliation.md) and the Closeout Notes in [Extension Design](extension-design.md).

---

## 1. Purpose

The Sprint 2 OfferBuddy Browser Extension is a lightweight application-tracking companion for supported job sites.

Its primary purpose is to reduce the effort required to capture real job applications in OfferBuddy.

The intended experience is:

> The user applies for jobs normally. OfferBuddy quietly makes sure successfully submitted applications are tracked.

The extension should not require the user to repeatedly open the browser extension popup or manually save every supported application.

The extension should remain unobtrusive during normal job browsing and intervene only when it provides useful information or requires user input.

---

## 2. Core Product Principles

### 2.1 Default quiet

OfferBuddy should remain visually quiet when there is nothing requiring the user's attention.

Normal job browsing must not produce unnecessary notifications, success indicators, popups, or repeated interactions.

### 2.2 Automatic preparation, minimal user action

OfferBuddy may automatically:

* detect the currently selected job;
* extract reliable job facts;
* evaluate supported eligibility signals;
* preserve application context;
* detect supported application workflows;
* detect reliable submission confirmation;
* track a confirmed application.

The user should not need to manually initiate these operations.

### 2.3 Submission confirmation, not Apply click

Clicking an Apply button does not mean that an application has been submitted.

Clicking a final Submit button also does not by itself prove successful submission.

An Application may be automatically tracked only when the relevant site adapter can reliably confirm that submission succeeded.

### 2.4 Reliable confirmation → automatic tracking

When a supported site provides reliable submission confirmation:

> OfferBuddy automatically tracks the application.

### 2.5 Uncertain submission → user confirmation

When OfferBuddy knows that an application flow was started but cannot reliably determine whether the application was successfully submitted:

> OfferBuddy asks the user whether the application was submitted.

### 2.6 No submission → no Application

Browsing a job, opening an application flow, clicking Apply, or abandoning an application must not create an OfferBuddy Application.

### 2.7 Site-specific facts stay inside site adapters

SEEK-specific and Indeed-specific DOM, URL, navigation, and confirmation rules belong to their respective site adapters.

Shared extension infrastructure should operate on normalized application lifecycle events rather than understanding site-specific page structures.

---

# 3. Supported Browsing Model

SEEK and Indeed primarily use a dynamic job browsing experience where a job list and the currently selected job description may exist within the same browsing surface.

OfferBuddy must therefore treat job selection as a dynamic context rather than assuming that each job corresponds to a traditional full-page navigation.

Conceptually:

```text
Job Search Surface
        |
        +-- Job A
        +-- Job B  <-- selected
        +-- Job C
        |
        +-- Current Job Description
```

The extension maintains one:

`CurrentJobContext`

for the currently selected supported job.

---

# 4. Dynamic Job Context

## 4.1 Context detection

When the selected job changes, the site adapter must detect the new job context.

This may be caused by:

* selecting another result;
* SPA navigation;
* URL changes;
* DOM replacement;
* asynchronous content loading;
* supported navigation between job surfaces.

## 4.2 Stale-context clearing

When a job change is detected, OfferBuddy must invalidate the previous job context before presenting information for the new job.

Conceptually:

```text
Job A selected
    |
CurrentJobContext = A
    |
User selects Job B
    |
Invalidate A
    |
Extract B
    |
CurrentJobContext = B
```

Eligibility warnings, extracted facts, tracking state, and Popover content from Job A must never remain visible as if they belonged to Job B.

## 4.3 Mutation coalescing

Supported job sites may generate many DOM mutations while changing the selected job.

The extension should coalesce/debounce equivalent changes rather than repeatedly rebuilding the same job context.

Correctness takes priority over reacting to every individual mutation.

---

# 5. Floating Companion

## 5.1 Purpose

The Floating Companion is the primary lightweight extension entry point during job browsing.

It is not intended to reproduce the OfferBuddy web application inside the job site.

For Sprint 2 its responsibilities are limited to:

1. providing a persistent lightweight OfferBuddy entry point;
2. signalling relevant eligibility warnings;
3. exposing the current job state through a small Popover;
4. providing lightweight application-tracking feedback;
5. supporting user confirmation when automatic tracking is not reliable.

---

## 5.2 Position

The Floating Companion should be rendered as a viewport-level overlay rather than being attached to the job-description DOM.

It should:

* remain fixed while the page or internal job-description container scrolls;
* remain within the visible viewport;
* avoid covering important site controls where practical;
* support user repositioning;
* preserve the user's preferred position.

For Sprint 2, edge-constrained vertical dragging is preferred over unrestricted free-form positioning.

---

## 5.3 Normal state

When the current job has no relevant warning requiring attention, the Floating Companion remains visually neutral.

No green success dot or "everything is OK" notification is required.

Absence of an attention indicator means there is nothing currently requiring the user's attention.

---

## 5.4 Eligibility attention state

When OfferBuddy detects a supported eligibility concern, the Floating Companion displays a small red attention indicator.

Examples include supported signals relating to:

* Australian citizenship;
* permanent residency;
* security clearance;
* working-right restrictions;
* sponsorship-related restrictions.

The red indicator means:

> OfferBuddy found something in this job that may require attention.

It must not automatically mean:

> You are not eligible.

OfferBuddy should surface the detected evidence and allow the user to make the final decision.

---

# 6. Popover

Clicking the Floating Companion opens a small Popover.

A Side Panel is not required for Sprint 2.

The Popover may display:

* job title;
* company;
* eligibility warning summary;
* relevant source evidence;
* application tracking state;
* manual/fallback tracking action where appropriate.

The Popover should remain intentionally small and focused.

It must not contain future features such as cover-letter generation, match scoring, resume tailoring, or ATS autofill in Sprint 2.

---

## 6.1 Popover and job-context changes

If the currently selected job changes while the Popover is open:

1. close the Popover;
2. invalidate the previous job context;
3. extract and validate the new job context;
4. update the Floating Companion state.

The Popover should not silently replace one job with another while remaining open.

---

# 7. Eligibility Behaviour

Eligibility screening runs automatically for the current validated job context.

The normal path is:

```text
Current job detected
        |
Eligibility screening
        |
   +----+----+
   |         |
No warning  Warning
   |         |
quiet      red dot
             |
          user opens
             |
        show evidence
```

OfferBuddy should avoid unnecessary positive status indicators.

The eligibility feature is intended primarily as an exception signal.

---

# 8. Application Lifecycle

The extension uses a logical application lifecycle independent of individual site implementations.

Conceptually:

```text
BROWSING
   |
   | application flow starts
   v
APPLICATION_PENDING
   |
   | user proceeds through application
   v
APPLICATION_IN_PROGRESS
   |
   | submission attempted
   v
AWAITING_CONFIRMATION
   |
   +----------------------------+
   |                            |
Reliable confirmation      Cannot confirm
   |                            |
   v                            v
CONFIRMED                  UNCERTAIN
   |                            |
   v                            v
AUTO TRACK                ASK USER
   |
   v
TRACKED
```

These are logical states. Implementation does not need to expose every state directly to the user.

---

# 9. Pending Application Context

When OfferBuddy detects that the user has entered a supported application workflow, it preserves a:

`PendingApplicationContext`

The context should contain sufficient validated information to associate a later submission confirmation with the original job.

Conceptually it may include:

```text
platform
externalJobId
title
company
location
sourceUrl
validated job facts required for ingestion
startedAt
```

The pending context must survive navigation away from the original job browsing surface where required.

It must not itself create an OfferBuddy Application.

---

# 10. Application Start

Application start indicates:

> The user has entered an application workflow for this job.

It does not indicate:

> The user applied successfully.

Examples include:

* SEEK Quick Apply opening the SEEK application workflow;
* Indeed "Apply with Indeed" opening Indeed Smart Apply;
* a supported Apply action transitioning into another application surface.

At this point OfferBuddy preserves the pending context and waits.

---

# 11. Application In Progress

While the user is completing:

* resume selection;
* employer questions;
* profile information;
* application review;
* other supported application steps;

OfferBuddy should remain quiet.

The Floating Companion may be hidden or visually minimized during focused application workflows when it provides no immediate value.

OfferBuddy must continue maintaining the pending application context in the background.

---

# 12. Submit Attempt

A click on:

* `Submit application`;
* `Submit your application`;
* or an equivalent site-specific action

may be observed as a lifecycle signal.

However:

> Submit click is never sufficient evidence for automatic tracking.

Possible outcomes after a Submit click include:

* validation failure;
* network failure;
* backend failure;
* site error;
* additional application steps;
* successful submission.

OfferBuddy therefore waits for reliable site confirmation.

---

# 13. Submission Confirmation

Submission confirmation is a site-adapter responsibility.

The adapter must use sufficiently reliable site-specific signals to determine that the application was successfully submitted.

Where practical, multiple independent signals should be combined.

Examples include:

* known success route;
* confirmation DOM;
* submitted/applied state;
* matching pending application context.

Only confirmed submission may trigger automatic tracking.

---

# 14. SEEK Application Tracking

## 14.1 Application identity

SEEK provides a stable job identifier in its job and application URLs.

Example lifecycle:

```text
/job/{jobId}
        |
        v
/job/{jobId}/apply
        |
        v
/job/{jobId}/apply/success
```

The same `{jobId}` can therefore be associated with the pending application throughout the supported SEEK workflow.

---

## 14.2 SEEK application start

Entering the supported SEEK application workflow for:

```text
/job/{jobId}/apply
```

establishes or updates the pending application context for:

```text
SEEK:{jobId}
```

No Application is created at this stage.

---

## 14.3 SEEK successful submission

Observed SEEK behaviour provides both:

1. a success route:

```text
/job/{jobId}/apply/success
```

2. explicit confirmation content equivalent to:

```text
Your application has been sent to <company>
```

The SEEK adapter may treat the combination of:

```text
matching PendingApplicationContext
+
matching SEEK jobId
+
known success route
+
expected success confirmation DOM
```

as high-confidence submission confirmation.

This may trigger automatic OfferBuddy tracking.

---

# 15. Indeed Application Tracking

## 15.1 Application workflow

Supported Indeed applications may transition from the job browsing surface to:

```text
smartapply.indeed.com
```

The workflow may include:

```text
resume selection
        |
employer questions
        |
review
        |
submit
        |
post-apply
```

---

## 15.2 Indeed pending context requirement

Unlike the observed SEEK success route, the Indeed Smart Apply success route does not necessarily expose the original external job identifier directly.

Therefore the original validated Indeed job identity must be preserved before leaving the job browsing context.

The pending context remains the authoritative association between the application workflow and the original Indeed job.

Company-name text on the success screen must not be used as the canonical job identity.

For example, presentation differences such as:

```text
Caseware
```

and:

```text
CaseWare International
```

must not cause identity reconstruction based on company text.

---

## 15.3 Indeed successful submission

Observed Indeed behaviour provides:

1. a post-application route equivalent to:

```text
.../form/post-apply
```

2. explicit confirmation content equivalent to:

```text
Your application was submitted to <company>
```

The Indeed adapter may treat the combination of:

```text
valid PendingApplicationContext
+
known post-apply route
+
expected submission confirmation DOM
```

as high-confidence submission confirmation.

This may trigger automatic OfferBuddy tracking using the job identity preserved in the pending context.

---

# 16. Automatic Tracking

When submission is reliably confirmed:

```text
SUBMISSION_CONFIRMED
        |
        v
validated PendingApplicationContext
        |
        v
authenticated OfferBuddy ingestion
        |
        v
Application tracked
```

The user should not need to manually open the extension and click Save.

---

## 16.1 Success feedback

Automatic tracking should produce lightweight feedback.

For example:

```text
✓ Tracked
```

The feedback should disappear automatically after a short period.

No confirmation modal or additional acknowledgement should be required.

The job site already provides the primary submission-success feedback; OfferBuddy only needs to confirm that tracking succeeded.

---

# 17. Idempotency

Submission confirmation may be observed more than once because of:

* page refresh;
* back/forward navigation;
* content-script reinjection;
* repeated DOM mutations;
* extension recovery;
* duplicate lifecycle events.

Repeated confirmation for the same canonical job/application must not create multiple Applications.

The extension should suppress unnecessary duplicate tracking attempts where practical.

The backend remains the authoritative duplicate-protection boundary.

---

# 18. Uncertain Submission

If OfferBuddy has a valid pending application context but cannot reliably confirm whether submission succeeded, it must not silently create an Application.

Instead, OfferBuddy may enter an attention state and ask:

```text
Did you submit this application?

<Job title>
<Company>

[ Yes, track it ]
[ Not yet ]
```

Selecting `Yes, track it` is an explicit user confirmation and may trigger tracking.

Selecting `Not yet` preserves or dismisses the pending state according to the surrounding workflow without creating an Application.

---

# 19. External ATS Behaviour

An Apply action may navigate from SEEK or Indeed to an external employer or ATS website.

For Sprint 2, OfferBuddy does not attempt broad ATS-specific submission detection.

Instead:

1. preserve the validated originating job context;
2. recognise that an external application flow has started where reliably detectable;
3. do not create an Application merely because the user left the job site;
4. automatically track only if reliable supported confirmation exists;
5. otherwise use explicit user confirmation.

External ATS autofill is out of scope for Sprint 2.

Broad ATS-specific automation is also out of scope.

---

# 20. Duplicate Behaviour

Duplicate handling is not a primary Extension interaction.

Supported recruitment sites may already indicate previously submitted applications, and the OfferBuddy backend provides duplicate validation.

If the backend reports that the Application is already tracked, the extension should treat this as a normal resolved outcome rather than a generic system failure.

A lightweight state such as:

```text
Already tracked
```

is sufficient for Sprint 2.

---

# 21. Authentication Failure

Automatic tracking requires a valid authenticated Extension-to-Backend relationship.

If submission is confirmed but OfferBuddy cannot track it because authentication is unavailable or expired:

* do not discard the confirmed/pending context prematurely;
* expose an auth-required state;
* allow the user to recover authentication;
* avoid silently losing the application.

Authentication recovery behaviour must remain consistent with the Extension authentication specification.

---

# 22. Tracking Failure

If submission is confirmed but backend tracking fails:

* preserve sufficient context for recovery;
* do not incorrectly display `Tracked`;
* expose a lightweight failure/attention state;
* allow retry where safe;
* preserve idempotency.

The recruitment-site application itself has already succeeded; OfferBuddy tracking failure must therefore be presented as a tracking problem, not an application-submission failure.

---

# 23. Site Adapter Responsibilities

Each supported site adapter owns site-specific detection behaviour.

Conceptually, adapters provide capabilities equivalent to:

```text
detectCurrentJobContext()
extractJobFacts()
detectEligibilitySignals()
detectApplicationStart()
detectSubmissionConfirmation()
```

Exact implementation interfaces may differ.

The important ownership rule is:

> Site adapters understand recruitment-site behaviour. Shared Extension Core understands OfferBuddy application lifecycle behaviour.

---

# 24. Shared Extension Core Responsibilities

Shared Extension Core owns:

* Floating Companion lifecycle;
* normalized current-job context;
* stale-context clearing;
* mutation/event coordination;
* pending application context;
* application lifecycle orchestration;
* authenticated backend ingestion;
* automatic tracking;
* idempotent tracking coordination;
* uncertain-submission fallback;
* tracking success/failure feedback.

It must not contain hard-coded SEEK or Indeed DOM assumptions.

---

# 25. Sprint 2 UX Summary

The intended daily-use experience is:

### Normal job

```text
Browse
  |
OfferBuddy detects job
  |
No warning
  |
OfferBuddy stays quiet
```

### Eligibility warning

```text
Browse
  |
Warning detected
  |
Floating Companion shows red dot
  |
User opens Popover if interested
  |
OfferBuddy shows warning + evidence
```

### SEEK / Indeed supported application

```text
Browse job
   |
Apply normally
   |
OfferBuddy preserves context
   |
Complete application
   |
Submit
   |
Recruitment site confirms success
   |
OfferBuddy confirms submission
   |
Auto Track
   |
✓ Tracked
```

### Uncertain / external application

```text
Browse job
   |
Apply
   |
OfferBuddy preserves context
   |
External / unsupported workflow
   |
Cannot reliably confirm submission
   |
Ask user
   |
"Did you submit this application?"
```

---

# 26. Sprint 2 Boundaries

The Floating Companion is intentionally extensible, but Sprint 2 must not expand its scope into a general job-search assistant.

The following are explicitly outside this specification:

* LinkedIn support;
* automatic job application;
* broad ATS integration;
* ATS form autofill;
* cover-letter generation;
* resume tailoring;
* job-match scoring;
* resume-job matching;
* interview preparation;
* background job crawling;
* broad AI assistant functionality.

Future capabilities may reuse the Floating Companion interaction surface without changing the Sprint 2 responsibility:

> Make application tracking accurate, quiet, and low-friction.

---

# 27. Acceptance-Level Behaviour Summary

The implementation should satisfy the following product behaviours:

* The Floating Companion persists across supported dynamic job browsing.
* The selected job determines the active OfferBuddy job context.
* Old job context is cleared before a new context becomes active.
* Normal jobs do not generate unnecessary visual notifications.
* Supported eligibility warnings produce an attention indicator.
* Warning evidence is available through the Popover.
* Opening an application workflow does not create an Application.
* Clicking Submit does not create an Application.
* Reliable SEEK submission confirmation can automatically track an Application.
* Reliable Indeed submission confirmation can automatically track an Application.
* Indeed application identity survives transition into Smart Apply.
* Repeated submission-confirmation detection does not create duplicate Applications.
* External or uncertain submission does not automatically create an Application.
* Uncertain submission can be resolved through explicit user confirmation.
* Tracking success produces lightweight feedback.
* Authentication or backend failure does not silently lose confirmed application context.
* Site-specific detection remains inside site adapters.
* Cover letters, match scoring, ATS autofill, auto-apply, and other future assistant capabilities remain outside Sprint 2.
