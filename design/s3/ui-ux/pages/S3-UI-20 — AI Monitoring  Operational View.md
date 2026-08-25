S3-UI-20 — AI Monitoring / Operational View

# S3-UI-20 — AI Monitoring / Operational View

```yaml
spec_id: S3-UI-20
title: AI Monitoring / Operational View
surface: ruoyi-admin
status: frozen
figma_page: S3-04 Admin / AI Governance
depends_on:
  - ai-provider-abstraction
  - ai-capability-router
  - ai-runtime-configuration
  - observability
  - audit
  - admin-security
s2_reuse: true
integration_type: ruoyi-native
```

## 1. Purpose

`AI Monitoring / Operational View` gives OfferBuddy operators a **small, production-useful view of AI health and usage**.

It answers:

> **Are OfferBuddy's AI capabilities working?**
> **Which capability/provider is being used?**
> **How often are requests succeeding or failing?**
> **How long are they taking?**
> **What token/cost footprint are we seeing?**
> **What was the latest operational failure?**

The intended scope is:

```text
AI executions
     │
     ▼
Operational telemetry
     │
     ├── Requests
     ├── Success
     ├── Failure
     ├── Latency
     ├── Provider / Model
     ├── Token usage
     ├── Estimated cost
     └── Last failure
             │
             ▼
        RuoYi Admin
```

This is deliberately **not** an enterprise observability platform.

------

# 2. Core Principle

The page is:

> **metadata-first operational monitoring**

not:

> **AI content inspection**

Therefore this UI may tell an operator:

```text
Focused Cover Letter
Provider: Provider A
Requests: 48
Success: 46
Failures: 2
Avg latency: 3.4s
```

but must not tell them:

```text
Candidate name
Candidate Profile
Resume contents
Job description
Prompt
Raw model response
Generated Cover Letter
```

That boundary is hard.

------

# 3. Scope

### In Scope

- request counts;
- success counts/rates;
- failure counts/rates;
- latency;
- capability;
- provider;
- model where operationally useful;
- token usage;
- estimated cost;
- last failure metadata;
- time-range filtering;
- capability/provider filtering;
- operational empty/loading/error states;
- RuoYi RBAC;
- safe diagnostics.

### Out of Scope

- raw prompts;
- raw responses;
- Candidate Profile content;
- Resume content;
- Cover Letter content;
- raw Job descriptions;
- prompt editor;
- provider API-key management;
- log explorer;
- distributed tracing UI;
- full APM;
- Grafana replacement;
- billing/reconciliation;
- Candidate usage analytics;
- employee productivity monitoring;
- manual editing of telemetry.

------

# 4. Figma Reference

**Figma Page**

```
S3-04 Admin / AI Governance
```

Relevant final reviewed frames include:

```text
AI Monitoring / Operational View
AI Summary Metrics
Capability / Provider Breakdown
Last Failure
```

Use the final reviewed RuoYi-native versions.

Earlier blurred, overlapping or exploratory components are obsolete.

`ANNOTATION — ...` nodes are documentation only and must not render.

------

# 5. RuoYi Integration

As with the other S3 Admin pages:

> **Reuse RuoYi. Do not redesign Admin.**

Use existing:

- page container;
- breadcrumb;
- filter form;
- cards where already supported;
- table;
- tags;
- pagination where required;
- loading state;
- empty state;
- error state;
- permission directives.

Figma defines content and hierarchy.

It does not define a new Admin framework.

------

# 6. Relationship to S3-UI-19

The two AI Admin pages have deliberately separate responsibilities.

```text
S3-UI-19
AI Runtime Configuration

"What should the system use?"
        │
        ▼
Enabled / Provider / Model / Config


S3-UI-20
AI Monitoring

"How is it behaving?"
        │
        ▼
Requests / Failures / Latency / Tokens / Cost
```

Do not merge configuration and monitoring into one giant control panel.

------

# 7. Data Source

Monitoring data comes from OfferBuddy's frozen AI operational observability instrumentation.

Conceptually:

```text
Business Capability
       ↓
AI Router
       ↓
Provider Adapter
       ↓
External Provider
       ↓
execution metadata
       ↓
AI Operational Observability
       ↓
Admin Monitoring API
       ↓
RuoYi
```

The Admin frontend must not call external AI provider monitoring APIs directly.

------

# 8. Capability Dimension

Metrics should be attributable to semantic OfferBuddy capabilities.

