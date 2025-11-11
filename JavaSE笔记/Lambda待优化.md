# Java Lambda表达式完整笔记

## 目录

- [1. Lambda表达式简介](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#1-lambda表达式简介)
- [2. Lambda表达式语法](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#2-lambda表达式语法)
- [3. 函数式接口](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#3-函数式接口)
- [4. Lambda表达式的使用场景](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#4-lambda表达式的使用场景)
- [5. 方法引用](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#5-方法引用)
- [6. 常用的函数式接口](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#6-常用的函数式接口)
- [7. 变量捕获](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#7-变量捕获)
- [8. Lambda与Stream API](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#8-lambda与stream-api)
- [9. 实际应用示例](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#9-实际应用示例)
- [10. 最佳实践](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#10-最佳实践)
- [11. 常见问题与陷阱](https://claude.ai/chat/55f064e0-335c-42e6-ba9c-f9e0b1bd0c3e#11-常见问题与陷阱)

------

## 1. Lambda表达式简介

### 1.1 什么是Lambda表达式

Lambda表达式是Java 8引入的一个重要特性，它允许我们以更简洁的方式表示匿名函数（即没有名称的函数）。Lambda表达式本质上是一个可传递的代码块，可以在以后执行一次或多次。

### 1.2 为什么需要Lambda表达式

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

**使用Lambda表达式：**

```java
// 使用Lambda表达式
Runnable runnable = () -> System.out.println("Hello World");
```

**优势：**

- 代码更简洁，可读性更强
- 减少样板代码
- 更好地支持函数式编程
- 便于并行处理

### 1.3 Lambda表达式的特点

- **类型推断**：编译器可以自动推断参数类型
- **延迟执行**：Lambda表达式不会立即执行，而是在需要时才执行
- **函数式接口**：Lambda表达式必须与函数式接口配合使用
- **访问外部变量**：可以访问外部的final或effectively final变量

------

## 2. Lambda表达式语法

### 2.1 基本语法结构

```
(参数列表) -> { 方法体 }
```

**组成部分：**

1. **参数列表**：用括号包围，多个参数用逗号分隔
2. **箭头符号**：`->`，将参数列表与方法体分隔开
3. **方法体**：包含Lambda表达式的代码逻辑

### 2.2 语法变体

#### 2.2.1 无参数Lambda表达式

```java
// 完整形式
() -> { System.out.println("Hello"); }

// 简化形式（单条语句可省略大括号）
() -> System.out.println("Hello")
```

#### 2.2.2 单参数Lambda表达式

```java
// 完整形式
(String s) -> { System.out.println(s); }

// 省略参数类型（类型推断）
(s) -> System.out.println(s)

// 省略括号（单个参数时）
s -> System.out.println(s)
```

#### 2.2.3 多参数Lambda表达式

```java
// 完整形式
(int a, int b) -> { return a + b; }

// 简化形式
(a, b) -> a + b

// 多条语句必须使用大括号和return
(a, b) -> {
    int sum = a + b;
    return sum;
}
```

#### 2.2.4 有返回值的Lambda表达式

```java
// 显式return（多条语句）
(int x, int y) -> {
    int result = x * y;
    return result;
}

// 隐式return（单条表达式）
(x, y) -> x * y
```

### 2.3 语法规则总结

| 情况       | 是否可以省略         | 示例            |
| ---------- | -------------------- | --------------- |
| 无参数     | 括号不能省略         | `() -> ...`     |
| 单个参数   | 括号可以省略         | `x -> ...`      |
| 多个参数   | 括号不能省略         | `(x, y) -> ...` |
| 参数类型   | 可以省略（类型推断） | `(x, y) -> ...` |
| 单条语句   | 大括号可以省略       | `x -> x * 2`    |
| 多条语句   | 大括号不能省略       | `x -> { ... }`  |
| return语句 | 单表达式时可省略     | `x -> x * 2`    |

------

## 3. 函数式接口

### 3.1 什么是函数式接口

函数式接口（Functional Interface）是只包含**一个抽象方法**的接口。Lambda表达式可以被赋值给函数式接口类型的变量。

### 3.2 @FunctionalInterface注解

```java
@FunctionalInterface
public interface MyFunction {
    void execute();
    
    // 可以有默认方法
    default void defaultMethod() {
        System.out.println("Default method");
    }
    
    // 可以有静态方法
    static void staticMethod() {
        System.out.println("Static method");
    }
}
```

**注意：**

- `@FunctionalInterface`注解不是必需的，但建议使用
- 该注解会在编译时检查接口是否符合函数式接口的定义
- 函数式接口可以有多个默认方法和静态方法，但只能有一个抽象方法

### 3.3 自定义函数式接口示例

```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
}

// 使用
public class LambdaExample {
    public static void main(String[] args) {
        // 加法
        Calculator add = (a, b) -> a + b;
        System.out.println(add.calculate(5, 3)); // 输出: 8
        
        // 减法
        Calculator subtract = (a, b) -> a - b;
        System.out.println(subtract.calculate(5, 3)); // 输出: 2
        
        // 乘法
        Calculator multiply = (a, b) -> a * b;
        System.out.println(multiply.calculate(5, 3)); // 输出: 15
    }
}
```

------

## 4. Lambda表达式的使用场景

### 4.1 替代匿名内部类

**场景1：线程创建**

```java
// 传统方式
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Thread running");
    }
});

// Lambda方式
Thread thread2 = new Thread(() -> System.out.println("Thread running"));
```

**场景2：事件监听（Android/Swing）**

```java
// 传统方式
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("Button clicked");
    }
});

// Lambda方式
button.setOnClickListener(v -> System.out.println("Button clicked"));
```

### 4.2 集合遍历和操作

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// 传统方式
for (String name : names) {
    System.out.println(name);
}

// Lambda方式
names.forEach(name -> System.out.println(name));

// 方法引用
names.forEach(System.out::println);
```

### 4.3 集合排序

```java
List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9);

// 传统方式
Collections.sort(numbers, new Comparator<Integer>() {
    @Override
    public int compare(Integer a, Integer b) {
        return a.compareTo(b);
    }
});

// Lambda方式
Collections.sort(numbers, (a, b) -> a.compareTo(b));

// 更简洁的方式
numbers.sort(Integer::compareTo);
```

### 4.4 条件过滤

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// 过滤出长度大于3的名字
List<String> filtered = names.stream()
    .filter(name -> name.length() > 3)
    .collect(Collectors.toList());
// 结果: [Alice, Charlie, David]
```

### 4.5 数据转换

```java
List<String> names = Arrays.asList("alice", "bob", "charlie");

// 转换为大写
List<String> upperNames = names.stream()
    .map(name -> name.toUpperCase())
    .collect(Collectors.toList());
// 结果: [ALICE, BOB, CHARLIE]
```

------

## 5. 方法引用

方法引用是Lambda表达式的一种简写形式，当Lambda表达式只是调用一个已存在的方法时，可以使用方法引用。

### 5.1 方法引用的类型

#### 5.1.1 静态方法引用

**语法：** `类名::静态方法名`

```java
// Lambda表达式
Function<String, Integer> parser = s -> Integer.parseInt(s);

// 方法引用
Function<String, Integer> parser = Integer::parseInt;

// 使用示例
List<String> numbers = Arrays.asList("1", "2", "3");
List<Integer> integers = numbers.stream()
    .map(Integer::parseInt)
    .collect(Collectors.toList());
```

#### 5.1.2 实例方法引用（对象的方法）

**语法：** `对象名::实例方法名`

```java
// Lambda表达式
Consumer<String> printer = s -> System.out.println(s);

// 方法引用
Consumer<String> printer = System.out::println;

// 使用示例
List<String> names = Arrays.asList("Alice", "Bob");
names.forEach(System.out::println);
```

#### 5.1.3 类的实例方法引用

**语法：** `类名::实例方法名`

```java
// Lambda表达式
Comparator<String> comparator = (s1, s2) -> s1.compareToIgnoreCase(s2);

// 方法引用
Comparator<String> comparator = String::compareToIgnoreCase;

// 使用示例
List<String> names = Arrays.asList("charlie", "Alice", "BOB");
names.sort(String::compareToIgnoreCase);
// 结果: [Alice, BOB, charlie]
```

#### 5.1.4 构造方法引用

**语法：** `类名::new`

```java
// Lambda表达式
Supplier<ArrayList<String>> listSupplier = () -> new ArrayList<>();

// 构造方法引用
Supplier<ArrayList<String>> listSupplier = ArrayList::new;

// 使用示例
List<String> list = listSupplier.get();
```

**带参数的构造方法引用：**

```java
public class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

// 使用构造方法引用
BiFunction<String, Integer, Person> personFactory = Person::new;
Person person = personFactory.apply("Alice", 25);
```

### 5.2 方法引用的使用场景

```java
// 1. 静态方法引用
List<String> strings = Arrays.asList("1", "2", "3");
strings.stream().map(Integer::parseInt);

// 2. 对象方法引用
strings.forEach(System.out::println);

// 3. 类的实例方法引用
strings.sort(String::compareTo);

// 4. 构造方法引用
List<Person> persons = strings.stream()
    .map(Person::new)
    .collect(Collectors.toList());
```

------

## 6. 常用的函数式接口

Java 8在`java.util.function`包中提供了大量常用的函数式接口。

### 6.1 核心函数式接口

#### 6.1.1 `Function<T, R>`

**作用：** 接收一个参数，返回一个结果

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}

// 示例
Function<String, Integer> strLength = s -> s.length();
System.out.println(strLength.apply("Hello")); // 输出: 5

Function<Integer, Integer> square = x -> x * x;
System.out.println(square.apply(5)); // 输出: 25
```

**常用方法：**

```java
// compose: 先执行参数函数，再执行当前函数
Function<Integer, Integer> multiplyBy2 = x -> x * 2;
Function<Integer, Integer> add3 = x -> x + 3;
Function<Integer, Integer> composed = multiplyBy2.compose(add3);
System.out.println(composed.apply(5)); // (5 + 3) * 2 = 16

// andThen: 先执行当前函数，再执行参数函数
Function<Integer, Integer> chained = multiplyBy2.andThen(add3);
System.out.println(chained.apply(5)); // (5 * 2) + 3 = 13
```

#### 6.1.2`Predicate<T>`

**作用：** 接收一个参数，返回boolean值（用于判断）

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}

// 示例
Predicate<Integer> isPositive = x -> x > 0;
System.out.println(isPositive.test(5));  // true
System.out.println(isPositive.test(-3)); // false

Predicate<String> isEmpty = s -> s.isEmpty();
System.out.println(isEmpty.test(""));    // true
System.out.println(isEmpty.test("Hi"));  // false
```

**常用方法：**

```java
Predicate<Integer> isEven = x -> x % 2 == 0;
Predicate<Integer> isPositive = x -> x > 0;

// and: 逻辑与
Predicate<Integer> isPositiveEven = isEven.and(isPositive);
System.out.println(isPositiveEven.test(4));  // true
System.out.println(isPositiveEven.test(-4)); // false

// or: 逻辑或
Predicate<Integer> isEvenOrPositive = isEven.or(isPositive);
System.out.println(isEvenOrPositive.test(3));  // true

// negate: 逻辑非
Predicate<Integer> isOdd = isEven.negate();
System.out.println(isOdd.test(3)); // true
```

#### 6.1.3 `Consumer<T>`

**作用：** 接收一个参数，无返回值（用于消费数据）

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}

// 示例
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hello World"); // 输出: Hello World

List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(name -> System.out.println("Hello, " + name));
```

**常用方法：**

```java
Consumer<String> c1 = s -> System.out.print(s.toUpperCase());
Consumer<String> c2 = s -> System.out.println(" - processed");

// andThen: 链式调用
Consumer<String> combined = c1.andThen(c2);
combined.accept("hello"); // 输出: HELLO - processed
```

#### 6.1.4 `Supplier<T>`

**作用：** 无参数，返回一个结果（用于提供数据）

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}

// 示例
Supplier<String> stringSupplier = () -> "Hello World";
System.out.println(stringSupplier.get()); // 输出: Hello World

Supplier<Double> randomSupplier = () -> Math.random();
System.out.println(randomSupplier.get()); // 输出: 随机数

Supplier<LocalDateTime> timeSupplier = LocalDateTime::now;
System.out.println(timeSupplier.get()); // 输出: 当前时间
```

### 6.2 扩展函数式接口

#### 6.2.1 `BiFunction<T, U, R>`

**作用：** 接收两个参数，返回一个结果

```java
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(5, 3)); // 输出: 8

BiFunction<String, String, String> concat = (s1, s2) -> s1 + s2;
System.out.println(concat.apply("Hello", " World")); // 输出: Hello World
```

#### 6.2.2 `BiConsumer<T, U>`

**作用：** 接收两个参数，无返回值

```java
BiConsumer<String, Integer> printer = (name, age) -> 
    System.out.println(name + " is " + age + " years old");
printer.accept("Alice", 25); // 输出: Alice is 25 years old

// Map遍历
Map<String, Integer> map = new HashMap<>();
map.put("Alice", 25);
map.put("Bob", 30);
map.forEach((key, value) -> System.out.println(key + ": " + value));
```

#### 6.2.3 `BiPredicate<T, U>`

**作用：** 接收两个参数，返回boolean值

```java
BiPredicate<String, Integer> isLengthEqual = (s, len) -> s.length() == len;
System.out.println(isLengthEqual.test("Hello", 5)); // true
System.out.println(isLengthEqual.test("Hi", 5));    // false
```

#### 6.2.4 `UnaryOperator<T>`

**作用：** 接收一个参数，返回相同类型的结果（Function的特殊形式）

```java
UnaryOperator<Integer> square = x -> x * x;
System.out.println(square.apply(5)); // 输出: 25

UnaryOperator<String> toUpper = String::toUpperCase;
System.out.println(toUpper.apply("hello")); // 输出: HELLO
```

#### 6.2.5 `BinaryOperator<T>`

**作用：** 接收两个相同类型的参数，返回相同类型的结果（BiFunction的特殊形式）

```java
BinaryOperator<Integer> add = (a, b) -> a + b;
System.out.println(add.apply(5, 3)); // 输出: 8

BinaryOperator<String> concat = (s1, s2) -> s1 + s2;
System.out.println(concat.apply("Hello", " World")); // 输出: Hello World

// 常用于reduce操作
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
System.out.println(sum); // 输出: 15
```

### 6.3 基本类型函数式接口

为了避免自动装箱/拆箱的性能开销，Java提供了基本类型的函数式接口：

```java
// IntPredicate, LongPredicate, DoublePredicate
IntPredicate isEven = x -> x % 2 == 0;
System.out.println(isEven.test(4)); // true

// IntFunction<R>, LongFunction<R>, DoubleFunction<R>
IntFunction<String> intToString = x -> "Number: " + x;
System.out.println(intToString.apply(5)); // Number: 5

// ToIntFunction<T>, ToLongFunction<T>, ToDoubleFunction<T>
ToIntFunction<String> stringLength = s -> s.length();
System.out.println(stringLength.applyAsInt("Hello")); // 5

// IntConsumer, LongConsumer, DoubleConsumer
IntConsumer printer = x -> System.out.println(x);
printer.accept(42); // 42

// IntSupplier, LongSupplier, DoubleSupplier
IntSupplier randomInt = () -> (int)(Math.random() * 100);
System.out.println(randomInt.getAsInt()); // 随机整数

// IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator
IntUnaryOperator square = x -> x * x;
System.out.println(square.applyAsInt(5)); // 25

// IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator
IntBinaryOperator add = (a, b) -> a + b;
System.out.println(add.applyAsInt(5, 3)); // 8
```

### 6.4 函数式接口总结表

| 接口                  | 参数类型 | 返回类型 | 抽象方法               | 用途       |
| --------------------- | -------- | -------- | ---------------------- | ---------- |
| `Function<T, R>`      | T        | R        | R apply(T t)           | 转换       |
| `Predicate<T>`        | T        | boolean  | boolean test(T t)      | 判断       |
| `Consumer<T>`         | T        | void     | void accept(T t)       | 消费       |
| `Supplier<T>`         | -        | T        | T get()                | 提供       |
| `BiFunction<T, U, R>` | T, U     | R        | R apply(T t, U u)      | 双参数转换 |
| `BiConsumer<T, U>`    | T, U     | void     | void accept(T t, U u)  | 双参数消费 |
| `BiPredicate<T, U>`   | T, U     | boolean  | boolean test(T t, U u) | 双参数判断 |
| `UnaryOperator<T>`    | T        | T        | T apply(T t)           | 一元操作   |
| `BinaryOperator<T>`   | T, T     | T        | T apply(T t1, T t2)    | 二元操作   |

------

## 7. 变量捕获

### 7.1 Lambda表达式可以访问的变量

Lambda表达式可以访问：

1. 静态变量
2. 实例变量
3. 局部变量（必须是final或effectively final）

### 7.2 Effectively Final概念

**Effectively Final**: 变量在初始化后没有被重新赋值，即使没有声明为final。

```java
public void example() {
    // 局部变量（effectively final）
    String prefix = "Hello ";
    
    // Lambda表达式可以访问
    Consumer<String> greeter = name -> System.out.println(prefix + name);
    greeter.accept("Alice"); // 输出: Hello Alice
    
    // 错误示例：如果修改了prefix，就不是effectively final了
    // prefix = "Hi "; // 编译错误
}
```

### 7.3 访问实例变量和静态变量

```java
public class LambdaCapture {
    private static String staticVar = "Static";
    private String instanceVar = "Instance";
    
    public void method() {
        String localVar = "Local";
        
        Runnable r = () -> {
            // 可以访问静态变量
            System.out.println(staticVar);
            
            // 可以访问实例变量
            System.out.println(instanceVar);
            
            // 可以访问局部变量（effectively final）
            System.out.println(localVar);
            
            // 可以修改实例变量和静态变量
            instanceVar = "Modified";
            staticVar = "Changed";
            
            // 不能修改局部变量
            // localVar = "Error"; // 编译错误
        };
        
        r.run();
    }
}
```

### 7.4 this引用

在Lambda表达式中，`this`关键字指向的是外部类的实例，而不是Lambda表达式本身。

```java
public class ThisExample {
    private String name = "Outer";
    
    public void method() {
        Runnable r = () -> {
            // this指向ThisExample实例
            System.out.println(this.name);
        };
        
        r.run(); // 输出: Outer
    }
    
    // 对比匿名内部类
    public void methodWithAnonymous() {
        Runnable r = new Runnable() {
            private String name = "Inner";
            
            @Override
            public void run() {
                // this指向Runnable实例
                System.out.println(this.name); // 输出: Inner
                
                // 访问外部类需要使用外部类名
                System.out.println(ThisExample.this.name); // 输出: Outer
            }
        };
        
        r.run();
    }
}
```

### 7.5 变量捕获的限制原因

Lambda表达式要求局部变量是final或effectively final的原因：

1. **并发安全性**：Lambda表达式可能在不同的线程中执行
2. **变量的生命周期**：局部变量在栈上分配，方法返回后会被销毁
3. **一致性**：确保Lambda表达式捕获的值不会改变

```java
public class CaptureExample {
    public Runnable createRunnable() {
        int localVar = 10;
        
        // Lambda捕获的是localVar的值（副本）
        Runnable r = () -> System.out.println(localVar);
        
        // 如果允许修改localVar，会导致不一致
        // localVar = 20; // 编译错误
        
        return r;
    } // localVar在这里被销毁，但Lambda仍然可以使用它的值
}
```

------

## 8. Lambda与Stream API

### 8.1 Stream API简介

Stream API是Java 8引入的用于集合操作的工具，与Lambda表达式配合使用，可以进行函数式编程。

**特点：**

- 声明式编程
- 支持链式操作
- 支持并行处理
- 延迟执行

### 8.2 创建Stream

```java
// 1. 从集合创建
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();
Stream<String> parallelStream = list.parallelStream();

// 2. 从数组创建
String[] array = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(array);

// 3. 使用Stream.of()
Stream<String> stream3 = Stream.of("a", "b", "c");

// 4. 无限流
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 1);
Stream<Double> randomStream = Stream.generate(Math::random);

// 5. 使用Builder
Stream<String> stream4 = Stream.<String>builder()
    .add("a").add("b").add("c").build();
```

### 8.3 中间操作（Intermediate Operations）

中间操作会返回一个新的Stream，支持链式调用。

#### 8.3.1 filter - 过滤

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

// 过滤偶数
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// 结果: [2, 4, 6]

// 多条件过滤
List<Integer> result = numbers.stream()
    .filter(n -> n > 2)
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// 结果: [4, 6]
```

#### 8.3.2 map - 映射转换

```java
List<String> words = Arrays.asList("hello", "world");

// 转换为大写
List<String> upperWords = words.stream()
    .map(s -> s.toUpperCase())
    .collect(Collectors.toList());
// 结果: [HELLO, WORLD]

// 转换为长度
List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList());
// 结果: [5, 5]
```

#### 8.3.3 flatMap - 扁平化映射

```java
List<List<Integer>> nestedList = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4),
    Arrays.asList(5, 6)
);

// 扁平化为单个列表
List<Integer> flatList = nestedList.stream()
    .flatMap(list -> list.stream())
    .collect(Collectors.toList());
// 结果: [1, 2, 3, 4, 5, 6]

// 字符串拆分示例
List<String> sentences = Arrays.asList("Hello World", "Java Lambda");
List<String> words = sentences.stream()
    .flatMap(s -> Arrays.stream(s.split(" ")))
    .collect(Collectors.toList());
// 结果: [Hello, World, Java, Lambda]
```

#### 8.3.4 distinct - 去重

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 4);
List<Integer> distinctNumbers = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// 结果: [1, 2, 3, 4]
```

#### 8.3.5 sorted - 排序

```java
List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9);

// 自然排序
List<Integer> sorted1 = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
// 结果: [1, 2, 5, 8, 9]

// 自定义排序
List<Integer> sorted2 = numbers.stream()
    .sorted((a, b) -> b - a) // 降序
    .collect(Collectors.toList());
// 结果: [9, 8, 5, 2, 1]
```

#### 8.3.6 peek - 查看元素

```java
List<Integer> numbers = Arrays.asList(1, 2, 3);

List<Integer> result = numbers.stream()
    .peek(n -> System.out.println("原始值: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("乘以2后: " + n))
    .collect(Collectors.toList());
```

#### 8.3.7 limit 和 skip

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);

// 取前3个
List<Integer> first3 = numbers.stream()
    .limit(3)
    .collect(Collectors.toList());
// 结果: [1, 2, 3]

// 跳过前3个
List<Integer> skip3 = numbers.stream()
    .skip(3)
    .collect(Collectors.toList());
// 结果: [4, 5, 6, 7, 8]

// 分页：跳过前10个，取5个
List<Integer> page = numbers.stream()
    .skip(10)
    .limit(5)
    .collect(Collectors.toList());
```

### 8.4 终止操作（Terminal Operations）

终止操作会触发Stream的计算并返回结果。

#### 8.4.1 forEach - 遍历

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

names.stream().forEach(name -> System.out.println(name));
// 或
names.stream().forEach(System.out::println);
```

#### 8.4.2 collect - 收集结果

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// 收集到List
List<String> list = names.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());

// 收集到Set
Set<String> set = names.stream()
    .collect(Collectors.toSet());

// 收集到Map
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(
        name -> name,           // key
        name -> name.length()   // value
    ));

// 连接字符串
String joined = names.stream()
    .collect(Collectors.joining(", "));
// 结果: "Alice, Bob, Charlie"

// 分组
Map<Integer, List<String>> grouped = names.stream()
    .collect(Collectors.groupingBy(String::length));
// 结果: {3=[Bob], 5=[Alice], 7=[Charlie]}

// 分区（按条件分成两组）
Map<Boolean, List<String>> partitioned = names.stream()
    .collect(Collectors.partitioningBy(s -> s.length() > 4));
// 结果: {false=[Bob], true=[Alice, Charlie]}
```

#### 8.4.3 reduce - 归约

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 求和
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
// 结果: 15

// 求积
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
// 结果: 120

// 求最大值
Optional<Integer> max = numbers.stream()
    .reduce((a, b) -> a > b ? a : b);

// 字符串连接
List<String> words = Arrays.asList("Hello", " ", "World");
String sentence = words.stream()
    .reduce("", (a, b) -> a + b);
// 结果: "Hello World"
```

#### 8.4.4 count - 计数

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

long count = names.stream()
    .filter(s -> s.length() > 3)
    .count();
// 结果: 2
```

#### 8.4.5 anyMatch, allMatch, noneMatch - 匹配

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 是否有任意元素满足条件
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);
// 结果: true

// 是否所有元素都满足条件
boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);
// 结果: true

// 是否没有元素满足条件
boolean noneNegative = numbers.stream()
    .noneMatch(n -> n < 0);
// 结果: true
```

#### 8.4.6 findFirst, findAny - 查找

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 查找第一个
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 3)
    .findFirst();
// 结果: Optional[4]

// 查找任意一个（并行流中更高效）
Optional<Integer> any = numbers.stream()
    .filter(n -> n > 3)
    .findAny();
```

#### 8.4.7 min, max - 最值

```java
List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9);

