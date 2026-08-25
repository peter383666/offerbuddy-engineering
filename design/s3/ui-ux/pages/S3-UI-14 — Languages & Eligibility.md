# S3-UI-14 — Languages & Eligibility

```yaml
spec_id: S3-UI-14
title: Languages & Eligibility
surface: web
status: frozen
figma_page: S3-02 Candidate Profile
depends_on:
  - candidate-profile
  - resume-import
s2_reuse: true
```

## 1. Purpose

`Languages & Eligibility` covers two Candidate Profile sections that are both reusable Candidate facts but have very different risk profiles.

The page answers two separate questions:

> **What languages can OfferBuddy truthfully say I use?**

and:

> **What work-rights / eligibility facts has the user explicitly confirmed?**

The second question is particularly sensitive because it affects sponsor signals, PR/citizenship requirements and job suitability.

Core rule:

```text
Languages
= ordinary Candidate facts

Eligibility / Work Rights
= explicit high-impact Candidate facts
```

Eligibility must never be weakly inferred from other Profile information.

------

# 2. Scope

### In Scope — Languages

- view current Languages;
- add Language;
- edit supported proficiency information;
- remove Language;
- validate duplicate entries;
- save through Candidate Profile;
- handle Resume Import proposals.

### In Scope — Eligibility

- view current work-rights / eligibility information;
- edit only the fields supported by the frozen model;
- maintain visa/work-rights information where applicable;
- maintain expiry date where applicable;
- explicitly represent unknown / not provided rather than guessing;
- save through Candidate Profile;
- handle aggregate `profileVersion`;
- handle OCC conflict.

### Out of Scope

- immigration advice;
- visa eligibility assessment;
- citizenship verification;
- PR prediction;
- sponsorship guarantees;
- legal interpretation of visa conditions;
- automatically inferring work rights from Education;
- automatically inferring work rights from Location;
- automatically inferring work rights from Resume;
- automatically changing Eligibility from Job requirements;
- automatically determining security clearance;
- migration pathway planning.

------

# 3. Entry Points

Primary paths:

```text
Candidate Profile Overview
        ↓
Languages
        ↓
Edit
```

and:

```text
Candidate Profile Overview
        ↓
Eligibility
        ↓
Edit
```

These may share a specification because their CRUD mechanics are similar, but they remain separate Profile sections and should not be merged into a single business object merely for frontend convenience.

------

# 4. Preconditions

Required:

- authenticated user;
- authorised Candidate Profile;
- current `profileVersion`.

Both sections may be empty.

Empty Eligibility means:

> **OfferBuddy does not currently know the answer.**

It does **not** mean:

- no work rights;
- sponsorship required;
- permanent residency;
- citizenship.

Unknown must remain unknown.

------

# 5. Route / Surface

**Surface:** OfferBuddy Web / Candidate Profile section editors.

Use existing Candidate Profile routing and page shell.

These are not:

- Extension settings;
- Admin settings;
- Job-specific fields;
- Application fields.

------

# 6. Figma Reference

**Figma Page**

```
S3-02 Candidate Profile
```

Relevant frozen frames:

- `Languages Edit`
- `Eligibility Edit`

Agent must confirm the current final Node IDs before implementation.

Any:

```text
ANNOTATION — ...
```

content is design documentation only.

------

# 7. Languages — UI Structure

The Languages section conceptually includes:

### Language Collection

Example:

```text
English
Professional working proficiency

Mandarin Chinese
Native / bilingual

                         Edit

+ Add language
```

Exact terminology and proficiency choices follow the frozen model/Figma.

### Language Editor

Conceptually:

```text
Language
Proficiency

Save
Cancel
```

Do not introduce:

- test scores;
- certification files;
- CEFR mappings;
- IELTS/PTE scores;

unless explicitly part of the frozen Candidate Profile schema.

------

# 8. Language Field Specification

Typical structure:

| Field       | Type                       | Required         | Behaviour                 |
| ----------- | -------------------------- | ---------------- | ------------------------- |
| Language    | Text / supported selection | Yes              | Actual language           |
| Proficiency | Enum / text                | Contract-defined | Use frozen allowed values |

The Agent must use backend-defined proficiency semantics.

Do not create a richer language taxonomy solely for UI convenience.

------

# 9. Language Duplicate Handling

Obvious duplicate Language records should be prevented.

For example:

```text
English
english
 ENGLISH
```

