# Sprint 3 Implementation Structure and Class-Level Design

## Publication Note

This is the complete frozen Phase 3 §3.18 design record. Its approved package, layer, class-responsibility, dependency and implementation rules are authoritative within the frozen architecture and API contract. Workflow remarks are retained for traceability and do not extend scope.

3.18 我建议定义为：

# 3.18 Implementation Structure & Class-Level Design

它的作用不是重新讨论架构，而是把 **3.1–3.17 已冻结的设计映射成真正可编码的 Spring Boot / React / Extension / Admin 结构**。

建议大纲如下：

1. **3.18.1 Package & Module Structure**
   - backend package layout
   - candidate / job / application / preparation / ai / event / admin / extension ingestion
   - module dependency direction
   - shared/common 包的边界
2. **3.18.2 Controller & API Adapter Design**
   - Controller responsibilities
   - Request/Response DTO
   - validation
   - exception mapping
   - requestId handling
3. **3.18.3 Application Service / Use Case Design**
   - command/query service
   - transaction boundary
   - orchestration responsibility
   - 不让 Controller 和 Repository 承担业务逻辑
4. **3.18.4 Domain Model & Aggregate Implementation**
   - Candidate Profile aggregate
   - Job/contentVersion
   - Preparation
   - Match/Resume/Cover Letter lifecycle
   - domain invariants
5. **3.18.5 Repository & Persistence Adapter Design**
   - JPA entities
   - repositories
   - aggregate persistence
   - ownership-scoped query
   - OCC implementation
6. **3.18.6 Mapper / DTO Conversion Strategy**
   - Entity ↔ Domain
   - Domain ↔ API DTO
   - AI DTO ↔ Domain result
   - 避免 mapper proliferation
7. **3.18.7 Async Worker Implementation Structure**
   - BusinessEvent worker
   - handler registry
   - claim/process/complete/fail
   - retry handling
   - transaction boundaries
8. **3.18.8 AI Provider Integration Structure**
   - AI capability interface
   - provider adapter
   - router
   - feature config
   - structured output validation
   - provider-neutral domain result
9. **3.18.9 Candidate Resume Import Implementation Flow**
   - upload
   - extraction
   - draft generation
   - persistence
   - acceptance transaction
10. **3.18.10 Job Intelligence Implementation Flow**
    - generate command
    - event
    - worker
    - persistence
    - staleness
11. **3.18.11 Match Implementation Flow**
    - version checks
    - current intelligence check
    - async generation
    - provenance
12. **3.18.12 Tailored Resume & Cover Letter Implementation Flow**
    - generation
    - artifact persistence
    - user editing
    - OCC
13. **3.18.13 Security Implementation Structure**
    - authenticated principal
    - ownership lookup
    - admin authorization
    - extension authentication
    - sensitive data handling
14. **3.18.14 Error Handling & Observability Implementation**
    - global exception handler
    - ApiError
    - requestId filter
    - structured logs
    - metrics/audit hooks
15. **3.18.15 Frontend / Extension Client Integration Structure**
    - React API clients
    - query/state model
    - polling async resources
    - stale/conflict UX
    - Extension API adapter
16. **3.18.16 Testing Structure**
    - unit tests
    - application/service tests
    - repository integration tests
    - controller contract tests
    - async worker tests
    - state-based verification
    - critical end-to-end flows
17. **3.18.17 Implementation Dependency Order**
    - 哪些模块先做
    - 哪些可以并行
    - checkpoint 拆分
    - GitHub Issue → Codex checkpoint 映射
18. **3.18.18 Final Class-Level Design Review**
    - 检查 package dependency
    - transaction boundary
    - duplicated abstractions
    - over-engineering
    - implementation readiness

我尤其建议 **3.18 不要变成“把每个 class 全画出来”**。重点应该是把关键 class、职责、依赖方向和 transaction boundary 定清楚，剩下的让实现阶段自然展开。

