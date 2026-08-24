# Phase 4 → Technical Design Deltas

Status: Required reconciliation items discovered during Phase 4 Final
UI/UX Review.

These are not invitations to reopen the entire Phase 3 Technical Design.

## TD-Delta-01 --- Temporary Pending Application Context

### Why It Exists

On SEEK/Indeed, clicking Apply may navigate away from the JD: - to a
separate SEEK/Indeed application flow; - to an external ATS/company
website.

Clicking Apply is not proof of submission, but OfferBuddy must retain
enough context to later ask/resolve whether the Application was actually
submitted.

### Required Design Outcome

Technical Design must explicitly support a temporary pending context
sufficient to:

-   identify the captured Job;
-   remember that an Apply flow was started;
-   survive the relevant browser navigation lifecycle;
-   later resolve via platform evidence or user confirmation;
-   safely expire/discard stale context;
-   preserve recoverability across transient tracking failures where
    appropriate.

### Important Boundary

This pending context is **not**: - a PostgreSQL Application lifecycle
status; - a user-visible Saved Job; - a durable Application created on
Apply click.

Keep it lightweight and scoped to the Extension workflow.

## TD-Delta-02 --- Platform Applied-State Detection

### Why It Exists

SEEK/Indeed may themselves expose reliable evidence that a job has
already been applied to.

When reliable evidence exists, OfferBuddy should avoid unnecessarily
asking `Did you apply?`.

### Required Design Outcome

SEEK and Indeed Site Adapters should support platform-specific
applied-state evidence detection where reliable.

This must not be implemented as a generic rule such as:

``` text
button.disabled == true → applied
```

Disabled buttons can represent unrelated states.

Applied-state detection may use platform-specific: - button
text/state; - explicit Applied labels; - known page/DOM state; - other
reliable platform evidence.

The Site Adapter owns platform-specific interpretation.

### UX Outcome

Reliable evidence:

``` text
Application detected
[ Track in OfferBuddy ]
```

No reliable evidence:

``` text
Did you apply?
[ Yes, I applied ]
[ Not yet ]
Didn't apply
```

## Closed / Non-Required Delta --- Screening Findings Persistence

Phase 4 does not require Things to Review to be persisted and reproduced
on Application Detail.

Therefore no new persistence/API design should be added solely for this
UI.

## Optional Enhancement --- Web Extension Installation Detection

Reliable Extension installation detection is not an S2 architecture
requirement.

If it can be implemented cheaply and reliably, Home may use it to hide
Extension discovery after installation.

If state is unknown: - do not assert that the Extension is not
installed; - do not add complex cross-context communication solely for
the banner.
