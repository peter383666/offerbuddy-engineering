# OfferBuddy Engineering

Production-oriented engineering documentation for OfferBuddy.

OfferBuddy is a job application tracking product and portfolio engineering project. Application source code lives in the separate [OfferBuddy source repository](https://github.com/peter383666/offerbuddy).

## Current Release

Sprint 2 documentation wrap-up is in progress on `docs/sprint-2`.

Production application: [offerbuddy.io](https://offerbuddy.io)

| View | Start here |
| --- | --- |
| What the product includes now | [Current Scope](product/current-scope.md) |
| What the system looks like now | [Architecture](architecture/README.md) |
| Why key decisions were made | [ADR Index](decisions/README.md) |
| How Sprint 2 was designed | [Sprint 2 Archive](delivery/s2/README.md) |

## Documentation Model

| Layer | Answers | Location |
| --- | --- | --- |
| Current system | What is OfferBuddy now? | `product/`, `architecture/`, `quality/`, `operations/` |
| Sprint history | How was Sprint N designed and delivered? | `delivery/sN/` (frozen after closure) |
| Decisions | Why was an important choice made? | `decisions/ADR-*` |

Governance: [Documentation Governance](operations/documentation-governance.md)

```text
Branch = Sprint-level isolation (docs/sprint-N)
Commit = Sprint design checkpoint
PR     = documentation review / release gate
Tag    = engineering-sN on main after merge
```

## Portal

### Product

- [Product Vision](product/product-vision.md)
- [Roadmap](product/roadmap.md)
- [Current Scope](product/current-scope.md)
- [Product Backlog](delivery/product-backlog.md)

### Architecture (current)

- [Architecture Index](architecture/README.md)
- [System Context](architecture/system-context.md)
- [Container Design](architecture/container-design.md)
- [Data Model](architecture/data-model.md)
- [API Design](architecture/api-design.md)

### Engineering Decisions

- [ADR Index](decisions/README.md)

### Delivery History

- [Sprint 0](delivery/s0/README.md)
- [Sprint 1](delivery/s1/README.md)
- [Sprint 2](delivery/s2/README.md)

### Quality

- [Testing Strategy](quality/testing-strategy.md)
- [Non-Functional Requirements](quality/non-functional-requirements.md)
- [Definition of Done](quality/definition-of-done.md)

### Operations

- [Documentation Governance](operations/documentation-governance.md)
- [Development Workflow](operations/development-workflow.md)
- [Deployment Strategy](operations/deployment-strategy.md)
- [Production Runbook](operations/production-runbook.md)
- [PostgreSQL Backup and Restore](operations/postgresql-backup-and-restore.md)

### Technology

- [Technology Stack](technology/tech-stack.md)

## Release History

| Version | Description |
| --- | --- |
| `engineering-v0.1` | Product and initial MVP architecture documentation established |
| `engineering-v0.2` | Sprint 1 documentation baseline |
| `engineering-s2` | Pending merge of `docs/sprint-2` |

## License

This repository contains engineering documentation for OfferBuddy. Application source code is maintained separately.