而且 **3.18.17 Implementation Dependency Order** 对你后面实际用 Codex 很重要，因为可以直接把设计转换成你现在采用的“小 checkpoint + TDD + 小 diff”开发方式。



# 3.18 Implementation Structure & Class-Level Design

这一节我建议作为 **Phase 3 Technical Design 最后一层“代码落地约束”**。3.17 已经冻结了 HTTP/API Contract；3.18 不再设计新的业务能力，而是回答：

> **这些 contract、domain rules、async workflows、AI ports、repositories，最终应该放在哪些 package、由哪些 class 负责，以及 class 之间允许怎样依赖。**

这样后面 Agent 可以根据这一节直接拆 implementation task，而不是自己重新发明架构。

------

## 3.18.1 Implementation Design Principles

S3 继续采用现有 **Spring Boot modular monolith**，不拆 microservice，也不引入 heavyweight DDD framework。

推荐整体原则是：

**package by business capability first, layer inside capability second.**

不要做：

```text
controller/
service/
repository/
entity/
dto/
```

这种全局 horizontal package。

否则 Candidate、Preparation、Resume、Application 等模块会逐渐互相直接调用 repository/entity，前面 3.1 定义的模块边界会失效。

推荐：

```text
com.offerbuddy
├── application
├── candidate
├── job
├── preparation
├── resume
├── coverletter
├── ai
├── event
├── extension
├── admin
├── security
└── shared
```

每个业务 capability 内部再分：

```text
candidate/
├── api
├── application
├── domain
└── infrastructure
```

这里：

| Layer                 | Responsibility                                |
| --------------------- | --------------------------------------------- |
| `api`                 | HTTP contract adaptation                      |
| `application`         | use case orchestration / transaction boundary |
| `domain`              | domain state + business rules                 |
| `infrastructure`      | persistence / external integrations           |
| cross-module contract | deliberately exposed immutable interface      |

依赖方向：

```text
API
 ↓
Application
 ↓
Domain
 ↑
Infrastructure
```

但这不是要求所有 class 都机械地有 interface。

**Interface 只应该出现在真正存在 boundary / substitution / external dependency 的位置。**

例如：

```text
CandidateProfileRepository
AiCapability
JobContentFetcher
BusinessEventPublisher
```

合理。

而：

```text
CandidateProfileService
CandidateProfileServiceImpl
```

如果实际上永远只有一个 implementation，则没有价值。

------

# 3.18.2 Top-Level Module Structure

S3 推荐最终 backend package：

```text
com.offerbuddy
│
├── candidate
│   ├── api
│   ├── application
│   ├── domain
│   └── infrastructure
│
├── job
│   ├── api
│   ├── application
│   ├── domain
│   └── infrastructure
│
├── preparation
│   ├── api
│   ├── application
│   ├── domain
│   └── infrastructure
│
├── resume
│   ├── api
│   ├── application
│   ├── domain
│   └── infrastructure
│
├── coverletter
│   ├── api
│   ├── application
│   ├── domain
│   └── infrastructure
│
├── application
│   └── ... existing S2 structure
│
├── ai
│   ├── application
│   ├── domain
│   ├── infrastructure
│   └── governance
│
├── event
│   ├── application
│   └── infrastructure
│
├── extension
│   └── api
│
├── admin
│   └── api
│
├── security
│
└── shared
    ├── error
    ├── web
    ├── persistence
    └── util
```

有一个很重要的约束：

> **`shared` 不是垃圾桶。**

Candidate DTO、Job enums、Resume helper、AI prompt logic 都不能因为“其他地方可能用到”就移进 shared。

真正可以进入 `shared` 的东西应非常有限，例如：

```text
ApiError
RequestIdFilter
PageResponse
Clock abstraction
common persistence audit fields
```

------

# 3.18.3 Candidate Module

Candidate 是 S3 最重要的 ownership root 和事实源。

推荐：

