# Documentation Governance

## Purpose

This document defines how OfferBuddy engineering documentation is branched, structured, reviewed, and released.

## Core Rule

> Sprint controls delivery history; Git controls document history; `main` represents current engineering truth.

## Three Documentation Dimensions

| Location | Answers | Lifecycle |
| --- | --- | --- |
| `delivery/sN/` | How was Sprint N designed and delivered? | Frozen after Sprint closure |
| Top-level `architecture/`, `product/`, `quality/`, `operations/` | What is the system now? | Living; updated during documentation sync |
| `decisions/ADR-*` | Why was an important technical choice made? | Accepted ADRs are not rewritten in place |

Sprint Design Records may emphasise delta, decision, and rationale. Living documents emphasise final current state and should not accumulate parallel `*-s1` / `*-s2` filename versions.

## Branch Model

```text
main
 │
 ├── docs/sprint-N   ← entire Sprint documentation workspace
 │
 └── PR → main → tag engineering-sN
```

Rules:

- **Branch = Sprint-level isolation**
- **Commit = Sprint design checkpoint**
- **Directory = documentation domain/structure**
- **PR = Sprint documentation review / release gate**
- **Tag = released documentation snapshot on `main`**

Do not create per-phase Sprint branches (`docs/sN-requirements`, `docs/sN-architecture`, …) unless multiple authors must review those phases independently.

## Sprint Documentation Lifecycle

```text
1. Create docs/sprint-N from main
2. Requirements / Architecture / Technical Design / UI/UX / Sprint Plan
3. Implementation (application repository)
4. Documentation sync with implemented reality
5. Sprint Review and Retrospective
6. Final documentation review
7. PR docs/sprint-N → main
8. Merge
9. Tag engineering-sN on main
10. Delete docs/sprint-N
11. Start Sprint N+1
```

Tag only after merge so `engineering-sN` always points at the accepted `main` snapshot.

## Directory Expectations

```text
delivery/sN/
├── README.md
├── requirements/
├── architecture/
├── technical-design/
├── ui-ux/                  # when applicable
├── sprint-plan.md
├── sprint-review.md
└── retrospective.md
```

## Related

- [Development Workflow](development-workflow.md)
- [Repository Governance](repository-governance.md)
- [Definition of Done](../quality/definition-of-done.md)
