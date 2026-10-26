# 第 0 阶段：本体迁移清单

**状态：候选语义映射，不是已批准的 DSL。** 审计日期：2026-10-26。

审计固定版本：
- `supplier-decision-harness`：`fcb54e72bc896231a3ff76be2d2d45a22b39d9b4`
- `data-collector-agent`：`ab76892f5a07e1a7d362a83864f3e065197471f3`

## 一、范围和权威来源

本清单审计已存在的领域契约与执行边界，不试图枚举每一个内部函数。

业务权威依据：
- [Supplier 核心领域契约](https://github.com/shqpsyd/supplier-decision-harness/blob/main/docs/design/02_domain_contracts.md)
- [Supplier 领域边界契约](https://github.com/shqpsyd/supplier-decision-harness/blob/main/docs/design/06_domain_boundary_contracts.md)
- [Supplier 工具与注册表契约](https://github.com/shqpsyd/supplier-decision-harness/blob/main/docs/implementation/tool_and_registry_contracts.md)
- [Supplier 业务策略](https://github.com/shqpsyd/supplier-decision-harness/blob/main/docs/implementation/policy.md)
- [Data Collector 规范契约](https://github.com/shqpsyd/data-collector-agent/blob/main/docs/contracts.md)

## 二、候选语义清单

下表的 Object、Action 等类型属于**拟议的 Ontology 表达形式**。只有通过后续确认和契约测试才能成为正式声明。

| 现有概念 | 对应源码或契约 | 候选类型 | 本体描述的内容 | 现有执行权威 |
| --- | --- | --- | --- | --- |
| RFQ | `domain.py::RFQ` | Object | 标识、类型、单位、字段状态 | 输入解析、规范化、Repository |
| Decision / QuoteDecision | `domain.py::Decision`、`service.py::DecisionService` | Object | 决策结果、状态及引用 | 状态推进、持久化 |
| Review | `domain.py::Review`、`DecisionService.review` | Object / Action 输入 | 人类决策及权限意图 | 审核权限、修订版本 |
| Fact | `domain.py::Fact` | Object / Value | 数据来源、时效、可信状态 | Fact Gateway、具体数据来源 |
| Gate | `domain.py::Gate` | Guard 结果 | 阻断状态、策略引用、解决方式 | 现有确定性业务检查 |
| PriceResult / CalculationTrace | `domain.py`、`calculator.py` | Value | 数值、单位、计算证据 | 原计算器，不复制公式 |
| PolicyRule | `domain.py::PolicyRule`、`policy_service.py` | Policy 引用 | 规则身份、版本、适用范围 | 已批准策略的执行与发布 |
| ProductResolution / PricingCapability | `product_resolution.py`、`service.py::pricing_capability_for` | 结果 / Guard | 型号匹配、制造能力与定价能力的区别 | 原领域检查 |
| Delivery Evidence | `delivery.py` | Evidence | 数量、版本、Scope、时效和审批关联 | 交期确认及核验机制 |
| FactGatewayPort | `domain.py::FactGatewayPort.get/refresh` | Query Binding 候选 | 读取契约、输入输出、证据要求 | 现有网关实现 |
| 创建报价决策 | `service.py::DecisionService.create` | Action | 建立建议/草稿，不代表批准报价 | 原工作流、计算、存储 |
| 审核报价决策 | `service.py::DecisionService.review` | Action | 人工审核、状态变更及授权 | 权威审核入口 |
| 报价回复 | `service.py::DecisionService.reply` | Query / Action 待判定 | 批准后回复的条件和效果 | 时效、审批状态、审计 |
| 回归案例 | `tests/test_regression_and_invariants.py`、`case_source.py` | 一致性证据 | 新旧执行行为验收条件 | 现有回归运行器 |
| FunctionSpec | `data_collector/models.py`、`functions.py` | Capability 描述 | 类型化接口与语义引用 | 函数目录与分发器 |
| Manifest | `data_collector/models.py`、`releases.py` | Binding Manifest | 数据来源、范围、适配版本 | 现有来源选择、Release |
| 稳定调用入口 | `data_collector/api.py::invoke` | Adapter 边界 | 可发布的能力调用契约 | 受限 worker 和沙箱 |
| 不可变发布 | `data_collector/releases.py` | 实现版本 | 引用已批准的绑定版本 | 发布、ACTIVE 和恢复流程 |
| 文件扫描与主动申报 | `scanner.py`、`declarations.py` | 数据治理事件 | 必要时向外暴露的状态 | 原文件系统追踪机制 |

特别注意：
- 部分 RFQ 字段可以缺失或歧义，不可直接将“已解析”理解为“可报价”。
- 制造能力的确认不等于定价能力已经得到确认。
- 文件的 `mtime` 或读取时间不是事实在业务上仍然有效的证明。
- `FactGatewayPort.refresh` 可能具有外部副作用；确认之前不得声明为纯只读 Query。
- `DecisionService.reply` 的实际副作用仍待审查。

## 三、首个完整业务切片

选定：**RFQ 接收 → 字段规范化与校验 → 获取有来源的材料事实 → 制造及定价能力检查 → 生成报价草稿 → 人工审核 → 在有效期内回复**。

这是一段沿用现有 `DecisionService` 的完整受控流程，不能为了演示跳过工艺、几何、制造、交期等硬性业务门控。

| 候选公开能力 | 类型 | 执行绑定建议 | 特别约束 |
| --- | --- | --- | --- |
| `GetRFQ` | Query | 既有 Repository / 领域服务，待确认 | 严格区分草稿和有效 RFQ |
| `GetMaterialPrice` | Query | `FactGatewayPort` | 必须携带来源和时效 |
| `GenerateQuotationDraft` | Action | `DecisionService.create` | 继承全部原有硬性门控 |
| `ReviewQuotationDecision` | Action | `DecisionService.review` | 人类授权与版本检查 |
| `GetApprovedReply` | 类型待定 | `DecisionService.reply` | 必须核实是否有副作用 |

禁止把 `calculator.calculate` 直接公开成不受门控的自主报价工具。

## 四、提取方法

**可通过静态扫描发现：** Pydantic 模型、Protocol 签名、FastAPI 路由、服务方法、测试 ID。

**必须补充业务语义：** 对象身份、币种与单位、策略适用范围、授权和确认、Action 效果、状态时效、来源证据、业务生命周期。

**不应该直接复制到本体：** 内部存储事务、计算公式实现、原始文件扫描算法、LLM 提示词和容器执行细节。

推荐流程：**扫描代码 → 候选映射 → 业务确认 → 契约测试 → 正式 DSL**，不能把 LLM 生成的候选直接发布为权威定义。

## 五、尚未解决的问题与退出条件

1. 确认 RFQ、Fact、Review 的稳定业务 ID 与版本。
2. 明确 Action 的状态变化、幂等性与真实副作用。
3. 定义 `UNKNOWN`、`AMBIGUOUS`、`UNVERIFIED`、`UNSUPPORTED` 等状态的不同语义。
4. 确认数据查询的授权范围及多工作区隔离方式。
5. 为 Data Collector 的 `FunctionSpec` / `Manifest` 设计最小 Binding 映射。
6. 在隔离环境获取实际旧路径输出快照。

**结论：** 本清单已经给出有源码依据的候选映射，可用于下一步 DSL 设计；但在完成上述确认前，不得将候选字段或 Action 自动转为正式可调用 API。