Optional<Integer> min = numbers.stream()
    .min(Integer::compareTo);
// 结果: Optional[1]

Optional<Integer> max = numbers.stream()
    .max(Integer::compareTo);
// 结果: Optional[9]
```

### 8.5 Stream操作组合示例

```java
public class StreamExample {
    public static void main(String[] args) {
        List<Person> persons = Arrays.asList(
            new Person("Alice", 25, "Developer"),
            new Person("Bob", 30, "Manager"),
            new Person("Charlie", 25, "Developer"),
            new Person("David", 35, "Manager"),
            new Person("Eve", 28, "Developer")
        );
        
        // 复杂查询：找出年龄大于25岁的开发者的名字，按名字排序
        List<String> result = persons.stream()
            .filter(p -> p.getAge() > 25)
            .filter(p -> "Developer".equals(p.getJob()))
            .map(Person::getName)
            .sorted()
            .collect(Collectors.toList());
        // 结果: [Eve]
        
        // 统计每个职位的平均年龄
        Map<String, Double> avgAgeByJob = persons.stream()
            .collect(Collectors.groupingBy(
                Person::getJob,
                Collectors.averagingInt(Person::getAge)
            ));
        // 结果: {Developer=26.0, Manager=32.5}
        
        // 找出年龄最大的人
        Optional<Person> oldest = persons.stream()
            .max(Comparator.comparing(Person::getAge));
    }
}
```

### 8.6 并行Stream

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 串行Stream
long sum1 = numbers.stream()
    .mapToLong(Integer::longValue)
    .sum();

// 并行Stream
long sum2 = numbers.parallelStream()
    .mapToLong(Integer::longValue)
    .sum();

// 控制并行度
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "4");
```

