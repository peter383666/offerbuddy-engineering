# Sprint 3 Final Scope and Feature Freeze

## Status

This document is the frozen Phase 1 requirements baseline. It publishes the final S3 scope review and supersedes exploratory requirement-analysis discussion where that discussion differs from this final boundary.

## Product Goal

> Evolve OfferBuddy from primarily an Application Tracker into a candidate-aware Application Assistant and Tracker, while preserving the low-friction Sprint 2 workflow.

Sprint 3 does not pursue automatic job discovery, automatic generation of all materials, or automatic submission. Its product flow is:

```text
Candidate truth
→ understand the opportunity
→ prepare selectively
→ user applies
→ OfferBuddy tracks
```

The governing product principle is:

> Apply broadly where appropriate; prepare deeply where it matters.

## Must Have

### Candidate Profile

OfferBuddy must provide a persistent, user-owned factual source of truth covering at least:

- experience;
- education;
- projects;
- skills;
- certifications and relevant qualifications;
- basic candidate information.

AI-generated candidate claims must be supported by Candidate evidence:

> No evidence → no AI claim.

### Reusable Role/Base Resumes

The user must be able to maintain multiple long-lived, reusable Role Resumes (represented as Base Resumes by the frozen technical/API model) for different role directions. They share one Candidate Profile but may emphasise different facts. A Role/Base Resume must not change automatically for every Job.

### Job Match Analysis

Focused Preparation must support an explainable analysis based on Candidate Profile and Job Intelligence, including:

- strong matches;
- partial matches;
- gaps;
- supporting evidence;
- important requirements.

A score may be presented, but explanation is more important than the number. Match does not decide whether the user should apply and must not represent hiring probability.

### Preparation Path

The optional Focused Path is:

```text
Prepare with OfferBuddy
→ persistent Job Preparation
→ Match
→ Resume
→ Cover Letter
→ things worth reviewing
→ open original Job
→ user applies
→ existing Sprint 2 Application tracking
```

Preparation is not a Saved Jobs product and does not create an Application implicitly.

### Job-specific Resume Tailoring

The user may explicitly select **Tailor for This Job**. Tailoring uses the Candidate Profile, selected Base Resume, Job, and Match where available. Generation must be an explicit user action and must remain truthful to Candidate evidence.

### Job-specific Cover Letter

The user may explicitly generate a Cover Letter from Candidate, Job, and relevant Match/Resume context. It is optional and is not required for every Job.

### Application Material Snapshot

After the user applies, OfferBuddy must be able to retain the resume actually used and, where applicable, the Cover Letter actually used. Application Detail must be able to answer what materials were submitted at that time.

### Preserve the Sprint 2 Fast Path

The existing low-friction path is the highest-priority regression requirement:

```text
Job site
→ user applies
→ Extension records
→ Application
→ tracking and Analytics
```

Sprint 3 Preparation remains optional. Candidate completion, Match, generated artefacts, AI availability, and Admin availability must not become prerequisites for Application recording.

## AI Governance — Must-have Core

### Provider and Model Configuration

Job Intelligence, Job Match, resume capabilities, and Cover Letter generation must consume backend capability abstractions whose provider/model selection can be configured without business modules selecting providers directly.

### Runtime Feature Control

Authorised Admin operations must be able to enable or disable Match, resume generation, and Cover Letter generation without redeploying the application.

### Secrets

Provider credentials must not appear as plaintext in Admin UI or ordinary database configuration.

### Basic Monitoring

Operational monitoring must cover, per capability where applicable:

- request, success, and failure counts;
- latency;
- provider/model;
- token usage when available;
- estimated cost where practical;
- last safe failure detail.

Monitoring must respect frozen privacy and AI-safety boundaries.

## Dynamic Reference Data

### Sponsorship Employer Signal

The minimum Sponsor capability is required because it helps identify opportunities worth additional attention. The Extension may show a known sponsorship-related employer signal alongside eligibility information, but the signal must never imply that a specific Job guarantees sponsorship.

### Operational Dataset Refresh

Authorised Admin operations must support the approved official-data lifecycle:

```text
external source/import
→ working dataset validation and limited correction
→ publish/activate
→ cache or snapshot refresh
```

Failure must leave the last known good published dataset active. Operational create/edit/delete-or-disable behaviour applies only to the working/import dataset and must not bypass publication or mutate the active published version in place.

## Should Have

The following are valuable but must not block the Sprint 3 feature-freeze outcome:

- DOCX resume export; PDF remains the core export;
- useful resume staleness guidance without a complex field-level dependency graph;
- warning when user-authored artefact content is not supported by Candidate Profile;
- richer AI cost analytics beyond basic monitoring.

## Best Effort

LinkedIn Extension support is best effort. SEEK and Indeed remain supported from Sprint 2. A LinkedIn Site Adapter must use the existing isolated adapter boundary and fail safely, but LinkedIn DOM, policy, or page-behaviour constraints must not delay the core Sprint 3 outcome.

## Explicitly Out of Scope

Sprint 3 does not include:

- Auto Apply, full application form completion, screening-question automation, automatic submission, or ATS workflow automation;
- proprietary job crawling, an OfferBuddy job feed, recommendation engine, or job-discovery product;
- mock/voice interviews, interview scoring, or an AI interview platform;
- learning plans, courses, quizzes, or a learning platform;
- ATS gaming, claimed ATS-pass guarantees, or hiring prediction;
- a drag-and-drop Resume Designer, arbitrary layout engine, or large template catalogue;
- automatic generation of unlimited Base Resumes;
- a Saved Jobs product with favourites, folders, collections, watchlists, or job CRM behaviour;
- an AI experimentation platform, prompt A/B testing, fine-tuning platform, vector-database platform, agent framework, or multi-agent orchestration;
- local model/GPU infrastructure or local LLM deployment. Provider abstraction must only preserve future compatibility.

The user remains the final decision-maker and submitter.

## Feature-freeze Boundary

Completion of Sprint 3 establishes the Major Feature Freeze. Attractive additional workflows remain Future Backlog rather than new implementation scope. Subsequent work prioritises:

- quality and bug fixing;
- UX polish and accessibility;
- performance and security;
- testing and deployment;
- observability;
- Chrome Web Store quality;
- documentation and portfolio presentation;
- real-world OfferBuddy usage.

## Capability Map

```text
Candidate layer
  Candidate Profile
  → reusable Role/Base Resumes

Opportunity layer
  Captured Job
  → Job Intelligence
  → eligibility and Sponsor signals
  → Job Match

Preparation layer
  Prepare with OfferBuddy
  → selected Role/Base Resume
  → optional Tailored Resume
  → optional Cover Letter
  → things worth reviewing
  → open original Job

Tracking layer
  User applies
  → existing Sprint 2 Application
  → material snapshots
  → status tracking and Analytics

Governance layer
  AI provider abstraction
  runtime configuration and feature controls
  monitoring
  dynamic reference data
```

## Delivery Boundary

Sprint 3 is large enough that each GitHub Issue must represent one reviewable delivery unit and each coding checkpoint must be smaller again. Candidate, Preparation, resume, Extension, Sponsor, and AI capabilities must not be implemented as monolithic Issues. The approved decomposition is maintained in the [Sprint 3 Delivery Plan](../../../delivery/s3/sprint-plan.md).
