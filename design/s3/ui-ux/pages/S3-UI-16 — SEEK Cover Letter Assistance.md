S3-UI-16 — SEEK Cover Letter Assistance

# S3-UI-16 — SEEK Cover Letter Assistance

```yaml
spec_id: S3-UI-16
title: SEEK Cover Letter Assistance
surface: browser-extension + web-configuration
status: frozen
figma_page: S3-01 Focused Apply / S3-02 Candidate Profile
depends_on:
  - candidate-profile
  - default-cover-letter
  - preparation
  - focused-cover-letter
  - extension
  - seek-platform-adapter
s2_reuse: true
```

## 1. Purpose

`SEEK Cover Letter Assistance` gives the user a fast way to fill the Cover Letter field during SEEK Quick Apply without turning every application into a Focused Apply workflow.

It supports two sources:

```text
Fast Apply
→ Default Cover Letter

Focused Apply
→ Prepared Focused Cover Letter
```

The core priority is:

```text
Prepared Focused Cover Letter
        ↓ if unavailable
Default Cover Letter
```

This capability exists because SEEK sometimes asks for a Cover Letter during Quick Apply, while Indeed and LinkedIn currently do not require the same default interaction.

------

## 2. Scope

### In Scope

- configure a reusable Default Cover Letter;
- store/edit that template through OfferBuddy Web;
- detect SEEK Quick Apply Cover Letter step;
- fill Role Title and Company Name placeholders;
- fill Default Cover Letter for Fast Apply;
- use existing job-specific Focused Cover Letter when one exists;
- ensure prepared Cover Letter takes priority over default template;
- let the user continue editing in SEEK after filling;
- handle missing template / unavailable prepared content safely.

### Out of Scope

- Auto Apply;
- submitting SEEK forms automatically;
- generating a full AI Cover Letter for every Fast Apply;
- Match inside the Extension;
- Cover Letter editing inside the Floating Assistant;
- Indeed/LinkedIn Cover Letter controls when their flows do not require them;
- automatically converting Default Cover Letter into Candidate Profile facts;
- automatically updating Default Cover Letter from a Focused Cover Letter.

------

## 3. Product Model

There are two separate Cover Letter concepts.

### Default Cover Letter

Reusable Fast Apply configuration.

Example structure:

```text
Dear Hiring Manager,

I am writing to apply for the [Role Title] position at
[Company Name].

...

Kind regards,
...
```

It is deliberately generic.

### Focused Cover Letter

A derived, job-specific artifact created in Focused Preparation.

```text
Candidate Profile
      +
Job
      +
Match / Preparation context
      ↓
Focused Cover Letter
```

These must never be collapsed into one resource.

------

## 4. Entry Points

### Configure Default Cover Letter

```text
Candidate Profile
      ↓
APPLICATION TOOLS
      ↓
Default Cover Letter
      ↓
Edit
```

### Fast Apply Usage

```text
SEEK Job
   ↓
Quick Apply
   ↓
Cover Letter step
   ↓
Extension
   ↓
Fill default cover letter
```

### Focused Apply Usage

```text
Prepare with OfferBuddy
      ↓
Focused Cover Letter ready
      ↓
Return to SEEK
      ↓
Quick Apply Cover Letter step
      ↓
Fill prepared cover letter
```

------

## 5. Preconditions

For Default fill:

- Extension active on supported SEEK page;
- Cover Letter input step detected;
- current Job identity available;
- Default Cover Letter configured.

For prepared fill:

- current Job resolved;
- authorised existing Preparation;
- suitable Focused Cover Letter available for that Job.

If neither source exists:

> do not invent a Cover Letter automatically unless another frozen capability explicitly allows it.

------

## 6. Figma Reference

Relevant frozen states include:

- Candidate Profile — Default Cover Letter Edit
- Floating Assistant — SEEK Cover Letter Step
- Floating Assistant — SEEK Prepared Cover Letter

The Agent must confirm final Node IDs before implementation.

