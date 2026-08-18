# Application Detail S2 Specification

## 1. Purpose

Application Detail remains an Application tracking page and gains S2 Job
Intelligence as a downstream enrichment section.

AI is not the page's primary identity.

## 2. Information Hierarchy

Recommended hierarchy:

``` text
Job Header / Overview

Application Tracking

Job Information / Original Posting

Job Intelligence
```

Do not introduce tabs in S2 solely for Job Intelligence.

## 3. Header / Overview

Prioritise:

1.  Job title
2.  Company
3.  Location/source where available
4.  Applied date
5.  Current Application status

Do not show implementation metadata such as: - Created via Browser
Extension - extraction confidence - AI model - event processing state

## 4. Application Tracking

Preserve existing S1 functionality: - current status; - status update; -
status history; - notes and other existing tracking behaviour.

Extension temporary states such as pending/confirmation must not become
Application lifecycle statuses.

A Web Application lifecycle begins from the actual tracked Application
semantics.

## 5. Job Information

Show useful persisted facts when available, such as: - source; -
location; - employment type; - salary text where present; - original job
posting link.

Omit absent optional fields rather than rendering repeated `Unknown`,
`N/A` or `Not available` noise.

## 6. Original Job Posting

Provide a clear external-link action when a source URL exists:

`View original job posting ↗`

The external-link affordance must make it clear that the user is leaving
OfferBuddy.

## 7. Captured Job Description

The backend may persist the captured Job description as the stable
source snapshot for system/AI purposes.

S2 Application Detail does **not** display the full captured Job
Description.

Primary user-facing reference remains the original job posting link.

If later product evidence shows that users frequently need expired
captured JDs, this may be reconsidered in a future sprint.

## 8. Things to Review

S2 does **not require** persisted Extension screening findings to be
shown on Application Detail.

Things to Review is required in the pre-application Extension
experience.

Do not add new persistence/API complexity solely to reproduce those
findings on Application Detail in S2.

## 9. Job Intelligence

Show exactly these S2 intelligence areas:

1.  Summary
2.  Responsibilities
3.  Requirements
4.  Skills

Recommended structure:

``` text
Job Intelligence

Summary
...

Responsibilities
• ...
• ...

Requirements
• ...
• ...

Skills
[ Java ] [ Spring Boot ] [ AWS ]
```

Do not add: - match score; - resume match; - candidate gaps; - resume
recommendations; - cover letter; - interview questions; - candidate fit
judgment.

## 10. Job Intelligence States

### Processing

``` text
Job Intelligence

Analysing this job...
```

Do not block the rest of Application Detail.

### Available

Render the four approved intelligence areas.

### Unavailable / Retryable

``` text
Job Intelligence

Analysis isn't available right now.

[ Retry analysis ]
```

### Insufficient Information

``` text
Job Intelligence

Not enough job information is available to analyse this role.
```

Do not label insufficient input as a generic AI failure.

## 11. Failure Isolation

A Job Intelligence failure must never turn the whole Application Detail
into an error page.

Application tracking remains available.

## 12. Responsive

Use a single structured page.

On narrow screens: - sections stack naturally; - Skill chips wrap; -
long content grows vertically; - no essential content is clipped; - no
horizontal page scrolling.
