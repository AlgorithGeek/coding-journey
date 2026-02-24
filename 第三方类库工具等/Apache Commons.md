# Apache Commons

## 简述

- **基本概念**：

  - Apache Commons 是由 Apache 软件基金会维护的一个顶级开源项目，致力于提供全面、可重用的 Java 核心通用组件

    它可以被看作是 Java 标准库（`java.lang`、`java.util` 等）的“超级扩展包”

- **核心定位**：

  - Java 开发界的“瑞士军刀”

    它的唯一目的就是解决日常开发中繁琐、重复的基础代码工作，贯彻 DRY（Don't Repeat Yourself）原则，让你彻底告别“重复造轮子”

- **本文档中**

  - 整理了一些通常常用的方法等



## `commons-lang3` (Java 核心基础增强)

### 根包(最核心)

> org.apache.commons.lang3 

#### StringUtils

##### 1. 判空与空白检查

> **💡 核心差异提示**：与 Java 原生或 Spring 相比，Apache Commons 最大的优势在于提供了直接的反向方法（如 `isNotBlank`），避免了代码中出现繁琐且容易看漏的 `!isBlank()`。

###### `isBlank` / `isNotBlank`

> **`isBlank(CharSequence cs)`** 
>
> **`isNotBlank(CharSequence cs)`**

- **返回值**: `boolean`

- **核心功能**:

  - `isBlank` 检查字符串是否为 `null`、空字符串 `""`，或者**仅仅包含空白字符**（如空格、制表符、换行符等）。
  - 它的判断逻辑与 Spring 的 `StringUtils.hasText()` 完全一致，只是返回值相反。
  - `isNotBlank` 则是 `isBlank` 的反义词，这是日常开发中**使用频率最高**的方法，用于确保一个字符串有实际内容。

- **示例**:

  ```Java
  // isBlank 示例
  StringUtils.isBlank(null);      // true
  StringUtils.isBlank("");        // true
  StringUtils.isBlank(" \n\t ");  // true (纯空白字符)
  StringUtils.isBlank("hello");   // false
  
  // isNotBlank 示例 (推荐使用)
  StringUtils.isNotBlank(" \n\t "); // false
  StringUtils.isNotBlank("hello");  // true
  ```





###### `isEmpty` / `isNotEmpty`

> **`isEmpty(CharSequence cs)`** 
>
> **`isNotEmpty(CharSequence cs)`**

- **返回值**: `boolean`

- **核心功能**:

  - 仅仅检查字符串是否为 `null` 或者长度为 0 的空字符串 `""`。
  - **注意**：它不关心字符串里是不是全都是空格。如果传入 `" "`（一个空格），`isEmpty` 会返回 `false`。

- **示例**:

  Java

  ```
  StringUtils.isEmpty(null);   // true
  StringUtils.isEmpty("");     // true
  StringUtils.isEmpty(" ");    // false (与 isBlank 的关键区别)
  StringUtils.isEmpty("bob");  // false
  ```

##### 2. 截取与提取

###### `substring`

> **`substring(String str, int start)`** 
>
> **`substring(String str, int start, int end)`**

- **返回值**: `String`

- **核心功能**:

  - Null 安全的字符串截取工具。
  - 最大的好处是：即使传入的字符串是 `null`，或者索引越界（比如字符串长度只有 2，你要求截取到 5），它也不会抛出 `NullPointerException` 或 `StringIndexOutOfBoundsException`，而是优雅地返回 `null` 或能截取的最大部分。

- **示例**:

  ```java
  StringUtils.substring(null, 2);       // null
  StringUtils.substring("hello", 2);    // "llo"
  StringUtils.substring("hello", 2, 4); // "ll"
  // 索引越界保护
  StringUtils.substring("abc", 0, 10);  // "abc" (原生 String.substring 会抛异常)
  ```



###### `substringBefore` / `substringAfter`

> **`substringBefore(String str, String separator)`** 
>
> **`substringAfter(String str, String separator)`**

- **返回值**: `String`

- **核心功能**:

  - 提取指定分隔符**第一次**出现之前（或之后）的字符串。
  - 非常适合用于处理 URL、文件路径或特定格式的文本。

- **示例**:

  ```java
  StringUtils.substringBefore("user@example.com", "@"); // "user"
  StringUtils.substringAfter("user@example.com", "@");  // "example.com"
  
  // 如果找不到分隔符，substringBefore 返回原字符串，substringAfter 返回空串
  StringUtils.substringBefore("hello", "x");            // "hello"
  StringUtils.substringAfter("hello", "x");             // ""
  ```



###### `substringBetween`

> **`substringBetween(String str, String tag)`** 
>
> **`substringBetween(String str, String open, String close)`**

- **返回值**: `String`

- **核心功能**:

  - 提取嵌套在两个相同（或不同）标签之间的字符串。
  - 在解析简单的 XML/HTML 标签或提取括号内的内容时极其好用。

- **示例**:

  ```java
  // 相同标签
  StringUtils.substringBetween("tagabctag", "tag"); // "abc"
  
  // 不同标签
  StringUtils.substringBetween("y = [1, 2, 3]", "[", "]"); // "1, 2, 3"
  ```



##### 3. 拼接与拆分

###### `join`

> **`join(Iterable<?> iterable, String separator)`** 
>
> **`join(Object[] array, String separator)`**

- **返回值**: `String`

- **核心功能**:

  - 将一个集合（如 `List`、`Set`）或数组中的元素，使用指定的分隔符优雅地拼接成一个长字符串。
  - 全程 Null 安全，且处理空集合时返回空字符串。

- **示例**:

  ```java
  List<String> list = Arrays.asList("apple", "banana", "orange");
  StringUtils.join(list, ", ");     // "apple, banana, orange"
  
  Object[] array = {1, 2, 3};
  StringUtils.join(array, "-");     // "1-2-3"
  ```



###### `split`

> **`split(String str)`** 
>
> **`split(String str, String separatorChars)`**

- **返回值**: `String[]`

- **核心功能**:

  - 按空白字符（默认）或指定字符将字符串拆分为数组。
  - **核心优势**：与原生的 `String.split()` 不同，它不是基于正则表达式的，因此性能更好，且不会遇到正则表达式转义报错的问题。连续的分隔符会被视为一个。

- **示例**:

  ```java
  // 默认按空白字符拆分
  StringUtils.split("hello  world \t java"); // {"hello", "world", "java"}
  
  // 按指定字符拆分，连续的 '.' 会被当成一个
  StringUtils.split("a..b.c", ".");        // {"a", "b", "c"}
  ```



##### 4. 填充与缩略 (UI 展示格式化神器)

###### `leftPad` / `rightPad`

> **`leftPad(String str, int size, char padChar)`** 
>
> **`rightPad(String str, int size, char padChar)`**

- **返回值**: `String`

- **核心功能**:

  - 如果字符串长度不足指定的 `size`，则在左侧（或右侧）自动填充指定的字符，直到达到目标长度。
  - 常用于生成固定位数的流水号、对齐打印日志等。

- **示例**:

  ```java
  // 生成 4 位数的编号，不足补 0
  StringUtils.leftPad("12", 4, '0');  // "0012"
  StringUtils.leftPad("999", 4, '0'); // "0999"
  
  // 右侧补齐空格进行对齐
  StringUtils.rightPad("Name", 10, ' '); // "Name      "
  ```



###### `abbreviate`

> **`abbreviate(String str, int maxWidth)`**

- **返回值**: `String`

- **核心功能**:

  - 缩略字符串工具。如果字符串长度超过 `maxWidth`，则将其截断，并在末尾自动加上省略号 `...`。
  - 极其适合用于日志打印（防止日志过长刷屏）或前端 UI 的简略展示。

