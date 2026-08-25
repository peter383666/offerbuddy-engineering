好，我们正式开始第一份。

# S3-UI-01 — Match Analysis

```yaml
spec_id: S3-UI-01
title: Match Analysis
surface: web
status: frozen
figma_page: S3-01 Focused Apply
figma_nodes:
  - "282:2"
depends_on:
  - candidate-profile
  - job
  - job-intelligence
  - preparation
  - match
s2_reuse: true
```

## 1. Purpose

Match Analysis 是 **Focused Apply 的第一个 Web 决策页面**。

它回答的核心问题是：

> **Is this job worth a focused application?**

用户已经在招聘网站看到一个感兴趣的 Job，并通过 Extension 的 `Prepare with OfferBuddy` 进入 Web App。

页面的目的不是简单给一个 AI 分数，而是帮助用户快速判断：

- Candidate 与 Job 的整体匹配程度；
- 哪些要求已经满足；
- 哪些要求存在 gap；
- 哪些要求需要用户特别注意；
- 是否值得继续花几分钟制作 Tailored Resume / Focused Cover Letter。

------

## 2. Scope

### In Scope

- 展示当前 Job 的基本 context；
- 展示 explainable Match result；
- 展示 strengths / matched requirements；
- 展示 gaps / missing or weak evidence；
- 展示需要关注的重要 Job requirement；
- 显示 Match generation / stale / failure 状态；
- 允许用户继续进入 Resume preparation；
- 允许用户返回招聘网站而不继续精投。

### Out of Scope

- 修改 Candidate Profile；
- 自动修改 Candidate facts；
- 在本页面编辑 Job；
- 自动投递；
- 在 Extension 中复制 Match result；
- Resume editing；
- Cover Letter editing；
- 根据 Match score 阻止用户申请；
- 根据 AI 推断 Candidate 没有明确提供的经历、技能或资格。

------

## 3. Entry Points

主要入口：

```text
Recruitment Website
        ↓
Extension Floating Assistant
        ↓
Prepare with OfferBuddy
        ↓
Focused Preparation
        ↓
Match Analysis
```

用户也可以从已有 Preparation 重新打开 Match Analysis。

如果 Match 已经生成且仍然 fresh，应优先展示已有结果，而不是无条件重新调用 AI。

------

## 4. Preconditions

进入本页面必须已经存在：

- authenticated user；
- authorised Candidate；
- canonical Job；
- Candidate + Job Preparation relationship。

Candidate Profile 可以不“100% 完整”，但必须满足 Match capability 所要求的最低输入条件。

如果 Profile 信息不足，应显式提示，而不是让 AI 补全事实。

Application **不是进入 Match Analysis 的前置条件**。

这符合冻结的 S3 模型：

> Preparation anchored to Candidate + Job, not Application.

------

## 5. Route / Surface

**Surface:** Web application

建议由 Focused Preparation route/context 承载，而不是把 Match 建模成独立于 Preparation 的产品入口。

最终实际 route naming 应遵循已冻结的前端/API routing convention；Coding Agent 不应根据本 specification 自行创造新的 backend resource model。

------

## 6. Figma Reference

**Figma Page:** `S3-01 Focused Apply`

**Final implementation frame:**

- `S3 Web — Match Decision`
- Node: `282:2`

早期 exploratory Match frame 不作为最终 implementation source。

Figma 中：

```text
ANNOTATION — ...
```

均为设计说明，**不得实现成用户界面**。

------

## 7. UI Structure

页面从上到下建议理解为：

### A. Existing OfferBuddy Web Shell

复用现有 S2：

- header；
- navigation；
- page width；
- typography；
- button system；
- common feedback components。

不得为了 S3 重写 Web shell。

### B. Job Context

展示足够识别当前 Job 的信息，例如：

- Role title；
- Company；
- Location；
- source / relevant job context。

目的只是让用户明确：

> “我现在分析的是哪份工作？”

不重新复制完整 Job Detail。

### C. Match Decision Summary

展示整体 Match 判断。

包括：

- overall Match indication；
- 简短结论；
- 对用户是否值得继续精投的解释。

Match score 是辅助信息，而不是业务 gate。

### D. Strengths / Matched Requirements

展示 Candidate Profile 中已有事实如何对应 Job requirements。

例如：

```text
Java / Spring Boot
✓ Strong evidence from Candidate Profile

Backend API development
✓ Supported by work experience
```

### E. Gaps / Attention Areas

展示：

- missing requirement；
- weak evidence；
- unclear requirement；
- eligibility concern；
- experience gap。

必须区分：

> **Candidate 明确不满足**

和

> **OfferBuddy 没有足够 Candidate evidence**

这两个语义不能混为一谈。

### F. Decision Actions

主要动作：

**Continue with tailored resume**

次要动作：

**Return to job / continue applying without focused preparation**

用户永远可以不继续 Focused Apply。

------

## 8. Data Dependencies

### Reads

Candidate：

- candidateId；
- profileVersion；
- capability-relevant Candidate facts。

Job：

- jobId；
- contentVersion；
- canonical Job content。

Job Intelligence：

