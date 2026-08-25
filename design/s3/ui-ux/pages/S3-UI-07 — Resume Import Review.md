继续。`Resume Import Review` 这一份需要比普通 Profile CRUD 更严格，因为它是 **外部 Resume 内容 → AI → authoritative Candidate Profile** 之间的事实确认边界。

# S3-UI-07 — Resume Import Review

```yaml
spec_id: S3-UI-07
title: Resume Import Review
surface: web
status: frozen
figma_page: S3-02 Candidate Profile
depends_on:
  - candidate-profile
  - resume-import
  - ai
s2_reuse: true
```

## 1. Purpose

`Resume Import Review` 让用户使用已有 Resume 快速建立或补充 Candidate Profile，同时保证 AI extraction **不能绕过用户确认直接建立 Candidate facts**。

核心问题：

> **OfferBuddy extracted these facts from your resume. Which ones should become part of your Candidate Profile?**

完整语义是：

```text
Resume
   ↓
Extract
   ↓
Proposed Candidate Facts
   ↓
User Review
   ↓
Accept / Edit / Reject
   ↓
Candidate Profile
```

而绝不能是：

```text
Resume
   ↓
AI
   ↓
Candidate Profile automatically updated
```

------

## 2. Scope

### In Scope

- 展示 Resume Import extraction result；
- 按 Candidate Profile section 展示 proposed facts；
- 区分现有 Profile facts 与新 proposal；
- 允许用户 review；
- 允许用户修正 extracted proposal；
- 允许用户选择接受/拒绝；
- 将用户最终确认的 facts 写入 Candidate Profile；
- 处理 extraction processing / failure；
- 处理 Profile OCC conflict；
- 成功后返回 Candidate Profile。

### Out of Scope

- 把 Resume 本身当成 Candidate Profile；
- 无确认自动写入；
- AI 自动覆盖现有 Candidate facts；
- 创建第二个 Candidate Profile；
- Base Resume management；
- Tailored Resume generation；
- Match；
- Cover Letter；
- Job-specific Resume optimisation；
- Resume layout/design editing；
- Application lifecycle。

------

## 3. Entry Points

主要入口：

```text
Candidate Profile — First Time
        ↓
Import Resume
        ↓
Upload / select Resume
        ↓
Extraction
        ↓
Resume Import Review
```

已有 Profile 用户也可以：

```text
Candidate Profile
        ↓
Import Resume
        ↓
Extraction
        ↓
Resume Import Review
```

后者尤其需要处理：

> **existing facts vs proposed facts**

而不能默认覆盖。

------

## 4. Preconditions

必须存在：

- authenticated user；
- authorised Candidate context；
- successfully supplied Resume；
- Resume Import business resource。

Import resource 可能处于：

```text
PROCESSING
READY_FOR_REVIEW
FAILED
ACCEPTED
```

具体状态名称以 frozen backend contract 为准，本 spec 不重新定义 domain enum。

对于 first-time user，可以在 import workflow 中建立必要的 Candidate aggregate identity，但：

> **Candidate aggregate exists ≠ extracted facts accepted.**

------

## 5. Route / Surface

**Surface:** OfferBuddy Web / Candidate Profile workflow.

它是 Candidate Profile onboarding/maintenance capability，不属于：

- Focused Preparation；
- Application；
- Extension；
- Admin。

------

## 6. Figma Reference

**Figma Page**

```
S3-02 Candidate Profile
```

**Final implementation frame**

```
Resume Import Review
```

实现前 Agent 应确认 frozen Figma 中当前 final Node ID。

Figma 中：

```text
ANNOTATION — ...
```

只用于解释 import/review semantics，不渲染到应用。

------

# 7. UI Structure

页面主要分为六部分。

### A. Existing Web Shell

复用 OfferBuddy：

- header；
- navigation；
- page container；
- buttons；
- form primitives；
- loading/error patterns。

------

### B. Import Context

明确告诉用户：

