# SEC-19-10 实施映射与原子设计清单

> **稳定 ID**：SEC-19-10  
> **所属文档**：[LLD-19](../../lld-19-document-governance.md)  
> **清单版本**：1

## 1. 实施任务拆分

LLD-19 当前为 Proposed，下列条目只是设计拆分，不是已授权、ready 或 in_progress 的活动任务。任务 11、13 只有取得真实中期批准并写入台账后才能激活；任务 12、14 始终作为设计生产任务管理。

| 任务 ID | 对应计划任务 | 类型 | 目标 | 文件 | 前置条件 | 完成定义 |
|---|---|---|---|---|---|---|
| TASK-STAGEA-011 | 任务 11 | 实现任务 | 文档、契约与聚合门禁实现及正向测试 | `scripts/check_doc_consistency.py`、`scripts/check_contracts.py`、`scripts/check_stage_a_gate.py`、`tests/stage_a/test_doc_consistency.py`、`tests/stage_a/test_contracts.py`、`tests/stage_a/test_stage_a_gate.py` | LLD-19 有效中期批准；治理 Schema 已冻结 | 三个检查器及正向测试通过 |
| DTASK-STAGEA-012 | 任务 12 | 设计生产任务 | 统一 Tier-1 规则并升级两阶段设计门禁 | `.github/workflows/doc-check.yml`、`documentation-system.md`、`agent-ontology-master-plan.md`、`dual-ontology-three-layer-architecture-baseline.md` | TASK-STAGEA-011 verified | Tier-1 无规范冲突，PR/push 门禁调用已验证检查器 |
| TASK-STAGEA-013 | 任务 13 | 实现任务 | 实现负向门禁测试夹具 | `tests/stage_a/` | TASK-STAGEA-011 verified；LLD-19 中期批准摘要仍有效 | 计划规定的全部负向输入均被拒绝 |
| DTASK-STAGEA-014 | 任务 14 | 设计生产任务 | 独立封版评审与发布视图 | `contracts/traceability/stage-a-traceability.v1.yaml` | 任务 0 至 13 中全部已登记前置任务均为 `verified|accepted` 且证据有效；独立审查者可用；当前 head SHA 与原子摘要稳定 | 发布评审绑定当前摘要，门禁最终通过 |

## 2. 执行与验证命令

### 2.1 先写失败测试

```powershell
python -m pytest tests/stage_a/test_doc_consistency.py -q
python -m pytest tests/stage_a/test_contracts.py -q
python -m pytest tests/stage_a/test_stage_a_gate.py -q
```

预期：对应实现尚不存在时失败；不得通过跳过或空断言伪造红灯阶段。

### 2.2 实现后验证

```powershell
python -m pytest tests/stage_a -q
ruff check scripts tests/stage_a
python scripts/check_stage_a_gate.py --mode local
```

预期：pytest 全部通过、ruff 无错误、合法 Proposed/In Review 仓库退出码 0；负向 fixtures 全部被拒绝。

## 3. 原子设计清单规则

- 清单覆盖主文档与十个 section；清单本身不包含自身摘要，避免循环依赖。
- `digest` 在进入评审前由工具按规范化 UTF-8 内容计算并写入结构化发布视图；本文不填造尚未计算或批准绑定的摘要。
- `atomic_design_manifest_digest` 对按 `normative_id` 排序后的 `normative_id|path|digest` UTF-8 行计算 SHA-256。
- 任一原子内容变化必须更新其 digest 和清单总摘要，并使既有批准失效。

## 4. 规范原子

| 原子 ID | 规范性 ID | 路径 | 唯一负责事实 | 摘要状态 |
|---|---|---|---|---|
| ATOM-19-001 | LLD-19 | `lld-19-document-governance.md` | 索引、决策摘要、评审状态、未解决问题 | 待评审流水线计算 |
| ATOM-19-002 | SEC-19-01 | `lld-19/sections/01-boundaries.md` | 职责、非职责、上下游、事实源、治理角色与双链 | 待评审流水线计算 |
| ATOM-19-003 | SEC-19-02 | `lld-19/sections/02-components.md` | 组件和目标文件结构 | 待评审流水线计算 |
| ATOM-19-004 | SEC-19-03 | `lld-19/sections/03-interfaces.md` | CLI、内部接口、鉴权、版本、诊断与退出码 | 待评审流水线计算 |
| ATOM-19-005 | SEC-19-04 | `lld-19/sections/04-data-model.md` | 实体、字段语义、关系、图不变量、所有权与保留 | 待评审流水线计算 |
| ATOM-19-006 | SEC-19-05 | `lld-19/sections/05-state-and-reliability.md` | 状态机、事务、并发、幂等、重试、补偿与回放 | 待评审流水线计算 |
| ATOM-19-007 | SEC-19-06 | `lld-19/sections/06-security.md` | 信任、身份、授权、隔离、脱敏、密钥、审计与威胁 | 待评审流水线计算 |
| ATOM-19-008 | SEC-19-07 | `lld-19/sections/07-non-functional.md` | 性能、容量、可用性、观测、降级、恢复与预算 | 待评审流水线计算 |
| ATOM-19-009 | SEC-19-08 | `lld-19/sections/08-deployment-migration.md` | 配置、依赖、CI、迁移与回滚 | 待评审流水线计算 |
| ATOM-19-010 | SEC-19-09 | `lld-19/sections/09-scenarios.md` | 3 核心/2 边界/2 异常场景 | 待评审流水线计算 |
| ATOM-19-011 | SEC-19-10 | `lld-19/sections/10-implementation-map.md` | 实施拆分、命令、完成定义与原子清单规则 | 待评审流水线计算 |

## 5. 完成定义

1. 主文档和十个 section 的相对链接全部可解析且不越出仓库。
2. DoR 十一项全部有唯一规范位置，未解决问题为空。
3. 七个场景绑定稳定 UR/PR/DR、测试 ID、计划路径和 Stage A。
4. 原子摘要由门禁计算并与批准 review 绑定，不存在手填伪造摘要。
5. 文档状态保持 Proposed/In Review，直至真实独立审批验证成功。
6. 后续 Schema、检查器和测试不得偏离本清单；变更必须先修订 LLD 并重新评审。
