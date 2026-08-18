# Product Backlog

## Purpose

This document records product work that is not committed to a sprint and routes approved scope to its authoritative sprint requirements. Engineering debt is maintained separately in the [Sprint 1 Technical Debt Register](sprint-1-technical-debt.md).

An item becomes sprint scope only after refinement, prioritisation, sizing, and an explicit sprint decision.

## Approved Sprint 2 Requirements

Sprint 2 Phase 1 Requirement Analysis is complete and approved. The Browser Extension, SEEK and Indeed Job Capture, eligibility screening, Job Intelligence, non-blocking AI enrichment, fallback AI URL parsing, and basic Application Analytics are no longer unrefined backlog candidates.

Their authoritative scope, priorities, exclusions, failure expectations, non-functional requirements, and deferred decisions are maintained in the [Sprint 2 Requirements](../product/sprint-2-requirements.md). Sprint 2 functionality is not yet represented as delivered.

Architecture, technical implementation, detailed UI/UX, extension authentication, event implementation, AI execution, and additional recruitment platforms remain deferred to later phases rather than open Requirement Analysis work.

## Later Product Possibilities

- resume generation or optimisation
- Cover Letter generation
- candidate/job match analysis
- recruiter-message assistance
- application-form assistance
- interview preparation
- email status detection
- notifications
- job discovery
- additional recruitment platforms beyond SEEK and Indeed
- additional identity providers
- subscription/team capabilities

These items have no committed sprint.

LinkedIn support, Auto Apply, automatic application-form submission, Cover Letter generation, resume tailoring, candidate/Job match scoring, and large Analytics/BI expansion are explicitly outside Sprint 2.

## Completed or Reconciled During Sprint 1

The earlier AI parsing checklist is no longer open backlog work. Existing tests cover valid and incomplete structured output, invalid output, provider timeout/failure mapping, page retrieval behaviour, and parsing controller/service paths.

A live Gemini integration test remains conditional on `GOOGLE_API_KEY`; normal CI intentionally stays offline. Further AI resilience work should be created only for a specific uncovered risk.

## Technical Follow-Up

Refer to [Sprint 1 Technical Debt](sprint-1-technical-debt.md) for frontend automated tests, correlation IDs, secrets/backup hardening, observability, staging, job identity rules, and maintainability work. Those items are not product features and must compete transparently for sprint capacity.
