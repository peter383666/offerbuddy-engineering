# OfferBuddy API Design

Version: 1.0 (Sprint 1 Draft)

---

# 1. Purpose

This document defines the REST API contract for the OfferBuddy Sprint 1 MVP.

The objective is to establish a stable API contract between the frontend and backend before implementation begins.

This document defines:

- API scope
- Business domains
- REST conventions
- Endpoint responsibilities
- Request and response contracts
- Authentication requirements
- Validation principles
- Error handling
- OpenAPI alignment

Implementation details such as Java classes, DTOs, services, repositories and persistence are intentionally excluded.

---

# 2. Sprint 1 Scope

Sprint 1 only covers the MVP.

Only APIs required by the approved Sprint 1 UI are included.

Features planned for later sprints are intentionally excluded.

---

# 3. Business Domains

The Sprint 1 backend consists of the following API domains.

## 3.1 Authentication

Responsibilities

- Google Sign In
- Authenticate user
- Retrieve current user
- Protect secured endpoints

---

## 3.2 Job Parsing

Responsibilities

- Accept a job URL
- Parse job information
- Return extracted data
- Allow frontend review before saving

Job parsing does not create an application.

Creating an application is handled by the Application domain.

---

## 3.3 Application

### Responsibilities

- Create application
- Update application
- Delete application
- Retrieve application list
- Retrieve application detail
- Search applications
- Filter applications
- Change application status

This is the primary business domain of Sprint 1.


### Job Immutability

A Job represents a job advertisement captured by OfferBuddy.

Once persisted, Job information is read-only in Sprint 1.

Users may view Job information through Application responses, but they
cannot modify an existing Job through the Application API.

Editable Application fields are limited to:

- `status`
- `appliedAt`
- `notes`

Job fields such as company name, job title, location, work type, work mode,
source URL and job description are not editable through Application update
operations.

When creating an Application, the backend may identify an existing Job or
create a new Job from the supplied parsed job information.

Multiple users may have separate Applications referencing the same Job.
Changes to one user's Application must not affect another user's Application
or the underlying Job.

The Application API represents Job data as a nested `job` object. This keeps
the API boundary aligned with the Job and Application domain boundary.

---

# 4. Home Page API Usage

The Home page is a composition of existing business APIs.

It does not introduce a dedicated Home API.

The current Sprint 1 Home page requires:

- Parse Job
- Recent Applications
- Current User

Future dashboard analytics are outside the Sprint 1 scope.

---

# 5. Out of Scope

The following features are excluded from Sprint 1.

- Application analytics
- Weekly statistics
- Monthly statistics
- Dashboard charts
- AI recommendations
- Resume optimisation
- AI interview
- Job crawler
- Auto apply
- Browser Extension API
- Mobile API
- Notification API
- Email tracking
- Team collaboration

---

# 6. API Design Principles

## 6.1 Base Path and Versioning

All Sprint 1 APIs use a versioned base path:

```text
/api/v1
```

Examples:

```text
/api/v1/applications
/api/v1/users/me
/api/v1/jobs/parse
```

API versioning is introduced from the first public API contract to support future backward-compatible evolution.

---

## 6.2 REST Resource Naming

API resources use nouns rather than verbs.

Resource names use lowercase plural nouns where applicable.

Preferred examples:

```text
GET /api/v1/applications
GET /api/v1/applications/{applicationId}
GET /api/v1/users/me
```

Avoid:

```text
/getApplications
/createApplication
/updateApplication
```

Actions that do not naturally map to CRUD operations may use a meaningful sub-resource or action-oriented endpoint when appropriate.

Example:

```text
POST /api/v1/jobs/parse
```

Job parsing is treated as an explicit business operation rather than a persistent CRUD resource.

---

## 6.3 HTTP Methods

The API uses standard HTTP methods according to resource semantics.

| Method   | Usage                                                            |
| -------- | ---------------------------------------------------------------- |
| `GET`    | Retrieve resources                                               |
| `POST`   | Create a resource or execute a non-idempotent business operation |
| `PUT`    | Replace or update a complete editable resource representation    |
| `PATCH`  | Perform a partial update                                         |
| `DELETE` | Delete a resource                                                |

Examples:

```text
POST   /api/v1/applications
GET    /api/v1/applications
GET    /api/v1/applications/{applicationId}
PUT    /api/v1/applications/{applicationId}
PATCH  /api/v1/applications/{applicationId}/status
DELETE /api/v1/applications/{applicationId}
```

---

## 6.4 JSON Naming Convention

JSON property names use `camelCase`.

Example:

```json
{
  "applicationId": "01JABC123",
  "companyName": "Atlassian",
  "jobTitle": "Software Engineer",
  "appliedAt": "2026-08-08"
}
```

Database naming conventions must not leak into the API contract.

For example, a database column named:

```text
company_name
```

