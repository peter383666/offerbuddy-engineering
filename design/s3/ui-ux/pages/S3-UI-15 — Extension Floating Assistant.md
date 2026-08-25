# S3-UI-15 — Extension Floating Assistant

```yaml
spec_id: S3-UI-15
title: Extension Floating Assistant
surface: browser-extension
status: frozen
figma_page: S3-01 Focused Apply
depends_on:
  - job-detection
  - application-status
  - sponsor-employer-snapshot
  - eligibility-signals
  - preparation
  - extension-auth
s2_reuse: true
```

## 1. Purpose

`Extension Floating Assistant` is the lightweight, context-sensitive OfferBuddy surface shown directly on supported recruitment websites.

It answers:

> **Is there anything useful OfferBuddy should tell me or help me do for this Job right now?**

The Floating Assistant is intentionally **not** a second OfferBuddy Web application.

Its responsibilities are narrow:

```text
Detect current Job
      ↓
Resolve useful state
      ↓
Show concise signal / action
      ↓
Open OfferBuddy Web when deeper work is needed
```

The most important product boundary is:

> **Extension detects, hints and hands off. Web App analyses and prepares.**

Therefore the Extension does **not** display Match Analysis, Resume review, or full Cover Letter editing.

------

## 2. Scope

### In Scope

- collapsed floating control;
- hover-expanded assistant;
- current Job recognition;
- already-applied state;
- sponsor employer signal;
- PR/citizenship/clearance or similar requirement warning;
- `Prepare with OfferBuddy`;
- preparation-ready state;
- opening the Web focused-preparation workflow;
- narrow backend reads required for current state;
- local Sponsor Employer snapshot/cache;
- safe loading/unavailable behaviour;
- platform-aware state resolution.

### Out of Scope

- Match score;
- Match explanation;
- Tailored Resume content;
- Resume editor;
- full Focused Cover Letter editor;
- Candidate Profile editor;
- full Application Detail;
- Auto Apply;
- automatic job submission;
- large Extension popup application;
- per-Job sponsor backend lookup on every navigation;
- general AI chatbot inside the Extension.

------

## 3. Supported Surface Model

The primary user interaction is the **floating mini-window embedded over the recruitment website**.

The intended behaviour is:

```text
Collapsed
   ↓
mouse hover / user interaction
   ↓
Expanded
   ↓
action
```

The user should not need to open the browser extension popup for the normal S3 workflow.

This is a product requirement, not just a visual preference.

------

## 4. Preconditions

The Extension may operate only when it has enough context to identify a supported Job page.

Potential prerequisites include:

- supported recruitment platform;
- Extension loaded and authorised;
- detectable current Job;
- current OfferBuddy user session where required.

If Job detection is uncertain:

> do not confidently attach sponsor/application/preparation state to the wrong Job.

A safe neutral state is preferable.

------

## 5. Figma Reference

**Figma Page:** `S3-01 Focused Apply`

Relevant final frames include:

- Collapsed
- Hover / Default
- Sponsor Signal
- Requirement Review
- Preparation Ready
- Already Applied

The SEEK Cover Letter states belong primarily to `S3-UI-16`, not this specification.

Design notes including:

- `ANNOTATION — Sponsor Data Sync`
- `ANNOTATION — Floating Assistant State Priority`
- `ANNOTATION — Extension Web Boundary`

are implementation documentation only and must not be rendered as UI.

------

# 6. Core Extension Boundary

The Extension may do:

```text
Job detection
Job-context hinting
Sponsor signal
Requirement warning
Open focused preparation
SEEK cover-letter assistance
Applied-state reflection
```

The Extension must not do:

```text
Match analysis
Resume generation/review
Focused Cover Letter review/edit
Candidate Profile maintenance
Complex AI workflows
```

In particular:

> **Do not show Match score in the Floating Assistant.**

The user sees Match after opening OfferBuddy Web.

This reduces complexity and keeps the Extension useful during rapid job browsing.

------

# 7. State Resolution Model

The Floating Assistant is one component with multiple contextual states.

It is not six independent mini-applications.

Conceptually:

```text
Current browser context
        +
Job identity
        +
Application state
        +
Preparation state
        +
Requirement signals
        +
Sponsor snapshot
        ↓
State Resolver
        ↓
Floating Assistant State
```

The state resolver must follow explicit priority rules.

------

# 8. State Priority

The frozen priority is conceptually:

```text
1. Already Applied
2. SEEK Cover Letter step
3. Important requirement warning
4. Preparation Ready
5. Sponsor Signal
6. Default / Prepare with OfferBuddy
```

SEEK Cover Letter step is specified in detail by `S3-UI-16`, but it may override the normal Floating Assistant presentation while the user is on that specific application step.

The implementation must avoid simultaneous contradictory states such as:

```text
Already applied
+
Prepare with OfferBuddy
```

for the same known application.

------

# 9. State — Collapsed

The collapsed state is the persistent minimal presence.

Purpose:

- indicate OfferBuddy is available;
- occupy minimal space;
- avoid interrupting job browsing.

Behaviour:

```text
Collapsed
   ↓ hover
Expanded state
```

It should not continuously show detailed text.

It should not create distracting animation or repeated backend calls simply because the cursor moves nearby.

------

# 10. State — Hover / Default

This is the normal expanded state when no higher-priority warning/state applies.

Primary intent:

> **Prepare with OfferBuddy**

This means:

> Open this Job in the OfferBuddy Web focused-preparation workflow.

It does **not** mean:

- save the Job;
- mark Applied;
- generate Match inside Extension;
- automatically create Resume;
- submit the application.

The user can also simply ignore the assistant and continue normal Fast Apply.

------

# 11. No “Save Job” Semantic

The Extension must not show:

> ```
> Save to OfferBuddy
> ```

as the normal Job action.

The S3 product model is not introducing a standalone Saved Job workflow.

For an interesting Job, the action is:

> **Prepare with OfferBuddy**

That creates/opens the Candidate + Job Preparation context according to the frozen backend workflow.

This distinction is important:

```text
Save Job
≠
Prepare for Job
```

S3 supports the latter.

------

# 12. State — Sponsor Signal

Sponsor Signal is shown when the current employer matches the locally available published Sponsor Employer dataset.

Example:

```text
Employer detected:
Company A

Local sponsor snapshot:
Company A → active match

↓
Sponsor Signal
```

The signal should be concise.

It may say, conceptually:

> Sponsorship signal found

and keep:

> ```
> Prepare with OfferBuddy
> ```

as the main actionable path.

The signal is helpful context, not a guarantee.

------

# 13. Sponsor Signal Meaning

A Sponsor Employer dataset match means:

> OfferBuddy has a known sponsorship-related employer signal.

It does **not** mean:

- this exact role offers sponsorship;
- the employer will sponsor this Candidate;
- the Candidate is eligible for sponsorship;
- a visa outcome is guaranteed.

The UI must not overstate dataset meaning.

Use wording consistent with evidence quality.

------

# 14. Sponsor Snapshot / Cache Model

Sponsor-supporting employers are rare relative to all employers.

Therefore the Extension should not repeatedly call the backend for every Job page just to ask:

> Is this employer on the sponsor list?

Frozen strategy:

```text
RuoYi Admin
   ↓
Published Sponsor Dataset vN
   ↓
Extension syncs versioned snapshot
   ↓
local Extension cache
   ↓
Job browsing performs local lookup
```

This is preferable to:

```text
Open Job 1 → backend lookup
Open Job 2 → backend lookup
Open Job 3 → backend lookup
...
```

------

# 15. Sponsor Snapshot Refresh

The Extension must support version-aware refresh.

Conceptually:

```text
Local snapshot v18

Backend published version v19
        ↓
normal sync/check mechanism
        ↓
download v19
        ↓
atomically replace local snapshot
```

The Extension should continue functioning with the last valid snapshot when a refresh temporarily fails.

Do not clear known data first and leave an empty cache during update.

------

# 16. Negative Sponsor Lookup

If an employer is **not** found locally:

> no sponsor signal is shown.

Do not perform an automatic backend fallback lookup for every missing employer.

This is deliberate.

Given the low positive rate, local negative lookup should generally be sufficient until the next published dataset update.

