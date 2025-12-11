# 初识函数式接口

## 1. 什么是函数式接口

函数式接口（Functional Interface）是 Java 8 引入的核心概念之一，它为 Lambda 表达式提供了“目标类型”

其严格定义是：**只包含一个抽象方法的接口**

这个特性是 Lambda 表达式在 Java 中实现的基础
编写一个 Lambda 表达式时，实际上是在提供该 **唯一抽象方法** 的具体实现

> Lambda 表达式只能被赋值给函数式接口类型的变量

Java 库中早已存在许多符合此定义的接口（例如 `java.lang.Runnable`, `java.util.Comparator`），Java 8 只是在语言层面对这一概念进行了正式化



一个常见的 **误解**：

- 误解：Lambda 表达式就是一个匿名内部类
- 解答：Lambda 表达式是提供函数式接口抽象方法实现的一种更简洁、更高性能的机制，它在底层是使用 `invokedynamic` 实现的，而不是匿名内部类，并且性能远超匿名内部类

> 我目前给它的理解是，它在特定条件下，可以用来替代匿名内部类



## 2. `@FunctionalInterface` 注解

`@FunctionalInterface` 是一个可选的注解，用于在 **编译时** 强制检查一个接口是否严格符合函数式接口的定义

**作用：**

1. **编译时检查**：如果一个接口被此注解标记，但它不符合函数式接口的条件（例如，它包含了零个或多个抽象方法），编译器将会报错
2. **明确意图**：它清晰地向其他开发者表明，这个接口是设计用于函数式编程的，即期望与 Lambda 表达式或方法引用配合使用

虽然这个注解不是必需的，但强烈建议在定义函数式接口时使用它，以保证接口定义的正确性

```JAVA
@FunctionalInterface
public interface MyFunction {
    // 1. 唯一的一个抽象方法
    void execute();

    // 2. 可以有多个默认方法
    default void defaultMethod() {
        System.out.println("Default method");
    }

    // 3. 可以有多个静态方法
    static void staticMethod() {
        System.out.println("Static method");
    }
}
```

> **注意：** 函数式接口可以包含任意数量的 `default`（默认）方法和 `static`（静态）方法，因为这些方法都有具体实现，不属于抽象方法



## 3. 自定义函数式接口

我们可以根据特定需求定义自己的函数式接口

**示例1：定义一个计算器接口**

```JAVA
// 使用注解
@FunctionalInterface
public interface Calculator {
    // 抽象方法定义了“需要做什么”：接受两个int，返回一个int
    int calculate(int a, int b);
}
```



# Lambda

> 个人认为 学习Lambda 要把思考重心放到 “传递一个行为” 上
>
> 因为你又不需要去考虑它处理的参数是什么，只需要考虑它是怎么去处理这个参数的

## 1. Lambda表达式简介

### 1.1 什么是Lambda表达式

Lambda表达式（Lambda Expression）是 Java 8 引入的一个核心特性，它提供了一种清晰简洁的方式来表示 **匿名函数**，即一个没有名称的函数

Lambda允许了我们将“行为”或“逻辑”本身作为数据来传递

在Java中，Lambda表达式是作为 **函数式接口**（Functional Interface）的实例来实现的。它实现了函数式接口中定义的那个唯一的抽象方法

写一个 Lambda 表达式时，你并**不是在*调用*它**，你**只是在*定义*它**



### 1.2 为什么需要Lambda表达式

在Java 8之前，如果想将一段代码逻辑作为参数传递，通常需要使用 **匿名内部类**

**传统方式的问题：**

```java
// 使用匿名内部类实现Runnable
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};
```

这种写法的语法非常臃肿，`new Runnable() { ... }` 这部分代码是“样板代码”（Boilerplate Code），它掩盖了我们真正关心的核心逻辑：`System.out.println(...)`



**使用Lambda表达式：**

```java
// 目标：同上
Runnable runnable = () -> System.out.println("Hello World from Lambda");
```

**优势：**

- 代码更简洁，可读性更强

- 减少样板代码

- 更好地支持函数式编程

- 便于并行处理



## 2. Lambda表达式语法

### 2.1 基本语法结构

```
(参数列表) -> { 方法体 }
```

**组成部分：**

1. **参数列表**：对应函数式接口中抽象方法的参数。可以为空（`()`），也可以有多个参数（`(x, y)`）
2. **箭头符号**：`->`，这是 Lambda 表达式的操作符，用于分隔参数列表和方法体
3. **方法体**：Lambda 表达式要执行的代码逻辑，即对函数式接口抽象方法的实现



### 2.2 语法变体

Lambda 表达式的语法非常灵活，编译器会根据上下文（即目标函数式接口）进行大量的**类型推断**，允许我们在多种情况下简化写法



#### 2.2.1 无参数Lambda表达式

当接口方法没有参数时（例如 `Runnable` 的 `run()` 方法）

```java
// 目标接口
@FunctionalInterface
interface Greeter {
    void sayHello();
}
```

```java
// 完整形式：使用大括号
Greeter greeter1 = () -> { System.out.println("Hello"); };

// 简化形式：如果方法体只有一条语句，可以省略大括号 {}
Greeter greeter2 = () -> System.out.println("Hello");
```



#### 2.2.2 单参数Lambda表达式

当接口方法有且只有一个参数时

```java
// 目标接口
@FunctionalInterface
interface Consumer {
    void accept(String s);
}
```

```java
// 完整形式：显式声明参数类型
Consumer c1 = (String s) -> { System.out.println(s); };

// 简化形式1：省略参数类型（类型推断）
// 编译器知道 Consumer 的 accept 方法需要一个 String，因此 s 被自动推断为 String
Consumer c2 = (s) -> System.out.println(s);

// 简化形式2：省略参数括号
// 当且仅当只有一个参数，且省略了参数类型时，可以省略参数列表的括号
Consumer c3 = s -> System.out.println(s);
```



#### 2.2.3 多参数Lambda表达式

当接口方法有两个或多个参数时（例如 `Calculator` 的 `calculate(int a, int b)` 方法）

```java
// 目标接口
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

```java
// 完整形式：显式声明参数类型
Calculator add1 = (int a, int b) -> { return a + b; };

// 简化形式：省略参数类型（类型推断）
// 编译器从 calculate(int a, int b) 推断出 a 和 b 都是 int
Calculator add2 = (a, b) -> { return a + b; };

