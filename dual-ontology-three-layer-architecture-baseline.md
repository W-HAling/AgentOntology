# 双本体三层核心架构与项目阶段新基线

> **文档版本**：v1.0  
> **文档性质**：Tier-1 架构基线  
> **生效日期**：2026-08-02  
> **决策来源**：[Palantir的本体论讨论.docx](./Palantir的本体论讨论.docx) 与用户确认  
> **取代范围**：取代原 W3-W4 MOD-07/MOD-09 直接编码计划  
> **目标边界**：先建设通用本体平台与连接器规范，再接入真实 CRM、ERP、OA 数据源

## 1. 架构目标

项目目标从单一 Agent 平台升级为“双本体三层企业智能体平台”：

1. **业务本体 Business Ontology**：定义企业业务世界中的对象、属性、关系、动作、规则、状态、权限和数据来源。
2. **Agent 本体 Agent Ontology**：定义执行者的目标、能力、知识、记忆、规划、动作与运行环境。
3. **受治理执行**：Agent 不直接操作源系统，只能通过业务本体注册的 Ontology Action，经 MCP Gateway、权限校验、HITL 和审计后执行。

## 2. 三层核心架构

### 2.1 语义层 Semantic Layer

语义层回答“企业业务世界里有什么，以及它们之间是什么关系”。

核心能力：

- ObjectType：业务对象类型；
- PropertyType：对象属性定义；
- RelationType：对象之间的关系定义；
- ObjectInstance：业务对象实例；
- SchemaVersion：本体 Schema 版本；
- SourceMapping：对象属性到源系统字段的映射；
- Query：按对象、属性、关系和图路径查询。

存储职责：

- Neo4j：对象实例、关系和图查询；
- PostgreSQL：Schema、版本、来源映射、权限和变更记录。

### 2.2 动力层 Kinetic Layer

动力层回答“业务对象可以做什么，以及如何安全地改变源系统状态”。

核心能力：

- ActionType：动作名称、输入、输出、风险等级；
- ActionBinding：动作到 REST API、MCP Tool、数据库命令或工作流的绑定；
- CredentialReference：凭据引用，不存储明文凭据；
- AuthorizationPolicy：RBAC/ABAC 权限规则；
- TransactionPolicy：事务、幂等键和一致性约束；
- CompensationAction：失败补偿与回滚动作；
- ActionExecution：动作执行状态、输入、输出和审计；
- SourceSync：业务系统数据到对象实例的增量同步。

执行原则：

- 所有动作必须注册后才能被 Agent 或 Workflow 发现；
- 高风险写操作必须通过 HITL；
- 动作调用必须携带用户身份和权限上下文；
- 写回失败必须执行补偿或进入人工处理；
- MCP 是统一 Agent 工具协议，不等同于源系统连接协议。

### 2.3 动态层 Dynamic Layer

动态层回答“在当前业务上下文中，系统应该如何决策、推演和执行”。

核心能力：

- Rule：绑定对象与动作的业务规则；
- Workflow：确定性业务编排；
- AgentPolicy：Agent 可用对象、关系和动作范围；
- Decision：LLM 或规则引擎产生的决策；
- Simulation：在隔离工作区内运行假设参数与情景推演；
- HITL Gate：高风险决策的人工批准；
- Audit：完整记录规则、模型、动作和状态变化。

动态层同时支持：

- Workflow 模式：路径预定义，适合稳定流程；
- Agent 模式：路径动态规划，适合开放任务；
- Hybrid 模式：Workflow 固定边界，Agent 在节点内自主执行。

## 3. 双本体关系

