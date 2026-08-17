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

## Sprint 2 Direction

Sprint 2 is not fully designed or committed in this document.

Current direction:

1. Browser extension capture as the primary product candidate
2. reduced manual application-entry friction
3. clearer separation of deterministic content acquisition from AI semantic analysis
4. analytics as a secondary product candidate
5. selected Sprint 1 technical-debt work based on risk and capacity

Browser extension and analytics are future work; neither exists in Sprint 1.

## Later Possibilities

Resume generation/optimisation, cover-letter generation, interview assistance, email status detection, and broader automation remain later product possibilities. They are not implied Sprint 2 commitments.

See [Product Backlog](product-backlog.md).
