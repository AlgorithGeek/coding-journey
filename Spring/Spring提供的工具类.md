# 简述

- Spring 框架提供了非常多功能强大的工具类，可以帮助开发者轻松处理文件操作、字符串处理、反射、集合操作等日常任务
- 这里找了一些常用的工具类



# `Assert` **断言工具**



# `StringUtils` **字符串操作**

## 基本概念

- `StringUtils` 是 Spring 框架 `org.springframework.util` 包下的一个强大的字符串处理工具类，它提供了大量静态方法，用于处理字符串

- 与 Java 自带的 `String` 类相比，`StringUtils` 的主要优势在于它对 `null` 值的处理非常友好

  在使用 `StringUtils` 的方法时，你几乎不需要进行 `if (str != null)` 这样的空指针判断，这使得代码更加简洁、健壮和易于维护



## 常用方法整理

### 1. 判空与匹配检查

#### `hasText`

> **`hasText(String str)`**

- **返回值**: `boolean`

- **核心功能**: 

  - 这是最常用也是最严格的检查。它会检查一个字符串是否**包含实际的文本内容**。以下三种情况都会返回 `false`：
    - 字符串为 `null`
    - 字符串是空串 `""`
    - 字符串**仅包含空白字符**（如空格、制表符 `\t`、换行符 `\n` 等）

- **示例**:

  ```Java
  // 返回 false 的情况
  StringUtils.hasText(null);         // false
  StringUtils.hasText("");           // false
  StringUtils.hasText(" ");          // false (单个空格)
  StringUtils.hasText("   \t \n  ");  // false (多个空白字符组合)
  
  // 返回 true 的情况
  StringUtils.hasText("hello");      // true
  StringUtils.hasText(" a ");        // true (因为包含了非空白字符 'a')
  ```



#### `hasLength`

> **`hasLength(String str)`**

- **返回值**: `boolean`

- **核心功能**: 

  - 这个方法的检查相对宽松，它只判断字符串**是否存在且长度大于0**。以下两种情况会返回 `false`：
    - 字符串为 `null`
    - 字符串是空串 `""`

- **重要区别**:

  -  `hasLength` **不关心**字符串的内容是否是空白字符

     只要字符串里有字符（包括空格），它的长度就大于0，因此会返回 `true`

- **示例**:

  ```Java
  // 返回 false 的情况
  StringUtils.hasLength(null);    // false
  StringUtils.hasLength("");      // false
  
  // 返回 true 的情况
  StringUtils.hasLength(" ");     // true (与 hasText 的关键区别)
  StringUtils.hasLength("hello"); // true
  ```



#### `containsWhitespace`

> **`containsWhitespace(CharSequence str)`**

- **返回值**: `boolean`

- **核心功能**: 

  - 检查给定的字符串中是否**包含至少一个空白字符**

    只要字符串中任何位置出现了空格、制表符或换行符等，该方法就会返回 `true`

- **示例**:

  ```java
  StringUtils.containsWhitespace("");        // false
  StringUtils.containsWhitespace("abc");     // false
  StringUtils.containsWhitespace("a b c");   // true (因为包含了空格)
  StringUtils.containsWhitespace("  \t");     // true (只包含空白字符)
  ```



#### `countOccurrencesOf`

> **`countOccurrencesOf(String str, String sub)`**

- **返回值**: `int`

- **核心功能**: 统计一个子字符串 (`sub`) 在主字符串 (`str`) 中出现的次数。这是一个**区分大小写**的非重叠计数

- **示例**:

  ```java
  StringUtils.countOccurrencesOf("hello world, hello everyone", "hello"); // 2
  StringUtils.countOccurrencesOf("java", "a");                            // 2
  StringUtils.countOccurrencesOf("ababab", "aba");     					
  												// 1 (非重叠计数，第一个 "aba" 匹配后，从第四个字符开始继续搜索)
  StringUtils.countOccurrencesOf("aaaaa", "aa");                          // 2 (非重叠计数)
  ```



#### `matchesCharacter`

> **`matchesCharacter(String str, char singleCharacter)`**

- **返回值**: `boolean`

- **核心功能**: 判断整个字符串是否**仅由**指定的某一个字符 (`singleCharacter`) 重复构成

- **示例**:

  ```java
  StringUtils.matchesCharacter("aaaa", 'a');   // true
  StringUtils.matchesCharacter("aaab", 'a');   // false
  StringUtils.matchesCharacter("b", 'a');      // false
  StringUtils.matchesCharacter(null, 'a');     // false
  ```



#### `substringMatch`

> **`substringMatch(CharSequence str, int index, CharSequence substring)`**

- **返回值**: `boolean`

- **核心功能**: 检查主字符串 (`str`) 从指定的起始索引 (`index`) 开始的部分，是否与给定的子字符串 (`substring`) 完全匹配

- **示例**:

  ```java
  CharSequence text = "hello spring framework";
  
  // 从索引 6 开始，是否是 "spring"
  boolean match1 = StringUtils.substringMatch(text, 6, "spring");
  // match1 的值是 true
  
  // 从索引 0 开始，是否是 "spring"
  boolean match2 = StringUtils.substringMatch(text, 0, "spring");
  // match2 的值是 false
  ```



### 2. 清洗、替换与删除

#### `trimAllWhitespace`

> **`trimAllWhitespace(String str)`**

- **返回值**: `String`

- **核心功能**: 移除字符串中**所有**的空白字符，包括开头、结尾以及字符串中间的部分。这是一个非常彻底的“净化”操作

- **示例**:

  ```java
  String textWithSpaces = "  hello   world  \t\n";
  String trimmedText = StringUtils.trimAllWhitespace(textWithSpaces);
  // trimmedText 的值是 "helloworld"
  ```



#### `replace`

>**`replace(String inString, String oldPattern, String newPattern)`**

- **返回值**: `String`

- **核心功能**: 将一个字符串 (`inString`) 中所有出现的旧子串 (`oldPattern`) 替换为新子串 (`newPattern`)。这是一个简单、直接的全局替换。

- **示例**:

  ```java
  String template = "User [name] is logged in from [ip].";
  String result1 = StringUtils.replace(template, "[name]", "admin");
  // result1 的值是 "User admin is logged in from [ip]."
  String result2 = StringUtils.replace(result1, "[ip]", "192.168.1.1");
  // result2 的值是 "User admin is logged in from 192.168.1.1."
  ```



#### `delete`

> **`delete(String inString, String pattern)`**

- **返回值**: `String`

- **核心功能**: 从输入字符串 (`inString`) 中删除所有出现的指定子串 (`pattern`)。可以看作是 `replace(inString, pattern, "")` 的一个快捷方式。

- **示例**:

  ```java
  String noisyData = "Error-[123]-Data";
  String cleanData = StringUtils.delete(noisyData, "-[123]-");
  // cleanData 的值是 "ErrorData"
  
  String withSpaces = "a b c d";
  String noSpaces = StringUtils.delete(withSpaces, " ");
  // noSpaces 的值是 "abcd"
  ```



#### `deleteAny`

> **`deleteAny(String inString, String charsToDelete)`**

- **返回值**: `String`

- **核心功能**: 从输入字符串 (`inString`) 中删除 `charsToDelete` 字符串中包含的**任何单个字符**。它不是按子串删除，而是按字符集进行过滤。

- **示例**:

  ```java
  String phoneNumber = "(021)-1234-5678";
  // 删除所有括号和短横线
  String cleanNumber = StringUtils.deleteAny(phoneNumber, "()-");
  // cleanNumber 的值是 "02112345678"
  ```



#### `trimArrayElements`

> **`trimArrayElements(String[] array)`**

- **返回值**: `String[]`

- **核心功能**: 

  - 对一个字符串数组中的**每一个元素**调用 Java 原生的 `String.trim()` 方法（去除首尾空白），然后返回一个新的数组

    原数组不会被修改

- **示例**:

  ```java
  String[] data = {"  apple ", " banana", "orange  "};
  String[] trimmedData = StringUtils.trimArrayElements(data);
  // trimmedData 的值是 {"apple", "banana", "orange"}
  ```



### 3. 路径与文件名操作

#### `cleanPath`

> **`cleanPath(String path)`**

- **返回值**: `String`

- **核心功能**: 

  - 规范化路径字符串
  - 它会处理 `.`、`..` 等相对路径指示符，并统一使用正斜杠 `/` 作为分隔符，使得路径比较和处理更加一致和安全

- **示例**:

  ```java
  String originalPath = "C:\\projects\\my-app\\..\\src\\main\\java\\./com/example/MyClass.java";
  String cleanPath = StringUtils.cleanPath(originalPath);
  // cleanPath 的值是 "C:/projects/src/main/java/com/example/MyClass.java"
  ```



#### `pathEquals`

> **`pathEquals(String path1, String path2)`**

- **返回值**: `boolean`

- **核心功能**: 

  - 在内部先对两个路径执行 `cleanPath` 规范化操作，然后再比较它们是否相等

    这是比较路径时推荐使用的方法，可以避免因分隔符或相对路径写法不同导致的误判

- **示例**:

  ```java
  String path1 = "C:/projects/data.txt";
  String path2 = "C:/projects/libs/../data.txt";
  boolean areEqual = StringUtils.pathEquals(path1, path2);
  // areEqual 的值是 true
  ```



#### `getFilename`

> **`getFilename(String path)`**

- **返回值**: `String`

- **核心功能**: 从一个完整的路径字符串中提取出文件名部分（即最后一个斜杠之后的部分）

- **示例**:

  ```java
  String filePath = "/home/user/documents/report.docx";
  String fileName = StringUtils.getFilename(filePath);
  // fileName 的值是 "report.docx"
  
  String windowsPath = "C:\\Users\\Admin\\Desktop\\image.jpg";
  String windowsFileName = StringUtils.getFilename(windowsPath);
  // windowsFileName 的值是 "image.jpg"
  ```



#### `getFilenameExtension`

> **`getFilenameExtension(String path)`**

- **返回值**: `String`

- **核心功能**: 从路径中提取文件的扩展名（不包含 `.`）。如果文件没有扩展名，则返回 `null`

- **示例**:

  ```java
  String path1 = "document.pdf";
  String ext1 = StringUtils.getFilenameExtension(path1);
  // ext1 的值是 "pdf"
  
  String path2 = "/home/user/archive.tar.gz";
  String ext2 = StringUtils.getFilenameExtension(path2);
  // ext2 的值是 "gz"
  
  String path3 = "README";
  String ext3 = StringUtils.getFilenameExtension(path3);
  // ext3 的值是 null
  ```



#### `stripFilenameExtension`

> **`stripFilenameExtension(String path)`**

- **返回值**: `String`

- **核心功能**: 移除路径中的文件扩展名部分，返回剩余的路径和主文件名

- **示例**:

  ```java
  String filePath = "/home/user/documents/report.docx";
  String strippedPath = StringUtils.stripFilenameExtension(filePath);
  // strippedPath 的值是 "/home/user/documents/report"
  ```



### 4. 字符串与集合/数组转换

#### `tokenizeToStringArray`

> **`tokenizeToStringArray(String str, String delimiters, boolean trimTokens, boolean ignoreEmptyTokens)`**

- **返回值**: `String[]`

- **核心功能**: 

  - 使用 `delimiters` 字符串中的**每一个字符**作为独立的分隔符来切分字符串

    它提供了强大的控制选项：`trimTokens`决定是否对切分出的每个元素执行`trim()`，`ignoreEmptyTokens`决定是否忽略因连续分隔符或`trim`后产生的空字符串

- **示例**:

  ```java
  String data = "apple,banana; cherry,,orange";
  // 使用,和;作为分隔符，去除首尾空格，并忽略空元素
  String[] result = StringUtils.tokenizeToStringArray(data, ",;", true, true);
  // result 的值是 {"apple", "banana", "cherry", "orange"}
  ```



#### `delimitedListToStringArray`

> **`delimitedListToStringArray(String str, String delimiter)`**

- **返回值**: `String[]`

