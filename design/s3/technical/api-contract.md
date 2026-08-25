# Sprint 3 API and Contract Detailed Design

## Publication Note

This is the complete frozen Phase 3 §3.17 design record. The final frozen decision and endpoint/DTO/error/OCC/idempotency semantics in this record are authoritative. Opening/closing workflow remarks are retained for design traceability and do not create additional implementation scope.

已保存。**3.16 Observability, Audit & Operational Diagnostics 现在正式作为 Phase 3 frozen baseline 的一部分。** 新对话可以直接从下一节开始，不需要重新回顾 3.1–3.16，除非后面发现 contradiction。

我建议下一部分进入：

# 3.17 API & Contract Detailed Design

这个顺序最自然。因为到 3.16 为止，我们已经把内部 runtime 设计基本完成了；接下来应该把这些内部设计收敛成**真正对 React、Extension、RuoYi/Admin 暴露的稳定 contract**。

核心问题是：

> **S3 的 Candidate / Job Intelligence / Match / Resume / Cover Letter / Generation / Admin 能力，最终通过什么 API contract 暴露？哪些是同步，哪些是异步？版本冲突、generation 状态、错误、幂等和权限在 HTTP 层到底长什么样？**

我建议下一部分按这个框架推进：

1. **3.17.1 API Design Principles & Boundaries**
   REST/resource-oriented 基线、Fast Path兼容、module API boundary、DTO ≠ Entity、同步/异步边界、API versioning原则。

2. **3.17.2 Candidate Profile API**
   Profile read/create/update、child facts如何提交、`profileVersion` OCC、Resume Import draft → review → accept/merge contract，以及 capability sufficiency 的表达。

3. **3.17.3 Job & Job Intelligence API**
   Prepared Job创建/读取、canonical Job与Application关系、`contentVersion`、Job Intelligence生成/读取，以及 prepared-before-application 生命周期。

4. **3.17.4 Match Analysis API**
   Candidate + Job anchored Match generation、读取、重新生成、provenance/version、explainable result，以及 stale result语义。

5. **3.17.5 Tailored Resume API**
   generation request、status/read、artifact retrieval、重新生成、Candidate/Job provenance，以及不得 silently mutate Candidate facts。

6. **3.17.6 Cover Letter API**
   与 Resume保持一致的 async generation contract，同时明确 Cover Letter自己的artifact/result语义，避免复制一套不一致API。

7. **3.17.7 Generation & Async API Pattern**
   这是很关键的一节。统一：

   ```text
   POST generation
   → accepted/created
   → generationId
   → status
   → SUCCEEDED / FAILED
   ```

   以及 polling、duplicate request、active generation、superseded/stale、retry/re-generation的HTTP语义。

8. **3.17.8 Application / Fast Path Compatibility**
   明确现有 S2 Application APIs哪些完全不动；S3只能 additive extension，不能让 Candidate/Profile/AI成为 Application create/update前置条件。也把 `applications.version` 的 409 contract正式落到API层。

9. **3.17.9 Error, OCC & Idempotency Contracts**
   把3.15和3.16真正落到HTTP：

   ```text
   VERSION_CONFLICT
   IDEMPOTENCY_CONFLICT
   GENERATION_ALREADY_ACTIVE
   RESOURCE_NOT_FOUND
   FORBIDDEN
   AI_TEMPORARILY_UNAVAILABLE
   ...
   ```

   明确400/401/403/404/409/422/5xx的边界，以及统一error envelope。

10. **3.17.10 RuoYi / Admin Operational APIs**
    AI Governance、AI diagnostics、business event diagnostics、audit、manual re-drive等Admin API contract；严格保持Admin不是domain owner。

11. **3.17.11 API Security & Data Exposure Review**
    用3.14重新pressure test所有DTO：ownership lookup、IDOR、防止Entity leakage、Candidate/Resume/AI敏感字段、Admin safe DTO、错误信息泄露等。

12. **3.17.12 Final API Inventory & Implementation Delta**
    最后输出真正有开发价值的清单：

    ```text
    Existing APIs unchanged
    Existing APIs modified
    New S3 endpoints
    Request DTOs
    Response DTOs
    Error codes
    OCC requirements
    Idempotency requirements
    Async endpoints
    Admin endpoints
    ```

    同时检查是否需要任何schema delta。正常来说3.17应该主要是contract delta，而不是再大量改DB。

完成 3.17 后，我们就会从：

```text
internal technical design
        ↓
stable external contracts
```

真正跨过去。这样后面的 implementation planning、Swagger/OpenAPI、GitHub Issues拆分都会容易很多。

**新对话直接发：`继续 3.17 API & Contract Detailed Design`** 就可以从 **3.17.1** 开始。



好，我们直接进入 **3.17 API & Contract Detailed Design**。

前面的 **3.1–3.16 全部视为 frozen baseline**。这一节不重新讨论模块边界、数据库模型、async 基础设施、安全模型、并发策略或 observability；这里只把这些已经确定的设计真正收敛成 **可以直接指导 Controller / DTO / Frontend / Extension / Admin 实现的 HTTP API 与 Contract**。

这一节我建议按下面顺序推进，而且仍然坚持你之前的方式：**一小节一小节定稿，不一次性铺完。**

### 3.17 API & Contract Detailed Design

**3.17.1 API Design Principles & Contract Boundaries**
先确定全局规则：

- API namespace / versioning
- resource-oriented vs command-oriented endpoint
- DTO boundary
- ID exposure
- HTTP semantics
- ownership enforcement boundary
- synchronous vs asynchronous contract
- Web / Extension / Admin contract separation

**3.17.2 Common Request / Response Contract**
统一：

- success response
- error response
- validation errors
- pagination
- timestamps
- enum representation
- requestId
- HTTP status mapping

**3.17.3 Candidate Profile API**

**3.17.4 Resume Import API**

**3.17.5 Job & Job Intelligence API**

**3.17.6 Preparation / Match API**

**3.17.7 Tailored Resume API**

**3.17.8 Cover Letter API**

**3.17.9 Application Compatibility / S2 Fast Path**

**3.17.10 Browser Extension Contract**

**3.17.11 Admin / AI Governance API**

**3.17.12 Async Operation & Status Contract**

**3.17.13 Optimistic Concurrency / Idempotency HTTP Contract**

**3.17.14 API Security & Data Exposure Rules**

**3.17.15 Final API Contract Review**

这里有一个重要原则：**3.17 不是重新设计业务。**

例如：

> “Match 应该怎样计算？”——已经 frozen，不属于 3.17。
> “Match API 应该返回哪些字段、什么状态、什么时候 202、什么时候 200？”——属于 3.17。

------

# 3.17.1 API Design Principles & Contract Boundaries

我建议先冻结以下规则。

## A. API namespace

继续使用统一 backend API namespace：

```
/api/...
```

S3 **不引入 `/api/v1`**。

原因是 OfferBuddy 当前仍然是单一产品、单 backend、React + Extension + Admin 的早期 production system，没有公开 third-party API compatibility obligation。

现在引入 `/v1` 反而意味着我们承诺维护正式 external API version lifecycle。

未来真正需要 breaking public contract 时再引入版本化。

所以：

**Decision 3.17.1-A**

> S3 HTTP APIs remain under `/api/**`.
> API versioning is not introduced solely for S3.

------

# B. API 按 domain capability 划分

Endpoint 应该反映我们已经 frozen 的 domain boundary，而不是数据库表。

例如：

```text
/api/candidate/profile
/api/candidate/resume-imports

/api/jobs/{jobId}
/api/jobs/{jobId}/intelligence

/api/preparations/...
```

而不是：

```text
/api/candidate-skills
/api/candidate-experiences
/api/match-results
/api/resume-sections
```

尤其 Candidate child tables 是 **Candidate Profile aggregate 的内部 persistence structure**。

因此不能因为 DB 有：

```text
candidate_skills
candidate_experiences
candidate_education
...
```

就暴露一套 CRUD API。

### Decision

> HTTP APIs expose domain capabilities and aggregate boundaries, not persistence tables.

------

# C. Candidate Profile 使用 aggregate contract

Candidate Profile 已经 frozen 为：

> authoritative source of candidate facts

所以 API 应该围绕：

```text
GET    /api/candidate/profile
PUT    /api/candidate/profile
```

以及明确的 capability endpoint。

而不是把：

```text
skill
experience
education
certification
language
```

全部做独立顶层 CRUD resource。

客户端提交 Profile 时，backend 负责 aggregate validation 和 persistence。

这同时与我们在 3.15 定下来的：

```
candidate_profiles.profile_version
```

保持一致。

Profile version 是 **整个 Candidate facts aggregate 的 OCC boundary**。

------

# D. Resource API + explicit command API 混合

OfferBuddy 不应该为了 REST purity 把所有业务动作硬塞成 CRUD。

普通资源生命周期使用 resource-oriented API：

```text
GET
POST
PUT
DELETE
```

但真正具有业务语义的操作使用 explicit command endpoint。

例如：

```text
POST /api/candidate/resume-imports
POST /api/candidate/resume-imports/{importId}/accept

POST /api/preparations/{preparationId}/match/generate
POST /api/preparations/{preparationId}/resumes
POST /api/preparations/{preparationId}/cover-letters
```

而不是设计成：

```text
PUT /match/status
PUT /resume/generated
```

### Decision

> CRUD semantics are used for resource state; explicit command endpoints are used for meaningful domain actions.

这对后面的 async/idempotency contract 也会非常重要。

------

# E. API DTO 与 Entity 完全分离

继续执行前面的 module-boundary 原则：

**JPA Entity 永远不直接作为 HTTP request/response。**

结构必须保持：

```text
HTTP
 ↓
Request DTO
 ↓
Application / Domain capability
 ↓
module-private persistence
 ↓
Response DTO
```

因此：

- Entity 不暴露给 frontend
- Repository model 不作为 API model
- AI provider DTO 不泄漏到业务 API
- internal event payload 不作为 HTTP DTO
- Admin DTO 与 Candidate-facing DTO 分开

这也防止未来数据库 migration 意外变成 API breaking change。

------

# F. Public identifier 只暴露 domain ID

这一点沿用 OfferBuddy 一直以来的设计：

数据库可能存在：

```text
id BIGINT
candidate_id UUID
job_id UUID
application_id UUID
...
```

HTTP contract：

**只暴露 UUID domain identifier。**

例如：

```json
{
  "candidateId": "...",
  "jobId": "...",
  "resumeId": "..."
}
```

不暴露：

```json
{
  "id": 1842
}
```

内部 surrogate PK：

```
BIGINT id
```

只属于 persistence layer。

------

# G. userId 默认不进入 Candidate-facing contract

因为我们在 3.14 已经冻结：

> identity comes only from backend security context.

所以不能出现这种客户端 contract：

```text
GET /api/users/{userId}/candidate
POST /api/users/{userId}/preparations
```

客户端也不能提交：

```json
{
  "userId": "..."
}
```

来声明 ownership。

应该是：

```text
GET /api/candidate/profile
```

backend：

```text
SecurityContext
      ↓
authenticated user
      ↓
Candidate ownership resolution
```

### Decision

> Candidate-facing APIs derive user identity from the authenticated backend security context. `userId` is not accepted as an ownership selector.

这直接落实 3.14 的：

**authentication ≠ authorization**。

------

# H. Application API 保持独立

这里尤其不能因为 S3 Preparation 引入 Candidate + Job，就把 S2 Application contract 重构掉。

已经 frozen：

> Application remains independently user-owned and independent of Candidate/S3 Preparation.

因此：

```text
/api/applications/**
```

继续是独立 capability。

不能变成：

```text
/api/candidates/{candidateId}/applications
```

也不能要求：

```text
Candidate Profile exists
Preparation exists
Match exists
```

才能记录 Application。

这就是 **S2 Fast Path preservation** 在 API 层的具体体现。

------

# I. Preparation API 不能以 Application 为父资源

同理，Preparation 已经冻结为：

> Candidate + Job

而不是：

> Application + Job

所以不能设计：

```text
/api/applications/{applicationId}/match
/api/applications/{applicationId}/resume
/api/applications/{applicationId}/cover-letter
```

核心 Preparation contract 必须独立于 Application。

Application 可以在 UI/workflow 层关联到同一个 Job，但不能成为 Preparation aggregate 的 ownership root。

------

# J. Job 不是 globally public API resource

3.14 已经冻结：

> Job is not globally public in S3.

因此即使客户端知道：

```text
jobId
```

也不能：

```text
GET /api/jobs/{jobId}
```

→ “UUID 存在，所以返回。”

backend 必须验证 authenticated user 是否拥有：

- authorised Application relationship，或
- authorised Preparation relationship，
- 或正在执行建立该 relationship 的合法 workflow。

换句话说：

**UUID ≠ authorization capability。**

这一规则后面 3.17.14 会具体化，但 contract boundary 现在先冻结。

------

# K. Sync / Async 由 operation semantics 决定

普通 DB-local operation：

```text
GET profile
PUT profile
GET application
update application
```

保持 synchronous。

涉及：

- external fetch
- AI
- expensive generation
- durable background processing

的操作允许：

```text
POST command
        ↓
persist business state + business_event
        ↓
return accepted operation
        ↓
worker
```

不能为了“API 看起来简单”让 HTTP request 等待不可控 AI/external latency。

这和 3.13 的 durable async design 完全一致。

但是：

> **不是所有 AI endpoint 强制 async。**

是否 async 应由已经定义的 capability latency / durability semantics 决定，而不是简单规定 “AI = 202”。

我们在后面的每个具体 API 中逐个定。

------

# L. Browser Extension 不是 trusted backend client

Extension 可以拥有专门的 API contract，但：

```text
Extension
   ↓
HTTP API
   ↓
same authentication
same authorization
same validation
same ownership rules
```

不能因为 Extension 是我们自己的代码就：

- 接受 trusted userId
- 绕过 ownership
- 接受未经验证的 Job HTML
- 直接调用 internal service endpoint
- 使用 Admin API

Extension contract 应该只暴露其需要的最小 capability。

------

# M. Admin API 与 user-facing API 明确隔离

RuoYi/Admin 已 frozen 为：

> operational client/boundary, not domain owner.

因此 Admin contract 应该独立 namespace，例如继续沿用现有 admin/backend convention，而不是让 admin 使用 Candidate-facing endpoint 加一个：

```text
?admin=true
```

更不能：

```text
ROLE_ADMIN → automatically bypass every domain rule
```

Admin endpoint 只暴露已经明确授权的：

- AI configuration
- feature toggles
- provider operational state
- AI operational metrics
- audit / diagnostic metadata

而不是 Candidate personal data browsing API。

------

# N. Contract 必须 identifier-/metadata-first

这和 3.16 observability 原则一致。

API response 只返回 capability 真正需要的数据。

特别不能因为“frontend 可能以后有用”就把：

```text
raw AI response
raw prompt
raw job HTML
internal event payload
provider diagnostic object
stack trace
secret/config value
```

塞进 response。

Internal operational metadata 与 product contract 分开。

------

# O. Contract evolution 原则

S3 内部 API evolution 默认遵循：

**additive-first**。

允许：

```text
add optional response field
add new endpoint
add new enum only where client contract explicitly tolerates unknown values
```

避免：

```text
rename field
change meaning
change type
remove field
silently change status semantics
```

因为现在已经存在至少：

```text
React Web
Browser Extension
RuoYi/Admin
```

三个 client boundary。

即使没有 public third-party API，contract stability 已经开始重要。

------

## 3.17.1 最终建议冻结

我建议把 **3.17.1 API Design Principles & Contract Boundaries** 定为：

> **OfferBuddy S3 exposes capability-oriented `/api/\**` HTTP contracts aligned with domain aggregates rather than database tables. Candidate-facing identity is derived exclusively from backend authentication context; internal surrogate IDs, persistence models, provider models and event models never cross the HTTP boundary. Candidate Profile is exposed as one aggregate with `profileVersion` as its concurrency boundary; Application remains an independently user-owned S2 Fast Path capability; Preparation remains Candidate + Job based and must not be nested under Application. Ordinary local operations are synchronous while durable external/AI workflows may expose explicit asynchronous command contracts. Browser Extension remains an untrusted client subject to the same authentication, authorization and validation rules, while RuoYi/Admin uses explicitly separated operational APIs and does not become a domain owner. Job identifiers never imply access. API evolution is additive-first, and HTTP contracts expose only capability-required business data rather than internal AI, event, diagnostic, secret or persistence details.**

我认为这一版和 **3.1–3.16 没有发现冲突**，而且给后面所有具体 endpoint 建立了足够明确的约束。

如果你认可，我们就把 **3.17.1 frozen**，下一步只进入 **3.17.2 Common Request / Response Contract**。





