# Stage A 双本体契约冻结实施计划

> **面向执行代理：** 必须按任务逐项执行并在每项后复核；推荐使用子代理驱动开发或计划执行工作流。所有步骤使用复选框跟踪。

**目标：** 完成双本体三层架构 Stage A 的需求调研事实源、规划基线、可直接实施的详细 LLD、机器可读契约、ADR、治理矩阵、执行台账、验收场景与自动化门禁，严格建立“规划 → 设计 → 执行”三级文档体系，使仓库具备进入 Stage B 的可审计条件。

**唯一真实依据原则：**

1. 用户需求调研事实源记录“用户是谁、要解决什么、为何重要、证据是什么”，使用稳定 `UR-*` 标识；未经需求事实源登记和确认的功能不得进入规划。
2. Tier-1 规划层记录目标、范围、优先级、阶段、约束和成功指标，使用 `PR-*` 标识并引用 `UR-*`；不得复制 Tier-2 接口和数据模型。
3. Tier-2 设计层以完整 LLD 作为唯一实施入口和原子设计清单，使用 `DR-*`、`ADR-*`、`SCN-*` 标识并引用 `PR-*`。ADR 与机器可读契约是 LLD 的规范性组成部分，必须由 LLD 以路径、版本和摘要固定；实施者只能从已批准 LLD 入口读取该组设计，不能绕过 LLD 选择单个产物。
4. Tier-3 执行层分为设计生产任务 `DTASK-*` 与实现任务 `TASK-*`：`DTASK-*` 引用已生效 `PR-*`，只产出或修订 LLD/ADR/契约；`TASK-*` 必须引用已批准 `DR-*`，只实现代码、迁移、配置、测试和交付证据。任何执行任务不得发明需求、接口、数据模型或安全策略。
5. 同一事实只允许一个规范性定义位置：架构取舍只定义在 ADR，接口与数据语法只定义在机器可读契约，模块边界、行为、非功能和实施映射只定义在 LLD；其他位置只能引用。LLD 清单固定所有组成物摘要，任一重复定义、摘要不匹配或语义冲突均直接阻断，不通过优先级静默裁决。跨层冲突按“已确认需求 → 生效规划 → 已批准 LLD 原子设计基线 → 执行产物”定位上游并发起变更。
6. 任何层级变更必须向上追溯来源、向下执行影响分析；需求未确认、规划未生效、LLD 未达到 Definition of Ready、追踪断链或执行偏离设计时，CI 必须阻断下一层。

**架构：** 先冻结用户需求与三级文档治理，再冻结共享标识、版本、错误和引用规则；随后以 MOD-15 为业务本体词汇源、MOD-17 为策略和审计事实源、MOD-16 为动作执行强制点、MOD-18 为隔离模拟执行边界。Agent、Workflow、Capability 和 RAG 模块只消费上述受治理契约，不保存源系统凭据、不直接调用源系统，也不建立第二套权限或审计事实源。

**技术栈：** Markdown、OpenAPI 3.1、JSON Schema 2020-12、Cypher、YAML、Python 3.12、pytest、PyYAML、jsonschema、openapi-spec-validator、GitHub Actions。Stage A 同步将 `documentation-system.md` 的 OpenAPI 规范从 3.0 升级为 3.1，仓库不得同时保留两个活动标准。

**Stage A 文档校验分类：**

- 完整 LLD：LLD-07/08/09/15/16/17/18，执行完整 Tier-2 模板、Given/When/Then、稳定场景 ID 和 3 核心/2 边界/2 异常校验。
- 影响分析：LLD-10/12/14，仅校验影响范围、职责边界、依赖、迁移约束和目标实现阶段；不得在 Stage A 要求完整设计或运行时目录。
- 迁移与废弃声明：LLD-13，仅校验 `Deprecated / Migration-only`、兼容期、转换与转发规则、禁止新增项、移除条件和验收判据。

---

## 一、文件结构与职责

### 新建需求、规划与执行治理文件

- `user-research-and-requirements.md`：用户需求调研事实源，保存来源、原始表述、研究限制、洞察、需求、确认状态和变更历史。
- `contracts/schemas/governance/user-requirement.v1.schema.json`：`UR-*` 机器可读条目 Schema。
- `contracts/schemas/governance/planning-requirement.v1.schema.json`：`PR-*` 规划条目 Schema。
- `contracts/schemas/governance/design-requirement.v1.schema.json`：`DR-*` 设计要求 Schema。
- `contracts/schemas/governance/design-task.v1.schema.json`：`DTASK-*` 设计生产任务 Schema。
- `contracts/schemas/governance/execution-task.v1.schema.json`：`TASK-*` 实现任务与证据 Schema。
- `contracts/schemas/governance/research-evidence.v1.schema.json`：调研来源、原文摘要与确认材料 Schema。
- `contracts/schemas/governance/requirements-traceability.v1.schema.json`：追溯节点、边、版本与完整性 Schema。
- `contracts/schemas/governance/execution-ledger.v1.schema.json`：执行台账集合、状态转换和证据 Schema。
- `contracts/schemas/governance/stage-traceability-view.v1.schema.json`：阶段发布追溯视图 Schema。
- `contracts/traceability/requirements-traceability.v1.yaml`：需求 → 规划 → 设计 → 执行 → 测试 → 验收的全生命周期双向追踪矩阵。
- `contracts/execution/stage-a-execution-ledger.v1.yaml`：仓库内唯一执行状态台账，替代外部 `topics.md` 和散落报告中的活动状态。

### 新建设计与决策文件

- `lld-08-workflow-orchestration.md`：基于 Ontology Action 的 Workflow/Hybrid 契约。
- `lld-10-multi-agent-collaboration.md`：多 Agent 双本体影响分析。
- `lld-12-skill-library.md`：Capability → ActionType 绑定契约。
- `lld-13-business-connectors.md`：MOD-13 迁移与废弃声明。
- `lld-14-governance-config.md`：Agent/Workflow 治理配置边界。
- `lld-15-ontology-service.md`：业务本体语义层契约。
- `lld-16-source-connector-runtime.md`：连接、映射、同步、写回、幂等和补偿契约。
- `lld-17-ontology-governance.md`：策略、版本审批、血缘和审计契约。
- `lld-18-ontology-simulation.md`：隔离模拟与结果对比契约。
- `lld-19-document-governance.md`：需求、规划、设计、执行、追溯、台账和门禁工具的完整 LLD；对应 `scripts/check_*.py`，不是业务运行时模块。
- `lld-15/sections/*.md`、`lld-16/sections/*.md`、`lld-17/sections/*.md`、`lld-18/sections/*.md`、`lld-19/sections/*.md`：当单份完整 LLD 超过可审阅规模时，按组件、接口、数据、状态机、安全、可靠性和验收拆分的规范性子设计；主 LLD 仅保存索引、边界和决策摘要，子文档不得重复定义同一事实。
- `adr/0001-postgresql-neo4j-storage-boundary.md`：双存储职责。
- `adr/0002-ontology-action-only-execution.md`：Action-only 执行。
- `adr/0003-mcp-is-tool-protocol-not-source-protocol.md`：MCP 与连接器边界。
- `adr/0004-schema-versioning-and-compatibility.md`：版本兼容规则。
- `adr/0005-outbox-replay-and-eventual-consistency.md`：Outbox 与重放。
- `adr/0006-saga-idempotency-and-compensation.md`：幂等、Saga 和补偿。
- `adr/0007-governance-and-audit-system-of-record.md`：治理与审计事实源。
- `adr/0008-simulation-production-isolation.md`：模拟隔离。
- `adr/0009-mod13-deprecation-and-mod16-migration.md`：MOD-13 迁移。