```text
candidate/
├── api/
│   ├── CandidateProfileController
│   ├── ResumeImportController
│   └── dto/
│
├── application/
│   ├── CandidateProfileCommandService
│   ├── CandidateProfileQueryService
│   ├── ResumeImportService
│   └── port/
│       └── CandidateProfileSnapshotProvider
│
├── domain/
│   ├── CandidateProfile
│   ├── CandidateSkill
│   ├── CandidateExperience
│   ├── CandidateEducation
│   ├── CandidateCertification
│   ├── CandidateLanguage
│   └── CandidateProfileRepository
│
└── infrastructure/
    └── persistence/
```

### `CandidateProfileController`

职责仅为：

```text
authentication context
→ request validation
→ application service
→ API response mapping
```

它不能：

```java
candidateProfileRepository.findByUserId(...)
```

也不能自行判断：

```java
if (request.getProfileVersion() ...)
```

这些属于 application/domain。

------

### `CandidateProfileCommandService`

负责所有 meaningful mutation：

```text
create profile
update personal facts
replace/update skills
update experience
education
certifications
languages
eligibility
```

核心 transaction pattern：

```text
load owned Candidate
→ verify expected profileVersion
→ apply domain mutation
→ increment profileVersion
→ save
→ commit
```

`profileVersion` 的改变必须集中在这里或 aggregate 内，而不能散落在 controller/repository。

------

### `CandidateProfileQueryService`

只负责 Candidate-facing query。

不能成为其他 module 获取 Candidate 数据的万能入口。

其他模块应该依赖：

```java
CandidateProfileSnapshotProvider
```

返回：

```java
CandidateProfileSnapshot
```

而不是：

```java
CandidateProfileEntity
```

这正好落实之前冻结的：

> repositories/entities remain module-private.

------

# 3.18.4 Job Module

Job 模块建议：

```text
job/
├── api/
│   ├── JobController
│   └── dto/
├── application/
│   ├── JobCommandService
│   ├── JobQueryService
│   ├── JobIntelligenceService
│   └── port/
│       ├── JobSnapshotProvider
│       └── JobContentFetcher
├── domain/
│   ├── Job
│   ├── JobIntelligence
│   ├── JobRepository
│   └── JobIntelligenceRepository
└── infrastructure/
```

### `Job`

负责 canonical job content，包括：

```text
title
company
location
description
source
externalJobId
sourceUrl
contentVersion
```

`contentVersion` 不是 ORM version。

它只在：

> semantic content affecting Preparation / AI-derived output

变化时递增。

所以 infrastructure 的：

```text
lastFetchedAt
fetchStatus
technical metadata
```

不能导致 `contentVersion++`。

------

### `JobIntelligenceService`

负责：

```text
Job canonical content
        ↓
AI job-analysis capability
        ↓
JobIntelligence
```

不能直接调用 OpenAI/Anthropic/provider SDK。

必须：

```java
JobIntelligenceService
        ↓
JobAnalysisCapability
        ↓
AiRouter
        ↓
Provider Adapter
```

------

# 3.18.5 Preparation Module

Preparation 是 Candidate + Job 的 orchestration boundary。

结构：

```text
preparation/
├── api/
│   ├── PreparationController
│   ├── MatchController
│   └── dto/
├── application/
│   ├── PreparationService
│   ├── MatchAnalysisService
│   ├── PreparationQueryService
│   └── async/
│       └── MatchGenerationEventHandler
├── domain/
│   ├── Preparation
│   ├── MatchAnalysis
│   ├── PreparationRepository
│   └── MatchAnalysisRepository
└── infrastructure/
```

最关键的是：

```text
Preparation
    candidateId
    jobId
```

而不是：

```text
applicationId
```

Application 可以关联/进入 preparation workflow，但不能成为 Preparation aggregate root。

------

## `PreparationService`

负责：

```text
authorise Candidate ownership
resolve CandidateSnapshot
resolve JobSnapshot
create/find Preparation
decide whether existing preparation remains current
schedule required intelligence/match work
```

