# 三级文档体系规范（地基文档 V1.2）

> **文档性质**：规划层（Tier-1）规范文档  
> **生效日期**：2026-08-03  
> **决策来源**：[Stage A 双本体契约冻结实施计划：任务 0](./stage-a-dual-ontology-contract-freeze-implementation-plan.md)  
> **关联文档**：[双本体三层核心架构与项目阶段新基线](./dual-ontology-three-layer-architecture-baseline.md)、[总体规划](./agent-ontology-master-plan.md)、[产品定位与商业模式](./product-positioning-and-business-model.md)、[LLD-19 文档治理与追溯门禁](./lld-19-document-governance.md)

---

## 1. 体系总览

### 1.1 三层结构

```
┌─────────────────────────────────────────┐
│  Tier-1 规划层（What & Why）              │
│  agent-ontology-master-plan.md          │
│  product-positioning-and-business-model.md│
│  documentation-system.md                │
│  → 回答：做什么、为什么做、给谁用         │
├─────────────────────────────────────────┤
│  Tier-2 设计层（How）                    │
│  lld-07-knowledge-base.md               │
│  lld-08-workflow-orchestration.md       │
│  lld-09-agent-runtime.md                │
│  lld-10-multi-agent-collaboration.md    │
│  lld-11-model-config.md                 │
│  lld-12-skill-library.md                │
│  lld-13-business-connectors.md          │
│  lld-14-governance-config.md            │
│  lld-15-ontology-service.md             │
│  lld-16-source-connector-runtime.md     │
│  lld-17-ontology-governance.md          │
│  lld-18-ontology-simulation.md          │
│  lld-19-document-governance.md          │
│  → 回答：怎么做、接口契约、数据模型        │
├─────────────────────────────────────────┤
│  Tier-3 执行层（Do）                     │
│  services/knowledge-base/               │
│  services/agent-runtime/                │
│  services/workflow-orchestrator/        │
│  services/ontology-service/             │
│  services/source-connector-runtime/     │
│  services/ontology-governance/          │
│  services/ontology-simulation/          │
│  frontend-pc/src/views/...              │
│  e2e-tests/...                          │
│  → 回答：代码实现、测试用例、部署脚本      │
└─────────────────────────────────────────┘
```

### 1.2 文档编号规则

| 层级 | 编号规则 | 示例 | 说明 |
|------|----------|------|------|
| Tier-1 规划层 | `{domain}-{topic}.md` | `product-positioning-and-business-model.md` | 无编号，按领域命名 |
| Tier-2 设计层 | `lld-{NN}-{module}.md` | `lld-07-knowledge-base.md` | NN 从 07 开始递增（01~06 已存在） |
| Tier-3 执行层 | 代码目录/文件 | `services/knowledge-base/` | 运行时模块目录与 LLD 模块名对应；仓库工具使用 LLD 声明的脚本路径 |

### 1.3 模块 ID 锚点

每个 Tier-2 LLD 必须定义 **模块 ID（Module ID）**，格式：`MOD-{NN}`

| LLD 编号 | 模块 ID | 模块名称 | 对应代码目录 | 新基线职责 |
|----------|---------|----------|--------------|------------|
| LLD-07 | MOD-07 | knowledge-base | `services/knowledge-base/` | 非结构化知识、文档、Embedding 与 RAG |
| LLD-08 | MOD-08 | workflow-orchestration | `services/workflow-orchestrator/` | 基于 Ontology Action 的 Workflow 与 Hybrid 编排 |
| LLD-09 | MOD-09 | agent-runtime | `services/agent-runtime/` | 消费业务本体并通过注册 Action 执行 |
| LLD-10 | MOD-10 | multi-agent-collaboration | `services/agent-runtime/collaboration/` | 基于统一业务本体交换任务和结果 |
| LLD-11 | MOD-11 | model-config | `services/model-config/` | LLM 与 Embedding 模型配置、路由和降级 |
| LLD-12 | MOD-12 | skill-library | `services/skill-library/` | Agent Capability 与 Ontology Action 的绑定和复用 |
| LLD-13 | MOD-13 | business-connectors | `services/connectors/` | Deprecated / Migration-only，仅保留兼容迁移，运行时能力并入 MOD-16 |
| LLD-14 | MOD-14 | governance-config | `services/governance/` | Agent 与 Workflow 治理，与 MOD-17 协同 |
| LLD-15 | MOD-15 | ontology-service | `services/ontology-service/` | 业务本体 Schema、对象、关系、动作与规则注册查询 |
| LLD-16 | MOD-16 | source-connector-runtime | `services/source-connector-runtime/` | 源连接、映射、同步、写回、幂等与补偿；承接 MOD-13 演进 |
| LLD-17 | MOD-17 | ontology-governance | `services/ontology-governance/` | 本体 RBAC/ABAC、版本审批、血缘与审计 |
| LLD-18 | MOD-18 | ontology-simulation | `services/ontology-simulation/` | 隔离情景、参数覆盖、推演与结果对比 |
| LLD-19 | MOD-19 | document-governance | `scripts/check_*.py` | 需求证据、设计治理、追溯、执行台账与 Stage A 门禁；仓库工具，不创建业务服务目录 |

