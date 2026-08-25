 **`S3-UI-10 — Skills`**

# S3-UI-10 — Skills

```yaml
spec_id: S3-UI-10
title: Skills
surface: web
status: frozen
figma_page: S3-02 Candidate Profile
depends_on:
  - candidate-profile
  - resume-import
s2_reuse: true
```

## 1. Purpose

`Skills` allows the user to maintain the reusable skills that are part of the authoritative Candidate Profile.

It answers:

> **What skills can OfferBuddy truthfully use when evaluating and preparing applications for me?**

Skills are especially important because they are consumed by multiple downstream capabilities:

```text
Candidate Profile Skills
          │
          ├── Job Match
          ├── Tailored Resume
          └── Focused Cover Letter
```

The Skills page therefore needs to remain simple for the user while maintaining a strict fact boundary for downstream AI.

------

# 2. Scope

### In Scope

- view existing Candidate skills;
- add a skill;
- edit supported skill information;
- remove a skill;
- prevent obvious duplicates;
- save changes through Candidate Profile;
- handle aggregate `profileVersion`;
- handle validation and OCC conflicts;
- support skills previously accepted through Resume Import;
- return to Candidate Profile Overview.

### Out of Scope

- Job-specific skill ranking;
- Match scoring;
- automatic skill inference from Job descriptions;
- automatically accepting Resume-extracted skills;
- endorsements;
- social/profile skill verification;
- complex skill taxonomy management;
- Admin-managed global skill catalogue;
- skill tests;
- proficiency prediction by AI;
- multiple skill profiles for different jobs.

------

# 3. Entry Points

Primary path:

```text
Candidate Profile Overview
        ↓
Skills
        ↓
Edit
        ↓
S3-UI-10 Skills
```

Skills may also have originated from:

```text
Resume Import
      ↓
Proposed Skill
      ↓
User Accepts
      ↓
Candidate Profile Skills
```

But after acceptance, the Skill is simply a Candidate Profile fact.

The Skills editor does not need to permanently distinguish:

> manually entered skill vs imported skill

unless provenance is explicitly required by the frozen backend contract.

------

# 4. Preconditions

Required:

- authenticated user;
- authorised Candidate Profile;
- current `profileVersion`.

Skills may be completely empty.

An empty Skills section is valid Candidate Profile state.

Some downstream capabilities may later report insufficient information, but the Profile itself remains valid.

------

# 5. Route / Surface

**Surface:** OfferBuddy Web / Candidate Profile section editor.

Use existing Candidate Profile routing and shell.

This is not:

- a Preparation screen;
- a Resume screen;
- an Extension screen;
- an Admin skill-management screen.

------

# 6. Figma Reference

**Figma Page**

```
S3-02 Candidate Profile
```

Use the final frozen:

> ```
> Skills Edit
> ```

frame.

The Agent must confirm its current Figma Node ID before implementation.

All `ANNOTATION — ...` nodes are documentation only.

------

# 7. UI Structure

The page consists of:

### A. Existing Web Shell

Reuse:

- header;
- navigation;
- container;
- typography;
- buttons;
- form controls;
- validation/error patterns.

### B. Section Context

Clearly identify:

> Candidate Profile → Skills

### C. Existing Skills

Display the Candidate's current skills as represented by the frozen Figma.

Depending on the final design this may use:

- rows;
- chips/tags;
- compact cards.

The visual implementation must follow Figma rather than inventing a different skill-management pattern.

### D. Add Skill

Provide the frozen interaction for adding another skill.

### E. Edit / Remove

Existing skills can be maintained without requiring the user to rebuild the entire Profile.

### F. Actions

Primary:

> Save

Secondary:

> Cancel / Back

------

# 8. Data Model

The UI must follow the frozen Candidate Skill contract.

Conceptually:

```text
Candidate
   ↓
Candidate Profile
   ↓
Candidate Skills
      ├── Java
      ├── Spring Boot
      ├── MySQL
      └── Redis
```

Do not assume Skills are just one comma-separated string if the backend model represents them as Candidate child records.

Likewise, do not create frontend-only fields that do not exist in the frozen domain.

------

# 9. Field Specification

Exact fields must follow Figma + frozen Candidate Profile contract.

The minimum concept is:

| Field                | Type             | Required         | Editable         | Behaviour         |
| -------------------- | ---------------- | ---------------- | ---------------- | ----------------- |
| Skill name           | Text             | Yes              | Yes              | Trim and validate |
| Other skill metadata | Contract-defined | Contract-defined | Contract-defined | Do not invent     |

If the frozen domain does **not** contain:

- proficiency;
- years of experience;
- skill category;

the Agent must not add them simply because other job platforms have them.

------

# 10. Add Skill

The user can explicitly add a truthful Candidate skill.

Example:

```text
Existing:

Java
Spring Boot
MySQL

Add:
Redis
```

Before Save, this is local editor state.

After successful Candidate Profile update:

```text
Redis
→ accepted Candidate fact
→ profileVersion increments
```

No AI approval is required for a skill the user explicitly enters themselves.

------

# 11. Edit Skill

If supported by the frozen UI/domain, the user can correct an existing skill.

For example:

```text
Spring
      ↓
Spring Boot
```

This is a meaningful Candidate fact revision.

Therefore:

```text
profileVersion + 1
```

on successful Profile mutation.

Editing a skill does not modify existing derived artifacts directly.

------

# 12. Remove Skill

The user can remove an existing skill.

Example:

```text
Candidate Profile v14

Skills:
Java
Spring Boot
Kubernetes
```

User removes Kubernetes:

```text
Profile v15

Skills:
Java
Spring Boot
```

This is especially important because downstream artifacts generated from v14 may still mention Kubernetes.

Therefore:

```text
Skill removed
      ↓
profileVersion changes
      ↓
existing Match / Resume / CL
may become stale
```

But the Skills page must **not automatically edit those artifacts**.

------

# 13. Duplicate Handling

The editor should prevent obvious duplicate Candidate skills.

For example:

```text
Java
java
 JAVA
```

should not normally become three Candidate Skill records.

At minimum duplicate detection should account for harmless formatting differences such as:

- surrounding whitespace;
- appropriate case-insensitive comparison.

However, the Agent must not build an aggressive semantic AI deduplication system.

For example:

```text
Spring
Spring Boot
```

must not automatically be assumed to be duplicates.

Likewise:

```text
Java
JavaScript
```

obviously are not duplicates.

Rule:

> **Prevent obvious duplicates; do not invent semantic equivalence.**

------

# 14. Relationship to Resume Import

Resume Import may propose:

```text
Java
Spring Boot
Redis
AWS
```

These do not become Candidate Skills until explicitly accepted.

Correct:

```text
Resume
  ↓
AI extracts AWS
  ↓
Proposed Skill
  ↓
User accepts
  ↓
Candidate Profile Skill
```

Incorrect:

```text
Resume contains AWS
  ↓
AI extracts AWS
  ↓
Skills page silently contains AWS
```

Once accepted, imported Skills behave like normal Candidate facts.

------

# 15. Relationship to Job Intelligence

This boundary is critical.

Suppose a Job requires:

```text
Java
AWS
Kubernetes
```

and Candidate Profile contains:

```text
Java
Spring Boot
MySQL
```

Job Intelligence / Match may identify:

```text
Java       → supported
AWS        → gap / unsupported
Kubernetes → gap / unsupported
```

It must **not** do this:

```text
Job requires AWS
       ↓
Candidate Skills += AWS
```

Job requirements never establish Candidate skills.

------

# 16. Relationship to Match

Match consumes Skills.

```text
Candidate Skills
       +
Job requirements
       ↓
Match Analysis
```

Match may classify:

- strong match;
- partial alignment;
- gap;
- missing evidence.

Match must not write back to Candidate Skills.

Direction is one-way:

```text
Profile → Match
```

not:

```text
Match → Profile
```

------

# 17. Relationship to Tailored Resume

Tailored Resume may:

- select relevant skills;
- reorder skills;
- emphasise job-relevant skills;
- omit irrelevant skills.

For example:

```text
Candidate Skills:

Java
Spring Boot
MySQL
Redis
React
```

For a backend role:

```text
Tailored Resume:

Java
Spring Boot
MySQL
Redis
```

This does not delete React from Candidate Profile.

Similarly, if the Job asks for AWS and Profile does not contain AWS:

> Tailored Resume must not add AWS as a Candidate skill.

------

# 18. Relationship to Focused Cover Letter

Focused Cover Letter may mention relevant accepted skills.

Example:

```text
Candidate Profile:
Java
Spring Boot
REST APIs
```

Cover Letter may say:

> experience delivering Java and Spring Boot REST APIs

where supported by Candidate facts/evidence.

It cannot introduce an unsupported skill solely because the Job requests it.

------

# 19. Actions

## Add

Creates a local unsaved skill entry.

Does not immediately call downstream AI.

## Edit

Changes local editor state.

## Remove

Marks/removes the skill in local editor state.

No immediate downstream regeneration.

## Save

Persists the resulting Candidate Profile Skills state using:

```text
expected profileVersion
```

### Success

- Candidate Profile updated;
- meaningful change increments `profileVersion`;
- authoritative data refreshed;
- return/remain according to frozen Figma.

### Validation Failure

Preserve local editor state.

### Request Failure

Preserve local editor state.

### OCC Conflict

Explicitly reject stale write.

Do not silently merge skill collections.

## Cancel

Discard unsaved local changes.

Do not mutate Candidate Profile.

------

