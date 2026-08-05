# SEC-19-08 CI、部署、迁移与回滚

> **稳定 ID**：SEC-19-08  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)

## 1. 运行拓扑

MOD-19 作为仓库工具运行于开发者本地和 GitHub Actions，不部署常驻服务。

```text
Developer / Pull Request / Push
              |
              v
      check_stage_a_gate.py
       |        |        |
       v        v        v
 doc check  contract   pytest
                |
                v
        GitHub read-only API
```

## 2. 配置

| 配置 | 必需模式 | 约束 |
|---|---|---|
| `GOVERNANCE_MODE` | 全部 | `local|github-pr|github-push` |
| `GITHUB_REPOSITORY_ID` | GitHub | 不可变数字/Node ID，以实现契约为准 |
| `GITHUB_PR_NUMBER` | github-pr | 正整数 |
| `GITHUB_HEAD_SHA` | github-pr | 40 位 Git SHA |
| `GITHUB_CURRENT_SHA` | github-push | 40 位 Git SHA |
| `GITHUB_TOKEN` | GitHub | secret 注入、只读、禁止日志输出 |
| `GOVERNANCE_REPORT_PATH` | 可选 | 必须位于工作区或 CI artifact 临时目录 |

命令行显式参数优先于非秘密环境变量；秘密只能来自环境。缺失必需配置返回 2。

## 3. 依赖版本

Stage A 基线使用 Python 3.12，并由 `requirements-stage-a.txt` 精确固定：

- `jsonschema[format]==4.25.0`
- `openapi-spec-validator==0.7.2`
- `PyYAML==6.0.2`
- `pytest==8.4.1`
- `ruff==0.12.7`

GitHub API 客户端优先使用 Python 标准库；若引入第三方依赖，必须先修订本 LLD 原子清单和依赖文件。

## 4. GitHub Actions 集成

PR 与 push 使用相同路径覆盖集合：Tier-1 文档、`lld-*.md`、`lld-*/sections/**`、`adr/**`、`contracts/**`、`scripts/check_*.py`、`tests/stage_a/**`、`requirements-stage-a.txt` 和工作流自身。

- PR 模式直接验证事件 PR 与 head SHA。
- push 模式读取发布视图的 PR/review 标识，经 API 确认 PR 已合并、merge commit 为当前提交祖先，再重验审批资格和摘要。
- 无法访问团队、CODEOWNERS 或协作者权限时失败；需要时使用只读 GitHub App secret 引用。
- 并发组按 workflow、repository、PR/ref 组成，新提交取消旧运行。

## 5. 迁移步骤

1. 引入 LLD-19 主文档与十个 section，状态保持 Proposed。
2. 创建治理 Schema、需求事实源、追溯矩阵和执行台账。
3. 先创建负向测试，确认缺检查器时失败。
4. 实现文档、契约和聚合检查器。
5. 在本地模式运行全部测试与 lint；本地不验证 Accepted。
6. 以 PR 模式启用非阻断观察，修复误报和断链。
7. 取得真实独立审批后启用 Stage A 阻断门禁。
8. Stage A 结束后按目标阶段启用 `--post-stage-a` 渐进存在性检查。

迁移期间旧 shell grep 只可保留为冗余提示，不得与 Python 门禁产生两个权威结论；切换完成后删除旧逻辑。

## 6. 回滚

- 工作流故障时可回滚到上一个已批准检查器版本，但不得跳过审批与追溯检查进入发布。
- Schema 回滚必须同时回滚数据文件和检查器；禁止新 Schema 数据由旧检查器静默接受。
- 若回滚导致当前批准摘要变化，批准自动失效，状态回到 In Review。
- 紧急情况下只能暂停发布，不允许 `continue-on-error` 或固定返回 0。

## 7. 验证命令与预期结果

```powershell
python -m pytest tests/stage_a -q
ruff check scripts tests/stage_a
python scripts/check_stage_a_gate.py --mode local
```

预期：测试全部通过、ruff 无错误、Proposed/In Review 内容在本地返回 0；任何 `Accepted` 自报、断链、摘要失配或不可解析输入返回非零。
