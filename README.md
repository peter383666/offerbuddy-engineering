# OfferBuddy Engineering

OfferBuddy is a job application tracking and optimisation platform initially designed for the Australian technology job market.

This repository documents the engineering process behind OfferBuddy, including product discovery, architecture decisions, delivery planning, quality strategy, and deployment design.

The production source code is maintained in a separate private repository.

## Project Goals

OfferBuddy aims to help job seekers:

- Save job opportunities from job advertisement links
- Track application progress in one place
- Reduce repetitive manual data entry
- Analyse application activity and conversion rates
- Prepare for future AI-assisted job matching and interview preparation

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

- User authentication
- Saving a job application
- Extracting basic job information from a URL
- Managing application status
- Viewing application history
- Basic application statistics

## Technology Direction

The current planned technology stack is:

- React or Next.js
- TypeScript
- Java 21
- Spring Boot
- PostgreSQL
- Redis where justified
- Docker
- GitHub Actions
- Cloud-based deployment

Technology decisions will be recorded through Architecture Decision Records.

## Documentation Status

This documentation evolves together with the product. Documents represent the current engineering direction and may be updated as new requirements and implementation constraints are discovered.