好，直接进入 **3.17.2 Common Request / Response Contract**。

这一节的目标不是发明一个“漂亮的统一包装层”，而是把所有 S3 API 都必须遵循的 HTTP contract 固定下来，尤其是：成功响应、错误响应、validation、pagination、时间格式、enum、requestId，以及 HTTP status mapping。

------

# 3.17.2 Common Request / Response Contract

## A. 不使用强制统一 success wrapper

我建议 **不要**把所有成功响应包成：

```json
{
  "code": 0,
  "message": "success",
  "data": {
  }
}
```

对于 OfferBuddy 这种 React + Spring Boot product API，这种 wrapper 没有明显收益，反而增加：

- frontend 解包层；
- OpenAPI schema 噪音；
- pagination 嵌套；
- HTTP status 与业务 `code` 双重语义；
- 空响应处理复杂度。

所以成功响应直接返回 resource / operation representation。

例如：

```json
{
  "candidateId": "87e57f14-...",
  "profileVersion": 7,
  "professionalSummary": "...",
  "skills": []
}
```

或者 collection：

```json
{
  "items": [],
  "page": 0,
  "size": 20,
  "totalElements": 42,
  "totalPages": 3
}
```

### Decision

> Successful APIs return their resource, collection or operation representation directly. OfferBuddy does not introduce a mandatory `{code, message, data}` success envelope.

------

# B. Error response 必须统一

错误则相反。

所有 API 应该拥有统一 machine-readable error contract。

推荐：

```json
{
  "code": "PROFILE_VERSION_CONFLICT",
  "message": "The candidate profile has changed.",
  "requestId": "01J...",
  "timestamp": "2026-08-24T10:45:12.381Z"
}
```

需要 field validation 时：

```json
{
  "code": "VALIDATION_FAILED",
  "message": "The request contains invalid fields.",
  "requestId": "01J...",
  "timestamp": "2026-08-24T10:45:12.381Z",
  "fieldErrors": [
    {
      "field": "email",
      "code": "INVALID_EMAIL",
      "message": "Enter a valid email address."
    }
  ]
}
```

这里我建议：

- `code`：稳定的 machine-readable contract；
- `message`：面向 client/user 的安全说明；
- `requestId`：对应 3.16；
- `timestamp`：错误发生时间；
- `fieldErrors`：仅 validation error 存在。

不返回：

```json
{
  "exception": "...",
  "stackTrace": "...",
  "sql": "...",
  "providerResponse": "..."
}
```

------

# C. Error `code` 是 contract，message 不是

这一点很重要。

Frontend 不应该写：

```javascript
if (error.message === "The candidate profile has changed.") {
}
```

应该依赖：

```text
PROFILE_VERSION_CONFLICT
```

因此：

> `code` has stable semantic meaning; `message` is presentation-supporting text and must not be used for application branching.

以后文案可以改，但 code 不应随意改。

例如：

```text
VALIDATION_FAILED
RESOURCE_NOT_FOUND
PROFILE_VERSION_CONFLICT
APPLICATION_VERSION_CONFLICT
IDEMPOTENCY_CONFLICT
OPERATION_NOT_ALLOWED
AI_FEATURE_DISABLED
```

具体 domain codes 在各 capability 小节再定。

------

# D. 不暴露 authorization difference

3.14 已经决定 ownership lookup 优先。

对于 Candidate / Preparation / Job 等 scoped resource：

如果：

- resource 不存在；
- resource 存在但不属于当前 user；

对普通 user API 默认不应暴露这种区别。

因此通常都：

```text
404 Not Found
```

而不是：

```text
403 → 我告诉攻击者这个 UUID 确实存在
```

真正的 role-level denial，例如：

```text
normal user calls admin endpoint
```

可以：

```text
403 Forbidden
```

### Decision

> Ownership-scoped resource absence and ownership mismatch normally collapse to 404. Explicit privilege/role denial may return 403.

------

# E. Validation 分成三层

我建议明确区分三种问题。

### 1. Syntactic / structural validation

例如：

```text
missing required field
invalid UUID
invalid email
string too long
invalid JSON
```

通常：

```text
400 Bad Request
```

Spring DTO / Bean Validation 处理。

------

### 2. Domain validation

请求 JSON 本身合法，但业务上不允许。

例如：

```text
resume generation requested before Profile is sufficiently complete
```

或者：

```text
cannot accept an already accepted import draft
```

这里不能都塞进 400。

如果是当前 resource state 与 command 不兼容，我建议通常：

```text
409 Conflict
```

因为请求本身格式没错，冲突来自当前 resource/business state。

------

### 3. Authorization validation

由 backend security context + ownership rules处理：

```text
401 / 403 / 404
```

不能伪装成 DTO validation。

------

# F. HTTP status 基线

我建议冻结以下 mapping。

| Situation                                                    | HTTP                                 |
| ------------------------------------------------------------ | ------------------------------------ |
| Successful GET                                               | `200`                                |
| Successful synchronous update                                | `200`                                |
| Resource created synchronously                               | `201`                                |
| Command accepted for async processing                        | `202`                                |
| Successful delete with no representation                     | `204`                                |
| Malformed / invalid request                                  | `400`                                |
| Missing / invalid authentication                             | `401`                                |
| Explicit privilege denied                                    | `403`                                |
| Missing or inaccessible owned resource                       | `404`                                |
| State/version/idempotency conflict                           | `409`                                |
| Unsupported media type                                       | `415`                                |
| Semantic validation that cannot reasonably be represented as state conflict | `422` only if specifically justified |
| Rate limiting                                                | `429`                                |
| Unexpected internal failure                                  | `500`                                |
| External dependency unavailable where request cannot be accepted durably | `502/503` depending on semantics     |

其中我特别建议：

**不要滥用 `422`.**

Spring ecosystem 和现有 OfferBuddy contract 可以主要靠：

```text
400
409
```

表达绝大多数 client/domain failure。

只有后面真的出现清晰语义才加 `422`。

------

# G. `401` 与 `403` 明确区别

`401 Unauthorized`：

> authentication missing / invalid / expired.

`403 Forbidden`：

> authenticated identity is valid, but lacks a required global/administrative privilege.

Ownership resource lookup：

> 通常 `404`。

这样 security semantics 很清楚。

------

# H. `requestId` contract

3.16 已 frozen：

> backend generates requestId per HTTP request and returns `X-Request-ID`.

因此每个 HTTP response：

```http
X-Request-ID: 01K...
```

包括：

```text
2xx
4xx
5xx
```

Error body 同时包含：

```json
{
  "requestId": "01K..."
}
```

成功 body **不需要**重复 `requestId`。

否则每个 DTO 都会被 operational concern 污染。

所以：

> `X-Request-ID` is the universal HTTP correlation surface; error bodies repeat it for support/debug usability. Successful business response models do not contain requestId.

`correlationId` 仍然 internal by default，不返回客户端。

------

# I. Timestamp 一律 ISO-8601 UTC

API timestamp 统一：

```text
2026-08-24T10:56:41.283Z
```

即：

- ISO-8601；
- UTC；
- `Z`；
- backend 不返回 local timezone dependent timestamp。

字段可以是：

```text
createdAt
updatedAt
generatedAt
completedAt
```

Frontend 自己转成 Sydney/local display。

不要返回：

```text
24/08/2026 20:56
```

也不要使用 epoch milliseconds 作为主要 contract。

### Decision

> API timestamps use ISO-8601 UTC instants. Local presentation belongs to clients.

------

# J. Date-only 与 timestamp 分开

如果某字段本质上只有日期，例如：

```text
employment start month/date
education completion date
```

则不要强行 timestamp。

例如：

```json
{
  "startDate": "2021-03-01"
}
```

如果 domain precision 只到 month，后面 Candidate contract 可以进一步定义：

```text
year + month
```

而不是偷偷用：

```text
2021-03-01T00:00:00Z
```

来伪装 precision。

这是数据真实性的一部分。

------

# K. JSON naming

统一使用：

```text
camelCase
```

例如：

```json
{
  "candidateId": "...",
  "profileVersion": 4,
  "createdAt": "...",
  "jobIntelligence": {}
}
```

不使用：

```text
snake_case
PascalCase
```

数据库 snake_case 不泄漏到 API。

------

# L. Null、missing 与 empty collection

这个最好现在就定，否则 frontend 很容易出现各种：

```text
undefined / null / []
```

混乱。

建议：

**Collection 永远返回 empty collection，不返回 null。**

例如：

```json
{
  "skills": [],
  "experiences": []
}
```

而不是：

```json
{
  "skills": null
}
```

Optional scalar 可以：

```json
{
  "phone": null
}
```

对于 update request：

`missing` 和 `null` 是否含义不同，要看具体 operation。

尤其 PUT / PATCH 后面必须明确，不能全局猜。

------

# M. S3 不随意引入 PATCH

Candidate Profile 等 aggregate 有：

```text
profileVersion
```

而且更新涉及完整业务一致性。

所以我倾向于：

```text
PUT /api/candidate/profile
```

作为明确 aggregate replacement/update contract。

不要为了 REST 时髦直接引入：

```text
PATCH
```

因为 PATCH 会马上引出：

- missing vs null；
- nested collection merge；
- skill delete；
- experience item merge；
- OCC semantics；

复杂度非常高。

对于独立 command 使用 POST。

### Decision

> S3 does not introduce generic JSON PATCH semantics. Resource updates use explicit PUT/update contracts or domain commands.

------

# N. Pagination 使用 page-based contract

现有 OfferBuddy Application List 已经使用 pagination，所以 S3 不需要另起 cursor model。

推荐 request：

```text
?page=0&size=20
```

response：

```json
{
  "items": [],
  "page": 0,
  "size": 20,
  "totalElements": 0,
  "totalPages": 0
}
```

约定：

- `page` zero-based；
- `size` backend 限制最大值；
- default `size` 例如 20；
- invalid negative page/size → `400`.

是否加：

```text
sort
```

按 endpoint individually define，不做任意字段 dynamic sorting。

避免：

```text
?sort=anythingUserSends
```

直接映射 DB column。

------

# O. Collection response 不直接返回 JSON array

我建议 paginated resource 永远返回 object：

```json
{
  "items": [],
  "page": 0,
  "size": 20,
  "totalElements": 0,
  "totalPages": 0
}
```

而不是：

```json
[
]
```

这样未来增加：

```text
filters
next
summary metadata
```

不会 breaking。

对于天然很小、不可分页的 nested collection，比如 profile skills，则直接 array 没问题。

------

# P. Enum contract 使用稳定 uppercase strings

推荐：

```json
{
  "status": "PROCESSING"
}
```

而不是：

```json
{
  "status": 2
}
```

也不要直接暴露数据库 ordinal。

例如：

```text
PENDING
PROCESSING
SUCCEEDED
FAILED
```

或者 Application 已存在的业务 status。

### Important

新增 enum value technically 可能破坏不健壮 client。

因此 client 应该：

- 不假设 enum 永远 exhaustive；
- 对未知值有 fallback。

但这不意味着 backend 可以随便改变语义。

------

# Q. Boolean 不用三态，除非 domain 真的是三态

如果字段是明确 yes/no：

```json
{
  "enabled": true
}
```

不要：

```text
"Y"
"1"
"enabled"
```

如果业务真的需要：

```text
YES / NO / UNKNOWN
```

那应该用 enum，而不是 nullable boolean 偷偷表达 UNKNOWN。

------

# R. Numeric contract

这些字段要保持明确类型：

```text
profileVersion → integer
application version → integer
contentVersion → integer
attemptCount → integer
latencyMs → integer/long
token counts → integer/long
```

money/cost 不用 binary floating point contract。

AI cost 后面 Admin contract 如需要，应明确：

```text
amount + currency
```

或者 decimal string/decimal representation，而不是模糊 `double`。

------

# S. Client 不提交 server-owned fields

Request DTO 不应该允许 client 设置：

```text
createdAt
updatedAt
candidateId
ownerUserId
eventId
correlationId
requestId
attemptCount
generatedAt
AI provider diagnostic data
```

如果 JSON 带了这些字段：

我们需要统一 Spring Jackson policy。

我建议 **request DTO 默认拒绝 unknown fields？**

这里我反而建议不要全局开启非常严格的 unknown-field rejection。

原因是 additive API evolution 时，旧/新 frontend rollout 可能有 compatibility friction。

更稳妥的是：

- 明确定义 DTO accepted fields；
- unknown properties 默认 ignore；
- security-sensitive server-owned property 根本不进入 DTO；
- semantic misuse 由 contract validation 防守。

这样 server-owned field 即使被提交也不会被绑定。

------

# T. Sensitive fields 不 echo

例如将来某 Admin operation：

```text
provider secret configured
```

请求中可能出现 credential。

成功 response 不能原样回：

```json
{
  "apiKey": "sk-..."
}
```

应只返回：

```json
{
  "configured": true
}
```

或者 masked metadata。

同样适用于 Candidate Import raw payload 等。

------

# U. Error message 不包含 sensitive/raw external data

例如 AI provider failure：

错误 API 可以：

```json
{
  "code": "AI_GENERATION_FAILED",
  "message": "The requested content could not be generated.",
  "requestId": "..."
}
```

不能：

```json
{
  "message": "OpenAI returned prompt rejected because the following resume text..."
}
```

Provider/debug details 走 3.16 structured diagnostics / audit，而不是 HTTP user contract。

------

# V. 建议的 common error schema

最终可以抽象为：

```text
ApiError
├── code
├── message
├── requestId
├── timestamp
└── fieldErrors[] optional

FieldError
├── field
├── code
└── message
```

这里不增加：

```text
path
exception
trace
correlationId
internalCode
provider
```

除非后面有非常明确的需求。

------

# 3.17.2 Proposed Frozen Decisions

我建议冻结为：

> **OfferBuddy S3 success responses expose resource, collection or operation representations directly and do not use a mandatory success envelope. Failures use one stable machine-readable `ApiError` contract containing `code`, safe `message`, `requestId`, UTC `timestamp`, and optional field-level validation errors. Business branching relies on stable error codes rather than human-readable messages. HTTP semantics follow standard status codes: 200/201/202/204 for success, 400 for malformed or structurally invalid requests, 401 for authentication failure, 403 for explicit privilege denial, 404 for absent or inaccessible ownership-scoped resources, and 409 for version, idempotency or business-state conflicts. Generic 422 usage is avoided unless a later capability demonstrates a clear need.**
>
> **All responses carry `X-Request-ID`; error bodies repeat the request ID while normal business DTOs do not. Correlation IDs remain internal. JSON uses camelCase; timestamps use ISO-8601 UTC; collections are empty rather than null; enums use stable string values rather than ordinals. Paginated APIs use zero-based page/size contracts and object-shaped collection responses. S3 does not introduce generic PATCH semantics. HTTP DTOs reject the persistence/entity boundary conceptually: server-owned identifiers, ownership data, audit fields, event data, AI diagnostics and secrets are neither client-controlled nor echoed back.**

我没有看到这套 contract 和 **3.1–3.16** 的 frozen decisions 有冲突。

下一步就是 **3.17.3 Candidate Profile API**。这一部分会开始真正落 endpoint、request/response DTO、`profileVersion` OCC，以及 profile 不存在时到底返回什么。



继续进入 **3.17.3 Candidate Profile API**。

这一节只落 Candidate Profile 的 HTTP contract。Candidate 的事实模型、ownership、`profileVersion` 语义、Resume Import 的事实建立规则，都沿用已经 frozen 的前序设计。

# 3.17.3 Candidate Profile API

## A. Resource identity

Candidate Profile 对当前登录用户是一个 singleton resource。

因此不需要：

```text
GET /api/candidates/{candidateId}/profile
```

也不需要客户端先知道 `candidateId`。

建议：

```text
GET /api/candidate/profile
PUT /api/candidate/profile
```

backend 从 security context 解析当前 user，再定位对应 Candidate。

这与 3.14 的 ownership model 一致。

------

## B. Profile 不存在时

这里建议明确：

```text
GET /api/candidate/profile
```

如果当前用户还没有 Candidate Profile：

```text
404 Not Found
```

error：

```json
{
  "code": "CANDIDATE_PROFILE_NOT_FOUND",
  "message": "Candidate profile was not found.",
  "requestId": "...",
  "timestamp": "..."
}
```

我不建议返回：

```json
{}
```

或者：

```json
{
  "exists": false
}
```

因为 profile 本身就是 resource；不存在就是 resource absence。

Frontend 如果需要 onboarding 状态，可以正常根据 404 判断。

------

## C. Profile 创建与更新

这里有两个可选方向：

```text
POST /api/candidate/profile
PUT  /api/candidate/profile
```

或者让：

```text
PUT /api/candidate/profile
```

同时支持 create/update。

我建议第二种。

