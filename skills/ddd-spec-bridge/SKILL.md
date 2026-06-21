---
name: ddd-spec-bridge
description: "把通过门禁的战术工件转译成语言中立的接口契约规范：端口清单（方法签名+契约语义）、不变量、领域事件 schema、验收准则，可选导出 OpenSpec 变更集。它是建模与落地之间的唯一交接物。当 ddd-model-review 通过、准备进入接口落地前触发。"
risk: safe
stage: specification
driver: both
source: self
tags: "[ddd, specification, contract, ports]"
---

# DDD Spec Bridge（规范衔接：语言中立接口契约）

规范阶段，**建模世界与代码世界之间的唯一桥**。前面产出的是"模型工件"（散落在多个 Skill 的表格），本 Skill 把它们收敛成**一份单一、自洽、可被任意面向接口语言直接消费的接口契约规范**。它是 `domain-driven-design:ddd-port-scaffold` 的唯一输入，也是 `domain-driven-design:ddd-acceptance` 的验收基线。一个硬纪律：**规范里零语言绑定**——只有端口、签名、语义、不变量、验收准则，不出现任何具体语言类型。

## 使用时机

- `domain-driven-design:ddd-model-review` 已判定通过 / 有条件通过且条件已补。
- 需要把建模工件冻结成可交接、可版本化的接口契约规范。
- 用户用 `/domain-driven-design:ddd-spec` 主动从一组战术工件导出规范。

## 输入要求

- **必需**：`domain-driven-design:ddd-aggregates` 的不变量表与聚合目录；`domain-driven-design:ddd-domain-interactions` 的事件/服务/仓储/工厂契约；`domain-driven-design:ddd-model-review` 的门禁结论。
- **可选**：`domain-driven-design:ddd-context-map` 的契约所有权（用于跨上下文事件 schema 的版本策略）；目标语言（仅作记录，不影响规范内容）。

## 流程

1. **汇总端口清单**：把所有仓储接口、领域服务接口、工厂接口收成一张"端口总表"，每个端口含：用途、所属聚合/上下文、全部方法。
2. **规范化方法签名（中立类型）**：用统一的中立类型记法重写每个方法——`方法名(参数: 中立类型) -> 返回类型 | null`，并标注**基数**（单个/集合）与**可空性**。禁止任何语言特有类型。
3. **固化契约语义**：每个方法附前置条件、后置条件、事务边界、查询边界、所守不变量编号、失败/空值语义。这是规范的"灵魂"，比签名更不可省。
4. **冻结领域事件 schema**：每个事件给出不可变载荷字段、类型、版本、owner 上下文、向后兼容策略（来自 context-map）。
5. **沉淀不变量为可验收命题**：把每条 INV 改写成"可观测、可测"的断言（"当 X 时，系统保证 Y"），供 `domain-driven-design:ddd-acceptance` 直接落成测试。
6. **写验收准则**：对每个关键用例与不变量，给出 Given-When-Then 形式的验收准则。
7. **（可选）导出 OpenSpec 变更集**：若团队用 OpenSpec/规范驱动开发，按其格式输出变更提案，把上述契约挂进工程规范库。
8. **版本与所有权标注**：给整份规范打版本号，标每个跨上下文契约的 owner，便于演进与回溯定位。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 端口契约规范 | 端口总表：每端口的方法签名（中立）+ 完整契约语义 |
| 领域事件 schema | 每事件：载荷字段+类型+版本+owner+兼容策略 |
| 不变量命题集 | 每条 INV → 可观测断言（当…保证…）|
| 验收准则 | 关键用例/不变量的 Given-When-Then |
| （可选）OpenSpec 变更集 | 符合 OpenSpec 的规范提案 |
| 规范元信息 | 版本号、覆盖的上下文、契约所有权、生成日期 |

## 校验清单

- [ ] 规范中**零语言特有类型**（无 `List<>`/`Promise`/注解/语言关键字）
- [ ] 每个端口方法都带完整契约语义（前置/后置/事务/查询边界/不变量/失败语义）
- [ ] 每条不变量都被改写为可观测、可测的断言
- [ ] 每个领域事件 schema 有版本与 owner，跨上下文事件有兼容策略
- [ ] 关键用例与不变量都有 Given-When-Then 验收准则
- [ ] 端口清单与 `domain-driven-design:ddd-domain-interactions` 完全对齐，无新增未建模的方法
- [ ] 规范自洽：方法引用的类型/事件都在规范内有定义

## 回溯触发

- 某契约缺少可表达的语义、或签名无法中立化 → 回退 `domain-driven-design:ddd-domain-interactions`。
- 转译中发现某不变量无法写成可观测断言（规则本身模糊）→ 回退 `domain-driven-design:ddd-aggregates`。
- 跨上下文事件 schema 的 owner/兼容策略缺失 → 回退 `domain-driven-design:ddd-context-map`。

## 示例

```text
输入：通过门禁的 Booking/RoomSchedule 战术工件。

→ domain-driven-design:ddd-spec-bridge 产出（节选，纯中立）：

== 端口契约规范 v1.0（上下文：Scheduling）==

Port: BookingRepository  [聚合: Booking]
  - findById(id: BookingId) -> Booking | null
      查询边界: 单聚合; 失败: 不存在返回 null
  - save(booking: Booking) -> void
      事务: 1 聚合/1 事务; 守: INV-2, INV-3; 失败: 乐观锁冲突抛 ConcurrencyError(中立名)

Port: RoomScheduleRepository  [聚合: RoomSchedule]
  - findByRoom(roomId: RoomId) -> RoomSchedule | null   [单聚合查询]
  - save(schedule: RoomSchedule) -> void                [1 聚合/1 事务; 守: INV-1]

Port: ConflictPolicy  [领域服务]
  - canReserve(roomId: RoomId, slot: Slot) -> Boolean
      前置: slot.start < slot.end; 后置: 无副作用(纯判定); 守: INV-1

== 领域事件 schema ==
  BookingSubmitted  v1  owner=Scheduling  兼容=加字段向后兼容
    { bookingId: BookingId, roomId: RoomId, slot: Slot, isHighValue: Boolean }

== 不变量命题集 ==
  INV-1: 当对 (roomId, slot) 已存在生效占用时，任何新的 reserve(同 slot) 必须失败。
  INV-3: 当房为高价值且未收到 approved=true 的 ApprovalDecided 时，Booking 不得进入"生效"。

== 验收准则（节选）==
  AC-1 (INV-1):
    Given 房 R 的 09:00-10:00 已被预订 B1 占用
    When  预订 B2 尝试占用 R 的 09:00-10:00
    Then  reserve 失败并产生 SlotReserveRejected，B1 占用不变

规范元信息：v1.0｜上下文 Scheduling｜契约 owner: Scheduling｜2026-06-20

→ 交给 domain-driven-design:ddd-port-scaffold 按目标语言剖面实例化接口骨架。
```
