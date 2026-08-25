S3-UI-06 — Candidate Profile Overview

继续 Candidate Profile 组。

# S3-UI-06 — Candidate Profile Overview

```yaml
spec_id: S3-UI-06
title: Candidate Profile Overview
surface: web
status: frozen
figma_page: S3-02 Candidate Profile
depends_on:
  - candidate-profile
  - resume-import
s2_reuse: true
```

## 1. Purpose

`Candidate Profile Overview` 是 Candidate Profile 的主入口，也是用户维护 **OfferBuddy authoritative candidate facts** 的中心页面。

它回答：

> **What does OfferBuddy currently know about me?**

这里不是 Resume 页面，也不是针对某个 Job 的 Preparation 页面。

Candidate Profile 为后续能力提供真实事实基础：

```text
Candidate Profile
       │
       ├── Job Match
       ├── Tailored Resume
       └── Focused Cover Letter
```

同时页面需要支持两类用户：

```text
Existing Profile
→ review / maintain facts

No Profile / First Time
→ create manually or import Resume
```

------

# 2. Scope

### In Scope

展示和管理：

- Personal Details
- Professional Summary
- Skills
- Experience
- Education
- Certifications
- Languages
- Eligibility / Work Rights

并提供：

- section-level Edit；
- first-time empty state；
- Resume Import entry；
- Profile completeness / capability guidance（如果 frozen UI 有表达）；
- Profile revision后的正确刷新。

另外，同一用户区域可以存在：

> Application Tools → Default Cover Letter

但它**不是 Candidate fact section**。

### Out of Scope

- Match result；
- Tailored Resume；
- Focused Cover Letter；
- Base Resume management；
- Job-specific optimisation；
- Application lifecycle；
- Auto Apply；
- AI 自动建立 Candidate facts；
- 多 Candidate personas / 多 Profile；
- Admin UI。

------

# 3. Entry Points

主要入口：

```text
OfferBuddy Web
    ↓
Candidate Profile
```

也可能由某个 capability 发现信息不足后引导：

```text
Focused Preparation
      ↓
Candidate information insufficient
      ↓
Candidate Profile
```

但是 Candidate Profile 不能因此变成 Focused Apply 专属页面。

它是 Candidate 自己的长期事实源。

------

# 4. Preconditions

只要求：

- authenticated user。

如果用户尚未拥有 Candidate Profile：

> 显示 First-Time state。

系统模型仍然是：

> **one active Candidate Profile per user**

Candidate ID 与 User ID 保持不同的领域 identity。

不应由前端假定：

```text
candidateId == userId
```

------

# 5. Route / Surface

**Surface:** OfferBuddy Web.

使用现有 Web application shell 和 routing conventions。

Candidate Profile 是独立用户能力，不属于：

- Application Detail；
- Preparation；
- Extension。

Coding Agent 应先检查现有 routing structure，再决定实际 React route placement，不得因为 spec 自行重构导航系统。

------

# 6. Figma Reference

**Figma Page**

```
S3-02 Candidate Profile
```

相关 final states：

- Candidate Profile main / populated state
- Candidate Profile first-time state

以及对应 section edit frames：

- Personal Details
- Professional Summary
- Skills
- Experience
- Education
- Certifications
- Languages
- Eligibility
- Default Cover Letter

具体 section behaviour由 `S3-UI-08` ～ `S3-UI-16` 进一步定义。

所有：

```text
ANNOTATION — ...
```

仅为设计文档，不属于实现 UI。

------

# 7. UI Structure

主页面的信息架构冻结为两层。

## A. Candidate Facts

```text
CANDIDATE FACTS

Personal Details
Professional Summary
Skills
Experience
Education
Certifications
Languages
Eligibility
```

这些内容共同构成 authoritative Candidate Profile。

------

## B. Application Tools

```text
APPLICATION TOOLS

Default Cover Letter
```

这一视觉/语义分组非常重要。

Default Cover Letter 虽然为了用户方便可以从 Profile 区域进入，但：

> **它不是 Candidate fact。**

不能放入 Profile provenance/fact semantics。

------

# 8. Populated Profile Structure

已有 Profile 时，每个 section 应：

- 显示有意义的摘要；
- 提供明确 `Edit`；
- 不要求用户进入编辑态才能知道当前保存了什么。