MOD-13 状态为 **Deprecated / Migration-only**，不再作为独立运行时继续扩展；`services/connectors/` 仅允许保留兼容壳层或迁移适配，存量能力迁移至 MOD-16 后删除独立部署单元。MOD-14 保留 Agent 与 Workflow 的治理配置和策略引用，MOD-17 负责业务本体权限策略、版本、血缘、审计和统一策略求值。两组模块不得重复实现连接执行、权限策略引擎或审计事实存储。

> **适用范围**：`MODULE.md` 规则仅适用于存在业务代码目录的运行时模块。MOD-19 的代码锚点、原子设计清单与批准摘要绑定由 [LLD-19](./lld-19-document-governance.md) 定义，不创建 `services/document-governance/`。

> **强制规则**：代码目录创建时，必须在根目录放置 `MODULE.md`，内容仅一行：`Module: MOD-{NN}`，用于 CI 校验。

---

## 2. 文档模板规范

### 2.1 Tier-1 规划层模板

```markdown
# {文档标题}（规划层 V{X.Y}）

> **文档性质**：规划层（Tier-1）  
> **生效日期**：{YYYY-MM-DD}  
> **决策来源**：{Q&A 确认/会议纪要/变更请求}  
> **关联文档**：{相关文档链接}

## 1. 背景与目标
## 2. 范围与边界
## 3. 关键决策
## 4. 风险与约束
## 5. 下一步行动

> **文档变更记录**
> - V{X.Y}（{YYYY-MM-DD}）：{变更描述}
```

### 2.2 Tier-2 设计层的三类文档

Stage A 的 Tier-2 产物分为三类，门禁不得把影响分析或迁移声明按完整 LLD 要求补造尚未进入设计阶段的实现细节。

| 类型 | Stage A 适用对象 | 必备内容 | 准入结果 |
|------|------------------|----------|----------|
| 完整 LLD | LLD-07/08/09/15/16/17/18/19 | 本节完整 DoR、稳定场景 ID、3 核心/2 边界/2 异常场景、原子设计清单 | 可进入独立设计评审；有效批准后才能授权实现任务 |
| 影响分析 | LLD-10/12/14 | 影响范围、职责与非职责、依赖、接口/数据/安全影响、迁移约束、目标实现阶段 | 仅证明 Stage A 已完成影响识别，不授权创建运行时实现任务 |
| 迁移声明 | LLD-13 | Deprecated / Migration-only 状态、能力承接方、兼容边界、禁止新增项、迁移与移除条件 | 仅约束兼容迁移，不授权扩展独立运行时 |

### 2.3 完整 LLD Definition of Ready

完整 LLD 在批准前必须明确且无待决项地覆盖以下十一项；每项规范性事实只能有一个定义位置，ADR、OpenAPI、Schema、Cypher 和规范性 section 均由 LLD 通过相对路径、版本和摘要纳入原子设计清单。

1. 模块职责、非职责、上下游和唯一事实源；
2. 组件边界、目录与文件级目标结构；
3. 同步 API、异步事件、内部接口、鉴权上下文、版本和错误契约；
4. 领域实体、字段类型、约束、索引、关系、所有权和数据保留；
5. 状态机、事务边界、并发控制、幂等、重试、超时、补偿和回放；
6. 认证、授权、租户隔离、字段脱敏、密钥引用、审计和威胁边界；
7. 性能、容量、可用性、可观测性、降级、恢复和资源预算；
8. 配置项、环境变量、依赖版本、部署拓扑、迁移和回滚；
9. 3 个核心、2 个边界、2 个异常场景，每个场景绑定 `UR-*`、`PR-*`、`DR-*`、测试 ID、计划路径和目标阶段；
10. 实施任务拆分、文件清单、命令、预期输出和完成定义；
11. 未解决问题列表为空；存在明确阻断项时，文档不得批准。

