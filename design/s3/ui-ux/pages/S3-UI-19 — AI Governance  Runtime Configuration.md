# S3-UI-19 — AI Governance / Runtime Configuration

```yaml
spec_id: S3-UI-19
title: AI Governance / Runtime Configuration
surface: ruoyi-admin
status: frozen
figma_page: S3-04 Admin / AI Governance
depends_on:
  - ai-provider-abstraction
  - ai-capability-router
  - runtime-configuration
  - secrets-management
  - admin-security
  - audit
  - observability
s2_reuse: true
integration_type: ruoyi-native
```

## 1. Purpose

`AI Governance / Runtime Configuration` provides the operational RuoYi Admin surface for controlling **how OfferBuddy's AI capabilities are enabled and routed at runtime**.

It answers four operational questions:

> **Which AI capabilities are enabled?**
> **Which provider/model should each capability use?**
> **What safe runtime parameters apply?**
> **Which externally managed secret reference does that provider use?**

Conceptually:

```text
RuoYi Admin
     │
     ▼
AI Runtime Configuration
     │
     ├── Capability enable/disable
     ├── Provider selection
     ├── Model selection/configuration
     ├── Runtime parameters
     └── Secret reference
             │
             ▼
       Validated Config
             │
             ▼
        AI Router
             │
             ▼
     Semantic AI Capability
             │
             ▼
       External Provider
```

This is **operational governance**, not a prompt engineering console.

------

# 2. Design Principle

OfferBuddy already uses RuoYi.

Therefore:

> **Do not redesign the Admin framework.**

Reuse native RuoYi:

- layout;
- sidebar;
- breadcrumb;
- query/list patterns;
- forms;
- dialogs/drawers;
- switches;
- select controls;
- permission directives;
- confirmation dialogs;
- loading/error patterns.

The Figma design defines the **content model and interaction**, not a replacement Admin design system.

------

# 3. Scope

### In Scope

- AI capability configuration list;
- capability enabled/disabled state;
- provider assignment;
- model configuration;
- approved runtime parameters;
- provider status/configuration state;
- secret reference/status;
- validation;
- edit/save;
- runtime configuration refresh semantics;
- auditability;
- RBAC;
- safe configuration display.

### Out of Scope

- raw secret values;
- API-key reveal;
- raw system prompts;
- raw user prompts;
- raw AI responses;
- Candidate data;
- Resume/Cover Letter content;
- arbitrary provider SDK configuration;
- arbitrary JSON config editor;
- prompt playground;
- AI chatbot;
- provider billing portal;
- automatic provider account creation;
- enterprise feature-flag platform;
- arbitrary workflow scripting.

------

# 4. Figma Reference

**Figma Page**

```
S3-04 Admin / AI Governance
```

Relevant final frames include the reviewed RuoYi-native AI Governance states.

The Agent must use the **latest reviewed frames**.

Earlier exploratory components, blurred objects or overlapping annotation/UI nodes are obsolete.

All:

```text
ANNOTATION — ...
```

nodes are implementation documentation only.

------

# 5. Core Domain Boundary

Business modules must continue depending on **semantic AI capabilities**, not provider SDKs.

Correct:

```text
Match Module
     │
     ▼
JobMatchAnalysisPort
     │
     ▼
AI Capability Router
     │
     ▼
Configured Provider
```

Incorrect:

```text
MatchService
    ↓
OpenAI SDK
```

or:

```text
ResumeService
    ↓
Gemini SDK
```

Admin configuration changes routing.

It does not change the business module architecture.

------

# 6. Capability-Oriented Configuration

Configuration should be organised around OfferBuddy capabilities.

Conceptually:

```text
JOB_INTELLIGENCE
RESUME_IMPORT
MATCH_ANALYSIS
TAILORED_RESUME
FOCUSED_COVER_LETTER
```

Use exact frozen capability identifiers from the backend.

Do not use UI labels as persistent identifiers.

For example:

```text
UI:
Focused Cover Letter

Domain key:
FOCUSED_COVER_LETTER
```

------

# 7. Why Capability-Level Routing

Different AI workloads have different characteristics.

Example:

```text
Resume Import
→ structured extraction

Match Analysis
→ reasoning + structured result

Tailored Resume
→ controlled generation

Focused Cover Letter
→ controlled generation
```

Therefore S3 allows routing by capability instead of enforcing:

```text
one provider + one model
for every AI feature
```

But this remains intentionally small.

It is not a general AI workflow engine.

------

# 8. Main Page Structure

