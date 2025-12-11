# `Optional`



------

## 1. Optional 简介

### 1.1 核心概念

`java.util.Optional<T>` 是 Java 8 引入的一个 **不可变的容器类**

它的核心作用非常直观：**它可以保存类型为 T 的非空值，也可以保存“空”（empty）**

> 在 Optional 出现之前，Java 中的“无值”通常是用 `null` 引用来表示的。
>
> 而 Optional 则将“值”和“无值”这两种状态封装在了一个对象内部，通过类型系统强制开发者去思考“如果值不存在该怎么办”



**JDK 源码核心片段解析**

理解 Optional 最好的方式是看它的数据结构（基于 OpenJDK 源码简化）：

```java
public final class Optional<T> {
    
    // 1. 内部使用一个变量 value 存储实际值
    // 如果 value 为 null，则表示该 Optional 为"Empty"（空）
    private final T value;

    // 2. 一个全局共享的空实例，避免重复创建空对象
    private static final Optional<?> EMPTY = new Optional<>();

    // 3. 私有构造函数：外部不能直接 new，必须通过静态工厂方法创建
    private Optional() {
        this.value = null;
    }

    // 4. 关键点：如果传入 value，它必须非空。
    // 这意味着 Optional 内部永远不会包装一个 null 值。
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }
}
```

**解读：**

1. **不可变性**：类被 `final` 修饰，且内部 `value` 也是 `final` 的。这意味着 **一旦 Optional 对象创建，其引用的内容就不能改变**
2. **非 Null 保证**：`Optional` 容器本身不应该为 null（否则就失去了意义），且它包装的非空 Optional 内部的 value 一定不是 null



### 1.2 设计目标与应用场景

Optional 的设计初衷 **并不是** 为了完全消除代码中的 `null`，而是为了在 API 层面建立一种 **语义明确的契约**



#### a. 语义表达（最重要的目标）

这是 Optional 最大的价值。当一个方法的返回值类型是 `Optional<User>` 时，它在明确地告诉调用者：

> “这个方法执行后，**可能** 找不到 User。你必须在编译阶段就处理‘找不到’的情况。”

**对比：**

- `User findUserById(String id)`: 开发者可能会忘记检查 `null`，直接调用 `user.getName()` 导致空指针异常
- `Optional<User> findUserById(String id)`: 
  - 开发者无法直接调用 `.getName()`，必须先通过 Optional 提供的 API（如 `orElse`, `ifPresent`）来解包



#### b. 为什么是 Java 8？

- Java 8 引入了 Stream API（流式编程）
  - 在流式处理链中（例如 `filter` 过滤后可能没有元素），如果直接返回 `null`，会导致链式调用中断或抛出异常
    - `Optional` 完美契合了函数式编程的**链式调用**风格，让数据处理流程更加流畅



### 1.3 技术陷阱与注意事项

在开始深入 API 之前，必须先明确 Optional 的几个**“不能做”**，这是新手最容易踩的坑



#### 🚨 陷阱 1：Optional 未实现序列化接口

`Optional` 类**没有**实现 `java.io.Serializable` 接口

- **后果**：
  - 如果你将 Optional 用作类的字段（Field），并且该类需要被序列化（例如在 Dubbo 传输、Redis 缓存、或写入磁盘时），会抛出 `NotSerializableException`
- **最佳实践**：**不要将 Optional 定义为类的成员变量**



#### 🚨 陷阱 2：不要作为方法参数

- **反模式**：

  ```java
  // ❌ 不推荐
  public void updateUser(Optional<User> userOpt) { ... }
  ```

- **原因**：

  - 这不仅让调用方代码变得臃肿（必须包装一层 `Optional.of`），而且调用方仍然可以传入 `null`（即 `updateUser(null)`），导致方法内部依然需要判空，失去了 Optional 的意义

- **结论**：Optional **主要是作为方法的返回值类型** 设计的



#### 🚨 陷阱 3：它有性能开销

- Optional 是一个对象。将一个简单的对象包装在 Optional 中意味着多创建了一个对象（Wrapper）

  在对内存极其敏感或高频调用的循环中，过度滥用 Optional 可能会带来 GC 压力





------

## 2. 为什么需要 Optional

在深入学习具体 API 之前，我们需要深刻理解为什么要引入这个工具

Java 语言架构师 Brian Goetz 曾说过：“Optional 的出现是为了提供一种更优雅的方式来处理‘空’值，从而减少 NullPointerException。”



### 2.1 痛点：Null 带来的“防御式编程”

在 Java 8 之前，处理嵌套对象（例如：用户 -> 地址 -> 邮箱）是开发者的噩梦。为了防止程序因空指针异常（NPE）崩溃，我们不得不编写大量的防御性代码



### 2.2 场景与解决方案

假设我们需要获取一个用户的邮箱地址，但用户可能为 null，地址可能为 null，邮箱也可能为 null



#### a. 传统写法

**传统写法（这种代码被称为“代码箭头”或“末日金字塔”）：**

```java
public String getUserEmailByName(String name) {
    User user = findUserByName(name);
    
    // 💀 第一层防御
    if (user != null) {
        Address address = user.getAddress();
        
        // 💀 第二层防御
        if (address != null) {
            Email email = address.getEmail();
            
            // 💀 第三层防御
            if (email != null) {
                return email.getValue();
            }
        }
    }
    return "Unknown";
}
```

- **这种写法的问题：**
  1. **业务逻辑被淹没**：核心逻辑只是“获取邮箱”，但代码中充斥着 80% 的 `if (x != null)` 检查
  2. **极易出错**：只要漏写一层检查，生产环境就可能报 NPE
  3. **可读性差**：多层嵌套导致代码阅读困难，维护成本高



#### b. 解决方案：从“命令式”到“声明式”

使用 Optional 后，我们不再关注“如何一步步检查空值”，而是关注“如果有值，我想做什么”

**Optional 改进后的写法：**

*(注：此处假设相关方法如 `findUserByName` 已经经过改造，返回的是 `Optional` 包装对象)*

```java
public String getUserEmailByName(String name) {
    return findUserByName(name)           // 返回 Optional<User>
            .flatMap(User::getAddress)    // 如果 User 存在，取 Address
            .flatMap(Address::getEmail)   // 如果 Address 存在，取 Email
            .map(Email::getValue)         // 如果 Email 存在，取 Value
            .orElse("Unknown");           // 如果上述任意环节为空，返回默认值
}
```

- **改进后的优势：**
  1. **扁平化逻辑**：原本的层层嵌套被拉直成了一条流水线，逻辑一目了然
  2. **自动判空**：`flatMap` 和 `map` 内部自动处理了 `null` 检查。如果中间任何一步为空，流程会自动跳到最后，不会抛出异常	
  3. **强类型约束**：编译器会强制你处理空值情况（例如你必须调用 `orElse` 或其他方法来终结流），而不是让你碰运气



