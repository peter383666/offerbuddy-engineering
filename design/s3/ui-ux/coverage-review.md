**S3 Page Specification Final Coverage Review**。

继续。现在进入 **S3 Page Specification Final Coverage Review**。

这一步不是继续增加文档，而是把三个东西逐一对账：

> **Phase 4 frozen Figma → 35 个最终 UI Frame → S3-UI-01～20 Specifications**

结论先给你：**当前没有发现需要新增 `S3-UI-21` 的独立业务页面。20 份 Specification 可以完整覆盖当前 35 个最终 Frame。** 但 review 后有几处需要在最终 Index 里特别标注，否则 Agent 仍可能误解。

# 1. Focused Apply — Coverage PASS

Figma 最终有 13 个 Frame，其中 Web 5 个核心页面 + Extension 多状态。

| Figma Frame                    | Specification | Result |
| ------------------------------ | ------------- | ------ |
| Match Decision                 | `S3-UI-01`    | ✅      |
| Create Tailored Resume         | `S3-UI-02`    | ✅      |
| Base Resume Preview            | `S3-UI-03`    | ✅      |
| Tailored Resume Review         | `S3-UI-04`    | ✅      |
| Focused Cover Letter Review    | `S3-UI-05`    | ✅      |
| Floating Assistant — Collapsed | `S3-UI-15`    | ✅      |
| Hover / Default                | `S3-UI-15`    | ✅      |
| Sponsor Signal                 | `S3-UI-15`    | ✅      |
| Requirement Review             | `S3-UI-15`    | ✅      |
| Preparation Ready              | `S3-UI-15`    | ✅      |
| Already Applied                | `S3-UI-15`    | ✅      |
| SEEK Cover Letter Step         | `S3-UI-16`    | ✅      |
| SEEK Prepared Cover Letter     | `S3-UI-16`    | ✅      |

这里合并是正确的。

尤其 Extension 不能拆成六份 spec，否则 Agent 容易实现成六套互相独立的 UI。

正确模型继续保持：

```text
Extension Floating Assistant
        │
        └── one state resolver
             ├── Collapsed
             ├── Default
             ├── Sponsor
             ├── Requirement
             ├── Preparation Ready
             └── Already Applied
```

### Review 发现的关键约束

必须继续在 Index 中强调：

```text
Extension
≠ Match surface
≠ Resume surface
≠ Cover Letter editor
```

Extension 只负责：

```text
detect
→ hint
→ handoff
→ SEEK fill
→ applied-state reflection
```

**Focused Apply coverage：PASS。**

------

# 2. Candidate Profile — Coverage PASS

Figma 最终有 12 个主要 Frame。

| Figma Frame                    | Specification | Result |
| ------------------------------ | ------------- | ------ |
| Candidate Profile — Main       | `S3-UI-06`    | ✅      |
| Candidate Profile — First Time | `S3-UI-06`    | ✅      |
| Resume Import Review           | `S3-UI-07`    | ✅      |
| Personal Details Edit          | `S3-UI-08`    | ✅      |
| Professional Summary Edit      | `S3-UI-09`    | ✅      |
| Skills Edit                    | `S3-UI-10`    | ✅      |
| Experience Edit                | `S3-UI-11`    | ✅      |
| Education Edit                 | `S3-UI-12`    | ✅      |
| Certifications Edit            | `S3-UI-13`    | ✅      |
| Languages Edit                 | `S3-UI-14`    | ✅      |
| Eligibility Edit               | `S3-UI-14`    | ✅      |
| Default Cover Letter Edit      | `S3-UI-16`    | ✅      |

这里有两个故意的跨目录映射。

### Languages + Eligibility

两张 Figma 页面，共用：

> ```
> S3-UI-14
> ```

这个可以保留。

但代码上**不能因此把它们建模成同一个 domain object/component**。

Spec 只是因为 CRUD contract 接近而合并文档。

------