// 注意：多参数时，参数列表的括号 (a, b) 永远不能省略
```



#### 2.2.4 有返回值的Lambda表达式

当接口方法有返回值时（例如 `Calculator` 返回 `int`）

```java
// 完整形式：方法体是代码块，必须使用大括号 {} 和 显式 return
Calculator multiply1 = (a, b) -> {
    int result = a * b;
    return result;
};

// 简化形式：隐式 return
// 如果方法体只有 *一条表达式*（注意：不是一条语句），可以省略大括号 {}
// 此时，该表达式的计算结果将作为方法的返回值被 *自动（隐式）返回*。
Calculator multiply2 = (a, b) -> a * b;
```

>**重要规则**：如果使用了大括号 `{}`，就必须显式使用 `return` 来返回值（除非方法返回值是 `void`）。如果没使用大括号，则不能使用 `return`



### 2.3 语法规则总结表

| 语法部分         | 规则                                    | 示例（左侧为原始，右侧为简化）                               |
| ---------------- | --------------------------------------- | ------------------------------------------------------------ |
| **参数列表**     |                                         |                                                              |
| 无参数           | 括号 `()` 不能省略                      | `() -> ...`                                                  |
| 单个参数         | 括号 `()` 可以省略（前提是省略了类型）  | `(s) -> ...` 变为 `s -> ...`                                 |
| 多个参数         | 括号 `()` 不能省略                      | `(x, y) -> ...`                                              |
| **参数类型**     |                                         |                                                              |
| 所有参数         | 类型可省略（编译器自动推断）            | `(int x, int y) -> ...` 变为 `(x, y) -> ...`                 |
| **方法体**       |                                         |                                                              |
| 单条语句         | 大括号 `{}` 可以省略                    | `() -> { System.out.println("Hi"); }` 变为 `() -> System.out.println("Hi")` |
| 多条语句         | 大括号 `{}` 不能省略                    | `(x, y) -> { int sum = x+y; return sum; }`                   |
| **返回值**       |                                         |                                                              |
| 方法体是单表达式 | `return` 和 `{}` 都必须省略（隐式返回） | `(x, y) -> { return x + y; }` 变为 `(x, y) -> x + y`         |
| 方法体是代码块   | `return` 必须显式声明（除非 `void`）    | `(x, y) -> { int sum = x+y; return sum; }`                   |



## 3. 常用的函数式接口

Java 8 在 `java.util.function` 包中提供了大量常用的函数式接口，以满足绝大多数日常开发需求

### 3.1 `Function<T, R>` - 转换型

**作用：** 接收一个 T 类型的参数，消费后，返回一个 R 类型的结果。**核心职责是“转换”**

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```



**示例：**

```java
// 示例1：将字符串转换为其长度（Integer）
Function<String, Integer> strLength = s -> s.length();
System.out.println(strLength.apply("Hello")); 		// 输出: 5

// 示例2：计算数字的平方
Function<Integer, Integer> square = x -> x * x;
System.out.println(square.apply(5)); 				// 输出: 25
```



**默认方法**

`Function` 接口提供了两个强大的默认方法，用于组合多个函数：

```java
Function<Integer, Integer> multiplyBy2 = x -> x * 2;
Function<Integer, Integer> add3 = x -> x + 3;

// 1. compose: g.compose(f) -> g(f(x))
// 先执行参数函数(add3)，再执行调用函数(multiplyBy2)
Function<Integer, Integer> composed = multiplyBy2.compose(add3);
System.out.println(composed.apply(5)); // 先 5+3=8, 再 8*2=16

// 2. andThen: f.andThen(g) -> g(f(x))
// 先执行调用函数(multiplyBy2)，再执行参数函数(add3)
Function<Integer, Integer> chained = multiplyBy2.andThen(add3);
System.out.println(chained.apply(5)); // 先 5*2=10, 再 10+3=13
```



### 3.2 `Predicate<T>` - 判断型

**作用：** 接收一个 T 类型的参数，返回一个 `boolean` 值。**核心职责是“判断”或“测试”**

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```



**示例：**

```java
// 示例1：判断整数是否为正数
Predicate<Integer> isPositive = x -> x > 0;
System.out.println(isPositive.test(5));  // true
System.out.println(isPositive.test(-3)); // false

// 示例2：判断字符串是否为空
Predicate<String> isEmpty = s -> s.isEmpty();
System.out.println(isEmpty.test(""));    // true
System.out.println(isEmpty.test("Hi"));  // false
```



**默认方法**

`Predicate` 接口提供了用于组合逻辑的默认方法：

```java
Predicate<Integer> isEven = x -> x % 2 == 0;
Predicate<Integer> isPositive = x -> x > 0;

// 1. and: 逻辑与 (&&)
Predicate<Integer> isPositiveEven = isEven.and(isPositive);
System.out.println(isPositiveEven.test(4));  // true (正数 且 偶数)
System.out.println(isPositiveEven.test(-4)); // false (不是正数)
System.out.println(isPositiveEven.test(3));  // false (不是偶数)

// 2. or: 逻辑或 (||)
Predicate<Integer> isEvenOrPositive = isEven.or(isPositive);
System.out.println(isEvenOrPositive.test(3));  // true (是正数)
System.out.println(isEvenOrPositive.test(-2)); // true (是偶数)

// 3. negate: 逻辑非 (!)
Predicate<Integer> isOdd = isEven.negate();
System.out.println(isOdd.test(3)); // true (不是偶数)
```



### 3.3 `Consumer<T>` - 消费型

**作用：** 接收一个 T 类型的参数，执行某个操作，没有返回值 (`void`)。**核心职责是“消费”**（即处理数据，但不返回结果）

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```



**示例：**

```java
// 示例1：打印字符串
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hello World"); // 输出: Hello World

// 示例2：在集合的 forEach 中最常用
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(name -> System.out.println("Hello, " + name));
```



**默认方法**

```java
Consumer<String> c1 = s -> System.out.print(s.toUpperCase());
Consumer<String> c2 = s -> System.out.println(" - processed");

// 1. andThen: 链式调用
// 先执行 c1，再执行 c2
Consumer<String> combined = c1.andThen(c2);
combined.accept("hello"); // 输出: HELLO - processed
```



### 3.4 `Supplier<T>` - 供给型

**作用：** 不接收任何参数，返回一个 T 类型的结果。**核心职责是“提供”或“生产”**（例如用于创建对象或生成数据）

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

**示例：**

