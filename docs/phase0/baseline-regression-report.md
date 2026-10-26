# 第 0 阶段：基线回归与黄金业务切片报告

审计日期：**2026-10-26**。

**证据等级：仅基于 GitHub 仓库的静态源码与测试定义审查。本次没有实际执行回归测试。**

固定版本：
- Supplier：`fcb54e72bc896231a3ff76be2d2d45a22b39d9b4`
- Data Collector：`ab76892f5a07e1a7d362a83864f3e065197471f3`

## 一、审计证据范围

已查看 Supplier 的 `src/supplier_harness/{domain,service,calculator}.py`、`tests/test_regression_and_invariants.py`、`docs/design/{02_domain_contracts,06_domain_boundary_contracts}.md`，以及 Data Collector 的 `data_collector/{functions,releases}.py`、`docs/contracts.md` 和相关模块及测试目录。

**发现测试文件和相关断言，只证明存在测试覆盖定义，不证明本次执行通过。**

Supplier 既有设计文档曾报告 **386 项 pytest 通过**、**61 条 Discovery 案例 + 1 条 Service Trajectory** 及 **470 条断言**。这些只是历史报告数字，本次尚未重新验证。

## 二、核心回归门槛与静态证据

| 验证项 | 已发现的测试或契约 | 迁移后必须保持的行为 | 本次状态 |
| --- | --- | --- | --- |
| 不完整/无效 RFQ | `02_domain_contracts.md`、`test_invalid_geometry_blocks_even_if_profile_ranges_match` | 无效几何信息不进入计算；区分缺失、非法、歧义 | 有测试定义；未执行 |
| 事实读取失败 | `test_tool_failure_becomes_auditable_escalation` | 升级处理、不计算、不泄漏凭据 | 有测试定义；未执行 |
| 人工风险确认 | `test_category_a_requires_specific_human_evidence` | 有证据可进入待审核，但不得直接批准 | 有测试定义；未执行 |
| 策略版本冲突 | `test_stale_promotion_base_and_regression_changes_are_rejected` | 拒绝过期策略发布，防止覆盖新版本 | 有测试定义；未执行 |
| 已批准报价过期 | `test_approved_quote_expiry_cannot_export` | 过期报价不能导出/回复 | 有测试定义；未执行 |
| 交期证据范围 | `test_delivery_confirmation_is_bound_to_quantity` | 交期确认绑定数量、版本和范围 | 有测试定义；未执行 |
| 反馈与策略发布 | `test_feedback_to_confirmed_regression_to_promotion` | 反馈不得绕过评估和批准直接更新策略 | 有测试定义；未执行 |
| 案例来源一致性 | `test_fixture_bundle_matches_source_document` | Discovery 源文件与生成 JSON 一致 | 有测试定义；未执行 |
| 生成代码隔离 | `data_collector/functions.py`、`docs/contracts.md` | 宿主不直接导入 AI 生成 Adapter | 代码与契约已审查；未执行 |
| 不可变发布 | `data_collector/releases.py::load_release/prepare` | 测试、哈希与 Release 一致性校验 | 代码与契约已审查；未执行 |
| 事实时效 | Data Collector `docs/contracts.md` | 禁止以 mtime/retrieved_at 作为业务新鲜度依据 | 契约已审查；未执行 |

## 三、选定的黄金业务切片

**输入环境：** 合成测试 RFQ（如 `fixtures.DEMO_RFQ`）、受控 Fact Gateway、固定时钟、冻结的 Policy/Configuration 快照、临时隔离 SQLite Repository。

需要回放的完整流程：

1. `DecisionService.create` 进行字段提取、规范化与校验。
2. 通过 `FactGatewayPort` 获取报价所需事实，保留来源与时效。
3. 检查几何、制造和定价能力；所有硬性门控通过后才计算。
4. 使用现有 Calculator 与经批准的参数、Profile 生成报价草稿。
5. 保存 Decision、Trace、版本、Gate 与 Evidence。
6. 在授权 Reviewer、正确修订版本条件下执行 `DecisionService.review`。
7. 只有在已批准且当前有效时才允许 `DecisionService.reply`；过期或版本失效应拒绝。

**严禁直接在同一真实业务数据库执行两次写入 Action 作为新旧行为对比。** 必须使用隔离数据库副本或确定性重放环境。

## 四、可复现命令（本次未执行）

在对应固定源码版本、已安装依赖且使用隔离工作目录的环境中运行：

```bash
cd supplier-decision-harness
PYTHONPATH=src .venv/bin/python -m pytest -q tests
PYTHONPATH=src .venv/bin/python -m pytest -q tests/test_regression_and_invariants.py
PYTHONPATH=src .venv/bin/python -m supplier_harness.case_source --check
```

运行前需对照仓库当前 README 和 CLI 确认命令与虚拟环境路径。Data Collector 示例：

```bash
cd data-collector-agent
uv run pytest -q tests
```

Data Collector 的沙箱集成测试可能需要 Linux OCI Runtime；环境不支持时必须明确说明，不得把跳过当作通过。

## 五、黄金记录结构草案

以下数据仅用于描述**将来要冻结的测试字段**，不是实际测得的价格或状态：

```json
{
  "case_id": "ONTOLOGY-GOLDEN-QUOTE-001",
  "source_revision": "supplier-main-fcb54e7",
  "fixture_ref": "DEMO_RFQ",
  "policy_snapshot_ref": "PINNED_POLICY",
  "clock": "FIXED_UTC",
  "expected": {
    "decision_status": "AWAITING_REVIEW_OR_ESCALATED_BY_GATES",
    "calculation_allowed": "ONLY_IF_ALL_HARD_GATES_PASS",
    "human_approval_required": true,
    "reply_requires_current_validity": true,
    "trace_and_evidence_preserved": true
  }
}
```

正式基线应在执行旧路径后填写实际状态、精确金额、Policy 引用、Evidence、错误码、版本和变更效果，不应使用占位值充当验证结果。

## 六、验收状态与结论

- [x] 确定首个完整业务切片及其现有执行入口。
- [x] 将已有测试定义映射到核心业务不变量。
- [x] 给出复现命令及应保存的黄金记录字段。
- [x] 区分静态审计、历史测试报告和本次执行结果。
- [ ] 在隔离环境运行固定版本的测试，记录退出码、时间、解释器和依赖版本。
- [ ] 实际冻结读取、创建、审核、过期、失败路径的输出快照。
- [ ] 第 1 阶段完成 Schema 到 Binding 的往返验证。

**结论：文档审计基线可以进入讨论与评审；行为等价验收尚未通过。** 后续开发者不可据此宣称迁移后的系统已经验证安全。