All `ANNOTATION — ...` nodes remain documentation only.

------

# 7. Default Cover Letter Configuration

The configuration surface belongs under:

```text
Candidate Profile area
      ↓
APPLICATION TOOLS
      ↓
Default Cover Letter
```

This placement is for UX convenience.

Semantically:

```text
Default Cover Letter
≠ Candidate fact
```

It must not participate in Candidate Profile factual provenance as if it were:

- Experience;
- Skills;
- Education;
- Eligibility.

------

# 8. Default Cover Letter Editor

The Web configuration surface should include:

- current template body;
- supported placeholders;
- Save;
- Cancel/Back;
- loading/saving/error states.

It is primarily a reusable text template editor.

Do not build:

- multi-template campaign management;
- rich design templates;
- per-platform template libraries;
- AI prompt builders.

unless separately designed.

------

# 9. Supported Placeholders

The core frozen use case includes at least:

```text
[Role Title]
[Company Name]
```

The actual supported placeholder syntax must be defined consistently by frontend/backend/Extension.

Do not silently add arbitrary variables such as:

```text
[Hiring Manager Name]
[Recruiter Name]
[Salary]
[Visa Type]
```

unless a trustworthy source and frozen contract exist.

------

# 10. Placeholder Resolution

Given:

```text
Role Title:
Senior Software Engineer

Company:
Atlassian
```

the Extension can convert:

```text
I am writing to apply for the [Role Title]
position at [Company Name].
```

into:

```text
I am writing to apply for the Senior Software Engineer
position at Atlassian.
```

This is deterministic substitution.

It is not an AI generation task.

------

# 11. Missing Placeholder Data

If the Extension cannot confidently resolve a value:

> do not fabricate it.

For example, if Company Name is uncertain, safer behaviours include:

- leave placeholder visible for user correction;
- avoid fill and explain missing context;
- use another frozen deterministic fallback if defined.

Do not ask AI to guess a company from ambiguous page text solely to complete the template.

------

# 12. Fast Apply Flow

The intended flow is deliberately fast:

```text
Job
 ↓
SEEK Quick Apply
 ↓
Cover Letter field
 ↓
OfferBuddy detects step
 ↓
Default Cover Letter ready
 ↓
Fill cover letter
 ↓
User reviews / continues applying
```

No Match generation is required.

No Candidate Profile preparation is required beyond whatever information already exists in the saved Default Cover Letter text.

This preserves the S2 Fast Path.

------

# 13. Focused Apply Flow

If the user already prepared a job-specific Cover Letter:

```text
Current Job
      +
Preparation
      ↓
Fresh Focused Cover Letter exists
```

then the Extension should offer:

> **Fill prepared cover letter**

instead of Default Cover Letter.

Priority:

```text
Fresh prepared Focused Cover Letter
        >
Default Cover Letter
```

------

# 14. Prepared Cover Letter Freshness

The Extension must not blindly prefer any old Focused Cover Letter merely because one exists.

The authoritative backend Preparation/Cover Letter state determines whether it is appropriate/current.

For example:

```text
Focused CL generated from Profile v12

Current Profile = v13

Artifact = stale
```

The Extension should not present stale prepared content as confidently "ready" if the frozen contract says it is stale.

Possible safe behaviour:

> Open OfferBuddy to review preparation

rather than filling stale content automatically.

The exact state follows backend artifact semantics.

------

# 15. Prepared-over-Default Rule

Correct:

```text
prepared fresh CL exists
        ↓
Fill prepared cover letter
```

Fallback:

```text
no usable prepared CL
        +
default template exists
        ↓
Fill default cover letter
```

Do not combine both texts.

Do not append the generic letter after the prepared letter.

------

# 16. Default Template Is User-Owned Configuration

The user can edit the template to reflect their preferred general application style.

The system should preserve that text.

It should not silently rewrite it based on:

- current Job;
- Match;
- Resume;
- AI output.

Job-specific tailoring belongs to Focused Cover Letter, not the Default Template.