should normally map to one language fact.

But do not infer semantic equivalence across language variants unless the frozen contract defines it.

For example:

```text
Mandarin Chinese
Cantonese
```

must not be merged.

------

# 10. Resume Import — Languages

Resume Import may propose:

```text
English — Fluent
Mandarin — Native
```

These remain proposals until accepted.

If proficiency is not stated:

```text
Languages: English, Mandarin
```

AI must not confidently invent:

```text
English — Native
```

The proposal may instead contain:

- language with unknown proficiency;
- or require user completion.

------

# 11. Eligibility — Purpose

Eligibility stores the Candidate's explicitly confirmed ability/status relevant to working in the target market.

It exists so downstream capabilities can make statements such as:

```text
Job requires Australian citizenship

Candidate Eligibility:
Temporary work rights

→ Requirement warning
```

or:

```text
Employer is on sponsor dataset

Candidate indicates sponsorship may be required

→ relevant sponsor signal
```

But OfferBuddy must never fabricate the Candidate side of this comparison.

------

# 12. Eligibility Field Specification

Exact fields must follow the frozen Candidate Profile schema and Figma.

Conceptually they may include:

| Field                   | Type                           | Required    | Behaviour                 |
| ----------------------- | ------------------------------ | ----------- | ------------------------- |
| Work-rights status      | Enum / supported value         | Optional    | Explicit user declaration |
| Visa type               | Text/enum where supported      | Optional    | User-confirmed            |
| Visa expiry             | Date/month where applicable    | Conditional | Only when relevant        |
| Sponsorship requirement | Explicit value where supported | Optional    | Must not be inferred      |
| Citizenship / PR status | Supported field only           | Optional    | Explicit user declaration |

Do not invent immigration categories beyond the frozen model.

------

# 13. Unknown / Not Provided State

Eligibility must support absence of information cleanly.

Correct:

```text
Work rights: Not provided
```

Incorrect:

```text
Work rights: Requires sponsorship
```

just because the field is empty.

Likewise:

```text
Citizenship: unknown
```

must not become:

```text
Not a citizen
```

Unknown and negative are not the same.

------

# 14. Location Boundary

This rule is frozen.

If Personal Details says:

```text
Location:
Sydney, NSW
```

OfferBuddy cannot infer:

```text
Australian citizen
Permanent resident
Unrestricted work rights
```

Correct:

```text
Location
≠
Eligibility
```

Location answers where the Candidate is.

Eligibility answers what work/legal status the Candidate explicitly declares.

------

# 15. Education Boundary

Similarly:

```text
Education:
Master's degree from an Australian university
```

does not establish:

```text
Post-study work visa
Permanent residency
Citizenship
Unrestricted work rights
```

Correct:

```text
Australian Education
≠
Australian work-rights fact
```

Education and Eligibility remain separate sections.

------

# 16. Resume Import — Eligibility

Resume Import must be highly conservative.

If Resume explicitly states:

```text
Australian Permanent Resident
```

it may become:

> Proposed Eligibility Fact

which still requires user acceptance.

If Resume states:

```text
Based in Sydney
```

AI must **not** propose:

> Permanent Resident.

If Resume states:

```text
No sponsorship required
```

the system may propose the corresponding supported fact, but user confirmation is still required.

------

# 17. Job Requirement Boundary

Job requirements must never write Candidate Eligibility.

Example:

```text
Job:
Australian citizens only
```

Correct:

```text
Job requirement
      +
Candidate Eligibility
      ↓
Compatibility warning
```

Incorrect:

```text
Job says citizen
      ↓
Candidate citizenship field = citizen
```

This seems obvious, but it is exactly the kind of accidental state contamination an implementation agent must never introduce.

------

# 18. Extension Relationship

The Browser Extension may use authorised, minimal eligibility-derived state to display contextual warnings.

Example:

```text
Job requires PR/citizenship
      ↓
Extension detects requirement
      ↓
Candidate status comparison
      ↓
Requirement Review signal
```

However:

- Extension does not own Eligibility;
- Extension does not freely download the entire Candidate Profile;
- Extension does not update Eligibility;
- Extension does not infer missing Candidate status.

The narrow API/cache behaviour must follow frozen S3 Extension contracts.

------

# 19. Sponsor Employer Relationship

Sponsor dataset answers:

> Is this employer known to support sponsorship?

Candidate Eligibility answers:

> What has the Candidate explicitly said about their work-rights situation?

They are different datasets.

```text
Sponsor Employer Dataset
        +
Candidate Eligibility
        +
Job Requirement
        ↓
Contextual assistance
```

Do not collapse these into one boolean such as:

```text
goodForCandidate = true
```

because the inputs have different provenance and meaning.

------

# 20. Visa / Expiry Behaviour

Where visa expiry is supported, preserve the actual date precision defined by the domain.

Do not create fake expiry dates.

For a status with no expiry:

```text
expiry = null
```

where contract-defined.

Do not use:

```text
2099-12-31
```

as a substitute for permanent/no-expiry status.

------

# 21. Expired Eligibility Information

If stored visa/work-rights information reaches its expiry date, the UI must not silently invent the Candidate's next status.

For example:

```text
Temporary visa
Expiry: 01 Mar 2027
```

after expiry does not imply:

```text
No work rights
```

or:

```text
Permanent resident
```

unless the user updates the Profile.

The fact may need a visible stale/outdated indication depending on frozen domain semantics, but automatic migration-status transition is out of scope.

------

# 22. Add / Edit / Remove Language

### Add

Creates a local draft Language fact.

### Edit

Updates supported language/proficiency values.

### Remove

Removes the Language after successful Save.

All participate in Candidate aggregate revision.

------

# 23. Edit Eligibility

Eligibility should generally be edited as explicit form fields rather than generated prose.

Example:

```text
Work rights:
[ Temporary work rights ]

Visa type:
[ ... ]

Expiry:
[ ... ]
```

This makes the truth boundary clearer than using a single free-text paragraph.

Use exact Figma/model fields.

------

# 24. Actions

## Save

For either Languages or Eligibility:

```text
Candidate Profile mutation
+
expected profileVersion
```

### Success

- authoritative Profile updates;
- meaningful change increments `profileVersion`;
- refreshed value shown.

### Validation Failure

Preserve local form state.

### Request Failure

Preserve local form state.

### Conflict

Explicit OCC conflict.

Never silent merge.

## Cancel / Back

Discard unsaved local edits.

Use existing dirty-state UX.

------

# 25. UI States

For Languages:

- Loading
- Empty
- Ready
- Creating
- Editing
- Dirty
- Duplicate Validation
- Saving
- Save Failure
- Conflict

For Eligibility:

- Loading
- Not Provided
- Ready
- Editing
- Dirty
- Validation Error
- Saving
- Save Failure
- Conflict

`Not Provided` must be a first-class valid display state.

------

# 26. Business Rules — Languages

1. Languages are Candidate Profile facts.
2. Languages are not Job-specific.
3. Resume Import only proposes Languages.
4. User acceptance establishes imported Language facts.
5. Job requirements cannot create Languages.
6. Match cannot create Languages.
7. Resume/Cover Letter cannot write Languages back.
8. Obvious duplicates should be prevented.
9. Proficiency must not be inferred without evidence.
10. Language changes increment Candidate `profileVersion`.
11. Languages do not have their own OCC version.

------

# 27. Business Rules — Eligibility

1. Eligibility is authoritative Candidate Profile data only after explicit user confirmation.
2. Eligibility must not be inferred from Location.
3. Eligibility must not be inferred from Education.
4. Eligibility must not be inferred from Job requirements.
5. Eligibility must not be inferred from Sponsor Employer status.
6. Resume Import may only propose Eligibility when source evidence supports it.
7. User acceptance is required for imported Eligibility.
8. Unknown does not mean negative.
9. Missing status does not mean sponsorship required.
10. Visa expiry does not determine the Candidate's next status.
11. Job Intelligence may consume Eligibility for warnings, not modify it.
12. Extension may consume narrow authorised eligibility context, not own it.
13. Match may identify requirement incompatibility but cannot change Candidate status.
14. Tailored Resume / Cover Letter cannot invent eligibility claims.
15. Eligibility changes increment Candidate `profileVersion`.
16. Eligibility does not have independent OCC.
17. Eligibility changes may make Match/Resume/CL stale where they depend on it.
18. Saving does not automatically regenerate those artifacts.
19. Candidate Eligibility is not immigration/legal advice.

------

# 28. Relationship to Match

Example:

```text
Job:
Must have unrestricted Australian work rights

Candidate Eligibility:
Temporary work rights
```

Match can report:

> Important eligibility mismatch / attention required.