**注意事项：**

- 并行Stream适合CPU密集型操作
- 数据量较小时，并行Stream的开销可能超过收益
- 确保操作是无状态且线程安全的

------

## 9. 实际应用示例

### 9.1 文件处理

```java
// 读取文件所有行
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.filter(line -> line.contains("Java"))
         .forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}

// 统计文件中的单词数
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    long wordCount = lines
        .flatMap(line -> Arrays.stream(line.split("\\s+")))
        .count();
    System.out.println("Total words: " + wordCount);
} catch (IOException e) {
    e.printStackTrace();
}
```

### 9.2 数据处理和分析

```java
public class DataAnalysis {
    static class Sale {
        private String product;
        private double amount;
        private String region;
        
        public Sale(String product, double amount, String region) {
            this.product = product;
            this.amount = amount;
            this.region = region;
        }
        
        // getters
        public String getProduct() { return product; }
        public double getAmount() { return amount; }
        public String getRegion() { return region; }
    }
    
    public static void main(String[] args) {
        List<Sale> sales = Arrays.asList(
            new Sale("Laptop", 1200.0, "North"),
            new Sale("Phone", 800.0, "South"),
            new Sale("Laptop", 1300.0, "North"),
            new Sale("Tablet", 500.0, "East"),
            new Sale("Phone", 850.0, "West")
        );
        
        // 1. 按地区分组统计销售额
        Map<String, Double> salesByRegion = sales.stream()
            .collect(Collectors.groupingBy(
                Sale::getRegion,
                Collectors.summingDouble(Sale::getAmount)
            ));
        System.out.println("Sales by region: " + salesByRegion);
        
        // 2. 找出销售额最高的产品
        Optional<Sale> topSale = sales.stream()
            .max(Comparator.comparing(Sale::getAmount));
        topSale.ifPresent(s -> 
            System.out.println("Top sale: " + s.getProduct() + " - $" + s.getAmount())
        );
        
        // 3. 计算平均销售额
        double avgSale = sales.stream()
            .mapToDouble(Sale::getAmount)
            .average()
            .orElse(0.0);
        System.out.println("Average sale: $" + avgSale);
        
        // 4. 按产品统计数量
        Map<String, Long> productCount = sales.stream()
            .collect(Collectors.groupingBy(
                Sale::getProduct,
                Collectors.counting()
            ));
        System.out.println("Product count: " + productCount);
    }
}
```