### 2.3 核心思维转变

上面的场景与示例中，这不仅仅是代码变短了，而是思维方式的改变：

| 维度         | 传统 Null 处理                                              | Optional 处理                                               |
| ------------ | ----------------------------------------------------------- | ----------------------------------------------------------- |
| **思维模式** | **命令式 (Imperative)**“先检查A是不是空，再拿B，再检查B...” | **声明式 (Declarative)** “我要拿邮箱，如果没有就给我默认值” |
| **关注点**   | 关注 **异常情况** (怎么防守 NPE)                            | 关注 **正常流程** (数据怎么流转)                            |
| **安全性**   | 依赖程序员的细心 (容易忘)                                   | 依赖类型系统 (编译器强制)                                   |



------

## 3. 创建 Optional 对象

`Optional` 的构造函数是私有的（`private`），因此我们无法直接使用 `new` 关键字创建实例。Java 提供了三个静态工厂方法来创建对象

虽然这看起来很简单，但 **选错创建方法** 是导致 Bug 的常见原因之一

### A. 创建“空”容器：`empty()`

> `Optional.empty()`

用于创建一个内部不包含任何值的 `Optional` 实例

```JAVA
// 创建一个空的 Optional
Optional<String> emptyOpt = Optional.empty();

// 验证
System.out.println(emptyOpt.isPresent()); // 输出: false
```

**注解：**

- **底层优化**：

  - `Optional.empty()` 返回的是一个 **全局共享的单例对象**

    无论你调用多少次，拿到的都是同一个对象，因此它对内存非常友好，不需要担心频繁创建空对象的性能损耗

- **最佳实践**：当你的方法需要返回 `Optional` 但没有数据时，请直接返回 `Optional.empty()`，而不是 `null`



### B. 严格非空包装：`of(T value)`

>`Optional.of(T value)`

用于包装一个 **绝对不为 null** 的值

- **关键特性**：如果你传入 `null`，它会立即抛出 `NullPointerException`

```JAVA
String name = "Java";
Optional<String> opt = Optional.of(name); // 正常

// 🚨这里会立🐎炸
String nullName = null;
try {
    Optional<String> errorOpt = Optional.of(nullName); 
} catch (NullPointerException e) {
    System.out.println("捕获到空指针异常：传入了 null");
}
```

**🤔 既然会报空指针，为什么还要用它？**

- 这涉及到一个重要的编程理念：**快速失败**。 如果你确信某个数据（比如配置文件的常量、必须存在的系统参数）**绝对不能 **为空，那么就应该使用 `Optional.of()`

  - 如果它意外为空，程序会 **立刻** 报错，让你在开发阶段就发现逻辑 Bug

  - 而不是让这个 `null` 混入系统，等到后续业务处理时才引发莫名其妙的错误



### C. 智能判空包装：`ofNullable(T value)`

>`Optional.ofNullable(T value)`

这是最常用的方法。它不仅能包装非空值，也能兼容 `null` 值

**逻辑**：

- 如果传入的值 `!= null`，它等价于 `Optional.of(value)`
- 如果传入的值 `== null`，它自动降级，等价于 `Optional.empty()`

```java
// 场景 1: 传入非空值
String user = "User A";
Optional<String> opt1 = Optional.ofNullable(user); 
// opt1.isPresent() 为 true

// 场景 2: 传入 null
String nullUser = null;
Optional<String> opt2 = Optional.ofNullable(nullUser); 
// opt2.isPresent() 为 false (安全，不会报错)
```

**最佳实践**： 当你不确定数据来源是否可靠（例如：数据库查询结果、第三方接口返回值、Map.get() 的结果）时，**无脑使用 `ofNullable`**



### ⭐ 千万不要给 Optional 赋值 null

这是一个新手极易犯的低级错误，它完全违背了 Optional 的设计初衷



**错误示范：**

```java
// ❌ 绝对禁止！
Optional<String> opt = null; 
```

**为什么不行？** 

- `Optional` 是一个容器对象

  如果你把容器本身设为 `null`，当你试图调用它的方法（如 `opt.isPresent()` 或 `opt.orElse("default")`）时，**会直接抛出空指针异常**



**正确做法：** 永远用 `Optional.empty()` 来表示“没有 Optional”，而不是用 `null`

```java
// ✅ 正确
Optional<String> opt = Optional.empty();
```



------

## 4. Optional 核心 API

Optional 的 API 设计非常丰富，为了方便理解，我们将它们分为四类：**判断、获取、条件执行、转换**



### 4.1 判断值是否存在

#### A. 基础检查：`isPresent()`

这是 Optional 最直观的方法

- 如果值存在，返回 `true`
- 如果值不存在（Empty），返回 `false`

```java
Optional<String> opt = Optional.of("Hello");

if (opt.isPresent()) {
    System.out.println("盒子不为空");
}
```



**🚨 请勿滥用 `isPresent()`**

**这是新手最大的误区！**

很多开发者刚接触 Optional 时，会写出这种代码：

```JAVA
// ❌ 反模式：新瓶装旧酒
Optional<User> userOpt = findUser();

if (userOpt.isPresent()) {
    User user = userOpt.get(); // 获取值
    System.out.println(user.getName());
} else {
    System.out.println("用户不存在");
}
```

**为什么这是错的？** 

- 这和传统的 `if (user != null)` 没有任何本质区别，甚至代码还变长了（多了一层包装对象）。如果只是为了这样写，完全没有必要使用 Optional

- **✅ 最佳实践：** 

  - 绝大多数情况下，你都不应该直接调用 `isPresent()`

    你应该优先使用后续章节会讲到的 **`ifPresent()`**（回调）或 **`map`**（转换）等高阶方法来替代显式的 `if` 判断



#### B. 语义化检查：isEmpty() (Java 11+)

在 Java 11 中，官方为了提高代码可读性，新增了 `isEmpty()` 方法

- 如果值不存在（Empty），返回 `true`
- 如果值存在，返回 `false`



**它的出现是为了消灭“逻辑非”：**

```JAVA
// 😐 Java 8 写法：读起来稍微有点绕（“如果不是存在的”）
if (!opt.isPresent()) { 
    return "默认值"; 
}

// 😎 Java 11+ 写法：语义清晰（“如果是空的”）
if (opt.isEmpty()) { 
    return "默认值"; 
}
```



### 4.2 安全获取值

把值放进容器很容易，但如何安全地把它拿出来，才是有点挑战的地方