原因是 Candidate Profile 是当前用户的 singleton、URL 是确定的，而且客户端表达的是：

> make my Candidate Profile reflect this accepted set of facts.

因此：

```text
PUT /api/candidate/profile
```

可以：

- profile 不存在 → create；
- profile 已存在 → update。

成功时：

- create → `201 Created`
- update → `200 OK`

这样也避免一个意义不大的 collection-style `POST`.

------

# D. `profileVersion` 的 request contract

这里必须落实 3.15 optimistic concurrency。

首次创建：

```json
{
  "profileVersion": null,
  "...": "..."
}
```

不过我更建议：

**create request 根本不要求 `profileVersion`。**

update 则必须携带：

```json
{
  "profileVersion": 7,
  ...
}
```

问题在于同一个 DTO 怎么区分 create/update。

最清晰的规则是：

- resource currently absent：`profileVersion` 必须 omitted/null；
- resource currently exists：`profileVersion` 必须提供并匹配；
- server 决定 create 还是 update；
- client 不能指定任意新 version。

如果现有 profile version 为 7：

```text
request profileVersion = 7
```

成功后：

```text
response profileVersion = 8
```

如果客户端提交 6：

```text
409 Conflict
{
  "code": "PROFILE_VERSION_CONFLICT",
  "message": "The candidate profile has changed. Refresh it before saving.",
  "requestId": "...",
  "timestamp": "..."
}
```

绝不自动覆盖。

------

# E. 为什么不使用 `If-Match`

理论上 HTTP OCC 可以使用：

```text
ETag
If-Match
```

但对当前 OfferBuddy，我建议不引入。

因为我们已经有明确 domain revision：

```text
profileVersion
```

而且这个 version 不只是 technical HTTP cache version，它还是：

- meaningful fact revision；
- concurrency token；
- derived artifact provenance revision。

所以把它作为业务 DTO 的显式字段更直观，也更容易被 React、AI derived output 和 diagnostics 使用。

------

# F. Response shape

Candidate Profile response 建议体现 aggregate：

```json
{
  "candidateId": "uuid",
  "profileVersion": 8,

  "personalDetails": {
    "fullName": "...",
    "email": "...",
    "phone": "...",
    "location": "..."
  },

  "professionalSummary": "...",

  "skills": [],

  "experiences": [],

  "education": [],

  "certifications": [],

  "languages": [],

  "eligibility": {},

  "createdAt": "...",
  "updatedAt": "..."
}
```

这里的重点不是字段最终名字现在一次定死，而是 contract shape：

> Profile 是一个 aggregate response，而不是多个独立 endpoint 拼装。

------

# G. `candidateId` 返回，但不能提交

Response 可以返回：

```text
candidateId
```

因为它是合法 domain identifier，后续 Preparation provenance 等可能需要引用。

但是 update request 不接受：

```text
candidateId
```

ownership 和 target resource 永远由当前 authenticated context 决定。

------

# H. Personal/contact facts

Candidate Profile 存储用户事实，所以 request 中可以提交：

```text
fullName
email
phone
location
```

但需要区分：

> Candidate contact email ≠ authentication account email.

即使二者最初一样，也不要在 contract 层把 Profile email 当作 login identity。

否则未来：

- Google account email
- preferred contact email

会耦合。

这和 Identity / Candidate module separation 一致。

------

# I. Experiences 是 nested aggregate data

Experience contract 可以类似：

```json
{
  "experienceId": "uuid",
  "company": "...",
  "title": "...",
  "location": "...",
  "startDate": "...",
  "endDate": null,
  "current": true,
  "items": [
    {
      "experienceItemId": "uuid",
      "content": "..."
    }
  ]
}
```

这里建议 child item 也有 stable domain ID。

原因不是把它们变成独立资源，而是为了：

- React list identity；
- edit/delete reconciliation；
- Resume Import merge；
- future AI provenance；
- diff/audit。

但：

```text
experienceId
experienceItemId
```

仍只是 aggregate child identifiers。

客户端不能通过：

```text
/api/experiences/{id}
```

独立访问它们。

------

# J. 新增 child item 的 ID

对于新 skill / experience / education 等 child：

我建议由 backend 生成 ID。

Create/update request 中：

- existing child → 带现有 child ID；
- new child → omit ID；
- backend 分配 ID。

不要让 browser 生成 authoritative UUID，除非以后 offline-first 有明确需要。

当前没有这个需求。

------

# K. Aggregate replacement semantics

这是这一节最关键的 contract 之一。

因为我们选择：

```text
PUT /api/candidate/profile
```

所以 request 中的 child collections 应按：

> authoritative submitted aggregate state

理解。

例如数据库现在：

```text
skills:
- Java
- Spring
- MySQL
```

请求：

```json
{
  "skills": [
    "Java",
    "Spring"
  ]
}
```

那么 MySQL 被删除。

不是：

> merge whatever is provided.

这让 semantics 很清晰。

同时这也是为什么 `profileVersion` 必须存在：整个 aggregate replacement 必须有 OCC protection。

------

# L. 不允许 silent normalisation 改变事实

Backend 可以做轻量 structural normalisation：

```text
trim whitespace
normalise obvious empty string → null where contract says optional
```

但是不能：

```text
"Java developer" → "Senior Java Engineer"
```

或者：

```text
"English: intermediate" → "English: fluent"
```

Candidate Profile 是 factual source。

AI 也不能在普通 PUT 时“顺便优化”。

任何 AI proposal 必须走 Resume Import / explicit review workflow。

------

# M. Sufficiency 不阻止 Profile 保存

前面已经 frozen：

> Profile optional globally; capability-specific sufficiency only.

所以：

```text
PUT /api/candidate/profile
```

不能因为用户没有：

- professional summary；
- certifications；
- languages；
- enough work experience；

就拒绝保存。

只进行：

- structural validation；
- factual consistency validation；
- required-within-field validation。

例如 experience：

```text
startDate > endDate
```

可以拒绝。

但：

```text
"You don't have enough skills"
```

不能作为 Profile save failure。

------

# N. Capability sufficiency 单独表达

对于 Resume/Match 等 capability，需要判断 Profile 是否足够。

不要把这些 flags 全塞进 Profile persistence semantics。

但 response 可以考虑返回一个 derived, non-authoritative hint，例如：

```json
{
  "capabilities": {
    "matchReady": true,
    "resumeReady": false,
    "coverLetterReady": true
  }
}
```

不过我建议 **3.17.3 暂时不要加**。

原因是这些 readiness rules 属于 Preparation capability，而且未来可能变化。

更干净的做法是由具体 Preparation command validation 返回 domain-specific reason。

避免 Candidate Profile API 开始依赖所有下游 capability。

------

# O. Delete Candidate Profile

S3 我建议不开放：

```text
DELETE /api/candidate/profile
```

至少不作为普通 product capability。

因为删除 Candidate Profile 会牵涉：

- derived Match;
- generated Resume;
- Cover Letter;
- provenance;
- privacy/data deletion;
- historical preparation references.

这不是一个简单 CRUD delete。

如果未来支持 account/data deletion，应由专门 privacy/account deletion workflow 管理，而不是普通 Candidate endpoint。

------

# P. Partial child CRUD 不开放

因此 S3 不提供：

```text
POST   /api/candidate/skills
DELETE /api/candidate/skills/{skillId}

POST   /api/candidate/experiences
PATCH  /api/candidate/experiences/{id}
```

全部通过：

```text
PUT /api/candidate/profile
```

保存 aggregate。

对于 MVP/S3，这会显著降低：

- endpoint 数量；
- OCC complexity；
- partial stale state；
- frontend coordination complexity。

------

# Q. Response freshness

成功 update response 应返回 **完整最新 Profile representation**，而不是：

```json
{
  "profileVersion": 8
}
```

这样 frontend 可直接替换本地 state。

典型流程：

```text
GET profile
   ↓
edit locally
   ↓
PUT profileVersion=7
   ↓
200 full profileVersion=8
   ↓
replace client state
```

很干净。

------

# R. Profile validation error example

例如：

```json
{
  "profileVersion": 7,
  "experiences": [
    {
      "startDate": "2025-01-01",
      "endDate": "2024-01-01"
    }
  ]
}
```

返回：

```text
400 Bad Request
{
  "code": "VALIDATION_FAILED",
  "message": "The request contains invalid fields.",
  "requestId": "...",
  "timestamp": "...",
  "fieldErrors": [
    {
      "field": "experiences[0].endDate",
      "code": "END_DATE_BEFORE_START_DATE",
      "message": "End date must not be before start date."
    }
  ]
}
```

这里不需要发明 domain exception hierarchy 给 frontend。

------

# S. Stale write 不返回最新 Profile

当 OCC conflict 时，我建议：

```text
409 PROFILE_VERSION_CONFLICT
```

但 response **不要直接附带服务器最新 Profile**。

原因：

如果服务器自动返回最新 resource，很容易诱导 frontend 做 automatic merge。

而 3.15 已 frozen：

> stale writes fail explicitly and are never silently merged.

正确 UI flow：

```text
409
 ↓
show conflict
 ↓
GET current profile
 ↓
user reviews/reconciles
```

这是更安全的事实管理方式。

------

# T. Proposed endpoints

最终这一小节可以冻结为只有：

```text
GET /api/candidate/profile
PUT /api/candidate/profile
```

非常克制。

Resume Import 是下一节单独 capability，不塞进这里。

------

# 3.17.3 Proposed Frozen Decisions

我建议冻结：

> **Candidate Profile is exposed as a current-user singleton resource through `GET /api/candidate/profile` and `PUT /api/candidate/profile`. User identity and Candidate ownership are derived exclusively from backend security context; `candidateId` may appear in responses but is never accepted as an ownership selector. Profile absence returns `404 CANDIDATE_PROFILE_NOT_FOUND`. PUT operates as an aggregate create/update contract: absence creates the profile and returns 201, while an existing profile requires its current `profileVersion`, performs optimistic concurrency validation, increments the meaningful profile revision on successful factual change, and returns the complete updated representation. Stale versions fail with `409 PROFILE_VERSION_CONFLICT` and are never automatically merged or overwritten.**
>
> **Candidate Profile remains one aggregate contract covering personal/contact facts, professional summary, skills, experiences and experience items, education, certifications, languages and eligibility. Persistence child tables are not exposed as independent HTTP resources. Existing aggregate children carry stable domain identifiers; new children may omit IDs and receive backend-generated identifiers. Submitted child collections represent the intended aggregate state rather than patch/merge fragments. Profile saving validates structure and factual consistency but does not require downstream Match/Resume/Cover-Letter sufficiency. AI-generated proposals cannot silently alter Candidate facts, generic PATCH is not introduced, and ordinary profile deletion is not exposed in S3.**

这一版也保持了 Candidate 作为 **truthful single source of truth** 的核心原则。

下一步进入 **3.17.4 Resume Import API**。这里会重点解决：

```
upload existing resume → extraction/import draft → user review → explicit acceptance → Candidate Profile
```

以及其中哪些步骤应该同步、哪些应该走 `202 + business_events`。



进入 **3.17.4 Resume Import API**。

这一节的核心不是“上传文件接口”本身，而是把已经 frozen 的事实建立边界真正落成 HTTP contract：

> **uploaded resume → controlled extraction → CandidateImportDraft → user review → explicit acceptance → Candidate Profile**

最重要的原则仍然是：

**AI 可以提出 Candidate facts，但不能因为“导入成功”就直接写入 Candidate Profile。**

------

# 3.17.4 Resume Import API

## A. Resume Import 是独立 Candidate capability

我建议 namespace：

```text
POST /api/candidate/resume-imports
GET  /api/candidate/resume-imports/{importId}
POST /api/candidate/resume-imports/{importId}/accept
```

暂时不把它设计成：

```text
/api/candidate/profile/import
```

因为 Resume Import 本身有：

- 独立生命周期；
- async processing；
- failure state；
- AI extraction result；
- review state；
- acceptance state。

它已经是一个真正的 workflow resource，而不是一次简单 Profile mutation。

------

# B. 创建 Import 使用 multipart upload

用户导入的是已有 resume 文件，因此建议：

```text
POST /api/candidate/resume-imports
Content-Type: multipart/form-data
```

核心 part：

```text
file
```

S3 不需要客户端同时提交：

```text
candidateId
userId
profileVersion
provider
prompt
```

身份全部来自 backend security context。

AI provider 由 AI Platform/router 决定，不属于用户-facing contract。

------

# C. 文件类型范围必须显式限制

我建议 S3 只接受明确支持的 resume formats，例如：

```text
PDF
DOCX
```

而不是“任何文件都试试看”。

拒绝：

```text
exe
zip
image bundle
arbitrary binary
```

Unsupported format：

```text
415 Unsupported Media Type
```

例如：

```json
{
  "code": "UNSUPPORTED_RESUME_FORMAT",
  "message": "The uploaded resume format is not supported.",
  "requestId": "...",
  "timestamp": "..."
}
```

文件过大则：

```text
413 Payload Too Large
```

如果现有 global upload infrastructure 已有 size limit，就复用，不另造 Resume Import 专属机制。

------

# D. Upload request 本身不等待 AI 完成

这里我建议明确使用 async。

流程：

```text
POST upload
   ↓
validate authentication / file
   ↓
persist import workflow state
   ↓
persist durable business_event in same short transaction
   ↓
return 202
   ↓
worker extracts / invokes AI
```

原因和 3.13 完全一致：

- document parsing 可能慢；
- AI latency 不稳定；
- provider failure 需要 retry；
- HTTP request 不应该长时间阻塞；
- import workflow 需要 crash recovery。

因此：

```text
POST /api/candidate/resume-imports
```

成功创建后：

```text
202 Accepted
```

而不是等 extraction 完成后才 `201`.

------

# E. Initial response

建议返回：

```json
{
  "importId": "uuid",
  "status": "PROCESSING",
  "createdAt": "2026-08-24T11:10:00Z"
}
```

也可以初始状态先是：

```text
PENDING
```

但我建议 HTTP-facing lifecycle 尽量表达产品状态，而不是暴露 `business_events` 内部状态。

例如 Resume Import API status：

```text
PROCESSING
READY_FOR_REVIEW
ACCEPTED
FAILED
```

而内部 business event 仍然：

```text
PENDING
PROCESSING
RETRY
SUCCEEDED
FAILED
```

两者不要机械一一映射。

------

# F. Import lifecycle

我建议 S3 对外只暴露：

```text
PROCESSING
READY_FOR_REVIEW
ACCEPTED
FAILED
```

意义：

### `PROCESSING`

Backend 正在进行：

- secure extraction；
- text normalisation；
- AI structured extraction；
- result validation。

用户不能 accept。

### `READY_FOR_REVIEW`

Import draft 已经生成。

但：

> Candidate Profile 还没有变化。

### `ACCEPTED`

用户已经显式确认一次 acceptance operation。

Candidate Profile 已通过正式 Candidate mutation transaction 更新。

### `FAILED`

Import processing 不能产生可 review draft。

------

# G. 不暴露 RETRY 状态

内部可能发生：

```text
business_event:
RETRY
attemptCount = 2
```

但用户-facing API 没必要看到：

```text
RETRY
RETRY_WAITING
PROVIDER_TIMEOUT
```

用户只需要看到：

```text
PROCESSING
```

直到：

```text
READY_FOR_REVIEW
```

或最终：

```text
FAILED
```

Operational detail 由 3.16 observability/Admin 处理。

------

# H. GET Import Status / Draft

```text
GET /api/candidate/resume-imports/{importId}
```

`PROCESSING` 时：

```json
{
  "importId": "uuid",
  "status": "PROCESSING",
  "createdAt": "...",
  "updatedAt": "..."
}
```

`READY_FOR_REVIEW` 时：

```json
{
  "importId": "uuid",
  "status": "READY_FOR_REVIEW",
  "draft": {
    "personalDetails": {},
    "professionalSummary": "...",
    "skills": [],
    "experiences": [],
    "education": [],
    "certifications": [],
    "languages": [],
    "eligibility": {}
  },
  "createdAt": "...",
  "updatedAt": "..."
}
```

这里 `draft` 是：

> proposed Candidate facts

不是 Candidate Profile representation。

------

# I. Draft 与 Profile contract 应相似，但不能复用同一个 DTO

这点值得明确。

它们可以在结构上相似，但不能直接：

```text
CandidateProfileResponse
```

兼作：

```text
CandidateImportDraft
```

因为两者语义完全不同：

```text
CandidateProfile
= accepted authoritative facts

CandidateImportDraft
= proposed untrusted/unaccepted facts
```

所以 application/domain type 必须分开。

否则未来开发人员很容易误用：

```java
candidateProfileRepository.save(importDraft)
```

这正是我们要防止的边界错误。

------

