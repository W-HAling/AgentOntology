# SEC-19-01 边界与治理模型

> **稳定 ID**：SEC-19-01  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)

## 1. 职责

MOD-19 负责定义并校验以下治理闭环：

1. 登记研究证据，区分原始证据、研究者解释、已确认需求和待验证假设。
2. 管理 `UR-*`、`PR-*`、`DR-*`、`DTASK-*`、`TASK-*` 的结构、引用与准入条件。
3. 维护全生命周期追溯图和 Stage A 发布视图。
4. 维护 Stage A 活动执行台账并校验状态与证据一致性。
5. 校验 LLD DoR、场景、相对链接、规范性事实唯一性和原子设计清单。
6. 校验 JSON Schema、OpenAPI、治理职责和跨契约引用。
7. 在可信 GitHub 上下文中验证人工审批证据；在本地上下文拒绝自报 `Accepted`。
8. 聚合文档、契约和测试检查，向 CI 返回稳定退出码。

## 2. 非职责

- 不替代 Product Owner、CODEOWNER 或独立审查者作出业务或设计批准。
- 不创建、修改或伪造 GitHub review、身份、权限、提交或审批证据。
- 不把外部 URI、Markdown 声明或提交作者自述当作信任根。
- 不承担业务本体、Action、HITL、策略求值或运行时审计事实存储。
- 不在 Stage A 强制要求 Stage B-D 尚未进入实施阶段的服务目录和测试文件存在。
- 不识别现实世界中的关联账号；该风险由组织 IAM、强制 SSO 和成员治理控制。

## 3. 上下游

| 方向 | 对象 | 契约 |
|---|---|---|
| 上游 | 用户研究证据 | 版本化文件、媒体类型、采集时间、来源、访问级别、SHA-256 |
| 上游 | Tier-1 规划 | PR、阶段、成功指标、约束、风险 |
| 上游 | GitHub | repository、PR、head SHA、review、User node ID、CODEOWNERS、权限 |
| 下游 | LLD/ADR/契约 | DR 与 DTASK 产物，规范性事实唯一位置 |
| 下游 | 实现任务 | 仅从有效批准 LLD 生成的 TASK |
| 下游 | CI/release gate | 结构化诊断、退出码、验证证据 |

## 4. 唯一事实源

| 治理事实 | 唯一事实源 |
|---|---|
| 原始研究材料 | 仓库内版本化证据文件；敏感材料仅允许受治理归档签名清单 |
| 用户需求 | `user-research-and-requirements.md` 中结构化 UR 条目 |
| 规划、设计、任务实体 | 对应治理 Schema 校验的结构化文档 |
| 活动任务状态 | `contracts/execution/stage-a-execution-ledger.v1.yaml` |
| 全量追溯关系 | `contracts/traceability/requirements-traceability.v1.yaml` |
| Stage A 发布选择 | `contracts/traceability/stage-a-traceability.v1.yaml` |
| LLD 设计 | LLD 主文档与其原子清单列出的规范性 section |
| 审批事实 | GitHub API 可回溯的 `APPROVED` review 及绑定摘要 |

Markdown 复选框、交付报告和仓库外任务列表不得声明为活动状态源。

## 5. 治理角色与 RACI

| 活动 | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| 证据登记 | 研究者/实施者 | Product Owner | 安全负责人 | 团队 |
| UR 确认 | Product Owner 对应 GitHub User | Product Owner | 研究者 | 团队 |
| LLD 编写 | 设计实施者 | 模块负责人 | 架构/安全审查者 | Product Owner |
| 独立批准 | 合格 CODEOWNER User | 路径责任人 | 实施者 | 团队 |
| 门禁执行 | CI GitHub App/Action | 仓库维护者 | 安全负责人 | PR 参与者 |
| 台账更新 | 任务执行者 | 阶段负责人 | 审查者 | 团队 |

同一主体不得同时作为同一批准对象的作者/实施者与独立审查者。

## 6. 两条治理链

- 设计生产链：`evidence -> UR -> PR -> DTASK -> DR/LLD/ADR/contract -> SCN -> design approval`。
- 实现交付链：`approved LLD -> TASK -> code/config/migration -> test -> verification evidence -> release gate`。

MOD-15 至 MOD-18 在 Stage A 仅登记未来 TASK ID、目标阶段和计划路径；除 LLD-19 门禁工具实现外，不得创建活动实现任务。
