---
description: 启动既有项目 DDD 改造（brownfield 驱动）。先逆向建模与接缝识别，再用绞杀者模式逐切片渐进重构。
argument-hint: <代码路径/模块/仓库> [--goal=解耦|拆分|可测试性] [--lang=沿用现有]
---

# /domain-driven-design:ddd-refactor

你是 DDD 体系的编排者。用户用本命令把一个**已存在、可运行**的系统渐进改造为 DDD 结构。核心纪律：**旧系统全程可运行、可回滚，绝不一次性重写。**

## 参数

- `$ARGUMENTS`：要改造的代码路径 / 模块 / 仓库（必需）。
- `--goal`：改造目标（解耦 / 拆分微服务 / 提升可测试性）。
- `--lang`：落地语言，缺省沿用现有技术栈。

## 你要做的

1. 调用 **`domain-driven-design:ddd-mode-router`** 确认 brownfield，并把改造目标拆成可独立推进的**切片**。
2. 按 **`workflow-brownfield`** 编排前段（逆向）：
   `domain-driven-design:ddd-code-survey → domain-driven-design:ddd-seam-finder → domain-driven-design:ddd-strangler-plan`。
   - `code-survey` 从代码反推现状领域模型与坏味道。
   - `seam-finder` 定位防腐层（ACL）落点与改造接缝。
   - `strangler-plan` 排出迁移批次、回滚策略、需要补的特征化测试。
3. **逐切片** 进入与新建相同的后段：
   `domain-driven-design:ddd-aggregates（局部）→ domain-driven-design:ddd-domain-interactions（局部）→ domain-driven-design:ddd-model-review →（通过）domain-driven-design:ddd-spec-bridge → domain-driven-design:ddd-port-scaffold → domain-driven-design:ddd-adapter-impl → domain-driven-design:ddd-acceptance`。
4. **每个切片落地前先补特征化测试**（characterization test）锁住现有行为，再以接口优先替换实现；切片完成、验收通过后再取下一片。
5. 始终保持旧路径可用，用绞杀者（strangler fig）方式逐步把流量切到新实现。

## 原则

- 不追求一步到位的"完美模型"，追求**每个切片可独立交付、可回滚**。
- 逆向建模得到的"现状模型"要与目标模型对照，差异即改造 backlog。

## 示例

```
/domain-driven-design:ddd-refactor src/order-monolith --goal=拆分 --lang=java
```
