# New Application S2 Delta Specification

## 1. Purpose

New Application remains the Web fallback/manual Application creation
path.

S2 does not remove the existing S1 URL parsing/manual capability.

## 2. Product Hierarchy

Preferred path for supported job sites:

``` text
SEEK / Indeed
→ Browser Extension
```

Fallback:

``` text
Other URL / unsupported flow
→ existing URL parsing where supported
→ manual input
```

## 3. Preserve Existing S1 Flow

Do not redesign the New Application form unless required for
compatibility with S2 data.

Existing URL parsing remains a secondary/fallback capability.

## 4. Optional Extension Guidance

A lightweight message may explain that SEEK and Indeed users can use the
Browser Extension while browsing.

This guidance must be secondary. A user already on New Application may
intentionally be: - entering an old Application; - using an unsupported
site; - recovering from an Extension issue; - choosing manual entry.

Do not obstruct the manual flow with repeated Extension promotion.

## 5. No Duplicate Screening Flow

Do not add a second full pre-application screening experience to New
Application solely for feature symmetry.

The Extension's screening value is specifically strongest before/during
the supported job-site application journey.

## 6. Unified Result

Applications created through New Application and Applications tracked
through the Extension converge on the same: - Application domain; -
Application List; - Application Detail; - async Job Intelligence
behaviour.

Do not create separate Web experiences based on ingestion method.

## 7. Missing Data

Manual Applications may have less complete Job data.

The UI must tolerate missing optional fields and omit them cleanly
rather than displaying excessive placeholder values.
