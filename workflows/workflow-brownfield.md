# Workflow：Brownfield（既有项目改造为 DDD）

> 驱动 B。先逆向建模与接缝识别，再用绞杀者模式逐切片渐进重构。由 `/domain-driven-design:ddd-refactor` 触发。
> 第一纪律：**旧系统全程可运行、可回滚，绝不一次性重写。**
> 配套：`README.md`（体系蓝图与流程图）。
> 命名空间：文中 `domain-driven-design:<skill>` 为插件内 Skill 的完整调用名（详见 README「命名空间约定」）。

## 1. 链路

```
domain-driven-design:ddd-mode-router
   → domain-driven-design:ddd-code-survey      （从代码逆向重建现状领域模型 + 坏味道清单）★G0
   → domain-driven-design:ddd-seam-finder      （定位接缝与防腐层 ACL 落点）
   → domain-driven-design:ddd-strangler-plan   （迁移批次 + 回滚策略 + 特征化测试需求）★G1
   → ┌─ 逐切片循环 ───────────────────────────────────────────────┐
     │  [补特征化测试锁住现状] ★G2                                  │
     │  domain-driven-design:ddd-aggregates（局部） → domain-driven-design:ddd-domain-interactions（局部） │
     │  → domain-driven-design:ddd-model-review →（通过）domain-driven-design:ddd-spec-bridge            │
     │  → domain-driven-design:ddd-port-scaffold → domain-driven-design:ddd-adapter-impl → domain-driven-design:ddd-acceptance ★G3 │
     │  → 流量切换 + 验收 → 下一切片                                 │
     └────────────────────────────────────────────────────────────┘
```

> 前段（逆向 + 规划）是 brownfield 独有；从局部战术建模起与 greenfield **汇流复用同一套 Skill**（见 §6）。

## 2. 阶段工件衔接契约

| 阶段 | 关键输入 | 关键产出 |
| :--- | :--- | :--- |
| domain-driven-design:ddd-code-survey | 代码库 + 改造目标 | 现状模型、隐含上下文、规则打捞、坏味道、行为黑盒 |
| domain-driven-design:ddd-seam-finder | 现状模型 + 坏味道 | 接缝清单、ACL 落点、切割风险、特征化测试需求 |
| domain-driven-design:ddd-strangler-plan | 接缝 + ACL + 风险 | 切片清单、批次顺序、切换/回滚策略、DoD、退役计划 |
| （逐切片）特征化测试 | 切片 + 行为黑盒 | 锁住旧行为的对照基线 |
| domain-driven-design:ddd-aggregates（局部）| 该切片词汇 + 现状规则 | 该切片不变量与聚合 |
| domain-driven-design:ddd-domain-interactions | 局部聚合 | 该切片端口契约（中立）|
| domain-driven-design:ddd-model-review → domain-driven-design:ddd-acceptance | 同 greenfield | 验收报告 + 切流量放行 |

## 3. 与 Greenfield 的关键差异

| 维度 | Greenfield | Brownfield |
| :--- | :--- | :--- |
| 起点 | 问题陈述 | 既有代码 |
| 建模方向 | 正向（需求→模型）| 先逆向（代码→现状模型），再正向校准 |
| 推进粒度 | 整体一条链路 | **逐切片**，每片独立交付/回滚 |
| 安全网 | 验收测试 | 切片前先补**特征化测试**兜住旧行为 |
| 迁移方式 | 直接构建 | **绞杀者**：新旧并存，逐步切流量 |
| ACL 角色 | 隔离外部系统 | 还要隔离"残留旧代码"，迁移后退役 |

## 4. 阶段门禁（gate）

| 门禁 | 位置 | 放行条件 |
| :--- | :--- | :--- |
| G0 现状门禁 | `domain-driven-design:ddd-code-survey` 后 | 现状模型可解释现有关键行为；坏味道与行为黑盒已列清 |
| G1 规划门禁 | `domain-driven-design:ddd-strangler-plan` 后 | 切片顺序、回滚策略、特征化测试范围获确认；满足"旧系统始终可运行" |
| G2 切片安全门禁 | 每切片落地前 | 特征化测试已覆盖该切片现有行为（无裸切）|
| G3 切片验收门禁 | 每切片 `domain-driven-design:ddd-acceptance` 后 | 新实现通过契约/不变量测试；行为与旧实现等价；可切流量且可回滚 |

