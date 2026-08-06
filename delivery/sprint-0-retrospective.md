# Sprint 0 Retrospective

## Sprint Objective

Reflect on the Sprint 0 engineering process, identify what worked well, determine what can be improved, and define actionable improvements for future sprints.

---

## What Went Well

### Engineering Foundation

- Successfully established the initial engineering foundation.
- Created clearly separated React frontend and Spring Boot backend projects.
- Made local PostgreSQL and Redis services reproducible through Docker Compose.
- Integrated PostgreSQL with the backend.
- Established version-controlled database migrations using Flyway.
- Documented the modular monolith architecture and its initial module boundaries.

### Development Workflow

- Adopted GitHub Flow using feature branches and pull requests.
- Tracked planned delivery work through GitHub Issues.
- Used pull requests to integrate the Sprint 0 deliverables.
- Introduced continuous integration for backend verification and frontend linting and builds.
- Kept the main branch aligned with the completed Sprint 0 increment.

### Documentation

- Developed engineering documentation alongside implementation.
- Kept product, architecture, technology, quality, and delivery decisions aligned.
- Recorded significant architecture choices through Architecture Decision Records.
- Corrected the MVP definition when AI-assisted job extraction became a confirmed requirement.
- Established a Product Backlog and project-level Definition of Done during Sprint Closing.

---

## What Didn't Go Well

### Branch Management

Work for Issue #4 was initially committed to the main branch before the intended feature branch workflow was followed.

This reduced the opportunity for isolated review and demonstrated that the active branch must be checked before implementation begins.

### Technology Evaluation

Spring Boot 4 required additional Flyway PostgreSQL configuration that was not identified during initial planning.

Compatibility between framework, migration tooling, and database-specific modules should have been validated earlier.

### Continuous Integration

The first CI pipeline did not pass immediately. Initial failures included:

- Missing Linux executable permission for the Maven Wrapper
- Frontend TypeScript strict-check failures

The workflow required additional iterations before it became stable.

### Sprint Scope

Future AI extraction testing tasks were incorrectly included in Sprint 0 even though the feature was not yet being implemented.

Those scenarios were subsequently moved into the testing strategy and Product Backlog. This preserved the identified quality work without misrepresenting it as committed Sprint 0 scope.

### Testing Foundation

The generated backend context-load test provided only a scaffold, not a complete backend testing foundation.

The frontend established linting and build checks but did not introduce Vitest, React Testing Library, `jsdom`, shared test setup, or a representative component test.

---

## Lessons Learned

- Verify the current Git branch before making any implementation change.
- Create and link the Issue and feature branch before development begins.
- Validate major framework and integration compatibility through a small technical spike.
- Keep the first CI pipeline focused on essential, repeatable checks.
- Run CI-equivalent commands locally before opening a pull request.
- Keep Sprint scope focused on the Sprint Goal and move future work into the Product Backlog.
- Distinguish between generated test scaffolding and a usable testing foundation.
- Treat Review, Retrospective, and engineering documentation updates as separate closing activities.

---

## Action Items

| Action | Target | Status |
|---|---|---|
| Protect the main branch | Sprint 1 | Planned |
| Introduce GitHub Issue templates | Sprint 0 Closing | Planned |
| Introduce a pull request template | Sprint 0 Closing | Planned |
| Define the project Definition of Done | Sprint 0 Closing | Completed |
| Document the OfferBuddy development workflow | Sprint 0 Closing audit | Proposed |
| Introduce the backend testing foundation | Sprint 1 | Planned |
| Introduce the frontend testing foundation | Sprint 1 | Planned |
| Add a pre-development branch verification step | Sprint 1 | Planned |
| Evaluate unfamiliar framework integrations through technical spikes | From Sprint 1 | Planned |

Action items marked as planned or proposed must be refined into backlog items or repository changes before they are treated as committed work.

---

## Overall Assessment

Sprint 0 achieved its primary objective.

The engineering foundation is ready to support iterative business development beginning in Sprint 1. The retrospective identified concrete improvements to repository governance, testing, technical evaluation, and scope management that will be addressed through Sprint Closing or future sprint planning.
