# Engineering Principles

## Documentation and Implementation Stay Aligned

Document intended behaviour before implementation where it improves decisions, then update the documentation to describe what was actually delivered.

## Small Iterations

Deliver working software in small, reviewable increments.

## Reviewable Delivery Flow

Use one focused issue or task, one feature branch, and one reviewable pull request. Integrate through `release`, promote verified work to `main`, and deploy an explicit immutable version.

## Continuous Integration

Relevant pull requests must pass CI before merging. CI is an automated gate; integration and production verification remain separate responsibilities.

## Living Documentation

Engineering documentation evolves together with the implementation.

## Sprint Closing

Every sprint ends with:

- Sprint Review
- Sprint Retrospective
- Engineering documentation update
- Release