#### A. 危险操作：`get()`

这是 Optional 最原始的方法，但也是最危险的方法

- **行为**：如果值存在，返回该值；如果值不存在，抛出 `java.util.NoSuchElementException`

```JAVA
Optional<String> emptyOpt = Optional.empty();

// 💣 炸弹：这行代码会直接导致程序崩溃
String value = emptyOpt.get(); 
```

**注：** 

- Java 官方架构师 Brian Goetz 曾承认，把这个方法命名为 `get()` 是一个失误，应该叫 `getOrThrow()`

  **最佳实践**：除非你刚刚用 `isPresent()` 检查过（但这又回到了反模式），或者你极其确信值一定存在，否则 **尽量避免使用 `get()`**



#### B. 提供默认值：`orElse(T other)`

当 Optional 为空时，返回一个备用的默认值

```JAVA
Optional<String> opt = Optional.empty();
String result = opt.orElse("默认用户"); // 返回 "默认用户"
```



**🚨 性能陷阱（重点）：** 无论 `Optional` 中是否有值，**`orElse` 中的参数都会被执行/计算**

```JAVA
Optional<String> opt = Optional.of("Hello");

// 即使 opt 里有值，createDefaultValue() 方法依然会被调用！
// 这意味着如果这个方法涉及数据库查询或复杂计算，性能会白白浪费
String result = opt.orElse(createDefaultValue()); 
```



#### C. 懒加载默认值：`orElseGet(Supplier)`⭐

这是大多数时候应该使用的“默认值”方法。 它接受一个 `Supplier`（提供者）函数接口，**只有在 Optional 为空时，才会执行这个函数**

```JAVA
Optional<String> opt = Optional.of("Hello");

// ✅ 性能优化：因为 opt 有值，Lambda 表达式根本不会执行
// 数据库查询完全被跳过了
String result = opt.orElseGet(() -> queryFromDatabase());
```



**对比总结（面试必考）：**

| 方法          | 参数类型             | 执行时机     | 适用场景                                      |
| ------------- | -------------------- | ------------ | --------------------------------------------- |
| **orElse**    | 值 (`T`)             | **总是执行** | 简单的常量字符串、数字（如 `"Unknown"`, `0`） |
| **orElseGet** | 函数 (`Supplier<T>`) | **按需执行** | **复杂对象创建、数据库查询、远程调用**        |



#### D. 异常处理：`orElseThrow()`

如果业务逻辑要求“值必须存在”，否则就是异常情况，请使用此方法

**基础版（Java 10+）：** 不传参，默认抛出 `NoSuchElementException`。这是 `get()` 的语义化替代品

```java
User user = findUser().orElseThrow();
```



**进阶版（自定义异常）：** 传入一个异常生成器，抛出你项目定义的业务异常

```java
// 如果找不到用户，抛出 UserNotFoundException
User user = findUserById(id)
    .orElseThrow(() -> new UserNotFoundException("用户 ID 不存在: " + id));
```



**最佳实践：** 在 Service 层调用 Repository 层时，经常使用这种模式来处理“按 ID 查询”的结果，从而避免在后续代码中传递 `null`



### 4.3 条件执行

当我们只想 **消费** Optional 中的值（例如打印日志、发送消息、更新状态），而不需要返回值时，这一组方法是你的首选

#### A. 如果存在就执行：`ifPresent(Consumer)`

这是替代 `if (opt.isPresent()) { ... }` 的标准方式

- **逻辑**：如果 Optional 中有值，就执行传入的 Lambda 表达式；如果是空的，什么都不做
- **参数**：`Consumer<T>`（消费型接口，有入参无返回值）



**代码对比：**

```JAVA
Optional<User> userOpt = findUser(1L);

// ❌ 笨拙的写法 (Imperative)
if (userOpt.isPresent()) {
    User user = userOpt.get();
    sendEmail(user);
}

// ✅ 优雅的写法 (Functional)
// 读作："如果用户存在，就给这个用户发邮件"
userOpt.ifPresent(user -> sendEmail(user));

// 或者使用方法引用，更简洁：
userOpt.ifPresent(this::sendEmail);
```



**实战场景**： 最常用于触发副作用，比如写库、发消息、记日志



#### B. 存在与否都处理：`ifPresentOrElse(Consumer, Runnable)` (Java 9+)

在 Java 8 中，`ifPresent` 有一个痛点：

- 它没法处理 `else` 的情况。如果你需要“如果存在打印值，如果不存在打印警告”，Java 8 只能被迫退回到 `isPresent()` 的写法

Java 9 补齐了这个短板

- **参数 1**：`Consumer<? super T>` —— 值存在时执行的操作
- **参数 2**：`Runnable` —— 值 **不存在** 时执行的操作（注意是 Runnable，不接受参数）



**代码演示：**

```JAVA
Optional<User> userOpt = findUser(1L);

userOpt.ifPresentOrElse(
    // 1. 如果存在 (Consumer)
    user -> {
        System.out.println("找到用户：" + user.getName());
        login(user);
    },
    // 2. 如果不存在 (Runnable)
    () -> {
        log.warn("用户 1L 未找到，可能是非法访问");
        alertAdmin();
    }
);
```



**注：** 

- 这个方法极其适合处理**非关键路径的业务逻辑**

  比如：“尝试从缓存获取数据，拿到了就更新 UI，没拿到就发起一个异步加载请求”，这种逻辑用 `ifPresentOrElse` 写出来非常漂亮，完全没有 `if-else` 的缩进嵌套感



#### C. 总结

| 需求场景                     | 推荐方法                               | Java 版本   |
| ---------------------------- | -------------------------------------- | ----------- |
| **只想消费值，没值就不管**   | `ifPresent(action)`                    | Java 8+     |
| **有值消费，没值也要做动作** | `ifPresentOrElse(action, emptyAction)` | **Java 9+** |
| **需要基于值进行下一步转换** | (下一节要讲的 `map` / `flatMap`)       | Java 8+     |



### 4.4 转换操作：链式调用的灵魂

Optional 真正的威力不在于简单的判空，而在于它可以像流水线一样，对内部的值进行连续的转换和处理

#### A. 基础转换：`map()`

如果熟悉 Java Stream API，那么 `Optional` 的 `map` 理解起来非常容易

- **逻辑**：
  - 如果 `Optional` 中有值，就对这个值应用提供的函数（Function），并将结果 **自动包装** 在一个新的 `Optional` 中
  - 如果原 `Optional` 为空，则直接返回空 `Optional`

**代码示例：**

