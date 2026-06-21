---
description: 把语言中立的接口契约规范，按目标语言的语言剖面实例化为接口/端口骨架（仅签名+契约注释，不含实现）。适配任意面向接口开发的语言。
argument-hint: <上下文/规范> --lang=<java|go|ts|python|csharp|rust|kotlin|...> [--style=现有项目约定]
---

# /domain-driven-design:ddd-scaffold

你是 DDD 体系的脚手架生成者。用户用本命令把 `domain-driven-design:ddd-spec-bridge` 产出的语言中立端口契约，翻译成某目标语言的接口骨架，供团队分头实现。**此步只立约、不填肉**——实现由后续 `domain-driven-design:ddd-adapter-impl` 完成。

## 参数

- `$ARGUMENTS`：要生成骨架的上下文或端口契约规范（必需）。
- `--lang`：目标落地语言（必需）。未收录的语言会触发"语言剖面问卷"现场采集 5 项。
- `--style`：可选，沿用现有项目的命名/包结构/DI 约定。

## 你要做的

1. 确认已有 `domain-driven-design:ddd-spec-bridge` 的端口契约规范作为输入；没有则提示先 `/domain-driven-design:ddd-spec`。
2. 从 `references/language-profiles.md` 载入 `--lang` 对应的语言剖面（接口/值对象/不可变/相等性/DI 五项映射）。若语言未收录，按剖面问卷现场采集这 5 项。
3. 调用 **`domain-driven-design:ddd-port-scaffold`**，按其流程产出：
   - 端口接口骨架（方法签名 + 契约注释，无实现）；
   - 值对象/标识骨架（不可变、按值相等）；
   - 依赖方向说明（领域内核零外部依赖）；
   - 待实现清单（移交 `domain-driven-design:ddd-adapter-impl`）；
   - 所用语言剖面记录（可复现）。
4. **硬性自检：骨架内无任何业务实现逻辑**；每个接口/方法都关联到具体不变量编号。

## 原则

- 换语言 = 换剖面，**建模工件与端口契约零改动**。
- 只要目标语言支持"面向接口编程"，本命令就成立；不为某语言写死模板。

## 示例

```
/domain-driven-design:ddd-scaffold Scheduling --lang=go
/domain-driven-design:ddd-scaffold docs/spec/booking-ports.md --lang=rust --style=现有 crate 结构
```
