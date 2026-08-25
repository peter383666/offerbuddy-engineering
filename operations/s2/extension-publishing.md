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

Load unpacked from `extension/dist` during development and validation. Store version is the Chrome `manifest.json` `version` field (not the npm package version).

## Chrome Web Store

| Item | Value |
| --- | --- |
| Listing | [OfferBuddy on Chrome Web Store](https://chromewebstore.google.com/detail/offerbuddy/ihdknldekiocanohajkgebmnhnneoeka) |
| Item ID | `ihdknldekiocanohajkgebmnhnneoeka` |
| Published package (S2 closeout) | Manifest `0.1.24` |

Store screenshots and popup-state assets for docs live under [`assets/screenshots/s2/extension/`](../../assets/screenshots/s2/extension/).

## Automated publish (application repo)

GitHub Actions workflow **Extension Publish** (`workflow_dispatch`):

1. `npm run verify` in `extension/`
2. Zip `extension/dist`
3. Upload to the existing Chrome Web Store item
4. Mode `upload` — upload only; mode `publish` — upload and submit for review

Required repository secrets: `CHROME_CLIENT_ID`, `CHROME_CLIENT_SECRET`, `CHROME_REFRESH_TOKEN`. Optional: `CHROME_EXTENSION_ID` (defaults to the item id above).

Use a dedicated OAuth Web client for store API access (not the production Google Login client). Google still reviews each submission; the workflow does not bypass store review. Uploads must bump `manifest.json` version above the currently published package.

## Automated verify CI

**Extension CI** runs `npm run verify` on Extension path changes (application `main` / `release` and PRs). Landing of that workflow on application `main` is tracked with the S2 CI follow-up branch when not yet merged.

## Security Expectations

- Credentials stay in privileged Extension storage / service worker boundaries
- Content scripts and Site Adapters must not receive Extension credentials
- Production host permissions should remain least-privilege for approved origins
- Store publish OAuth secrets stay in GitHub Actions secrets only

## Related

- [Extension Validation](../../quality/s2/extension-validation.md)
- [Browser Extension Architecture](../../architecture/s2/browser-extension-architecture.md)
- [Release Notes](../../delivery/s2/release-notes.md)
