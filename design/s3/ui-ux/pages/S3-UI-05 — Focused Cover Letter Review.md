# S3-UI-05 — Focused Cover Letter Review

# S3-UI-05 — Focused Cover Letter Review

```yaml
spec_id: S3-UI-05
title: Focused Cover Letter Review
surface: web
status: frozen
figma_page: S3-01 Focused Apply
depends_on:
  - candidate-profile
  - job
  - job-intelligence
  - preparation
  - match
  - tailored-resume
  - cover-letter
s2_reuse: true
```

## 1. Purpose

`Focused Cover Letter Review` 是 Focused Apply Web workflow 的最后一个主要内容准备页面。

它回答：

> **For this specific job, is the job-specific cover letter ready to use?**

与 Fast Apply 的 Default Cover Letter 完全不同。

Focused Cover Letter 基于：

```text
Candidate Profile
       +
Current Job
       +
Job Intelligence
       +
Match Analysis
       +
relevant Preparation context
       ↓
Focused Cover Letter
```

用户可以：

- 查看生成结果；
- 做必要的人工修改；
- 保存；
- 在输入发生变化时识别 stale；
- 合理地 regenerate；
- 完成 Preparation 后返回招聘网站继续真实投递。

核心原则：

> **OfferBuddy prepares the application; the recruitment website remains the place where the application is actually submitted.**

------

# 2. Scope

### In Scope

- 展示当前 Job context；
- 展示 job-specific Cover Letter；
- Review generated content；
- 有限内容编辑；
- Save；
- Regenerate；
- processing / failure / stale / conflict 状态；
- 返回招聘网站继续申请；
- 将 Preparation 表达为 ready；
- 供 Extension 后续 SEEK Cover Letter assistance 使用已经准备好的 Cover Letter。

### Out of Scope

- Auto Apply；
- OfferBuddy Web 代替 SEEK / Indeed / LinkedIn 提交申请；
- Application 自动创建为 `APPLIED`；
- 修改 Candidate Profile；
- 修改 Base Resume；
- 修改 Tailored Resume；
- 修改 Job；
- 将当前 Focused Cover Letter 覆盖成 Default Cover Letter；
- 将 Default Cover Letter 当成 Focused Cover Letter；
- Full rich-text document designer；
- Extension 内完整 Cover Letter editor。

------

# 3. Entry Points

主要路径：

```text
Match Analysis
      ↓
Create Tailored Resume
      ↓
Tailored Resume Review
      ↓
Continue to Cover Letter
      ↓
Focused Cover Letter Review
```

已有 Preparation 也可以重新打开：

```text
Existing Preparation
      ↓
Cover Letter exists
      ↓
Open Cover Letter
      ↓
Focused Cover Letter Review
```

如果当前 Job 不需要 Cover Letter，用户不应该因为 S3 workflow 而被强迫生成一份。

因此 Cover Letter 是 Focused Preparation capability，但不是所有外部 Application 的强制 prerequisite。

------

# 4. Preconditions

必须存在：

- authenticated user；
- authorised Candidate；
- canonical Job；
- Candidate + Job Preparation。

通常存在：

- Match；
- Tailored Resume。

但实现不能把“Tailored Resume 必须成功生成”错误建模为 Cover Letter 的绝对领域 prerequisite，除非 frozen API contract 明确定义如此。

Focused Cover Letter resource 可以处于：

```text
PROCESSING
READY
FAILED
STALE
```

Application 仍然可以尚未提交。

------

# 5. Route / Surface

**Surface:** Web application / Focused Preparation.

它属于：

```text
Preparation
   ├── Match
   ├── Tailored Resume
   └── Focused Cover Letter  ← current
```

不是：

```text
Application
   └── Cover Letter
```

虽然 Application 提交完成后可以通过 Application Detail 回看 Preparation，但 artifact 的生成生命周期仍然属于 Preparation。

------

# 6. Figma Reference

**Figma Page**

```
S3-01 Focused Apply
```

**Implementation surface**

最终 `Focused Cover Letter Review` frame。

Agent 实现前必须以 frozen Figma 中该 final frame 为视觉 source of truth，并确认实际 Node ID，而不是使用早期 exploratory Cover Letter frame。

所有：

```text
ANNOTATION — ...
```

均为设计说明，不属于应用 UI。

------

# 7. UI Structure

页面从上到下：

### A. Existing OfferBuddy Web Shell

复用：

- Header；
- Navigation；
- Page container；
- typography；
- buttons；
- loading/error components。

------

### B. Job / Preparation Context

明确当前 Cover Letter 对应：

- Role title；
- Company；
- current Preparation。

用户必须能一眼确认：

> 这封信是给哪家公司、哪个职位的？

不需要复制完整 Job Detail。

------

### C. Cover Letter Identity / Status

明确这是：

