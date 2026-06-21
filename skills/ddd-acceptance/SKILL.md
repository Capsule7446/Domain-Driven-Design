---
name: ddd-acceptance
description: "面向接口的验收：把规范里的不变量命题与验收准则落成可执行测试——不变量测试、端口契约测试、用例验收测试，验证实现忠于契约且不变量恒真。当 ddd-adapter-impl 已产出实现、需要在交付/切流量前验收时触发，是落地三步的最后一步。"
risk: caution
stage: implementation
driver: both
source: self
tags: "[ddd, implementation, testing, acceptance, contract-test]"
---

# DDD Acceptance（面向接口的验收）

落地三步的最后一步，也是**门禁的最终执行者**。它不验"代码跑没跑"，而验"实现是否忠于 `domain-driven-design:ddd-spec-bridge` 的契约、不变量是否在各种路径下恒真"。因为验证锚定在**语言中立的契约**上而非具体实现细节，所以同一套验收意图可在任何目标语言复用——换语言只换测试的语法外壳，验收命题不变。在改造链路里，它还是"切流量"的放行闸。

## 使用时机

- `domain-driven-design:ddd-adapter-impl` 已产出领域实现与适配器。
- 交付前 / 改造切片切流量前的强制验收。
- 需要为端口契约建立长期回归防线时。

## 输入要求

- **必需**：`domain-driven-design:ddd-spec-bridge` 的不变量命题集与 Given-When-Then 验收准则；`domain-driven-design:ddd-adapter-impl` 的实现与依赖装配。
- **可选**：brownfield 切片的**特征化测试**（characterization test，锁住旧行为，作对照基线）；性能/并发验收要求。

## 流程

1. **不变量测试**：每条 INV 命题写一组测试，覆盖正例（保持为真）与反例（非法操作被拒绝），尤其测边界与并发竞态（如同时 reserve 同一 slot）。
2. **端口契约测试**：针对每个端口接口写**与实现无关**的契约测试，验证前置/后置条件、查询边界、失败/空值语义。同一契约测试应能套在该端口的任何适配器实现上（便于换实现/换语言）。
3. **用例验收测试**：把规范的 Given-When-Then 准则逐条落成端到端用例测试，走应用服务编排路径。
4. **事件与最终一致验收**：验证最终一致路径——事件发布、订阅处理、补偿逻辑在故障注入下仍收敛到一致状态。
5. **依赖方向回归**：加一条架构测试/静态检查，断言领域核心不依赖基础设施（防止后续腐化）。
6. **（brownfield）行为等价验收**：用特征化测试对照新旧实现，确认对外可观测行为一致后方可切流量。
7. **汇总验收报告**：列出每条 INV/AC 的覆盖与通过状态，给出"可交付/可切流量"结论或阻断项。

## 输出

| 工件 | 结构要求 |
| :--- | :--- |
| 不变量测试集 | 每条 INV：正例+反例+边界/并发用例，标注 INV 编号 |
| 端口契约测试 | 与实现无关；可复用于该端口的任何适配器 |
| 用例验收测试 | 由 Given-When-Then 准则一一落成 |
| 最终一致验收 | 事件/补偿在故障注入下的收敛验证 |
| 架构依赖测试 | 断言领域核心零基础设施依赖 |
| 验收报告 | 每条 INV/AC 覆盖与通过状态 + 交付/切流量结论 |

## 校验清单

- [ ] 每条不变量都有正例 + 反例测试，关键并发竞态已覆盖
- [ ] 每个端口都有与实现无关的契约测试（可换适配器复用）
- [ ] `domain-driven-design:ddd-spec-bridge` 的每条 Given-When-Then 都有对应用例测试
- [ ] 最终一致路径在故障注入下验证收敛，补偿逻辑被测
- [ ] 架构依赖测试在 CI 中持续守护依赖方向
- [ ] （brownfield）特征化测试确认新旧行为等价后才放行切流量
- [ ] 验收报告对每条 INV/AC 给出明确通过/阻断结论

## 回溯触发

- 不变量测试反例无法被实现拒绝 → 回退 `domain-driven-design:ddd-adapter-impl`（实现未守不变量）。
- 契约测试暴露端口语义不足以验证 → 回退 `domain-driven-design:ddd-spec-bridge`。
- 反复出现的竞态说明事务/聚合边界有问题 → 回退 `domain-driven-design:ddd-aggregates`。
- （brownfield）特征化测试与新实现行为不等价且非预期 → 回退 `domain-driven-design:ddd-code-survey`（旧行为理解不足）。

## 示例

```text
输入：Booking/RoomSchedule 的 Go 实现 + 规范的 INV 命题与 AC。

→ domain-driven-design:ddd-acceptance 产出（节选）：

不变量测试（INV-1 防双占）：
  正例: 空闲 slot → Reserve 成功，发 SlotReserved
  反例: 已占 slot → Reserve 返回 ErrSlotConflict，占用集合不变
  并发: 两 goroutine 同时 Reserve 同一 slot → 恰一个成功（乐观锁/串行化保证）

端口契约测试（RoomScheduleRepository，实现无关）：
  - save 后 findByRoom 能取回等价聚合（往返一致）
  - 并发 save 同一聚合的旧版本 → ConcurrencyError（后置语义）
  套用对象: 内存实现 + Pg 实现 都必须通过同一套契约测试 ✅

用例验收（AC-1）:
  Given 09:00-10:00 已被 B1 占
  When  B2 提交占同段
  Then  B2 转 Rejected，B1 不受影响，产生 SlotReserveRejected   → PASS

最终一致验收:
  注入"占用成功但 Booking 落库失败" → 补偿触发释放 slot，系统收敛 → PASS

架构依赖测试:
  断言 domain 包导入图不含 infrastructure/* → PASS

验收报告：INV-1/2/3 全覆盖通过；AC-1..AC-5 全通过 → 结论：可交付 / 可切流量。
```
