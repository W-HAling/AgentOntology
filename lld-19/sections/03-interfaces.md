# SEC-19-03 接口契约

> **稳定 ID**：SEC-19-03  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)

## 1. 对外接口范围

MOD-19 不暴露同步 HTTP API，也不发布异步业务事件。唯一对外接口是 Python CLI、进程退出码、标准输出诊断和 GitHub Actions 调用契约。

## 2. CLI

### CLI-19-01 文档一致性

```powershell
python scripts/check_doc_consistency.py --stage-a --format text
python scripts/check_doc_consistency.py --post-stage-a --format json
```

- `--stage-a`：只要求当前 Stage A 已到期产物存在；未来服务和测试仅校验计划登记。
- `--post-stage-a`：按目标阶段启用 LLD、MODULE.md、代码与测试存在性检查。
- `--format`：`text|json`，默认 `text`。
- 两个阶段参数互斥且必须二选一。

### CLI-19-02 契约检查

```powershell
python scripts/check_contracts.py --mode local --format text
python scripts/check_contracts.py --mode github-pr --format json
python scripts/check_contracts.py --mode github-push --format json
```

- `local`：不调用 GitHub；只允许 `Proposed` 或 `In Review`。
- `github-pr`：验证当前 PR、head SHA 和 review。
- `github-push`：从发布视图回溯已合并 PR、merge commit 祖先关系和批准证据。

### CLI-19-03 聚合门禁

```powershell
python scripts/check_stage_a_gate.py --mode local
```

依次执行文档检查、契约检查和 `python -m pytest tests/stage_a -q`。任一子进程非零时停止并传播分类后的最终退出码。

## 3. 鉴权上下文

| 模式 | 必需上下文 | 来源 |
|---|---|---|
| local | 无远程凭据 | 本地进程 |
| github-pr | repository ID、PR number、head SHA、只读 GitHub token/App token | GitHub Actions 事件和 secret 引用 |
| github-push | repository ID、current commit SHA、只读 GitHub token/App token | GitHub Actions 事件和发布视图 |

令牌只能通过环境 secret 注入，不得出现在参数、日志、文档或仓库文件。权限必须最小化为读取仓库元数据、PR、review、CODEOWNERS/团队与协作者权限所需范围。

## 4. 内部接口

```python
@dataclass(frozen=True)
class Diagnostic:
    code: str
    severity: Literal["error", "warning"]
    path: str
    message: str
    subject_id: str | None

class Check(Protocol):
    def run(self, context: CheckContext) -> Sequence[Diagnostic]: ...

class ApprovalProvider(Protocol):
    def verify(self, request: ApprovalRequest) -> ApprovalResult: ...
```

实现必须保持输入只读、结果不可变、诊断稳定排序。内部异常需转换为 `DG-19-INPUT-*` 或 `DG-19-INTERNAL-*`，不得输出堆栈中的秘密值。

## 5. 版本契约

- CLI 主版本与治理 Schema 主版本独立演进。
- v1 CLI 参数删除、语义反转或退出码重分类属于破坏性变更。
- JSON 输出必须包含 `schema_version`、`tool_version`、`mode`、`exit_code`、`diagnostics`。
- 未知字段可由消费者忽略；既有必填字段不得在 v1 内删除。

## 6. 退出码

| 退出码 | 含义 | 示例 |
|---|---|---|
| 0 | 所有适用检查通过 | Proposed 文档在本地结构合法 |
| 1 | 内容、契约、追溯或审批冲突 | 摘要失配、断链、伪造 confirmed |
| 2 | 必需输入缺失或不可解析 | YAML 语法错误、CI 缺 repository ID |

测试失败属于内容验证失败并返回 1；检查器自身未捕获异常必须规范化为 2。

## 7. 稳定诊断码

| 诊断码 | 条件 |
|---|---|
| DG-19-EVIDENCE-MISSING | 引用证据不存在 |
| DG-19-DIGEST-MISMATCH | 重算摘要与声明不一致 |
| DG-19-STATE-INVALID | 非法状态或状态跃迁 |
| DG-19-TRACE-BROKEN | 追溯端点、版本或链路断裂 |
| DG-19-TRACE-CYCLE | 存在禁止的有向环 |
| DG-19-LLD-DOR | LLD 缺 DoR 项或存在待决项却声明批准 |
| DG-19-APPROVAL-UNTRUSTED | 审批主体、资格或上下文不可信 |
| DG-19-LINK-BROKEN | 相对链接目标不存在或越界 |
| DG-19-INPUT-PARSE | 输入不可解析 |
| DG-19-INTERNAL | 检查器内部失败 |