> **Focused Cover Letter**

而不是 Default Cover Letter。

可以表达：

- Generated for current Job；
- Ready / stale / processing 状态；
- relevant generation context。

------

### D. Cover Letter Content

展示完整 job-specific Cover Letter。

内容应基于真实 Candidate facts，同时针对当前 Job：

- 调整 opening；
- 选择 relevant experience；
- 强调 relevant skills；
- 联系 Job requirements；
- 使用正确 Role / Company context。

------

### E. Editing

允许用户对当前 artifact 做有限人工修正：

- wording；
- paragraph；
- tone；
- remove unnecessary content；
- correct presentation。

这里不是 Full document designer。

------

### F. Actions

核心 actions：

- `Save`
- `Regenerate`
- `Return to job` / equivalent completion CTA

具体 button copy 以 frozen Figma 为准。

------

# 8. Data Dependencies

### Candidate Profile

读取：

- candidateId；
- current profileVersion；
- capability-relevant truthful facts。

------

### Job

读取：

- jobId；
- current contentVersion；
- Role title；
- Company；
- relevant canonical content。

------

### Job Intelligence / Match

可提供：

- important requirements；
- relevant strengths；
- gaps/attention areas；
- evidence alignment。

它们用于**选择和强调** Candidate facts。

不能建立新事实。

------

### Tailored Resume

可以作为当前 Preparation 的相关 artifact context。

但 Cover Letter 不能因为 Tailored Resume 中存在 unsupported statement，就把它升级成事实。

最终 truth boundary 仍然是 Candidate Profile。

------

### Focused Cover Letter

读取：

- coverLetterId；
- preparationId；
- content；
- status；
- provenance；
- source profileVersion；
- source job contentVersion；
- artifact version/OCC token；
- generation/failure state。

### Writes

只修改：

> 当前 Focused Cover Letter artifact。

不修改：

- Candidate Profile；
- Default Cover Letter；
- Base Resume；
- Tailored Resume；
- Job；
- Application status。

------

# 9. Field Specification

| Content                      | Editable   | Rule                                |
| ---------------------------- | ---------- | ----------------------------------- |
| Greeting                     | Yes        | User may adjust                     |
| Opening paragraph            | Yes        | Job-specific                        |
| Candidate experience wording | Yes        | Must remain truthful                |
| Skills emphasis              | Yes        | Must be supported                   |
| Job/company references       | Yes        | Must reflect current Job            |
| Closing                      | Yes        | User may adjust                     |
| Candidate factual claims     | Limited    | Must remain supported by Profile    |
| Contact/personal facts       | Restricted | Candidate Profile remains authority |

`Editable` 不等于可以绕过 factual boundary。

------

# 10. Actions

## Save

保存用户对当前 Focused Cover Letter 的修改。

### Success

- artifact 更新；
- dirty state 清除；
- remain on page。

### Failure

必须保留本地修改。

### Conflict

发生 OCC conflict：

> 明确提示，不 silent overwrite。

------

## Regenerate

用于：

- 用户明确要求重新生成；
- artifact stale；
- previous generation failed / unsuitable；
- authoritative inputs 已变化。

重新生成使用当前 authoritative inputs。

不得简单把按钮实现成：

> “再抽一次 AI 文案”。

Processing 时禁止重复触发。

------

## Return to Job

这是整个 Focused Preparation workflow 非常关键的动作。

语义：

> Preparation 已经准备好，现在回招聘网站完成真实投递。

概念流程：

```text
Focused Cover Letter ready
          ↓
Return to job
          ↓
SEEK / Indeed / LinkedIn
          ↓
User completes external application
```

`Return to Job` **不等于**：

```text
mark Application = APPLIED
```

真正 Applied 状态必须来自真实投递完成后的相应 S2/Extension integration。

------

## SEEK Prepared Cover Letter Fill

这不是本 Web 页面上的直接 action，但当前 artifact 可以被后续 Extension capability 使用：

```text
Focused Cover Letter ready
        ↓
Return to SEEK
        ↓
SEEK Quick Apply asks for Cover Letter
        ↓
Extension detects prepared CL
        ↓
Fill prepared cover letter
```

此时：

> **Prepared Focused Cover Letter 优先于 Default Cover Letter。**

具体 Extension 行为由 `S3-UI-16` 定义。

------

# 11. UI States

### Loading

读取当前 artifact。

### Processing

Cover Letter generation 进行中。

不得使用虚假百分比。

### Ready / Clean

生成完成且无未保存修改。

### Editing / Dirty

用户已经修改。

离开时不得静默丢失内容。

### Saving

禁止重复 Save。

### Save Failure

保留用户编辑内容。

### Generation Failure

提供安全 retry。

不得显示 raw provider error。

### Stale

例如：