------

# 17. Employer Matching

Employer matching should use the frozen Sponsor Employer dataset semantics, including where applicable:

- canonical employer name;
- aliases;
- domain.

The Extension should not implement an uncontrolled AI fuzzy matcher that can confidently equate unrelated companies.

Matching must favour safety over excessive recall.

For example:

```text
ABC Technologies Pty Ltd
ABC Tech
```

may be aliases if present in the dataset.

But similar generic company names must not be assumed identical without dataset support.

------

# 18. State — Requirement Review

The Extension may identify high-impact Job requirements such as:

- Australian citizenship required;
- permanent residency required;
- unrestricted work rights;
- security clearance;
- other explicit eligibility constraints defined by Job Intelligence/detection rules.

When such a requirement is important, the expanded assistant should surface a concise warning/review state.

Example:

```text
Citizenship requirement detected

Review requirement
Prepare with OfferBuddy
```

The purpose is to prevent the user wasting time on an obviously constrained Job.

------

# 19. Candidate Eligibility Comparison

Where Candidate Eligibility is available and authorised, the system may compare:

```text
Job requirement
       +
Candidate Eligibility
       ↓
warning / attention signal
```

For example:

```text
Job:
Australian citizenship required

Candidate:
Temporary work rights

→ strong requirement warning
```

But if Candidate Eligibility is unknown:

```text
Candidate status: unknown
```

the Extension must not conclude:

> Candidate is ineligible.

Instead it can say:

> Citizenship requirement detected — review before applying.

Unknown remains unknown.

------

# 20. Extension Eligibility Data Boundary

The Extension must not freely store or download the complete Candidate Profile.

If Candidate eligibility context is needed, use the narrowest frozen contract.

The Extension does not need:

- full Experience;
- full Education;
- professional summary;
- Resume content.

Avoid converting the Extension cache into a shadow Candidate Profile database.

------

# 21. State — Preparation Ready

After the user has opened OfferBuddy Web and completed or substantially prepared the focused workflow, the Extension may show:

> **Focused preparation ready**

The CTA should remain simple, such as:

> Open OfferBuddy

or the exact frozen Figma copy.

It should not show:

```text
Match: 82%
Resume: Ready
Cover Letter: Ready
```

inside the Floating Assistant.

The detailed preparation content remains in Web App.

------

# 22. Preparation Ready Meaning

Preparation Ready means:

> There is an existing preparation context the user can reopen.

It does not mean:

> Application submitted.

The distinction is:

```text
Preparation Ready
      ↓
User still on recruitment website
      ↓
External application not necessarily submitted
```

Only actual submission/completion updates Application lifecycle.

------

# 23. State — Already Applied

If the current Job has already been submitted as an Application for the current user, the Extension must not offer another focused-apply CTA for the same known application context.

Correct state:

> Already applied

Possible action:

> View Application

or equivalent frozen behaviour.

Incorrect:

```text
Already applied
+
Prepare with OfferBuddy
+
Apply again
```

The recruitment site's own duplicate-application behaviour is part of the natural environment and OfferBuddy should respect the existing Application record.

------

# 24. Already Applied Detection

Detection must use authorised OfferBuddy Application state, not only page text.

Conceptually:

```text
Detected Job identity
      ↓
OfferBuddy authorised application lookup
      ↓
existing Application
      ↓
Already Applied state
```

The implementation must follow frozen Job/Application identity semantics.

Do not use company + title alone if a stronger Job identity is available.

------

# 25. Fast Apply Remains Default Behaviour

The user should not need to interact with OfferBuddy for every Job.

Normal browsing behaviour:

```text
Job
 ↓
Extension visible
 ↓
User ignores it
 ↓
continues SEEK / Indeed / LinkedIn application
```

This is valid and expected.

S3 must not introduce mandatory Preparation before application.

------

# 26. Prepare with OfferBuddy Action

## Trigger

User decides this particular Job deserves focused preparation.

## Behaviour

The Extension invokes the frozen narrow workflow for creating/resolving:

```text
canonical Job
+
Candidate + Job Preparation
```

and opens OfferBuddy Web.