单文件无法清晰审阅时，使用 `sections/01-boundaries.md` 至 `sections/10-implementation-map.md` 的拆分结构。拆分规则、审批摘要和原子清单计算以 [LLD-19](./lld-19-document-governance.md) 为唯一设计依据。

### 2.4 完整 LLD 索引模板

```markdown
# LLD-{NN} {模块名称}低层设计

> **模块 ID**：MOD-{NN}  
> **文档性质**：设计层（Tier-2）  
> **状态**：Draft | Proposed | In Review | Accepted  
> **上游文档**：[总体规划](./agent-ontology-master-plan.md)  
> **对应代码**：`services/{module-name}/` 或仓库工具路径  
> **目标阶段**：Stage {A-F}  
> **版本**：{SemVer}

## 1. 设计定位
## 2. Definition of Ready 结论
## 3. 规范性 section 索引
## 4. 决策摘要
## 5. 审批与变更状态
## 6. 未解决问题
## 7. 变更记录
```

完整 LLD 可以内联十项设计内容，也可以采用上述索引加规范性 section。索引不得复制 section 中的规范事实。

### 2.5 影响分析模板

```markdown
# LLD-{NN} {模块名称}影响分析

> **模块 ID**：MOD-{NN}  
> **文档类型**：影响分析  
> **目标完整设计阶段**：Stage {X}

## 1. 影响范围
## 2. 职责与非职责
## 3. 上下游依赖
## 4. 接口、数据与安全影响
## 5. 迁移约束
## 6. 目标实现阶段
## 7. 未解决问题与阻断项
```

### 2.6 迁移声明模板

```markdown
# LLD-13 Business Connectors 迁移声明

> **模块 ID**：MOD-13  
> **文档类型**：迁移声明  
> **状态**：Deprecated / Migration-only

## 1. 存量职责
## 2. 能力承接方
## 3. 兼容边界
## 4. 禁止新增项
## 5. 迁移步骤与验证
## 6. 移除条件与回滚
```

### 2.7 Tier-3 执行层规范

- **代码路径**：运行时模块目录必须与 LLD 模块名对应并包含 `MODULE.md`（内容：`Module: MOD-{NN}`）；仓库工具使用 LLD 声明的脚本和测试路径
- **README**：每个服务/模块必须有 `README.md`，引用对应 LLD 文档链接
- **测试文件**：单元测试命名 `test_{module}_{feature}.py`，E2E 测试命名 `e2e-{NN}-{scenario}.spec.ts`

---

## 3. 文档-代码联动机制

### 3.1 治理 ID 与双向锚点

| 对象 | ID 格式 | 用途 |
|------|---------|------|
| 研究证据 | `EVD-{DOMAIN}-{NNN}` | 绑定来源、访问级别和内容摘要 |
| 用户需求 | `UR-{DOMAIN}-{NNN}` | 记录经证据支持的用户需求 |
| 规划要求 | `PR-{DOMAIN}-{NNN}` | 把已确认 UR 转换为范围、指标和阶段目标 |
| 设计要求 | `DR-MOD{NN}-{NNN}` | 绑定 PR、LLD section、ADR、契约、场景和 NFR |
| 设计生产任务 | `DTASK-{STAGE}-{NNN}` | 仅生产或修订 LLD、ADR、契约和设计门禁 |
| 实现任务 | `TASK-{STAGE}-{NNN}` | 仅由有效批准 LLD 授权代码、配置、迁移和测试实现 |
| 场景 | `SCN-{MODULE}-{TYPE}-{NNN}` | 绑定需求、设计要求、测试 ID、路径和目标阶段 |
| 规范性 section / 原子 | `SEC-{NN}-{NN}` / `ATOM-{NN}-{NNN}` | 固定大型 LLD 的唯一规范位置和摘要清单 |
| 模块 | `MOD-{NN}` | 连接 LLD 与对应代码或仓库工具 |

ID 的字段、状态、关系和校验不变量由 [LLD-19 数据模型与追溯图](./lld-19/sections/04-data-model.md) 唯一定义。本文只规定各层必须使用稳定 ID，不复制治理 Schema 的字段级事实。