# J. AI confidence 不应成为“事实真实性”

可以考虑 AI extraction 中存在内部 confidence。

但 S3 user-facing contract 我建议不要大面积返回：

```text
confidence: 0.92
```

原因是：

- provider confidence 未必校准；
- 0.92 很容易让用户误认为“92%真实”；
- Candidate facts 最终仍需用户确认。

如果确实需要帮助 review，更好的方向是后续增加：

```text
requiresReview
sourceSection
```

但 S3 可以保持简单：

> 用户 review 整个 draft。

------

# K. 原始 resume 内容不返回

GET import 不能返回：

```text
rawText
rawHtml
base64 file
AI prompt
AI raw response
```

即使 backend 处理过程中曾暂时存在。

3.14 / 3.16 的 privacy 和 telemetry 边界继续成立。

如果 UI 未来需要让用户重新查看上传文件，那应该是独立 secure file-access capability，不通过 import JSON response 塞进去。

S3 暂时不需要。

------

# L. User review 必须提交显式 accepted state

这里有两种设计。

第一种：

```text
POST /accept
```

body 只：

```json
{
  "profileVersion": 4
}
```

意思是“接受 AI draft 原样”。

第二种：

用户可以在 review UI 修改 draft，再 accept：

```json
{
  "profileVersion": 4,
  "acceptedProfile": {
    ...
  }
}
```

我强烈建议第二种。

因为真实 workflow 应该是：

```text
AI proposes
 ↓
user reviews
 ↓
user corrects / removes / adds
 ↓
accept
```

否则如果用户发现 AI 抽取错了一行：

> 他还得先 accept 错误事实，再去 Profile 编辑。

这违背了我们设计 Resume Import 的初衷。

------

# M. Acceptance body

因此建议：

```text
POST /api/candidate/resume-imports/{importId}/accept
```

body：

```json
{
  "profileVersion": 4,
  "profile": {
    "personalDetails": {},
    "professionalSummary": "...",
    "skills": [],
    "experiences": [],
    "education": [],
    "certifications": [],
    "languages": [],
    "eligibility": {}
  }
}
```

这里的 `profile` 表示：

> the complete factual Candidate Profile state the user explicitly accepts after reviewing the import.

这意味着 acceptance 不是：

> “把 draft merge 进去。”

而是：

> “我确认这是我希望 Candidate Profile 成为的事实集合。”

------

# N. Existing Profile merge 应在 UI review 阶段完成

这是一个很关键的决定。

假设当前 Profile 已有：

```text
Java
Spring
AWS
```

Resume draft 抽出了：

```text
Java
Spring Boot
MySQL
```

backend **不应该偷偷决定**：

```text
union everything
```

也不能自动：

```text
replace everything with draft
```

正确流程应该是：

```text
existing Candidate Profile
        +
Resume Import Draft
        ↓
frontend review / merge UI
        ↓
explicit accepted complete profile
        ↓
accept
```

因此 authoritative merge decision 属于用户。

Backend 只做：

- validation；
- OCC；
- persistence。

------

# O. GET Import 可以辅助返回 base profile provenance

为了让 frontend 知道这个 draft 是基于哪个 Profile 状态开始的，我建议 Import resource 保存并返回：

```text
baseProfileVersion
```

例如：

```json
{
  "importId": "...",
  "status": "READY_FOR_REVIEW",
  "baseProfileVersion": 4,
  "draft": {}
}
```

这很有价值。

因为用户可能：

```text
upload resume at profileVersion 4
 ↓
while AI processes
 ↓
edit profile elsewhere
 ↓
profileVersion becomes 5
 ↓
resume import becomes READY
```

这样 UI 能知道：

> 这个 draft 的 review context 起始于 version 4。

------

# P. Acceptance 必须执行 current `profileVersion` OCC

即使：

```text
baseProfileVersion = 4
```

accept 时仍必须提交客户端当前 review 所基于的：

```text
profileVersion
```

如果 DB 已经是 5，但 client 提交 4：

```text
409 PROFILE_VERSION_CONFLICT
```

不能：

> “反正这是 Resume Import，就覆盖掉。”

Resume Import 绝不能绕过 Candidate Profile OCC。

------

# Q. Profile 不存在时的 acceptance

如果用户还没有 Candidate Profile：

Import 可以照样开始。

这符合：

> Resume Import can be Candidate onboarding capability.

此时：

```text
baseProfileVersion = null
```

review 后：

```text
POST /accept
```

body 中：

```text
profileVersion omitted/null
```

如果此时 Profile 仍不存在：

- Candidate Profile created；
- initial meaningful version established；
- import → `ACCEPTED`.

如果 processing 期间另一个 workflow 已创建了 Profile：

那么 accept 不能 silent overwrite。

应该：

```text
409 PROFILE_VERSION_CONFLICT
```

要求重新 review against current profile。

------

# R. Acceptance transaction 必须原子化

`POST /accept` 不需要 AI。

它应是短 local PostgreSQL transaction：

```text
lock/check import state
check ownership
check current profileVersion
validate submitted profile
persist Candidate Profile
increment profileVersion
mark import ACCEPTED
commit
```

必须保证：

> 不会出现 Profile 已更新，但 Import 还是 READY_FOR_REVIEW；

也不能：

> Import ACCEPTED，但 Profile transaction rolled back。

这符合 3.15 的 local strong consistency。

------

# S. Acceptance 应同步返回

因为 accept 只是 local transaction，不涉及 external/AI call。

所以：

```text
POST /accept
```

成功：

```text
200 OK
```

返回最新 Candidate Profile representation，例如：

```json
{
  "candidateId": "...",
  "profileVersion": 5,
  ...
}
```

我不建议再返回：

```json
{
  "importStatus": "ACCEPTED",
  "profile": {}
}
```

因为客户端真正关心的 authoritative result 是 Candidate Profile。

Import 状态之后 GET 也可看到 `ACCEPTED`。

------

# T. Acceptance 只能执行一次

如果 import 已：

```text
ACCEPTED
```

再次：

```text
POST /accept
```

不应该再次修改 Candidate Profile。

这里我们要考虑 idempotency semantics。

我建议：

如果已经 ACCEPTED，并且这是同一个 import：

```text
409 RESUME_IMPORT_ALREADY_ACCEPTED
```

而不是默默再执行。

为什么不是简单 200 idempotent？

因为第二次请求可能来自：

- stale UI；
- double click；
- client retry。

真正的 transport retry/idempotency 可以由 3.17.13 的 Idempotency-Key contract 处理。

业务 command 本身应该明确：

> import has already been consumed.

------

# U. Processing 时不能 accept

如果：

```text
status = PROCESSING
```

调用 `/accept`：

```text
409 RESUME_IMPORT_NOT_READY
```

如果：

```text
FAILED
```

调用：

```text
409 RESUME_IMPORT_NOT_READY
```

或者更具体：

```text
RESUME_IMPORT_FAILED
```

我倾向保留一个：

```text
RESUME_IMPORT_NOT_READY
```

避免不必要的 error code proliferation。

------

# V. Failed response

例如：

```json
{
  "importId": "...",
  "status": "FAILED",
  "failure": {
    "code": "RESUME_EXTRACTION_FAILED",
    "message": "The resume could not be processed."
  },
  "createdAt": "...",
  "updatedAt": "..."
}
```

这里可以返回一个安全 product-level failure code。

不能返回：

```text
OpenAI 429
Tika stacktrace
Apache POI exception
provider raw response
```

这些只进入 diagnostics。

------

# W. Failure code 应保持粗粒度

建议 user-facing 最多类似：

```text
RESUME_EXTRACTION_FAILED
UNSUPPORTED_RESUME_CONTENT
AI_EXTRACTION_FAILED
```

甚至可以更克制，只留：

```text
RESUME_IMPORT_FAILED
```

我建议 S3 先保持：

```text
RESUME_IMPORT_FAILED
```

加 safe message。

原因是用户通常没有办法根据 provider failure 采取不同操作。

Operationally meaningful detail 放 Admin/observability。

------

# X. Retry 设计

这里我建议 **S3 不提供用户手动 retry endpoint**：

```text
POST /resume-imports/{id}/retry
```

内部已经有 3.13 automatic retry。

如果最终 FAILED：

用户可以重新：

```text
POST /api/candidate/resume-imports
```

创建新的 import attempt。

这样生命周期简单得多。

------

# Y. Import history

S3 暂时不需要：

```text
GET /api/candidate/resume-imports
```

做完整历史列表。

目前 UI workflow 只需要知道刚创建的 `importId`。

所以最小 contract 保持：

```text
POST create
GET one
POST accept
```

未来如果产品真的要显示 Import History，再加 collection endpoint。

------

# Z. Polling

Frontend 在收到：

```text
202 Accepted
```

后可以 polling：

```text
GET /api/candidate/resume-imports/{importId}
```

直到：

```text
READY_FOR_REVIEW
FAILED
```

S3 不需要为了这个功能引入：

```text
WebSocket
SSE
```

这些基础设施在当前阶段没有必要。

Poll interval 可以由 frontend 合理控制，例如数秒级，而不是高频请求。

------

# AA. Import 与 source file lifetime

这里我建议 contract 层明确：

> uploaded resume file is an input artefact, not Candidate Profile truth.

因此：

- file storage lifecycle；
- extracted transient content；
- retention period；

属于 privacy/storage implementation policy，而不作为 Candidate API model。

如果我们后面在 implementation design 需要确定 retention，可以单独落 operational detail，但 API 不承诺“永久保存原始 resume”。

这对隐私也更安全。

------

# AB. Candidate fact provenance

接受后 Candidate Profile 的事实来源从语义上是：

```text
USER_ACCEPTED_RESUME_IMPORT
```

而不是：

```text
AI
```

因为真正建立事实的是：

> user acceptance.

如果 DB 设计里以后有 provenance metadata，则应记录：

```text
importId
accepted by user
source = resume import
```

但这些不需要全部返回 Candidate-facing response。

这和前面 frozen 的：

> AI may propose but cannot silently establish facts

完全一致。

------

# AC. Request ownership

所有：

```text
GET /resume-imports/{importId}
POST /resume-imports/{importId}/accept
```

都必须 authorised lookup。

如果 import：

- 不存在；
- 属于其他 user；

统一：

```text
404
```

不泄漏存在性。

------

# AD. Suggested HTTP flow

完整 API workflow：

```text
1. POST /api/candidate/resume-imports
   multipart file
        ↓
   202 Accepted
   {
     importId,
     status: PROCESSING
   }

2. GET /api/candidate/resume-imports/{importId}
        ↓
   200
   PROCESSING

3. GET ...
        ↓
   200
   READY_FOR_REVIEW
   {
     baseProfileVersion,
     draft
   }

4. User reviews / edits / merges

5. POST /api/candidate/resume-imports/{importId}/accept
   {
     profileVersion,
     profile: complete accepted profile
   }
        ↓
   200 latest CandidateProfile
```

如果并发修改：

```text
step 5
 ↓
409 PROFILE_VERSION_CONFLICT
 ↓
GET current profile
 ↓
re-review / reconcile
```

非常符合我们整个 S3 的 consistency model。

------

# 3.17.4 Proposed Frozen Decisions

我建议冻结为：

> **Resume Import is a separate Candidate workflow resource exposed through `POST /api/candidate/resume-imports`, `GET /api/candidate/resume-imports/{importId}`, and `POST /api/candidate/resume-imports/{importId}/accept`. Creation accepts only supported resume file formats through multipart upload, derives identity from backend authentication context, persists the import work and corresponding durable business event transactionally, and returns `202 Accepted` without waiting for extraction or AI processing. User-facing lifecycle states are limited to product-level states such as `PROCESSING`, `READY_FOR_REVIEW`, `ACCEPTED`, and `FAILED`; internal event retry/provider states remain operational details.**
>
> **A completed import produces a `CandidateImportDraft`, not Candidate facts. The draft is semantically and structurally separate from the authoritative Candidate Profile even where their fields are similar. Raw uploaded content, extracted text, AI prompts, provider responses and internal diagnostics are never returned through the Candidate-facing import contract. Resume Import may operate when no Candidate Profile exists and records the profile revision, if any, against which the review began.**
>
> **Acceptance is an explicit synchronous user command. The client submits the complete factual Candidate Profile state it has reviewed and accepted, together with the relevant current `profileVersion` when a Profile exists. Merge decisions between the current Profile and AI-proposed draft belong to the user/review UI; the backend never silently unions, replaces or auto-merges Candidate facts. Acceptance performs ownership checks, profile optimistic concurrency validation, factual validation, Candidate Profile persistence/version advancement, and import transition to `ACCEPTED` atomically in one local transaction. A stale profile revision fails with `409 PROFILE_VERSION_CONFLICT`; a non-ready or already-consumed import cannot modify Candidate facts. Automatic processing retries remain internal, and S3 does not add manual retry, WebSocket/SSE, or import-history APIs unless later product requirements justify them.**

这版和 **3.13 async、3.14 AI data boundary、3.15 OCC、3.16 diagnostics** 是完全对齐的。

下一步是 **3.17.5 Job & Job Intelligence API**。这部分最重要的是把“canonical Job 可以独立于 Application 存在”“Job 不 globally public”“contentVersion 只表示语义变化”“Job Intelligence 是 derived data”几个决定统一到 HTTP contract。



进入 **3.17.5 Job & Job Intelligence API**。

这一节要把前面已经冻结的几个关键事实真正落实到 HTTP contract：

- canonical Job 可以在没有 Application 的情况下存在；
- Job 不是 globally public resource；
- `contentVersion` 只在语义内容发生变化时递增；
- Job Intelligence 是 derived data，不是 Job 原始事实；
- Preparation 依赖 Candidate + Job；
- Browser Extension 可以帮助建立 Job，但不能绕过 authorization / validation。

------

# 3.17.5 Job & Job Intelligence API

## A. Job 是独立 canonical resource

S3 里 Job 不再只是 Application 创建过程里的附属对象。

因此应该允许：

```text
POST /api/jobs
GET  /api/jobs/{jobId}
```

而不是只能通过：

```text
POST /api/applications
```

间接产生 Job。

这是 Candidate + Job Preparation Path 能独立存在的前提。

------

# B. 创建 Job 不要求 Application

例如用户在 LinkedIn/SEEK/Indeed 上看到一个岗位，想先做 Match / Resume / Cover Letter，但还没有申请。

此时可以：

```text
POST /api/jobs
```

成功建立 canonical Job。

之后再进入：

```text
Preparation
```

而 Application 仍然是独立 S2 Fast Path。

所以明确：

> Creating a Job does not create an Application.

同样：

> Creating an Application does not automatically mean Preparation exists.

两条路径保持解耦。

------

# C. Job creation contract

建议 request 只提交 canonical job facts/input，例如：

```json
{
  "source": {
    "platform": "LINKEDIN",
    "externalJobId": "123456",
    "url": "https://..."
  },
  "title": "Software Engineer",
  "companyName": "Example Pty Ltd",
  "location": "Sydney NSW",
  "description": "...",
  "employmentType": "FULL_TIME"
}
```

具体字段可结合现有 S2 Job schema 最终落地，但 contract 原则是：

> 客户端提交的是 Job business content，不是 persistence metadata。

不能提交：

```text
jobId
contentVersion
createdAt
updatedAt
ownerUserId
intelligenceStatus
AI provider
correlationId
```

这些由 backend 管理。

------

# D. Job URL / platform metadata 不是 ownership

这一点要特别明确。

例如：

```text
platform = LINKEDIN
externalJobId = 12345
```

只是 source identity / deduplication metadata。

它不能意味着：

> 所有知道这个 LinkedIn job 的 OfferBuddy 用户都可以访问同一个 Job resource。

S3 的 Job authorization 仍然必须通过当前用户合法 relationship。

------

# E. Canonical Job 与用户 relationship 分开理解

数据库中 canonical Job 可以复用或去重，但 HTTP authorization 不应暴露这种内部共享性。

也就是说，backend 可以内部判断：

```text
same platform + same externalJobId
```

已经存在 canonical Job。

但 client contract 不需要知道：

> “这是别人之前创建的 Job。”

成功 response 只给当前 workflow 已授权的 Job representation。

因此 deduplication 是 backend concern，不是 access-control shortcut。

------

# F. `POST /api/jobs` 的成功语义

这里有两种情况：

### 新 canonical Job

返回：

```text
201 Created
```

### 已存在相同 canonical Job

不建议返回：

```text
409 JOB_ALREADY_EXISTS
```

因为用户并没有做错事。

如果 backend 根据 canonical identity 发现已有 Job，可以：

- 建立/确认当前 user 的合法 workflow relationship；
- 返回现有 Job representation。

这里可以返回：

```text
200 OK
```

或者统一做 idempotent create semantics。

