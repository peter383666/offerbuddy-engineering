# Architecture

Living documentation for OfferBuddy's **current** system architecture.

These documents answer:

> What is OfferBuddy now?

They are updated during each Sprint's documentation sync. Historical design rationale for a specific Sprint lives under `delivery/sN/architecture/` and is frozen after that Sprint closes.

| Document | Purpose |
| --- | --- |
| [System Context](system-context.md) | External actors and system boundary |
| [Container Design](container-design.md) | Deployable containers and major modules |
| [Data Model](data-model.md) | Current persistence model |
| [API Design](api-design.md) | Current public API contract |
| [OpenAPI](openapi.yaml) | Machine-readable API description |

Sprint 2 design-time architecture (how S2 was planned) is archived at [`delivery/s2/architecture/architecture-design.md`](../delivery/s2/architecture/architecture-design.md).

During Sprint 2 wrap-up, living documents above will be synced to the implemented Extension, Business Events, Job Intelligence, and Analytics state.
