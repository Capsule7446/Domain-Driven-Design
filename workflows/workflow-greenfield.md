# Workflow：Greenfield（0→1 新建）

> 驱动 A。从模糊需求到接口优先的可运行代码。由 `/domain-driven-design:ddd-new` 触发，以 `domain-driven-design:ddd-mode-router` 开场。
> 配套：`README.md`（体系蓝图与流程图）。
> 命名空间：文中 `domain-driven-design:<skill>` 为插件内 Skill 的完整调用名（详见 README「命名空间约定」）。

## 1. 链路

```
domain-driven-design:ddd-mode-router
   → domain-driven-design:ddd-scope            （收敛范围）
   → domain-driven-design:ddd-discover         （事件风暴）
   → domain-driven-design:ddd-subdomains       （子域分类）
   → domain-driven-design:ddd-contexts         （限界上下文 + 通用语言）
   → domain-driven-design:ddd-context-map      （集成模式）★G1
   → domain-driven-design:ddd-aggregates       （不变量 → 聚合）
   → domain-driven-design:ddd-domain-interactions （事件/服务/仓储/工厂，接口语义化）
   → domain-driven-design:ddd-model-review     （质量门禁）★G2
   → domain-driven-design:ddd-spec-bridge      （语言中立接口契约规范）
   → domain-driven-design:ddd-port-scaffold    （按语言剖面生成接口骨架）★G3
   → domain-driven-design:ddd-adapter-impl     （接口背后实现领域逻辑 + 适配器）
   → domain-driven-design:ddd-acceptance       （不变量/契约/用例验收）
```

## 2. 阶段工件衔接契约（上一站产出 = 下一站输入）

这是"工件可串联"的硬约束：每一站的输出表必须满足下一站的输入要求，否则不放行。

| 阶段 | 关键输入（来自上游）| 关键产出（喂给下游）|
| :--- | :--- | :--- |
| domain-driven-design:ddd-scope | 用户诉求 | 问题陈述、目标/非目标、术语种子、风险 |
| domain-driven-design:ddd-discover | 问题陈述 + 术语种子 | 事件流、命令/事件目录、策略、热点、泳道 |
| domain-driven-design:ddd-subdomains | 事件流 + 泳道 + 价值主张 | 能力清单、子域分类、核心域声明 |
| domain-driven-design:ddd-contexts | 子域分类 + 热点 + 术语种子 | 上下文目录、各上下文词汇表、边界 ADR |
| domain-driven-design:ddd-context-map | 上下文目录 + 跨界待解 | 关系矩阵、集成模式、契约所有权、一致性语义 |
| domain-driven-design:ddd-aggregates | 词汇表 + 策略 + 一致性语义 | 不变量表、聚合目录、事务边界、一致性策略 |
| domain-driven-design:ddd-domain-interactions | 聚合目录 + 不变量 + 契约所有权 | 事件契约、领域服务/仓储/工厂接口（中立）|
| domain-driven-design:ddd-model-review | 全部战略+战术工件 | 五维评分、问题清单、回溯指令、门禁结论 |
| domain-driven-design:ddd-spec-bridge | 通过门禁的战术工件 | 端口契约规范、事件 schema、不变量命题、验收准则 |
| domain-driven-design:ddd-port-scaffold | 端口契约规范 + 语言剖面 | 目标语言接口/值对象骨架、待实现清单 |
| domain-driven-design:ddd-adapter-impl | 接口骨架 + 契约语义 | 领域实现 + 适配器 + 依赖装配 |
| domain-driven-design:ddd-acceptance | 实现 + 不变量命题 + 验收准则 | 测试集 + 验收报告 + 交付结论 |

## 3. 阶段门禁（gate）

每个 Skill 的"校验清单"全过才放行。三个**强门禁**额外要求停下与用户确认：

| 门禁 | 位置 | 放行条件 |
| :--- | :--- | :--- |
| G1 战略门禁 | `domain-driven-design:ddd-context-map` 后 | 上下文边界与契约所有权获用户确认；核心域已声明且每个核心上下文有 ACL/隔离 |
| G2 模型门禁 | `domain-driven-design:ddd-model-review` 后 | 不变量表达率 ≥ 60%；无阻断级一致性问题；目标全覆盖；无未决回溯项 |
| G3 落地门禁 | `domain-driven-design:ddd-port-scaffold` 后 | 端口契约可在目标语言完整表达；依赖只向内；骨架零实现 |

## 4. 完整回溯矩阵