## 5. 完整回溯矩阵

| 触发条件 | 在哪发现 | 回退到 | 修复动作 |
| :--- | :--- | :--- | :--- |
| 该重写而非改造（核心不可救）| domain-driven-design:ddd-code-survey | domain-driven-design:ddd-mode-router | 重新分流/重定目标 |
| 改造目标在代码现实下不成立 | domain-driven-design:ddd-code-survey | domain-driven-design:ddd-mode-router | 与用户重定目标 |
| 现状模型不足以定位接缝 | domain-driven-design:ddd-seam-finder | domain-driven-design:ddd-code-survey | 补逆向勘测 |
| ACL 翻不动旧模型（结构冲突）| domain-driven-design:ddd-seam-finder | domain-driven-design:ddd-code-survey | 补理解，必要时重定目标 |
| 切片无法解耦/独立交付 | domain-driven-design:ddd-strangler-plan | domain-driven-design:ddd-seam-finder | 接缝选错，重找缝 |
| 特征化测试覆盖不了旧行为 | G2 / domain-driven-design:ddd-acceptance | domain-driven-design:ddd-code-survey | 行为黑盒未解，补理解 |
| 切片边界与聚合边界冲突 | domain-driven-design:ddd-aggregates（局部）| domain-driven-design:ddd-strangler-plan | 重排切片 |
| 新旧行为不等价（非预期）| domain-driven-design:ddd-acceptance | domain-driven-design:ddd-code-survey | 旧行为理解不足 |
| 端口契约语言无法表达 | domain-driven-design:ddd-port-scaffold | domain-driven-design:ddd-spec-bridge | 调整契约表达 |

> 进入局部战术建模后，§4 greenfield 回溯矩阵中与 aggregates/interactions/review/spec 相关的条目同样适用。

## 6. 与 Greenfield 的分叉与汇流

二者在 `domain-driven-design:ddd-mode-router` 分叉，在**局部战术建模（`domain-driven-design:ddd-aggregates` 起）汇流**到同一套 validation→spec→落地。改造驱动的独特性只在**前段**（逆向建模 + 接缝 + 迁移规划）；一旦定位到要改的切片，用的就是和新建完全一样的战术建模与接口优先落地——**最大化复用，避免维护两套战术 Skill。**

## 7. 端到端走查示例（订单单体拆分）

```
/domain-driven-design:ddd-refactor src/order-monolith --goal=拆分 --lang=java

router      → brownfield；切片：履约、计费
code-survey → OrderService 上帝类；Order.status 同名异义(物流/支付)；退款=行为黑盒 ── G0 ✅
seam-finder → 缝1履约(事件)、缝2计费(接口)、缝3物流读(视图)；两个 ACL ── 接缝地图
strangler   → 切片序 S1物流读→S2履约→S3计费；S3 留到机制成熟；回滚=特性开关 ── G1 用户确认 ✅
─ 切片 S2（履约）──────────────────────────────────────────
  特征化测试 → 锁"下单必扣库存/库存不足必拒单" ── G2 ✅
  aggregates(局部) → 履约不变量与聚合（发货单）
  interactions → OrderPlaced 事件、FulfillmentRepository 端口（中立）
  review → 通过；spec → 履约端口契约；scaffold(java) → interface 骨架
  impl → 履约领域实现 + ACL-Fulfillment 读旧库；accept → 契约/特征化等价 ✅
  流量切换 → 开关 5%→50%→100% 灰度，稳定 7 天 ── G3 ✅ → 取下一片
─ 下一切片 S3（计费）… 同上循环
```

## 8. 编排纪律

- **一次只动一片**：完成、验收、可回滚后再取下一片。
- **先兜底再替换**：没有特征化测试不动实现（G2 是硬门禁）。
- **沿用现有语言**：除非明确要换栈，落地取现有技术栈对应的语言剖面。
- **ACL 是临时的**：迁移完成后按 `domain-driven-design:ddd-strangler-plan` 的退役计划下线临时 ACL 与旧字段，不留永久债。
- **绞杀者纪律**：旧路径全程可用、可回滚，新实现灰度接管。
```
