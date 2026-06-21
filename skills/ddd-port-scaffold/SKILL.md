---
name: ddd-port-scaffold
description: "接口优先脚手架：把语言中立的端口契约（来自 ddd-spec-bridge）按目标语言的语言剖面，实例化为接口/抽象骨架——只含签名与契约注释，不含实现。适配任意面向接口开发的语言。"
risk: safe
stage: implementation
driver: both
source: self
tags: "[ddd, implementation, ports, language-agnostic]"
---

# DDD Port Scaffold

阶段 VI 的第一步，也是**语言无关落地的关键**。它把领域模型沉淀的"端口契约"翻译成目标语言的接口构造，**领域逻辑此刻仍不落地**（由 `domain-driven-design:ddd-adapter-impl` 接手）。一句话：先立约，再填肉。

## 使用时机

- `domain-driven-design:ddd-spec-bridge` 已产出语言中立的端口契约规范。
- 需要在某目标语言里搭出"接口骨架 + 类型签名"，供团队/下游分头实现。
- 既有项目改造中，为某个切片定义新的防腐层/端口边界。

## 输入要求

- **必需**：端口契约规范（来自 `domain-driven-design:ddd-spec-bridge`）；聚合目录与不变量表（来自 `domain-driven-design:ddd-aggregates`）。
- **必需**：`语言剖面` 入参——目标语言标识（`java|go|ts|python|csharp|rust|kotlin|...`）。
- **可选**：既有代码风格约定（命名、包结构）、依赖注入方式偏好。

## 流程

1. **载入语言剖面**：从 `references/language-profiles.md` 取该语言的构造映射（接口/值对象/不可变/相等性/DI）。若语言未收录，按"剖面问卷"现场采集这五项即可支持新语言。
2. **映射端口**：每个中立端口 → 目标语言的接口/trait/Protocol，逐方法翻译签名，**保留契约注释**（前置条件、查询边界、事务边界、不变量引用）。
3. **映射值对象与标识**：值对象 → 该语言的不可变值类型（按值相等）；实体标识 → ID 类型；区分清楚。
4. **生成依赖方向骨架**：领域包仅声明接口与领域类型；基础设施包留空实现占位（待 `domain-driven-design:ddd-adapter-impl`）。确保依赖只向内。
5. **植入门禁注释**：在每个接口/方法上以注释标注其对应的不变量编号与一致性策略，便于 `domain-driven-design:ddd-acceptance` 写契约测试。
6. **输出骨架文件清单**（仅签名 + 注释，无逻辑），并标出"待实现"占位点。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 端口接口骨架 | 目标语言接口文件：方法签名 + 契约注释，无实现 |
| 值对象/标识骨架 | 不可变值类型与 ID 类型声明，含相等性策略 |
| 依赖方向说明 | 包/模块结构 + 依赖指向（领域内核零外部依赖） |
| 待实现清单 | 表格：文件、占位点、对应聚合/不变量、移交给 `domain-driven-design:ddd-adapter-impl` |
| 所用语言剖面 | 记录采用的剖面与五项构造映射，确保可复现 |

## 校验清单

- [ ] 每个端口方法都保留了契约注释（前置条件/事务/查询边界）
- [ ] 值对象用了该语言的不可变构造，且按值相等
- [ ] 领域内核未引入任何基础设施/框架依赖（依赖只向内）
- [ ] 骨架内**无业务实现逻辑**（实现是下一个 Skill 的职责）
- [ ] 每个接口/方法已关联到具体不变量编号
- [ ] 若语言未收录，已通过剖面问卷补齐五项并记录

## 回溯触发

- 某端口契约无法用目标语言的接口表达（如需要联合类型/和类型而语言不支持）→ 回退到 `domain-driven-design:ddd-spec-bridge` 调整契约表达。
- 翻译时发现端口方法跨了多个聚合的私有数据 → 回退到 `domain-driven-design:ddd-aggregates` 重审边界。

## 语言剖面问卷（支持任意新语言）

对未收录语言，只需回答这 5 项即可生成骨架：

1. 接口/抽象类型用什么构造？（interface / trait / Protocol / abstract）
2. 不可变值类型怎么表达？（record / frozen dataclass / readonly / data class）
3. 按值相等如何实现？（自动 / 需手写 equals+hashCode / derive）
4. 依赖注入约定？（构造器注入 / 显式传参 / 容器）
5. 错误/空值表达？（异常 / Result / Option / null）

## 示例

```text
/domain-driven-design:ddd-scaffold Booking --lang=go

输入端口契约（节选）：
  Port: BookingRepository
    - findById(id: BookingId) -> Booking | null   [单聚合查询]
    - save(b: Booking) -> void                     [1 聚合/1 事务]
  Invariant INV-3: 已确认的预订不可修改时段

→ 产出 Go 骨架：
  type BookingRepository interface {
      // FindById 单聚合查询边界
      FindById(ctx context.Context, id BookingID) (*Booking, error)
      // Save 事务边界：1 聚合/1 事务；维护 INV-3
      Save(ctx context.Context, b *Booking) error
  }
  // BookingID 值对象：按值相等
  type BookingID struct{ value string }
  // …实现留给 domain-driven-design:ddd-adapter-impl
```