| 触发条件 | 在哪发现 | 回退到 | 修复动作 |
| :--- | :--- | :--- | :--- |
| 价值主张不清 / 目标自相矛盾 | domain-driven-design:ddd-subdomains, domain-driven-design:ddd-discover | domain-driven-design:ddd-scope | 重述问题与价值 |
| 事件流与目标矛盾 / 热点纠缠 | domain-driven-design:ddd-discover | domain-driven-design:ddd-scope | 重划范围 |
| 能力无法归类（缺事件）| domain-driven-design:ddd-subdomains | domain-driven-design:ddd-discover | 补事件风暴 |
| 找不到/超过 3 个核心域 | domain-driven-design:ddd-subdomains | domain-driven-design:ddd-scope | 收敛价值主张 |
| 核心能力被劈到两上下文 | domain-driven-design:ddd-contexts | domain-driven-design:ddd-subdomains | 重分类子域 |
| 同名异义未被边界切开 | domain-driven-design:ddd-contexts | domain-driven-design:ddd-discover | 补热点澄清 |
| 关系无论如何都强耦合 | domain-driven-design:ddd-context-map | domain-driven-design:ddd-contexts | 重划边界 |
| 不变量需跨上下文强制 | domain-driven-design:ddd-aggregates | domain-driven-design:ddd-contexts / domain-driven-design:ddd-context-map | 调整边界/一致性语义 |
| 列不变量时缺触发事件 | domain-driven-design:ddd-aggregates | domain-driven-design:ddd-discover | 补事件 |
| 事件需携带他聚合私有数据 | domain-driven-design:ddd-domain-interactions | domain-driven-design:ddd-aggregates | 重审聚合边界 |
| 不变量表达率 < 60% / 数据袋聚合 | domain-driven-design:ddd-model-review | domain-driven-design:ddd-aggregates | 按不变量重切聚合 |
| 某目标无建模覆盖 | domain-driven-design:ddd-model-review | domain-driven-design:ddd-discover / domain-driven-design:ddd-scope | 补事件或补目标 |
| 契约语义无法中立化 | domain-driven-design:ddd-spec-bridge | domain-driven-design:ddd-domain-interactions | 补契约语义 |
| 不变量无法写成可观测断言 | domain-driven-design:ddd-spec-bridge | domain-driven-design:ddd-aggregates | 澄清规则 |
| 端口契约目标语言无法表达 | domain-driven-design:ddd-port-scaffold | domain-driven-design:ddd-spec-bridge | 调整契约表达 |
| 端口跨多聚合私有数据 | domain-driven-design:ddd-port-scaffold | domain-driven-design:ddd-aggregates | 重审边界 |
| 不变量无法在单聚合内强制 | domain-driven-design:ddd-adapter-impl | domain-driven-design:ddd-aggregates | 重切聚合 |
| 反复并发竞态 | domain-driven-design:ddd-acceptance | domain-driven-design:ddd-aggregates | 复核事务/聚合边界 |

## 5. 端到端走查示例（会议室预订）

```
/domain-driven-design:ddd-new 会议室预订系统，支持冲突检测和审批 --lang=ts

router   → greenfield；落地语言 ts（延后到 G3 确认）
scope    → 问题：多人抢同一时段同一房；非目标：访客签到/餐饮/租赁
discover → 事件流：预订已提交→冲突已检出→确认/待审批→审批已决→生效
subdomain→ 核心域«预订与冲突»；支撑«审批»；通用«通知/身份»
contexts → Scheduling / Approval / Notification / Identity；切开 Booking 同名异义
ctx-map  → Scheduling↔Approval 事件订阅(最终一致)；Identity 用 ACL  ── G1 用户确认 ✅
aggregates→ INV-1 房×时段不双占(RoomSchedule)；INV-2/3 预订状态(Booking)；1聚合/1事务
interact → 事件 BookingSubmitted/ApprovalDecided；端口 BookingRepository/ConflictPolicy（中立）
review   → 发现"取消已生效预订"缺事件链 → 回退 discover 补 → 复评 ── G2 通过 ✅
spec     → 端口契约规范 v1.0（零语言绑定）+ INV 命题 + AC
scaffold → 取 ts 语言剖面，生成 interface + branded type 骨架 ── G3 通过 ✅
impl     → 领域核心实现 INV；仓储/LDAP 适配器；DI 在最外层装配
accept   → INV-1 并发双占测试、端口契约测试、AC 用例全过 → 可交付
```

## 6. 编排纪律

- **非一口气跑完**：在 G1/G2 处停下与用户确认关键工件（子域分类、上下文边界、聚合边界）。
- **工件即输入**：严格按 §2 的衔接契约传递结构化产出，不在阶段间丢信息。
- **语言延后绑定**：落地语言在 G3（`domain-driven-design:ddd-port-scaffold`）前才需确定，前段全程语言无关。
- **回溯是常态**：按 §4 矩阵触发回溯不是失败，是模型在收敛；记录每次回溯及其原因。
