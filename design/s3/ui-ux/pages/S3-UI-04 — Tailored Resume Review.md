S3-UI-04 — Tailored Resume Review

继续。下面进入生成后真正需要用户确认的 Resume 页面。

# S3-UI-04 — Tailored Resume Review

```yaml
spec_id: S3-UI-04
title: Tailored Resume Review
surface: web
status: frozen
figma_page: S3-01 Focused Apply
figma_nodes:
  - "286:2"
depends_on:
  - candidate-profile
  - job
  - preparation
  - match
  - resume
s2_reuse: true
```

## 1. Purpose

`Tailored Resume Review` 是 Focused Apply 中 Tailored Resume 生成完成后的 review surface。

它回答的问题是：

> **OfferBuddy 为这份 Job 准备的 Resume 是否可以使用？**

用户需要能够：

- 查看生成结果；
- 理解这是针对当前 Job 的 Tailored Resume；
- 对结果做必要的人工修改；
- 保存修改；
- 重新生成（仅在合理场景下）；
- 继续进入 Focused Cover Letter；
- 识别 Candidate Profile / Job 变化造成的 stale 状态。

核心原则：

> **Tailored Resume 是 derived artifact，不是 Candidate Profile，也不是 Base Resume。**

因此这里的修改默认只影响：

> **当前 Preparation 的当前 Tailored Resume。**

------

# 2. Scope

### In Scope

- 展示当前 Job context；
- 展示生成完成的 Tailored Resume；
- Review Resume 内容；
- 对当前 Tailored Resume 做有限人工编辑；
- 保存 Tailored Resume；
- 表达 unsaved changes；
- 表达 stale state；
- 在允许情况下重新生成；
- 继续到 Focused Cover Letter；
- 返回 Preparation 其他阶段。

### Out of Scope

- 自动更新 Candidate Profile；
- 自动更新 Base Resume；
- Full Resume Designer；
- 任意拖拽式 Resume builder；
- 多模板排版系统；
- Candidate Profile fact creation；
- Job editing；
- Application submission；
- Auto Apply；
- 将当前 Tailored Resume 自动设置成新的 Base Resume；
- 在 Extension 中编辑 Resume。

------

# 3. Entry Points

主要路径：

```text
Match Analysis
      ↓
Create Tailored Resume
      ↓
Choose Starting Point
      ↓
Generate
      ↓
Processing
      ↓
Tailored Resume Review
```

已有 Preparation 也可以：

```text
Preparation
      ↓
Resume already exists
      ↓
Open Resume
      ↓
Tailored Resume Review
```

如果 Resume 已 stale，仍然可以进入 Review，但必须明确显示 stale 状态。

------

# 4. Preconditions

必须存在：

- authenticated user；
- authorised Candidate；
- canonical Job；
- Candidate + Job Preparation；
- Tailored Resume business resource。

Tailored Resume resource 可能处于：

```text
PROCESSING
READY
FAILED
STALE
```

只有存在可 review 的生成结果时才展示 Resume document。

Application 仍然不是 prerequisite。

------

# 5. Route / Surface

**Surface:** Web application / Focused Preparation.

Tailored Resume Review 属于当前 Preparation，而不是独立的通用 Resume Builder。

概念结构：

```text
Preparation
   │
   ├── Match
   │
   ├── Tailored Resume  ← current surface
   │
   └── Focused Cover Letter
```

------

# 6. Figma Reference

**Figma Page**

```
S3-01 Focused Apply
```

**Frame**

```
S3 Web — Tailored Resume Review
```

**Node**

```
286:2
```

实现以 final Figma frame 为视觉 source of truth。

所有：

```text
ANNOTATION — ...
```

仅为设计说明。

------

# 7. UI Structure

页面从上到下分为以下区域。

### A. Existing OfferBuddy Web Shell

复用现有：

- navigation；
- header；
- page container；
- buttons；
- typography；
- common feedback patterns。

------

### B. Job / Preparation Context

让用户明确：

> 这份 Resume 是针对哪一个 Job。

显示足够识别当前 Job 的：

- Role title；
- Company；
- relevant preparation context。

不要复制完整 Job Detail。

------

### C. Resume Status / Provenance Context

明确告诉用户：

> Tailored for this job

如果使用 Base Resume，可适当表达 starting point，例如：

> Based on: Backend Engineer Resume

如果从 Candidate Profile 创建：

> Built from Candidate Profile

这里表达的是 provenance/context，不是让用户修改 provenance。

------

### D. Tailored Resume Document

展示生成后的 Resume。

用户应能够 review：

