# Sprint 2 Retrospective

## What Went Well

### Capture Friction Dropped Where It Matters

The Extension moved Application recording into the SEEK/Indeed browsing flow. Explicit Save, pairing, and Track API made the preferred path real without removing the Web fallback.

### Core / Async Separation Held

Business Events kept Job Intelligence and Analytics off the Core write path. Successful tracking does not wait on Gemini or projection freshness. That architectural bet survived implementation.

### Proportional Architecture Continued

Sprint 2 extended the modular monolith and PostgreSQL. No broker, no microservices, and no Redis dependency were required to deliver the increment.

### Adapter Isolation Paid Off

SEEK and Indeed knowledge stayed behind Site Adapters. Shared Extension infrastructure could evolve (companion, lifecycle, popup states) without collapsing into one platform-specific script.

### Verification Depth Improved in the Right Places

Backend Testcontainers coverage expanded into pairing, events, intelligence, and analytics. Extension Vitest covered adapters, lifecycle, and privileged boundaries. Frontend tests appeared for key S2 surfaces.

### Documentation Closeout Became Explicit

The dual-layer / reader-intent documentation model and reconciliation artifacts forced planned-vs-implemented honesty instead of polishing stale “not started” banners.

## What Did Not Go Well

### Tracking Model Docs Conflicted

UI v2 / Extension Design emphasised explicit Save, while the tracking specification and shipped code also implemented confirmation-based auto-ingest. Closeout had to accept dual paths after the fact rather than deciding one authority earlier.

### Eligibility Contract Was Incomplete Mid-Sprint

Requirements listed multiple eligibility concepts; UI and extractors settled on citizenship/PR only. The gap was known in the sprint plan watch items and still created documentation churn.

### CI Did Not Fully Catch Up to New Surfaces

Extension verification and Frontend Vitest exist locally but are not fully represented as GitHub Actions gates. Local precheck helps, but CI remains uneven across packages.

### Living Docs Lagged Implementation Again

Architecture and design status tables continued to say “not delivered” after the vertical existed. Sprint 1’s documentation-drift lesson recurred and required a dedicated closeout pass.

### Site Reality Remains Fragile

SEEK/Indeed DOM and SPA behaviour still demand manual current-site checks. Fixtures reduce risk; they do not eliminate production page drift.

## What We Learned

- Fast capture and asynchronous enrichment work as a product principle when Core and downstream failure domains stay separate.
- Extension UX authority must be decided before multiple specs diverge; otherwise implementation becomes the tie-breaker.
- Eligibility screening needs an explicit surfacing contract, not only a requirements list.
- Local package verify scripts are necessary but not a substitute for CI gates on every shippable client.
- Public engineering docs should record deltas and rationale, not mirror private class inventories.
- Confirmation-based tracking can coexist with explicit Save only if product messaging and README language stay aligned.

## What We Will Change

- Decide Extension UX authority earlier when multiple design drafts exist.
- Close eligibility/UI contracts before implementation issues that depend on them.
- Add Extension CI and Frontend test gates when capacity allows.
- Keep implementation status / reconciliation as a required closeout artifact each sprint.
- Treat current-site Extension validation as release-critical, not optional polish.
- Update living architecture during documentation sync rather than only at sprint end when practical.

## Stop / Start / Continue

### Stop

- Leaving contradictory UI/technical specs both marked “final”.
- Treating “Phase complete” documentation status as “implemented”.
- Assuming local `npm test` / `verify` without CI implies release-protected automation.

### Start

- Recording design deltas in a reconciliation table before rewriting living docs.
- Gating shippable clients (Extension, Frontend tests) in CI when they become release-critical.
- Keeping Sprint Design Records frozen and living docs current as separate responsibilities.

### Continue

- Modular monolith + PostgreSQL Business Events for proportional async work.
- Site Adapter isolation for recruitment platforms.
- Risk-focused backend Testcontainers coverage.
- Explicit-SHA production deployment and migration/rollback separation.
- Sprint-end Review, Retrospective, and engineering documentation tag.

## Sprint 3 Implications

Sprint 3 should not be designed by this retrospective. The clearest implications are:

- protect SEEK/Indeed capture reliability and Extension messaging clarity
- improve CI completeness for Extension/Frontend
- only expand platforms or companion intelligence with an explicit product decision
- keep Core/async separation as a non-negotiable architecture constraint

## Related

- [Sprint Review](sprint-review.md)
- [Known Limitations](known-limitations.md)
- [Deferred Items](deferred-items.md)
- [Implementation Reconciliation](implementation-reconciliation.md)