- **示例**:

  ```java
  // maxWidth 至少需要为 4 (因为 "..." 占了 3 个字符)
  String longText = "This is a very long descriptive text.";
  StringUtils.abbreviate(longText, 15); // "This is a ve..."
  ```



##### 5. 大小写转换

###### `capitalize` / `uncapitalize`

> **`capitalize(String str)`** 
>
> **`uncapitalize(String str)`**

- **返回值**: `String`

- **核心功能**:

  - 仅将字符串的首字母转换为大写（或小写），其余部分保持不变。

- **示例**:

  Java

  ```java
  StringUtils.capitalize("cat");   // "Cat"
  StringUtils.uncapitalize("Cat"); // "cat"
  ```



###### `upperCase` / `lowerCase`

> **`upperCase(String str)`** 
>
> **`lowerCase(String str)`**

- **返回值**: `String`

- **核心功能**:

  - Null 安全的整体大小写转换。如果传入 `null`，直接返回 `null` 而不报错。

- **示例**:

  ```java
  StringUtils.upperCase(null);     // null
  StringUtils.upperCase("hello");  // "HELLO"
  ```





#### ObjectUtils **对象安全操作**

##### 1. 默认值与首选值 (Null 值的优雅处理)

> **💡 核心差异提示**：
>
> 这是 `ObjectUtils` 中最闪光的功能。在处理可能为 `null` 的对象时，它能让你用一行代码完成“如果有值就用原值，没值就用默认值”的逻辑

###### `defaultIfNull` / `getIfNull`

> **`defaultIfNull(T object, T defaultValue)`** 
>
> **`getIfNull(T object, Supplier<T> defaultSupplier)`**

- **返回值**: `T` (泛型，与传入对象类型一致)

- **核心功能**:

  - `defaultIfNull`：如果传入的 `object` 为 `null`，则返回指定的 `defaultValue`；如果不为 `null`，则返回 `object` 本身。
  - `getIfNull`：这是 Java 8 之后的增强版。如果构建默认值的过程非常消耗性能，你可以传入一个 `Supplier`，只有在 `object` 为 `null` 时才会触发执行该 Supplier 来获取默认值（懒加载机制）。

- **示例**:

  ```java
  // 传统写法
  String status = getStatus();
  if (status == null) {
      status = "UNKNOWN";
  }
  
  // 使用 defaultIfNull (极其简洁)
  String status2 = ObjectUtils.defaultIfNull(getStatus(), "UNKNOWN");
  
  // 使用 getIfNull (懒加载，适用于默认值需要查数据库或复杂计算的场景)
  String config = ObjectUtils.getIfNull(getConfig(), () -> queryConfigFromDatabase());
  ```



###### `firstNonNull`

> **`firstNonNull(T... values)`**

- **返回值**: `T`

- **核心功能**:

  - 传入多个参数（可变参数），它会按顺序查找，并返回**第一个不是 `null` 的值**。如果所有传入的值都是 `null`，则最终返回 `null`
  - 非常适合用于有多级回退策略（Fallback）的场景

- **示例**:

  ```java
  // 按照优先级获取配置：先看用户自定义，再看系统环境变量，最后用默认值
  String finalConfig = ObjectUtils.firstNonNull(
      userCustomConfig,
      systemEnvConfig,
      "DEFAULT_CONFIG"
  );
  ```



##### 2. 多对象判空 (告别冗长的 && 和 ||)

###### `allNotNull` / `anyNotNull`

> **`allNotNull(Object... values)`** 
>
> **`anyNotNull(Object... values)`**

- **返回值**: `boolean`

- **核心功能**:

  - `allNotNull`：检查传入的所有对象，**是否全都不为 `null`**。只要有一个为 `null` 或传入的数组为空，就返回 `false`。
  - `anyNotNull`：检查传入的所有对象中，**是否至少有一个不为 `null`**。
  - 极大简化了表单多字段联合校验的逻辑。

- **示例**:

  ```java
  // 传统写法
  if (user != null && user.getName() != null && user.getAge() != null) { ... }
  
  // 使用 allNotNull
  if (ObjectUtils.allNotNull(user, user.getName(), user.getAge())) { ... }
  
  // 只要手机号或邮箱填了一个就可以
  if (ObjectUtils.anyNotNull(phone, email)) { ... }
  ```





###### `allNull` / `anyNull`

> **`allNull(Object... values)`** **`anyNull(Object... values)`**

- **返回值**: `boolean`
- **核心功能**:
  - 与上方方法相对：`allNull` 判断是否**全为 `null`**，`anyNull` 判断是否**至少包含一个 `null`**



##### 3. 万能判空

###### `isEmpty` / `isNotEmpty`

