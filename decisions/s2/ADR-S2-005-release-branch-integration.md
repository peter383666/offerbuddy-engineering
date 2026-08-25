# ADR-S2-005: Release-Branch Integration

## Status

Accepted — used for Sprint 2 delivery.

## Context

Sprint 2 spanned Extension, backend, frontend, migrations, and Admin-adjacent ops work. Feature branches needed a stable integration line before `main` without turning every merge into an immediate production deploy.

## Decision

Integrate Sprint 2 application work through a `release/sprint-2` line (and review branches as needed), then merge the verified increment to `main` for production deploy by explicit SHA. Engineering documentation closeout used a separate `docs/sprint-2` line, merged to engineering `main` and tagged `engineering-s2`. Application release evidence is recorded against application `main` / tag `v0.3.0`.

## Alternatives

- Trunk-only development with no release branch
- Long-lived environment branches per package
- Auto-deploy every merge to `main` without SHA gates

## Consequences

- Sprint integration can stabilize before production cutover
- Docs and app release tags remain separate repositories / tags
- Stale release/review branches should be deleted after merge to avoid drift

## Related

- [Sprint Review](../../delivery/s2/sprint-review.md)
- [Release Notes](../../delivery/s2/release-notes.md)
- [ADR-008](../ADR-008-single-host-production.md)