不要让 controller 自己拼：

```java
candidateService...
jobService...
matchService...
resumeService...
```

否则 HTTP layer 会变成 workflow engine。

------

# 3.18.6 Match Analysis

`MatchAnalysisService` 应该是一个独立 application service。

输入 conceptual model：

```java
CandidateProfileSnapshot
JobSnapshot
JobIntelligenceSnapshot
```

输出：

```java
MatchAnalysis
```

保存：

```text
candidateProfileVersion
jobContentVersion
jobIntelligenceVersion
AI provenance
generatedAt
```

这样 staleness：

```text
current profileVersion != source profileVersion
OR
current job contentVersion != source jobContentVersion
```

可以 deterministic 判断。

不能要求 AI 重新分析：

> “这个 match 是否 stale？”

staleness 是 application rule，不是 AI judgement。

------

# 3.18.7 Resume Module

推荐：

```text
resume/
├── api/
│   ├── TailoredResumeController
│   └── dto/
├── application/
│   ├── TailoredResumeService
│   ├── TailoredResumeQueryService
│   └── async/
│       └── ResumeGenerationEventHandler
├── domain/
│   ├── TailoredResume
│   ├── TailoredResumeRepository
│   └── ResumeGenerationPolicy
└── infrastructure/
```

### `TailoredResumeService`

核心职责：

```text
authorise preparation
→ obtain CandidateSnapshot
→ obtain Job/Intelligence/Match snapshots
→ verify prerequisites
→ create generation intent
→ persist PENDING business state + event
→ return
```

真正 AI call 由 worker/event handler 执行。

绝对不要：

```java
@Transactional
public Resume generate(...) {
    repository.save(...);
    openAiClient.call(...);
    repository.save(...);
}
```

因为这违反已经冻结的 async transaction boundary。

------

# 3.18.8 Cover Letter Module

Cover Letter 与 Resume 保持同样 architecture pattern：

```text
coverletter/
├── api/
├── application/
├── domain/
└── infrastructure/
```

但不要为了减少代码做：

```java
AbstractAiDocumentGenerator<T>
```

然后 Resume / Cover Letter 继承它。

这两个现在业务规则相似，但未来非常容易不同。

可以共享的是 AI infrastructure mechanism：

```text
AI routing
metrics
provider execution
failure mapping
```

不能共享它们的 domain workflow。

------

# 3.18.9 AI Module

AI 推荐明确拆成两个概念：

```text
business semantic capabilities
          ↓
AI routing/runtime governance
          ↓
provider adapters
```

对应：

```text
ai/
├── domain/
│   ├── JobAnalysisCapability
│   ├── MatchAnalysisCapability
│   ├── ResumeGenerationCapability
│   ├── CoverLetterGenerationCapability
│   └── ResumeExtractionCapability
│
├── application/
│   ├── AiRouter
│   ├── AiExecutionService
│   └── AiFeaturePolicy
│
├── governance/
│   ├── AiProviderConfigurationService
│   ├── AiFeatureConfigurationService
│   └── AiOperationalQueryService
│
└── infrastructure/
    ├── openai/
    ├── anthropic/
    └── ...
```

业务模块依赖：

```java
ResumeGenerationCapability
```

而不是：

```java
AiService.generate(prompt)
```

这一点非常重要。

禁止出现这种万能接口：

```java
String chat(String prompt);
```

否则所有 prompt engineering、schema interpretation、business semantics 都会泄露到 Candidate/Resume/Job service。

------

# 3.18.10 Business Event / Async Worker

继续复用已经存在的 `business_events` infrastructure。

不再新增：

```text
TaskService
AsyncTaskManager
OperationRepository
OutboxRepository
```

推荐 runtime shape：

```text
BusinessEventDispatcher
       ↓
BusinessEventHandlerRegistry
       ↓
typed handler
```

例如：

```text
MatchGenerationEventHandler
ResumeGenerationEventHandler
CoverLetterGenerationEventHandler
JobIntelligenceEventHandler
```