Conceptually, using native RuoYi:

```text
AI Governance
│
├── Runtime Configuration
│
│   ├── Capability
│
│   ├── Enabled
│
│   ├── Provider
│
│   ├── Model
│
│   ├── Config Status
│
│   └── Actions
│
└── Provider Configuration
    ├── Provider
    ├── Availability
    ├── Secret Reference Status
    └── Actions
```

Exact grouping follows final Figma.

Do not over-expand the page into a dashboard if the final design uses simple operational tables/forms.

------

# 9. Capability Configuration List

Each row represents an AI capability configuration.

Conceptually:

| Capability | Enabled | Provider | Model | Status | Updated | Actions |
| ---------- | ------- | -------- | ----- | ------ | ------- | ------- |
|            |         |          |       |        |         |         |

The table should communicate:

> what will OfferBuddy attempt to use when this capability runs?

It should not expose internal prompt contents.

------

# 10. Enabled / Disabled

Admin may enable or disable an AI capability where the frozen runtime configuration supports it.

Example:

```text
TAILORED_RESUME
Enabled = false
```

means:

> new Tailored Resume AI execution is unavailable according to capability semantics.

It does **not** mean:

- delete existing Tailored Resumes;
- delete Candidate Profile;
- delete Preparation;
- delete previous AI telemetry.

Disabling execution and deleting business data are entirely different operations.

------

# 11. Disabled Capability UX

Business UI should receive/use the frozen capability-availability semantics.

It should not discover disablement only after an expensive provider request.

Conceptually:

```text
User requests Tailored Resume
        ↓
Capability configuration
        ↓
disabled
        ↓
controlled unavailable response
```

The exact Web UX belongs to the corresponding capability specification.

Admin merely controls the operational state.

------

# 12. Provider Selection

For an enabled capability, Admin may choose from configured/supported AI providers.

Conceptually:

```text
Match Analysis

Provider:
[ Provider A ▼ ]
```

The options must come from the backend-supported provider registry/configuration.

Do not allow arbitrary free-text provider class names.

------

# 13. Provider Abstraction

Provider selection must resolve through the AI infrastructure layer.

Correct:

```text
Capability
    ↓
Router
    ↓
Provider adapter
```

The Admin must not make business modules aware of:

```text
provider-specific SDK request objects
provider-specific credentials
provider-specific response structures
```

Provider adapters translate between OfferBuddy semantic contracts and external APIs.

------

# 14. Model Selection

Where runtime model selection is supported:

```text
Provider:
Provider A

Model:
model-x
```

The allowed model options should follow the frozen provider/config registry.

Avoid unrestricted free-text model names unless the backend contract explicitly allows them.

This prevents accidental unsupported runtime configuration.

------

# 15. Runtime Parameters

Only approved runtime parameters should be configurable.

Possible examples **only if present in the frozen contract**:

```text
temperature
max output tokens
timeout
```

Do not automatically expose every provider SDK parameter.

The Admin configuration should represent OfferBuddy operational needs, not mirror an external provider API.

------

# 16. No Arbitrary JSON Configuration

Avoid a field such as:

```text
Provider Config:
{
   ...anything...
}
```

for normal S3 operation.

Why:

- difficult to validate;
- easy to misconfigure;
- exposes provider-specific complexity;
- weakens typed contracts;
- difficult to audit safely.

Prefer explicit supported fields.

------

# 17. Provider Configuration

Provider-level configuration may include operational metadata required by the frozen design.

Conceptually:

```text
Provider
Status
Endpoint / region where supported
Secret reference
Operational availability
```

Exact fields follow backend configuration model.

Provider configuration is not Candidate-facing.

------

# 18. Secrets Boundary

This is a hard security boundary.

OfferBuddy Admin must **not store or display plaintext provider API secrets in the database/UI**.

Correct conceptual model:

```text
Runtime Configuration
      │
      └── secretRef
             │
             ▼
    External secret source
             │
             ▼
      Runtime Provider
```

The Admin may display:

```text
Secret: Configured
```

or:

```text
Secret reference: OPENAI_API_KEY
```

where safe and frozen.

It must not display:

```text
sk-xxxxxxxxxxxxxxxx
```

------

# 19. Secret Reference

A secret reference identifies externally managed secret configuration.

Examples conceptually:

```text
OPENAI_API_KEY
GEMINI_API_KEY
```

Actual naming comes from deployment configuration.

The Admin should not become a secrets vault.

------

# 20. Secret Status

Useful safe states may include:

```text
Configured
Missing
Invalid / unavailable
Unknown
```

depending on frozen runtime capabilities.

If the system cannot safely validate a secret without making an external request:

> do not pretend that `Configured` means the credential is valid.

`Configured` may only mean:

> expected secret reference is resolvable/present.

------

# 21. No Secret Reveal

There must be no:

```text
Show API Key
Copy Secret
Reveal Secret
```

action.

Even privileged Admin users do not need plaintext credentials to operate AI routing.

Secret changes belong to deployment/secret-management infrastructure.

------

# 22. Configuration Edit

Edit should use a standard RuoYi form/dialog.

Conceptually:

```text
Edit AI Capability

Capability:
Match Analysis

Enabled:
[ ON ]

Provider:
[ Provider A ▼ ]

Model:
[ model-x ▼ ]

Approved Runtime Parameters:
...

[ Cancel ] [ Confirm ]
```

Capability identifier should normally not be casually editable after creation.

------

# 23. Validation

Backend must validate configuration before accepting it.

Examples:

```text
enabled capability
→ provider required

provider selected
→ provider must be supported

model selected
→ model must be supported for provider/capability

parameter
→ allowed range

secret reference required
→ resolvable according to config semantics
```

Frontend validation improves UX.

Backend remains authoritative.

------

# 24. Invalid Configuration

The system must not silently activate a clearly invalid configuration.

Example:

```text
MATCH_ANALYSIS
enabled = true
provider = null
```

should fail validation.

Similarly:

```text
provider = Provider A
model = model-only-supported-by-B
```

must not be accepted if the provider registry knows it is invalid.

------

# 25. Configuration Status

The UI may expose a concise status such as:

```text
Ready
Disabled
Configuration required
Secret missing
Invalid configuration
```

Exact states come from the backend.

Do not make the frontend independently derive a complex provider health state from scattered fields.

------

# 26. Save Semantics

On Save:

```text
Admin edit
    ↓
backend validation
    ↓
persist runtime configuration
    ↓
audit
    ↓
runtime configuration becomes effective
```

The exact activation mechanism must follow the frozen backend design.

Do not assume application restart is required unless the configuration contract says so.

------

# 27. Runtime Refresh

S3 runtime configuration should support the frozen runtime refresh strategy without building an enterprise configuration platform.

Conceptually:

```text
DB/runtime config changed
       ↓
configuration service/cache
       ↓
AI Router sees new config
```

Possible implementation may use:

- short-lived local cache;
- explicit invalidation;
- another frozen mechanism.

The UI should not encode assumptions about implementation internals.

------

# 28. In-Flight Requests

Changing provider/configuration does not retroactively mutate an AI request already in progress.

Conceptually:

```text
Request A starts
→ config/provider X

Admin changes config
→ provider Y

Request A completes under X

Request B starts
→ provider Y
```

Do not attempt to migrate an in-flight external AI call to another provider.

------

# 29. Existing Derived Artifacts

Changing provider/model does not automatically invalidate already generated business artifacts merely because the infrastructure changed.

Example:

```text
Resume generated yesterday with Provider A

Today Admin changes to Provider B
```

The Resume does not become stale solely because of provider change.

Artifact staleness remains primarily based on frozen business provenance such as:

```text
Candidate profileVersion
Job contentVersion
```

unless the domain contract explicitly defines additional semantic provenance.

------

# 30. Configuration Change vs Regeneration

Admin changing:

```text
Focused Cover Letter
Provider A → Provider B
```

must not trigger:

```text
regenerate every existing Cover Letter
```

New generation/regeneration happens only through the normal business workflow.

------

# 31. Feature Disable vs Existing Artifacts

Likewise:

```text
TAILORED_RESUME = disabled
```

does not mean existing Tailored Resume resources disappear.

Users may still be allowed to view previously generated artifacts according to the frozen capability semantics.

Admin controls execution availability, not historical data erasure.

------

# 32. Failure Strategy

Provider routing must fail according to the frozen AI infrastructure policy.

Do not invent automatic cross-provider fallback merely because multiple providers exist.

For example, this must not be assumed:

```text
Provider A fails
→ silently send Candidate data to Provider B
```

That changes both operational and external-data boundaries.

Fallback is allowed only if explicitly frozen/configured.

------

# 33. External AI Data Boundary

Every configured provider remains outside OfferBuddy's trusted boundary.

Changing provider changes where approved AI payload data is sent.

Therefore provider routing is a privileged operational action.

The Admin UI should treat it accordingly.

------

# 34. Minimum Data Principle

