---
name: ddd-domain-interactions
description: "把聚合之间与对外的协作落成语言中立的接口契约：领域事件、领域服务、仓储、工厂——每一个都只产出方法签名 + 契约语义（前置/后置条件、事务与查询边界、不变量引用），不绑任何语言。当聚合与不变量已定、需要定义它们如何协作时触发，是接口优先落地的真正起点。"
risk: safe
stage: tactical
driver: both
source: self
tags: "[ddd, tactical, interfaces, ports, events]"
---

# DDD Domain Interactions（领域协作的接口语义化）

战术阶段第二步，也是**本体系"接口优先、语言无关"主张的源头**。聚合定义了"什么必须为真"，本 Skill 定义"它们怎么协作把事做完"：哪些事实要以**领域事件**广播、哪些跨聚合编排逻辑放进**领域服务**、聚合怎样被**仓储**取存、复杂构造怎样交给**工厂**。关键纪律：这里产出的全部是**语言中立的契约**（签名 + 语义），绝不写实现、绝不绑语言——这正是后续 `domain-driven-design:ddd-spec-bridge` → `domain-driven-design:ddd-port-scaffold` 能在任意语言落地的前提。

## 使用时机

- `domain-driven-design:ddd-aggregates` 已产出聚合目录、不变量表、一致性策略。
- 需要定义聚合的取存方式、跨聚合协作、对外发布的事件契约。
- **被回溯触发**：`domain-driven-design:ddd-spec-bridge` 发现某契约语义缺失、`domain-driven-design:ddd-port-scaffold` 发现端口无法表达、或 `domain-driven-design:ddd-acceptance` 发现无契约可测时回到这里。

## 输入要求

- **必需**：`domain-driven-design:ddd-aggregates` 的聚合目录、不变量表、事务与一致性策略；`domain-driven-design:ddd-context-map` 的契约所有权与一致性语义。
- **可选**：性能/查询场景（影响仓储查询方法）、外部系统接口（影响端口边界）。

## 流程

1. **定义领域事件契约**：把 `domain-driven-design:ddd-aggregates` 标为"最终一致"的跨聚合协作，落成具名领域事件。每个事件：名字（过去式）、不可变载荷（**只含订阅方所需、不泄露发布聚合私有内部**）、由哪个聚合在何时发布、owner 上下文（来自 context-map）。
2. **设计领域服务**：把"不属于任何单一聚合、需要协调多个聚合或调用领域规则"的逻辑，定义为无状态领域服务接口（如冲突检测协调 Booking 与 RoomSchedule）。注意区别于应用服务（应用服务做编排/事务/鉴权，不含业务规则）。
3. **设计仓储接口**：每个聚合根一个仓储端口。方法以聚合为单位取存，**不暴露内部实体的独立增删改**；查询方法按真实用例设计，标注查询边界（单聚合/集合）。仓储接口属于领域层，实现属于基础设施层。
4. **设计工厂**：仅当聚合构造有复杂不变量或多步装配时才引入工厂端口；简单构造用聚合根构造器即可，不过度设计。
5. **为每个契约写语义注释**：前置条件、后置条件、可能触发的不变量编号、事务边界、失败/空值语义。**语义比签名更重要**——它是 `domain-driven-design:ddd-acceptance` 写契约测试的依据。
6. **标接口归属层**：明确每个接口属于领域层（端口）还是应用层（编排），并标出依赖方向（领域只依赖抽象）。
7. **自检无语言泄露**：通读所有契约，确保没有出现具体语言/框架类型（无 `List<>`、无 `Promise`、无注解），只用中立的"类型名 + 基数 + 可空性"。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 领域事件目录 | 表格：事件名（过去式）、不可变载荷字段、发布聚合、触发时机、owner 上下文、订阅方 |
| 领域服务接口 | 每个：服务名、方法签名（中立）、协调的聚合、对应不变量、前置/后置条件 |
| 仓储接口 | 每个聚合根一份：方法签名 + 查询边界 + 事务语义；标注"属领域层端口" |
| 工厂接口（按需）| 仅复杂构造：方法签名 + 装配的不变量 |
| 契约语义表 | 每个方法：前置/后置条件、不变量引用、失败与空值语义 |
| 层与依赖标注 | 每接口的层归属（领域端口/应用编排）+ 依赖方向 |

## 校验清单

- [ ] 每个领域事件载荷不含发布聚合的私有内部数据（仅订阅方所需）
- [ ] 每个聚合根恰好一个仓储接口；仓储不暴露内部实体的独立 CRUD
- [ ] 跨聚合编排逻辑落在领域服务，而非塞进某个聚合根
- [ ] 应用服务（编排/事务/鉴权）与领域服务（业务规则）已区分
- [ ] 每个方法都有契约语义（前置/后置/不变量/失败语义），不止签名
- [ ] 全部契约语言中立：无任何具体语言类型、框架注解、集合实现
- [ ] 仓储接口、领域服务接口都归属领域层，依赖只向内

## 回溯触发

- 某事件不携带发布聚合私有数据就无法表达 → 回退 `domain-driven-design:ddd-aggregates`（聚合边界划错）。
- 一个领域服务需要直接读写另一上下文的内部 → 回退 `domain-driven-design:ddd-context-map`（应走集成/ACL）。
- 仓储被迫提供"改聚合内部某实体"的方法 → 回退 `domain-driven-design:ddd-aggregates`（聚合根封装不足）。

## 示例

```text
输入：Booking / RoomSchedule 聚合 + 一致性策略（最终一致，事件协作）。

→ domain-driven-design:ddd-domain-interactions 产出（节选）：

领域事件目录：
  | BookingSubmitted | {bookingId, roomId, slot, isHighValue} | Booking 提交时发布 | owner=Scheduling | 订阅=RoomSchedule, Approval |
  | SlotReserved / SlotReserveRejected | {bookingId, roomId, slot} | RoomSchedule 占用成功/失败 | owner=Scheduling | 订阅=Booking |
  | ApprovalDecided | {bookingId, approved, approver} | Approval 决议后 | owner=Approval | 订阅=Scheduling |
  注：BookingSubmitted 不含 Booking 内部 Attendees 全量明细（订阅方不需要）。

领域服务接口（中立签名）：
  ConflictPolicy
    - canReserve(roomId: RoomId, slot: Slot) -> bool      [读 RoomSchedule；强制 INV-1]
      前置：slot 合法且 start<end；后置：不修改任何聚合（纯判定）

仓储接口（领域层端口）：
  BookingRepository
    - findById(id: BookingId) -> Booking | null            [单聚合查询]
    - save(b: Booking) -> void                              [事务: 1 聚合/1 事务; 守 INV-2,3]
  RoomScheduleRepository
    - findByRoom(roomId: RoomId) -> RoomSchedule | null     [单聚合查询]
    - save(s: RoomSchedule) -> void                         [事务: 1 聚合/1 事务; 守 INV-1]

工厂接口：
  （本例 Booking 构造简单，用根构造器，不引入工厂。）

契约语义表（节选）：
  | RoomSchedule.reserve(slot) | 前置: canReserve 为真 | 后置: 占用集合+slot, 发 SlotReserved | 失败: 已占→发 SlotReserveRejected | 守 INV-1 |
  | Booking.markApproved() | 前置: 状态=待审批 | 后置: 状态→生效 | 守 INV-3 |

层与依赖：
  仓储接口、ConflictPolicy 均属领域端口；应用服务 SubmitBookingHandler 负责事务/编排，依赖这些端口（向内依赖）。

→ 交给 domain-driven-design:ddd-model-review 做全局体检与门禁。
```
