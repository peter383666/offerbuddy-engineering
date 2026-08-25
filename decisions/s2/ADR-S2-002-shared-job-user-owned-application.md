# ADR-S2-002: Shared Job / User-Owned Application

## Status

Accepted — implemented in Sprint 2.

## Context

Multiple users (and one user via Web or Extension) may encounter the same external posting. Application lifecycle and ownership must stay per-user while posting facts remain reusable.

## Decision

Treat Job as shared posting facts keyed by `sourcePlatform + externalJobId`, with selective source-fact refresh that does not null-erase known fields. Treat Application as one authenticated user's relationship and lifecycle with that Job. Create-or-reuse must not reset an existing Application status. Record `creation_source` as `WEB` or `EXTENSION`. The backend never trusts a client-supplied `userId` as ownership proof.

## Alternatives

- Duplicate full Job rows per user/Application
- Allow the client to assert ownership via payload fields
- Reset Application status on every re-track

## Consequences

- Core tracking stays consistent across Web and Extension
- Shared Job data does not imply unrestricted API visibility
- Status history and Analytics can reason over real transitions

## Related

- [Data Architecture](../../architecture/s2/data-architecture.md)
- [Database Design](../../design/s2/technical/database-design.md)