运行时模块通过 LLD 头部 `MOD-*` 与代码目录中的 `MODULE.md` 形成双向锚点。MOD-19 等仓库工具通过 LLD 原子设计清单、批准版本与摘要绑定到目标脚本和测试，不适用业务服务目录规则。

### 3.2 规范性事实唯一性

- Tier-1 只定义项目目标、范围、文档分类和准入规则；
- 完整 LLD 是模块设计的唯一实施入口，ADR 与机器可读契约是其引用并固定摘要的规范组成物；
- 同一规范性事实只能由一个 LLD section、ADR 或契约定义，其他文档使用稳定 ID 和相对链接引用；
- 实施者必须从已批准 LLD 入口读取完整原子设计清单，不得绕过 LLD 单独选择 ADR 或契约；
- 摘要、审批和冲突判定规则以 [LLD-19](./lld-19-document-governance.md) 为准。

### 3.3 CI 校验机制

Stage A 的检查器组件、CLI、退出码、GitHub 审批验证、安全边界和聚合顺序由 [LLD-19](./lld-19-document-governance.md) 唯一定义。CI 至少覆盖：

1. 三类 Tier-2 文档按各自模板校验，完整 LLD 额外校验十一项 DoR、3/2/2 场景、相对链接、未决项和原子设计清单；
2. OpenAPI 3.1、JSON Schema、Cypher、ADR 与 LLD 引用可解析，版本和摘要一致；
3. `MOD-*`、`MODULE.md`、LLD 与代码路径在对应阶段满足双向一致性；
4. 治理 ID 全局唯一，追溯关系合法且双向可达；
5. 台账状态与证据一致，活动实现任务绑定有效批准 LLD、版本、正文摘要和原子清单摘要；
6. 同一规范性事实不存在重复定义或语义冲突。

### 3.4 变更同步规则

| 变更类型 | 同步要求 | 审核要求 |
|----------|----------|----------|
| 新增模块 | 先写 LLD，再建代码目录 | LLD 评审通过后方可编码 |
| 接口变更 | 先改 LLD 契约，再改代码 | 契约变更需标注 Breaking/Non-breaking |
| 数据模型变更 | 先改 LLD Schema，再迁移数据库 | 必须提供迁移脚本 |
| 纯代码优化 | 不改 LLD，代码注释更新即可 | 无需 LLD 评审 |
| 验收标准变更 | 先改完整 LLD 的场景规范位置，再改测试用例 | 验收变更需 Product Owner 确认 |

---

## 4. 场景化验收标准

### 4.1 格式规范

每个完整 LLD 的场景规范位置必须包含稳定 `SCN-*` ID 和 **Given/When/Then** 格式的验收场景：

```gherkin
### 场景 1：用户上传 PDF 文档到知识库

**Given** 用户已登录系统，且拥有知识库写入权限  
**And** 知识库服务已启动，Qdrant 向量库连接正常  

**When** 用户通过 PC 前端上传一份 10MB 的 PDF 文档  
**And** 文档解析服务提取文本内容  
**And** 向量化服务将文本分块并生成 embedding  
**And** 向量数据写入 Qdrant  

**Then** 系统返回文档 ID 和上传成功状态  
**And** 用户可在知识库列表中看到该文档  
**And** 用户可通过关键词检索到该文档内容  
**And** 检索结果包含文档原文片段和相似度得分  

**验收测试**：`e2e-tests/e2e-07-upload-document.spec.ts`
```

### 4.2 场景分类

以下数量仅适用于完整 LLD；影响分析与迁移声明按第 2.2 节各自必备内容校验。

| 场景类型 | 数量要求 | 覆盖要求 |
|----------|----------|----------|
| 核心场景（Happy Path） | 每个完整 LLD ≥ 3 个 | 覆盖主要用户旅程 |
| 边界场景 | 每个完整 LLD ≥ 2 个 | 覆盖输入边界、并发边界、资源边界 |
| 异常场景 | 每个完整 LLD ≥ 2 个 | 覆盖网络失败、服务宕机、数据异常 |

### 4.3 验收通过标准

