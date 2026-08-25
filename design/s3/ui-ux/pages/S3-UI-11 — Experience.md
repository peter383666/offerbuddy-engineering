S3-UI-11 — Experience

# S3-UI-11 — Experience

```yaml
spec_id: S3-UI-11
title: Experience
surface: web
status: frozen
figma_page: S3-02 Candidate Profile
depends_on:
  - candidate-profile
  - resume-import
s2_reuse: true
```

## 1. Purpose

`Experience` allows the user to maintain their authoritative employment/work experience history in Candidate Profile.

It answers:

> **What work experience can OfferBuddy truthfully use as evidence when analysing jobs and preparing applications?**

Experience is more than a Resume paragraph. It is structured Candidate data:

```text
Experience
├── Company
├── Role
├── Employment period
├── Current-role state
├── Description / context
└── Experience Items
    ├── responsibility / achievement / contribution
    ├── responsibility / achievement / contribution
    └── ...
```

This structure matters because Match, Tailored Resume and Focused Cover Letter need to select **specific evidence**, rather than treating the user's entire career as one block of text.

------

# 2. Scope

### In Scope

- display all Candidate experiences;
- add an Experience;
- edit an Experience;
- remove an Experience;
- edit start/end dates;
- represent current employment correctly;
- maintain Experience Items;
- add/edit/remove Experience Items;
- validate employment periods;
- save through Candidate Profile;
- use aggregate `profileVersion`;
- handle Resume Import accepted Experience records;
- handle validation / request failure / OCC conflict.

### Out of Scope

- job-specific experience rewriting;
- Resume bullet generation;
- Match scoring;
- automatically inferring achievements;
- employment verification;
- reference checking;
- LinkedIn employment synchronisation;
- multiple Candidate Profiles;
- automatically calculating or claiming exact "years of experience" as a new fact;
- automatically creating Experience from a JD;
- Resume layout/design.

------

# 3. Entry Points

Primary:

```text
Candidate Profile Overview
        ↓
Experience
        ↓
Edit
        ↓
S3-UI-11 Experience
```

Experience may also originate from Resume Import:

```text
Resume
   ↓
AI extraction
   ↓
Proposed Experience
   ↓
User Review / Edit / Accept
   ↓
Candidate Profile Experience
```

Once accepted, imported Experience behaves like any other Candidate Profile fact.

------

# 4. Preconditions

Required:

- authenticated user;
- authorised Candidate Profile;
- current `profileVersion`.

Experience can be empty.

Candidate Profile remains valid when Experience is empty, although some downstream capabilities may not have sufficient evidence to operate meaningfully.

------

# 5. Route / Surface

**Surface:** OfferBuddy Web / Candidate Profile section editor.

Experience belongs to Candidate Profile.

It does not belong to:

```text
Application
Job
Preparation
Tailored Resume
```

Those capabilities consume Experience; they do not own it.

------

# 6. Figma Reference

**Figma Page**

```
S3-02 Candidate Profile
```

Use the final frozen Experience editing states, including the detailed Experience editor that was added specifically to support employment dates.

The implementation Agent must confirm the final Node IDs from Figma before coding.

Any:

```text
ANNOTATION — ...
```

frames are specification notes only.

------

# 7. UI Structure

The Experience surface has two conceptual levels.

## A. Experience Collection

Shows the Candidate's employment records.

Example:

```text
EXPERIENCE

Senior Software Engineer
ABC Technology
Jan 2021 – Present
                         Edit

Software Engineer
XYZ Systems
Mar 2018 – Dec 2020
                         Edit

+ Add experience
```

The user should be able to understand their employment history without opening every record.

------

## B. Experience Editor

Editing one Experience exposes the structured fields required by the frozen Candidate Profile model.

Conceptually:

```text
Company
Role / Job Title

Start Date
End Date
[ ] I currently work here

Description / Context

Experience Items
  • ...
  • ...

+ Add item

Save
Cancel
```

Exact layout/copy follows frozen Figma.

------

# 8. Experience Data Structure

The existing S3 domain distinction must be preserved:

```text
candidate_experiences
        │
        └── candidate_experience_items
```

Conceptually:

```text
Experience
│
├── identity / parent fields
│
└── Experience Items
```

The Agent must not flatten this into one giant unstructured Resume text field.

Likewise, it should not create a completely independent aggregate for every bullet.

------

# 9. Experience Fields

Exact schema names follow the frozen backend contract.

Conceptually:

| Field                 | Type       | Required    | Behaviour                    |
| --------------------- | ---------- | ----------- | ---------------------------- |
| Company               | Text       | Yes         | User-entered employer        |
| Role / Job Title      | Text       | Yes         | Candidate's actual role      |
| Start Date            | Month/date | Yes         | Employment start             |
| End Date              | Month/date | Conditional | Required unless current      |
| Current Role          | Boolean    | Yes         | Controls end-date semantics  |
| Description / Context | Multiline  | Optional    | General role/company context |
| Experience Items      | Collection | Optional    | Structured evidence/content  |

Do not invent additional fields such as:

- salary;
- manager;
- employer phone;
- reason for leaving;
- employment verification;
- reference contacts;

unless present in the frozen model.

------

# 10. Employment Date Behaviour

This is a frozen requirement.

Experience must explicitly support editing employment time.

Correct model:

```text
Start Date
+
End Date
```

or:

```text
Start Date
+
Currently working here
```

### Historical Role

Example:

```text
Start: March 2019
End:   August 2022
Current: false
```

### Current Role

Example:

```text
Start: September 2022
Current: true
End: null
```

Do not persist:

```text
endDate = "Present"
```

as a fake date if the domain contract represents current employment structurally.

`Present` is presentation.

`currentRole + null endDate` is business state where defined by the frozen model.

------

# 11. Date Validation

At minimum:

### Start ≤ End

Invalid:

```text
Start: January 2024
End:   January 2022
```

### Current Role

If:

```text
current = true
```

then End Date should be disabled/cleared according to the frozen domain contract.

### Switching Current → Historical

If the user unchecks:

> I currently work here

an End Date becomes required where the contract requires one.

### Future Dates

Use frozen backend validation.

Do not invent aggressive assumptions such as:

> employment can never have a future end month

if the domain permits known notice/employment end dates.

Frontend and backend rules must agree.

------

# 12. Date Precision

The UI should preserve the precision supported by the frozen data model.

If employment history is month-based:

```text
2021-03
```

do not fabricate:

```text
2021-03-01
```

as a meaningful Candidate fact merely because a date picker prefers full dates.

Similarly, do not display artificial day precision that the user never provided.

This is especially important for Resume Import.

------

# 13. Experience Description

`Description / Context` represents reusable context about the role.

For example:

> Worked within the order/trade team responsible for order placement workflows and operational support for an e-commerce platform.

It is Candidate Profile information.

It is not automatically:

> the final Resume paragraph.

Tailored Resume may later rewrite/select this information for a specific Job.

------

# 14. Experience Items

Experience Items represent reusable evidence associated with an Experience.

Examples:

```text
Implemented order-processing APIs using Java and Spring Boot.

Integrated promotion and coupon services into order placement workflows.

Supported production issue investigation for order-related operational complaints.
```

The purpose is to give downstream capabilities **granular truthful evidence**.

Conceptually:

```text
Experience
   │
   ├── context
   │
   └── evidence items
          │
          ├── Match
          ├── Tailored Resume
          └── Focused Cover Letter
```

------

# 15. Experience Item Actions

Within an Experience, the user can maintain items according to the frozen Figma.

### Add Item

Creates local unsaved item.

### Edit Item

Corrects/rephrases existing Candidate evidence.

### Remove Item

Removes that Candidate fact/evidence from the Profile after Save.

All are part of the parent Candidate Profile mutation.

They do **not** get independent OCC semantics.

------

# 16. Experience Item Truth Boundary

Experience Items are free text, but they remain Candidate facts.

The user can write:

> Developed REST APIs for order-processing workflows.

The system must not silently turn this into:

> Architected a globally distributed order platform processing millions of transactions per day.

