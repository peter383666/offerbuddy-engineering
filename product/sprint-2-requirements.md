# Sprint 2 Requirements

## Document Status

| Item | Value |
| --- | --- |
| Phase | Phase 1 — Requirement Analysis |
| Status | Completed and approved |
| Purpose | Authoritative Sprint 2 product requirements |

This document defines what Sprint 2 must achieve and why. It does not define architecture, technical design, UI layout, implementation tasks, or delivery sequencing.

## Product Motivation

Sprint 1 delivered a usable OfferBuddy MVP, but real usage showed that recording an Application still adds too much friction to the normal job-search workflow. The user may need to copy a URL, switch applications, wait for AI parsing, or reconfirm information already visible on the job page.

Sprint 2 exists primarily to reduce that friction.

The primary product goal is:

> Make OfferBuddy practical enough to use during every normal job application.

The guiding requirement-level principle is:

> **Fast capture, asynchronous enrichment.**

This principle describes the required user outcome, not an implementation mechanism.

## Scope Priorities

1. Browser Extension and lower-friction Job Capture
2. eligibility screening and Job Intelligence
3. basic Application Analytics

The Browser Extension is the primary Sprint 2 feature. Application Analytics is secondary and must not expand into a large business-intelligence project.

## Product Concepts

### Job and Application

- A **Job** contains reusable or shared job-posting information.
- An **Application** is an authenticated user's own application and tracking relationship to a Job.

The Browser Extension adds another ingestion path but does not change these concepts or ownership semantics. Client-provided ownership information is never authoritative.

### Job Capture and Job Intelligence

**Job Capture** obtains reliable facts from a supported job page.

**Job Intelligence** interprets the semantic meaning of the captured job description.

AI should not be required to infer basic page facts that OfferBuddy can capture reliably. Job Capture is critical to saving the Job and Application; Job Intelligence is non-critical enrichment.

## Browser Extension

Sprint 2 introduces the OfferBuddy Browser Extension as part of OfferBuddy, not as a separate product.

The extension must allow a user to capture a Job while browsing a supported recruitment platform and save the detected Job/Application with minimal additional interaction.

### Supported Platforms

The initial supported platforms are:

- SEEK
- Indeed

Sprint 2 does not attempt to support every recruitment website. LinkedIn is explicitly out of scope.

Supported pages may use dynamic rendering, SPA-style navigation, job-detail panels, or changing page structures. When OfferBuddy cannot reliably capture required information, it must fail safely and provide understandable feedback rather than silently saving misleading data.

### Captured Job Facts

For supported job pages, OfferBuddy must capture reliable facts where available, including:

- source platform;
- external job identifier;
- job URL;
- job title;
- company;
- location;
- job description or content required for downstream processing.

### User Outcomes

The extension must provide understandable feedback for:

- successful save;
- already recorded or duplicate capture;
- authentication required;
- extraction or detection failure;
- save failure.

The exact extension interface and interaction layout are deferred to UI/UX Design.

## Eligibility Screening

OfferBuddy must help identify obvious eligibility restrictions before a user spends unnecessary time on a Job.

Relevant information includes:

- Australian citizenship requirements;
- permanent residency requirements;
- security-clearance requirements;
- Australian working-rights requirements;
- visa sponsorship restrictions or availability when stated.

This is a screening and warning capability, not an absolute automated decision about whether the user can apply. Requirements can be ambiguous; for example, “full working rights in Australia” must not automatically be treated as identical to “permanent residency required”.

The product must support outcomes equivalent to:

- no explicit restriction detected;
- eligibility requires review;
- explicit restriction detected;
- unable to determine.

The technical classification method is deferred.

## Core Save and Duplicate Behaviour

- Application ownership must be associated with the authenticated OfferBuddy user and enforced server-side.
- Manipulated extension input must not allow access to or ownership of another user's Applications.
- Repeated capture must not silently create unwanted duplicate Applications.
- Duplicate outcomes must be understandable to the user.
- Existing approved duplicate behaviour remains authoritative until changed through a separate product decision; Sprint 2 requirements do not redefine its detailed algorithm.
- If required Job information cannot be captured reliably, OfferBuddy must not silently create misleading or corrupt Job/Application data.

## AI URL Parsing

Sprint 1 AI URL parsing is not removed in Sprint 2, but it is no longer the preferred acquisition path.

- Browser Extension capture becomes the preferred path.
- AI URL parsing remains a secondary fallback.
- AI URL parsing may remain available through New Application.
- It may be removed from more prominent entry points.
- Sprint 2 makes no major new investment in AI-based webpage acquisition.

The exact UI placement is deferred to UI/UX Design.

## Job Intelligence

After core Job/Application capture, OfferBuddy must be able to produce structured Job Intelligence from the job description, including:

- responsibilities;
- requirements;
- relevant skills;
- a concise factual job understanding or summary where appropriate.

Job Intelligence is non-critical enrichment. OfferBuddy must distinguish completed analysis from incomplete or failed analysis sufficiently to avoid misleading the user.

AI provider selection, prompts, persistence, schemas, retry behaviour, concurrency, and execution design are not Phase 1 decisions.

## Non-Blocking AI Enrichment

Expensive or non-critical AI processing must not block core Application creation.

Required behaviour:

1. core Job/Application capture succeeds independently;
2. the user receives the core save result without waiting for AI analysis;
3. AI Job Intelligence may complete afterward;
4. slow, unavailable, or failed AI does not roll back or invalidate an otherwise valid Application.