> Resume imported. Review the information before adding it to your profile.

同时显示足够识别 source Resume 的信息，例如：

- file name；
- import status。

不需要展示 AI/provider 技术细节。

------

### C. Review Explanation

这里必须明确表达：

> Nothing is added to your Candidate Profile until you confirm it.

这是产品行为，不只是帮助文字。

------

### D. Proposed Facts

按照 Candidate Profile structure 展示：

```text
Personal Details

Professional Summary

Skills

Experience

Education

Certifications

Languages

Eligibility
```

但只展示：

- Resume 中实际提取出的 proposal；
- 或需要用户 review 的相关 existing/proposed comparison。

不要为了保持 section 数量而让 AI 填补 Resume 中不存在的信息。

------

### E. Existing vs Proposed

已有 Profile 时，应帮助用户理解：

```text
Current Profile
vs
Resume Proposal
```

例如：

```text
Experience

CURRENT PROFILE

Software Engineer
ABC Pty Ltd
Jan 2021 – Present


PROPOSED FROM RESUME

Senior Software Engineer
ABC Pty Ltd
Jan 2021 – Present
```

用户决定是否接受修改。

系统不能因为 Resume 看起来“更新”就自动覆盖。

------

### F. Confirmation Actions

主要动作：

> **Apply accepted changes**

次要动作：

> **Cancel**

具体 copy 以 frozen Figma 为准。

------

# 8. Review Unit

Import Review 的核心单位是：

> **proposed fact/change**

而不是：

> 整份 Resume 一键全部信任。

不同 section 可以采用适合该数据结构的 review interaction。

例如 Skills：

```text
✓ Java
✓ Spring Boot
✓ MySQL
□ Android
```

Experience：

```text
Experience proposal
        ↓
review full record
        ↓
accept / edit / reject
```

不能为了实现方便把复杂 Experience 拆成大量互相独立、可能造成半条事实的数据碎片。

------

# 9. Proposed Fact States

每个 reviewable proposal 在 UI 层至少需要表达：

```text
Proposed
Accepted
Rejected
Edited
```

这些是 review interaction semantics，不要求 backend 使用完全相同 enum。

### Proposed

AI extraction result，尚未获得用户认可。

### Accepted

用户确认 proposal 可以进入 Candidate Profile。

### Rejected

用户明确不希望它进入 Candidate Profile。

### Edited

用户先修改 proposal，再接受最终版本。

关键原则：

> **Edited + accepted user value，而不是原始 AI value，才是最终 Candidate fact。**

------

# 10. Data Dependencies

### Reads — Resume Import

需要：

- import resource ID；
- status；
- source metadata；
- proposed facts；
- extraction failure state；
- relevant import provenance。

### Reads — Candidate Profile

已有 Profile 时：

- candidateId；
- profileVersion；
- current Candidate facts。

### Writes — Candidate Profile

只有用户确认：

```text
Apply accepted changes
```

后，才允许 Candidate facts write。

### Import Resource

Import workflow 可以记录：

- extraction lifecycle；
- proposals；
- review status。

但不能让 unaccepted proposal 被其他 capability 当作 Candidate Profile facts。

------

# 11. Field Specification

Proposal fields最终遵循 Candidate Profile section definitions。

典型 review requirements：

| Section              | Review Behaviour                                  |
| -------------------- | ------------------------------------------------- |
| Personal Details     | Review extracted values carefully                 |
| Professional Summary | Editable before acceptance                        |
| Skills               | Accept/reject individual meaningful skills        |
| Experience           | Review coherent experience records                |
| Education            | Review qualification/institution/dates            |
| Certifications       | Review issuer/name/date where present             |
| Languages            | Review language/proficiency                       |
| Eligibility          | High caution; never infer unsupported work rights |

Eligibility 尤其严格。

AI 不能从：

> “Studied in Australia”

推断：

> “Permanent work rights”

也不能从 Resume 没写 visa status 就自行填充。

------

# 12. Actions