unless those claims are supported by accepted Candidate facts.

Downstream AI can improve wording, but cannot inflate:

- scale;
- responsibility;
- seniority;
- metrics;
- technologies;
- achievements.

------

# 17. Metrics and Achievements

Metrics deserve special care.

Suppose Resume Import extracts:

> Improved performance by 40%.

That is a proposal.

It becomes Candidate evidence only after user acceptance.

If Candidate Profile only says:

> Improved query performance.

Tailored Resume cannot invent:

> 40% improvement.

Likewise AI should not create fake numbers merely because quantified Resume bullets are considered stylistically stronger.

------

# 18. Add Experience

User explicitly adds a new employment record.

Before Save:

> local editor state.

After successful Save:

```text
new Experience
        ↓
Candidate Profile fact
        ↓
profileVersion increments
```

No AI confirmation is needed for facts explicitly entered by the user.

------

# 19. Edit Experience

Changes may include:

- company;
- role;
- dates;
- current state;
- description;
- Experience Items.

Any meaningful persisted change is a Candidate fact revision.

Therefore:

```text
profileVersion + 1
```

on successful aggregate update.

------

# 20. Remove Experience

Removing an Experience removes its associated Candidate evidence from the authoritative Profile.

Conceptually:

```text
Remove Experience
       ↓
Remove parent + its Experience Items
       ↓
Profile revision
```

UI should clearly associate deletion with the complete Experience.

Do not accidentally leave orphaned Experience Items.

Backend database/domain integrity remains authoritative.

------

# 21. Removal Consequences

Suppose:

```text
Profile v12
   ↓
Experience A
   ↓
Tailored Resume generated
mentions Experience A
```

User removes Experience A:

```text
Profile v13
```

The existing Tailored Resume does **not** get silently rewritten.

Instead:

```text
Tailored Resume
sourceProfileVersion = 12
        ↓
STALE
```

The same principle applies to Match and Focused Cover Letter.

------

# 22. Resume Import Behaviour

Resume Import must preserve Experience as a coherent record.

For example:

```text
ABC Pty Ltd
Software Engineer
Jan 2021 – Aug 2024

• Developed...
• Integrated...
```

should be reviewed conceptually as:

```text
Proposed Experience
├── Company
├── Role
├── Dates
└── Items
```

not as unrelated proposals:

```text
Accept company?
Accept role?
Accept start month?
Accept each random token?
```

The user can edit fields during review, but the record must retain semantic coherence.

------

# 23. Existing Experience vs Import Proposal

If Profile already contains:

```text
ABC Pty Ltd
Software Engineer
2021 – 2024
```

and import proposes:

```text
ABC Pty Ltd
Senior Software Engineer
2021 – 2024
```

OfferBuddy must not automatically decide:

> same employer + dates = overwrite role.

Resume Import Review handles the difference.

The user decides whether the proposal:

- updates existing Experience;
- is corrected;
- is rejected;
- represents a distinct Experience where appropriate.

------

# 24. Experience Deduplication

Preventing duplicate employment records is useful, but dangerous if over-automated.

Obvious exact duplicate:

```text
ABC
Software Engineer
Jan 2021 – Dec 2022
```

entered twice can be flagged.

But:

```text
ABC
Software Engineer
2020 – 2022

ABC
Senior Software Engineer
2022 – 2024
```

may represent a promotion.

Do not auto-merge them.

Similarly, overlapping employment periods are not inherently invalid.

Users can legitimately have:

- concurrent jobs;
- part-time work;
- contract work;
- internal promotions.

------

# 25. Ordering

Experience Overview should use the frozen Figma ordering.

Normally employment history is presented using current/recent roles first.

However:

> display ordering does not change factual dates.

Do not introduce arbitrary manual ranking unless already in the frozen domain/UI.

Tailored Resume may later reorder/emphasise content independently without changing Profile ordering.

------

# 26. Relationship to Match

Match uses Experience as evidence:

```text
Job Requirement
       +
Candidate Experience
       ↓
Evidence alignment
```

For example:

```text
Requirement:
Spring Boot APIs

Evidence:
Experience Item — built Spring Boot REST APIs

→ supported
```

But:

```text
Requirement:
AWS

No Candidate evidence

→ gap / unsupported
```

Match must not modify Experience to improve the score.

------

# 27. Relationship to Tailored Resume

Tailored Resume can:

- select relevant Experiences;
- select relevant Experience Items;
- reorder evidence;
- condense wording;
- rewrite truthful wording;
- omit irrelevant items.

It cannot:

- invent an employer;
- invent a role;
- extend dates;
- increase seniority;
- invent technology;
- invent metrics.

And Tailored Resume edits never automatically update this Experience editor.

------

# 28. Relationship to Focused Cover Letter

Focused Cover Letter can select relevant Experience evidence.

For example:

```text
Profile:
Order processing experience
Java/Spring APIs
Operational support
```

may become a job-specific paragraph.

But the resulting paragraph remains a derived artifact.

It does not become a new Experience Item automatically.

------

# 29. Actions

## Add Experience

Open new Experience editing state.

## Edit Experience

Open selected Experience.

## Remove Experience

Remove in local editing state / confirmed interaction according to Figma.

## Add/Edit/Remove Experience Item

Modify the current Experience locally.

## Save

Persist the Candidate Profile change using:

```text
expected profileVersion
```

### Success

- authoritative Profile updated;
- meaningful revision increments `profileVersion`;
- refreshed Experience state shown.

### Validation Failure

Keep local data.

### Request Failure

Keep local data.

### OCC Conflict

Explicit conflict.

Never silent merge.

## Cancel / Back

Discard unsaved changes according to existing dirty-state UX.

------

# 30. UI States

### Collection Loading

Experiences loading.

### Empty

No Experience saved.

Provide clear Add action.

### Collection Ready

Existing Experience summaries visible.

### Creating

New Experience form.

### Editing

Existing Experience form.

### Dirty

Unsaved modifications exist.

### Validation Error

Invalid fields/dates/items.

### Saving

Prevent duplicate Save.

### Save Failure

Preserve complete Experience draft, including Items.

### Conflict

Profile changed elsewhere.

### Delete / Remove Confirmation

If frozen Figma uses confirmation for destructive removal, follow it.

Do not invent inconsistent confirmation behaviour outside existing OfferBuddy conventions.

------

# 31. Business Rules

1. Experience is authoritative Candidate Profile data.
2. Experience is structured data, not merely Resume text.
3. Experience contains employment period semantics.
4. Current employment must be represented structurally.
5. `Present` is presentation, not a fabricated end date.
6. Preserve source date precision.
7. Experience can contain multiple Experience Items.
8. Experience Items belong to their parent Experience.
9. Experience Items do not have independent OCC versions.
10. Candidate Profile uses aggregate `profileVersion`.
11. Add/edit/remove Experience is a meaningful Profile revision.
12. Add/edit/remove Experience Item is a meaningful Profile revision.
13. Resume Import can propose Experience but cannot establish it automatically.
14. Job requirements cannot create Experience.
15. Match cannot create Experience.
16. Tailored Resume cannot create Profile Experience.
17. Focused Cover Letter cannot create Profile Experience.
18. AI cannot invent employers, roles, dates, responsibilities, technologies or metrics.
19. Overlapping employment dates are not automatically invalid.
20. Same employer may legitimately have multiple Experience records.
21. Promotion histories must not be auto-merged.
22. Removing Experience may make derived artifacts stale.
23. Saving does not automatically regenerate derived artifacts.
24. S2 Fast Apply remains independent.

------

# 32. API Mapping

Use the frozen Phase 3 Candidate Profile contracts.

The existing database model already distinguishes:

```text
candidate_experiences
candidate_experience_items
```

The Agent must respect the frozen API aggregate semantics.

Do not infer from UI controls that public APIs must become:

```text
POST   /api/experiences
PUT    /api/experiences/{id}
DELETE /api/experiences/{id}

POST   /api/experience-items
...
```

unless these are the actual frozen contracts.

UI CRUD interaction does not dictate aggregate API boundaries.