> **`isEmpty(Object object)`** **`isNotEmpty(Object object)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断一个任意类型的对象是否为空。
  - 它的强大之处在于它能自动识别类型并作出合理的判断：支持判断 `CharSequence`（字符串）、`Array`（数组）、`Collection`（集合）、`Map` 等是否为 null 或长度为 0。
  - *注：这与 Spring 的 `ObjectUtils.isEmpty()` 核心思想一致。*

- **示例**:

  ```java
  ObjectUtils.isEmpty(null);             // true
  ObjectUtils.isEmpty("");               // true
  ObjectUtils.isEmpty(new ArrayList<>());// true
  ObjectUtils.isEmpty(new int[0]);       // true
  ```





##### 4. 对象克隆 (安全的 Clone)

###### `clone` / `cloneIfPossible`

> **`clone(T obj)`** 
>
> **`cloneIfPossible(T obj)`**

- **返回值**: `T`

- **核心功能**:

  - `clone`：以 Null 安全的方式克隆一个对象。如果对象不支持克隆（未实现 `Cloneable`），会返回 `null`
  - `cloneIfPossible`：如果传入的对象支持克隆，则返回克隆的新对象；如果不支持克隆，则**原样返回**该对象本身（不报错）

- **示例**:

  ```java
  Date originalDate = new Date();
  Date copiedDate = ObjectUtils.clone(originalDate);
  ```



#### ArrayUtils **数组增强操作**

##### 1. 判空与长度检查

> **💡 核心差异提示**：在 Java 中直接调用 `array.length` 时，如果 `array` 为 `null` 会抛出空指针异常。`ArrayUtils` 提供了极其安全的检查方式。

###### `isEmpty` / `isNotEmpty`

> **`isEmpty(Object[] array)`** (以及各种基本类型数组的重载) 
>
> **`isNotEmpty(Object[] array)`**

- **返回值**: `boolean`

- **核心功能**:

  - `isEmpty`：判断数组是否为 `null` 或者长度为 0。
  - `isNotEmpty`：判断数组是否既不为 `null` 且包含至少一个元素。
  - 注意：它为所有基本类型（如 `int[]`, `long[]`）和对象类型（`Object[]`）都提供了重载方法。

- **示例**:

  ```java
  String[] arr1 = null;
  String[] arr2 = new String[0];
  String[] arr3 = {"a", "b"};
  
  ArrayUtils.isEmpty(arr1);    // true
  ArrayUtils.isEmpty(arr2);    // true
  ArrayUtils.isNotEmpty(arr3); // true
  ```



###### `isSameLength`

> **`isSameLength(Object[] array1, Object[] array2)`**

- **返回值**: `boolean`

- **核心功能**:

  - 安全地比较两个数组的长度是否相同。将 `null` 视为长度为 0。

- **示例**:

  ```java
  Object[] a = null;
  Object[] b = new Object[0];
  Object[] c = new Object[2];
  
  ArrayUtils.isSameLength(a, b); // true (null 和 空数组长度都被视为 0)
  ArrayUtils.isSameLength(a, c); // false
  ```



##### 2. 元素查找与包含

###### `contains`

> **`contains(Object[] array, Object objectToFind)`**

- **返回值**: `boolean`

- **核心功能**:

  - 检查数组中是否包含指定的元素。底层使用的是 `equals()` 方法进行比较。如果数组为 `null`，安全返回 `false`。

- **示例**:

  ```java
  String[] fruits = {"apple", "banana", "cherry"};
  ArrayUtils.contains(fruits, "banana"); // true
  ArrayUtils.contains(fruits, "orange"); // false
  ArrayUtils.contains(null, "apple");    // false
  ```



###### `indexOf` / `lastIndexOf`

> **`indexOf(Object[] array, Object objectToFind)`** 
>
> **`lastIndexOf(Object[] array, Object objectToFind)`**

- **返回值**: `int`

- **核心功能**:

  - 找出指定元素在数组中**第一次**（或**最后一次**）出现的索引。如果没找到或数组为 `null`，则返回 `-1`。

- **示例**:

  ```java
  int[] numbers = {10, 20, 30, 20, 50};
  ArrayUtils.indexOf(numbers, 20);     // 1
  ArrayUtils.lastIndexOf(numbers, 20); // 3
  ArrayUtils.indexOf(numbers, 99);     // -1
  ```



##### 3. 数组的增删操作 (打破长度固定的痛点)

> **⚠️ 重要注意**：
>
> - 由于 Java 原生数组的长度是不可变的，以下所有 `add` 和 `remove` 方法**都不会修改原数组**，而是**在底层创建一个新数组并返回**。请务必接收返回值！

###### `add` / `addAll`

> **`add(T[] array, T element)`** **`addAll(T[] array1, T... array2)`**

- **返回值**: `T[]` (返回一个包含了新元素的新数组)

- **核心功能**:

  - `add`：将指定的元素追加到数组的末尾。
  - `addAll`：将两个数组合并为一个新数组。

- **示例**:

  ```java
  String[] initial = {"a", "b"};
  
  // 追加单个元素 (必须用变量接收返回值)
  String[] added = ArrayUtils.add(initial, "c"); // {"a", "b", "c"}
  
  // 合并数组
  String[] more = {"d", "e"};
  String[] combined = ArrayUtils.addAll(initial, more); // {"a", "b", "d", "e"}
  ```



###### `remove` / `removeElement`

> **`remove(T[] array, int index)`** 
>
> **`removeElement(T[] array, Object element)`**

- **返回值**: `T[]` (返回删除了元素的新数组)

- **核心功能**:

  - `remove`：根据**索引位置**删除元素。
  - `removeElement`：根据**元素值**删除第一次出现的匹配项。

- **示例**:

  ```java
  String[] colors = {"red", "green", "blue", "green"};
  
  // 按索引删除
  String[] noRed = ArrayUtils.remove(colors, 0); // {"green", "blue", "green"}
  
  // 按值删除 (只删除第一个匹配的)
  String[] noGreen = ArrayUtils.removeElement(colors, "green"); // {"red", "blue", "green"}
  ```



##### 4. 类型转换与工具操作

###### `toObject` / `toPrimitive` (装箱与拆箱)

> **`toObject(int[] array)`** 
>
> **`toPrimitive(Integer[] array)`**

- **返回值**: 对应的包装类数组 / 基本类型数组

- **核心功能**:

  - 原生 Java 很难直接将 `int[]` 转换成 `Integer[]`（不能直接强转，通常需要写循环或 Stream API）。这两个方法可以一键完成基本类型数组和包装类数组之间的互相转换。

- **示例**:

  ```java
  int[] primitiveArray = {1, 2, 3};
  
  // 转为包装类数组
  Integer[] objectArray = ArrayUtils.toObject(primitiveArray);
  
  // 转回基本类型数组
  int[] backToPrimitive = ArrayUtils.toPrimitive(objectArray);
  ```



###### `reverse`

> **`reverse(Object[] array)`**

- **返回值**: `void`

- **核心功能**:

  - 原地反转数组的元素顺序。与 `add`/`remove` 不同，这个方法**直接修改原数组**，不需要接收返回值。

- **示例**:

  ```java
  String[] letters = {"A", "B", "C"};
  ArrayUtils.reverse(letters);
  // letters 此时变成了 {"C", "B", "A"}
  ```



#### BooleanUtils **布尔值安全与转换操作**

##### 1. Null 安全的真假判断 (彻底告别自动拆箱 NPE)

> **💡 核心差异提示**：在原生 Java 中，如果你写 `if (boolObj)` 且 `boolObj` 为 `null`，底层在自动拆箱为基本类型 `boolean` 时会直接抛出 `NullPointerException`。`BooleanUtils` 提供了极其优雅的防御方式。

###### `isTrue` / `isNotTrue`

###### `isFalse` / `isNotFalse`

> **`isTrue(Boolean bool)`** 
>
> **`isNotTrue(Boolean bool)`**
>
> **`isFalse(Boolean bool)`**

- **返回值**: `boolean`

- **核心功能**:

  - `isTrue`：只有当传入的包装类对象**确实不为 `null` 且值为 `true`** 时，才返回 `true`。其他所有情况（包括传入 `null`）都安全地返回 `false`。
  - `isNotTrue`：正好相反，只要不是 `true`（包括 `null` 和 `false`），都返回 `true`。
  - `isFalse`：只有明确为 `false` 才返回 `true`，传入 `null` 会安全地返回 `false`。

- **示例**:

  ```java
  Boolean b1 = Boolean.TRUE;
  Boolean b2 = Boolean.FALSE;
  Boolean b3 = null;
  
  // 危险的原生写法: if (b3) { ... } // 运行时直接抛出 NPE
  
  // 安全的 BooleanUtils 写法
  BooleanUtils.isTrue(b1); // true
  BooleanUtils.isTrue(b2); // false
  BooleanUtils.isTrue(b3); // false (安全避开 NPE)
  
  BooleanUtils.isNotTrue(b3); // true
  ```



##### 2. 默认值兜底机制

###### `toBooleanDefaultIfNull`

> **`toBooleanDefaultIfNull(Boolean bool, boolean valueIfNull)`**

- **返回值**: `boolean` (基本类型)

- **核心功能**:

  - 将包装类 `Boolean` 安全地拆箱为基本类型 `boolean`。如果传入的包装类为 `null`，则返回你指定的备用默认值 `valueIfNull`。

- **示例**:

  Java

  ```
  Boolean userAgreed = null;
  
  // 如果未明确表态（null），默认视为不统一 (false)
  boolean isAgreed = BooleanUtils.toBooleanDefaultIfNull(userAgreed, false); 
  ```

##### 3. 智能字符串解析 (对接老旧系统和奇葩接口的救星)

###### `toBoolean`

> **`toBoolean(String str)`** 
>
> **`toBoolean(int val)`**

- **返回值**: `boolean`

- **核心功能**:

  - 极其强大的字符串与数字解析器。原生 `Boolean.parseBoolean()` 只有在字符串严格等于 `"true"`（忽略大小写）时才返回 `true`，其余全为 `false`。
  - `BooleanUtils.toBoolean` 则能听懂人类的习惯表达：它会将 `"true"`, `"on"`, `"y"`, `"t"`, `"yes"` 解析为 `true`，将 `"false"`, `"off"`, `"n"`, `"f"`, `"no"` 解析为 `false`。如果传入无法识别的值或 `null`，返回 `false`。
  - 对于数字：`0` 为 `false`，非 `0` 全部为 `true`。

- **示例**:

  ```java
  // 解析字符串
  BooleanUtils.toBoolean("yes");  // true
  BooleanUtils.toBoolean("on");   // true
  BooleanUtils.toBoolean("y");    // true
  BooleanUtils.toBoolean("false");// false
  BooleanUtils.toBoolean("no");   // false
  
  // 解析数字
  BooleanUtils.toBoolean(1);      // true
  BooleanUtils.toBoolean(-1);     // true
  BooleanUtils.toBoolean(0);      // false
  ```



##### 4. 人性化的格式化输出

###### `toStringYesNo` / `toStringOnOff` / `toStringTrueFalse`

> **`toStringYesNo(Boolean bool)`** 
>
> **`toStringOnOff(Boolean bool)`**

- **返回值**: `String`

- **核心功能**:

  - 将布尔值转换为对普通用户更友好的英文单词。
  - 如果传入 `null`，这些方法都会安全地返回 `null`。

- **示例**:

  ```java
  Boolean status = true;
  
  BooleanUtils.toStringYesNo(status);    // "yes"
  BooleanUtils.toStringOnOff(status);    // "on"
  BooleanUtils.toStringTrueFalse(status);// "true"
  
  BooleanUtils.toStringYesNo(false);     // "no"
  BooleanUtils.toStringOnOff(null);      // null
  ```



#### RandomStringUtils **随机字符串生成**

##### 1. 常用场景快捷生成 (字母与数字系列)

> **💡 核心差异提示**：
>
> - 在没有这个工具类之前，如果要生成一个随机字符串，通常需要自己定义一个包含所有字符的数组或字符串，然后用 `java.util.Random` 循环拼接，非常繁琐。
>
>   `RandomStringUtils` 将这些常见需求全部封装成了静态方法。 
>
> - **⚠️ 安全提醒**：这些方法底层默认使用的是 `java.util.Random`，这意味着它们**不是加密安全的**。
>
>   可以用来生成普通的短信验证码或订单号后缀，但**绝对不要**用来生成高安全级别的 Token 或密钥！

###### `randomAlphanumeric` (字母 + 数字)

> **`randomAlphanumeric(int count)`** **`randomAlphanumeric(int minLengthInclusive, int maxLengthExclusive)`**

- **返回值**: `String`

- **核心功能**:

  - 生成指定长度（或指定长度范围）的随机字符串，内容**仅包含大小写字母和数字**
  - 这是使用频率最高的方法，非常适合生成邀请码、普通密码、或者无特殊字符的随机标识

- **示例**:

  ```Java
  // 生成一个长度为 8 的字母数字混合字符串
  String inviteCode = RandomStringUtils.randomAlphanumeric(8); // 例: "h9KJ3bA1"
  
  // 生成长度在 6 (包含) 到 10 (不包含) 之间的混合字符串
  String dynamicCode = RandomStringUtils.randomAlphanumeric(6, 10); // 例: "Z8qX2"
  ```



###### `randomNumeric` (纯数字)

> **`randomNumeric(int count)`**

- **返回值**: `String`

- **核心功能**:

  - 生成指定长度的随机字符串，内容**仅包含数字 (0-9)**。
  - **绝佳场景**：生成手机短信验证码（如 4 位或 6 位纯数字验证码）。

- **示例**:

  ```Java
  // 生成 6 位纯数字验证码
  String smsCode = RandomStringUtils.randomNumeric(6); // 例: "849201"
  ```



###### `randomAlphabetic` (纯字母)

> **`randomAlphabetic(int count)`**

- **返回值**: `String`

- **核心功能**:

  - 生成指定长度的随机字符串，内容**仅包含大小写字母 (a-z, A-Z)**。

- **示例**:

  ```Java
  // 生成一个 5 位纯字母字符串
  String randomName = RandomStringUtils.randomAlphabetic(5); // 例: "XyBqz"
  ```



##### 2. 特殊字符与自定义字符集

###### `randomAscii` (包含符号)

> **`randomAscii(int count)`**

- **返回值**: `String`

- **核心功能**:

  - 生成指定长度的随机字符串，内容包含可见的 ASCII 字符（ASCII 码在 32 到 127 之间的字符）。
  - 除了字母和数字，它**还会包含各种标点符号**（如 `!`, `@`, `#`, 甚至空格等）。