- **核心功能**: 

  - 将一个字符串按照**完整的分隔符字符串** (`delimiter`) 切分成数组
  - 这与 `tokenizeToStringArray` 不同，后者将分隔符字符串中的每个字符都视为分隔符

- **示例**:

  ```java
  String record = "item1||item2||item3";
  // 使用 "||" 作为完整的分隔符
  String[] result = StringUtils.delimitedListToStringArray(record, "||");
  // result 的值是 {"item1", "item2", "item3"}
  ```



#### `commaDelimitedListToStringArray`

> **`commaDelimitedListToStringArray(String str)`**

- **返回值**: `String[]`

- **核心功能**:  `delimitedListToStringArray(str, ",")` 的快捷方式，专门用于快速处理逗号分隔的字符串

- **示例**:

  ```java
  String csvLine = "first, second ,third";
  String[] items = StringUtils.commaDelimitedListToStringArray(csvLine);
  // items 的值是 {"first", " second ", "third"} (注意：不会自动trim)
  ```



#### `commaDelimitedListToSet`

> **`commaDelimitedListToSet(String str)`**

- **返回值**: `Set<String>`

- **核心功能**: 

  - 将逗号分隔的字符串转换为一个 `Set` 集合

    这在需要自动去重的场景下非常有用，并且能保持元素的原有顺序（使用`LinkedHashSet`）

- **示例**:

  ```java
  String tags = "java,spring,java,docker";
  Set<String> tagSet = StringUtils.commaDelimitedListToSet(tags);
  // tagSet 的内容是 {"java", "spring", "docker"}
  ```



#### `collectionToDelimitedString`

> **`collectionToDelimitedString(Collection<?> coll, String delim, String prefix, String suffix)`**

- **返回值**: `String`

- **核心功能**: 

  - 将一个集合转换为使用指定分隔符连接的字符串，并且可以为每个元素添加统一的前缀和后缀

    这在构建SQL的IN子句或格式化日志时特别有用

- **示例**:

  ```java
  List<String> userIds = List.of("101", "102", "103");
  // 为SQL IN子句格式化
  String inClause = StringUtils.collectionToDelimitedString(userIds, ",", "'", "'");
  // inClause 的值是 "'101','102','103'"
  ```



#### `arrayToCommaDelimitedString`

> **`arrayToCommaDelimitedString(Object[] arr)`**

- **返回值**: `String`

- **核心功能**: 将一个数组的所有元素用逗号连接成一个字符串

- **示例**:

  ```java
  String[] frameworks = {"Spring", "Hibernate", "MyBatis"};
  String result = StringUtils.arrayToCommaDelimitedString(frameworks);
  // result 的值是 "Spring,Hibernate,MyBatis"
  ```



#### `removeDuplicateStrings`

> **`removeDuplicateStrings(String[] array)`**

- **返回值**: `String[]`

- **核心功能**: 移除字符串数组中的重复元素，并保持原始元素的首次出现顺序

- **示例**:

  ```java
  String[] arrayWithDups = {"a", "b", "c", "a", "b"};
  String[] uniqueArray = StringUtils.removeDuplicateStrings(arrayWithDups);
  // uniqueArray 的值是 {"a", "b", "c"}
  ```



### 5. 其他常用操作

#### `capitalize`

> **`capitalize(String str)`**

- **返回值**: `String`

- **核心功能**: 将字符串的第一个字符转换为大写，字符串的其余部分保持不变

- **示例**:

  ```java
  String greeting = "hello world";
  String capitalized = StringUtils.capitalize(greeting);
  // capitalized 的值是 "Hello world"
  ```



#### `uncapitalize`

> **`uncapitalize(String str)`**

- **返回值**: `String`

- **核心功能**: 将字符串的第一个字符转换为小写，字符串的其余部分保持不变。常用于将类名转换为符合JavaBeans规范的属性名

- **示例**:

  ```java
  String className = "PersonService";
  String propertyName = StringUtils.uncapitalize(className);
  // propertyName 的值是 "personService"
  ```



#### `unqualify`

> **`unqualify(String qualifiedName)`**

- **返回值**: `String`

- **核心功能**: 获取一个由点 (`.`) 分隔的限定名中的最后一部分。最常见的用途是从类的完全限定名中提取出简单的类名

- **示例**:

  ```java
  String fullName = "java.util.ArrayList";
  String simpleName = StringUtils.unqualify(fullName);
  // simpleName 的值是 "ArrayList"
  ```



#### `quote`

> **`quote(String str)`**

- **返回值**: `String`

- **核心功能**: 给指定的字符串前后加上单引号 (`'`)

- **示例**:

  ```java
  String value = "pending";
  String quotedValue = StringUtils.quote(value);
  // quotedValue 的值是 "'pending'"
  ```



#### `quoteIfString`

> **`quoteIfString(Object obj)`**

- **返回值**: `Object`

- **核心功能**: 

  - 判断传入的对象是否是一个 `String`

    如果是，则为其加上单引号并返回一个新的`String`；

    如果不是，则原样返回该对象

- **示例**:

  ```java
  Object val1 = "success";
  Object val2 = 100;
  
  Object result1 = StringUtils.quoteIfString(val1);
  // result1 的值是 "'success'" (类型为String)
  
  Object result2 = StringUtils.quoteIfString(val2);
  // result2 的值是 100 (类型为Integer)
  ```



# `ObjectUtils` **对象操作**

## 基本概念

- 核心设计目的是**以“null 安全” (null-safe) 的方式处理 Java 对象**



## 常用方法整理

### 数组与元素计算

#### `addObjectToArray` (末尾添加)

> **`static <A, O extends A> A[] addObjectToArray(A[] array, O obj)`**

- **返回值**: `A[]` (新数组)

- **核心功能**:

  - 将给定的对象 `obj` 追加到给定的数组 `array` 的末尾
  - 总是返回一个**新**的数组，包含了原数组的所有元素以及新添加的元素（因为Java数组长度不可变）
  - 如果输入的 `array` 为 `null`，它会创建一个只包含 `obj` 的新数组

- **示例**:

  ```java
  String[] original = {"a", "b"};
  String newItem = "c";
  
  String[] result = ObjectUtils.addObjectToArray(original, newItem);
  // result 值为 {"a", "b", "c"}
  
  String[] resultFromNull = ObjectUtils.addObjectToArray(null, "first");
  // resultFromNull 值为 {"first"}
  ```



#### `addObjectToArray` (指定位置添加)

> **`static <A, O extends A> A[] addObjectToArray(A[] array, O obj, int position)`**

- **返回值**: `A[]` (新数组)

- **核心功能**:

  - 将给定的对象 `obj` 添加到数组 `array` 的指定 `position` 位置
  - 同样返回一个**新**的数组
  - 原数组中从 `position` 开始的元素会向后移动一位
  - 如果输入的 `array` 为 `null`，它会创建一个只包含 `obj` 的新数组（此时 `position` 参数被忽略）

- **示例**:

  ```java
  String[] original = {"a", "c"};
  String newItem = "b";
  
  String[] result = ObjectUtils.addObjectToArray(original, newItem, 1);
  // result 值为 {"a", "b", "c"}
  ```



#### `toObjectArray`

> **`static Object[] toObjectArray(Object source)`**

- **返回值**: `Object[]`

- **核心功能**:

  - 将一个给定的 `source` 对象转换为 `Object[]` 数组
  - 核心用途是**将原始类型数组 (primitive array) 转换为其对应的包装类型数组**（例如 `int[]` 转为 `Integer[]`）
  - 如果 `source` 本身就是 `Object[]`，则直接返回（类型强转）
  - 如果 `source` 是 `null`，返回一个空的 `Object[]` (长度为0)
  - 如果 `source` 不是一个数组，则抛出 `IllegalArgumentException`

- **示例**:

  ```java
  int[] primitiveArray = {1, 2, 3};
  
  Object[] result = ObjectUtils.toObjectArray(primitiveArray);
  // result 是一个 Integer[] 数组，值为 {1, 2, 3}
  
  Object[] emptyResult = ObjectUtils.toObjectArray(null);
  // emptyResult 是一个长度为 0 的 Object[]
  ```



#### `containsElement`

> **`static boolean containsElement(Object[] array, Object element)`**

- **返回值**: `boolean`

- **核心功能**:

  - 以 "null 安全" 的方式检查给定的 `array` (对象数组) 是否包含指定的 `element`
  - 如果 `array` 为 `null`，直接返回 `false`
  - 遍历数组时，使用 `nullSafeEquals` 进行比较，因此可以正确处理数组中包含 `null` 元素或查找 `null` 元素的情况

- **示例**:

  ```java
  String[] array = {"apple", "banana", null};
  
  boolean hasBanana = ObjectUtils.containsElement(array, "banana");
  // hasBanana 为 true
  
  boolean hasCherry = ObjectUtils.containsElement(array, "cherry");
  // hasCherry 为 false
  
  boolean hasNull = ObjectUtils.containsElement(array, null);
  // hasNull 为 true
  
  boolean result4 = ObjectUtils.containsElement(null, "a");
  // result4 为 false
  ```



#### `isArray`

> **`static boolean isArray(Object obj)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断给定的 `obj` 是否是一个数组
  - 只要是数组类型（包括**对象数组**如 `String[]` 和**原始类型数组**如 `int[]`）都返回 `true`
  - 如果 `obj` 为 `null`，则返回 `false`

- **示例**:

  ```java
  String[] stringArray = {"a", "b"};
  int[] intArray = {1, 2};
  String notAnArray = "hello";
  
  boolean result1 = ObjectUtils.isArray(stringArray);
  // result1 为 true
  
  boolean result2 = ObjectUtils.isArray(intArray);
  // result2 为 true
  
  boolean result3 = ObjectUtils.isArray(notAnArray);
  // result3 为 false
  
  boolean result4 = ObjectUtils.isArray(null);
  // result4 为 false
  ```



#### `isEmpty` (数组)

> **`static boolean isEmpty(Object[] array)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断给定的**数组** `array` 是否为空
  - “空”的定义是：`array` 为 `null` **或者** `array.length == 0`
  - **注意**: 这是一个重载方法。另有一个 `isEmpty(Object obj)` 功能更广（能判断集合、Map、CharSequence等），这个方法是专用于数组的

- **示例**:

  ```java
  String[] populatedArray = {"a", "b"};
  String[] emptyArray = {};
  String[] nullArray = null;
  
  boolean result1 = ObjectUtils.isEmpty(populatedArray);
  // result1 为 false
  
  boolean result2 = ObjectUtils.isEmpty(emptyArray);
  // result2 为 true
  
  boolean result3 = ObjectUtils.isEmpty(nullArray);
  // result3 为 true
  ```



### 判空

#### `isEmpty`

> **`static boolean isEmpty(Object obj)`**

- **返回值**: `boolean`

- **核心功能**:

  - **（最常用的方法之一）** 判断给定的 `obj` 是否为“空”
  - 这是一个复合判断，满足以下**任意一个**条件即返回 `true`:
    - `obj` == `null`
    - `obj` 是 `Optional` 类型, 且 `Optional.empty()`
    - `obj` 是 `CharSequence` (如 `String`), 且长度为 0
    - `obj` 是 `Collection` (如 `List`), 且 `collection.isEmpty()`
    - `obj` 是 `Map`, 且 `map.isEmpty()`
    - `obj` 是**数组** (Array), 且长度为 0
  - 如果 `obj` 是其他任何非 `null` 对象, 则返回 `false`

- **示例**:

  ```java
  // Null
  ObjectUtils.isEmpty(null); // true
  
  // String
  ObjectUtils.isEmpty(""); // true
  ObjectUtils.isEmpty(" "); // false
  
  // Collection
  ObjectUtils.isEmpty(new ArrayList<>()); // true
  
  // Map
  ObjectUtils.isEmpty(new HashMap<>()); // true
  
  // Array
  ObjectUtils.isEmpty(new String[0]); // true
  
  // Optional
  ObjectUtils.isEmpty(Optional.empty()); // true
  ObjectUtils.isEmpty(Optional.of("a")); // false
  
  // Other objects
  ObjectUtils.isEmpty(new Object()); // false
  ObjectUtils.isEmpty(10); // false
  ```