```text
业务本体
├── ObjectType / PropertyType / RelationType
├── ObjectInstance / SourceMapping
├── ActionType / ActionBinding
└── Rule / Policy / Simulation

Agent 本体
├── Agent / Goal / Capability
├── Knowledge / Memory / Planning
├── Action / Environment
└── Execution / Checkpoint / Audit

连接关系
Agent.Capability → Ontology.ActionType
Agent.Knowledge → Ontology.ObjectType / RelationType / MOD-07
Agent.Action → Ontology.ActionExecution → MCP Gateway
Agent.Environment → SourceSystem / SourceMapping
Agent.Planning → Rule / Workflow / Simulation
```

业务本体是 Agent 认识和操作企业世界的语言；Agent 本体是使用该语言完成目标的执行者模型。

## 4. 平台模块重构

### 4.1 新增模块

| LLD | 模块 ID | 模块 | 主要职责 |
|-----|---------|------|----------|
| LLD-15 | MOD-15 | ontology-service | 业务本体 Schema、对象、关系、动作、规则注册与查询 |
| LLD-16 | MOD-16 | source-connector-runtime | 数据源连接器、字段映射、增量同步、写回和补偿 |
| LLD-17 | MOD-17 | ontology-governance | RBAC/ABAC、策略、版本审批、数据血缘和审计 |
| LLD-18 | MOD-18 | ontology-simulation | 隔离情景、参数覆盖、推演执行和结果对比 |

### 4.2 现有模块重新定位

| 模块 | 新定位 |
|------|--------|
| MOD-07 knowledge-base | 非结构化知识、文档、Embedding 与 RAG，不承载业务对象关系 |
| MOD-08 workflow-orchestration | 动态层确定性 Workflow 与 Hybrid 编排 |
| MOD-09 agent-runtime | 动态层 Agent 执行者，消费业务本体而不是自定义业务接口 |
| MOD-10 multi-agent | 多 Agent 协作，基于统一业务本体交换任务和结果 |
| MOD-11 model-config | LLM/Embedding 模型配置与路由 |
| MOD-12 skill-library | Agent Capability 与 Ontology Action 的绑定和复用 |
| MOD-13 business-connectors | Deprecated / Migration-only；禁止新增运行时能力，`services/connectors/` 仅保留兼容壳层或迁移适配，存量能力迁移至 MOD-16 后删除独立部署单元 |
| MOD-14 governance | 与 MOD-17 协同，负责 Agent 与 Workflow 治理 |
| mcp-gateway | 动力层受治理动作执行入口 |
| hitl-service | 动力层与动态层高风险操作审批闸门 |
| crm/oa-mcp-server | 未来作为 MOD-16 的具体连接器适配器 |

## 5. 通用本体平台边界

当前采用“先平台后数据源”。本阶段完成：

- 通用 ObjectType、PropertyType、RelationType、ActionType、Rule Schema；
- Neo4j + PostgreSQL 双存储；
- REST/OpenAPI 与 MCP 元数据发现接口；
- 连接器 SDK 与 Action Binding 规范；
- RBAC/ABAC、HITL、审计、幂等、补偿协议；
- CRM、ERP、OA 三域完整 Schema 包与模拟适配器；
- Agent Runtime 基于本体发现对象和动作；
- Workflow 基于本体动作执行确定性流程；
- Docker Compose 本地演示环境。

本阶段不承诺：

- 未提供沙箱和凭据的真实 CRM、ERP、OA 系统联调；
- 针对特定厂商 SAP、Salesforce、用友、金蝶、飞书、钉钉的生产连接器；
- 跨真实系统的分布式 ACID 事务；
- 与 Palantir 产品实现或内部协议完全一致。

## 6. 三业务域 Schema 基线

### 6.1 CRM

核心对象：Customer、Contact、Opportunity、Contract、SalesActivity、SalesRepresentative。

核心关系：

- Customer HAS_CONTACT Contact；
- Customer HAS_OPPORTUNITY Opportunity；
- Opportunity GENERATES Contract；
- SalesRepresentative OWNS Opportunity；
- SalesActivity RELATES_TO Customer。

核心动作：

