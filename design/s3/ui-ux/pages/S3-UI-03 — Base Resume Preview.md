# S3-UI-03 — Base Resume Preview

```

```

# S3-UI-03 — Base Resume Preview

```yaml
spec_id: S3-UI-03
title: Base Resume Preview
surface: web
status: frozen
figma_page: S3-01 Focused Apply
figma_nodes:
  - "299:5"
depends_on:
  - candidate-profile
  - preparation
  - resume
s2_reuse: true
```

## 1. Purpose

`Base Resume Preview` 解决一个非常具体的问题：

> **用户在选择 Base Resume 作为 Tailored Resume 的 starting point 之前，可以先看看这份 Resume 到底是什么。**

它不是 Resume Editor，也不是 Resume Designer。

用户从 `S3-UI-02 — Create Tailored Resume` 点击某份 Base Resume 的 `View resume` 后进入这里，确认内容后可以：

- `Use this resume`
- `Back`

核心原则：

> **Preview is read-only and must preserve the current Focused Preparation context.**

------

## 2. Scope

### In Scope

- 展示所选 Base Resume；
- 展示足够识别 Resume 的 metadata；
- read-only 查看 Resume 内容；
- 选择该 Resume 作为当前 Tailored Resume starting point；
- 返回 starting-point selection；
- 保持当前 Candidate + Job Preparation context；
- 处理 loading / unavailable / access failure。

### Out of Scope

- 编辑 Base Resume；
- Tailored Resume generation；
- Candidate Profile editing；
- Resume Import；
- 创建新 Resume；
- 删除 Resume；
- 修改 Job；
- Match Analysis；
- Cover Letter；
- Application submission；
- Full Resume Designer。

特别禁止把：

> ```
> Preview
> ```

实现成：

> ```
> Preview + Edit
> ```

S3 没有因为这个页面引入新的复杂 Resume Designer。

------

## 3. Entry Points

主要入口：

```text
S3-UI-02 Create Tailored Resume
        ↓
Existing Base Resumes
        ↓
View resume
        ↓
S3-UI-03 Base Resume Preview
```

Preview 不应成为独立主导航入口。

它是当前 Focused Preparation 中的辅助决策状态。

------

## 4. Preconditions

必须存在：

- authenticated user；
- authorised Candidate；
- current Preparation；
- current Job context；
- selected Base Resume；
- Base Resume 属于当前 Candidate。

不要求 Application 已经存在。

不要求该 Base Resume 已经被选作 starting point。

换句话说：

```text
View resume
≠
Select resume
```

查看本身不能产生选择副作用。

------

## 5. Route / Surface

**Surface:** Web application / Focused Preparation sub-surface.

可以通过 route、nested route、modal-like page state 等现有 React routing pattern 实现，但应遵循当前 OfferBuddy Web architecture。

Page Specification 不强制 Agent 创建新的独立 top-level route。

关键 requirement 是：

```text
Preparation context
+
Job context
+
return context
```

不能丢失。

------

## 6. Figma Reference

**Figma Page**

```
S3-01 Focused Apply
```

**Frame**

```
S3 Web — Base Resume Preview
```

**Node**

```
299:5
```

视觉实现以该 final frame 为准。

Figma 中 `ANNOTATION — ...` 内容不属于应用 UI。

------

## 7. UI Structure

页面从上到下：

### A. Existing OfferBuddy Web Shell

继续复用现有：

- Header；
- Navigation；
- page container；
- typography；
- buttons；
- spacing。

不建立 Resume-specific Web shell。

### B. Context / Back Navigation

提供清晰的返回动作，例如：

> ```
> ← Back
> ```

语义是：

> 返回当前 Preparation 的 Create Tailored Resume / starting-point selection。

不能简单返回一个通用 Resume library，从而丢失当前 Job workflow。

### C. Resume Identity

展示足够的信息确认当前 Resume，例如：

- Resume name；
- relevant update metadata；
- Base Resume status/type where useful。

例如：

> Backend Engineer Resume

### D. Resume Preview

read-only 展示 Resume 的实际用户可见内容。

应尽可能反映用户将要选择的 Base Resume，而不是仅显示：

- extracted keywords；
- Candidate Profile；
- AI summary。

用户需要判断的是：

> “是不是我想用的那份简历？”

### E. Primary Action

> **Use this resume**

明确选择当前 Base Resume 作为 `S3-UI-02` 的 starting point。

### F. Secondary Action

> **Back**

返回但不选择。

------

## 8. Data Dependencies

### Reads — Base Resume

需要：

- resumeId；
- display name；
- Candidate ownership；
- resume content / renderable representation；
- relevant metadata；
- availability/status；
- version/provenance information where defined by frozen Resume contract。