#### `unwrapOptional`

> **`static Object unwrapOptional(Object obj)`**

- **返回值**: `Object`

- **核心功能**:

  - 拆开（解包）一个可能是 `Optional` 类型的对象
    - 如果 `obj` 是一个**非空**的 `Optional` (如 `Optional.of("value")`)，则返回其内部包含的值 ( `"value"` )
    - 如果 `obj` 是一个**空**的 `Optional` ( `Optional.empty()` )，则返回 `null`
    - 如果 `obj` **不是 `Optional` 类型** (比如就是一个 `String` 或 `Integer`)，则原样返回 `obj`
    - 如果 `obj` 是 `null`，则返回 `null`

- **示例**:

  ```java
  Object val1 = Optional.of("test");
  Object val2 = Optional.empty();
  Object val3 = "I am not an Optional";
  Object val4 = null;
  
  Object result1 = ObjectUtils.unwrapOptional(val1);
  // result1 的值是 "test" (类型为String)
  
  Object result2 = ObjectUtils.unwrapOptional(val2);
  // result2 的值是 null
  
  Object result3 = ObjectUtils.unwrapOptional(val3);
  // result3 的值是 "I am not an Optional" (类型为String)
  
  Object result4 = ObjectUtils.unwrapOptional(val4);
  // result4 的值是 null
  ```



### 哈希/相等

#### `nullSafeEquals`

> **`static boolean nullSafeEquals(Object o1, Object o2)`**

- **返回值**: `boolean`

- **核心功能**:

  - **（最常用的方法之一）** 以 "null 安全" 的方式比较两个对象 `o1` 和 `o2` 是否相等。
  - **逻辑**:
    1. 如果 `o1 == o2` (指向同一对象)，返回 `true`。
    2. 如果 `o1` 或 `o2` 中**有且仅有**一个为 `null`，返回 `false`。
    3. （此时两者都不为null）
    4. 如果 `o1` 是数组，则调用 `java.util.Arrays.equals` 比较数组内容。
    5. 否则，调用 `o1.equals(o2)`。

- **示例**:

  ```java
  String s1 = "hello";
  String s2 = new String("hello");
  String s3 = null;
  
  // 两个 null 相等
  ObjectUtils.nullSafeEquals(null, null); // true
  
  // null 与非 null 不相等
  ObjectUtils.nullSafeEquals(s1, s3); // false
  ObjectUtils.nullSafeEquals(s3, s1); // false
  
  // 两个非 null 对象
  ObjectUtils.nullSafeEquals(s1, s2); // true
  
  // 数组比较 (这是它强于 Objects.equals 的地方)
  int[] a1 = {1, 2};
  int[] a2 = {1, 2};
  ObjectUtils.nullSafeEquals(a1, a2); // true (会比较数组内容)
  ```



#### `nullSafeHashCode`

> **`static int nullSafeHashCode(Object obj)`**

- **返回值**: `int`

- **核心功能**:

  - 以 "null 安全" 的方式获取 `obj` 的哈希码。
  - **逻辑**:
    1. 如果 `obj` 为 `null`，返回 `0`。
    2. 如果 `obj` 是数组，则调用 `java.util.Arrays.hashCode` (会基于内容计算哈希码)。
    3. 否则，返回 `obj.hashCode()`。

- **示例**:

  ```java
  String s1 = "hello";
  String s2 = null;
  int[] arr = {1, 2};
  
  int hash1 = ObjectUtils.nullSafeHashCode(s1);
  // hash1 为 "hello" 的哈希码
  
  int hash2 = ObjectUtils.nullSafeHashCode(s2);
  // hash2 为 0
  
  int hash3 = ObjectUtils.nullSafeHashCode(arr);
  // hash3 为数组 {1, 2} 基于内容的哈希码
  ```



#### `nullSafeHash`

> **`static int nullSafeHash(Object... elements)`**

- **返回值**: `int`

- **核心功能**:

  - 计算多个元素（可变参数 `elements`）组合在一起的哈希码。
  - 类似于 `java.util.Objects.hash(Object...)`，但功能更强。
  - 它会委托 `nullSafeHashCode(Object)` 挨个处理 `elements` 中的每一个元素。
  - **核心优势**: 它可以正确处理 `elements` 中**某个元素本身就是数组**的情况，而 `Objects.hash()` 会把数组当作一个普通对象处理，导致哈希码不一致。

- **示例**:

  ```java
  String name = "user";
  int[] permissions = {1, 3, 5};
  
  // 推荐使用 ObjectUtils.nullSafeHash
  int hash1 = ObjectUtils.nullSafeHash(name, permissions);
  
  // 对比 java.util.Objects.hash
  int hash2 = java.util.Objects.hash(name, permissions);
  
  // 假设 permissions 换了一个实例，但内容相同
  int[] permissionsB = {1, 3, 5};
  int hash3 = ObjectUtils.nullSafeHash(name, permissionsB);
  int hash4 = java.util.Objects.hash(name, permissionsB);
  
  // hash1 == hash3 (true) - 因为 ObjectUtils 会计算数组内容
  // hash2 == hash4 (false) - 因为 Objects.hash 计算的是数组对象的引用地址
  ```



### 字符串表示/标识

#### `nullSafeToString`

> **`static String nullSafeToString(Object obj)`**

- **返回值**: `String`

- **核心功能**:

  - **（最常用的方法之一）** 以 "null 安全" 的方式获取对象的字符串表示。
  - **逻辑**:
    1. 如果 `obj` 为 `null`，返回字符串 `"null"`
    2. 如果 `obj` 是一个**数组**（无论是对象数组 `Object[]` 还是原始类型数组如 `int[]`, `boolean[]` 等），它会自动调用对应的重载方法，将其内容转换为字符串，如 `"{1, 2, 3}"`
    3. 否则，返回 `obj.toString()`

- **示例**:

  ```java
  String s = "hello";
  int[] arr = {1, 2, 3};
  
  String result1 = ObjectUtils.nullSafeToString(s);
  // result1 的值是 "hello"
  
  String result2 = ObjectUtils.nullSafeToString(null);
  // result2 的值是 "null" (字符串)
  
  String result3 = ObjectUtils.nullSafeToString(arr);
  // result3 的值是 "{1, 2, 3}"
  ```



#### `getDisplayString`

> **`static String getDisplayString(Object obj)`**

- **返回值**: `String`

- **核心功能**:

  - 获取一个用于“显示”的字符串，它与 `nullSafeToString` 的主要区别在于如何处理 `null`
  - **逻辑**:
    1. 如果 `obj` 为 `null`，返回一个**空字符串 `""`**
    2. 否则，返回 `obj.toString()`
  - **注意**: 此方法**不会**像 `nullSafeToString` 那样特殊处理数组。如果传入数组，它将返回数组默认的 `toString()` 结果 (例如 `[I@...`)

- **示例**:

  ```java
  String s = "hello";
  
  String result1 = ObjectUtils.getDisplayString(s);
  // result1 的值是 "hello"
  
  String result2 = ObjectUtils.getDisplayString(null);
  // result2 的值是 "" (空字符串)
  ```



#### `nullSafeConciseToString`

> **`static String nullSafeConciseToString(Object obj)`**

- **返回值**: `String`

- **核心功能**:

  - 获取一个“简洁”的字符串表示，主要用于日志和调试，避免打印出大型集合或数组的全部内容
  - **逻辑**:
    - `null` -> `"null"`
    - `Optional.empty()` -> `"Optional.empty"`
    - 空数组 -> `"{}"`
    - 非空数组/Map -> `"{...}"` (只显示省略号)
    - Collection -> `"[...]"` (只显示省略号)
    - 其他 -> `obj.toString()`

- **示例**:

  ```java
  List<String> list = List.of("a", "b", "c");
  int[] arr = {1, 2, 3, 4, 5};
  
  String result1 = ObjectUtils.nullSafeConciseToString(list);
  // result1 的值是 "[...]"
  
  String result2 = ObjectUtils.nullSafeConciseToString(arr);
  // result2 的值是 "{...}"
  
  String result3 = ObjectUtils.nullSafeConciseToString("hello");
  // result3 的值是 "hello"
  ```



#### `identityToString`

> **`static String identityToString(Object obj)`**

- **返回值**: `String`

- **核心功能**:

  - 安全地获取对象的“身份”字符串，格式为 `ClassName@IdentityHashCode`
  - 这是 `Object.toString()` 的默认行为，但 `ObjectUtils` 提供了 null 安全的实现
  - **逻辑**:
    1. 如果 `obj` 为 `null`，返回一个**空字符串 `""`**
    2. 否则，返回 `obj.getClass().getName() + "@" + getIdentityHexString(obj)`

- **示例**:

  ```java
  Object myObject = new Object();
  
  String result1 = ObjectUtils.identityToString(myObject);
  // result1 的值类似于 "java.lang.Object@1f32e575"
  
  String result2 = ObjectUtils.identityToString(null);
  // result2 的值是 "" (空字符串)
  ```



#### `nullSafeClassName`

> **`static String nullSafeClassName(Object obj)`**

- **返回值**: `String`

- **核心功能**:

  - 安全地获取对象的类名
  - **逻辑**:
    1. 如果 `obj` 为 `null`，返回字符串 `"null"`
    2. 如果 `obj` 本身就是一个 `Class` 对象，返回 `obj.getName()`
    3. 否则，返回 `obj.getClass().getName()`

- **示例**:

  ```java
  String s = "hello";
  
  String result1 = ObjectUtils.nullSafeClassName(s);
  // result1 的值是 "java.lang.String"
  
  String result2 = ObjectUtils.nullSafeClassName(null);
  // result2 的值是 "null" (字符串)
  
  String result3 = ObjectUtils.nullSafeClassName(String.class);
  // result3 的值是 "java.lang.String"
  ```



#### `getIdentityHexString`

> **`static String getIdentityHexString(Object obj)`**

- **返回值**: `String`

- **核心功能**:

  - 获取对象“身份哈希码”的十六进制字符串
  - 这等同于 `System.identityHashCode(obj)` 并将其转换为十六进制
  - （`identityToString` 方法内部就调用了此方法）

- **示例**:

  ```java
  Object myObject = new Object();
  
  String result1 = ObjectUtils.getIdentityHexString(myObject);
  // result1 的值类似于 "1f32e575" (没有类名和@)
  ```



### 枚举工具

#### `caseInsensitiveValueOf`

> **`static <E extends Enum<?>> E caseInsensitiveValueOf(E[] enumValues, String constant)`**

- **返回值**: `E` (枚举实例)

- **核心功能**:

  - `Enum.valueOf()` 的一个**忽略大小写**的安全替代方案
  - 它会遍历 `enumValues` 数组 (通常是 `YourEnum.values()`)，查找与 `constant` 字符串名称相匹配（忽略大小写）的枚举实例

- **注意**:

  - 如果找不到匹配项，它会抛出 `IllegalArgumentException`
  - 推荐**总是**先使用 `containsConstant()` 进行检查，以避免异常

- **示例**:

  ```java
  // 假设有枚举: enum Day { MONDAY, TUESDAY }
  
  String input = "monday";
  
  // 确保安全(推荐做法)
  if (ObjectUtils.containsConstant(Day.values(), input)) {
      Day day = ObjectUtils.caseInsensitiveValueOf(Day.values(), input);
      // day 的值是 Day.MONDAY
  }
  
  // 不安全的做法 (如果 "mon" 不存在，会抛出异常)
  // Day day = ObjectUtils.caseInsensitiveValueOf(Day.values(), "mon");
  ```



#### `containsConstant` (重载)

> **`static <E extends Enum<?>> boolean containsConstant(E[] enumValues, String constant, boolean caseSensitive)`**
>
> **`static <E extends Enum<?>> boolean containsConstant(E[] enumValues, String constant)`**

- **返回值**: `boolean`

- **核心功能**:

  - 检查给定的 `enumValues` 数组 (通常是 `YourEnum.values()`) 是否包含一个指定名称的常量