```JAVA
Optional<String> nameOpt = Optional.of("java");

// 转换逻辑：String -> Integer
// 1. 取出 "java"
// 2. 计算 length() -> 4
// 3. 包装成 Optional<Integer>
Optional<Integer> lenOpt = nameOpt.map(String::length);

System.out.println(lenOpt.get()); // 输出 4
```



**自动判空特性（重点）：** 如果你的转换函数返回了 `null`，`map` 会非常智能地把它转换成 `Optional.empty()`，而不会报空指针异常

```JAVA
Optional<User> userOpt = findUser();

// 即使 user.getEmail() 返回 null，这里也不会炸
// 结果会变成一个空的 Optional<String>
Optional<String> emailOpt = userOpt.map(User::getEmail);
```



#### B. 扁平化转换：`flatMap()`

这是很多人的噩梦，但逻辑其实很简单

**场景引入：为什么要用 flatMap？**

> `map` 方法有一个死规矩：**不管你返回给我什么东西，我都要把它塞进一个新的 Optional 盒子里**

假设我们有以下类结构，每个方法都返回 `Optional`（这是非常标准的现代 API 设计）：

```JAVA
class User {
    // 用户可能有地址，也可能没有
    public Optional<Address> getAddress() { ... }
}

class Address {
    // 地址可能有街道信息，也可能没有
    public Optional<String> getStreet() { ... }
}
```



**如果我们强行用 `map`，会发生什么？**

```JAVA
Optional<User> userOpt = findUser();

// ❌ 灾难现场：嵌套的洋葱圈
// map 会把结果自动再包一层 Optional
Optional<Optional<Address>> result = userOpt.map(User::getAddress);
```

此时，变量 `result` 的类型变成了 `Optional<Optional<Address>>`。你想取值时，必须 `result.get().get()`，这极其丑陋且难以处理



**✅ 解决方案：`flatMap`**

`flatMap`（Flat Map，扁平化映射）的作用是：**如果你的转换函数本身就已经返回了 Optional，我就不再多包一层了，直接把那个 Optional 拿来用**

```JAVA
// ✅ 优雅的链式调用
Optional<String> streetOpt = userOpt
    .flatMap(User::getAddress)  // 返回 Optional<Address>，不嵌套
    .flatMap(Address::getStreet); // 返回 Optional<String>，不嵌套

System.out.println(streetOpt.orElse("未知街道"));
```



#### C. map vs flatMap 决策指南

| 转换函数的返回值                  | 应该使用的方法  | 结果类型                                     |
| --------------------------------- | --------------- | -------------------------------------------- |
| 普通对象 (如 `String`, `Integer`) | **`map()`**     | `Optional<T>`                                |
| **已经是一个 `Optional` 对象**    | **`flatMap()`** | `Optional<T>` (拒绝 `Optional<Optional<T>>`) |

**最佳实践口诀：**

> 看着方法的返回值：
>
> - 如果是普通值，用 `map`。
> - 如果是 `Optional`，用 `flatMap`。



#### D. 实战：构建防御性数据获取链

还记得第 2 节那个恐怖的嵌套 `if-null` 吗？现在我们可以用 `flatMap` 完整重写它

**需求**：获取用户所在城市的名称，如果任何环节缺失，返回 "Unknown"

```JAVA
public String getUserCityName(Long userId) {
    return findUserById(userId)          // 返回 Optional<User>
            .flatMap(User::getAddress)   // User -> Optional<Address>
            .flatMap(Address::getCity)   // Address -> Optional<City>
            .map(City::getName)          // City -> String (getName返回普通String，所以用map)
            .orElse("Unknown");          // 兜底
}
```

这段代码不仅安全（绝无 NPE），而且逻辑清晰得像在读英语句子



### 4.5 过滤操作：`filter()`

如果你习惯了使用 `if` 语句来检查某个值是否符合条件，`filter` 方法将彻底改变你的编码方式

#### A. 基础用法

`filter` 的逻辑非常直接：

1. **判断**：如果 Optional 中有值，并且这个值满足你给定的条件（Predicate）
2. **保留**：返回包含该值的 Optional
3. **丢弃**：如果值不满足条件（或者本身就是空的），直接返回 `Optional.empty()`



**代码对比：**

```java
// ❌ 传统写法：嵌套的 if
User user = findUser();
if (user != null) {
    if (user.getAge() >= 18) {
        // ...处理成年用户
    }
}

// ✅ Optional 写法：一行搞定
Optional<User> adultUserOpt = findUser()
    .filter(user -> user.getAge() >= 18);
```



#### B. 实战场景：多重校验

`filter` 最强大的地方在于它可以 **链式调用** ，这在处理多重校验逻辑时非常清晰

**需求**：查找一个订单，要求必须是“已支付”状态，且金额大于 100 元，且没有过期

```java
public Optional<Order> findValidHighValueOrder(Long orderId) {
    return orderRepository.findById(orderId)
            // 第一关：检查状态
            .filter(order -> order.getStatus() == OrderStatus.PAID)
            // 第二关：检查金额
            .filter(order -> order.getAmount() > 100)
            // 第三关：检查有效期
            .filter(order -> !order.isExpired());
}
```

**注：** 如果在以前，这通常需要写 3 个 `if` 语句或者一个超长的 `&&` 表达式。使用 `filter` 后，每一行代码对应一个业务规则，阅读起来就像在读需求文档



### 4.6 Java 9+ 增强功能

随着 Java 版本的迭代，Optional 也在不断进化。以下两个方法是 Java 9 引入的重要补充，如果你在使用 JDK 8 之后的版本，一定要学会使用

#### A. 备选方案链：`or()`

我们在 4.2 节学过 `orElse` 和 `orElseGet`，它们用于在 Optional 为空时返回一个**确定的值（T）**

但是，如果你的备选方案 **也是一个 Optional** 怎么办？



**场景**：

你需要查找用户

1. 先查 Redis 缓存
2. 如果没有，再查 MySQL 数据库
3. 如果还没有，再查归档数据库

在 Java 8 中，这种“链式查找”写起来非常痛苦。但在 Java 9 中，`or()` 方法完美解决了这个问题

- **签名**：`Optional<T> or(Supplier<Optional<T>> supplier)`
- **作用**：如果当前 Optional 为空，不直接返回 T，而是尝试返回另一个 Optional

```JAVA
// ✅ 优雅的责任链模式
public Optional<User> findUser(Long id) {
    return findInRedis(id)              // 尝试 Redis
            .or(() -> findInMysql(id))  // Redis 没找到？试着返回 MySQL 的 Optional
            .or(() -> findInArchive(id)); // 还没找到？试着返回归档的 Optional
}
```

**关键区别：**

- `orElseGet`：返回 `T`（大结局，取出了值）
- `or`：返回 `Optional<T>`（还没结束，只是换了个盒子继续传递）



