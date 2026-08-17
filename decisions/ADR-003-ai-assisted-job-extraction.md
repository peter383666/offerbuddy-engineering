# ADR-003: Use AI-Assisted Extraction for MVP Job Capture

## Status

Accepted

## Date

3 August 2026

## Context

The initial OfferBuddy workflow was designed around manual job application entry.

However, manually copying job titles, company names, locations, descriptions, and other information from job advertisements creates much of the same friction as the spreadsheet workflow that OfferBuddy is intended to replace.

The MVP needs to provide a clear improvement over manual spreadsheet entry.

## Decision

OfferBuddy will include AI-assisted job information extraction in the MVP.

A user will submit a job advertisement URL. The backend will attempt to retrieve relevant content and send it to an external AI provider for structured extraction.

The extracted information will be returned as an editable draft. The user must review and confirm the information before it is saved as a job application record.

Manual entry will remain available when page retrieval or AI extraction fails.

## Scope

The MVP AI capability is limited to extracting job advertisement information. It may extract:

- Job title
- Company name
- Location
- Employment type
- Salary information
- Job description
- Skills or keywords
- Source platform

The following capabilities remain outside the MVP:

- Resume optimisation
- Resume-to-job matching
- Interview question generation
- Mock interviews
- Cover letter generation
- Automated job applications

## Consequences

### Positive

- Reduces repetitive manual data entry
- Creates a stronger improvement over spreadsheet tracking
- Provides an immediately visible AI capability
- Supports job data normalisation
- Creates a foundation for future job analysis features

### Negative

- Introduces dependency on an external provider
- Introduces usage cost
- AI output may be inaccurate
- Job website retrieval may fail
- Additional validation and error handling are required
- Provider rate limits and outages must be considered

## Risk Controls

- AI output will be treated as untrusted draft data.
- The backend will validate structured responses.
- The user must review information before saving.
- Manual entry will remain available.
- Provider credentials will not be committed to source control.
- Requests will use timeouts and content-size limits.
- The AI provider will not access the database directly.
- Advanced AI features will remain outside the MVP.

## Alternatives Considered

### Manual entry only

Rejected because it would not sufficiently reduce the main friction of the existing spreadsheet workflow.

### Rule-based HTML parsing only

Rejected as the only solution because job advertisement websites use inconsistent and frequently changing structures.

Rule-based parsing may still be used to retrieve and clean page content before AI extraction.

### Browser extension first

Rejected for the initial MVP because it would add another deployable client and browser-specific development work.

A browser extension may be introduced after the web workflow is proven.

## Implementation Outcome

Sprint 1 implements synchronous server-side page retrieval followed by Gemini semantic extraction. The backend validates and maps the structured response, returns an editable draft, and does not persist data until the user creates an Application.

Production experience confirmed two separate failure and latency boundaries: deterministic page-content acquisition and AI semantic analysis. Job sites can block or limit server-side acquisition even when the AI provider is healthy. A future browser extension may improve acquisition, but it is not part of Sprint 1.
