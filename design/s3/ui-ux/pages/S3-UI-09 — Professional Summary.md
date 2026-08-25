# S3-UI-09 — Professional Summary

```yaml
spec_id: S3-UI-09
title: Professional Summary
surface: web
status: frozen
figma_page: S3-02 Candidate Profile
depends_on:
  - candidate-profile
s2_reuse: true
```

## 1. Purpose

`Professional Summary` allows the user to maintain the reusable professional summary stored in the authoritative Candidate Profile.

It answers:

> **How do I describe my professional background at a profile level, independently of any specific job?**

This is the Candidate's **general professional summary**, not a job-specific Resume summary.

The distinction is:

```text
Candidate Profile
Professional Summary
        │
        │ reusable truthful representation
        ▼
Focused Preparation
        │
        ├── Match
        │
        ├── Tailored Resume
        │     └── may rewrite/emphasise for this Job
        │
        └── Focused Cover Letter
```

The Candidate Profile version remains the reusable factual baseline.

------

## 2. Scope

### In Scope

- display current Professional Summary;
- edit the summary;
- validate content;
- save through Candidate Profile;
- handle `profileVersion`;
- handle save/conflict states;
- return to Candidate Profile Overview.

### Out of Scope

- generating a job-specific Resume summary;
- tailoring the summary to the currently viewed Job;
- Match Analysis;
- Cover Letter writing;
- modifying Skills or Experience;
- automatically adding inferred Candidate facts;
- multiple summaries for different jobs;
- Resume template management;
- AI-generated factual enrichment without explicit user confirmation.

------

## 3. Entry Points

Primary path:

```text
Candidate Profile Overview
        ↓
Professional Summary
        ↓
Edit
        ↓
S3-UI-09 Professional Summary
```

The user may also be directed here if a downstream capability identifies insufficient Candidate Profile information.

However, the editor remains part of Candidate Profile—not part of that Job's Preparation.

------

## 4. Preconditions

Required:

- authenticated user;
- authorised Candidate Profile;
- current `profileVersion`.

Professional Summary may be empty.

An empty summary does not invalidate the entire Candidate Profile.

Capability-specific requirements determine whether a downstream feature needs it.

------

## 5. Route / Surface

**Surface:** OfferBuddy Web / Candidate Profile section editor.

Reuse existing Candidate Profile navigation and Web shell.

Do not create a separate:

> AI Profile Writer

product or route for this capability.

------

## 6. Figma Reference

**Figma Page**

```
S3-02 Candidate Profile
```

**Implementation frame**

Final `Professional Summary Edit` frame.

Agent must confirm the current frozen Figma Node ID before implementation.

`ANNOTATION — ...` nodes are design documentation only.

------

# 7. UI Structure

### A. Existing Web Shell

Reuse:

- navigation;
- header;
- page container;
- typography;
- buttons;
- form components;
- feedback patterns.

### B. Section Context

Clearly identify:

> Candidate Profile → Professional Summary

### C. Summary Editor

A multiline text-editing surface containing the current Profile summary.

The editor is for maintaining the Candidate's general professional representation.

It should not encourage the user to enter:

- a specific company;
- a specific Job title being applied for;
- job-specific keyword stuffing.

### D. Supporting Guidance

If the frozen Figma contains guidance, it should help the user understand what belongs here.

Conceptually:

> Summarise your professional background, experience and strengths. Keep this general; OfferBuddy can tailor it for individual jobs later.

Do not add excessive AI-writing UI not represented by the frozen design.

### E. Actions

Primary:

> Save

Secondary:

> Cancel / Back

Exact labels follow Figma.

------

# 8. Data Dependencies

### Reads

Candidate Profile:

- candidateId;
- profileVersion;
- professionalSummary.

### Writes

Only:

> Candidate Profile professional summary.

Must not modify:

- Skills;
- Experience;
- Education;
- Base Resume;
- Tailored Resume;
- Cover Letter;
- Application.

------

# 9. Field Specification

| Field                | Type           | Required                 | Editable | Behaviour                                                    |
| -------------------- | -------------- | ------------------------ | -------- | ------------------------------------------------------------ |
| Professional Summary | Multiline text | Capability/model-defined | Yes      | Trim meaningless surrounding whitespace; respect frozen length constraints |

The implementation must use the actual frozen API/database validation limits rather than inventing arbitrary frontend limits.

If a character limit is shown in Figma, frontend and backend rules must agree.

------

# 10. Truth Boundary

Professional Summary is natural-language content, but that does **not** make it exempt from Candidate Profile truth rules.

For example, suppose Profile contains:

```text
Experience:
Software Engineer — 6 years

Skills:
Java
Spring Boot
MySQL
Redis
```

The summary can truthfully express:

```text
Software engineer with over six years of experience
building backend and web systems using Java, Spring Boot,
MySQL and Redis.
```

It cannot silently become:

```text
Principal cloud architect with ten years of AWS
microservices leadership experience.
```

unless those facts are supported by accepted Candidate Profile facts.

The rule is:

