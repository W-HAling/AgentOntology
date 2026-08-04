# SEC-19-09 场景化验收

> **稳定 ID**：SEC-19-09  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)  
> **目标阶段**：Stage A

以下测试均为计划测试；路径在对应 TASK 进入活动状态后才要求存在。场景直接绑定任务 0 已登记的现有 `UR-ONTO-*`、`PR-ONTO-*`，并为本 LLD 建立 `DR-MOD19-*` 节点；这些引用只表达追溯关系，不把 UR/PR 声明为已确认或生效，也不把 DR 声明为已批准。

## 1. 核心场景

### SCN-19-CORE-001 已确认需求形成设计生产链

**绑定**：`UR-ONTO-008`、`PR-ONTO-006`、`DR-MOD19-001`  
**测试 ID**：`TEST-19-001`  
**计划路径**：`tests/stage_a/test_doc_consistency.py::test_confirmed_requirement_has_complete_design_chain`

**Given** 仓库证据存在且摘要匹配，需求确认由合格 GitHub User/CODEOWNER 对当前 head SHA 作出  
**When** 门禁校验 `evidence -> UR -> PR -> DTASK -> DR/LLD/contract -> SCN`  
**Then** 设计生产链通过，且结果不创建或伪造任何审批记录

### SCN-19-CORE-002 已批准 LLD 授权门禁工具实现任务

**绑定**：`UR-ONTO-008`、`PR-ONTO-006`、`DR-MOD19-002`  
**测试 ID**：`TEST-19-002`  
**计划路径**：`tests/stage_a/test_stage_a_gate.py::test_approved_lld_authorizes_matching_task`

**Given** LLD-19 具有可回溯独立批准，TASK 绑定其版本、正文摘要和原子清单摘要  
**When** 检查器重算全部摘要并验证实现交付链  
**Then** TASK 可进入 ready，且代码、测试和验证证据可双向追溯到批准 LLD 与 UR

### SCN-19-CORE-003 聚合门禁通过

**绑定**：`UR-ONTO-008`、`PR-ONTO-006`、`DR-MOD19-003`  
**测试 ID**：`TEST-19-003`  
**计划路径**：`tests/stage_a/test_stage_a_gate.py::test_gate_returns_zero_for_valid_repository`

**Given** 文档、契约、台账、追溯、审批和测试输入全部有效  
**When** 执行聚合门禁  
**Then** 文档检查、契约检查和 pytest 依次通过，最终退出码为 0

## 2. 边界场景

### SCN-19-EDGE-001 Stage A 未来任务不要求产物存在

**绑定**：`UR-ONTO-001`、`PR-ONTO-001`、`DR-MOD19-004`  
**测试 ID**：`TEST-19-004`  
**计划路径**：`tests/stage_a/test_doc_consistency.py::test_stage_a_allows_future_stage_artifact_paths`

**Given** MOD-15 至 MOD-18 未来 TASK 仅登记 ID、计划路径与目标阶段且未标活动  
**When** 使用 `--stage-a` 校验  
**Then** 不因未来代码或测试文件尚不存在而失败，但错误标为活动时必须失败

### SCN-19-EDGE-002 大型合法追溯图在线性预算内完成

**绑定**：`UR-ONTO-008`、`PR-ONTO-006`、`DR-MOD19-005`  
**测试 ID**：`TEST-19-005`  
**计划路径**：`tests/stage_a/test_doc_consistency.py::test_large_trace_graph_within_budget`

**Given** 追溯图包含 10,000 节点和 20,000 条合法边  
**When** 执行唯一性、端点、版本、环与可达性检查  
**Then** 在 NFR-19-002 预算内完成且不发生超过 512MiB 的内存使用

## 3. 异常场景

### SCN-19-ERR-001 本地伪造 Accepted 被拒绝

**绑定**：`UR-ONTO-008`、`PR-ONTO-006`、`DR-MOD19-006`  
**测试 ID**：`TEST-19-006`  
**计划路径**：`tests/stage_a/test_contracts.py::test_local_accepted_claim_is_rejected`

**Given** 文档自报 Accepted，但没有可信 PR、review、CODEOWNER 和权限上下文  
**When** 本地模式执行审批验证  
**Then** 返回 `DG-19-APPROVAL-UNTRUSTED` 和退出码 1，不生成替代审批

### SCN-19-ERR-002 摘要或追溯断链阻断发布

**绑定**：`UR-ONTO-008`、`PR-ONTO-006`、`DR-MOD19-007`  
**测试 ID**：`TEST-19-007`  
**计划路径**：`tests/stage_a/test_stage_a_gate.py::test_digest_or_trace_break_blocks_gate`

**Given** LLD 原子清单摘要失配，或追溯边端点不存在  
**When** 聚合门禁执行  
**Then** 输出 `DG-19-DIGEST-MISMATCH` 或 `DG-19-TRACE-BROKEN`，跳过发布并返回退出码 1

## 4. 验收总则

- 七个场景全部实现并通过后，才满足 LLD-19 场景门禁。
- 每个测试必须包含至少一个失败断言，证明门禁不会静默放行。
- 场景引用的 UR/PR/DR 已登记为追溯节点，但引用关系不改变各实体的独立生命周期状态，不得据此声称用户已确认、规划已生效或设计已批准。
