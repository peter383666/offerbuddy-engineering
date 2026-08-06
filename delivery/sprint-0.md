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
- [x] Write the initial product vision
- [x] Define MVP scope
- [x] Draft initial user stories
- [x] Confirm AI-assisted job parsing as an MVP capability
- [x] Update MVP documents to reflect AI-assisted job capture

### Architecture

- [x] Create the system context diagram
- [x] Confirm modular monolith architecture
- [x] Record ADR-001
- [x] Define initial application modules
- [x] Draft the initial data model
- [x] Define the AI-assisted parsing flow
- [x] Define the AI provider boundary
- [x] Define manual fallback behaviour
- [x] Define parsing status handling
- [x] Define AI response validation requirements

### Development Foundation

- [x] Create backend application
- [x] Create frontend application
- [x] Add Docker Compose
- [x] Add PostgreSQL
- [x] Add project-level README
- [x] Confirm local startup process
- [ ] Select the initial AI provider and model
- [ ] Create the job extraction provider interface
- [ ] Define the structured extraction response schema
- [ ] Configure AI credentials through environment variables
    
### Delivery

- [x] Define branch naming rules
- [x] Define commit message conventions
- [x] Create the first GitHub issue
- [x] Create the first feature branch
- [x] Open and merge the first pull request

### Quality

- [x] Add basic GitHub Actions workflow
- [x] Define the initial Definition of Done
- [ ] Add backend test foundation
- [ ] Add frontend test foundation

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

---

## Sprint Outcome

Sprint 0 was completed successfully.

The engineering foundation for OfferBuddy has been established, including the project structure, local development environment, database migration strategy, architecture direction, and continuous integration pipeline.

All seven committed delivery items were completed. Unfinished tasks identified during the sprint remain visible above and will be refined through the product backlog before being committed to a future sprint.

See the following documents for details:

- [Sprint 0 Review](sprint-0-review.md)
- [Sprint 0 Retrospective](sprint-0-retrospective.md)