- Professional Summary；
- Skills；
- Experience；
- Education；
- Certifications；
- 其他当前 Resume 实际包含的 sections。

UI 不应额外制造 Candidate Profile 中不存在的 Resume section。

------

### E. Editing State

Tailored Resume 可以进行必要的人工调整。

但这里不是 Full Resume Designer。

允许的交互重点是：

> **内容 review / wording correction / removal / small adjustment**

而不是：

- page builder；
- arbitrary layout editor；
- template marketplace；
- typography designer。

------

### F. Actions

核心 actions：

- `Save`
- `Regenerate`（适用时）
- `Continue to Cover Letter`

以及必要的 Back/navigation。

------

# 8. Data Dependencies

### Reads — Tailored Resume

包括：

- tailoredResumeId；
- preparationId；
- status；
- content；
- selected starting point provenance；
- source profileVersion；
- source job contentVersion；
- generation metadata；
- stale state；
- version/concurrency token where defined。

------

### Reads — Candidate Profile

用于 provenance / validation：

- candidateId；
- current profileVersion。

UI 不需要重新加载并复制整个 Profile。

------

### Reads — Job

- jobId；
- current contentVersion；
- Role；
- Company。

------

### Reads — Preparation

- preparationId；
- current workflow state；
- associated Candidate + Job。

------

### Writes

只修改：

> current Tailored Resume artifact.

不得隐式修改：

- Candidate Profile；
- Base Resume；
- Job；
- Application。

------

# 9. Field Specification

Tailored Resume 实际 editable content 应遵循 frozen Resume domain representation。

概念上：

| Content                      | Editable   | Truth Boundary                           |
| ---------------------------- | ---------- | ---------------------------------------- |
| Summary wording              | Yes        | Cannot introduce unsupported facts       |
| Skill presentation           | Limited    | Must remain supported by Candidate facts |
| Experience wording           | Yes        | Cannot invent experience                 |
| Experience ordering/emphasis | Yes        | Within supported facts                   |
| Education wording            | Limited    | Cannot invent qualification              |
| Certifications               | Limited    | Cannot invent certification              |
| Contact/personal facts       | Restricted | Candidate Profile remains authoritative  |

一个特别重要的实现原则：

> **Editable does not mean fact-authoritative.**

例如用户在 Tailored Resume 中修改一句描述，并不自动更新 Candidate Profile。

------

# 10. Actions

## Save

### Trigger

用户修改当前 Tailored Resume 后点击 Save。

### Behaviour

保存当前 Tailored Resume artifact。

必须携带 frozen contract 所要求的：

- resource identity；
- version / concurrency information。

### Success

- Resume 更新；
- dirty state 清除；
- remain on Review page。

### Conflict

如果 Tailored Resume 已被其他 write 更新：

> 不允许 silent overwrite。

按照 frozen OCC semantics 显示 conflict。

------

## Continue to Cover Letter

### Trigger

用户确认 Resume 可以继续使用。

### Behaviour

进入：

> ```
> S3-UI-05 — Focused Cover Letter Review
> ```

如果有未保存修改：

不得静默丢失。

可以：

- require save；
- 或使用现有 OfferBuddy unsaved-changes pattern。

不要自行发明复杂 draft mechanism。

------

## Regenerate

Regenerate 不是普通文本编辑的替代按钮。

合理场景包括：

- 用户明确希望重新生成；
- Resume stale；
- previous generation result不适用；
- upstream inputs 已发生有效变化。

Regenerate 必须基于**当前 authoritative inputs**。

不能：

```text
Old Profile facts
+ click regenerate repeatedly
→ random Resume lottery
```

### Behaviour

使用 frozen Resume generation command。

异步时：

```text
Regenerate
   ↓
202
   ↓
Tailored Resume PROCESSING
   ↓
poll/read business resource
   ↓
READY
```

------

## Back

返回 Preparation 前一步或 overview。

不得：

- 删除 Tailored Resume；
- 回滚 Candidate Profile；
- 创建 Application。

------

# 11. UI States

## Loading

正在加载 existing Tailored Resume。

------

## Processing

AI generation 尚未完成。

显示明确的 processing state。

不得显示 fake progress percentage。

------

## Ready / Clean

Resume 已生成且无本地未保存修改。

------

## Ready / Editing

用户正在修改 Resume。

------

## Dirty

存在 unsaved changes。

导航离开前不能静默丢失。

------

## Saving

禁用重复 Save。

提供明确状态。

------

## Save Failure

保留用户当前编辑内容。

不能因为 API failure 清空 editor。

------

## Generation Failure

如果 generation failed：

