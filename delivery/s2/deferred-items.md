# Sprint 2 Deferred Items

## Purpose

Work intentionally left out of Sprint 2 completion or deferred after closeout. Items here are not implied Sprint 3 commitments until refined and accepted into a later sprint.

## Product Deferrals

- LinkedIn and additional recruitment platforms
- Broader eligibility surfacing beyond citizenship/PR
- Auto Apply / automatic form submission
- Cover Letter generation, resume tailoring, candidate/Job match scoring
- Large Analytics/BI expansion
- Notifications, email status detection, interview tooling

## Engineering Follow-Ups

- Merge Extension CI + Frontend Vitest CI gate (`review/s2-ci-followups`) to application `main`
- Legacy `NO_RESPONSE` status migration vs long-term retention decision
- Stronger request correlation / observability if revisit triggers are met
- Staging environment and managed secret storage (carried from earlier debt themes)
- Next Chrome Web Store package upload after bumping `manifest.json` above `0.1.24`

## Documentation Follow-Ups After Tag

Completed in post-tag sync:

- Extract concise `decisions/s2/ADR-S2-*` records
- Mark `ui-ux/v1/` superseded in-file
- Record Chrome Web Store publication outcome in release notes

Optional polish:

- Expand Authentication / Security and Failure Handling extracts under `design/s2/technical/` if readers need more than architecture pointers

## Related

- [Known Limitations](known-limitations.md)
- [Product Backlog](../product-backlog.md)
- [Sprint 1 Technical Debt](../s1/technical-debt.md)
