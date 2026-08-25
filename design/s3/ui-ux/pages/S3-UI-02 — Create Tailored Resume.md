S3-UI-02 — Create Tailored Resume

继续。下面是第二份正式 Page Specification。

# S3-UI-02 — Create Tailored Resume

```yaml
spec_id: S3-UI-02
title: Create Tailored Resume
surface: web
status: frozen
figma_page: S3-01 Focused Apply
figma_nodes:
  - "298:2"
depends_on:
  - candidate-profile
  - job
  - preparation
  - match
  - resume
s2_reuse: true
```

## 1. Purpose

`Create Tailored Resume` 是 Focused Apply 中从 **“这个 Job 是否值得精投”** 进入 **“我要用什么 Resume 作为这次精投的起点”** 的决策页面。

核心问题不是：

> Generate a resume?

而是：

> **Which truthful resume starting point should OfferBuddy tailor for this job?**

用户可能已经拥有多份 Resume，例如：

```text
Backend Engineer Resume
Full Stack Resume
General Software Engineer Resume
```

也可能没有任何 Base Resume。

因此 S3 的模型冻结为：

```text
Candidate Profile
      │
      │ authoritative facts
      ▼
Multiple Base Resumes
      │
      ├── Backend Resume
      ├── Full Stack Resume
      └── General Resume
      │
      ▼
Choose starting point
      │
      ▼
Job-specific Tailored Resume
```

Base Resume 是表达/组织 Candidate facts 的一种已有方式。

它**不是第二份 Candidate Profile**。

------

# 2. Scope

### In Scope

- 展示当前 Focused Preparation / Job context；
- 展示已有 Base Resumes；
- 允许查看 Base Resume；
- 允许选择一个 Base Resume；
- 允许不使用 Base Resume、直接从 Candidate Profile 开始；
- 请求生成当前 Job-specific Tailored Resume；
- 表达 generating / failure / stale-input 等必要状态；
- 进入 Tailored Resume Review。

### Out of Scope

- 在这里编辑 Candidate Profile；
- 在这里编辑 Base Resume；
- 创建第二个 Candidate Profile；
- 上传 Resume 并自动覆盖 Candidate facts；
- 修改 Job；
- Cover Letter generation；
- Application submission；
- Auto Apply；
- 自动决定用户“必须使用”哪份 Base Resume；
- 允许 AI 发明 Base Resume 中不存在、Candidate Profile 中也不存在的事实。

------

# 3. Entry Points

主要路径：

```text
Recruitment Website
        ↓
Prepare with OfferBuddy
        ↓
Match Analysis
        ↓
Continue with Tailored Resume
        ↓
Create Tailored Resume
```

也可能从已有 Preparation 中重新进入 Resume preparation。

如果当前 Preparation 已经存在 Tailored Resume，应根据当前 artifact 状态进入适当页面，而不是无条件重新生成。

例如：

```text
Preparation
   ↓
Tailored Resume exists + fresh
   ↓
Tailored Resume Review
```

而不是：

```text
Preparation
   ↓
Create another Tailored Resume automatically
```

------

# 4. Preconditions

必须存在：

- authenticated user；
- authorised Candidate；
- Candidate Profile；
- canonical Job；
- Candidate + Job Preparation。

通常已经存在 Match Analysis，但 Resume capability 不应该被 UI 错误建模成“Match score 必须达到某个阈值”。

用户即使看到较弱 Match，也可以继续准备 Resume。

Candidate Profile 必须满足 Tailored Resume capability 所需的最低事实条件。

如果事实不足：

> 提示用户补充 Candidate Profile。

不能让 AI 自己填空。

------

# 5. Route / Surface

**Surface:** Web application / Focused Preparation state.

该页面属于同一个 Preparation workflow。

概念上：

```text
Preparation
├── Match
├── Resume
└── Cover Letter
```

不要将 Create Tailored Resume 建模成独立于 Candidate + Job Preparation 的孤立生成器。

------

# 6. Figma Reference

**Figma Page**

```
S3-01 Focused Apply
```

**Final Frame**

```
S3 Web — Create Tailored Resume
```

**Node**

```
298:2
```

相关下一状态：

```
S3 Web — Base Resume Preview
```

Node：

```
299:5
```

以及：

```
S3 Web — Tailored Resume Review
```

Node：

```
286:2
```

设计 annotation 不属于应用 UI。

------

# 7. UI Structure

页面从上到下主要包括：

### A. Existing OfferBuddy Web Shell

继续复用 S2 Web：

