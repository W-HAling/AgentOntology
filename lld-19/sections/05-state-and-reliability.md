# SEC-19-05 状态机与可靠性

> **稳定 ID**：SEC-19-05  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)

## 1. 需求状态机 SM-19-UR

| 当前 | 事件 | 守卫 | 下一状态 |
|---|---|---|---|
| discovered | evidence_validated | 来源存在、摘要匹配、分类合法 | validated |
| validated | approval_verified | 合格 User/CODEOWNER、maintain/admin、当前摘要与 head SHA 匹配 | confirmed |
| confirmed | replacement_approved | `supersedes` 指向新 confirmed UR | superseded |
| discovered/validated/confirmed | rejection_approved | 有可回溯拒绝决策 | rejected |

禁止跳过 `validated`、自行写入 `confirmed`、从终态恢复。审批失效时，发布视图中的有效状态回退为 `validated`；源文件中的历史声明必须通过变更提交修正，门禁在修正前失败。

## 2. LLD 评审状态机 SM-19-LLD

`Draft -> Proposed -> In Review -> Accepted -> Superseded`

- `Draft -> Proposed`：DoR、3/2/2 场景、链接和原子清单完整，未解决问题为空。
- `Proposed -> In Review`：存在目标 PR，但尚无有效独立批准。
- `In Review -> Accepted`：GitHubApprovalVerifier 通过全部身份、资格、head SHA、摘要与独立性守卫。
- `Accepted -> In Review`：正文、section、原子摘要、CODEOWNERS、权限或 head SHA 变化。
- `Accepted -> Superseded`：后继版本获得有效批准。

本地模式永远不得产生 `Accepted` 结论。

## 3. 任务状态机 SM-19-TASK

| 当前 | 可达状态 | 必需证据 |
|---|---|---|
| planned | ready、blocked | 仅登记未来工作；命令、验证命令、证据、开始/完成时间为空；计划路径允许尚不存在 |
| ready | in_progress、blocked | 已批准设计基线；文件与命令完整 |
| in_progress | blocked、verified | 已记录开始时间与真实工作产物证据；verified 需验证证据与摘要 |
| blocked | ready、in_progress | 阻塞解除记录 |
| verified | accepted、in_progress | 独立批准；证据变化时回到 in_progress |
| accepted | in_progress | 产物或批准失效时重新验证 |

## 4. 检查运行状态机 SM-19-GATE

`INITIALIZING -> PARSING -> VALIDATING_DOCS -> VALIDATING_CONTRACTS -> VERIFYING_APPROVALS -> RUNNING_TESTS -> PASSED|FAILED`

- 解析失败直接进入 `FAILED(exit=2)`。
- 内容、契约、审批或测试失败进入 `FAILED(exit=1)`。
- 仅所有适用步骤成功进入 `PASSED(exit=0)`。

## 5. 事务边界

检查器是只读进程，不修改治理文件。单次运行以启动时解析出的仓库快照、commit SHA 和 PR head SHA 为一致性边界；验证期间检测到 head SHA 变化必须失败，禁止混用两个版本的证据。

台账变更通过单个 Git 提交或 PR 变更集原子提交。任务状态、证据路径与证据摘要必须在同一变更集中更新，否则门禁拒绝中间状态。

## 6. 并发控制

- 同一 PR 的多个 CI 运行使用 GitHub Actions concurrency group，新的 head SHA 取消旧运行。
- 检查器不共享可变全局状态；缓存键必须包含仓库 ID、commit SHA、文件路径与摘要。
- GitHub API 分页响应在单次运行内固定；发现 head SHA 漂移立即终止。

## 7. 幂等、重试、超时

- 本地文件校验纯函数化，同一输入产生同一排序结果。
- GitHub GET 请求仅对 429、502、503、504 重试 3 次，退避 1s/2s/4s，并遵守 `Retry-After`。
- 401、403、404 和语义不一致不重试。
- 单请求连接超时 5s、读取超时 15s；整次远程验证预算 120s。

## 8. 补偿、回放与恢复

检查器无外部写副作用，因此不执行业务补偿。失败恢复通过修正文档或恢复可信 GitHub 上下文后重新运行。每次运行输出包含 commit SHA、模式、工具版本和诊断，可在同一 commit 上回放。

若已发布批准因摘要或资格变化失效，补偿动作是阻断后续实现/发布、将有效评审状态回到 `In Review` 并重新取得独立批准；不得保留旧批准强行放行。