#### B. 流式对接：`stream()` ⭐

这个方法将 Optional 与 Stream API 彻底打通了

- **作用**：
  - 如果 Optional 有值，把它变成一个包含 1 个元素的 `Stream`
  - 如果 Optional 为空，把它变成一个 `Stream.empty()`



**实战用法：去除列表中的空值**

假设你有一个列表，里面包含多个 `Optional<User>`（可能有的找到了，有的没找到），你现在只想提取出所有存在的 User 对象，放到一个 List 里

```JAVA
List<Optional<User>> userOptionals = Arrays.asList(
    Optional.empty(), 
    Optional.of(new User("A")), 
    Optional.empty(), 
    Optional.of(new User("B"))
);

// Java 9 之前的写法（很啰嗦）
List<User> usersOld = userOptionals.stream()
    .filter(Optional::isPresent) // 先过滤掉空的
    .map(Optional::get)          // 再拆包
    .collect(Collectors.toList());

// ✅ Java 9 之后的写法 (利用 flatMap + stream)
List<User> usersNew = userOptionals.stream()
    .flatMap(Optional::stream)   // 自动把 Empty 的 Optional 变成空流并在合并时消失
    .collect(Collectors.toList());

// 结果：[User(A), User(B)]
```

**注：** `flatMap(Optional::stream)` 是处理“Optional 列表”或“流中包含 Optional”时的标准范式，非常简洁高效



## 5. Optional 与 Stream API

Optional 和 Stream 是 Java 8 函数式编程的两大支柱

- **Stream** 关注数据的 **流转与处理**（集合操作）
- **Optional** 关注数据的 **缺失与安全**（单个值操作）

当它们结合在一起时，能解决很多复杂的集合处理问题

### 5.1 Stream 操作产生 Optional

在使用 Stream 时，很多**终端操作（Terminal Operations）** 的结果是不确定的（可能找不到元素），因此它们天然返回 `Optional`

**常见的返回 Optional 的操作：**

- `findFirst()`: 获取流中的第一个元素
- `findAny()`: 获取流中的任意一个元素
- `min()` / `max()`: 根据比较器获取最小/最大值
- `reduce()`: 归约操作（部分重载方法）



**代码示例：**

```java
List<User> users = getUserList();

// 1. 查找第一个年龄大于 18 的用户
// 返回 Optional<User>，因为可能一个成年的都没有
Optional<User> firstAdult = users.stream()
    .filter(user -> user.getAge() > 18)
    .findFirst();

// 2. 查找年龄最大的用户
// 返回 Optional<User>，因为列表可能是空的
Optional<User> oldestUser = users.stream()
    .max(Comparator.comparing(User::getAge));

// 3. 计算所有订单的总金额
// reduce 返回 Optional<BigDecimal>
Optional<BigDecimal> totalAmount = orders.stream()
    .map(Order::getAmount)
    .reduce(BigDecimal::add);
```



**✅ 最佳实践：** 拿到 Stream 返回的 Optional 后，直接使用我们第 4 节学过的链式方法（`ifPresent`、`map`、`orElse`）处理，不要断开链条

```java
// 优雅的一条龙处理
users.stream()
    .filter(User::isVIP)
    .findFirst()
    .ifPresent(this::sendVipGift); // 找到了就发礼物，没找到啥也不干
```



### 5.2 `Optional` 转换为 Stream (Java 9+)

#### 知识点

在 Java 9 中，Optional 增加了一个看似简单却极具威力的方法：`stream()`

- **签名**：`Stream<T> stream()`
- **逻辑**：
  - 如果 Optional 有值，返回一个**包含该值的 Stream**（长度为 1）
  - 如果 Optional 为空，返回一个**空 Stream**（长度为 0）

**它的核心作用是：将 Optional 视为一个特殊的集合（最多 1 个元素），从而无缝融入 Stream 流水线**



#### ⚡️ 如何优雅地过滤空值？

**场景**：你有一个 `List<Optional<String>>`（比如从多个不可靠的数据源收集来的结果），你只想提取出那些 **有值** 的字符串，放到一个新列表里

**方法 A：Java 8 写法（略显笨拙）**

```java
List<Optional<String>> rawData = Arrays.asList(
    Optional.of("Hello"), 
    Optional.empty(), 
    Optional.of("World")
);

List<String> result = rawData.stream()
    .filter(Optional::isPresent) // 1. 先把空的剔除
    .map(Optional::get)          // 2. 再把皮剥掉
    .collect(Collectors.toList());
```

**缺点**：用到了 `isPresent` 和 `get`，不够优雅



**方法 B：Java 9+ 写法（flatMap + stream）🔥**

利用 `flatMap` 的“扁平化”特性：

- 它会把每个 Optional 转换成的 Stream（有的有1个元素，有的有0个）全部通过“拼接”的方式合并起来。空 Stream 会自动消失

```java
List<String> result = rawData.stream()
    // Optional::stream 会把 Empty 变成空流，把 Present 变成单元素流
    // flatMap 会自动丢弃空流，只保留有值的流
    .flatMap(Optional::stream) 
    .collect(Collectors.toList());
    
// 结果：["Hello", "World"]
```

**解：** 这是处理“批量 Optional”最标准的范式。记住这个公式：**`stream().flatMap(Optional::stream)` = 自动拆包并去空**



### 5.3 复杂组合实战

来看一个稍微复杂的业务场景，结合 `map`、`flatMap` 和 `Stream`

**需求**：获取用户最近一笔有效订单的金额

1. 找到用户（可能不存在）
2. 获取用户的订单列表（可能为空）
3. 在订单列表中筛选出有效的
4. 找到时间最近的一笔
5. 获取金额

```java
public BigDecimal getLatestOrderAmount(Long userId) {
    return userRepository.findById(userId) // 1. Optional<User>
        .map(User::getOrders)              // 2. Optional<List<Order>>
        .stream()                          // 3. 转换为 Stream<List<Order>>
        .flatMap(Collection::stream)       // 4. 铺平为 Stream<Order> (关键！)
        .filter(Order::isValid)            // 5. 筛选有效
        .max(Comparator.comparing(Order::getDate)) // 6. 找最近的 -> Optional<Order>
        .map(Order::getAmount)             // 7. 取金额 -> Optional<BigDecimal>
        .orElse(BigDecimal.ZERO);          // 8. 兜底
}
```



**陷阱提示：** 注意第 2 步到第 3 步。

- `User::getOrders` 返回的是 `List<Order>`。
- 所以 `map` 之后是 `Optional<List<Order>>`。
- 如果不调用 `stream().flatMap(...)`，就被困在 Optional 里了，很难对 List 内部的元素进行 `filter` 或 `max` 操作