## Edit Proposal

用户可以修正 extraction error。

例如：

```text
AI extracted:
Spring Developer

User corrects:
Software Engineer
```

修改的是：

> proposal

还不是 Candidate Profile。

------

## Accept Proposal

标记该 proposal 将在最终确认时进入 Candidate Profile。

不要求每点一次 Accept 就立即产生一个 Profile write。

------

## Reject Proposal

proposal 不进入 Candidate Profile。

Reject 不意味着：

> Candidate 明确没有这个能力。

它只意味着：

> 不接受本次 Resume Import proposal。

------

## Apply Accepted Changes

这是整个页面真正的 Candidate Profile mutation boundary。

概念上：

```text
Reviewed proposals
       +
current profileVersion
       ↓
Candidate Profile update
       ↓
new profileVersion
```

### Success

返回 Candidate Profile Overview，并读取 authoritative Profile。

### Conflict

如果 review 期间 Profile 已变化：

```text
Review started
profileVersion = 5

Another tab changes Profile
→ version 6

Apply import using version 5
→ 409
```

绝不能自动 merge。

------

## Cancel

退出 Import Review。

不得把 proposals 写入 Candidate Profile。

------

# 13. UI States

### Processing

AI extraction 尚未完成。

显示：

> extracting / preparing review

不要 fake percentage。

### Ready for Review

展示 proposals。

### Editing Proposal

用户正在修正。

### No Extractable Facts

Resume 可读取，但没有足够可靠的 Candidate facts。

明确说明。

不要：

> AI 自己补一份 Profile。

### Extraction Failed

提供安全 retry / return。

不得显示 raw provider error。

### Applying

最终确认正在保存。

禁止重复提交。

### Apply Failure

保留用户 review decisions。

### Conflict

Candidate Profile 已发生 revision。

明确要求用户重新加载/重新确认。

不能 silent merge。

### Applied

Profile update 成功。

返回 authoritative Candidate Profile。

------

# 14. Business Rules

1. Resume Import output 是 **proposal**。
2. AI extraction result 不是 Candidate fact。
3. 用户明确接受后才建立 Candidate fact。
4. 用户可以在接受前修改 proposal。
5. 被拒绝 proposal 不进入 Candidate Profile。
6. Reject 不代表 Candidate 明确不存在该事实。
7. Existing Candidate facts 不得被 Resume Import 自动覆盖。
8. Candidate Profile 是 authoritative fact source。
9. Resume 不是 Candidate fact authority。
10. 一个 User 仍然只有一个 active Candidate Profile。
11. Import 不创建第二个 persona/Profile。
12. Partial acceptance 必须支持。
13. Import 不要求用户接受所有 extraction。
14. AI 不得填补 Resume 没有支持的信息。
15. Eligibility / Work Rights 不得根据弱信号推断。
16. Import 成功产生 meaningful Candidate fact revision，因此更新 `profileVersion`。
17. Candidate child sections不获得自己的 OCC versions。
18. Profile write 必须使用 aggregate-level optimistic concurrency。
19. Import 不自动 regenerate Match。
20. Import 不自动 regenerate Tailored Resume。
21. Import 不自动 regenerate Focused Cover Letter。
22. Existing derived artifacts可根据新的 `profileVersion` 变 stale。

------

# 15. Existing Fact Conflict Behaviour

这里必须区分两种完全不同的 conflict。

### A. Semantic Review Difference

例如：

```text
Current Profile:
Software Engineer

Resume Proposal:
Senior Software Engineer
```

这是用户 review 的内容差异。

UI 可以让用户选择：

- keep current；
- accept proposal；
- edit final value。

### B. OCC Conflict

例如：

```text
Review loaded Profile v7
        ↓
another write creates v8
        ↓
Apply using v7
        ↓
409
```

这是并发一致性问题。

不能因为 UI 已经有“Current vs Proposed”就认为可以自动 merge v8。

------

# 16. API Mapping

