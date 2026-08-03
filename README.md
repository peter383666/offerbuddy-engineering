# OfferBuddy Engineering

OfferBuddy is an AI-assisted job application tracking and optimisation platform initially designed for the Australian technology job market.

This repository documents the engineering process behind OfferBuddy, including product discovery, architecture decisions, delivery planning, quality strategy, and deployment design.

The production source code is maintained in a separate private repository.

## Project Goals

OfferBuddy aims to help job seekers:

- Save job opportunities by pasting job advertisement URLs
- Use AI to extract key job information
- Review and correct automatically extracted information
- Track application progress in one place
- Reduce repetitive manual data entry
- Analyse application activity and conversion rates
- Prepare for future AI-assisted matching and interview preparation

## Engineering Approach

The project is being developed iteratively using:

- Incremental MVP delivery
- A modular monolith architecture
- Architecture Decision Records
- Versioned product and technical documentation
- Automated testing and continuous integration
- Docker-based deployment

## Repository Structure

| Directory       | Purpose                                          |
| --------------- | ------------------------------------------------ |
| `product/`      | Product vision, MVP scope, and user stories      |
| `architecture/` | System context, container design, and data model |
| `decisions/`    | Architecture Decision Records                    |
| `delivery/`     | Roadmap, sprint plans, and delivery progress     |
| `quality/`      | Testing strategy and non-functional requirements |
| `operations/`   | Deployment and operational design                |
| `assets/`       | Architecture diagrams and product screenshots    |

## Current Status

**Phase:** Sprint 0 — Product and Engineering Foundation

Current priorities:

1. Define the MVP scope
2. Confirm the initial architecture
3. Establish the codebase and development workflow
4. Prepare the local development environment
5. Deliver the first end-to-end job application tracking flow

## Planned MVP

The first release will focus on:

- Google authentication
- Submitting a job advertisement URL
- Retrieving publicly accessible job content
- AI-assisted extraction of structured job information
- Reviewing and correcting extracted information
- Manual entry when extraction is unavailable
- Saving and managing job applications
- Updating application statuses
- Searching and filtering applications
- Viewing basic application statistics

## Technology Direction

The current planned technology stack is:

- React
- TypeScript
- Vite
- Java 21
- Spring Boot
- PostgreSQL
- Google OAuth 2.0 and OpenID Connect
- External AI extraction provider
- Docker
- GitHub Actions
- Docker-based cloud deployment

Technology decisions will be recorded through Architecture Decision Records.

The MVP includes AI only for job advertisement information extraction.

Resume optimisation, interview generation, mock interviews, and automated applications remain future capabilities.

## Documentation Status

This documentation evolves together with the product. Documents represent the current engineering direction and may be updated as new requirements and implementation constraints are discovered.