Admin configuration determines provider routing.

It does **not** determine that the provider receives all available OfferBuddy data.

Each semantic capability continues to build its own minimum approved payload.

Example:

```text
Focused Cover Letter capability
        ↓
approved Candidate facts
+
approved Job facts
+
approved preparation context
        ↓
AI provider
```

Not:

```text
entire database
→ AI provider
```

------

# 35. Prompt Boundary

Raw system prompts / prompt templates are intentionally not an S3 Admin feature.

Do not add:

```text
Prompt Editor
System Prompt
Raw Prompt Preview
```

to the RuoYi page.

Why:

- unnecessary operational exposure;
- increases accidental breakage;
- increases sensitive data risk;
- expands S3 into prompt-management infrastructure.

Prompt/template versioning remains application-controlled where required.

------

# 36. Raw AI Response Boundary

Admin must not display raw AI provider responses as normal governance data.

Operational monitoring can show:

```text
success/failure
latency
provider
model
token usage
cost estimate
error category
```

but not Candidate-derived generated content.

Detailed monitoring is covered by `S3-UI-20`.

------

# 37. Provider Health

Runtime Configuration should avoid becoming a full monitoring page.

A concise provider/config state may be shown, but:

> operational request statistics belong to AI Monitoring.

Conceptually:

```text
S3-UI-19
→ What is configured?

S3-UI-20
→ How is it behaving?
```

Keep these responsibilities distinct.

------

# 38. RuoYi Permissions

Use the actual frozen permission identifiers.

Conceptually permissions may distinguish:

```text
AI config list/view
AI config edit
AI capability enable/disable
```

Do not rely on button visibility.

Backend authorization is mandatory.

Provider routing changes should require privileged permission.

------

# 39. Admin Is Operational Only

RuoYi Admin is not a universal OfferBuddy domain bypass.

AI Governance Admin must not provide access to:

- Candidate Profile;
- Candidate Resume;
- Focused Cover Letter;
- Application private content;

just because those capabilities use AI.

Operational configuration and user business data remain separated.

------

# 40. Audit Requirements

Important privileged actions must be auditable:

```text
capability enabled
capability disabled
provider changed
model changed
runtime parameter changed
```

Audit should identify:

- actor;
- capability/config identifier;
- operation;
- timestamp;
- safe before/after metadata where permitted;
- result.

Never audit plaintext secrets.

------

# 41. Audit Before / After Values

Safe configuration metadata may be captured where appropriate.

Example:

```text
Capability:
MATCH_ANALYSIS

Provider:
Provider A → Provider B
```

Do not include:

```text
secret old value
secret new value
raw prompt
```

Auditability must not become a secret-leak mechanism.

------

# 42. Observability

Operational telemetry may include:

```text
requestId
admin actor identifier
capability key
provider identifier
model identifier
operation
success/failure
duration
```

Do not log:

- API keys;
- raw prompts;
- raw responses;
- Candidate content.

------

# 43. Configuration Cache Failure

If runtime configuration refresh temporarily fails, the backend should follow the frozen safe configuration/cache behaviour.

The Admin UI should not claim a configuration is active merely because the HTTP save returned unless the API contract guarantees activation.

If activation state is exposed separately, show it accurately.

Do not invent distributed configuration synchronisation UI for S3.

------

# 44. Provider Secret Missing

Example:

```text
Focused Cover Letter
Enabled: Yes
Provider: Provider A
Secret: Missing
```

This configuration is not operationally ready.

UI should clearly show:

> Configuration required / Secret missing

Do not expose the expected secret value.

The fix occurs in deployment secret configuration, not by typing the secret into RuoYi.

------

# 45. Provider Unavailable

If the provider is temporarily unavailable:

- do not automatically disable the capability configuration;
- operational monitoring records failures;
- business workflow follows normal retry/failure semantics.

Configuration and transient provider health are different concepts.

------

# 46. AI Retry Boundary

Admin Runtime Configuration should not expose arbitrary retry tuning unless explicitly part of the frozen configuration contract.

Retries are governed by the AI/async infrastructure design.

Avoid allowing an Admin to casually configure:

```text
retry = 100
timeout = 20 minutes
```

and destabilise the system.

------

# 47. Cost Controls

If S3 frozen runtime config includes bounded token/cost parameters, expose only those approved controls.

Otherwise cost is monitoring information, not an arbitrary budget-management subsystem.

Do not build:

- billing;
- budgets;
- invoice reconciliation;
- financial alerts;

as part of this page.

------

