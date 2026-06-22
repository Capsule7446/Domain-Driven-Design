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
| Rust | 构造期校验返回 `Result<Self, DomainError>`；方法以 `Result<_, DomainError>` 拒绝；`panic!` 仅留给"不可能违反"的内部断言 | `Option<T>` |

> 统一纪律：**非法状态不可构造**——尽量把校验放到值对象/聚合的构造入口（返回 `Result`/抛异常/`require`），让"已存在的对象 = 已满足不变量"。错误要翻译成**领域语义错误**（`OrderAlreadyConfirmed`），不外泄底层异常（`SQLException`、`null`）。

## 5. 未收录语言：剖面问卷（现场采集 5 项）

任何支持"面向接口编程"的语言都能纳入。对上表未收录的语言，问下面 5 个问题，答完即得一行剖面（恰好对应 §1/§4 各列），交给 `domain-driven-design:ddd-port-scaffold` 直接生成骨架——**建模工件零改动，换语言 = 换这 5 个答案**。

| # | 采集项 | 要回答的问题 | 它决定 |
| :--- | :--- | :--- | :--- |
| 1 | **端口/接口** | 用什么声明"只有签名、无实现"的纯抽象契约？ | 端口怎么写（§1 端口行、§3 骨架） |
| 2 | **值对象（不可变 + 按值相等）** | 如何定义"创建后不可变、按值比较相等"的小类型？ | 值对象/领域事件怎么写（§1 值对象/相等行） |
| 3 | **标识类型（防基本类型偏执）** | 如何把 `OrderId` 做成与 `string`/`int` **不可混用**的独立类型？ | ID 类型怎么写（§1 标识行） |
| 4 | **依赖注入** | 领域如何拿到端口的实现？关键：领域**绝不 `new` 具体实现**。 | 依赖方向怎么落（§1 DI 行、§2） |
| 5 | **错误与缺失** | 非法操作怎么拒绝、缺失值怎么表达？异常还是 `Result`？null 还是 `Option`？ | 不变量强制与错误怎么写（§4） |

补充确认两点：**领域内核能否零外部依赖地组织**（独立 module/包/crate）、**有无编译期或 lint 手段守护依赖方向**——这是 §2 的落点。

### 走查示例：把 Swift 纳入（未收录语言）

| 采集项 | Swift 答案 |
| :--- | :--- |
| 1 端口/接口 | `protocol` |
| 2 值对象 | `struct`（值语义）+ 全 `let` 字段；遵从 `Equatable` 自动按值相等 |
| 3 标识类型 | 包装 `struct OrderId: Hashable { let value: UUID }`（或 `RawRepresentable`） |
| 4 依赖注入 | 构造器注入（`init(repo: OrderRepository)`）；组合根在最外层装配 |
| 5 错误/缺失 | `throws` + 领域 `Error`（或 `Result`）；缺失用 `Optional`（`Order?`） |

得到的端口骨架（对照 §3 同一契约）：

```swift
protocol OrderRepository {
    func findById(_ id: OrderId) throws -> Order?   // 单聚合查询
    func save(_ order: Order) throws                // 1 聚合 / 1 事务
}
```

依赖规则落点（§2 对应）：领域放独立 Swift Package target，只导出 `protocol` 与领域类型；基础设施 target 实现并 `import` 领域，领域**不反向依赖**——靠 package targets 的可见性边界 + 构造器注入隔离。契约语义（单聚合、1 聚合/1 事务、不变量编号）以注释保留，不随语言丢失。

## 6. 剖面完整性校验清单

一行剖面在交给 `domain-driven-design:ddd-port-scaffold` 前，逐项对照：

- [ ] §5 的 5 项全部有明确答案，且补充确认了"领域可独立组织 + 有守护依赖方向的手段"
- [ ] 端口是**纯抽象**：只签名、零实现、不泄漏框架/基础设施类型
- [ ] 值对象**不可变 + 按值相等**；ID 类型与基本类型**不可混用**（防基本类型偏执）
- [ ] 领域**不依赖具体实现**：依赖只向内，DI 在最外层组合根装配
- [ ] 错误/缺失表达统一，且翻译为**领域语义错误**，不外泄底层异常/`null`
- [ ] 契约语义（前置/后置/事务边界/不变量编号）以注释随骨架保留，换语言不丢失