```text
Generated:
profileVersion = 18
jobContentVersion = 7

Current:
profileVersion = 19
jobContentVersion = 7

→ Focused Cover Letter stale
```

必须明确展示。

### Conflict

artifact write OCC conflict。

不能自动覆盖。

------

# 12. Business Rules

1. Focused Cover Letter 是 **Job-specific derived artifact**。
2. 它属于 Candidate + Job Preparation。
3. 它不是 Default Cover Letter。
4. Default Cover Letter 服务 Fast Apply / SEEK quick filling。
5. Focused Cover Letter 服务精投。
6. Candidate Profile 是 Candidate fact authority。
7. Match、Resume、Job Intelligence 可以帮助选择强调内容，但不能创建 Candidate facts。
8. 用户编辑 Focused Cover Letter 不自动修改 Candidate Profile。
9. 用户编辑 Focused Cover Letter 不自动修改 Default Cover Letter。
10. Focused Cover Letter 不覆盖 Base Resume 或 Tailored Resume。
11. AI 不得发明工作经历、技能、学历、证书、资格、签证/工作权利等事实。
12. Application submission 仍发生在招聘网站。
13. `Return to Job` 不自动把 Application 标记为 Applied。
14. Focused Preparation 不破坏 S2 Fast Path。
15. Cover Letter 不应该成为 Indeed/LinkedIn 等不要求 Cover Letter 流程的额外障碍。
16. SEEK 要求 Cover Letter 时，已有 fresh Focused Cover Letter 优先于 Default Template。
17. Artifact 必须保留 provenance。
18. Candidate `profileVersion` / Job `contentVersion` semantic changes 可以造成 stale。

------

# 13. API Mapping

必须使用 Phase 3 §3.17 frozen Cover Letter / Preparation contracts。

概念能力包括：

### Read Focused Cover Letter

读取当前 Preparation 的 Cover Letter resource/state。

### Generate / Regenerate

请求当前 Preparation 的 job-specific Cover Letter generation。

AI work 可能：

```text
POST command
    ↓
202 Accepted
    ↓
business resource PROCESSING
    ↓
read/poll Cover Letter
    ↓
READY
```

不要创建 generic frontend AI operation abstraction。

### Update Cover Letter

保存用户修改。

必须遵循 frozen concurrency semantics。

Agent 不得根据 editor 自行创建新的 API contract。

------

# 14. Concurrency / Staleness

与 Tailored Resume 一样，需要区分：

### Artifact OCC

解决：

> 当前 Cover Letter 被并发修改怎么办？

例如：

```text
artifact version = 3
A saves → 4
B tries saving version 3
→ 409
```

不能 silent merge。

### Provenance Staleness

解决：

> 这封 Cover Letter 是否仍基于当前 Candidate / Job？

例如：

```text
sourceProfileVersion = 18
currentProfileVersion = 19

→ stale
```

Stale 不等于自动删除。

用户可以看到旧 artifact，但必须明确知道它不是最新输入生成的。

------

# 15. Navigation / Transitions

| From                   | Action        | Destination / State  |
| ---------------------- | ------------- | -------------------- |
| Tailored Resume Review | Continue      | Focused Cover Letter |
| Existing Preparation   | Open CL       | Focused Cover Letter |
| No CL yet              | Generate      | Processing           |
| Processing             | Success       | Ready                |
| Processing             | Failure       | Failed               |
| Review                 | Edit          | Dirty                |
| Dirty                  | Save          | Saving               |
| Saving                 | Success       | Ready                |
| Saving                 | Conflict      | Conflict             |
| Stale                  | Regenerate    | Processing           |
| Ready                  | Return to Job | Recruitment website  |
| SEEK after return      | CL required   | Extension assistance |

------

# 16. S2 Reuse

### Reuse

- Web shell；
- button patterns；
- editor/form conventions；
- loading/error state；
- unsaved changes handling；
- external Job/Application context；
- existing Extension/application completion integration where applicable。

### S3 Delta

增加：

- job-specific Focused Cover Letter；
- review/edit；
- provenance/staleness；
- prepared CL handoff to SEEK Extension。

### Do Not

- replace S2 Fast Apply；
- require Focused CL for every Application；
- rewrite Application submission；
- create Auto Apply；
- duplicate editor inside Extension。

------

# 17. Component Reuse

优先检查现有 React components。

可能需要的 responsibilities：

```text
FocusedCoverLetterReview
CoverLetterEditor
ArtifactStatusNotice
PreparationActionBar
```

名称仅示意。

尤其不要分别创建：

```text
DefaultCoverLetterEditor
FocusedCoverLetterEditor
RandomOtherCoverLetterEditor
```

如果 underlying text editing behaviour 可以复用，应共享基础 editor，但保持业务 container/state 分离。

------

# 18. Responsive / Surface Behaviour

主要设计目标为 desktop Web。