- structured requirements；
- skills；
- responsibilities；
- eligibility / requirement signals；
- relevant intelligence provenance/state。

Preparation：

- preparationId；
- Candidate/Job association；
- preparation state。

Match：

- match resource；
- source profileVersion；
- source job contentVersion；
- generation status；
- explainable analysis；
- provenance/staleness metadata。

### Writes

本页面本身**不修改 Candidate facts 或 Job facts**。

可能产生的业务 command：

- request/generate Match；
- regenerate stale Match；
- continue preparation workflow。

### Derived / Async Data

Match 是 derived AI artifact。

因此 UI 必须接受：

```text
requested
→ processing
→ ready
```

以及：

```text
processing
→ failed
```

而不是假设每次 HTTP request 都同步返回最终 Match。

------

## 9. Field Specification

本页面主要为 read-oriented decision UI，因此没有普通 CRUD form。

核心展示字段可理解为：

| Field               | Type         | Editable | Behaviour                         |
| ------------------- | ------------ | -------- | --------------------------------- |
| Job title           | Text         | No       | Current canonical Job             |
| Company             | Text         | No       | Current Job                       |
| Match indication    | Derived      | No       | Explainable decision aid          |
| Strengths           | Derived list | No       | Must reference Candidate evidence |
| Gaps                | Derived list | No       | Must not invent Candidate facts   |
| Requirement signals | Derived      | No       | From Job/Job Intelligence         |
| Freshness           | State        | No       | Based on provenance versions      |

------

## 10. Actions

### Continue with Tailored Resume

**Trigger:** user decides this Job deserves a focused application.

**Behaviour:**

Continue the same Preparation into:

> ```
> S3-UI-02 — Create Tailored Resume
> ```

Do not create a new unrelated Job/Application workflow.

**Success:** Resume preparation surface opens.

**Failure:** Keep user on Match page and provide actionable feedback.

------

### Return to Job

**Trigger:** user decides not to spend more time preparing.

**Behaviour:** return to the originating recruitment context where possible.

This must not:

- delete Preparation;
- create fake Application;
- mark Job as applied;
- require Resume generation.

------

### Regenerate Match

Only available when regeneration is semantically justified, particularly stale or previously failed Match.

Do not provide an unrestricted “keep asking AI until I like the score” interaction.

------

## 11. UI States

### Loading

Initial resource loading.

Do not display an empty Match result while data is still loading.

### Generating

Match requested but AI work is still processing.

Display:

- clear progress state;
- disabled duplicate generation action;
- navigation that does not corrupt the Preparation.

Do not implement fake percentage progress unless supported by backend semantics.

### Ready

Fresh Match result available.

Display full decision UI.

### Insufficient Profile

Candidate data does not satisfy Match capability requirements.

Explain what information is missing.

Do not fabricate missing facts.

### Failed

Generation failed.

Provide retry where backend semantics allow it.

Do not expose provider raw error.

### Stale

Existing Match provenance no longer matches current Candidate/Job semantic versions.

Example:

```text
Match source profileVersion = 7
Current Candidate profileVersion = 8
```

The old result must not silently masquerade as current.

Offer regeneration.

### Unavailable

Required Job Intelligence or upstream business resource is not ready/available.

Represent the actual business-resource state.

------

## 12. Business Rules

1. Match is derived from **Candidate Profile + Job/Job Intelligence**.
2. Candidate Profile remains authoritative for Candidate facts.
3. Match must never create Candidate facts.
4. A high score does not automatically submit or continue an application.
5. A low score does not prevent the user from applying.
6. Focused Apply remains optional.
7. Match is explainable. UI must not reduce the capability to an unexplained number.
8. Evidence absence must not automatically be represented as proven absence.
9. Match belongs to the Preparation workflow, not Application lifecycle.
10. Application may not yet exist.
11. Match must carry provenance sufficient to determine staleness.
12. Candidate `profileVersion` and Job `contentVersion` changes may make Match stale.

------

## 13. API Mapping

Implementation must use the **frozen Phase 3 §3.17 Match / Preparation contracts**.

The Agent must not infer endpoint names from this document.

Conceptually required operations are:

### Read Preparation / Match

Retrieve current authorised Preparation and its Match state.

### Generate Match

Request Match generation for the existing Candidate + Job Preparation.

AI/external work may return **HTTP 202**.

The frontend then observes the **business resource**, rather than creating a generic client-side operation/task model.

### Regenerate Stale Match

Use the frozen command semantics and current provenance/version requirements.

Any stale command conflict must follow the frozen API behaviour.

------

## 14. Concurrency / Staleness

Relevant provenance:

```text
Candidate.profileVersion
Job.contentVersion
```

Match records the versions from which it was derived.

Example:

```text
Generated:
profileVersion = 12
jobContentVersion = 4

Current:
profileVersion = 13
jobContentVersion = 4

→ Match is stale
```

UI response:

> Candidate Profile has changed since this analysis was generated.

Provide an appropriate regenerate action.

Do **not** silently regenerate without communicating state when that would hide meaningful provenance from the user.

