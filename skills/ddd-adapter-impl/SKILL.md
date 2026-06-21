---
name: ddd-adapter-impl
description: "在接口骨架背后实现领域逻辑与基础设施适配器：聚合根行为、领域服务、仓储/外部系统适配器，严格遵守依赖规则（领域只依赖抽象）。当 ddd-port-scaffold 已产出目标语言接口骨架、需要填入真正实现时触发，是落地三步的第二步。"
risk: caution
stage: implementation
driver: both
source: self
tags: "[ddd, implementation, adapter, hexagonal]"
---

# DDD Adapter Impl（接口背后的实现）

落地三步的第二步——**先立约（port-scaffold）、再填肉（本 Skill）、后验收（acceptance）**。它在已生成的接口骨架背后写真实代码，但纪律极严：**领域核心只实现领域逻辑、只依赖端口抽象；一切技术细节（DB、HTTP、消息队列、外部 API）都封在实现端口的适配器里。** 换语言 = 换骨架 + 换适配器，领域逻辑的"形状"不变。这一步 `risk: caution`，因为它真正写入可运行代码。

## 使用时机

- `domain-driven-design:ddd-port-scaffold` 已产出目标语言的接口/值对象骨架与"待实现清单"。
- 需要把契约语义落成可运行的领域实现与基础设施适配器。
- 改造切片中，已为该切片定义好端口、准备替换旧实现。

## 输入要求

- **必需**：`domain-driven-design:ddd-port-scaffold` 的接口骨架与待实现清单；`domain-driven-design:ddd-spec-bridge` 的契约语义与不变量命题；所用语言剖面。
- **可选**：项目现有工程规范（包结构、测试框架、DI 方式）、目标数据库/消息中间件。

## 流程

1. **实现值对象与标识**：先把不可变值对象、ID 类型按契约填实（构造校验、按值相等），它们是聚合行为的基石。
2. **实现聚合根行为（领域核心）**：把每个根方法实现为**在内部强制其不变量**的行为——非法状态跃迁直接拒绝（抛中立领域错误 / 返回 Result）。聚合内部数据保持封装，外部只经根方法改动。
3. **实现领域服务**：把跨聚合的纯业务判定/编排逻辑落实，仅依赖端口抽象，不碰基础设施。
4. **实现仓储适配器（基础设施层）**：在领域定义的仓储接口背后实现持久化——ORM/SQL/NoSQL 皆可，但**领域模型与持久化模型解耦**（必要时做映射，禁止把 DB 行当聚合）。维护"1 聚合/1 事务"。
5. **实现外部系统适配器（ACL）**：对接 LDAP/邮件/支付等，把外部概念翻译成领域语言，隔离在防腐层内，绝不让外部模型渗进领域核心。
6. **接事件发布/订阅**：实现领域事件的发布与处理器，落实跨聚合的最终一致与补偿逻辑（来自一致性策略）。
7. **装配依赖（DI）**：按语言剖面的 DI 约定，在应用/启动层把适配器注入领域端口——**依赖在最外层组装，核心永不 new 具体实现**。
8. **依赖方向自检**：静态核对领域核心未 import 任何基础设施/框架；所有向外的调用都经端口。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 聚合/值对象实现 | 根方法内强制不变量；值对象不可变、按值相等；标注实现的 INV 编号 |
| 领域服务实现 | 仅依赖端口；无基础设施引用 |
| 仓储适配器 | 实现仓储端口；领域↔持久化映射；维护事务边界 |
| 外部系统适配器（ACL）| 把外部模型翻译为领域语言；隔离在防腐层 |
| 事件发布/订阅实现 | 发布点 + 处理器 + 补偿逻辑 |
| 依赖装配 | 外层 DI 组装；依赖方向核对结果 |

## 校验清单

- [ ] 领域核心**零基础设施/框架依赖**（静态可验证）
- [ ] 每个根方法都在内部强制其声明的不变量，非法操作被拒绝
- [ ] 仓储适配器维护"1 聚合/1 事务"，领域与持久化模型解耦
- [ ] 外部系统一律经 ACL 适配器接入，外部概念不进领域核心
- [ ] 依赖在最外层装配，核心代码从不实例化具体实现
- [ ] 实现与 `domain-driven-design:ddd-spec-bridge` 契约语义逐条对应，无擅自加方法
- [ ] 最终一致路径的事件发布/订阅与补偿均已实现

## 回溯触发

- 实现时发现某不变量无法在单聚合内强制 → 回退 `domain-driven-design:ddd-aggregates`。
- 某端口签名无法支撑真实实现（缺参数/语义）→ 回退 `domain-driven-design:ddd-spec-bridge` / `domain-driven-design:ddd-domain-interactions`。
- 外部系统模型与领域概念冲突到 ACL 翻不动 → 回退 `domain-driven-design:ddd-context-map`（集成模式）。

## 示例

```text
输入：Go 语言的 Booking/RoomSchedule 接口骨架 + 契约语义。语言剖面: go。

→ domain-driven-design:ddd-adapter-impl 产出（节选，仅示意结构，非完整代码）：

领域核心（domain 包，零外部依赖）：
  // RoomSchedule.Reserve 在内部强制 INV-1
  func (s *RoomSchedule) Reserve(slot Slot) (SlotReserved, error) {
      if s.isOccupied(slot) {            // 守 INV-1
          return SlotReserved{}, ErrSlotConflict   // 中立领域错误
      }
      s.occupied = append(s.occupied, slot)
      return SlotReserved{RoomID: s.roomID, Slot: slot}, nil
  }
  // ConflictPolicy 领域服务：仅依赖 RoomScheduleRepository 端口

基础设施（infrastructure 包，实现领域端口）：
  // PgRoomScheduleRepository 实现 domain.RoomScheduleRepository
  // - 领域聚合 ↔ 表行映射，不把 sql.Row 当聚合
  // - Save 在单事务内只写一个 RoomSchedule，乐观锁版本号
  type PgRoomScheduleRepository struct{ db *sql.DB }

ACL 适配器：
  // LdapIdentityAdapter 把 LDAP 条目翻译成 domain.Employee（外部概念不外泄）

依赖装配（main / 应用启动层）：
  repo := infra.NewPgRoomScheduleRepository(db)   // 外层 new 具体实现
  policy := domain.NewConflictPolicy(repo)        // 注入端口
  handler := app.NewSubmitBookingHandler(policy, bookingRepo, eventBus)

依赖方向自检：domain 包 import 列表中无 database/sql、无 net/http ✅

→ 交给 domain-driven-design:ddd-acceptance 写不变量/契约/用例测试验收。
```
