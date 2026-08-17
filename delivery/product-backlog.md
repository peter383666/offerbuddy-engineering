# Product Backlog

## Purpose

This document records product work that is not committed to a sprint. Engineering debt is maintained separately in the [Sprint 1 Technical Debt Register](sprint-1-technical-debt.md).

An item becomes sprint scope only after refinement, prioritisation, sizing, and an explicit sprint decision.

## Highest-Level Sprint 2 Candidates

### Browser Extension Capture

Potential outcome: capture job-page content from the user's browser and send structured source content to OfferBuddy, reducing dependence on server-side page retrieval.

This is the leading Sprint 2 product direction, not committed or implemented functionality.

Questions requiring refinement:

- supported browsers and job sites
- authenticated communication with the OfferBuddy API
- page-content selection and privacy boundaries
- fallback when a page changes or blocks access
- division of responsibility between extension acquisition and backend AI analysis

### Reduce Manual Entry Friction

Potential improvements should be evaluated against the real capture workflow. The objective is faster reliable capture, not automation for its own sake.

### Analytics

Potential secondary capability:

- application totals and status distribution
- time-period activity
- outcome/funnel reporting

Analytics depends on clearly defined product questions and trustworthy data. It is not part of Sprint 1 and is secondary to capture friction in the current direction.

## Later Product Possibilities

- resume generation or optimisation
- cover-letter generation
- interview preparation
- email status detection
- notifications
- job discovery
- additional identity providers
- subscription/team capabilities

These items have no committed sprint.

## Completed or Reconciled During Sprint 1

The earlier AI parsing checklist is no longer open backlog work. Existing tests cover valid and incomplete structured output, invalid output, provider timeout/failure mapping, page retrieval behaviour, and parsing controller/service paths.

A live Gemini integration test remains conditional on `GOOGLE_API_KEY`; normal CI intentionally stays offline. Further AI resilience work should be created only for a specific uncovered risk.

## Technical Follow-Up

Refer to [Sprint 1 Technical Debt](sprint-1-technical-debt.md) for frontend automated tests, correlation IDs, secrets/backup hardening, observability, staging, job identity rules, and maintainability work. Those items are not product features and must compete transparently for sprint capacity.
