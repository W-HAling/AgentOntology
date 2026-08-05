# SEC-19-04 数据模型与追溯图

> **稳定 ID**：SEC-19-04  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)

## 1. 通用约束

- 所有 ID 全局唯一并匹配各自前缀；版本使用 SemVer；时间使用带时区 RFC 3339。
- 所有路径为仓库根目录相对 POSIX 路径，不允许绝对路径、反斜杠或 `..` 越界。
- 摘要格式为 `sha256:<64 个小写十六进制字符>`。
- 普通文件节点对仓库文件原始字节计算 SHA-256；指向执行台账的 `design_task|implementation_task` 节点对台账中同 ID 的任务对象执行“UTF-8、JSON、键名排序、紧凑分隔符、保留数组顺序”的规范化序列化后计算 SHA-256，禁止对包含摘要引用的整份台账做自引用摘要。
- YAML/JSON 顶层必须声明 `$schema`，未知字段默认拒绝。
- 实体正文由对应治理 Schema 唯一定义；本节定义跨实体语义和关系约束。

## 2. 核心实体

| 实体 | ID | 必备语义 |
|---|---|---|
| ResearchEvidence | `EVD-{DOMAIN}-{NNN}` | path、media_type、captured_at、source、access_level、digest、classification |
| UserRequirement | `UR-{DOMAIN}-{NNN}` | title、statement、source_refs、verbatim_quotes、actors、problem、desired_outcome、priority、constraints、acceptance_signals、confidence、status、confirmation fields、supersedes |
| PlanningRequirement | `PR-{DOMAIN}-{NNN}` | UR 引用、scope、priority、success_metrics、target_stage、constraints、risks、PO confirmation |
| DesignRequirement | `DR-MOD{NN}-{NNN}` | PR 引用、LLD section、ADR、contract、scenario、NFR |
| DesignTask | `DTASK-{STAGE}-{NNN}` | PR 引用、allowed_artifact_kinds、files、commands、expected_results、evidence、status |
| ExecutionTask | `TASK-{STAGE}-{NNN}` | design refs、批准 LLD 路径/版本/正文摘要/原子摘要、文件、前置条件、命令、预期结果、验证、证据、状态、阻塞与完成时间 |
| TraceNode | 任一稳定 ID | kind、version、status、path、digest |
| TraceEdge | `EDGE-{NNNN}` | relation、source、target、source_version、target_version |
| ExecutionLedger | `LEDGER-STAGE-A` | version、stage、updated_at、design_tasks、implementation_tasks |
| StageTraceabilityView | `VIEW-STAGE-A` | 节点/边 ID 选择器、发布评审；不得复制规范正文 |
| AtomicDesignItem | `ATOM-19-{NNN}` | normative_id、path、section、kind、digest、owner |

## 3. 状态字段

### 3.1 UR

`discovered -> validated -> confirmed -> superseded|rejected`

只有 `confirmed` 可被活动 PR 引用。`confirmation_evidence_ref`、`confirmed_subject`、`confirmed_at` 必须共同存在且经 GitHubApprovalVerifier 验证；字段齐全不等于批准有效。

### 3.2 DTASK/TASK

`planned|ready|in_progress|blocked|verified|accepted`

- `verified` 必须有存在且摘要匹配的验证证据。
- `accepted` 必须有独立且有效的批准证据。
- `blocked` 必须有非空阻塞原因。
- `verified|accepted` 必须有 `completed_at`。

## 4. 追溯图

### 4.1 节点 kind

`evidence|user_requirement|planning_requirement|design_requirement|lld|adr|contract|scenario|design_task|implementation_task|artifact|test|verification|release_gate`

### 4.2 允许关系

| source kind | relation | target kind |
|---|---|---|
| evidence | supports | user_requirement |
| user_requirement | realized_by | planning_requirement |
| planning_requirement | scheduled_by | design_task |
| planning_requirement | refined_by | design_requirement |
| design_task | produces | design_requirement/lld/adr/contract |
| design_requirement | specified_by | lld/adr/contract |
| design_requirement | accepted_by | scenario |
| lld | approved_by | verification |
| lld | authorizes | implementation_task |
| implementation_task | produces | artifact |
| artifact | verified_by | test |
| test | evidenced_by | verification |
| verification | gates | release_gate |

其他 kind/relation 组合一律非法。`source_version` 和 `target_version` 必须等于端点版本；普通文件节点摘要必须匹配当前文件，任务节点摘要必须匹配执行台账中的同 ID 规范化任务对象。

## 5. 图不变量

1. 节点 ID、边 ID 均唯一，所有端点存在。
2. 设计生产链与实现交付链分别可达，跨链只能通过 `lld authorizes implementation_task` 连接。
3. 除 `supersedes` 历史引用外，活动图必须为有向无环图。
4. `superseded|rejected` UR 不得到达活动任务。
5. 活动 TASK 必须有且仅有一个有效批准 LLD 基线。
6. 任何 artifact/test 都必须可向上追到 UR 和批准 LLD。
7. 需求追溯事实源顶层只保存 `nodes` 与 `edges`；实体正文留在各自事实源，禁止在追溯文件中复制 evidence、UR、PR 或 DR 正文。
8. Stage A 发布视图仅引用已存在节点/边；`planned` TASK 元数据不得伪装为活动节点。
9. 执行台账中的每个 DTASK/TASK 都必须有对应追溯节点，并至少通过一条合法边接入其上游 PR 或设计基线。

## 6. 所有权与保留

| 数据 | 所有者 | 保留 |
|---|---|---|
| 证据元数据与脱敏摘录 | Product Owner/研究负责人 | 项目存续期加组织规定审计期 |
| UR/PR/DR/任务/追溯 | 对应文档 CODEOWNER | Git 历史永久保留 |
| CI 诊断与验证证据 | 仓库维护者 | 至少覆盖当前及上一发布基线 |
| GitHub 审批引用 | 仓库维护者 | 与被批准版本同寿命 |

删除通过新版本、supersedes/rejected 状态和 Git 历史表达，不允许覆写历史来抹除审计链。