- **示例**:

  ```java
  // 生成带有标点符号的复杂随机密码
  String complexPwd = RandomStringUtils.randomAscii(10); // 例: "P#7!g_2+Qx"
  ```



###### `random` (自定义候选字符)

> **`random(int count, String chars)`** 
>
> **`random(int count, char... chars)`**

- **返回值**: `String`

- **核心功能**:

  - 终极自定义方案。你可以指定一个字符串（或字符数组），生成的随机字符串将**只从你提供的这些字符中抽取**。
  - **绝佳场景**：生成图形验证码时，通常需要剔除容易混淆的字符（比如数字 `0` 和字母 `O`，数字 `1` 和小写字母 `l`）。

- **示例**:

  ```java
  // 去掉了容易混淆的 0, O, 1, l, I 的自定义字符集
  String safeChars = "23456789abcdefghijkmnopqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXYZ";
  
  // 从安全的字符集中生成 4 位图形验证码
  String captcha = RandomStringUtils.random(4, safeChars); // 例: "3xH9"
  ```



### 数学包

>org.apache.commons.lang3.math

#### NumberUtils **数字安全操作与转换**

##### 1. 安全的类型转换 (彻底告别 NumberFormatException)

> **💡 核心差异提示**：这是 `NumberUtils` 最常用的功能。它能在转换失败、遇到 `null` 或空字符串时，安静地返回一个默认值，绝不抛出异常破坏程序运行。

###### `toInt` / `toLong` / `toDouble` 等

> **`toInt(String str)`** 
>
> **`toInt(String str, int defaultValue)`** 
>
> *(同理包含 `toLong`、`toDouble`、`toFloat`、`toShort`、`toByte` 的一系列重载)*

- **返回值**: `int` (或其他对应的基本数据类型)

- **核心功能**:

  - 尝试将一个字符串转换为基本类型的数字
  - 如果传入的字符串是 `null`、空字符串，或者**根本不是合法的数字格式**（转换失败），它会直接返回你指定的 `defaultValue`
  - 如果调用没有 `defaultValue` 的单参数版本，转换失败时默认返回 `0`

- **示例**:

  ```Java
  // 原生危险写法：Integer.parseInt(null) 或 parseInt("abc") 会直接抛出异常报错
  
  // 安全转换，失败返回 0
  NumberUtils.toInt(null);       // 0
  NumberUtils.toInt("");         // 0
  NumberUtils.toInt("123a");     // 0 (解析失败)
  NumberUtils.toInt("123");      // 123
  
  // 安全转换，失败返回自定义默认值 (推荐用法)
  NumberUtils.toInt(null, -1);   // -1
  NumberUtils.toInt("abc", 99);  // 99
  
  // 浮点数同样适用
  NumberUtils.toDouble("3.14", 0.0); // 3.14
  ```



##### 2. 字符串数字格式验证

###### `isParsable`

> **`isParsable(String str)`**

- **返回值**: `boolean`

- **核心功能**:

  - 检查给定的字符串是否可以被原生 Java 方法（如 `Integer.parseInt()`、`Long.parseLong()` 等）成功解析。
  - 它能正确识别开头的加号 `+` 或减号 `-`，但不认识十六进制（`0x`开头）或后缀标识符（如 `123L` 的 `L`）。

- **示例**:

  ```Java
  NumberUtils.isParsable("123");    // true
  NumberUtils.isParsable("-123");   // true
  NumberUtils.isParsable("3.14");   // true
  NumberUtils.isParsable("0x1A");   // false (不认识十六进制)
  NumberUtils.isParsable("123L");   // false (不认识类型后缀)
  ```



###### `isCreatable`

> **`isCreatable(String str)`**

- **返回值**: `boolean`

- **核心功能**:

  - 比 `isParsable` 更加宽泛和强大，它检查字符串**是否是一个合法的 Java 数字字面量**。
  - 它不仅能识别普通数字，还能识别八进制（`0`开头）、十六进制（`0x`开头）、科学计数法（带有 `e` 或 `E`），以及带有类型限定符的数字（如以 `f`、`F`、`d`、`D`、`l`、`L` 结尾的字符串）。

