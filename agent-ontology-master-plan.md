# 企业级智能体本体基座（Agent Ontology）用户需求及系统规划

**文档版本**：v3.0  
**编写日期**：2026-08-02  
**文档类型**：项目级规划（Master Planning / Tier-1）  
**当前架构基线**：[双本体三层核心架构](./dual-ontology-three-layer-architecture-baseline.md)  
**需求调研来源**：[Palantir的本体论讨论.docx](./Palantir的本体论讨论.docx)  
**前置文档**：[产品定位与商业模式](./product-positioning-and-business-model.md)、[三级文档体系规范](./documentation-system.md)  
**关联文档**：[企业级智能体架构 v1.1](./enterprise-agent-architecture-v1.1.md)、[LLD-05 编排层详细设计](./lld-05-orchestration-layer.md)

---

## 目录

- [一、项目愿景与定位](#一项目愿景与定位)
- [二、核心需求澄清](#二核心需求澄清)
- [三、系统设计目标](#三系统设计目标)
- [四、核心结构（本体模型）](#四核心结构本体模型)
- [五、核心应用架构](#五核心应用架构)
- [六、后台配置能力清单](#六后台配置能力清单)
- [七、当前项目能力对标](#七当前项目能力对标)
- [八、分阶段实施路线图](#八分阶段实施路线图)
- [九、技术选型与依赖](#九技术选型与依赖)
- [十、风险与缓解策略](#十风险与缓解策略)
- [十一、文档体系与执行保障](#十一文档体系与执行保障)
- [附录 A：术语表](#附录-a术语表)
- [附录 B：参考资料](#附录-b参考资料)
- [附录 C：LLD 拆分清单](#附录-clld-拆分清单)

---

## 一、项目愿景与定位

### 1.1 项目愿景

构建一个**企业级双本体智能体基座**：以 Palantir 风格的业务本体三层架构作为企业业务语义和受控操作底座，以 Agent Ontology 作为执行者模型，使企业能够通过可视化配置构建、部署和治理具备业务理解、自主规划、受控执行、工作流编排与多智能体协作能力的应用。

系统中的两个本体边界如下：

- **业务本体（Business Ontology）**：定义 Customer、Order、Contract 等对象、属性、关系、动作、规则、权限和数据来源；
- **Agent 本体（Agent Ontology）**：定义 Agent、Goal、Capability、Knowledge、Memory、Planning、Action、Environment；
- **连接原则**：Agent 的 Capability 只能绑定经过注册和治理的 Ontology Action，Agent 不直接操作业务系统 API、数据库或凭据。

### 1.2 项目定位

本项目是一个**企业内部 AI 能力中台**（内部工具优先），而非对外销售的 SaaS 产品或面向开发者的通用 SDK。

**核心定位**：
- **内部工具优先**：先服务企业内部业务系统接入与 AI 工作流自动化，跑通端到端场景后再评估对外产品化
- **企业级场景**：强调数据主权、安全合规、私有化部署
- **平台型基座**：与 Dify/Coze 同赛道，但专注于企业私有化部署与治理

**与竞品的差异**：
- **vs Dify/Coze**：我们不是云端 SaaS，而是私有化部署的企业级替代方案，核心差异在于数据不出域、安全治理、业务系统深度集成
- **vs Codex/千问办公**：不同赛道，他们是垂直型 AI 应用（代码生成/办公助手），我们是平台型 Agent 基座，无直接竞争关系

**三大差异化锚点（三者均衡）**：
1. **企业级安全与治理**：基于 MCP 网关（OAuth 2.1 + JWKS + HMAC）+ HITL 人工审批 + 全链路审计日志
2. **业务本体与连接器运行时**：通过语义层、动力层、动态层统一对象、关系、动作、规则、权限和源系统映射；MCP 是 Agent 工具协议，连接器负责源协议、凭据、映射、事务与补偿
3. **双本体建模**：业务本体描述企业业务世界，Agent Ontology 8 要素描述执行者；两者通过 Capability → Ontology Action 形成受治理执行闭环

### 1.3 与当前项目的关系

当前项目（Phase 1~4）已完成**安全与协议底座**的建设，包括：
- MCP 网关（OAuth 2.1 + JWKS + HMAC 签名 + 语义路由 + 审计）
- HITL 人工介入服务（审批工单 + SLA 超时 + PostgreSQL 持久化）
- CRM 业务系统连接示例（10 个工具 + 权限过滤）
- 前端工程骨架（PC + H5 + OAuth 登录闭环 + 6 个业务页面原型）
- 三层测试体系（单测 + 集成 + E2E）

**本文档是下一阶段（Phase 5~8）的总纲**，指导从「安全与协议底座」向「智能体本体基座」的演进。

---

## 二、核心需求澄清

### 2.1 核心需求（用户原话）

> 想实现一个企业级智能体本体基座，它可以：
> 1. **管理知识库**：支持文档上传、向量化、语义检索（RAG）
> 2. **自主编排 AI 工作流**：工作流的节点可以调用智能体
> 3. **智能体自主性**：智能体可以自主规划、意图识别、任务拆解、Tool 调用、MCP 连接、协同 Sub Agent
> 4. **连接其他业务系统**：拉取数据，执行特定的业务场景（如 ERP、CRM、数据库、OA）

### 2.2 需求拆解（7 项核心能力）

| # | 核心能力 | 详细描述 | 优先级 |
|---|----------|----------|--------|
| 1 | **知识库管理** | 支持文档上传（PDF/Word/TXT/Markdown）、自动切片、向量化（Embedding）、语义检索（RAG）、知识库版本管理 | P0 |
| 2 | **工作流编排** | 可视化拖拽式工作流设计器、节点类型（LLM/Tool/Condition/Loop/HITL）、工作流版本管理、工作流执行引擎 | P0 |
| 3 | **智能体自主性** | 智能体可自主规划（Planning）、意图识别（Intent Recognition）、任务拆解（Task Decomposition）、Tool 调用（Function Calling） | P0 |
| 4 | **多智能体协同** | 支持 Supervisor/Worker 角色定义、智能体间消息传递、智能体注册/发现、智能体协同工作流 | P1 |
| 5 | **MCP 连接** | 通过 MCP 协议统一收口，支持业务系统插件化接入（ERP/CRM/数据库/OA），支持工具路由、限流、审计 | ✅ 已实现 |
| 6 | **业务系统连接** | 支持常见业务系统的预置连接器（CRM/ERP/OA/数据库），支持自定义连接器开发 | P1 |
| 7 | **端到端业务场景** | 支持常见业务场景的端到端实现（如客户流失预警、智能客服、自动化报表生成） | P2 |

### 2.3 非功能性需求

| # | 非功能性需求 | 详细描述 | 优先级 |
|---|--------------|----------|--------|
| 1 | **安全性** | OAuth 2.1 + OIDC + PKCE 认证、HMAC 签名注入、数据脱敏、审计日志 | ✅ 已实现 |
| 2 | **可治理性** | HITL 人工介入、SLA 超时、权限过滤、数据主权 | ✅ 已实现 |
| 3 | **可扩展性** | MCP 协议统一收口、业务系统插件化接入、水平扩展 | 🔄 部分实现 |
| 4 | **可观测性** | Prometheus 指标、审计日志、全链路追踪、性能监控 | 🔄 部分实现 |
| 5 | **高可用性** | 多副本部署、故障转移、降级策略、断点恢复 | 🔄 部分实现 |
| 6 | **性能** | 工作流执行延迟 < 5s（P95）、知识库检索延迟 < 500ms（P95）、并发支持 1000+ 智能体 | P1 |

---

## 三、系统设计目标

### 3.1 总体目标

构建一个**拥有完整后台配置能力的企业级智能体本体基座**，使企业能够通过可视化配置的方式，快速构建、部署、治理智能体应用。

### 3.2 后台配置能力清单（8 大模块）

| # | 配置模块 | 功能描述 | 当前状态 |
|---|----------|----------|----------|
| 1 | **模型配置** | LLM 模型接入（OpenAI/Anthropic/本地模型）、模型参数配置（temperature/top_p/max_tokens）、模型路由策略（成本/性能/延迟）、模型降级策略 | ❌ 缺失 |
| 2 | **业务系统对接配置** | 业务系统连接器配置（ERP/CRM/数据库/OA）、连接参数配置（URL/认证/超时）、连接测试、连接监控 | 🔄 部分实现（仅 CRM 示例） |
| 3 | **Capability 绑定管理** | Capability 输入输出契约、ActionType 绑定、版本管理、PolicyBindingRef 与调用指标展示 | ❌ 缺失 |
| 4 | **MCP 元数据与监控** | MCP Server/Tool 发现、ActionType/ActionBinding 映射、版本与调用监控；权限由 MOD-17 求值 | 🔄 部分实现（mcp-gateway 已实现路由与审计） |
| 5 | **AI 编排** | 工作流可视化设计器、节点类型（LLM/Tool/Condition/Loop/HITL）、工作流版本管理、工作流执行引擎 | ❌ 缺失（仅有 LLD-05 设计文档） |
| 6 | **智能体管理** | 智能体定义（Goal/Capability/Knowledge/Memory/Planning/Action/Environment）、智能体生命周期管理、智能体监控 | ❌ 缺失 |
| 7 | **知识库管理** | 知识库创建/删除、文档上传/切片/向量化、语义检索（RAG）、知识库版本管理 | ❌ 缺失 |
| 8 | **治理配置** | HITL 人工介入规则、SLA 超时配置、权限过滤规则、数据脱敏规则、审计日志查询 | 🔄 部分实现（HITL 已实现，其他缺失） |

### 3.3 目标用户角色

| 角色 | 职责 | 核心诉求 |
|------|------|----------|
| **平台管理员** | 配置模型、业务系统、MCP 库、治理规则 | 简单易用的配置界面、完善的权限管理 |
| **智能体开发者** | 创建智能体、定义 Goal/Capability/Knowledge/Memory/Planning/Action/Environment | 强大的编排能力、丰富的 Skill 库、完善的调试工具 |
| **业务用户** | 使用智能体应用、与智能体交互、审批 HITL 工单 | 简单易用的交互界面、快速的响应速度 |
| **审计人员** | 查询审计日志、监控智能体行为、分析智能体性能 | 完整的审计日志、可视化的监控面板 |

---

## 四、核心结构（本体模型）

### 4.1 本体模型概述

**Agent Ontology（智能体本体）**是对智能体的形式化描述，定义了智能体的核心结构与能力。本体模型是平台的核心数据结构，所有智能体的创建、配置、执行、治理都基于本体模型。

### 4.2 核心结构（8 个要素）

```typescript
interface AgentOntology {
  id: string;                      // 智能体唯一标识
  name: string;                    // 智能体名称
  description: string;             // 智能体描述
  version: string;                 // 智能体版本
  agent: Agent;                    // 智能体本体
  goal: Goal;                      // 目标
  capability: Capability[];        // 技能列表
  knowledge: Knowledge[];          // 知识列表
  memory: Memory;                  // 记忆
  planning: Planning;              // 规划
  action: Action[];                // 行动列表
  environment: Environment[];      // 环境列表
  governance: Governance;          // 治理配置
  metadata: Metadata;              // 元数据
}
```

#### 4.2.1 Agent（智能体）

**定义**：智能体是平台的执行单元，负责执行具体的任务。

**核心属性**：
```typescript
interface Agent {
  id: string;                      // 智能体唯一标识
  name: string;                    // 智能体名称
  type: AgentType;                 // 智能体类型（orchestrator/worker/critic/tool）
  role: string;                    // 智能体角色（如"数据分析师"/"客服助手"）
  persona: string;                 // 智能体人设（性格/语气/风格）
  llm_config: LLMConfig;           // LLM 配置（模型/参数/路由策略）
  status: AgentStatus;             // 智能体状态（active/paused/offline）
}
```

**职责**：
- 执行任务（通过 Planning 生成计划，通过 Action 执行行动）
- 与其他智能体协同（通过消息传递机制）
- 上报执行状态（通过监控与审计日志）

---

#### 4.2.2 Goal（目标）

**定义**：目标是智能体的使命，定义了智能体要解决什么问题、达成什么结果。

**核心属性**：
```typescript
interface Goal {
  id: string;                      // 目标唯一标识
  description: string;             // 目标描述（如"提升客户满意度"）
  success_criteria: string[];      // 成功标准（如"客户满意度 > 90%"）
  constraints: string[];           // 约束条件（如"预算 < 10000 元"）
  priority: Priority;              // 优先级（low/medium/high/critical）
  deadline: Date;                  // 截止时间
}
```

**职责**：
- 拆解任务（将目标拆解为可执行的子任务）
- 规划任务（生成任务执行计划）
- 验收任务（验证任务执行结果是否满足成功标准）

---

#### 4.2.3 Capability（能力）

**定义**：能力是 Agent 对受治理业务动作的可复用声明，只描述语义、契约和绑定，不包含独立执行逻辑。

**核心属性**：
```typescript
interface Capability {
  id: string;
  name: string;
  description: string;
  input_schema: JSONSchema;
  output_schema: JSONSchema;
  ontology_action_type_id: string;
  policy_binding_ref: string;
  cost_profile_ref: string;
  version: string;
}
```

**职责**：
- 声明 Agent 能完成的业务能力；
- 通过 MOD-12 绑定 MOD-15 注册的 Ontology ActionType；
- 复用输入输出契约、版本和成本画像；
- 不保存 Prompt、Tool、代码、连接器、凭据或权限规则，权限统一由 MOD-17 求值。

---

#### 4.2.4 Knowledge（知识）

**定义**：知识是 Agent 可使用的受治理知识引用，包括 MOD-07 的非结构化 RAG 知识，以及 MOD-15 的业务对象和关系类型引用。

**核心属性**：
```typescript
interface Knowledge {
  id: string;
  name: string;
  knowledge_base_refs: string[];
  ontology_object_type_refs: string[];
  ontology_relation_type_refs: string[];
  retrieval_policy_ref: string;
  version: string;
}
```

**职责**：
- 引用 MOD-07 管理的文档、Embedding 和 RAG 检索配置；
- 引用 MOD-15 管理的 ObjectType 与 RelationType；
- 通过 MOD-17 约束可读取的对象、字段和数据范围；
- 不保存 API URL、数据库连接、源系统凭据或向量数据库连接配置。

---

#### 4.2.5 Memory（记忆）

**定义**：记忆是智能体的上下文，定义了智能体可以记住的信息。

**核心属性**：
```typescript
interface Memory {
  short_term: ShortTermMemory;     // 短期记忆（当前会话上下文）
  long_term: LongTermMemory;       // 长期记忆（跨会话知识沉淀）
  working: WorkingMemory;          // 工作记忆（当前任务状态）
}

interface ShortTermMemory {
  max_tokens: number;              // 最大 token 数
  compression_strategy: string;    // 压缩策略（summarization/truncation）
}

interface LongTermMemory {
  storage: StorageConfig;          // 存储配置（PostgreSQL/Redis/向量数据库）
  retrieval_strategy: string;      // 检索策略（similarity/recency/importance）
}

interface WorkingMemory {
  state_schema: JSONSchema;        // 状态 Schema
  checkpoint_strategy: string;     // Checkpoint 策略（PostgreSQL/Redis）
}
```

**职责**：
- 管理智能体的上下文（短期/长期/工作记忆）
- 支持上下文压缩（长对话场景）
- 支持断点恢复（Checkpoint 持久化）

---

#### 4.2.6 Planning（规划）

**定义**：规划是智能体的思考过程，定义了智能体如何拆解任务、生成计划。

**核心属性**：
```typescript
interface Planning {
  strategy: PlanningStrategy;      // 规划策略（react/plan_and_execute/reflexion）
  max_iterations: number;          // 最大迭代次数
  llm_config: LLMConfig;           // LLM 配置（用于规划）
  prompt_template: string;         // Prompt 模板
  validation_rules: string[];      // 验证规则（如"计划必须包含至少一个行动"）
}
```

**职责**：
- 拆解任务（将 Goal 拆解为可执行的子任务）
- 生成计划（生成任务执行计划）
- 验证计划（验证计划的合理性与可行性）

---

#### 4.2.7 Action（行动）

**定义**：行动是智能体对已注册 Ontology Action 的受治理引用，不保存源工具、连接器或凭据配置。

**核心属性**：
```typescript
interface Action {
  id: string;
  name: string;
  ontology_action_type_id: string;
  input_mapping: Record<string, string>;
  output_mapping: Record<string, string>;
  execution_policy_ref: string;
  timeout_seconds: number;
}
```

**职责**：
- 引用业务本体中已注册的 Ontology Action；
- 将规划上下文映射到动作输入，并将动作输出映射回执行状态；
- 遵循 MOD-16、MOD-17、MCP Gateway 与 HITL 的授权和执行结果；
- 禁止保存 MCP Tool、自定义工具、代码执行器或源系统绑定。

---

#### 4.2.8 Environment（环境）

**定义**：环境是智能体可运行的逻辑上下文，仅描述租户、业务域、数据作用域和部署隔离级别。

**核心属性**：
```typescript
interface Environment {
  id: string;
  name: string;
  tenant_id: string;
  business_domain_refs: string[];
  data_scope_refs: string[];
  deployment_stage: "simulation" | "sandbox" | "production";
  status: "active" | "inactive";
}
```

**职责**：
- 约束 Agent 可见的业务域、对象范围和运行阶段；
- 为 MOD-17 策略求值提供租户与数据作用域上下文；
- 引用 MOD-16 管理的 SourceSystem 和 SourceMapping，不保存 URL、认证信息、MCP Server 配置或连接参数；
- 连接测试、凭据引用、字段映射和源系统监控统一由 MOD-16 负责。

---

### 4.3 本体模型的存储与管理

**存储方案**：
- **Neo4j**：存储业务本体 ObjectInstance、RelationInstance 与图路径查询；
- **PostgreSQL**：存储业务本体 Schema、版本、SourceMapping、权限和变更记录，以及 Agent Ontology 八要素的结构化定义；
- **向量数据库**：存储非结构化知识库的向量数据，不承载业务对象关系；
- **Redis**：存储智能体短期记忆、工作记忆与运行热缓存；
- **对象存储**：存储知识库原始文档和大对象。

**管理接口**：
- **REST API**：提供本体模型的 CRUD 接口
- **GraphQL**：提供本体模型的灵活查询接口
- **WebSocket**：提供智能体执行状态的实时推送接口

---

## 五、核心应用架构

### 5.1 架构分层

```text
┌────────────────────────────────────────────────────────────────────┐
│ 用户交互与开放接口                                                  │
│ PC 管理后台 / H5 / Open API                                        │
└──────────────────────────────┬─────────────────────────────────────┘
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│ 动态层 Dynamic Layer                                               │
│ MOD-08 Workflow / MOD-09 Agent Runtime / Rule / Hybrid / MOD-18   │
│ Agent Planning 只能选择 Capability 已绑定的 Ontology Action        │
└──────────────────────────────┬─────────────────────────────────────┘
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│ 语义层 Semantic Layer                                              │
│ MOD-15：ObjectType / RelationType / ObjectInstance / ActionType    │
│ Neo4j：实例与关系；PostgreSQL：Schema、版本、映射和治理元数据       │
└──────────────────────────────┬─────────────────────────────────────┘
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│ 动力层 Kinetic Layer                                               │
│ MOD-17 策略决策 → MOD-16 ActionBinding/幂等/补偿 → MCP Gateway     │
│ 高风险动作经 hitl-service；凭据、字段映射和源协议仅由 MOD-16 管理   │
└──────────────────────────────┬─────────────────────────────────────┘
                               ▼
┌────────────────────────────────────────────────────────────────────┐
│ 源系统与知识支撑                                                    │
│ CRM / ERP / OA 连接器适配器；MOD-07 文档、Embedding 与 RAG          │
│ Agent 不直接访问源系统 API、数据库、凭据或连接配置                  │
└────────────────────────────────────────────────────────────────────┘
```

治理能力横切三层：MOD-14 管理 Agent/Workflow 治理配置和策略引用；MOD-17 统一管理 RBAC/ABAC、版本、血缘与审计事实；MOD-08/09/16/18 仅生产统一运行事件。

### 5.2 核心调用链

```
用户请求
  ↓
[Agent Ontology] 解析智能体本体（Agent/Goal/Capability/Knowledge/Memory/Planning/Action/Environment）
  ↓
[Agent Runtime] 启动智能体执行引擎
  ↓
[Planning] 拆解任务、生成计划（ReAct/Plan-and-Execute/Reflexion）
  ↓
[Action] 解析 Ontology Action 引用与参数映射
  ↓
[MOD-17] 基于用户、Agent、租户、对象和动作上下文执行 RBAC/ABAC 策略决策
  ↓
[MOD-16] 解析 ActionBinding、幂等与补偿策略，高风险动作触发 HITL
  ↓
[MCP Gateway] OAuth 2.1 校验 + HMAC 签名注入 + 受治理路由 + 审计事件
  ↓
[连接器适配器] 按源协议执行读写，不向 Agent 暴露凭据和源系统结构
  ↓
[Knowledge] 检索相关知识（RAG）
  ↓
[Memory] 更新智能体记忆（短期/长期/工作记忆）
  ↓
[HITL] 人工介入（如需审批）
  ↓
返回结果给用户
```

### 5.3 关键流程

#### 5.3.1 智能体创建流程

1. **定义 Agent**：配置智能体名称/类型/角色/人设/LLM 配置
2. **定义 Goal**：配置目标描述/成功标准/约束条件/优先级/截止时间
3. **配置 Capability**：从 MOD-12 选择已绑定 Ontology ActionType 的能力，或创建新的受治理绑定
4. **配置 Knowledge**：从知识库选择知识源，或创建新知识库
5. **配置 Memory**：配置短期/长期/工作记忆策略
6. **配置 Planning**：配置规划策略（ReAct/Plan-and-Execute/Reflexion）
7. **配置 Action**：从 Action Registry 选择已注册 Ontology Action，配置输入输出映射
8. **配置 Environment**：选择租户、业务域、数据作用域和运行阶段，不配置源系统连接
9. **配置 Governance**：引用 MOD-14 管理的 Agent/Workflow 治理配置，由 MOD-17 统一求值权限策略
10. **发布智能体**：校验所有 Capability 均绑定合法 Ontology Action 后进入可用状态

#### 5.3.2 工作流执行流程

1. **接收用户请求**：用户通过前端或 API 发起请求
2. **解析智能体本体**：加载智能体的 Agent/Goal/Capability/Knowledge/Memory/Planning/Action/Environment
3. **启动规划引擎**：根据 Planning 策略拆解任务、生成计划
4. **执行行动**：根据计划调用已注册 Ontology Action，经 MOD-17 授权后由 MOD-16 和 MCP 网关执行
5. **检索知识**：根据需要从 Knowledge 检索相关知识（RAG）
6. **更新记忆**：将执行结果写入 Memory（短期/长期/工作记忆）
7. **人工介入**：如触发 HITL 规则，暂停执行，等待人工审批
8. **返回结果**：将执行结果返回给用户
9. **记录审计日志**：记录完整的执行链路（请求/响应/工具调用/人工介入）

---

## 六、后台配置能力清单

### 6.1 模型配置

**功能**：
- LLM 模型接入（OpenAI/Anthropic/本地模型）
- 模型参数配置（temperature/top_p/max_tokens/stop_sequences）
- 模型路由策略（成本优先/性能优先/延迟优先）
- 模型降级策略（主模型失败时切换到备用模型）
- 模型监控（调用次数/延迟/成本/错误率）

**界面**：
- 模型列表页：展示已接入的模型（名称/提供商/状态/调用次数/成本）
- 模型配置页：配置模型参数（temperature/top_p/max_tokens）
- 模型路由页：配置路由策略（成本/性能/延迟）
- 模型监控页：展示模型监控指标（调用次数/延迟/成本/错误率）

---

### 6.2 业务系统对接配置

**功能**：
- 业务系统连接器配置（ERP/CRM/数据库/OA）
- 连接参数配置（URL/认证/超时）
- 连接测试（验证连接是否可用）
- 连接监控（监控连接状态/延迟/错误率）
- 连接器市场（预置连接器 + 自定义连接器）

**界面**：
- 连接器列表页：展示已配置的连接器（名称/类型/状态/延迟/错误率）
- 连接器配置页：配置连接参数（URL/认证/超时）
- 连接器测试页：测试连接是否可用
- 连接器监控页：展示连接监控指标（状态/延迟/错误率）

---

### 6.3 Capability 绑定管理（MOD-12）

**功能**：
- Capability 语义与输入输出契约定义；
- Capability → MOD-15 Ontology ActionType 绑定；
- 绑定版本管理（创建/发布/回滚）；
- PolicyBindingRef 配置，权限策略由 MOD-17 统一定义和求值；
- 绑定调用指标展示，不承载执行逻辑。

**界面**：
- Capability 列表页：展示名称、描述、版本和绑定的 ActionType；
- Capability 配置页：配置输入输出映射、ActionType 和 PolicyBindingRef；
- Capability 市场页：复用已批准的绑定模板；
- 权限结果页：只展示 MOD-17 策略决策，不在 MOD-12 定义第二套权限。

---

### 6.4 MCP 元数据与监控

**功能**：
- MCP Server 与 Tool 元数据发现；
- MCP Tool 元数据映射到 MOD-15 ActionType 和 MOD-16 ActionBinding；
- Tool 调用权限由 MOD-17 统一定义和求值；
- MOD-16/MCP Gateway 作为唯一执行强制点；
- MCP Tool 版本、调用次数、延迟和错误率监控。

**界面**：
- MCP Server 列表页：展示已发现 Server、状态和 Tool 数量；
- MCP Tool 列表页：展示元数据、关联 ActionType/ActionBinding 和调用指标；
- MCP Tool 映射页：配置元数据映射，不配置独立权限或旁路执行；
- MCP Tool 监控页：展示调用指标和 MOD-17 授权结果。

---

### 6.5 AI 编排

**功能**：
- 工作流可视化设计器（拖拽式节点编排）
- 节点类型（LLM/Tool/Condition/Loop/HITL/Sub-workflow）
- 工作流版本管理（创建/发布/回滚）
- 工作流执行引擎（同步/异步/定时触发）
- 工作流监控（执行次数/成功率/延迟/错误率）

**界面**：
- 工作流列表页：展示已创建的工作流（名称/描述/版本/执行次数）
- 工作流设计器：可视化拖拽式节点编排
- 工作流执行页：查看工作流执行历史与详情
- 工作流监控页：展示工作流监控指标

---

### 6.6 智能体管理

**功能**：
- 智能体定义（Agent/Goal/Capability/Knowledge/Memory/Planning/Action/Environment）
- 智能体生命周期管理（创建/启动/暂停/停止/删除）
- 智能体监控（执行次数/成功率/延迟/成本）
- 智能体日志（执行日志/错误日志/审计日志）
- 智能体版本管理

**界面**：
- 智能体列表页：展示已创建的智能体（名称/类型/状态/执行次数）
- 智能体配置页：配置智能体本体（Agent/Goal/Capability/Knowledge/Memory/Planning/Action/Environment）
- 智能体监控页：展示智能体监控指标
- 智能体日志页：查看智能体执行日志与错误日志

---

### 6.7 知识库管理

**功能**：
- 知识库创建/删除
- 文档上传（PDF/Word/TXT/Markdown）
- 文档切片与向量化（Embedding）
- 语义检索（RAG）
- 知识库版本管理
- 知识库权限管理

**界面**：
- 知识库列表页：展示已创建的知识库（名称/文档数量/向量数量/创建时间）
- 文档管理页：上传/删除/预览文档
- 检索测试页：测试语义检索效果
- 知识库配置页：配置切片大小/重叠/Embedding 模型/检索策略

---

### 6.8 治理配置

**功能**：
- HITL 人工介入规则（什么情况下需要人工审批）
- SLA 超时配置（工单超时时间）
- 权限过滤规则（数据范围/操作权限）
- 数据脱敏规则（敏感字段脱敏）
- 审计日志查询（按时间/用户/操作类型查询）

**界面**：
- HITL 规则页：配置 HITL 人工介入规则
- SLA 配置页：配置 SLA 超时时间
- 权限规则页：配置权限过滤规则
- 脱敏规则页：配置数据脱敏规则
- 审计日志页：查询审计日志

---

## 七、当前项目能力对标

### 7.1 已实现能力（可复用）

| 能力 | 当前状态 | 复用价值 |
|------|----------|----------|
| **MCP 协议统一收口** | ✅ 完整实现（OAuth 2.1 + JWKS + HMAC + 语义路由 + 审计） | 工作流节点调用智能体时，可通过 mcp-gateway 统一收口 |
| **CRM 业务系统连接** | ✅ 完整实现（10 个工具 + 权限过滤） | 其他业务系统可参考 CRM Server 实现模式 |
| **HITL 人工介入** | ✅ 完整实现（审批工单 + SLA 超时 + PostgreSQL 持久化） | 工作流中的 HITL 闸门可直接复用 |
| **前端工程骨架** | ✅ 完整实现（PC + H5 + OAuth 登录闭环 + 6 个业务页面原型） | 知识库管理/工作流编排/智能体管理页面可在现有骨架上扩展 |
| **三层测试体系** | ✅ 完整实现（单测 + 集成 + E2E） | 新功能开发可复用测试框架与 CI 编排脚本 |

### 7.2 缺失能力（需补齐）

| 能力 | 当前状态 | 补齐路径 |
|------|----------|----------|
| **知识库管理** | ❌ 完全缺失 | Stage A 修订 LLD-07 边界；Stage D 实现非结构化知识、文档、Embedding、RAG 与前端管理 |
| **工作流编排** | ❌ 仅有设计文档 | Stage A 重写 LLD-08；Stage D 实现基于 Ontology Action 的 Workflow、Hybrid、HITL 与补偿编排 |
| **智能体自主性** | ❌ 完全缺失 | Stage A 修订 LLD-09；Stage D 实现规划、意图识别、任务拆解与受治理 Action 调用 |
| **多智能体协同** | ❌ 完全缺失 | Stage A 完成影响分析；Stage D 在统一业务本体上实现 Supervisor/Worker 协作 |
| **模型配置** | ❌ 完全缺失 | Stage D 前冻结 LLD-11；Stage D 接入 LLM/Embedding、路由和降级 |
| **Skill 库管理** | ❌ 完全缺失 | Stage A 完成影响分析；Stage D 实现 Capability → Ontology Action 绑定，管理界面可后置 |
| **智能体管理** | ❌ 完全缺失 | Stage A 冻结治理边界；Stage D 实现 Agent 生命周期、运行监控和策略引用 |

---

## 八、分阶段实施路线图

### 8.1 当前有效路线

原“有限并行 8 周”路线及原 Phase 5-8 直接编码顺序作为历史规划保留，不再具有执行效力。当前唯一有效路线为 [双本体三层核心架构与项目阶段新基线](./dual-ontology-three-layer-architecture-baseline.md) 定义的 Stage A-F：

| 阶段 | 核心交付 | 通过门禁 |
|------|----------|----------|
| Stage A | LLD-15~18；修订 LLD-07/08/09；完成 LLD-10/12/14 影响分析；三域 Schema、ADR、验收场景 | 指定 LLD 评审通过，OpenAPI、JSON Schema、Cypher 与治理职责矩阵无冲突 |
| Stage B | MOD-15；MOD-17 策略存储、版本治理、血缘和审计底座；双存储与三域 Schema | 对象关系可创建、查询、版本化和隔离，治理元数据可追溯 |
| Stage C | MOD-16；Action Registry；MOD-17 动作授权接入；HITL；幂等、补偿与模拟适配器 | 每域完成读写、授权拒绝、高风险审批和副作用动作补偿链路 |
| Stage D | MOD-08/09；MOD-18 隔离模拟；规则、Hybrid、RAG、Checkpoint 与三层记忆 | Agent 和 Workflow 只能执行注册 Ontology Action，旁路调用被拒绝；模拟场景可隔离执行 |
| Stage E | CRM、ERP、OA 模拟端到端、跨域编排、模拟、审计与 E2E | 所有动作验证成功、失败、授权与审计；高风险动作验证审批；副作用写动作验证幂等与补偿 |
| Stage F | 满足真实 API、沙箱、凭据和映射前置条件后逐域接入 | 上层契约不变；凭据隔离、映射、同步重放、限流超时、写回补偿、审计和回滚演练通过 |

### 8.2 范围缩减原则

1. 可先延后管理后台高级可视化和批量编辑；
2. 可后置 MOD-10 多 Agent 并行协作，保留单 Agent 与确定性 Workflow；
3. 可后置 MOD-12 技能市场界面，保留 Capability → Ontology Action 注册契约；
4. 可减少 Stage E 跨域组合场景数量，但 CRM、ERP、OA 每域至少保留一条完整读写链路；
5. 不得裁剪 Stage A 契约、MOD-15/16/17、HITL、RBAC/ABAC、幂等、补偿、审计、模拟隔离和 Agent 禁止直连源系统约束。

### 8.3 历史路线归档

原 W1-W8、Phase 5-8 和 MOD-13 独立运行时计划仅用于追溯历史决策。MOD-13 状态为 **Deprecated / Migration-only**：禁止新增运行时能力，存量连接器定义与实例管理能力由 LLD-16 规定兼容迁移方案，迁移完成后不再保留独立部署单元。

---

## 九、技术选型与依赖

### 9.1 后端技术栈

| 技术 | 用途 | 版本 |
|------|------|------|
| **Python** | 后端服务开发语言（编排引擎/Agent 运行时/知识库） | 3.11+ |
| **FastAPI** | Web 框架 | 0.115+ |
| **LangGraph** | 工作流编排引擎 | 1.x |
| **LangChain** | LLM 集成框架 | 0.3+ |
| **Node.js/TypeScript** | API 网关/业务服务（复用现有 mcp-gateway 技术栈） | 20+ |
| **PostgreSQL** | 关系型数据库（本体模型/HITL/Checkpoint） | 15+ |
| **Redis** | 缓存（短期记忆/工作记忆/Nonce） | 7+ |
| **Qdrant** | 向量数据库（知识库，已确认） | 最新版 |
| **Celery** | 异步任务队列（工作流执行/文档向量化） | 5+ |
| **OpenAI/Anthropic** | LLM 提供商 | 最新版 |
| **HuggingFace Transformers** | 本地 LLM 推理 | 4.36+ |

### 9.2 前端技术栈

| 技术 | 用途 | 版本 |
|------|------|------|
| **Vue 3** | 前端框架 | 3.4+ |
| **Ant Design Vue** | PC 端 UI 组件库 | 4.x |
| **uni-app** | H5 端跨平台框架 | 3.0+ |
| **Pinia** | 状态管理 | 2.2+ |
| **vue-router** | 路由管理 | 4.x |
| **Vue Flow** | 工作流可视化设计器 | 1.x |
| **Monaco Editor** | 代码编辑器（Skill 定义/工作流定义） | 0.45+ |

### 9.3 基础设施

| 技术 | 用途 | 版本 |
|------|------|------|
| **Docker** | 容器化部署 | 24+ |
| **Docker Compose** | 本地一键部署（MVP 交付形态，已确认） | 2.x |
| **Kubernetes** | 容器编排（企业版后置） | 1.28+ |
| **Prometheus** | 监控指标采集 | 2.48+ |
| **Grafana** | 监控可视化 | 10+ |
| **MinIO/S3** | 对象存储（知识库原始文档） | 最新版 |

---

## 十、风险与缓解策略

### 10.1 技术风险

| 风险 | 影响 | 概率 | 缓解策略 |
|------|------|------|----------|
| **LLM 成本过高** | 高 | 中 | 模型路由策略（成本优先）、本地模型部署、缓存常见查询结果 |
| **向量数据库性能瓶颈** | 中 | 中 | 选择合适的向量数据库（Qdrant/Milvus 性能优于 Pinecone）、分片部署 |
| **工作流执行延迟** | 高 | 中 | 异步执行（Celery）、并行执行（LangGraph Send API）、Checkpoint 断点恢复 |
| **多智能体协同复杂性** | 高 | 高 | 选择成熟的协同框架（CrewAI/AutoGen）、限制协同智能体数量（< 10） |

### 10.2 业务风险

| 风险 | 影响 | 概率 | 缓解策略 |
|------|------|------|----------|
| **用户学习成本高** | 中 | 高 | 提供丰富的预置模板（Skill 库/工作流模板/智能体模板）、完善的文档与教程 |
| **业务系统集成难度** | 中 | 中 | 提供预置连接器（CRM/ERP/OA/数据库）、提供连接器开发 SDK |
| **数据安全与合规** | 高 | 低 | 已实现 OAuth 2.1 + HMAC 签名 + 审计日志 + 数据脱敏，持续加强 |

### 10.3 项目风险

| 风险 | 影响 | 概率 | 缓解策略 |
|------|------|------|----------|
| **1 人+AI 开发，双本体范围较大** | 高 | 高 | 严格执行 Stage A-F 门禁；超期时按第 8.2 节缩减非核心体验，不裁剪治理与执行安全能力 |
| **Python 编排 + Node.js 网关混合架构** | 中 | 中 | 接口契约先行（OpenAPI），CI 强制契约测试，通信协议限定 REST/JSON |
| **Qdrant 运维复杂度** | 中 | 中 | Docker Compose 集成 Qdrant，提供一键启停脚本，数据卷持久化 |
| **三者均衡导致资源分散** | 中 | 中 | 每 2 周一个里程碑，里程碑内聚焦单一锚点交付 |
| **文档与代码漂移** | 高 | 中 | ID 锚点 + CI 校验（见第十一章），LLD 评审通过后方可编码 |

---

## 十一、文档体系与执行保障

### 11.1 三级文档体系

本项目采用「规划 → 设计 → 执行」三级文档体系（详见 [documentation-system.md](./documentation-system.md)）：

- **Tier-1 规划层**：本文档 + [产品定位与商业模式](./product-positioning-and-business-model.md) + [文档体系规范](./documentation-system.md)
- **Tier-2 设计层**：LLD-07 ~ LLD-18，其中 LLD-13 为迁移态历史模块，当前新增核心为 LLD-15~18
- **Tier-3 执行层**：代码 + 测试 + 部署脚本，目录与 LLD 模块一一对应

### 11.2 防漂移机制

1. **ID 锚点**：每份 LLD 定义模块 ID（MOD-NN），代码目录通过 `MODULE.md` 声明同一 ID
2. **CI 校验**：`.github/workflows/doc-check.yml` 校验 MODULE.md 存在性、LLD 编号一致性、LLD 内容完整性
3. **场景化验收**：每份 LLD 第 6 节必须包含 Given/When/Then 验收场景，与 E2E 测试用例一一对应
4. **变更同步**：接口/数据模型/验收标准变更必须先改 LLD 再改代码

### 11.3 LLD 编写顺序（双本体新阶段基线）

原 8 周 MOD-07/MOD-09 直接编码顺序已被 [双本体三层核心架构](./dual-ontology-three-layer-architecture-baseline.md) 取代。后续按 Stage A → F 执行：

| 顺序 | LLD | 模块 ID | 阶段 | 门禁 |
|------|-----|---------|------|------|
| 1 | LLD-15 业务本体服务 | MOD-15 | Stage A | Schema、对象、关系、动作与规则契约冻结 |
| 2 | LLD-16 连接器运行时 | MOD-16 | Stage A | 连接器、映射、同步、写回、幂等和补偿契约冻结 |
| 3 | LLD-17 本体治理 | MOD-17 | Stage A | RBAC/ABAC、版本审批、血缘与审计契约冻结 |
| 4 | LLD-18 本体模拟 | MOD-18 | Stage A | 隔离场景、参数覆盖与结果对比契约冻结 |
| 5 | LLD-07 知识库修订 | MOD-07 | Stage A | 明确仅承载非结构化知识和 RAG |
| 6 | LLD-09 Agent 运行时修订 | MOD-09 | Stage A | Agent 通过 Ontology Action 执行，不直连业务系统 |
| 7 | LLD-08 工作流编排重写 | MOD-08 | Stage A | Workflow 与 Hybrid 模式基于 Ontology Action |
| 8 | LLD-11 模型配置 | MOD-11 | Stage D 前 | LLM/Embedding 供应商、路由与降级契约冻结 |
| 9 | LLD-10/12/14 | MOD-10/12/14 | Stage D | 多 Agent、Skill 与 Agent 治理完成双本体映射 |

Stage B 完成语义层平台；Stage C 完成动力层平台；Stage D 完成动态层平台；Stage E 使用模拟适配器完成 CRM、ERP、OA 三域完整验证；Stage F 在具备真实 API、沙箱和凭据后接入真实数据源。

---

## 附录 A：术语表

| 术语 | 定义 |
|------|------|
| **Business Ontology** | 业务本体，形式化定义业务对象、属性、关系、动作、规则、权限和数据来源 |
| **Semantic Layer** | 语义层，定义 ObjectType、PropertyType、RelationType 与业务对象实例 |
| **Kinetic Layer** | 动力层，将 Ontology Action 绑定到源系统连接器，负责同步、写回、幂等、事务与补偿 |
| **Dynamic Layer** | 动态层，在业务本体上运行规则、Workflow、Agent、模拟、HITL 和审计 |
| **Ontology Action** | 业务本体中已注册、带输入输出 Schema、权限、风险和补偿定义的受治理动作 |
| **Agent Ontology** | Agent 本体，定义 Agent、Goal、Capability、Knowledge、Memory、Planning、Action、Environment |
| **MCP** | Model Context Protocol，Agent 发现和调用工具的统一协议；不替代连接器的源协议、凭据、映射与事务职责 |
| **HITL** | Human-in-the-Loop，人在回路，人工介入机制 |
| **RAG** | Retrieval-Augmented Generation，检索增强生成，通过检索相关知识增强 LLM 生成能力 |
| **ReAct** | Reasoning and Acting，推理与行动，一种智能体规划算法 |
| **Saga** | 一种分布式事务模式，通过补偿机制保证最终一致性 |
| **WDL** | Workflow Definition Language，工作流定义语言 |
| **LLM** | Large Language Model，大语言模型 |
| **Embedding** | 向量化，将文本转换为向量表示 |
| **OIDC** | OpenID Connect，基于 OAuth 2.0 的身份认证协议 |
| **PKCE** | Proof Key for Code Exchange，授权码交换证明密钥，防止授权码截获攻击 |
| **JWKS** | JSON Web Key Set，JSON Web 密钥集，用于 JWT 签名验证 |
| **HMAC** | Hash-based Message Authentication Code，基于哈希的消息认证码 |
| **SLA** | Service Level Agreement，服务级别协议 |
| **WORM** | Write Once Read Many，一次写入多次读取，用于审计日志归档 |

---

## 附录 B：参考资料

1. [企业级智能体架构 v1.1](./enterprise-agent-architecture-v1.1.md)
2. [LLD-01 MCP 网关详细设计](./lld-01-mcp-gateway.md)
3. [LLD-05 编排层详细设计](./lld-05-orchestration-layer.md)
4. [LLD-06 HITL 服务详细设计](./lld-06-hitl-service.md)
5. [LangGraph 官方文档](https://langchain-ai.github.io/langgraph/)
6. [MCP 协议规范](https://modelcontextprotocol.io/)
7. [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)
8. [CrewAI 官方文档](https://docs.crewai.com/)
9. [AutoGen 官方文档](https://microsoft.github.io/autogen/)

---

## 附录 C：LLD 拆分清单

> 本清单是 Tier-1 → Tier-2 的拆分索引，每份 LLD 的编写模板见 [documentation-system.md](./documentation-system.md) 第 2.2 节。阶段与优先级以第 8 章和第 11.3 节为准，旧 W1-W8 标签全部失效。

### C.1 LLD-07 知识库（MOD-07，Stage A 修订，Stage D 实现）

- **代码目录**：`services/knowledge-base/`
- **核心内容**：文档解析（PDF/Word/TXT/MD）、切片策略、Embedding 管线、Qdrant 集成、RAG 检索（similarity/MMR/hybrid）、知识库 CRUD API、版本管理
- **依赖**：无前置 LLD；外部依赖 Qdrant + Embedding 模型
- **前端页面**：`KnowledgeView.vue`（列表/上传/检索测试/配置）
- **验收场景**：上传 PDF→检索命中、边界（超大文档/空文档）、异常（Qdrant 宕机降级）

### C.2 LLD-08 工作流编排（MOD-08，Stage A 重写，Stage D 实现）

- **代码目录**：`services/workflow-orchestrator/`
- **核心内容**：LangGraph StateGraph 集成、WDL 工作流定义语言、节点类型（LLM/Tool/Condition/Loop/HITL/Sub-workflow）、执行引擎（同步/异步/定时）、Saga 补偿链、Checkpoint 持久化（PostgreSQL+Redis）
- **依赖**：LLD-09（Agent 运行时）、hitl-service（已有）
- **前端页面**：`WorkflowView.vue`（Vue Flow 拖拽设计器 + 执行历史 + 监控）
- **验收场景**：创建 3 节点工作流→执行成功、HITL 闸门审批流、失败补偿回滚
- **迁移说明**：吸收现有 `lld-05-orchestration-layer.md` 内容并按新模板重写

### C.3 LLD-09 Agent 运行时（MOD-09，Stage A 修订，Stage D 实现）

- **代码目录**：`services/agent-runtime/`
- **核心内容**：Agent 本体加载与实例化、规划算法（ReAct/Plan-and-Execute）、意图识别、任务拆解、Function Calling 驱动、记忆管理（短期/长期/工作记忆）、执行状态机
- **依赖**：LLD-11（模型配置）；通过 MCP 网关调用工具（已有）
- **验收场景**：单 Agent ReAct 完成任务、意图识别→工具调用链路、Checkpoint 断点恢复

### C.4 LLD-10 多 Agent 协作（MOD-10，Stage D，可后置）

- **代码目录**：`services/agent-runtime/collaboration/`
- **核心内容**：Supervisor/Worker 角色、消息传递（发布/订阅）、注册/发现、协同工作流
- **依赖**：LLD-09
- **验收场景**：Supervisor 分派任务给 2 个 Worker 并汇总结果
- **砍除预案**：MVP 阶段单 Agent 串行执行，本模块整体后置

### C.5 LLD-11 模型配置（MOD-11，Stage D 前冻结）

- **代码目录**：`services/model-config/`
- **核心内容**：LLM 接入（OpenAI/Anthropic/本地）、参数配置、路由策略（成本/性能/延迟）、降级策略、调用监控
- **依赖**：无
- **前端页面**：`ModelConfigView.vue`
- **验收场景**：主模型失败自动切换备用模型、路由策略按成本选模型

### C.6 LLD-12 Capability 绑定库（MOD-12，Stage D，可后置界面）

- **代码目录**：`services/skill-library/`
- **核心内容**：Capability 输入输出契约、Capability → ActionType 绑定、绑定版本、PolicyBindingRef 和复用模板；不承载执行逻辑或独立权限策略
- **依赖**：LLD-09、LLD-15、LLD-17
- **前端页面**：`SkillLibraryView.vue`
- **范围缩减预案**：可后置独立管理界面，但绑定必须保存于 MOD-12 并经校验；禁止硬编码到 Agent 配置或绕过 MOD-17

### C.7 LLD-13 业务连接器（MOD-13，Deprecated / Migration-only）

- **生命周期**：禁止新增运行时能力；存量契约和实例管理能力迁移至 MOD-16
- **历史代码目录**：`services/connectors/`，仅允许兼容壳层或迁移适配，迁移完成后删除独立部署单元
- **迁移目标**：LLD-16 / `services/source-connector-runtime/`
- **验收场景**：存量配置可迁移，兼容期调用转发至 MOD-16，迁移后无 Agent 直接依赖 MOD-13

### C.8 LLD-14 治理配置（MOD-14，Stage A 影响分析，Stage D 配置能力）

- **代码目录**：`services/governance/`
- **核心内容**：Agent/Workflow 治理配置、HITL 触发条件、SLA 和 PolicyBindingRef；不实现本体权限策略引擎或审计事实存储
- **依赖**：MOD-17、hitl-service、MOD-08、MOD-09
- **职责边界**：策略定义、版本和求值归 MOD-17；动作执行强制点归 MOD-16/MCP Gateway；工单生命周期归 hitl-service；MOD-14 只管理 Agent/Workflow 治理配置及策略引用
- **范围缩减**：可后置配置界面，不得绕过 MOD-17 硬编码权限或审批策略

---

**文档结束**

本文档定义项目级目标与规划；[双本体三层核心架构基线](./dual-ontology-three-layer-architecture-baseline.md) 是架构与阶段执行的最高优先级事实源，[documentation-system.md](./documentation-system.md) 定义文档治理规则，各 LLD 定义模块契约，历史文档仅用于追溯。发生冲突时按“架构基线 → 本文档当前有效章节 → 文档体系规范 → 已批准 LLD → 历史文档”的顺序裁决。