------

# 17. Default Cover Letter and Candidate Facts

The template may naturally contain Candidate claims, such as:

> I am a software engineer with over six years of experience.

However, the template itself is still not Candidate Profile authority.

If Candidate facts change, OfferBuddy should not automatically parse the Default Cover Letter and write those claims back into Profile.

Correct direction:

```text
Candidate may manually maintain template
```

not:

```text
Default Cover Letter
→ parse
→ Candidate facts
```

------

# 18. Focused Cover Letter and Default Template

Likewise:

```text
Focused Cover Letter
```

does not automatically update:

```text
Default Cover Letter
```

A highly tailored letter for one employer must not become the generic future template by accident.

The two resources have separate lifecycle/intent.

------

# 19. SEEK Step Detection

The SEEK platform adapter is responsible for detecting when the user has reached the Cover Letter input step.

The product state should not depend on one brittle global DOM selector where avoidable.

Conceptually:

```text
SEEK adapter
    ↓
application step context
    ↓
coverLetterRequired = true
    ↓
Floating Assistant state
```

Platform-specific detection remains isolated from shared Extension product logic.

------

# 20. State Priority

When SEEK Cover Letter step is active, this capability overrides the normal Floating Assistant state.

Conceptually:

```text
Already Applied
      ↑ final applied lifecycle

While actively applying:
SEEK Cover Letter Step
      ↓
prepared CL?
   yes → Prepared Fill
   no
      ↓
default template?
   yes → Default Fill
   no  → no fill / configuration guidance
```

Sponsor/Requirement signals should not crowd the UI while the user is actively completing the Cover Letter field.

------

# 21. State — Default Cover Letter Ready

The Extension shows a concise state such as:

> Default cover letter ready

and a primary action:

> Fill cover letter

It may explain that Role/Company placeholders will be replaced.

Do not display the entire Default Cover Letter in the floating mini-window.

------

# 22. State — Prepared Cover Letter Ready

When a usable Focused Cover Letter exists:

> Prepared cover letter ready

Primary:

> Fill prepared cover letter

The UI should communicate that the job-specific letter is being used instead of the generic template.

Again:

> do not show a full editor inside the Floating Assistant.

------

# 23. Fill Behaviour

Fill action should:

1. resolve selected source;
2. resolve deterministic placeholders where applicable;
3. locate current SEEK Cover Letter input;
4. populate the field;
5. trigger required browser/React input events safely;
6. leave the user in control.

It should **not**:

- click Continue automatically;
- submit the application;
- bypass SEEK validation;
- hide the filled content from the user.

------

# 24. Existing User Content

If the SEEK Cover Letter input already contains user-entered content, the Extension must not silently destroy it.

Safe behaviours include:

- ask/confirm replacement according to frozen UI conventions;
- disable one-click fill when non-empty;
- use an explicit Replace action if designed.

The Agent should reuse existing Extension form-fill safety conventions.

Silent overwrite of meaningful user text is not acceptable.

------

# 25. User Editing After Fill

After filling, the user owns the SEEK form field and may edit it.

OfferBuddy does not need to continuously resync the field to its source.

Correct:

```text
Fill
 ↓
User edits in SEEK
 ↓
User submits their chosen final text
```

Incorrect:

```text
User edits
 ↓
Extension notices difference
 ↓
rewrites field again
```

------

# 26. Application Submission

Fill completion does not mean application completion.

```text
Cover Letter filled
      ≠
Application Applied
```

Only actual SEEK submission through the existing application capture flow updates Application state.

The Extension must not mark Applied immediately after Fill.

------

# 27. Default Cover Letter Data Dependencies

### Reads

- current Default Cover Letter resource/configuration;
- its body;
- supported placeholder/version metadata where defined.

### Writes

Web editor can update Default Cover Letter configuration.

It must not write Candidate Profile facts.

------

# 28. Prepared Cover Letter Data Dependencies

Extension needs only a narrow authorised response sufficient to fill the current Job.

It should not download:

- all Cover Letters;
- full Preparation history;
- full Candidate Profile.

Conceptually:

```text
current Job
    ↓
authorised Preparation lookup
    ↓
usable focused Cover Letter for this Job
```

------

# 29. Local Storage Boundary

Default Cover Letter body should not automatically be permanently copied into browser local storage unless the frozen security design requires it.

Prefer narrow retrieval/caching consistent with Extension architecture.

Prepared Cover Letters are even more job-specific and should not become a broad local library.

If short-lived caching is used, keep:

- minimal;
- job-scoped;
- appropriately invalidated.

------

# 30. Web Default Cover Letter Save

The editor should use the frozen Default Cover Letter/Application Tool API contract.

If no distinct API contract exists and it is represented by another frozen configuration resource, use that resource.

Do not put Default Cover Letter into Candidate Profile fact update purely because the entry is visually on the Profile page.

------

# 31. UI States — Web Configuration

Required states:

- Loading
- Empty / not configured
- Ready
- Editing / Dirty
- Saving
- Save Failure
- Saved
- Conflict if the resource uses OCC

If Default Cover Letter has independent concurrency/version semantics in the frozen contract, use those.

Do **not** automatically reuse Candidate `profileVersion` merely because the editor appears under Candidate Profile.

This is important because Default Cover Letter is not Candidate facts.

------

# 32. UI States — Extension

Required:

### Step Detected / Resolving

Determine whether prepared/default content is available.

### Prepared Ready

Job-specific content available.

### Default Ready

No prepared content; generic template available.

### No Cover Letter Configured

Offer a concise route to configure one where appropriate.

Do not force the user to leave SEEK if they prefer to type manually.

### Fill In Progress

Prevent duplicate clicks.

### Fill Success

Allow user to continue reviewing the SEEK field.

### Fill Failure

Do not lose any pre-existing field content.

Provide safe retry/manual fallback.

### Stale Prepared Artifact

Do not represent stale prepared content as fresh.

------

# 33. Business Rules

1. Default Cover Letter and Focused Cover Letter are separate resources/concepts.
2. Default Cover Letter serves Fast Apply.
3. Focused Cover Letter serves Focused Apply.
4. Fresh prepared Focused Cover Letter takes priority over Default Cover Letter for the same Job.
5. Default Cover Letter is not Candidate Profile fact data.
6. Default Cover Letter does not write Candidate facts.
7. Focused Cover Letter does not overwrite Default Cover Letter.
8. Default Cover Letter does not replace Focused Cover Letter.
9. Fast Apply does not require Match.
10. Fast Apply does not require Tailored Resume.
11. Fast Apply does not require Focused Preparation.
12. Placeholder replacement should be deterministic, not AI-based.
13. Missing placeholder values must not be invented.
14. Extension fill does not submit SEEK.
15. Extension fill does not mark Application Applied.
16. User may edit content after fill.
17. Extension must not continuously overwrite user edits.
18. Existing non-empty Cover Letter content must not be silently overwritten.
19. SEEK-specific Cover Letter UI should not appear on platforms where the step does not exist.
20. Extension remains narrow; no full Cover Letter editor.
21. Stale prepared artifact must not masquerade as fresh.

------

# 34. API Mapping

Use frozen Phase 3 contracts for:

- Default Cover Letter / user application-tool configuration;
- Preparation;
- Focused Cover Letter;
- Extension narrow access.

Agent must not invent a broad endpoint such as:

```text
GET /api/extension/cover-letters
```

returning every user Cover Letter.

Conceptually required operations are:

### Read Default Cover Letter

Retrieve current user's configured template.

### Update Default Cover Letter

Persist template configuration through its approved resource.

### Resolve Prepared Cover Letter

Given authorised current Job context, determine whether a usable Focused Cover Letter exists.

### Fill

DOM/form interaction occurs locally in Extension; it is not itself a backend business write.

------

# 35. Concurrency

### Default Cover Letter

Use whatever independent OCC/version semantics the frozen resource defines.