```java
// 示例1：提供一个固定的字符串
Supplier<String> stringSupplier = () -> "Hello World";
System.out.println(stringSupplier.get()); // 输出: Hello World

// 示例2：提供一个随机数
Supplier<Double> randomSupplier = () -> Math.random();
System.out.println(randomSupplier.get()); // 输出: 0.123... (随机数)

// 示例3：提供当前时间（方法引用写法，后续会讲）
Supplier<LocalDateTime> timeSupplier = LocalDateTime::now;
System.out.println(timeSupplier.get()); // 输出: 2023-10-27T10:30:00...
```



## 4. 扩展函数式接口

### 4.1 “Bi”系列接口（双参数）

`Bi` 接口是核心接口的双参数版本

#### 4.1.1 `BiFunction<T, U, R>`

**作用：** `Function` 的双参数版本。接收两个类型可能不同的参数（T 和 U），返回一个结果（R）

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}
```



**示例**

```java
// 示例1：
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(5, 3)); // 输出: 8

// 示例2：
BiFunction<String, String, String> concat = (s1, s2) -> s1 + s2;
System.out.println(concat.apply("Hello", " World")); // 输出: Hello World
```



#### 4.1.2 `BiConsumer<T, U>`

**作用：** `Consumer` 的双参数版本。接收两个参数，无返回值 (`void`)

```java
@FunctionalInterface
public interface BiConsumer<T, U> {
    void accept(T t, U u);
}
```



**示例**

```java
// 示例1：
BiConsumer<String, Integer> printer = (name, age) -> 
    System.out.println(name + " is " + age + " years old");
printer.accept("Alice", 25); // 输出: Alice is 25 years old

// 示例2：遍历Map
Map<String, Integer> map = new HashMap<>();
map.put("Alice", 25);
map.put("Bob", 30);
// Map.forEach 接受一个 BiConsumer
map.forEach((key, value) -> System.out.println(key + ": " + value));
```



#### 4.1.3 `BiPredicate<T, U>`

**作用：** `Predicate` 的双参数版本。接收两个参数，返回 `boolean` 值

```java
@FunctionalInterface
public interface BiPredicate<T, U> {
    boolean test(T t, U u);
}
```

```java
// 示例：
BiPredicate<String, Integer> isLengthEqual = (s, len) -> s.length() == len;
System.out.println(isLengthEqual.test("Hello", 5)); // true
System.out.println(isLengthEqual.test("Hi", 5));    // false
```



### 4.2 “Operator”系列接口（同类型操作）

Operator 接口是一种特殊的 `Function`，它们要求 **所有参数类型和返回类型必须相同**

#### 4.2.1 `UnaryOperator<T>`

**作用：** `Function<T, T>` 的特殊形式。接收一个 T 类型参数，返回一个 T 类型结果（一元操作）

```java
// UnaryOperator<T> 继承自 Function<T, T>
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    // 它没有自己的抽象方法，复用 Function 的 apply 方法
}
```



**示例**

```java
// 示例1：
UnaryOperator<Integer> square = x -> x * x;
System.out.println(square.apply(5)); // 输出: 25

// 示例2：
UnaryOperator<String> toUpper = String::toUpperCase;
System.out.println(toUpper.apply("hello")); // 输出: HELLO
```



#### 4.2.2 `BinaryOperator<T>`

**作用：** `BiFunction<T, T, T>` 的特殊形式。接收两个 T 类型参数，返回一个 T 类型结果（二元操作）

```java
// BinaryOperator<T> 继承自 BiFunction<T, T, T>
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    // 它没有自己的抽象方法，复用 BiFunction 的 apply 方法
}
```



**示例**

```java
// 示例1：
BinaryOperator<Integer> add = (a, b) -> a + b;
System.out.println(add.apply(5, 3)); // 输出: 8

// 示例2：
BinaryOperator<String> concat = (s1, s2) -> s1 + s2;
System.out.println(concat.apply("Hello", " World")); // 输出: Hello World

// 示例3：常用于 Stream 的 reduce 操作
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
// (a, b) -> a + b 就是一个 BinaryOperator
int sum = numbers.stream().reduce(0, (a, b) -> a + b); 
System.out.println(sum); // 输出: 15
```



## 5. 基本类型函数式接口

### 概述

**为什么需要基本类型接口？**

- Java 有 8 种基本数据类型（`int`, `double`, `long` 等）和它们对应的包装类（`Integer`, `Double`, `Long` 等）

- **性能开销问题：** 如果我们使用标准函数式接口，例如 `Function<Integer, Integer>`，在处理大量 `int` 数据时，会发生：

  1. **自动装箱**: 将 `int` 转换为 `Integer` 对象
  2. **自动拆箱:** 将 `Integer` 对象转回 `int`

  这种频繁的对象创建和垃圾回收会带来显著的性能开销，尤其是在高性能计算或大规模数据处理中

**解决方案：** 

- 为了避免这种开销，`java.util.function` 包为 `int`, `long`, `double` 三种最常用的基本类型提供了专门的函数式接口

  这些接口直接操作基本类型，完全避免了装箱和拆箱



### 5.1 基本类型 Predicate (判断)

**核心作用：** 对 `int`, `long`, `double` 类型的输入值进行判断，返回 `boolean` 结果。用于替代 `Predicate<Integer>` 等包装类版本

#### 5.1.1 `IntPredicate`

**接口定义：**

```JAVA
@FunctionalInterface
public interface IntPredicate {
    /**
     * 对给定的 int 参数进行判断
     * @param value the input argument
     * @return true if the input argument matches the predicate,
     * otherwise false
     */
    boolean test(int value);
    
    // 包含 and, or, negate 默认方法
}
```



**使用示例：**

```JAVA
// 1. 判断是否为偶数
IntPredicate isEven = x -> x % 2 == 0;
System.out.println("4是偶数吗? " + isEven.test(4)); // true
System.out.println("5是偶数吗? " + isEven.test(5)); // false

// 2. 结合默认方法
IntPredicate isPositive = x -> x > 0;
IntPredicate isPositiveEven = isPositive.and(isEven);
System.out.println("4是正偶数吗? " + isPositiveEven.test(4));  // true
System.out.println("-4是正偶数吗? " + isPositiveEven.test(-4)); // false
```



#### 5.1.2 `LongPredicate`

**接口定义：**

```JAVA
@FunctionalInterface
public interface LongPredicate {
    /**
     * 对给定的 long 参数进行判断
     */
    boolean test(long value);
    
    // ... 
}
```



**使用示例：**

```JAVA
// 判断一个 long 类型的数字是否大于 10 亿
LongPredicate isOverBillion = num -> num > 1_000_000_000L;
long largeNum = 1_500_000_000L;
System.out.println(largeNum + " 大于10亿吗? " + isOverBillion.test(largeNum)); // true
```



#### 5.1.3 `DoublePredicate`

**接口定义：**

```JAVA
@FunctionalInterface
public interface DoublePredicate {
    /**
     * 对给定的 double 参数进行判断
     */
    boolean test(double value);
    
