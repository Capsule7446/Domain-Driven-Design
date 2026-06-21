---
name: ddd-mode-router
description: "DDD 流程入口路由：判定是 0→1 新建还是既有项目改造，采集上下文，选择正确的起始 Skill 与 Workflow。当用户说要用 DDD 做开发、重构、引入领域模型时触发。"
risk: safe
stage: router
driver: both
source: self
tags: "[ddd, router, entry]"
---

# DDD Mode Router

体系的唯一统一入口。它本身不建模、不写代码，只负责**判定驱动类型、补齐最小上下文、把控制权交给正确的下游 Skill 或 Workflow**。

## 使用时机

- 用户表达"想用 DDD 做/改造某系统"但未指明从哪开始。
- `/domain-driven-design:ddd-new` 或 `/domain-driven-design:ddd-refactor` 命令被触发后的第一步。
- 当一次会话里出现既有代码与新需求混杂，需要先分流时。

## 输入要求

- **必需**：用户的目标描述（自然语言即可：要做什么 / 要改什么）。
- **可选**：是否已有代码库（路径或仓库）、已有的建模工件、技术约束、目标落地语言。

## 流程

1. **判定驱动类型**：依据下表二选一（或拆分为多切片分别归类）。
   - 有可运行的既有代码、目标是"渐进引入 DDD/解耦/拆分" → **brownfield（改造）**。
   - 无既有实现或愿意重写、目标是"从需求新建" → **greenfield（0→1）**。
2. **采集最小上下文**：按驱动类型补齐进入下游所需的最小信息（见"上下文门禁"）。
3. **选择入口 Skill**：
   - greenfield → 默认 `domain-driven-design:ddd-scope`；若需求已清晰可跳到 `domain-driven-design:ddd-discover`。
   - brownfield → 默认 `domain-driven-design:ddd-code-survey`；若已有现状模型可跳到 `domain-driven-design:ddd-seam-finder`。
   - 仅体检既有模型 → `domain-driven-design:ddd-model-review`；仅导规范 → `domain-driven-design:ddd-spec-bridge`。
4. **登记目标语言**：记录落地语言（默认延后到阶段 VI 再定），供 `domain-driven-design:ddd-port-scaffold` 使用。
5. **输出路由决策卡**，移交下游；不替下游做实质建模。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 驱动判定 | greenfield / brownfield / 混合（列出各切片归类）+ 判定理由 |
| 上下文清单 | 表格：已知项、缺失项、阻塞性缺失（必须补） |
| 路由决策卡 | 入口 Skill、对应 Workflow、目标落地语言（或"延后"）、跳过的阶段及理由 |

## 校验清单

- [ ] 驱动类型已明确判定（混合场景已按切片拆分）
- [ ] 阻塞性缺失信息已向用户问清，未"猜"关键边界
- [ ] 入口 Skill 与所选 Workflow 一致
- [ ] 若选择跳过某阶段，已说明可跳过的前提条件成立

## 回溯触发

- 入口为 `domain-driven-design:ddd-discover` 但发现需求边界其实模糊 → 回退到 `domain-driven-design:ddd-scope`。
- 入口为 `domain-driven-design:ddd-seam-finder` 但现状模型不足以定位接缝 → 回退到 `domain-driven-design:ddd-code-survey`。

## 上下文门禁（最小上下文）

| 驱动 | 进入下游前必须具备 |
| :--- | :--- |
| greenfield | 一句可被业务方认可的问题陈述；关键参与方；硬约束（合规/时限/团队） |
| brownfield | 代码库可访问路径；改造目标（解耦/拆分/可测试性）；不可中断的运行约束 |

## 示例

```text
/domain-driven-design:ddd-refactor 我们的单体订单系统耦合严重，想逐步拆出履约和计费两块，用 DDD 重构，但线上不能停。

→ 驱动判定：brownfield（混合切片：履约、计费各一）
→ 入口：domain-driven-design:ddd-code-survey
→ Workflow：workflow-brownfield
→ 落地语言：延后（沿用现有 Java 栈，阶段 VI 取 java 剖面）
```
