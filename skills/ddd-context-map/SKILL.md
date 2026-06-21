---
name: ddd-context-map
description: "上下文映射：刻画限界上下文之间的关系与集成模式（ACL/OHS/PL、Customer-Supplier、Conformist、Shared Kernel、Partnership），明确契约所有权与团队权力关系。当 ddd-contexts 已划定多个上下文、需要决定它们如何集成与谁说了算时触发。"
risk: safe
stage: strategic
driver: both
source: self
tags: "[ddd, strategic, context-map, integration]"
---

# DDD Context Map（上下文映射）

战略阶段收尾。上下文划好之后，**真正的复杂度在它们之间的缝**。本 Skill 用 DDD 的标准关系语汇，把"谁依赖谁、谁的契约说了算、用什么模式翻译"显性化。它防的是两类典型灾难：核心上下文被上游模型同化（应上防腐层）、以及集成靠口头约定导致契约漂移（应明确所有权）。

## 使用时机

- `domain-driven-design:ddd-contexts` 已产出上下文目录与跨界关系待解清单。
- 需要确定每条跨界关系的集成模式、契约归属与防腐策略。
- **被回溯触发**：`domain-driven-design:ddd-domain-interactions` 发现某集成需要的事件携带他上下文私有数据、或 `domain-driven-design:ddd-port-scaffold` 发现契约无处落时回到这里。

## 输入要求

- **必需**：`domain-driven-design:ddd-contexts` 的上下文目录、词汇表、跨界关系待解清单。
- **可选**：组织权力关系（哪个团队强势）、现有系统的既成集成、外部系统契约。

## 流程

1. **列关系矩阵**：把所有上下文两两之间存在交互的填进矩阵，标出方向（谁调用谁/谁订阅谁）。
2. **判定权力关系**：对每条关系定性——
   - **Customer-Supplier**：下游能影响上游排期。
   - **Conformist**：下游只能顺从上游模型（无话语权）。
   - **Partnership**：双方共进退、共担失败。
   - **Shared Kernel**：共享一小块模型/代码（慎用，强耦合）。
3. **选集成模式**：对每条关系选落地模式——
   - **ACL 防腐层**：保护本上下文模型不被外部概念污染（**核心上下文对外几乎都该上 ACL**）。
   - **OHS 开放主机服务 + PL 发布语言**：上游对多个下游提供稳定公开契约。
   - **事件订阅**：异步、最终一致的解耦集成。
4. **指派契约所有权**：每条集成契约（API/事件 schema）必须有唯一 owner 上下文，负责版本与兼容。
5. **标一致性语义**：每条跨界交互是强一致（同步事务）还是最终一致（事件）？为 `domain-driven-design:ddd-aggregates` 的事务边界与 `domain-driven-design:ddd-spec-bridge` 的契约语义打底。
6. **画上下文映射图**：用一张图汇总所有上下文、关系类型、集成模式、ACL 位置。
7. **核对核心域保护**：确认每个核心上下文对所有上游都有 ACL 或等价隔离。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 关系矩阵 | 上下文 × 上下文：方向 + 关系类型（CS/Conformist/Partnership/Shared Kernel）|
| 集成模式表 | 表格：关系、模式（ACL/OHS+PL/事件订阅）、同步还是异步、一致性语义 |
| 契约所有权表 | 表格：契约（API/事件）、owner 上下文、消费方、版本策略 |
| 上下文映射图 | 文本/Mermaid：节点=上下文，边=关系类型 + 集成模式，标出 ACL 落点 |
| 核心保护核对 | 每个核心上下文的上游隔离方式确认 |

## 校验清单

- [ ] 每条跨界关系都定了权力类型与集成模式
- [ ] 每个核心上下文对外部依赖都有 ACL 或等价隔离
- [ ] 每条契约有且仅有一个 owner 上下文
- [ ] 每条交互标了同步/异步与一致性语义（强/最终）
- [ ] 慎用 Shared Kernel：若用了，已论证耦合可控且团队是 Partnership
- [ ] 集成模式与 `domain-driven-design:ddd-contexts` 的团队归属/康威约束自洽

## 回溯触发

- 某关系无论怎么映射都强耦合到无法独立演进 → 回退 `domain-driven-design:ddd-contexts` 重划边界。
- 集成需要交换的数据其实属于第三个未识别的概念 → 回退 `domain-driven-design:ddd-discover`/`domain-driven-design:ddd-contexts`。

## 示例

```text
输入：domain-driven-design:ddd-contexts 的 [Scheduling][Approval][Notification][Identity] + 跨界待解清单。

→ domain-driven-design:ddd-context-map 产出（节选）：

关系矩阵：
  | 上游＼下游 | Scheduling | Approval | Notification | Identity |
  | Scheduling | — | 发起(事件) | 触发(事件) | 消费身份 |
  | Approval | 回传结果(事件) | — | 触发(事件) | 消费身份 |
  | Identity(LDAP) | 供身份 | 供身份 | — | — |

集成模式表：
  | 关系 | 模式 | 同步/异步 | 一致性 |
  | Scheduling→Approval | 事件订阅（预订待审批）| 异步 | 最终一致 |
  | Approval→Scheduling | 事件订阅（审批结果）| 异步 | 最终一致 |
  | *→Notification | OHS+PL（通知发布语言）| 异步 | 最终一致 |
  | *→Identity | ACL 包 LDAP | 同步 | 强一致（读）|

契约所有权：
  | 事件 BookingPendingApproval | owner=Scheduling | 消费=Approval | 向后兼容加字段 |
  | 事件 ApprovalDecided | owner=Approval | 消费=Scheduling | 同上 |
  | 通知发布语言 NotifyMessage | owner=Notification(OHS) | 消费=全体 | 版本化 |

上下文映射图（Mermaid 摘要）：
  Identity --ACL--> Scheduling
  Scheduling --事件--> Approval --事件--> Scheduling   (双向，最终一致)
  Scheduling/Approval --OHS--> Notification

核心保护核对：
  Scheduling(core)：对 Identity 用 ACL；对 Approval/Notification 仅发/收事件，
  外部概念不进核心模型。✅

→ 交给 domain-driven-design:ddd-aggregates 在各上下文内设计聚合与事务边界。
```
