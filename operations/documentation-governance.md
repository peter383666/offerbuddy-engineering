# Documentation Governance

## Purpose

Defines how OfferBuddy engineering documentation is branched, structured, reviewed, and released.

## Highest-Level Closeout Rule

> S2 documentation closeout is a reconciliation and release-record exercise, not a new architecture/design phase. Preserve valid S2 decisions, align documentation with implemented behaviour, explicitly document meaningful deltas, mark superseded material, and avoid introducing S3 scope.

## Core Rule

> Sprint controls delivery history; Git controls document history; `main` represents current engineering truth.

## Four Reader Layers

| Layer | Answers | Location |
| --- | --- | --- |
| Architecture | Why is the system shaped this way? | `architecture/`, `architecture/s2/` |
| Technical Design | How was S2 specified to work? | `design/s2/technical/`, `design/s2/ui-ux/` |
| ADR | Why was an important choice made? | `decisions/`, `decisions/s2/` |
| Delivery Evidence | What was delivered and released? | `delivery/s2/` |

Do not organise the final repo by Phase 1 / Phase 2 / Phase 3 process order. Phases are design process; directories are reader intent.

## Source-of-Truth Hierarchy

When documents conflict during closeout:

```text
1. Current implemented behaviour in the private application repo
2. Final Sprint 2 GitHub Issues / merged PR behaviour
3. Final UI/UX v2 specification
4. Final Phase 3 Technical Design
5. Phase 2 Architecture Design
6. Phase 1 Requirements
7. Older S1 / early S2 documentation
```

> Implementation can correct design documentation, but documentation must not invent implementation.

Distinguish:

- **implementation detail** — usually no architecture change
- **design delta** — update technical design / UI specs
- **architectural delta** — explicit architecture + ADR update

## Branch Model

```text
main
 │
 ├── docs/sprint-N
 │
 └── PR → main → tag engineering-sN
```

- Branch = Sprint isolation
- Commit = design / closeout checkpoint
- PR = documentation review gate
- Tag = released snapshot on `main` after merge

## Public Repo Boundary

The engineering repo must not become a mirror of private source code. Prefer requirements, architecture, design decisions, data/API concepts, failure behaviour, testing strategy, delivery process, and release evidence over class inventories and full implementation listings.

## Related

- [Development Workflow](development-workflow.md)
- [Repository Governance](repository-governance.md)
- [Definition of Done](../quality/definition-of-done.md)