Examples:

```text
JOB_INTELLIGENCE
RESUME_IMPORT
MATCH_ANALYSIS
TAILORED_RESUME
FOCUSED_COVER_LETTER
```

Use exact frozen backend identifiers.

The UI may display friendly labels.

This lets the operator answer:

> Is Match Analysis failing?

rather than only:

> Provider A has some failures.

------

# 9. Provider Dimension

Monitoring should also identify the provider actually used.

Conceptually:

```text
Capability:
MATCH_ANALYSIS

Provider:
Provider A
```

This is particularly useful after runtime routing changes.

Do not assume current configuration tells you which provider handled historical requests.

Monitoring should use **execution-time metadata**.

------

# 10. Model Dimension

Where model identifier is recorded safely:

```text
Provider A
Model X
```

may be displayed.

This is operational metadata.

Do not expose provider request bodies or model configuration internals merely because the model name is available.

------

# 11. Time Range

The page should support the small operational time-range filtering frozen in Figma/API.

Conceptually:

```text
Last 24 hours
Last 7 days
Last 30 days
```

or the exact final options.

Avoid building a full arbitrary analytics query builder.

------

# 12. Filters

Useful filters include:

```text
Time Range
Capability
Provider
```

and model only if frozen/necessary.

Filters should be backend-supported.

Do not load all historical telemetry into the browser and filter it locally.

------

# 13. Summary Metrics

The main operational summary should include the frozen metrics.

Core set:

```text
Requests
Success
Failures
Latency
Tokens
Estimated Cost
```

The purpose is rapid operational diagnosis.

Avoid adding vanity metrics merely because telemetry exists.

------

# 14. Request Count

`Requests` represents AI execution attempts according to the frozen observability definition.

The UI must use backend semantics consistently.

Do not independently derive request count from business entities such as:

```text
number of resumes
+
number of cover letters
```

because retry/execution semantics differ.

------

# 15. Success

Success means the AI execution reached the frozen successful operational outcome.

Example:

```text
Requests: 100
Success:   96
Failures:   4
```

If retry attempts are included/excluded according to backend telemetry semantics, the UI must use that definition.

Do not invent another frontend interpretation.

------

# 16. Success Rate

If displayed:

```text
Success Rate
= successful executions / relevant executions
```

The backend should preferably provide the metric or authoritative numerator/denominator.

Handle zero requests safely.

Do not show:

```text
NaN%
Infinity%
```

------

# 17. Failure

Failure count represents operationally failed AI executions.

Failures should be categorised where the frozen telemetry supports safe categories.

Examples:

```text
TIMEOUT
RATE_LIMIT
PROVIDER_ERROR
INVALID_RESPONSE
VALIDATION_ERROR
INTERNAL_ERROR
```

Use actual backend categories.

Do not expose raw provider error payloads.

------

# 18. Latency

The page should display useful aggregate latency.

For example, according to frozen metrics:

```text
Average latency
```

and possibly another approved percentile if already designed.

Do not create a full latency distribution analytics system for S3.

Units must be obvious:

```text
3.2 s
850 ms
```

and consistent.

------

# 19. Token Usage

Where the provider returns usage metadata, OfferBuddy may record/display:

```text
Input tokens
Output tokens
Total tokens
```

or the smaller frozen representation.

Tokens are operational metadata.

They must not be used to reconstruct prompt content.

------

# 20. Missing Token Data

Not every provider/model necessarily returns identical token metadata.

Therefore the UI must support:

```text
Token usage: unavailable
```

rather than inventing `0`.

Important distinction:

```text
0
≠
unknown
```

------

# 21. Estimated Cost

Where OfferBuddy has sufficient pricing/usage information, the page may display:

> **Estimated cost**

The word **Estimated** matters.

The UI must not present this as:

> exact provider invoice.

Conceptually:

```text
usage metadata
    +
configured/reference pricing
    ↓
estimated operational cost
```

------

# 22. Cost Currency

Use the frozen backend/configured currency semantics.

Do not let the frontend silently assume:

```text
$ = AUD
```

or:

```text
$ = USD
```

without explicit backend meaning.

Display currency clearly where cost is shown.

------

# 23. Missing Cost Data

If usage/pricing information is incomplete:

```text
Estimated cost: unavailable
```

is preferable to:

```text
$0.00
```

because zero implies known zero cost.

------

# 24. Cost Boundary

