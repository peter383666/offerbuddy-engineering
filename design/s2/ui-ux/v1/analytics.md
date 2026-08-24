# Analytics Dashboard S2 Specification

## 1. Purpose

Analytics is a small standalone top-level page that helps the user
understand:

1.  how many Applications they submitted;
2.  what those Applications currently look like by status;
3.  how many reached Interview and Offer;
4.  basic conversion through those stages.

Analytics is a secondary S2 feature, not a BI/reporting product.

## 2. Navigation

Top-level destination:

`Analytics`

Do not place the full Dashboard inside Home.

## 3. Page Structure

``` text
Analytics                              Last 30 days ▼

Applications     Interviews     Offers

Applications Over Time
[ simple trend chart ]

Application Status              Conversion
[ horizontal bars ]             Application → Interview
                                Interview → Offer
```

## 4. Time Range

Single page-level selector.

Options: - Last 7 days - Last 30 days - Last 90 days - All time

Default: - Last 30 days

Do not add: - custom date picker; - per-widget date ranges; - comparison
period selector; - user-selected chart granularity.

## 5. Cohort Semantics

For a selected time range, the Analytics cohort is:

> Applications whose `applied_at` falls within the selected range.

All Dashboard sections analyse that same cohort.

`applied_at` is date-level business data; do not invent hour-level
precision.

## 6. Summary Metrics

### Applications

Count of Applications in the selected cohort.

### Interviews

Count of cohort Applications that **ever reached Interview**, regardless
of current status.

Example:

``` text
Applied → Interview → Rejected
```

This Application still counts as an Interview.

### Offers

Count of cohort Applications that **ever reached Offer**, regardless of
current status.

Do not derive Interviews/Offers only from current status.

## 7. Applications Over Time

Purpose:

> show when Applications in the selected period were submitted.

Basis: - `applied_at`

Suggested automatic display granularity: - Last 7 days → day - Last 30
days → week - Last 90 days → week - All time → month

The exact aggregation implementation must remain consistent with backend
Analytics semantics.

No user-controlled granularity selector.

## 8. Current Application Status

Show current status distribution for the selected cohort.

Fixed order:

1.  Applied
2.  Interview
3.  Offer
4.  Rejected
5.  Withdrawn
6.  No Response

Do not dynamically reorder statuses by count.

The status counts should sum to the cohort Application count under the
established domain model.

`No Response` is the explicit domain status. Do not automatically infer
No Response based on elapsed time.

Preferred visualisation: - horizontal bars; - label + numeric value
always visible.

Do not replace with a donut/pie chart in S2.

## 9. Conversion

### Application → Interview

``` text
Applications in cohort that ever reached Interview
---------------------------------------------------
Applications in cohort
```

Display: - percentage; - numerator/denominator.

Example:

``` text
Application → Interview

14.3%

6 of 42 applications
```

### Interview → Offer

``` text
Applications in cohort that ever reached Offer
------------------------------------------------
Applications in cohort that ever reached Interview
```

Example:

``` text
Interview → Offer

16.7%

1 of 6 interviews
```

### Zero Denominator

If there are no Interviews:

``` text
Interview → Offer

—
No interviews yet
```

Do not display `0%` when the denominator is zero.

If there are Interviews but no Offers, `0%` is valid.

## 10. Interactions

Primary Dashboard interaction: - change time range.

Charts and metric cards are informational in S2.

Do not add: - chart drill-down; - metric-card navigation to filtered
Applications; - cross-page filter state; - sorting; - dashboard
configuration.

Desktop chart tooltips/focus values may provide exact values, but
tooltips must not contain essential information unavailable elsewhere.

## 11. Loading

Use section/card skeletons for initial loading.

When the time range changes: - preserve page structure; - refresh the
data without blanking the whole page where practical.

## 12. Empty States

### User Has No Applications

Use a page-level onboarding empty state:

``` text
Your application insights will appear here

Track applications in OfferBuddy to see
activity, status and conversion over time.

[ View Applications ]
```

Do not render a dashboard full of zero-value charts.

### Selected Range Has No Applications

If historical Applications exist but the selected range is empty:

``` text
No applications in this period.

Try a longer time range.
```

Keep the time-range selector available.

## 13. Error

Page-level example:

``` text
Couldn't load analytics

Your applications are unaffected.

[ Try again ]
```

Where technically possible, degrade locally if one section fails while
other valid Analytics data is available.

Do not imply that Analytics failure means Application data was lost.

## 14. Responsive

Desktop: - three summary metrics in a row; - trend chart full width; -
Status and Conversion may use a two-column row.

Narrow: - keep summary compact where readable; - stack Status and
Conversion.

Mobile: - single-column content; - summary may remain three compact
columns if readable; - charts respond to container width; - no
horizontal page scrolling.

Trend chart: - reduce axis-label density on narrow screens; - do not
depend on hover; - support tap/focus where exact values are exposed.

## 15. Accessibility

-   Every chart must expose textual/numeric meaning.
-   Colour is never the sole status indicator.
-   Status labels and counts remain visible.
-   Conversion exposes both percentage and counts.
-   Interactive chart points, if used, require accessible labels.
-   Keyboard users must be able to operate the time-range control.

## 16. Explicit S2 Exclusions

Do not add: - custom dates; - previous-period comparison; - source
performance; - SEEK vs Indeed comparison; - company analytics; -
location analytics; - salary analytics; - skills analytics; - Job
Intelligence analytics; - sponsorship/citizenship analytics; -
time-to-interview; - time-to-offer; - predictive insights; - benchmarks
against other users; - advanced funnels; - cohort analysis; -
export/report builder; - configurable dashboards.