**✅ 技巧：** 当你发现 `Optional` 里面包裹的是一个 `Collection`（集合），并且你想对集合里的元素操作时，

- **请立刻使用 `stream().flatMap(Collection::stream)` 将其打散**



## 6. 最佳实践

Optional 是把双刃剑。用好了是“防空卫士”，用不好就是“代码污染”

### 6.1 应该使用 Optional 的核心场景

Optional 的设计初衷非常明确，它主要是为了解决 **返回值的语义问题**

#### ✅ 场景 1：作为公共 API 的返回值（核心）

这是 Optional 最正统、最推荐的用法。 当你的方法设计为“可能找不到结果”时，不要返回 `null`，请返回 `Optional`

**为什么？** 这是一种 **契约编程** 。你通过方法签名直接告诉调用者：“别大意，这里可能会空，请做好准备。”

```java
// ✅ 推荐：调用者一眼就能看出 user 可能不存在
public Optional<User> findUserById(Long id) {
    // 这里的 database.query 如果返回 null，ofNullable 会自动处理
    return Optional.ofNullable(database.query(id));
}
```



#### ✅ 场景 2：在链式调用中作为“中间人”

在复杂的业务流处理中，Optional 是非常好的“胶水”。它可以让数据在多个可能为空的方法之间安全传递，而不需要每次都写 `if (x != null)`

**示例：获取用户简介** 如果不使用 Optional，这行代码可能需要在 3 个地方做 `null` 检查

```java
public String getUserBio(Long userId) {
    return findUser(userId)              // 返回 Optional<User>
            .flatMap(User::getProfile)   // Profile 可能为空，返回 Optional<Profile>
            .map(Profile::getBio)        // Bio 可能为空，返回 String (map 自动包装为 Optional)
            .orElse("No bio available"); // 一气呵成，任何环节断裂都走兜底
}
```



#### ⚠️ 场景 3：作为可选的方法参数（需谨慎）

在 Java 社区的主流规范（如 IntelliJ IDEA 的默认检查、Effective Java）中，**尽量不要把 Optional 用作参数**

**为什么通常不推荐？** 因为它强迫调用者必须包装数据，让代码变得啰嗦：

```java
// 定义
public void createUser(String name, Optional<String> email) { ... }

// 调用者必须这么写，很麻烦：
createUser("Bob", Optional.of("bob@example.com"));
createUser("Alice", Optional.empty());
```



**那什么时候可以用？** 只有当你 **不想写大量的重载方法**（Overload），且方法内部逻辑并不复杂时，可以作为一种妥协方案

```java
// ✅ 勉强可接受的场景：避免写 2 个重载方法
// 此时把它当做一个"携带 null 信息的容器"来用
public void updateConfig(String key, Optional<String> value) {
    // 如果传入了 value，就更新；如果传入 empty，就不更新（或者重置）
    value.ifPresent(v -> configMap.put(key, v));
}
```



**最佳替代方案：** 通常建议使用**方法重载**：

```java
public void createUser(String name) {
    createUser(name, null);
}

public void createUser(String name, String email) {
    // ...
}
```



#### 总结：Optional 的“适用区”

| 代码位置      | 推荐度 | 备注                                     |
| ------------- | ------ | ---------------------------------------- |
| **返回值**    | ⭐⭐⭐⭐⭐  | **最核心用途**，强制调用者判空           |
| **局部变量**  | ⭐⭐⭐    | 仅在 stream/chain 链式处理的中间环节使用 |
| **方法参数**  | ⭐      | **不推荐**，除非是为了减少重载方法的数量 |
| **字段/属性** | 💀      | **严禁**                                 |



### 6.2 不应该使用 Optional 的场景

Optional 是为了“返回值”而生的。如果把它用在其他地方，往往弊大于利。以下是必须避开的“雷区”

#### ❌ 场景 1：禁止作为类的字段 (Field)

这是新手最常犯的错误

**代码对比：**

```java
// ❌ 错误示范：极度不推荐
public class User {
    // 1. Optional 没有实现 Serializable 接口，会导致序列化失败
    // 2. 每个字段多包装一层对象，内存占用增加
    private Optional<String> email; 
}

// ✅ 正确示范
public class User {
    // 字段本身直接用普通类型（允许为 null）
    private String email;

    // 只在 Getter 方法中包装，对外暴露 Optional
    public Optional<String> getEmail() {
        return Optional.ofNullable(email);
    }
}
```

**深度解析：**

1. **序列化灾难**：

   - `java.util.Optional` 并没有实现 `Serializable` 接口

     如果你把 User 对象存入 Redis、通过 Dubbo 传输或者写入磁盘，会直接报 `NotSerializableException`

     虽然有些序列化框架（如 Jackson）可以通过配置支持，但这增加了不必要的复杂性

2. **内存开销**：

   - Java 对象头本身就有开销

     如果你的 User 对象有 10 个字段全用 Optional 包装，这就意味着你需要额外创建 10 个 Wrapper 对象

     在大数据量场景下，这是不可忽视的 GC 压力



#### ❌ 场景 2：禁止作为集合的泛型 (Collections)

**原则**：集合（List, Set, Map）本身就已经具备了“空”的表达能力（即 `size() == 0` 或 `isEmpty()`）。千万不要“脱裤子放屁”

```java
// ❌ 错误：这会让调用者发疯
// 这是一个"可能包含'可能不存在的字符串'的列表"？
List<Optional<String>> list = new ArrayList<>();

// ❌ 错误：Map 的 Key 绝不能是 Optional
Map<Optional<String>, String> map = new HashMap<>();

// ✅ 正确：
List<String> list = new ArrayList<>(); 
// 如果没有值，就返回一个空列表 Collections.emptyList()
```



**提示：** 如果在 Map 的 Value 中使用了 Optional（如 `Map<String, Optional<User>>`），请反思一下：为什么不直接不存这个 Key 呢？

- `map.get("key")` 返回 `null` 表示不存在
- 没必要存一个 `Optional.empty()` 进去占位置



#### ❌ 场景 3：构造函数与 Setter 参数

这与我们在 6.1 中讨论的“普通方法参数”类似，但在构造函数中更为严重

```java
// ❌ 错误：强迫调用者为了创建一个对象，还要先创建 Optional
public User(String name, Optional<String> email) {
    this.name = name;
    this.email = email.orElse(null); // 还要拆包赋值给字段，多此一举
}

// 调用时非常丑陋：
User u = new User("Bob", Optional.of("bob@example.com"));
```



**✅ 最佳实践：** 使用**重载（Overloading）** 或 **构建者模式（Builder Pattern）**