例如：

```text
Experience

Senior Software Engineer
Company A
2021 – 2024

Software Engineer
Company B
2018 – 2021

                         Edit
```

Overview 的职责是：

> **review current facts**

而不是把所有 section 同时变成巨大 form。

------

# 9. First-Time State

没有 Candidate Profile 内容时，页面必须避免给用户展示八个巨大空表单。

主要 onboarding options：

### Create Profile Manually

进入逐 section 建立 Profile 的流程。

### Import Resume

进入 Resume Import capability：

```text
Upload / import Resume
        ↓
AI extraction
        ↓
Resume Import Review
        ↓
Accept / Reject proposed facts
        ↓
Candidate Profile
```

Import **不能直接完成最后一步**。

------

# 10. Data Dependencies

### Reads

Candidate Profile aggregate：

- candidateId；
- profileVersion；
- personal/contact details；
- professional summary；
- skills；
- experiences；
- education；
- certifications；
- languages；
- eligibility。

### Related Reads

Application Tools：

- Default Cover Letter configuration。

注意：

Default Cover Letter 即使通过同一个 screen fetch，也不应在 domain/client state 中伪装成 Candidate fact。

### Writes

Overview 本身原则上不直接修改各 section facts。

Writes 发生在对应 Edit surfaces。

------

# 11. Candidate Profile Aggregate

前端必须理解 Candidate Profile 是一个 aggregate，而不是一组互不相关的 CRUD 页面。

概念模型：

```text
CandidateProfile
│
├── Personal Details
├── Professional Summary
├── Skills
├── Experiences
├── Education
├── Certifications
├── Languages
└── Eligibility

profileVersion
```

Meaningful Candidate fact revision：

```text
successful meaningful fact change
        ↓
profileVersion increments
```

Candidate child sections **没有各自独立的 domain OCC version**。

因此不要发明：

```text
skillVersion
experienceVersion
educationVersion
```

------

# 12. Actions

## Edit Section

例如：

```text
Skills
  ↓
Edit
  ↓
S3-UI-10 Skills
```

编辑成功：

```text
Save
 ↓
Candidate Profile updated
 ↓
profileVersion changed
 ↓
return / refresh Overview
```

------

## Import Resume

进入：

> ```
> S3-UI-07 — Resume Import Review
> ```

实际 import/extraction workflow 可以包含 upload/processing state，但任何 AI extracted facts 必须经过 Review。

------

## Application Tool — Default Cover Letter

进入：

> ```
> S3-UI-16 — SEEK Cover Letter Assistance
> ```

中的 Default Cover Letter configuration surface。

不要把它路由成 Candidate Profile fact editor。

------

# 13. UI States

## Loading

Candidate Profile aggregate 正在加载。

使用现有 Web loading pattern。

------

## First Time

没有 meaningful Candidate Profile content。

展示 onboarding。

------

## Populated

展示 Candidate Facts sections。

------

## Partially Complete

允许存在。

Candidate Profile **不要求全局 100% complete**。

例如用户可以只有：

```text
Personal Details
Skills
Experience
```

仍然保存 Profile。

具体 AI capability 是否有足够输入，由 capability-specific sufficiency 决定。

------

## Read Failure

提供安全 retry。

------

## Section Updated

保存 section 后重新读取 authoritative aggregate/version。

不要依赖旧 local object 猜测新的 `profileVersion`。

------

# 14. Business Rules

1. 一个 User 对应一个 active Candidate Profile。
2. `candidateId` 与 `userId` 是不同 identity。
3. Candidate Profile 是 Candidate facts 的 authoritative source。
4. Candidate Profile 可以 partially complete。
5. 不存在全局“Profile 不完整所以 OfferBuddy 不能用”的规则。
6. 每个 capability 可以有自己的 minimum sufficiency。
7. Resume Import 只能 **propose facts**。
8. 只有用户明确接受后，imported facts 才能进入 Candidate Profile。
9. Match 不修改 Candidate Profile。
10. Tailored Resume 不修改 Candidate Profile。
11. Focused Cover Letter 不修改 Candidate Profile。
12. Base Resume 不成为事实源。
13. `profileVersion` 代表 meaningful Candidate fact revision。
14. Candidate child sections 不获得独立 OCC version。
15. Candidate fact 更新可能使现有 derived artifacts stale。
16. Default Cover Letter 不是 Candidate fact。
17. Application lifecycle 与 Candidate Profile 独立。
18. S2 Fast Path 不要求 Candidate Profile。