### 新建机器可读契约

- `contracts/openapi/*.v1.yaml`：MOD-15～18 API 契约。
- `contracts/schemas/common/*.schema.json`：标识、用户上下文、错误、分页等共享模型。
- `contracts/schemas/ontology/*.schema.json`：本体、连接、治理、模拟核心模型。
- `contracts/schemas/events/*.schema.json`：执行、授权、审计、血缘事件。
- `contracts/domains/{crm,erp,oa}/ontology.v1.json`：三域 Schema 包。
- `contracts/cypher/*.cypher`：平台约束与三域图模型。
- `contracts/governance/responsibility-matrix.v1.yaml`：职责唯一性矩阵。
- `contracts/traceability/stage-a-traceability.v1.yaml`：条款到 LLD、契约、ADR、场景和检查项的追踪。

### 新建门禁实现与测试

- `scripts/check_doc_consistency.py`：LLD、章节、场景、链接和 MOD ID 校验。
- `scripts/check_contracts.py`：OpenAPI、JSON Schema、三域、Cypher 和治理矩阵校验。
- `scripts/check_stage_a_gate.py`：聚合 Stage A 门禁并返回确定性退出码。
- `tests/stage_a/test_doc_consistency.py`：文档检查正反例。
- `tests/stage_a/test_contracts.py`：契约检查正反例。
- `tests/stage_a/test_stage_a_gate.py`：聚合门禁退出码测试。
- `requirements-stage-a.txt`：固定校验依赖版本。

### 修改现有文件

- `lld-07-knowledge-base.md`：限定为非结构化知识与 RAG。
- `lld-09-agent-runtime.md`：改为 ActionTypeRef、PolicyBindingRef 驱动。
- `.github/workflows/doc-check.yml`：改为 Stage A Python 门禁。
- `documentation-system.md`：统一 OpenAPI 3.1、三类 Stage A 文档模板和分阶段测试门禁。
- `agent-ontology-master-plan.md`：消除 LLD-10/12/14 阶段歧义，并统一 LLD-15→17→16→18 的契约冻结顺序。
- `dual-ontology-three-layer-architecture-baseline.md`：澄清只读与副作用 Action 的幂等、补偿和人工恢复条件。

---

## 二、实施任务

### 任务 0：冻结用户需求事实源与三级文档治理

**文件：**
- 新建：`user-research-and-requirements.md`
- 新建：`lld-19-document-governance.md`
- 新建：`lld-19/sections/*.md`
- 新建：`contracts/schemas/governance/user-requirement.v1.schema.json`
- 新建：`contracts/schemas/governance/planning-requirement.v1.schema.json`
- 新建：`contracts/schemas/governance/design-requirement.v1.schema.json`
- 新建：`contracts/schemas/governance/design-task.v1.schema.json`
- 新建：`contracts/schemas/governance/execution-task.v1.schema.json`
- 新建：`contracts/schemas/governance/research-evidence.v1.schema.json`
- 新建：`contracts/schemas/governance/requirements-traceability.v1.schema.json`
- 新建：`contracts/schemas/governance/execution-ledger.v1.schema.json`
- 新建：`contracts/schemas/governance/stage-traceability-view.v1.schema.json`
- 新建：`contracts/traceability/requirements-traceability.v1.yaml`
- 新建：`contracts/execution/stage-a-execution-ledger.v1.yaml`
- 修改：`documentation-system.md`

- [ ] **步骤 1：建立可审计的用户调研记录**

`user-research-and-requirements.md` 必须区分“原始证据”“研究者解释”“已确认需求”和“待验证假设”，禁止把推断伪装成用户原话。文档至少包含：研究目标、研究范围、来源清单、参与角色、时间、采集方式、原始表述摘录、痛点、期望结果、现有替代方案、约束、冲突观点、研究限制、洞察、需求条目、非目标、开放问题、确认记录和变更历史。现有 `Palantir的本体论讨论.docx`、主计划中的用户原话、产品定位中的三轮 Q&A 声明必须登记为来源；缺失的逐轮 Q&A 原文必须明确标记为“证据缺失/待补录”，不得补造内容。

每个需求使用稳定 ID `UR-{DOMAIN}-{NNN}`，并包含：`title`、`statement`、`source_refs`、`verbatim_quotes`、`actors`、`problem`、`desired_outcome`、`priority`、`constraints`、`acceptance_signals`、`confidence`、`status`、`confirmation_evidence_ref`、`confirmed_subject`、`confirmed_at`、`supersedes`。状态只允许 `discovered → validated → confirmed → superseded|rejected`；只有 `confirmed` 需求可进入生效规划。Stage A 的信任根固定为仓库内版本化证据文件与 GitHub PR 人工审批，不接受任意外部 URI 自报：证据文件记录媒体类型、采集时间、原始来源、访问级别和 SHA-256；确认主体必须是 `User` 类型的 GitHub 不可变 node ID，且为需求文件路径 CODEOWNER、权限为 maintain/admin，并通过绑定当前证据摘要和需求摘要的 `APPROVED` review 确认。敏感原文不能入库时，只入库脱敏摘录和由受治理归档系统导出的签名清单；Stage A 默认受信归档系统 allowlist 为空，因此此类需求保持 `validated`，不得进入规划，直到后续 ADR 明确 URI scheme、公钥、签名算法和身份映射。检查器重算仓库证据摘要，并拒绝缺证据、摘要不匹配、主体不可解析或确认发生在证据之前。

- [ ] **步骤 2：冻结规划、设计和执行条目模型**

