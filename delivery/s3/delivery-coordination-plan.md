# Sprint 3 Delivery Units and Two-Agent Coordination

## Purpose

This unit defines implementation-sized delivery boundaries and a safe two-agent coordination model. It does not create GitHub Issues, assign permanent machine ownership, or replace the final acceptance criteria in frozen design/page specifications.

## Delivery-unit rules

Each delivery unit must:

- produce one coherent capability or shared prerequisite;
- have explicit entry and exit evidence;
- keep schema, backend, client, tests, and documentation visible when they belong to the same business change;
- avoid bundling unrelated cleanup;
- remain small enough for one focused PR and state-based verification;
- stop if implementation requires a frozen contract change.

A delivery unit may use several Agent coding checkpoints. GitHub Issue remains the delivery unit; an Agent checkpoint remains the coding unit.

## Planned delivery units

| ID | Delivery unit | Entry dependency | Included outcome | Excludes |
| --- | --- | --- | --- | --- |
| `S3-F01` | Shared revisions and diagnostics | V9 baseline | V10 aggregate versions, correlation propagation, frozen common errors/OCC tests | Candidate/Preparation features |
| `S3-F02` | AI capability runtime foundation | Shared API/security conventions | Semantic ports, provider adapter reuse, router/config/execution metadata, safe fake providers | Business prompts and feature UI |
| `S3-C01` | Candidate Profile backend/API | V11; `S3-F01` | Owned Candidate aggregate with `profileVersion` and contract tests | Web UI and Resume Import |
| `S3-C02` | Candidate Profile Web | `S3-C01` contract | `S3-UI-06`, `08`–`14` overview/edit states | Resume Import review |
| `S3-C03` | Resume Import vertical slice | `S3-C01`, `S3-F02`, V12 | Upload, async extraction, draft review UI, explicit accept/merge | OCR and Resume Library |
| `S3-J01` | Job revision and Intelligence delta | `S3-F01`; existing Job/Intelligence | Prepared Job lifecycle, `contentVersion`, source-version-aware Intelligence | Match |
| `S3-P01` | Preparation context | `S3-C01`, `S3-J01`, V13 | Candidate + Job workspace, readiness and summary contracts | Match generation |
| `S3-P02` | Match Analysis vertical slice | `S3-P01`, `S3-F02` | Async grounded Match and `S3-UI-01` | Resume/Cover Letter |
| `S3-R01` | Resume storage/rendering foundation | Object-storage/rendering contract, V14 | Base documents/assets, authorised preview/download, failure isolation | Tailoring AI |
| `S3-R02` | Tailored Resume vertical slice | `S3-P02`, `S3-R01`, `S3-F02` | `S3-UI-02`–`04`, generation, limited edit/OCC, regeneration | Full Resume Designer |
| `S3-L01` | Focused Cover Letter vertical slice | `S3-P02`, `S3-F02`, V15 | `S3-UI-05`, generation/edit/re-render | Default/Fast template fill |
| `S3-A01` | Application Detail integration | `S3-P01`; existing Application page | `S3-UI-17` additive Preparation summary | New Application Detail page |
| `S3-E01` | LinkedIn Site Adapter | Existing Extension adapter contract | Reliable LinkedIn extraction/lifecycle fixtures | Preparation backend and UI redesign |
| `S3-E02` | Floating assistant and Preparation hand-off | `S3-E01`, `S3-P01` contract | `S3-UI-15` states, capture/reuse/open flow | Match or editors inside Extension |
| `S3-S01` | Sponsor persistence/publication backend | V18; RuoYi security | Working/import correction, validate/publish, Redis/published projection | Generic Sponsor product |
| `S3-S02` | Sponsor Admin and Extension signal | `S3-S01` contract | `S3-UI-18`, snapshot cache/refresh, local page lookup | Second source of truth or per-page required request |
| `S3-Q01` | Default/SEEK Cover Letter assistance | Candidate tool contract; Extension SEEK adapter | `S3-UI-16`, deterministic Default fill and fresh Focused source priority | Auto Apply and AI on Fast Path |
| `S3-O01` | AI Admin configuration | `S3-F02`, V16–V17 | `S3-UI-19` RuoYi runtime configuration with RBAC/OCC | Prompt editor/secrets manager |
| `S3-O02` | AI monitoring | `S3-F02`; safe telemetry | `S3-UI-20` metadata aggregates/failure detail | Raw prompt/response/user-data browser |
| `S3-I01` | Cross-surface integration and release candidate | All accepted units | End-to-end Focused/Fast paths, regression, migration upgrade, operational readiness | New scope |

