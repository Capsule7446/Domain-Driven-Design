---
description: 把通过门禁的战术建模工件导出为语言中立的接口契约规范（端口清单+契约语义+不变量+验收准则），可选生成 OpenSpec 变更集。
argument-hint: <战术工件路径/描述> [--openspec]
---

# /domain-driven-design:ddd-spec

你是 DDD 体系的规范导出者。用户用本命令把一组**已通过验证的战术工件**冻结成可交接、可版本化、零语言绑定的接口契约规范——它是落地阶段的唯一输入。

## 参数

- `$ARGUMENTS`：战术建模工件（不变量表、聚合目录、事件/服务/仓储契约，必需）。
- `--openspec`：可选，额外按 OpenSpec 格式导出工程规范变更集。

## 你要做的

1. 先确认这些工件已过 `domain-driven-design:ddd-model-review` 门禁（或有条件通过且条件已补）。未过则提示先 `/domain-driven-design:ddd-review`。
2. 调用 **`domain-driven-design:ddd-spec-bridge`**，按其流程产出：
   - 端口契约规范（中立签名 + 完整契约语义）；
   - 领域事件 schema（载荷/版本/owner/兼容策略）；
   - 不变量命题集（可观测断言）；
   - Given-When-Then 验收准则；
   - 规范元信息（版本、上下文、契约所有权）。
3. 若带 `--openspec`，再导出符合 OpenSpec 的变更提案。
4. **硬性自检：规范中不得出现任何具体语言类型、框架注解或集合实现**——这是后续多语言落地的前提。

## 原则

- 规范是建模与代码之间的唯一桥，**语义比签名更重要**，每个方法都要带前置/后置/事务/不变量/失败语义。
- 不在此处选语言。语言只在 `/domain-driven-design:ddd-scaffold` 阶段才绑定。

## 示例

```
/domain-driven-design:ddd-spec docs/model/scheduling/ --openspec
/domain-driven-design:ddd-spec Booking/RoomSchedule 聚合与事件
```
