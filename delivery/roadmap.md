# Delivery Roadmap

## Purpose

This roadmap records the delivered sequence and current high-level direction. Future items are not committed until they are refined and accepted into a sprint.

## Sprint 0 — Engineering Foundation

**Status:** Completed

Delivered:

- React and Spring Boot foundations
- Docker Compose development infrastructure
- PostgreSQL integration and Flyway
- initial GitHub Actions CI
- product, architecture, quality, operations, and delivery documentation

Related documents:

- [Sprint 0 Plan](sprint-0.md)
- [Sprint 0 Review](sprint-0-review.md)
- [Sprint 0 Retrospective](sprint-0-retrospective.md)

## Sprint 1 — Production Job Application Tracker

**Status:** Implementation and production hardening complete; documentation/release closure in progress.

Delivered:

- Google authentication and server session
- manual and AI-assisted application capture
- create, list, search, filter, sort, and pagination
- application detail, update, status tracking, and deletion
- home page and frontend/backend integration
- PostgreSQL persistence
- production EC2/Nginx/HTTPS deployment
- independent frontend/backend CI/CD
- backup/restore and restart/recovery verification
- immutable-SHA rollback capability

Related documents:

- [Sprint 1 Plan](sprint-1.md)
- [Sprint 1 Review](sprint-1-review.md)
- [Sprint 1 Retrospective](sprint-1-retrospective.md)
- [Sprint 1 Technical Debt](sprint-1-technical-debt.md)

The final Sprint 1 tag/release follows documentation approval and sprint closure.

## Sprint 2 — Lower-Friction Job Capture

**Phase 1 status:** Requirement Analysis completed and approved.

**Phase 2 status:** Architecture Design completed and approved.

**Delivery status:** Sprint 2 functionality is not yet recorded as implemented or delivered.

Confirmed requirement scope:

1. OfferBuddy Browser Extension as the primary Sprint 2 feature
2. initial Job Capture support for SEEK and Indeed
3. eligibility screening with explicit uncertainty rather than absolute automated decisions
4. Browser Extension capture as the preferred path, with AI URL parsing retained as fallback
5. separation of reliable Job Capture from semantic Job Intelligence
6. non-blocking AI enrichment that cannot invalidate successful core Application creation
7. basic Application Analytics as a secondary capability
8. continued server-enforced authentication, Application ownership, and understandable duplicate/failure behaviour

LinkedIn, Auto Apply, Cover Letter generation, resume tailoring, candidate/Job match scoring, complex AI orchestration, distributed event infrastructure, microservices, Kubernetes, and a large Analytics/BI expansion are not Sprint 2 deliverables.

The approved architecture retains the Spring Boot modular monolith and PostgreSQL, adds the Browser Extension/Site Adapter ingestion path, and uses lightweight Business Events for downstream Job Intelligence and Analytics without introducing a message broker. Exact APIs, schemas, Extension credentials, event mechanics, persistence details, and UI/UX remain deferred to later design phases.

See the authoritative [Sprint 2 Requirements](../product/sprint-2-requirements.md) and [Sprint 2 Architecture Design](../architecture/sprint-2-architecture-design.md).

## Later Possibilities

Resume generation/optimisation, Cover Letter generation, candidate/Job match analysis, recruiter-message assistance, interview assistance, email status detection, additional job platforms, application-form assistance, and broader automation remain later product possibilities. They are not implied Sprint 2 commitments.

See [Product Backlog](product-backlog.md).