# 48. UI States

Required page states:

### Configuration List

- Loading
- Ready
- Load failure

### Capability Edit

- Ready
- Dirty
- Validation error
- Saving
- Save failure
- Conflict where applicable
- Saved

### Configuration Status

- Ready
- Disabled
- Configuration required
- Secret missing
- Invalid

### Permission

- standard RuoYi unauthorised/forbidden behaviour.

------

# 49. Concurrency

Runtime configuration is privileged mutable operational state.

If the frozen contract provides OCC/version semantics, the UI must send the expected version.

Example:

```text
Admin A loads config v8
Admin B saves v9
Admin A saves v8
→ 409
```

Do not silently overwrite the newer configuration.

Do not reuse:

```text
Candidate profileVersion
Application version
Job contentVersion
```

for AI configuration.

------

# 50. Idempotency

Configuration update is normally an idempotent resource update according to its frozen contract.

Enable/disable or other command-style POSTs must follow §3.17 idempotency semantics where defined.

Frontend should prevent accidental duplicate submissions while saving.

------

# 51. Async Behaviour

Normal configuration editing should remain synchronous if the frozen implementation allows it.

Do not introduce asynchronous infrastructure merely for a small DB configuration update.

External AI execution remains asynchronous where its business capability requires it.

Admin config update and AI generation are separate concerns.

------

# 52. Security

Required:

- authenticated RuoYi Admin;
- server-side RBAC;
- validated provider/model values;
- no plaintext secrets;
- no client-controlled Admin identity;
- no Candidate data exposure;
- safe audit;
- safe logs.

Treat provider/model changes as privileged operational mutations.

------

# 53. Privacy

AI Governance deals with configuration metadata.

It must not become a convenient interface for viewing user AI content.

Do not display:

```text
Candidate prompt
Resume prompt
Cover Letter prompt
provider raw response
Candidate Profile JSON
```

on this page.

------

# 54. Performance

Configuration list is small.

Do not overengineer pagination if the frozen RuoYi implementation does not require it, but reuse standard patterns where appropriate.

Runtime AI calls must not query the Admin frontend.

The backend AI router consumes validated configuration directly from backend configuration infrastructure/cache.

------

# 55. Failure Behaviour

### Config Load Failure

Show standard RuoYi retry/error state.

### Validation Failure

Keep form values and show field-level errors.

### Conflict

Do not overwrite newer config silently.

### Save Failure

Previous valid runtime configuration remains authoritative.

### Secret Missing

Show safe configuration status.

### Provider Runtime Failure

Handled through AI execution/monitoring, not by deleting/changing config automatically.

------

# 56. API Mapping

Use frozen §3.17 Admin/AI Governance contracts.

Conceptually:

```text
list AI capability configurations
read capability configuration
update capability configuration
read provider configuration/status
```

Use exact frozen paths.

Do not create endpoints such as:

```text
POST /admin/ai/run-prompt
POST /admin/ai/test-anything
GET  /admin/ai/raw-responses
GET  /admin/ai/secrets
```

unless explicitly frozen—which they are not part of this UI specification.

------

# 57. S2 Reuse

### Reuse

- RuoYi shell;
- RBAC;
- permission directives;
- table/form/dialog primitives;
- operational logging conventions;
- existing AI provider abstraction from earlier implementation where applicable.

### S3 Add

- capability-oriented runtime routing;
- provider/model selection;
- safe runtime configuration;
- capability enable/disable;
- secret-reference status;
- governance audit integration.

### Do Not

- replace RuoYi;
- rewrite provider abstraction inside business modules;
- create a general AI management platform.

------

# 58. Component Reuse

Possible frontend responsibilities:

```text
AiRuntimeConfigList
AiCapabilityConfigDialog
ProviderConfigSummary
ConfigStatusTag
```

Names are illustrative.

Prefer RuoYi's existing primitives over custom component libraries.

Backend responsibilities conceptually remain:

```text
AiCapabilityConfigService
AiCapabilityRouter
AiProviderRegistry
AiRuntimeConfigRepository
```

Exact architecture follows frozen Phase 3 design.

------

# 59. Accessibility / UX

- Enabled/Disabled must not rely only on colour.
- Provider/model controls need explicit labels.
- Dangerous routing changes should be understandable before confirmation where required.
- Secret state should say `Configured/Missing`, never display secret value.
- Validation should identify the exact invalid field.
- Save progress prevents accidental duplicate actions.
- Configuration status should be understandable without requiring knowledge of provider SDK terminology.

------

