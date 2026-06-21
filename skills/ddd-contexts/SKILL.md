---
name: ddd-contexts
description: "限界上下文与通用语言：把子域落成一组边界清晰的限界上下文，为每个上下文确立独立、自洽的通用语言与词汇表，用 ADR 记录边界决策。当 ddd-subdomains 已完成分类、需要把'同一个词在不同语境含义不同'的问题切开时触发。"
risk: safe
stage: strategic
driver: both
source: self
tags: "[ddd, strategic, bounded-context, ubiquitous-language]"
---

# DDD Contexts（限界上下文 + 通用语言）

战略阶段的核心一步。子域是"问题空间"的划分，**限界上下文（Bounded Context）是"解空间"的边界**——在这条边界内，每个术语只有一个明确含义，模型自洽。本 Skill 的价值在于斩断"通用语言污染"：当"预订"在排期语境和计费语境含义不同时，强行用一个模型会让两边都拧巴。一个上下文 = 一套通用语言 = 一个可独立演进的模型。

## 使用时机

- `domain-driven-design:ddd-subdomains` 已产出子域分类与核心域声明。
- 出现"同一个词不同人理解不一样"或"一个模型想同时服务两类需求"的张力。
- **被回溯触发**：`domain-driven-design:ddd-aggregates` 发现不变量跨多个上下文、或 `domain-driven-design:ddd-context-map` 发现边界划得无法集成时回到这里重划。

## 输入要求

- **必需**：`domain-driven-design:ddd-subdomains` 的子域分类、核心域声明；`domain-driven-design:ddd-discover` 的热点与术语种子。
- **可选**：组织架构（康威定律线索）、现有系统边界、团队划分。

## 流程

1. **从子域映射上下文**：子域与上下文不必 1:1，但以子域为起点。一个核心子域常对应一个核心上下文。
2. **用"语言一致性"切边界**：在哪里同一个术语开始变味（"预订"在排期=时段占用，在计费=一笔费用）？变味处就是边界。**语言断裂线 = 上下文边界**。
3. **为每个上下文建独立词汇表**：同名异义的术语必须在各自上下文里分别定义，并标出它在别处的对应概念（为 `domain-driven-design:ddd-context-map` 的翻译做准备）。
4. **判定上下文性质**：标注每个上下文承载的子域类型（core/supporting/generic），继承投入等级。
5. **写边界 ADR**：每条非显然的边界决策写一条架构决策记录（背景/决策/被否方案/后果），尤其是"为什么把 A、B 分开"或"为什么合并"。
6. **对齐组织（康威）**：检查边界是否与团队归属冲突——长期看系统结构会向组织结构收敛，边界最好顺着团队切。
7. **登记跨界关系待解**：把"上下文之间要交换什么"列为待办，交给 `domain-driven-design:ddd-context-map`。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 上下文目录 | 表格：上下文名、承载子域、性质、一句话职责、负责团队 |
| 通用语言词汇表 | 每上下文一张：术语、本上下文定义、他处对应概念、歧义来源 |
| 边界 ADR | 每条关键边界：背景、决策、被否选项、后果 |
| 跨界关系待解清单 | 表格：上下文 A ↔ B、需要交换的概念、方向 |

## 校验清单

- [ ] 每个上下文内，每个术语只有唯一含义（无同上下文内歧义）
- [ ] `domain-driven-design:ddd-discover` 的同名异义热点都已被边界切开并分别定义
- [ ] 每条非显然边界都有 ADR，记录了被否方案
- [ ] 上下文性质（core/supporting/generic）与子域分类一致
- [ ] 边界与团队归属无硬冲突（或已记录康威风险）
- [ ] 跨界要交换的概念已登记，未遗漏集成需求

## 回溯触发

- 切完发现某核心能力被劈到两个上下文、无法自洽 → 回退 `domain-driven-design:ddd-subdomains` 重分类。
- 词汇表暴露出 `domain-driven-design:ddd-discover` 未澄清的歧义 → 回退 `domain-driven-design:ddd-discover` 补热点澄清。

## 示例

```text
输入：domain-driven-design:ddd-subdomains 的核心域«预订与冲突» + 子域 [审批][通知][身份]。

→ domain-driven-design:ddd-contexts 产出（节选）：

上下文目录：
  | 上下文 | 子域 | 性质 | 职责 | 团队 |
  | Scheduling 排期 | 预订与冲突 | core | 时段占用、冲突检测、预订生命周期 | 预订组 |
  | Approval 审批 | 审批 | supporting | 高价值房审批流与规则 | 预订组 |
  | Notification 通知 | 通知 | generic | 多渠道消息推送 | 平台组 |
  | Identity 身份 | 身份 | generic | 员工身份与角色（接 LDAP）| 平台组 |

通用语言词汇表（节选）：
  Scheduling：
    - 预订(Booking)：对某会议室某时段的占用主张，有生命周期。
    - 时段(Slot)：可被占用的最小时间单元，跨时区统一存 UTC。
  Approval：
    - 预订(BookingRef)：仅是一个待审批对象的引用 ID + 摘要，不含排期细节。
    ⚠ 同名异义：Scheduling 的"预订"是完整聚合，Approval 只认它的引用。

边界 ADR-01：为什么 Scheduling 与 Approval 分开
  背景：审批规则常变、且只作用于部分房间。
  决策：拆为两个上下文，Approval 通过事件订阅 Scheduling。
  被否：合并进 Scheduling —— 会让冲突检测的核心模型被审批规则污染。
  后果：需要一条 Scheduling→Approval 的集成（交给 context-map）。

跨界关系待解：
  | Scheduling → Approval | 传"待审批预订引用" | 出 |
  | Approval → Scheduling | 回传"审批结果" | 入 |
  | Scheduling → Notification | 触发确认/驳回通知 | 出 |

→ 交给 domain-driven-design:ddd-context-map 定集成模式与契约所有权。
```