- **测试通过**：所有 E2E 测试用例通过（Playwright/pytest）
- **契约通过**：API 契约测试通过（OpenAPI 校验/Protobuf 校验）
- **性能通过**：核心接口满足完整 LLD 非功能设计中定义的性能指标
- **文档同步**：代码变更与 LLD 变更同步提交，CI 校验通过

---

## 5. 阶段与里程碑（1 人 + AI 模式）

### 5.1 Stage A-F 架构优先路线

原 W1-W8 的 MOD-07/MOD-09 直接编码排期已失效。后续执行以 [双本体三层核心架构与项目阶段新基线](./dual-ontology-three-layer-architecture-baseline.md) 为准，并按以下门禁顺序推进：

| 阶段 | 核心交付 | 阶段门禁 |
|------|----------|----------|
| Stage A：架构与契约冻结 | 冻结需求事实源与文档治理；完成 LLD-19、LLD-15~18 与 LLD-07/08/09 修订；完成 LLD-10/12/14 影响分析和 LLD-13 迁移声明；冻结三域 Schema、ADR、治理职责矩阵与验收场景 | 任务 0 治理产物可解析；完整 LLD 达到 DoR 并按批准状态受控；影响分析与迁移声明通过分类校验；OpenAPI 3.1、JSON Schema、Cypher 与治理职责矩阵无冲突；追溯与台账一致 |
| Stage B：语义层平台 | MOD-15；MOD-17 策略存储、版本治理、血缘和审计底座；Neo4j/PostgreSQL 存储、Schema 版本、对象关系查询、三域 Schema 导入 | CRM、ERP、OA 对象和关系可创建、查询、版本化并隔离，治理元数据可追溯 |
| Stage C：动力层平台 | MOD-16、Action Registry、MOD-17 动作授权接入、MCP Tool 发现、HITL、幂等、补偿、模拟适配器 | 每个业务域至少完成一条完整读写、授权拒绝、高风险审批和副作用动作补偿链路 |
| Stage D：动态层平台 | MOD-08/09、MOD-18 隔离模拟、规则引擎、Hybrid、MOD-07 RAG、Checkpoint、三层记忆 | Agent 和 Workflow 只能通过注册 Ontology Action 操作业务对象，越权调用被拒绝；模拟场景可隔离执行 |
| Stage E：三域完整验证 | CRM/ERP/OA 端到端业务流、跨域编排、模拟、审计、Docker Compose 与 E2E | 所有动作验证成功、失败、授权与审计；高风险动作验证审批、拒绝和超时；副作用写动作验证幂等与补偿；只读动作验证无副作用 |
| Stage F：真实数据源接入 | 在 API、沙箱、凭据管理和字段映射齐备后逐域替换模拟适配器 | 上层契约不变；凭据隔离、映射、同步重放、限流超时、写回补偿、审计和回滚演练通过 |

阶段之间禁止跳过门禁。Stage F 不属于当前无真实系统凭据条件下的承诺范围。

### 5.2 文档治理的分阶段门禁

文档治理按产物成熟度启用，禁止在 Stage A 提前要求 Stage B-D 代码目录存在，也禁止以未来任务记录替代活动实现任务。

| 门禁阶段 | 校验范围 | 允许结果 | 阻断条件 |
|----------|----------|----------|----------|
| A0：结构建立 | Tier-1、三类 Tier-2 文档、治理 Schema、相对链接、ID 格式 | `Draft` 或 `Proposed`；未来 TASK 仅登记 ID、目标阶段和计划路径 | 文件不可解析、类型误判、ID 重复、绝对链接或越界路径 |
| A1：设计评审 | 完整 LLD DoR、3/2/2 场景、规范事实唯一性、原子清单、OpenAPI 3.1/Schema/Cypher/ADR 一致性 | `Proposed` 或 `In Review` | DoR 缺项、未决项非空却申请批准、重复规范事实、契约冲突 |
| A2：实现授权 | GitHub 独立批准、LLD 版本/正文摘要/原子清单摘要、`TASK-*` 绑定 | 仅有效批准 LLD 可授权活动实现任务 | 本地自报批准、审查者不合格、摘要失配、活动 TASK 无批准 LLD |
| A3：验证与封版 | 测试、验证证据、执行台账、双向追溯、发布视图 | 证据匹配后进入 `verified`，独立验收后进入 `accepted` | 断链、证据缺失、台账冲突、发布视图复制规范正文或引用失效 |
| Post-Stage-A：渐进存在性 | 已进入目标阶段的服务目录、`MODULE.md`、代码、配置、迁移和测试 | 仅对已批准且已激活的模块要求产物存在 | 提前要求未来产物，或已激活任务缺少计划产物 |