### 9.3 集合转换

```java
public class CollectionTransform {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Alice", 50000, "IT"),
            new Employee("Bob", 60000, "HR"),
            new Employee("Charlie", 55000, "IT"),
            new Employee("David", 70000, "Finance")
        );
        
        // List转Map（员工名到薪水的映射）
        Map<String, Integer> nameToSalary = employees.stream()
            .collect(Collectors.toMap(
                Employee::getName,
                Employee::getSalary
            ));
        
        // 按部门分组
        Map<String, List<Employee>> byDept = employees.stream()
            .collect(Collectors.groupingBy(Employee::getDepartment));
        
        // 提取所有员工名字
        List<String> names = employees.stream()
            .map(Employee::getName)
            .collect(Collectors.toList());
        
        // 过滤并转换
        Set<String> highEarnerNames = employees.stream()
            .filter(e -> e.getSalary() > 55000)
            .map(Employee::getName)
            .collect(Collectors.toSet());
    }
}
```

### 9.4 复杂业务逻辑

```java
public class OrderProcessing {
    static class Order {
        private String id;
        private double amount;
        private String status;
        private LocalDate date;
        
        // 构造函数和getters
    }
    
    public List<Order> processPendingOrders(List<Order> orders) {
        return orders.stream()
            // 过滤待处理订单
            .filter(order -> "PENDING".equals(order.getStatus()))
            // 按日期排序
            .sorted(Comparator.comparing(Order::getDate))
            // 过滤金额大于100的订单
            .filter(order -> order.getAmount() > 100)
            // 限制处理前10个
            .limit(10)
            // 转换订单状态
            .peek(order -> order.setStatus("PROCESSING"))
            .collect(Collectors.toList());
    }
    
    public double calculateTotalRevenue(List<Order> orders, 
                                       LocalDate startDate, 
                                       LocalDate endDate) {
        return orders.stream()
            .filter(order -> "COMPLETED".equals(order.getStatus()))
            .filter(order -> {
                LocalDate orderDate = order.getDate();
                return !orderDate.isBefore(startDate) && 
                       !orderDate.isAfter(endDate);
            })
            .mapToDouble(Order::getAmount)
            .sum();
    }
}
```

