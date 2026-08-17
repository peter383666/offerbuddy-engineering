# Sprint 1 – MVP Foundation

## Sprint Information

| Item           | Value       |
| -------------- | ----------- |
| Sprint         | Sprint 1    |
| Start Date     | 2026-08-06  |
| Target Release | v0.2.0      |
| Status         | Implementation complete; release closure in progress |

---

# Overview

Sprint 1 marks the beginning of OfferBuddy as a real product rather than a project foundation.

The objective of this sprint is to deliver the first production-ready MVP that replaces the current Excel-based job application tracker with a modern web application.

This sprint focuses on building a complete end-to-end workflow that can be used every day during the job search.

---

# Sprint Goal

Deliver a usable OfferBuddy MVP that replaces the spreadsheet-based job
application tracking workflow.

---

# Business Goal

Enable users to manage their job applications through OfferBuddy instead of spreadsheets.

At the end of this sprint, OfferBuddy should become the primary daily tool for tracking job applications.

---

# Scope

## Progress

| Workstream | Final status | Evidence |
|---|---|---|
| Sprint planning | Completed | Sprint goal, scope, success criteria, and risks recorded here |
| Technical design | Completed | Architecture and ADRs reconciled with implementation |
| Database design | Completed | PostgreSQL schema documented from Flyway V1/V2 |
| UI design | Completed for Sprint 1 | Login, Home, Applications, New Application, detail, and edit flows implemented |
| REST API design | Completed | Checked-in contract aligned to Spring controllers and DTOs |
| Backend implementation | Completed | Authentication, parsing, application, persistence, security, and health capabilities delivered |
| Frontend implementation | Completed | Authenticated Sprint 1 routes and application workflows delivered |
| Automated testing | Partially completed | Broad backend coverage; frontend lint/build but no unit/component suite |
| Production deployment | Completed | EC2, Nginx, HTTPS, Compose, CI/CD, health, backup/recovery, and rollback verified |

- Google authentication
- Create an Application manually or from reviewed job-parsing results
- View Applications
- Search and filter Applications
- View Application detail
- Update Application data and status
- Delete an Application
- Home page
- Frontend/backend integration
- Automated testing
- Production deployment

---

# Out of Scope

The following features are intentionally excluded from Sprint 1.

- Application analytics
- Application statistics
- Total application count
- Advanced reporting
- AI cover letter generation
- Resume analysis
- Interview management and AI interview assistance
- Automated job application
- Automated job tracking
- Browser extension
- Email automation
- Public SaaS features

These will be delivered in future phases.

---

# Implementation Order

1. Finalize technical design
2. Authentication
3. Application creation
4. Application list and search
5. Application detail
6. Home page
7. Frontend/backend integration
8. Sprint-level testing
9. Production deployment

This order aligns with GitHub Issues #14 through #22.

---

# Success Criteria

Sprint 1 is considered successful when all of the following conditions are met.

## Product

* User can sign in.
* User can create an application.
* User can view applications.
* User can update applications.
* User can delete applications.
* Data is stored in PostgreSQL.
* The application is usable on a daily basis.

## Engineering

* Backend and frontend are integrated.
* Docker deployment works locally.
* The Sprint 1 application is deployed to the selected production environment.
* Core unit tests pass.
* Engineering documentation is updated before release closure.
* GitHub Release v0.2.0 is published after documentation approval and final tagging.

## Business

The Excel spreadsheet is no longer used for daily job tracking.

OfferBuddy becomes the primary application tracking tool.

---

# Deliverables

* Running MVP application
* Backend REST API
* React frontend
* PostgreSQL database
* Flyway migrations
* Docker deployment
* Updated engineering documentation
* GitHub Release v0.2.0
* Sprint Review
* Sprint Retrospective

---

# Definition of Done

A Sprint 1 task is complete only when all of the following are satisfied.

* Requirements completed
* UI implemented
* Backend implemented
* Database migration completed
* Tests passed
* Documentation updated
* Code reviewed
* Git committed
* Pull request merged

---

# Sprint Backlog

The Sprint Backlog will be managed through GitHub Issues.

Feature work was integrated through the `release` branch, verified, and promoted to the stable `main` baseline. Production deployment selects an explicit immutable SHA/version.

---

# Risks

* Scope expansion beyond MVP
* UI redesign causing implementation delays
* Database changes affecting existing migrations
* Third-party authentication integration
* Time constraints due to part-time work

---

# Expected Outcome

At the end of Sprint 1, OfferBuddy should be capable of replacing Excel for daily job application tracking and provide a stable foundation for future AI-powered features.

---

# Final Outcome

Sprint 1 delivered the production application described by the product and engineering goals. The implemented system supports the core daily tracking workflow and is live at [offerbuddy.io](https://offerbuddy.io).

The repository cannot independently prove that the user has permanently stopped using the previous spreadsheet, so that business-behaviour criterion remains a product validation question rather than an engineering claim.

Implementation and production hardening are complete. The final tag/GitHub Release remains intentionally pending until the documentation, Sprint Review, Retrospective, and closure decision are approved.

---

# Related Documents

* [Sprint 1 Technical Debt](sprint-1-technical-debt.md) — developer-facing debt, contract drift, and accepted design decisions from the Sprint 1 code review
* [Sprint 1 Review](sprint-1-review.md) — what Sprint 1 delivered and the acceptance result
* [Sprint 1 Retrospective](sprint-1-retrospective.md) — how delivery went and what was learned
* [Product Backlog](product-backlog.md) — work not committed to the current sprint
* [API Design](../architecture/api-design.md)
* [Data Model](../architecture/data-model.md)