- **版本区别**:

  - `containsConstant(..., constant)`: **默认忽略大小写**进行比较
  - `containsConstant(..., constant, caseSensitive)`:
    - `caseSensitive = true`: 进行严格区分大小写的比较
    - `caseSensitive = false`: 进行忽略大小写的比较 (同上一个方法)

- **示例**:

  ```java
  // 假设有枚举: enum Day { MONDAY, TUESDAY }
  
  // 1. 默认 (忽略大小写)
  boolean r1 = ObjectUtils.containsConstant(Day.values(), "monday");
  // r1 的值是 true
  
  boolean r2 = ObjectUtils.containsConstant(Day.values(), "WEDNESDAY");
  // r2 的值是 false
  
  // 2. 指定大小写敏感
  boolean r3 = ObjectUtils.containsConstant(Day.values(), "monday", true);
  // r3 的值是 false (因为 "monday" != "MONDAY")
  
  boolean r4 = ObjectUtils.containsConstant(Day.values(), "MONDAY", true);
  // r4 的值是 true
  ```



### 异常工具

#### `isCheckedException`

> **`static boolean isCheckedException(Throwable ex)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断一个 `Throwable` (异常或错误) 是否是一个 "Checked Exception" (受检异常)
  - **逻辑**:
    - 如果 `ex` 是 `RuntimeException` (运行时异常) 或其子类，返回 `false`
    - 如果 `ex` 是 `Error` (错误) 或其子类，返回 `false`
    - 其他所有情况 (即继承自 `Exception` 但不继承自 `RuntimeException` 的)，返回 `true`

- **示例**:

  ```java
  IOException ioEx = new IOException("File not found");
  NullPointerException npe = new NullPointerException();
  Error error = new StackOverflowError();
  
  boolean result1 = ObjectUtils.isCheckedException(ioEx);
  // result1 的值是 true (IOException 是受检异常)
  
  boolean result2 = ObjectUtils.isCheckedException(npe);
  // result2 的值是 false (NullPointerException 是 RuntimeException)
  
  boolean result3 = ObjectUtils.isCheckedException(error);
  // result3 的值是 false (Error 不是异常)
  ```



#### `isCompatibleWithThrowsClause`

> **`static boolean isCompatibleWithThrowsClause(Throwable ex, Class<?>... declaredExceptions)`**

- **返回值**: `boolean`

- **核心功能**:

  - 检查一个抛出的异常 `ex` 是否与方法签名中 `throws` 子句声明的异常类型 `declaredExceptions` 兼容
  - **逻辑**:
    - 遍历 `declaredExceptions` 列表
    - 如果 `ex` 是 `declaredExceptions` 中**任意一个类**的实例，或者是其**子类**的实例，则返回 `true`
    - 如果 `declaredExceptions` 为空，或 `ex` 与所有声明的类型都不兼容，则返回 `false`

- **示例**:

  ```java
  // 假设一个方法签名: 
  // void myMethod() throws IOException, SQLException { ... }
  Class<?>[] declared = { IOException.class, java.sql.SQLException.class };
  
  // 1. 抛出 FileNotFoundException (是 IOException 的子类)
  Throwable ex1 = new java.io.FileNotFoundException();
  boolean r1 = ObjectUtils.isCompatibleWithThrowsClause(ex1, declared);
  // r1 的值是 true
  
  // 2. 抛出 NullPointerException (不兼容)
  Throwable ex2 = new NullPointerException();
  boolean r2 = ObjectUtils.isCompatibleWithThrowsClause(ex2, declared);
  // r2 的值是 false
  
  // 3. 抛出 SQLException (精确匹配)
  Throwable ex3 = new java.sql.SQLException();
  boolean r3 = ObjectUtils.isCompatibleWithThrowsClause(ex3, declared);
  // r3 的值是 true
  ```



# `BeanUtils` **Bean操作**

## 基本概念

- `BeanUtils` 是 Spring 框架提供的一个**静态工具类**，位于 `org.springframework.beans` 包下

  它的主要作用是**方便地操作 Java Bean (POJO) 的属性**

- 它最核心、最广为人知的功能就是：**对象属性复制**



## 常用方法整理

### 拷贝复制

#### `copyProperties` (基础复制)

> **`public static void copyProperties(Object source, Object target)`**

- **返回值**: `void`

- **核心功能**:

  - 将源对象 (`source`) 的属性值复制到目标对象 (`target`)
  - 复制是基于 `source` 对象的 `get` 方法和 `target` 对象的 `set` 方法
  - 只有当 `source` 和 `target` 拥有**相同属性名**，并且**数据类型兼容**时，才会发生复制
  - 这是一个**浅拷贝**（Shallow Copy）。如果属性是对象（如 `List` 或自定义类），复制的是引用（内存地址），而不是对象本身
  - **[重要]** 此方法会**复制 `null` 值**。如果 `source` 中的某个属性为 `null`，它会将 `target` 中对应的属性也设置为 `null`

- **示例**:

  ```java
  // 假设 UserDTO 和 User 都有 username 和 email 属性
  UserDTO dto = new UserDTO();
  dto.setUsername("testuser");
  dto.setEmail("test@example.com");
  
  User user = new User();
  // 此时 user 的 username 和 email 都是 null
  
  // 执行复制
  BeanUtils.copyProperties(dto, user);
  
  // 复制后:
  // user.getUsername() 的值是 "testuser"
  // user.getEmail() 的值是 "test@example.com"
  ```



#### `copyProperties` (忽略指定属性)

> **`public static void copyProperties(Object source, Object target, String... ignoreProperties)`**

- **返回值**: `void`

- **核心功能**:

  - `copyProperties` 的增强版，提供了一个“黑名单” (`ignoreProperties`)
  - `ignoreProperties` 是一个字符串数组，用于指定**不需要**从 `source` 复制到 `target` 的属性名
  - 这是最常用的重载方法，主要用于**防止 `null` 值覆盖**或在更新操作中**忽略 `id`** 这类不应被修改的字段

- **示例**: (用于防止 `null` 值覆盖)

  ```java
  // 场景：只想更新用户名，不想更新邮箱
  UserDTO dto = new UserDTO();
  dto.setUsername("newUser");
  dto.setEmail(null); // DTO的Email为null
  
  User user = new User();
  user.setUsername("oldUser");
  user.setEmail("old@example.com"); // 数据库中查出的用户，Email有值
  
  // 执行复制，但忽略 "email" 属性
  BeanUtils.copyProperties(dto, user, "email");
  
  // 复制后:
  // user.getUsername() 的值是 "newUser"
  // user.getEmail() 的值是 "old@example.com" (因为 "email" 被忽略，未被 null 覆盖)
  ```



#### `copyProperties` (限制可编辑属性)

> **`public static void copyProperties(Object source, Object target, Class<?> editable)`**

- **返回值**: `void`

- **核心功能**:

  - 这是一个不常用的重载方法，提供了一个“白名单” (`editable`)
  - `editable` 参数通常是一个**接口**或**父类**
  - 此方法**仅**复制那些在 `editable` 类（或接口）中定义了 `set` 方法的属性
  - 它用于严格限制哪些属性可以被更新，通常用于特定的业务场景

- **示例**:

  ```java
  // 1. 定义一个接口，作为“白名单”
  public interface UpdatableUser {
      void setUsername(String username);
      // 注意：这个接口没有 setEmail 方法
  }
  
  // 2. 准备数据
  UserDTO dto = new UserDTO();
  dto.setUsername("newUsername");
  dto.setEmail("new@example.com"); // DTO中有Email
  
  User user = new User();
  user.setEmail("old@example.com"); // User中也有Email
  
  // 3. 执行复制，白名单为 UpdatableUser.class
  BeanUtils.copyProperties(dto, user, UpdatableUser.class);
  
  // 复制后:
  // user.getUsername() 的值是 "newUsername" (因为 setUsername 在白名单中)
  // user.getEmail() 的值是 "old@example.com" (因为 setEmail 不在白名单中，被忽略)
  ```



### 实例化

#### `instantiateClass` (使用默认构造函数)

> **`public static <T> T instantiateClass(Class<T> clazz)`**

- **返回值**: `T` (泛型，即 `clazz` 对应的类型的实例)

- **核心功能**:

  - 实例化一个给定的类 (`clazz`)
  - 此方法会**自动查找并使用该类的默认（无参）构造函数**来创建对象
  - 相比于 `clazz.newInstance()` (已废弃) 或 `clazz.getDeclaredConstructor().newInstance()`，`instantiateClass` 提供了更好的封装和异常处理：
    1. 它会尝试访问**非 public** (例如 `private`) 的无参构造函数
    2. 它将反射相关的受检异常 (Checked Exceptions, 如 `NoSuchMethodException`) 转换为 Spring 的运行时异常 (Unchecked Exceptions, 如 `BeanInstantiationException`)，使代码更简洁

- **示例**:

  ```java
  // 假设 MyService 有一个 public 或 private 的无参构造函数
  // public class MyService {
  //     private MyService() { ... } // 即使是 private 也能实例化
  // }
  
  try {
      Class<MyService> clazz = MyService.class;
  
      // 使用 Spring BeanUtils 实例化
      MyService serviceInstance = BeanUtils.instantiateClass(clazz);
  
      // serviceInstance 是 MyService 的一个新实例
  
  } catch (BeanInstantiationException e) {
      // 如果 MyService 缺少无参构造函数，或实例化失败，则会抛出此异常
      System.err.println("实例化失败: " + e.getMessage());
  }
  ```

- **主要用途**: 当你确定一个类有默认构造函数时，用它来创建实例。



#### `instantiateClass` (使用指定构造函数)

> **`public static <T> T instantiateClass(Constructor<T> ctor, Object... args)`**

- **返回值**: `T` (泛型，即构造函数 `ctor` 对应类的实例)

- **核心功能**:

  - 通过一个**特定**的构造函数 (`ctor`) 和**对应**的参数 (`args`) 来实例化一个类。
  - 这是最灵活的实例化方式，允许你调用任意参数的构造函数。
  - 同样，它也会处理 `private` 的构造函数，并将受检异常转换为运行时异常。

- **示例**:

  ```java
  // 假设 MyRepository 有一个带参构造函数
  // public class MyRepository {
  //     private String dataSourceUrl;
  //
  //     public MyRepository(String dataSourceUrl) {
  //         this.dataSourceUrl = dataSourceUrl;
  //     }
  // }
  
  try {
      String url = "jdbc:mysql://localhost:3306/mydb";
  
      // 1. 获取指定参数类型的构造函数
      Constructor<MyRepository> ctor = MyRepository.class.getDeclaredConstructor(String.class);
  
      // 2. 使用该构造函数和参数来实例化
      MyRepository repoInstance = BeanUtils.instantiateClass(ctor, url);
  
      // repoInstance 是 MyRepository 的一个新实例
      // 并且其 dataSourceUrl 属性已被设置为 "jdbc:mysql://localhost:3306/mydb"
  
  } catch (NoSuchMethodException e) {
      System.err.println("未找到匹配的构造函数: " + e.getMessage());
  } catch (BeanInstantiationException e) {
      System.err.println("实例化失败: " + e.getMessage());
  }
  ```

- **主要用途**: 在依赖注入 (DI) 框架或需要精确控制实例化过程的场景中，用于调用带参构造函数



### 属性描述/定位

#### `getPropertyDescriptors`

> **`public static PropertyDescriptor[] getPropertyDescriptors(Class<?> beanClass)`**

- **返回值**: `PropertyDescriptor[]` (属性描述符数组)

- **核心功能**:

  - 获取一个类 (`beanClass`) 的**所有**属性描述符
  - 这是 Spring 对 Java 内省功能 (`java.beans.Introspector`) 的封装，并添加了**缓存**机制，因此性能优于直接调用 Java 的 API
  - 返回的数组包含了所有（`public`）具有 `get` 或 `set` 方法的属性
  - **[注意]** 返回结果中通常会包含一个名为 `"class"` 的属性（来自 `Object.getClass()`），在使用时可能需要过滤掉

- **示例**:

  ```java
  // 假设 User 类有 getUsername() 和 setAge(int age) 方法
  PropertyDescriptor[] pds = BeanUtils.getPropertyDescriptors(User.class);
  
  // 遍历所有属性
  for (PropertyDescriptor pd : pds) {
      if ("class".equals(pd.getName())) {
          continue; // 过滤掉 'class' 属性
      }
  
      System.out.println("属性名: " + pd.getName());
      System.out.println("  -> 读方法: " + pd.getReadMethod()); // (例如 getUsername)
      System.out.println("  -> 写方法: " + pd.getWriteMethod()); // (例如 setAge)
      System.out.println("  -> 属性类型: " + pd.getPropertyType());
  }
  // 输出示例：
  // 属性名: username
  //   -> 读方法: public java.lang.String com.example.User.getUsername()
  //   -> 写方法: null (如果只有get方法)
  //   -> 属性类型: class java.lang.String
  // 属性名: age
  //   -> 读方法: null (如果只有set方法)
  //   -> 写方法: public void com.example.User.setAge(int)
  //   -> 属性类型: int
  ```



#### `getPropertyDescriptor`

> **`public static PropertyDescriptor getPropertyDescriptor(Class<?> beanClass, String propertyName)`**

- **返回值**: `PropertyDescriptor` (单个属性描述符)

- **核心功能**:

  - 获取一个类 (`beanClass`) 中**指定名称** (`propertyName`) 的属性描述符
  - 这比 `getPropertyDescriptors` 更高效，因为它只查找并缓存你需要的单个属性
  - 如果 `beanClass` 中不存在名为 `propertyName` 的属性，此方法将返回 `null`（或在某些情况下抛出 `IntrospectionException`，Spring 会将其捕获并包装为 `FatalBeanException`）

- **示例**:

  ```java
  try {
      PropertyDescriptor pd = BeanUtils.getPropertyDescriptor(User.class, "username");
  
      if (pd != null) {
          System.out.println("找到了 'username' 属性");
          // 获取 'username' 属性的 getter 方法
          Method getter = pd.getReadMethod(); 
          // 获取 'username' 属性的 setter 方法
          Method setter = pd.getWriteMethod();
  
          System.out.println("Getter: " + getter.getName());
          System.out.println("Setter: " + setter.getName());
      } else {
          System.out.println("未找到 'username' 属性");
      }
  
  } catch (FatalBeanException e) {
      System.err.println("内省失败: " + e.getMessage());
  }
  ```



#### `findPropertyForMethod` (方法反查属性)

> **`public static PropertyDescriptor findPropertyForMethod(Method method)`**

- **返回值**: `PropertyDescriptor`

- **核心功能**:

  - 这是一个“反向查找”功能
  - 你提供一个 `Method` 对象（这个方法必须是一个 `get` 或 `set` 方法），它会找出该方法对应的属性描述符
  - 例如，如果你传入 `User.class.getMethod("getUsername")`，它会返回 "username" 属性的 `PropertyDescriptor`

- **示例**:

  ```java
  // 1. 先获取一个方法的引用
  Method method = User.class.getMethod("getUsername");
  
  // 2. 通过方法反向查找它属于哪个属性
  PropertyDescriptor pd = BeanUtils.findPropertyForMethod(method);
  
  // 3. pd 现在是 "username" 属性的描述符
  System.out.println("该方法对应的属性名: " + pd.getName()); // "username"
  
  // --- 示例2：使用 setter ---
  Method methodSet = User.class.getMethod("setAge", int.class);
  PropertyDescriptor pd2 = BeanUtils.findPropertyForMethod(methodSet);
  System.out.println("该方法对应的属性名: " + pd2.getName()); // "age"
  ```



#### `findPropertyForMethod` (带类上下文的反查)

> **`public static PropertyDescriptor findPropertyForMethod(Method method, Class<?> beanClass)`**

- **返回值**: `PropertyDescriptor`

- **核心功能**:

  - 与上一个 `findPropertyForMethod(method)` 功能相同，但提供了 `beanClass` 上下文

  - 在你知道 `method` 属于哪个 `beanClass` 的情况下，调用此方法可以更高效地利用 Spring 的缓存
  - 当你处理泛型或CGLIB代理类时，`method.getDeclaringClass()` 可能不是你期望的原始 Bean 类，此时显式传入 `beanClass` 会更准确

- **示例**:

  ```java
  Method method = User.class.getMethod("getUsername");
  
  // 显式提供 User.class 作为上下文，以便 Spring 查找缓存
  PropertyDescriptor pd = BeanUtils.findPropertyForMethod(method, User.class);
  
  System.out.println("该方法对应的属性名: " + pd.getName()); // "username"
  ```



### 简单类型判断

#### `isSimpleValueType`

> **`public static boolean isSimpleValueType(Class<?> type)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断一个类 (`type`) 是否为**“简单值类型”**
  - 这是**最严格**的检查。它主要用于判断一个类型是否可以被Spring的转换服务 (Conversion Service) 方便地处理（例如，从 `String` 转换而来）
  - **“简单值类型”** 包括:
    1. **基本类型** (Primitives): `int`, `boolean`, `char` 等
    2. **包装类型** (Wrappers): `Integer`, `Boolean`, `Character` 等
    3. `Enum` (枚举)
    4. `String` 或其他 `CharSequence`
    5. `Number` 的子类 (如 `BigDecimal`, `AtomicInteger`)
    6. `Date` 的子类 (如 `java.sql.Date`, `Timestamp`)
    7. `Temporal` (Java 8 时间 API)
    8. `UUID`
    9. `URI`, `URL`, `Locale`, `Class`