### 9.5 Optional与Lambda

```java
public class OptionalExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        // 查找并处理
        names.stream()
            .filter(name -> name.startsWith("C"))
            .findFirst()
            .ifPresent(name -> System.out.println("Found: " + name));
        
        // 提供默认值
        String result = names.stream()
            .filter(name -> name.startsWith("Z"))
            .findFirst()
            .orElse("Not found");
        
        // 抛出异常
        String name = names.stream()
            .filter(n -> n.startsWith("Z"))
            .findFirst()
            .orElseThrow(() -> new RuntimeException("Name not found"));
        
        // 链式处理
        Optional<String> optional = Optional.of("hello");
        String upper = optional
            .map(String::toUpperCase)
            .filter(s -> s.length() > 3)
            .orElse("default");
    }
}
```

------

## 10. 最佳实践

### 10.1 Lambda表达式的优势

1. **代码简洁性**

```java
// 之前：8行
List<String> result = new ArrayList<>();
for (String s : list) {
    if (s.length() > 3) {
        result.add(s.toUpperCase());
    }
}

// 之后：3行
List<String> result = list.stream()
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

1. **函数式编程思维**

- 关注"做什么"而不是"怎么做"
- 声明式编程，更易理解和维护
- 减少可变状态，降低bug风险

1. **并行处理**

```java
// 轻松实现并行处理
list.parallelStream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());
```

### 10.2 命名与可读性

#### 10.2.1 使用清晰的参数名

```java
// 不好
list.stream().filter(x -> x.length() > 3)