- **示例**:

  ```Java
  NumberUtils.isCreatable("123");      // true
  NumberUtils.isCreatable("0x123");    // true (合法的十六进制)
  NumberUtils.isCreatable("123L");     // true (合法的 long 型)
  NumberUtils.isCreatable("1.2e3");    // true (合法的科学计数法)
  ```



##### 3. 智能创建 Number 对象

###### `createNumber`

> **`createNumber(String str)`**

- **返回值**: `java.lang.Number`

- **核心功能**:

  - 极其智能的解析器。它将字符串转变为 `java.lang.Number` 的具体子类实例。
  - 只要字符串是一个合法的数字，它会自动判断并返回最合适、精度最不丢失的类型（比如 `Integer`、`Long`、`Float`、`Double` 甚至是处理超大数字的 `BigDecimal` 或 `BigInteger`）。

- **示例**:

  ```java
  // 自动根据字符串特征返回不同的类型
  Number n1 = NumberUtils.createNumber("123");   // 返回 Integer 实例
  Number n2 = NumberUtils.createNumber("123L");  // 返回 Long 实例
  Number n3 = NumberUtils.createNumber("3.14F"); // 返回 Float 实例
  Number n4 = NumberUtils.createNumber("12345678901234567890"); // 返回超大数的最佳载体
  ```



##### 4. 数组最值查找

###### `max` / `min`

> **`max(int... array)`** (支持所有基本数据类型数组的重载) 
>
> **`min(int... array)`**

- **返回值**: 与数组元素一致的基本数据类型

- **核心功能**:

  - 快速找出一个基本类型数组中的最大值或最小值。
  - 如果传入的数组是 `null` 或者是空数组，它会抛出 `IllegalArgumentException`。

- **示例**:

  ```java
  int[] numbers = {10, 5, 8, 20, 2};
  
  int maxVal = NumberUtils.max(numbers); // 20
  int minVal = NumberUtils.min(numbers); // 2
  ```



### 时间包

>org.apache.commons.lang3.time





### 元组包

>org.apache.commons.lang3.tuple





## `commons-io` (文件与流处理神器)

### 核心包(io)

> org.apache.commons.io

#### FileUtils **文件与目录操作**

> **💡 核心差异提示**：`FileUtils` 提供了一系列极其易用的静态方法来操作文件和目录，特别是它的读写操作会自动处理流的打开和关闭，并且其目录操作（如递归删除、复制）弥补了原生 `File` 类的巨大短板。

##### 1. 快捷读写文件 (告别繁琐的 IO 流)

###### `readFileToString` / `readLines`

> **`readFileToString(File file, Charset encoding)`** 
>
> **`readLines(File file, Charset encoding)`**

- **返回值**: `String` / `List<String>`

- **核心功能**:

  - `readFileToString`：将整个文件的内容一次性读取到一个 `String` 中。
  - `readLines`：按行读取文件内容，返回一个包含所有行的 `List<String>`。
  - **优势**：自动处理了 `FileInputStream` 的开启、缓冲和最终的 `close()` 关闭动作，不会造成资源泄露。

- **示例**:

  ```java
  File file = new File("config.txt");
  
  // 读取为单行长字符串
  String content = FileUtils.readFileToString(file, StandardCharsets.UTF_8);
  
  // 按行读取为列表
  List<String> lines = FileUtils.readLines(file, StandardCharsets.UTF_8);
  for (String line : lines) {
      System.out.println(line);
  }
  ```



###### `writeStringToFile` / `writeByteArrayToFile`

> **`writeStringToFile(File file, String data, Charset encoding)`** 
>
> **`writeStringToFile(File file, String data, Charset encoding, boolean append)`**
>
>  **`writeByteArrayToFile(File file, byte[] data)`**

- **返回值**: `void`

- **核心功能**:

  - 将字符串或字节数组直接写入目标文件。如果目标文件所在的父目录不存在，它会**自动创建所有必需的父目录**。
  - 通过 `append` 参数可以控制是**覆盖**写入还是在文件末尾**追加**写入。

- **示例**:

  ```java
  File file = new File("logs/app.log");
  
  // 覆盖写入
  FileUtils.writeStringToFile(file, "Hello IO!", StandardCharsets.UTF_8);
  
  // 追加写入
  FileUtils.writeStringToFile(file, "\nNew Line", StandardCharsets.UTF_8, true);
  ```



##### 2. 强力创建与删除 (原生 File 类的痛点)

###### `forceMkdir`

> **`forceMkdir(File directory)`**

- **返回值**: `void`

- **核心功能**:

  - 创建一个目录，包括任何必要但不存在的父目录。这等同于原生 `File.mkdirs()`，但如果因为权限等原因创建失败，它会抛出明确的 `IOException` 而不是仅仅返回一个模糊的 `false`。

- **示例**:

  ```java
  File dir = new File("/path/to/deep/nested/folder");
  FileUtils.forceMkdir(dir);
  ```



###### `deleteDirectory` / `cleanDirectory` / `forceDelete`

> **`deleteDirectory(File directory)`** 
>
> **`cleanDirectory(File directory)`** 
>
> **`forceDelete(File file)`**

- **返回值**: `void`

- **核心功能**:

  - `deleteDirectory`：**递归删除一个目录**。（原生 `File.delete()` 如果目录不为空则会失败，而这个方法会连同里面的所有子文件、子文件夹一起删光）。
  - `cleanDirectory`：**清空一个目录但不删除该目录本身**。
  - `forceDelete`：强制删除一个文件或空目录。如果文件不存在会抛出 `FileNotFoundException`。

- **示例**:

  ```java
  File tempDir = new File("temp_workspace");
  
  // 清空里面的所有临时文件，但保留 temp_workspace 文件夹
  FileUtils.cleanDirectory(tempDir);
  
  // 连同 temp_workspace 文件夹一起从硬盘上彻底抹除
  FileUtils.deleteDirectory(tempDir);
  ```



##### 3. 文件与目录复制

###### `copyFile` / `copyDirectory`

> **`copyFile(File srcFile, File destFile)`** 
>
> **`copyDirectory(File srcDir, File destDir)`**

- **返回值**: `void`

- **核心功能**:

  - 将文件或整个目录复制到新的位置，并且默认会**保留文件的最后修改时间（file dates）**。
  - 如果目标文件或目录不存在，会自动创建。

- **示例**:

  ```java
  File source = new File("source.txt");
  File dest = new File("backup/source_backup.txt");
  
  // 复制单个文件
  FileUtils.copyFile(source, dest);
  
  File srcDir = new File("my_project");
  File destDir = new File("backup/my_project");
  
  // 递归复制整个文件夹及其内容
  FileUtils.copyDirectory(srcDir, destDir);
  ```



##### 4. 大小统计与人性化显示

###### `sizeOfDirectory` / `byteCountToDisplaySize`

> **`sizeOfDirectory(File directory)`** 
>
> **`byteCountToDisplaySize(long size)`**

- **返回值**: `long` / `String`

- **核心功能**:

  - `sizeOfDirectory`：原生 API 无法直接获取文件夹大小（只能获取单文件大小），此方法会递归计算目录下所有文件的总大小（以字节为单位）。
  - `byteCountToDisplaySize`：将字节数转换为人类可读的格式（例如将 `1024` 转为 `"1 KB"`，将 `1048576` 转为 `"1 MB"`）。

- **示例**:

  ```java
  File dir = new File("C:/Downloads");
  
  // 获取整个文件夹的总字节数
  long sizeBytes = FileUtils.sizeOfDirectory(dir);
  
  // 转为直观的 KB/MB/GB 字符串显示
  String displaySize = FileUtils.byteCountToDisplaySize(sizeBytes);
  System.out.println("Downloads 文件夹大小: " + displaySize); // 例如 "2 GB"
  ```

#### IOUtils **数据流操作**