规划要求使用 `PR-{DOMAIN}-{NNN}`，必须引用一个或多个 `UR-*`，包含范围、优先级、成功指标、目标阶段、约束、风险和 Product Owner 确认。设计要求使用 `DR-MOD{NN}-{NNN}`，必须引用 `PR-*`，并绑定 LLD 章节、ADR、OpenAPI/Schema/Cypher、场景和非功能指标。设计生产任务使用 `DTASK-{STAGE}-{NNN}`，必须引用已生效 `PR-*`，仅允许创建或修订 LLD、ADR、契约和设计门禁。实现任务使用 `TASK-{STAGE}-{NNN}`，必须同时记录 `design_refs`、`approved_lld_ref`、`approved_lld_version`、`approved_lld_digest` 和 `atomic_design_manifest_digest`；检查器必须验证这些值与发布视图中的有效批准及 LLD 原子设计清单一致。实现任务包含具体文件、前置条件、执行命令、预期结果、验证命令、证据路径、状态、阻塞原因和完成时间；未批准 LLD 不得产生实现任务。任务 0～10、12、14 分配 `DTASK-*`；任务 11、13 属于门禁工具实现与测试，必须引用已中期批准的 LLD-19 并分配 `TASK-*`；所有任务 0～14 均写入执行台账。

- [ ] **步骤 3：编写 LLD-19 并把 LLD 提升为设计唯一真实依据**

先编写 `lld-19-document-governance.md`，完整定义需求证据、UR/PR/DR、DTASK/TASK、追溯图、执行台账、检查器组件、CLI、退出码、安全边界、GitHub 集成、状态机和验收场景；任务 0 后续 Schema 与任务 11 检查器必须由 LLD-19 原子设计清单固定。随后在 `documentation-system.md` 中冻结完整 LLD 的 Definition of Ready。每份完整 LLD 在批准前必须明确且无待决项地包含：

1. 模块职责、非职责、上下游和唯一事实源；
2. 组件边界、目录与文件级目标结构；
3. 同步 API、异步事件、内部接口、鉴权上下文、版本和错误契约；
4. 领域实体、字段类型、约束、索引、关系、所有权和数据保留；
5. 状态机、事务边界、并发控制、幂等、重试、超时、补偿和回放；
6. 认证、授权、租户隔离、字段脱敏、密钥引用、审计和威胁边界；
7. 性能、容量、可用性、可观测性、降级、恢复和资源预算；
8. 配置项、环境变量、依赖版本、部署拓扑、迁移和回滚；
9. 3 核心/2 边界/2 异常场景，每个场景绑定 `UR-*`、`PR-*`、`DR-*`、测试 ID、计划路径和目标阶段；
10. 实施任务拆分、文件清单、命令、预期输出和完成定义；
11. 未解决问题列表必须为空，或被明确阻断且文档不得批准。

如果一个 LLD 无法在单文件中清晰审阅，必须按 `sections/01-boundaries.md`、`02-components.md`、`03-interfaces.md`、`04-data-model.md`、`05-state-and-reliability.md`、`06-security.md`、`07-non-functional.md`、`08-deployment-migration.md`、`09-scenarios.md`、`10-implementation-map.md` 拆分；每个规范性事实只能有一个定义位置，主文档通过相对链接和稳定 ID 建索引。

- [ ] **步骤 4：建立仓库内唯一执行台账**

`stage-a-execution-ledger.v1.yaml` 是 Stage A 活动状态唯一事实源，并由 `execution-ledger.v1.schema.json` 校验。顶层固定包含 `$schema`、`ledger_id`、`version`、`stage`、`updated_at`、`design_tasks`、`implementation_tasks`；条目 ID 全局唯一。每个 `DTASK-*` 记录 `status: planned|ready|in_progress|blocked|verified|accepted`、`planning_refs`、`allowed_artifact_kinds`、`files`、`commands`、`evidence` 和时间字段；每个 `TASK-*` 使用相同状态集并记录 `design_refs`。只有证据存在且摘要匹配时可进入 `verified`，只有独立批准证据有效时可进入 `accepted`。Markdown 复选框和交付报告只展示台账快照，不得作为活动状态源；仓库外 `topics.md` 仅作为历史输入，禁止继续承载当前进度。

- [ ] **步骤 5：建立全生命周期双向追溯**

`requirements-traceability.v1.yaml` 由 `requirements-traceability.v1.schema.json` 校验。顶层固定仅包含 `$schema`、`matrix_id`、`version`、`generated_at`、`nodes`、`edges`，不得复制 evidence、UR、PR 或 DR 正文；节点包含全局唯一 `id`、`kind: evidence|user_requirement|planning_requirement|design_requirement|lld|adr|contract|scenario|design_task|implementation_task|artifact|test|verification|release_gate`、`version`、`status`、`path`、`digest`；边包含全局唯一 `id`、允许的关系类型、`source`、`target`、`source_version`、`target_version`。追溯使用两条无歧义链：设计生产链为 `source evidence → UR → PR → DTASK → DR/LLD/ADR/contract → SCN → design approval`；实现交付链为 `approved LLD → TASK → code/config/migration → test → verification evidence → release gate`。Stage A 必须完成设计生产链；除 LLD-19 门禁工具实现所需 TASK 外，MOD-15～18 只登记未来 TASK ID、目标阶段和计划路径，不创建活动实现任务。检查器验证节点和边唯一、端点存在、版本/摘要匹配、允许的 kind/edge 组合、无非法环，并支持从任一需求向下查到当前阶段证据、从任一代码/测试向上查到已批准 LLD 和用户需求。`stage-a-traceability.v1.yaml` 由独立 Schema 校验且仅保存节点/边 ID 选择器和发布评审，不复制节点正文。`superseded/rejected` 需求不得有活动任务；孤立规划、孤立设计、活动实现无批准 LLD、无需求代码和无证据完成状态均为门禁错误。

- [ ] **步骤 6：验证需求事实源与三级文档治理**

测试必须覆盖：缺来源需求、把推断标成原话、确认材料缺失或摘要失配、确认主体不是合格 GitHub User/CODEOWNER、任意外部 URI 自报、伪造 `confirmed`、未确认需求进入规划、规划无 `UR-*`、设计生产任务无 `PR-*` 或产出实现文件、实现任务未绑定已批准 LLD/版本/正文摘要/原子清单摘要、LLD 存在未决项、LLD 原子设计清单摘要失配、ADR/契约/LLD 规范性事实重复或语义冲突、追溯矩阵节点/边重复、两条链 kind/edge 组合非法、断端点、非法环、未来 TASK 被误标活动、任务 0～14 任一未入台账、执行台账不符合 Schema、台账与计划复选框冲突、外部状态源仍被标为活动、PR 审批无法在 push 回溯或 merge commit 不匹配、任一方向追溯断链。预期所有负向样例被拒绝，既有已确认用户需求可完整追到 Stage A 设计生产任务，LLD-19 工具实现绑定中期批准基线，MOD-15～18 未来实现只能从最终批准 LLD 产生。