The mechanism for asynchronous or non-blocking processing is deferred to Architecture Design and Technical Design after the separately proposed event framework has been reviewed.

Sprint 2 Phase 1 does not require Spring events, Spring `@Async`, LMAX Disruptor, Kafka, RabbitMQ, database-backed events, transactional outbox, event scanning, a particular worker model, or any specific event class.

## Basic Application Analytics

Sprint 2 includes basic Application Analytics as a secondary capability to help users understand job-search activity and outcomes.

Useful views include:

- Application activity over time;
- counts by Application status;
- counts by source platform where available;
- basic conversion indicators such as Application to interview and interview to offer;
- other simple lifecycle or conversion views already consistent with the product.

Sprint 2 does not include a large Analytics/BI expansion. Data storage, caching, projections, event processing, and detailed dashboard design are deferred.

## Authentication and Ownership

The Browser Extension is another OfferBuddy client alongside the Web application.

- Extension operations must be associated with an authenticated OfferBuddy user.
- Unauthenticated users must receive understandable authentication-required feedback.
- Application ownership remains enforced server-side.
- Client input must not determine authoritative ownership.
- Manipulated extension input must not expose another user's Applications.
- Sensitive credentials and secrets must not be unnecessarily exposed to browser clients.
- Existing Web authentication must continue to work.

The extension authentication mechanism, token or session model, credential storage, and OAuth details are deferred to Architecture Design and Technical Design.

## Failure Expectations

### Capture Failure

If required Job information cannot be reliably detected, OfferBuddy must fail safely, avoid silently creating misleading data, and provide understandable feedback.

### Duplicate Capture

Repeated capture must result in understandable duplicate behaviour rather than silently creating an unwanted Application.

### AI Latency and Failure

Core capture/save must not wait for expensive AI enrichment. AI failure must not invalidate successful core Application creation.

### Job-Site Change

If SEEK or Indeed changes prevent reliable capture, the extension must fail safely and inform the user.

### Authentication Failure

Authentication problems must produce understandable authentication-required feedback rather than an unexplained save failure.

SLA targets, timeout values, retry counts, delivery guarantees, exactly-once semantics, and recovery algorithms are not defined in Phase 1.

## Non-Functional Requirements

### Usability

The Browser Extension must materially reduce Application-recording friction compared with the Sprint 1 workflow.

### Responsiveness

Core capture/save must not depend on the completion of expensive AI processing. Phase 1 does not define numerical latency targets.

### Reliability

Optional or downstream Job Intelligence failures must not invalidate valid core Job/Application data.

### Security

Authentication and ownership must remain enforced. Client input must not determine authoritative ownership.

### Maintainability

SEEK and Indeed support must produce consistent OfferBuddy outcomes rather than separate, inconsistent product experiences.

### Extensibility

Sprint 2 must not prevent later support for additional recruitment platforms or job-assistance capabilities. Those capabilities are not designed or committed here.

### Privacy and Least Privilege

The Browser Extension must access only what is reasonably required for the approved functionality. Exact browser permissions are deferred to technical design.

## Explicitly Out of Scope

Sprint 2 does not deliver:

- LinkedIn support;
- support for every recruitment platform;
- automated job application or Auto Apply;
- automatic application-form submission;
- Cover Letter generation;
- resume tailoring or automated resume generation;
- candidate/Job match scoring;
- complex AI workflow or orchestration;
- Kafka or RabbitMQ;
- mandatory transactional outbox;
- dead-letter queue infrastructure;
- exactly-once processing guarantees;
- a distributed event platform;
- microservice decomposition;
- Kubernetes;
- large Analytics/BI dashboard expansion.

## Future Direction — Not Sprint 2 Scope

The Browser Extension may later evolve from a capture tool into a broader job-page companion. Future ideas include:

- Cover Letter generation;
- resume tailoring;
- candidate/Job match analysis;
- recruiter-message assistance;
- additional job platforms;
- application-form assistance.

These are future product ideas only and have no Sprint 2 implementation commitment.

## Deferred and Unresolved Decisions

The following are intentionally deferred and must not be inferred from these requirements.

### Architecture Design / Technical Design

- Browser Extension implementation, including Manifest, content scripts, service worker, popup, messaging, SPA handling, DOM strategy, platform extraction strategy, and project structure.
- Initial browser, supported versions, and extension distribution approach.
- Extension authentication and credential mechanism.
- Event implementation, including the proposed prior framework, Disruptor, outbox, workers, queues, and concurrency.
- AI provider, prompts, persistence, execution, retry, and concurrency.
- Analytics storage, projection, caching, or event-processing design.

### UI/UX Design

- Exact extension interface and feedback presentation.
- Exact New Application placement for fallback AI URL parsing.
- Detailed Analytics presentation.

### Future Product Scope

- Additional recruitment platforms beyond SEEK and Indeed.
- Broader job-page companion capabilities.

## Requirement Boundary

Phase 1 confirms WHAT and WHY. It deliberately does not define architecture, components, modules, package or class names, database or event schemas, API endpoints, queue technology, threading, Redis use, deployment topology, DOM selectors, Manifest implementation, AI provider, or AI prompts.

## Related Documents

- [Product Vision](product-vision.md)
- [Sprint 1 Product Scope](mvp-scope.md)
- [Sprint 1 User Stories](user-stories.md)
- [Delivery Roadmap](../delivery/roadmap.md)
- [Product Backlog](../delivery/product-backlog.md)
- [Sprint 1 Non-Functional Requirements](../quality/non-functional-requirements.md)
