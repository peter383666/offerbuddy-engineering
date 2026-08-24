# Browser Extension UI/UX Specification

## 1. Purpose

The Browser Extension is the primary S2 client capability for SEEK and
Indeed.

It helps the user:

1.  capture reliable job-page facts;
2.  identify key requirements before applying;
3.  remember application intent across navigation;
4.  confirm whether an application was actually submitted;
5.  track the confirmed Application in OfferBuddy.

The Extension must remain lightweight. It is not an ATS automation
product.

## 2. Supported Platforms

S2: - SEEK - Indeed

Do not design generic arbitrary-site behaviour as an S2 requirement.

## 3. Surface Model

The Extension has two primary UI surfaces:

### Injected Entry

A small OfferBuddy entry/status element placed near the supported
job-page apply area where practical.

It may show: - OfferBuddy identity; - a concise screening status; - an
action to review details.

It must not become a mini Application Detail page.

Example:

``` text
OfferBuddy
⚠ 2 things to review
Review →
```

### Extension Popup

The compact primary interaction surface for: - captured job context; -
Things to Review; - application confirmation; - platform-applied
detection; - tracking success/error.

## 4. Active Job Handling

SEEK and Indeed may change the active job without a traditional full
page reload.

When the active job changes:

-   detect the new active job;
-   clear stale presentation from the previous job;
-   capture the new job facts;
-   recompute relevant screening;
-   never display previous-job findings against the new job.

If active job identity is temporarily uncertain, prefer a neutral/hidden
state over stale information.

## 5. Capture

Capture reliable page facts before the user leaves the JD context.

Typical facts include the Technical Design-defined job identity and
persisted Job snapshot.

Capture itself is not a user-facing Application action.

Do not label capture as: - Saved Application - Applied - Tracked

## 6. Things to Review

The Extension may show evidence-based findings such as:

``` text
⚠ Australian citizenship required
⚠ NV1 security clearance required
○ Sponsorship not mentioned
✓ Visa sponsorship mentioned
```

Rules:

-   Findings describe JD evidence.
-   Do not declare the user eligible/ineligible.
-   Do not generate a match score.
-   Do not prevent Apply.
-   Colour must not be the only carrier of meaning.
-   Unknown evidence must not be presented as a definite negative.

## 7. Apply Intent

Observing the user click an Apply / Quick Apply / Apply on company site
action means:

> the user has started or intended to start an application flow.

It does **not** mean:

> the application was submitted.

Do not create an Application solely because an Apply button was clicked.

## 8. Pending Application Context

Before navigation removes the JD context, retain a temporary pending
context sufficient to later resolve the application outcome.

This state is not a persisted OfferBuddy Application status.

User-facing concept:

> Application to confirm

Do not expose internal event/storage terminology.

## 9. Resolving Whether the User Applied

Resolution order:

1.  reliable platform-applied evidence, when available;
2.  explicit user confirmation when platform evidence is unavailable.

### Reliable Platform Evidence

If SEEK or Indeed reliably indicates that the job was applied to, show:

``` text
Application detected

<Job title>
<Company>

[ Track in OfferBuddy ]
```

Do not infer applied state from `button.disabled` alone. Detection is
platform-specific.

### Unknown Outcome

If no reliable platform evidence exists:

``` text
Did you apply?

<Job title>
<Company>

[ Yes, I applied ]
[ Not yet ]
Didn't apply
```

Behaviour:

-   `Yes, I applied` → attempt to track.
-   `Not yet` → preserve pending context.
-   `Didn't apply` → discard/resolve the pending context without
    creating an Application.

No second confirmation modal is required after `Yes, I applied`.

## 10. External ATS / Company Sites

Typical flow:

``` text
Indeed
→ Apply on company site
→ Workday / company ATS
```

S2 does not:

-   inject into arbitrary ATS pages;
-   detect generic ATS submission;
-   autofill ATS forms;
-   monitor form progress;
-   support arbitrary company sites as extension adapters.

The pending context remains the fallback mechanism. Outcome is resolved
through platform evidence where available or user confirmation.

## 11. Tracking

Use `Track in OfferBuddy` when the application is already known to have
been submitted.

While waiting for backend confirmation:

``` text
Tracking...
```

On confirmed success:

``` text
✓ Tracked in OfferBuddy

View Application →
```

`Tracked in OfferBuddy` must never appear before confirmed backend
success.

## 12. Tracking Failure

Use a persistent recovery state:

``` text
Couldn't track application

Your application information is still here.

[ Try again ]
```

Do not discard pending context because of a transient tracking failure.

Do not rely only on a toast for a recoverable tracking failure.

## 13. Authentication

When tracking requires authentication and the user is not authenticated:

-   clearly explain that OfferBuddy sign-in is required;
-   preserve safe pending context where technically supported;
-   do not imply the Application has been tracked;
-   after authentication, return the user to a recoverable
    confirmation/tracking state where practical.

Exact credential mechanism remains governed by Technical Design.

## 14. Compact Layout Rules

-   Dedicated compact Extension layout.
-   No horizontal scrolling.
-   Vertical overflow may scroll.
-   Use the same typography family, semantic language, radius and button
    hierarchy as the Web App.
-   Use tighter spacing than Web.
-   Long evidence text wraps; do not clip essential text.

## 15. Button Hierarchy

For confirmation:

-   Primary: `Yes, I applied`
-   Secondary: `Not yet`
-   Tertiary/text: `Didn't apply`

For reliable platform evidence:

-   Primary: `Track in OfferBuddy`

For recovery:

-   `Try again` uses the appropriate recovery hierarchy without creating
    multiple competing primary actions.

## 16. Explicit Exclusions

Do not add:

-   Saved Jobs
-   application status management inside the Extension
-   analytics
-   full Job Intelligence detail
-   resume matching
-   cover letters
-   ATS autofill
-   auto-apply
-   generic ATS adapters
-   LinkedIn
