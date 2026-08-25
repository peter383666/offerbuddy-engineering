S3-UI-13 — Certifications

# S3-UI-13 — Certifications

```yaml
spec_id: S3-UI-13
title: Certifications
surface: web
status: frozen
figma_page: S3-02 Candidate Profile
depends_on:
  - candidate-profile
  - resume-import
s2_reuse: true
```

## 1. Purpose

`Certifications` lets the user maintain verified-by-the-user certification facts that OfferBuddy may use in Match, Tailored Resume and Focused Cover Letter generation.

It answers:

> **What professional certifications can OfferBuddy truthfully claim that I hold?**

Certifications are reusable Candidate Profile facts, not Job-specific optimisation data.

```text
Candidate Profile Certifications
            │
            ├── Match
            ├── Tailored Resume
            └── Focused Cover Letter
```

## 2. Scope

### In Scope

- view saved certifications;
- add certification;
- edit certification;
- remove certification;
- maintain issuer and dates where supported;
- maintain expiry information where supported;
- validate required fields;
- save through Candidate Profile;
- use aggregate `profileVersion`;
- support accepted Resume Import certification proposals;
- handle request failure and OCC conflict.

### Out of Scope

- credential verification with external issuers;
- certification exam booking;
- certificate file vault;
- badge/social verification;
- automatic certification equivalency;
- automatic certification inference from Skills/Experience;
- adding certifications because a Job requires them;
- training recommendations;
- certification renewal workflows unless separately designed.

## 3. Entry Points

Primary path:

```text
Candidate Profile Overview
        ↓
Certifications
        ↓
Edit
        ↓
S3-UI-13 Certifications
```

A certification may also originate through Resume Import:

```text
Resume
  ↓
AI extraction
  ↓
Proposed Certification
  ↓
Review / Edit / Accept
  ↓
Candidate Profile
```

## 4. Preconditions

Required:

- authenticated user;
- authorised Candidate Profile;
- current `profileVersion`.

Certifications may be empty.

An empty certification section is valid Profile state.

## 5. Route / Surface

**Surface:** OfferBuddy Web / Candidate Profile section editor.

Certifications belong to Candidate Profile, not:

- Job;
- Application;
- Preparation;
- Extension;
- Admin.

## 6. Figma Reference

**Figma Page:** `S3-02 Candidate Profile`

Use the final frozen `Certifications Edit` frame and confirm the current Figma Node ID before implementation.

Any `ANNOTATION — ...` content is documentation only.

## 7. UI Structure

The page has two conceptual layers.

### Certification Collection

Show current certifications in a compact reviewable form, for example:

```text
AWS Certified Developer – Associate
Amazon Web Services
Issued 2026
                           Edit

Oracle Certified Professional
Oracle
...
                           Edit

+ Add certification
```

Exact visible metadata follows frozen Figma and schema.

### Certification Editor

Conceptually:

```text
Certification Name
Issuing Organisation
Issue Date
Expiry Date / Does Not Expire
Credential metadata if explicitly supported

Save
Cancel
```

Do not add credential fields merely because other résumé products commonly use them.

## 8. Data Structure

Certification is a Candidate child record under Candidate Profile.

Conceptually:

```text
Candidate Profile
      ↓
Certifications
 ├── Certification A
 └── Certification B
```

It does not become an independent Candidate aggregate.

## 9. Field Specification

Exact fields follow frozen Candidate Profile contract.

Typical model:

| Field               | Type                              | Required                  | Behaviour                        |
| ------------------- | --------------------------------- | ------------------------- | -------------------------------- |
| Certification Name  | Text                              | Yes                       | Actual certification title       |
| Issuer              | Text                              | Contract-defined          | Actual issuing organisation      |
| Issue Date          | Month/year or supported precision | Optional/contract-defined | Preserve precision               |
| Expiry Date         | Month/year or supported precision | Conditional               | Only when certification expires  |
| Does Not Expire     | Boolean                           | If supported              | Controls expiry semantics        |
| Credential ID / URL | Contract-defined                  | Optional                  | Only if frozen model supports it |

Agent must not invent unsupported metadata.

## 10. Certification Name Truth Boundary

Certification titles must reflect actual user-confirmed credentials.

If the user holds:

```text
AWS Certified Developer – Associate
```

OfferBuddy cannot change that to:

```text
AWS Certified Solutions Architect – Professional
```

because the Job requests architecture experience.

Similarly:

```text
Microsoft Azure Fundamentals
```

cannot be presented as a higher-level Azure certification.

Job relevance affects inclusion/emphasis, not certification identity.

## 11. Job Requirement Boundary

This is a critical rule.

Suppose a Job requires:

```text
AWS Certified Solutions Architect
```

and Candidate Profile has no such certification.

Correct:

```text
Job requirement
      +
Candidate Certifications
      ↓
Match
      ↓
Certification gap / missing evidence
```

Incorrect:

```text
Job requires AWS certification
      ↓
Candidate Profile += AWS certification
```

Job requirements never create Candidate credentials.

