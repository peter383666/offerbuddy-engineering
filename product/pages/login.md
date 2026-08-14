# Login Page Specification

## Overview

| Item | Value |
|---|---|
| Page | Login |
| Status | Design approved for implementation |
| Primary purpose | Authenticate a user with Google |
| Primary user | A job seeker who is not signed in |
| Figma page | `FINAL — Login O+Check` |
| Source frame | `OfferBuddy — Final Login — Desktop` (`114:3`) |
| Reference viewport | 1440 × 1024 px |

## Purpose

The Login page gives users a single, low-friction way to enter OfferBuddy while reinforcing the product value proposition and explaining how Google account data is used.

## User Goal

> Sign in securely and continue to OfferBuddy with minimal effort.

## Page Structure

The desktop layout is divided into two full-height regions:

1. **Authentication panel** — white background, 820 px wide.
2. **Product progress panel** — light-grey background, 620 px wide.

The reference design is fixed at 1440 × 1024 px. The implementation must preserve this visual proportion on large screens while using responsive CSS rather than absolute page-level positioning.

## Authentication Panel

### Brand mark

- Display the OfferBuddy O+check brand mark near the top-left.
- Reference size: 62 × 62 px.
- The mark is a decorative brand image and should use an empty alternative text when the product name is available elsewhere to assistive technology.

### Google authentication action

- Label: **Continue with Google**
- Reference size: 470 × 76 px.
- White background, 1 px neutral border, 12 px corner radius.
- Include the Google `G` mark on the left and centre the action label visually.
- Use a semantic `button`, not a generic container.
- The button must expose visible hover, focus, pressed, disabled, and loading states.
- Activating it starts the Google authentication flow.
- Prevent duplicate authentication requests while loading.

### Privacy explanation

- Heading: **Private by default**
- Supporting copy: **Google is used only to authenticate your account.**
- Present the message in a light-grey card with a privacy icon.
- Reference size: 470 × 124 px with a 12 px corner radius.
- The privacy message is informational and must not be interactive.

## Product Progress Panel

### Introductory content

- Eyebrow: **FROM APPLICATION TO OFFER**
- Heading: **See your progress at a glance.**
- Supporting copy: **A clear timeline for every role you care about.**

### Progress timeline

Display a vertical three-step example timeline. The timeline is illustrative and must not be exposed as real user application data.

| Company | Position | Status | Status treatment |
|---|---|---|---|
| Canva | Software Engineer | Applied | Neutral grey |
| Atlassian | Backend Engineer | Interview | Blue |
| Google | Software Engineer | Offer ✓ | Green |

Each example application card contains:

- A company identifier tile.
- Company name.
- Position title.
- Status badge.

The cards use a white background, neutral border, and 14 px corner radius. A vertical neutral line connects three coloured status markers.

### Closing message

- Copy: **Stay focused. Know what comes next.**
- Display below the timeline and centre it within the product panel.

## Frontend Component Inventory

The recommended component breakdown is:

- `LoginPage`
- `BrandMark`
- `GoogleSignInButton`
- `PrivacyNotice`
- `ProductProgressPanel`
- `ApplicationProgressTimeline`
- `ApplicationPreviewCard`
- `StatusBadge`

`ApplicationPreviewCard` and `StatusBadge` should be data-driven so the three examples do not require duplicated markup.

## Required Exported Assets

Only the following visual elements require exported files from Figma:

| Asset | Suggested filename | Format | Reference size | Purpose |
|---|---|---|---|---|
| OfferBuddy O+check mark | `offerbuddy-mark.svg` | SVG | 62 × 62 px | Product branding |
| Applied timeline marker | `timeline-applied.svg` | SVG | 16 × 16 px | Neutral timeline state |
| Interview timeline marker | `timeline-interview.svg` | SVG | 16 × 16 px | Interview timeline state |
| Offer timeline marker | `timeline-offer.svg` | SVG | 16 × 16 px | Successful timeline state |
| Privacy icon circle | `privacy-icon.svg` | SVG | 48 × 48 px | Privacy notice |

The Figma frame contains no raster source images. Download and commit the exported SVG files to the frontend repository; do not use Figma's temporary asset URLs in production.

The Google sign-in logo should preferably come from the authentication provider's official branding asset or supported sign-in component. If the product uses the Figma rendering, verify it against Google's current branding requirements before release.

## Elements That Must Be Implemented in Code

Do not export the following elements as images:

- Page backgrounds and split layout.
- All text and typography.
- Google sign-in button container.
- Privacy notice card.
- Timeline line.
- Application preview cards.
- Company identifier tiles and their text glyphs.
- Status badges.
- Borders, corner radii, spacing, and colours.

These elements must remain responsive, accessible, and easy to update through HTML, reusable frontend components, and CSS/design tokens.