S3 cost monitoring is informational.

It is not:

- provider invoice reconciliation;
- accounting;
- budgeting;
- chargeback;
- tax reporting;
- financial ledger.

Do not overbuild this capability.

------

# 25. Capability Breakdown

The page may show operational metrics by capability.

Conceptually:

| Capability       | Requests | Success | Failures | Avg Latency | Tokens | Est. Cost |
| ---------------- | -------- | ------- | -------- | ----------- | ------ | --------- |
| Job Intelligence | 120      | 116     | 4        | 2.1s        | …      | …         |
| Resume Import    | 42       | 41      | 1        | 4.2s        | …      | …         |
| Match Analysis   | 78       | 75      | 3        | 3.5s        | …      | …         |

Exact columns follow frozen Figma/API.

This is a summary table, not a user-content table.

------

# 26. Provider Breakdown

Where useful, the page may also show metrics by actual provider.

Example:

```text
Provider A
Requests: 180
Failures: 4

Provider B
Requests: 60
Failures: 3
```

This helps detect provider-specific operational issues.

Do not interpret correlation as automatic routing policy.

Admin decides routing separately in `S3-UI-19`.

------

# 27. Historical Provider Accuracy

Suppose:

```text
09:00–12:00
Match → Provider A

12:00
Admin changes config

12:00–17:00
Match → Provider B
```

Monitoring must preserve actual execution provider.

It must not rewrite the morning metrics as Provider B just because Provider B is now configured.

------

# 28. Last Failure

The page includes a concise operational `Last Failure` area.

Purpose:

> Give the operator enough information to know what failed and where to investigate.

Safe example:

```text
Last failure

Capability: Focused Cover Letter
Provider: Provider A
Time: 14:32
Category: TIMEOUT
Request ID: ...
```

Potentially include a safe sanitised error summary if frozen.

------

# 29. Last Failure Must Not Contain Content

Never show:

```text
Prompt:
"Peter is a software engineer..."

Response:
"..."

Candidate:
...

Resume:
...
```

The Last Failure panel is metadata-first.

------

# 30. Provider Error Sanitisation

External providers may return error messages containing unexpected content.

Therefore raw provider error strings must not automatically flow into Admin UI.

Correct:

```text
Category: RATE_LIMIT
Safe summary: Provider rate limit reached
```

Incorrect:

```text
rawProviderResponse.error.message
```

rendered without sanitisation.

Backend owns error normalisation.

------

# 31. Request ID

Where useful, show:

```text
requestId
```

to connect the Admin view with backend diagnostics.

The frozen observability model distinguishes:

```text
requestId
```

for one HTTP request and:

```text
correlationId
```

for logical cross-async tracing.

`correlationId` remains internal by default.

Do not casually expose it merely because it exists.

------

# 32. Correlation

Operational diagnostics may internally correlate:

```text
HTTP
 ↓
business event
 ↓
worker
 ↓
AI request
```

using the frozen correlation model.

The Admin UI does not need to become a distributed trace viewer.

If deeper investigation is needed, operators use normal backend observability tooling.

------

# 33. Business Event Relationship

AI operations may execute asynchronously through the existing PostgreSQL `business_events` infrastructure.

Monitoring can reflect the resulting AI execution metadata.

But this page is **not** a `business_events` queue-management screen.

Do not add:

```text
retry event
edit event
delete event
change processing state
```

controls here.

------

# 34. Retry Semantics

Automatic retries may result in multiple attempts according to frozen observability semantics.

The UI must use backend-provided aggregate definitions.

Do not assume:

```text
1 business command = exactly 1 provider request
```

because retries can exist.

This is another reason frontend must not calculate operational metrics from business-resource counts.

------

# 35. Failure vs Business Failure

Distinguish:

```text
AI operational execution failure
```

from:

```text
business validation / user input issue
```

where the backend telemetry model distinguishes them.

Do not label every unsuccessful business operation:

> Provider failure.

Error categories should preserve the actual boundary.

------

# 36. No Candidate-Level Monitoring

Do not build a table such as:

| Candidate | AI Requests | Resume | Cost |
| --------- | ----------- | ------ | ---- |
|           |             |        |      |

That is outside S3 monitoring purpose and unnecessarily increases privacy exposure.

Monitoring should aggregate around:

```text
capability
provider
model
time
operational outcome
```

not Candidate identity.

------

# 37. No Application-Level Monitoring

Likewise, this page should not become:

```text
Application #123
→ Match prompt
→ Resume generation
→ Cover Letter
```

Application/business diagnostics remain separate from aggregate AI operational monitoring.

------

# 38. No Raw Execution Browser

Do not build a general table containing every AI request and its full payload.

If the final Figma contains recent failures/events, rows must remain metadata-only.

Allowed examples:

```text
time
capability
provider
model
outcome
latency
safe error category
requestId
```

Not:

```text
prompt
response
Candidate JSON
Job HTML
```

------

# 39. Monitoring Retention

The UI should use the retention available from frozen observability storage.

Do not imply indefinite historical analytics if backend retention is shorter.

S3 does not require building a data warehouse for AI metrics.

------

# 40. Data Freshness

Monitoring is operational.

It does not need hard real-time streaming.

A normal refresh/read model is sufficient according to frozen backend design.

Do not add WebSocket/live-stream infrastructure solely for this page unless already present.

------

# 41. Manual Refresh

If final RuoYi design provides refresh:

```text
Refresh
```

it should simply reload current monitoring data.

Do not make refresh trigger:

- AI provider health test;
- business retry;
- configuration reload;
- telemetry recalculation beyond backend read semantics.

------

# 42. Auto Refresh

Do not introduce aggressive polling.

If periodic refresh is implemented, use a conservative interval appropriate for an operational Admin page.

There is no need for:

```text
poll every second
```

for S3.

------

# 43. Configuration Link

If final Figma provides navigation between:

```text
AI Monitoring
```

and:

```text
AI Runtime Configuration
```

that is appropriate.

But monitoring must not contain inline provider-routing mutations simply for convenience.

The operator should consciously move to configuration when changing behaviour.

------

# 44. Provider Failure Does Not Auto-Reconfigure

Example:

```text
Provider A
Failure rate increases
```

Monitoring reports it.

It must **not** automatically execute:

```text
switch all capabilities to Provider B
```

unless a separately frozen fallback policy exists.

Monitoring observes.

Configuration governs.

------

# 45. Capability Disablement

If a capability is disabled in `S3-UI-19`, monitoring may naturally show fewer/no new requests.

The monitoring page should not interpret this as provider failure.

Where useful, current configuration state can be linked or labelled, but configuration remains authoritative elsewhere.

------

# 46. Loading State

While metrics are loading:

- use normal RuoYi loading/skeleton behaviour;
- do not show fake zeroes.

A zero is a business value.

Loading is a UI state.

Keep them distinct.

------

# 47. Empty State

If no AI requests exist in the selected time range:

Correct:

> No AI activity for this period.

Not:

```text
0 failures = Everything healthy
```

because no executions occurred.

------

# 48. Partial Data

Some metrics may be unavailable while others exist.

Example:

```text
Requests: 100
Success: 98
Latency: 2.4s
Tokens: unavailable
Cost: unavailable
```

The whole monitoring page should not fail simply because one provider lacks token/cost metadata.

------

# 49. Monitoring API Failure

If the monitoring read fails:

- show standard RuoYi error/retry state;
- do not alter AI runtime configuration;
- do not infer that providers are down.

Failure to load monitoring data is not equivalent to AI service failure.

------

# 50. UI States

Required:

### Page

- Loading
- Ready
- Empty
- Partial Data
- Load Failure

### Filters

- Ready
- Applying
- No Results

### Metrics

- Value available
- Value unavailable

### Last Failure

- Failure available
- No failures in period
- Metadata unavailable

------

# 51. Permissions

Use RuoYi RBAC.

Conceptually:

```text
AI monitoring:view
```

or the actual frozen permission identifier.

Viewing operational monitoring does not automatically grant:

```text
AI configuration:edit
```

These permissions may be separate.

Frontend directives improve UX.

Backend authorization remains mandatory.

------

# 52. Admin Is Not Universal Data Access

An Admin authorised to view AI monitoring does not thereby gain access to:

- Candidate Profile;
- Resume;
- Cover Letter;
- Job raw content;
- Application private content.

This remains one of the frozen S3 security principles.

------

# 53. Privacy

AI monitoring must be designed to remain useful without exposing Candidate content.

Allowed:

```text
Capability
Provider
Model
Latency
Tokens
Cost
Outcome
Safe error category
Request ID
Timestamp
```

Disallowed:

```text
Candidate facts
Resume
Cover Letter
Raw Job HTML
Raw prompt
Raw response
Secrets
```