IDs are planning references, not created Issue numbers. A unit may be split before Issue creation when its diff or risk cannot remain reviewable; it must not be merged with an unrelated unit merely to reduce Issue count.

## Two-agent delivery model

Use one short-lived feature branch per delivery unit in the private source repository, normally targeting `release/sprint-3`. The engineering repository remains on `docs/sprint-3` for planning and later documentation reconciliation.

### Shared-foundation phase

`S3-F01` owns high-conflict schema/core API/event files and should complete before broad parallel work. `S3-F02` may proceed in parallel only after semantic AI port names and common execution metadata are agreed; it should avoid the same migration and error-handler files.

### Parallel lanes after foundations

| Lane A candidate | Lane B candidate | Why overlap is limited |
| --- | --- | --- |
| `S3-C01` Candidate backend | `S3-E01` LinkedIn adapter | Backend/Flyway versus isolated Extension adapter/fixtures |
| `S3-C02` Candidate Web | `S3-J01` Job delta | Web Candidate routes/components versus backend Job module |
| `S3-C03` Resume Import | `S3-P01` Preparation context after Candidate contract | Separate capability modules; coordinate event registry/migration ownership |
| `S3-P02` Match | `S3-R01` storage/rendering foundation | Preparation AI versus infrastructure adapter/asset flow |
| `S3-R02` Tailored Resume | `S3-L01` Cover Letter | Separate artefact modules after shared generation contract |
| `S3-S01` Sponsor backend | `S3-O01` AI Admin UI against contract | Sponsor service/schema versus RuoYi AI surfaces |
| `S3-S02` Sponsor clients | `S3-O02` AI monitoring | Extension/Admin Sponsor files versus monitoring APIs/views |

These are safe pairings, not permanent machine assignments. Choose the next pair from accepted prerequisites and current file ownership.

## Sequential and high-conflict areas

Do not develop these concurrently without an explicit single owner:

- V10–V18 migration numbering and shared schema revisions;
- common API errors, request context, security configuration, and OpenAPI setup;
- `business_events` entity/claim/processor contracts and handler registration;
- Preparation generation tables and artefact heads;
- Web `App.tsx`, top navigation, shared API types/client, and route guards;
- Extension messages, adapter export registry, manifest permissions, and companion state resolver;
- RuoYi menu/permission SQL and shared Admin API utilities;
- production Compose/environment configuration.

The owner of a shared file publishes the contract/checkpoint before dependent branches rebase. Avoid parallel “temporary” versions of the same DTO, event name, migration, or UI state enum.

## Branch and PR protocol

1. Start each code unit from the current verified `release/sprint-3` integration point.
2. Name branches by focused capability/Issue once Issues exist; do not reuse the documentation branch convention in the source repository.
3. Keep migrations immutable after merge and reserve the next migration name before concurrent schema work.
4. Rebase or update from the integration branch before final verification; never resolve semantic conflicts by choosing both versions.
5. PR descriptions identify frozen design/page specs, schema/config effects, verification evidence, and known limitations.
6. Merge prerequisite units before dependants unless a contract-only parallel branch has been explicitly approved.
7. Run integration/regression at the integration branch, not only on feature branches.

## Agent checkpoint protocol

Within each delivery unit, use small checkpoints:

```text
contract/test boundary
→ minimal implementation
→ focused tests
→ state-based verification
→ review diff
→ commit
→ next checkpoint
```

Agents must inspect existing code before adding abstractions, preserve unrelated worktree changes, and stop on frozen-design contradiction. No checkpoint should attempt an entire multi-surface delivery unit in one prompt.

## Coordination readiness

The units are ready to become GitHub Issues in a later planning/action step. Before Issue creation, confirm the current source integration branch, assign one owner for each high-conflict foundation, and attach authoritative design/page-spec links. This document does not start implementation or allocate final machine ownership.

Expanded Issue-ready definitions, execution waves, TDD/state verification requirements, and migration ownership are in [GitHub Issue Backlog (Two-Agent)](github-issue-backlog.md).