is exposed through the API as:

```text
companyName
```

---

## 6.5 Resource Identifiers

Public API resources use domain identifiers rather than exposing database implementation details where practical.

Example:

```text
/applications/{applicationId}
```

The API contract refers to the identifier as:

```text
applicationId
```

The frontend must treat resource identifiers as opaque values and must not derive business meaning from their internal format.

---

## 6.6 Date and Time Format

The API uses ISO 8601 formats.

Date-only values use:

```text
YYYY-MM-DD
```

Example:

```json
{
  "appliedAt": "2026-08-08"
}
```

Timestamp values use ISO 8601 with timezone information.

Example:

```json
{
  "createdAt": "2026-08-08T04:30:00Z"
}
```

Backend persistence timezone and timezone conversion are implementation concerns and must not change the external API contract.

---

## 6.7 HTTP Status Codes

The API uses standard HTTP status codes.

| Status                      | Usage                                                                   |
| --------------------------- | ----------------------------------------------------------------------- |
| `200 OK`                    | Successful retrieval or update                                          |
| `201 Created`               | Resource successfully created                                           |
| `204 No Content`            | Successful operation with no response body                              |
| `400 Bad Request`           | Invalid request syntax or validation                                    |
| `401 Unauthorized`          | Authentication is missing or invalid                                    |
| `403 Forbidden`             | Authenticated user is not authorised                                    |
| `404 Not Found`             | Requested resource does not exist                                       |
| `409 Conflict`              | Request conflicts with current resource state                           |
| `422 Unprocessable Content` | Request is structurally valid but cannot be processed where appropriate |
| `500 Internal Server Error` | Unexpected server error                                                 |

The API should not use `200 OK` for all outcomes.

---

## 6.8 Pagination

Collection endpoints that may grow over time use pagination.

Example:

```text
GET /api/v1/applications?page=0&size=20
```

Default pagination:

```text
page = 0
size = 20
```

The backend must enforce a maximum page size to prevent excessively large requests.

The exact pagination response contract is defined in the Common Response Model section.

---

## 6.9 Filtering

Filtering is expressed using query parameters.

Example:

```text
GET /api/v1/applications?status=APPLIED
```

Multiple filters may be combined where supported:

```text
GET /api/v1/applications?companyName=Atlassian&status=APPLIED
```

Only documented filtering parameters form part of the API contract.

---

## 6.10 Search

Search requirements should be expressed explicitly rather than through a generic unrestricted query mechanism.

For Sprint 1, application search must support the fields required by the approved UI.

Example:

```text
GET /api/v1/applications?companyName=Atlassian
```

Partial matching and case-sensitivity behaviour will be defined in the Application API contract.

---

## 6.11 Sorting

Collection endpoints may support sorting where required.

Example:

```text
GET /api/v1/applications?sort=appliedAt,desc
```

The API must document which fields are sortable.

The backend must not accept arbitrary database column names as sort fields.

---

## 6.12 Request Validation

API requests are validated at the API boundary.

Validation includes, where applicable:

* required fields
* string length
* valid URL format
* valid enum values
* date formats
* allowed value ranges

Validation failures return a consistent error response.

Java validation annotations are implementation details and are not part of this API design document.

---

## 6.13 Response Design

API responses expose only information required by the consumer.

Persistence entities must not be returned directly.

Different use cases may have different response representations.

For example:

```text
Application List
    → lightweight application summary

Application Detail
    → full application representation
```

This prevents list endpoints from returning unnecessary detail and avoids coupling the public contract to the database model.

---

## 6.14 Error Handling

All API errors use a consistent error representation.

The error response must provide sufficient information for:

* frontend error handling
* debugging
* validation feedback
* API observability

Internal stack traces, SQL details, implementation class names, and sensitive system information must never be exposed to clients.

The detailed error contract is defined in the Error Model section.

---

## 6.15 Authentication

Endpoints are authenticated by default unless explicitly documented as public.

Authenticated resources must only expose data belonging to the current authenticated user.

Authentication mechanism details will be defined as part of the Authentication API and implementation design.

---

## 6.16 API and Database Separation

The REST API contract must remain independent from the physical database schema.

Database changes should not automatically require API changes.

Examples of database implementation details that should not leak into the API include:

* table names
* database column names
* internal surrogate keys where not part of the domain contract
* indexes
* persistence relationships
* database-specific data types

The API is designed around business concepts rather than database tables.

---

## 6.17 Backward Compatibility

After the Sprint 1 API contract is frozen, incompatible changes should be avoided within `/api/v1`.

Potential breaking changes include:

* renaming response fields
* removing response fields
* changing field data types
* changing endpoint paths
* changing existing enum semantics
* making previously optional request fields mandatory

Breaking changes should be introduced through a new API version when necessary.

---

## 6.18 OpenAPI as the Executable Contract