------

# 54. Logs vs Monitoring UI

The monitoring UI should consume structured operational data.

It should not scrape application log text to build metrics.

Likewise, do not expose a raw log viewer simply to satisfy monitoring requirements.

Structured telemetry is the contract.

------

# 55. Observability Alignment

This UI follows frozen §3.16:

```text
Application observability
Business/workflow observability
AI operational observability
Privileged audit
```

`S3-UI-20` primarily surfaces:

> **AI operational observability**

It should not merge all four categories into one screen.

------

# 56. Audit Boundary

Viewing monitoring data normally does not require business mutation audit.

Configuration changes are audited in `S3-UI-19`.

If monitoring access itself requires privileged-access audit under frozen security policy, reuse that infrastructure.

Do not invent a second audit system.

------

# 57. Security

Required:

- authenticated RuoYi Admin;
- backend RBAC;
- metadata-only responses;
- sanitised provider errors;
- no secret exposure;
- no raw AI content;
- no Candidate payloads;
- no client-controlled actor identity.

Monitoring endpoints are Admin operational APIs, not public Web APIs.

------

# 58. Performance

Metrics should be aggregated server-side.

Incorrect:

```text
Browser downloads 100,000 AI execution rows
        ↓
JavaScript calculates metrics
```

Correct:

```text
Admin query
        ↓
backend aggregate
        ↓
small monitoring response
```

Use pagination only for any bounded metadata event/failure table if one exists.

------

# 59. API Mapping

Use frozen §3.17 Admin / AI Governance contracts.

Conceptually:

```text
GET AI monitoring summary
GET AI monitoring breakdown
GET safe latest failure metadata
```

or their frozen equivalents.

Do not invent APIs such as:

```text
GET /admin/ai/prompts
GET /admin/ai/responses
GET /admin/ai/candidate-requests
GET /admin/ai/secrets
```

------

# 60. S2 Reuse

### Reuse

- RuoYi shell;
- RuoYi cards/table/filter patterns;
- RBAC;
- structured logging conventions;
- existing AI provider abstraction;
- existing S2/S3 observability infrastructure.

### S3 Add

- AI execution metrics;
- provider/capability breakdown;
- token/cost visibility;
- safe last-failure diagnostics.

### Do Not

- introduce Prometheus/Grafana replacement;
- introduce distributed tracing product;
- introduce AI content viewer;
- introduce provider billing system.

------

# 61. Component Reuse

Possible frontend responsibilities:

```text
AiMonitoringPage
AiMetricsSummary
AiCapabilityMetricsTable
AiProviderMetricsTable
AiLastFailurePanel
AiMonitoringFilters
```

Names are illustrative.

Prefer existing RuoYi components.

Backend may conceptually provide:

```text
AiMonitoringQueryService
AiMetricsRepository
AiFailureSummaryProjection
```

Exact implementation follows frozen Phase 3 architecture.

------

# 62. Accessibility / UX

- Metric labels must be explicit.
- Failure/success must not rely only on colour.
- Latency units must be visible.
- Cost currency must be visible.
- `Unavailable` must be distinguishable from `0`.
- Filters need proper labels.
- Tables should use normal keyboard/accessibility behaviour.
- Last Failure should be readable without exposing raw technical dumps.
- Do not overwhelm operators with unnecessary telemetry.

------

# 63. Acceptance Criteria

### Monitoring

-  AI Monitoring uses native RuoYi components.
-  RuoYi shell is not redesigned.
-  User can view AI request volume.
-  User can view success/failure information.
-  User can view latency.
-  User can view capability breakdown.
-  User can view provider breakdown where frozen.
-  Actual execution provider is preserved historically.
-  Model can be shown where available.
-  Token usage can be shown where available.
-  Missing token usage is shown as unavailable, not zero.
-  Estimated cost can be shown where available.
-  Estimated cost is clearly labelled estimated.
-  Currency is explicit.
-  Missing cost is shown as unavailable, not zero.
-  Time-range filtering works through backend-supported queries.
-  Capability/provider filters are supported according to final design.
-  Metrics are aggregated server-side.
-  Retry semantics use backend definitions.
-  Empty period is distinguished from healthy activity.
-  Partial metric availability does not fail the whole page.

### Failure Diagnostics

