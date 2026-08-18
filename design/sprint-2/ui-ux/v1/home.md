# Home Page S2 Specification

## 1. Purpose

Home is a lightweight job-search snapshot and recent-activity surface.

It is not: - the Analytics Dashboard; - the primary New Application
flow; - a Job Intelligence dashboard.

S1's prominent Paste Job URL entry is no longer the centre of Home in S2
because the Browser Extension is the preferred SEEK/Indeed capture path.

## 2. Page Structure

``` text
Home

Applications     Interviews     Offers
     82               9             2

[ Optional Extension discovery ]

Recent Applications                         View all →
...
```

## 3. All-Time Snapshot

Show three all-time metrics:

-   Applications
-   Interviews
-   Offers

No time-range selector on Home.

Semantics: - Applications = tracked Applications. - Interviews =
Applications that have ever reached Interview. - Offers = Applications
that have ever reached Offer.

Do not interpret Interviews/Offers only from current status.

Do not add: - conversion rates; - trend charts; - status distribution; -
previous-period comparison.

Those belong to Analytics.

## 4. Extension Discovery

A lightweight Extension discovery section may be shown when relevant.

Preferred behaviour: - if reliable installation state is available at
low complexity, hide the discovery section when installed; - if
installation state is unknown, do not assert that the Extension is not
installed.

Unknown must not be treated as not installed.

Do not make reliable Web-to-Extension installation detection a mandatory
S2 architecture dependency solely for this banner.

Example discovery copy:

``` text
Capture jobs while you browse

Use OfferBuddy with SEEK and Indeed to capture
job details and review key requirements.
```

The exact installation/navigation action depends on the approved
distribution mechanism.

## 5. Recent Applications

Recent Applications is the primary Home content after the snapshot.

Show approximately the most recent useful set (for example, five
rows/cards; exact count may follow the existing S1 implementation).

Each item should expose:

-   Job title
-   Company
-   Source when available
-   Applied date
-   Current status

Click/tap navigates to Application Detail.

Provide:

`View all applications →`

## 6. No New Application CTA

Do not place a primary `[ New Application ]` action on Home in S2.

New Application remains available from the Applications area as a
fallback/manual path.

## 7. Empty State

If there are no Applications:

``` text
No applications yet

Applications you track with OfferBuddy will appear here.
```

Do not force a large manual New Application CTA into this state.

Extension discovery, when applicable, can provide the primary onboarding
guidance.

## 8. Loading / Partial Failure

Snapshot metrics may use lightweight skeletons.

If snapshot metrics fail but Recent Applications load: - keep Recent
Applications usable; - degrade the snapshot locally.

Do not replace the whole Home page with a global error because an
optional summary request failed.

## 9. Responsive Behaviour

Desktop: - metrics may be presented as three cards/summary items in one
row; - Recent Applications uses the existing desktop list/table pattern.

Narrow/mobile: - metrics may remain a compact three-column summary if
readable, otherwise stack; - Recent Applications reflows into stacked
rows/cards; - no horizontal page scrolling.