    // ...
}
```



**使用示例：**

```JAVA
// 判断一个 double 是否在 (0.0, 1.0) 开区间内
DoublePredicate isBetweenZeroAndOne = d -> d > 0.0 && d < 1.0;
System.out.println("0.5 在 (0, 1) 之间吗? " + isBetweenZeroAndOne.test(0.5));  // true
System.out.println("1.5 在 (0, 1) 之间吗? " + isBetweenZeroAndOne.test(1.5)); // false
```



### 5.2 基本类型 Consumer (消费)

**核心作用：**接收一个 `int`, `long`, `double` 类型的参数，执行某个操作，但不返回任何结果（“消费”掉这个参数），用于替代 `Consumer<Integer>` 等包装类版本

#### 5.2.1 `IntConsumer`

**接口定义：**

```JAVA
@FunctionalInterface
public interface IntConsumer {
    /**
     * 对给定的 int 参数执行操作
     * @param value the input argument
     */
    void accept(int value);
    
    /**
     * 返回一个组合的 IntConsumer，
     * 它会先执行当前 Consumer 的操作，然后再执行 after 的操作
     */
    default IntConsumer andThen(IntConsumer after) {
        // ...
    }
}
```



**使用示例：**

```JAVA
// 1. 打印值
IntConsumer printer = x -> System.out.println("Value: " + x);
printer.accept(42); // 输出: Value: 42

// 2. 结合 andThen 链式调用
IntConsumer doubleIt = x -> System.out.println("Doubled: " + (x * 2));
IntConsumer printAndDouble = printer.andThen(doubleIt);

printAndDouble.accept(10);
// 输出:
// Value: 10
// Doubled: 20
```



#### 5.2.2 `LongConsumer`

**接口定义：**

```JAVA
@FunctionalInterface
public interface LongConsumer {
    /**
     * 对给定的 long 参数执行操作
     */
    void accept(long value);
    
    // ...
}
```



**使用示例：**

```JAVA
// 模拟将日志写入数据库（这里用打印代替）
LongConsumer logToDb = id -> {
    System.out.println("[LOG] Processing ID: " + id);
    // 假设这里有数据库写入操作...
};

logToDb.accept(9876543210L); // 输出: [LOG] Processing ID: 9876543210
```



#### 5.2.3 `DoubleConsumer`

**接口定义：**

```JAVA
@FunctionalInterface
public interface DoubleConsumer {
    /**
     * 对给定的 double 参数执行操作
     */
    void accept(double value);
    
    // ...
}
```



**使用示例：**

```JAVA
// 计算并打印平方根
DoubleConsumer printSqrt = d -> {
    if (d >= 0) {
        System.out.println("平方根: " + Math.sqrt(d));
    } else {
        System.out.println("负数没有实数平方根");
    }
};

printSqrt.accept(64.0); // 输出: 平方根: 8.0
printSqrt.accept(-9.0); // 输出: 负数没有实数平方根
```



### 5.3 基本类型 Supplier (供应)

**核心作用：** 不接收任何参数，返回一个 `int`, `long`, `double` 类型的基本数据。用于替代 `Supplier<Integer>` 等包装类版本

**关键区别：** 它们的方法名不是 `get()`，而是 `getAsInt()`, `getAsLong()`, `getAsDouble()`

#### 5.3.1 `IntSupplier`

**接口定义：**

```JAVA
@FunctionalInterface
public interface IntSupplier {
    /**
     * 获取一个 int 类型的结果
     * @return an int result
     */
    int getAsInt();
}
```



**使用示例：**

```JAVA
// 1. 提供一个 0-99 的随机整数
IntSupplier randomIntSupplier = () -> (int)(Math.random() * 100);
System.out.println("随机整数: " + randomIntSupplier.getAsInt()); 

// 2. 提供一个固定的计数值
final int currentCounter = 1024;
IntSupplier counter = () -> currentCounter;
System.out.println("当前计数值: " + counter.getAsInt()); // 输出: 1024
```



#### 5.3.2 `LongSupplier`

**接口定义：**

```JAVA
@FunctionalInterface
public interface LongSupplier {
    /**
     * 获取一个 long 类型的结果
     */
    long getAsLong();
}
```

**使用示例：**

```JAVA
// 1. 提供当前的毫秒时间戳
LongSupplier timestampSupplier = () -> System.currentTimeMillis();
System.out.println("当前时间戳: " + timestampSupplier.getAsLong());

// 2. 模拟从配置中读取一个 long 型 ID
LongSupplier configId = () -> 987654321000L; // 假设这是从某处读取的
System.out.println("配置ID: " + configId.getAsLong());
```



#### 5.3.3 `DoubleSupplier`

**接口定义：**

```JAVA
@FunctionalInterface
public interface DoubleSupplier {
    /**
     * 获取一个 double 类型的结果
     */
    double getAsDouble();
}
```

**使用示例：**

```JAVA
// 1. 提供一个高精度的随机数 (0.0 到 1.0)
DoubleSupplier randomDouble = () -> Math.random();
System.out.println("随机数: " + randomDouble.getAsDouble());

// 2. 提供一个常量，如圆周率
DoubleSupplier piSupplier = () -> Math.PI;
System.out.println("圆周率: " + piSupplier.getAsDouble());
```



### 5.4 基本类型 Function (转换)

这类接口专用于处理基本类型与对象（泛型 `T` 或 `R`）之间的转换，是 `Function<T, R>` 针对基本类型的特化版本



**分为两大类：**

1. **`XxxFunction<R>`**：接收一个基本类型参数，返回一个对象 `R`
2. **`ToXxxFunction<T>`**：接收一个对象 `T`，返回一个基本类型结果



#### 5.4.1 `XxxFunction<R>` (基本类型 -> 对象)

**核心作用：** 将 `int`, `long`, `double` 转换为任意对象 `R`。 **方法名：** `apply(value)`

- **`IntFunction<R>`**: `int -> R`
- **`LongFunction<R>`**: `long -> R`
- **`DoubleFunction<R>`**: `double -> R`



**`IntFunction<R>` 示例**

**接口定义：**

```java
@FunctionalInterface
public interface IntFunction<R> {
    /**
     * 将 int 参数应用于此函数
     * @param value the function argument
     * @return the function result
     */
    R apply(int value);
}
```



**使用示例：**

```java
// 1. 将 int 转换为 String
IntFunction<String> intToString = x -> "Number:" + x;
System.out.println(intToString.apply(5)); // 输出: Number:5