---

### 任务 1：冻结契约元规则和 Stage A 校验依赖

**文件：**
- 新建：`requirements-stage-a.txt`
- 新建：`contracts/schemas/common/identifier.v1.schema.json`
- 新建：`contracts/schemas/common/user-context.v1.schema.json`
- 新建：`contracts/schemas/common/error-response.v1.schema.json`
- 新建：`contracts/schemas/common/page.v1.schema.json`

- [ ] **步骤 1：固定 Python 校验依赖**

写入 `requirements-stage-a.txt`：

```text
jsonschema[format]==4.25.0
openapi-spec-validator==0.7.2
PyYAML==6.0.2
pytest==8.4.1
ruff==0.12.7
```

- [ ] **步骤 2：定义共享标识符**

`identifier.v1.schema.json` 使用 JSON Schema 2020-12，要求资源 ID 满足 `^[a-z][a-z0-9_]{2,63}$`，租户 ID 满足 `^tenant_[a-z0-9]{8,32}$`，版本满足 SemVer，时间使用 RFC 3339 `date-time`。

- [ ] **步骤 3：定义向后兼容的用户上下文与统一错误响应**

`user-context.v1.schema.json` 保留现有跨层字段 `user_id`、`tenant_id`、`data_scope`、`role`、`team_member_ids`，并新增可选 `attributes`、`trace_id`。`actor_user_id` 与 `roles` 不进入 v1 必填字段；如后续版本引入，必须分别与 `user_id`、`role` 建立版本化双向映射。Header/JWT 映射固定为：`X-User-Id` → `user_id`、`X-Tenant-Id` → `tenant_id`、`X-Data-Scope` → `data_scope`、`X-Team-Member-Ids` → `team_member_ids`，`role` 从已验证 JWT claim 读取。`team_member_ids` 在 `data_scope != team` 时默认为空数组；身份、租户、范围或角色缺失时拒绝请求，不得静默提权。契约测试必须覆盖既有五字段载荷兼容、Header/JWT 映射、缺字段拒绝和 team 范围成员列表校验。错误响应固定为：

```json
{
  "code": "ONTOLOGY_SCHEMA_CONFLICT",
  "message": "schema version conflicts with active version",
  "trace_id": "01J0TRACE1234567890",
  "details": [{"field": "version", "reason": "must be greater than 1.2.0"}]
}
```

- [ ] **步骤 4：安装依赖并验证四个共享 Schema 可编译**

运行：

```powershell
python -m pip install -r requirements-stage-a.txt
python -c "import json, pathlib; from jsonschema.validators import validator_for; [validator_for(json.loads(p.read_text(encoding='utf-8'))).check_schema(json.loads(p.read_text(encoding='utf-8'))) for p in pathlib.Path('contracts/schemas/common').glob('*.json')]; print('common schemas valid')"
```

预期输出：`common schemas valid`。

---

### 任务 2：编写九份跨模块 ADR 并进入独立评审

**文件：** 新建 `adr/0001-*.md` 至 `adr/0009-*.md`。

- [ ] **步骤 1：使用统一 ADR 结构**

每份 ADR 正文必须包含：`状态：Proposed`、上下文、决策、替代方案、后果、安全影响、迁移影响、回滚条件、关联契约和 `document_digest`；作者与实施者身份、审批主体、时间和证据统一记录在发布追溯视图，不在 ADR 正文复制。身份使用 GitHub 不可变用户 `node_id`，不得使用可改名 login 或自由文本。可信审查证据固定为 GitHub PR 的 `APPROVED` review，绑定 `repository_id`、`pull_request_number`、`head_sha`、review `node_id` 和当前文档摘要。合格独立审查者必须同时满足：主体类型为 `User`，是当前变更路径的 CODEOWNER，审批时仓库权限为 `maintain` 或 `admin`，且 node ID 不在 PR 作者、提交作者或 `implemented_by_subjects` 集合中；Bot、App、外部无维护权限账号一律无效。本地 ADR 状态只允许 `Proposed` 或 `In Review`，生效 `Accepted` 状态仅由发布追溯视图中的有效审查记录派生。本计划的可验证信任边界是“受组织 IAM 管理的不同 GitHub 主体”；实名、关联账号识别和组织入离职治理属于仓库外 IAM 控制，不宣称由 Stage A CI 证明。

- [ ] **步骤 2：冻结九项决策**

决策必须明确：PostgreSQL 管 Schema/版本/映射/策略元数据，Neo4j 管实例关系；Agent/Workflow 仅执行注册 Action；MCP 不替代源协议；Schema 使用 SemVer；跨存储用 Outbox 与重放；副作用动作必须幂等且可补偿；MOD-17 是策略/血缘/审计事实源；Simulation 不得写生产；MOD-13 禁止新增运行时能力并迁移到 MOD-16。

- [ ] **步骤 3：扫描未决占位符**

运行：

```powershell
$matches = Select-String -Path adr/*.md -Pattern 'TODO|TBD|待定|待补充' -CaseSensitive:$false
if ($matches) { $matches; exit 1 }
Write-Output 'ADR drafts complete and ready for independent review'
```

预期输出：`ADR drafts complete and ready for independent review`；此步骤不将 `Proposed` 视为占位符，也不替代独立审批。

---

### 任务 3：完成 LLD-15 业务本体服务与核心 Schema

**文件：**
- 新建：`lld-15-ontology-service.md`
- 新建：`contracts/openapi/ontology-service.v1.yaml`
- 新建：`contracts/schemas/ontology/object-type.v1.schema.json`
- 新建：`contracts/schemas/ontology/property-type.v1.schema.json`
- 新建：`contracts/schemas/ontology/relation-type.v1.schema.json`
- 新建：`contracts/schemas/ontology/action-type.v1.schema.json`
- 新建：`contracts/schemas/ontology/rule.v1.schema.json`
- 新建：`contracts/schemas/ontology/source-mapping.v1.schema.json`

- [ ] **步骤 1：按 Tier-2 模板写完整 LLD-15**

模块 ID 为 `MOD-15`，代码目标为 `services/ontology-service/`。定义 `draft → review → approved → active → deprecated` 状态机、PostgreSQL/Neo4j 边界、Outbox、乐观锁、租户隔离、分页和错误码。场景使用稳定前缀 `A15-*`，至少包含 3 核心、2 边界、2 异常场景。

- [ ] **步骤 2：定义核心模型约束**