handler pattern：

```text
claim event
    ↓
load business resource
    ↓
verify work still valid
    ↓
execute external/AI work
    ↓
short transaction
    ↓
update business resource
    ↓
mark event succeeded
```

特别需要 **revalidation**。

例如 Resume event 被创建时：

```text
profileVersion = 7
```

worker 开始运行前 Candidate 已变成：

```text
profileVersion = 8
```

worker 不能偷偷用 8 重新生成，也不能把 7 生成出的结果标成 current。

应该根据具体 workflow：

```text
abort/supersede/stale
```

并保留 provenance。

------

# 3.18.11 Persistence Structure

我建议继续避免把 JPA entity 直接作为 domain/API object。

例如：

```text
domain/
    CandidateProfile

infrastructure/persistence/
    CandidateProfileEntity
    CandidateProfileJpaRepository
    CandidateProfileRepositoryAdapter
```

逻辑：

```text
Application Service
       ↓
CandidateProfileRepository       ← domain/application boundary
       ↑
CandidateProfileRepositoryAdapter
       ↓
CandidateProfileJpaRepository
       ↓
JPA
```

如果现有 OfferBuddy S1/S2 没有这么严格地分 domain entity / persistence entity，也**不需要为了 S3 重写整个项目**。

这里采用 pragmatic rule：

> 新的复杂 S3 aggregate 优先保持 persistence implementation private；现有简单 S2 code 不进行无收益重构。

也就是避免“为了架构漂亮”把 Sprint 1/2 全部翻掉。

------

# 3.18.12 API DTO Boundary

3.17 中定义的 API request/response 必须有明确 DTO。

例如：

```text
UpdateCandidateProfileRequest
CandidateProfileResponse
CreatePreparationRequest
PreparationResponse
MatchAnalysisResponse
GenerateTailoredResumeRequest
TailoredResumeResponse
```

禁止：

```java
@PostMapping
CandidateProfileEntity update(
    @RequestBody CandidateProfileEntity entity
)
```

API DTO 和 persistence entity 永远不是同一个 object。

同时：

```text
Request DTO
≠ Domain Command
```

对于简单 case 可以直接 mapping。

复杂 mutation 推荐：

```text
UpdateCandidateProfileRequest
        ↓
UpdateCandidateProfileCommand
        ↓
CandidateProfileCommandService
```

但不要机械地给每个 GET 都创建 command/query class。

------

# 3.18.13 Security Ownership Resolution

安全 ownership 逻辑不能散落：

```java
if (!entity.getUserId().equals(currentUserId))
```

推荐形成统一 server-side resolver，例如：

```text
CurrentUser
CandidateAccessService
ApplicationAccessService
PreparationAccessService
```

典型调用：

```java
CandidateProfile profile =
    candidateAccess.requireOwnedCandidate(candidateId);
```

Preparation：

```java
Preparation preparation =
    preparationAccess.requireOwnedPreparation(preparationId);
```

内部仍然应该优先通过 authorised repository lookup：

```text
candidateId + currentUserId
```

而不是：

```text
findById
→ compare owner
```

这与 3.14 的信息泄露边界保持一致。

------

# 3.18.14 Concurrency & Idempotency Classes

OCC 不应由 controller 实现。

Candidate：

```text
CandidateProfileCommandService
    expectedProfileVersion
         ↓
CandidateProfile
         ↓
profileVersion check
```

Application：

```text
ApplicationCommandService
    expectedVersion
```

Job：

```text
JobCommandService
    semantic change detection
    contentVersion
```

重复 POST：

```text
IdempotencyService
IdempotencyRecordRepository
```

如果之前 3.15 最终决定利用现有 infrastructure/state，就沿用那个实现，而不要每个 module 自己写：

```text
ResumeIdempotencyService
CoverLetterIdempotencyService
PreparationIdempotencyService
```

Idempotency 属于 HTTP/business command execution concern，不是 domain entity 本身。