编辑区域在较窄 viewport 下应保持可读。

不要求：

- mobile document editor；
- Extension full editor。

Extension 只执行其 narrow assistance capability。

------

# 19. Accessibility / UX Requirements

- Editor 可 keyboard 操作；
- Save / Regenerate / Return to Job hierarchy 清晰；
- stale/failure 不只依赖颜色；
- generation 中按钮正确 disabled；
- Save failure 保留内容；
- unsaved changes 不静默丢失；
- 返回招聘网站的 CTA 必须明确表达“继续申请”，避免让用户误以为 OfferBuddy 已经投递完成。

最后这一点尤其重要。

------

# 20. Analytics / Observability

不得因 Page Spec 随意增加 product analytics。

Operational tracing 可关联：

- requestId；
- preparationId；
- coverLetterId；
- correlationId（internal）；
- AI capability/provider operational metadata。

不得记录：

- Cover Letter body；
- Candidate facts；
- raw Job description；
- AI prompt；
- raw provider response。

------

# 21. Security / Privacy

Focused Cover Letter 属于 Candidate-owned data。

授权：

```text
backend security context
        ↓
Candidate ownership
        ↓
Preparation
        ↓
Cover Letter
```

不能依赖 client 提供的 candidateId 证明权限。

Extension 读取 prepared Cover Letter 时同样必须经过 narrow authorised contract。

Admin 不获得通用读取用户 Cover Letter 正文的能力。

------

# 22. Acceptance Criteria

-  User can enter Focused Cover Letter from current Preparation.
-  Current Role and Company are identifiable.
-  UI clearly distinguishes Focused Cover Letter from Default Cover Letter.
-  Generated content is job-specific.
-  Generated Candidate claims remain grounded in Candidate Profile.
-  User can review the full Cover Letter.
-  User can make permitted edits.
-  Editing affects only the current Focused Cover Letter.
-  Editing does not modify Candidate Profile.
-  Editing does not modify Default Cover Letter.
-  User can save edits.
-  Save failure preserves local content.
-  Concurrent stale write is not silently overwritten.
-  User can regenerate when appropriate.
-  Duplicate generation is prevented during processing.
-  Generation failure can be safely retried.
-  Candidate Profile revision can make Cover Letter stale.
-  Job semantic revision can make Cover Letter stale.
-  Stale state is visible.
-  User can return to the recruitment website.
-  Returning to the Job does not automatically mark the Application as Applied.
-  External application remains the actual submission point.
-  SEEK Extension can later use the prepared Cover Letter.
-  Prepared Focused Cover Letter takes priority over Default Cover Letter for the same SEEK Job.
-  Fast Apply remains available without Focused Cover Letter.
-  Extension does not become a full Cover Letter editor.
-  No raw AI/provider-sensitive data is exposed.

------

# 23. Agent Implementation Notes

Before implementation:

1. Read frozen Preparation/Cover Letter API contracts.
2. Inspect existing S2 Web editor/form patterns.
3. Inspect Extension/application completion integration before implementing return/handoff behaviour.
4. Use the final frozen Focused Cover Letter Figma frame; confirm its current Node ID before coding.
5. Keep Focused Cover Letter and Default Cover Letter as different business concepts.
6. Treat Candidate Profile as fact authority.
7. Do not update Candidate Profile from Cover Letter edits.
8. Do not update Default Cover Letter from Focused Cover Letter edits.
9. Do not create Application merely because Cover Letter is ready.
10. Do not mark Application `APPLIED` when `Return to Job` is clicked.
11. Do not implement Auto Apply.
12. Preserve artifact OCC and provenance staleness as separate concerns.
13. Use business-resource async state.
14. Do not invent generic AI operation APIs.
15. Reuse existing text-editor primitives where practical.
16. Keep the Extension implementation narrow.
17. Ignore Figma annotations.
18. Do not redesign the frozen UI.

```
## Focused Apply Web specifications complete

这样 `S3-UI-01` ～ `S3-UI-05` 已经形成完整闭环：

​```text
S3-UI-01
Match Analysis
      ↓
S3-UI-02
Choose Base Resume / Create Tailored Resume
      ↕
S3-UI-03
Base Resume Preview
      ↓
S3-UI-04
Tailored Resume Review
      ↓
S3-UI-05
Focused Cover Letter Review
      ↓
Return to recruitment website
      ↓
External submission
      ↓
Application status update
```

这里最后一个边界尤其值得冻结：**“Preparation ready” 和 “Application applied” 是两个不同事件。**

OfferBuddy 可以帮用户把 Match、Resume、Cover Letter 全部准备好，但只有外部招聘网站真正完成投递后，才进入 Application 的 applied lifecycle。

下一组就进入 **Candidate Profile specifications**，从 `S3-UI-06 — Candidate Profile Overview` 开始。