# 60. Acceptance Criteria

-  AI Governance uses native RuoYi components.
-  RuoYi shell is not redesigned.
-  AI configuration is organised by semantic capability.
-  Exact backend capability keys are used.
-  Capability can be enabled/disabled where frozen contract permits.
-  Disabling a capability does not delete existing artifacts.
-  Provider can be selected from supported providers.
-  Provider is not arbitrary free text.
-  Model can be selected according to supported configuration.
-  Invalid provider/model combinations are rejected.
-  Only approved runtime parameters are exposed.
-  No arbitrary provider JSON editor is introduced.
-  Business modules remain provider-agnostic.
-  Provider routing occurs through AI abstraction/router.
-  Plaintext provider secrets are not stored/displayed by Admin.
-  No Reveal/Copy API Key action exists.
-  Secret references/status can be represented safely.
-  Missing secret is clearly distinguishable from Ready.
-  Admin cannot edit raw system prompts.
-  Admin cannot inspect raw user prompts.
-  Admin cannot inspect raw provider responses.
-  Configuration changes do not regenerate existing artifacts.
-  Provider change alone does not make existing artifacts stale.
-  In-flight request continues under its resolved configuration.
-  New requests use subsequently effective configuration.
-  Automatic provider fallback is not invented.
-  AI capability continues to send only minimum approved data.
-  Provider/config changes require server-side permission.
-  Configuration mutations are auditable.
-  Secrets are excluded from audit/logs.
-  Candidate data is not exposed through AI Governance.
-  OCC conflict is explicit where contract defines versioning.
-  Save failure preserves previous valid configuration.
-  Provider runtime failure does not silently mutate configuration.
-  S2 behaviour remains unaffected.

------

# 61. Agent Implementation Notes

1. Inspect the existing OfferBuddy AI provider abstraction before implementation.
2. Inspect the final RuoYi AI Governance Figma frames.
3. Ignore old blurred/overlapping exploratory frames.
4. Reuse native RuoYi components.
5. Read frozen §3.17 Admin/AI contracts before creating frontend DTOs.
6. Model UI around semantic AI capabilities, not provider SDK methods.
7. Keep business modules provider-independent.
8. Use the central AI capability router.
9. Restrict provider choices to backend-supported registry values.
10. Restrict model choices to supported provider/capability combinations.
11. Expose only explicitly approved runtime parameters.
12. Do not add arbitrary JSON configuration.
13. Never accept/display plaintext provider secrets in RuoYi.
14. Use external secret references.
15. Do not implement secret reveal.
16. Do not implement raw prompt editing.
17. Do not implement raw response viewing.
18. Treat enable/disable as execution governance, not data deletion.
19. Do not regenerate existing artifacts after configuration changes.
20. Do not mark artifacts stale solely because provider/model changed.
21. Do not migrate in-flight AI requests when configuration changes.
22. Do not invent provider fallback.
23. Preserve minimum-data AI boundaries.
24. Enforce RBAC server-side.
25. Audit safe before/after configuration metadata.
26. Exclude secrets/prompts/responses from logs and audit.
27. Handle OCC conflicts explicitly if configured.
28. Keep previous valid configuration on failed update.
29. Keep monitoring concerns out of this page.
30. Do not redesign frozen UI.

```
`S3-UI-19` 可以冻结。

这里最重要的架构关系是：

​```text
                  RuoYi Admin
                       │
                Runtime Config
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Enabled        Provider        Model
                       │
                       ▼
                AI Capability Router
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Provider A   Provider B   Provider ...
          │
          └──── External AI Trust Boundary
```

而不是：

```text
MatchService ─────→ OpenAI
ResumeService ────→ Gemini
CoverLetter ──────→ OpenAI
```

另外把一个容易被 Agent 做过头的地方锁死：**RuoYi 不是 Prompt Engineering Console，也不是 Secrets Manager。** Admin 管的是 routing/configuration；secret 仍然在外部 secret/config 环境里，raw prompt/raw response 都不能因为“Admin”身份而开放。

下一份就是这一组最后一个：

**`S3-UI-20 — AI Monitoring / Operational View`**

它会把我们 Figma 已经 review 的 `requests / success / failure / latency / provider / token / estimated cost / last failure` 做成最终 specification，同时严格限制 **不展示 Candidate 内容、raw prompt、raw response**。完成 `S3-UI-20` 后，我们就可以做一次 **S3 Page Specification Final Coverage Review**，确认所有冻结 UI 都有 Agent 可执行 specification，没有漏页。