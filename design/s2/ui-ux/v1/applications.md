# Applications List S2 Delta Specification

## 1. Purpose

Applications remains the primary user-owned management surface.

S2 must not create separate list experiences for: - Extension-created
Applications; - manually created Applications; - URL-parsed
Applications.

Once tracked, they are all Applications.

## 2. Preserve S1 Behaviour

Reuse the existing S1: - search; - filters; - pagination; - status
presentation; - navigation to Application Detail; - existing
list/table/card interaction.

Do not redesign the page without a specific S2 requirement.

## 3. Source Context

S2 may add lightweight source context where available:

-   SEEK
-   Indeed
-   other existing/manual source semantics

Preferred presentation is secondary metadata, for example:

``` text
Sydney NSW · SEEK
```

or an existing neutral metadata/badge pattern.

Do not prominently expose ingestion method such as: - Browser
Extension - AI URL parser - Manual form

Source is business context. Ingestion method is implementation context.

## 4. Do Not Add Job Intelligence to the List

Do not add: - AI summary; - responsibilities; - requirements; -
skills; - AI processing status; - screening findings.

The list is for finding and managing Applications, not rereading the JD.

## 5. Responsive

Desktop: - preserve existing S1 list/table where practical.

Narrow/mobile: - reflow to a stacked row/card if necessary; - preserve
title, company, applied date and status; - source/location can combine
into secondary metadata; - avoid horizontal table scrolling as the
normal mobile experience.
