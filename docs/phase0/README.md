# 本体迁移：第 0 阶段

## 目标与状态

本阶段用于**固定现有系统的业务语义边界、执行所有权和迁移前验证基线**，以避免后续先设计 DSL、再倒逼现有业务重构。

**当前状态：文档基线已形成；行为等价验证尚未完成。**

这里的“完成”仅指基于指定源码版本进行的结构审计、候选概念映射和测试证据整理，不表示新旧接口已经通过实际运行对照。

## 交付文档

| 文档 | 主要内容 |
| --- | --- |
| [本体迁移清单](ontology-migration-inventory.md) | 现有模块与函数对应的候选 Object、Query、Action、Policy、Binding，以及首个完整报价切片 |
| [语义边界与所有权](semantic-boundaries.md) | Ontology、Runtime、Supplier Harness 和 Data Collector 的职责、权威来源、安全及扩展原则 |
| [基线回归与黄金业务切片报告](baseline-regression-report.md) | 现有测试定义、需要守住的业务不变量、复现命令和未完成的执行验证 |

## 审计依据与版本固定

审计日期：**2026-10-26**。

- [supplier-decision-harness 固定版本](https://github.com/shqpsyd/supplier-decision-harness/tree/fcb54e72bc896231a3ff76be2d2d45a22b39d9b4)
- [data-collector-agent 固定版本](https://github.com/shqpsyd/data-collector-agent/tree/ab76892f5a07e1a7d362a83864f3e065197471f3)

现有 Supplier Domain Service、Policy Service、Calculator 和 Data Collector 的隔离执行、不可变 Release 机制，仍是各自业务与执行行为的权威实现。

## 进入第 1 阶段之前

1. 审核并确认候选对象与 Action 的业务含义，不把文件中的候选名称直接当作正式 DSL。
2. 在隔离环境执行固定版本的测试，记录命令、退出码、依赖版本和运行时间。
3. 生成实际可比较的报价、审核、过期、错误处理和证据输出快照。
4. 基于验证结果定义 DSL v0.1、Binding Contract 与 Compiler 的最小范围。

不能以历史测试报告中的“通过数量”代替本次执行结果。
