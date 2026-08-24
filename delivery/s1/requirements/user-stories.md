# Sprint 1 User Stories

## Purpose

These stories describe the final Sprint 1 behaviour. They are an implementation-aligned record, not the original backlog.

## Authentication

### US-001 — Sign in with Google

As a job seeker, I can sign in with Google so that I can use OfferBuddy without creating another password.

Delivered acceptance criteria:

- The login page starts the backend Google OAuth/OIDC flow.
- Successful authentication creates or identifies one local user from the Google subject.
- The backend establishes a server-side session and redirects to the frontend.
- Protected APIs return `401` when no valid session exists.
- A user can access only applications owned by that user.

### US-002 — Maintain and end a session

As a signed-in user, I remain authenticated across page refreshes and can sign out when finished.

Delivered acceptance criteria:

- The browser sends the `JSESSIONID` cookie with API requests.
- `GET /api/v1/users/me` restores the frontend's authenticated state.
- Logout invalidates the server session and expires the session cookie.
- State-changing requests require Spring Security CSRF protection.

## Job Capture

### US-003 — Parse a job URL

As a job seeker, I can submit a job URL so that OfferBuddy can prepare a draft without requiring me to copy every field.

Delivered acceptance criteria:

- The backend accepts a non-blank URL through `POST /api/v1/jobs/parse`.
- Unsupported or unavailable sources return a defined error response.
- The backend fetches available page content and asks the configured AI provider for structured fields.
- The response uses the implemented job fields, including `responsibilities` and `requirements`.
- Parsing does not persist a job or application.

### US-004 — Review or enter job information

As a job seeker, I can correct a parsed draft or enter job information manually so that I control what is saved.

Delivered acceptance criteria:

- Parsed values populate the application form and remain editable before saving.
- Manual entry is available without a successful parse.
- Company name, job title, and applied date are required for creation.
- Optional source and job-detail fields may be absent.

## Application Tracking

### US-005 — Create an application

As a job seeker, I can save reviewed job information as an application.

Delivered acceptance criteria:

- Creation persists a job and application in one transaction.
- A new application starts in `APPLIED` status.
- A repeated application matching the implemented duplicate rules returns `409 DUPLICATE_APPLICATION`.
- The response contains the created application and nested job details.

### US-006 — Find applications

As a job seeker, I can browse and narrow my applications so that the tracker remains useful as the list grows.

Delivered acceptance criteria:

- The list is server-paginated and scoped to the authenticated user.
- Search supports company name and job title.
- Filtering supports the current status.
- Sorting supports `appliedAt`, `createdAt`, `updatedAt`, and `companyName` with ascending or descending direction.
- The response provides page, size, total elements, and total pages.

### US-007 — View and maintain an application

As a job seeker, I can inspect and update an application as its current state changes.

Delivered acceptance criteria:

- Detail lookup uses the public application UUID and user ownership.
- The user can edit status, applied date, and notes.
- The user can update status through the status endpoint.
- The user can delete an owned application.
- Another user's identifier does not expose that user's data.

### US-008 — Use the home page

As a job seeker, I can start capture and see recent applications from the home page.

Delivered acceptance criteria:

- The home page provides AI-assisted and manual capture paths.
- It displays recent applications using the application list API.
- It does not depend on a separate dashboard or analytics API.

## Not Delivered in Sprint 1

Persisted parsing progress, application status history, analytics, browser extension capture, resume and cover-letter generation, and interview tooling remain future work.