------

# 33. Concurrency

Candidate Profile aggregate OCC applies to Experience.

Example:

```text
Experience editor loads:

profileVersion = 25
```

Another tab changes Skills:

```text
profileVersion = 26
```

User saves Experience with expected:

```text
25
```

Result:

```text
409 Conflict
```

Do not reason:

> "Only Skills changed, so Experience can safely merge."

Frozen S3 consistency model explicitly rejects this.

------

# 34. Complex Draft Preservation

Experience is more complex than Skills, so failed writes must preserve the **whole local draft** where practical:

```text
Company
Role
Dates
Current state
Description
Experience Items
```

A network failure must not reset the user to the server's previous Experience.

This is especially important after the user has written several Experience Items.

------

# 35. OCC Conflict vs Draft Preservation

These are different requirements.

On `409`:

- preserve local draft so the user does not lose work;
- do **not** automatically apply it to the latest Profile.

Correct:

```text
Local draft
     +
Conflict warning
     +
Reload/review path
```

Incorrect:

```text
409
 ↓
fetch latest Profile
 ↓
automatically patch local Experience
 ↓
Save again
```

------

# 36. Derived Artifact Staleness

Example:

```text
Profile v30

Experience:
Software Engineer
2018 – 2024
```

Tailored Resume generated from v30.

User corrects:

```text
2019 – 2024
```

Profile becomes v31.

Even though the difference looks small:

> it is a meaningful Candidate fact change.

The old Resume may contain the wrong duration/date and must therefore be stale according to provenance semantics.

------

# 37. Navigation / Transitions

| From              | Action           | Destination / State              |
| ----------------- | ---------------- | -------------------------------- |
| Candidate Profile | Edit Experience  | Experience Collection            |
| Collection        | Add              | New Experience                   |
| Collection        | Edit             | Experience Editor                |
| Editor            | Add Item         | Dirty                            |
| Editor            | Edit Item        | Dirty                            |
| Editor            | Remove Item      | Dirty                            |
| Editor            | Save             | Saving                           |
| Saving            | Success          | Experience Collection / Overview |
| Saving            | Validation Error | Editing                          |
| Saving            | Request Failure  | Editing                          |
| Saving            | OCC Conflict     | Conflict                         |
| Editor            | Cancel           | Collection / Overview            |

------

# 38. S2 Reuse

### Reuse

- Web shell;
- Profile section patterns;
- inputs;
- multiline fields;
- date/month controls;
- buttons;
- collection controls;
- validation;
- dirty-state handling;
- error/loading patterns.

### S3 Delta

Introduce structured Candidate Experience and Experience Item management.

### Do Not

- create a Resume-specific employment database;
- couple Experience to Application;
- build employment verification;
- introduce AI achievement generation as part of CRUD;
- alter S2 Fast Apply.

------

# 39. Component Reuse

Possible responsibilities:

```text
ExperienceSection
ExperienceList
ExperienceCard
ExperienceEditor
ExperienceItemEditor
EmploymentPeriodFields
ProfileSaveActions
ProfileConflictNotice
```

Names are illustrative.

`EmploymentPeriodFields` is a particularly useful reusable responsibility because the relationship between:

```text
startDate
endDate
current
```

should not be reimplemented inconsistently in Resume Import and Profile editing.

------

# 40. Responsive / Surface Behaviour

Primary frozen target:

> Desktop Web.

At narrower widths:

```text
Start Date | End Date
```

may become:

```text
Start Date
End Date
```

Experience Items should remain readable.

Actions must stay associated with their correct Experience/Item.

No separate mobile employment editor is required.

------

# 41. Accessibility / UX Requirements

- Company and Role inputs have clear labels.
- Date controls have semantic labels.
- Current-role checkbox/toggle is keyboard accessible.
- End Date disabled state is understandable.
- Add/Edit/Remove Item controls identify their target.
- Validation errors identify the affected field.
- Remove Experience is clearly destructive.
- Save failure preserves the draft.
- Conflict preserves the draft while requiring review.
- Empty state has a clear Add action.

------

