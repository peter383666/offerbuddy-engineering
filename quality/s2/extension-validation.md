# Sprint 2 Extension Validation

## Purpose

How the Browser Extension is validated for Sprint 2 closeout.

## Automated

From `extension/`:

```text
npm test
npm run verify   # tests + production build
```

Coverage includes SEEK/Indeed adapters, lifecycle/tracking, companion behaviour, pairing/save orchestration, popup states, and credential-boundary rules.

There is **no dedicated Extension GitHub Actions workflow** at closeout. Extension verification is local / precheck until a workflow is added.

## Manual Platform Checks

Current-site checks remain mandatory because SEEK/Indeed DOM and SPA behaviour drift.

Minimum:

1. Load `extension/dist` via Chrome Developer Mode.
2. Confirm SEEK full-page and side-panel detection.
3. Confirm Indeed full-page and side-panel detection.
4. Confirm capture failure when required identity/title is missing.
5. Confirm citizenship/PR review signalling; generic working-rights wording must not falsely trigger PR finding.
6. Complete pairing and Save against the intended backend environment.
7. Confirm duplicate / auth-required / failure outcomes are understandable.
8. Confirm companion quiet default and eligibility attention behaviour on supported surfaces.

Detailed operator notes live in the private extension README; this public doc records the validation expectation without mirroring source structure.

## Packaging

- `npm run build:local` — local backend/frontend origins
- `npm run build` — production origin pack for offerbuddy.io

Chrome Web Store publishing steps, store listing copy, and review status are operational release work; see [Extension Publishing](../../operations/s2/extension-publishing.md).

## Related

- [Test Strategy](test-strategy.md)
- [Implementation Status](../../delivery/s2/implementation-status.md)
