# Java Stream 流完全指南

## 目录

1. [Stream 基础概念](https://claude.ai/chat/151100cb-dd4d-4cdd-9c46-afa893e2ffb6#stream-基础概念)
2. [创建 Stream](https://claude.ai/chat/151100cb-dd4d-4cdd-9c46-afa893e2ffb6#创建-stream)
3. [中间操作](https://claude.ai/chat/151100cb-dd4d-4cdd-9c46-afa893e2ffb6#中间操作)
4. [终端操作](https://claude.ai/chat/151100cb-dd4d-4cdd-9c46-afa893e2ffb6#终端操作)
5. [收集器 Collectors](https://claude.ai/chat/151100cb-dd4d-4cdd-9c46-afa893e2ffb6#收集器-collectors)
6. [实战示例](https://claude.ai/chat/151100cb-dd4d-4cdd-9c46-afa893e2ffb6#实战示例)
7. [性能优化](https://claude.ai/chat/151100cb-dd4d-4cdd-9c46-afa893e2ffb6#性能优化)
8. [最佳实践](https://claude.ai/chat/151100cb-dd4d-4cdd-9c46-afa893e2ffb6#最佳实践)
9. [常见陷阱](https://claude.ai/chat/151100cb-dd4d-4cdd-9c46-afa893e2ffb6#常见陷阱)

------

## Stream 基础概念

### 什么是 Stream?

Stream 是 Java 8 引入的一个抽象概念,用于以声明式方式处理数据集合。它不是数据结构,不存储数据,而是对数据源(如集合、数组)进行计算操作的管道。

### Stream 的特点

- **不存储数据**: Stream 不是数据结构,不存储元素
- **函数式编程**: 支持 lambda 表达式,代码简洁
- **惰性执行**: 中间操作不会立即执行,只有遇到终端操作才会执行
- **可消费性**: Stream 只能被消费一次,消费后不能再使用
- **支持并行**: 可以轻松转换为并行流,利用多核处理器

### Stream 操作分类

```
数据源 → 中间操作(0个或多个) → 终端操作(1个)
```

**中间操作(Intermediate Operations)**:

- 返回一个新的 Stream
- 采用惰性执行,不会立即执行
- 可以链式调用

**终端操作(Terminal Operations)**:

- 触发实际的计算
- 产生结果或副作用
- 执行后 Stream 不能再使用

------

## 创建 Stream

### 1. 从集合创建

```java
// List 创建
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();
Stream<String> parallelStream = list.parallelStream(); // 并行流

// Set 创建
Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 3));
Stream<Integer> stream2 = set.stream();

// Map 创建
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);
Stream<Map.Entry<String, Integer>> stream3 = map.entrySet().stream();
Stream<String> keyStream = map.keySet().stream();
Stream<Integer> valueStream = map.values().stream();
```

### 2. 从数组创建

```java
// 对象数组
String[] array = {"a", "b", "c"};
Stream<String> stream4 = Arrays.stream(array);
Stream<String> stream5 = Stream.of(array);

// 基本类型数组
int[] intArray = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(intArray);

// 指定范围
Stream<String> stream6 = Arrays.stream(array, 0, 2); // 索引 0-1
```

### 3. 通过 Stream.of()

```java
Stream<String> stream7 = Stream.of("a", "b", "c");
Stream<Integer> stream8 = Stream.of(1, 2, 3, 4, 5);
Stream<String> emptyStream = Stream.empty(); // 空流
```

### 4. 通过 Stream.builder()

```java
Stream<String> stream9 = Stream.<String>builder()
    .add("a")
    .add("b")
    .add("c")
    .build();
```

### 5. 无限流

```java
// 迭代生成
Stream<Integer> iterateStream = Stream.iterate(0, n -> n + 2)
    .limit(10); // 0, 2, 4, 6, 8...

// Java 9+ 带条件的迭代
Stream<Integer> iterateStream2 = Stream.iterate(0, n -> n < 20, n -> n + 2);

// 随机生成
Stream<Double> generateStream = Stream.generate(Math::random)
    .limit(5);
```

### 6. 基本类型流

```java
// IntStream
IntStream intStream1 = IntStream.range(1, 5);      // 1, 2, 3, 4
IntStream intStream2 = IntStream.rangeClosed(1, 5); // 1, 2, 3, 4, 5
IntStream intStream3 = IntStream.of(1, 2, 3);

// LongStream
LongStream longStream = LongStream.range(1L, 1000000L);

// DoubleStream
DoubleStream doubleStream = DoubleStream.of(1.1, 2.2, 3.3);
```

### 7. 从文件创建

```java
// 读取文件每一行
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}
```

### 8. 从字符串创建

```java
// 字符流
IntStream charStream = "hello".chars();

// 正则表达式分割
Stream<String> wordStream = Pattern.compile("\\s+")
    .splitAsStream("hello world java stream");
```

------

## 中间操作

### 1. filter - 过滤

过滤出符合条件的元素。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 过滤偶数
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// 结果: [2, 4, 6, 8, 10]

// 多条件过滤
List<Integer> result = numbers.stream()
    .filter(n -> n % 2 == 0)
    .filter(n -> n > 5)
    .collect(Collectors.toList());
// 结果: [6, 8, 10]
```

### 2. map - 映射转换

将每个元素转换为另一个元素。

```java
List<String> words = Arrays.asList("hello", "world", "java");

// 转大写
List<String> upperCase = words.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// 结果: ["HELLO", "WORLD", "JAVA"]

// 获取长度
List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList());
// 结果: [5, 5, 4]

// 对象转换
List<User> users = Arrays.asList(
    new User("Alice", 25),
    new User("Bob", 30)
);
List<String> names = users.stream()
    .map(User::getName)
    .collect(Collectors.toList());
```

### 3. flatMap - 扁平化映射

将多个流合并为一个流。

```java
List<List<Integer>> nestedList = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);

// 扁平化
List<Integer> flatList = nestedList.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
// 结果: [1, 2, 3, 4, 5, 6, 7, 8, 9]

// 字符串拆分
List<String> sentences = Arrays.asList("hello world", "java stream");
List<String> allWords = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList());
// 结果: ["hello", "world", "java", "stream"]
```

### 4. distinct - 去重

去除重复元素(基于 equals 方法)。

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5, 5);

List<Integer> distinctNumbers = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// 结果: [1, 2, 3, 4, 5]

// 对象去重(需要重写 equals 和 hashCode)
List<User> uniqueUsers = users.stream()
    .distinct()
    .collect(Collectors.toList());
```

### 5. sorted - 排序

对流中的元素进行排序。

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

// 自然排序
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
// 结果: [1, 2, 3, 5, 8, 9]

// 逆序排序
List<Integer> reverseSorted = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
// 结果: [9, 8, 5, 3, 2, 1]

// 自定义排序
List<User> users = Arrays.asList(
    new User("Alice", 25),
    new User("Bob", 30),
    new User("Charlie", 20)
);

// 按年龄排序
List<User> sortedByAge = users.stream()
    .sorted(Comparator.comparing(User::getAge))
    .collect(Collectors.toList());

// 多条件排序
List<User> multiSorted = users.stream()
    .sorted(Comparator.comparing(User::getAge)
                      .thenComparing(User::getName))
    .collect(Collectors.toList());
```

### 6. peek - 查看

用于调试,查看流中的元素。

```java
List<Integer> result = numbers.stream()
    .peek(n -> System.out.println("原始值: " + n))
    .filter(n -> n > 5)
    .peek(n -> System.out.println("过滤后: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("转换后: " + n))
    .collect(Collectors.toList());
```

### 7. limit - 限制数量

限制流中元素的数量。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> limited = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());
// 结果: [1, 2, 3, 4, 5]

// 配合无限流使用
List<Integer> first10Even = Stream.iterate(0, n -> n + 2)
    .limit(10)
    .collect(Collectors.toList());
// 结果: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

### 8. skip - 跳过元素

跳过前 n 个元素。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> skipped = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());
// 结果: [6, 7, 8, 9, 10]

// 分页
int pageSize = 3;
int pageNumber = 2; // 第2页(0-based)
List<Integer> page = numbers.stream()
    .skip(pageNumber * pageSize)
    .limit(pageSize)
    .collect(Collectors.toList());
// 结果: [7, 8, 9]
```

### 9. mapToInt / mapToLong / mapToDouble

转换为基本类型流,避免装箱拆箱。

```java
List<String> strings = Arrays.asList("1", "2", "3", "4", "5");

// 转换为 IntStream
int sum = strings.stream()
    .mapToInt(Integer::parseInt)
    .sum();

// IntStream 特有方法
IntSummaryStatistics stats = strings.stream()
    .mapToInt(Integer::parseInt)
    .summaryStatistics();

System.out.println("总和: " + stats.getSum());
System.out.println("平均值: " + stats.getAverage());
System.out.println("最大值: " + stats.getMax());
System.out.println("最小值: " + stats.getMin());
System.out.println("数量: " + stats.getCount());
```

### 10. boxed - 装箱

将基本类型流转换为包装类型流。

```java
IntStream intStream = IntStream.range(1, 10);
Stream<Integer> boxedStream = intStream.boxed();

List<Integer> list = IntStream.range(1, 10)
    .boxed()
    .collect(Collectors.toList());
```

------

## 终端操作

### 1. forEach - 遍历

对每个元素执行操作。

```java
List<String> list = Arrays.asList("a", "b", "c");

// forEach
list.stream().forEach(System.out::println);

// forEachOrdered (保证顺序,即使在并行流中)
list.parallelStream().forEachOrdered(System.out::println);
```

### 2. collect - 收集

将流转换为其他数据结构。

```java
List<String> list = Arrays.asList("a", "b", "c");

// 转 List
List<String> resultList = list.stream().collect(Collectors.toList());

// 转 Set
Set<String> resultSet = list.stream().collect(Collectors.toSet());

// 转 Map
Map<String, Integer> map = list.stream()
    .collect(Collectors.toMap(s -> s, String::length));

// 转数组
String[] array = list.stream().toArray(String[]::new);

// 拼接字符串
String joined = list.stream().collect(Collectors.joining(", "));
// 结果: "a, b, c"
```

### 3. reduce - 归约

将流中的元素组合起来。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 求和
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
// 或
int sum2 = numbers.stream()
    .reduce(0, Integer::sum);

// 求最大值
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);

// 字符串连接
List<String> words = Arrays.asList("Hello", " ", "World");
String sentence = words.stream()
    .reduce("", (a, b) -> a + b);
// 结果: "Hello World"

// 累积计算
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
// 结果: 120 (1*2*3*4*5)
```

### 4. count - 计数

统计流中元素的数量。

```java
List<String> list = Arrays.asList("a", "b", "c", "d", "e");

long count = list.stream().count();
// 结果: 5

long evenCount = Arrays.asList(1, 2, 3, 4, 5, 6).stream()
    .filter(n -> n % 2 == 0)
    .count();
// 结果: 3
```

### 5. anyMatch / allMatch / noneMatch - 匹配

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// anyMatch: 任意一个满足
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);
// 结果: true

// allMatch: 全部满足
boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);
// 结果: true

// noneMatch: 全部不满足
boolean noNegative = numbers.stream()
    .noneMatch(n -> n < 0);
// 结果: true
```

### 6. findFirst / findAny - 查找

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// findFirst: 返回第一个元素
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 2)
    .findFirst();
// 结果: Optional[3]

// findAny: 返回任意一个元素(并行流中性能更好)
Optional<Integer> any = numbers.parallelStream()
    .filter(n -> n > 2)
    .findAny();
// 结果: 可能是 3, 4, 或 5
```

### 7. min / max - 最值

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

// 最小值
Optional<Integer> min = numbers.stream()
    .min(Integer::compareTo);
// 结果: Optional[1]

// 最大值
Optional<Integer> max = numbers.stream()
    .max(Integer::compareTo);
// 结果: Optional[9]

// 对象的最值
List<User> users = Arrays.asList(
    new User("Alice", 25),
    new User("Bob", 30),
    new User("Charlie", 20)
);

Optional<User> youngest = users.stream()
    .min(Comparator.comparing(User::getAge));
```

### 8. toArray - 转数组

```java
List<String> list = Arrays.asList("a", "b", "c");

// 转 Object[]
Object[] objArray = list.stream().toArray();

// 转指定类型数组
String[] strArray = list.stream().toArray(String[]::new);

// IntStream 转数组
int[] intArray = IntStream.range(1, 6).toArray();
```

------

## 收集器 Collectors

### 1. 基本收集

```java
List<String> list = Arrays.asList("a", "b", "c", "d");

// toList
List<String> resultList = list.stream().collect(Collectors.toList());

// toSet
Set<String> resultSet = list.stream().collect(Collectors.toSet());

// toCollection (指定集合类型)
LinkedList<String> linkedList = list.stream()
    .collect(Collectors.toCollection(LinkedList::new));

TreeSet<String> treeSet = list.stream()
    .collect(Collectors.toCollection(TreeSet::new));
```

### 2. toMap - 转换为 Map

```java
List<User> users = Arrays.asList(
    new User("Alice", 25),
    new User("Bob", 30),
    new User("Charlie", 20)
);

// 基本转换
Map<String, Integer> nameToAge = users.stream()
    .collect(Collectors.toMap(User::getName, User::getAge));

// 处理重复 key
Map<Integer, String> ageToName = users.stream()
    .collect(Collectors.toMap(
        User::getAge,
        User::getName,
        (existing, replacement) -> existing // 保留第一个
    ));

// 指定 Map 类型
TreeMap<String, Integer> treeMap = users.stream()
    .collect(Collectors.toMap(
        User::getName,
        User::getAge,
        (v1, v2) -> v1,
        TreeMap::new
    ));
```

### 3. joining - 字符串拼接

```java
List<String> list = Arrays.asList("A", "B", "C", "D");

// 直接拼接
String result1 = list.stream().collect(Collectors.joining());
// 结果: "ABCD"

// 指定分隔符
String result2 = list.stream().collect(Collectors.joining(", "));
// 结果: "A, B, C, D"

// 指定前缀、分隔符、后缀
String result3 = list.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// 结果: "[A, B, C, D]"
```

### 4. groupingBy - 分组

```java
List<User> users = Arrays.asList(
    new User("Alice", 25, "IT"),
    new User("Bob", 30, "HR"),
    new User("Charlie", 20, "IT"),
    new User("David", 25, "HR")
);

// 按部门分组
Map<String, List<User>> byDept = users.stream()
    .collect(Collectors.groupingBy(User::getDepartment));
// 结果: {IT=[Alice, Charlie], HR=[Bob, David]}

// 按年龄分组,统计人数
Map<Integer, Long> ageCount = users.stream()
    .collect(Collectors.groupingBy(
        User::getAge,
        Collectors.counting()
    ));

// 多级分组
Map<String, Map<Integer, List<User>>> multiGroup = users.stream()
    .collect(Collectors.groupingBy(
        User::getDepartment,
        Collectors.groupingBy(User::getAge)
    ));

// 分组后映射
Map<String, List<String>> deptNames = users.stream()
    .collect(Collectors.groupingBy(
        User::getDepartment,
        Collectors.mapping(User::getName, Collectors.toList())
    ));
```

### 5. partitioningBy - 分区

按条件分为两组(true/false)。

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 按奇偶分区
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
// 结果: {false=[1,3,5,7,9], true=[2,4,6,8,10]}

// 分区后计数
Map<Boolean, Long> partitionCount = numbers.stream()
    .collect(Collectors.partitioningBy(
        n -> n % 2 == 0,
        Collectors.counting()
    ));
```

### 6. 统计相关

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// counting - 计数
Long count = numbers.stream().collect(Collectors.counting());

// summingInt - 求和
Integer sum = numbers.stream().collect(Collectors.summingInt(n -> n));

// averagingInt - 平均值
Double average = numbers.stream().collect(Collectors.averagingInt(n -> n));

// summarizingInt - 统计信息
IntSummaryStatistics stats = numbers.stream()
    .collect(Collectors.summarizingInt(n -> n));
System.out.println("总和: " + stats.getSum());
System.out.println("平均值: " + stats.getAverage());
System.out.println("最大值: " + stats.getMax());
System.out.println("最小值: " + stats.getMin());
System.out.println("数量: " + stats.getCount());

// maxBy / minBy
Optional<Integer> max = numbers.stream()
    .collect(Collectors.maxBy(Integer::compareTo));
```

### 7. reducing - 归约收集器

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 求和
Integer sum = numbers.stream()
    .collect(Collectors.reducing(0, Integer::sum));

// 求积
Integer product = numbers.stream()
    .collect(Collectors.reducing(1, (a, b) -> a * b));

// 字符串拼接
List<String> words = Arrays.asList("Hello", "World", "Java");
String sentence = words.stream()
    .collect(Collectors.reducing("", (a, b) -> a + " " + b));
```

### 8. collectingAndThen - 收集后再转换

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 收集到List后转为不可变List
List<Integer> unmodifiableList = numbers.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    ));

// 统计后取平均值
Double average = numbers.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.summarizingInt(n -> n),
        IntSummaryStatistics::getAverage
    ));
```

------

## 实战示例

### 示例1: 数据过滤和转换

```java
// 场景: 从用户列表中找出年龄大于25岁的用户姓名,并转大写
List<User> users = Arrays.asList(
    new User("alice", 23),
    new User("bob", 28),
    new User("charlie", 30),
    new User("david", 22)
);

List<String> result = users.stream()
    .filter(user -> user.getAge() > 25)
    .map(User::getName)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// 结果: ["BOB", "CHARLIE"]
```

### 示例2: 复杂分组统计

```java
// 场景: 统计各部门的平均工资
List<Employee> employees = Arrays.asList(
    new Employee("Alice", "IT", 8000),
    new Employee("Bob", "IT", 9000),
    new Employee("Charlie", "HR", 7000),
    new Employee("David", "HR", 7500),
    new Employee("Eve", "Finance", 10000)
);

Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
// 结果: {IT=8500.0, HR=7250.0, Finance=10000.0}
```

### 示例3: 扁平化处理

```java
// 场景: 从多个订单中提取所有商品名称
class Order {
    private List<String> items;
    // constructor, getters
}

List<Order> orders = Arrays.asList(
    new Order(Arrays.asList("Apple", "Banana")),
    new Order(Arrays.asList("Orange", "Grape")),
    new Order(Arrays.asList("Apple", "Watermelon"))
);

List<String> allItems = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .distinct()
    .sorted()
    .collect(Collectors.toList());
// 结果: [Apple, Banana, Grape, Orange, Watermelon]
```

### 示例4: 多条件排序

```java
// 场景: 按部门升序、工资降序排列员工
List<Employee> sorted = employees.stream()
    .sorted(Comparator.comparing(Employee::getDepartment)
                      .thenComparing(Comparator.comparing(Employee::getSalary).reversed()))
    .collect(Collectors.toList());
```

### 示例5: 分页

```java
// 场景: 实现简单的分页功能
List<Integer> data = IntStream.rangeClosed(1, 100)
    .boxed()
    .collect(Collectors.toList());

int pageSize = 10;
int pageNumber = 3; // 第3页 (0-based)

List<Integer> page = data.stream()
    .skip(pageNumber * pageSize)
    .limit(pageSize)
    .collect(Collectors.toList());
// 结果: [31, 32, 33, 34, 35, 36, 37, 38, 39, 40]
```

### 示例6: 去重保留最新

```java
// 场景: 根据用户ID去重,保留最新的记录
class UserRecord {
    private String userId;
    private LocalDateTime timestamp;
    private String data;
    // constructor, getters
}

List<UserRecord> records = Arrays.asList(
    new UserRecord("001", LocalDateTime.now().minusDays(2), "old"),
    new UserRecord("001", LocalDateTime.now(), "new"),
    new UserRecord("002", LocalDateTime.now(), "data")
);

Map<String, UserRecord> latestRecords = records.stream()
    .collect(Collectors.toMap(
        UserRecord::getUserId,
        record -> record,
        (existing, replacement) -> 
            existing.getTimestamp().isAfter(replacement.getTimestamp()) 
                ? existing : replacement
    ));
```

### 示例7: 词频统计

```java
// 场景: 统计文本中各单词出现的频率
String text = "hello world hello java world java stream";

Map<String, Long> wordFrequency = Arrays.stream(text.split(" "))
    .collect(Collectors.groupingBy(
        word -> word,
        Collectors.counting()
    ));
// 结果: {hello=2, world=2, java=2, stream=1}

// 找出出现最多的单词
Optional<Map.Entry<String, Long>> mostFrequent = wordFrequency.entrySet().stream()
    .max(Map.Entry.comparingByValue());
```

### 示例8: 数据转换和聚合

```java
// 场景: 计算购物车总价
class CartItem {
    private String name;
    private double price;
    private int quantity;
    // constructor, getters
}

List<CartItem> cart = Arrays.asList(
    new CartItem("Apple", 2.5, 3),
    new CartItem("Banana", 1.5, 5),
    new CartItem("Orange", 3.0, 2)
);

double totalPrice = cart.stream()
    .mapToDouble(item -> item.getPrice() * item.getQuantity())
    .sum();
// 结果: 21.0
```

### 示例9: 条件过滤和分组

```java
// 场景: 按成绩等级分组学生
class Student {
    private String name;
    private int score;
    // constructor, getters
}

List<Student> students = Arrays.asList(
    new Student("Alice", 95),
    new Student("Bob", 82),
    new Student("Charlie", 67),
    new Student("David", 78)
);

Map<String, List<Student>> gradeGroups = students.stream()
    .collect(Collectors.groupingBy(student -> {
        int score = student.getScore();
        if (score >= 90) return "A";
        else if (score >= 80) return "B";
        else if (score >= 70) return "C";
        else return "D";
    }));
// 结果: {A=[Alice], B=[Bob], C=[David], D=[Charlie]}
```

### 示例10: 复杂对象构建

```java
// 场景: 从多个数据源构建报表
List<Integer> ids = Arrays.asList(1, 2, 3);
Map<Integer, String> names = Map.of(1, "Alice", 2, "Bob", 3, "Charlie");
Map<Integer, Integer> scores = Map.of(1, 95, 2, 88, 3, 92);

class Report {
    private int id;
    private String name;
    private int score;
    // constructor, getters
}

List<Report> reports = ids.stream()
    .map(id -> new Report(
        id,
        names.getOrDefault(id, "Unknown"),
        scores.getOrDefault(id, 0)
    ))
    .filter(report -> report.getScore() >= 90)
    .sorted(Comparator.comparing(Report::getScore).reversed())
    .collect(Collectors.toList());
```

------

## 性能优化

### 1. 使用基本类型流

避免装箱拆箱的性能损耗。

```java
// ❌ 不推荐
int sum = IntStream.range(1, 1000000)
    .boxed()
    .reduce(0, Integer::sum);

// ✅ 推荐
int sum = IntStream.range(1, 1000000)
    .sum();
```

### 2. 短路操作

使用 `findFirst`、`findAny`、`anyMatch` 等短路操作。

```java
// ❌ 不推荐 - 会遍历所有元素
boolean hasEven = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList())
    .size() > 0;

// ✅ 推荐 - 找到第一个就停止
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);
```

### 3. 并行流的使用

适合 CPU 密集型任务,数据量大的场景。

```java
// 串行流
long count = IntStream.range(1, 10000000)
    .filter(n -> isPrime(n))
    .count();

// 并行流 - 适合 CPU 密集型
long count = IntStream.range(1, 10000000)
    .parallel()
    .filter(n -> isPrime(n))
    .count();
```

**注意事项:**

- 数据量小时,并行流可能更慢(线程开销)
- 避免在并行流中使用有状态的操作
- 注意线程安全问题

### 4. 避免无意义的操作

```java
// ❌ 不推荐 - 多次遍历
List<Integer> result = numbers.stream()
    .filter(n -> n > 0)
    .collect(Collectors.toList())
    .stream()
    .map(n -> n * 2)
    .collect(Collectors.toList());

// ✅ 推荐 - 一次遍历
List<Integer> result = numbers.stream()
    .filter(n -> n > 0)
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

### 5. 选择合适的收集器

```java
// ❌ 低效
String result = list.stream()
    .reduce("", (a, b) -> a + b);

// ✅ 高效
String result = list.stream()
    .collect(Collectors.joining());
```

### 6. 延迟执行的利用

```java
// 中间操作不会立即执行
Stream<Integer> stream = numbers.stream()
    .filter(n -> {
        System.out.println("filtering: " + n);
        return n > 5;
    })
    .map(n -> {
        System.out.println("mapping: " + n);
        return n * 2;
    });

// 只有调用终端操作时才执行
List<Integer> result = stream.collect(Collectors.toList());
```

------

## 最佳实践

### 1. 优先使用 Stream API

```java
// ❌ 传统方式
List<String> result = new ArrayList<>();
for (String s : list) {
    if (s.length() > 3) {
        result.add(s.toUpperCase());
    }
}

// ✅ Stream 方式 - 更简洁
List<String> result = list.stream()
    .filter(s -> s.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 2. 合理使用方法引用

```java
// Lambda 表达式
list.stream().map(s -> s.toUpperCase())

// 方法引用 - 更简洁
list.stream().map(String::toUpperCase)
```

### 3. 避免副作用

```java
// ❌ 不推荐 - 修改外部变量
List<Integer> results = new ArrayList<>();
numbers.stream()
    .filter(n -> n > 5)
    .forEach(n -> results.add(n)); // 副作用

// ✅ 推荐 - 使用 collect
List<Integer> results = numbers.stream()
    .filter(n -> n > 5)
    .collect(Collectors.toList());
```

### 4. 及时关闭资源流

```java
// ✅ 使用 try-with-resources
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.filter(line -> line.contains("error"))
         .forEach(System.out::println);
}
```

### 5. 合理选择串行或并行

```java
// 数据量小或简单操作 - 串行
list.stream().map(String::toUpperCase).collect(Collectors.toList());

// 数据量大且 CPU 密集 - 并行
largeList.parallelStream()
    .filter(this::complexComputation)
    .collect(Collectors.toList());
```

### 6. 处理 Optional

```java
// ❌ 不推荐
Optional<String> result = list.stream().findFirst();
if (result.isPresent()) {
    System.out.println(result.get());
}

// ✅ 推荐
list.stream()
    .findFirst()
    .ifPresent(System.out::println);

// 或提供默认值
String value = list.stream()
    .findFirst()
    .orElse("default");
```

### 7. 链式调用的可读性

```java
// ✅ 适当换行提高可读性
List<String> result = users.stream()
    .filter(user -> user.isActive())
    .filter(user -> user.getAge() > 18)
    .map(User::getName)
    .map(String::toUpperCase)
    .sorted()
    .distinct()
    .collect(Collectors.toList());
```

### 8. 复用 Stream 配置

```java
// ❌ 不能复用 Stream
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
stream.count(); // 抛出异常: stream has already been operated upon

// ✅ 使用 Supplier
Supplier<Stream<String>> streamSupplier = () -> list.stream();
streamSupplier.get().forEach(System.out::println);
long count = streamSupplier.get().count();
```

------

## 常见陷阱

### 1. Stream 只能使用一次

```java
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
// ❌ 再次使用会抛出 IllegalStateException
stream.count();
```

### 2. 并行流的线程安全问题

```java
// ❌ 线程不安全
List<Integer> results = new ArrayList<>();
IntStream.range(1, 1000).parallel()
    .forEach(results::add); // ArrayList 非线程安全

// ✅ 使用线程安全的集合或 collect
List<Integer> results = IntStream.range(1, 1000)
    .parallel()
    .boxed()
    .collect(Collectors.toList());
```

### 3. 修改源数据

```java
// ❌ 避免在 Stream 操作中修改源集合
list.stream()
    .forEach(s -> list.remove(s)); // 可能抛出 ConcurrentModificationException
```

### 4. 无限流忘记 limit

```java
// ❌ 无限流
Stream.iterate(0, n -> n + 1)
    .forEach(System.out::println); // 永不停止

// ✅ 使用 limit
Stream.iterate(0, n -> n + 1)
    .limit(100)
    .forEach(System.out::println);
```

### 5. peek 的误用

```java
// ❌ peek 不应该用于修改元素(虽然技术上可以)
list.stream()
    .peek(user -> user.setAge(user.getAge() + 1))
    .collect(Collectors.toList());

// ✅ 使用 map
list.stream()
    .map(user -> {
        user.setAge(user.getAge() + 1);
        return user;
    })
    .collect(Collectors.toList());
```

### 6. groupingBy 的 null 值

```java
// ❌ 如果分组键为 null 会抛出 NullPointerException
Map<String, List<User>> grouped = users.stream()
    .collect(Collectors.groupingBy(User::getDepartment));

// ✅ 处理 null
Map<String, List<User>> grouped = users.stream()
    .collect(Collectors.groupingBy(
        user -> user.getDepartment() != null 
            ? user.getDepartment() 
            : "Unknown"
    ));
```

### 7. 装箱性能问题

```java
// ❌ 不必要的装箱
Stream<Integer> stream = IntStream.range(1, 1000000).boxed();
int sum = stream.reduce(0, Integer::sum);

// ✅ 使用基本类型流
int sum = IntStream.range(1, 1000000).sum();
```

### 8. 过度使用 Stream

```java
// ❌ 简单场景过度使用
String first = list.stream().findFirst().orElse(null);

// ✅ 直接访问更简单
String first = list.isEmpty() ? null : list.get(0);
```

------

## 总结

### Stream 的优势

- ✅ 代码简洁,可读性强
- ✅ 函数式编程风格
- ✅ 支持并行处理
- ✅ 惰性求值,提高性能
- ✅ 丰富的 API

### 使用建议

1. 数据转换优先使用 Stream
2. 简单场景不必强求 Stream
3. 注意性能和线程安全
4. 合理使用并行流
5. 处理好 Optional 和 null
6. 避免副作用和状态修改

### 学习路径

1. 掌握基本的创建和操作
2. 熟悉常用的中间和终端操作
3. 理解 Collectors 的使用
4. 学习性能优化技巧
5. 实践中积累经验

------

## 附录: 常用操作速查表

| 操作      | 类型 | 用途   | 示例                              |
| --------- | ---- | ------ | --------------------------------- |
| filter    | 中间 | 过滤   | `.filter(x -> x > 0)`             |
| map       | 中间 | 转换   | `.map(String::toUpperCase)`       |
| flatMap   | 中间 | 扁平化 | `.flatMap(Collection::stream)`    |
| distinct  | 中间 | 去重   | `.distinct()`                     |
| sorted    | 中间 | 排序   | `.sorted()`                       |
| limit     | 中间 | 限制   | `.limit(10)`                      |
| skip      | 中间 | 跳过   | `.skip(5)`                        |
| peek      | 中间 | 查看   | `.peek(System.out::println)`      |
| collect   | 终端 | 收集   | `.collect(Collectors.toList())`   |
| forEach   | 终端 | 遍历   | `.forEach(System.out::println)`   |
| reduce    | 终端 | 归约   | `.reduce(0, Integer::sum)`        |
| count     | 终端 | 计数   | `.count()`                        |
| anyMatch  | 终端 | 匹配   | `.anyMatch(x -> x > 0)`           |
| findFirst | 终端 | 查找   | `.findFirst()`                    |
| min/max   | 终端 | 最值   | `.min(Comparator.naturalOrder())` |

------

**Happy Streaming! 🚀**