Do not automatically tie it to Candidate `profileVersion`.

### Focused Cover Letter

Artifact OCC and provenance/staleness remain governed by `S3-UI-05` / frozen Cover Letter contract.

The Extension should consume the server's usable/stale result rather than recomputing provenance independently.

------

# 36. Security

The Extension is untrusted.

Server-side identity/authorization remains mandatory for:

- Default Cover Letter reads;
- prepared Focused Cover Letter reads;
- Preparation resolution.

Do not accept:

```text
candidateId
userId
```

from the Extension as authorization proof.

Prepared Cover Letter retrieval must be scoped to the current authorised Job/Preparation.

------

# 37. Privacy

Cover Letter bodies are sensitive user-authored/generated content.

Do not log:

- Default Cover Letter body;
- Focused Cover Letter body;
- filled SEEK field contents.

Do not expose them in RuoYi monitoring.

AI monitoring may record operational metadata about Focused Cover Letter generation, not content.

------

# 38. Observability

Useful operational information may include:

- platform = SEEK;
- extension version;
- detected step;
- selected source category: `prepared` / `default`;
- fill success/failure;
- requestId for backend retrieval;
- current Job identifier.

Do not record body text or populated field value.

------

# 39. Performance

SEEK Cover Letter assistance should feel immediate.

Avoid invoking AI during Quick Apply for the Default flow.

Correct:

```text
saved template
+
local deterministic substitution
→ fill
```

Not:

```text
SEEK Cover Letter step
→ call AI
→ wait several seconds
→ fill
```

The latter would defeat the Fast Apply purpose.

------

# 40. Failure Behaviour

If OfferBuddy cannot retrieve prepared/default content:

> SEEK remains fully usable.

The user can manually type/paste a Cover Letter and continue.

OfferBuddy assistance must never block the external application.

------

# 41. Navigation / Transitions

| From              | Action                       | Destination / State               |
| ----------------- | ---------------------------- | --------------------------------- |
| Candidate Profile | Edit Default CL              | Default CL Editor                 |
| Default CL Editor | Save                         | Saved                             |
| SEEK Apply        | Cover Letter step detected   | Resolve source                    |
| Resolve           | Fresh prepared exists        | Prepared Ready                    |
| Resolve           | No prepared + default exists | Default Ready                     |
| Resolve           | Neither exists               | No configured assistance          |
| Prepared Ready    | Fill                         | SEEK field populated              |
| Default Ready     | Fill                         | Substituted template populated    |
| Filled            | User edits                   | Remain in SEEK                    |
| Filled            | User submits                 | Existing Application capture flow |
| Fill failure      | Retry/manual                 | Remain in SEEK                    |

------

# 42. S2 Reuse

### Reuse

- existing Extension architecture;
- existing floating assistant;
- existing SEEK platform adapter;
- existing form-fill utilities if already present;
- S2 application capture/submission tracking;
- Web text-editor/form primitives;
- existing authentication/session model.

### S3 Delta

Add:

- Default Cover Letter configuration;
- SEEK Quick Apply Cover Letter-step state;
- deterministic default-template fill;
- prepared Focused Cover Letter fill;
- source priority.

### Do Not

- rewrite SEEK integration;
- add Auto Apply;
- automatically Continue/Submit;
- invoke AI for generic Fast Apply;
- turn Extension into a full editor.

------

# 43. Component / Module Reuse

Possible responsibilities:

```text
DefaultCoverLetterEditor
CoverLetterTemplateResolver
SeekCoverLetterStepAdapter
CoverLetterFillController
PreparedCoverLetterClient
```

Names are illustrative.

The source-selection logic should be shared rather than duplicated across separate visual states:

```text
resolveSource()
→ prepared
→ default
→ none
```

------

# 44. Accessibility / UX Requirements

- Fill actions must be keyboard accessible where supported by current assistant.
- Source meaning should be explicit:
  - `Prepared cover letter`
  - `Default cover letter`