`ActionType` 必须包含 `input_schema_ref`、`output_schema_ref`、`authorization_policy_ref`、`risk_level`、`side_effect` 和 `hitl_policy`。`action-type.v1.schema.json` 使用 `if/then/else` 或 `oneOf` 冻结条件约束：当 `side_effect: none` 时禁止出现 `idempotency_policy` 和 `compensation_action_ref`；当 `side_effect != none` 时必须提供 `idempotency_policy`，并在可补偿动作中提供 `compensation_action_ref`。不可补偿副作用必须显式声明 `recovery_policy: manual_recovery_required` 且 `hitl_policy.required: true`，禁止以字段缺失表示不可补偿。

- [ ] **步骤 3：定义 OpenAPI 3.1 接口**

至少提供 Schema 注册/评审/激活、对象类型查询、关系查询、Action 发现和版本查询；所有错误引用统一错误响应。

- [ ] **步骤 4：通过保留基准 URI的检查器验证 OpenAPI 与 Schema**

先实现或使用 `scripts/check_contracts.py` 的文件路径入口，以 OpenAPI 文件 URI 作为外部相对 `$ref` 的基准 URI，不得先将 YAML 脱离来源路径加载为普通字典后调用 `validate_spec`。运行：

```powershell
python scripts/check_contracts.py --openapi contracts/openapi/ontology-service.v1.yaml
```

预期输出包含：`ontology-service.v1.yaml: valid`。测试必须同时覆盖合法外部相对 `$ref` 可解析，以及断链 `$ref` 被拒绝。

---

### 任务 4：先冻结 LLD-17 的策略、血缘和审计契约

**文件：**
- 新建：`lld-17-ontology-governance.md`
- 新建：`contracts/openapi/ontology-governance.v1.yaml`
- 新建：`contracts/schemas/ontology/authorization-policy.v1.schema.json`
- 新建：`contracts/schemas/events/policy-decision.v1.schema.json`
- 新建：`contracts/schemas/events/audit-event.v1.schema.json`
- 新建：`contracts/schemas/events/lineage-event.v1.schema.json`

- [ ] **步骤 1：定义唯一策略事实源**

MOD-17 负责 RBAC/ABAC 模型、版本、审批、发布、对象/关系/Action/字段授权、脱敏、PolicyDecision、DataLineage 和不可变 AuditEvent；MOD-14 只保存治理配置和 PolicyBindingRef。

- [ ] **步骤 2：定义授权请求和决策**

授权输入包含用户、Agent、Workflow、租户、对象、动作、字段和环境；决策输出包含 `allow|deny|require_hitl`、策略版本、命中规则、脱敏指令和原因。

- [ ] **步骤 3：定义审计完整字段**

AuditEvent 必须追踪用户、Agent、Workflow、模型、规则、Schema 版本、策略版本、输入摘要、源系统结果、补偿结果和 trace ID，禁止其他模块作为审计事实存储。

- [ ] **步骤 4：验证治理契约与完整场景集**

运行与任务 3 相同的文件路径 OpenAPI 校验，并编译三个事件 Schema；LLD-17 场景使用稳定前缀 `A17-*`，至少包含 3 核心、2 边界、2 异常场景。预期契约与场景校验全部通过。

---

### 任务 5：完成 LLD-16 连接器运行时与执行契约

**文件：**
- 新建：`lld-16-source-connector-runtime.md`
- 新建：`contracts/openapi/source-connector-runtime.v1.yaml`
- 新建：`contracts/schemas/ontology/action-binding.v1.schema.json`
- 新建：`contracts/schemas/events/action-execution.v1.schema.json`

- [ ] **步骤 1：定义连接器模型**

定义 SourceSystem、ConnectorDefinition、ConnectorInstance、CredentialReference、SourceMapping、SyncCursor、ActionBinding、ActionExecution、IdempotencyRecord、CompensationAction 和 OutboxEvent。CredentialReference 只能保存密钥引用和版本，禁止 `password`、`secret_value`、`token_value` 字段。

- [ ] **步骤 2：冻结执行顺序**

固定为：接收身份上下文 → MOD-17 策略决策 → 高风险动作触发 HITL → 校验幂等键与摘要 → 解析 ActionBinding → MCP Gateway/适配器执行 → 写 ActionExecution/Outbox → 失败补偿或人工处理。

- [ ] **步骤 3：定义执行接口**

至少提供 ActionBinding 注册与查询、动作执行、执行状态查询、补偿、同步启动、游标查询和重放；`MOD-16` 不定义独立权限策略。

- [ ] **步骤 4：补充 MOD-13 兼容迁移规则与完整场景集**

旧连接器配置经显式转换写入 MOD-16；兼容壳层只转发，禁止新增业务能力；迁移完成后删除独立部署单元。LLD-16 场景使用稳定前缀 `A16-*`，至少包含 3 核心、2 边界、2 异常场景，覆盖幂等重放、补偿成功和不可补偿人工恢复。

---

### 任务 6：完成 LLD-18 隔离模拟契约

**文件：**
- 新建：`lld-18-ontology-simulation.md`
- 新建：`contracts/openapi/ontology-simulation.v1.yaml`
- 新建：`contracts/schemas/ontology/simulation-definition.v1.schema.json`

- [ ] **步骤 1：定义模拟模型**

定义 SimulationDefinition、SimulationRun、Workspace、SnapshotRef、ParameterOverride、ResultComparison、ResourceQuota 和 CleanupPolicy。

- [ ] **步骤 2：冻结生产隔离**

模拟工作区只能读取固定生产快照；所有写动作必须命中 simulation binding；检测到 production ActionBinding 时立即拒绝并产生审计事件。

- [ ] **步骤 3：定义生命周期与血缘**

状态为 `created → preparing → running → completed|failed|cancelled → cleaned`，结果关联输入 Schema、对象快照、规则版本、策略版本、模型版本和执行事件。

- [ ] **步骤 4：验证完整场景集及超时、取消、资源限制和清理**

LLD-18 场景使用稳定前缀 `A18-*`，至少包含 3 核心、2 边界、2 异常场景；场景集必须覆盖超配额拒绝、超时清理、取消清理和生产写保护异常。

---

### 任务 7：冻结 CRM、ERP、OA 三域 Schema 包与 Cypher 模型

**文件：**
- 新建：`contracts/domains/crm/ontology.v1.json`
- 新建：`contracts/domains/erp/ontology.v1.json`
- 新建：`contracts/domains/oa/ontology.v1.json`
- 新建：`contracts/cypher/platform-constraints.cypher`
- 新建：`contracts/cypher/crm-model.v1.cypher`
- 新建：`contracts/cypher/erp-model.v1.cypher`
- 新建：`contracts/cypher/oa-model.v1.cypher`

- [ ] **步骤 1：录入 CRM 基线**

必须精确包含 6 对象、5 关系、6 动作：Customer、Contact、Opportunity、Contract、SalesActivity、SalesRepresentative；HAS_CONTACT、HAS_OPPORTUNITY、GENERATES、OWNS、RELATES_TO；以及基线定义的六个 CRM 动作。