- Header；
- Navigation；
- Page container；
- typography；
- button/card patterns。

------

### B. Preparation / Job Context

让用户明确当前 Resume 是为哪份 Job 准备。

至少能够识别：

- Role title；
- Company；
- relevant preparation context。

不需要复制整个 Job Detail。

------

### C. Page Heading

表达：

> Create Tailored Resume

并简短说明：

> Choose a starting point for this job-specific resume.

------

### D. Existing Base Resumes

展示当前 Candidate 已保存的 Base Resumes。

例如：

```text
Backend Engineer Resume
Updated 12 Aug 2026

Full Stack Resume
Updated 03 Aug 2026
```

每张 Base Resume 至少提供：

- name；
- useful metadata；
- selection state；
- `View resume`。

如果设计中存在推荐提示，可以作为辅助决策信息，但不能替用户自动选择并隐藏其他选择。

------

### E. Build from Candidate Profile

始终提供：

> **Build from Candidate Profile**

其语义是：

> 不采用已有 Resume 的结构/措辞作为 starting point，而是直接根据 Candidate Profile 中的真实事实，为当前 Job 生成新的 Tailored Resume。

它不是：

> “让 AI 自己写一份 Resume”。

------

### F. Primary Action

选择 starting point 后：

> **Create tailored resume**

触发当前 Preparation 的 Tailored Resume generation。

------

# 8. Data Dependencies

### Reads — Candidate Profile

至少包括：

- candidateId；
- profileVersion；
- capability-relevant facts。

Candidate Profile 是事实边界。

------

### Reads — Base Resumes

每份 Base Resume 至少需要：

- resumeId；
- display name；
- relevant metadata；
- ownership；
- lifecycle/status；
- preview availability；
- provenance where applicable。

Base Resume 必须属于当前 Candidate。

------

### Reads — Job

- jobId；
- contentVersion；
- current canonical Job content。

------

### Reads — Preparation

- preparationId；
- candidateId；
- jobId；
- current preparation state。

------

### Reads — Match

可用于帮助 Tailored Resume 理解：

- strengths；
- gaps；
- relevant requirements；
- evidence alignment。

但 Match 仍然是 derived artifact，而不是 Candidate fact source。

------

### Writes / Commands

生成 Tailored Resume。

Command 概念上需要表达：

```text
Preparation
+
Candidate Profile version
+
Job content version
+
selected starting point
```

其中 starting point 可以是：

```text
Base Resume
```

或者：

```text
Candidate Profile
```

------

# 9. Field Specification

本页面没有传统 CRUD form，但有一个重要 selection contract：

| Field          | Type               | Required    | Editable | Behaviour                           |
| -------------- | ------------------ | ----------- | -------- | ----------------------------------- |
| Starting point | Selection          | Yes         | Yes      | Base Resume or Candidate Profile    |
| Base Resume    | Resource selection | Conditional | Yes      | Must belong to Candidate            |
| Job            | Context            | Yes         | No       | Current Preparation Job             |
| Candidate      | Context            | Yes         | No       | Derived from authorised Preparation |

用户必须明确选择一个 starting point。

------

# 10. Actions

## View Resume

**Trigger**

用户想先查看某个 Base Resume。

**Behaviour**

进入：

> ```
> S3-UI-03 — Base Resume Preview
> ```

必须保留当前 Create Tailored Resume context。

例如：

```text
Create Tailored Resume
        ↓
View Backend Resume
        ↓
Base Resume Preview
        ↓
Use this resume
        ↓
Create Tailored Resume
```

或者：

```text
Preview
↓
Back
↓
Selection screen
```

不要让 Preview 变成 workflow dead end。

------

## Select Base Resume

设置当前 starting point。

这只是前端选择，不应立即触发 AI generation。

------

## Build from Candidate Profile

选择 Candidate Profile 作为 starting point。

不创建新的 Profile。

不改变现有 Base Resumes。

------

## Create Tailored Resume

**Trigger**

用户已经选择 starting point。

**Behaviour**

向 frozen Resume/Preparation API 发起生成 command。

AI operation 可能异步执行。

### Success

进入：

> Tailored Resume Review

如果 API 返回 accepted/processing：

进入 generating state，并观察 business resource。

### Failure

保持 Preparation context。

允许安全 retry。

不能创建多个不可解释的 duplicate Tailored Resume artifacts。

------

# 11. UI States

## Loading

正在加载：

- Preparation；
- Base Resumes；
- relevant Candidate/Job state。

------

