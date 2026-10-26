# ontology-core：本体驱动的 Agent 平台核心

本仓库用于逐步构建 **Ontology DSL、编译器、能力投影和轻量运行时**，使不同业务 Agent 可以通过统一业务语义调用现有业务服务和数据适配器。

目前处于 **第 0 阶段（基线审计与语义切片）**，尚未实现或发布 Ontology 编译器、SDK 或生产运行时。

## 当前交付

- [第 0 阶段文档入口](docs/phase0/README.md)
- [本体迁移清单](docs/phase0/ontology-migration-inventory.md)：现有代码到本体概念的候选映射。
- [语义边界与所有权](docs/phase0/semantic-boundaries.md)：明确本体与现有业务系统的职责分工。
- [基线回归与黄金业务切片报告](docs/phase0/baseline-regression-report.md)：现有测试证据、待执行验证和迁移验收门槛。

## 基本原则

1. 业务语义统一，但不重复实现现有报价计算、审核、数据采集和存储机制。
2. 本体定义、实现绑定、Agent 需求契约分别版本化。
3. 所有生成接口均需经过类型、权限、状态和业务约束验证。
4. 数据来源与事实的新鲜度、证据、确认状态必须显式表达。
5. 先迁移一条可验证的报价业务切片，再扩展到 Projection、Translation Agent 和 Data Agent。

> **状态说明：** 当前已完成的工作是源码静态审计和文档化，不代表已运行端到端回归测试或通过行为等价验收。