### Default Cover Letter

虽然 Figma 在 Candidate Profile Page：

```text
S3-02 Candidate Profile
```

但 specification 属于：

> ```
> S3-UI-16 — SEEK Cover Letter Assistance
> ```

这是故意的。

最终 Index 必须写：

```text
Visual location:
Candidate Profile → Application Tools

Business ownership:
Fast Apply / Cover Letter configuration

NOT:
Candidate facts
```

否则 Agent 很容易把 `defaultCoverLetter` 塞进 Candidate fact aggregate。

------

# 3. Candidate Profile Fact Boundary — PASS

所有事实页面现在都满足同一个模型：

```text
Candidate Profile
profileVersion
│
├── Personal Details
├── Professional Summary
├── Skills
├── Experience
├── Education
├── Certifications
├── Languages
└── Eligibility
```

所有 meaningful change：

```text
Profile vN
   ↓
write
   ↓
Profile vN+1
```

并且没有任何 spec 错误引入：

```text
skillVersion
experienceVersion
educationVersion
languageVersion
```

这和 Phase 3 frozen OCC 一致。

### Conflict contract 也完整

所有 Candidate editor 都已经明确：

```text
stale profileVersion
→ 409
→ preserve local work
→ explicit review
→ NEVER silent merge
```

因此这一块没有 gap。

------

# 4. Resume Import Trust Boundary — PASS

`S3-UI-07` 已经覆盖：

```text
Resume
  ↓
AI extraction
  ↓
Proposal
  ↓
Review
  ↓
Accept / Edit / Reject
  ↓
Candidate Profile
```

没有任何其他 Page Spec 允许绕过这个流程。

这一点非常重要，因为否则可能出现：

```text
Resume Import
→ Skills automatically saved

Resume Import
→ Education automatically saved

Resume Import
→ Eligibility inferred
```

当前 01～20 specs 已全部明确禁止。

**PASS。**

------

# 5. Multiple Resume Model — PASS

三个层级现在也一致：

```text
Candidate Profile
authoritative facts
        │
        ▼
Multiple Base Resumes
reusable presentation
        │
        ▼
Current Job Tailored Resume
derived artifact
```

对应：

- `S3-UI-02` Choose Base Resume
- `S3-UI-03` Preview
- `S3-UI-04` Tailored Resume Review

没有发现需要单独建立：

> Base Resume Management Dashboard

的冻结 UI。

所以 **不增加新 spec**。

未来如果真的设计一个完整 Resume Library，那是新的 scope，不属于当前 S3 UI freeze。

------

# 6. Application Detail — Coverage PASS

Figma：

> ```
> S3 Application Detail — Focused Preparation
> ```

对应：

> ```
> S3-UI-17
> ```

而且已经明确：

```text
S2 Application Detail
        +
small S3 Preparation integration
```

不是：

```text
new S3 Application Detail
```

这是当前 Page Spec Pack 里非常重要的 Agent guardrail。

### 实现阶段应该只有

```text
existing ApplicationDetailPage
        │
        └── PreparationSummary / entry
```

而不是：

```text
ApplicationDetailPage
S3ApplicationDetailPage
```

**PASS。**

------

# 7. RuoYi Sponsor Employers — Coverage PASS

Figma 五个 Sponsor-related Frame：

| Frame                | Spec       |
| -------------------- | ---------- |
| CRUD List            | `S3-UI-18` |
| Create/Edit          | `S3-UI-18` |
| Batch Import         | `S3-UI-18` |
| Publish Confirmation | `S3-UI-18` |
| Published            | `S3-UI-18` |

合并成一份是正确的，因为它是一个业务 workflow：

```text
CRUD
 ↓
Working Dataset
 ↓
Batch Import
 ↓
Review
 ↓
Publish
 ↓
Snapshot vN
```

而不是五个独立产品功能。

------

# 8. Sponsor Dataset → Extension — PASS

