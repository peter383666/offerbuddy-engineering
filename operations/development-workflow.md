# Development Workflow

## Purpose

This document describes how a change moves from development to a verified OfferBuddy release. Deployment detail is maintained in [Deployment Strategy](deployment-strategy.md).

## Workflow

```text
issue or defined task
  -> feature/*
  -> local verification
  -> pull request to release
  -> CI
  -> integration verification
  -> promotion to main
  -> immutable release tag
  -> explicit production SHA deployment
  -> production verification
```

## Feature Branch

A `feature/*` branch contains one focused change or coherent change set.

Before opening a pull request:

- meet the acceptance criteria
- run relevant backend tests
- run frontend lint/build when applicable
- update affected documentation
- identify migrations, configuration changes, and technical debt
- confirm no secrets or generated local files are included

Feature-branch pushes do not automatically run every path-filtered CI workflow. CI runs when a relevant pull request targets `release` or `main`, or through manual dispatch.

## Release Branch

`release` is the integration and verification branch for the next release candidate.

It is used to:

- combine approved changes
- run boundary CI
- create immutable candidate artifacts
- verify frontend/backend integration
- deploy a candidate SHA when production-like verification is required
- resolve release-blocking issues before promotion

Presence on `release` does not mean the code is production-ready.

## Main

`main` is the verified stable baseline. Changes reach it after release integration is accepted.

A merge to `main` causes relevant CI to produce main-associated immutable artifacts. Production still selects an explicit artifact SHA; it does not blindly follow the branch tip.

## Tag

A tag is an immutable release marker created from the accepted stable commit. It records the released source identity but does not itself replace deployment verification.

Sprint 1 tagging occurs only after documentation and sprint closure.

## Production

Deployment uses manually dispatched frontend/backend workflows and an explicit verified SHA whenever practical.

After deployment:

- check backend health
- check the HTTPS homepage
- run the relevant authentication/application smoke path
- inspect Nginx and container logs
- confirm persistence where the change affects data
- record the deployed version
- retain the previous known-good SHA

## Pull Requests and Review

A pull request should include:

- purpose and scope
- implementation summary
- evidence of relevant tests/builds
- database or configuration impact
- documentation impact
- known limitations and follow-up debt
- rollback considerations for production-affecting changes

Documentation-only changes should be committed separately from production-code cleanup when practical.

## CI Is a Gate, Not the Release Decision

CI verifies the configured build, tests, lint, and artifact generation for one commit. Integration verification and production verification remain explicit responsibilities because OAuth, Nginx, EC2, external job sites, Gemini, and persistent data cross boundaries not proven by ordinary CI.