# 42. Analytics / Observability

Operational telemetry may contain:

- requestId;
- candidateId;
- profileVersion;
- operation type;
- success/failure category.

Do not log:

- employer name;
- role title;
- employment dates;
- Experience description;
- Experience Items;
- Resume extraction text.

These are Candidate facts.

------

# 43. Security / Privacy

Experience is sensitive Candidate-owned data.

Authorization must resolve through backend Candidate ownership.

Do not expose full Experience freely to:

- Browser Extension;
- RuoYi Admin;
- operational logs;
- product analytics.

AI capabilities may receive the minimum authorised Candidate evidence required by their frozen capability boundary.

------

# 44. Acceptance Criteria

-  User can view existing Experiences.
-  Empty Experience state is supported.
-  User can add an Experience.
-  User can edit an Experience.
-  User can remove an Experience.
-  Company is editable.
-  Role is editable.
-  Start Date is editable.
-  End Date is editable for historical roles.
-  Current Role is represented explicitly.
-  Current Role correctly controls End Date semantics.
-  `Present` is not persisted as a fake date where structured current-state exists.
-  Date precision is not artificially increased.
-  Invalid Start > End is rejected.
-  Overlapping separate Experiences are not automatically rejected.
-  Same-employer promotion records are not automatically merged.
-  Experience Description can be maintained where supported.
-  Experience Items can be added.
-  Experience Items can be edited.
-  Experience Items can be removed.
-  Items remain associated with the correct parent Experience.
-  Resume Import Experience requires explicit acceptance.
-  Imported Experience is reviewed as a coherent record.
-  AI cannot invent employment facts or metrics.
-  Save uses Candidate aggregate `profileVersion`.
-  No Experience/Item-specific OCC version is introduced.
-  Save failure preserves the complete draft.
-  OCC conflict preserves local work but does not auto-merge it.
-  Meaningful Experience changes increment `profileVersion`.
-  Experience changes can make derived artifacts stale.
-  Derived artifacts are not automatically regenerated.
-  S2 Fast Apply remains independent.
-  Candidate Experience ownership is enforced server-side.

------

# 45. Agent Implementation Notes

1. Read the frozen Candidate Profile Experience contracts and actual schema first.
2. Inspect all final Experience Figma states before coding.
3. Confirm current Figma Node IDs.
4. Preserve the existing `candidate_experiences → candidate_experience_items` relationship.
5. Do not flatten Experience into one Resume-text field.
6. Implement employment time explicitly.
7. Treat `current` and `endDate` consistently with the backend contract.
8. Preserve month/date precision.
9. Do not automatically reject overlapping employment.
10. Do not auto-merge same-employer records.
11. Keep Experience Items attached to their parent Experience.
12. Do not create Item-level OCC versions.
13. Use Candidate aggregate `profileVersion`.
14. Preserve the entire draft on request failure/conflict.
15. Never automatically retry a stale write against a newer Profile.
16. Treat Resume Import Experience as proposal until acceptance.
17. Do not let Job/Match/Resume/CL write back Experience.
18. Do not generate fake achievements or metrics.
19. Do not automatically regenerate derived artifacts after Save.
20. Reuse existing Web/Profile/date/form primitives.
21. Ignore Figma annotations.
22. Do not redesign the frozen UI.

```
## `S3-UI-11` 可以冻结

这一份把你之前特别问过的 **Experience 时间编辑**正式固定下来了：

​```text
Historical role
Start Date + End Date

Current role
Start Date + Current = true
           + End Date = null
```

而不是简单保存一个 `"Present"` 字符串。

另外 Experience 的粒度也固定为：

```text
Experience
│
├── Company
├── Role
├── Employment Period
├── Description / Context
│
└── Experience Items
    ├── evidence 1
    ├── evidence 2
    └── evidence 3
```

这对后面的 Match 和 Tailored Resume 很重要：AI 可以从 **真实的 granular evidence** 中选择，而不是只能拿整段 Resume 文本猜。

下一份进入 **`S3-UI-12 — Education`**，然后是 Certifications、Languages / Eligibility。