The final API contract will be represented using OpenAPI 3.x.

The OpenAPI specification must remain consistent with the decisions documented here.

Swagger UI may be used to:

* inspect available endpoints
* review request and response schemas
* test APIs during development
* provide interactive API documentation

OpenAPI describes the API contract, while Swagger UI is a presentation and testing interface for that contract.

---

# 7. Authentication and User API

## 7.1 Authentication Approach

OfferBuddy Sprint 1 uses Google as the external identity provider.

Authentication is based on OAuth 2.0 / OpenID Connect.

The backend is responsible for:

* initiating the Google authentication flow
* handling the authentication callback
* identifying or creating the OfferBuddy user
* establishing the authenticated application session
* protecting secured API endpoints

The frontend must not manage Google credentials directly beyond initiating the sign-in flow.

The authenticated OfferBuddy session is maintained using a secure HTTP-only cookie managed by the backend.

---

## 7.2 Authentication Flow

The expected Sprint 1 authentication flow is:

```text
User
  ↓
React Login Page
  ↓
Spring Boot Authentication Endpoint
  ↓
Google OAuth / OpenID Connect
  ↓
Google Authentication
  ↓
Spring Boot Callback
  ↓
Create or identify OfferBuddy user
  ↓
Create authenticated session
  ↓
Redirect to OfferBuddy Home
```

After authentication, subsequent API requests include the session cookie automatically.

The frontend does not need to attach an authentication token manually to every API request.

---

## 7.3 Login Endpoint

The frontend initiates Google authentication through:

```http
GET /oauth2/authorization/google
```

This endpoint is provided by the backend authentication framework and redirects the browser to Google.

It is not exposed under `/api/v1` because it represents an authentication protocol flow rather than a business REST resource.

---

## 7.4 Authentication Callback

Google redirects the authenticated user back to the backend OAuth callback endpoint.

Example:

```http
GET /login/oauth2/code/google
```

The callback is handled by the backend authentication framework.

The callback endpoint is not directly called by the frontend application.

After successful authentication, the backend redirects the user to the OfferBuddy frontend.

Example:

```text
/
```

or another configured authenticated landing page.

The final redirect URL is environment-specific configuration and is not part of the business API contract.

---

## 7.5 Current User

### Endpoint

```http
GET /api/v1/users/me
```

Returns basic information about the currently authenticated OfferBuddy user.

### Authentication

Required.

### Success Response

```http
200 OK
```

Example:

```json
{
  "userId": "USR_01ABC123",
  "displayName": "Peter",
  "email": "peter@example.com",
  "avatarUrl": "https://example.com/avatar.jpg"
}
```

The response must expose only user information required by the Sprint 1 frontend.

Sensitive authentication-provider data must not be exposed.

---

## 7.6 Unauthenticated Current User Request

If the client calls:

```http
GET /api/v1/users/me
```

without a valid authenticated session, the backend returns:

```http
401 Unauthorized
```

The API must not redirect normal REST API calls to the Google login page.

The frontend is responsible for handling the `401` response and displaying or navigating to the login experience.

---

## 7.7 Logout

### Endpoint

```http
POST /api/v1/auth/logout
```

### Authentication

Required.

### Behaviour

The endpoint invalidates the current OfferBuddy authenticated session.

### Success Response

```http
204 No Content
```

After logout, subsequent requests to protected APIs return:

```http
401 Unauthorized
```

Logout from OfferBuddy does not necessarily revoke the user's Google account session or Google account permissions.

---

## 7.8 Authentication Requirements for Business APIs

All Sprint 1 business APIs are authenticated by default.

Examples:

```text
POST   /api/v1/jobs/parse
GET    /api/v1/applications
POST   /api/v1/applications
GET    /api/v1/applications/{applicationId}
PUT    /api/v1/applications/{applicationId}
PATCH  /api/v1/applications/{applicationId}/status
DELETE /api/v1/applications/{applicationId}
```

These endpoints require a valid OfferBuddy authenticated session.

---

## 7.9 User Data Isolation

Authenticated users may only access resources that belong to their own OfferBuddy account.

For example, when requesting:

```http
GET /api/v1/applications/{applicationId}
```

the backend must verify that the requested application belongs to the authenticated user.

The frontend must never be trusted to provide ownership information.

The following request pattern must therefore be avoided:

```json
{
  "userId": "USR_01ABC123",
  "applicationId": "APP_01XYZ789"
}
```

The authenticated user identity is derived from the server-side authentication context.

---

## 7.10 Session Cookie Security

The authentication session cookie should use appropriate security settings.

Expected production characteristics include:

* `HttpOnly`
* `Secure`
* appropriate `SameSite` configuration
* limited session lifetime
* invalidation on logout

