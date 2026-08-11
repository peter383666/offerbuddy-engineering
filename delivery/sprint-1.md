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

Build the first usable MVP of OfferBuddy that completely replaces Excel for daily job application tracking.

---

# Business Goal

Enable users to manage their job applications through OfferBuddy instead of spreadsheets.

At the end of this sprint, OfferBuddy should become the primary daily tool for tracking job applications.

---

# Sprint Scope

## Progress

| Workstream | Status | Evidence / next step |
|---|---|---|
| Sprint planning | Completed | Sprint goal, scope, success criteria, and risks recorded here |
| Database design | Completed | Domain and persistence decisions recorded in `architecture/data-model.md` |
| UI design | Substantially completed | Design system and Home, Applications, and New Application specifications recorded |
| REST API design | Completed | API contract and OpenAPI specification aligned for Sprint 1 |
| Database migration alignment | Ready to implement | Align the initial Flyway schema with the final data and API contracts |
| Backend implementation | Not started | Begin after migration alignment |
| Frontend implementation | Not started | Implement against the reviewed OpenAPI contract |
| Automated testing | Not started | Add backend and frontend foundations with implementation |

The API contract is the frontend/backend integration boundary. Flyway
migrations and backend persistence models must preserve the rules in the data
model without leaking table structure into API responses.

## Product

* User authentication
* Dashboard
* AI Job Parsing
* Job Application management
* Company management (basic)
* Search and filtering
* Application status management

## Engineering

* UI design
* Database design
* REST API design
* Backend implementation
* Frontend implementation
* Unit testing
* Local Docker deployment
* Engineering documentation updates

---

# Out of Scope

The following features are intentionally excluded from Sprint 1.

* Resume Analysis
* Interview Management
* Chrome Extension
* Automated Job Tracking
* AI Interview Assistant
* Email Automation
* Analytics Dashboard
* Public SaaS features

These will be delivered in future phases.

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

The first implementation item must align the Sprint 1 Flyway migration with the
accepted data model and OpenAPI contract before Application persistence code is
built on top of it.

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