It must not change either side.

If Candidate Eligibility is unknown:

Match should represent:

> insufficient Candidate information

rather than:

> Candidate does not qualify

unless the Job logic and frozen contract explicitly support that conclusion.

------

# 29. Relationship to Tailored Resume

Tailored Resume should generally avoid inserting visa/work-rights information unless relevant and explicitly supported by the generation contract.

If it does include such information, it must use Candidate Eligibility facts exactly.

It cannot turn:

```text
Temporary work rights
```

into:

```text
Full Australian working rights
```

for better Job alignment.

------

# 30. Relationship to Focused Cover Letter

Same rule.

If the Candidate has an explicitly supported fact such as:

```text
No sponsorship required
```

a Cover Letter may use it when relevant.

If Candidate Eligibility is unknown:

> Cover Letter must not make a confident eligibility claim.

------

# 31. API Mapping

Use frozen Candidate Profile contracts.

The UI's separate section editors do not imply independent public aggregates.

Do not invent APIs such as:

```text
PUT /api/eligibility
POST /api/languages
```

unless already frozen.

Persist according to the existing Candidate Profile contract and aggregate semantics.

------

# 32. Concurrency

Both Languages and Eligibility participate in:

```text
CandidateProfile.profileVersion
```

Example:

```text
Eligibility editor loads v16

Another tab edits Experience
→ v17

Eligibility Save using v16
→ 409
```

No silent merge.

This is especially important for Eligibility because automatically applying stale legal/work-rights data could create materially misleading outputs.

------

# 33. Collection Conflict — Languages

Do not automatically union Language lists after OCC conflict.

Example:

```text
Latest:
English
Mandarin

Local stale:
English
Japanese
```

Do not assume:

```text
English + Mandarin + Japanese
```

is intended.

Explicit review is required.

------

# 34. Derived Artifact Staleness

Example:

```text
Profile v22
Eligibility:
Requires sponsorship
```

Match generated from v22.

User updates:

```text
Profile v23
Eligibility:
No sponsorship required
```

Existing Match may now contain a materially incorrect warning.

Therefore:

```text
Match sourceProfileVersion = 22
Current profileVersion = 23
→ stale
```

The Eligibility editor itself does not regenerate Match automatically.

------

# 35. Navigation / Transitions

| From              | Action           | Destination / State        |
| ----------------- | ---------------- | -------------------------- |
| Candidate Profile | Edit Languages   | Languages Editor           |
| Languages         | Add/Edit/Remove  | Dirty                      |
| Languages         | Save             | Saving                     |
| Candidate Profile | Edit Eligibility | Eligibility Editor         |
| Eligibility       | Change fields    | Dirty                      |
| Eligibility       | Save             | Saving                     |
| Saving            | Success          | Overview / Saved           |
| Saving            | Failure          | Editing                    |
| Saving            | OCC Conflict     | Conflict                   |
| Editor            | Cancel           | Candidate Profile Overview |

------

# 36. S2 Reuse

Reuse:

- Web shell;
- Candidate Profile form/collection patterns;
- inputs;
- select controls;
- date controls;
- buttons;
- validation;
- dirty-state handling;
- loading/error components.

S3 delta:

> Candidate Languages and explicit Eligibility management.

Do not:

- build immigration workflow;
- add visa-advice UI;
- build a second Extension eligibility database;
- create Admin Candidate Eligibility management;
- alter S2 Fast Apply.

------

# 37. Component Reuse

Possible responsibilities:

```text
LanguageSection
LanguageList
LanguageEditor
EligibilityEditor
EligibilityStatusFields
ProfileSaveActions
ProfileConflictNotice
```

Names are illustrative.

Do not force Languages and Eligibility into the same generic editor just because this specification documents them together.

Their business semantics are different.

------

# 38. Responsive / Surface Behaviour

Primary frozen target:

> Desktop Web.

At narrower widths:

- form fields stack;
- date inputs remain readable;
- Language collection remains manageable;
- warning/help text does not overwhelm the form.

No separate mobile workflow is required.

------

# 39. Accessibility / UX Requirements

### Languages

- Language/proficiency controls have labels.
- Remove action identifies the correct language.
- Duplicate errors are understandable.
- Empty state provides clear Add.

### Eligibility

- every field has explicit label;
- unknown/not-provided state is clearly communicated;
- visa expiry field is associated with the relevant status;
- status meaning must not depend on colour;
- potentially important work-rights information should use plain language;
- saving conflict must be especially clear.