跨 Specification 对账也正确：

### Admin

```
S3-UI-18
working dataset
→ explicit publish
→ versioned snapshot
```

### Extension

```
S3-UI-15
snapshot sync
→ local cache
→ employer lookup
```

两边没有矛盾。

最终核心性能规则保持：

```text
Employer found locally
→ sponsor signal

Employer NOT found locally
→ do nothing
→ NO per-job backend sponsor query
```

这是实现 Agent 必须特别看到的 cross-spec contract。

------

# 9. AI Runtime Admin — Coverage PASS

Figma：

- AI Runtime Configuration
- AI Capability Edit

统一对应：

> ```
> S3-UI-19
> ```

正确。

因为：

```text
List/config overview
        ↓
Edit one capability
```

本质是一个 Admin capability。

没有必要拆 `AI Provider Page`、`Model Page`、`Secret Page`。

尤其：

> Secret 根本不是 Admin 页面。

Secret 只是：

```text
external secret
      ↓
safe reference/status
      ↓
RuoYi
```

------

# 10. AI Monitoring — Coverage PASS

Figma：

- AI Monitoring
- AI Failure Detail

统一对应：

> ```
> S3-UI-20
> ```

这个也是正确的。

Failure Detail 不是 raw debug console，而是 Monitoring 的 drill-down。

允许：

```text
requestId
capability
provider
model
latency
attempt
safe failure category
```

禁止：

```text
Candidate Profile
prompt
raw response
Resume
Cover Letter
raw Job description
secret
```

没有遗漏单独页面。

------

# 11. Async State Coverage — PASS

我们再从 Phase 3 的 async contract 反向检查 UI。

需要 async 的主要 S3 capabilities：

```text
Resume Import
Job Intelligence
Match
Tailored Resume
Focused Cover Letter
```

UI spec 覆盖情况：

| Capability           | Async UI                                             |
| -------------------- | ---------------------------------------------------- |
| Resume Import        | `S3-UI-07` ✅                                         |
| Match                | `S3-UI-01` ✅                                         |
| Tailored Resume      | `S3-UI-02/04` ✅                                      |
| Focused Cover Letter | `S3-UI-05` ✅                                         |
| Job Intelligence     | 主要作为 upstream/business state，不独立设计用户页 ✅ |

共同 contract 已明确：

```text
POST
 ↓
202 Accepted
 ↓
business resource PROCESSING
 ↓
read/poll business resource
 ↓
READY / FAILED
```

而不是：

```text
generic operationId
```

**没有 UI gap。**

------

# 12. Loading / Empty / Error Coverage — PASS

当前主要 surfaces 已全部有 loading/error semantics。

### Candidate Profile

包含：

```text
Loading
Empty
Editing
Saving
Save Error
409 Conflict
```

### AI-derived artifacts

包含：

```text
Loading
Processing
Ready
Failed
Stale
```

### Extension

包含：

```text
safe neutral
resolving
backend unavailable
snapshot unavailable
```

### Admin

包含：

```text
Loading
Ready
Validation error
Request failure
Permission failure
Import failure
Publish failure
```

因此无需为这些状态增加新的 Figma Page Specification。

------

# 13. Staleness Coverage — PASS

所有 relevant derived artifacts 都已经使用：

```text
profileVersion
contentVersion
```

进行 provenance/staleness。

覆盖：

```text
Match
Tailored Resume
Focused Cover Letter
```

并且 Application：

```text
applications.version
```

没有被混进 derived-artifact provenance。

三个 version 继续严格分离：

```text
Candidate → profileVersion
Job       → contentVersion
Application → version
```

**PASS。**

------

# 14. 一个需要在 Final Index 特别补充的点

虽然没有缺 UI，但 Page Specs 里存在很多跨文档 dependency。

Coding Agent 如果一次只拿一份 spec，很容易漏掉。

所以最终 Index 不能只是：

```text
S3-UI-01
S3-UI-02
...
```