## No Base Resumes

不能显示“你无法继续”。

应该突出：

> Build from Candidate Profile

用户仍然可以完成 Focused Apply。

------

## Base Resumes Available

展示 selection list。

------

## Selected

明确显示当前 starting point。

------

## Previewing

由 `S3-UI-03` 负责。

------

## Generating

Tailored Resume 正在生成。

必须：

- disable duplicate submit；
- 显示明确 processing state；
- 保持当前 Preparation；
- 不伪造百分比。

------

## Failed

安全展示 generation failure。

不要暴露 raw provider response。

------

## Input Stale

例如：

```text
User opens Create Tailored Resume
profileVersion = 12

Before generation:
Candidate Profile becomes version 13
```

不能继续假装使用 version 12。

应按照 frozen API conflict/staleness semantics 处理。

------

# 12. Business Rules

### Rule 1 — One Candidate Profile

一个 Candidate 使用一个 authoritative Candidate Profile。

多 Resume 不代表多 Profile。

------

### Rule 2 — Multiple Base Resumes Allowed

一个 Candidate 可以保存多份 Base Resume。

这些 Resume 可以强调不同职业方向。

------

### Rule 3 — Base Resume Is Not Truth Authority

如果：

```text
Base Resume says X
Candidate Profile does not support X
```

Tailoring 不应因为 Base Resume 有这句话就把 X 当作新事实。

Candidate Profile 仍然是事实边界。

------

### Rule 4 — Base Resume Is a Starting Point

Base Resume 可以提供：

- structure；
- wording；
- ordering；
- emphasis；
- previously selected truthful content。

但不是事实授权来源。

------

### Rule 5 — Build from Profile Is Always Valid

用户不应该被迫先创建 Base Resume 才能 Focused Apply。

------

### Rule 6 — Tailored Resume Is Job-specific

Tailored Resume 属于当前 Preparation context。

不能把生成结果偷偷覆盖某份 Base Resume。

------

### Rule 7 — No Fabrication

AI 可以：

- select；
- reorder；
- rewrite；
- condense；
- emphasise。

AI 不可以：

- invent employment；
- invent skills；
- invent qualifications；
- invent achievements；
- inflate experience duration；
- create false eligibility claims。

------

### Rule 8 — Application Is Not Required

Resume preparation 发生在 Application submission 之前。

Application 不是 generation prerequisite。

------

# 13. API Mapping

必须使用 Phase 3 §3.17 已冻结的：

- Candidate/Profile contracts；
- Preparation contracts；
- Tailored Resume contracts。

Agent 不得根据 UI 自行创建 endpoint。

概念能力包括：

### List / Read Available Resume Starting Points

获取当前 Candidate 可用的 Base Resumes。

### Read Base Resume

供 Preview 使用。

### Generate Tailored Resume

为当前 Preparation 创建 Job-specific Tailored Resume。

如果工作涉及 AI：

```text
POST command
→ 202 Accepted
→ business resource processing
→ poll/read Tailored Resume resource
```

不要创建通用：

```text
/api/operations/{randomId}
```

除非 frozen contract 明确如此；S3 当前设计原则是不这么做。

------

# 14. Concurrency / Staleness

Tailored Resume provenance 至少受以下输入影响：

```text
Candidate.profileVersion
Job.contentVersion
selected Base Resume provenance/version where applicable
```

例如：

```text
Generation requested with:

profileVersion = 12
jobContentVersion = 5
baseResume = backend-resume
```

如果 command 到达时 authoritative inputs 已变化：

> 必须遵循 frozen conflict semantics。

不能 silently merge。

------

# 15. Navigation / Transitions

| From                   | Action             | Destination                     |
| ---------------------- | ------------------ | ------------------------------- |
| Match Analysis         | Continue           | Create Tailored Resume          |
| Create Resume          | View Base Resume   | Base Resume Preview             |
| Preview                | Back               | Create Tailored Resume          |
| Preview                | Use this Resume    | Create Resume / selected        |
| Create Resume          | Build from Profile | Profile starting point selected |
| Create Resume          | Generate           | Generating                      |
| Generating             | Ready              | Tailored Resume Review          |
| Generating             | Failure            | Failure state                   |
| Tailored Resume exists | Open Resume        | Tailored Resume Review          |

------

# 16. S2 Reuse

### Reuse

- Web shell；
- navigation；
- buttons；
- cards；
- loading/error patterns；
- existing Resume-related infrastructure where suitable。

### S3 Delta

Introduce:

- Base Resume selection；
- Candidate Profile starting point；
- Preparation-specific Tailored Resume generation。

### Do Not

- make Resume mandatory for Fast Apply；
- modify Application workflow；
- replace Candidate Profile with Resume parsing；
- create another generic Resume builder unrelated to Job Preparation。

------

# 17. Component Reuse

Agent 必须先检查现有 React implementation。

可能需要的 S3-specific concepts：

```text
ResumeStartingPointSelector
BaseResumeCard
ResumeSelectionState
TailoredResumeGenerationState
```

名称不是 contract。

更重要的是 component responsibility。

不要为每个 Resume card state 创建完全不同组件。

------

# 18. Responsive / Surface Behaviour

Frozen Figma 主要针对 desktop Web。

Resume cards 在较窄 desktop width 下可以按照现有 OfferBuddy responsive conventions wrap/stack。

不要在 S3 自动设计 mobile resume editor。

------

# 19. Accessibility / UX Requirements

Selection 必须不仅靠 border colour 表达。

Base Resume card 应支持明确的：

- selected state；
- focus state；
- keyboard selection where existing component conventions support it。

Generate button 在没有 starting point 时：

- disabled；
- 或通过明确 validation 阻止提交。

Generation 中不能允许重复点击。

------

# 20. Analytics / Observability

不要新增无批准的产品 analytics。

生成 Tailored Resume 的 operational tracing 应遵循 Phase 3 Observability：

```text
HTTP request
→ requestId
→ correlationId
→ business_event
→ worker
→ AI
```

Frontend 不记录：

- Resume body；
- Candidate Profile body；
- Job description；
- AI prompt；
- raw AI response。

------

# 21. Security / Privacy

所有 Candidate/Base Resume/Preparation access 必须 server-side authorised。

不能通过：

```text
candidateId from browser
```

来证明 ownership。

Extension/Web client 均为 untrusted client。

Base Resume preview 不得允许读取其他 Candidate 的 Resume。

------

# 22. Acceptance Criteria

-  User can enter Create Tailored Resume from Focused Preparation.
-  Existing Base Resumes are available as starting points.
-  User can preview a Base Resume before selecting it.
-  Preview preserves the Focused Preparation context.
-  User can return from Preview without losing workflow state.
-  User can choose a Base Resume.
-  User can choose Build from Candidate Profile.
-  User can continue without having any Base Resume.
-  Only one starting point is used for one generation command.
-  Starting-point selection does not immediately invoke AI.
-  Tailored Resume generation belongs to the current Preparation.
-  Application is not required.
-  Candidate Profile remains the factual source of truth.
-  Base Resume cannot establish new Candidate facts.
-  Tailored Resume does not overwrite Base Resume.
-  Async generation is represented correctly.
-  Duplicate generation is prevented while processing.
-  Generation failure can be safely recovered.
-  Stale Candidate/Job input is not silently ignored.
-  Fast Apply remains unaffected.

------

# 23. Agent Implementation Notes

Before writing code:

1. Inspect existing Resume-related S2 code first.
2. Read frozen Candidate/Profile, Preparation and Resume API contracts.
3. Use Figma node `298:2` for this surface.
4. Use `299:5` together with `S3-UI-03` for preview behaviour.
5. Do not model Base Resume as Candidate Profile.
6. Do not create one Profile per Resume.
7. Do not require Base Resume.
8. Do not allow Base Resume content to bypass Candidate fact boundaries.
9. Do not create Application just because Resume generation begins.
10. Do not overwrite Base Resume after tailoring.
11. Preserve provenance/version information supplied by backend.
12. Do not invent generic AI task APIs.
13. Reuse existing Web shell/components.
14. Do not redesign the frozen Figma.

```
## 一个值得现在固定的工程语义

这份 spec 把之前 UI 讨论中的一个重要问题正式落下来了：

**Candidate Profile、Base Resume、Tailored Resume 是三个不同层级。**

​```text
Candidate Profile
   authoritative facts
          │
          ├───────────────┐
          ▼               ▼
   Backend Resume    Full Stack Resume
      base expression / starting points
          │
          └──────┬────────┘
                 ▼
          Current Job
                 │
                 ▼
         Tailored Resume
          derived artifact
```

所以后面 Agent 即使看到“多简历”，也不能错误实现成“多 Profile”。

下一份进入 **`S3-UI-03 — Base Resume Preview`**。它会比前两份短很多，重点冻结 **View → Back / Use this resume → 保留 Preparation context → read-only preview**。