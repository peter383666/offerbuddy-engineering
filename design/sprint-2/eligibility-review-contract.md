# Sprint 2 Eligibility Review Contract

## Purpose

Close the narrow product/UI handoff between Sprint 2 requirements (citizenship, permanent residency, clearance, working rights, sponsorship) and the final Extension UI v2.0 (Review Detail limited to citizenship / permanent residency).

This document records the approved S2 contract. It does not reopen Phase 4, add match scoring, or invent an eligibility dashboard.

## Authority

| Source | Role |
| --- | --- |
| [UI/UX Page Specification v2.0 — Final](ui-ux/OfferBuddy%20S2%20—%20UI%20UX%20Page%20Specification%20v2.0%20—%20Final.md) §§3–5 | Binding Extension Review Required / Review Detail behaviour |
| [Sprint 2 Requirements](../../product/sprint-2-requirements.md) eligibility screening | Categories that may appear in job text; screening is advisory, not an absolute apply decision |
| This contract | How non–citizenship/PR categories are treated in S2 Extension UI |

## S2 surface: what Review shows

**In scope for Review Required and Review Detail**

- Australian citizenship / permanent residency requirement language with page evidence
- Factual copy only (for example “appears to require…”)
- Detected JD wording as evidence
- User-controlled inspect path: warning → Review → Review Detail → Save still available

**Out of scope for Review Required and Review Detail in S2**

- Security clearance
- Working rights (including ambiguous phrases such as “full working rights in Australia”)
- Visa sponsorship availability or restrictions
- Any default row of passed checks (Working rights: OK, Citizenship: OK, …)

Those categories remain **product-relevant text that may exist on the job page**, but S2 does **not** surface them as Extension Review attention. They must not be inferred into citizenship/PR, and must not open a separate Review Detail.

## Outcome mapping (implementation-facing)

| Outcome | Meaning in S2 | Extension behaviour |
| --- | --- | --- |
| No restriction | No citizenship/PR evidence on the current job | Ready / companion save path; no Review Required |
| Review required | Citizenship/PR evidence found; user has not opened detail | Lightweight “Potential eligibility requirement detected” + Review |
| Explicit restriction | Citizenship/PR is the only explicit restriction category shown in S2 | Review Detail: Citizenship / PR required + evidence |
| Unable to determine | Clearance, working rights, sponsorship, missing text, or ambiguous wording that is not citizenship/PR | Do **not** invent a finding; stay on the no-Review path (silent omit). Do not show an “unable to determine” dashboard cell |

Ambiguous working rights **must not** be treated as permanent residency.

## Non-goals

- Absolute automated “you can / cannot apply” decisions
- Eligibility analytics dashboard on Home, popup default view, or Application Detail
- Expanding Review Detail to clearance / working rights / sponsorship in S2
- AI-owned eligibility decisions (see [Job Intelligence Design](job-intelligence-design.md))

## Implementation anchors

- Extension finding kind is citizenship/PR only (`citizenship-pr`)
- SEEK / Indeed adapters extract only that finding; generic working-rights language must not produce it
- Popup / Floating Companion: Review Required and Review Detail only when that finding exists
- Issue: [#44](https://github.com/peter383666/offerbuddy/issues/44)

## Change control

Post–Sprint 2 expansion of Review Detail categories needs a new product/UI decision. Until then, implementers must not invent additional Review surfaces for the omitted categories.