我建议：

> 新建时 `201`，复用已有 canonical Job 时 `200`。

这样比较准确。

------

# G. 但“复用”不能凭裸 Job lookup

复用 canonical Job 时 backend 必须同时确保当前 workflow 已合法建立 relationship。

例如：

```text
Candidate + Job Preparation relationship
```

或者对应 Application relationship。

不能：

```text
POST /api/jobs
→ find existing job
→ directly expose it
```

而没有授权关系。

------

# H. `GET /api/jobs/{jobId}`

返回 canonical Job business representation，例如：

```json
{
  "jobId": "uuid",
  "contentVersion": 3,
  "source": {
    "platform": "LINKEDIN",
    "url": "https://..."
  },
  "title": "Software Engineer",
  "companyName": "Example Pty Ltd",
  "location": "Sydney NSW",
  "description": "...",
  "employmentType": "FULL_TIME",
  "createdAt": "...",
  "updatedAt": "..."
}
```

但必须 authorised lookup。

如果：

- Job 不存在；
- Job 存在但当前 user 没有合法 Application/Preparation relationship；

都：

```text
404 Not Found
```

统一 error，例如：

```text
JOB_NOT_FOUND
```

------

# I. Job 不提供 global list/search API

S3 不应该出现：

```text
GET /api/jobs
GET /api/jobs/search
```

作为所有 canonical Jobs 的浏览接口。

因为已经 frozen：

> Job is not globally public.

用户寻找自己的 Application Job，应通过 Application capability。

用户寻找自己的 Preparation Job，应通过 Preparation workflow。

Job API 只负责具体 authorised resource。

------

# J. Job update 是否允许

这里要和现有设计保持一致，但 S3 引入一个新问题：

Job content 可能来自：

- Browser Extension；
- pasted URL parsing；
- user manual correction；
- source refresh。

所以完全 immutable 会太死。

我建议区别：

### Canonical Job factual content 可以受控更新

例如：

```text
PUT /api/jobs/{jobId}
```

允许当前 authorised workflow 修正：

```text
title
company
location
description
source fields
```

但是它不是普通“任何字段编辑”。

更重要的是：

> meaningful semantic change drives `contentVersion`.

------

# K. `contentVersion` OCC + provenance

前面已经冻结：

> `contentVersion` increments only for semantic changes affecting preparation/derived AI outputs, not operational metadata.

因此 update request 应带：

```json
{
  "contentVersion": 3,
  ...
}
```

如果当前 DB：

```text
contentVersion = 4
```

则：

```text
409 JOB_CONTENT_VERSION_CONFLICT
```

不能 silent overwrite。

所以 Job 的 OCC 和 Candidate Profile 是类似的，但语义不同：

```text
profileVersion
= Candidate factual revision

contentVersion
= Job semantic content revision
```

------

# L. 什么变化应该 increment `contentVersion`

例如：

```text
job description changed
title meaningfully changed
company changed
location changed where relevant
employment type changed
requirements changed
```

这些会影响：

```text
Job Intelligence
Match
Resume
Cover Letter
```

所以：

```text
contentVersion++
```

但下面不应该：

```text
lastFetchedAt
fetchAttemptCount
source health metadata
updated operational timestamp
AI processing metadata
```

导致版本递增。

否则 downstream artifacts 会被无意义地标记 stale。

------

# M. Job update response

成功：

```text
200 OK
```

返回完整最新 canonical Job。

例如：

```text
contentVersion: 4
```

如果提交内容与当前 semantic content 实际相同：

我建议 backend 可以保持：

```text
contentVersion = 3
```

而不是每次 PUT 必然 +1。

因为我们已经定义：

> version represents meaningful semantic revision.

所以 PUT 不是“每次请求就是新 revision”。

------

# N. Job content normalization

Backend 可以做有限 canonical normalization：

```text
trim whitespace
normalise platform enum
normalise known URL form
collapse obvious duplicated whitespace
```

但不能未经用户/明确 extraction flow 就：

```text
rewrite job description
summarise requirements
change title to a guessed canonical title
```

这些属于 derived intelligence，不属于 canonical Job facts。

------

# O. Raw HTML 不进入 canonical Job API response

Extension / URL parsing workflow 可能拿到：

```text
raw HTML
DOM extraction
page fragments
```

但 Job API 应只暴露 normalized canonical content。

不能 response：

```json
{
  "rawHtml": "..."
}
```

这同时符合 3.14 / 3.16 的 external content boundary 和 telemetry/privacy rule。

------

# P. Job Intelligence 是 sub-resource

建议：

```text
GET /api/jobs/{jobId}/intelligence
```

作为 derived sub-resource。

为什么放在 Job 下？

因为 Job Intelligence 的 provenance root 就是：

```text
Job + contentVersion
```

它不是 Candidate-dependent。

这和 Match 完全不同：

```text
Match = Candidate + Job
```

所以 Job Intelligence 自然属于：

```text
Job derived capability
```

------

# Q. Job Intelligence generation

建议使用 explicit command：

```text
POST /api/jobs/{jobId}/intelligence/generate
```

而不是：

```text
PUT /api/jobs/{jobId}/intelligence
```

因为这是：

> derived AI generation action

不是用户直接设置 intelligence state。

------

# R. Generation 应走 async

Job Intelligence 需要 AI，并且可能包含：

- extraction；
- classification；
- requirements analysis；
- provider latency；
- retry。

所以：

```text
POST /api/jobs/{jobId}/intelligence/generate
```

建议：

```text
202 Accepted
```

创建/触发 durable async work。

流程：

```text
authorise Job
 ↓
capture current contentVersion
 ↓
persist generation intent/business_event
 ↓
202
 ↓
worker
```

------

# S. Generate command response

建议：

```json
{
  "jobId": "uuid",
  "contentVersion": 3,
  "status": "PROCESSING"
}
```

不需要暴露：

```text
eventId
provider
retry count
correlationId
```

这些内部处理。

------

# T. Intelligence lifecycle

对外建议：

```text
NOT_AVAILABLE
PROCESSING
READY
FAILED
STALE
```

但我建议 `GET` 语义更干净一点：

如果从未生成：

```text
404 JOB_INTELLIGENCE_NOT_FOUND
```

而不是返回一个虚拟：

```text
NOT_AVAILABLE
```

对于已存在 resource：

```text
PROCESSING
READY
FAILED
STALE
```

这样 resource semantics 更明确。

------

# U. `STALE` 的定义

这是 Job Intelligence contract 的关键。

假设：

```text
Job contentVersion = 4
```

而 Intelligence：

```text
sourceContentVersion = 3
```

那么：

```text
status = STALE
```

即：

> intelligence 曾经成功生成过，但已经不是针对当前 Job semantic content。

它不能继续被当作最新可信输入。

------

# V. 不自动隐藏 stale intelligence

GET 可以返回：

```json
{
  "jobId": "...",
  "sourceContentVersion": 3,
  "currentContentVersion": 4,
  "status": "STALE",
  "intelligence": {
    ...
  }
}
```

但这里我们要决定：

是否返回旧 intelligence body。

我建议 **可以返回，但必须明确标记 STALE**。

原因：

- UI 可以告诉用户“Job changed, regenerate analysis”；
- diagnostics/debug 更透明；
- 不必假装以前的 result 不存在。

但下游 Match/Resume command 必须按自身规则决定是否接受 stale upstream。

通常应拒绝使用 stale intelligence。

------

# W. Intelligence representation

Job Intelligence 可以包含类似：

```json
{
  "jobId": "...",
  "sourceContentVersion": 4,
  "status": "READY",
  "summary": "...",
  "seniority": "...",
  "responsibilities": [],
  "requiredSkills": [],
  "preferredSkills": [],
  "keywords": [],
  "requirements": [],
  "generatedAt": "..."
}
```

具体字段应沿用前面 frozen 的 Job Intelligence domain model。

这里原则是：

> 返回稳定 semantic result，而不是 provider-specific structured output。

不暴露：

```text
model
prompt
temperature
raw JSON
provider schema
```

除非 Admin diagnostics 另有需要。

------

# X. Intelligence generation captures version

Worker 开始时不能只：

```text
load latest Job
```

然后不管 request originally targeted 什么版本。

正确 provenance 应该绑定：

```text
jobId
sourceContentVersion
```

如果 command 是基于 version 4 发起，最终 result 必须明确：

```text
sourceContentVersion = 4
```

如果处理期间 Job 变成 version 5：

那么 version 4 的 result 完成后应立即是：

```text
STALE
```

不能错误地标记 READY for version 5。

------

# Y. Generate command 是否需要 `contentVersion`

我建议 **需要客户端显式提交**：

```json
{
  "contentVersion": 4
}
```

为什么？

因为这样客户端明确表示：

> generate intelligence for the Job state I am currently viewing.

如果用户页面还是 version 3，但 Job 已是 4：

```text
409 JOB_CONTENT_VERSION_CONFLICT
```

而不是悄悄针对 version 4 开始 AI。

这和我们整个 3.15 的 stale-write/command philosophy 一致。

------

# Z. Existing current intelligence 时重复 generate

如果：

```text
Job contentVersion = 4
Intelligence READY sourceContentVersion = 4
```

用户再次：

```text
POST /generate
```

怎么处理？

我建议：

```text
200 OK
```

返回已有 current Intelligence，而不是无意义再烧一次 AI token。

除非未来提供显式：

```text
forceRegenerate
```

但 S3 不需要。

这样 generation command 具备 business-level reuse semantics。

------

# AA. Processing 中重复 generate

如果同一：

```text
jobId + contentVersion
```

已经 PROCESSING：

可以：

```text
202 Accepted
```

返回现有 processing state。

不要重复创建多个 AI generation jobs。

这需要 backend 在 service / business event 层做 dedup。

3.15 的 idempotency design可以支撑这个原则。

------

# AB. Failed generation 后再次 generate

如果同一个 contentVersion 最终 FAILED：

用户再次调用 `/generate` 可以视为新的 explicit attempt。

这次允许：

```text
202 Accepted
```

并创建新的 generation work。

内部 automatic retries 已经耗尽后，用户的再次尝试是新的 business command。

------

# AC. Intelligence failure response

GET 如果最终失败：

```json
{
  "jobId": "...",
  "sourceContentVersion": 4,
  "status": "FAILED",
  "failure": {
    "code": "JOB_INTELLIGENCE_GENERATION_FAILED",
    "message": "Job intelligence could not be generated."
  },
  "updatedAt": "..."
}
```

同 Resume Import 一样：

不返回 provider detail。

------

# AD. AI feature disabled

如果 Admin AI Governance 把：

```text
JOB_INTELLIGENCE
```

feature disabled：

`POST /generate` 应立即拒绝，不创建 async work。

建议：

```text
409 Conflict
{
  "code": "AI_FEATURE_DISABLED",
  "message": "Job intelligence generation is currently unavailable.",
  ...
}
```

为什么不是 503？

因为这是应用配置下的明确业务状态，不是临时网络故障。

------

# AE. Job Intelligence 不接受 client edits

S3 不提供：

```text
PUT /api/jobs/{jobId}/intelligence
PATCH ...
```

因为它是 derived artifact。

如果 AI result 不好，用户可以：

- 修正 canonical Job；
- regenerate。

不能让用户手动修改后还把它标记成“AI-derived intelligence”。

------

# AF. Job deletion

我建议 S3 不开放：

```text
DELETE /api/jobs/{jobId}
```

原因类似 Candidate Profile，但更明显：

Job 可能已经被：

```text
Application
Preparation
Match
Resume
Cover Letter
```

引用。

删除一个 canonical Job 不是普通 resource delete，而是 lifecycle/data-retention 问题。

因此 S3 暂不提供。

------

# AG. Browser Extension 与 Job API

Extension 可以通过合法 backend API 建立 Job。

但它不应该直接把：

```text
full raw DOM
raw HTML
arbitrary scripts
```

提交给 canonical Job endpoint。

更合理的是 Extension：

```text
extracts bounded fields
 ↓
POST canonical Job input
```

或者走专门 extension ingestion contract，我们会在 3.17.10 再定。

所以当前 `/api/jobs` 仍然只接受 canonical business fields。

------

# AH. Application-created Job compatibility

现有 S2 Application creation 如果已经包含 Job creation逻辑，不需要全部推翻。

可以有内部 application service 调用 Job capability：

```text
create/find canonical Job
 ↓
create Application
```

但 HTTP contract 仍然保持：

```text
/api/applications
```

对 S2 Fast Path 友好。

前端用户不需要为了记一次 Application 先手动调用 `/api/jobs`。

这就是：

> API capability separation ≠ forcing clients into unnecessary choreography.

------

# AI. Recommended endpoints

最终建议 Job 这一节最小 contract：

```text
POST /api/jobs
GET  /api/jobs/{jobId}
PUT  /api/jobs/{jobId}

GET  /api/jobs/{jobId}/intelligence
POST /api/jobs/{jobId}/intelligence/generate
```

不提供：

```text
GET /api/jobs
DELETE /api/jobs/{jobId}
PUT/PATCH intelligence
manual provider selection
raw HTML endpoint
```

------

# AJ. End-to-end flow

典型 Preparation Path：

```text
1. User / Extension captures job

2. POST /api/jobs
      ↓
   201 new / 200 canonical reuse
      ↓
   jobId + contentVersion

3. POST /api/jobs/{jobId}/intelligence/generate
   {
     contentVersion
   }
      ↓
   202 PROCESSING

4. GET /api/jobs/{jobId}/intelligence
      ↓
   PROCESSING
      ↓
   READY

5. Preparation / Match uses:
   candidate profileVersion
   +
   job contentVersion
   +
   current Job Intelligence
```

如果 Job 后来修改：

```text
contentVersion 4 → 5
```

旧 intelligence：

```text
sourceContentVersion = 4
status = STALE
```

然后重新 generate。

这使整个 provenance chain 非常清楚。

------

# 3.17.5 Proposed Frozen Decisions

我建议冻结为：

> **Canonical Job is an independent authorised resource exposed through `POST /api/jobs`, `GET /api/jobs/{jobId}`, and controlled `PUT /api/jobs/{jobId}` operations. Creating or reusing a Job does not create an Application, and Application remains independently usable through the S2 Fast Path. Backend canonicalisation/deduplication may reuse an existing Job internally, but canonical identity never grants access: every Job read/update requires an authorised Application, Preparation, or relationship-establishing workflow, and absent/inaccessible Jobs collapse to 404. S3 exposes no global Job catalogue/search API and no ordinary Job deletion API.**
>
> **Job semantic state is protected by `contentVersion`. Authorised updates submit the version they observed; stale updates fail explicitly with `409 JOB_CONTENT_VERSION_CONFLICT`. `contentVersion` advances only when canonical business content changes in a way that can affect derived preparation results, and does not advance for operational/fetch/processing metadata. Canonical Job APIs expose normalized business content rather than raw HTML, DOM data, AI output, provider metadata or operational diagnostics.**
>
> **Job Intelligence is a Job-derived sub-resource exposed through `GET /api/jobs/{jobId}/intelligence` and explicit `POST /api/jobs/{jobId}/intelligence/generate`. Generation is asynchronous and durable, captures the requested Job `contentVersion`, and returns 202 when work is running. Intelligence provenance includes its source content revision; a previously successful result becomes `STALE` whenever the current Job content revision differs. Repeated generation for an already-current result reuses that result rather than needlessly invoking AI, while duplicate processing for the same Job revision is deduplicated. Failed generations may be explicitly retried through a new generate command after internal retries are exhausted. Job Intelligence is provider-neutral derived data and cannot be directly edited by clients.**

这套设计把 Job 本身、Job Intelligence，以及后续 Match 的 provenance 链条分得很清楚。

下一步进入 **3.17.6 Preparation / Match API**。这一节会真正把：

```
Candidate profileVersion + Job contentVersion + Job Intelligence revision/state → Match Analysis
```

落成 API，同时要处理 Preparation 到底需不需要独立 `preparationId`、Match regeneration、staleness 和 Application independence。





进入 **3.17.6 Preparation / Match API**。

这一节要把 Preparation 作为 **Candidate + Job 的独立准备上下文**正式落成 HTTP contract，并把 Match 的输入版本、生成、状态、staleness、重生成和 Application 解耦全部固定下来。

# 3.17.6 Preparation / Match API

## A. Preparation 是独立 workflow resource

前面已经 frozen：

> Preparation is optional and anchored to Candidate + Job, not Application.

因此我建议明确引入：

```text
POST /api/preparations
GET  /api/preparations/{preparationId}
```

Preparation 不是单纯 UI concept，而是后续：

```text
Match
Tailored Resume
Cover Letter
```

共同依附的业务上下文。

这样比把所有 API 都挂在：

