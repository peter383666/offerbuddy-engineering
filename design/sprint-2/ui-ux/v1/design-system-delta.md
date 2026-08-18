# S2 Design System Delta

## 1. Principle

S2 extends the existing OfferBuddy Design System.

Do not create a separate S2 visual system.

When an existing S1 component can satisfy an S2 requirement through
composition or a small variant, reuse it. Do not create a new component
solely because the feature belongs to S2.

## 2. Reuse Existing Foundations

Reuse where already defined:

-   typography;
-   spacing;
-   radius;
-   borders;
-   shadows;
-   page/container layout;
-   buttons;
-   inputs;
-   selects/dropdowns;
-   links;
-   cards;
-   navigation;
-   Application status badges.

## 3. Visual Direction

Continue: - black/white-led interface; - restrained accent usage; -
professional, simple hierarchy; - typography-first presentation.

Do not introduce: - new brand palette; - AI gradients; -
robot/magic-wand/sparkle visual language; - colourful BI-dashboard
styling; - separate Extension branding; - new typography system.

## 4. Semantic Feedback

Define/confirm presentation semantics for:

-   Positive
-   Attention
-   Neutral / Information
-   Error

These are separate from Application lifecycle statuses.

Example:

-   Positive → `✓ Tracked in OfferBuddy`
-   Attention → `⚠ Australian citizenship required`
-   Neutral → `○ Sponsorship not mentioned`
-   Error → `Couldn't track application`

## 5. Finding Item

Reusable structure:

``` text
[Icon] Finding text
       Optional supporting text
```

Variants: - Attention - Positive - Neutral

Provide: - standard Web density; - compact Extension density.

Meaning must remain the same across surfaces.

## 6. Application Status

Continue using the existing S1 status system for:

-   Applied
-   Interview
-   Offer
-   Rejected
-   Withdrawn
-   No Response

Do not create a second Analytics-specific status system.

## 7. Source Metadata

Provide a lightweight source presentation for: - SEEK - Indeed -
existing/manual source semantics

Prefer neutral metadata or an existing subtle badge.

Do not heavily reproduce external platform branding.

## 8. Link Patterns

External: `View original job posting ↗`

Internal: `View Application →`

Define/reuse a consistent external-link affordance.

## 9. Metric Summary / Metric Card

Required for Home and Analytics.

Structure:

``` text
Label
Value
```

Examples: - Applications / 42 - Interviews / 6 - Offers / 1

Support standard and compact layout where necessary.

Do not add unused S2 variants such as: - trend delta; - sparkline; -
progress; - comparison; - icon metric.

## 10. Analytics Primitives

Only the following S2 chart primitives are required:

### Trend Chart

For Applications Over Time.

### Horizontal Status Bar

For current Application Status.

### Conversion Metric

Structure:

``` text
Application → Interview
14.3%
6 of 42 applications
```

Do not add S2 chart-system requirements for: - pie; - donut; - funnel; -
radar; - heatmap; - stacked chart.

## 11. Chart Colour

-   Use a restrained palette.
-   Trend chart is a single primary series.
-   Status meaning must not depend on colour.
-   Existing status accents may be used subtly.
-   Labels and numeric values remain visible.

## 12. Section Header

Reuse/standardise a consistent section-heading hierarchy for:

-   Application
-   Job Information
-   Job Intelligence
-   Applications Over Time
-   Application Status
-   Conversion

Do not allow each feature to invent its own heading treatment.

## 13. Job Intelligence

Job Intelligence is a normal content section.

Do not create a large special `AI Card` with: - gradients; -
`Powered by AI`; - sparkle decoration; - model branding.

## 14. Skill Chip

Reuse an existing neutral Tag/Chip if available.

If absent, add a basic neutral chip suitable for: - Java - Spring Boot -
AWS - PostgreSQL

Do not imply proficiency levels or categories that are not part of the
data semantics.

## 15. Section-Level Async State

Define/reuse a general section-state pattern for: - loading; -
empty/insufficient; - error/recovery.

Do not create AI-specific loading infrastructure solely for Job
Intelligence.

## 16. Skeleton

Support where appropriate: - Metric skeleton - Chart skeleton -
Content-section skeleton

Extension should prefer compact textual state over filling the popup
with large skeleton blocks.

## 17. Extension Compact Surface

Define compact-surface rules:

-   tighter spacing;
-   same typography family;
-   same semantic icons;
-   same button hierarchy;
-   same radius language;
-   vertical overflow allowed;
-   no horizontal overflow.

Compact is a density variant, not a separate brand system.

## 18. Extension Injected Status Entry

Required S2-specific component/pattern.

Responsibilities: - identify OfferBuddy; - show concise state; - provide
review/entry action when needed.

Example:

``` text
OfferBuddy
⚠ 2 things to review
Review →
```

Do not expand it into a mini Application Detail.

## 19. Extension Action Hierarchy

Confirmation: - Primary → Yes, I applied - Secondary → Not yet -
Tertiary/text → Didn't apply

Platform evidence: - Primary → Track in OfferBuddy

## 20. Icons

Required/likely: - Attention - Positive/check - Neutral/info - External
link - Chevron - Retry/refresh where already consistent - navigation
icons if existing navigation uses icons

Do not introduce decorative AI iconography as a requirement.

## 21. Responsive Variants

Explicitly support: - Metric standard/compact - Application row
desktop/stacked - responsive chart container - Finding
standard/compact - wrapping Skill Chip group

Extension compact variants are not implemented by pretending the popup
is merely another Web breakpoint.

## 22. Focus

If the existing Design System does not define a clear visible focus
treatment, S2 must add/clarify one.

Focus treatment applies consistently across Web and Extension.
