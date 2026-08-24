# Sprint 2 Extension Publishing

## Purpose

Records the public operational expectation for packaging and distributing the OfferBuddy Browser Extension.

## Packaging

From the private application repository `extension/` package:

| Build | Intent |
| --- | --- |
| `npm run build:local` | Local backend/frontend origins for unpacked development |
| `npm run build` | Production pack targeting offerbuddy.io |
| `npm run verify` | Tests + production build |

Load unpacked from `extension/dist` during development and validation.

## Distribution

Sprint 2 delivers a Chrome Manifest V3 Extension for SEEK and Indeed.

Chrome Web Store listing, screenshots, privacy disclosures, review submission, and store rollout are release/operations activities. Store assets for popup states are retained under [`assets/screenshots/s2/extension/`](../../assets/screenshots/s2/extension/).

This engineering repo does not claim a specific store publication date or review outcome unless recorded later in release notes.

## Security Expectations

- Credentials stay in privileged Extension storage / service worker boundaries
- Content scripts and Site Adapters must not receive Extension credentials
- Production host permissions should remain least-privilege for approved origins

## Related

- [Extension Validation](../../quality/s2/extension-validation.md)
- [Browser Extension Architecture](../../architecture/s2/browser-extension-architecture.md)