```text
/api/jobs/{jobId}/...
```

更清晰，因为 Match/Resume/Cover Letter 都同时依赖 Candidate 和 Job。

------

# B. 为什么需要 `preparationId`

理论上我们可以把 Preparation 用：

```text
candidateId + jobId
```

隐式表示。

但我建议显式 `preparationId`，原因有几个：

- HTTP resource identity 更稳定；
- 后续 Match/Resume/Cover Letter 都有共同父上下文；
- authorization 可以统一从 Preparation resolve Candidate ownership；
- future history/provenance 更容易；
- 不需要客户端在每次调用时重复 candidateId + jobId；
- Candidate identity 本来就不应该由客户端自由指定。

因此：

```text
preparationId
```

是业务 domain ID，不是 surrogate DB id。

------

# C. 创建 Preparation

建议：

```text
POST /api/preparations
```

request：

```json
{
  "jobId": "uuid"
}
```

不提交：

```text
candidateId
userId
applicationId
profileVersion
```

Backend：

```text
authenticated user
   ↓
resolve Candidate
   ↓
authorise Job relationship / relationship-establishing workflow
   ↓
create or reuse Candidate + Job Preparation
```

这里 Candidate 来自 security context。

------

# D. 创建 Preparation 不要求 Application

这条必须再次落到 API：

```text
POST /api/preparations
{
  "jobId": "..."
}
```

不能要求：

```json
{
  "applicationId": "..."
}
```

因为用户完全可以：

```text
发现岗位
→ Preparation
→ Match
→ Resume
→ Cover Letter
→ 最后才申请
```

甚至最后根本没申请。

------

# E. 同一 Candidate + Job 建议只有一个 active Preparation

S3 我建议：

> one logical Preparation per Candidate + Job.

因此重复：

```text
POST /api/preparations
{
  "jobId": sameJob
}
```

不创建多个平行 Preparation。

如果已经存在：

```text
200 OK
```

返回现有 Preparation。

如果新创建：

```text
201 Created
```

这样可以避免：

```text
Preparation A
Preparation B
Preparation C
```

都针对同一个 Candidate + Job，导致用户不知道哪个才是当前工作空间。

------

# F. Preparation representation

建议：

```json
{
  "preparationId": "uuid",
  "jobId": "uuid",
  "candidateId": "uuid",
  "createdAt": "...",
  "updatedAt": "..."
}
```

但这里我建议不要急着塞：

```text
profileVersion
jobContentVersion
matchStatus
resumeStatus
coverLetterStatus
```

全部进 Preparation root。

因为这些是各 derived capability 自己的 provenance/state。

否则 Preparation DTO 很快会变成一个巨大 dashboard projection。

S3 root contract保持简单。

------

# G. Preparation ownership

所有：

```text
GET /api/preparations/{preparationId}
```

以及后续：

```text
/preparations/{id}/match
/preparations/{id}/resumes
/preparations/{id}/cover-letters
```

都先通过 Preparation → Candidate ownership 做 authorised lookup。

如果不存在或不属于当前 user：

```text
404 PREPARATION_NOT_FOUND
```

不能靠传 candidateId 做 ownership assertion。

------

# H. 不开放 global Preparation list，除非 UI 真需要

S3 最小 contract 我建议暂时不加：

```text
GET /api/preparations
```

因为当前 Preparation 通常从某个 Job / Application workflow 进入。

如果产品 UI 后面真的需要“全部准备中的岗位”页面，再添加 list API。

现在不提前设计。

------

# I. Match 是 Preparation-derived sub-resource

Match 不是 Job-only artifact，也不是 Candidate-only artifact。

它依赖：

```text
Candidate Profile
+
Job
+
current Job Intelligence
```

因此最自然的 API 是：

```text
GET  /api/preparations/{preparationId}/match
POST /api/preparations/{preparationId}/match/generate
```

而不是：

```text
/api/jobs/{jobId}/match
```

也不是：

```text
/api/applications/{applicationId}/match
```

这完全符合 frozen domain boundary。

------

# J. Match generation input provenance

Match 至少必须绑定：

```text
candidateProfileVersion
jobContentVersion
```

以及 Job Intelligence 对应的 source revision。

例如：

```text
Candidate Profile version = 8
Job contentVersion = 4
Job Intelligence sourceContentVersion = 4
```

生成出的 Match 应明确记录：

```text
sourceProfileVersion = 8
sourceJobContentVersion = 4
```

Job Intelligence 可以通过 version relationship证明其 freshness，不一定还要额外暴露一个 intelligenceVersion，除非之前 DB/domain 已经明确需要。

------

# K. Match generate command 必须携带客户端看到的版本

建议：

```text
POST /api/preparations/{preparationId}/match/generate
```

body：

```json
{
  "profileVersion": 8,
  "jobContentVersion": 4
}
```

这样 client 明确表达：

> Generate the match for the Candidate and Job state I am currently reviewing.

Backend 检查：

```text
current profileVersion == 8
current job contentVersion == 4
```

否则：

```text
409
```

对应：

```text
PROFILE_VERSION_CONFLICT
JOB_CONTENT_VERSION_CONFLICT
```

不能悄悄切换到 newer facts。

------

# L. 为什么不能 backend 自动使用“最新版本”

看起来这样更方便：

```text
POST /match/generate
{}
```

backend 自动读取最新 Profile + Job。

但这会破坏我们在 3.15 定下来的核心原则：

> stale commands must not silently operate on newer state.

例如用户正在看：

```text
Profile v8
```

另一个 tab 更新到了：

```text
v9
```

如果 backend 自动拿 v9 做 Match，用户以为结果针对自己正在看的 v8。

这是隐蔽 consistency bug。

所以版本必须显式。

------

# M. 生成前必须要求 current Job Intelligence

Match generation 不能拿 stale Job Intelligence。

因此 backend 在 generate command 时检查：

```text
Job Intelligence exists
status = READY
sourceContentVersion = current jobContentVersion
```

否则不能开始 Match。

如果 intelligence 不存在：

```text
409 JOB_INTELLIGENCE_REQUIRED
```

如果 stale：

```text
409 JOB_INTELLIGENCE_STALE
```

如果 processing：

```text
409 JOB_INTELLIGENCE_NOT_READY
```

这里我倾向稍微收敛 error code：

```text
JOB_INTELLIGENCE_NOT_READY
```

可以覆盖 missing / processing / stale，并通过 safe message说明用户应该先生成/刷新 Job Intelligence。

不过从 frontend UX 看，`STALE` 和 `PROCESSING` 可能需要不同按钮状态。

所以建议保留：

```text
JOB_INTELLIGENCE_NOT_READY
JOB_INTELLIGENCE_STALE
```

两种即可，不需要更多。

------

# N. Candidate Profile sufficiency 在 Match command 处验证

Profile 保存时不要求 Match-ready。

但这里要验证 capability sufficiency。

例如 Candidate 完全空白，只存在名字：

```text
POST /match/generate
```

应该拒绝。

建议：

```text
409 CANDIDATE_PROFILE_INSUFFICIENT_FOR_MATCH
```

response：

```json
{
  "code": "CANDIDATE_PROFILE_INSUFFICIENT_FOR_MATCH",
  "message": "Add more candidate profile information before generating a match.",
  "requestId": "...",
  "timestamp": "..."
}
```

如果需要告诉 frontend 缺什么，可以扩展：

```json
{
  "details": {
    "missing": [
      "SKILLS",
      "EXPERIENCE"
    ]
  }
}
```

但这里我建议先不要扩展通用 ApiError schema。

S3 可以由 UI 基于 Candidate Profile自己判断并引导。

Backend authoritative validation仍然必须存在。

------

# O. Match generation 是 async

Match 明显是 AI-derived operation。

所以：

```text
POST /api/preparations/{id}/match/generate
```

成功接受：

```text
202 Accepted
```

流程：

```text
authorise Preparation
 ↓
validate Profile version
 ↓
validate Job version
 ↓
validate current Job Intelligence
 ↓
validate Match sufficiency
 ↓
persist generation intent/business_event
 ↓
202
```

Worker 后续生成。

------

# P. Match user-facing lifecycle

建议：

```text
PROCESSING
READY
FAILED
STALE
```

如果从未生成：

```text
GET /match
```

返回：

```text
404 MATCH_NOT_FOUND
```

而不是虚拟 `NOT_AVAILABLE`。

这个规则和 Job Intelligence 保持一致。

------

# Q. Match `STALE` 定义

Match 已经 READY：

```text
sourceProfileVersion = 8
sourceJobContentVersion = 4
```

后来：

```text
Profile → 9
```

或者：

```text
Job → 5
```

则：

```text
status = STALE
```

无论 Match 本身是否“看起来仍然差不多”，都不能把它当 current result。

因为 provenance 已经变化。

------

# R. Job Intelligence 重生成是否让 Match stale

如果 Job：

```text
contentVersion = 4
```

Job Intelligence 原先也是基于 4，后来用户手动重新 generate，仍然基于：

```text
contentVersion = 4
```

这里是否让 Match stale？

我建议：

**S3 不因为同一 Job contentVersion 的 Intelligence重新生成而自动 stale Match，除非 Job Intelligence 本身有独立 meaningful revision contract。**

原因是我们目前已经冻结的 Job semantic provenance 是：

```text
contentVersion
```

如果没有定义 `intelligenceVersion`，就不能在 3.17 API 层偷偷新增另一个 provenance dimension。

所以当前 Match provenance仍然以：

```text
profileVersion + jobContentVersion
```

为核心。

如果未来 Job Intelligence algorithm/model revision需要强制下游 regeneration，那应该作为 future additive provenance design，而不是在这里临时扩张。

------

# S. Match response shape

建议：

```json
{
  "matchId": "uuid",
  "preparationId": "uuid",

  "sourceProfileVersion": 8,
  "sourceJobContentVersion": 4,

  "currentProfileVersion": 8,
  "currentJobContentVersion": 4,

  "status": "READY",

  "score": 78,

  "summary": "...",

  "strengths": [
    {
      "title": "...",
      "explanation": "..."
    }
  ],

  "gaps": [
    {
      "title": "...",
      "explanation": "..."
    }
  ],

  "matchedSkills": [],
  "missingOrWeakSkills": [],
  "recommendations": [],

  "generatedAt": "..."
}
```

具体字段应保持和前面 frozen Match model一致，但有几个原则现在可以定。

------

# T. Match score 必须 explainable

Match response 不能只有：

```json
{
  "score": 78
}
```

至少要有能帮助用户理解：

```text
strengths
gaps
matched evidence/reasons
recommendations
```

这与 S3 的产品目标一致：

> explainable Job Match Analysis/score.

但同时不能暴露：

```text
chain-of-thought
hidden reasoning
provider raw reasoning
```

返回的是产品级解释，而不是模型内部推理过程。

------

# U. Match 不得制造 Candidate facts

Match response可以说：

```text
"Your Java experience aligns strongly with..."
```

前提是对应 Candidate Profile 中确实存在。

不能生成：

```text
"You have Kubernetes production experience"
```

如果 Profile 没有这个事实。

因此 Match generation必须严格基于：

```text
Candidate Profile authoritative facts
+
Job facts/intelligence
```

而不是“补全用户可能会有的能力”。

------

# V. Gap 不是负面 Candidate fact

例如：

```text
missingOrWeakSkills: ["Kubernetes"]
```

只是 Job Match derived analysis。

它不能写回：

```text
Candidate Profile
```

更不能自动新增：

```text
candidate_skills
```

这继续保持 derived data 与 factual source separation。

------

# W. `current*Version` 是否值得返回

我建议返回：

```text
sourceProfileVersion
sourceJobContentVersion
currentProfileVersion
currentJobContentVersion
```

为什么？

这样 frontend 可以直接判断：

```text
source == current → READY
source != current → STALE
```

而不需要额外 GET Candidate + Job 才知道。

不过这里也可以由 backend直接返回 status。

即便如此，current versions 对 UI display / regenerate command 很有帮助。

因此我倾向保留。

------

# X. Match status 应由 backend计算

Client 不应该自己仅凭：

```text
sourceProfileVersion != currentProfileVersion
```

完全定义 status。

Backend 是 authoritative。

例如未来可能还有：

```text
FAILED
PROCESSING
```

所以 client 读取：

```text
status
```

而 version fields只是解释/provenance。

------

# Y. 重复 generate：current READY

如果已经存在：

```text
READY
sourceProfileVersion = current profileVersion
sourceJobContentVersion = current jobContentVersion
```

再次：

```text
POST /match/generate
```

我建议：

```text
200 OK
```

直接返回现有 Match。

不要重新花 AI token。

和 Job Intelligence保持一致。

------

# Z. 重复 generate：same version PROCESSING

如果相同输入组合：

```text
preparationId
profileVersion=8
jobContentVersion=4
```

已经 PROCESSING：

重复 command：

```text
202 Accepted
```

返回现有 processing representation。

不创建第二个生成任务。

------

# AA. STALE 后 generate

如果旧 Match：

```text
sourceProfileVersion=8
sourceJobContentVersion=4
STALE
```

当前：

```text
profileVersion=9
jobContentVersion=4
```

用户：

```text
POST /generate
{
  profileVersion: 9,
  jobContentVersion: 4
}
```

这应该创建新的 Match generation work：

```text
202 Accepted
```

旧 Match可以保留做历史/provenance，但当前 API 应指向最新 logical Match state。

------

# AB. 是否需要 Match history API

S3 我建议不提供：

```text
GET /preparations/{id}/matches
GET /matches/{matchId}
```

的历史 browsing capability。

当前产品只需要：

> current/latest Match for this Preparation.

内部可以保留历史记录或 artifact provenance，但 HTTP contract先保持最小。

后续如果需要：

> Compare Match before/after Profile improvement

再做。

------

# AC. Match failure

GET 可以返回：

```json
{
  "matchId": "...",
  "preparationId": "...",
  "sourceProfileVersion": 8,
  "sourceJobContentVersion": 4,
  "status": "FAILED",
  "failure": {
    "code": "MATCH_GENERATION_FAILED",
    "message": "The match analysis could not be generated."
  },
  "updatedAt": "..."
}
```

同样不暴露：

```text
provider
model
prompt
raw response
stack trace
```

------

# AD. AI feature disabled

如果：

```text
MATCH_ANALYSIS
```

被 Admin feature toggle关闭：

```text
POST /match/generate
```

立即：

```text
409 AI_FEATURE_DISABLED
```

不创建 business event。

------

# AE. Match 不能直接编辑

不提供：

```text
PUT /match
PATCH /match
```

因为它是 derived analysis。

用户如果认为结果不合理，可以：

```text
correct Candidate Profile
or
correct Job
or
regenerate
```

不能手动改 score 然后让系统误以为这是 generated result。

------

# AF. Preparation 不依赖 Match

这一点也需要明确。

创建：

```text
Preparation
```

不等于必须生成 Match。

理论上用户未来可以直接生成 Resume / Cover Letter，如果 capability规则允许。

不过我们前面 frozen 的 S3 design里，Tailored Resume / Cover Letter 会依赖一定 Match Analysis上下文。

具体在下一节分别确定。

但 Preparation root 本身不应该强制 Match existence。

------

# AG. Application 与 Preparation 的关系

Application 可以引用同一个 Job，但 API 不把 Application嵌入 Preparation。

例如：

```text
Application
    └── Job A

Preparation
    ├── Candidate
    └── Job A
```

这两个 resource的共同点是：

```text
Job A
```

不是：

```text
Preparation belongs to Application
```

因此不提供：

```text
POST /api/applications/{applicationId}/preparation
```

作为核心 contract。

UI 可以从 Application Detail 按钮：

```text
Prepare for this job
```

内部做：

```text
POST /api/preparations
{ jobId }
```

这就足够了。

------

# AH. Preparation deletion

S3 我建议不开放：

```text
DELETE /api/preparations/{id}
```

因为它可能挂着：

```text
Match
Resume
Cover Letter
```

以及 provenance。

如果未来需要“remove from preparation workspace”，应该先决定是：

- hide/archive；
- hard delete；
- privacy deletion。

不是现在用普通 DELETE解决。

------

# AI. 推荐最小 API

最终建议：

```text
POST /api/preparations
GET  /api/preparations/{preparationId}

GET  /api/preparations/{preparationId}/match
POST /api/preparations/{preparationId}/match/generate
```

暂时不提供：

```text
GET /api/preparations
DELETE /api/preparations/{id}
PUT/PATCH /match
match history
application-nested preparation APIs
```

------

# AJ. End-to-end Match flow

典型流程：