```java
// 方式一：重载构造函数
public User(String name) {
    this(name, null);
}

public User(String name, String email) {
    this.name = name;
    this.email = email;
}

// 方式二：Builder 模式 (Lombok @Builder)
User user = User.builder()
    .name("Bob")
    .email("bob@example.com") // 可选调用
    .build();
```



#### 总结：Optional 的“禁区”

| 场景                  | 推荐度       | 核心理由                           |
| --------------------- | ------------ | ---------------------------------- |
| **类的字段 (Field)**  | 💀 **严禁**   | 序列化失败、内存浪费。             |
| **集合泛型**          | 💀 **严禁**   | 集合本身已有判空机制，增加复杂度。 |
| **方法/构造函数参数** | ⚠️ **不推荐** | 污染调用方代码，建议用重载代替。   |



### 6.3 编码规范

#### A. 拒绝“新瓶装旧酒”

我们在 4.1 节提到过，这是最严重的坏味道

**❌ 坏味道：** 把 Optional 当作 `null` 检查工具，依旧使用命令式编程风格

```JAVA
Optional<User> userOpt = findUser();
// 这和 if (user != null) 有什么区别？
if (userOpt.isPresent()) {
    System.out.println(userOpt.get().getName());
}
```

**✅ 推荐规范：** 强制自己使用函数式风格（`ifPresent`、`map`、`orElse`）。哪怕只有一行代码，也要习惯这种思维转换

```JAVA
findUser().ifPresent(user -> System.out.println(user.getName()));
```



#### B. 性能敏感：orElseGet 优先

除非默认值是**常量**或**已有变量**，否则一律使用 `orElseGet`

**❌ 隐形性能杀手：**

```JAVA
// 即使 opt 非空，queryDatabase() 也会被执行！
// 这可能会导致多余的 DB 查询
User user = opt.orElse(queryDatabase());
```

**✅ 推荐规范：**

```JAVA
// 只有 opt 为空时，才会执行 Lambda 表达式
User user = opt.orElseGet(() -> queryDatabase());
```



#### C. 变量命名规范

Optional 类型的变量名应该体现出“可能为空”的语义

**❌ 模糊的命名：**

```JAVA
Optional<User> user = findUser(); 
// 后续代码容易把它当成普通 User 对象处理
```

**✅ 推荐规范：** 通常建议使用 `maybe` 前缀或 `Opt` 后缀

```JAVA
Optional<User> userOpt = findUser(); 
// 或者
Optional<User> maybeUser = findUser();
```



#### D. 控制链式调用的长度

虽然链式调用很爽，但如果写得太长（超过 5 行），调试起来会非常痛苦（Debug 很难打断点进中间步骤）

**❌ 过长的链条：**

```JAVA
return findUser(id)
    .map(User::getAddress)
    .map(Address::getCity)
    .filter(city -> city.length() > 5)
    .map(String::toUpperCase)
    .map(s -> "City: " + s)
    .orElse("Unknown");
```



**✅ 推荐规范：** 适度拆分。将中间的复杂逻辑提取为独立方法，或者拆成两个 Optional 变量

```JAVA
Optional<Address> addressOpt = findUser(id).flatMap(User::getAddress);

return addressOpt
    .map(Address::getCity)
    .filter(c -> c.length() > 5)
    .map(String::toUpperCase)
    .orElse("Unknown");
```



### 6.4 架构决策：Optional 与 null 的抉择

在项目中，我们是否应该彻底消灭 `null`？ **讲师观点：不需要，也没必要**

Optional 应该主要用于 **公共 API 的边界**，而在 **私有方法内部**，使用 `null` 往往更高效、更简单



#### 原则 1：公共 API (Service/Manager) -> 必须 Optional

公共方法被多个模块调用，契约必须明确

```JAVA
public interface UserService {
    // 明确告诉调用方：可能查不到
    Optional<User> findById(Long id);
}
```



#### 原则 2：私有/底层方法 (Private/DAO) -> 可以 null

在类内部的私有 helper 方法，或者底层的极简 DAO 层，如果确信调用者能安全处理 `null`，或者为了极致性能（少创建一个对象），返回 `null` 是完全可以接受的

```JAVA
private User queryRawData(Long id) {
    // 内部私有方法，直接返回 null 没问题，不需要包装
    if (dbData == null) return null;
    return mapToUser(dbData);
}

// 公共入口负责包装
public Optional<User> findById(Long id) {
    return Optional.ofNullable(queryRawData(id));
}
```



#### 总结：决策树

1. **是方法的返回值吗？**
   - 否（参数/字段） -> **用普通类型 (null)**
   - 是 -> **进入下一步**
2. **是公共 API 吗？**
   - 否（私有方法） -> **可以用 null**
   - 是 -> **必须用 Optional**



------

## 7. 常见误区

Optional 是一个强大的工具，但它不是银弹。很多开发者因为对它期望过高或理解有误，反而写出了更难以维护的代码

> “银弹”这个词在软件工程中是一个非常经典的术语，源自西方民间传说
>
> **1. 字面意思（典故）：** 
>
> 在西方传说中，狼人（Werewolf）是一种极其难杀死的怪物，普通的子弹打不死它，**只有用银做的子弹（Silver Bullet）才能一击毙命**。因此，“银弹”被引申为“能解决棘手问题的杀手锏”或“万能药”
>
> **2. 软件工程中的含义：** 
>
> 这个词在 IT 界爆火，是因为图灵奖得主 Frederick Brooks（也就是《人月神话》的作者）写过一篇著名的论文叫**《没有银弹》（No Silver Bullet）**
>
> 他的核心观点是：**在软件开发中，没有任何一种单一的技术或管理方法（像银弹一样），能够将软件开发的生产力或可靠性在十年内提高一个数量级**



### 误区 1: Optional 能完全消除 NullPointerException

**❌ 错误认知**： “只要我用了 Optional，我的代码就再也不会报空指针异常了”

**🧠 纠偏**： Optional 只能保证**“当你拿到这个盒子时，提醒你盒子可能为空”**。它无法解决以下问题：

1. **盒子里的对象本身有问题**：Optional 包装的 User 对象非空，但 `user.getName()` 可能返回 `null`
2. **Optional 变量本身为 null**：我们在前面强调过，绝对不要把 Optional 变量赋值为 `null`，但编译器无法阻止别人这么做

**代码演示：**

```java
// 场景：你以为安全的链式调用
Optional<User> userOpt = findUser();

// 如果 findUser() 返回了 null（而不是 Optional.empty()），这里直接报 NPE
// 如果 userOpt.get() 成功了，但 user.getAddress() 返回 null，下面还是 NPE
String city = userOpt.get().getAddress().getCity(); 
```