> **Natural-language rewriting may improve expression; it may not create Candidate facts.**

------

# 11. Relationship to Other Candidate Facts

Professional Summary is part of Candidate Profile, but it should remain consistent with the rest of the aggregate.

Conceptually:

```text
Skills ──────────┐
Experience ──────┤
Education ───────┤
Certifications ──┤
                 ▼
       Professional Summary
```

This does **not** mean the system automatically regenerates Professional Summary whenever another section changes.

The user remains responsible for the saved Profile summary.

------

# 12. AI Assistance Boundary

S3 does not require a separate AI workflow for this editor unless explicitly present in the frozen UI/API.

If AI assistance is later used to suggest wording:

```text
Candidate Facts
      ↓
AI suggestion
      ↓
User Review
      ↓
Explicit Save
      ↓
Professional Summary
```

Not:

```text
Candidate Facts
      ↓
AI
      ↓
Profile silently changed
```

Any AI-generated text is a proposal until the user accepts/saves it.

Agent must not add an AI button merely because this is a text field.

------

# 13. Actions

## Save

### Trigger

User submits a changed Professional Summary.

### Behaviour

Update Candidate Profile using current expected:

```text
profileVersion
```

### Success

- summary saved;
- meaningful Profile revision increments `profileVersion`;
- dirty state cleared;
- Overview reflects authoritative saved value.

### Validation Failure

Remain on editor.

Preserve content.

### Request Failure

Preserve content.

### Conflict

Explicitly handle stale `profileVersion`.

Do not overwrite newer Candidate Profile data.

------

## Cancel / Back

Return to Candidate Profile without saving.

If content is dirty, use existing OfferBuddy unsaved-change behaviour.

Do not silently save.

------

# 14. UI States

### Loading

Current Profile summary is loading.

### Empty

No summary exists yet.

Show an editable empty state—not an error.

### Ready

Current summary displayed.

### Editing / Dirty

User has changed content.

### Validation Error

Input violates frozen validation rules.

### Saving

Disable duplicate Save.

### Saved

Update successful.

### Save Failure

Keep local text.

### Conflict

Candidate Profile changed after this editor loaded.

Require review/reload rather than silent merge.

------

# 15. Business Rules

1. Professional Summary is part of Candidate Profile.
2. Candidate Profile remains the authoritative Candidate fact source.
3. Professional Summary is general—not Job-specific.
4. There is one Profile summary, not one summary per Job.
5. Tailored Resume may create a Job-specific summary derived from Profile facts without modifying this field.
6. Focused Cover Letter may reuse relevant facts without modifying this field.
7. Professional Summary cannot establish unsupported Candidate facts merely because it is free text.
8. AI suggestions, if used, require explicit user acceptance.
9. Saving uses aggregate-level `profileVersion`.
10. There is no `professionalSummaryVersion`.
11. A meaningful Summary change increments Candidate `profileVersion`.
12. Existing derived artifacts may consequently become stale.
13. Save does not automatically regenerate Match, Resume or Cover Letter.
14. Empty Professional Summary is allowed unless a specific capability requires it.
15. S2 Fast Apply does not depend globally on Professional Summary.

------

# 16. Relationship to Tailored Resume

This distinction must be preserved by implementation:

### Candidate Profile Summary

Example:

```text
Software engineer with six years of experience delivering
backend, web and data-related systems...
```

Reusable Candidate representation.

### Tailored Resume Summary

For a Java backend Job, OfferBuddy might derive:

```text
Backend software engineer with six years of experience
building Java/Spring-based APIs and distributed business systems...
```

This is:

> Tailored Resume content.

It does **not** overwrite:

> Candidate Profile → Professional Summary.

Correct direction:

```text
Profile Summary
       +
Candidate Facts
       +
Job
       ↓
Tailored Resume Summary
```

Never automatically reverse it.

------

# 17. API Mapping

Use frozen Phase 3 Candidate Profile contracts.

Conceptually:

### Read Candidate Profile

Retrieve:

- current professional summary;
- current `profileVersion`.

### Update Candidate Profile

Save the changed summary using expected aggregate version.

Agent must not invent:

```text
POST /api/ai/write-summary
```

or:

```text
PUT /api/candidate/summary
```

unless already defined by the frozen contracts.

------

# 18. Concurrency / Staleness

Candidate aggregate OCC applies.

Example:

```text
Professional Summary editor
loads Profile v20

Another tab changes Experience
→ Profile v21

Summary Save with expected v20
→ 409 Conflict
```

Even though the other user action changed Experience:

> do not silently merge.

This protects the semantic Candidate aggregate and prevents stale editors from unknowingly writing against an older Profile state.

------

# 19. Derived Artifact Impact

Saving the Professional Summary:

```text
Profile v20
   ↓
Summary update
   ↓
Profile v21
```

Existing:

```text
Match(sourceProfileVersion=20)
TailoredResume(sourceProfileVersion=20)
FocusedCoverLetter(sourceProfileVersion=20)
```

may now be stale.

The editor does not regenerate them.

Their own surfaces handle staleness.