### Reads — Preparation Context

需要保留：

- preparationId；
- current Job；
- Candidate；
- current workflow context。

### Writes

Preview 本身：

> **No domain write.**

点击 `Use this resume` 主要表达 UI workflow selection。

如果 frozen API 将 starting-point selection 持久化为业务状态，则必须使用其正式 contract；否则不得为了 Preview 自行增加 persistence endpoint。

------

## 9. Field Specification

本页面没有 editable fields。

| Field               | Type     | Editable | Behaviour                 |
| ------------------- | -------- | -------- | ------------------------- |
| Resume name         | Text     | No       | Identifies Base Resume    |
| Resume metadata     | Metadata | No       | Supporting identification |
| Resume content      | Document | No       | Read-only preview         |
| Preparation context | Context  | No       | Must be preserved         |

------

## 10. Actions

### Use This Resume

**Trigger**

用户确认这就是希望用于当前 Job 的 Base Resume。

**Behaviour**

将该 Resume 作为当前 Tailored Resume starting point。

然后返回 `S3-UI-02` 的 selected state，或者按照 frozen workflow 直接进入等价的下一步确认状态。

推荐交互语义：

```text
Preview
   ↓
Use this resume
   ↓
Create Tailored Resume
   ↓
Backend Resume ✓ selected
   ↓
Create tailored resume
```

这里有一个重要限制：

> **`Use this resume` 不等于立即调用 AI。**

用户明确点击：

> ```
> Create tailored resume
> ```

才触发 generation。

这样可以避免：

```text
View
→ Use
→ accidentally starts expensive AI generation
```

------

### Back

返回 `S3-UI-02`。

不得：

- 生成 Resume；
- 修改 Candidate Profile；
- 修改 Base Resume；
- 创建 Application；
- 清除 Preparation。

------

## 11. UI States

### Loading

正在获取 Base Resume 内容。

显示 existing OfferBuddy loading pattern。

不能先显示空白 document 并让用户误以为 Resume 是空的。

### Ready

完整 read-only preview 可用。

### Unavailable

Base Resume resource 存在，但当前 preview/content 暂时不可用。

提供：

- clear explanation；
- Back。

不能用 Candidate Profile 自动生成一份“差不多的 preview”来代替。

### Access Failure

资源不属于当前 Candidate 或 authorization failure。

遵循 frozen API security semantics。

不得展示任何 Resume 内容。

### Request Failure

网络/服务错误。

允许安全 retry read。

------

## 12. Business Rules

1. Base Resume Preview 是 **read-only**。
2. Preview 不修改 Base Resume。
3. Preview 不修改 Candidate Profile。
4. Preview 不触发 Tailored Resume generation。
5. `View resume` 不等于 `Select resume`。
6. `Use this resume` 只确定 starting point。
7. Tailored Resume generation 需要后续明确 action。
8. Base Resume 必须属于当前 authorised Candidate。
9. Base Resume 不是 Candidate fact authority。
10. 即使 Preview 中存在某句话，也不能因此绕过 Candidate Profile truth boundary。
11. Preview 必须保持当前 Preparation context。
12. Application 不是 prerequisite。

------

## 13. API Mapping

使用 Phase 3 §3.17 已冻结的 Resume / Candidate ownership contracts。

### Read Base Resume

使用 frozen Resume read capability 获取 authorised Base Resume。

Agent 不得因为需要 Preview 就自行创建类似：

```text
GET /api/resume-preview/{id}
```

除非 frozen API contract 已经定义这样的 endpoint。

如果现有 Resume representation 已足够渲染，应直接复用。

### Selection

`Use this resume` 是否需要 backend command，取决于 frozen Preparation/Resume contract。

如果 selection 只是 generation command 前的 UI input：

> 保持 client workflow state 即可。

不要为了保存一个临时 radio selection 创建新的 domain endpoint。

------

## 14. Concurrency / Staleness

Preview 是 read-only，因此不产生传统 stale-write conflict。

但是必须考虑：

```text
User opens Resume A
        ↓
Resume A changes/deactivates elsewhere
        ↓
User clicks Use this resume
```

最终 generation command 必须由 backend 验证 starting point 仍然有效。

Frontend 不能因为之前 preview 成功就认为资源永久有效。

Candidate Profile / Job 的真正 generation staleness 由 `S3-UI-02` 和 Tailored Resume generation contract 负责。

------

## 15. Navigation / Transitions

| From                   | Action               | Destination / State               |
| ---------------------- | -------------------- | --------------------------------- |
| Create Tailored Resume | View Resume          | Base Resume Preview               |
| Preview                | Back                 | Create Tailored Resume            |
| Preview                | Use this resume      | Create Tailored Resume — selected |
| Preview                | Read failure retry   | Preview loading                   |
| Preview                | Resource unavailable | Remain / Back                     |