**✅ 正确做法**： Optional 解决的是 **“Optional 对象本身的判空契约”**，而不是递归地解决所有字段的空指针。你依然需要利用 `map/flatMap` 来安全地解引用



### 误区 2: Optional 是万能的 (到处都用)

**❌ 错误认知**： “既然 Optional 这么好，那我把所有可能为空的字段、参数全部改成 Optional”

**🧠 纠偏**： Optional 是有 **成本** 的（对象包装开销、序列化问题）。过度使用会导致“泛型地狱”和代码可读性下降

**代码对比：**

```java
// ❌ 走火入魔：为了用而用
public class User {
    private Optional<String> firstName; // 字段勿用！
    private Optional<String> lastName;  // 字段勿用！
    private Optional<Integer> age;      // 字段勿用！
}

// ✅ 恰到好处：
public class User {
    // 字段保持简单
    private String firstName;
    private String lastName;
    
    // 只有在计算属性或 getter 中，为了提示调用者才使用
    public Optional<String> getMiddleName() {
        return Optional.ofNullable(middleName);
    }
}
```



### 误区 3: 直接使用 get() 是安全的

**❌ 错误认知**： “我知道这里一定有值，所以直接用 `get()` 没问题”

**🧠 纠偏**： 

- 在代码生命周期的演进中，**“一定有值”的假设往往会被打破**

  直接调用 `get()` 就像在代码里埋了一颗地雷，它的危险程度和直接调用 `null` 对象没有区别，甚至更隐蔽（因为编译器不报错）

**讲师建议**： 在代码审查（Code Review）中，看到 `get()` 应该直接打回

**代码演示：**

```java
// ❌ 危险写法：如果 findUser() 逻辑变了，或者数据脏了，这里直接炸
User user = findUser().get(); 

// ✅ 安全替代方案 1：给个默认值
User user = findUser().orElse(new GuestUser());

// ✅ 安全替代方案 2：明确抛出具体异常（方便排错）
User user = findUser().orElseThrow(() -> 
    new UserNotFoundException("核心业务中断：无法获取当前用户"));
```



### 误区 4: Optional.of() 可以接受 null

**❌ 错误认知**： “反正 Optional 就是用来包 null 的，所以我无脑用 `Optional.of(变量)` 就行了。”

**🧠 纠偏**： 

- `Optional.of(value)` 的设计初衷是 **“快速失败”（Fail-Fast）**。它要求传入的值**必须非空**

  如果你传入 `null`，它会立即抛出 NPE，这在某些严格的数据校验场景下是有用的，但如果你只是想“安全地包装一个可能为空的值”，那你选错方法了

**代码演示：**

```java
String value = null;

// ❌ 炸裂：这里会立即抛出 NullPointerException
// 如果你不理解它的原理，你会觉得“为什么用了 Optional 还会报空指针？”
Optional<String> opt = Optional.of(value);

// ✅ 正确：
Optional<String> optSafe = Optional.ofNullable(value); // 返回 Optional.empty()
```



### 误区 5: 嵌套 Optional 是好的设计

**❌ 错误认知**： “Optional 套 Optional，说明我保护得严密”

**🧠 纠偏**： `Optional<Optional<String>>`（俗称“洋葱圈”）是 API 设计失败的典型标志。它不仅让代码极难阅读，还迫使调用者进行两次解包（`get().get()`），极易出错

**出现原因**： 通常是因为在 `map` 函数中调用了一个本身就返回 `Optional` 的方法

**代码对比：**

```java
// 假设 getManager() 返回 Optional<Manager>

// ❌ 错误：使用 map 导致嵌套
Optional<User> user = findUser();
Optional<Optional<Manager>> nested = user.map(User::getManager);

// 如果想获取 Manager 的名字，你得这么写（极其丑陋）：
String name = nested.orElse(Optional.empty())
                    .map(Manager::getName)
                    .orElse("Unknown");

// ✅ 正确：使用 flatMap 拒绝嵌套
Optional<Manager> manager = user.flatMap(User::getManager);
String nameSafe = manager.map(Manager::getName).orElse("Unknown");
```



### 误区 6: 在 Optional 上使用 == 比较

**❌ 错误认知**： “Optional 是单例的吗？或者它像枚举一样？我可以用 `==` 来判断它们是否相等吗？”

**🧠 纠偏**： 

- Optional 是一个普通的对象（虽然 `empty()` 是单例，但 `of()` 出来的都是新对象）。在 Java 中，引用类型的 `==` 永远比较的是**内存地址**

  更重要的是，Optional 被定义为 **“基于值的类（Value-based Classes）”**。这意味着你不应该关注它的身份（Identity），而应该关注它内部的值

**代码演示：**

```java
Optional<String> opt1 = Optional.of("Hello");
Optional<String> opt2 = Optional.of("Hello");

// ❌ 错误：返回 false！
// 尽管它们包装的值是一样的，但这依然是两个不同的包装器对象。
if (opt1 == opt2) { 
    System.out.println("相等"); 
}

// ✅ 正确：返回 true
// Optional 重写了 equals 方法，它会比较内部包装的值。
if (opt1.equals(opt2)) { 
    System.out.println("相等"); 
}
```

Java 官方文档明确指出：**“程序员应该将相等的 Optional 实例视为可互换的，并且不应该对实例进行同步（synchronization）操作”**



## 8. 性能考虑

Optional 是为了 **代码优雅性** 和 **安全性** 牺牲了少量性能的典型案例

在 99% 的业务系统（CRUD、Web 服务）中，这个开销可以忽略不计；但在 1% 的高性能场景中，它可能成为瓶颈



### 8.1 内存开销：不可忽视的包装成本

Optional 是一个**对象**。这意味着每次你使用 Optional，都在堆内存中创建了一个新的实例

**内存布局对比：**

- **普通引用** (`String str = "hello"`): 仅占用 **4-8 字节**（引用本身）
- **Optional 包装** (`Optional<String> opt`):
  - Optional 对象头：**12-16 字节**
  - 内部 value 引用：**4-8 字节**
  - **总计**：约 **16-24 字节**。

**结论**：Optional 的内存占用通常是普通引用的 **3-4 倍**



### 8.2 GC 压力：对象的生与死

除了内存占用，更隐蔽的成本是 **垃圾回收 (GC)**

```java
// 假设这个方法每秒被调用 10 万次
public Optional<User> findUser() {
    // 每次调用都会 new 一个 Optional 对象
    return Optional.ofNullable(database.query()); 
}
```

如果你的系统是高频交易系统或底层中间件，每秒创建数百万个短命的 Optional 对象，会给 CPU 带来巨大的 GC 压力，导致 "Stop The World" 停顿