- 显示安全 failure message；
- 允许合理 retry；
- 不展示 raw provider response。

------

## Stale

如果 authoritative input 已变化：

```text
Resume generated from:
profileVersion = 12
jobContentVersion = 5

Current:
profileVersion = 13
jobContentVersion = 5
```

Resume 必须被识别为 stale。

UI 应明确告诉用户：

> Candidate Profile has changed since this resume was generated.

并提供适当 regeneration path。

------

## Conflict

保存时发生 OCC conflict。

不能：

> fetch latest → automatically overwrite → pretend success.

用户必须得到明确反馈。

------

# 12. Business Rules

1. Tailored Resume 是 **Job-specific derived artifact**。
2. Tailored Resume 属于 Preparation。
3. Application 不是 prerequisite。
4. Candidate Profile 是 Candidate fact authority。
5. Base Resume 是 starting point，不是 fact authority。
6. Tailored Resume 修改不会自动反向更新 Candidate Profile。
7. Tailored Resume 修改不会自动覆盖 Base Resume。
8. AI 不得制造 Candidate facts。
9. 用户编辑 Tailored Resume 也不自动建立新的 Candidate Profile facts。
10. 如果用户希望永久修改 Candidate facts，应进入 Candidate Profile workflow。
11. Resume 可以 reorder / emphasise / rewrite truthful facts。
12. Resume 不得伪造 experience、skill、education、certification、eligibility。
13. Match 可帮助决定强调什么，但不能建立事实。
14. Resume 必须保留 provenance。
15. Candidate `profileVersion` 或 Job `contentVersion` semantic change 可以使 Resume stale。
16. Stale artifact 不得静默表现为 current。
17. Fast Apply 不依赖 Tailored Resume。

------

# 13. API Mapping

使用 Phase 3 §3.17 frozen Tailored Resume / Preparation contracts。

Agent 不得根据 editor 自行设计新 API。

概念能力包括：

### Read Tailored Resume

读取当前 Preparation 的 Tailored Resume resource。

### Update Tailored Resume

保存用户对当前 artifact 的允许修改。

必须遵循 OCC。

### Generate / Regenerate

使用 frozen Resume generation command。

AI/external operation 使用业务资源状态，而不是 generic frontend AI task。

------

# 14. Concurrency / Staleness

这里存在两个不同概念，Agent 不能混淆。

### A. Artifact OCC

解决：

> 两个 write 谁覆盖谁？

例如：

```text
Tailored Resume version 3
       ↓
Browser A edits
Browser B edits
       ↓
A saves → version 4
B saves version 3
       ↓
409 conflict
```

B 不能 silent overwrite。

### B. Provenance Staleness

解决：

> Resume 还是不是基于当前事实生成的？

例如：

```text
Resume
sourceProfileVersion = 12

Candidate Profile
profileVersion = 13

→ Resume stale
```

这不一定意味着 Resume 内容立刻消失。

而是意味着：

> 用户必须知道它已经不是基于当前 Candidate facts 的最新生成结果。

两者必须分别处理。

------

# 15. Navigation / Transitions

| From                   | Action           | Destination / State        |
| ---------------------- | ---------------- | -------------------------- |
| Create Tailored Resume | Generation ready | Tailored Resume Review     |
| Preparation            | Open Resume      | Tailored Resume Review     |
| Review                 | Edit             | Editing                    |
| Editing                | Save             | Saving                     |
| Saving                 | Success          | Ready                      |
| Saving                 | Conflict         | Conflict                   |
| Review                 | Regenerate       | Processing                 |
| Processing             | Ready            | Review                     |
| Review                 | Continue         | Focused Cover Letter       |
| Review                 | Back             | Preparation previous state |
| Stale                  | Regenerate       | Processing                 |

------

# 16. S2 Reuse

### Reuse

- Web shell；
- buttons；
- form/editor conventions；
- loading/error patterns；
- unsaved-changes pattern if already present；
- existing document presentation components where suitable。

### S3 Delta

增加：

- Preparation-specific Tailored Resume Review；
- limited artifact editing；
- provenance/staleness handling；
- transition to Focused Cover Letter。

### Do Not

- introduce Full Resume Designer；
- rewrite Candidate Profile；
- rewrite S2 Application；
- require Tailored Resume for Fast Apply；
- add Resume editor to Extension。

------

# 17. Component Reuse

Agent 应先检查当前 frontend。

可能出现的 S3-specific responsibilities：

```text
TailoredResumeReview
ResumeDocumentEditor
ArtifactStatusNotice
StaleArtifactNotice
ResumeActionBar
```

名称不作为强制要求。

如果 existing editor/document components 可以满足需求：

