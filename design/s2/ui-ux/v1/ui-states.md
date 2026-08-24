# Cross-Product UI States and Feedback

## 1. Core Rules

1.  Success appears only after confirmed success.
2.  Unknown remains unknown.
3.  Optional capability failure must not look like core Application
    failure.
4.  Tell the user what they need to understand or act on; do not expose
    infrastructure state.

## 2. Loading

### Blocking Loading

Use only when a user-triggered action must complete before the next
state.

Example:

``` text
Tracking...
```

Disable duplicate submission while the request is in flight.

### Background Loading

Use section-level loading for: - Job Intelligence; - Analytics; -
Extension screening where appropriate.

Do not cover an entire Application page with a global loading overlay
because an optional section is processing.

## 3. Success

Use concise business-language feedback:

-   `Tracked in OfferBuddy`
-   `Status updated`
-   `Application added`

Do not expose persistence/event implementation language.

## 4. Toast Rules

Use toast for transient completed feedback that does not require a
decision or recovery.

Good: - Status updated - Notes saved - Analysis retry started

Do not use toast as the only presentation for: - `Did you apply?` -
recoverable tracking failure - decisions requiring user action

Those require persistent inline/card/popup state.

## 5. Warnings and Findings

Warnings require clear evidence.

Example:

`⚠ Australian citizenship required`

Do not transform evidence into candidate judgment such as:

`You are not eligible.`

Use neutral/informational treatment for absence/unknown-style findings:

`○ Sponsorship not mentioned`

Use positive evidence only when supported:

`✓ Visa sponsorship mentioned`

## 6. Error Pattern

Preferred structure:

``` text
What happened

Context/safety message

[ Recovery action ]
```

Example:

``` text
Couldn't track application

Your application information is still here.

[ Try again ]
```

Optional capability example:

``` text
Couldn't load analytics

Your applications are unaffected.

[ Try again ]
```

## 7. Empty vs Processing vs Unavailable

Do not collapse these into `No data`.

Examples:

-   Empty: `No applications yet`
-   Filter empty: `No applications match your filters`
-   Processing: `Analysing this job...`
-   Insufficient input:
    `Not enough job information is available to analyse this role`
-   Error: `Analysis isn't available right now`

## 8. Job Intelligence Presentation States

User-facing states:

-   Processing
-   Available
-   Unavailable/retryable
-   Insufficient information

Do not expose internal event/worker states such as: - queued; -
claimed; - retry number; - dead; - event created.

## 9. Vocabulary

Use consistently:

  -----------------------------------------------------------------------
  Concept                             UI wording
  ----------------------------------- -----------------------------------
  User-owned submitted job            Application
  application                         

  Persist confirmed Application in    Track / Tracked in OfferBuddy
  OfferBuddy                          

  Temporary Extension unresolved      Application to confirm
  context                             

  User has not completed application  Not yet
  yet                                 

  User abandoned application          Didn't apply

  Platform has reliable submitted     Application detected
  evidence                            

  AI semantic enrichment              Job Intelligence

  Source URL                          Original job posting

  Pre-application evidence            Things to review

  Application lifecycle state         Status
  -----------------------------------------------------------------------

Avoid using `Save to OfferBuddy` for the Extension Application workflow
because it is ambiguous.

## 10. Modal Rules

Avoid modals for: - screening findings; - Job Intelligence; -
Analytics; - normal Extension confirmation; - simple recoverable errors.

Use confirmation modals only where existing destructive/irreversible S1
behaviour genuinely requires them.

`Yes, I applied` does not require a second confirmation modal.

## 11. Navigation Language

External: - `View original job posting ↗`

Internal: - `View Application →` - `View all applications →`

## 12. Local Degradation

Examples: - Home summary fails → Recent Applications still render. - Job
Intelligence fails → Application tracking still works. - Analytics fails
→ Applications remain unaffected.

Never turn a secondary-capability failure into a core-product failure.