------

# 20. Navigation / Transitions

| From              | Action             | Destination / State        |
| ----------------- | ------------------ | -------------------------- |
| Candidate Profile | Edit Summary       | Summary Editor             |
| Empty Summary     | Enter text         | Dirty                      |
| Editor            | Save               | Saving                     |
| Saving            | Success            | Overview / Saved           |
| Saving            | Validation failure | Editing                    |
| Saving            | Request failure    | Editing                    |
| Saving            | OCC conflict       | Conflict                   |
| Editor            | Cancel             | Candidate Profile Overview |

------

# 21. S2 Reuse

### Reuse

- Web shell;
- multiline inputs/editor;
- form validation;
- buttons;
- dirty-state handling;
- loading/error patterns.

### S3 Delta

Add authoritative Candidate Profile Professional Summary editing.

### Do Not

- build a separate Resume-summary editor here;
- introduce Job context;
- introduce Match;
- create a new AI writing workflow without a frozen requirement;
- modify S2 Fast Apply.

------

# 22. Component Reuse

Prefer existing primitives.

Possible responsibilities:

```text
CandidateProfileSectionEditor
ProfessionalSummaryForm
ProfileSaveActions
ProfileConflictNotice
```

Names are illustrative.

If Tailored Resume / Cover Letter uses a reusable text-editor primitive, the primitive can be shared.

Business containers and save semantics must remain distinct.

------

# 23. Responsive / Surface Behaviour

Primary frozen design:

> Desktop Web.

At smaller widths:

- editor uses available width;
- guidance remains readable;
- actions remain visible.

No separate mobile Profile-writing experience is required.

------

# 24. Accessibility / UX Requirements

- Multiline editor must have a clear label.
- Keyboard editing must work normally.
- Validation messages must be associated with the field.
- Dirty state must not depend solely on colour.
- Saving must prevent duplicate submission.
- Save failure must preserve text.
- Cancel must not silently save.
- Empty state must not look like an error.

------

# 25. Analytics / Observability

No new product analytics are required.

Operational telemetry may include:

- requestId;
- candidateId;
- profileVersion;
- success/failure metadata.

Must not log:

- Professional Summary text;
- Candidate Profile content;
- AI-generated suggestion text if AI assistance exists.

------

# 26. Security / Privacy

Professional Summary is Candidate-owned Profile data.

Authorization must be server-side through Candidate ownership.

Do not expose the summary through:

- Extension unless a narrow approved capability requires derived data;
- RuoYi Admin;
- logs;
- general analytics.

Client-supplied Candidate identifiers do not establish authorization.

------

# 27. Acceptance Criteria

-  User can open Professional Summary from Candidate Profile.
-  Existing summary is displayed.
-  Empty summary is supported.
-  User can edit the summary.
-  Summary remains Profile-level rather than Job-specific.
-  User can save valid changes.
-  Save uses Candidate aggregate `profileVersion`.
-  Meaningful change increments `profileVersion`.
-  No separate Summary OCC version is introduced.
-  Validation failure preserves user input.
-  Request failure preserves user input.
-  Stale Profile write produces explicit conflict.
-  Conflict is not silently merged.
-  Cancel does not save changes.
-  Summary cannot silently introduce unsupported Candidate facts.
-  AI assistance, if present, cannot silently update the Profile.
-  Tailored Resume summary does not overwrite Profile summary.
-  Save does not automatically regenerate downstream artifacts.
-  S2 Fast Apply remains independent.

------

# 28. Agent Implementation Notes

1. Read the frozen Candidate Profile API/data contract first.
2. Inspect the final Professional Summary Figma frame and confirm its Node ID.
3. Reuse existing multiline form/editor primitives.
4. Treat the summary as Candidate Profile data.
5. Do not make it Job-specific.
6. Do not create multiple Profile summaries for different Jobs.
7. Do not treat free text as permission to invent Candidate facts.
8. Do not add AI generation unless explicitly required by frozen design/contracts.
9. If AI suggestions exist, require user acceptance before persistence.
10. Use aggregate-level `profileVersion`.
11. Do not create `professionalSummaryVersion`.
12. Preserve text after failed save.
13. Do not silently merge 409 conflicts.
14. Do not update Profile from Tailored Resume changes.
15. Do not automatically regenerate Match/Resume/Cover Letter.
16. Ignore Figma annotation nodes.
17. Do not redesign the frozen UI.

```
`S3-UI-09` 可以冻结。

这里固定下来的核心关系是：

​```text
Candidate Profile
Professional Summary
        │
        │ general + truthful
        ▼
      Job context
        │
        ▼
Tailored Resume Summary
```

**可以从 Profile Summary 向下 Tailor，但不能把 Tailored Resume 的内容自动反写回来。**

下一份进入 **`S3-UI-10 — Skills`**。这份会比 Summary 更值得仔细处理，因为 Skills 涉及 **add / edit / remove、Resume Import proposal、技能去重，以及删除 Skill 后 `profileVersion` 与已有 Match/Resume/CL stale 的关系**。