# Current Scope

Living summary of what OfferBuddy currently includes.

## Status

Sprint 2 implementation is complete in the application repository. Engineering documentation closeout is released on `main` as tag `engineering-s2`.

## Included

- Google sign-in and server-managed Web sessions
- Manual and AI-assisted URL application capture (secondary path)
- Application list, search, filter, sort, pagination, detail, update, and deletion
- Browser Extension capture for SEEK and Indeed as the preferred capture path
- Extension pairing and authenticated Extension tracking
- Application creation-source and status history
- Asynchronous Job Intelligence enrichment
- Basic Application Analytics
- PostgreSQL persistence with Flyway migrations
- Production HTTPS deployment on a single EC2 host

## Not Included

- LinkedIn or broad multi-platform capture
- Auto Apply / automatic form submission
- Resume or Cover Letter generation
- Candidate/Job match scoring
- Large Analytics/BI expansion
- Microservices, message brokers, or Kubernetes

See [Product Vision](product-vision.md), [Roadmap](roadmap.md), and the Sprint Design Records under [`delivery/`](../delivery/).