核心要求：

```text
Create Tailored Resume
        ↓
Preview
        ↓
Back / Use
        ↓
same Preparation
```

------

## 16. S2 Reuse

### Reuse

如果 S2/现有代码已经存在 Resume rendering capability，应优先复用：

- document rendering；
- common page shell；
- buttons；
- loading/error components；
- typography。

### S3 Delta

增加：

> 从 Focused Preparation 中查看 Base Resume 并选择其作为 starting point。

### Do Not

- 创建第二套 Resume renderer；
- 创建 Full Resume Designer；
- 重写 Candidate Profile；
- 将 Preview 混入 Application Detail；
- 影响 Fast Apply。

------

## 17. Component Reuse

优先检查现有 Resume/document presentation components。

可能需要的概念：

```text
BaseResumePreview
ResumeDocumentView
ResumePreviewHeader
```

名称不是 contract。

如果现有 document renderer 可以满足 Figma：

> reuse it.

不要为了完全相同的数据创建 S3-specific duplicate renderer。

------

## 18. Responsive / Surface Behaviour

Frozen design 以 desktop Web 为主。

Resume document 在较窄 viewport 下应：

- 保持可读；
- 避免横向内容被截断；
- 使用现有 Web responsive behaviour。

不要求实现独立 mobile Resume Preview。

------

## 19. Accessibility / UX Requirements

- Back action 必须明确。
- Primary/secondary action hierarchy 与 Figma 一致。
- Document text 应保持可读。
- Keyboard 用户能够访问 Back 和 Use actions。
- Loading 状态必须有明确语义。
- Error 不应只通过颜色表达。
- Read-only preview 不应出现让用户误认为可编辑的 form controls。

------

## 20. Analytics / Observability

不因为用户查看 Resume 自动新增产品 analytics。

正常 request diagnostics 可包含：

- requestId；
- authorised resource identifier；
- correlation context where applicable。

不得记录：

- Resume正文；
- Candidate Profile正文；
- sensitive Candidate data。

------

## 21. Security / Privacy

Base Resume 属于 Candidate-owned data。

Backend 必须执行 ownership authorization。

不能：

```text
GET resumeId from URL
→ frontend checks candidateId
→ assume authorised
```

正确模型：

```text
request
→ backend security context
→ authorised Resume lookup
→ result
```

ownership failure 不得泄漏 Resume 是否属于另一个用户。

------

## 22. Acceptance Criteria

-  User can open a Base Resume from Create Tailored Resume.
-  Correct Base Resume identity is visible.
-  Actual Base Resume is presented read-only.
-  Preview does not modify Base Resume.
-  Preview does not modify Candidate Profile.
-  Preview does not generate Tailored Resume.
-  User can return with Back.
-  Back preserves the current Preparation.
-  User can choose `Use this resume`.
-  Selection preserves the current Preparation.
-  Selected Resume is reflected in Create Tailored Resume.
-  Selection itself does not invoke AI generation.
-  User still explicitly triggers `Create tailored resume`.
-  Loading state is represented.
-  Read failure is represented.
-  Unavailable Resume does not get replaced with fabricated preview content.
-  Candidate ownership is enforced server-side.
-  Fast Apply remains unchanged.

------

## 23. Agent Implementation Notes

Before implementation:

1. Inspect existing Resume rendering code.
2. Read frozen Resume API contract.
3. Use Figma node `299:5`.
4. Treat the surface as read-only.
5. Preserve `preparationId` / current Focused Apply context.
6. Do not turn Preview into an editor.
7. Do not create a new Resume Designer.
8. Do not trigger AI when `Use this resume` is clicked.
9. Keep generation as the explicit action defined by `S3-UI-02`.
10. Do not treat Base Resume content as authoritative Candidate facts.
11. Perform no client-side ownership authorization assumptions.
12. Reuse existing S2/Web components wherever practical.
13. Ignore Figma annotation nodes.
14. Do not redesign the frozen frame.

```
### `S3-UI-03` 可以冻结

这份 spec 最关键的是把一个很容易被 Agent 做错的行为固定住：

**`View resume` → `Use this resume` → `Create tailored resume` 是三个不同动作。**

特别是 `Use this resume` **不会立即消耗一次 AI generation**。

下一份进入 **`S3-UI-04 — Tailored Resume Review`**。这会重新变成一份比较重要的 spec，因为要定义 **AI 生成后用户到底能 review/edit 什么、如何保存、如何处理 provenance/stale，以及不能把修改反向污染 Candidate Profile/Base Resume**。
```