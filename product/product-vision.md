# Product Vision

## Product

OfferBuddy is a job application tracking product with AI-assisted job capture. It is a deployed portfolio engineering project built around a real job-search workflow, not a mature SaaS platform.

## Problem

Job seekers often split application information across spreadsheets, bookmarks, notes, and job platforms. This makes capture repetitive and makes the current state of an application harder to maintain.

OfferBuddy first addresses that concrete workflow: capture a job, confirm its details, create an application, and keep its status and notes in one place.

## Target User

The initial user is an individual technology professional applying for roles in Australia. Sprint 1 supports individual accounts only; it does not implement recruiter, team, billing, or organisation workflows.

## Delivered in Sprint 1

Sprint 1 provides:

- Google sign-in and a persistent server-managed session
- an authenticated home page and user-owned application data
- job detail extraction from a submitted URL using an external AI provider
- an editable extracted draft and a manual-entry path
- application creation, list, search, status filtering, sorting, and pagination
- application detail, edit, status update, and deletion
- PostgreSQL persistence
- a deployed HTTPS production application

AI output is untrusted draft data. A user reviews the extracted fields and explicitly saves an application; the AI provider does not write to the database.

## Sprint 1 Limitations

- Job-page retrieval depends on what the source website permits and exposes to a server-side HTTP request.
- Parsing is synchronous and can add noticeable latency.
- Some job sites may block retrieval or return content that is incomplete for extraction.
- Manual entry remains available, but it still requires the user to type the job details.
- Sprint 1 stores only the current application status, not status history.
- Sprint 1 has no analytics dashboard or browser extension.

## Product Principles

### Solve the tracking workflow first

The deployed product must remain useful as a job application tracker without depending on future automation.

### Keep the user in control

AI-extracted information is reviewable and editable before persistence. Missing or inaccurate extraction must not silently become confirmed business data.

### Preserve a manual path

Users can create an application manually when job-page retrieval or AI parsing is unsuitable.

### Keep architecture proportional

OfferBuddy uses a modular monolith and a single-host production deployment. Microservices, Kubernetes, and event-streaming infrastructure are not justified by the current scale.

## Future Direction

Future work is separate from the delivered Sprint 1 product. Current high-level direction includes:

- a browser extension to reduce application-capture friction
- separating deterministic content acquisition from AI semantic analysis more clearly
- analytics as a secondary capability
- later evaluation of resume, cover-letter, and interview assistance

These items are candidates for later sprints and are not implemented Sprint 1 functionality.
