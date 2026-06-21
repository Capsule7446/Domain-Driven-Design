# 语言剖面（Language Profiles）

> 供 `domain-driven-design:ddd-port-scaffold` / `domain-driven-design:ddd-adapter-impl` 使用。把语言中立的 DDD 构造映射到各面向接口语言。
> 只要目标语言支持"面向接口编程"，即可纳入。未收录语言用文末"剖面问卷"现场采集 5 项即可。

## 1. 核心构造映射表

| 中立构造 | Java | Kotlin | C# | Go | TypeScript | Python | Rust |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 端口/接口 | `interface` | `interface` | `interface` | `interface` | `interface` | `Protocol` / `ABC` | `trait` |
| 聚合根/实体 | `class` | `class` | `class` | `struct`+方法 | `class` | `class` | `struct`+`impl` |
| 值对象 | `record` | `data class` | `record`/`record struct` | `struct`(值语义) | `readonly` 类型/`Object.freeze` | `@dataclass(frozen=True)` | `struct`+`#[derive(...)]` |
| 不可变性 | `final` 字段 | `val` | `init`-only / `readonly` | 无 setter + 值传递 | `readonly` | `frozen=True` | 默认不可变 |
| 按值相等 | 手写或 `record` 自动 | `data class` 自动 | `record` 自动 | 手写 `Equal()` | 手写或结构比较 | `dataclass` 自动 `__eq__` | `#[derive(PartialEq)]` |
| 标识（ID）类型 | 包装 `record` | `value class` | `readonly record struct` | 命名 `type` | branded type | `NewType`/frozen dc | newtype `struct` |
| 依赖注入 | 构造器注入(+Spring) | 构造器注入 | DI 容器/构造器 | 显式传参/wire | 构造器注入 | `Depends`/构造器 | 显式传参 |
| 错误/空值 | 异常 / `Optional` | 异常 / `Result` 库 | 异常 / `Result<T>` | `error` 返回值 | 异常 / `Result` 联合 | 异常 / `Optional` | `Result` / `Option` |
| 领域事件 | 不可变 `record` | `data class` | `record` | `struct` | `readonly` 类型 | frozen dc | `struct` |
| 仓储接口位置 | 领域包 | 领域包 | 领域程序集 | 领域包 | 领域模块 | 领域包 | 领域 crate |

## 2. 依赖规则在各语言的落点

所有语言统一遵守：**领域内核零外部依赖；基础设施实现领域定义的接口**。差异只在物理组织方式：

- **Java/Kotlin**：领域接口在 `domain` 包，实现在 `infrastructure` 包，靠包/模块边界 + 构造器注入隔离。
- **C#**：领域在独立 `Domain` 程序集（不引用任何基础设施程序集），实现放 `Infrastructure`。
- **Go**：领域包定义 `interface`，基础设施包实现；用 `internal/` 防外部误引。
- **TypeScript**：领域模块只导出接口与领域类型，适配器在 `infrastructure/`；用 path/lint 规则禁止反向 import。
- **Python**：领域用 `Protocol`/`ABC` 定义端口，基础设施实现；DI 用构造器或 `Depends`。
- **Rust**：领域 crate 定义 `trait`，基础设施 crate 实现；编译期保证依赖方向。

## 3. 一个端口在 7 种语言的样子

中立契约：

```
Port: OrderRepository
  - findById(id: OrderId) -> Order | null     [单聚合查询]
  - save(order: Order) -> void                [1 聚合/1 事务]
```

| 语言 | 骨架（仅签名） |
| :--- | :--- |
| Java | `interface OrderRepository { Optional<Order> findById(OrderId id); void save(Order order); }` |
| Kotlin | `interface OrderRepository { fun findById(id: OrderId): Order?; fun save(order: Order) }` |
| C# | `interface IOrderRepository { Order? FindById(OrderId id); void Save(Order order); }` |
| Go | `type OrderRepository interface { FindById(ctx context.Context, id OrderID) (*Order, error); Save(ctx context.Context, o *Order) error }` |
| TypeScript | `interface OrderRepository { findById(id: OrderId): Promise<Order \| null>; save(order: Order): Promise<void>; }` |
| Python | `class OrderRepository(Protocol): def find_by_id(self, id: OrderId) -> Order \| None: ...; def save(self, order: Order) -> None: ...` |
| Rust | `trait OrderRepository { fn find_by_id(&self, id: OrderId) -> Option<Order>; fn save(&self, order: &Order) -> Result<(), Error>; }` |

> 注意：契约语义（单聚合查询、1 聚合/1 事务、不变量编号）以注释保留在每种语言里，不随语言丢失。

## 4. 不变量强制与错误表达（落地最易跑偏处）

聚合根方法要在**内部强制不变量**，非法操作必须被拒绝。各语言的惯用表达：

| 语言 | 拒绝非法操作的惯用法 | 空值/缺失 |
| :--- | :--- | :--- |
| Java | 抛领域异常（`throw new DomainException`）/ 返回 `Optional` | `Optional<T>` |
| Kotlin | `require/check` + 领域异常 / `Result` | `T?` |
| C# | 抛领域异常 / `Result<T>` | `T?` |
| Go | 返回 `(T, error)`，error 为领域错误值 | 返回 `nil` + 哨兵 error |
| TypeScript | 抛领域错误类 / 返回 `Result<T,E>` 联合 | `T \| null` |
| Python | 抛领域异常 / 返回 `Result` | `Optional[T]` / `T \| None` |
| Rus