# 20. UI States

### Loading

Current Candidate Skills loading.

### Empty

No Skills currently saved.

Display a useful empty state with Add capability.

### Ready

Existing Skills displayed.

### Editing / Dirty

At least one local add/edit/remove exists.

### Duplicate Validation

An obvious duplicate has been entered.

Show local actionable feedback.

### Validation Error

A Skill violates frozen validation rules.

### Saving

Prevent duplicate Save.

### Saved

Profile successfully updated.

### Save Failure

Keep local changes.

### Conflict

Profile changed since editor load.

Require review/reload.

------

# 21. Business Rules

1. Skills are authoritative Candidate Profile facts.
2. Skills belong to the single active Candidate Profile.
3. Candidate Skills are not Job-specific.
4. Candidate Skills are not Resume-specific.
5. Job requirements cannot create Candidate Skills.
6. Match cannot create Candidate Skills.
7. Tailored Resume cannot create Candidate Skills.
8. Cover Letter cannot create Candidate Skills.
9. Resume Import can only propose Skills.
10. Explicit user acceptance makes imported Skills Candidate facts.
11. User-entered Skills are explicit Candidate facts after successful Save.
12. Obvious duplicates should be prevented.
13. Semantic skill equivalence must not be guessed aggressively.
14. Add/edit/remove is a meaningful Candidate fact revision.
15. Candidate Profile aggregate uses `profileVersion`.
16. Candidate Skills do not have independent OCC versions.
17. Removing a Skill can make existing derived artifacts stale.
18. Saving Skills does not automatically regenerate those artifacts.
19. Empty Skills are allowed at Profile level.
20. S2 Fast Apply remains independent.

------

# 22. API Mapping

Use the frozen Candidate Profile API contracts.

Candidate Skill persistence must respect the actual Candidate aggregate/child-table design.

Do not invent independent public APIs such as:

```text
POST /api/skills
DELETE /api/skills/{skillId}
```

simply because the UI has Add/Remove controls, unless those operations are explicitly part of the frozen API contract.

The UI interaction:

```text
Add
Edit
Remove
```

does not imply:

> three separate domain APIs.

Save according to the frozen Candidate Profile contract.

------

# 23. Concurrency

Candidate Profile OCC is aggregate-level.

Example:

```text
Skills editor loads:

profileVersion = 30
```

Meanwhile Experience changes:

```text
profileVersion = 31
```

User saves Skills using:

```text
expectedProfileVersion = 30
```

Backend:

```text
409 Conflict
```

The frontend must not decide:

> "Experience changed, Skills didn't, so merging is safe."

That would violate the frozen consistency model.

------

# 24. Collection Merge Warning

Skills are particularly tempting for an Agent to auto-merge.

For example:

```text
Server:
Java
Spring Boot
Redis

Local stale editor:
Java
Spring Boot
AWS
```

Agent must **not** automatically produce:

```text
Java
Spring Boot
Redis
AWS
```

after a conflict.

Why?

Because `Redis` may have been deliberately removed locally, or `AWS` may have been added against an outdated Profile context.

Frozen rule remains:

> **409 → explicit review, never silent merge.**

------

# 25. Derived Artifact Staleness

Example:

```text
Profile v40
Skills:
Java
Spring Boot
AWS

        ↓

Tailored Resume
sourceProfileVersion = 40
mentions AWS
```

User removes AWS:

```text
Profile v41
```

Existing Tailored Resume:

```text
sourceProfileVersion = 40
```

is now stale.

The Skills page does not mutate its text.

Instead:

```text
Tailored Resume Review
→ stale warning
→ regenerate if user chooses
```

The same principle applies to Match and Focused Cover Letter.

------

# 26. Navigation / Transitions

| From              | Action           | Destination / State        |
| ----------------- | ---------------- | -------------------------- |
| Candidate Profile | Edit Skills      | Skills Editor              |
| Empty             | Add              | Dirty                      |
| Ready             | Add/Edit/Remove  | Dirty                      |
| Dirty             | Save             | Saving                     |
| Saving            | Success          | Overview / Saved           |
| Saving            | Validation Error | Editing                    |
| Saving            | Failure          | Editing                    |
| Saving            | OCC Conflict     | Conflict                   |
| Editor            | Cancel           | Candidate Profile Overview |

------

# 27. S2 Reuse

### Reuse

- Web shell;
- Candidate Profile section patterns;
- inputs;
- buttons;
- chips/tags where already available;
- validation;
- loading/error patterns;
- dirty-state handling.

### S3 Delta

Introduce authoritative Candidate Skills management.

### Do Not

- introduce a global Skills service;
- introduce a taxonomy platform;
- couple Skills to Applications;
- add Job requirements into Profile;
- modify S2 Fast Apply.

------

# 28. Component Reuse

Possible responsibilities:

```text
CandidateProfileSectionEditor
SkillsEditor
SkillItem
SkillInput
ProfileSaveActions
ProfileConflictNotice
```

Names are not contracts.

Prefer one reusable collection-editing pattern where suitable for:

- Skills;
- Languages;
- Certifications,

without forcing structurally different domains into an inappropriate generic abstraction.

------

# 29. Responsive / Surface Behaviour

Primary design:

> Desktop Web.

At narrower widths:

- skill controls may wrap;
- Add input remains usable;
- remove/edit actions remain associated with the correct Skill.

Do not introduce a separate mobile flow.

------

# 30. Accessibility / UX Requirements

- Add/Edit/Remove actions must be keyboard accessible.
- Remove controls need accessible names identifying the Skill.
- Selected/editing state cannot rely solely on colour.
- Duplicate errors should explain which value conflicts.
- Empty state must provide a clear Add action.
- Save failure must preserve the collection edits.
- Removing a Skill should not be easy to trigger accidentally through ambiguous controls.

------

# 31. Analytics / Observability

Operational telemetry may include:

- requestId;
- candidateId;
- profileVersion;
- number of collection changes where useful;
- success/failure category.

Do not log:

- Skill names;
- full Candidate Profile;
- Resume Import extraction content.

The observability goal is operation diagnosis, not Candidate profiling.

------

# 32. Security / Privacy

Skills are Candidate-owned data.

Backend authorization must resolve ownership from security context.

The Browser Extension does not receive unrestricted Candidate Skills.

RuoYi Admin does not become a Candidate Profile editor.

AI capabilities receive only data allowed by their specific frozen data boundary.

------

# 33. Acceptance Criteria

-  User can open Skills from Candidate Profile.
-  Existing Skills are visible.
-  Empty Skills state is supported.
-  User can add a Skill.
-  User can edit a Skill where supported.
-  User can remove a Skill.
-  Obvious duplicates are prevented.
-  `Java` and trivial case/whitespace variants do not become duplicate records.
-  Semantic equivalence is not aggressively guessed.
-  Job requirements cannot automatically become Candidate Skills.
-  Match cannot write Skills into Candidate Profile.
-  Tailored Resume cannot write Skills into Candidate Profile.
-  Cover Letter cannot write Skills into Candidate Profile.
-  Resume Import Skills require explicit acceptance.
-  Save uses Candidate aggregate `profileVersion`.
-  No Skill-level OCC version is introduced.
-  Save failure preserves local edits.
-  Concurrent Profile update produces explicit conflict.
-  Skill collections are not silently auto-merged after conflict.
-  Meaningful Skill changes increment `profileVersion`.
-  Removing a Skill can make derived artifacts stale.
-  Derived artifacts are not automatically regenerated.
-  S2 Fast Apply remains independent.
-  Candidate Skill data is authorised server-side.

------

# 34. Agent Implementation Notes

1. Read frozen Candidate Profile and Candidate Skill contracts.
2. Inspect the final Skills Figma frame and confirm its Node ID.
3. Inspect existing collection/chip/form components before creating new ones.
4. Model Skills according to the existing Candidate child-record schema.
5. Do not collapse structured Skills into a comma-separated database concept.
6. Do not invent proficiency/years/category fields.
7. Prevent only obvious duplicates unless backend contract defines stronger normalisation.
8. Do not use Job requirements to populate Candidate Skills.
9. Keep Resume Import proposals outside Profile until acceptance.
10. Use aggregate-level `profileVersion`.
11. Do not create `skillVersion`.
12. Do not silently merge Skill collections after 409.
13. Preserve local edits after request failure.
14. Do not automatically regenerate Match/Resume/Cover Letter.
15. Do not build a global skill taxonomy/Admin feature.
16. Reuse existing Web/Profile components.
17. Ignore Figma annotations.
18. Do not redesign the frozen UI.

```
`S3-UI-10` 可以冻结。

这里最重要的是把 **Skill 的四种来源语义**彻底分开：

​```text
User explicitly adds
        ↓
Candidate Skill after Save

Resume Import extracts
        ↓
Proposal
        ↓
Accept
        ↓
Candidate Skill

Job requires skill
        ↓
Requirement only
        ✕
Not Candidate Skill

AI derived artifact mentions/emphasises skill
        ↓
Derived content only
        ✕
Not Candidate Skill
```

尤其 Agent 很容易看到 JD 里有 `AWS`，为了提高 Match/Resume 效果顺手把它补进 Profile——这是 S3 明确不能发生的。

下一份进入 **`S3-UI-11 — Experience`**。这份会是 Candidate Profile 编辑组里最重要、也最复杂的一份，我们会把之前专门讨论过的 **company / role / start date / end date / current role / description / experience items，以及时间编辑和 Resume Import coherent-record handling** 全部冻结下来。