- search_customers；
- score_opportunity；
- create_opportunity；
- update_opportunity_stage；
- generate_contract_draft；
- approve_contract。

### 6.2 ERP

核心对象：Supplier、Material、PurchaseRequest、PurchaseOrder、InventoryItem、GoodsReceipt。

核心关系：

- Supplier SUPPLIES Material；
- PurchaseRequest CREATES PurchaseOrder；
- PurchaseOrder CONTAINS Material；
- Material HAS_INVENTORY InventoryItem；
- PurchaseOrder RECEIVED_AS GoodsReceipt。

核心动作：

- query_inventory；
- evaluate_supplier；
- create_purchase_request；
- create_purchase_order；
- receive_goods；
- cancel_purchase_order。

### 6.3 OA

核心对象：Employee、Department、LeaveRequest、ApprovalTask、Document、Meeting。

核心关系：

- Employee BELONGS_TO Department；
- Employee SUBMITS LeaveRequest；
- LeaveRequest REQUIRES ApprovalTask；
- Employee AUTHORS Document；
- Employee ATTENDS Meeting。

核心动作：

- query_employee_schedule；
- submit_leave_request；
- approve_leave_request；
- reject_leave_request；
- publish_document；
- schedule_meeting。

## 7. 新阶段路线

### Stage A：双本体架构与契约冻结

交付：

- 修订 Tier-1 总体规划；
- LLD-15 业务本体服务；
- LLD-16 连接器运行时；
- LLD-17 本体治理；
- LLD-18 情景模拟；
- 修订 LLD-07、LLD-08、LLD-09 的双本体边界；
- 完成 LLD-10、LLD-12、LLD-14 的影响分析和迁移声明；
- 三域 Schema 契约；
- ADR、治理职责矩阵与验收场景。

门禁：LLD-15~18 与修订后的 LLD-07/08/09 通过评审，LLD-10/12/14 完成影响分析；OpenAPI、JSON Schema、Cypher 模型和治理职责矩阵无冲突。

### Stage B：语义层平台

交付：

- ontology-service；
- MOD-17 策略存储、版本治理、数据血缘和审计底座；
- Neo4j/PostgreSQL 存储与迁移；
- Schema 版本管理；
- 对象、关系、图查询；
- 三域 Schema 导入；
- 本体管理后台基础页面。

门禁：三域对象和关系可创建、查询、版本化并隔离，策略版本、血缘和审计治理元数据可追溯。

### Stage C：动力层平台

交付：

- connector-runtime；
- Action Registry；
- MOD-17 动作授权接入；
- MCP Tool 发现；
- 权限、HITL、幂等、补偿；
- 三域模拟 REST/MCP 适配器；
- 动作执行追踪。

门禁：每个业务域至少完成完整读写链路，并完成授权允许、授权拒绝、高风险审批、幂等和副作用动作补偿验证。

### Stage D：动态层平台

交付：

- MOD-09 Agent Runtime；
- MOD-08 Workflow；
- MOD-18 ontology-simulation 隔离执行引擎；
- 规则引擎；
- Hybrid 执行；
- MOD-07 RAG；
- Checkpoint 与三层记忆；
- Agent/Workflow 本体化执行。

门禁：Agent 和 Workflow 均只能通过注册 Action 操作业务对象，越权调用被拒绝。

### Stage E：三域完整验证

交付：

- CRM、ERP、OA 三域端到端业务流；
- 跨域查询与动作编排；
- 情景模拟；
- 治理与审计报告；
- Docker Compose 演示环境；
- E2E 验收套件。

门禁：三域所有定义动作完成成功、失败、授权与审计验证；高风险动作额外完成审批、拒绝和超时验证；具有外部副作用的写动作额外完成幂等与补偿验证；只读动作验证无副作用。

### Stage F：真实数据源接入

前置条件：用户提供系统 API、沙箱环境、凭据管理方案和字段映射。

