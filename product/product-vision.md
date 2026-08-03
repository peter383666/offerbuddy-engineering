# Product Vision

## Product

OfferBuddy is an AI-assisted job application tracking and optimisation platform designed to help job seekers capture job opportunities, manage applications, and improve their job search process in one place.

## Problem

Job seekers often manage applications using spreadsheets, browser bookmarks, notes, email, and multiple job platforms.

This creates several problems:

- Job information must be copied manually
- Application statuses become difficult to maintain
- Important follow-up dates may be missed
- Users cannot easily measure application outcomes
- Interview preparation is disconnected from the original job advertisement
- It is difficult to identify which job search strategies are working

## Target Users

The initial target users are technology professionals applying for jobs in Australia.

The first internal user is the product owner, who is actively applying for software engineering roles and currently manages applications manually.

## Vision

OfferBuddy will provide a single workflow for:

1. Capturing a job opportunity
2. Tracking the application lifecycle
3. Analysing application outcomes
4. Preparing for interviews
5. Improving future applications

## Initial Value Proposition

For the first release, OfferBuddy should make job tracking significantly faster and more reliable than using a spreadsheet.

A user should be able to paste a job advertisement URL, allow OfferBuddy to extract key job information using AI, review and correct the extracted information, and save the opportunity as an application record.

Manual entry must remain available when automatic retrieval or AI extraction is unavailable.

## MVP AI Capability

The MVP includes a limited AI-assisted information extraction capability.

AI will be used to convert job advertisement content into structured data such as:

- Job title
- Company name
- Location
- Employment type
- Salary information when available
- Job description
- Required skills or keywords
- Source platform

AI output is treated as a draft rather than trusted business data.

The user must be able to review and edit the extracted information before saving it.

## Long-Term Direction

Future versions may include:

- Resume-to-job matching
- Resume optimisation
- Application quality recommendations
- Interview question generation
- Mock interview workflows
- Follow-up reminders
- Browser extension integration
- Email-based status detection
- Job discovery and recommendation
- Automated application assistance
- Support for markets outside Australia

## Product Principles

### Solve a real workflow first

The product must first replace the current spreadsheet-based workflow before introducing advanced automation.

### Manual fallback remains available

Users should always be able to enter and correct information manually when automated retrieval or AI extraction is unavailable or inaccurate.

### Incremental delivery

Each release should provide usable value without requiring the complete long-term product.

### Avoid premature complexity

The first release should not use microservices, Kubernetes, or unnecessary distributed infrastructure.

### Maintain user control

AI-generated or automatically extracted data should be reviewable and editable by the user.

### AI assists but does not decide

AI-generated information must be treated as a draft.

Users must be able to review, correct, or replace AI-generated values before those values become part of their application records.