If existing Preparation exists:

> open/reuse it.

Do not create duplicate preparations blindly.

If the Job content has meaningfully changed, the backend's frozen Job `contentVersion` / preparation staleness semantics apply.

------

# 27. Opening Web App

The handoff should preserve enough context for OfferBuddy Web to know:

- which Job;
- which Preparation;
- where the user came from where relevant.

The Extension must not put sensitive Candidate data into query parameters merely for convenience.

Use the frozen secure handoff/API pattern.

------

# 28. Return Path

After Web preparation, the user returns to the recruitment site.

Conceptually:

```text
Extension
   ↓
OfferBuddy Web
   ↓
Match
   ↓
Resume
   ↓
Cover Letter
   ↓
Return to Job
   ↓
Recruitment website
```

The Extension then reflects current Preparation/Application state.

It does not need to reproduce the Web results.

------

# 29. Application Completion

Once the user actually completes external submission:

```text
Recruitment website submission
          ↓
existing S2 / Extension application capture flow
          ↓
Application updated/created
          ↓
Extension state becomes Already Applied
```

Preparation itself does not mark the Application as Applied.

------

# 30. Platform Behaviour

The Extension supports the frozen supported recruitment platforms, including the current S3 target set such as:

- SEEK;
- Indeed;
- LinkedIn.

Platform integrations may have different DOM extraction details, but they must feed the same high-level Extension state model.

Avoid letting platform-specific code define different product semantics.

------

# 31. SEEK Special Case

SEEK has an additional Cover Letter step in Quick Apply.

While on that step:

> ```
> S3-UI-16 — SEEK Cover Letter Assistance
> ```

takes precedence over normal Floating Assistant actions.

Other current target platforms do not need this state unless their flow actually requests similar Cover Letter input.

Do not show irrelevant Cover Letter actions on Indeed/LinkedIn just for consistency.

------

# 32. Data Dependencies

### Local / Extension State

May include:

- detected platform;
- detected Job identity/context;
- sponsor snapshot version;
- sponsor employer data;
- temporary UI state.

### Backend Reads

Narrow authorised reads may include:

- Application state;
- Preparation existence/state;
- current published Sponsor dataset version;
- capability-specific requirement/eligibility result where contract-defined.

### Backend Commands

- resolve/create Preparation;
- application capture/update through existing S2 path where applicable.

### Must Not Store Locally Without Need

- full Candidate Profile;
- full Resume;
- Cover Letter body except narrow SEEK fill requirements defined in `S3-UI-16`;
- Match content.

------

# 33. Loading Behaviour

The Floating Assistant should avoid becoming unusable while one backend read is pending.

Use progressive resolution where practical.

For example:

```text
Job detected
    ↓
default safe state
    ↓
application/preparation state resolves
    ↓
state updates
```

But do not briefly show dangerous actions that become invalid milliseconds later if the current Job is already known to be applied.

Use existing state/cache where possible to minimise flicker.

------

# 34. Backend Failure Behaviour

If OfferBuddy backend cannot be reached:

- recruitment website remains fully usable;
- Extension should fail gracefully;
- do not block the external application;
- avoid misleading sponsor/application state;
- show minimal unavailable state only if useful.

The Extension is assistance, not a dependency for job-site operation.

------

# 35. Sponsor Cache Failure Behaviour

If sponsor snapshot refresh fails but a valid prior snapshot exists:

> keep using prior published snapshot.

If no valid snapshot exists:

> simply omit sponsor signals.

Do not block Preparation/Fast Apply because sponsor data is unavailable.

------

# 36. State Freshness

Different data has different freshness requirements.

### Sponsor Dataset

Versioned published snapshot.

### Application State

Should be refreshed appropriately when current Job/application events change.

### Preparation State

Should reflect newly completed Web preparation after return.

The Extension must not use one generic cache TTL strategy for every domain resource if the frozen contracts distinguish them.

------

# 37. UI Actions by State

