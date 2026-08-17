# Sprint 1 Retrospective

## What Went Well

### End-to-End Product Delivery

The sprint moved OfferBuddy from an engineering foundation to a usable production workflow. Authentication, capture, application creation, search/filter/pagination, detail, update, home, persistence, and deployment were completed across the frontend and backend.

### Security and Data Ownership

Google OIDC and Spring Security provided a focused authentication scope. User identity is derived from the server session, and application queries are user-scoped rather than trusting client-supplied ownership.

### Backend Test Depth

Business services, controllers, security, PostgreSQL repositories, transactions, OpenAPI, job-page acquisition, AI parsing, and production configuration received targeted tests. Testcontainers kept important persistence tests representative of production PostgreSQL.

### Production Hardening

Nginx/HTTPS, Docker Compose, health checks, independent CI/CD, persistent PostgreSQL, backup/restore verification, container restart, EC2 reboot recovery, and immutable-SHA rollback were exercised rather than left as architecture diagrams.

### Proportional Architecture

The modular monolith and single-host deployment kept delivery understandable. Sprint 1 did not introduce microservices, Kubernetes, Kafka, or other infrastructure without a current requirement.

## What Did Not Go Well

### Content Acquisition and AI Were Too Tightly Coupled

The synchronous parsing request combines job-page retrieval and AI semantic extraction. A blocked or incomplete page can fail before AI adds value, while provider latency affects the same user request. Production experience made these two responsibilities more distinct than the original design suggested.

### Manual Capture Still Has Friction

Manual entry preserves product availability but does not solve the effort of copying job information. This limits the product's advantage when server-side page retrieval cannot access a listing.

### Production Configuration Required Iteration

Local success did not guarantee production correctness. OAuth redirect schemes/hosts, canonical-domain handling, cookie behaviour, and Nginx path forwarding required production-specific review. In particular, Nginx had to preserve the `/api` prefix when proxying requests.

### Release Identity Needed Tightening

Frontend and backend initially treated “latest” differently. The workflow was corrected to resolve branch-specific successful CI SHAs, while explicit full SHA remained the preferred deployment identity.

### Frontend Automated Testing Did Not Land

Frontend CI verifies lint, types, and production compilation, but critical session, CSRF, list, and create behaviour still depends on manual testing.

### Documentation Drifted Behind Implementation

Planning documents continued to describe dashboard statistics, status history, old API field names, and planned production work after the implementation had changed. Closing documentation required a substantial reconciliation pass.

## What We Learned

- Deterministic content acquisition and AI semantic analysis are separate responsibilities and failure domains.
- Production deployment reveals proxy, OAuth, cookie, DNS, and filesystem issues that local integration does not reproduce.
- A database dump is not enough; restoration must be exercised.
- Immutable build artifacts make version identification and rollback simpler.
- CI passing proves configured automation for one commit, not complete integration or production readiness.
- Application rollback does not reverse database migrations or data changes.
- Same-origin Nginx deployment simplifies session cookies, CSRF, OAuth callbacks, and frontend API configuration.
- Architecture should remain proportional to the product and team scale.
- AI-assisted implementation still requires code review, documentation reconciliation, and developer understanding of security, transactions, persistence, and operations.

## What We Will Change

- Separate page/content acquisition more clearly from AI semantic extraction in future design work.
- Prioritise reduced capture friction when planning Sprint 2.
- Keep explicit SHA/version selection as the normal production deployment path.
- Add release-candidate verification criteria before promotion to `main`.
- Maintain an active technical debt register rather than hiding gaps in planning documents.
- Introduce frontend automated tests for the highest-risk session and application flows.
- Improve request correlation, logging, backup location/automation, and secret management when their revisit triggers are met.
- Continue a sprint-end engineering review, documentation reconciliation, Review, Retrospective, and release decision.

## Stop / Start / Continue

### Stop

- Treating planning documentation as current after implementation has changed.
- Using an ambiguous mutable artifact name as the normal production identity.
- Treating a successful dump command or CI run as complete recovery/release evidence.
- Expanding infrastructure for portfolio appearance rather than a measured need.

### Start

- Tracking documentation alignment as part of implementation completion.
- Adding focused frontend tests for authentication, CSRF, and critical forms.
- Recording the deployed frontend and backend SHA during verification.
- Separating acquisition failures, AI failures, and user-facing fallback behaviour in design and testing.

### Continue

- Risk-focused backend tests with PostgreSQL Testcontainers.
- Manual release control and production smoke testing.
- Restore verification, restart checks, and recovery exercises.
- Immutable artifacts and known-good-SHA rollback.
- Modular-monolith and single-host architecture while current scale supports them.

## Sprint 2 Implications

Sprint 2 should not be fully designed by this retrospective. The clearest implications are:

- Browser extension capture is the primary candidate for reducing acquisition and manual-entry friction.
- Analytics is a secondary candidate rather than a prerequisite for capture improvements.
- AI should focus more on semantic understanding than being responsible for raw content acquisition.
- Technical debt work must be selected explicitly alongside product scope.

These are future directions, not delivered features or a committed Sprint 2 backlog.