- [ ] **步骤 2：录入 ERP 基线**

必须精确包含 Supplier、Material、PurchaseRequest、PurchaseOrder、InventoryItem、GoodsReceipt；五个基线关系；六个基线动作。

- [ ] **步骤 3：录入 OA 基线**

必须精确包含 Employee、Department、LeaveRequest、ApprovalTask、Document、Meeting；五个基线关系；六个基线动作。

- [ ] **步骤 4：补齐动作治理属性**

所有动作包含输入/输出 Schema、策略引用、风险、副作用和 HITL。只读动作固定 `side_effect: none`，且禁止幂等和补偿字段；可补偿副作用动作必须包含幂等策略与补偿动作引用；不可补偿副作用动作必须声明 `manual_recovery_required` 并要求 HITL。

- [ ] **步骤 5：生成图约束并校验端点**

平台约束负责租户、类型、实例、版本唯一性；域 Cypher 只声明合法标签、关系和起止对象，不复制平台核心约束。

---

### 任务 8：修订 LLD-07、LLD-09 并重写 LLD-08

**文件：**
- 修改：`lld-07-knowledge-base.md`
- 修改：`lld-09-agent-runtime.md`
- 新建：`lld-08-workflow-orchestration.md`

- [ ] **步骤 1：收窄 LLD-07**

明确 MOD-07 只管理 Document、Chunk、Embedding 和 RAG；业务对象、关系、动作由 MOD-15 管理；Knowledge 只保存 KnowledgeBaseRef 与 ObjectTypeRef/RelationTypeRef；访问控制消费 MOD-17 决策。

- [ ] **步骤 2：修订 LLD-09 执行链**

删除 Capability 直绑 MCP Tool、Agent 直取连接器清单、Agent 保存 HMAC/凭据等设计；改为 CapabilityBinding → ActionTypeRef → MOD-17 → MOD-16/MCP Gateway。Agent 仅保存 ExecutionRef、CheckpointRef、PolicyBindingRef、SimulationRef 和 SchemaVersionRef。

- [ ] **步骤 3：从 LLD-05 重写 LLD-08**

保留有效的 Workflow、Checkpoint、HITL 和 Saga 设计；所有业务节点改为 OntologyActionRef；发布时校验 Action 版本；Hybrid 节点中 Agent 可用 Action 不得超出 Workflow 边界；运行事件发送 MOD-17。

- [ ] **步骤 4：补齐三份文档的场景数量**

每份至少 3 核心、2 边界、2 异常，并分配稳定 ID：`A07-*`、`A08-*`、`A09-*`。

---

### 任务 9：完成 LLD-10/12/14 影响分析和 LLD-13 迁移声明

**文件：** 新建 `lld-10-multi-agent-collaboration.md`、`lld-12-skill-library.md`、`lld-13-business-connectors.md`、`lld-14-governance-config.md`。

- [ ] **步骤 1：LLD-10 限定跨 Agent 数据**

Agent 间只交换 TaskRef、ObjectRef、ActionRef 和结构化结果，不复制本体事实，不绕过授权和执行强制点。

- [ ] **步骤 2：LLD-12 限定 Capability 绑定**

只管理 Capability 输入输出契约、ActionType 绑定、版本、PolicyBindingRef 和模板；禁止源系统地址、凭据、执行器和独立权限策略。

- [ ] **步骤 3：LLD-14 限定治理配置**

管理 Agent/Workflow 治理配置、HITL 条件、SLA 和策略引用；策略定义/版本/求值归 MOD-17，工单生命周期归 hitl-service，动作强制执行归 MOD-16/MCP Gateway。

- [ ] **步骤 4：LLD-13 固定迁移态**

状态为 Deprecated / Migration-only，明确兼容期、转换规则、转发规则、禁止新增项、移除条件和验收判据。

---

### 任务 10：建立治理职责矩阵和双向追踪矩阵

**文件：**
- 新建：`contracts/governance/responsibility-matrix.v1.yaml`
- 新建：`contracts/traceability/stage-a-traceability.v1.yaml`

- [ ] **步骤 1：写职责唯一性矩阵**

每项包含 `owner`、`control_plane`、`decision_point`、`enforcement_point`、`event_producer`、`system_of_record`。覆盖 RBAC/ABAC、Agent/Workflow 治理、HITL、字段脱敏、Action 执行、Simulation、Schema 版本、血缘和审计。

- [ ] **步骤 2：写全生命周期追踪矩阵**

`stage-a-traceability.v1.yaml` 作为 Stage A 发布视图，引用而不复制 `requirements-traceability.v1.yaml` 中的规范条目。每条 Stage A 范围要求必须沿设计生产链从 `UR-*` 经 `PR-*`、`DTASK-*` 映射到 `DR-*`、LLD 章节、契约文件、ADR、场景 ID、设计验证 ID 和 CI 检查 ID；LLD-19 还必须沿实现交付链映射到活动 `TASK-*`、测试与验证证据。MOD-15～18 仅登记未来实现任务 ID、计划测试 ID、计划路径和目标实现阶段，不得把未来任务标为活动或要求文件存在。任何 Stage A 已存在产物引用不存在时门禁失败；进入对应实现阶段后，门禁升级为活动 TASK、目标代码与测试文件必须存在且执行通过。

- [ ] **步骤 3：记录独立评审证据**

追踪矩阵的每个评审记录必须包含 `status`、`author_subject`、`implemented_by_subjects`、`reviewer_subject`、`reviewed_at`、`evidence`、`document_digest`。身份字段使用 GitHub `node_id`；`evidence` 必须绑定仓库 ID、PR 编号、当前 head SHA 和 review node ID。实施代理不得通过仅修改字段自行批准；本地无可信 PR 上下文时 `Accepted` 必须失败。CI 通过 GitHub API 重取 PR 作者、提交作者与 `APPROVED` review，验证 reviewer 为 `User`、匹配当前路径 CODEOWNERS、审批时具有 `maintain/admin` 权限，并规范化为不可变 node ID；审查者必须与作者/实施者集合无交集，review 必须对应当前 head SHA，摘要必须匹配当前内容。资格、身份或证据无法解析时失败；摘要、head SHA、CODEOWNERS 或权限变化后既有批准自动失效并回到 `In Review`。

- [ ] **步骤 4：人工复核唯一事实源**

确认策略决策点唯一为 MOD-17，Action 强制点唯一为 MOD-16/MCP Gateway，HITL 工单事实源唯一为 hitl-service，审计事实源唯一为 MOD-17。

---

### 任务 11：实现 Stage A 文档与契约校验程序