必须使用 Phase 3 §3.17 frozen Resume Import + Candidate Profile contracts。

概念能力：

### Start Resume Import

创建/import source 并开始 extraction。

AI work 可返回：

```text
202 Accepted
```

### Read Resume Import

读取 business resource 状态和 reviewable proposals。

### Apply Reviewed Import

把**用户明确接受后的值**应用到 Candidate Profile。

必须携带 frozen contract 所需 `profileVersion` / concurrency semantics。

不要自行设计：

```text
POST /profile/ai-autofill
```

不要让 frontend直接把 raw AI response 当 Candidate Profile update request。

------

# 17. Async Behaviour

Resume extraction 是典型 async capability：

```text
Upload
  ↓
Import resource created
  ↓
business_event
  ↓
worker
  ↓
AI extraction
  ↓
proposal resource READY
  ↓
Review
```

Frontend 观察：

> Resume Import business resource

而不是：

```text
generic-ai-task-123
```

Refresh/reopen 后仍应能够根据业务 resource 恢复状态。

------

# 18. Concurrency

Candidate Profile 使用：

```text
profileVersion
```

整个 aggregate 参与 OCC。

例：

```text
Import Review based on v10
        ↓
Skills changed elsewhere → v11
        ↓
Apply accepted import with v10
        ↓
409 Conflict
```

前端不得：

```text
fetch v11
+
blindly reapply previous import
```

用户必须重新确认，因为新的 Profile 内容可能改变 import decision。

------

# 19. Navigation / Transitions

| From              | Action            | Destination / State        |
| ----------------- | ----------------- | -------------------------- |
| Candidate Profile | Import Resume     | Import workflow            |
| Import            | Extraction starts | Processing                 |
| Processing        | Success           | Resume Import Review       |
| Processing        | Failure           | Failed                     |
| Review            | Edit proposal     | Edited                     |
| Review            | Accept            | Accepted                   |
| Review            | Reject            | Rejected                   |
| Review            | Apply             | Applying                   |
| Applying          | Success           | Candidate Profile Overview |
| Applying          | Conflict          | Conflict                   |
| Review            | Cancel            | Candidate Profile          |
| Failed            | Retry             | Processing                 |

------

# 20. S2 Reuse

### Reuse

- Web shell；
- upload components if existing；
- buttons；
- form controls；
- cards；
- loading/error patterns；
- authentication。

### S3 Delta

增加：

- Resume extraction；
- proposed-fact review；
- explicit acceptance；
- Candidate Profile integration。

### Do Not

- build another Resume editor；
- alter S2 Fast Apply；
- couple Import to Application；
- turn imported Resume into automatic Base Resume unless separately defined by Resume capability；
- bypass Profile review.

最后一点也很重要：

> **Importing a Resume into Candidate Profile and storing/managing a Base Resume are related UX concepts, but they are not automatically the same domain command.**

------

# 21. Component Reuse

可能需要的 component responsibilities：

```text
ResumeImportReview
ImportSection
ProposedFact
CurrentVsProposed
ReviewDecision
ImportApplyBar
```

具体名称不冻结。

应共享 Candidate Profile section presentation/form primitives。

尤其 Experience/Education 的 review display 应尽可能复用后续 Profile edit components，而不是维护两套完全不同的数据编辑逻辑。

------

# 22. Responsive / Surface Behaviour

主要 frozen design：

> desktop Web.

复杂 `Current vs Proposed` 在较窄 viewport 可以：

```text
Current
↓
Proposed
```

纵向排列。

不要强行保持双栏导致内容不可读。

不要求 Extension/mobile Import workflow。

------

# 23. Accessibility / UX Requirements

- Accept / Reject 不能只靠颜色表达。
- Proposal selection 必须 keyboard accessible。
- Edited state 要明确。
- Existing vs Proposed label 必须清晰。
- Applying 时禁止重复提交。
- Extraction failure 应提供 actionable recovery。
- OCC conflict 必须说明需要重新确认。
- Cancel 不得意外保存。
- 用户必须明确知道什么时候 facts 才真正进入 Profile。

