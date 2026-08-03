
# ADR-001: Use a Modular Monolith for the MVP

## Status

Accepted

## Date

3 August 2026

## Context

OfferBuddy is initially being developed by one engineer.

The first release focuses on a small number of closely related business capabilities:

* Google authentication
* AI-assisted job information extraction
* Job application recording
* Application status tracking
* Search and filtering
* Basic application statistics

The product is initially intended for personal use.

It may later evolve into a multi-user SaaS platform, but the immediate goal is to deliver a usable MVP quickly, use it in a real job search workflow, and improve it based on actual experience.

The system requires clear business boundaries, but it does not currently require independently deployed services.

A microservices architecture would introduce additional complexity, including:

* Multiple deployable applications
* Inter-service communication
* Distributed transactions
* Service discovery
* Network failure handling
* Centralised logging
* Distributed tracing
* More complex local development
* More complex testing
* More expensive deployment
* More difficult debugging

This complexity would not provide sufficient value during the MVP stage.

---

## Decision

OfferBuddy will use a modular monolith architecture for the MVP.

The backend will be deployed as one Spring Boot application.

The application will contain clearly separated business modules with controlled dependencies.

Initial modules may include:

```text
com.offerbuddy
├── auth
├── user
├── company
├── job
├── application
├── parsing
├── dashboard
├── integration
└── shared
```

The system will use one primary PostgreSQL database.

The React frontend will remain a separate web application communicating with the backend through REST APIs.

---

## Definition

A modular monolith is a single deployable backend application whose internal business areas are organised into explicit modules.

It combines:

* The operational simplicity of a monolith
* The internal separation of a modular architecture

It does not mean that all code can depend on all other code.

Modules should have clear responsibilities and controlled interaction.

---

## Module Responsibilities

### Authentication

Responsible for:

* Google authentication integration
* Current user resolution
* Authenticated session handling
* Authentication failure handling

### User

Responsible for:

* Local user profiles
* External identity relationships
* User account state

### Company

Responsible for:

* Company records
* Basic company matching
* Company-related business rules

### Job

Responsible for:

* Confirmed job information
* Job description
* Job source information
* Job skills
* Job-company relationship

### Application

Responsible for:

* Confirmed application records
* Application lifecycle
* Application status changes
* Application notes
* User ownership validation
* Duplicate application warnings

### Parsing

Responsible for:

* Job URL parsing workflow
* Job page content preparation
* AI extraction coordination
* AI response validation
* Manual fallback
* Parsing status handling

### Dashboard

Responsible for:

* Application totals
* Status summaries
* Interview rate
* Offer rate
* Time-based application statistics

### Integration

Responsible for external service communication, including:

* Google identity services
* Job advertisement websites
* AI providers

### Shared

Responsible only for genuinely cross-cutting technical concerns, such as:

* Common error structures
* Auditing
* Time abstraction
* Request correlation
* Shared infrastructure utilities

The shared module must not become a collection of unrelated business logic.

---

## Module Boundary Principles

The backend should follow these principles:

* Each business concept should have one primary owning module.
* Modules should expose intentional interfaces.
* Modules should not access another module's internal repository directly.
* External provider DTOs should not leak into business modules.
* Business logic should not be placed in controllers.
* Database entities should not be returned directly through APIs.
* Cross-module calls should remain explicit.
* Cyclic module dependencies should be avoided.
* Shared code should be kept minimal.
* Module boundaries should reflect business responsibilities rather than technical layers only.

---

## Example Dependency Direction

A possible dependency direction is:

```text
API Controller
        ↓
Application Service
        ↓
Domain Logic
        ↓
Repository Interface
        ↓
Persistence Implementation
```

For AI-assisted job capture:

```text
Application API
        ↓
Parsing Workflow
        ↓
Job Information Extractor Interface
        ↓
AI Provider Implementation
```

The parsing module may coordinate AI extraction, but the AI integration implementation should remain isolated from the application lifecycle.

---

## Database Decision

The modular monolith will initially use one PostgreSQL database.

Each module may own its relevant tables logically.

Examples:

```text
auth
    external_identity

user
    user_account

company
    company

job
    job

application
    application
    application_status_history

parsing
    job_parsing_request
```

A separate database per module is not required.