## 12. Resume Import

Resume Import can propose certification facts.

Example:

```text
Resume:
AWS Certified Developer – Associate
```

becomes:

```text
Proposed Certification
```

Only after user review and acceptance does it become Candidate Profile data.

If extraction is ambiguous:

```text
AWS Developer
```

the AI must not silently upgrade it to a formal certification title.

The user must resolve/correct ambiguity.

## 13. Issue and Expiry Dates

Preserve actual precision.

If the user only knows:

```text
Issued: 2025
```

do not invent:

```text
2025-01-01
```

as a meaningful Candidate fact.

If a certification does not expire, represent that structurally where the frozen model supports it rather than storing a fabricated distant date.

Example:

```text
doesNotExpire = true
expiryDate = null
```

rather than:

```text
expiryDate = 2099-12-31
```

## 14. Expired Certification Semantics

If expiry is supported and the expiry date is in the past:

> the certification record may still be a truthful historical Candidate fact.

Do not automatically delete it.

However, downstream Match/Resume/CL should be able to distinguish:

- certification held historically;
- certification currently valid;

where the domain contract exposes sufficient information.

The Certifications editor itself should display the factual state rather than inventing a validity policy.

## 15. Duplicate Handling

Obvious duplicate records can be flagged.

For example:

```text
AWS Certified Developer – Associate
AWS
2026
```

entered twice.

But do not over-merge similar certifications:

```text
AWS Certified Developer – Associate
AWS Certified Solutions Architect – Associate
```

These are different credentials.

Likewise multiple renewals or versions may legitimately exist depending on the frozen model.

## 16. Relationship to Match

Match consumes certifications as evidence.

Example:

```text
Requirement:
AWS certification preferred

Candidate:
AWS Certified Developer – Associate

→ relevant evidence
```

Or:

```text
Requirement:
Security clearance / certification X

Candidate:
none

→ gap / unsupported
```

Match does not write certifications back.

## 17. Relationship to Tailored Resume

Tailored Resume may:

- include relevant certifications;
- omit irrelevant certifications;
- change display ordering;
- use concise formatting.

It cannot change certification identity or validity facts.

Example:

```text
Candidate Profile:
AWS Certified Developer – Associate
```

may be formatted more compactly, but not upgraded.

## 18. Relationship to Focused Cover Letter

Cover Letter may mention a certification when relevant and supported.

For example:

> I also hold the AWS Certified Developer – Associate certification.

Only if Candidate Profile supports it.

Cover Letter edits do not modify Certifications.

## 19. Actions

### Add Certification

Creates a local draft.

### Edit Certification

Changes existing Candidate fact locally.

### Remove Certification

Removes the certification from the pending Profile update.

### Save

Persist Candidate Profile change with:

```text
expected profileVersion
```

#### Success

- Candidate Profile updated;
- meaningful revision increments `profileVersion`;
- authoritative Certifications refreshed.

#### Validation Failure

Keep local draft.

#### Request Failure

Keep local draft.

#### Conflict

Explicit stale-write conflict.

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

If destructive confirmation is already part of OfferBuddy conventions/Figma, reuse it consistently.

## 21. Business Rules

1. Certifications are authoritative Candidate Profile facts.
2. Certification identity must remain truthful.
3. Job requirements cannot create certifications.
4. Match cannot write certifications.
5. Tailored Resume cannot write certifications.
6. Focused Cover Letter cannot write certifications.
7. Resume Import can only propose certifications.
8. User acceptance establishes imported certification facts.
9. Similar certification names must not be automatically treated as equivalent.
10. Issue/expiry date precision must be preserved.
11. Non-expiring certifications should use structural semantics where supported.
12. Expired certifications are not automatically deleted.
13. Add/edit/remove is a meaningful Candidate fact revision.
14. Candidate Profile uses aggregate `profileVersion`.
15. Certification records do not have separate OCC versions.
16. Certification changes may make derived artifacts stale.
17. Save does not automatically regenerate derived artifacts.
18. Empty Certifications are valid.
19. S2 Fast Apply remains independent.

## 22. API Mapping

Use the frozen Candidate Profile contracts.

Do not infer separate certification endpoints solely because the UI supports CRUD.

Avoid inventing:

```text
POST /api/certifications
PUT /api/certifications/{id}
DELETE /api/certifications/{id}
```

unless already frozen in §3.17.

UI collection editing may still be persisted through Candidate aggregate update semantics.

## 23. Concurrency

Candidate aggregate OCC applies.

Example:

```text
Certification editor loads Profile v9

Another tab updates Experience
→ Profile v10

Certification save with expected v9
→ 409
```

Do not silently merge just because the changed section differs.

## 24. Collection Conflict

Certification collections must not be auto-unioned after conflict.

Example:

```text
Server latest:
Certification A
Certification B

Local stale draft:
Certification A
Certification C
```

Agent cannot assume the desired result is:

```text
A + B + C
```

`B` may have been intentionally removed locally, or server changes may affect user intent.

Explicit review is required.

## 25. Derived Artifact Staleness