// 2. 将 int 转换为一个自定义对象 (例如 数组)
IntFunction<int[]> createArray = size -> new int[size];
int[] newArr = createArray.apply(10);
System.out.println("创建的数组长度: " + newArr.length); // 输出: 10
```

*( `LongFunction<R>` 和 `DoubleFunction<R>` 用法类似 )*



#### 5.4.2 `ToXxxFunction<T>` (对象 -> 基本类型)

**核心作用：** 将任意对象 `T` 转换为 `int`, `long`, `double`

**关键区别：** 它们的方法名不是 `apply()`，而是 `applyAsInt()`, `applyAsLong()`, `applyAsDouble()`

- **`ToIntFunction<T>`**: `T -> int`
- **`ToLongFunction<T>`**: `T -> long`
- **`ToDoubleFunction<T>`**: `T -> double`



**`ToIntFunction<T>` 示例**

**接口定义：**

```java
@FunctionalInterface
public interface ToIntFunction<T> {
    /**
     * 将对象 T 应用于此函数，返回一个 int 结果
     * @param value the function argument
     * @return the function result
     */
    int applyAsInt(T value);
}
```



**使用示例：**

```java
// 1. 获取字符串的长度
ToIntFunction<String> stringLength = s -> s.length();
System.out.println("Hello 的长度: " + stringLength.applyAsInt("Hello")); // 输出: 5

// 2. 获取自定义对象的某个 int 属性
class User {
    private int age;
    public User(int age) { this.age = age; }
    public int getAge() { return age; }
}

ToIntFunction<User> getUserAge = user -> user.getAge();
User user = new User(30);
System.out.println("用户年龄: " + getUserAge.applyAsInt(user)); // 输出: 30
```

*( `ToLongFunction<T>` 和 `ToDoubleFunction<T>` 用法类似 )*



### 5.6 基本类型 Operator (运算)

`Operator` 接口是 `Function` 接口的特化版本，专用于“参数类型和返回值类型相同”的场景。基本类型 Operator 则是专用于 `int`, `long`, `double` 类型的运算

**关键区别：** 它们的方法名是 `applyAsInt()`, `applyAsLong()`, `applyAsDouble()`

#### 5.6.1 `XxxUnaryOperator` (一元运算)

**核心作用：** 接收一个基本类型参数，返回一个**相同**的基本类型结果。 它等效于 `IntFunction<Integer>` 这种形式，但避免了装箱

- **`IntUnaryOperator`**: `int -> int`
- **`LongUnaryOperator`**: `long -> long`
- **`DoubleUnaryOperator`**: `double -> double`



**`IntUnaryOperator` 示例**

**接口定义：**

```java
@FunctionalInterface
public interface IntUnaryOperator {
    /**
     * 将 int 参数应用于此操作
     * @param operand the operator argument
     * @return the operator result
     */
    int applyAsInt(int operand);
    
    // 同样包含 compose 和 andThen 默认方法
}
```



**使用示例：**

```java
// 1. 计算平方
IntUnaryOperator square = x -> x * x;
System.out.println("5的平方: " + square.applyAsInt(5)); // 输出: 25

// 2. 计算自增
IntUnaryOperator increment = x -> x + 1;
System.out.println("5自增: " + increment.applyAsInt(5)); // 输出: 6
```

*( `LongUnaryOperator` 和 `DoubleUnaryOperator` 用法类似 )*

#### 5.6.2 `XxxBinaryOperator` (二元运算)

**核心作用：** 接收两个**相同**的基本类型参数，返回一个**相同**的基本类型结果。 它等效于 `BiFunction<Integer, Integer, Integer>`，但避免了装箱

- **`IntBinaryOperator`**: `(int, int) -> int`
- **`LongBinaryOperator`**: `(long, long) -> long`
- **`DoubleBinaryOperator`**: `(double, double) -> double`



**`IntBinaryOperator` 示例**

**接口定义：**

```java
@FunctionalInterface
public interface IntBinaryOperator {
    /**
     * 将两个 int 参数应用于此操作
     * @param left the first operator argument
     * @param right the second operator argument
     * @return the operator result
     */
    int applyAsInt(int left, int right);
}
```



**使用示例：**

```java
// 1. 计算两数之和
IntBinaryOperator add = (a, b) -> a + b;
System.out.println("5 + 3 = " + add.applyAsInt(5, 3)); // 输出: 8

// 2. 计算两数中的较大值
IntBinaryOperator max = (a, b) -> a > b ? a : b;
System.out.println("5 和 3 中的较大值: " + max.applyAsInt(5, 3)); // 输出: 5
```

*( `LongBinaryOperator` 和 `DoubleBinaryOperator` 用法类似 )*



# 方法引用

## 1. 什么是方法引用

> Method Reference

方法引用是 Lambda 表达式的一种 **语法糖**

当一个 Lambda 表达式的实现逻辑 **仅仅是调用一个已经存在的方法**，并且该方法的 **参数列表和返回值类型** 与 Lambda 表达式所实现的函数式接口的抽象方法 **一致** 时，就可以使用方法引用来进一步简化代码

方法引用的所有“动态参数”都必须由函数式接口的抽象方法来提供

> 但是它 **不是一个简单的直接方法调用** ，而是通过一个更高级的优化机制来实现的



**核心思想：** 直接引用现有的方法来“实现”函数式接口



## 2. 方法引用的类型

### 2.1 静态方法引用

**语法：** `类名::静态方法名`

**适用情况：** Lambda 表达式的主体只是调用了一个静态方法，并且 Lambda 的参数列表与该静态方法的参数列表完全一致

**示例：** `Function<String, Integer>` 接口的抽象方法是 `Integer apply(String t)`

**Lambda 表达式**

```java
// Lambda 表达式：
// 参数 s 被直接传递给了 Integer.parseInt(s)
Function<String, Integer> parserLambda = s -> Integer.parseInt(s);
// 使用
System.out.println(parserRef.apply("123")); // 输出: 123
```



**方法引用**

```java
// 方法引用：
// 编译器推断出参数 s 应该被传递给 Integer.parseInt 方法
Function<String, Integer> parserRef = Integer::parseInt;
// 使用
System.out.println(parserRef.apply("123")); // 输出: 123
```



### 2.2 实例方法引用

**语法：** `对象实例名::实例方法名`

**适用情况：** Lambda 表达式的主体只是调用了一个 **已存在的、特定对象的** 实例方法，并且 Lambda 的参数列表与该实例方法的参数列表完全一致

**核心思想：** 您已经有了一个明确的对象（如 `System.out`），您想让 Lambda 的参数被这个对象的某个方法（如 `println`）所处理



**基础示例：`System.out::println`**

- **函数式接口：** `Consumer<String>`
- **抽象方法：** `void accept(String t)`
- **特定对象：** `System.out` (一个 `PrintStream` 实例)
- **目标方法：** `System.out` 对象的 `println(String s)` 方法

`accept(String t)` 的签名（接收一个 `String`，无返回）与 `println(String s)` 的签名完美匹配

```java
// Lambda 表达式：
// 参数 s 被直接传递给了 System.out.println(s)
Consumer<String> printerLambda = s -> System.out.println(s);