**文件：**
- 新建：`scripts/check_doc_consistency.py`
- 新建：`scripts/check_contracts.py`
- 新建：`scripts/check_stage_a_gate.py`
- 新建：`tests/stage_a/test_doc_consistency.py`
- 新建：`tests/stage_a/test_contracts.py`
- 新建：`tests/stage_a/test_stage_a_gate.py`

- [ ] **步骤 1：先写文档检查失败测试**

测试错误 MOD ID、完整 LLD 缺 Definition of Ready 项或 3/2/2 场景、影响分析缺职责/依赖/迁移约束、LLD-13 缺废弃与移除条件、断链，以及 Stage A 阶段错误要求服务目录存在。需求与层级测试覆盖 `UR → PR → DR → TASK` 缺边、未确认需求下沉、同一规范性事实多处定义、LLD 未解决问题非空却批准、执行任务缺文件/命令/预期结果/验证证据、执行台账状态与证据不一致。运行：

```powershell
python -m pytest tests/stage_a/test_doc_consistency.py -q
```

预期：因脚本尚不存在而失败。

- [ ] **步骤 2：实现文档检查**

按开头定义的三类文档集合执行差异化校验：完整 LLD（含 LLD-19）检查文件名/LLD/MOD 一致、完整 Definition of Ready、原子设计清单及组成物摘要、3/2/2 场景、Given/When/Then 和稳定场景 ID；影响分析仅检查影响范围、职责边界、依赖、迁移约束和目标实现阶段；LLD-13 仅检查迁移与废弃声明字段。所有类型检查相对链接。检查器同时验证需求事实源证据分类与确认摘要、`UR → PR → DR → DTASK/TASK` 状态和双向追溯、规范性事实单一定义位置、追溯/台账 Schema、执行证据和 LLD 无未决项批准规则。`--stage-a` 不要求 Stage B-D 服务目录或计划运行时测试文件存在，只要求场景登记测试 ID、计划路径和目标阶段；`--post-stage-a` 按目标阶段开启 LLD ↔ MODULE.md 双向检查和目标测试文件存在性检查。

- [ ] **步骤 3：先写契约检查失败测试**

覆盖合法外部相对 `$ref` 正向解析、断链 `$ref`、重复 `$id`、非法关系端点、缺授权引用、副作用动作缺幂等或补偿、高风险动作缺 HITL、只读动作错误携带幂等/补偿、不可补偿动作缺人工恢复策略、重复审计事实源、模拟绑定生产写端点、MOD-13 新增运行时职责，以及 UserContext 既有五字段兼容和缺失授权范围拒绝。评审门禁负向样例必须覆盖：作者/实施者与审查者 node ID 相同、身份字段缺失或不是 node ID、reviewer 为 Bot/App、reviewer 不是当前路径 CODEOWNER、审批时权限低于 maintain、`reviewed_at`/`evidence` 缺失、仓库或 PR 不匹配、review 非 `APPROVED`、review 不属于当前 head SHA、API 返回身份与文档声明不一致、`document_digest` 与重算摘要不一致、文档内容或 CODEOWNERS/权限变化后仍保留 `Accepted`，以及本地无可信 PR 上下文却声明 `Accepted`。

- [ ] **步骤 4：实现契约检查**

校验 OpenAPI、JSON Schema、三域完整性、Cypher 与域定义一致、治理职责唯一、追踪引用和评审证据。OpenAPI 校验必须从文件路径建立基准 URI，通过解析器/registry 解析共享 Schema 的外部相对 `$ref`；不得使用丢失来源 URI 的裸字典校验。评审校验必须重算 ADR/LLD 正文摘要；本地模式只允许 `Proposed/In Review`。CI 模式使用 GitHub API 按 repository ID、PR 编号、head SHA 与 review node ID 获取不可变身份和审批状态，验证 reviewer 类型为 `User`、属于当前变更路径 CODEOWNERS、审批时权限为 `maintain/admin`、`Accepted` 的摘要与当前内容一致、审查者不在 PR 作者和提交作者 node ID 集合中，并拒绝身份或资格不可解析、声明与 API 不一致、缺证据和摘要/head SHA/CODEOWNERS/权限过期；任何失败返回非零退出码并由聚合门禁传播。检查器不尝试识别现实中的关联账号，该风险由组织 IAM、强制 SSO 和 CODEOWNERS 成员治理承担，并在治理矩阵登记为外部控制。

- [ ] **步骤 5：实现聚合门禁**

`check_stage_a_gate.py` 顺序运行两类检查和 pytest，成功返回 0，内容冲突返回 1，缺失或不可解析输入返回 2。

- [ ] **步骤 6：运行全部测试、lint 和门禁**

```powershell
python -m pytest tests/stage_a -q
ruff check scripts tests/stage_a
python scripts/check_stage_a_gate.py
```

预期：pytest 全部通过、ruff 无错误、最终退出码 0。

---

### 任务 12：统一 Tier-1 文档规则并升级 GitHub Actions 为两阶段阻断门禁

**文件：** 修改 `.github/workflows/doc-check.yml`、`documentation-system.md`、`agent-ontology-master-plan.md`、`dual-ontology-three-layer-architecture-baseline.md`。

- [ ] **步骤 1：扩大触发范围**

`pull_request.paths` 与 `push.paths` 使用相同覆盖集合，但评审证据解析使用不同上下文：PR 事件直接验证当前 PR、head SHA 与 review；push 事件从发布视图中的 `repository_id`、`pull_request_number`、`review_node_id`、`reviewed_head_sha` 和 `merge_commit_sha` 回溯 GitHub API，确认该 PR 已合并且 merge commit 为当前提交祖先，并重新验证当时 CODEOWNERS、权限、审批和文档摘要。无法回溯时失败，不降级为自报。覆盖路径至少显式包含：`user-research-and-requirements.md`、`dual-ontology-three-layer-architecture-baseline.md`、`agent-ontology-master-plan.md`、`documentation-system.md`、`product-positioning-and-business-model.md`、`stage-a-dual-ontology-contract-freeze-implementation-plan.md`、`lld-*.md`、`lld-*/sections/**`、`adr/**`、`contracts/**`、`scripts/check_*.py`、`tests/stage_a/**`、`requirements-stage-a.txt` 和 `.github/workflows/doc-check.yml`。

- [ ] **步骤 2：用 Python 门禁替换 shell grep**

工作流使用 Python 3.12，安装 `requirements-stage-a.txt`，授予验证 PR、review、CODEOWNERS、协作者权限和合并关系所需的最小只读 GitHub 权限；若默认 `GITHUB_TOKEN` 无法读取组织团队或协作者权限，必须使用只读 GitHub App 凭据引用，禁止在仓库保存令牌。PR 事件向检查器传递 repository ID、PR 编号和 head SHA；push 事件传递 repository ID 和当前 commit SHA，并由检查器从发布视图读取已存 PR/review 标识后回溯 API。任一数据源不可访问时门禁失败，不得降级为信任文档自报。依次执行：

