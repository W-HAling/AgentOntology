# SEC-19-02 组件与目标结构

> **稳定 ID**：SEC-19-02  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)

## 1. 组件边界

| 组件 ID | 组件 | 职责 | 禁止行为 |
|---|---|---|---|
| CMP-19-01 | EvidenceValidator | 证据元数据、文件摘要、时间顺序、原话分类校验 | 获取或补造缺失原文 |
| CMP-19-02 | RequirementValidator | UR/PR/DR 状态、字段、准入与引用校验 | 把 `validated` 自动提升为 `confirmed` |
| CMP-19-03 | TaskLedgerValidator | DTASK/TASK、台账状态、命令、证据、时间校验 | 从 Markdown 复选框推导活动状态 |
| CMP-19-04 | TraceGraphValidator | 节点、边、版本、摘要、链路、环和双向查询校验 | 自动修复断链 |
| CMP-19-05 | LldValidator | DoR、3/2/2 场景、链接、未决项、原子清单校验 | 把格式通过解释为人工批准 |
| CMP-19-06 | ContractValidator | Schema/OpenAPI/跨引用/职责唯一性校验 | 使用丢失来源 URI 的裸字典解析外部 `$ref` |
| CMP-19-07 | GitHubApprovalVerifier | PR、review、User、CODEOWNERS、权限、摘要验证 | 信任文档自报身份或本地 `Accepted` |
| CMP-19-08 | GateOrchestrator | 顺序聚合检查、测试与退出码 | 吞掉子检查失败或降级放行 |
| CMP-19-09 | DiagnosticReporter | 稳定错误码、人类可读和机器可读输出 | 输出令牌、敏感原文或密钥 |

## 2. 目标目录

```text
AgentOntology/
├── lld-19-document-governance.md
├── lld-19/sections/
│   ├── 01-boundaries.md
│   ├── 02-components.md
│   ├── 03-interfaces.md
│   ├── 04-data-model.md
│   ├── 05-state-and-reliability.md
│   ├── 06-security.md
│   ├── 07-non-functional.md
│   ├── 08-deployment-migration.md
│   ├── 09-scenarios.md
│   └── 10-implementation-map.md
├── contracts/schemas/governance/
│   ├── user-requirement.v1.schema.json
│   ├── planning-requirement.v1.schema.json
│   ├── design-requirement.v1.schema.json
│   ├── design-task.v1.schema.json
│   ├── execution-task.v1.schema.json
│   ├── research-evidence.v1.schema.json
│   ├── requirements-traceability.v1.schema.json
│   ├── execution-ledger.v1.schema.json
│   └── stage-traceability-view.v1.schema.json
├── contracts/traceability/
│   ├── requirements-traceability.v1.yaml
│   └── stage-a-traceability.v1.yaml
├── contracts/execution/stage-a-execution-ledger.v1.yaml
├── scripts/
│   ├── check_doc_consistency.py
│   ├── check_contracts.py
│   └── check_stage_a_gate.py
└── tests/stage_a/
    ├── fixtures/
    ├── test_doc_consistency.py
    ├── test_contracts.py
    └── test_stage_a_gate.py
```

这是目标结构而非存在性声明。Stage A 的渐进门禁必须依据条目目标阶段判断文件是否应存在。

## 3. 处理流水线

1. 发现受治理文件并按稳定路径排序。
2. 解析 UTF-8 Markdown、YAML、JSON 和 OpenAPI。
3. 使用文件 URI 作为基准解析相对链接与 `$ref`。
4. 执行 Schema 和文档结构检查。
5. 构建不可变追溯图视图并执行语义检查。
6. 按运行模式验证 GitHub 审批。
7. 聚合并稳定排序诊断。
8. 输出文本或 JSON 报告并返回退出码。

## 4. 内部依赖规则

- 组件只读取仓库工作树与显式 GitHub API 响应，不隐式访问任意网络地址。
- 解析器不得修改源文件；所有校验保持确定性和可重复运行。
- GitHubApprovalVerifier 失败不得由其他组件覆盖。
- GateOrchestrator 按“输入可解析 -> 文档 -> 契约 -> 测试”的顺序执行；前序不可解析时返回输入错误，不执行可能产生误导的后续语义检查。