```text
1. Candidate Profile
   profileVersion = 8

2. Canonical Job
   contentVersion = 4

3. Job Intelligence
   READY
   sourceContentVersion = 4

4. POST /api/preparations
   {
     jobId
   }
      ↓
   preparationId

5. POST /api/preparations/{id}/match/generate
   {
     profileVersion: 8,
     jobContentVersion: 4
   }
      ↓
   202 PROCESSING

6. GET /api/preparations/{id}/match
      ↓
   READY
   sourceProfileVersion = 8
   sourceJobContentVersion = 4
```

后来 Candidate Profile更新：

```text
8 → 9
```

则 GET Match：

```text
status = STALE
sourceProfileVersion = 8
currentProfileVersion = 9
```

用户点击 regenerate：

```text
POST /match/generate
{
  profileVersion: 9,
  jobContentVersion: 4
}
```

新的 Match生成。

这个 provenance chain非常清晰。

------

# 3.17.6 Proposed Frozen Decisions

我建议冻结为：

> **Preparation is an explicit Candidate + Job workflow resource exposed through `POST /api/preparations` and `GET /api/preparations/{preparationId}`. The authenticated user’s Candidate is resolved server-side; clients submit only the authorised `jobId`, never `candidateId`, `userId` or `applicationId`. Preparation remains independent of Application, and one logical Preparation is reused for the same Candidate + Job pair rather than creating parallel active workspaces. Preparation ownership is always resolved through Candidate ownership, with absent or inaccessible resources returning 404.**
>
> **Match is a Preparation-derived sub-resource exposed through `GET /api/preparations/{preparationId}/match` and explicit `POST /api/preparations/{preparationId}/match/generate`. Generation is asynchronous and durable. The command explicitly supplies the Candidate `profileVersion` and Job `contentVersion` visible to the client; backend generation never silently switches to newer state. Match generation requires a sufficiently complete Candidate Profile and current READY Job Intelligence for the same Job content revision. Version conflicts or stale/not-ready prerequisites fail explicitly before async work is created.**
>
> **Each Match records its source Candidate Profile revision and Job content revision. A successful Match becomes `STALE` whenever either authoritative revision changes. User-facing Match states are `PROCESSING`, `READY`, `FAILED`, and `STALE`; absence is represented as 404 rather than a synthetic empty resource. A current READY result is reused rather than regenerated unnecessarily, and duplicate processing for the same Preparation/profileVersion/jobContentVersion tuple is deduplicated. Match responses provide product-level explainable score/reasons/strengths/gaps grounded only in authoritative Candidate facts and Job information, never model chain-of-thought or invented Candidate facts. Match is derived data and cannot be directly edited or written back into Candidate Profile.**

这版把 **Preparation resource identity、Candidate + Job anchor、explicit versioned generation 和 Match staleness** 都定住了。

下一步就是 **3.17.7 Tailored Resume API**，在那里要重点处理：

```
Match + Candidate facts + Job → generated resume artifact
```

以及 **不能捏造经历、artifact version/provenance、重新生成、编辑后的 resume 到底还是不是 AI-generated artifact**。



好，我们从 **3.17.7** 开始，一口气完成 **3.17 API & Contract Detailed Design**。前面的 **3.17.1–3.17.6** 继续视为已确认 baseline；下面每一节都按“设计 → 冻结结论”的方式推进。

------

# 3.17.7 Tailored Resume API

Tailored Resume 的事实边界已经非常明确：

> 它是基于真实 Candidate Profile、目标 Job 和 Match context 生成的 derived artifact。
> 它可以重组、强调、改写表达，但绝不能创造 Candidate Profile 中不存在的事实。

## A. Resume 属于 Preparation

因此 API 应该是：

```text
POST /api/preparations/{preparationId}/resumes
GET  /api/preparations/{preparationId}/resumes/{resumeId}
PUT  /api/preparations/{preparationId}/resumes/{resumeId}
```

这里我建议 **Resume 使用独立 `resumeId`**，而不是只有：

```text
/preparation/{id}/resume
```

原因是 Resume 和 Match 不一样。

Match 是“当前分析结果”，而 Resume 是用户可能真正使用、下载、修改的 artifact。即使 S3 UI 当前只展示一个主要版本，独立 artifact identity 仍然值得保留。

------

## B. 生成 Resume 是 command

生成：

```text
POST /api/preparations/{preparationId}/resumes
```

request：

```json
{
  "profileVersion": 8,
  "jobContentVersion": 4,
  "matchId": "uuid"
}
```

这里显式指定：

- Candidate Profile revision；
- Job revision；
- Match result。

Backend 必须验证 Match：

```text
status = READY
sourceProfileVersion = 8
sourceJobContentVersion = 4
```

如果 Match stale：

```text
409 MATCH_STALE
```

如果 Match 不存在或未完成：

```text
409 MATCH_NOT_READY
```

------

## C. Resume generation async

生成涉及 AI，因此：

```text
POST /resumes
→ 202 Accepted
```

response：

```json
{
  "resumeId": "uuid",
  "status": "PROCESSING",
  "sourceProfileVersion": 8,
  "sourceJobContentVersion": 4,
  "sourceMatchId": "uuid"
}
```

business state + event 必须在一个短 transaction 中提交。

------

## D. Resume lifecycle

建议：

```text
PROCESSING
READY
FAILED
STALE
```

这里 `STALE` 的定义：

如果：

```text
sourceProfileVersion != currentProfileVersion
```

或者：

```text
sourceJobContentVersion != currentJobContentVersion
```

则 Resume 是 stale。

即使用户已经手动编辑它，也不能假装它仍然针对当前 authoritative inputs。

------

## E. Resume 内容结构化保存

不要只存：

```text
generatedText
```

更适合的 contract 是 structured resume artifact，例如：

```json
{
  "resumeId": "uuid",
  "status": "READY",
  "sourceProfileVersion": 8,
  "sourceJobContentVersion": 4,
  "sourceMatchId": "uuid",

  "content": {
    "professionalSummary": "...",
    "skills": [],
    "experiences": [],
    "education": [],
    "certifications": []
  },

  "generatedAt": "...",
  "updatedAt": "..."
}
```

这样 React 可以真正编辑，而不是编辑一大坨 AI prose。

------

## F. 用户可以编辑 Tailored Resume

这里和 Match 不一样。

Resume 是最终用户 artifact，因此：

```text
PUT /api/preparations/{preparationId}/resumes/{resumeId}
```

应该允许。

但编辑的是：

> presentation / wording / selection of already-supported facts

而不是把 Candidate Profile 当作背景数据库然后允许 Resume 创造新事实。

------

## G. Resume 自己需要 artifact version

建议 Resume 有：

```text
version
```

例如：

```json
{
  "resumeId": "...",
  "version": 3
}
```

这是 Resume artifact 自身 OCC，与：

```text
profileVersion
jobContentVersion
```

完全不同。

例如两个 tab 同时编辑 Resume：

```text
resume.version = 3
```

客户端提交：

```text
version = 3
```

成功：

```text
version = 4
```

stale edit：

```text
409 RESUME_VERSION_CONFLICT
```

------

## H. 用户修改 Resume 不更新 Candidate Profile

这是非常重要的边界。

例如用户把：

```text
"Developed REST APIs"
```

改成：

```text
"Designed and delivered REST APIs supporting order workflows"
```

只修改 Resume artifact。

不能自动写回：

```text
candidate_experience_items
```

如果用户真的发现 Profile 本身事实不完整，应去 Candidate Profile 更新。

------

## I. Backend 必须验证事实来源

Resume update 不能完全信任前端，因为用户可能直接篡改 JSON 加：

```text
"Kubernetes production experience"
```

问题是产品是否应该阻止用户自己写任何新内容。

这里我建议 S3 的边界是：

> OfferBuddy-generated claims must remain traceable to Candidate facts, but user-authored resume text is not silently promoted into Candidate Profile.

对于用户手动编辑后的 Resume，我们不需要构建复杂的 semantic fact verifier。

因此保存时进行：

- structural validation；
- ownership validation；
- OCC；

而不是每次调用 AI 判断有没有 hallucination。

但是系统在生成阶段必须严格 grounded。

------

## J. Generated 与 edited provenance

建议 Resume response 包含：

```text
generatedAt
updatedAt
```

并可以有：

```text
edited: true/false
```

或者更好的 domain state：

```text
origin = AI_GENERATED
hasUserEdits = true
```

不过 S3 不需要过度建模。

我建议只保留必要 metadata，不在 API 中制造复杂 provenance enum。

------

## K. 同输入重复生成

如果用户再次：

```text
POST /resumes
```

即使已有当前 Resume，我建议 **允许生成新的 resumeId**。

这一点和 Match 不同。

原因是用户可能合理地希望：

> “再给我一个版本。”

Resume 是 creative artifact，不应该强制 deduplicate 成一个结果。

但 transport-level duplicate submission 仍由 Idempotency-Key 处理。

------

## L. Resume 不 hard delete

S3 暂不提供：

```text
DELETE /resumes/{resumeId}
```

后面可以有 archive/delete workflow，但不是现在的核心。

------

### 3.17.7 Frozen

> Tailored Resume is a Preparation-owned, independently identifiable derived artifact. Generation is requested through `POST /api/preparations/{preparationId}/resumes`, requires explicit current Candidate Profile and Job revisions plus a current READY Match grounded in those revisions, and executes asynchronously. Each Resume records its source profile revision, Job content revision and Match identity; authoritative input changes make it stale.
>
> Generated Resume content must be grounded in Candidate Profile facts, but the resulting artifact is user-editable. User edits affect only the Resume artifact and never silently update Candidate Profile. Resume artifacts carry their own optimistic-concurrency version independent from Candidate `profileVersion` and Job `contentVersion`. Multiple intentionally generated Resume variants may coexist, while transport duplicates are handled through the common idempotency contract.

------

# 3.17.8 Cover Letter API

Cover Letter 也是 Preparation artifact，但比 Resume 更适合 document-style content。

## A. Endpoints

建议：

```text
POST /api/preparations/{preparationId}/cover-letters
GET  /api/preparations/{preparationId}/cover-letters/{coverLetterId}
PUT  /api/preparations/{preparationId}/cover-letters/{coverLetterId}
```

同样有独立：

```text
coverLetterId
```

------

## B. Generation input

request：

```json
{
  "profileVersion": 8,
  "jobContentVersion": 4,
  "matchId": "uuid"
}
```

Match 依然必须：

```text
READY
current
same profileVersion
same jobContentVersion
```

------

## C. Async

```text
POST /cover-letters
→ 202 Accepted
```

------

## D. Cover Letter provenance

记录：

```text
sourceProfileVersion
sourceJobContentVersion
sourceMatchId
```

Candidate/Job revision变化：

```text
status = STALE
```

------

## E. User-editable

Cover Letter 是最终文稿，因此用户应能：

```text
PUT /cover-letters/{id}
```

并有自己的：

```text
version
```

OCC stale edit：

```text
409 COVER_LETTER_VERSION_CONFLICT
```

------

## F. Grounding rules

AI 可以：

- 调整语气；
- 选择 Candidate Profile 中最相关经历；
- 将 Candidate + Job Match 组织成 narrative；
- 表达 motivation。

不能：

- 发明公司经历；
- 发明项目；
- 发明技能；
- 发明学历；
- 发明签证/eligibility；
- 编造 measurable achievements。

用户后续手工修改属于用户-owned artifact edit，但不会写回 Candidate Profile。

------

## G. Multiple variants

和 Resume 相同：

> explicit repeated generation 可以产生多个 Cover Letter variants。

不能像 Match 那样强制 reuse current artifact。

------

### 3.17.8 Frozen

> Cover Letter is a Preparation-owned, independently identifiable, user-editable derived artifact exposed through create/read/update APIs. AI generation is asynchronous and requires explicit current Candidate Profile and Job revisions plus a current Match for those inputs. Each Cover Letter records its provenance and becomes stale when authoritative Candidate or Job inputs change.
>
> AI generation may reorganise and articulate relevant Candidate facts but may not invent unsupported facts or silently alter Candidate Profile. Cover Letter edits use independent artifact optimistic concurrency and remain local to the document. Intentionally generated variants may coexist.

------

# 3.17.9 Application Compatibility / S2 Fast Path

这一节的任务就是防止 S3 API 破坏已经成功运行的 S2。

## A. Application API 独立保留

现有：

```text
/api/applications/**
```

继续工作。

创建 Application 不要求：

```text
Candidate Profile
Preparation
Match
Resume
Cover Letter
Job Intelligence
```

------

## B. Application update 使用自己的 version

沿用 3.15：

```text
applications.version
```

request 必须携带：

```text
version
```

stale：

```text
409 APPLICATION_VERSION_CONFLICT
```

不自动 retry。

------

## C. Application 可以和 Preparation 共用 Job

例如：

```text
Application → Job A
Preparation → Candidate + Job A
```

但 API 仍然不嵌套。

Application Detail 页面可以返回：

```text
jobId
```

Frontend 再通过：

```text
POST /api/preparations
{ jobId }
```

进入 Preparation。

------

## D. Preparation 不改变 Application lifecycle

生成 Resume、Match、Cover Letter：

不能自动：

```text
application.status = APPLIED
```

同样保存 Application：

不能自动创建 Match。

------

## E. S2 Job ingestion compatibility

Application creation 中现有 Job create/find logic 可以继续内部复用。

不强迫 frontend 改成：

```text
POST /jobs
POST /applications
```

两个调用。

API boundary 是 capability boundary，不是强迫 client choreograph internal modules。

------

### 3.17.9 Frozen

> S3 does not redefine the existing Application HTTP capability. Application remains independently user-owned, available without Candidate Profile or Preparation, and protected by its own optimistic-concurrency version. S2 Fast Path calls may internally reuse canonical Job services but are not forced into additional client choreography. Preparation and Application may reference the same Job yet remain independent resources and workflows; actions in one do not silently mutate lifecycle state in the other.

------

# 3.17.10 Browser Extension Contract

这里主要针对：

- SEEK
- Indeed
- LinkedIn

但不是 Auto Apply。

## A. Extension 不是 privileged client

Extension 和 React Web 一样：

```text
untrusted client
```

不能拥有 backend secret、admin privilege 或 ownership bypass。

------

## B. Extension 最小 capability

S3 Extension 主要做：

```text
capture current job
→ send bounded structured job input
→ receive canonical jobId
```

建议专门 namespace：

```text
POST /api/extension/jobs/import
```

而不是让 Extension 提交 arbitrary raw internal models。

------

## C. Input

例如：

```json
{
  "platform": "LINKEDIN",
  "url": "...",
  "externalJobId": "...",
  "title": "...",
  "companyName": "...",
  "location": "...",
  "description": "..."
}
```

Backend 必须：

- validate platform；
- validate URL；
- limit field sizes；
- treat page-derived content as untrusted external content；
- normalize canonical Job input；
- create/reuse legitimate Job relationship。

------

## D. 不上传整个 raw DOM

S3 不接受：

```text
pageHtml
document.documentElement.outerHTML
scripts
cookies
browser storage
```

Extension 应做 bounded extraction。

如果未来某网站需要 backend parsing raw HTML，这是另一个明确 capability，不应默认开放。

------

## E. 不接受 userId/candidateId

一样由 auth context resolution。

------

## F. 不做 Auto Apply

Extension S3 contract 不包括：

```text
submit application
fill arbitrary forms
click Apply
upload resume automatically
answer employer questions
```

这是明确 future backlog。

------

## G. Authentication

Extension 使用正常 user authentication mechanism，但 token storage/refresh implementation属于 security implementation，不在 HTTP domain contract 中泄漏特殊 trust assumption。

------

### 3.17.10 Frozen

> The Browser Extension is an untrusted first-party client with a deliberately narrow ingestion contract. It may submit bounded, structured Job data extracted from supported job sites, but cannot assert ownership, select AI providers, access Admin APIs, submit raw browser state/cookies/scripts, or bypass normal validation and authorization. Extension-originated page content is always treated as untrusted external input. S3 Extension APIs support Job capture and preparation initiation only; Auto Apply and arbitrary employer-form automation remain outside S3 scope.

------

# 3.17.11 Admin / AI Governance API

Admin 只负责运营 AI capability，不拥有 Candidate domain。

建议 namespace：

```text
/api/admin/ai/**
```

或遵循现有 RuoYi admin convention。

## A. Feature configuration

例如：

```text
GET /api/admin/ai/features
PUT /api/admin/ai/features/{featureKey}
```

feature：

```text
JOB_INTELLIGENCE
MATCH_ANALYSIS
RESUME_GENERATION
COVER_LETTER_GENERATION
RESUME_IMPORT
```

------

## B. Provider routing

例如：

```text
GET /api/admin/ai/providers
PUT /api/admin/ai/features/{featureKey}/routing
```

允许配置：

```text
enabled provider
model reference/config key
fallback policy
```

但 API 不能暴露 plaintext secret。

------