------

# 15. API Mapping

使用 Phase 3 §3.17 frozen Candidate Profile contracts。

概念能力包括：

### Read Candidate Profile

读取当前 authenticated user's authorised Candidate Profile。

### Update Candidate Profile Sections

使用 frozen Candidate Profile update semantics。

所有 meaningful writes 必须遵守：

```text
profileVersion
```

OCC contract。

### Resume Import

使用 frozen Resume Import contracts。

不要因为 UI onboarding 自行创建：

```text
POST /api/profile/auto-fill
```

这类绕过 Review 的 endpoint。

------

# 16. Concurrency / Staleness

Candidate Profile 的关键 OCC token：

```text
profileVersion
```

例如：

```text
Browser reads profileVersion = 8

Tab A updates Skills
→ profileVersion = 9

Tab B tries saving Experience
with expected profileVersion = 8

→ conflict
```

即使 Tab B 修改的是不同 child section，也**不能因为字段不同就自动 merge**。

Frozen principle：

> stale writes fail explicitly and are never silently merged.

UI 必须让用户重新加载当前 Profile，然后重新确认修改。

------

# 17. Derived Artifact Impact

Candidate Profile 修改后可能影响：

```text
Match
Tailored Resume
Focused Cover Letter
```

例如：

```text
Profile v8
  ↓
Match generated from v8

Profile updated → v9
  ↓
existing Match provenance = v8
  ↓
Match stale
```

Overview 不需要主动重新生成所有 artifacts。

尤其不要：

```text
Profile Save
→ automatically call Match AI
→ automatically regenerate Resume
→ automatically regenerate CL
```

这会造成成本、延迟和不可控副作用。

Derived surfaces 自己处理 staleness。

------

# 18. Navigation / Transitions

| From              | Action                | Destination              |
| ----------------- | --------------------- | ------------------------ |
| Main navigation   | Candidate Profile     | Overview                 |
| First Time        | Create manually       | Relevant Profile editing |
| First Time        | Import Resume         | Resume Import            |
| Overview          | Edit Personal Details | S3-UI-08                 |
| Overview          | Edit Summary          | S3-UI-09                 |
| Overview          | Edit Skills           | S3-UI-10                 |
| Overview          | Edit Experience       | S3-UI-11                 |
| Overview          | Edit Education        | S3-UI-12                 |
| Overview          | Edit Certifications   | S3-UI-13                 |
| Overview          | Edit Languages        | S3-UI-14                 |
| Overview          | Edit Eligibility      | S3-UI-14                 |
| Application Tools | Default Cover Letter  | S3-UI-16                 |

------

# 19. S2 Reuse

### Reuse

- existing Web shell；
- authenticated user context；
- page layout；
- buttons；
- cards；
- form controls；
- feedback/error patterns。

### S3 Delta

Candidate Profile 是 S3 新 authoritative Candidate capability。

### Do Not

- make Profile mandatory for S2 Fast Apply；
- rebuild existing authentication；
- rebuild navigation unnecessarily；
- merge Candidate Profile with Application；
- use Resume as Profile replacement。

------

# 20. Component Reuse

建议实现上存在一个稳定的 Overview composition，例如：

```text
CandidateProfilePage
    │
    ├── CandidateFacts
    │      ├── ProfileSection
    │      ├── ProfileSection
    │      └── ...
    │
    └── ApplicationTools
```

具体 component name 不冻结。

关键是避免为：

```text
SkillsCard
ExperienceCard
EducationCard
CertificationCard
LanguageCard
```

复制五套几乎相同的 card shell。

应尽可能共享 section presentation primitive。

------

# 21. Responsive / Surface Behaviour

Primary frozen design 为 desktop Web。

较窄宽度：

- section cards 可 stack；
- content 保持 readable；
- Edit action 不应脱离对应 section。

不要求在 S3 创建独立 mobile Candidate Profile product。

------

# 22. Accessibility / UX Requirements