-  Last Failure shows safe operational metadata.
-  Failure includes capability where available.
-  Failure includes provider where available.
-  Failure includes timestamp.
-  Failure includes safe error category.
-  Request ID can be shown where useful.
-  Raw provider error payload is not exposed.
-  Raw prompt is never displayed.
-  Raw response is never displayed.
-  Candidate data is never displayed.
-  Resume/Cover Letter content is never displayed.
-  Raw Job content is never displayed.

### Boundaries

-  Monitoring does not edit runtime configuration.
-  Monitoring does not automatically switch providers.
-  Monitoring does not retry business events.
-  Monitoring does not become a queue-management UI.
-  Monitoring does not become a log explorer.
-  Monitoring does not become a tracing product.
-  Monitoring does not become provider billing reconciliation.
-  Monitoring view permission does not imply config-edit permission.
-  Admin monitoring access does not bypass Candidate ownership.
-  Backend RBAC is enforced.
-  S2 behaviour remains unaffected.

------

# 64. Agent Implementation Notes

1. Inspect frozen §3.16 Observability design before implementation.
2. Inspect final AI Monitoring Figma frame.
3. Reuse native RuoYi components.
4. Do not recreate the Admin shell.
5. Read frozen monitoring API contracts before creating frontend DTOs.
6. Use semantic capability identifiers.
7. Use actual execution provider/model metadata.
8. Do not derive historical provider from current runtime config.
9. Aggregate metrics server-side.
10. Do not calculate operational metrics from Resume/CL/business-resource counts.
11. Preserve backend retry/attempt semantics.
12. Keep `0` and `unavailable` distinct.
13. Label cost as Estimated.
14. Always show cost currency explicitly.
15. Do not build accounting/billing features.
16. Normalise/sanitise provider failures on backend.
17. Never pass raw provider errors directly to RuoYi.
18. Never expose prompts.
19. Never expose raw responses.
20. Never expose Candidate Profile content.
21. Never expose Resume/Cover Letter content.
22. Never expose raw Job HTML/description.
23. Never expose secrets.
24. Keep requestId available for operational diagnosis where frozen.
25. Keep correlationId internal by default.
26. Do not build distributed tracing UI.
27. Do not build business-event management here.
28. Do not auto-switch providers based on metrics.
29. Keep monitoring and runtime configuration separate.
30. Handle partial metrics gracefully.
31. Do not poll aggressively.
32. Enforce monitoring RBAC server-side.
33. Ignore Figma annotations.
34. Do not redesign frozen UI.

```
`S3-UI-20` 可以冻结。

这样 **S3 Admin / AI Governance UI 也闭环了**：

​```text
                 RuoYi Admin
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
 S3-UI-19 Runtime Config    S3-UI-20 Monitoring
          │                       │
          │                       │
   What should run?         How is it running?
          │                       │
    Enabled / Provider       Requests
    Model / Parameters       Success / Failure
    Secret reference         Latency
          │                  Tokens / Est. Cost
          │                  Last Failure
          ▼                       ▲
       AI Router ───────→ AI Execution
```

其中必须保持：

```text
Runtime Configuration ≠ Monitoring
Monitoring ≠ Logs
Monitoring ≠ Prompt Viewer
Monitoring ≠ Candidate Data Viewer
Monitoring ≠ Grafana replacement
```

## S3 Page Specification 当前进度

到这里，我们已经完成到了 **`S3-UI-20`**。Candidate Profile、Resume Import、Tailored Resume、Focused Cover Letter、Extension、SEEK Cover Letter、Application Detail Integration、Sponsor Employers Admin、AI Governance 和 AI Monitoring 这些主要 S3 surface 都已经转成了 Agent 可执行 specification。

**下一步我建议先不要继续机械地写 `S3-UI-21`。**

应该进入我们之前约定的：

> **S3 Page Specification Final Coverage Review**

这一步会拿 **最终冻结的 Figma Pages / S3 UI inventory / S3-UI-01～20** 三边对账，重点检查四件事：

```text
Frozen Figma UI
      │
      ├── 是否每个实现页面都有 specification？
      ├── 是否有 specification 实际没有对应 UI？
      ├── 是否遗漏 loading / empty / stale / error / conflict state？
      └── 是否存在 Agent 看完仍然会产生歧义的跨页面关系？
```

Review 完以后，我们再生成最终的 **S3 Page Specification Index + implementation order**。这样 Phase 5 真正交给 Agent 时，就不是拿着 20 份散落文档自己猜先后关系了。