PR 与 push 必须调用同一组已验证检查器；GitHub 上下文中的审批回溯、失败关闭和回滚规则引用 [LLD-19 CI、部署、迁移与回滚](./lld-19/sections/08-deployment-migration.md)。

### 5.3 阶段检视机制

- 每个阶段结束时执行一次架构、契约、测试和文档一致性检视；
- 任一门禁未通过时，不得启动下一阶段编码；
- LLD 完成后必须按模板清单自审并通过评审；
- 进度偏差超过 20% 时优先缩减非核心体验，不得删除安全、治理、幂等、补偿、审计或隔离能力。

### 5.4 执行台账与全生命周期追溯

- Stage A 活动状态唯一事实源为 [执行台账](./contracts/execution/stage-a-execution-ledger.v1.yaml)；Markdown 复选框、交付报告和仓库外任务列表仅可展示快照。
- 全量关系唯一事实源为 [需求追溯矩阵](./contracts/traceability/requirements-traceability.v1.yaml)；Stage A 发布视图只保存节点/边 ID 选择器和发布评审，不复制规范正文。
- 设计生产链为 `evidence -> UR -> PR -> DTASK -> DR/LLD/ADR/contract -> SCN -> design approval`。
- 实现交付链为 `approved LLD -> TASK -> code/config/migration -> test -> verification evidence -> release gate`。
- 任一需求必须可向下追到当前阶段证据，任一代码或测试必须可向上追到已批准 LLD 和用户需求；断链、非法环、孤立节点、无证据完成状态或台账冲突均阻断门禁。

台账字段、状态转换、边类型、摘要和批准资格不在本文重复定义，统一引用 [LLD-19 边界与治理模型](./lld-19/sections/01-boundaries.md)、[数据模型与追溯图](./lld-19/sections/04-data-model.md)及对应机器可读 Schema。

### 5.5 范围缩减优先级

当进度超出预期时，按以下顺序缩减范围：

1. **先延后**：本体管理后台的高级可视化、批量编辑和体验增强；
2. **再延后**：MOD-10 多 Agent 并行协作，保留单 Agent 与确定性 Workflow；
3. **再延后**：MOD-12 技能市场和管理界面，保留 Capability → Ontology Action 的静态注册契约；
4. **再收敛**：Stage E 的跨域组合场景数量，但每个 CRM、ERP、OA 域必须保留至少一条完整读写链路；
5. **不得裁剪**：Stage A 契约、MOD-15/16/17 核心能力、HITL、RBAC/ABAC、幂等、补偿、审计、模拟隔离以及 Agent 禁止直连源系统的约束。

---

## 6. 工具链与自动化

### 6.1 文档工具

- **编写**：Markdown（VS Code + Markdown All in One 插件）
- **图表**：Mermaid（嵌入 Markdown）/ Excalidraw（架构图）
- **版本**：Git 管理，与代码同仓库

### 6.2 契约工具

- **API 契约**：统一使用 OpenAPI 3.1（YAML 格式）；Stage A 目标目录为 `contracts/openapi/`，目录建立前不要求存在；禁止新增 OpenAPI 3.0 契约
- **数据契约**：JSON Schema Draft 2020-12，存放于 `contracts/schemas/`
- **契约测试**：`openapi-spec-validator`、`jsonschema` 与 Stage A Python 检查器
- **类型生成**：仅从通过门禁的 OpenAPI 3.1 / JSON Schema 生成 TypeScript/Python 类型

### 6.3 CI 集成

- **GitHub Actions**：`.github/workflows/doc-check.yml`
- **聚合入口**：`scripts/check_stage_a_gate.py`
- **校验内容**：文档分类与 DoR、相对链接、治理 ID、OpenAPI 3.1/Schema/ADR/Cypher 一致性、台账、追溯、审批摘要和分阶段存在性
- **失败处理**：任一检查失败即返回非零并阻塞合并或发布，不允许 `continue-on-error`

---

## 7. 与现有文档的关系

### 7.1 已有 LLD-01~06 的处理

现有 6 份 LLD 文档属于 **Phase 1-4 的设计文档**，部分已实现（如 LLD-01~03），部分未实现（如 LLD-05 编排层）。

