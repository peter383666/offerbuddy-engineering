# Responsive and Accessibility Specification

## 1. Strategy

-   Web App: desktop-first, responsive down to mobile.
-   Browser Extension: dedicated compact Extension layout.
-   S2 does not require a mobile app or mobile-first redesign.

## 2. Responsive Behaviour

Use the existing implementation breakpoints where appropriate. Product
design defines behaviour, not arbitrary new pixel values.

### Desktop

-   Full navigation.
-   Multi-column layouts where useful.
-   Existing S1 desktop patterns are preserved.

### Tablet / Narrow Desktop

-   Reduce columns.
-   Preserve information hierarchy.
-   Avoid squeezing complex two-column content beyond readability.

### Mobile

-   Primary content becomes single-column where needed.
-   No normal horizontal page scrolling.
-   Lists/tables may reflow into stacked rows/cards.

## 3. Home

-   Snapshot metrics may remain three compact columns if readable.
-   Otherwise stack/reflow.
-   Recent Applications reflow rather than requiring horizontal table
    scrolling.

## 4. Applications

On narrow screens preserve: - title; - company; - applied date; -
current status.

Source/location may combine as secondary metadata.

## 5. Application Detail

-   Single structured page.
-   Sections stack naturally.
-   Skill chips wrap.
-   Long Job Intelligence content expands vertically.
-   Essential text must not be clipped.

## 6. Analytics

Desktop: - metrics row; - full-width trend; - Status + Conversion may
share a row.

Narrow/mobile: - stack Status and Conversion; - charts respond to
container width; - reduce axis-label density; - do not require
horizontal scrolling; - do not rely on hover.

## 7. Extension

-   Dedicated compact variants.
-   No horizontal scrolling.
-   Vertical overflow may scroll.
-   Long findings wrap.
-   Do not paste full Web cards into the popup.

## 8. Keyboard

All core actions must be keyboard usable.

Use semantic interactive elements: - button; - link; - select/menu; -
form controls.

Do not implement interactive `<div>` elements without equivalent
keyboard semantics.

Extension confirmation tab order should follow visual order:

1.  Yes, I applied
2.  Not yet
3.  Didn't apply

## 9. Focus

Every interactive element requires a visible focus treatment.

Includes: - navigation; - buttons; - links; - dropdowns; - Application
actions; - Extension controls; - interactive chart points where present.

Do not remove focus outlines without an accessible replacement.

## 10. Colour and Contrast

Colour must not be the only carrier of meaning.

Examples must include text/icon/numeric meaning:

``` text
⚠ Australian citizenship required
○ Sponsorship not mentioned
✓ Sponsorship mentioned
```

Analytics bars must retain status labels and counts.

Secondary grey text must maintain sufficient contrast against its
background.

## 11. Screen Reader Semantics

Application items should expose a meaningful grouped label/context
rather than unrelated text fragments.

Analytics examples: - `Applications, 42` -
`Application to Interview, 14.3 percent, 6 of 42 applications`

Charts must provide textual/numeric meaning independent of visual
geometry.

## 12. Dynamic Updates

Meaningful state changes should be available to assistive technology
without announcing every background operation.

Examples: - Tracking → Tracked - Tracking → Error - Job Intelligence
processing → Available

## 13. Motion

Use only subtle functional motion.

Do not add: - animated chart entrances; - confetti; - bouncing AI
indicators; - continuous pulsing status elements.

Respect `prefers-reduced-motion`.

## 14. Text Growth

Design for real content, not fixed Figma line counts.

-   Long findings wrap.
-   Responsibilities/requirements may grow.
-   Skill groups wrap.
-   Content must not overlap or clip when text is longer or browser text
    size increases.

## 15. Tooltip Rule

Tooltip is enhancement only.

Essential information must remain understandable without: - mouse
hover; - tooltip; - colour.

## 16. Accessibility Target

New and modified S2 surfaces should target **WCAG 2.2 AA-aligned**
interaction, contrast and semantic accessibility practices.

This is an engineering/design target, not an unaudited claim of formal
certification.
