# Testing Strategy

## Purpose

This document defines the testing direction for OfferBuddy. Detailed automated tests are implemented alongside the relevant product capability rather than being treated as complete when only their scenarios have been identified.

## Test Levels

OfferBuddy will use an appropriate combination of:

- Unit tests for isolated business rules and transformations
- Integration tests for Spring Boot, PostgreSQL, Flyway, and external integration boundaries
- Component tests for React user interactions
- End-to-end tests for critical user workflows when those workflows are implemented

Tests should be deterministic, independently repeatable, and suitable for continuous integration.

## Backend Testing Foundation

The backend testing foundation should include:

- JUnit 5
- Spring Boot Test
- A context-load test
- Testcontainers when database-backed integration tests are introduced

The generated Spring Boot test class is only an initial scaffold. The foundation is complete when the test configuration and conventions support meaningful business and integration tests.

## Frontend Testing Foundation

The frontend testing foundation should include:

- Vitest
- React Testing Library
- `jsdom`
- Shared test setup
- At least one representative component test

ESLint and production builds are quality checks, but they do not replace frontend tests.

## AI Extraction Testing

The AI extraction feature must be tested against both successful and degraded provider responses.

Planned scenarios:

- Valid structured extraction response
- Incomplete extraction response
- Invalid structured output
- Provider timeout
- Provider unavailable or failed request
- Manual entry fallback when parsing cannot complete

AI output must be treated as untrusted test input. Tests should confirm schema validation, missing-field handling, safe failure behaviour, and the requirement for user confirmation before persistence.

These scenarios will be implemented when the AI extraction feature enters an active development sprint.