- **保留**：已实现 LLD 作为历史参考，不强制迁移到新模板
- **迁移**：LLD-05（编排层）需按新模板重写为 LLD-08，补充场景化验收标准
- **废弃**：无废弃文档，全部保留为知识资产

### 7.2 当前文件与 Stage A 目标结构

以下清单中的“已有”表示当前磁盘可直接导航；“待新增/待补齐”表示 Stage A 目标，不得在文件创建前参与 LLD↔MODULE 双向完整性校验。

```text
AgentOntology/
├── documentation-system.md               # 已有：Tier-1 文档体系规范
├── agent-ontology-master-plan.md          # 已有：Tier-1 总体规划
├── product-positioning-and-business-model.md # 已有：Tier-1 产品定位
├── dual-ontology-three-layer-architecture-baseline.md # 已有：Tier-1 双本体三层架构基线
├── lld-01-mcp-gateway.md                  # 已有：Tier-2
├── lld-02-stdio-mcp-security.md           # 已有：Tier-2
├── lld-03-business-mcp-oa-crm.md          # 已有：Tier-2
├── lld-04-crm-server.md                   # 已有：Tier-2
├── lld-05-orchestration-layer.md          # 已有：Tier-2，待迁移
├── lld-06-hitl-service.md                 # 已有：Tier-2
├── lld-07-knowledge-base.md               # 已有：Tier-2，Stage A 修订
├── lld-08-workflow-orchestration.md       # 待新增：Tier-2 Stage A
├── lld-09-agent-runtime.md                # 已有：Tier-2，Stage A 修订
├── lld-10-multi-agent-collaboration.md    # 待新增：Tier-2 Stage A 影响分析
├── lld-11-model-config.md                 # 待新增：Tier-2 Stage D 前冻结
├── lld-12-skill-library.md                # 待新增：Tier-2 Stage A 影响分析
├── lld-13-business-connectors.md          # 待新增迁移说明，Deprecated / Migration-only
├── lld-14-governance-config.md            # 待新增：Tier-2 Stage A 影响分析
├── lld-15-ontology-service.md             # 待新增：Tier-2 Stage A
├── lld-16-source-connector-runtime.md     # 待新增：Tier-2 Stage A
├── lld-17-ontology-governance.md          # 待新增：Tier-2 Stage A
├── lld-18-ontology-simulation.md          # 待新增：Tier-2 Stage A 完整 LLD
├── lld-19-document-governance.md          # 已有：Tier-2 Stage A 文档治理完整 LLD
├── lld-19/sections/                        # 已有：LLD-19 十个规范性 section
├── contracts/                             # Stage A 治理、追溯、执行与业务契约
├── contracts/openapi/                     # Stage A 目标：OpenAPI 3.1 契约
├── services/                              # Stage B-D 目标；目录创建后写入 MODULE.md
├── .github/workflows/doc-check.yml        # 已有：待按任务 12 升级为分阶段门禁
└── scripts/check_stage_a_gate.py           # 待任务 11 实现：Stage A 聚合检查器
```

CI 按第 5.2 节分阶段启用：Stage A 先校验已有文件、三类设计文档、治理 ID、DoR、契约、台账和追溯；只有 LLD 有效批准且实现任务在台账中激活后，才校验对应代码目录、`MODULE.md`、测试和迁移产物。

---

## 8. 变更记录

| 版本 | 日期 | 变更内容 | 变更人 |
|------|------|----------|--------|
| V1.0 | 2025-08-02 | 初始版本，基于三轮 Q&A 确认沉淀 | AI + 用户 |
| V1.1 | 2026-08-02 | 对齐双本体三层架构，新增 MOD-15~18，替换旧 8 周排期并明确 MOD-13/14 演进边界 | AI + 用户 |
| V1.2 | 2026-08-03 | 对齐任务 0 与 LLD-19：纳入 MOD-19、OpenAPI 3.1、三类设计文档、完整 LLD DoR、治理 ID、台账、双向追溯和分阶段门禁 | AI + 用户 |

---

> **下一步行动**
> 1. 按执行台账推进任务 0 后续治理产物并保持追溯摘要一致
> 2. 依据三类模板完成 Stage A 设计产物，完整 LLD 达到 DoR 后再申请独立批准
> 3. 仅在批准 LLD 授权活动实现任务后启用对应代码与测试存在性门禁