Do not mutate old Match content and pretend it came from the newer Profile.

------

## 15. Navigation / Transitions

| From                 | Action                  | Destination                  |
| -------------------- | ----------------------- | ---------------------------- |
| Extension            | Prepare with OfferBuddy | Match Analysis / Preparation |
| Match generating     | generation completes    | Match Ready                  |
| Match ready          | Continue                | Create Tailored Resume       |
| Match ready          | Return to Job           | Recruitment website          |
| Match stale          | Regenerate              | Generating                   |
| Match failed         | Retry                   | Generating                   |
| Existing Preparation | Open Match              | Match Analysis               |

------

## 16. S2 Reuse

### Preserve

- existing OfferBuddy Web shell;
- authentication;
- navigation;
- common button/input/status patterns;
- Job/Application infrastructure where applicable.

### S3 Delta

Introduce Focused Preparation Match UI.

### Do Not

- rewrite S2 Application flow;
- require Match for Fast Apply;
- make Candidate Profile mandatory globally;
- merge Match into Application Detail;
- add Match to Extension.

------

## 17. Component Reuse

Before creating components, Agent must inspect the current React codebase.

Prefer existing:

- Page shell;
- Header/navigation;
- Card;
- Button;
- Badge/status;
- Loading indicator;
- Error presentation;
- typography/spacing tokens.

Likely S3-specific components may include concepts such as:

```text
MatchSummary
MatchStrengthList
MatchGapList
PreparationStatus
StaleArtifactNotice
```

These names are illustrative, **not mandatory component names**.

Do not build an S3-only design system.

------

## 18. Responsive / Surface Behaviour

Primary frozen design is desktop Web.

Implementation should remain reasonably responsive using the existing OfferBuddy Web layout conventions.

Do not invent a separate mobile product experience.

Extension has its own specification and must not reuse this full Match layout.

------

## 19. Accessibility / UX Requirements

- Match meaning must not depend solely on colour.
- Strength/gap state should include textual/iconographic meaning.
- Loading and generation actions must expose disabled state.
- Errors must be actionable.
- Keyboard navigation should follow existing Web component behaviour.
- Stale result warning must be visible before the user treats old analysis as current.

------

## 20. Analytics / Observability

Do not introduce new product analytics solely from this UI specification.

Operational diagnostics should preserve existing identifiers where applicable:

- requestId;
- correlationId internally;
- preparationId;
- match resource identifier;
- AI capability/provider metadata according to observability rules.

Frontend must not log:

- Candidate Profile content;
- Resume content;
- raw Job description;
- AI prompt;
- raw AI response.

------

## 21. Security / Privacy

Candidate/Preparation/Match access is authorised server-side through Candidate ownership.

Frontend identifiers are not authorization proof.

Do not expose:

- AI credentials;
- raw prompts;
- raw provider responses;
- internal secrets;
- another user's Preparation/Match.

Ownership failures follow frozen API security semantics.

------

## 22. Acceptance Criteria

-  User can enter Match Analysis from Focused Preparation.
-  Fast Apply works without Match Analysis.
-  Current Job context is clearly identifiable.
-  Ready Match provides an explainable decision, not only a score.
-  Strengths are grounded in Candidate facts.
-  Missing evidence is not falsely represented as a confirmed missing capability.
-  Match cannot modify Candidate Profile.
-  User can continue to Tailored Resume.
-  User can abandon Focused Apply and return to the Job.
-  Async generation state is represented correctly.
-  Duplicate generation actions are prevented while processing.
-  Failure state supports safe retry.
-  Stale Match is visibly identified.
-  Candidate `profileVersion` changes can invalidate Match.
-  Job `contentVersion` changes can invalidate Match.
-  Extension does not display the Match result.
-  Existing S2 Fast Path is unchanged.
-  No raw AI/provider-sensitive content is exposed.

------

## 23. Agent Implementation Notes

Before implementation:

1. Read the frozen Match/Preparation API contracts in Phase 3 §3.17.
2. Inspect existing S2 React routing and shared components.
3. Inspect existing Job/Application UI before creating equivalent components.
4. Use Figma node `282:2` as the visual source.
5. Ignore Figma `ANNOTATION — ...` nodes as application UI.
6. Preserve Candidate + Job Preparation as the workflow anchor.
7. Do not introduce Application as a prerequisite.
8. Do not implement Match inside the Browser Extension.
9. Do not create a generic frontend `AI task` abstraction when the API exposes business-resource polling.
10. Do not infer Candidate facts from Match output.
11. Do not silently hide stale provenance.
12. Do not redesign the frozen Figma layout during implementation.

```
### S3-UI-01 Review Result

这份现在已经具备我们想要的 Agent contract 结构：

**Figma 告诉 Agent 长什么样；Page Spec 告诉 Agent 怎么工作；Phase 3 告诉 Agent 后端和领域为什么必须这样工作。**

下一份应该是 **`S3-UI-02 — Create Tailored Resume`**。这一份会比较关键，因为要把我们已经确定的 **Candidate Profile → multiple Base Resumes → choose starting point → job-specific Tailored Resume** 模型完整固定下来。
```