------

# 3.18.15 Exception Design

建议保持少量明确 application exceptions：

```text
ResourceNotFoundException
ConflictException
StaleVersionException
InvalidStateException
CapabilityUnavailableException
ExternalDependencyException
```

不要：

```text
CandidateNotFoundException
JobNotFoundException
ResumeNotFoundException
CoverLetterNotFoundException
PreparationNotFoundException
...
```

无限 expansion。

API 层通过统一：

```text
GlobalExceptionHandler
```

映射到 3.17 冻结的：

```text
400
401
403/404
409
422
503
...
```

domain/application code 不应该知道：

```java
HttpStatus.CONFLICT
```

------

# 3.18.16 Class-Level Dependency Example

完整的 Tailored Resume 路径应该类似：

```text
TailoredResumeController
        │
        ▼
TailoredResumeService
        │
        ├── PreparationAccessService
        │
        ├── CandidateProfileSnapshotProvider
        │
        ├── JobSnapshotProvider
        │
        ├── MatchAnalysisSnapshotProvider
        │
        ├── TailoredResumeRepository
        │
        └── BusinessEventPublisher
                         │
                         ▼
               business_events
                         │
                         ▼
            ResumeGenerationEventHandler
                         │
                         ├── ResumeGenerationCapability
                         │          │
                         │          ▼
                         │       AiRouter
                         │          │
                         │          ▼
                         │    Provider Adapter
                         │
                         └── TailoredResumeRepository
```

注意这里不存在：

```text
Controller
 → OpenAI
```

不存在：

```text
ResumeService
 → CandidateRepository
```

也不存在：

```text
ResumeService
 → JobJpaRepository
```

跨模块拿的是 contract/snapshot，而不是别人的 persistence model。

------

# 3.18.17 Cross-Module Contract Rule

这是我认为应该明确写进 Technical Design 的一条硬规则：

> **A module may expose application-level immutable contracts, but must not expose repositories, persistence entities, provider-specific DTOs, or mutable domain aggregates for direct use by another module.**

典型 contract：

```java
CandidateProfileSnapshotProvider
JobSnapshotProvider
MatchAnalysisSnapshotProvider
```

对应 immutable snapshot：

```text
CandidateProfileSnapshot
JobSnapshot
JobIntelligenceSnapshot
MatchAnalysisSnapshot
```

Snapshot 中只暴露 downstream capability 真正需要的数据。

不能因为方便直接做：

```java
CandidateProfile getCandidate(...)
```

否则 Resume/Match 很快就会依赖 Candidate aggregate internal structure。

------

# 3.18.18 Naming Conventions

Agent implementation 时统一使用以下 semantics：

| Suffix              | Meaning                                  |
| ------------------- | ---------------------------------------- |
| `Controller`        | HTTP boundary                            |
| `CommandService`    | state-changing application use cases     |
| `QueryService`      | read use cases                           |
| `Service`           | meaningful application/domain capability |
| `Repository`        | domain persistence abstraction           |
| `RepositoryAdapter` | persistence implementation               |
| `JpaRepository`     | Spring Data repository                   |
| `Capability`        | semantic AI/external capability boundary |
| `Provider`          | cross-module snapshot/read contract      |
| `Adapter`           | external/infrastructure implementation   |
| `EventHandler`      | async business event consumer            |
| `Request`           | HTTP input                               |
| `Response`          | HTTP output                              |
| `Command`           | internal mutation input                  |
| `Snapshot`          | immutable cross-module read model        |

尤其不要滥用：

```text
Manager
Helper
Utils
Processor
Handler
```

如果一个 class 叫：

```java
ResumeHelper
```

通常说明 responsibility 还没有设计清楚。

------

# 3.18.19 Transaction Boundary

Application service 是主要 transaction owner。

典型 synchronous mutation：

```text
Controller
     ↓
@Transactional Application Service
     ↓
Domain + Repository
```

Transaction 内允许：