| State              | Primary Action            | Secondary / Behaviour         |
| ------------------ | ------------------------- | ----------------------------- |
| Collapsed          | Hover / expand            | None                          |
| Default            | Prepare with OfferBuddy   | User may ignore               |
| Sponsor Signal     | Prepare with OfferBuddy   | Show sponsor context          |
| Requirement Review | Review / Prepare          | Show important requirement    |
| Preparation Ready  | Open OfferBuddy           | Continue external application |
| Already Applied    | View Application / status | No new preparation CTA        |
| SEEK CL Step       | Defined in S3-UI-16       | Overrides normal state        |

Exact button copy follows frozen Figma.

------

# 38. Business Rules

1. Extension is a lightweight assistant.
2. Extension is not a second OfferBuddy Web App.
3. Match is not displayed in Extension.
4. Resume content is not displayed/edited in Extension.
5. Focused Cover Letter review is not performed in Extension.
6. Fast Apply remains possible without Extension interaction.
7. `Prepare with OfferBuddy` starts/opens Focused Preparation.
8. `Prepare with OfferBuddy` is not Save Job.
9. S3 does not introduce standalone Saved Job workflow.
10. Preparation does not equal Application.
11. Preparation Ready does not equal Applied.
12. Already Applied must suppress duplicate preparation/application CTA where the Job is known as already applied.
13. Sponsor signal comes primarily from local versioned snapshot.
14. Missing sponsor lookup does not trigger per-Job backend lookup.
15. Sponsor match is a signal, not sponsorship guarantee.
16. Employer matching uses controlled canonical/alias semantics.
17. Requirement warning does not establish Candidate eligibility.
18. Unknown Candidate eligibility remains unknown.
19. Extension does not own Candidate Profile.
20. External recruitment site remains the application-submission surface.
21. Real submission updates Application lifecycle.
22. SEEK Cover Letter assistance is context-specific.
23. Indeed/LinkedIn should not receive SEEK-only UI without actual platform need.
24. Backend/Extension failure must not break recruitment-site usage.

------

# 39. API Mapping

Use frozen Phase 3 §3.17 Extension, Job, Preparation and S2 Application compatibility contracts.

Agent must not invent broad endpoints such as:

```text
GET /api/extension/all-user-data
GET /api/extension/full-profile
POST /api/extension/analyse-everything
```

The Extension API surface must remain narrow.

Conceptual operations include:

### Resolve Current Job

Use supported platform data to resolve/import canonical Job according to frozen contract.

### Read Current Application State

Determine whether current authorised user already has an Application for the resolved Job/context.

### Read/Resolve Preparation

Determine whether focused Preparation exists and its business state.

### Create/Open Preparation

Triggered only when user chooses `Prepare with OfferBuddy`.

### Sponsor Dataset Version / Snapshot

Use the frozen versioned dataset contract.

Do not request sponsor status individually for every current employer.

------

# 40. Security

The Browser Extension is explicitly an **untrusted client boundary**.

Therefore:

- do not trust userId/candidateId from Extension;
- do not trust Job ownership claims;
- server resolves identity from security context;
- server authorises Application/Preparation access;
- Extension gets only minimum required data.

Do not put privileged Admin APIs into Extension merely because both need Sponsor Employer data.

Published Extension snapshot is a separate narrow capability.

------

# 41. Privacy

Do not send/log entire page HTML unnecessarily.

External job content must follow the frozen external-content boundary.

Extension telemetry must not include:

- Candidate Profile;
- Resume;
- Cover Letter;
- full raw Job HTML;
- sensitive DOM form values.

SEEK Cover Letter contents are addressed separately under `S3-UI-16` and should still be handled minimally.

------

# 42. Observability

Useful diagnostics may include:

- platform;
- extension version;
- Job identifier;
- requestId;
- snapshot version;
- state-resolution result category;
- error category.

Do not log:

- raw Job description;
- Candidate eligibility values;
- application form content;
- cover-letter body.

------

# 43. Performance Requirements

Because the assistant appears during normal rapid job browsing:

- collapsed/hover interaction must feel immediate;
- sponsor lookup should be local;
- avoid unnecessary AI calls;
- avoid per-hover backend calls;
- avoid repeated requests when the Job context has not changed;
- reuse cached authorised state where appropriate.

Particularly:

> Hover itself must not trigger expensive backend/AI work.

