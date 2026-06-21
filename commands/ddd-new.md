---
description: 启动 DDD 0→1 新建全链路（greenfield 驱动）。从模糊需求一路推到接口优先的可运行代码。
argument-hint: <业务/问题描述> [--lang=java|go|ts|python|csharp|rust|kotlin]
---

# /domain-driven-design:ddd-new

你是 DDD 体系的编排者。用户用本命令从零启动一个新系统/新模块的领域建模与落地。

## 参数

- `$ARGUMENTS`：业务或问题描述（必需）。
- `--lang`：目标落地语言（可选，缺省则延后到阶段 VI 再确认）。

## 你要做的

1. 先调用 **`domain-driven-design:ddd-mode-router`** 确认这确实是 greenfield；若描述里其实有既有代码，提示改用 `/domain-driven-design:ddd-refactor`。
2. 按 **`workflow-greenfield`** 编排，依次推进：
   `domain-driven-design:ddd-scope → domain-driven-design:ddd-discover → domain-driven-design:ddd-subdomains → domain-driven-design:ddd-contexts → domain-driven-design:ddd-context-map → domain-driven-design:ddd-aggregates → domain-driven-design:ddd-domain-interactions → domain-driven-design:ddd-model-review →（门禁通过）domain-driven-design:ddd-spec-bridge → domain-driven-design:ddd-port-scaffold → domain-driven-design:ddd-adapter-impl → domain-driven-design:ddd-acceptance`。
3. **每个 Skill 的"校验清单"是硬门禁**：不全过不进入下一阶段。`domain-driven-design:ddd-model-review` 报告的回溯条件触发时，按 workflow 的回溯矩阵退回对应上游。
4. 阶段之间**停下来与用户确认关键工件**（子域分类、上下文边界、聚合边界），不要一口气跑完。
5. 落地阶段读取 `--lang`（或此时询问），交给 `domain-driven-design:ddd-port-scaffold` 的语言剖面。

## 原则

- 命令本身不做建模判断，只负责编排与门禁；实质工作都在 Skill 里。
- 始终用通用语言（ubiquitous language）与用户沟通，避免技术黑话。

## 示例

```
/domain-driven-design:ddd-new 我们要做一个会议室预订系统，支持冲突检测和审批 --lang=ts
```
