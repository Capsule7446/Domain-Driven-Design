---
name: ddd-model-review
description: "全局模型质量门禁：对战略+战术工件做一致性、完整性、可实现性评估，量化打分，输出问题清单与可触发的回溯指令。是 0→1 与改造链路在进入落地前的强制体检站。当战术建模完成、准备进入规范/落地前触发。"
risk: safe
stage: validation
driver: both
source: self
tags: "[ddd, validation, gate, quality]"
---

# DDD Model Review（模型验证门禁）

验证阶段，整套体系的**批判性关卡**。前面所有 Skill 都在"建"，本 Skill 专门"拆"——用量化标准检查模型是否一致、完整、可落地，并在不达标时**精确指出回退到哪个上游 Skill**。它是体系"双向闭环"原则的执行者：没有它，回溯就只是口号。它不提新模型，只做裁判与导流。

## 使用时机

- `domain-driven-design:ddd-domain-interactions` 已产出事件/服务/仓储/工厂契约（greenfield），或某改造切片完成局部战术建模（brownfield）。
- 进入 `domain-driven-design:ddd-spec-bridge` 前的强制门禁。
- 用户用 `/domain-driven-design:ddd-review` 主动给现有模型做体检（无论模型来自本体系还是外部）。

## 输入要求

- **必需**：战略工件（子域分类、上下文目录与映射）、战术工件（不变量表、聚合目录、接口契约）。
- **可选**：`domain-driven-design:ddd-scope` 的目标/非目标与价值主张（用于完整性回溯校验）。

## 流程

1. **一致性检查**：通用语言是否在工件间一致？同名术语含义是否在同一上下文唯一？聚合/事件命名是否与词汇表对齐？
2. **完整性检查**：`domain-driven-design:ddd-scope` 的每条目标是否都有对应的事件流→聚合→契约链路覆盖？有没有"价值主张相关路径"在某一阶段断掉？
3. **不变量健壮性检查**：计算**不变量表达率** = 被聚合根方法显式强制的不变量数 / 不变量总数。逐条核对每条 INV 是否真的有归属聚合在守。
4. **聚合健康检查**：是否存在"无不变量的聚合"（数据袋）、"跨聚合强一致写"、"事件携带他聚合私有数据"？
5. **可实现性检查**：每个端口契约是否语言中立、可被目标语言接口表达？仓储是否以聚合为单位、未泄露内部？依赖方向是否只向内？
6. **量化打分**：对一致性/完整性/不变量/聚合健康/可实现性五维各打分，给总评（通过 / 有条件通过 / 不通过）。
7. **生成回溯指令**：对每个未达标项，按"回溯矩阵"映射到具体上游 Skill，给出可执行的回退建议；通过项明确放行到 `domain-driven-design:ddd-spec-bridge`。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 五维评分表 | 一致性/完整性/不变量表达率/聚合健康/可实现性，各打分 + 依据 |
| 问题清单 | 表格：问题、严重度（阻断/重要/建议）、所在工件、证据 |
| 回溯指令 | 表格：未达标项 → 回退目标 Skill → 具体修改建议 |
| 门禁结论 | 通过 / 有条件通过（列条件）/ 不通过（必须回溯项）|

## 校验清单（门禁阈值）

- [ ] **不变量表达率 ≥ 60%**（低于此判定聚合多为数据容器 → 回溯）
- [ ] 无"阻断级"一致性问题（通用语言冲突、同上下文同名异义）
- [ ] `domain-driven-design:ddd-scope` 每条目标都被某条建模链路覆盖（完整性无断点）
- [ ] 无跨聚合强一致写；无事件泄露他聚合私有数据
- [ ] 全部端口契约语言中立且可被接口表达
- [ ] 每个未达标项都已映射到明确的回溯目标，无"悬空问题"

## 回溯触发（回溯矩阵）

| 判定 | 回退到 |
| :--- | :--- |
| 不变量表达率 < 60% / 存在数据袋聚合 | `domain-driven-design:ddd-aggregates` |
| 事件携带他聚合私有数据 | `domain-driven-design:ddd-aggregates` → 必要时 `domain-driven-design:ddd-domain-interactions` |
| 同名术语跨上下文冲突 | `domain-driven-design:ddd-contexts` |
| 不变量需跨上下文强制 | `domain-driven-design:ddd-contexts` / `domain-driven-design:ddd-context-map` |
| 某目标无建模覆盖（完整性断点）| `domain-driven-design:ddd-discover`（缺事件）或 `domain-driven-design:ddd-scope`（缺目标定义）|
| 核心域/支撑域错配 | `domain-driven-design:ddd-subdomains` |
| 契约语言无法表达 | `domain-driven-design:ddd-domain-interactions`（契约语义）|

## 示例

```text
输入：会议室预订的全套战略+战术工件。

→ domain-driven-design:ddd-model-review 产出（节选）：

五维评分：
  一致性      ✅ 通过   术语与词汇表一致
  完整性      ⚠ 有缺口  "取消已生效预订"目标无对应事件链
  不变量表达率 ✅ 100%   INV-1 由 RoomSchedule、INV-2/3 由 Booking 显式强制
  聚合健康    ✅ 通过   无数据袋；无跨聚合强一致写
  可实现性    ✅ 通过   契约均中立、可接口化

问题清单：
  | 取消已生效预订缺事件与释放逻辑 | 重要 | 事件目录 | domain-driven-design:ddd-discover 泳道里有"预订已取消"但未落事件 |
  | 审批超时无补偿策略 | 建议 | 一致性策略表 | ApprovalDecided 未定义超时分支 |

回溯指令：
  | 取消链缺失 → 回退 domain-driven-design:ddd-discover 补"取消/释放"事件，再过 aggregates/interactions |

门禁结论：有条件通过 —— 补齐"取消"链路后放行 domain-driven-design:ddd-spec-bridge；其余为建议项可并行处理。

→（补齐后）交给 domain-driven-design:ddd-spec-bridge 导出语言中立接口契约规范。
```
