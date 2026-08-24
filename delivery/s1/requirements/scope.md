# Sprint 1 Product Scope

## Purpose

This document distinguishes the Sprint 1 product that was delivered from ideas that appeared in the original MVP plan but were not part of the final increment.

## Sprint 1 Goal

Replace the core spreadsheet-based tracking workflow with a production web application that can capture job information, create an application, and maintain the user's current application state.

## Delivered in Sprint 1

### Authentication

- Sign in with Google using OAuth 2.0/OpenID Connect
- Create or identify the local OfferBuddy user after successful authentication
- Maintain a server-side authenticated session
- Sign out
- Protect application APIs and isolate records by authenticated user

### Job Capture

- Submit an HTTP or HTTPS job URL for parsing
- Retrieve supported public job-page content from the backend
- Use Google Gemini to extract structured job fields
- Return an editable draft without automatically creating an application
- Continue with manual entry without a successful parse
- Validate required company name, job title, and applied date when saving

Job parsing is a synchronous request. Sprint 1 does not persist parsing jobs or parsing-status history.

### Application Tracking

- Create an application; every new application starts as `APPLIED`
- View a paginated application list
- Search by company name and job title
- Filter by current application status
- Sort by supported application fields
- View application detail
- Edit current status, applied date, and notes
- Update status directly
- Delete an application

Supported statuses are:

- `APPLIED`
- `NO_RESPONSE`
- `INTERVIEW`
- `OFFER`
- `REJECTED`
- `WITHDRAWN`

Sprint 1 stores the current status only. It does not store a status-change history.

### Home

- Start AI-assisted or manual application capture
- View recent applications
- Navigate to the applications list and application detail

The home page is composed from existing application and user APIs. It is not an analytics dashboard.

### Production Delivery

- PostgreSQL persistence managed through Flyway
- React frontend and Spring Boot backend deployed to AWS EC2
- Nginx and HTTPS
- Independent frontend and backend CI/CD
- Immutable SHA-based deployment and rollback capability
- Health, backup/restore, restart, and recovery verification

## Scope Changes from the Original Plan

The original MVP scope mentioned persisted parsing states, application statistics, a basic dashboard, status history, and broader job editing. Those capabilities were not delivered in Sprint 1.

AI-assisted extraction was delivered, but its production limitations made the distinction between page-content acquisition and AI semantic extraction more important than the original plan suggested.

## Known Limitations

- Server-side page retrieval cannot reliably acquire content from every job site.
- AI parsing may be slow, incomplete, or unavailable.
- Application job details become read-only after creation; Sprint 1 edits status, applied date, and notes only.
- Duplicate application protection rejects a repeated application for the same matched job.
- No frontend unit or component test suite exists yet.

## Future Sprints

The following are not delivered Sprint 1 features:

- Browser extension
- Analytics and application-outcome reporting
- Resume generation or optimisation
- Cover-letter generation
- Interview preparation or mock interviews
- Email status detection
- Automated job applications
- Subscription billing or team accounts
- Microservices, Kafka, or Kubernetes

Sprint 2 direction may prioritise browser-based capture and reduced manual-entry friction. That direction is not a commitment or completed functionality in this document.

## Sprint 1 Result

Sprint 1 satisfies the core product goal: an authenticated user can capture or manually enter job information, create an application, find and maintain applications, and use the workflow in production instead of a spreadsheet.
