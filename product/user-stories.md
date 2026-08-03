# MVP User Stories

## Purpose

This document defines the initial user stories for the OfferBuddy MVP.

The stories focus on replacing the current spreadsheet-based job application tracking workflow with a usable web application.

Each user story may later be converted into one or more GitHub Issues for implementation.

## AI Output Principle

AI-generated values are draft suggestions.

No AI-generated job information becomes a confirmed application record until the user reviews and saves it.

---

## User Roles

### Registered User

A registered user is a job seeker who uses OfferBuddy to save job opportunities, track applications, and review application activity.

The MVP supports only individual job seekers. Recruiter, administrator, and team-based workflows are outside the current scope.

---

# Epic 1 — Authentication

## US-001 — Sign In with Google

**As a job seeker,**
I want to sign in to OfferBuddy using my Google account
so that I can access the application quickly without creating another password.

### Acceptance Criteria

* The user can select a **Continue with Google** button from the login page.
* The user is redirected to Google's authentication and consent flow.
* After successful authentication, the user is redirected back to OfferBuddy.
* A new OfferBuddy user record is created automatically during the user's first successful login.
* An existing user is matched using the Google account identity.
* The user's email address, display name, and profile image may be stored when provided by Google.
* The same Google account must not create duplicate OfferBuddy users.
* The user can access only their own job application records.
* If authentication fails or is cancelled, the user remains logged out and sees an appropriate error message.

---

## US-002 — Maintain an Authenticated Session

**As a signed-in user,**
I want OfferBuddy to remember my authenticated session
so that I do not need to sign in again whenever I refresh the page.

### Acceptance Criteria

* The user remains signed in after refreshing the browser.
* The backend validates the user's authentication before returning protected data.
* Unauthenticated users cannot access protected application pages or APIs.
* Expired or invalid sessions redirect the user to the login page.
* Authentication credentials must not be exposed in page URLs or browser logs.

---

## US-003 — Sign Out

**As a signed-in user,**
I want to sign out of OfferBuddy
so that other people using the same device cannot access my application records.

### Acceptance Criteria

* The user can select a sign-out action from the application.
* The current OfferBuddy session is invalidated.
* The user is redirected to the login page.
* Protected pages and APIs cannot be accessed after sign-out.
* Signing out of OfferBuddy does not need to sign the user out of their Google account.

---

# Epic 2 — AI-Assisted Job Capture

## US-004 — Submit a Job Advertisement URL

**As a job seeker,** I want to paste a job advertisement URL
so that OfferBuddy can help me capture the opportunity without copying every field manually.

### Acceptance Criteria

- The user can submit a valid HTTP or HTTPS URL.
- The system rejects malformed or unsupported URLs with a clear message.
- The original URL is preserved.
- The user can continue with manual entry if the URL cannot be processed.
- Submitting a URL does not immediately create a final application record.

---

## US-005 — Extract Job Information Using AI

**As a job seeker,** I want OfferBuddy to extract key information from the job advertisement
so that I can record the opportunity more quickly.

### Acceptance Criteria

- The system attempts to retrieve relevant content from the submitted job URL.
- Retrieved content is sent to the configured AI extraction provider.
- The system requests a structured response.
- The extraction may include:
  - Job title
  - Company name
  - Location
  - Employment type
  - Salary information
  - Job description
  - Skills or keywords
  - Source platform
- Missing information does not cause the entire extraction to fail.
- Invalid AI responses are handled safely.
- The user sees an appropriate message if extraction cannot be completed.
- AI extraction failure does not prevent manual entry.

---

## US-006 — Review Extracted Job Information

**As a job seeker,** I want to review and edit AI-extracted information
so that inaccurate information is not saved to my application records.

### Acceptance Criteria

- Extracted values are displayed in an editable form.
- The user can change any extracted value.
- The user can add missing values.
- The system clearly indicates that the information was automatically extracted.
- The system does not treat the AI response as final until the user confirms it.
- Required fields are validated before saving.
- The original job advertisement URL remains visible.

---

## US-007 — Enter Job Information Manually

**As a job seeker,** I want to enter job information manually
so that I can still save an opportunity when automatic extraction is unavailable.

### Acceptance Criteria

- Manual entry is available without successful AI extraction.
- The user can enter the required job and company information.
- The user can save the original URL even when parsing fails.
- The system records that manual entry was used where appropriate.
- A failed extraction does not block application creation.

---

## US-008 — View Parsing Progress

**As a job seeker,** I want to understand whether job information is being processed
so that I know when to wait, retry, or enter information manually.

### Acceptance Criteria

- The interface shows when extraction is processing.
- The interface shows when extraction completes.
- The interface shows when extraction partially succeeds.
- The interface shows when extraction fails.
- The user can retry where appropriate.
- The user can switch to manual entry without waiting indefinitely.
