# S3-UI-12 — Education

```yaml
spec_id: S3-UI-12
title: Education
surface: web
status: frozen
figma_page: S3-02 Candidate Profile
depends_on:
  - candidate-profile
  - resume-import
s2_reuse: true
```

## 1. Purpose

`Education` lets the user maintain the authoritative education history used by OfferBuddy when evaluating and preparing applications.

It answers:

> **What qualifications and study history can OfferBuddy truthfully use for me?**

Education is reusable Candidate Profile data. It is not tailored per Job.

```text
Candidate Profile Education
          │
          ├── Match
          ├── Tailored Resume
          └── Focused Cover Letter
```

## 2. Scope

### In Scope

- view saved Education records;
- add Education;
- edit Education;
- remove Education;
- maintain qualification/institution/study period fields defined by the frozen model;
- support current/in-progress study where the model allows it;
- validate dates;
- save through Candidate Profile;
- use aggregate `profileVersion`;
- handle Resume Import proposals;
- handle save failure and OCC conflict.

### Out of Scope

- credential verification;
- transcript management;
- GPA conversion;
- qualification equivalency assessment;
- automatically inferring Australian qualification levels;
- automatically inferring work rights from Australian study;
- Job-specific education rewriting;
- multiple Candidate Profiles;
- education recommendations.

## 3. Entry Points

Primary path:

```text
Candidate Profile Overview
        ↓
Education
        ↓
Edit
        ↓
S3-UI-12 Education
```

Education may also enter through Resume Import:

```text
Resume
  ↓
Proposed Education
  ↓
Review / Edit / Accept
  ↓
Candidate Profile Education
```

## 4. Preconditions

Required:

- authenticated user;
- authorised Candidate Profile;
- current `profileVersion`.

Education can be empty.

An empty Education section does not invalidate Candidate Profile globally.

## 5. Route / Surface

**Surface:** OfferBuddy Web / Candidate Profile section editor.

Education belongs to Candidate Profile, not:

- Application;
- Preparation;
- Resume;
- Extension;
- Admin.

## 6. Figma Reference

**Figma Page:** `S3-02 Candidate Profile`

Use the final frozen `Education Edit` frame.

Agent must confirm the current final Node ID from Figma before implementation.

`ANNOTATION — ...` nodes are documentation only.

## 7. UI Structure

The surface has two conceptual levels.

### Education Collection

Show saved education records in a concise form, for example:

```text
Master of Information Technology
University of Wollongong
2024 – 2026
                           Edit

Bachelor of Business Administration
...
                           Edit

+ Add education
```

Exact content and arrangement follow Figma.

### Education Editor

The editor contains only the fields defined by the frozen Candidate Profile contract.

Conceptually:

```text
Institution
Qualification / Degree
Field of Study
Start Date
End Date
Current / In Progress if supported
Additional supported metadata

Save
Cancel
```

Do not add common résumé fields unless the frozen model actually contains them.

## 8. Data Structure

Education is a Candidate child collection under the single Candidate Profile aggregate.

Conceptually:

```text
Candidate Profile
      ↓
Education
 ├── record 1
 ├── record 2
 └── record 3
```

It does not receive its own aggregate-level concurrency model.

## 9. Field Specification

Exact names follow frozen schema/Figma.

Typical contract:

| Field                  | Type                              | Required         | Behaviour                                  |
| ---------------------- | --------------------------------- | ---------------- | ------------------------------------------ |
| Institution            | Text                              | Yes              | User-entered institution                   |
| Qualification / Degree | Text                              | Yes              | Actual awarded/pursued qualification       |
| Field of Study         | Text                              | Contract-defined | Do not infer                               |
| Start Date             | Month/year or supported precision | Contract-defined | Preserve entered precision                 |
| End Date               | Month/year or supported precision | Conditional      | Required for completed study where defined |
| Current / In Progress  | Boolean                           | If supported     | Controls end-date semantics                |

Do not invent:

- GPA;
- WAM;
- honours;
- accreditation;
- ranking;
- degree equivalency;

unless explicitly supported.

## 10. Qualification Truth Boundary

Education wording must reflect actual user-confirmed facts.

For example:

```text
Master of Information Technology
University of Wollongong
```

must not become:

```text
Master of Computer Science
```

because a Job prefers Computer Science.

Likewise:

```text
Business Administration
```

must not be automatically renamed to:

```text
Information Systems Management
```

to improve Match.

Job requirements may affect emphasis, not facts.

## 11. Study Dates

Date rules mirror the Profile's general truthfulness requirements.

Historical/completed study:

```text
Start: Feb 2024
End:   Jan 2026
```

Current study, if supported:

```text
Start: Feb 2025
Current: true
End: null
```

`Present` is display semantics, not necessarily a persisted fake date.

Preserve the actual precision the user provides.

Do not invent day-level precision from month/year input.

## 12. Completion Semantics

