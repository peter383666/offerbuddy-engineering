# OfferBuddy Engineering

> AI-assisted Job Application Tracker

Engineering documentation for the OfferBuddy project.

---

# Project Status

| Area | Status |
|------|--------|
| Product Design | ✅ Complete |
| Architecture | ✅ Complete |
| Sprint 0 | 🚧 In Progress |
| Sprint 1 | ⏳ Planned |
| Backend Development | ⏳ Planned |
| Frontend Development | ⏳ Planned |
| Deployment | ⏳ Planned |

---

# Project Overview

OfferBuddy is an AI-assisted job application tracking platform designed to help job seekers organise applications, reduce repetitive manual work, and improve their job search through structured workflows and data-driven insights.

Unlike a simple spreadsheet or bookmark manager, OfferBuddy focuses on the complete application lifecycle—from recording submitted applications to analysing job search performance and, in future versions, supporting semi-automated and automated application workflows.

This repository contains the engineering documentation for the project, including product planning, architecture, technical decisions, delivery planning, and engineering standards.

The application source code is maintained in a separate repository.

---

# Vision

Build a production-quality software project using an industry-standard software development lifecycle while solving a real problem faced during the Australian technology job search process.

The long-term vision is to evolve OfferBuddy from a personal productivity tool into a SaaS platform for job seekers.

---

# Development Philosophy

OfferBuddy is developed using an iterative and incremental approach.

The objective is not to build every planned feature immediately.

Each iteration delivers a working improvement that can be used in real job searching.

Documentation evolves together with the product.

Architecture decisions are documented before implementation whenever practical.

---

# Engineering Principles

- Simplicity before complexity
- Build working software first
- Architecture supports future growth
- AI assists users rather than replacing user decisions
- Documentation evolves with implementation
- Measure before optimising
- Avoid premature microservices
- Deliver value in small iterations

---

# Software Development Lifecycle

```text
Requirements
        ↓
Architecture & Design
        ↓
Sprint Planning
        ↓
Implementation
        ↓
Testing
        ↓
Deployment
        ↓
Feedback
        ↓
Next Iteration
```

Current stage:

**Sprint 0 — Engineering Setup**

---

# Repository Structure

```text
offerbuddy-engineering/
│
├── product/
│   ├── product-vision.md
│   ├── mvp-scope.md
│   └── user-stories.md
│
├── technology/
│   └── tech-stack.md
│
├── architecture/
│   ├── system-context.md
│   ├── container-design.md
│   └── data-model.md
│
├── decisions/
│   ├── ADR-001-modular-monolith.md
│   ├── ADR-002-google-authentication.md
│   ├── ADR-003-ai-assisted-job-extraction.md
│   └── ADR-004-ai-provider-abstraction.md
│
├── delivery/
│   ├── roadmap.md
│   └── sprint-0.md
│
├── quality/
│   └── testing-strategy.md
│
├── operations/
│   └── deployment-strategy.md
│
└── assets/
    ├── diagrams/
    └── screenshots/
```

---

# Documentation

## Product

| Document | Purpose |
|----------|---------|
| Product Vision | Product goals and long-term direction |
| MVP Scope | Initial product scope |
| User Stories | Functional requirements |

---

## Architecture

| Document | Purpose |
|----------|---------|
| System Context | External systems and boundaries |
| Container Design | High-level application architecture |
| Data Model | Business entities and relationships |

---

## Technology

| Document | Purpose |
|----------|---------|
| Tech Stack | Selected technologies and technical rationale |

---

## Architecture Decision Records

| ADR | Description |
|-----|-------------|
| ADR-001 | Modular Monolith |
| ADR-002 | Google Authentication |
| ADR-003 | AI-assisted Job Parsing |
| ADR-004 | AI Provider Abstraction |

---

# Technology Stack

## Frontend

- React
- TypeScript
- Vite

## Backend

- Java 21
- Spring Boot

## Database

- PostgreSQL

## Authentication

- Google OAuth 2.0
- OpenID Connect

## Infrastructure

- Docker
- GitHub Actions

## AI

- External Large Language Model Provider

---

# Architecture Overview

> Architecture diagrams will be added as the project evolves.

Planned diagrams include:

- System Context Diagram
- Container Diagram
- Entity Relationship Diagram
- Application Flow Diagram

---

# MVP Workflow

```text
Paste Job URL
        ↓
AI extracts job information
        ↓
User reviews and edits
        ↓
Confirm application details
        ↓
Record submitted application
        ↓
Track application lifecycle
        ↓
Analyse job search performance
```

---

# Roadmap

## Phase 1

AI-assisted application recording

## Phase 2

Application analytics

## Phase 3

Resume optimisation

## Phase 4

Interview preparation

## Phase 5

Semi-automated application workflow

## Phase 6

Automated job application platform

---

# Current Milestone

## Sprint 0

Engineering foundation

Completed

- Product definition
- MVP scope
- User stories
- Technology selection
- Architecture
- Data model
- Architecture Decision Records

In Progress

- Engineering setup
- Repository structure
- Development workflow
- GitHub Project
- Milestone planning

Next

Sprint 1

Record the first job application

---

# Release History

| Version | Description |
|----------|-------------|
| engineering-v0.1 | Product and architecture documentation completed |

---

# Future Enhancements

The following capabilities are intentionally deferred until after the MVP:

- Resume optimisation
- Resume matching
- AI interview preparation
- Browser extension
- Email integration
- Semi-automated applications
- Automated job applications
- Multi-user SaaS platform

---

# License

This repository contains engineering documentation for the OfferBuddy project.

Application source code is maintained separately.