- **示例**:

  ```java
  // --- 返回 true ---
  BeanUtils.isSimpleValueType(int.class);       // true (基本类型)
  BeanUtils.isSimpleValueType(Integer.class);   // true (包装类型)
  BeanUtils.isSimpleValueType(String.class);    // true
  BeanUtils.isSimpleValueType(BigDecimal.class); // true (Number子类)
  BeanUtils.isSimpleValueType(Date.class);        // true
  BeanUtils.isSimpleValueType(DayOfWeek.class);   // true (任意 Enum)
  
  // --- 返回 false ---
  BeanUtils.isSimpleValueType(int[].class);      // false (数组不是简单 "值" 类型)
  BeanUtils.isSimpleValueType(List.class);       // false (Collection 不是)
  BeanUtils.isSimpleValueType(User.class);      // false (自定义 Bean)
  BeanUtils.isSimpleValueType(Object.class);    // false
  ```



#### `isSimpleProperty`

> **`public static boolean isSimpleProperty(Class<?> type)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断一个类 (`type`) 是否为**“简单属性”**
  - 根据官方文档 (Spring 6.2.12)，“简单属性”被定义为：
    1. 一个**“简单值类型”**（即 `isSimpleValueType(type)` 返回 `true`）
    2. 或一个**“简单值类型”的数组**
  - 此方法用于 Spring 内部（例如依赖检查）判断一个属性是否应该被视为“简单的”

- **示例**:

  ```java
  // --- 返回 true ---
  // (所有 isSimpleValueType 为 true 的)
  BeanUtils.isSimpleProperty(int.class);       // true
  BeanUtils.isSimpleProperty(String.class);    // true
  
  // (额外的 - 简单值类型的数组)
  BeanUtils.isSimpleProperty(int[].class);      // true 
  BeanUtils.isSimpleProperty(String[].class);   // true
  BeanUtils.isSimpleProperty(Date[].class);     // true
  
  // --- 返回 false ---
  BeanUtils.isSimpleProperty(User.class);      // false (自定义 Bean)
  BeanUtils.isSimpleProperty(User[].class);     // false (非 "简单值类型" 的数组)
  BeanUtils.isSimpleProperty(List.class);       // false (Collection)
  BeanUtils.isSimpleProperty(Map.class);        // false (Map)
  ```

- **与 `isSimpleValueType` 的关键区别**:

  - `isSimpleProperty` 额外**包含**了**简单值类型的数组** (如 `int[]`, `String[]`)
  - `isSimpleValueType` **不包含**任何数组



# `CollectionUtils` **集合操作**

## 基本概念

- 位于 `org.springframework.util` 包下，主要用于**简化和安全地处理Java中的集合（Collections）和Map**



## 常用方法整理

### 判空

#### `isEmpty` (Collection)

> **`public static boolean isEmpty(@Nullable Collection<?> collection)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断传入的 `Collection` 是否为 `null` 或者 `empty` (即 `size()` 为 0)。
  - 如果是 `null` 或 `empty`，则返回 `true`；
  - 否则返回 `false`。

- **示例**:

  ```java
  List<String> list1 = new ArrayList<>();
  List<String> list2 = null;
  List<String> list3 = List.of("apple", "banana");
  
  // result1 为 true
  boolean result1 = CollectionUtils.isEmpty(list1);
  
  // result2 为 true
  boolean result2 = CollectionUtils.isEmpty(list2);
  
  // result3 为 false
  boolean result3 = CollectionUtils.isEmpty(list3);
  ```



#### `isEmpty` (Map)