Example:

```text
Profile v12
Certification:
AWS Certified Developer – Associate
```

Tailored Resume generated from v12 includes it.

User removes/corrects the certification:

```text
Profile v13
```

Old Tailored Resume becomes stale through provenance.

The Certifications page does not directly rewrite the artifact.

## 26. Navigation / Transitions

| From              | Action              | Destination / State   |
| ----------------- | ------------------- | --------------------- |
| Candidate Profile | Edit Certifications | Certifications        |
| Empty/Ready       | Add                 | New Certification     |
| Ready             | Edit                | Certification Editor  |
| Editor            | Save                | Saving                |
| Saving            | Success             | Overview / Collection |
| Saving            | Failure             | Editing               |
| Saving            | OCC Conflict        | Conflict              |
| Editor            | Cancel              | Overview / Collection |

## 27. S2 Reuse

Reuse:

- Web shell;
- Candidate Profile collection patterns;
- text inputs;
- date controls;
- buttons;
- validation;
- loading/error patterns;
- dirty-state handling.

S3 delta:

> Candidate Certification management.

Do not:

- build credential verification;
- introduce certification catalogue/admin;
- couple Certifications to Application;
- change S2 Fast Apply.

## 28. Component Reuse

Possible responsibilities:

```text
CertificationSection
CertificationList
CertificationCard
CertificationEditor
CredentialDateFields
ProfileSaveActions
ProfileConflictNotice
```

Names are illustrative only.

Prefer existing Candidate Profile collection primitives.

## 29. Responsive / Surface Behaviour

Primary target:

> Desktop Web.

At narrower widths:

- fields may stack;
- certification metadata remains readable;
- Edit/Remove actions stay associated with the correct record.

No separate mobile flow is required.

## 30. Accessibility / UX Requirements

- Certification and issuer fields have labels.
- Expiry controls clearly communicate state.
- `Does not expire` is keyboard accessible where present.
- Remove actions identify the target record.
- Errors map to specific fields.
- Save failure preserves the draft.
- Conflict preserves local work but requires explicit review.
- Empty state includes clear Add action.

## 31. Analytics / Observability

Operational telemetry may contain:

- requestId;
- candidateId;
- profileVersion;
- operation status.

Do not log:

- certification names;
- issuer names;
- credential IDs;
- credential URLs;
- dates;
- Resume Import content.

## 32. Security / Privacy

Certifications are Candidate-owned Profile data.

Backend ownership authorization is mandatory.

Do not expose full Candidate Certifications freely to:

- Extension;
- RuoYi Admin;
- analytics;
- operational logging.

Downstream AI receives only capability-authorised data.

## 33. Acceptance Criteria

-  Existing Certifications are visible.
-  Empty Certifications state is supported.
-  User can add a certification.
-  User can edit a certification.
-  User can remove a certification.
-  Certification name remains truthful.
-  Issuer remains truthful.
-  Issue date follows frozen precision.
-  Expiry date is supported where defined.
-  Non-expiring state is represented structurally where supported.
-  Expired certification is not automatically deleted.
-  Job requirements cannot add a certification.
-  Match cannot write Certifications.
-  Tailored Resume cannot write Certifications.
-  Cover Letter cannot write Certifications.
-  Resume Import requires explicit acceptance.
-  Similar certification names are not automatically merged/equated.
-  Save uses Candidate aggregate `profileVersion`.
-  No certification-specific OCC version is introduced.
-  Request failure preserves local draft.
-  OCC conflict is explicit.
-  Certification collections are not silently auto-merged.
-  Meaningful changes increment `profileVersion`.
-  Certification changes can make derived artifacts stale.
-  Derived artifacts are not automatically regenerated.
-  S2 Fast Apply remains independent.

## 34. Agent Implementation Notes

1. Read frozen Candidate Certification schema/API first.
2. Inspect the final Certifications Figma frame and confirm Node ID.
3. Reuse Candidate Profile collection/date/form patterns.
4. Do not upgrade or reinterpret certification names.
5. Do not create certifications from Job requirements.
6. Treat Resume Import certifications as proposals until accepted.
7. Preserve date precision.
8. Do not represent non-expiring credentials using fake dates.
9. Do not automatically delete expired certifications.
10. Use Candidate aggregate `profileVersion`.
11. Do not create certification-level OCC versions.
12. Preserve local draft on request failure/conflict.
13. Do not silently merge certification collections.
14. Do not let Match/Resume/Cover Letter write back Certifications.
15. Do not automatically regenerate downstream artifacts.
16. Ignore Figma annotations.
17. Do not redesign the frozen UI.

```
`S3-UI-13` 可以冻结。

下一份是最后一个合并的 Candidate fact specification：**`S3-UI-14 — Languages & Eligibility`**。这一份虽然把两个 UI section 放在同一个 spec 里，但我会在文档内部严格分开，因为它们的风险等级不同：Languages 是普通 Candidate fact；**Eligibility / Work Rights 是高敏感、不能由 location/education/Resume/AI 弱推断的事实**。
```