- 每个 `Edit` 必须与 section 明确关联。
- Empty section 不能只显示空白。
- First-Time onboarding actions 要明确。
- Profile completeness 不只用颜色表达。
- Loading 与 empty 必须区分。
- Section save failure 后提供 actionable feedback。
- 用户必须能知道哪些信息当前已经保存。

------

# 23. Analytics / Observability

不要从本 spec 发明新的 product analytics。

正常 operational telemetry 可记录：

- requestId；
- candidateId；
- profileVersion；
- operation result。

不得记录：

- Candidate personal details；
- professional summary；
- experience正文；
- education正文；
- eligibility正文；
- Resume import raw content。

------

# 24. Security / Privacy

Candidate Profile 是敏感 Candidate-owned data。

Identity 只能来自 backend security context。

不能允许 client：

```text
GET /candidate/{candidateId}
```

然后仅凭前端 candidateId 判断 ownership。

Server-side authorised lookup 是必要条件。

Extension 不获得整个 Candidate Profile 的自由读取能力。

Admin 也不是 Candidate Profile universal viewer。

------

# 25. Acceptance Criteria

-  Candidate Profile has a clear Web entry point.
-  Existing Candidate facts are visible by section.
-  Candidate Facts and Application Tools are visually/semantically separated.
-  Personal Details are represented.
-  Professional Summary is represented.
-  Skills are represented.
-  Experience is represented.
-  Education is represented.
-  Certifications are represented.
-  Languages are represented.
-  Eligibility / Work Rights are represented.
-  Default Cover Letter is not represented as a Candidate fact.
-  First-time user receives clear onboarding.
-  First-time user can choose manual creation.
-  First-time user can choose Resume Import.
-  Resume Import does not silently establish Candidate facts.
-  Partial Candidate Profile is allowed.
-  Each fact section can enter its appropriate editing surface.
-  Candidate Profile uses aggregate-level `profileVersion`.
-  Child sections do not introduce independent OCC versions.
-  Stale writes are not silently merged.
-  Profile changes do not automatically regenerate derived AI artifacts.
-  S2 Fast Apply remains usable without Candidate Profile.
-  Candidate-owned data is authorised server-side.

------

# 26. Agent Implementation Notes

Before implementation:

1. Read the frozen Candidate Profile domain/API design.
2. Inspect existing S2 Web shell and routing.
3. Inspect Figma `S3-02 Candidate Profile` final Overview and First-Time frames.
4. Confirm current Figma node IDs before implementation.
5. Treat Candidate Profile as one aggregate.
6. Use one aggregate-level `profileVersion`.
7. Do not create child OCC versions.
8. Do not create multiple Candidate Profiles for different Resume types.
9. Keep Default Cover Letter outside Candidate facts.
10. Do not treat Resume Import output as accepted facts.
11. Do not automatically regenerate Match/Resume/Cover Letter after Profile save.
12. Do not require Profile globally for Fast Apply.
13. Reuse shared section/card/form primitives.
14. Do not expose Candidate Profile freely to Extension/Admin.
15. Ignore Figma annotation nodes.
16. Do not redesign the frozen UI.

```
## `S3-UI-06` 冻结点

这份 spec 最重要的是把 Candidate Profile 的工程边界钉死：

​```text
                 Candidate Profile
                 profileVersion
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      Match      Tailored Resume   Focused CL
     derived        derived         derived
```

**Profile 改变可以让下面三个东西 stale，但不能在 Profile Save 时自动把三个 AI workflow 全跑一遍。**

同时：

```text
Candidate Profile
├── CANDIDATE FACTS
│   ├── Personal Details
│   ├── Summary
│   ├── Skills
│   ├── Experience
│   ├── Education
│   ├── Certifications
│   ├── Languages
│   └── Eligibility
│
└── APPLICATION TOOLS
    └── Default Cover Letter
```

这样 Agent 后面不会因为 Default Cover Letter 恰好出现在 Profile 页面，就把它塞进 `candidate_profiles` aggregate 的事实语义里。

下一份是 **`S3-UI-07 — Resume Import Review`**。这份会重点处理整个 S3 最重要的数据安全边界之一：

> **AI extracted ≠ Candidate fact；Proposed → Review → Accept/Reject → Profile.**