> **💡 核心差异提示**：原生 Java 处理流的读写不仅代码臃肿，还容易因为忘记处理缓冲区导致性能问题。`IOUtils` 内部已经为你优化了缓冲机制。 
>
> **⚠️ 避坑警告**：`IOUtils` 的绝大多数读写方法（如 `copy`、`toString`）**都不会自动关闭流**。设计上它认为“谁打开的流，就该由谁关闭”。强烈建议配合 Java 7 的 `try-with-resources` 语法一起使用。

##### 1. 流的无缝对接 (下载/上传神器)

###### `copy`

> **`copy(InputStream input, OutputStream output)`** 
>
> **`copyLarge(InputStream input, OutputStream output)`**

- **返回值**: `int` / `long` (返回成功拷贝的字节数)

- **核心功能**:

  - 将一个输入流中的数据，源源不断地复制（写入）到一个输出流中。
  - 它是 Web 后端开发中**实现文件下载、文件上传转发最常用的方法**，底层默认使用了 4KB (老版本) 或 8KB (新版本) 的缓冲区。
  - 如果复制的数据可能超过 2GB，请务必使用 `copyLarge`，否则 `copy` 返回的 `int` 值会溢出为负数（抛出 `ArithmeticException`）。

- **示例**:

  ```Java
  // Web 下载文件的经典写法 (配合 try-with-resources 自动关流)
  try (InputStream in = new FileInputStream("large_video.mp4");
       OutputStream out = response.getOutputStream()) {
  
      // 一行代码搞定流的搬运！
      IOUtils.copy(in, out); 
  
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```



##### 2. 流与内容的极简转换

###### `toString`

> **`toString(InputStream input, Charset encoding)`**

- **返回值**: `String`

- **核心功能**:

  - 将输入流（`InputStream`）里的所有内容一次性读取出来，并转换成一个完整的字符串。
  - **绝佳场景**：调用第三方 HTTP 接口时，将返回的响应流（Response Stream）直接转为 JSON 字符串。

- **示例**:

  ```Java
  // 假设 connection 是一个 HttpURLConnection 对象
  try (InputStream in = connection.getInputStream()) {
      // 直接把网络流读成字符串
      String jsonResponse = IOUtils.toString(in, StandardCharsets.UTF_8);
      System.out.println(jsonResponse);
  }
  ```



###### `toByteArray`

> **`toByteArray(InputStream input)`**

- **返回值**: `byte[]`

- **核心功能**:

  - 将整个输入流的内容读取到一个字节数组（`byte[]`）中。
  - **绝佳场景**：读取图片、音频等二进制流，或者需要在内存中反复多次处理流数据的场景。

- **示例**:

  ```Java
  try (InputStream in = new FileInputStream("image.png")) {
      // 把图片流完全读入内存字节数组
      byte[] imageBytes = IOUtils.toByteArray(in);
  }
  ```



##### 3. 快捷写入流

###### `write`

> **`write(String data, OutputStream output, Charset encoding)`** 
>
> **`write(byte[] data, OutputStream output)`**

- **返回值**: `void`

- **核心功能**:

  - 将字符串或字节数组直接写入到指定的输出流中。
  - 经常配合 `HttpServletResponse` 的 `OutputStream` 向前端直接输出文本或错误信息。

- **示例**:

  ```Java
  String message = "{\"status\": \"error\", \"msg\": \"Forbidden\"}";
  
  try (OutputStream out = response.getOutputStream()) {
      // 直接将字符串用 UTF-8 编码写入输出流
      IOUtils.write(message, out, StandardCharsets.UTF_8);
  }
  ```



##### 4. 流的内容比对

###### `contentEquals`

> **`contentEquals(InputStream input1, InputStream input2)`**

- **返回值**: `boolean`

- **核心功能**:

  - 比较两个输入流的内容是否完全相同。
  - 相比于逐个字节去写 `while` 循环比较，这个方法极大地简化了判断逻辑。

- **示例**:

  ```Java
  try (InputStream in1 = new FileInputStream("file_v1.txt");
       InputStream in2 = new FileInputStream("file_v2.txt")) {
  
      boolean isSame = IOUtils.contentEquals(in1, in2);
      System.out.println("两个文件内容是否一致: " + isSame);
  }
  ```

> *(注：早年很流行的 `IOUtils.closeQuietly()` 方法已被官方标记为废弃（Deprecated）。在 Java 7 引入 `try-with-resources` 语法后，官方强烈建议由 Java 语言层面来接管流的关闭动作)*。



#### FilenameUtils **文件名与路径处理**

> **💡 核心差异提示**：这个类的所有操作纯粹是**基于字符串处理**的，它**完全不会**去访问真实的物理硬盘。也就是说，哪怕你传给它的路径在硬盘上根本不存在，它也能完美帮你解析出文件名和后缀。 另外，它极其聪明，能同时认识 Windows 的 `\` 和 Unix 的 `/`。

##### 1. 信息提取 (获取名字和后缀)

> 以路径 `C:\dev\project\file.txt` 为例：

###### `getName` / `getBaseName` / `getExtension`

> **`getName(String filename)`** 
>
> **`getBaseName(String filename)`** 
>
> **`getExtension(String filename)`**

- **返回值**: `String`

- **核心功能**:

  - `getName`：获取完整的文件名（包含后缀），即 `file.txt`。
  - `getBaseName`：获取主文件名（不包含路径和后缀），即 `file`。
  - `getExtension`：安全地提取文件扩展名（不包含前面的 `.`），即 `txt`。如果文件没有后缀，返回空字符串 `""`。

- **示例**:

  ```java
  // 无论 Linux 风格还是 Windows 风格的路径，它都能正确识别
  String path = "/var/log/nginx/access.log";
  
  String name = FilenameUtils.getName(path);       // "access.log"
  String baseName = FilenameUtils.getBaseName(path);   // "access"
  String extension = FilenameUtils.getExtension(path); // "log"
  ```



##### 2. 路径提取与操作

###### `getFullPath` / `removeExtension`

> **`getFullPath(String filename)`** 
>
> **`removeExtension(String filename)`**

- **返回值**: `String`

- **核心功能**:

  - `getFullPath`：提取出文件所在的目录路径（包含最末尾的分隔符）。
  - `removeExtension`：将文件路径中的后缀名部分砍掉，保留剩下的所有内容。

- **示例**:

  ```java
  String path = "C:\\projects\\app.properties";
  
  String dirPath = FilenameUtils.getFullPath(path);    // "C:\projects\"
  String noExt = FilenameUtils.removeExtension(path);  // "C:\projects\app"
  ```



##### 3. 跨平台与规范化 (防止路径穿越漏洞)

###### `normalize`

> **`normalize(String filename)`**

- **返回值**: `String` (如果路径非法则返回 `null`)

- **核心功能**:

  - 规范化一个路径。它会处理掉路径中多余的 `.`（当前目录）和 `..`（上一级目录），并且会统一修复不一致的路径分隔符。
  - **绝佳场景**：极其适合用来防御 Web 安全中的**目录遍历攻击**（Directory Traversal Attack）。如果用户上传的文件名包含恶意的 `../` 企图跳出限制目录，`normalize` 可以将其抹平或直接返回 `null`。

- **示例**:

  ```java
  // 混乱的路径：包含双斜杠、单点和双点
  String messyPath = "C:\\foo\\\\bar\\..\\baz\\.\\file.txt";
  
  // 规范化处理
  String cleanPath = FilenameUtils.normalize(messyPath);
  // cleanPath 会变成: "C:\foo\baz\file.txt"
  ```



###### `separatorsToUnix` / `separatorsToWindows`

> **`separatorsToUnix(String path)`** 
>
> **`separatorsToWindows(String path)`**

- **返回值**: `String`

- **核心功能**:

  - 强制将路径中的所有分隔符替换为 Unix 风格（`/`）或 Windows 风格（`\`）。这在拼接跨平台路径或者存入数据库前统一格式时非常有用。

- **示例**:

  Java

  ```
  String winPath = "C:\\user\\docs\\a.txt";
  String unixPath = FilenameUtils.separatorsToUnix(winPath); 
  // unixPath 会变成: "C:/user/docs/a.txt"
  ```

##### 4. 后缀名判断

##### `isExtension`

> **`isExtension(String filename, String extension)`** **`isExtension(String filename, String... extensions)`** **`isExtension(String filename, Collection<String> extensions)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断文件是否属于指定的扩展名。最强大的地方在于它**可以同时传入多个扩展名（白名单）**。
  - **注意**：它对大小写是敏感的（在 Linux 上），在 Windows 上则忽略大小写。