## Visual Tokens

### Typography

- Font family: Inter.
- Primary heading: 40 px, 47 px line height, semi-bold.
- Eyebrow: 14 px, semi-bold, uppercase.
- Body copy: 14–17 px, regular.
- Card title: 18 px, semi-bold.
- Button label: 18 px, medium.
- Status label: 14 px, medium.

### Colours

| Role | Value |
|---|---|
| Primary text | `#1F1F1F` |
| Secondary text | `#6B6B6B` |
| Product panel background | `#F7F7F7` |
| Border | `#D9D9D9` / `#E1E1E1` |
| Brand green | `#22A06B` |
| Interview blue | `#1677FF` |
| Offer green | `#178A5C` |
| Applied badge background | `#F0F0F0` |
| Interview badge background | `#EAF3FF` |
| Offer badge background | `#E6F6EE` |

### Shape and spacing

- Authentication button and privacy card radius: 12 px.
- Application card radius: 14 px.
- Status badge radius: 15 px.
- Timeline width: 2 px.
- Use the project spacing scale where possible rather than copying every absolute Figma coordinate.

## Behaviour

### Initial state

- The Google sign-in button is enabled.
- No error message is shown.

### Loading state

- Disable the Google sign-in button.
- Show a progress indicator and an accessible loading label.
- Preserve the page layout to avoid content movement.

### Authentication success

- Complete the application authentication/session flow.
- Redirect the user to the authenticated landing page.

### Authentication failure

- Keep the user on the Login page.
- Display a clear, non-technical error message close to the authentication action.
- Return focus to the error message or authentication region.
- Allow the user to retry.

### Existing session

- If the user already has a valid session, bypass the Login page and redirect to the authenticated landing page.

## Accessibility Requirements

- All interactive controls must be keyboard accessible.
- Show a high-contrast visible focus indicator.
- The Google action must have an accessible name matching its visible label.
- Decorative timeline markers should be hidden from assistive technology.
- Expose the example progress content as a meaningful ordered list or mark the complete panel decorative if it is intentionally excluded from the reading flow.
- Do not rely on badge colour alone; preserve the visible status labels.
- Maintain WCAG AA text contrast.
- Respect reduced-motion preferences if loading or transition animation is added.

## Responsive Behaviour

### Desktop (≥ 1024 px)

- Use the two-column layout shown in Figma.
- Keep the authentication content comfortably centred within the left panel.
- Keep the progress panel at approximately 43% of the viewport width, with a practical minimum width for its cards.

### Tablet and mobile (< 1024 px)

- Prioritise authentication content.
- Stack the product panel below the authentication panel or hide its illustrative timeline when vertical space is constrained.
- Keep the Google sign-in button and privacy card within the viewport with 24 px minimum horizontal page padding.
- Use a full-width button up to its 470 px maximum width.
- Do not preserve desktop absolute coordinates.

## Security and Privacy Requirements

- Use the backend-supported Google OAuth/OIDC flow; do not implement authentication solely in frontend code.
- Do not expose client secrets in the frontend bundle.
- Validate OAuth state and redirect destinations.
- Avoid storing identity tokens in insecure browser storage.
- Ensure the privacy statement remains accurate for the implemented data flow.

## Analytics

If product analytics is enabled, record only non-sensitive events such as:

- `login_viewed`
- `google_sign_in_started`
- `google_sign_in_succeeded`
- `google_sign_in_failed`

Do not include identity tokens, OAuth codes, email addresses, or other personal data in analytics payloads.

## Acceptance Criteria

- The page visually matches Figma frame `114:3` at 1440 × 1024 px.
- The user can start Google authentication from a semantic, keyboard-accessible button.
- Loading, success, failure, and existing-session paths are handled.
- The five required SVG assets are stored locally in the frontend project.
- No production code references expiring Figma asset URLs.
- Layout and decorative UI are implemented in code rather than exported as a page image.
- The mobile layout remains usable without horizontal scrolling.
- Automated tests cover button interaction, loading/disabled behaviour, failure messaging, and redirect behaviour.

## Implementation Notes for AI Coding Agents

- Treat Figma-generated React/Tailwind output as visual reference only.
- Adapt the page to the frontend repository's existing framework, routing, component library, styling conventions, and design tokens.
- Do not add Tailwind unless the frontend already uses it.
- Prefer semantic layout primitives and responsive CSS over Figma's absolute coordinates.
- Reuse the project's existing button, card, typography, and authentication components where available.
- Implement the timeline from structured data rather than three hard-coded component copies.
- Use the five exact exported SVG assets; do not redraw their vector paths.
- Use the final frame `114:3` as the source of truth. Ignore loose text nodes located outside that frame on the Figma Page.