Cross-module database access should still be controlled through application code rather than allowing every module to query every table directly.

---

## Transaction Model

The modular monolith allows related business operations to use local database transactions.

For example, when recording an application, the backend may perform the following within one transaction:

```text
Find or create Company
        ↓
Create Job
        ↓
Create Application
        ↓
Create initial status history
        ↓
Link parsing request
        ↓
Commit
```

This avoids introducing distributed transaction problems during the MVP.

---

## Consequences

### Positive

* Faster MVP delivery
* Lower infrastructure cost
* Easier local development
* Simpler deployment
* Easier debugging
* Simpler transaction management
* Easier integration testing
* Clearer ownership for a solo engineer
* Supports incremental module development
* Allows future extraction of services if justified

### Negative

* The backend is deployed as one unit.
* One module can affect the whole backend if boundaries are poorly maintained.
* The application may become tightly coupled if module rules are ignored.
* Independent module scaling is not available.
* Deployment of one change requires redeploying the backend application.
* Strong discipline is required to prevent the monolith becoming unstructured.

---

## Risks

### Risk: The modular monolith becomes a big ball of mud

Mitigation:

* Maintain business-oriented modules.
* Avoid cyclic dependencies.
* Keep controllers thin.
* Keep business logic inside the owning module.
* Review module boundaries during pull requests.
* Add architecture tests later if useful.

### Risk: Premature design for future microservices

Mitigation:

* Do not create distributed infrastructure now.
* Do not introduce messaging without a real asynchronous requirement.
* Do not create unnecessary network boundaries.
* Keep module interfaces simple and application-owned.

### Risk: Future SaaS scale exceeds the architecture

Mitigation:

* Measure actual bottlenecks.
* Scale the backend vertically first.
* Add database indexes and query optimisation.
* Add multiple backend instances if needed.
* Extract a module into a service only when there is a clear operational or organisational reason.

---

## Alternatives Considered

### Traditional Layered Monolith

Example:

```text
controller
service
repository
entity
```

This structure is simple, but over time it often groups code by technical type rather than business capability.

As the product grows, related business logic becomes distributed across large technical packages.

A business-oriented modular structure provides clearer long-term boundaries.

### Microservices

Microservices were rejected for the MVP because:

* The project is developed by one engineer.
* The business scope is still evolving.
* Independent deployments are not required.
* Independent scaling is not required.
* The operational cost would be higher.
* Distributed failure handling would slow delivery.
* Local development would become more difficult.
* The architecture would repeat complexity experienced in earlier portfolio work.

### Serverless Functions

A fully serverless backend was not selected because:

* The product contains connected transactional workflows.
* Spring Boot aligns better with the engineering portfolio.
* Splitting workflows into many functions could make the system harder to understand.
* Provider-specific serverless constraints are not currently justified.

Serverless components may still be used selectively in future versions.

---

## Future Evolution

The modular monolith does not prevent future service extraction.

A module may become a separate service when there is a demonstrated need, such as:

* Independent scaling
* Independent deployment frequency
* Heavy background processing
* Different availability requirements
* Clear ownership by a separate team
* Strong security isolation
* Significant operational boundaries

Possible future candidates may include:

* Job parsing
* Notifications
* Email processing
* Automated job application execution
* AI interview services

Service extraction should be driven by evidence rather than speculation.

---

## Validation

This decision should be reviewed when one or more of the following becomes true:

* Multiple engineering teams require independent deployment.
* One module requires significantly different scaling.
* One module causes unacceptable deployment coupling.
* Long-running background workloads affect the main application.
* Security or compliance requires stronger isolation.
* Operational evidence shows that the monolith is no longer sufficient.

Until then, the modular monolith remains the preferred architecture.

---

## Related Documents

* `architecture/container-design.md`
* `architecture/data-model.md`
* `technology/tech-stack.md`
* `decisions/ADR-003-ai-assisted-job-extraction.md`
* `decisions/ADR-004-ai-provider-abstraction.md`

---

## Outcome

OfferBuddy will begin as a modular monolith using:

* One React frontend
* One Spring Boot backend
* One PostgreSQL database
* Clear internal business modules
* External integrations isolated behind application-owned interfaces

This structure provides enough engineering discipline for future growth while keeping the MVP practical to build, operate, and improve.