// 方法引用：
// 编译器推断出 Consumer<String> 的参数 s，
// 应该被传递给 System.out 对象的 println 方法
Consumer<String> printerRef = System.out::println;

// 使用
printerRef.accept("Hello World"); // 输出: Hello World

// 集合遍历中的应用
List<String> names = Arrays.asList("Alice", "Bob");
// names.forEach(s -> System.out.println(s))
names.forEach(System.out::println);
```



**特殊实例：`this::成员方法名`**

- 这只是上述情况的一个特例，“特定对象实例名”恰好是 `this`（即当前类的实例）

- **适用情况：** 在一个类的实例方法内部，需要将 **当前实例** 的 **另一个方法** 作为 Lambda 表达式传递时

```java
public class MyProcessor {

    // 这是一个普通的实例方法
    public void processString(String s) {
        System.out.println("Processing: " + s);
    }

    // 这是另一个实例方法，它需要使用 Lambda
    public void doWork(List<String> list) {
        
        // 传统 Lambda (明确使用 this)
        list.forEach(s -> this.processString(s));
        
        // 方法引用：
        // "this" 就是那个 "特定对象实例"
        list.forEach(this::processString); 
    }
}
```



**特殊实例：`super::成员方法名`**

- 这是另一个更特殊的特例，“特定对象实例名”是 `super`（即指向父类的引用）

- **适用情况：** 在子类中，您想引用的不是当前类重写（override）的方法，而是 **父类的原始实现** 时

```java
class BaseProcessor {
    public void process(String s) {
        System.out.println("Base Processing: " + s);
    }
}

class SubProcessor extends BaseProcessor {

    @Override
    public void process(String s) {
        System.out.println("Sub Processing: " + s);
    }

    // 这个方法想专门调用父类的 process
    public void doBaseWork(List<String> list) {
        
        // 传统 Lambda
        list.forEach(s -> super.process(s));
        
        // 方法引用：
        // "super" 就是那个 "特定对象实例"
        list.forEach(super::process);
    }
}
```



### 2.3 类的实例方法引用

**语法：** `类名::实例方法名`

**适用情况：** 这是 **最特殊** 的一种

- 适用于这种情况：需要一个 Lambda 表达式，而这个 Lambda 做的**唯一**一件事，就是 **拿它的第一个参数** 去 **调用一个实例方法** ，并把 **剩余的参数** 传给那个方法

  > 是不是很难理解，是的，别的都符合我们对 “方法调用” 的直觉，而这个很难理解，是因为它不一样
  >
  > - 我是这样进行理解的：
  >
  >   我们知道，引用一个方法有两种常见形式：
  >
  >   1. **静态方法**：`类名::静态方法` (如 `Integer::parseInt`)
  >   2. **特定实例的方法**：`对象名::实例方法` (如 `System.out::println`)
  >
  >   这两种都好理解
  >
  >   但如果我们想调用类似于 `String` 的 `toUpperCase()` 的这种方法,这种方法是必须用一个实例对象去调用的，也就是需要一个 `String` 类型的对象
  >
  >   - 而我们手头**没有**一个像 `System.out` 那样现成的、特定的 `String` 对象，但是我们还想使用方法引用来方方便便的写代码，咋办呀
  >
  >     **那我们应该怎么做呢？**
  >
  >     我们可以自己写一个静态方法嘛，因为静态方法的这种调用最简单:
  >
  >     1. 这个静态方法的 **第一个参数** ，就用来接收未来的“调用者”对象
  >     2. **后续参数** 和那个实例方法的参数列表保持一致
  >     3. 在方法体里，用第一个参数去调用实例方法
  >     4. 成功！！！
  >
  >   **⭐妈呀，麻烦死我了**
  >
  >   我服了，我们总不能为每个想引用的实例方法，都去创建一个对应的、多一个“调用者”参数的静态辅助方法吧
  >
  >   - **是的**
  >
  >     Java 语言的设计者早就想到了这一步
  >
  >     `类名::实例方法名` 这种语法，就是帮我们 **在底层自动完成了“将第一个参数转变为调用者”的逻辑**
  >
  >     现在，你只需要关心：**函数式接口的抽象方法”和“⭐我们的逻辑”能不能对应上** 就行，毕竟加入了一个“调用者”的参数嘛，这个还是很重要的hhh
  >
  >   **完美!**

> 叽里咕噜的说的很麻烦，直接这样说叭：
>
> - s1 -> s1.A()		        **等价于**	ClassName::A
> - (s1, s2) -> s1.A(s2)	        **等价于**	ClassName::A
> - (s1, s2, s3) -> s1.A(s2, s3)	**等价于**        ClassName::A

> 编译器识别到使用的是`类名::成员方法`的这种形式，就会去进行特殊的处理
>
> - 把 Lambda 的第一个参数当作“调用者”，后面的参数当作“方法参数”，前面已经说过



**示例 1：Lambda 有两个参数 (例如 `Comparator<T>`)**

- **函数式接口：** `Comparator<String>`
- **抽象方法：** `int compare(String s1, String s2)`

```java
// Lambda 表达式：
// 第一个参数 s1 成为了方法的调用者
// 第二个参数 s2 成为了方法的参数
Comparator<String> comparatorLambda = (s1, s2) -> s1.compareToIgnoreCase(s2);

// 使用
List<String> names = Arrays.asList("charlie", "Alice", "BOB");
names.sort(String::compareToIgnoreCase);
// 结果: [Alice, BOB, charlie]
```

```java
// 方法引用：
// 编译器识别到这种模式：
// 1. 这是一个 Comparator<String>，需要一个 (String, String) -> int 的实现。
// 2. String::compareToIgnoreCase 是一个 (String) -> int 的实例方法。
// 3. 编译器自动将 Lambda 的 (s1, s2) 映射为 s1.compareToIgnoreCase(s2)。
Comparator<String> comparatorRef = String::compareToIgnoreCase;

