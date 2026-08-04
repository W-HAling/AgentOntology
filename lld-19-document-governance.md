# LLD-19 文档治理与追溯门禁低层设计

> **模块 ID**：MOD-19  
> **文档性质**：设计层（Tier-2）  
> **状态**：Proposed  
> **批准状态**：未批准；本文档不声明、推断或替代任何人工审批  
> **上游文档**：[Stage A 双本体契约冻结实施计划](./stage-a-dual-ontology-contract-freeze-implementation-plan.md)、[三级文档体系规范](./documentation-system.md)  
> **对应代码**：`scripts/check_doc_consistency.py`、`scripts/check_contracts.py`、`scripts/check_stage_a_gate.py`  
> **目标阶段**：Stage A  
> **版本**：0.1.0  
> **日期**：2026-08-03

---

## 1. 设计定位

LLD-19 是需求证据、规划、设计、执行、追溯、执行台账与 Stage A 文档门禁的设计唯一真实依据。它设计的是仓库治理工具，不是业务运行时服务，不创建 `services/document-governance/`，也不参与在线请求链路。

本文档采用“主文档索引 + 十个规范性 section”的结构。每项规范性事实只在一个 section 定义；主文档仅提供边界摘要、稳定索引、决策摘要和评审状态，避免主文档与 section 重复定义。

## 2. Definition of Ready 结论

| DoR 项 | 唯一定义位置 | 当前结论 |
|---|---|---|
| 职责、非职责、上下游、事实源 | [SEC-19-01](./lld-19/sections/01-boundaries.md) | 已设计 |
| 组件、目录、文件结构 | [SEC-19-02](./lld-19/sections/02-components.md) | 已设计 |
| CLI、内部接口、鉴权、版本、错误与退出码 | [SEC-19-03](./lld-19/sections/03-interfaces.md) | 已设计 |
| 实体、字段约束、关系、所有权、保留 | [SEC-19-04](./lld-19/sections/04-data-model.md) | 已设计 |
| 状态机、事务、并发、幂等、重试、补偿、回放 | [SEC-19-05](./lld-19/sections/05-state-and-reliability.md) | 已设计 |
| 认证、授权、隔离、脱敏、密钥、审计、威胁边界 | [SEC-19-06](./lld-19/sections/06-security.md) | 已设计 |
| 性能、容量、可用性、可观测性、降级、恢复、预算 | [SEC-19-07](./lld-19/sections/07-non-functional.md) | 已设计 |
| 配置、依赖、CI 拓扑、迁移、回滚 | [SEC-19-08](./lld-19/sections/08-deployment-migration.md) | 已设计 |
| 3 核心、2 边界、2 异常场景 | [SEC-19-09](./lld-19/sections/09-scenarios.md) | 已设计 |
| 实施拆分、文件、命令、预期输出、原子清单 | [SEC-19-10](./lld-19/sections/10-implementation-map.md) | 已设计 |
| 未解决问题 | 本节 | 空 |

## 3. 规范性 section 索引

1. [SEC-19-01 边界与治理模型](./lld-19/sections/01-boundaries.md)
2. [SEC-19-02 组件与目标结构](./lld-19/sections/02-components.md)
3. [SEC-19-03 接口契约](./lld-19/sections/03-interfaces.md)
4. [SEC-19-04 数据模型与追溯图](./lld-19/sections/04-data-model.md)
5. [SEC-19-05 状态机与可靠性](./lld-19/sections/05-state-and-reliability.md)
6. [SEC-19-06 安全与可信审批边界](./lld-19/sections/06-security.md)
7. [SEC-19-07 非功能设计](./lld-19/sections/07-non-functional.md)
8. [SEC-19-08 CI、部署、迁移与回滚](./lld-19/sections/08-deployment-migration.md)
9. [SEC-19-09 场景化验收](./lld-19/sections/09-scenarios.md)
10. [SEC-19-10 实施映射与原子设计清单](./lld-19/sections/10-implementation-map.md)

场景化验收标准统一采用 **Given / When / Then**，接口契约与数据模型的规范正文分别由 SEC-19-03 和 SEC-19-04 唯一定义。

## 4. 决策摘要

| 决策 ID | 决策 |
|---|---|
| DEC-19-001 | 仓库内版本化证据与可回溯 GitHub PR 审批构成 Stage A 信任根；本地自报不构成批准。 |
| DEC-19-002 | 需求、规划、设计生产、实现交付采用两条无歧义追溯链。 |
| DEC-19-003 | Stage A 执行台账是活动状态唯一事实源，Markdown 复选框仅可作为展示。 |
| DEC-19-004 | LLD 批准是生成活动实现任务的前置条件；批准必须绑定正文摘要与原子清单摘要。 |
| DEC-19-005 | 检查器默认拒绝：缺失、不可解析、身份不可信、摘要不匹配或审批上下文不可回溯均失败。 |
| DEC-19-006 | 主文档与 section 使用相对链接；原子清单对所有规范组成物计算 SHA-256。 |
| DEC-19-007 | 追溯事实源仅保存节点与边；实体正文不在矩阵中复制。 |
| DEC-19-008 | DR/LLD 正文不自报批准，审批仅由发布视图中的独立评审事实派生。 |
| DEC-19-009 | `planned` 只表示未启动计划；已产生真实工作产物的任务必须按证据进入活动状态。 |

## 5. 审批与变更状态

本文档当前为 `Proposed`，没有 `APPROVED` review、合格 CODEOWNER 审查者、当前 head SHA、review node ID 或正文摘要绑定证据，因此不得标记为 `Accepted`，不得作为“已批准 LLD”签发实现任务。

后续进入 `In Review` 或 `Accepted` 时，只能由检查器根据 GitHub API、仓库 ID、PR 编号、当前 head SHA、review node ID、CODEOWNERS、审查时权限及文档摘要计算结果确定。任何正文、section、原子清单、CODEOWNERS 或权限变化都会使既有批准失效并回到 `In Review`。

## 6. 未解决问题

无。外部审批证据尚未产生属于生命周期状态，不是设计待决项。

## 7. 变更记录

| 版本 | 日期 | 状态 | 说明 |
|---|---|---|---|
| 0.1.0 | 2026-08-03 | Proposed | 建立主文档和十个规范性 section；未声明人工批准。 |
