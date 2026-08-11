# Sprint 1 – MVP Foundation

## Sprint Information

| Item           | Value       |
| -------------- | ----------- |
| Sprint         | Sprint 1    |
| Start Date     | 2026-08-06  |
| Target Release | v0.2.0      |
| Status         | In Progress |

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

| Workstream | Status | Evidence / next step |
|---|---|---|
| Sprint planning | Completed | Sprint goal, scope, success criteria, and risks recorded here |
| Technical design | Completed | Architecture, API, and database designs aligned for implementation |
| Database design | Completed | Detailed schema design reflected at domain level in `architecture/data-model.md` |
| UI design | Substantially completed | Design system and Home, Applications, and New Application specifications recorded |
| REST API design | Completed | API contract and OpenAPI specification aligned for Sprint 1 |
| Backend implementation | Not started | Begin after the technical-design checkpoint |
| Frontend implementation | Not started | Implement against the reviewed OpenAPI contract |
| Automated testing | Not started | Add backend and frontend foundations with implementation |

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
* Engineering documentation is updated.
* GitHub Release v0.2.0 is published.

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

Each feature will be implemented using the GitHub Flow workflow.

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
