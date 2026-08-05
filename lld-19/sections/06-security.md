# SEC-19-06 安全与可信审批边界

> **稳定 ID**：SEC-19-06  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)

## 1. 信任边界

| 区域 | 信任级别 | 控制 |
|---|---|---|
| 仓库版本化文件 | 条件可信 | 必须受 Git、Schema、摘要与路径规则约束 |
| 本地工作树声明 | 不可信审批源 | 仅允许 Proposed/In Review |
| GitHub API 响应 | 条件可信 | TLS、repository ID、PR、head SHA、node ID 绑定 |
| 任意外部 URI | 默认不可信 | Stage A allowlist 为空，禁止用作确认依据 |
| CI secret | 高敏感 | 仅环境注入、最小权限、日志脱敏 |
| 受治理归档 | 未启用 | ADR 未冻结前不得用于 confirmed |

## 2. 身份与认证

确认主体和审查者必须是 GitHub `User` 类型的不可变 node ID。登录名、邮箱、显示名、Bot、App、团队名或文档自报均不能替代 node ID。

CI 使用 GitHub OIDC/短期 GitHub App token 或平台提供的只读 token。长期个人访问令牌不得写入仓库。身份解析失败时默认拒绝。

## 3. 授权与独立性

有效批准必须同时满足：

1. review 状态为 `APPROVED`，属于目标 repository 和 PR。
2. review 对应当前 head SHA；push 模式还需 PR 已合并且 merge commit 是当前提交祖先。
3. reviewer 是变更路径的 CODEOWNER，审批时权限为 `maintain` 或 `admin`。
4. reviewer node ID 不在 PR 作者和提交作者/实施者 node ID 集合中。
5. review node ID、reviewed_at、repository ID、PR number、head SHA 和 document digest 完整且与 API 一致。

检查器不尝试检测现实世界关联账号；组织 IAM、强制 SSO 和 CODEOWNERS 成员治理是外部补偿控制。

## 4. 租户与仓库隔离

MOD-19 没有业务租户数据面。治理隔离键为 GitHub repository ID，所有 PR、review、commit 和权限查询必须绑定同一 repository ID。仓库名或 URL 只用于显示，不作为授权键。

## 5. 数据最小化与脱敏

- 敏感原文不能入库时，只保存脱敏摘录和签名清单引用。
- 诊断只输出稳定 ID、仓库相对路径和原因，不输出 token、Authorization header、私有原文或完整 GitHub API 响应。
- JSON 报告中的主体仅保留不可变 node ID；不额外复制邮箱等个人信息。

## 6. 密钥引用

允许的密钥来源仅为 GitHub Actions secret、GitHub App 安装令牌或组织密钥管理服务的引用。配置文件仅保存 secret 名称，不保存值。日志过滤必须覆盖 `Authorization`、`token`、`secret`、`private_key` 和签名材料。

## 7. 审计

每次 CI 门禁记录：repository ID、event type、PR number、head/current SHA、工具版本、开始/结束时间、退出码、诊断码集合、已验证 review node ID。审计记录不得声称“批准者已批准”之外的意图，也不得由检查器生成批准。

## 8. 威胁与控制

| 威胁 ID | 威胁 | 控制 |
|---|---|---|
| THR-19-01 | 修改 YAML 伪造 confirmed/Accepted | GitHub API 重取、摘要重算、CODEOWNER 与权限验证 |
| THR-19-02 | 用外部 URI 自报证据 | allowlist 默认空、拒绝网络取证 |
| THR-19-03 | 作者自批 | node ID 集合不相交守卫 |
| THR-19-04 | 旧 review 重放 | 绑定当前 head SHA、正文和原子清单摘要 |
| THR-19-05 | 相对路径逃逸 | 仓库根规范化与 `..` 拒绝 |
| THR-19-06 | `$ref` 指向不可信网络 | 仅允许仓库内文件 URI，相对解析 |
| THR-19-07 | 日志泄密 | 结构化脱敏与响应最小化 |
| THR-19-08 | CI 权限不足时降级放行 | fail closed，返回非零退出码 |
| THR-19-09 | TOCTOU 混用不同提交 | 单次运行固定 head SHA，漂移即失败 |