## C. Secrets

Admin response 只能：

```json
{
  "provider": "OPENAI",
  "credentialConfigured": true
}
```

不能：

```json
{
  "apiKey": "sk-..."
}
```

S3 Secrets 仍在 secret management/environment boundary。

------

## D. Operational metrics

Admin 可查询：

```text
GET /api/admin/ai/metrics
```

可返回：

```text
request count
success count
failure count
latency
provider
model
token usage
estimated cost
last failure time/code
```

但不能默认返回：

```text
prompt
raw response
candidate content
resume content
job description
```

------

## E. Admin configuration concurrency

对于 mutable runtime config，我建议拥有：

```text
version
```

Admin PUT 携带 current version。

stale config edit：

```text
409 AI_CONFIG_VERSION_CONFLICT
```

避免两个 admin tab 覆盖配置。

------

## F. Feature disabled semantics

用户调用 AI capability时，由 runtime config读取。

如果 disabled：

```text
409 AI_FEATURE_DISABLED
```

而不是让 provider router偷偷 fallback 到其他行为。

------

## G. Admin 不能 browse Candidate data

不添加：

```text
/api/admin/candidates
/api/admin/resumes
```

仅仅因为技术上可以。

任何未来 support access 必须独立做 privileged access + audit design。

------

### 3.17.11 Frozen

> RuoYi/Admin exposes separate, privilege-protected AI operational contracts for feature enablement, provider routing, runtime configuration and aggregated operational metrics. Admin remains an operational client rather than a Candidate/Preparation domain owner. Secrets are never returned in plaintext, and user content, prompts and raw provider responses are excluded from normal Admin monitoring APIs. Mutable AI configuration is versioned for explicit optimistic concurrency. Business-facing AI commands honour the current feature/runtime configuration and reject disabled capabilities before work is enqueued.

------

# 3.17.12 Async Operation & Status Contract

前面多个 endpoint 都用了 `202`，这里统一。

## A. 不引入 generic `/operations/{id}` 平台

S3 已经有业务 resource：

```text
ResumeImport
JobIntelligence
Match
Resume
CoverLetter
```

它们自己就能表达状态。

所以没必要再造：

```text
/api/operations/{operationId}
```

然后所有业务都跳转去 polling generic task。

这会重复 `business_events` 概念并污染产品 API。

------

## B. 202 必须返回业务 resource identity

例如：

```json
{
  "resumeId": "...",
  "status": "PROCESSING"
}
```

而不是：

```json
{
  "eventId": "..."
}
```

`eventId` 是 internal async identity。

------

## C. Poll business resource

Frontend poll：

```text
GET ResumeImport
GET JobIntelligence
GET Match
GET Resume
GET CoverLetter
```

不 poll business_events。

------

## D. Internal retries不可见

内部：

```text
PENDING
PROCESSING
RETRY
```

对外可以仍然：

```text
PROCESSING
```

只有 exhausted/terminal failure 才：

```text
FAILED
```

------

## E. Terminal state

通常：

```text
READY
FAILED
ACCEPTED
STALE
```

由 capability决定。

------

## F. HTTP 失败和 async FAILED 不一样

例如：

```text
POST /match/generate
```

request validation就失败：

```text
409
```

意味着：

> work never accepted.

而：

```text
202
```

之后 resource最终：

```text
FAILED
```

意味着：

> work accepted but processing ultimately failed.

这个 distinction 必须稳定。

------

## G. Retry semantics

automatic retry：

- same eventId；
- same correlationId；
- same business operation/provenance。

用户在 terminal FAILED 后再次显式 generate：

> new business attempt。

------

### 3.17.12 Frozen

> OfferBuddy does not introduce a generic public async-task or operation API. Async HTTP commands return 202 only after durable business state and corresponding work intent are accepted, and return the relevant business resource identity rather than internal event identifiers. Clients poll business resources for product-level states, while internal PENDING/RETRY/lease/attempt details remain hidden. A synchronous 4xx means work was not accepted; a later business-resource FAILED state means previously accepted work ultimately failed. Automatic retries retain the same durable business intent, while an explicit user retry after terminal failure represents a new business attempt.

------

# 3.17.13 Optimistic Concurrency / Idempotency HTTP Contract

这一节把 3.15 映射到 HTTP。

## A. OCC 使用 explicit domain versions

已经确定：

```text
Candidate Profile → profileVersion
Job → contentVersion
Application → version
Resume → version
Cover Letter → version
AI Admin config → version
```

不统一改成 ETag。

------

## B. Stale write 一律 409

统一：

```text
409 Conflict
```

machine codes：

```text
PROFILE_VERSION_CONFLICT
JOB_CONTENT_VERSION_CONFLICT
APPLICATION_VERSION_CONFLICT
RESUME_VERSION_CONFLICT
COVER_LETTER_VERSION_CONFLICT
AI_CONFIG_VERSION_CONFLICT
```

Backend 绝不自动：

```text
reload latest → retry mutation
```

------

## C. Command input version

AI generation command显式携带它所依赖的 authoritative revisions。

这样不仅是 OCC，也是 provenance declaration。

------

## D. Idempotency-Key

对于可能因为：

- double-click；
- browser retry；
- network timeout；
- mobile/extension retry；

产生重复副作用的 POST command，我建议正式支持：

```http
Idempotency-Key: <opaque-client-generated-key>
```

适用：

```text
POST /resume-imports
POST /preparations
POST /jobs where needed
POST /match/generate
POST /resumes
POST /cover-letters
POST /resume-imports/{id}/accept
```

不是所有 GET/PUT 都需要。

------

## E. Idempotency key scope

建议：

```text
authenticated principal
+
endpoint/business command
+
idempotency key
```

作为 scope。

不同 user 即使 key 相同也不冲突。

------

## F. Same key same request

如果 request已成功接受：

> 返回之前的逻辑结果，不重复执行业务副作用。

例如第一次：

```text
202 resumeId=R1
```

network response lost。

再次相同 key：

```text
202/200 representation for R1
```

不能生成 R2。

------

## G. Same key different payload

必须：

```text
409 IDEMPOTENCY_CONFLICT
```

不能复用 key 做不同 command。

------

## H. Durable idempotency state

3.15 如果已经决定必要 durable state，则实现按 frozen DB design。

HTTP 这里只定义 semantics，不指定表名。

------

## I. Automatic retries vs client retries

automatic worker retry：

> 不依赖 HTTP Idempotency-Key。

它们由 existing business_event lifecycle管理。

Idempotency-Key 解决的是 HTTP command submission duplicate。

------

### 3.17.13 Frozen

> OfferBuddy exposes domain-specific optimistic-concurrency versions directly in HTTP contracts rather than adopting a generic ETag model. Any stale mutation or stale version-bound command fails with 409 and an explicit machine-readable conflict code; the backend never silently reloads newer authoritative state and retries the user’s mutation.
>
> Side-effecting POST commands support an `Idempotency-Key` contract where duplicate submission risk exists. Keys are scoped to the authenticated caller and command context. Repeating the same logical request with the same key reuses the previously accepted/resulting business operation, while reuse of the same key with a materially different request fails with `409 IDEMPOTENCY_CONFLICT`. HTTP idempotency is distinct from internal business-event retry semantics.

------

# 3.17.14 API Security & Data Exposure Rules

这一节把 3.14/3.16 落到所有 API。

## A. Authentication identity only from server context

不接受：

```text
userId
ownerId
candidateOwner
```

作为 ownership declaration。

------

## B. Authorised lookup first

优先：

```text
findByResourceIdAndOwner(...)
```

而不是：

```text
findById
→ then compare owner
```

避免 accidental data disclosure。

------

## C. 404 ownership masking

owned resource：

```text
not found
or wrong owner
→ 404
```

------

## D. Admin 是 privilege，不是 universal domain bypass

ROLE_ADMIN 不等于自动可以：

```text
read every Candidate
download every Resume
view every Job Description
```

必须有专门 privileged support capability 才能访问。

------

## E. DTO allowlist

Request DTO 只暴露需要字段。

永远不接受：

```text
createdBy
ownerUserId
eventId
correlationId
attemptCount
providerSecret
```

------

## F. Sensitive/raw content boundary

不返回：

```text
AI prompts
raw AI responses
raw HTML
cookies
auth tokens
secrets
provider credential
stack traces
SQL
internal event payload
```

------

## G. External content untrusted

Job description、Extension data、uploaded resume都必须：

- size limited；
- format validated；
- sanitized/normalized as appropriate；
- never interpreted as trusted instruction to backend/AI control plane。

尤其 Job text里的：

```text
Ignore previous instructions...
```

只是 Job content，不是 system instruction。

------

## H. AI input minimization

不同 AI capability只发送需要的数据。

例如 Match：

```text
Candidate factual snapshot
Job canonical/intelligence snapshot
```

不顺手把：

```text
full account info
unrelated applications
other resumes
```

发送给 provider。

------

## I. Error minimization

Client error不能包含：

```text
whether another user's resource exists
provider raw failure
database object names
class names
```

------

## J. Response minimization

API contract遵循：

> capability-required data only.

不能因为 DB entity里有字段就全部 serialize。

------

## K. Audit-sensitive admin mutation

AI config等 privileged mutations必须进入 3.16 audit trail：

```text
actor
action
target/config key
before/after safe metadata
requestId
timestamp
result
```

但 secrets不进入 audit content。

------

### 3.17.14 Frozen

> All OfferBuddy APIs treat every client—including Web, Extension and Admin—as outside the trusted domain boundary. Authentication identity comes only from backend security context, ownership is enforced through authorised resource lookups, and ownership mismatch normally collapses to 404. Administrative privilege does not create a universal bypass into Candidate or Preparation data.
>
> HTTP contracts follow strict data minimisation: persistence metadata, secrets, tokens, raw external/browser content, provider prompts/responses, event internals and diagnostic implementation details are excluded unless a specifically authorised capability requires otherwise. Uploaded resumes, job-page content and extension-derived data remain untrusted inputs throughout processing. AI calls receive only capability-required snapshots, and privileged operational mutations are auditable without recording plaintext secrets or unnecessary personal content.

------

# 3.17.15 Final API Contract Review

现在把整个 3.17 从头检查一遍。

------

## 1. Domain boundary consistency

API 已经与 3.1 module boundary 对齐：

```text
Candidate
Job
Application
Preparation
AI Platform/Admin
Extension
```

没有暴露 repositories/entities。

**PASS**

------

## 2. S2 Fast Path preservation

Application：

```text
does not require Candidate
does not require Preparation
does not require AI
```

S3 没有破坏它。

**PASS**

------

## 3. Candidate truth boundary

Candidate Profile：

```text
authoritative accepted facts
```

Resume Import：

```text
draft only until explicit acceptance
```

Match/Resume/Cover Letter：

```text
never write facts back
```

**PASS**

------

## 4. Preparation anchor

Preparation：

```text
Candidate + Job
```

不是：

```text
Application + Job
```

所有 Match/Resume/Cover Letter 都挂在 Preparation。

**PASS**

------

## 5. Job lifecycle

Canonical Job：

- 可以 Application 之前存在；
- 不 globally public；
- `contentVersion` 有 meaningful semantic meaning；
- Job Intelligence独立 derived。

**PASS**

------

## 6. Version/provenance chain

完整链：

```text
Candidate
profileVersion
      +
Job
contentVersion
      ↓
Job Intelligence
sourceContentVersion
      ↓
Match
sourceProfileVersion
sourceJobContentVersion
      ↓
Resume / Cover Letter
sourceProfileVersion
sourceJobContentVersion
sourceMatchId
```

再加 artifact自己的：

```text
Resume.version
CoverLetter.version
```

语义完全没有混淆。

**PASS**

------

## 7. Concurrency

所有 user-owned mutable authoritative/resource state都明确：

```text
Candidate → profileVersion
Application → version
Job semantic content → contentVersion
Resume → version
Cover Letter → version
Admin config → version
```

stale：

```text
409
```

不自动 merge。

**PASS**

------

## 8. Async model

所有外部/AI重操作：

```text
Resume Import
Job Intelligence
Match
Resume Generation
Cover Letter Generation
```

都允许 durable async。

普通 DB update：

```text
Profile update
Resume edit
Cover Letter edit
Import accept
Application update
```

保持短同步 transaction。

完全符合 3.13。

**PASS**

------

## 9. Business event boundary

HTTP 从未暴露：

```text
eventId
claimToken
lease
retry count
```

product status 和 internal event status 分离。

**PASS**

------

## 10. Idempotency

HTTP duplicate submission与 worker retry明确区分。

这与 3.15 一致。

**PASS**

------

## 11. Security

不存在这种 dangerous API：

```text
/api/users/{userId}/candidate
/api/admin/candidates/*
?admin=true
```

ownership全部 backend resolution。

**PASS**

------

## 12. AI governance

Business API不选择 provider。

例如没有：

```json
{
  "provider": "OPENAI"
}
```

这种 user request。

Provider routing属于 AI Platform/Admin。

**PASS**

------

## 13. Secrets

没有 secret从 Admin response回显。

**PASS**

------

## 14. Observability

`X-Request-ID`：

```text
all HTTP responses
```

error body也含 requestId。

`correlationId`：

```text
internal
```

完全符合 3.16。

**PASS**

------

## 15. API complexity

最终 S3 API 没有变成巨型 CRUD platform。

大体结构保持：

```text
/api/candidate/profile
/api/candidate/resume-imports/**

/api/jobs/**
/api/jobs/{id}/intelligence/**

/api/applications/**

/api/preparations/**
/api/preparations/{id}/match/**
/api/preparations/{id}/resumes/**
/api/preparations/{id}/cover-letters/**

/api/extension/**

/api/admin/ai/**
```

结构足够清楚。

**PASS**

------

# Final 3.17 API Surface

最终可以概括为：

```text
Candidate
GET  /api/candidate/profile
PUT  /api/candidate/profile

POST /api/candidate/resume-imports
GET  /api/candidate/resume-imports/{importId}
POST /api/candidate/resume-imports/{importId}/accept


Job
POST /api/jobs
GET  /api/jobs/{jobId}
PUT  /api/jobs/{jobId}

GET  /api/jobs/{jobId}/intelligence
POST /api/jobs/{jobId}/intelligence/generate


Application
Existing /api/applications/** S2 contract preserved


Preparation
POST /api/preparations
GET  /api/preparations/{preparationId}

GET  /api/preparations/{preparationId}/match
POST /api/preparations/{preparationId}/match/generate

POST /api/preparations/{preparationId}/resumes
GET  /api/preparations/{preparationId}/resumes/{resumeId}
PUT  /api/preparations/{preparationId}/resumes/{resumeId}

POST /api/preparations/{preparationId}/cover-letters
GET  /api/preparations/{preparationId}/cover-letters/{coverLetterId}
PUT  /api/preparations/{preparationId}/cover-letters/{coverLetterId}


Extension
POST /api/extension/jobs/import


Admin / AI Governance
GET/PUT APIs under /api/admin/ai/**
```

------

# 3.17 Final Frozen Decision

**3.17 API & Contract Detailed Design 可以冻结。**

最终原则是：

> OfferBuddy S3 HTTP APIs are capability-oriented, aggregate-aware and security-context-driven. They preserve the S2 Application Fast Path while introducing explicit Candidate + Job Preparation workflows for Job Intelligence, Match, Tailored Resume and Cover Letter. Candidate facts remain authoritative only after explicit user-controlled Profile mutation or Resume Import acceptance; AI-derived outputs remain separate artifacts with clear provenance and staleness rules.
>
> Domain revisions are explicit in contracts and are used consistently for concurrency and provenance. External/AI workflows use durable asynchronous commands and expose product-level resource state rather than event infrastructure. Mutable user artifacts have independent optimistic concurrency, while stale writes and stale generation commands fail explicitly rather than being silently merged or retried against newer state.
>
> All clients remain untrusted, ownership is resolved server-side, Admin is operational rather than domain-owning, and HTTP contracts minimise exposure of internal persistence, event, provider, secret and diagnostic data. Contract evolution remains additive-first and no unnecessary generic task platform, public Job catalogue, PATCH framework, or provider-specific API surface is introduced in S3.

**结论：3.17 完成，review 通过，可以 frozen。**

到这里，Phase 3 已经从 **3.1 一直到 3.17** 连成完整闭环：模块 → domain → DB → async → security → concurrency → observability → HTTP contract，暂时没有发现需要回滚前面 frozen decision 的矛盾。

下一节就应该进入 **3.18 Implementation / Package / Class-Level Technical Design**（如果我们原 Phase 3 outline 里下一节名称略有不同，就按原 outline 名称继续），开始把现在这些 contract 落成 Spring Boot package、Controller/Application Service/DTO/Mapper/Port 的具体结构。