还必须提供 **implementation dependency graph**。

我建议固定成：

```text
Candidate Profile
S3-UI-06
  │
  ├─07 Import
  ├─08 Personal
  ├─09 Summary
  ├─10 Skills
  ├─11 Experience
  ├─12 Education
  ├─13 Certifications
  └─14 Languages / Eligibility
          │
          ▼
Focused Apply
          │
          ├─01 Match
          │
          ├─02 Create Resume
          │    └─03 Base Resume Preview
          │
          ├─04 Tailored Resume
          │
          └─05 Focused Cover Letter
                   │
                   ▼
               Extension
             15 + 16
                   │
                   ▼
            Application Detail
                  17
```

Admin 独立：

```text
18 Sponsor Dataset
     ↓
15 Extension snapshot

19 AI Runtime Config
     ↓
AI capabilities
     ↓
20 AI Monitoring
```

------

# 15. Page Spec ≠ GitHub Issue

Final review 再次确认：

**20 specs 不应该机械生成 20 GitHub Issues。**

例如 Candidate Profile：

```text
S3-UI-08
S3-UI-09
S3-UI-10
S3-UI-11
S3-UI-12
S3-UI-13
S3-UI-14
```

可能在 Phase 5 delivery planning 里属于：

> Candidate Profile frontend Epic

再按照工程 workload 拆 Issue。

而：

```text
S3-UI-18
```

一个 spec 反而可能拆：

```text
Sponsor CRUD backend
Sponsor Admin frontend
Batch import
Dataset publish
Extension snapshot integration
```

Specification 和 delivery issue 是两种不同抽象。

------

# 16. Final Specification Inventory

最终冻结仍为 **20 份**：

```text
FOCUSED APPLY

S3-UI-01  Match Analysis
S3-UI-02  Create Tailored Resume
S3-UI-03  Base Resume Preview
S3-UI-04  Tailored Resume Review
S3-UI-05  Focused Cover Letter Review


CANDIDATE PROFILE

S3-UI-06  Candidate Profile Overview
S3-UI-07  Resume Import Review
S3-UI-08  Personal Details
S3-UI-09  Professional Summary
S3-UI-10  Skills
S3-UI-11  Experience
S3-UI-12  Education
S3-UI-13  Certifications
S3-UI-14  Languages & Eligibility


EXTENSION / FAST APPLY

S3-UI-15  Extension Floating Assistant
S3-UI-16  SEEK Cover Letter Assistance


S2 INTEGRATION

S3-UI-17  Application Detail Integration


RUOYI ADMIN

S3-UI-18  Sponsor Employers Admin / CRUD
S3-UI-19  AI Governance / Runtime Configuration
S3-UI-20  AI Monitoring / Operational View
```

------

# 17. Coverage Review Result

最终结果：

> **S3 Page Specification Coverage Review — APPROVED**

统计：

```text
Final Figma UI Frames:      35
Engineering Specifications: 20
Uncovered Frames:            0
Orphan Specifications:       0
Major UI workflow gaps:      0
```

之所以是 **35 → 20** 而不是 35 → 35，是因为我们已经正确把：

- visual states；
- CRUD workflow states；
- Extension state machine；
- Admin drill-down；

合并到真正的 engineering surface specification 里。

这比给 Agent 35 个碎片文档更好。

------

## 下一步

现在最合适的是生成整个 Page Specification Pack 的最后两个入口文档：

**`README.md` + `S3 UI Specification Index`**

Index 不只是目录，还会包含：

- Spec ID
- surface
- Figma reference
- dependencies
- backend capability
- S2 reuse
- recommended implementation order

这样下一阶段 Agent 开始编码时，只需要先读 Index，就知道应该加载哪些 specs，而不是自己在 20 个 Markdown 文件之间猜依赖。

完成 Index 后，这整个 **S3 Page Specification Pack 就可以正式 Freeze**。