// 好
list.stream().filter(name -> name.length() > 3)
```

#### 10.2.2 复杂逻辑提取为方法

```java
// 不好：Lambda太复杂
list.stream()
    .filter(person -> person.getAge() > 18 && 
                     person.getSalary() > 5000 &&
                     person.getDepartment().equals("IT"))
    .collect(Collectors.toList());

// 好：提取为方法
list.stream()
    .filter(this::isEligibleEmployee)
    .collect(Collectors.toList());

private boolean isEligibleEmployee(Person person) {
    return person.getAge() > 18 && 
           person.getSalary() > 5000 &&
           person.getDepartment().equals("IT");
}
```

#### 10.2.3 方法引用优于Lambda

```java
// 可以用方法引用时，优先使用方法引用
list.stream().map(s -> s.toUpperCase())  // 不推荐
list.stream().map(String::toUpperCase)   // 推荐

list.forEach(s -> System.out.println(s)) // 不推荐
list.forEach(System.out::println)        // 推荐
```

### 10.3 性能考虑

#### 10.3.1 避免在循环中创建Lambda

```java
// 不好：每次循环都创建新的Lambda
for (int i = 0; i < 1000; i++) {
    list.stream().filter(s -> s.length() > 3).count();
}

// 好：Lambda定义在循环外
Predicate<String> lengthFilter = s -> s.length() > 3;
for (int i = 0; i < 1000; i++) {
    list.stream().filter(lengthFilter).count();
}
```

#### 10.3.2 选择合适的集合操作

```java
// 只需要判断是否存在，不要用filter().count() > 0
// 不好
boolean hasLongName = names.stream()
    .filter(s -> s.length() > 10)
    .count() > 0;

