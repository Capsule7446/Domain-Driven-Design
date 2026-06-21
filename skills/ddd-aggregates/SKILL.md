---
name: ddd-aggregates
description: "从不变量出发设计聚合：识别必须始终为真的业务规则，把强制它们的实体/值对象圈成聚合，确定聚合根、事务边界与跨聚合的一致性策略。当某上下文已划定、需要把模型落成可保护不变量的构造块时触发。"
risk: safe
stage: tactical
driver: both
source: self
tags: "[ddd, tactical, aggregate, invariant]"
---

# DDD Aggregates（聚合设计）

战术阶段第一步，也是**整套体系里最容易做错、最值得做对**的一环。核心方法论只有一句：**聚合不是按"数据相关"划的，而是按"必须在同一事务里保持为真的不变量"划的。** 聚合根是这组不变量的唯一守门人；一个事务只改一个聚合实例；跨聚合一致性走最终一致。做对了，并发与一致性问题大半消失；做错了（聚合变成大数据袋），处处是锁与脏数据。

## 使用时机

- `domain-driven-design:ddd-context-map` 已确定上下文边界与跨界一致性语义。
- 需要把某上下文内的概念落成聚合、确定事务与一致性边界。
- **被回溯触发**：`domain-driven-design:ddd-domain-interactions` 发现事件要携带他聚合私有数据、`domain-driven-design:ddd-model-review` 报"不变量表达率 < 60%"、或 `domain-driven-design:ddd-port-scaffold` 发现端口跨多聚合私有数据时回到这里。

## 输入要求

- **必需**：目标上下文的词汇表（`domain-driven-design:ddd-contexts`）、事件流与策略（`domain-driven-design:ddd-discover`）、跨界一致性语义（`domain-driven-design:ddd-context-map`）。
- **可选**：性能/并发要求、预期数据规模、现有数据模型（brownfield 切片时）。

## 流程

1. **先列不变量，不画对象**：把本上下文里"任何时刻都必须为真"的业务规则逐条写出（"已确认预订的时段不可被他人占用""审批通过前预订不可生效"）。**不变量是聚合设计的唯一第一性依据**。
2. **给每条不变量定作用域**：判断强制它需要哪些数据在"同一瞬间一致"。需要同事务保证的数据集，就是一个聚合的候选边界。
3. **圈聚合、选聚合根**：把强制同一组不变量所需的实体/值对象圈进一个聚合；指定唯一聚合根作为对外入口与一致性守门人。外部只能引用聚合根，不能直接持有内部实体。
4. **区分实体与值对象**：有标识、有生命周期、需追踪变化的是实体；由属性定义、可整体替换、无独立标识的是值对象（时段、金额、地址）。**优先用值对象**——它天然不可变、好测、无别名问题。
5. **定事务边界**：一条铁律——**一个事务只创建/修改一个聚合实例**。需要协调多个聚合的，改用领域事件 + 最终一致。
6. **设计跨聚合一致性**：列出"A 变了 B 也要跟着变"的关系，明确哪些必须强一致（多半说明边界划错、该并成一个聚合），哪些可最终一致（用事件）。
7. **聚合大小体检**：聚合过大（一个聚合含十几个实体、加载即慢、并发即锁）→ 多半把"数据相关"误当"不变量相关"，拆小。聚合过小到不保护任何不变量 → 可能只是值对象。
8. **用聚合引用而非对象引用**：聚合之间通过 ID 引用，不持有对方对象，保证可独立加载与演进。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 不变量表 | 表格：编号(INV-n)、规则（业务语言）、作用域、强制时机、所属聚合 |
| 聚合目录 | 表格：聚合名、聚合根、内含实体/值对象、对外暴露的根方法 |
| 值对象清单 | 表格：值对象、属性、不可变性与相等性说明 |
| 事务边界声明 | 每聚合：一事务可改的范围、被禁止的跨聚合写 |
| 一致性策略表 | 表格：跨聚合关系、强/最终一致、若最终一致用哪个事件、补偿策略 |
| 聚合引用图 | 聚合之间仅以 ID 互引的关系图 |

## 校验清单

- [ ] 每个聚合都至少强制一条具名不变量（INV-n），否则它不该是聚合
- [ ] 每条不变量都归属到唯一聚合，且该聚合能在单事务内强制它
- [ ] 一个事务只改一个聚合实例，无跨聚合写
- [ ] 聚合之间只用 ID 引用，不持有对方对象
- [ ] 优先值对象：可整体替换的概念未被错误建成实体
- [ ] 跨聚合一致性已逐条标强/最终，最终一致项都指定了事件与补偿
- [ ] 不存在"大数据袋"聚合（加载慢/高并发锁点）

## 回溯触发

- 某不变量必须跨两个上下文才能强制 → 回退 `domain-driven-design:ddd-contexts`（边界割裂了一致性）。
- 列不变量时发现缺触发事件 → 回退 `domain-driven-design:ddd-discover` 补事件。
- 跨聚合"强一致"项过多、聚合被迫合并成巨无霸 → 回退 `domain-driven-design:ddd-context-map` 复核边界与一致性语义。

## 示例

```text
输入：Scheduling 上下文词汇表 + 策略（冲突即拒绝；高价值需审批）。

→ domain-driven-design:ddd-aggregates 产出（节选）：

不变量表：
  | INV-1 | 同一会议室同一时段至多一个生效预订 | 房×时段 | 提交/确认时 | RoomSchedule |
  | INV-2 | 已确认预订的时段在取消前不可被改写 | 单预订 | 全生命周期 | Booking |
  | INV-3 | 高价值房预订未审批通过不得置为"生效" | 单预订 | 状态跃迁时 | Booking |

聚合目录：
  | 聚合 | 根 | 内含 | 根方法（摘要）|
  | Booking | Booking(实体) | Slot(值), BookingStatus(值), Attendees(值) | submit(), confirm(), cancel(), markApproved() |
  | RoomSchedule | RoomSchedule(实体) | Room(引用ID), 已占时段集合(值) | reserve(slot), release(slot) |

  说明：冲突检测(INV-1)由 RoomSchedule 守门；预订自身状态规则(INV-2/3)由 Booking 守门。
  Booking 仅以 RoomId + Slot 引用房，不持有 RoomSchedule 对象。

值对象清单：
  | Slot | {start:UTC, end:UTC, tz} | 不可变；按值相等；跨时区统一存 UTC |
  | BookingStatus | 枚举+跃迁规则 | 不可变；只能经根方法跃迁 |

事务边界：
  - 一个事务只改一个 Booking 或一个 RoomSchedule，不能同时改两者。
  - "提交预订占用时段"涉及 Booking + RoomSchedule 两聚合 → 不能同事务。

一致性策略：
  | Booking 提交 → RoomSchedule 占时段 | 最终一致 | 事件 BookingSubmitted | 补偿：占用失败则 Booking 转 Rejected |
  | Approval 结果 → Booking 置生效 | 最终一致 | 事件 ApprovalDecided | 补偿：超时未决则提醒/自动驳回 |

聚合引用图：
  Booking --(RoomId)--> Room ；RoomSchedule --(RoomId)--> Room ；二者经事件协作，不互持对象。

→ 交给 domain-driven-design:ddd-domain-interactions 把事件/服务/仓储/工厂接口语义化。
```