------

# 24. Analytics / Observability

Operational telemetry 可以记录：

- requestId；
- correlationId；
- candidateId；
- import resource ID；
- extraction status；
- AI capability/provider metadata；
- latency/failure category。

不得记录：

- raw Resume；
- extracted personal data；
- proposed experience正文；
- Candidate Profile正文；
- raw prompt；
- raw AI response。

------

# 25. Security / Privacy

Resume Import 涉及高敏感 Candidate data。

必须：

- backend security context ownership；
- Candidate-authorised import access；
- narrow AI data boundary；
- no raw Resume logging；
- no cross-user resource access。

Extension 不参与 Resume Import。

RuoYi Admin 不应提供查看用户 Resume 原文/Import proposal 的通用后台能力。

------

# 26. Acceptance Criteria

-  User can start Resume Import from Candidate Profile.
-  Import extraction can execute asynchronously.
-  Processing state survives normal business-resource reload semantics.
-  Extracted information is clearly labelled as proposed.
-  Proposed information is not automatically added to Candidate Profile.
-  Existing Candidate facts remain visible where comparison is necessary.
-  Existing facts are not automatically overwritten.
-  User can accept proposed facts.
-  User can reject proposed facts.
-  User can edit proposed facts before acceptance.
-  Partial acceptance is supported.
-  Rejected proposal is not treated as confirmed absence.
-  Experience proposals are reviewed as coherent records.
-  Eligibility is not inferred from unsupported signals.
-  User must explicitly apply accepted changes.
-  Cancel does not modify Candidate Profile.
-  Successful application increments meaningful Candidate `profileVersion`.
-  Candidate child sections do not introduce separate OCC versions.
-  Concurrent Profile change produces explicit conflict.
-  Conflict is not silently merged.
-  Apply failure preserves review decisions where practical.
-  Successful import returns to authoritative Candidate Profile.
-  Profile update does not automatically regenerate Match/Resume/Cover Letter.
-  Resume Import does not affect S2 Fast Apply.
-  Raw Resume/AI content is not exposed through logs/Admin.

------

# 27. Agent Implementation Notes

1. Read frozen Resume Import and Candidate Profile API contracts first.
2. Inspect the final `Resume Import Review` Figma frame and confirm its Node ID.
3. Treat every AI extraction result as **proposal**, never accepted fact.
4. Do not bind raw AI response directly to Candidate Profile entities.
5. Keep explicit user confirmation as the mutation boundary.
6. Support partial acceptance.
7. Keep coherent child records such as Experience intact during review.
8. Do not infer Eligibility.
9. Use Candidate aggregate-level `profileVersion`.
10. Do not create child-section OCC versions.
11. Do not silently merge a 409.
12. Use Resume Import business-resource state for async processing.
13. Do not invent a generic AI-task API.
14. Do not automatically create/update Base Resume unless another frozen contract explicitly requests it.
15. Do not regenerate downstream artifacts after import.
16. Reuse Candidate Profile form/display components.
17. Keep Resume Import unavailable to Extension/Admin except explicitly approved operational metadata.
18. Ignore Figma annotations.
19. Do not redesign the frozen UI.

```
这份可以冻结。这里最关键的工程链条就是：

​```text
AI extracted
     ≠
Candidate fact

AI extracted
     ↓
Proposal
     ↓
User Accept / Edit
     ↓
Profile write
     ↓
profileVersion + 1
     ↓
Candidate fact
```

这也意味着 **Resume Import Review 是 S3 的 trust boundary UI**，不是普通的“AI 自动填表”。

下一步进入具体 Candidate Profile 编辑态：**`S3-UI-08 — Personal Details`**。从这里开始几份会更偏表单 contract，我们可以把字段、validation、Save/Cancel、`profileVersion` 和 Agent reuse 边界逐个固定下来。