Job navigation/context change is a more appropriate resolution trigger.

------

# 44. Local Cache Rules

The Agent should distinguish:

### Safe cache candidates

- published Sponsor Employer snapshot;
- its version metadata;
- non-sensitive current-page resolution state.

### Avoid broad persistent caching

- full Candidate Profile;
- Resume bodies;
- Match;
- full Application records;
- Eligibility history.

Use browser-extension storage according to frozen security design and minimal-data principle.

------

# 45. Job Navigation Detection

Recruitment sites often behave as SPAs.

The Extension must handle:

```text
Job A
 ↓
user clicks Job B
 ↓
URL/DOM changes without full page reload
 ↓
Assistant resolves Job B
```

It must not continue showing Job A's:

- sponsor signal;
- applied status;
- preparation status.

Platform adapters should notify the state resolver when current Job identity changes.

------

# 46. Race Condition Handling

Example:

```text
Job A selected
→ backend state request starts

User quickly selects Job B
→ second request starts

Job A response arrives late
```

Incorrect:

> UI shows Job A state while user is on Job B.

Implementation must associate asynchronous results with the Job/context that requested them.

Stale async responses should be ignored.

This is especially important during fast job-list browsing.

------

# 47. Already Applied Race

If Application submission completes in another tab/window or via extension capture:

> the Floating Assistant should refresh/transition to Already Applied when the relevant state becomes known.

Do not require full browser restart.

The exact refresh mechanism should reuse the existing S2 Extension integration where practical.

------

# 48. Navigation / Transitions

| From                | Trigger / Action        | Destination / State                           |
| ------------------- | ----------------------- | --------------------------------------------- |
| Supported Job       | Extension loads         | Collapsed                                     |
| Collapsed           | Hover                   | Resolved expanded state                       |
| Default             | Prepare with OfferBuddy | OfferBuddy Web Preparation                    |
| Sponsor Signal      | Prepare                 | Web Preparation                               |
| Requirement Review  | Prepare                 | Web Preparation                               |
| Web Preparation     | Return                  | Preparation Ready / appropriate current state |
| Preparation Ready   | Open OfferBuddy         | Existing Preparation                          |
| External submission | Application captured    | Already Applied                               |
| Already Applied     | View Application        | OfferBuddy Application                        |
| Job A               | User selects Job B      | Resolve state for Job B                       |
| SEEK CL step        | Platform detects step   | S3-UI-16 state                                |

------

# 49. S2 Reuse

### Reuse

- existing Browser Extension architecture;
- existing platform adapters;
- existing Job detection/import logic;
- existing S2 Application capture/update;
- existing authentication/session handling;
- current floating-window implementation;
- current page-change detection where available.

### S3 Delta

Add:

- Focused Preparation entry;
- Sponsor snapshot signal;
- requirement warning;
- Preparation Ready state;
- refined Already Applied behaviour;
- integration with S3 Web.

### Do Not

- rewrite the Extension from scratch;
- replace S2 Application capture;
- build a second browser popup product;
- add Match to the Extension;
- implement Auto Apply.

------

# 50. Component / Module Reuse

Implementation may conceptually contain:

```text
FloatingAssistant
StateResolver
PlatformJobContext
SponsorSnapshotStore
PreparationStateClient
ApplicationStateClient
```

Names are illustrative.

Platform-specific adapters should handle detection.

Product/business state resolution should not be duplicated separately for SEEK, Indeed and LinkedIn wherever it can be shared.

------

# 51. Accessibility / UX Requirements

- Expanded assistant must remain keyboard accessible where practical.
- Hover should not be the only possible interaction if keyboard/focus activation can be supported by the existing widget.
- State meaning must not depend only on colour.
- Sponsor/requirement signals should use concise plain language.
- CTA hierarchy must be obvious.
- Assistant must not cover critical recruitment-site controls unnecessarily.
- Collapse/expand must not cause disruptive page layout shifts.
- Already Applied must be clearly distinguishable from Preparation Ready.

------

# 52. Acceptance Criteria