------

# 40. Analytics / Observability

Operational telemetry may include:

- requestId;
- candidateId;
- profileVersion;
- section operation;
- success/failure metadata.

Do not log:

- language values;
- proficiency;
- visa type;
- work-rights status;
- citizenship/PR fields;
- expiry dates.

Eligibility data is sensitive Candidate information.

------

# 41. Security / Privacy

Candidate Languages and particularly Eligibility are Candidate-owned sensitive data.

Authorization must be server-side.

Extension access must be narrow and capability-specific.

RuoYi Admin must not become a Candidate eligibility viewer/editor.

Do not expose Eligibility through:

- general operational logs;
- analytics payloads;
- unrestricted Extension APIs.

AI receives only the minimum approved Eligibility facts required by the capability.

------

# 42. Acceptance Criteria

### Languages

-  Existing Languages are visible.
-  Empty Languages state is supported.
-  User can add a Language.
-  User can edit supported Language information.
-  User can remove a Language.
-  Obvious duplicate Languages are prevented.
-  Proficiency is not invented when unknown.
-  Resume Import Languages require explicit acceptance.
-  Job requirements cannot populate Languages.
-  Save uses Candidate aggregate `profileVersion`.

### Eligibility

-  Current Eligibility is visible when provided.
-  Not-provided state is supported.
-  User can explicitly edit supported Eligibility fields.
-  Location does not infer Eligibility.
-  Australian Education does not infer Eligibility.
-  Job requirements do not alter Eligibility.
-  Sponsor Employer status does not alter Eligibility.
-  Resume Import cannot establish Eligibility without explicit acceptance.
-  Unknown does not become negative.
-  Missing information does not automatically mean sponsorship required.
-  Visa expiry is represented accurately where supported.
-  Expired status does not trigger an invented replacement status.
-  Match may consume but cannot modify Eligibility.
-  Extension may consume only narrow authorised state.
-  Tailored Resume cannot invent work-rights claims.
-  Focused Cover Letter cannot invent work-rights claims.
-  Save failure preserves local form state.
-  OCC conflict is explicit.
-  No section-specific OCC version is introduced.
-  Meaningful changes increment `profileVersion`.
-  Eligibility change can make derived artifacts stale.
-  Downstream artifacts are not automatically regenerated.
-  S2 Fast Apply remains independent.

------

# 43. Agent Implementation Notes

1. Read frozen Candidate Languages and Eligibility schema/contracts first.
2. Inspect both final Figma frames and confirm Node IDs.
3. Keep Languages and Eligibility as distinct Candidate Profile sections.
4. Do not infer Eligibility from Location.
5. Do not infer Eligibility from Australian Education.
6. Do not infer Candidate status from Job requirements.
7. Do not infer Candidate status from Sponsor Employer data.
8. Preserve explicit `unknown/not provided` semantics.
9. Treat Resume Import Eligibility as proposal only.
10. Do not invent proficiency levels for Languages.
11. Use Candidate aggregate `profileVersion`.
12. Do not create Language/Eligibility OCC versions.
13. Never silently merge a stale Eligibility write.
14. Do not automatically transition visa status after expiry.
15. Keep Extension access narrow.
16. Do not expose Candidate Eligibility in RuoYi Admin.
17. Do not automatically regenerate derived artifacts.
18. Reuse existing Profile/form/date primitives.
19. Ignore Figma annotations.
20. Do not redesign the frozen UI.

```
`S3-UI-14` 可以冻结。

这样 **Candidate Profile fact specifications `S3-UI-06` ～ `S3-UI-14` 已经全部完成**。核心事实链现在完整覆盖：

​```text
Personal Details
Professional Summary
Skills
Experience
Education
Certifications
Languages
Eligibility
        │
        ▼
Candidate Profile + profileVersion
        │
        ├── Match
        ├── Tailored Resume
        └── Focused Cover Letter
```

下一组进入 **Extension / Fast Apply**：

**`S3-UI-15 — Extension Floating Assistant`**。

这份会把已经冻结的 `Collapsed / Hover / Sponsor Signal / Requirement Review / Preparation Ready / Already Applied` 统一建模成一个状态机，并明确 **Extension 不展示 Match**、Sponsor snapshot 优先本地缓存、`Prepare with OfferBuddy` 只是打开 Web focused preparation。