> reuse first.

尤其不要出现：

```text
BaseResumeRenderer
TailoredResumeRenderer
ApplicationResumeRenderer
```

三套几乎一样的 document renderer。

------

# 18. Responsive / Surface Behaviour

主要 frozen surface 是 desktop Web。

Resume document 应保持：

- readable width；
- stable editing experience；
- existing responsive shell behaviour。

不要求：

- mobile Resume Designer；
- Extension Resume editor。

------

# 19. Accessibility / UX Requirements

- Editor 必须可 keyboard 操作。
- Save / Continue / Regenerate hierarchy 清晰。
- Dirty state 不得只依赖颜色。
- Stale warning 必须可感知。
- Save failure 后不得丢失用户输入。
- Processing 时禁止重复 generation。
- Disabled button 应有正确 semantic state。
- Error message 应可行动，而不是只显示 `Something went wrong`。

------

# 20. Analytics / Observability

不从 Page Spec 新增 product analytics。

Operational diagnostics 遵循 frozen observability design。

可以关联：

- requestId；
- correlationId；
- preparationId；
- tailoredResumeId；
- business event / AI capability metadata。

不得 log：

- Resume正文；
- Candidate facts；
- raw Job description；
- prompt；
- raw AI response。

------

# 21. Security / Privacy

Tailored Resume 是 Candidate-owned sensitive business data。

Backend 必须通过 Candidate ownership authorization。

Admin 不拥有 universal bypass。

Frontend 不得信任：

- candidateId；
- preparationId；
- tailoredResumeId

作为授权依据。

AI provider 仍位于 OfferBuddy trust boundary 外。

仅发送 frozen AI boundary 允许的数据。

------

# 22. Acceptance Criteria

-  User can open the generated Tailored Resume.
-  Current Job context is visible.
-  Resume is clearly identified as job-specific.
-  Starting-point provenance can be understood where relevant.
-  User can review Resume content.
-  User can make permitted edits.
-  Editing affects only the current Tailored Resume.
-  Editing does not update Candidate Profile.
-  Editing does not overwrite Base Resume.
-  Unsupported Candidate facts are not introduced by generation.
-  User can save edits.
-  Save failure preserves local user edits.
-  Concurrent stale write is not silently overwritten.
-  User can continue to Focused Cover Letter.
-  Unsaved changes are not silently discarded.
-  User can regenerate when appropriate.
-  Duplicate generation is prevented while processing.
-  Async generation state is represented.
-  Candidate Profile revision can make Resume stale.
-  Job semantic revision can make Resume stale.
-  Stale Resume is visibly identified.
-  Fast Apply remains independent.
-  Extension does not contain this editor.
-  No raw AI/provider-sensitive data is exposed.

------

# 23. Agent Implementation Notes

Before implementation:

1. Read frozen Tailored Resume API contract.
2. Read Candidate Profile provenance/version rules.
3. Inspect existing document/editor components.
4. Use Figma node `286:2`.
5. Treat Tailored Resume as a Preparation artifact.
6. Do not create Application as part of Resume generation.
7. Do not update Candidate Profile when Resume is edited.
8. Do not overwrite Base Resume.
9. Separate **artifact OCC conflict** from **input provenance staleness**.
10. Preserve local edits on save failure.
11. Do not silently merge 409 conflicts.
12. Use business-resource async state.
13. Do not create generic AI operation IDs.
14. Do not implement Full Resume Designer.
15. Do not expose Resume functionality inside Extension.
16. Reuse existing S2 components before creating new ones.
17. Ignore Figma annotations.
18. Do not redesign the frozen UI.

```
## 一个这里需要正式冻结的细节

`Tailored Resume Review` 中的 **Edit** 应理解为：

> **对当前 job-specific artifact 做有限内容修正。**

而不是把 S3 偷偷扩张成 Resume Designer。

同时这里有三层数据必须严格单向区分：

​```text
Candidate Profile
(authoritative facts)
        ↓
Base Resume
(reusable expression / starting point)
        ↓
Tailored Resume
(job-specific derived artifact)
```

在 Tailored Resume 中人工改一句话，**不会反向修改 Profile，也不会覆盖 Base Resume**。如果用户发现根本事实错了，应回 Candidate Profile 修正；Profile version 更新后，当前 Tailored Resume 根据 provenance 进入 stale。

这样后面 Agent 就不会为了“编辑 Resume”顺手做出双向同步这种很危险的行为。

下一份是 **`S3-UI-05 — Focused Cover Letter Review`**，完成后 `Focused Apply Web` 这一组 5 份 specification 就全部完成。