-  Extension remains collapsed/minimal during normal browsing.
-  User can expand the assistant without opening the browser-extension popup.
-  Current Job context is resolved on supported platforms.
-  Job navigation updates the assistant to the new Job.
-  Late responses for a previous Job do not overwrite the current Job state.
-  Default state offers `Prepare with OfferBuddy`.
-  User can ignore OfferBuddy and continue Fast Apply.
-  No standalone `Save Job` workflow is introduced.
-  Sponsor signal uses the locally cached published dataset.
-  Missing employer in local snapshot does not trigger per-Job backend sponsor lookup.
-  Sponsor dataset supports version-aware refresh.
-  Last valid sponsor snapshot survives refresh failure.
-  Sponsor signal does not imply guaranteed sponsorship.
-  Requirement warnings can be surfaced.
-  Unknown Candidate Eligibility is not treated as ineligible.
-  Extension does not own or freely store Candidate Profile.
-  `Prepare with OfferBuddy` opens/reuses the correct Web Preparation.
-  Existing Preparation is not duplicated unnecessarily.
-  Preparation Ready state does not display Match score.
-  Preparation Ready does not mean Applied.
-  Already Applied suppresses a new preparation/application CTA.
-  Existing S2 Application capture remains the real applied-state integration.
-  External recruitment site remains the submission surface.
-  SEEK Cover Letter step can override normal assistant state.
-  Indeed/LinkedIn do not receive irrelevant SEEK-only Cover Letter actions.
-  Backend failure does not block recruitment-site usage.
-  Hover does not trigger expensive AI work.
-  Match is never rendered inside the Extension.
-  Resume review/editing is never rendered inside the Extension.
-  Full Focused Cover Letter editing is never rendered inside the Extension.
-  Server-side authorization is enforced for Application/Preparation resources.

------

# 53. Agent Implementation Notes

1. Inspect the existing S2 Extension implementation before writing new code.
2. Reuse the existing floating-window interaction; do not replace it with a popup-first design.
3. Inspect the final S3 Floating Assistant Figma states.
4. Treat the states as one state machine, not separate features.
5. Implement explicit state priority.
6. Never display Match score/result in the Extension.
7. Do not add Resume/Cover Letter editors to the Extension.
8. Keep `Prepare with OfferBuddy` as the focused-preparation handoff.
9. Do not rename the action to Save unless a future Saved Job feature is explicitly designed.
10. Reuse existing Job identity/application capture logic.
11. Prefer local Sponsor Employer snapshot lookup.
12. Do not add a backend sponsor lookup for every negative local match.
13. Implement dataset version refresh atomically.
14. Preserve the last valid snapshot on refresh failure.
15. Do not overstate Sponsor dataset confidence.
16. Keep Eligibility access narrow.
17. Preserve unknown Candidate eligibility semantics.
18. Associate async state responses with the current Job identity to avoid SPA navigation races.
19. Reuse S2 applied-state integration.
20. Do not mark Application Applied when Preparation becomes ready.
21. Keep recruitment-site submission outside OfferBuddy Web.
22. Do not trigger AI from hover.
23. Follow frozen Extension API contracts; do not request broad user data.
24. Keep local persistent storage minimal.
25. Ignore Figma annotations.
26. Do not redesign the frozen UI.

```
`S3-UI-15` 可以冻结。

这份最核心的产品状态机可以压缩成：

​```text
                   ┌─ Already Applied
                   │
                   ├─ SEEK CL Step → S3-UI-16
Current Job ───────┼─ Requirement Warning
                   │
                   ├─ Preparation Ready
                   │
                   ├─ Sponsor Signal
                   │
                   └─ Default
                         ↓
                Prepare with OfferBuddy
                         ↓
                   OfferBuddy Web
```

而且这里正式把性能设计也固定下来了：

```text
Sponsor employer?
        ↓
Extension local versioned snapshot
        ↓
found     → show signal
not found → do nothing
```

**不是每点击一个 Job 就问一次 backend。**

下一份是 **`S3-UI-16 — SEEK Cover Letter Assistance`**。它会把 `Default Cover Letter Edit + SEEK default fill + prepared Focused Cover Letter fill + prepared-over-default priority` 合并成一个完整 capability specification。