交付：逐域替换模拟适配器，不修改上层本体、Agent 和 Workflow 契约。

门禁：上层契约兼容；凭据隔离、字段映射校验、增量同步与重放、限流与超时、写回幂等、补偿、审计完整性和生产回滚演练全部通过。

## 8. 治理与运行时职责矩阵

| 能力 | 定义与控制面 | 决策/执行点 | 事件与记录 |
|------|--------------|-------------|------------|
| 本体对象与 Action 的 RBAC/ABAC | MOD-17 | MOD-17 策略决策，MOD-16/MCP Gateway 强制执行 | MOD-17 持久化决策与审计 |
| Agent/Workflow 治理配置 | MOD-14 | MOD-08/MOD-09 读取策略引用，不自行求值本体权限 | MOD-08/MOD-09 生产运行事件 |
| HITL 触发条件 | MOD-14 配置，发布到 MOD-17 | MOD-16/MCP Gateway 触发，hitl-service 管理工单生命周期 | MOD-17 汇聚审批审计 |
| 数据脱敏与字段访问 | MOD-17 | MOD-16/MCP Gateway 强制执行 | MOD-17 记录策略版本和结果 |
| Action 执行与补偿 | MOD-16 | MOD-16 与具体连接器适配器 | MOD-16 生产统一 ActionExecution 事件，MOD-17 持久化 |
| Simulation | 业务本体定义情景与参数 Schema | MOD-18 隔离执行 | MOD-18 生产结果，MOD-17 记录血缘与审计 |

Agent 本体仅保存 `ExecutionRef`、`CheckpointRef`、`PolicyBindingRef` 和 `SimulationRef` 等引用。MOD-08、MOD-09、MOD-16、MOD-18 生产统一审计事件，MOD-17 负责事件规范、不可变持久化、数据血缘和查询，禁止各模块建立独立审计事实源。

## 9. 核心验收原则

1. 同一业务对象不暴露源系统表结构给 Agent。
2. Agent 通过 ObjectType、RelationType 和 ActionType 理解业务上下文。
3. 每个 Action 有输入输出 Schema、权限、风险、幂等和补偿定义。
4. 所有写动作可追溯到用户、Agent、规则、模型、输入和源系统结果。
5. 高风险动作未获批准不能写回。
6. 模拟执行不得修改生产对象或源系统。
7. 三域 Schema 使用同一平台引擎，不为每个域复制核心代码。
8. 未来替换真实连接器时，上层本体和 Agent 契约保持稳定。

## 10. 风险与控制

| 风险 | 等级 | 控制措施 |
|------|------|----------|
| 三域完整范围过大 | 高 | 平台能力先行，三域通过 Schema 包和模拟适配器复用引擎 |
| Neo4j/PostgreSQL 一致性 | 高 | PostgreSQL 管 Schema，Neo4j 管实例关系；使用 Outbox 和可重放同步 |
| 跨系统事务不可控 | 高 | Saga、幂等键、补偿动作和人工介入，不承诺全局 ACID |
| Agent 越权或误操作 | 高 | Action 白名单、ABAC、HITL、参数约束、审计与速率限制 |
| 将 MCP 等同连接器 | 中 | MCP 仅作为工具协议，连接器仍负责源协议、映射、凭据和事务 |
| 将 RAG 等同本体 | 中 | Qdrant 仅承载非结构化知识，本体语义由 Neo4j/PostgreSQL 承载 |
| 将概念性讨论当作官方实现 | 中 | 文档明确“Palantir 风格”，不声称复刻专有内部实现 |

## 11. 被取代的计划

[w3-w4-mod07-mod09-implementation-design.md](./w3-w4-mod07-mod09-implementation-design.md) 作为历史决策保留，但其“直接开始 MOD-07/MOD-09 一次完整编码”的执行结论已失效。

任何后续编码必须从 Stage A 开始，并以本文件及新增 LLD 为依据。