- Do not rely solely on colour.
- Fill-in-progress prevents duplicate action.
- Existing user content must be protected.
- Failure should explain that the user can continue manually.
- The assistant should not obscure the SEEK form field it is helping fill.

------

# 45. Acceptance Criteria

-  User can configure a Default Cover Letter in OfferBuddy Web.
-  Default Cover Letter is grouped under Application Tools rather than Candidate Facts.
-  Editing Default Cover Letter does not mutate Candidate Profile facts.
-  SEEK Cover Letter step can be detected.
-  Extension does not show SEEK Cover Letter assistance outside the relevant step.
-  Default Cover Letter can be filled during Fast Apply.
-  `[Role Title]` is replaced from current Job context when available.
-  `[Company Name]` is replaced from current Job context when available.
-  Missing placeholder data is not fabricated.
-  Default fill does not invoke AI.
-  Existing fresh Focused Cover Letter is detected for the same Job.
-  Fresh prepared Cover Letter takes priority over Default Cover Letter.
-  Prepared and Default letters are never concatenated.
-  Stale prepared Cover Letter is not represented as fresh.
-  User can manually edit the SEEK field after fill.
-  Extension does not continuously overwrite user edits.
-  Existing non-empty user content is not silently destroyed.
-  Fill does not click Continue.
-  Fill does not submit the application.
-  Fill does not mark Application as Applied.
-  Actual submission continues through existing S2 Application capture.
-  Backend failure does not block SEEK usage.
-  Cover Letter bodies are not logged.
-  Extension receives only narrow authorised Cover Letter data.
-  Indeed/LinkedIn do not receive irrelevant SEEK-only states.
-  No full Cover Letter editor is introduced into the Extension.

------

# 46. Agent Implementation Notes

1. Inspect existing SEEK Extension code before adding new behaviour.
2. Inspect the frozen Default Cover Letter and SEEK Extension Figma states.
3. Confirm current Node IDs.
4. Keep Default Cover Letter outside Candidate factual aggregate semantics.
5. Do not use Candidate `profileVersion` unless the frozen Default Cover Letter resource explicitly does.
6. Keep Default and Focused Cover Letters separate.
7. Implement source priority as `fresh prepared > default > none`.
8. Do not use stale prepared content as if current.
9. Use deterministic placeholder substitution.
10. Do not invoke AI for default Fast Apply fill.
11. Do not invent missing company/role values.
12. Protect existing SEEK field content from silent overwrite.
13. Do not auto-submit or auto-continue.
14. Do not mark Application Applied after Fill.
15. Reuse S2 application capture.
16. Keep body contents out of logs/monitoring.
17. Keep Extension APIs narrow.
18. Do not build a full Extension Cover Letter editor.
19. Keep SEEK-only behaviour inside the SEEK platform adapter/integration boundary.
20. Ignore Figma annotations.
21. Do not redesign frozen UI.

```
`S3-UI-16` 可以冻结。

这样 Extension/Fast Apply 的关系就完整了：

​```text
                    Normal Job
                        │
           ┌────────────┴────────────┐
           │                         │
       Fast Apply                Focused Apply
           │                         │
           │                  Prepare with OfferBuddy
           │                         │
           │                     Web App
           │                 Match → Resume → CL
           │                         │
           └───────────┬─────────────┘
                       ▼
               SEEK Cover Letter Step
                       │
             prepared CL exists?
                  │           │
                 yes          no
                  │           │
          Fill prepared    Fill default
                  └─────┬─────┘
                        ▼
                  User continues
                        ▼
                External submission
```

这里有一个很重要的性能冻结：**Fast Apply Cover Letter 是模板替换，不走 AI。** 只有用户主动进入 Focused Apply 时，才承担 job-specific AI preparation 的成本和等待。

下一份进入 **`S3-UI-17 — Application Detail Integration`**。这份会非常强调一个原则：**S2 Application Detail 是 baseline，S3 只增加 Focused Preparation 的入口，不重写已有页面。**