> **`public static boolean isEmpty(@Nullable Map<?, ?> map)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断传入的 `Map` 是否为 `null` 或者 `empty` (即 `size()` 为 0)。
  - 如果是 `null` 或 `empty`，则返回 `true`；
  - 否则返回 `false`。

- **示例**:

  ```java
  Map<String, Object> map1 = new HashMap<>();
  Map<String, Object> map2 = null;
  Map<String, Object> map3 = Map.of("key", "value");
  
  // result1 为 true
  boolean result1 = CollectionUtils.isEmpty(map1);
  
  // result2 为 true
  boolean result2 = CollectionUtils.isEmpty(map2);
  
  // result3 为 false
  boolean result3 = CollectionUtils.isEmpty(map3);
  ```



### 容器快速创建（按预期容量）

#### `newHashMap`

> **`public static <K, V> HashMap<K, V> newHashMap(int expectedSize)`**

- **返回值**: `HashMap<K, V>`

- **核心功能**:

  - 实例化一个新的 `HashMap`。
  - 其初始容量 (initial capacity) 经过计算，足以容纳 `expectedSize` 个元素，而无需立即进行 resize（扩容）或 rehash（重新哈希）操作。

- **示例**:

  ```java
  // 预期要存储100个元素
  int expectedSize = 100;
  
  // 创建一个HashMap，其初始容量被优化以容纳100个元素
  Map<String, Integer> map = CollectionUtils.newHashMap(expectedSize);
  
  // ... 后续可以添加元素，而不会立即触发扩容
  ```



#### `newLinkedHashMap`

> **`public static <K, V> LinkedHashMap<K, V> newLinkedHashMap(int expectedSize)`**

- **返回值**: `LinkedHashMap<K, V>`

- **核心功能**:

  - 实例化一个新的 `LinkedHashMap`。
  - 其初始容量经过计算，足以容纳 `expectedSize` 个元素，而无需立即进行 resize 或 rehash 操作。

- **示例**:

  ```java
  // 预期要存储50个元素
  int expectedSize = 50;
  
  // 创建一个LinkedHashMap，优化了初始容量以容纳50个元素
  Map<String, Integer> linkedMap = CollectionUtils.newLinkedHashMap(expectedSize);
  ```



#### `newHashSet`

> **`public static <E> HashSet<E> newHashSet(int expectedSize)`**

- **返回值**: `HashSet<E>`

- **核心功能**:

  - 实例化一个新的 `HashSet`。
  - 其初始容量经过计算，足以容纳 `expectedSize` 个元素，而无需立即进行 resize 或 rehash 操作。

- **示例**:

  ```java
  // 预期要存储75个元素
  int expectedSize = 75;
  
  // 创建一个HashSet，优化了初始容量以容纳75个元素
  Set<String> set = CollectionUtils.newHashSet(expectedSize);
  ```



#### `newLinkedHashSet`

> **`public static <E> LinkedHashSet<E> newLinkedHashSet(int expectedSize)`**

- **返回值**: `LinkedHashSet<E>`

- **核心功能**:

  - 实例化一个新的 `LinkedHashSet`。
  - 其初始容量经过计算，足以容纳 `expectedSize` 个元素，而无需立即进行 resize 或 rehash 操作。

- **示例**:

  ```java
  // 预期要存储25个元素
  int expectedSize = 25;
  
  // 创建一个LinkedHashSet，优化了初始容量以容纳25个元素
  Set<String> linkedSet = CollectionUtils.newLinkedHashSet(expectedSize);
  ```



### 元素访问 (Element Access)

#### `firstElement` (Set)

> **`@Nullable public static <T> T firstElement(@Nullable Set<T> set)`**

- **返回值**: `T` (可能为 `null`)

- **核心功能**:

  - 获取给定 `Set` 的第一个元素。
  - 如果 `Set` 是 `SortedSet`，则调用 `first()` 方法。
  - 否则，依赖迭代器 (Iterator) 获取第一个元素。
  - 如果 `Set` 为 `null` 或 `empty`，则返回 `null`。

- **示例**:

  ```java
  Set<String> linkedSet = new LinkedHashSet<>(List.of("apple", "banana"));
  Set<String> sortedSet = new TreeSet<>(List.of("cherry", "apple"));
  Set<String> emptySet = Collections.emptySet();
  
  // result1 的值是 "apple"
  String result1 = CollectionUtils.firstElement(linkedSet);
  
  // result2 的值是 "apple" (TreeSet 排序后)
  String result2 = CollectionUtils.firstElement(sortedSet);
  
  // result3 的值是 null
  String result3 = CollectionUtils.firstElement(emptySet);
  ```



#### `firstElement` (List)

> **`@Nullable public static <T> T firstElement(@Nullable List<T> list)`**

- **返回值**: `T` (可能为 `null`)

- **核心功能**:

  - 获取给定 `List` 的第一个元素 (即索引为 0 的元素)。
  - 如果 `List` 为 `null` 或 `empty`，则返回 `null`。

- **示例**:

  ```java
  List<String> list1 = List.of("apple", "banana");
  List<String> list2 = null;
  
  // result1 的值是 "apple"
  String result1 = CollectionUtils.firstElement(list1);
  
  // result2 的值是 null
  String result2 = CollectionUtils.firstElement(list2);
  ```



#### `lastElement` (Set)

> **`@Nullable public static <T> T lastElement(@Nullable Set<T> set)`**

- **返回值**: `T` (可能为 `null`)

- **核心功能**:

  - 获取给定 `Set` 的最后一个元素。
  - 如果 `Set` 是 `SortedSet`，则调用 `last()` 方法。
  - 否则，遍历所有元素来获取最后一个（假设是 `LinkedHashSet` 等有序 `Set`）。
  - 如果 `Set` 为 `null` 或 `empty`，则返回 `null`。

- **示例**:

  ```java
  Set<String> linkedSet = new LinkedHashSet<>(List.of("apple", "banana"));
  Set<String> sortedSet = new TreeSet<>(List.of("cherry", "apple"));
  
  // result1 的值是 "banana"
  String result1 = CollectionUtils.lastElement(linkedSet);
  
  // result2 的值是 "cherry" (TreeSet 排序后)
  String result2 = CollectionUtils.lastElement(sortedSet);
  ```



#### `lastElement` (List)

> **`@Nullable public static <T> T lastElement(@Nullable List<T> list)`**

- **返回值**: `T` (可能为 `null`)

- **核心功能**:

  - 获取给定 `List` 的最后一个元素 (即索引为 `size() - 1` 的元素)。
  - 如果 `List` 为 `null` 或 `empty`，则返回 `null`。

- **示例**:

  ```java
  List<String> list1 = List.of("apple", "banana", "cherry");
  List<String> list2 = new ArrayList<>();
  
  // result1 的值是 "cherry"
  String result1 = CollectionUtils.lastElement(list1);
  
  // result2 的值是 null
  String result2 = CollectionUtils.lastElement(list2);
  ```



### 包含/匹配与实例判断

#### `contains` (Iterator)

> **`public static boolean contains(@Nullable Iterator<?> iterator, Object element)`**

- **返回值**: `boolean`

- **核心功能**:

  - 检查给定的 `Iterator` (迭代器) 是否包含给定的 `element`。
  - 会遍历迭代器，使用 `Objects.equals()` 来比较元素。
  - 如果 `iterator` 为 `null`，返回 `false`。

- **示例**:

  ```java
  List<String> list = List.of("apple", "banana");
  Iterator<String> iterator = list.iterator();
  
  // result1 为 true
  boolean result1 = CollectionUtils.contains(iterator, "apple");
  
  // 注意：迭代器已被消耗，再次调用需要重新获取
  iterator = list.iterator(); 
  // result2 为 false
  boolean result2 = CollectionUtils.contains(iterator, "cherry");
  ```



#### `contains` (Enumeration)

> **`public static boolean contains(@Nullable Enumeration<?> enumeration, Object element)`**

- **返回值**: `boolean`

- **核心功能**:

  - 检查给定的 `Enumeration` (枚举) 是否包含给定的 `element`。
  - 会遍历 `Enumeration`，使用 `Objects.equals()` 来比较元素。
  - 如果 `enumeration` 为 `null`，返回 `false`。

- **示例**:

  ```java
  Vector<String> vector = new Vector<>(List.of("one", "two"));
  Enumeration<String> enumeration = vector.elements();
  
  // result1 为 true
  boolean result1 = CollectionUtils.contains(enumeration, "one");
  
  // 再次调用需要重新获取 Enumeration
  enumeration = vector.elements();
  // result2 为 false
  boolean result2 = CollectionUtils.contains(enumeration, "three");
  ```



#### `containsInstance`

> **`public static boolean containsInstance(@Nullable Collection<?> collection, Object element)`**

- **返回值**: `boolean`

- **核心功能**:

  - 检查给定的 `Collection` 是否包含给定的 `element` **实例**。
  - 比较时使用的是 `==` (引用相等)，而不是 `.equals()` (值相等)。
  - 如果 `collection` 为 `null`，返回 `false`。

- **示例**:

  ```java
  String s1 = "hello";
  String s2 = new String("hello");
  String s3 = s1;
  
  List<String> list = List.of(s1);
  
  // result1 为 true (s1 和 s1 是同一个实例)
  boolean result1 = CollectionUtils.containsInstance(list, s1);
  
  // result2 为 false (s2 和 s1 值相等，但不是同一个实例)
  boolean result2 = CollectionUtils.containsInstance(list, s2);
  
  // result3 为 true (s3 和 s1 指向同一个实例)
  boolean result3 = CollectionUtils.containsInstance(list, s3);
  ```



#### `containsAny`

> **`public static boolean containsAny(Collection<?> source, Collection<?> candidates)`**

- **返回值**: `boolean`

- **核心功能**:

  - 检查 `source` 集合中是否包含了 `candidates` 集合中的**任意一个**元素。
  - 如果 `source` 或 `candidates` 为 `null` 或 `empty`，返回 `false`。

- **示例**:

  ```java
  List<String> source = List.of("A", "B", "C");
  List<String> candidates1 = List.of("C", "D");
  List<String> candidates2 = List.of("X", "Y");
  
  // result1 为 true (因为 "C" 存在于 source 中)
  boolean result1 = CollectionUtils.containsAny(source, candidates1);
  
  // result2 为 false (因为 "X" 和 "Y" 都不存在于 source 中)
  boolean result2 = CollectionUtils.containsAny(source, candidates2);
  ```



#### `findFirstMatch`

> **`@Nullable public static <E> E findFirstMatch(Collection<?> source, Collection<E> candidates)`**

- **返回值**: `E` (可能为 `null`)

- **核心功能**:

  - 查找并返回 `candidates` 集合中**第一个**也存在于 `source` 集合中的元素。
  - 迭代顺序取决于 `candidates` 集合的实现。
  - 如果没有找到匹配项，或者任一集合为 `null` 或 `empty`，则返回 `null`。

- **示例**:

  ```java
  List<String> source = List.of("A", "B", "C");
  
  // 使用 LinkedHashSet 保证 "C" 在 "B" 之前被检查
  Set<String> candidates1 = new LinkedHashSet<>(List.of("C", "B", "X")); 
  List<String> candidates2 = List.of("X", "Y");
  
  // result1 的值是 "C" (因为 "C" 是 candidates1 中第一个匹配 source 的)
  String result1 = CollectionUtils.findFirstMatch(source, candidates1);
  
  // result2 的值是 null (没有匹配项)
  String result2 = CollectionUtils.findFirstMatch(source, candidates2);
  ```



### 类型与值查找

#### `findValueOfType` (单个类型)

> **`@Nullable public static <T> T findValueOfType(Collection<?> collection, @Nullable Class<T> type)`**

- **返回值**: `T` (可能为 `null`)

- **核心功能**:

  - 在集合中查找给定 `type` 类型的**单个**值。
  - 如果 `collection` 为 `null`、`type` 为 `null`、未找到该类型的值，或者找到了**多个**该类型的值，均返回 `null`。
  - 只返回严格匹配该类型（或其子类型）的唯一实例。

- **示例**:

  ```java
  List<Object> items = new ArrayList<>();
  items.add("hello");
  items.add(100);
  items.add("world");
  
  // result1 的值是 100 (Integer 类型的值只有一个)
  Integer result1 = CollectionUtils.findValueOfType(items, Integer.class);
  
  // result2 的值是 null (String 类型的值有 "hello" 和 "world" 两个，不唯一)
  String result2 = CollectionUtils.findValueOfType(items, String.class);
  
  // result3 的值是 null (Double 类型的值不存在)
  Double result3 = CollectionUtils.findValueOfType(items, Double.class);
  ```



#### `findValueOfType` (多个类型)

> **`@Nullable public static Object findValueOfType(Collection<?> collection, Class<?>[] types)`**

- **返回值**: `Object` (可能为 `null`)

- **核心功能**:

  - 在集合中按 `types` 数组的顺序查找**单个**值。
  - 它会首先搜索 `types[0]`，如果找到**唯一**的值，则立即返回；
  - 如果 `types[0]` 未找到或找到多个，它会接着搜索 `types[1]`，以此类推。
  - 如果 `collection` 或 `types` 为 `null`，或者遍历完所有类型都未找到唯一值，则返回 `null`。

- **示例**:

  ```java
  List<Object> items = new ArrayList<>();
  items.add("hello");
  items.add(100);
  
  // 1. 优先查找 Double，再查找 Integer
  Class<?>[] types1 = {Double.class, Integer.class};
  // result1 的值是 100 (Double 未找到，Integer 找到唯一的 100)
  Object result1 = CollectionUtils.findValueOfType(items, types1);
  
  // 2. 优先查找 String，再查找 Integer
  Class<?>[] types2 = {String.class, Integer.class};
  // result2 的值是 "hello" (String 找到唯一的 "hello"，不再查找 Integer)
  Object result2 = CollectionUtils.findValueOfType(items, types2);
  ```



#### `hasUniqueObject`

> **`public static boolean hasUniqueObject(Collection<?> collection)`**

- **返回值**: `boolean`

- **核心功能**:

  - 判断给定的 `Collection` 是否只包含一个**唯一的对象**。
  - “唯一的对象”有两种情况：
    1. 集合中只有一个元素。
    2. 集合中有多个元素，但所有元素都是指向**同一个实例**（使用 `==` 比较）。
  - 如果 `collection` 为 `null` 或 `empty`，返回 `false`。

- **示例**:

  ```java
  String s1 = "unique";
  
  List<String> list1 = List.of(s1);
  List<String> list2 = List.of(s1, s1, s1); // 多个引用指向同一个实例
  List<String> list3 = List.of("a", "b"); // 两个不同实例
  List<String> list4 = List.of(new String("a"), new String("a")); // 两个值相同但实例不同的对象
  
  // result1 为 true (只有一个元素)
  boolean result1 = CollectionUtils.hasUniqueObject(list1);
  
  // result2 为 true (所有元素都是 s1 这个实例)
  boolean result2 = CollectionUtils.hasUniqueObject(list2);
  
  // result3 为 false (包含两个不同实例)
  boolean result3 = CollectionUtils.hasUniqueObject(list3);
  
  // result4 为 false (两个实例，即使 .equals() 为 true)
  boolean result4 = CollectionUtils.hasUniqueObject(list4);
  ```



#### `findCommonElementType`

> **`@Nullable public static Class<?> findCommonElementType(Collection<?> collection)`**

- **返回值**: `Class<?>` (可能为 `null`)

- **核心功能**:

  - 查找并返回集合中所有元素的**共同元素类型**（通常是最近的公共父类或接口）。
  - 如果集合为 `null` 或 `empty`，返回 `null`。
  - 如果找不到明确的共同类型（例如，混合了 `String` 和 `Integer` 的 `List<Object>`，其共同类型是 `Object`，但 Spring 认为这不够“明确”？*更正：根据文档，它会找到最近的公共超类*）。

- **示例**:

  ```java
  List<String> strList = List.of("a", "b");
  List<Number> numList = List.of(10, 20.5); // 包含 Integer 和 Double
  List<Object> objList = List.of("hello", 100);
  List<Integer> emptyList = new ArrayList<>();
  
  // result1 的值是 String.class
  Class<?> result1 = CollectionUtils.findCommonElementType(strList);
  
  // result2 的值是 Number.class (Integer 和 Double 的共同父类)
  Class<?> result2 = CollectionUtils.findCommonElementType(numList);
  
  // result3 的值是 Object.class (String 和 Integer 的共同父类)
  Class<?> result3 = CollectionUtils.findCommonElementType(objList);
  
  // result4 的值是 null (集合为空)
  Class<?> result4 = CollectionUtils.findCommonElementType(emptyList);
  ```



### 转换/适配

#### `arrayToList`

> **`public static List<?> arrayToList(@Nullable Object source)`**

- **返回值**: `List<?>`

- **核心功能**:

  - 将传入的数组（`Object[]` 或**基本类型数组**）转换为 `List`
  - 如果 `source` 是基本类型数组（如 `int[]`），它将被转换为相应包装类型的 `List`（如 `List<Integer>`）
  - 如果 `source` 为 `null`，则返回一个空的 `List`
  - **注意**: 此方法主要用于处理在运行时才能确定是对象数组还是基本类型数组的 `Object`

- **示例**:

  ```java
  // 1. Object 数组
  String[] strArray = new String[] {"a", "b"};
  List<?> list1 = CollectionUtils.arrayToList(strArray);
  // list1 的内容是 ["a", "b"] (类型为 ArrayList<String>)
  
  // 2. 基本类型数组
  int[] intArray = new int[] {1, 2, 3};
  List<?> list2 = CollectionUtils.arrayToList(intArray);
  // list2 的内容是 [1, 2, 3] (类型为 ArrayList<Integer>)
  
  // 3. null
  List<?> list3 = CollectionUtils.arrayToList(null);
  // list3 是一个 empty List
  ```



#### `toArray`

> **`public static <A, E extends A> A[] toArray(Enumeration<E> enumeration, A[] array)`**

- **返回值**: `A[]`

- **核心功能**:

  - 将给定的 `Enumeration`（枚举）中的所有元素填充到给定类型的数组中
  - `Enumeration` 中的元素必须与 `array` 的类型兼容
  - **注意**: 根据文档，返回的数组将是与传入数组**不同的实例**

- **示例**:

  ```java
  Vector<String> vector = new Vector<>(List.of("one", "two"));
  Enumeration<String> enumeration = vector.elements();
  
  // 提供一个目标类型的数组（大小可以为 0）
  String[] targetArray = new String[0];
  
  // resultArray 是一个新实例，内容为 ["one", "two"]
  String[] resultArray = CollectionUtils.toArray(enumeration, targetArray);
  ```



#### `toIterator`

> **`public static <E> Iterator<E> toIterator(@Nullable Enumeration<E> enumeration)`**

- **返回值**: `Iterator<E>`

- **核心功能**:

  - 将一个（可能为 `null` 的）`Enumeration` 适配（转换）为一个 `Iterator`
  - 这对于在需要 `Iterator` 接口的现代 API 中使用旧版 `Enumeration` 非常有用
  - 如果 `enumeration` 为 `null`，则返回一个空的 `Iterator`

- **示例**:

  ```java
  Vector<String> vector = new Vector<>(List.of("A", "B"));
  // Vector.elements() 返回一个旧的 Enumeration
  Enumeration<String> enumeration = vector.elements();
  
  // 将 Enumeration 转换为 Iterator
  Iterator<String> iterator = CollectionUtils.toIterator(enumeration);
  
  // 现在可以使用 for-each 循环或 iterator API
  while (iterator.hasNext()) {
      System.out.println(iterator.next()); // 输出 A, 然后输出 B
  }
  
  // null 示例
  Iterator<String> emptyIterator = CollectionUtils.toIterator(null);
  // emptyIterator.hasNext() 为 false
  ```



### 合并操作

#### `mergeArrayIntoCollection`

> **`public static <E> void mergeArrayIntoCollection(@Nullable Object array, Collection<E> collection)`**

- **返回值**: `void`

- **核心功能**:

  - 将给定数组 `array` 中的所有元素合并到 `collection` 集合中。
  - `collection` 必须是泛型兼容的（即 `array` 中的元素可以被添加到 `collection`）。
  - 如果 `array` 为 `null` 或 `collection` 为 `null`，则不执行任何操作。
  - 同样支持基本类型数组（例如 `int[]` 会被当作 `Integer` 添加）。

- **示例**:

  ```java
  List<String> names = new ArrayList<>(List.of("Alice", "Bob"));
  String[] newNames = new String[] {"Charlie", "David"};
  
  // 将 newNames 数组合并到 names 列表中
  CollectionUtils.mergeArrayIntoCollection(newNames, names);
  
  // names 列表现在的内容是 ["Alice", "Bob", "Charlie", "David"]
  
  // 基础类型数组示例
  List<Integer> numbers = new ArrayList<>(List.of(1, 2));
  int[] newNumbers = new int[] {3, 4};
  CollectionUtils.mergeArrayIntoCollection(newNumbers, numbers);
  // numbers 列表现在的内容是 [1, 2, 3, 4]
  ```



#### `mergePropertiesIntoMap`

> **`public static <K, V> void mergePropertiesIntoMap(@Nullable Properties props, Map<K, V> map)`**

- **返回值**: `void`

- **核心功能**:

  - 将 `Properties` 对象 `props` 中的所有键值对（key-value pairs）合并到目标 `Map` 中。
  - `map` 的键和值类型必须与 `Properties` 中的键值对兼容（`Properties` 默认键值都是 `String`）。
  - 如果 `props` 或 `map` 为 `null`，则不执行任何操作。

- **示例**:

  ```java
  Properties settings = new Properties();
  settings.setProperty("user.name", "admin");
  settings.setProperty("user.level", "10");
  
  // 目标 Map
  Map<String, String> configMap = new HashMap<>();
  configMap.put("server.port", "8080");
  
  // 合并 Properties
  CollectionUtils.mergePropertiesIntoMap(settings, configMap);
  
  // configMap 现在的内容是:
  // {user.name=admin, user.level=10, server.port=8080}
  ```



### MultiValueMap 辅助

#### `toMultiValueMap`

> **`toMultiValueMap(Map<K, List<V>> targetMap)`**

- **返回值**: `MultiValueMap<K, V>`

- **核心功能**:

  - 将一个标准的 `Map<K, List<V>>` 适配（包装）成一个 Spring 的 `MultiValueMap<K, V>` 接口
  - **这是一个包装器**，而不是一次拷贝
  - 对返回的 `MultiValueMap` 的修改会直接反映到底层的原始 `Map` 中，反之亦然

- **示例**:

  ```java
  // 1. 准备一个普通的 Map
  Map<String, List<String>> normalMap = new HashMap<>();
  normalMap.put("fruits", new ArrayList<>(List.of("apple")));
  
  // 2. 将它包装成 MultiValueMap
  // MultiValueMap<String, String> multiMap = CollectionUtils.toMultiValueMap(normalMap);
  // (假设我们有 MultiValueMap 接口的访问权限，上面的 API 在 Spring 中可用)
  // (为了演示，我们假设 multiMap 是返回的对象)
  
  // 3. 使用 MultiValueMap 的便捷方法 'add'
  // multiMap.add("fruits", "banana");
  
  // 4. 检查原始 Map，它也被修改了
  // normalMap.get("fruits") 的值现在是 ["apple", "banana"]
  // System.out.println(normalMap);
  
  // 示例（模拟环境）
  System.out.println("原始 Map: " + normalMap);
  // 假设 CollectionUtils.toMultiValueMap(normalMap) 返回了一个 multiMap
  // multiMap.add("fruits", "banana");
  normalMap.get("fruits").add("banana"); // 模拟 multiMap.add 的效果
  System.out.println("修改后 Map: " + normalMap);
  // 输出:
  // 原始 Map: {fruits=[apple]}
  // 修改后 Map: {fruits=[apple, banana]}
  ```



#### `unmodifiableMultiValueMap`

> **`unmodifiableMultiValueMap(MultiValueMap<? extends K, ? extends V> targetMap)`**

- **返回值**: `MultiValueMap<K, V>`

- **核心功能**:

  - 返回指定 `MultiValueMap` 的一个**不可修改的视图** (unmodifiable view)。
  - 对返回的 Map 尝试进行任何写入操作（如 `add`, `put`, `remove`, `set` 等）都会抛出 `UnsupportedOperationException`。

- **示例**:

  ```java
  // 1. 准备一个 MultiValueMap (需要 Spring 依赖)
  // MultiValueMap<String, String> originalMap = new LinkedMultiValueMap<>();
  // originalMap.add("key", "value1");
  
  // 2. 创建一个不可修改的视图
  // MultiValueMap<String, String> unmodifiableMap =
  //     CollectionUtils.unmodifiableMultiValueMap(originalMap);
  
  // 3. 读取操作正常
  // System.out.println(unmodifiableMap.getFirst("key")); // 输出 "value1"
  
  // 4. 尝试写入操作
  try {
      // unmodifiableMap.add("key", "value2"); // 这行会抛出异常
      System.out.println("尝试写入...");
      // 模拟抛出异常
      throw new UnsupportedOperationException("这是一个不可修改的 Map");
  } catch (UnsupportedOperationException e) {
      System.out.println("写入失败: " + e.getMessage());
  }
  
  // 输出:
  // 尝试写入...
  // 写入失败: 这是一个不可修改的 Map
  ```



### 组合 Map（合成视图/可控写入）

#### `compositeMap` (双 Map 只读组合)

> **`compositeMap(Map<K, V> first, Map<K, V> second)`**

- **返回值**: `Map<K, V>`

- **核心功能**:

  - 将两个 Map 合并为一个组合视图 Map (`first` 优先)。
  - 这是一个**部分不可修改**的 Map。
  - 在这个返回的 Map 上调用 `put()` 或 `putAll()` 方法会抛出 `UnsupportedOperationException`。
  - 当 `first` 和 `second` 中有相同的 key 时，会优先保留 `first` 中的键值对。

- **示例**:

  ```Java
  Map<String, String> map1 = Map.of("A", "Apple", "B", "Banana");
  Map<String, String> map2 = Map.of("B", "Old Banana", "C", "Cherry");
  
  // A=Apple, B=Banana (来自 map1), C=Cherry (来自 map2)
  Map<String, String> composite = CollectionUtils.compositeMap(map1, map2);
  
  System.out.println(composite.get("A")); // 输出 "Apple"
  System.out.println(composite.get("B")); // 输出 "Banana" (map1 优先)
  System.out.println(composite.get("C")); // 输出 "Cherry"
  
  try {
      // 尝试写入，将抛出 UnsupportedOperationException
      composite.put("D", "Date");
  } catch (UnsupportedOperationException e) {
      System.out.println("写入失败，这是一个只读组合Map。");
  }
  ```



#### `compositeMap` (双 Map 自定义写入)



> **`compositeMap(Map<K, V> first, Map<K, V> second, @Nullable BiFunction<K, V, V> putFunction, @Nullable Consumer<Map<K, V>> putAllFunction)`**

- **返回值**: `Map<K, V>`

- **核心功能**:

  - 将两个 Map 合并为一个组合视图 Map (`first` 优先)
  - 允许自定义 `put()` 和 `putAll()` 的行为
  - 调用 `put()` 时，会执行你传入的 `putFunction`；如果 `putFunction` 为 `null`，则抛出 `UnsupportedOperationException`。
  - 调用 `putAll()` 时，会执行你传入的 `putAllFunction`；如果 `putAllFunction` 为 `null`，则抛出 `UnsupportedOperationException`
  - 键冲突时，`first` 优先

- **示例**:

  ```java
  Map<String, String> map1 = new HashMap<>(Map.of("A", "Apple"));
  Map<String, String> map2 = new HashMap<>(Map.of("B", "Banana"));
  
  // 定义 put 行为：允许写入，但所有 value 都转为大写
  BiFunction<String, String, String> putFunc = (key, value) -> {
      // 假设我们决定只允许写入到 map1
      return map1.put(key, value.toUpperCase());
  };
  
  // 定义 putAll 行为：将所有条目添加到 map1
  Consumer<Map<String, String>> putAllFunc = (map) -> {
      map.forEach(putFunc::apply);
  };
  
  Map<String, String> composite = CollectionUtils.compositeMap(map1, map2, putFunc, putAllFunc);
  
  System.out.println(composite.get("A")); // "Apple"
  System.out.println(composite.get("B")); // "Banana"
  
  // 调用自定义的 put
  composite.put("C", "Cherry");
  
  // 检查源 map (map1) 是否被修改
  System.out.println(map1); // {A=Apple, C=CHERRY}
  // 检查组合 map (它会反映底层的变化)
  System.out.println(composite.get("C")); // "CHERRY"
  ```



# `ReflectionUtils` **反射操作**



# `ClassUtils` **Class操作**



# `FileCopyUtils` **文件/流操作**



# `StreamUtils` 流操作



# `DigestUtils` **MD5 摘要**

## 基本概念

- `DigestUtils` 是 Spring Core 提供的一个**轻量级摘要工具类**，集中做 **MD5** 相关的便捷操作

  它是**静态方法**组成、**无状态**、**线程安全**的

- 注意：它**只封装了 MD5**

  如果你需要 SHA-1/256/512、HMAC 等，请使用 JDK 的 `MessageDigest`/`Mac`，或 Apache Commons Codec 的 `org.apache.commons.codec.digest.DigestUtils`等



## 关于 MD5 加密算法

### 1. 什么是 MD5？

- MD5（Message-Digest Algorithm 5）是一种曾被广泛使用的**密码哈希函数**（也常被称为“消息摘要算法”）

- 它的核心功能是：

  **接收任意长度的输入数据（如文本、文件、图片等），并产生一个固定长度的输出，即一个 128 位（16 字节）的哈希值（或称“摘要”）**

  - 这个 128 位的哈希值通常用一个 32 个字符的十六进制数来表示（例如：`e10adc3949ba59abbe56e057f20f883e`）



### 2. MD5 的核心特性

- MD5 之所以流行，因为它具备以下几个关键特性：
  1. **固定长度输出**： 无论你输入的是一个单词 "Hello"，还是一部 2GB 的电影，MD5 计算后输出的**永远**是一个 128 位的哈希值（即 32 个十六进制字符）
  2. **确定性**： 只要输入内容完全相同，无论何时、何地、用何种工具计算，得到的 MD5 哈希值**永远**是相同的
     - `MD5("Hello")` 永远等于 `8b1a9953c4611296a827abf8c47804d7`
  3. **雪崩效应（高灵敏性）**： 输入的任何微小变化（哪怕只是一个字母大小写、一个空格）都会导致输出的哈希值发生巨大且完全不同的变化。
     - `MD5("password123")` -> `ef59a30ff543b5168e805fbe0b396c3b`
     - `MD5("Password123")` -> `e5e3c7c236f0f5b41052efa4187f5e92`
  4. **单向性（不可逆性）**：
     - **正向计算容易**：从原始数据计算出 MD5 值非常快（就像你刚用的 `DigestUtils`）
     - **反向破解极难**：从一个 MD5 哈希值（如 `e10adc3949ba59abbe56e057f20f883e`）几乎不可能*直接*推算出原始数据
     - *注意：虽然不能“算”出来，但黑客可以通过“查字典”（即彩虹表）的方式进行“碰撞”破解*



### 3. MD5 的主要用途

- 基于以上特性，MD5 主要用于以下几个方面：

  1. **校验数据完整性（文件校验）** 

     - 这是 MD5 **最常用且仍然有效**的场景
       1. **场景**：你从网站下载一个大文件（如系统镜像、软件安装包）。网站通常会提供一个 MD5 值
       2. **过程**：你下载文件后，在本地对该文件也计算一次 MD5 值
       3. **对比**：如果你计算出的 MD5 值与网站提供的完全一致，就证明这个文件在下载过程中没有损坏、没有被篡改

  2. **数据“指纹”（唯一标识）** 

     - 在数据库或某些系统中，MD5 可以用来快速比较两个大对象（如文件内容、图片）是否相同

       比较 32 个字符的 MD5 值，远比逐字节比较整个文件的效率高得多

  3. **（不推荐）密码存储**

     - **历史做法**：

       - 以前，系统为了不存储用户的明文密码，会存储密码的 MD5 值

         用户登录时，系统计算用户输入密码的 MD5，与数据库中的 MD5 比较

     - **为何不推荐**：见下一节



### 4. MD5 的安全性问题⭐

- MD5 在**密码学意义上被认为是“已破解”或“不安全”的**

  - **核心缺陷：碰撞漏洞（Collision Vulnerability）**

    -  在 2004 年，密码学家（包括中国的王小云教授）证明了可以（相对）容易地找到两个**不同的输入**，它们却能产生**完全相同**的 MD5 哈希值

      - **这意味着什么？**

        1. **密码破解**：

           - 黑客可以构建一个庞大的“彩虹表”（Rainbow Table），存储常用密码及其 MD5 值的对应关系

             如果你的密码（如 "123456"）的 MD5 值泄露了，黑客通过查表瞬间就能知道你的明文密码

        2. **伪造签名**：

           - 攻击者可以创建一份“假”合同，使其 MD5 值与“真”合同的 MD5 值相同，从而实现欺诈。



### 5. 总结：现在还该用 MD5 吗？

- **不应该用于**：

  - 密码存储（绝对禁止！）

    > 虽然很多地方抱着侥幸的心理还在用这个，比如我自己学习过程中写项目也会用这个

  - 数字签名

  - SSL 证书

  - 任何对“防碰撞”有严格安全要求的场景

- **仍然可以用于**：

  - **文件完整性校验**（这是它的主要阵地）
  - 数据去重（作为“指纹”）
  - 非安全相关的哈希（例如负载均衡中的哈希路由）

- **安全替代方案**： 如果需要存储密码或用于安全签名，请使用更强大的哈希算法，例如：

  - **SHA-256** （SHA-2 系列）
  - **SHA-3**
  - 对于密码，更推荐使用 **bcrypt** 或 **Argon2**，它们专门设计用于对抗密码破解



## 常用方法整理

- MD5（以及所有哈希算法）的计算对象是**字节序列（bytes）**，所以形参几乎都是字节数组(`byte[]`)，或者字节流(`InputStream`)

### `md5Digest` (重载 1)

> **`static byte[] md5Digest(byte[] bytes)`**

- **返回值**: `byte[]`

- **核心功能**:

  - 计算给定字节数组的 MD5 摘要

- **示例**:

  ```java
  import org.springframework.util.DigestUtils;
  
  String input = "Hello Spring";
  byte[] inputBytes = input.getBytes("UTF-8");
  
  // 计算 MD5 摘要（返回字节数组）
  byte[] digest = DigestUtils.md5Digest(inputBytes);
  
  // digest 将会是一个16字节的数组
  ```



### `md5Digest` (重载 2)

> **`static byte[] md5Digest(InputStream inputStream)`**

- **返回值**: `byte[]`

- **核心功能**:

  - 计算给定输入流 (InputStream) 的 MD5 摘要
  - **注意**: 此方法不会关闭传入的 `InputStream`

- **示例**:

  ```java
  import org.springframework.util.DigestUtils;
  import java.io.InputStream;
  import java.io.FileInputStream;
  
  try (InputStream is = new FileInputStream("myFile.txt")) {
      // 计算文件的 MD5 摘要
      byte[] digest = DigestUtils.md5Digest(is);
      // digest 将会是一个16字节的数组
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```



### `md5DigestAsHex` (重载 1)

> **`static String md5DigestAsHex(byte[] bytes)`**

- **返回值**: `String`

- **核心功能**:

  - 计算给定字节数组的 MD5 摘要，并以十六进制字符串（32个字符）的形式返回
  - 这是最常用的方法之一

- **示例**:

  ```java
  import org.springframework.util.DigestUtils;
  
  String input = "Hello Spring";
  byte[] inputBytes = input.getBytes("UTF-8");
  
  // 计算 MD5 摘要并返回 Hex 字符串
  String hexDigest = DigestUtils.md5DigestAsHex(inputBytes);
  
  // hexDigest 的值类似于 "d41d8cd98f00b204e9800998ecf8427e" (具体值取决于输入)
  System.out.println(hexDigest); 
  ```



### `md5DigestAsHex` (重载 2)

> **`static String md5DigestAsHex(InputStream inputStream)`**

- **返回值**: `String`

- **核心功能**:

  - 计算给定输入流 (InputStream) 的 MD5 摘要，并以十六进制字符串（32个字符）的形式返回
  - **注意**: 此方法不会关闭传入的 `InputStream`

- **示例**:

  ```java
  import org.springframework.util.DigestUtils;
  import java.io.InputStream;
  import java.io.FileInputStream;
  
  try (InputStream is = new FileInputStream("myFile.txt")) {
      // 计算文件的 MD5 摘要并返回 Hex 字符串
      String hexDigest = DigestUtils.md5DigestAsHex(is);
  
      System.out.println(hexDigest);
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```



### `appendMd5DigestAsHex` (重载 1)

> **`static StringBuilder appendMd5DigestAsHex(byte[] bytes, StringBuilder builder)`**

- **返回值**: `StringBuilder`

- **核心功能**:

  - 计算给定字节数组的 MD5 摘要（十六进制形式），并将其追加到传入的 `StringBuilder` 对象中
  - 返回传入的 `StringBuilder` 实例，以支持链式调用

- **示例**:

  ```java
  import org.springframework.util.DigestUtils;
  
  String input = "Hello Spring";
  byte[] inputBytes = input.getBytes("UTF-8");
  
  StringBuilder sb = new StringBuilder("File Checksum: ");
  
  // 将计算得到的 Hex 字符串追加到 sb 后面
  DigestUtils.appendMd5DigestAsHex(inputBytes, sb);
  
  // sb.toString() 的值类似于 "File Checksum: d41d8cd98f00b204e9800998ecf8427e"
  System.out.println(sb.toString());
  ```



### `appendMd5DigestAsHex` (重载 2)

> **`static StringBuilder appendMd5DigestAsHex(InputStream inputStream, StringBuilder builder)`**

- **返回值**: `StringBuilder`

- **核心功能**:

  - 计算给定输入流 (InputStream) 的 MD5 摘要（十六进制形式），并将其追加到传入的 `StringBuilder` 对象中
  - **注意**: 此方法不会关闭传入的 `InputStream`

- **示例**:

  ```java
  import org.springframework.util.DigestUtils;
  import java.io.InputStream;
  import java.io.FileInputStream;
  
  StringBuilder sb = new StringBuilder("Data Checksum: ");
  
  try (InputStream is = new FileInputStream("myFile.txt")) {
      // 将计算得到的 Hex 字符串追加到 sb 后面
      DigestUtils.appendMd5DigestAsHex(is, sb);
      System.out.println(sb.toString());
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```



# `Base64Utils` Base64 编解码



# `HtmlUtils` **HTML 转义**