Sprint 1 uses same-origin frontend and API deployment. State-changing requests
must retain Spring Security CSRF protection, and cross-origin requests must be
restricted to explicitly configured trusted frontend origins. The frontend must
send session credentials and the CSRF token using the configured cookie/header
convention.

Exact cookie configuration is an implementation and deployment concern.

---

## 7.11 Authentication Error Behaviour

Authentication-related API behaviour follows standard HTTP semantics.

| Scenario                                             | Status             |
| ---------------------------------------------------- | ------------------ |
| Missing authenticated session                        | `401 Unauthorized` |
| Invalid or expired session                           | `401 Unauthorized` |
| Authenticated but not permitted to access a resource | `403 Forbidden`    |
| Requested owned resource does not exist              | `404 Not Found`    |

The API must not expose internal authentication provider errors or sensitive security details to the frontend.

---

## 7.12 Sprint 1 Authentication Scope

Sprint 1 authentication includes:

* Google Sign In
* authenticated OfferBuddy session
* current user retrieval
* logout
* protection of business APIs
* user-level data isolation

The following are outside Sprint 1:

* email/password registration
* password reset
* multi-factor authentication
* multiple OAuth providers
* refresh-token API exposed to the frontend
* API keys
* organisation accounts
* role-based administration


---

# 8. Job Parsing API

## 8.1 Purpose

The Job Parsing API extracts job information from a user-provided job URL.

It supports the Sprint 1 Home page flow:

```text
Paste Job URL
    ↓
Parse Job
    ↓
Review extracted information
    ↓
Create Application
```

Job parsing does not create or persist an application.

---

## 8.2 Parse Job

### Endpoint

```http
POST /api/v1/jobs/parse
```

### Authentication

Required.

### Request

```json
{
  "url": "https://www.seek.com.au/job/12345678"
}
```

### Request Fields

| Field | Type   | Required | Description                    |
| ----- | ------ | -------- | ------------------------------ |
| `url` | string | Yes      | Job advertisement URL to parse |

### Validation

The URL must:

* be present
* be a valid HTTP or HTTPS URL
* not exceed the configured maximum length

An invalid URL returns:

```http
400 Bad Request
```

---

## 8.3 Success Response

### Status

```http
200 OK
```

Example:

```json
{
  "sourceUrl": "https://www.seek.com.au/job/12345678",
  "jobTitle": "Software Engineer",
  "companyName": "Atlassian",
  "location": "Sydney NSW",
  "description": "We are looking for a software engineer...",
  "employmentType": "FULL_TIME",
  "workplaceType": "HYBRID",
  "salaryText": "$120,000-$140,000",
  "visaSponsorship": null,
  "sourcePlatform": "SEEK"
}
```

The response contains extracted job information available from the source.

The OpenAPI contract represents this payload as `ParsedJobResponse`. It is
separate from the confirmed `JobResponse` nested in an Application because a
parse result must preserve `sourceUrl` but may have missing extracted fields,
whereas a confirmed Job requires company and position but may originate from
manual entry without a URL.

The frontend must allow the user to review and modify extracted values before creating an application.

---

## 8.4 Optional Fields

Job advertisements from different sources may provide different information.

Therefore, extracted fields other than `sourceUrl` may be absent.

Example:

```json
{
  "sourceUrl": "https://example.com/job/123",
  "jobTitle": "Backend Engineer",
  "companyName": null,
  "location": null,
  "description": "..."
}
```

A partially parsed job is still considered a successful parse when useful information can be extracted.

The API must not fail solely because one optional field is unavailable.

---

## 8.5 Parsing Result Semantics

The API distinguishes between:

### Successful Parse

The URL is accessible and sufficient job information is extracted.

```http
200 OK
```

### Partial Parse

The URL is accessible and some useful job information is extracted, but one or more optional fields are unavailable.

```http
200 OK
```

Missing values are returned as `null` or omitted according to the final response schema convention.

### Parse Failure

The URL is valid but usable job information cannot be extracted.

The API returns a defined business error response.

Example error code:

```text
JOB_PARSE_FAILED
```

---

## 8.6 Unsupported Job Source

Sprint 1 may support common job advertisement sources and generic publicly accessible job pages.

If a source cannot be processed, the API returns an appropriate error response.

Example:

```http
422 Unprocessable Content
```

```json
{
  "code": "JOB_SOURCE_UNSUPPORTED",
  "message": "This job source is not currently supported."
}
```

The exact error representation must follow the common Error Model defined later in this document.

---

## 8.7 Inaccessible Job URL

If the supplied URL is valid but cannot be retrieved, the API must return a controlled error rather than exposing network or scraping implementation details.

Possible causes include:

* page not found
* access denied
* authentication required
* source blocking automated access
* timeout
* source unavailable

Example business error:

```text
JOB_PAGE_UNAVAILABLE
```

The client should receive a user-friendly error message and may allow manual application entry.

---