If the model distinguishes completed vs ongoing study, use the frozen domain field.

Do not infer completion merely because an end date is in the past if the backend contract stores explicit status differently.

Similarly, do not claim:

> degree completed

if Candidate Profile only records study attendance without confirmed completion.

## 13. Australian Study Boundary

This is especially important.

Education such as:

```text
University in Australia
```

does **not** itself establish:

- citizenship;
- permanent residency;
- unrestricted work rights;
- visa type;
- sponsorship need.

Correct relationship:

```text
Education
≠
Eligibility
```

Eligibility remains a separate Candidate Profile section.

## 14. Resume Import

Resume Import may propose an Education record.

Correct:

```text
Resume
  ↓
AI extracts qualification
  ↓
Proposed Education
  ↓
User reviews institution / degree / dates
  ↓
Accept
  ↓
Candidate Profile
```

Incorrect:

```text
AI sees university name
→ Profile updated automatically
```

If Resume Import misses or misreads degree details, the user edits the proposal before acceptance.

## 15. Duplicate Handling

Obvious duplicate records may be warned about, but do not over-merge.

Possible duplicate:

```text
University A
Master of IT
2024–2026
```

entered twice.

But these may be distinct:

```text
University A
Graduate Certificate
2023

University A
Master of IT
2024–2026
```

Do not merge simply because institution names match.

## 16. Relationship to Match

Match may evaluate requirements such as:

```text
Bachelor's degree in Computer Science or related field
```

against Candidate Education.

It may conclude:

- supported;
- related qualification;
- unclear;
- gap.

It must not modify Education to increase the score.

## 17. Relationship to Tailored Resume

Tailored Resume may:

- include relevant Education;
- omit less relevant Education;
- change ordering;
- shorten formatting.

It cannot alter qualification facts.

Example:

```text
Candidate Profile:
Master of Information Technology
```

Tailored Resume may abbreviate:

```text
Master of IT
```

only if this is a faithful presentation, not a semantic qualification change.

## 18. Relationship to Focused Cover Letter

Cover Letter may mention relevant education, for example:

> I recently completed postgraduate study in information technology in Australia.

Only if Candidate Profile supports that fact.

The resulting letter does not write back into Education.

## 19. Actions

### Add Education

Creates a local draft record.

### Edit Education

Changes the selected Education record locally.

### Remove Education

Removes the record from the pending Profile update.

### Save

Persist through Candidate Profile using:

```text
expected profileVersion
```

#### Success

- authoritative Profile updated;
- meaningful fact revision increments `profileVersion`;
- refreshed Education displayed.

#### Validation Failure

Preserve local values.

#### Request Failure

Preserve local values.

#### Conflict

Explicit 409/stale-write handling.

No silent merge.

### Cancel

Discard unsaved changes.

## 20. UI States

- Loading
- Empty
- Ready
- Creating
- Editing
- Dirty
- Validation Error
- Saving
- Save Failure
- Conflict

If destructive remove confirmation is used in existing OfferBuddy patterns/Figma, reuse it consistently.

## 21. Business Rules

1. Education is authoritative Candidate Profile data.
2. Education is not Job-specific.
3. Education is not Resume-specific.
4. Resume Import only proposes Education.
5. Explicit acceptance establishes imported Education facts.
6. Job requirements cannot create or alter Education facts.
7. Match cannot write Education.
8. Tailored Resume cannot write Education.
9. Focused Cover Letter cannot write Education.
10. Qualification names must not be upgraded/reinterpreted for matching purposes.
11. Australian study does not imply work rights.
12. Education date precision must be preserved.
13. Add/edit/remove is a meaningful Profile revision.
14. Candidate Profile aggregate uses `profileVersion`.
15. Education records do not receive separate OCC versions.
16. Removing/changing Education may make derived artifacts stale.
17. Saving does not automatically regenerate those artifacts.
18. Empty Education is allowed at Profile level.
19. S2 Fast Apply remains independent.

## 22. API Mapping

Use frozen Candidate Profile contracts.

Do not infer independent public APIs purely from collection UI controls.

For example, do not automatically introduce:

```text
POST /api/education
PUT /api/education/{id}
DELETE /api/education/{id}
```

unless frozen §3.17 already defines that contract.

UI CRUD semantics can still be implemented through Candidate aggregate update semantics.

## 23. Concurrency

Candidate aggregate OCC applies.

Example:

```text
Education editor loads Profile v17

Another tab edits Skills
→ Profile v18

Education Save using v17
→ 409
```

No automatic merge simply because different child sections changed.

## 24. Collection Conflict

Do not auto-merge Education lists after conflict.

For example:

```text
Server:
Bachelor
Master

Local stale draft:
Bachelor
Certification-like Education X
```

Frontend must not assume union is correct.

Require explicit reload/review.

## 25. Derived Artifact Staleness

Example:

```text
Profile v20
Education:
Master of Information Technology
```

Focused Cover Letter generated from v20.

User corrects qualification details:

```text
Profile v21
```

Existing Cover Letter becomes stale according to provenance rules.

Education editor does not rewrite it directly.

## 26. Navigation / Transitions

| From              | Action         | Destination / State   |
| ----------------- | -------------- | --------------------- |
| Candidate Profile | Edit Education | Education Editor      |
| Empty/Ready       | Add            | New Education         |
| Ready             | Edit           | Education Editor      |
| Editor            | Save           | Saving                |
| Saving            | Success        | Overview / Collection |
| Saving            | Failure        | Editing               |
| Saving            | OCC Conflict   | Conflict              |
| Editor            | Cancel         | Overview / Collection |

## 27. S2 Reuse

Reuse:

- Web shell;
- Candidate Profile section patterns;
- text inputs;
- month/year controls;
- buttons;
- validation;
- collection handling;
- dirty state;
- loading/error feedback.

S3 delta:

> authoritative Candidate Education management.

Do not:

- build credential verification;
- introduce education taxonomy;
- couple Education to Applications;
- change S2 Fast Apply.

## 28. Component Reuse

Possible responsibilities:

```text
EducationSection
EducationList
EducationCard
EducationEditor
StudyPeriodFields
ProfileSaveActions
ProfileConflictNotice
```

Names are illustrative.

Where employment and education share date semantics, reuse date primitives where appropriate without forcing incompatible business rules into one component.

## 29. Responsive / Surface Behaviour

Desktop Web is the frozen target.

At narrower widths:

- form columns may stack;
- dates remain readable;
- Edit/Remove remain associated with the correct Education record.

No separate mobile workflow is required.

## 30. Accessibility / UX Requirements

- Institution and qualification inputs have labels.
- Date controls have semantic labels.
- Current/in-progress state is keyboard accessible if present.
- Remove actions identify the target Education record.
- Validation errors map to fields.
- Save failure preserves the draft.
- Conflict preserves draft but does not auto-apply it.
- Empty state offers a clear Add action.

## 31. Analytics / Observability

Operational telemetry may include:

- requestId;
- candidateId;
- profileVersion;
- operation success/failure.

Do not log:

- institution;
- qualification;
- education dates;
- field of study;
- Resume Import extraction content.

## 32. Security / Privacy

Education is Candidate-owned Profile data.

Server-side Candidate ownership is mandatory.

Do not expose full Education history freely to:

- Extension;
- RuoYi Admin;
- operational logs;
- analytics.

Downstream AI receives only data permitted by its frozen capability boundary.

## 33. Acceptance Criteria

-  User can view existing Education records.
-  Empty Education state is supported.
-  User can add Education.
-  User can edit Education.
-  User can remove Education.
-  Institution is maintained accurately.
-  Qualification is maintained accurately.
-  Study dates follow frozen precision.
-  Current/in-progress study works where supported.
-  End-date semantics are consistent with current status.
-  Qualification facts are not rewritten to improve Job Match.
-  Australian study does not change Eligibility.
-  Resume Import Education requires explicit acceptance.
-  Duplicate detection does not over-merge distinct qualifications.
-  Job requirements cannot write Education.
-  Match cannot write Education.
-  Tailored Resume cannot write Education.
-  Cover Letter cannot write Education.
-  Save uses aggregate `profileVersion`.
-  No Education-specific OCC version is introduced.
-  Request failure preserves draft.
-  OCC conflict is explicit and not silently merged.
-  Meaningful change increments `profileVersion`.
-  Education changes can make derived artifacts stale.
-  Derived artifacts are not automatically regenerated.
-  S2 Fast Apply remains independent.

## 34. Agent Implementation Notes

1. Read frozen Candidate Education schema/API before coding.
2. Inspect the final Education Figma frame and confirm Node ID.
3. Reuse existing Candidate Profile collection/form patterns.
4. Preserve qualification wording and date precision.
5. Do not infer qualification equivalency.
6. Do not infer work rights from Australian study.
7. Treat Resume Import as proposal until acceptance.
8. Do not over-merge same-institution records.
9. Use Candidate aggregate `profileVersion`.
10. Do not create Education-level OCC versions.
11. Preserve local draft on failure/conflict.
12. Do not silently merge a stale collection write.
13. Do not let Job/Match/Resume/Cover Letter write back Education.
14. Do not automatically regenerate downstream artifacts.
15. Ignore Figma annotations.
16. Do not redesign the frozen UI.

```
`S3-UI-12` 可以冻结。

下一份是 **`S3-UI-13 — Certifications`**。这份相对短一些，但要特别固定：证书名称/issuer/date 必须是真实事实；**Job 要求某个 certification 不能让 AI 自动把它加到 Profile**，而且 certification expiry 如果 schema 支持，需要和“是否仍有效”区分清楚。
```