```text
DB read
domain validation
DB write
business_event creation
```

Transaction 内禁止：

```text
AI call
HTTP fetch
browser/network call
email
provider SDK
long-running parsing
```

AI/external workflow：

```text
TX 1
business state + event
COMMIT

↓

external/AI execution

↓

TX 2
result / failure update
COMMIT
```

这应该成为 Agent 实现 S3 时非常明确的约束。

------

# 3.18.20 Testing Boundary

class design 同时决定测试粒度。

推荐：

```text
Domain rule
→ unit test

Application service orchestration
→ service test with mocked ports

Repository adapter
→ PostgreSQL integration test

Controller/API contract
→ MVC/API test

Business event handler
→ handler integration/service test

Provider adapter
→ contract/integration test

Critical workflow
→ limited end-to-end test
```

不要给每一个 trivial DTO mapper 写大量 unit test。

重点应该覆盖之前冻结的风险：

```text
ownership
OCC
idempotency
staleness
async retries
duplicate events
provider failure
AI malformed result
cross-module boundaries
```

------

# 3.18.21 Agent Implementation Guardrails

因为你准备让 Agent 承担更多实现，这一部分值得明确成为 repository instruction。

Agent 生成 S3 code 时必须遵循：

1. Existing V1–V9 migrations immutable；新增 DB change 只能 additive migration。
2. 不改变 S2 Application Fast Path。
3. Preparation anchor 永远是 Candidate + Job。
4. 不把 Candidate/Job/Application persistence entity 暴露给其他 module。
5. 不让业务 module 直接依赖 AI provider SDK。
6. 不在 DB transaction 中执行 AI/network work。
7. 不自行创建新的 generic async/task infrastructure。
8. 不绕过现有 ownership/security context。
9. 不静默解决 stale OCC conflict。
10. 不因为方便把业务 class 塞入 `shared`。
11. 不发明 3.17 以外的新 API contract。
12. 如果实现需要违反 frozen design，必须停止并把 contradiction 提出来，而不是自行修改 architecture。

这一组我会建议后面直接复制一部分进入 Agent coding instructions。

------

# 3.18.22 Final Implementation Shape

最终 S3 的运行结构可以压缩成：

```text
React / Extension / Admin
            │
            ▼
        API Controllers
            │
            ▼
     Application Services
       │      │       │
       │      │       └──── Business Events
       │      │                   │
       │      ▼                   ▼
       │    Domain          Async Handlers
       │      │                   │
       ▼      ▼                   ▼
 Cross-module Ports          AI Capabilities
       │                          │
       ▼                          ▼
   Snapshots                    AI Router
                                  │
                                  ▼
                           Provider Adapter

          Persistence Adapters
                  │
                  ▼
              PostgreSQL
```

这跟我们前面冻结的所有设计是完全一致的：

**Candidate owns facts → Preparation coordinates Candidate + Job → derived artifacts consume immutable snapshots → AI is behind semantic capability ports → async work uses `business_events` → persistence stays module-private → API/security/concurrency boundaries remain server-owned.**

------

## 3.18 Review

我认为 **3.18 可以批准并冻结**，而且这一节有一个很实际的价值：

**到这里 Phase 3 已经从“architecture”走到了“Agent 可以按 class 开始写”的精度。**

尤其是现在已经明确：

```text
what modules exist
what each layer owns
what the important classes are
where transaction boundaries sit
how modules communicate
how async handlers work
where AI belongs
where security/OCC/staleness belong
what Agent is forbidden to invent
```

因此 **不建议再继续往 `3.19 Detailed Method Signatures` 这种层次走**。再往下就已经进入 implementation，而不是 Technical Design 了。

我的建议是把 **3.18 Implementation Structure & Class-Level Design 作为 Phase 3 Technical Design 的最后一个正式设计章节**。下一步进入 **Phase 3 Final Technical Design Review / freeze**，然后就可以开始你之前计划的文档同步与 Agent implementation preparation。