// 好
boolean hasLongName = names.stream()
    .anyMatch(s -> s.length() > 10);
```

#### 10.3.3 合理使用并行流

```java
// 小数据量：串行更快
if (list.size() < 1000) {
    list.stream().forEach(this::process);
} else {
    list.parallelStream().forEach(this::process);
}

// CPU密集型任务适合并行
// IO密集型任务不适合并行
```

### 10.4 异常处理

#### 10.4.1 Lambda中的受检异常

```java
// 方法1：包装为运行时异常
list.stream()
    .map(s -> {
        try {
            return Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new RuntimeException(e);
        }
    })
    .collect(Collectors.toList());

// 方法2：提取为方法
list.stream()
    .map(this::parseIntSafely)
    .collect(Collectors.toList());

private Integer parseIntSafely(String s) {
    try {
        return Integer.parseInt(s);
    } catch (NumberFormatException e) {
        return 0; // 或返回默认值
    }
}

// 方法3：使用函数式异常处理工具
@FunctionalInterface
public interface ThrowingFunction<T, R> {
    R apply(T t) throws Exception;
    
    static <T, R> Function<T, R> unchecked(ThrowingFunction<T, R> f) {
        return t -> {
            try {
                return f.apply(t);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
}

// 使用
list.stream()
    .map(ThrowingFunction.unchecked(s -> methodThatThrows(s)))
    .collect(Collectors.toList());
```

### 10.5 避免副作用

```java
// 不好：有副作用
List<String> results = new ArrayList<>();
list.stream()
    .filter(s -> s.length() > 3)
    .forEach(s -> results.add(s)); // 副作用

// 好：使用collect
List<String> results = list.stream()
    .filter(s -> s.length() > 3)
    .collect(Collectors.toList());
```

### 10.6 何时不使用Lambda

1. **逻辑过于复杂**
   - 超过3-5行的Lambda应该提取为方法
2. **需要重复使用**
   - 相同的Lambda在多处使用时，应提取为方法
3. **性能关键代码**
   - 在极端性能要求的场景，传统循环可能更快
4. **调试困难**
   - 复杂的Stream链可能难以调试

------

## 11. 常见问题与陷阱

### 11.1 无限流导致的内存问题

```java
// 危险：无限流没有终止条件
Stream.iterate(0, n -> n + 1)
    .forEach(System.out::println); // 永远不会停止

// 正确：使用limit限制
Stream.iterate(0, n -> n + 1)
    .limit(100)
    .forEach(System.out::println);
```

### 11.2 Stream只能使用一次

```java
// 错误示例
Stream<String> stream = list.stream();
long count = stream.count();
stream.forEach(System.out::println); // 抛出IllegalStateException

// 正确：需要多次操作时，重新创建Stream
long count = list.stream().count();
list.stream().forEach(System.out::println);
```

### 11.3 修改Lambda捕获的变量

```java
// 错误：不能修改局部变量
int sum = 0;
list.forEach(n -> sum += n); // 编译错误

// 正确方法1：使用Stream的reduce
int sum = list.stream().reduce(0, Integer::sum);

// 正确方法2：使用数组（不推荐）
int[] sum = {0};
list.forEach(n -> sum[0] += n);

// 正确方法3：使用AtomicInteger
AtomicInteger sum = new AtomicInteger(0);
list.forEach(n -> sum.addAndGet(n));
```

### 11.4 并行流的陷阱

```java
// 问题1：顺序依赖
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
// 并行流可能打乱顺序
numbers.parallelStream().forEach(System.out::println);

// 解决：使用forEachOrdered保持顺序
numbers.parallelStream().forEachOrdered(System.out::println);

// 问题2：共享可变状态（线程不安全）
List<Integer> results = new ArrayList<>(); // 不是线程安全的
numbers.parallelStream()
    .forEach(n -> results.add(n * 2)); // 可能出现并发问题

// 解决：使用collect
List<Integer> results = numbers.parallelStream()
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

### 11.5 装箱拆箱的性能开销

```java
// 不好：频繁装箱拆箱
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b); // Integer自动装箱拆箱

// 好：使用基本类型流
int sum = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();
```

### 11.6 空指针异常

```java
List<String> list = Arrays.asList("a", null, "c");

// 可能抛出NullPointerException
list.stream()
    .map(s -> s.toUpperCase())
    .forEach(System.out::println);

// 解决方法1：过滤null
list.stream()
    .filter(Objects::nonNull)
    .map(String::toUpperCase)
    .forEach(System.out::println);

// 解决方法2：使用Optional
list.stream()
    .map(s -> Optional.ofNullable(s)
                     .map(String::toUpperCase)
                     .orElse("NULL"))
    .forEach(System.out::println);
```

### 11.7 Collectors.toMap的键冲突

```java
List<Person> persons = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30),
    new Person("Alice", 28) // 重复的键
);

// 错误：键冲突抛出异常
Map<String, Integer> map = persons.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge
    )); // IllegalStateException: Duplicate key

// 解决：提供合并函数
Map<String, Integer> map = persons.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge,
        (age1, age2) -> age1 // 保留第一个值，或使用其他合并策略
    ));
```

### 11.8 短路操作的误用

```java
// 问题：limit后的操作不会全部执行
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

numbers.stream()
    .peek(n -> System.out.println("Processing: " + n))
    .limit(2)
    .forEach(System.out::println);
// 只处理前2个元素

// 注意：并行流中limit的行为可能不同
numbers.parallelStream()
    .limit(2)
    .forEach(System.out::println); // 顺序不确定
```

### 11.9 日期时间相关的陷阱

```java
// 使用LocalDate等新的日期时间API
List<LocalDate> dates = Arrays.asList(
    LocalDate.of(2024, 1, 1),
    LocalDate.of(2024, 6, 15),
    LocalDate.of(2024, 12, 31)
);

// 过滤今年的日期
int currentYear = LocalDate.now().getYear();
List<LocalDate> thisYear = dates.stream()
    .filter(date -> date.getYear() == currentYear)
    .collect(Collectors.toList());
```