## 8.8 Parsing Timeout

Job parsing depends on an external website and therefore must not wait indefinitely.

The backend must enforce a bounded timeout.

If parsing cannot complete within the allowed duration, the operation returns a controlled failure.

Example:

```text
JOB_PARSE_TIMEOUT
```

Exact timeout values are implementation and operational configuration rather than part of the public API contract.

---

## 8.9 External Content Safety

Job advertisement content must be treated as untrusted external input.

The backend must not assume that retrieved page content is safe or well-formed.

The implementation should protect against:

* unexpectedly large pages
* malformed content
* redirects to unsupported locations
* unsafe URI schemes
* access to internal network resources
* external content being interpreted as trusted system instructions

The Job Parsing API must only return the information required by the OfferBuddy application workflow.

---

## 8.10 Source URL Preservation

The original job URL must be preserved in the parsing result.

Example:

```json
{
  "sourceUrl": "https://www.seek.com.au/job/12345678"
}
```

This value can later be submitted as part of the Create Application request.

Parsing the URL does not automatically persist it.

---

## 8.11 Idempotency Behaviour

Parsing the same job URL multiple times does not create duplicate application records because parsing does not persist an application.

Repeated requests may return updated information if the external job page has changed.

The API therefore does not guarantee an identical response across time.

---

## 8.12 Sprint 1 Scope

Sprint 1 Job Parsing includes:

* accepting a job URL
* retrieving job-page information
* extracting fields required by the current Application UI
* supporting partial parsing
* returning controlled parsing errors
* preserving the original source URL

The following capabilities are outside Sprint 1:

* automatic creation of applications
* automatic application submission
* background job crawling
* scheduled job monitoring
* bulk URL parsing
* automatic job discovery
* job recommendation
* continuous job-page change detection

---

# 9. Application API

## 9.1 Purpose

The Application API manages job application records owned by the authenticated user.

It supports the Sprint 1 user flows for:

* creating an application
* viewing application lists
* searching and filtering applications
* viewing application details
* editing an application
* changing application status
* deleting an application

All endpoints require authentication.

---

## 9.2 Create Application

### Endpoint

```http
POST /api/v1/applications
```

### Request

```json
{
  "job": {
    "companyName": "Atlassian",
    "jobTitle": "Backend Engineer",
    "location": "Sydney NSW",
    "employmentType": "FULL_TIME",
    "workplaceType": "HYBRID",
    "salaryText": "$120,000-$140,000",
    "visaSponsorship": null,
    "sourcePlatform": "SEEK",
    "sourceUrl": "https://www.seek.com.au/job/12345678"
  },
  "appliedAt": "2026-08-08",
  "notes": "Applied through SEEK."
}
```

### Request Fields

| Field       | Type   | Required | Description                                      |
| ----------- | ------ | -------- | ------------------------------------------------ |
| `job`       | object | Yes      | Confirmed job information                        |
| `appliedAt` | date   | Yes      | Date the application was submitted               |
| `notes`     | string | No       | User notes                                       |

The nested `job` object requires `companyName` and `jobTitle`. Location,
employment type, workplace type, salary, visa sponsorship, source platform,
source URL, description and extracted detail fields are optional so manual
entry remains available when no job URL exists.

Clients do not submit an initial status. The backend creates every Application
with `APPLIED` status and records the corresponding initial status-history entry.

### Success Response

```http
201 Created
```

Example:

```json
{
  "applicationId": "APP_01JABC123",
  "job": {
    "companyName": "Atlassian",
    "jobTitle": "Backend Engineer",
    "location": "Sydney NSW",
    "employmentType": "FULL_TIME",
    "workplaceType": "HYBRID",
    "salaryText": "$120,000-$140,000",
    "visaSponsorship": null,
    "sourcePlatform": "SEEK",
    "sourceUrl": "https://www.seek.com.au/job/12345678"
  },
  "status": "APPLIED",
  "appliedAt": "2026-08-08",
  "notes": "Applied through SEEK.",
  "createdAt": "2026-08-08T05:20:00Z",
  "updatedAt": "2026-08-08T05:20:00Z"
}
```

The created application belongs to the currently authenticated user.

The client must not provide a `userId`.

---

## 9.3 List Applications

### Endpoint

```http
GET /api/v1/applications
```

### Authentication

Required.

### Query Parameters

Supported parameters include:

| Parameter     | Type    | Required | Description                  |
| ------------- | ------- | -------- | ---------------------------- |
| `page`        | integer | No       | Zero-based page number       |
| `size`        | integer | No       | Number of records per page   |
| `companyName` | string  | No       | Search by company name       |
| `jobTitle`    | string  | No       | Search by position title     |
| `status`      | string  | No       | Filter by application status |
| `sort`        | string  | No       | Sort field and direction     |

Example:

```http
GET /api/v1/applications?page=0&size=20&jobTitle=engineer&status=APPLIED&sort=appliedAt,desc
```

### Success Response

```http
200 OK
```

Example:

```json
{
  "content": [
    {
      "applicationId": "APP_01JABC123",
      "companyName": "Canva",
      "jobTitle": "Backend Engineer",
      "location": "Sydney NSW",
      "workplaceType": "HYBRID",
      "visaSponsorship": null,
      "status": "APPLIED",
      "appliedAt": "2026-08-06"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

The list response is a lightweight representation and does not need to contain all application detail fields.

---

## 9.4 Search Behaviour

Company-name search supports partial matching.

Example:

```http
GET /api/v1/applications?companyName=can
```

may match:

```text
Canva
Canva Pty Ltd
```

Company-name search should be case-insensitive from the API consumer's perspective.

Search behaviour is limited to documented searchable fields.

Sprint 1 does not provide unrestricted full-text search across all application fields.

---

## 9.5 Sorting

The Sprint 1 Application API supports sorting by approved fields only.

Initial supported sort fields:

```text
appliedAt
createdAt
updatedAt
companyName
```

Example:

```http
GET /api/v1/applications?sort=appliedAt,desc
```

The default sort order is:

```text
appliedAt descending
```

unless otherwise required by the approved UI.

---

## 9.6 Get Application Detail

### Endpoint

```http
GET /api/v1/applications/{applicationId}
```

### Authentication

Required.

### Success Response

```http
200 OK
```

Example:

```json
{
  "applicationId": "APP_01JABC123",
  "job": {
    "companyName": "Atlassian",
    "jobTitle": "Backend Engineer",
    "location": "Sydney NSW",
    "employmentType": "FULL_TIME",
    "workplaceType": "HYBRID",
    "salaryText": "$120,000-$140,000",
    "visaSponsorship": null,
    "sourcePlatform": "SEEK",
    "sourceUrl": "https://www.seek.com.au/job/12345678"
  },
  "status": "INTERVIEW",
  "appliedAt": "2026-08-03",
  "notes": "Technical interview scheduled.",
  "createdAt": "2026-08-03T02:15:00Z",
  "updatedAt": "2026-08-08T01:10:00Z"
}
```

The endpoint returns only applications owned by the authenticated user.

If the resource does not exist or is not accessible to the current user, the API returns:

```http
404 Not Found
```

---

## 9.7 Update Application

### Endpoint

```http
PUT /api/v1/applications/{applicationId}
```

### Authentication

Required.

### Request

```json
{
  "status": "INTERVIEW",
  "appliedAt": "2026-08-03",
  "notes": "First interview completed."
}
```

The request represents the complete editable state of the application.

### Success Response

```http
200 OK
```

Example:

```json
{
  "applicationId": "APP_01JABC123",
  "job": {
    "companyName": "Atlassian",
    "jobTitle": "Backend Engineer",
    "location": "Sydney NSW",
    "sourceUrl": "https://www.seek.com.au/job/12345678"
  },
  "status": "INTERVIEW",
  "appliedAt": "2026-08-03",
  "notes": "First interview completed.",
  "createdAt": "2026-08-03T02:15:00Z",
  "updatedAt": "2026-08-08T05:40:00Z"
}
```

---

## 9.8 Update Application Status

### Endpoint

```http
PATCH /api/v1/applications/{applicationId}/status
```

### Authentication

Required.

### Request

```json
{
  "status": "INTERVIEW"
}
```

### Success Response

```http
200 OK
```

Example:

```json
{
  "applicationId": "APP_01JABC123",
  "status": "INTERVIEW",
  "updatedAt": "2026-08-08T05:40:00Z"
}
```

This endpoint is intended for UI interactions where only the application status changes.

---

## 9.9 Application Status Values

Sprint 1 supports the following application statuses:

- `APPLIED` — The application has been submitted and is awaiting an outcome.
- `NO_RESPONSE` — No response has been received from the employer.
- `INTERVIEW` — The application has progressed to an interview stage.
- `OFFER` — An offer has been received.
- `REJECTED` — The employer has explicitly rejected the application.
- `WITHDRAWN` — The user has withdrawn from the application process.

Application status changes are user-driven in Sprint 1.

OfferBuddy does not automatically transition an application from `APPLIED` to
`NO_RESPONSE`. The user selects `NO_RESPONSE` when it reflects the current
application outcome.

Unsupported status values return `400 Bad Request`.

The database may persist the status as a string. Supported status validation
is enforced by the application layer.
## 9.10 Delete Application

### Endpoint

```http
DELETE /api/v1/applications/{applicationId}
```

### Authentication

Required.

### Success Response

```http
204 No Content
```

The response contains no body.

If the application does not exist or is not accessible to the authenticated user:

```http
404 Not Found
```

The API contract does not expose whether deletion is physically implemented as a hard delete or soft delete.

---

## 9.11 Recent Applications

The Home page retrieves recent applications using the existing collection endpoint.

Example:

```http
GET /api/v1/applications?page=0&size=3&sort=appliedAt,desc
```

A dedicated endpoint such as:

```text
/api/v1/home/recent-applications
```

is intentionally not introduced in Sprint 1.

This allows the Home page and Application page to reuse the same Application resource contract.

---

## 9.12 Ownership

Application ownership is determined from the authenticated user.

Clients must never submit ownership fields such as:

```json
{
  "userId": "USR_123"
}
```

for application create, update, search, or delete operations.

All queries are scoped to the authenticated user's data.

---

## 9.13 Validation

Application requests must validate externally visible rules including:

* required company name
* required job title
* valid application status
* valid date format
* valid job URL when provided
* supported field lengths

Exact validation limits will be documented in the OpenAPI schema before API freeze.

---

## 9.14 Sprint 1 Application API Summary

| Method   | Endpoint                                      | Purpose                              |
| -------- | --------------------------------------------- | ------------------------------------ |
| `POST`   | `/api/v1/applications`                        | Create application                   |
| `GET`    | `/api/v1/applications`                        | List, search and filter applications |
| `GET`    | `/api/v1/applications/{applicationId}`        | Get application detail               |
| `PUT`    | `/api/v1/applications/{applicationId}`        | Update application                   |
| `PATCH`  | `/api/v1/applications/{applicationId}/status` | Update application status            |
| `DELETE` | `/api/v1/applications/{applicationId}`        | Delete application                   |

---

# 10. Common Response Model

## 10.1 Success Responses

Successful API responses use standard HTTP status codes and return the resource representation directly.

Example:

```http
GET /api/v1/applications/{applicationId}
```

```json
{
  "applicationId": "APP_01JABC123",
  "job": {
    "companyName": "Atlassian",
    "jobTitle": "Backend Engineer"
  },
  "status": "INTERVIEW",
  "appliedAt": "2026-08-03"
}
```

The API does not wrap successful responses in a generic envelope such as:

```json
{
  "code": 0,
  "message": "success",
  "data": {}
}
```

HTTP status codes provide the primary indication of request success.

---

## 10.2 Created Resources

Successful resource creation returns:

```http
201 Created
```

The response body contains the created resource representation.

Example:

```json
{
  "applicationId": "APP_01JABC123",
  "job": {
    "companyName": "Atlassian",
    "jobTitle": "Backend Engineer"
  },
  "status": "APPLIED",
  "appliedAt": "2026-08-08",
  "createdAt": "2026-08-08T05:20:00Z",
  "updatedAt": "2026-08-08T05:20:00Z"
}
```

Where appropriate, the response may also include a `Location` header pointing to the newly created resource.

Example:

```text
Location: /api/v1/applications/APP_01JABC123
```

---

## 10.3 No-Content Responses

Operations that complete successfully without requiring a response body return:

```http
204 No Content
```

Examples include:

```text
DELETE /api/v1/applications/{applicationId}
POST   /api/v1/auth/logout
```

A `204 No Content` response must not contain a response body.

---

## 10.4 Collection Responses

Paginated collection endpoints return pagination metadata together with the collection.

Example:

```json
{
  "content": [
    {
      "applicationId": "APP_01JABC123",
      "companyName": "Canva",
      "jobTitle": "Backend Engineer",
      "status": "APPLIED",
      "appliedAt": "2026-08-06"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 42,
  "totalPages": 3
}
```

### Pagination Fields

| Field           | Type    | Description                        |
| --------------- | ------- | ---------------------------------- |
| `content`       | array   | Resources in the current page      |
| `page`          | integer | Zero-based current page number     |
| `size`          | integer | Requested page size                |
| `totalElements` | integer | Total number of matching resources |
| `totalPages`    | integer | Total number of available pages    |

An empty collection is returned as:

```json
{
  "content": [],
  "page": 0,
  "size": 20,
  "totalElements": 0,
  "totalPages": 0
}
```

An empty result is not treated as an error.

---

# 11. Error Model

## 11.1 Purpose

All REST API errors use a consistent error representation.

The error model is designed to support:

* frontend error handling
* user-friendly error messages
* request validation feedback
* debugging and observability
* stable machine-readable error handling

---

## 11.2 Standard Error Response

Example:

```json
{
  "code": "APPLICATION_NOT_FOUND",
  "message": "The requested application could not be found.",
  "timestamp": "2026-08-08T06:10:00Z",
  "path": "/api/v1/applications/APP_01JABC123",
  "requestId": "req_01JXYZ789"
}
```

### Fields

| Field       | Type      | Required | Description                           |
| ----------- | --------- | -------- | ------------------------------------- |
| `code`      | string    | Yes      | Stable machine-readable error code    |
| `message`   | string    | Yes      | Human-readable error description      |
| `timestamp` | timestamp | Yes      | Time the error response was generated |
| `path`      | string    | Yes      | Requested API path                    |
| `requestId` | string    | No       | Request correlation identifier        |

The `code` field is intended for frontend logic.

The `message` field is intended primarily for user-facing or diagnostic information and should not be used as a stable programmatic identifier.

---

## 11.3 Validation Error Response

Validation failures return:

```http
400 Bad Request
```

Example:

```json
{
  "code": "VALIDATION_FAILED",
  "message": "One or more request fields are invalid.",
  "timestamp": "2026-08-08T06:10:00Z",
  "path": "/api/v1/applications",
  "requestId": "req_01JXYZ789",
  "fieldErrors": [
    {
      "field": "companyName",
      "code": "REQUIRED",
      "message": "Company name is required."
    },
    {
      "field": "job.sourceUrl",
      "code": "INVALID_URL",
      "message": "Job URL must be a valid HTTP or HTTPS URL."
    }
  ]
}
```

### Field Error Model

| Field     | Type   | Description                       |
| --------- | ------ | --------------------------------- |
| `field`   | string | Invalid request field             |
| `code`    | string | Machine-readable validation error |
| `message` | string | Human-readable validation message |

---

## 11.4 Authentication Errors

Missing, invalid, or expired authentication returns:

```http
401 Unauthorized
```

Example error code:

```text
AUTHENTICATION_REQUIRED
```

---

## 11.5 Authorisation Errors

An authenticated user who is not permitted to perform an operation may receive:

```http
403 Forbidden
```

Example error code:

```text
ACCESS_DENIED
```

Where resource ownership privacy is important, the API may return `404 Not Found` instead of revealing that another user's resource exists.

---

## 11.6 Resource Not Found

When an application does not exist or is not accessible to the current user:

```http
404 Not Found
```

Example:

```json
{
  "code": "APPLICATION_NOT_FOUND",
  "message": "The requested application could not be found.",
  "timestamp": "2026-08-08T06:10:00Z",
  "path": "/api/v1/applications/APP_01JABC123",
  "requestId": "req_01JXYZ789"
}
```

---

## 11.7 Conflict Errors

A request that conflicts with the current state of a resource may return:

```http
409 Conflict
```

Sprint 1 does not reject a possible duplicate Application solely because the
same job appears to have been recorded before. Duplicate detection may return a
warning in a future contract, but it is not a `409 Conflict` rule in the current
Sprint 1 API.

The status remains reserved for a concrete resource-state conflict introduced
by a future contract change.

---

## 11.8 Job Parsing Errors

Job parsing may return business-specific error codes including:

```text
JOB_PARSE_FAILED
JOB_SOURCE_UNSUPPORTED
JOB_PAGE_UNAVAILABLE
JOB_PARSE_TIMEOUT
```

These codes allow the frontend to distinguish parsing failures and provide appropriate fallback behaviour.

Implementation-specific network, parsing, scraping, or AI errors must not be exposed directly to the client.

---

## 11.9 Unexpected Server Errors

Unexpected server failures return:

```http
500 Internal Server Error
```

Example:

```json
{
  "code": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred.",
  "timestamp": "2026-08-08T06:10:00Z",
  "path": "/api/v1/applications",
  "requestId": "req_01JXYZ789"
}
```

The response must not expose:

* stack traces
* SQL statements
* database names
* internal class names
* infrastructure details
* credentials
* tokens
* internal exception messages

Detailed technical information should be recorded in server-side logs rather than returned to the client.

---

## 11.10 Error Code Naming

Error codes use uppercase `SNAKE_CASE`.

Examples:

```text
VALIDATION_FAILED
APPLICATION_NOT_FOUND
AUTHENTICATION_REQUIRED
ACCESS_DENIED
JOB_PARSE_FAILED
JOB_SOURCE_UNSUPPORTED
INTERNAL_SERVER_ERROR
```

Once published as part of `/api/v1`, error codes should remain stable where frontend behaviour depends on them.

---


# 12. API Review and Freeze

The Sprint 1 API contract has been reviewed against:

- Sprint 1 MVP scope
- approved Figma UI
- Sprint 1 database schema
- authentication requirements
- job/application domain boundaries

Key decisions:

- The Home page does not have a dedicated API domain.
- Job parsing and application creation are separate operations.
- Job data is read-only once persisted in Sprint 1.
- Application data is user-owned and editable.
- Job and Application persistence models are not exposed directly as database entities.
- Application status validation is enforced by the application layer.
- Application collection APIs are paginated.
- Successful responses use standard HTTP semantics.
- Errors use a consistent machine-readable error model.

The Sprint 1 API contract is considered frozen after the final OpenAPI specification is reviewed and accepted.