// 使用
List<String> names = Arrays.asList("charlie", "Alice", "BOB");
names.sort(String::compareToIgnoreCase);
// 结果: [Alice, BOB, charlie]
```



**示例 2：Lambda 只有一个参数 (例如 `Function<T, R>`)**

- **函数式接口：** `Function<String, Integer>`
- **抽象方法：** `Integer apply(String s)`

```java
// Lambda 表达式：
// 第一个 (也是唯一一个) 参数 s 成为调用者
// 该方法 (length) 没有参数
Function<String, Integer> lengthLambda = s -> s.length();

// 使用
System.out.println(lengthRef.apply("Hello")); // 5
```

```java
// 方法引用：
// 编译器识别到这种模式：
// 1. 这是一个 Function<String, Integer>，需要一个 (String) -> Integer 的实现
// 2. String::length 是一个 () -> int 的实例方法。
// 3. 编译器自动将 Lambda 的 (s) 映射为 s.length()
Function<String, Integer> lengthRef = String::length;

// 使用
System.out.println(lengthRef.apply("Hello")); // 5
```



### 2.4 构造方法引用

**语法：** `类名::new`

**适用情况：** Lambda 表达式的主体只是调用了一个类的构造方法。编译器会自动匹配函数式接口的抽象方法签名，去寻找并调用对应的构造方法

**示例 1：无参构造**

- **函数式接口：** `Supplier<ArrayList<String>>`
- **抽象方法：** `ArrayList<String> get()`

```java
// Lambda 表达式：
Supplier<ArrayList<String>> listSupplierLambda = () -> new ArrayList<>();

// 使用
List<String> list = listSupplierRef.get();
```

```java
// 方法引用：
// 编译器匹配到 get() 方法的签名 (无参，返回 ArrayList)，
// 并自动调用 ArrayList 的无参构造函数
Supplier<ArrayList<String>> listSupplierRef = ArrayList::new;

// 使用
List<String> list = listSupplierRef.get();
```



**示例 2：带参构造**

- **函数式接口：** `BiFunction<String, Integer, Person>`
- **抽象方法：** `Person apply(String s, Integer i)`

```java
public class Person {
    private String name;
    private int age;
    
    // 构造函数
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
        System.out.println("创建了 Person: " + name + ", " + age + "岁");
    }
}
```

```java
// Lambda 表达式：
BiFunction<String, Integer, Person> personFactoryLambda = (name, age) -> new Person(name, age);

// 使用
Person person = personFactoryRef.apply("Alice", 25); 
// 输出: 创建了 Person: Alice, 25岁
```

```java
// 方法引用：
// 编译器匹配到 apply(String, Integer) 的签名，
// 并自动调用 Person(String, int) 构造函数 (会自动拆箱 Integer 为 int)
BiFunction<String, Integer, Person> personFactoryRef = Person::new;

// 使用
Person person = personFactoryRef.apply("Alice", 25); 
// 输出: 创建了 Person: Alice, 25岁
```



**示例 3：数组构造方法引用**

- **函数式接口：** `Function<Integer, Integer[]>`
- **抽象方法：** `Integer[] apply(Integer size)`

```java
// Lambda 表达式：
// 接收一个 size，返回一个指定大小的数组
Function<Integer, Integer[]> arrayCreatorLambda = (size) -> new Integer[size];

// 使用
Integer[] myArray = arrayCreatorRef.apply(10); 
System.out.println(myArray.length); // 输出: 10
```

```java
// 方法引用：
// 编译器匹配到 apply(Integer) 的签名，
// 并自动调用数组的构造方法 (new Integer[size])
Function<Integer, Integer[]> arrayCreatorRef = Integer[]::new;

// 使用
Integer[] myArray = arrayCreatorRef.apply(10); 
System.out.println(myArray.length); // 输出: 10
```



# Lambda 变量捕获

Lambda 表达式一个强大的特性是它可以“捕获”其定义范围（Enclosing Scope）内的变量。但是，这种访问不是无限制的，特别是对局部变量

## 1. 访问范围与"Effectively Final"限制

Lambda 表达式一个强大的特性是它可以“捕获”其定义范围（Enclosing Scope）内的变量



### 1.1 Lambda 可以访问什么？

- Lambda 表达式可以无限制地访问：

  1. **静态变量**（类的静态成员）	
  2. **实例变量**（类的成员变量）

  但它在访问 **局部变量**（Local Variables，即方法内的变量）时，有一个特殊且严格的限制



### 1.2 核心限制：`final`或`Effectively Final`

局部变量必须是 `final` 或 `Effectively Final`，Lambda 才可以访问，否则不可以



**a. 什么是 `final`？** 

明确地在变量前写了 `final` 关键字，表示它不可变

```java
final String prefix = "Hello ";
Consumer<String> greeter = name -> System.out.println(prefix + name);
```



**b. 什么是 `Effectively Final`？**

就是**事实上的 `final`**，意思是你虽然 **没有** 写 `final` 关键字，但是这个变量在**初始化之后，再也没有被修改（重新赋值）过**。编译器会“视其为” `final`



这是 Java 8 引入的语法糖，目的是为了让代码更简洁，不必到处写 `final`

```java
public void example() {
    // 1. "prefix" 在这里初始化
    String prefix = "Hello "; 
    
    // prefix = "Hi "; // 编译错误！"Variable used in lambda expression should be final or effectively final"
    
    // 2. Lambda 访问了它
    Consumer<String> greeter = name -> System.out.println(prefix + name);
    greeter.accept("Alice"); // 输出: Hello Alice
    
    // 3. 如果您试图在 Lambda 访问 *之后*（或之前）修改它，
    //    它就不再是 "Effectively Final"
    
    // prefix = "Hi "; // 编译错误！"Variable used in lambda expression should be final or effectively final"
}
```



### 1.3 为什么会有这个限制？

这个限制是理解变量捕获的关键，它源于 **变量生命周期** 的根本不同

`final` 或 `Effectively Final` 的限制的根本原因在于**局部变量与 Lambda 表达式自身生命周期的差异**，以及 Java 采取的**捕获机制**



#### a. 变量的“存活区域”与“生命周期”差异

在 Java 中，不同类型的变量存活在不同的内存区域，拥有不同的生命周期：

- **局部变量 (Local Variables):** 存活在方法的 **栈 (Stack)** 帧中
  - 它们的生命周期与方法执行绑定：当方法执行完毕，其栈帧会被销毁，所有局部变量也随之 **销毁**
- **Lambda 表达式 (作为对象):** Lambda 表达式本身是一个对象，存活在 **堆 (Heap)** 上
  - 它的生命周期可能 **远远长于** 定义它的方法。例如，一个 Lambda 可能被作为返回值返回，或传递给另一个线程异步执行



#### b. 生命周期冲突示例

考虑以下场景：

```Java
public Runnable createGreeter(String greeting) {
    // 局部变量 greeting 存活在 createGreeter 方法的栈上
    
    // 定义一个 Lambda 表达式
    // 这个 Lambda 引用了局部变量 greeting
    Runnable greeterTask = () ->  System.out.println(greeting + " World!");
    
    // createGreeter 方法执行完毕，其栈帧被销毁，
    // 局部变量 greeting 也随之销毁
    return greeterTask; // 返回的是堆上的 Lambda 对象
}