```yaml
- run: python scripts/check_doc_consistency.py --stage-a
- run: python scripts/check_contracts.py
- run: python -m pytest tests/stage_a -q
- run: ruff check scripts tests/stage_a
- run: python scripts/check_stage_a_gate.py
```

- [ ] **步骤 3：同步 OpenAPI 3.1、文档分类与分阶段测试规则**

将 `documentation-system.md` 的活动 OpenAPI 标准、模板和工具链统一升级为 3.1；增加完整 LLD、Stage A 影响分析、迁移与废弃声明三类文档规则，明确只有完整 LLD 执行完整模板与 3/2/2 场景校验。将验收标准改为：Stage A 只要求文档、契约、追踪和门禁测试通过，并登记未来运行时测试 ID/路径/目标阶段；进入目标实现阶段后才要求对应 E2E 文件存在且执行通过。将 `doc-check.yml` 标记为“已有，Stage A 重构”。

- [ ] **步骤 4：消除总体规划的阶段与顺序歧义**

修订 `agent-ontology-master-plan.md`：LLD-10/12/14 明确为“Stage A 影响分析，Stage D 完整设计与实现”；Stage A 契约冻结顺序统一为 `LLD-15 → LLD-17 → LLD-16 → LLD-18`，原因是先冻结策略决策契约再冻结执行强制点契约。

- [ ] **步骤 5：澄清 Action 条件治理基线**

修订 `dual-ontology-three-layer-architecture-baseline.md`：所有 Action 必须有输入/输出 Schema、权限、风险和副作用定义；只读 Action 显式声明 `side_effect: none` 且不携带幂等/补偿字段；副作用 Action 必须有幂等定义，并按可补偿性提供补偿动作或 `manual_recovery_required` 与 HITL。

- [ ] **步骤 6：本地执行与 CI 相同的命令**

预期所有命令成功，且历史 LLD-00～06 不因新模板要求误报。

---

### 任务 13：执行负向门禁验证

**文件：** 仅修改 `tests/stage_a/` 中的测试夹具，不污染正式契约。

- [ ] **步骤 1：验证安全边界**

构造 Agent 契约含源系统 URL、CredentialReference 含明文密钥、MOD-18 指向 production 写绑定，预期检查器拒绝。

- [ ] **步骤 2：验证动作治理**

构造缺输入/输出 Schema、缺授权、高风险缺 HITL、副作用缺幂等或补偿，预期检查器拒绝。

- [ ] **步骤 3：验证图与职责边界**

构造关系端点不存在、MOD-14/MOD-17 同为策略事实源、MOD-13 新增执行职责，预期检查器拒绝。

- [ ] **步骤 4：验证正向样例不被误杀**

完整三域契约、合法外部相对 `$ref`、既有五字段 UserContext、只读无副作用动作、合法 Simulation binding 和唯一治理职责全部通过。

---

### 任务 14：Stage A 独立封版评审与最终检查

**文件：** 独立审查者不直接修改规范性 ADR/LLD 内容；审批事实只写入 `contracts/traceability/stage-a-traceability.v1.yaml`。ADR/LLD 正文状态保持 `Proposed` 或 `In Review`，其生效状态由发布视图中绑定当前摘要的有效评审记录派生；实施代理不得自行写入批准记录，不新增运行时代码。

- [ ] **步骤 1：按依赖顺序评审**

顺序固定为：Tier-1 → ADR → LLD-15 → LLD-17 → LLD-16 → LLD-18 → 核心契约 → 三域契约 → LLD-07/09/08 → LLD-10/12/14/13 → 治理矩阵 → CI。

- [ ] **步骤 2：扫描禁用占位符与未定义引用**

由 `check_doc_consistency.py` 解析 Markdown，仅在普通文本中扫描 `TODO`、`TBD`、`待定`、`待补充`，忽略 fenced code block，且不将合法 Python `...` 表达式视为占位符。运行：

```powershell
python scripts/check_doc_consistency.py --stage-a --check-placeholders
```

预期输出包含：`No unresolved placeholders found`。

- [ ] **步骤 3：执行最终门禁**

```powershell
python scripts/check_doc_consistency.py --stage-a
python scripts/check_contracts.py
python -m pytest tests/stage_a -q
ruff check scripts tests/stage_a
python scripts/check_stage_a_gate.py
```

预期全部成功，最终退出码 0。

- [ ] **步骤 4：核对 Stage A 通过条件**

必须同时满足：用户调研来源、原话、解释、假设和已确认需求可区分，确认材料及摘要有效，证据缺口未被伪造补全；所有活动 `PR-*` 可追到已确认 `UR-*`，所有 `DR-*` 可追到规划，所有 `DTASK-*` 可追到生效规划，所有未来 `TASK-*` 只能追到已批准设计；LLD-15～18 与 LLD-07/08/09 达到 Definition of Ready，其原子设计清单中的 ADR/契约摘要一致，并在发布视图中获得带审查者、时间、证据和当前文档摘要的有效独立评审；LLD-10/12/14 影响分析完整且未被升级为 Stage A 完整设计；LLD-13 迁移边界明确；三域对象/关系/动作与基线一致；OpenAPI 3.1/JSON Schema/Cypher/治理矩阵零冲突；每份 ADR 在发布视图中具有绑定当前摘要的有效 Accepted 评审记录，正文无需自改为 Accepted；每个场景已登记 `UR/PR/DR`、测试 ID、计划路径和目标实现阶段；追溯矩阵与执行台账通过各自 Schema，Stage A 台账与证据一致且不存在仓库外活动状态源；所有 Stage A 正向测试通过且负向样例被拦截；未创建 MOD-15～18 运行时代码。

---

## 三、自检结果

- 基线覆盖：已覆盖 Stage A 的 Tier-1、LLD-15～18、LLD-07/08/09、LLD-10/12/14、LLD-13 迁移、三域 Schema、ADR、治理矩阵、验收场景和自动门禁。
- 边界覆盖：已明确 MOD-15/16/17/18、MOD-12/13/14、MCP Gateway、hitl-service 的唯一职责。
- 测试覆盖：每个检查器均包含先失败后实现的正反例，最终命令包含 pytest、ruff、文档检查、契约检查和聚合门禁。
- 范围控制：未安排 Stage B-D 运行时代码、真实连接器或基础设施部署。
- 占位符检查：计划中没有要求执行者自行补全的 TODO/TBD；所有路径、命令、门禁和预期结果均已指定。