- **示例**:

  ```java
  String file = "avatar.png";
  
  // 判断单后缀
  boolean isPng = FilenameUtils.isExtension(file, "png"); // true
  
  // 判断多后缀 (极其适合做上传文件的类型白名单校验)
  boolean isValidImage = FilenameUtils.isExtension(file, "jpg", "jpeg", "png", "gif"); // true
  ```



## `commons-collections4` (集合高级运算)

### 根包

> org.apache.commons.collections4

#### CollectionUtils **集合通用与数学运算**

> **💡 核心差异提示**：与 Spring 提供的 `CollectionUtils` 偏向于基础判空不同，Apache Commons 的 `CollectionUtils` 堪称**集合的数学运算大师**。它支持各种集合的交并补差，并且对 `null` 值极其包容。

##### 1. 判空与安全防御

###### `isEmpty` / `isNotEmpty`

> **`isEmpty(Collection<?> coll)`** **`isNotEmpty(Collection<?> coll)`**

- **返回值**: `boolean`

- **核心功能**:

  - `isEmpty`：判断集合是否为 `null` 或者大小为 0。
  - `isNotEmpty`：判断集合既不为 `null` 且包含至少一个元素。

- **示例**:

  Java

  ```
  List<String> list1 = null;
  List<String> list2 = new ArrayList<>();
  
  CollectionUtils.isEmpty(list1);    // true
  CollectionUtils.isEmpty(list2);    // true
  ```



###### `emptyIfNull` (防止 For 循环 NPE 的神器)

> **`emptyIfNull(Collection<T> collection)`**

- **返回值**: `Collection<T>`

- **核心功能**:

  - 如果传入的集合为 `null`，则返回一个不可变的空集合；如果不为 `null`，则原样返回。
  - **绝佳场景**：极其适合用在 `for` 循环或 `Stream` 操作前，彻底干掉 `if (list != null)` 这个烦人的判断。

- **示例**:

  Java

  ```
  List<String> users = getUserListFromDb(); // 可能返回 null
  
  // 危险写法：如果 users 为 null，直接抛出 NullPointerException
  // for (String user : users) { ... }
  
  // 优雅写法：不用写 if，即使为 null 也能安全跳过循环
  for (String user : CollectionUtils.emptyIfNull(users)) {
      System.out.println(user);
  }
  ```



##### 2. 集合的数学运算 (告别双重 For 循环)

> ⚠️ 注意：以下数学运算方法**不会修改原集合**，而是返回一个包含了计算结果的**新集合**。

###### `union` (并集)

> **`union(Iterable<? extends O> a, Iterable<? extends O> b)`**

- **返回值**: `Collection<O>`

- **核心功能**:

  - 求两个集合的**并集**，即把两个集合的所有元素合并在一起。如果元素有重复，它会保留最大基数（即保留出现次数较多的那个集合中的个数）。

- **示例**:

  Java

  ```
  List<String> listA = Arrays.asList("A", "B", "C");
  List<String> listB = Arrays.asList("B", "C", "D");
  
  Collection<String> result = CollectionUtils.union(listA, listB);
  // 结果为: ["A", "B", "C", "D"]
  ```



###### `intersection` (交集)

> **`intersection(Iterable<? extends O> a, Iterable<? extends O> b)`**

- **返回值**: `Collection<O>`

- **核心功能**:

  - 求两个集合的**交集**，即找出两个集合中**共同拥有**的元素。
  - **绝佳场景**：找出同时购买了商品 A 和商品 B 的用户 ID。

- **示例**:

  Java

  ```
  List<String> listA = Arrays.asList("A", "B", "C");
  List<String> listB = Arrays.asList("B", "C", "D");
  
  Collection<String> result = CollectionUtils.intersection(listA, listB);
  // 结果为: ["B", "C"]
  ```



###### `subtract` (差集)

> **`subtract(Iterable<? extends O> a, Iterable<? extends O> b)`**

- **返回值**: `Collection<O>`

- **核心功能**:

  - 求集合 A 减去集合 B 的**差集**，即找出存在于 A 中，但**不存在于 B 中**的元素。
  - **绝佳场景**：全量更新数据时，计算哪些记录是被删掉的，或者哪些是新增的。

- **示例**:

  Java

  ```
  List<String> oldIds = Arrays.asList("1", "2", "3");
  List<String> newIds = Arrays.asList("2", "3", "4");
  
  // 找出被删掉的 ID (存在于 old，但 new 里面没有了)
  Collection<String> deletedIds = CollectionUtils.subtract(oldIds, newIds); 
  // 结果为: ["1"]
  ```



##### 3. 比较与包含

###### `isEqualCollection`

> **`isEqualCollection(Collection<?> a, Collection<?> b)`**

- **返回值**: `boolean`

- **核心功能**:

  - 检查两个集合是否**完全相等（无视顺序）**。
  - 它会检查两个集合包含的元素是否完全一致，并且每个元素的出现次数也必须完全一样。原生 `List.equals()` 必须要求元素顺序也一致，而这个方法完美避开了顺序干扰。

- **示例**:

  Java

  ```
  List<String> listA = Arrays.asList("A", "B", "B");
  List<String> listB = Arrays.asList("B", "A", "B");
  
  // 原生判断：false (因为顺序不同)
  listA.equals(listB); 
  
  // CollectionUtils判断：true (元素和频次都相同，无视顺序)
  CollectionUtils.isEqualCollection(listA, listB);
  ```



###### `containsAny`