// 在 main 方法中：
// Runnable task = createGreeter("Hello"); 	// 调用 createGreeter，这句结束后，greeting 被销毁
// task.run(); 								// 此时，task 试图访问一个已经销毁的 greeting 变量！
```

- 如果没有特殊机制，当 `task.run()` 尝试访问 `greeting` 时，它所指向的内存区域可能已经无效，这会导致程序崩溃或不可预测的行为



#### c. Java 的解决方案：值捕获

为了避免上述生命周期冲突，Java 对 Lambda 捕获 **局部变量** 采取了 “**值捕获**”（Capture-by-Value）机制：

- 当 Lambda 表达式被创建时，如果它引用了任何局部变量，这些局部变量在那个时刻的 **值会被复制一份，存储在 Lambda 表达式的实例中**
- Lambda 表达式在执行时，访问的实际上是它自己保存的这份“**副本**”，而不是原始的局部变量

这样一来，即使原始的局部变量在方法结束后被销毁，Lambda 表达式仍然可以通过其内部的副本访问到所需的值，解决了生命周期不一致的问题



#### d. "Effectively Final" 限制的原因

现在我们理解了 Lambda 捕获的是局部变量的“值副本”。那么，为什么还要求它必须是 `final` 或 `Effectively Final` 呢？

- **避免混淆：** 如果 Java 允许在 Lambda 创建之后，原始局部变量的值被修改（例如 `greeting = "Hi";`），那么就会出现：
  - 原始变量 (`greeting`) 的值为 `"Hi"`
  - Lambda 内部保存的副本 (`greeting` 的副本) 的值仍为 `"Hello"`
- **语义不一致：** 这种情况下，同一个变量名 `greeting` 在 Lambda 内部和外部将表示不同的值，这将极大地增加代码理解的难度，引入潜在的错误和不可预测性

因此，`final` 或 `Effectively Final` 的限制，并非为了性能或技术实现的复杂性，而是为了**保证逻辑上的清晰和数据的一致性**，防止程序员因为原始变量与捕获副本的不一致而产生误解和 bug。它强制要求被捕获的局部变量必须是事实上的常量



## 2. 实例变量、静态变量与 `this`

### 2.1 访问实例变量和静态变量

- **实例变量 (Instance Variables):** 存活在 **堆 (Heap)** 上，与对象实例（`this`）绑定
- **静态变量 (Static Variables):** 存活在 **方法区 (Method Area)** ，与类绑定

**关键区别：** 

- 它们的生命周期都 **长于** 方法（也长于 Lambda）。当 Lambda 访问它们时，**不需要** “值捕获”（复制副本）

  相反，Lambda 捕获的是对 **“对象实例 (`this`)”** 的引用（对于实例变量）或对 **“类”** 的引用（对于静态变量）

  Lambda 是通过这些引用去 **直接读写** 堆或方法区上的 **原始数据** 的

```java
public class LambdaCapture {
    
    private static String staticVar = "Static"; // 存活在方法区
    private String instanceVar = "Instance";     // 存活在堆上
    
    public void method() {
        // 局部变量 (存活在栈上)
        String localVar = "Local"; 
        
        Runnable r = () -> {
            // 1. 访问局部变量 (捕获值副本)
            System.out.println(localVar);
            
            // 2. 访问实例变量 (捕获 this 引用)
            //    可以直接修改
            System.out.println(this.instanceVar); 
            this.instanceVar = "Modified"; 
            
            // 3. 访问静态变量 (捕获类引用)
            //    可以直接修改
            System.out.println(LambdaCapture.staticVar);
            LambdaCapture.staticVar = "Changed";
            
            // 4. 不能修改局部变量
            // localVar = "Error"; // 编译错误，违反 Effectively Final
        };
        
        r.run();
        
        System.out.println(instanceVar); // 输出: Modified
        System.out.println(staticVar);   // 输出: Changed
    }
}
```

**总结：**

- 捕获**局部变量**：捕获其**值 (副本)**。必须是 `Effectively Final`
- 捕获**实例/静态变量**：捕获其**引用 (访问路径)**。可以**随意修改**



### 2.2 `this` 引用的区别

这是 Lambda 和匿名内部类的一个**重大区别**

- **Lambda 表达式：** Lambda 没有自己的 `this` 上下文。Lambda 内部的 `this` **等同于** 其外部（定义它的那个方法）的 `this`
- **匿名内部类：** 匿名内部类是一个 **完整的类**，它有 **自己** 的 `this` 上下文

```java
public class ThisExample {
    
    private String name = "OuterClass"; // 外部类的实例变量
    
    public void methodWithLambda() {
        Runnable r = () -> {
            // 这里的 'this' 指向 ThisExample 的实例
            System.out.println(this.name); 
        };
        r.run();
    }
    
    public void methodWithAnonymous() {
        Runnable r = new Runnable() {
            // 匿名内部类可以有自己的实例变量
            private String name = "AnonymousInnerClass"; 
            
            @Override
            public void run() {
                // 这里的 'this' 指向 Runnable 的实例
                System.out.println(this.name); // 输出: AnonymousInnerClass
                
                // 必须使用 '外部类.this' 才能访问到外部
                System.out.println(ThisExample.this.name); // 输出: OuterClass
            }
        };
        r.run();
    }
}

// ---- 测试 ----
// ThisExample example = new ThisExample();
// example.methodWithLambda();     // 输出: OuterClass
// example.methodWithAnonymous();  // 输出: AnonymousInnerClass
//                                //       OuterClass
```

- **结论：** Lambda 表达式在词法上（lexically）绑定 `this`，使得 `this` 的行为更符合直觉，即它总是指向您在编写代码时所处的那个类的实例



