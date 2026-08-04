# SEC-19-07 非功能设计

> **稳定 ID**：SEC-19-07  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)

## 1. 性能与容量

| 指标 ID | 指标 | 目标 |
|---|---|---|
| NFR-19-001 | 500 个治理文件本地完整检查 | P95 <= 30s |
| NFR-19-002 | 10,000 节点、20,000 边图校验 | P95 <= 10s，峰值内存 <= 512MiB |
| NFR-19-003 | 单文件解析 | 50MiB 上限，超限返回输入错误 |
| NFR-19-004 | GitHub 远程审批验证 | 正常 API 条件下 P95 <= 120s |
| NFR-19-005 | 诊断数量 | 默认最多 1,000 条，截断时输出总数和截断标记 |

图校验使用邻接表和拓扑排序，时间复杂度目标为 `O(V+E)`；文件摘要流式计算，避免一次性复制大文件。

## 2. 可用性与确定性

- 本地文件检查不依赖网络，目标可重复执行成功率 99.9%。
- GitHub API 不可访问、权限不足或限流耗尽时门禁失败，不降级为信任本地声明。
- 相同 commit、工具版本、模式与远程事实必须产生相同退出码和稳定排序诊断。

## 3. 可观测性

### 3.1 结构化日志

字段：`run_id`、`repository_id`、`commit_sha`、`mode`、`phase`、`duration_ms`、`files_checked`、`nodes_checked`、`edges_checked`、`diagnostic_count`、`exit_code`。日志不得包含秘密或敏感原文。

### 3.2 指标

- `governance_check_duration_seconds{phase}`
- `governance_diagnostics_total{code}`
- `governance_files_checked_total{kind}`
- `governance_github_requests_total{status}`
- `governance_gate_runs_total{result}`

### 3.3 CI 摘要

GitHub Step Summary 展示结果、耗时、前 50 条诊断和修复路径；完整 JSON 作为构建产物保存，不在控制台展开敏感上下文。

## 4. 降级策略

- `text` 输出失败时仍尝试最小 stderr 诊断并返回原分类退出码。
- JSON 报告写入失败不得把门禁结果改为成功。
- 缓存损坏时丢弃缓存并重算；不得使用无法校验来源的缓存结果。
- GitHub API 失败只允许受限重试，不允许跳过审批检查。

## 5. 恢复与资源预算

- 检查器无持久化数据库；恢复单位是一次无状态重跑。
- CI 运行预算：2 vCPU、2GiB RAM、10 分钟；超过预算视为失败。
- 临时文件写入系统临时目录，进程退出时清理；报告产物最大 20MiB。
- 依赖安装必须固定精确版本并使用 lock/requirements 文件，防止不可重复构建。

## 6. 兼容性

- Python 3.12 是 Stage A CI 基线。
- 输入编码固定 UTF-8；行尾兼容 LF/CRLF，摘要规范化规则必须由原子清单版本固定。
- Windows 本地与 Ubuntu CI 对同一仓库内容必须得到一致语义结果。
