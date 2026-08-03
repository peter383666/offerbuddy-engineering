# Sprint 0 — Engineering Foundation

## Sprint Goal

Establish the product, engineering, repository, and development foundations required to begin MVP implementation.

## Sprint Duration

Start date: 3 August 2026

Target duration: 1 week

## Objectives

- Establish the engineering documentation repository
- Define the product vision
- Define and limit the MVP scope
- Confirm the initial system architecture
- Establish the source code repository structure
- Define the Git and pull request workflow
- Create the initial backend and frontend applications
- Prepare the local Docker development environment
- Confirm that the system can run locally

## Planned Work

### Product

- [x] Create engineering repository
- [x] Create source code repository
- [x] Clone both repositories locally
- [ ] Write the initial product vision
- [ ] Define MVP scope
- [ ] Draft initial user stories
- [x] Confirm AI-assisted job parsing as an MVP capability
- [x] Update MVP documents to reflect AI-assisted job capture

### Architecture

- [ ] Create the system context diagram
- [ ] Confirm modular monolith architecture
- [ ] Record ADR-001
- [ ] Define initial application modules
- [ ] Draft the initial data model
- [ ] Define the AI-assisted parsing flow
- [ ] Define the AI provider boundary
- [ ] Define manual fallback behaviour
- [ ] Define parsing status handling
- [ ] Define AI response validation requirements

### Development Foundation

- [ ] Create backend application
- [ ] Create frontend application
- [ ] Add Docker Compose
- [ ] Add PostgreSQL
- [ ] Add project-level README
- [ ] Confirm local startup process
- [ ] Select the initial AI provider and model
- [ ] Create the job extraction provider interface
- [ ] Define the structured extraction response schema
- [ ] Configure AI credentials through environment variables

### Delivery

- [ ] Define branch naming rules
- [ ] Define commit message conventions
- [ ] Create the first GitHub issue
- [ ] Create the first feature branch
- [ ] Open and merge the first pull request

### Quality

- [ ] Add backend test foundation
- [ ] Add frontend test foundation
- [ ] Add basic GitHub Actions workflow
- [ ] Define the initial Definition of Done
- [ ] Define test cases for valid AI extraction responses
- [ ] Define test cases for incomplete AI responses
- [ ] Define test cases for invalid structured output
- [ ] Define test cases for provider timeout and failure
- [ ] Confirm manual entry works when parsing fails

## Definition of Done

Sprint 0 is complete when:

- The engineering repository clearly explains the product and MVP
- The source repository contains runnable frontend and backend applications
- PostgreSQL can run locally using Docker
- A basic CI workflow runs successfully
- At least one change has been delivered through a feature branch and pull request
- The architecture decisions are documented
- The AI-assisted job parsing workflow is documented consistently.
- The system architecture includes the external AI provider.
- Manual entry and AI failure fallback are explicitly defined.
- The planned provider integration does not bypass backend validation.

## Notes

Sprint 0 does not deliver end-user functionality. Its purpose is to establish a professional and repeatable delivery foundation.
