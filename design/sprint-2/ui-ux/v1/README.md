# OfferBuddy S2 UI/UX Specification

Status: Final specification after Phase 4 UI/UX review\
Scope: OfferBuddy Sprint 2 (S2)

## 1. Purpose

This directory is the implementation-facing UI/UX specification for
OfferBuddy S2.

It complements the S2 Technical Design documents already stored under
`design/s2/`.

-   **Figma** is the visual source of truth.
-   **These specifications** are the behaviour, state, interaction,
    responsive and accessibility source of truth.
-   **The private code repository** implements the approved Figma and
    these specifications.

If a Figma frame and this specification appear inconsistent, resolve the
inconsistency before implementation rather than inventing behaviour.

## 2. S2 Product Focus

S2 has three product capabilities:

1.  Browser Extension as the primary job-search workflow for SEEK and
    Indeed.
2.  Asynchronous Job Intelligence enrichment after job/application data
    has been persisted.
3.  Small, focused Application Analytics.

S2 must remain an Application Tracker first. AI and Analytics are
supporting capabilities.

## 3. Primary User Journey

``` text
SEEK / Indeed
    ↓
Extension detects active job
    ↓
Capture reliable page facts
    ↓
Show Things to Review
    ↓
User starts Apply flow
    ↓
Remember temporary pending job/application context
    ↓
SEEK / Indeed apply flow OR external company ATS
    ↓
Resolve outcome
    ├─ reliable platform applied evidence
    └─ user confirmation when platform evidence is unavailable
    ↓
Track in OfferBuddy
    ↓
Application available immediately
    ↓
Async Job Intelligence
```

Clicking an Apply button is **not** proof that an application was
submitted.

## 4. Web Information Architecture

``` text
OfferBuddy
├── Home
├── Applications
│   ├── Application List
│   ├── Application Detail
│   └── New Application
└── Analytics
```

Do not add a top-level Jobs page in S2.

## 5. Specification Files

-   `browser-extension.md`
-   `home.md`
-   `applications.md`
-   `application-detail.md`
-   `new-application.md`
-   `analytics.md`
-   `ui-states.md`
-   `responsive-accessibility.md`
-   `design-system-delta.md`
-   `technical-design-deltas.md`

## 6. Global UX Principles

### Truth before convenience

-   Apply intent is not submission.
-   Unknown state remains unknown.
-   `Tracked in OfferBuddy` is shown only after backend confirmation.
-   Screening findings must be evidence-based, not candidate eligibility
    judgments.
-   Optional capability failure must not look like core Application
    failure.

### Business language before implementation language

Expose: - Application - Status - SEEK / Indeed - Original job posting -
Job Intelligence - Things to review

Do not expose normal implementation details such as: - Site Adapter -
event state - AI model - extraction confidence - created by extension -
retry worker state

### Progressive enhancement

Application tracking must remain usable if: - Job Intelligence is
processing or unavailable; - Analytics is unavailable; - an original job
URL later becomes inaccessible.

## 7. Figma Screen Mapping

The S2 Product page should contain final visual specifications for:

### Browser Extension

-   Injected entry on supported job pages
-   Normal captured/screened state
-   Things to Review
-   Apply/pending state
-   Application confirmation
-   Platform-applied evidence state
-   Track success
-   Track/error recovery
-   Authentication-required state where applicable

### Web

-   Home
-   Applications list S2 delta
-   Application Detail S2 delta
-   New Application S2 delta
-   Analytics

### Design System

-   S2 semantic feedback
-   Finding Item
-   Metric Card
-   Analytics primitives
-   Extension compact variants
-   loading/error/empty patterns
-   focus states

Not every permutation requires a full Figma screen. Key states must be
visualised; secondary permutations may be specified in Markdown.

## 8. Explicit S2 UI/UX Exclusions

Do not introduce:

-   LinkedIn support
-   generic ATS support
-   ATS autofill
-   auto-apply
-   Saved Jobs
-   top-level Jobs page
-   resume match scoring
-   resume tailoring
-   cover-letter generation
-   interview-preparation features
-   predictive analytics
-   source-performance analytics
-   salary analytics
-   skills analytics
-   configurable dashboards
-   reporting/export builders
-   complex funnel visualisations
-   AI-specific visual gimmicks

## 9. Phase 4 Definition of Done

Phase 4 is complete when:

1.  Figma S2 Product screens are complete.
2.  These UI/UX specifications are reviewed and consistent with Figma.
3.  Required Technical Design Deltas are reconciled with Phase 3.
4.  No unresolved UI behaviour requires the coding agent to invent
    product rules.