> **`containsAny(Collection<?> coll1, Collection<?> coll2)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断集合 `coll1` 中是否包含了 `coll2` 中的**任意一个**元素。只要有至少一个交集，就返回 `true`。

- **示例**:

  Java

  ```
  List<String> userRoles = Arrays.asList("USER", "EDITOR");
  List<String> adminRoles = Arrays.asList("ADMIN", "SUPER_ADMIN", "EDITOR");
  
  // 判断该用户是否拥有 adminRoles 里的任意一个权限
  boolean hasAccess = CollectionUtils.containsAny(userRoles, adminRoles); // true
  ```



##### 4. 安全添加

###### `addIgnoreNull`

> **`addIgnoreNull(Collection<T> collection, T object)`**

- **返回值**: `boolean` (是否添加成功)

- **核心功能**:

  - 尝试向集合中添加一个元素，**如果该元素为 `null`，则默默忽略，绝不抛异常也不会添加进集合**。
  - 非常适合用于收集可能返回 `null` 的方法结果，避免向 List 中塞入一堆 `null` 导致后续处理报错。

- **示例**:

  Java

  ```
  List<String> validNames = new ArrayList<>();
  
  CollectionUtils.addIgnoreNull(validNames, "Alice");
  CollectionUtils.addIgnoreNull(validNames, null); // 忽略
  CollectionUtils.addIgnoreNull(validNames, "Bob");
  
  // validNames 最终只有 ["Alice", "Bob"]
  ```



#### MapUtils **Map 安全取值与操作**

> **💡 核心差异提示**：`MapUtils` 最精髓的功能在于它提供了一整套自带**强制类型转换**和**默认值兜底**的 `get` 方法。无论 Map 里的 value 存的是字符串、数字还是其他奇葩格式，它都能尽可能安全地帮你转换成你想要的类型。

##### 1. 安全取值与类型转换 (彻底告别强转和 NPE)

###### `getString` / `getInteger` / `getBoolean` / `getLong` 等

> **`getString(Map<?,?> map, Object key)`** 
>
> **`getString(Map<?,?> map, Object key, String defaultValue)`** 
>
> *(同理包含 `getInteger`, `getBoolean`, `getDouble`, `getShort`, `getMap` 等一整套重载方法)*

- **返回值**: 对应的具体数据类型

- **核心功能**:

  - 尝试从 Map 中获取指定 key 的值，并**安全地转换**为指定的类型。
  - 如果 key 不存在，或者 Map 为 `null`，或者转换彻底失败，它会返回 `null`（或者返回你指定的 `defaultValue` 默认值）。
  - **超强兼容性**：如果 Map 里存的是字符串 `"123"`，你调用 `getInteger`，它会自动帮你把字符串解析成整型 `123`！同样，存的是数字 `1`，调用 `getBoolean` 会自动解析成 `true`。

- **示例**:

  Java

  ```
  Map<String, Object> map = new HashMap<>();
  map.put("age", "25");       // 故意存成了字符串
  map.put("active", 1);       // 故意存成了数字 1
  map.put("name", "Alice");
  
  // 原生危险写法 (极易报错)
  // Integer age = (Integer) map.get("age"); // 直接抛出 ClassCastException 强转异常！
  
  // MapUtils 优雅写法
  Integer age = MapUtils.getInteger(map, "age");       // 自动将 "25" 转换为 Integer 25
  Boolean isActive = MapUtils.getBoolean(map, "active");// 自动将 1 转换为 true
  
  // 带有默认值的兜底写法 (推荐)
  String city = MapUtils.getString(map, "city", "Unknown"); // 找不到 key，返回 "Unknown"
  ```



##### 2. 判空与防御

###### `isEmpty` / `isNotEmpty`

> **`isEmpty(Map<?,?> map)`** 
>
> **`isNotEmpty(Map<?,?> map)`**

- **返回值**: `boolean`

- **核心功能**:

  - 极其基础但必不可少的安全判空。`isEmpty` 检查 Map 是否为 `null` 或者大小为 0。

- **示例**:

  Java

  ```
  Map<String, String> map = null;
  MapUtils.isEmpty(map); // true
  ```



###### `emptyIfNull`

> **`emptyIfNull(Map<K,V> map)`**

- **返回值**: `Map<K,V>`

- **核心功能**:

  - 与 `CollectionUtils.emptyIfNull` 如出一辙。如果传入的 Map 为 `null`，返回一个不可变的空 Map。
  - **绝佳场景**：用于增强 `for` 循环遍历 Map，防止抛出空指针异常。

- **示例**:

  Java

  ```
  Map<String, String> configMap = getConfigFromDb(); // 可能返回 null
  
  // 优雅遍历，无需写 if (configMap != null)
  for (Map.Entry<String, String> entry : MapUtils.emptyIfNull(configMap).entrySet()) {
      System.out.println(entry.getKey() + ": " + entry.getValue());
  }
  ```



##### 3. 调试与输出神器

###### `verbosePrint` / `debugPrint`

> **`verbosePrint(PrintStream out, Object label, Map<?,?> map)`**

- **返回值**: `void`

- **核心功能**:

  - 打印 Map 的终极神器！原生 Map 的 `toString()` 往往杂乱无章，如果遇到 Map 里面嵌套 Map，更是根本没法看。
  - `verbosePrint` 会将 Map 及其内部嵌套的所有层级，**以极其漂亮的缩进格式、分行打印出来**，每个 key 和 value 都会清晰展示。
  - **绝佳场景**：在开发阶段，排查超级复杂的长串 JSON 转成的 Map 时使用。

- **示例**:

  Java

  ```
  Map<String, Object> nestedMap = new HashMap<>();
  // 假设里面塞了一堆复杂的层级...
  
  // 直接打印到控制台，"MyConfig" 是你给这次打印打的标签
  MapUtils.verbosePrint(System.out, "MyConfig", nestedMap);
  ```



#### ListUtils **List 专属与分片操作**

> **💡 核心差异提示**：虽然 `CollectionUtils` 已经提供了通用的集合操作，但 `ListUtils` 专门针对 `List` 接口提供了更精确的实现（例如返回结果确保是 `List` 而不是普通的 `Collection`），并且包含了一个极度好用的列表分片功能。

##### 1. 大集合分片 (处理海量数据的神技)

###### `partition`

> **`partition(List<T> list, int size)`**

- **返回值**: `List<List<T>>`

- **核心功能**:

  - 将一个大的 `List` 按照指定的 `size`（大小）均匀切分成多个小的 `List` 子集。
  - **绝佳场景**：**数据库批量插入**（比如你有 10 万条数据，数据库限制每次只能 `insert` 1000 条，用它一切就搞定了）或者**批量调用远程 API**（比如接口限制每次最多只能传 50 个 ID）。

- **示例**:

  Java

  ```
  List<Integer> largeList = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
  
  // 按每 3 个元素拆分为一个子集合
  List<List<Integer>> subLists = ListUtils.partition(largeList, 3);
  
  // 拆分结果 subLists 包含 3 个子 List:
  // [1, 2, 3]
  // [4, 5, 6]
  // [7, 8]
  
  // 数据库批量插入经典用法：
  for (List<Integer> batch : subLists) {
      // userMapper.batchInsert(batch);
  }
  ```



##### 2. 判空与默认值 (保证返回值类型)

###### `emptyIfNull` / `defaultIfNull`

> **`emptyIfNull(List<T> list)`** 
>
> **`defaultIfNull(List<T> list, List<T> defaultList)`**

- **返回值**: `List<T>`

- **核心功能**:

  - `emptyIfNull`：如果传入的 `List` 为 `null`，则返回一个不可变的空 `List`；如果不为 `null`，则原样返回。
  - `defaultIfNull`：如果传入的 `List` 为 `null`，则返回你指定的备用 `defaultList`。
  - **优势**：它与 `CollectionUtils.emptyIfNull` 类似，但它的返回值**明确规定是 `List`**。如果你实现一个接口方法要求必须返回 `List`，用这个包兜底就不会发生类型冲突。

- **示例**:

  Java

  ```
  public List<String> getUserNames() {
      List<String> namesFromDb = userMapper.getNames(); // 可能会返回 null
  
      // 绝对安全地返回 List，即使查不到也不会抛空指针或类型错误
      return ListUtils.emptyIfNull(namesFromDb);
  }
  ```



##### 3. 集合运算 (保留 List 特性的拼接)

###### `union` / `intersection`

> **`union(List<? extends E> list1, List<? extends E> list2)`** **`intersection(List<? extends E> list1, List<? extends E> list2)`**

- **返回值**: `List<E>`

- **核心功能**:

  - `union`：将 `list2` 追加到 `list1` 后面，合并成一个新的 `List` 并返回（底层使用了 `addAll`）。
  - `intersection`：求两个 `List` 的交集，返回一个新的 `List`。
  - **优势**：与 `CollectionUtils` 中的同名方法相比，`ListUtils` 明确返回 `List` 类型，在处理强依赖 `List` 接口特征（如有序性、允许通过索引访问）的数据时更加方便。

- **示例**:

  Java

  ```
  List<String> list1 = Arrays.asList("A", "B");
  List<String> list2 = Arrays.asList("C", "D");
  
  // 拼接两个 List (返回新 List，不污染原数据)
  List<String> combined = ListUtils.union(list1, list2); // ["A", "B", "C", "D"]
  ```



## `commons-codec` (加密与编解码)





## `commons-text` (文本高级处理)