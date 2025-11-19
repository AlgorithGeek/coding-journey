# Jackson

## 1. Jackson 简介

### 1.1 简述

Jackson 是一个基于 Java 的、高性能的 JSON 处理库

它在 Java 生态系统中被广泛用于将 Java 对象 转换为 JSON 字符串 (**序列化**)，以及将 JSON 字符串转换回 Java 对象 (**反序列化**)

由于其强大的功能和卓越的性能，Jackson 已成为事实上的标准，特别是在构建 RESTful Web 服务中，JSON 是主要的数据交换格式



### 1.2 主要特点

**高性能**：

- Jackson 具备高吞吐量和较低的内存开销

  它通过优化的数据绑定、字节流处理以及对 `afterburner` 等模块的 JIT 优化支持，实现了快速的序列化与反序列化操作



**功能强大与成熟**：

- **完整 JSON 支持**：全面支持 JSON 规范，包括处理复杂的嵌套对象、数组、Map 和基本数据类型
- **多态与泛型**：能够正确处理 Java 的泛型（如 `List<String>`）和多态类型
- **数据格式支持**：除了 JSON，Jackson 的生态还支持其他数据格式，如 XML, CSV, YAML, Avro, Protobuf 等（通过不同的模块）



**Spring 生态默认集成**：

- Spring Boot 框架默认选择 Jackson 作为其 `spring-boot-starter-web` 中的 JSON 处理器

  这意味着在 Spring 环境中，HTTP 请求体和响应体的 JSON 转换会被自动处理



**灵活的配置性**：

- **注解驱动**：提供了大量注解（如 `@JsonProperty`, `@JsonIgnore`）用于在类级别或字段级别精细控制序列化和反序列化行为
- **模块化配置**：可以通过 `ObjectMapper` 注册模块（Modules）和配置特性（Features）来全局定义行为





### 1.3 Maven依赖

Jackson 的库被设计为模块化的



#### 基础模块

`jackson-databind` 是我们最常直接添加的依赖，它会自动将其所需的基础模块(`jackson-core` 和 `jackson-annotations`)一并引入

```xml
<!-- 
  核心依赖：jackson-databind (数据绑定)
  功能：这是 Jackson 的主要模块，提供了 ObjectMapper 类
       ObjectMapper 是用于实现 Java 对象 (POJO) <-> JSON 转换的核心
       它封装了所有底层的解析和生成逻辑，是功能最全、最易用的入口
  传递性：
  1. 自动引入 `jackson-core`
  2. 自动引入 `jackson-annotations`
-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>
```



**传递性依赖（自动引入）**

当你添加 `jackson-databind` 时，以下两个包也会被 Maven 自动下载：

1. **`jackson-core`（核心 - 流式处理）**
   - **功能**：提供了底层的、高性能的流式 API（`JsonParser` 和 `JsonGenerator`）
   - **用途**：
     - `databind` 模块在内部使用这个库来实际读取和写入 JSON 数据，它不包含数据绑定逻辑，只负责高效的 JSON 语法解析和生成
2. **`jackson-annotations`（注解）**
   - **功能**：提供了 Jackson 的标准注解集
   - **用途**：
     - 例如 `@JsonProperty`, `@JsonIgnore`, `@JsonFormat` 等这些注解用于在你的 Java 类中声明如何进行序列化或反序列化



#### 常用可选模块

Jackson 默认只知道如何处理标准的 Java 类型。对于 Java 8 引入的新类型或特定编译特性，你需要手动添加相应的模块

```xml
<!-- 
  可选模块：Java 8 时间支持 (JSR-310)
  功能：添加对 Java 8 `java.time` 包中日期和时间类库的支持。
  原因：没有此模块，Jackson 不知道如何将 "2023-01-01T12:00:00" 
       这样的字符串转换为 LocalDateTime 对象（反之亦然），会抛出异常。
  使用：添加依赖后，需要通过 ObjectMapper.registerModule(new JavaTimeModule()) 来注册。
       (在 Spring Boot 中，如果检测到此依赖，通常会自动注册。)
-->
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.15.2</version>
</dependency>
```

```xml
<!-- 
  可选模块：Java 8 参数名支持
  功能：允许 Jackson 在反序列化时，能够识别并使用构造函数或工厂方法的参数名。
  原因：默认情况下，Java 编译后的 `.class` 文件不保留参数名（只显示 arg0, arg1）。
       此模块利用 Java 8 的新特性来获取真实的参数名。
  用途：这对于反序列化不可变对象（Immutable Objects）至关重要，
       因为它们通常依赖于全参构造函数来创建实例。
       (同样，在 Spring Boot 中会自动配置)。
-->
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-parameter-names</artifactId>
    <version>2.15.2</version>
</dependency>
```



------

## 2. 核心模块

Jackson 的架构分为三个核心模块，这种分层设计允许开发者根据需要选择使用



### 2.1 jackson-core

**功能**：提供了底层的、高性能的 **流式 API (Streaming API)**

**详细说明**： 

- 这是 Jackson 的基础。它不关心 Java 对象 的绑定，

  只负责高效地读写 JSON 数据流，一次处理一个 JSON "令牌"（Token，如 `START_OBJECT`, `FIELD_NAME`, `VALUE_STRING` 等）

  - **`JsonGenerator`**：用于 **写入** JSON 数据

  - **`JsonParser`**：用于 **解析** JSON 数据

**使用场景**： 

- 这种方式性能最高、内存占用最低，因为它不需要将整个 JSON 文档加载到内存中

  但是，它使用起来非常繁琐和冗长。通常，只有在处理超大 JSON 文件或对性能有极致要求的场景下，才会直接使用它

  > 这个不会用到，说实话在此之前我都没见过这个，绑定的我估计底层就是这个

- `jackson-databind` 模块在内部封装了它



**示例代码**

```java
// JsonGenerator - 写入 JSON
JsonFactory factory = new JsonFactory();
JsonGenerator generator = factory.createGenerator(new File("output.json"), JsonEncoding.UTF8);

generator.writeStartObject(); // {
generator.writeStringField("name", "张三"); // "name": "张三"
generator.writeNumberField("age", 25); // "age": 25
generator.writeEndObject(); // }
generator.close();

// JsonParser - 解析 JSON
JsonParser parser = factory.createParser(new File("input.json"));
while (parser.nextToken() != JsonToken.END_OBJECT) {
    String fieldName = parser.getCurrentName();
    if ("name".equals(fieldName)) {
        parser.nextToken(); // 移动到 value
        System.out.println("Name: " + parser.getText());
    } else if ("age".equals(fieldName)) {
        parser.nextToken(); // 移动到 value
        System.out.println("Age: " + parser.getIntValue());
    }
}
parser.close();
```



### 2.2  jackson-annotations

**功能**：提供了一套标准的 Java **注解**，用于配置 JSON 序列化和反序列化的行为

**详细说明**： 

- 该模块本身不包含任何处理逻辑，它只定义了注解

  这些注解被 `jackson-databind` 模块识别和使用，从而允许开发者在 POJO 类上直接声明数据如何被转换



**常用注解示例**：

- `@JsonProperty`：
  - 自定义 POJO 字段与 JSON 属性之间的映射名称（例如 `private String userName;` 映射为 `{"user_name": "..."}`）
- `@JsonIgnore`：
  - 在序列化或反序列化时完全忽略某个字段
- `@JsonFormat`：
  - 定义日期、时间等类型的输出格式（例如 `@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd")`）
- `@JsonInclude`：
  - 控制在序列化时是否包含 null、空值或默认值字段（例如 `JsonInclude.Include.NON_NULL`）
- `@JsonCreator`：
  - 指定用于反序列化的构造函数或工厂方法，常用于不可变对象



### 2.3 jackson-databind

**功能**：提供了 **数据绑定** 功能，这是 Jackson 最常用、功能最强大的模块

**详细说明**： 该模块是绝大多数 Java 开发者与 Jackson 交互的入口。它提供了两种主要的数据绑定模式：

1. **简单数据绑定**：
   - **核心类**：`ObjectMapper`
   - **功能**：
     - 提供了 `writeValueAsString()` (`POJO -> JSON`) 和 `readValue()` (`JSON -> POJO`) 方法
   - **协同工作**：
     - `ObjectMapper` 在内部使用 `jackson-core` 来执行实际的读写，并使用 `jackson-annotations` 来应用 POJO 上的注解配置
2. **树模型**：
   - **核心类**：`JsonNode`
   - **功能**：允许将 JSON 读/写为内存中的树形结构
   - **使用场景**：
     - 当 JSON 结构不固定，或者你不需要将其完全映射为 POJO，而是只想灵活地读取或修改 JSON 的特定部分时，`JsonNode` 非常有用



------

## 3. `ObjectMapper`

**ObjectMapper** 是 `jackson-databind` 模块的核心，它是 Java 开发者与 Jackson 交互的主要入口点

它负责在 Java 对象（POJO）和 JSON 数据之间执行双向转换



### 3.1 常用方法 (读写 API)

`ObjectMapper` 提供的方法可以大致分为三类：序列化（写）、反序列化（读）、以及树模型/类型转换



#### a. 序列化 (Java 对象 → JSON)

这些方法用于将 Java 对象转换为 JSON 格式



##### 示例前提

*为了使后面的示例可运行，我们假设已定义了 `mapper` 和 `user` 对象*

```java
// --- 示例中通用的设置 ---
// 1. 创建 ObjectMapper 实例
ObjectMapper mapper = new ObjectMapper();
```

```java
// 2. 创建一个简单的 POJO (User.java)
class User {
    public String name;
    public int age;
    
    public User() {}
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    //  getter/setter 省略...
    
    @Override
    public String toString() {
        return "User{" + "name='" + name + '\'' + ", age=" + age + '}';
    }
}
```

```java
// 3. 创建一个 User 实例
User user = new User("张三", 25);
// --- 后续示例均基于以上代码 ---
```



##### `writeValueAsString(Object)`

- `writeValueAsString(Object value)`

- **功能**：将一个 Java 对象序列化为 JSON **字符串**。这是最常用的序列化方法

- **返回值**：`String` (JSON 格式的字符串)

- **使用示例**：

  ```java
  try {
      String jsonString = mapper.writeValueAsString(user);
      // 输出: {"name":"张三","age":25}
      System.out.println(jsonString);
  } catch (JsonProcessingException e) {
      e.printStackTrace();
  }
  ```



##### `writeValue(File,Object)`

- `writeValue(File resultFile, Object value)`

- **功能**：将一个 Java 对象序列化为 JSON，并将其**写入指定文件**

- **返回值**：`void`

- **使用示例**：

  ```java
  try {
      // 将 user 对象序列化并写入 user.json 文件
      mapper.writeValue(new File("user.json"), user);
      // (user.json 文件内容将变为: {"name":"张三","age":25})
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```



##### `writeValue(OutputStream,Object)`

- `writeValue(OutputStream out, Object value)`

- **功能**：将一个 Java 对象序列化为 JSON，并将其 **写入输出流 **

- **返回值**：`void`

- **使用示例**：(这里我们使用 `System.out` 作为输出流)

  ```java
  try {
      // 将 JSON 直接输出到控制台
      mapper.writeValue(System.out, user);
      // 控制台输出: {"name":"张三","age":25}
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```



##### `writeValueAsBytes(Object)`

- `writeValueAsBytes(Object value)`

- **功能**：将一个 Java 对象序列化为 JSON **字节数组** (`byte[]`)

- **返回值**：`byte[]`

- **使用示例**：

  ```java
  try {
      byte[] jsonBytes = mapper.writeValueAsBytes(user);
      // 输出字节数组的字符串表示形式
      System.out.println(new String(jsonBytes, "UTF-8"));
      // 输出: {"name":"张三","age":25}
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```



#### b. 反序列化(JSON → Java对象)

这些方法用于将 JSON 数据转换为 Java 对象

##### 示例前提

*为使示例可运行，我们假设已定义了 `mapper`、`User` 类和 `jsonString` 等变量*

```java
// --- 示例中通用的 JSON 数据 ---
String jsonString = "{\"name\":\"张三\",\"age\":25}";
String jsonListString = "[{\"name\":\"张三\",\"age\":25}, {\"name\":\"李四\",\"age\":30}]";
String jsonMapString = "{\"user1\":{\"name\":\"张三\",\"age\":25}, \"user2\":{\"name\":\"李四\",\"age\":30}}";
// 假设 "user.json" 文件的内容为: {"name":"张三","age":25}
```

```java
// --- 示例中通用的 User 类 ---
class User {
    public String name;
    public int age;
    
    // 注意：反序列化通常需要一个无参构造函数
    public User() {}
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public String toString() {
        return "User{" + "name='" + name + '\'' + ", age=" + age + '}';
    }
}
```

```java
// --- 示例中通用的 ObjectMapper ---
ObjectMapper mapper = new ObjectMapper();
// --- 示例均基于以上代码 ---
```



##### `readValue(String,Class)`

- `readValue(String content, Class<T> valueType)`

- **功能**：从 JSON **字符串**中读取数据，并将其转换为**指定的简单 Java 类型** (例如 `User.class`)

- **返回值**：`T` (即 `valueType` 类型的实例)

- **使用示例**：

  ```java
  try {
      User user = mapper.readValue(jsonString, User.class);
      // 输出: User{name='张三', age=25}
      System.out.println(user); 
  } catch (JsonProcessingException e) {
      e.printStackTrace();
  }
  ```



##### `readValue(File,Class)`

- `readValue(File src, Class<T> valueType)`

- **功能**：从**文件**中读取 JSON 数据，并将其转换为指定的简单 Java 类型

- **返回值**：`T` (即 `valueType` 类型的实例)

- **使用示例**：

  ```java
  try {
      // 确保 user.json 文件存在且内容为 {"name":"张三","age":25}
      User user = mapper.readValue(new File("user.json"), User.class);
      // 输出: User{name='张三', age=25}
      System.out.println(user);
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```



##### `readValue(InputStream,Class)`

- `readValue(InputStream src, Class<T> valueType)`

- **功能**：从**输入流**中读取 JSON 数据，并将其转换为指定的简单 Java 类型

- **返回值**：`T` (即 `valueType` 类型的实例)

- **使用示例**：(这里使用 `ByteArrayInputStream` 模拟文件流)

  ```java
  try (InputStream input = new ByteArrayInputStream(jsonString.getBytes("UTF-8"))) {
      User user = mapper.readValue(input, User.class);
      // 输出: User{name='张三', age=25}
      System.out.println(user);
  } catch (IOException e) {
      e.printStackTrace();
  }
  ```





##### `readValue(String,TypeReference)`

- `readValue(String content, TypeReference<T> valueTypeRef)`

- **功能**：从 JSON 字符串中读取数据，并将其转换为**复杂的泛型类型**（例如 `List<User>` 或 `Map<String, User>`）

- **返回值**：`T` (即 `TypeReference` 所指定的泛型类型实例)

- **注意**：`TypeReference` 是一个抽象类，使用时必须创建它的匿名子类 (`new TypeReference<...>() {}`)

- **使用示例 1 (JSON → List)**：

  ```java
  try {
      List<User> userList = mapper.readValue(jsonListString, new TypeReference<List<User>>() {});
      // 输出: [User{name='张三', age=25}, User{name='李四', age=30}]
      System.out.println(userList);
  } catch (JsonProcessingException e) {
      e.printStackTrace();
  }
  ```



- **使用示例 2 (JSON → Map)**：

  ```java
  try {
      Map<String, User> userMap = mapper.readValue(jsonMapString, new TypeReference<Map<String, User>>() {});
      // 输出: {user1=User{name='张三', age=25}, user2=User{name='李四', age=30}}
      System.out.println(userMap);
  } catch (JsonProcessingException e) {
      e.printStackTrace();
  } 
  ```



#### 3. 树模型与类型转换

`JsonNode` 是 Jackson 中表示 JSON 数据的树状结构

当 JSON 结构复杂、多变，或者你只需要访问其中几个字段时，使用树模型会比 POJO 绑定更灵活

##### 示例前提

*(为使示例可运行，我们假设已定义了 `mapper`、`User` 类和 `jsonString`)*

```java
// --- 示例中通用的 JSON 数据 ---
String jsonString = "{\"name\":\"张三\",\"age\":25, \"contact\": {\"email\": \"zhang@example.com\"}}";
```

```java
// --- 示例中通用的 User 类 ---
class User {
    public String name;
    public int age;
    // ... (构造函数、getter/setter、toString 同上)
}
```

```java
// --- 示例中通用的 ObjectMapper ---
ObjectMapper mapper = new ObjectMapper();
// --- 示例均基于以上代码 ---
```



##### `readTree(String)`

- `readTree(String content)`

- **功能**：将 JSON 字符串解析为一个 `JsonNode` (树的根节点)

- **返回值**：`JsonNode`

- **使用示例 (读取与导航)**：

  ```JAVA
  try {
      JsonNode rootNode = mapper.readTree(jsonString);
  
      // 1. 使用 path() - 推荐，安全
      // path() 方法在找不到节点时会返回一个 "MissingNode"，不会抛出空指针
      String name = rootNode.path("name").asText();
      int age = rootNode.path("age").asInt();
      String email = rootNode.path("contact").path("email").asText();
      String phone = rootNode.path("contact").path("phone").asText("N/A"); // 提供默认值
  
      // 输出: 张三
      System.out.println(name);
      // 输出: 25
      System.out.println(age);
      // 输出: zhang@example.com
      System.out.println(email);
      // 输出: N/A (因为 phone 节点不存在)
      System.out.println(phone);
  
      // 2. 使用 get() - 需要判空
      // get() 方法在找不到节点时返回 null，链式调用可能导致 NullPointerException
      if (rootNode.has("contact") && rootNode.get("contact").has("email")) {
          String emailFromGet = rootNode.get("contact").get("email").asText();
          // 输出: zhang@example.com
          System.out.println(emailFromGet);
      }
  
  } catch (JsonProcessingException e) {
      e.printStackTrace();
  }
  ```



##### `valueToTree(Object)`

- `valueToTree(Object fromValue)`

- **功能**：将一个 Java 对象 (POJO) 转换为 `JsonNode` 树模型

- **返回值**：`JsonNode`

- **使用示例**：

  ```JAVA
  User user = new User("李四", 30);
  JsonNode userNode = mapper.valueToTree(user);
  
  String name = userNode.path("name").asText();
  // 输出: 李四
  System.out.println(name);
  ```



##### `treeToValue(TreeNode,Class)`

- `treeToValue(TreeNode n, Class<T> valueType)`

- **功能**：将一个 `JsonNode` (或其子类 `ObjectNode`/`ArrayNode`) 转换回指定的 Java 对象 (POJO)

- **返回值**：`T` (即 `valueType` 类型的实例)

- **使用示例**：

  ```JAVA
  try {
      JsonNode rootNode = mapper.readTree(jsonString);
  
      // 将整个树节点转换回 User 对象
      User user = mapper.treeToValue(rootNode, User.class);
  
      // 输出: User{name='张三', age=25} (contact 字段会被忽略)
      System.out.println(user);
  
  } catch (JsonProcessingException e) {
      e.printStackTrace();
  }
  ```



### 3.2 `ObjectMapper` 配置

`ObjectMapper` 提供了大量配置项，允许开发者精细控制序列化和反序列化的行为



#### a. 序列化配置 (`SerializationFeature`)

这类配置控制 Java 对象如何被转换成 JSON

- 通过 `mapper.enable(...)` 或 `mapper.disable(...)` 来设置



##### `SerializationFeature.INDENT_OUTPUT` (美化输出)

- **功能**：使输出的 JSON 字符串带缩进和换行，使其更易读

- **默认**：`false` (输出为紧凑的单行 JSON)

- **配置**：`mapper.enable(SerializationFeature.INDENT_OUTPUT);`

- **示例**：

  - **默认 (false)**: `{"name":"张三","age":25}`

  - **启用 (true)**:

    ```
    {
      "name" : "张三",
      "age" : 25
    }
    ```



##### `SerializationFeature.WRITE_DATES_AS_TIMESTAMPS` (日期格式)

- **功能**：控制 `java.util.Date` 类型的序列化格式
- **默认**：`true` (序列化为 long 型的数字时间戳, 如 `1678886400000`)
- **配置**：`mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);`
- **说明**：禁用后，Jackson 会尝试使用 `DateFormat` 将其格式化为字符串
  - **(注意)**：此配置对 Java 8 的  `LocalDate/LocalDateTime`  (需要  `JavaTimeModule`)  **无效**



##### `SerializationFeature.FAIL_ON_EMPTY_BEANS` (空对象处理)

- **功能**：当序列化一个没有任何可序列化字段 (没有 public 字段或 getter) 的 "空" Java 对象时，是否抛出异常
- **默认**：`true` (抛出 `JsonMappingException`)
- **配置**：`mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);`
- **说明**：禁用后，空对象会被安全地序列化为 `{}`



##### `SerializationFeature.WRITE_ENUMS_USING_TO_STRING` (枚举序列化)

- **功能**：控制枚举序列化的方式

- **默认**：`false` (使用 `enum.name()`，即枚举的常量名)

- **配置**：`mapper.enable(SerializationFeature.WRITE_ENUMS_USING_TO_STRING);`

- **说明**：

  - 启用后，将调用枚举的 `toString()` 方法获取其值

    这在你需要为枚举定义特定 的 JSON 值（例如小写或特定描述）时很有用



#### b. 字段包含策略 (`JsonInclude`)

`setSerializationInclusion(...)` 用于全局控制**哪些值**的字段在序列化时应该被**忽略**



##### `JsonInclude.Include.ALWAYS`

- **功能**：默认策略。序列化所有字段，无论其值是否为 `null`
- **示例输出**：`{"name":"张三", "email":null}`



##### `JsonInclude.Include.NON_NULL`

- **功能**：**最常用配置**。序列化时忽略所有值为 `null` 的字段。这可以显著减小 JSON 载荷的大小
- **配置**：`mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);`
- **示例输出**：`{"name":"张三"}` (假设 `email` 字段为 `null`)



##### `JsonInclude.Include.NON_EMPTY`

- **功能**：比 `NON_NULL` 更严格。`null`、空字符串 (`""`)、空集合/Map 都会被忽略
- **配置**：`mapper.setSerializationInclusion(JsonInclude.Include.NON_EMPTY);`
- **示例输出**：`{"name":"张三"}` (假设 `email` 为 `null` 或 `""`，`pets` 为 `Collections.emptyList()`)



#### c. 反序列化配置 (`DeserializationFeature`)

这类配置控制 JSON 如何被转换成 Java 对象



##### `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES` (未知属性)

- **功能**：**最常用配置**。当 JSON 中的某个字段在 Java POJO 中不存在时，是否抛出异常
- **默认**：`true` (抛出 `UnrecognizedPropertyException`)
- **配置**：`mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);`
- **说明**：在实际 API 对接中，**强烈建议禁用此特性**，以增强系统的健壮性，防止因上游 JSON 增加新字段而导致服务崩溃



##### `DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT` (空字符串转Null)

- **功能**：允许将 JSON 中的空字符串 (`""`) 绑定到 POJO 时，视作 `null`
- **默认**：`false`
- **配置**：`mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);`
- **说明**：当 `{"email":""}` 传来时，启用此配置会使 `user.getEmail()` 返回 `null` 而不是 `""`



##### `DeserializationFeature.ACCEPT_FLOAT_AS_INT`

- **功能**：是否允许将 JSON 中的浮点数 (如 `5.0`) 截断并反序列化为 Java 的整数 (`int`) 类型 (变为 `5`)
- **默认**：`true`
- **配置**：`mapper.disable(DeserializationFeature.ACCEPT_FLOAT_AS_INT);`
- **说明**：禁用后，如果试图将 `5.0` 赋值给 `int` 字段，会抛出异常，增强类型安全性



#### d. 解析器特性 (`JsonParser.Feature`)

这些是更底层的解析器配置，用于兼容非标准的 JSON 格式 (例如手写的 JSON)

- **`JsonParser.Feature.ALLOW_SINGLE_QUOTES`**
  - **功能**：允许 JSON 字符串使用单引号 (`'`) 而不是标准的双引号 (`"`)
  - **配置**：`mapper.enable(JsonParser.Feature.ALLOW_SINGLE_QUOTES);`
  - **示例**：`{'name':'张三'}`



- **`JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES`**

  - **功能**：允许 JSON 中的字段名 (key) 不带引号

    > 这给整成JavaScript了啊

  - **配置**：`mapper.enable(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES);`

  - **示例**：`{name:"张三", age:25}`



- **`JsonParser.Feature.ALLOW_COMMENTS`**
  - **功能**：允许 JSON 中包含 C/C++ 风格的注释 ( `//` 和 `/* ... */` )
  - **配置**：`mapper.enable(JsonParser.Feature.ALLOW_COMMENTS);`
  - **示例**：`{ "name":"张三" // 这是一个名字 }`



#### e. 模块化扩展 (Modules)

Jackson 的核心 `databind` 模块默认不支持某些 Java 类型

模块 (Module) 是用来扩展 `ObjectMapper` 功能的插件，通过 `registerModule()` 注册



##### `JavaTimeModule` (Java 8 时间支持)

- **功能**：**现代 Java 开发必备模块**。添加对 `LocalDate`, `LocalDateTime`, `ZonedDateTime` 等 JSR-310 类型的序列化和反序列化支持
- **依赖**：需要引入 `com.fasterxml.jackson.datatype:jackson-datatype-jsr310`
- **配置**：`mapper.registerModule(new JavaTimeModule());`
- **说明**：注册后，`LocalDateTime` 会被序列化为标准 ISO 格式 (如 `"2025-11-11T12:00:00"`)，而不是数字时间戳



##### `ParameterNamesModule` (参数名支持)

- **功能**：支持将 JSON 字段自动映射到构造函数的参数名上（而不需要 `@JsonProperty` 注解），尤其适用于 Kotlin 或 Immutables 这类库
- **依赖**：需要引入 `com.fasterxml.jackson.module:jackson-module-parameter-names`。
- **配置**：`mapper.registerModule(new ParameterNamesModule());`



#### f. 命名策略

>`PropertyNamingStrategy` / `PropertyNamingStrategies`

用于自动转换 Java 字段名 (通常是驼峰式 `camelCase`) 和 JSON 字段名 (通常是其他风格) 之间的映射关系



##### `PropertyNamingStrategies.SNAKE_CASE` (下划线策略)

- **功能**：在 Java 的 `userName` 和 JSON 的 `user_name` 之间自动进行双向转换
- **配置**：`mapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);`
- **示例**：
  - Java: `user.getUserName()`
  - JSON (序列化/反序列化): `{"user_name": "...", "user_age": ...}`



##### 其他策略

- **`PropertyNamingStrategies.KEBAB_CASE`** (短横线策略)
  - **功能**：在 Java 的 `userName` 和 JSON 的 `user-name` 之间自动进行双向转换
  - **配置**：`mapper.setPropertyNamingStrategy(PropertyNamingStrategies.KEBAB_CASE);`



- **`PropertyNamingStrategies.LOWER_CAMEL_CASE`** (小驼峰策略)
  - **功能**：在 Java 的 `userName` 和 JSON 的 `userName` 之间进行映射 (即 Java 默认命名方式)
  - **配置**：`mapper.setPropertyNamingStrategy(PropertyNamingStrategies.LOWER_CAMEL_CASE);`
  - **说明**：这是 Jackson 的默认策略，通常不需要显式设置



- **`PropertyNamingStrategies.UPPER_CAMEL_CASE`** (大驼峰策略 / PascalCase)
  - **功能**：在 Java 的 `userName` 和 JSON 的 `UserName` (首字母大写) 之间自动进行双向转换
  - **配置**：`mapper.setPropertyNamingStrategy(PropertyNamingStrategies.UPPER_CAMEL_CASE);`



#### ⭐配置示例

```java
/**
 * Jackson ObjectMapper 的全局配置
 */
@Configuration
public class JacksonConfig {

    @Bean
    @Primary // 确保 Spring 优先使用这个自定义的 ObjectMapper
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();

        // === 核心配置：健壮性与现代 Java 支持 ===

        // 1. 注册 Java 8 时间模块 (必备)
        // 支持 LocalDate, LocalDateTime 等
        mapper.registerModule(new JavaTimeModule());
        
        // 2. 禁用：将日期序列化为时间戳 (配合 JavaTimeModule)
        // 序列化为 "2025-11-11T12:00:00" 而不是 1678886400000
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

        // 3. 禁用：反序列化时, 忽略 JSON 中存在而 Java 类中不存在的属性
        // 这是增强 API 健壮性的关键配置，防止上游增加字段导致服务崩溃
        mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);

        // === 推荐配置：优化 JSON 输出 ===

        // 4. 配置：序列化时, 仅包含非 null 值的字段
        // (可选: NON_EMPTY 包含 null, "" 和空集合)
        // 这可以显著减小 JSON 响应体的大小
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        
        // 5. 禁用：序列化空 POJO (没有字段) 时不抛异常
        // 默认会抛异常，禁用后会序列化为 {}
        mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
        
        // === 可选配置：根据团队规范选择 ===
        
        // 6. (可选) 统一命名策略：驼峰转下划线
        // Java: "userName" <--> JSON: "user_name"
        // mapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);
        
        // 7. (可选) 美化输出 (仅用于开发/调试)
        // 在生产环境应关闭此选项以节省带宽
        // mapper.enable(SerializationFeature.INDENT_OUTPUT);

        return mapper;
    }
}
```



### 3.3 `ObjectMapper` 线程安全

`ObjectMapper` 的设计是**线程安全**的

> 意思是：创建一"个" `ObjectMapper` 实例，然后在多个线程（例如 Spring Boot 处理的多个并发 Web 请求）中 同时使用这"同一个"实例时，不会导致内部数据混乱或出错

这意味着你**不应该**在每次需要序列化或反序列化时都 `new ObjectMapper()`。`new` 的开销很大，因为它需要查找和注册所有模块 

**最佳实践**：

- 在应用程序启动时配置一个 `ObjectMapper` 单例，并在所有地方共享该实例

  在 Spring Boot 应用中，这通常通过定义一个 `@Bean` 来实现

  > 用的时候可以`@Autowired`



------

## 4. 常用注解大全

Jackson 的注解提供了对序列化和反序列化过程的精细化控制。它们可以覆盖 `ObjectMapper` 的全局配置

> 注解的优先级更高
>
> **`ObjectMapper` 配置 (如 `mapper.set...`)**：这是**全局默认**规则。它对 `mapper` 处理的**所有** POJO 生效
>
> **注解 (如 `@JsonInclude`)**：这是**局部特定**规则。它**只**对它所标注的那个**类**或**字段**生效



### 4.1. 类级别注解

这些注解直接标注在 `public class ...` 之上，用于定义整个类的 JSON 转换行为

#### `@JsonIgnoreProperties`

**核心功能**：在类级别上指定要忽略的一个或多个属性

**常用属性**：

- `value = {"field1", "field2"}`：一个字符串数组，列出在序列化和反序列化时都希望忽略的 POJO 字段名

- `ignoreUnknown = true`：**(非常常用)** 仅在**反序列化**时生效

  - 如果 JSON 字符串中包含此类中不存在的字段，**不**抛出异常，而是安全地忽略它们

    这等同于 `ObjectMapper` 的 `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES = false` 配置，但只对当前类生效

**示例1：忽略指定属性 (双向忽略)**

```JAVA
// 序列化和反序列化时, 都会忽略 "password" 和 "internalId" 字段
@JsonIgnoreProperties({"password", "internalId"})
public class User {
    private String username;
    private String password;   // 此字段将被忽略
    private Long internalId; // 此字段将被忽略
}

// 序列化: new User("test", "123", 1L) -> {"username":"test"}
// 反序列化: {"username":"test", "password":"123"} -> User(username="test", password=null, ...)
```



**示例2：忽略未知属性 (推荐的反序列化配置)**

```JAVA
// 推荐在 DTO (Data Transfer Object) 上使用此注解
@JsonIgnoreProperties(ignoreUnknown = true)
public class User {
    private String username;
    private Integer age;
}

// JSON: {"username":"test", "age":20, "newField":"abc"}
// 反序列化时, "newField" 在 User 类中不存在, 但不会导致程序
// 抛出 UnrecognizedPropertyException 异常, 而是被安全忽略。
```



#### `@JsonInclude`

**核心功能**：在**序列化**时，全局控制此类中**哪些值**的字段**不**被输出。这是一种比 `ObjectMapper` 全局配置更精细的控制

**常用属性**：`value = JsonInclude.Include...`

- `ALWAYS`：(默认) 序列化所有字段
- `NON_NULL`：**(常用)** 仅序列化值不为 `null` 的字段
- `NON_EMPTY`：仅序列化非 `null`、非空字符串 (`""`)、非空集合/Map/数组的字段
- `NON_DEFAULT`：仅序列化值不为 Java 默认值（如 `0`, `false`, `null`）的字段

**示例：仅包含非 null 值**

```JAVA
// 只有非 null 的字段才会被序列化到 JSON 字符串中
@JsonInclude(JsonInclude.Include.NON_NULL)
public class User {
    private String name;    // "张三" -> {"name":"张三"}
    private String email;   // null -> (此字段在 JSON 中不存在)
    private Integer age;    // 0 -> {"age":0} (因为 0 不是 null)
}

// 如果使用 NON_DEFAULT, 那么 age=0 的字段也会被忽略
```



#### `@JsonPropertyOrder`

**核心功能**：在**序列化**时，精确控制字段输出到 JSON 字符串中的顺序

**常用属性**：

- `value = {"field1", "field2"}`：按数组中指定的顺序排列字段
- `alphabetic = true`：按字段名称的字母顺序排列

**示例：指定字段顺序**

```JAVA
// 默认序列化顺序可能不可预测 (e.g., {"age":25, "email":"...", "id":1, "name":"..."})

// 指定顺序后
@JsonPropertyOrder({"id", "name", "age", "email"})
public class User {
    private String name;
    private Integer age;
    private Long id;
    private String email;
}
// 序列化: {"id":1, "name":"...", "age":25, "email":"..."}

// 按字母顺序
@JsonPropertyOrder(alphabetic = true)
public class User { ... }
// 序列化: {"age":25, "email":"...", "id":1, "name":"..."}
```



#### `@JsonRootName`

**核心功能**：在序列化和反序列化时，为对象包裹一层根 (root) 节点

**前提条件**：此注解**默认不生效**。你必须在 `ObjectMapper` 上显式启用"包裹/解包"功能

- `mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);` (序列化时启用)
- `mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);` (反序列化时启用)

**示例：包裹根节点**

```JAVA
@JsonRootName("user") // 定义根节点的名称
public class User {
    public String name;
    public User(String name) { this.name = name; }
}

// --- 配置 ---
// ObjectMapper mapper = new ObjectMapper();
// mapper.enable(SerializationFeature.WRAP_ROOT_VALUE);
// mapper.enable(DeserializationFeature.UNWRAP_ROOT_VALUE);

// --- 序列化 ---
// mapper.writeValueAsString(new User("张三"));
// 输出: {"user":{"name":"张三"}}
// (如果没启用 WRAP_ROOT_VALUE, 输出: {"name":"张三"})

// --- 反序列化 ---
// String json = "{\"user\":{\"name\":\"李四\"}}";
// User user = mapper.readValue(json, User.class);
// (如果没启用 UNWRAP_ROOT_VALUE, 会因找不到 "name" 字段而失败)
```



### 4.2. 字段级别注解

#### `@JsonProperty`

**核心功能**：这是最核心的字段注解，用于**自定义字段的名称、访问权限和必要性**



**常用属性**：

- `value` (默认属性)：指定该字段在 JSON 中对应的 **key** 名称
- `required`：(反序列化) 标记此字段为必需。如果 JSON 中缺少该字段，反序列化将失败 (默认为 `false`)
- `index`：(序列化) 指定一个数字，用于在 `@JsonPropertyOrder` 之外对字段进行排序，`index` 值越小越靠前
- `access`：控制字段在序列化/反序列化时的可访问性



**`access` 属性详解**：

- `JsonProperty.Access.READ_ONLY` (只读)：
  - **序列化时包含** (Java → JSON)
  - **反序列化时忽略** (JSON → Java)
  - 适用：`createdAt`, `id` 等由后端生成、不希望被前端传入的字段
- `JsonProperty.Access.WRITE_ONLY` (只写)：
  - **序列化时忽略** (Java → JSON)
  - **反序列化时包含** (JSON → Java)
  - 适用：`password`, `newEmail` 等只用于接收前端数据、但不希望在响应中返回的字段
- `JsonProperty.Access.READ_WRITE`：(默认) 序列化和反序列化都包含
- `JsonProperty.Access.AUTO`：(默认) 由 Jackson 自动推断



**示例**：

```java
public class User {

    // 1. 字段重命名：Java 驼峰 "username" ↔ JSON 下划线 "user_name"
    @JsonProperty("user_name")
    private String username;

    // 2. 必填字段：如果 JSON 中没有 "email"，反序列化将抛出异常
    @JsonProperty(required = true)
    private String email;

    // 3. 只写 (WRITE_ONLY)：用于接收密码，但序列化(返回)时会忽略
    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    private String password;

    // 4. 只读 (READ_ONLY)：用于返回创建时间，但反序列化(传入)时会忽略
    @JsonProperty(access = JsonProperty.Access.READ_ONLY)
    private String createdAt;

    // 5. 排序：index=1 会比其他未指定 index 的字段靠前
    @JsonProperty(index = 1)
    private Long id;
}
```





#### `@JsonIgnore`

**核心功能**：在字段级别**完全忽略**该属性

**说明**：

- 等同于 `@JsonProperty(access = JsonProperty.Access.READ_ONLY)` 和 `WRITE_ONLY` 同时设置

  > （虽然 `access` 没有这个选项），它在序列化和反序列化时都会被彻底忽略

**示例**：

```JAVA
public class User {
    private String username;

    // "password" 字段将完全不参与序列化和反序列化
    @JsonIgnore
    private String password;
}
// 序列化: {"username":"..."}
// 反序列化: {"username":"...", "password":"..."} -> password 字段为 null
```



#### `@JsonAlias`

**核心功能**：**仅在反序列化时**，为字段提供多个"别名"

**说明**：

- 这在兼容不同版本的 API 或外部数据源时极其有用

  无论 JSON 中传来的是 `value` 数组中的哪一个 key，都会被正确映射到该 Java 字段上

  **注意：序列化时不受影响** (仍使用 `JsonProperty` 或字段原名)

**示例**：

```JAVA
public class User {
    // 无论 JSON 是 {"user_name":"..."}, {"userName":"..."} 还是 {"name":"..."}
    // 都能被正确反序列化到 "username" 字段上
    @JsonAlias({"user_name", "userName", "name"})
    private String username;
}
```



#### `@JsonFormat`

**核心功能**：对特定字段的**格式**进行精细控制，最常用于**日期**和**数字**

**常用属性**：

- `pattern`：定义格式，如 `"yyyy-MM-dd HH:mm:ss"`
- `timezone`：定义时区，如 `"GMT+8"` 或 `"Asia/Shanghai"`
- `shape`：定义序列化后的 JSON 类型 (形状)

**示例1：格式化日期 (Date / LocalDateTime)**

```JAVA
public class Order {
    // 格式化 java.util.Date，并指定时区
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createTime;

    // 格式化 Java 8 的 LocalDateTime (通常不需要时区)
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime updateTime;
}
// 序列化输出: "createTime":"2025-11-11 14:30:00"
```

**示例2：(重要) 解决 JavaScript 精度丢失**

```JAVA
public class Product {
    // 将 Long/long 类型序列化为 JSON 字符串
    @JsonFormat(shape = JsonFormat.Shape.STRING)
    private Long id;
}
// 序列化: {"id":"9007199254740993"}
// 而不是: {"id": 9007199254740993} (这个数字 JS 无法精确表示)

// **原因**：JavaScript 的 Number 类型是 64 位浮点数，其最大安全整数
// (Number.MAX_SAFE_INTEGER) 仅为 2^53 - 1。Java 的 Long 是 64 位整数。
// 超过该值的 Long 型 ID 传递给 JS 会导致精度丢失 (数字被截断)。
// 将其序列化为字符串可确保 JS 安全地处理该 ID。
```



#### `@JsonUnwrapped`

**核心功能**：在序列化/反序列化时，将一个**嵌套对象**的字段"扁平化"到其父对象中

**示例**：

```JAVA
public class User {
    private String name;

    // "address" 对象内部的字段将被提升到 User 这一层
    @JsonUnwrapped
    private Address address;
}

class Address {
    private String street;
    private String city;
}

// --- 序列化 ---
// User user = ... (name="张三", address=new Address("街道", "城市"))
// mapper.writeValueAsString(user);
// 输出 (扁平化): {"name":"张三", "street":"街道", "city":"城市"}
// 默认 (不加注解): {"name":"张三", "address":{"street":"街道", "city":"城市"}}

// --- 反序列化 ---
// String json = "{\"name\":\"李四\", \"street\":\"新街道\", \"city\":\"新城市\"}";
// User user = mapper.readValue(json, User.class);
// user.getAddress() 会被正确地构建 (street="新街道", city="新城市")
```



#### `@JsonRawValue`

**核心功能**：**仅序列化**时，告诉 Jackson "这个字段的值已经是一个 JSON 字符串了，请你 **不要** 再对它进行转义，把它**原样**嵌入到输出中"

**说明**：这在你希望将一个预先序列化好的 JSON 片段嵌入到另一个 JSON 响应中时非常有用

**示例**：

```JAVA
public class Response {
    // 假设 jsonData 字段的值是字符串 "{\"key\":\"value\"}"
    @JsonRawValue
    private String jsonData;

    private int status;
}

// --- 序列化 ---
// Response res = ... (jsonData = "{\"key\":\"value\"}", status = 200)
// mapper.writeValueAsString(res);

// 启用 @JsonRawValue:
// {"jsonData":{"key":"value"}, "status":200}
// (jsonData 的值被当作 JSON 对象嵌入)

// 默认 (不加注解):
// {"jsonData":"{\"key\":\"value\"}", "status":200}
// (jsonData 的值被当作普通字符串转义)
```



### 4.3. 构造器注解

#### `@JsonCreator`

**核心功能**：**仅在反序列化时**，指定 Jackson 应该使用哪一个构造器（或静态工厂方法）来创建 POJO 实例

**关键用途**：

1. **反序列化不可变对象**：当你的类字段是 `final` 且没有 `setter` 方法时，这是**唯一**的反序列化方式
2. **指定多个构造器中的一个**：当一个类有多个构造器时，使用 `@JsonCreator` 告诉 Jackson 哪一个是用于 JSON 反序列化的

**重要规则**：

- `@JsonCreator` 标注的构造器，其所有参数**必须**使用 `@JsonProperty` 来显式标注，以便 Jackson 知道 JSON 中的 `key` 对应哪个构造器参数



**示例：反序列化不可变对象**

```JAVA
public class User {
    private final String name;  // final 字段
    private final Integer age; // final 字段

    // 1. @JsonCreator 告诉 Jackson 使用这个构造器
    @JsonCreator
    public User(
            // 2. @JsonProperty 将 JSON "name" 映射到 "name" 参数
            @JsonProperty("name") String name, 
            // 3. @JsonProperty 将 JSON "age" 映射到 "age" 参数
            @JsonProperty("age") Integer age) {

        this.name = name;
        this.age = age;
    }

    // 没有 setter 方法
    public String getName() { return name; }
    public Integer getAge() { return age; }
}

// String json = "{\"name\":\"张三\", \"age\":30}";
// User user = mapper.readValue(json, User.class); // 将成功调用构造器
```



### 4.4. 方法级别注解

#### `@JsonGetter` / `@JsonSetter`

**核心功能**：将注解标注在 `getter` 或 `setter` 方法上，其作用等同于在对应的字段上使用 `@JsonProperty`

**`@JsonGetter`**：

- (序列化) 将一个方法的**返回值**作为 JSON 的一个**key**的值
- **主要用途**：用于序列化**计算属性**（Computed Properties），即这个属性并非一个实际的字段，而是通过计算得来的

**`@JsonSetter`**：

- (反序列化) 将 JSON 中的一个**key**的值，通过该**setter 方法**注入
- **主要用途**：用于在反序列化时执行**自定义逻辑**（如拆分、转换）

**示例：计算 `fullName`**

```JAVA
public class User {
    private String firstName;
    private String lastName;

    // 1. @JsonGetter (序列化)
    // 序列化时，会额外生成一个 "fullName" 字段
    @JsonGetter("fullName")
    public String getFullName() {
        return firstName + " " + lastName;
    }

    // 2. @JsonSetter (反序列化)
    // 反序列化时，如果 JSON 中有 "fullName"，则调用此方法
    @JsonSetter("fullName")
    public void setFullName(String fullName) {
        // 在 setter 中执行自定义的拆分逻辑
        if (fullName != null) {
            String[] parts = fullName.split(" ");
            if (parts.length >= 2) {
                this.firstName = parts[0];
                this.lastName = parts[1];
            }
        }
    }

    // ... firstName 和 lastName 的常规 getter/setter ...
}
```

```JAVA
// --- 序列化 ---
// User user = new User("John", "Doe");
// mapper.writeValueAsString(user);
// 输出: {"firstName":"John", "lastName":"Doe", "fullName":"John Doe"}

// --- 反序列化 ---
// String json = "{\"fullName\":\"Peter Jones\"}";
// User user = mapper.readValue(json, User.class);
// user.getFirstName() -> "Peter"
// user.getLastName() -> "Jones"
```



#### `@JsonAnyGetter` / `@JsonAnySetter`

**核心功能**：用于处理**动态的、未知的 JSON 属性**

**`@JsonAnyGetter`**：

- (序列化) 标注在一个返回 `Map` 的方法上。Jackson 会将这个 Map 中的**所有键值对**"扁平化"地添加到 JSON 对象的顶层

**`@JsonAnySetter`**：

- (反序列化) 标注在一个接收 `(String key, Object value)` 参数的方法上

  当 Jackson 遇到 JSON 中**无法匹配**到任何已知字段（如 `name`）的属性时，会调用此方法，将未知的 `key` 和 `value` 传入



**示例：处理动态扩展字段**

```JAVA
public class ExtendableBean {

    // 已知字段
    private String name;

    // 用于存储所有未知字段的 Map
    private Map<String, Object> properties = new HashMap<>();

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    // 1. @JsonAnyGetter (序列化)
    // 序列化时，"properties" Map 里的内容会被展开
    @JsonAnyGetter
    public Map<String, Object> getProperties() {
        return properties;
    }

    // 2. @JsonAnySetter (反序列化)
    // 反序列化时，所有 "未知" 字段都会被丢进这个 Map
    @JsonAnySetter
    public void add(String key, Object value) {
        properties.put(key, value);
    }
}
```

```java
// --- 反序列化 (收集未知字段) ---
// String json = "{\"name\": \"张三\", \"age\": 25, \"city\": \"北京\"}";
// ExtendableBean bean = mapper.readValue(json, ExtendableBean.class);
// bean.getName() -> "张三"
// bean.getProperties() -> {age=25, city="北京"}

// --- 序列化 (展开 Map) ---
// bean.getProperties().put("title", "Engineer");
// mapper.writeValueAsString(bean);
// 输出: {"name":"张三", "age":25, "city":"北京", "title":"Engineer"}
```



### 4.5. 类型处理注解

#### `@JsonTypeInfo` / `@JsonSubTypes`

**核心功能**：在序列化时，向 JSON 中**添加类型信息**；在反序列化时，根据该类型信息**实例化正确的子类**

**`@JsonTypeInfo`** (标注在父类上)：用于定义**如何**存储类型信息

- `use = JsonTypeInfo.Id.NAME`：使用一个逻辑名称（如 "dog"）来标识类型，而不是完整的 Java 类名。
- `include = JsonTypeInfo.As.PROPERTY`：将类型信息作为 JSON 对象的一个**新属性**（key-value 对）存储。
- `property = "type"`：定义这个新属性的 **key** 名称为 "type"。

- **`@JsonSubTypes`** (与 `@JsonTypeInfo` 一起标注在父类上)：用于定义**逻辑名称**与**具体子类**之间的映射关系。



**示例：序列化和反序列化动物列表**

```JAVA
// 1. 在父类上定义类型处理规则
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,       // 策略：使用逻辑名称
    include = JsonTypeInfo.As.PROPERTY, // 方式：作为新属性
    property = "type"                 // 属性名："type"
)
// 2. 定义 "名称" 和 "子类" 的映射
@JsonSubTypes({
    @JsonSubTypes.Type(value = Dog.class, name = "dog"), // "dog" 字符串 ↔ Dog.class
    @JsonSubTypes.Type(value = Cat.class, name = "cat")  // "cat" 字符串 ↔ Cat.class
})
public abstract class Animal {
    private String name;
    // ... getter / setter ...
}

// 3. 定义子类 Dog
class Dog extends Animal {
    private String breed; // 品种
    // ... getter / setter ...
}

// 4. 定义子类 Cat
class Cat extends Animal {
    private Integer livesLeft; // 剩余生命
    // ... getter / setter ...
}
```

```JAVA
// --- 序列化 ---
// List<Animal> animals = new ArrayList<>();
// animals.add(new Dog("旺财", "哈士奇"));
// animals.add(new Cat("咪咪", 9));

// mapper.writeValueAsString(animals);
// 输出:
// [
//   {"type":"dog", "name":"旺财", "breed":"哈士奇"},
//   {"type":"cat", "name":"咪咪", "livesLeft":9}
// ]

// --- 反序列化 ---
// String json = "[{\"type\":\"dog\", ...}, {\"type\":\"cat\", ...}]";
// List<Animal> list = mapper.readValue(json, new TypeReference<List<Animal>>(){});
// Jackson 会根据 "type" 字段正确创建 Dog 和 Cat 实例
```



### 4.6. 值处理注解

#### `@JsonValue`

**核心功能**：**仅在序列化时**，告诉 Jackson **忽略对象本身**，转而调用**唯一**被此注解标注的方法，并**只**序列化该方法的**返回值**

**主要用途**：

- 常用于 `Enum`（枚举）或简单的包装类 (Wrapper Class)，使其序列化为你想要的特定值（如 `code`、`id`），而不是默认的字符串名称

**示例：序列化枚举为 `code` 值**

```JAVA
public enum Status {
    ACTIVE(1, "激活"),
    INACTIVE(0, "未激活");

    private Integer code;
    private String desc;

    Status(Integer code, String desc) {
        this.code = code;
        this.desc = desc;
    }

    // 序列化 Status.ACTIVE 时, Jackson 会调用此方法
    // 并将返回值 1 作为 JSON 的值
    @JsonValue
    public Integer getCode() {
        return code;
    }

    public String getDesc() {
        return desc;
    }
}
```

```JAVA
// class MyObject { public Status status = Status.ACTIVE; }
// mapper.writeValueAsString(new MyObject());
// 输出: {"status":1}
// (如果没有 @JsonValue, 默认输出: {"status":"ACTIVE"})
```



#### `@JsonSerialize` / `@JsonDeserialize`

**核心功能**：为某个字段**完全自定义**其序列化或反序列化的逻辑。这是 Jackson 中最底层的定制化工具

**`@JsonSerialize(using = ...)`**：指定一个 `JsonSerializer` 的子类，用于**序列化** (Java → JSON)

**`@JsonDeserialize(using = ...)`**：指定一个 `JsonDeserializer` 的子类，用于**反序列化** (JSON → Java)



**示例：自定义金额序列化 (保留两位小数的字符串)**

```JAVA
public class Order {

    // 序列化时，使用 MoneySerializer
    @JsonSerialize(using = MoneySerializer.class)
    private BigDecimal amount;

    // 反序列化时，使用 MoneyDeserializer
    @JsonDeserialize(using = MoneyDeserializer.class)
    private BigDecimal price;

    // ...
}

// 1. 自定义序列化器
// 将 BigDecimal 序列化为保留两位的字符串
public class MoneySerializer extends JsonSerializer<BigDecimal> {
    @Override
    public void serialize(BigDecimal value, JsonGenerator gen, 
                          SerializerProvider serializers) throws IOException {
        if (value == null) {
            gen.writeNull();
        } else {
            // 自定义逻辑：保留两位小数, 四舍五入, 转为字符串
            String formatted = value.setScale(2, RoundingMode.HALF_UP).toString();
            gen.writeString(formatted);
        }
    }
}

// 2. 自定义反序列化器 (示例)
// 将字符串转为 BigDecimal
public class MoneyDeserializer extends JsonDeserializer<BigDecimal> {
    @Override
    public BigDecimal deserialize(JsonParser p, 
                                  DeserializationContext ctxt) throws IOException {
        String text = p.getText();
        if (text == null || text.isEmpty()) {
            return null;
        }
        return new BigDecimal(text);
    }
}

// --- 序列化 ---
// Order order = ... (amount = new BigDecimal("123.4567"))
// mapper.writeValueAsString(order);
// 输出: {"amount":"123.46", "price":null}

// --- 反序列化 ---
// String json = "{\"amount\":\"99.99\", \"price\":\"50.00\"}";
// Order order = mapper.readValue(json, Order.class);
// order.getPrice() -> new BigDecimal("50.00")
```



### 4.7. 视图注解

#### `@JsonView`

**核心功能**：允许你定义**多个视图** (View)，并在序列化时**动态选择**只输出属于特定视图的字段

**主要用途**：当同一个 POJO (如 `User`) 需要在不同场景下返回不同子集的数据时

- **场景1 (Public API)**：只返回 `username`, `email` (公开信息)
- **场景2 (Internal Admin)**：返回 `username`, `email`, `password`, `ssn` (所有信息)

**使用步骤**：

**第1步：定义视图接口** 

- 创建一个类（如 `Views`），在其中定义多个**空的** `public static class` 作为视图标识

```JAVA
// 1. 定义视图
public class Views {
    // 视图 A: 公开视图
    public static class Public {}
    
    // 视图 B: 内部视图
    // (注意: Internal 继承了 Public)
    // 这意味着属于 "Internal" 视图的字段, 也自动属于 "Public"
    // [更正]：这个继承关系应该反过来， Internal 应该包含 Public 的所有内容
    // (更正2)：不，笔记是对的。Internal 继承 Public 意味着 "Internal 视图" 比 "Public 视图" 更广泛。
    // 让我们重新审视：
    // 如果 Internal extends Public，
    // 那么 @JsonView(Views.Public.class) 的字段，
    // 在使用 "Internal" 视图序列化时也会被包含。
    
    // [最佳实践]：让更详细的视图继承更基础的视图。
    // Internal 视图应该包含 Public 视图的所有字段。
    public static class Internal extends Public {}
}
```



**第2步：在字段上标注视图** 

- 使用 `@JsonView` 注解 POJO 的字段，指定该字段属于哪个（或哪些）视图

```JAVA
public class User {
    
    // "username" 字段属于 Public 视图
    @JsonView(Views.Public.class)
    private String username;
    
    // "email" 字段属于 Public 视图
    @JsonView(Views.Public.class)
    private String email;
    
    // "password" 字段只属于 Internal 视图
    @JsonView(Views.Internal.class)
    private String password;
    
    // "ssn" (社保号) 字段只属于 Internal 视图
    @JsonView(Views.Internal.class)
    private String ssn;
}
```



**第3步：序列化时指定视图** 

- 在序列化时，不能再使用 `mapper.writeValueAsString()`，而必须使用 `mapper.writerWithView()` 来指定要使用的视图

```JAVA
// 准备数据
User user = new User("user", "a@b.com", "pass123", "123-456");
ObjectMapper mapper = new ObjectMapper().enable(SerializationFeature.INDENT_OUTPUT);

// --- 场景1：使用 Public 视图序列化 ---
// (只包含 @JsonView(Views.Public.class) 的字段)
String publicJson = mapper.writerWithView(Views.Public.class)
                          .writeValueAsString(user);
// publicJson 输出:
// {
//   "username" : "user",
//   "email" : "a@b.com"
// }


// --- 场景2：使用 Internal 视图序列化 ---
// (包含 @JsonView(Views.Internal.class) 
//  和它继承的 @JsonView(Views.Public.class) 的字段)
String internalJson = mapper.writerWithView(Views.Internal.class)
                            .writeValueAsString(user);
// internalJson 输出:
// {
//   "username" : "user",
//   "email" : "a@b.com",
//   "password" : "pass123",
//   "ssn" : "123-456"
// }
```



## 5. Spring 集成

Jackson 是 Spring Boot 默认的 JSON 处理库

Spring Boot 通过自动配置，将 Jackson 深度集成到 Spring MVC 中，使其无缝地处理 HTTP 请求和响应中的 JSON 数据



### 5.1.  `MappingJackson2HttpMessageConverter`消息转换器

> 它是Spring中的东西
>
> 我之前在理解混淆的那段时间，我一直以为这个是Jackson中的一个工具，原来是Spring中的，现在思考清晰了，感觉职责分离的好明确啊，设计的真好

#### a. 基本概念

在 Spring MVC 中，`HttpMessageConverter` (消息转换器) 是一个接口，负责将 HTTP 请求体（如 JSON）转换为 Java 对象，以及将 Java 对象（返回值）转换为 HTTP 响应体

`MappingJackson2HttpMessageConverter` 就是这个接口的 Jackson 实现

`MappingJackson2HttpMessageConverter`内部 **持有**一个 `ObjectMapper` 实例，并调用该实例的 `readValue` 和 `writeValue` 方法来完成 JSON 和 Java 对象之间的转换，它内部的这个 `ObjectMapper`，是已经注册到 IOC 容器中的，通常我们会自己配置一个



#### b. **工作原理 (反序列化)**

1. 一个 HTTP POST/PUT 请求（`Content-Type: application/json`）到达服务器
2. Spring MVC 看到 `@RequestBody` 注解
3. 它选择 `MappingJackson2HttpMessageConverter`
4. 转换器调用 Jackson 的 `ObjectMapper.readValue(request.getInputStream(), User.class)` 将 JSON 转换为 `User` 对象



#### c. 工作原理 (序列化)

1. 一个 Controller 方法返回一个 Java 对象（如 `User`）
2. Spring MVC 看到 `@ResponseBody` 注解（或 `@RestController`，它自带 `@ResponseBody`）
3. 它选择 `MappingJackson2HttpMessageConverter`
4. 转换器调用 Jackson 的 `ObjectMapper.writeValue(response.getOutputStream(), user)` 将 `User` 对象序列化为 JSON 字符串并写入响应



#### d. 代码体现 (controller)

你完全不需要关心 `HttpMessageConverter` 的存在，只需要使用注解：

```JAVA
@RestController
public class UserController {

    // 1. @RequestBody 触发【反序列化】
    // MappingJackson2HttpMessageConverter 将请求体 JSON 转为 User 对象
    @PostMapping("/users")
    public Result createUser(@RequestBody User user) {
        // ... 业务逻辑 ...
        return Result.success(user);
    }

    // 2. @ResponseBody (来自 @RestController) 触发【序列化】
    // MappingJackson2HttpMessageConverter 将返回的 User 对象转为 JSON 响应
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        User user = userService.getById(id);
        return user; 
    }
}
```



#### e. 在 Spring Boot 中自定义转换器

**推荐方式**：

- 如果你想修改 Jackson 的行为（例如，添加 `NON_NULL` 配置），你只需要**替换 Spring Boot 自动配置的那个 `ObjectMapper`** 即可

- Spring Boot 的自动配置非常智能：
  - **如果你在配置类中定义了一个 `@Bean` 且类型为 `ObjectMapper`，Spring Boot 将会使用你定义的这个 Bean，而不是它自己创建的默认 Bean**

- 示例

  ```java
  /**
   * Jackson ObjectMapper 的全局配置
   */
  @Configuration
  public class JacksonConfig {
  
      @Bean
      @Primary // 确保 Spring 优先使用这个自定义的 ObjectMapper
      public ObjectMapper objectMapper() {
          ObjectMapper mapper = new ObjectMapper();
  
          // === 核心配置：健壮性与现代 Java 支持 ===
  
          // 1. 注册 Java 8 时间模块 (必备)
          // 支持 LocalDate, LocalDateTime 等
          mapper.registerModule(new JavaTimeModule());
          
          // 2. 禁用：将日期序列化为时间戳 (配合 JavaTimeModule)
          // 序列化为 "2025-11-11T12:00:00" 而不是 1678886400000
          mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
  
          // 3. 禁用：反序列化时, 忽略 JSON 中存在而 Java 类中不存在的属性
          // 这是增强 API 健壮性的关键配置，防止上游增加字段导致服务崩溃
          mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
  
          // === 推荐配置：优化 JSON 输出 ===
  
          // 4. 配置：序列化时, 仅包含非 null 值的字段
          // (可选: NON_EMPTY 包含 null, "" 和空集合)
          // 这可以显著减小 JSON 响应体的大小
          mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
          
          // 5. 禁用：序列化空 POJO (没有字段) 时不抛异常
          // 默认会抛异常，禁用后会序列化为 {}
          mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS);
          
          // === 可选配置：根据团队规范选择 ===
          
          // 6. (可选) 统一命名策略：驼峰转下划线
          // Java: "userName" <--> JSON: "user_name"
          // mapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);
          
          // 7. (可选) 美化输出 (仅用于开发/调试)
          // 在生产环境应关闭此选项以节省带宽
          // mapper.enable(SerializationFeature.INDENT_OUTPUT);
  
          return mapper;
      }
  }
  ```

  - 这个被定制过的 `ObjectMapper` Bean 会被自动注入到 `MappingJackson2HttpMessageConverter` 中，从而影响所有的 `@RequestBody` 和 `@ResponseBody`



#### f. (高级) 完全覆盖：使用 `WebMvcConfigurer`

如果你有非常特殊的需求（例如，你需要配置**多个**不同行为的 `HttpMessageConverter`），你可以完全覆盖 Spring Boot 的默认设置

> 这部分具体实现可见 SpringBoot 的笔记中



### 5.2. `Jackson2ObjectMapperBuilder`⭐

Spring Boot 还提供了另一种更“原生”、更推荐的定制方式：

- 在这里，我们**不**直接创建 `ObjectMapper` 的 `@Bean` ，而是创建一个 `Jackson2ObjectMapperBuilder`  的 `@Bean`



**核心思想**：

1. **不**去 `new ObjectMapper()`
2. 创建并配置一个 `Jackson2ObjectMapperBuilder` 的 Bean
3. Spring Boot 会 **自动检测** 到这个 `Builder` Bean
4. Spring Boot 会使用 **你配置的这个 `Builder`** 来**创建**那个全局的、单例的 `ObjectMapper`



**工作流程**：

-  `@Bean Jackson2ObjectMapperBuilder` → Spring Boot 接收 → Spring Boot 用它 `build()` → 生成全局 `ObjectMapper` → 注入 `HttpMessageConverter`



**配置更简洁**：`Builder` 提供了链式调用的 API，比直接操作 `ObjectMapper` 更清晰



**配置示例**： 这是在 Spring Boot 中定制 Jackson 的**首选方式之一**

```java
@Configuration
public class JacksonConfig {

    @Bean
    public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder() {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();

        // 1. 自定义配置

        // 序列化时, 仅包含非 null 字段
        builder.serializationInclusion(JsonInclude.Include.NON_NULL);

        // 反序列化时, 忽略未知属性
        // (注意: .failOnUnknownProperties(false) 是前文
        // .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES) 的等效写法)
        builder.failOnUnknownProperties(false);

        // 序列化空对象 (POJO) 时不抛异常
        builder.failOnEmptyBeans(false);

        // 属性命名策略: Java 驼峰 ↔ JSON 下划线
        builder.propertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);

        // 格式化输出 (美化 JSON)
        builder.indentOutput(true);

        // 时区和日期格式
        // [注意]：如果设置了 .dateFormat()，它会覆盖 JavaTimeModule (JSR-310)
        //          对 LocalDateTime 等新时间类型的序列化。
        //          通常我们不设置这个，而是依赖 JavaTimeModule 的默认 ISO 格式。
        // builder.dateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        // builder.timeZone(TimeZone.getTimeZone("GMT+8"));

        // 2. 自动配置
        // Spring Boot 会自动为你添加 JavaTimeModule, ParameterNamesModule 等。
        // 你也可以手动添加:
        // builder.modules(new JavaTimeModule());

        return builder;
    }
}
```



### 5.3. Jackson 与 Redis 序列化

#### 概述

默认情况下, `RedisTemplate<String, Object>` 使用 Java 的 `JdkSerializationRedisSerializer` 来序列化 Value

这会将 Java 对象序列化为一长串不可读的字节，存在以下问题：

1. **可读性差**：无法在 Redis 客户端中直观地查看和修改数据
2. **空间占用大**：序列化后的字节体积远大于 JSON
3. **跨语言问题**：其他编程语言无法解析 Java 的序列化字节

**最佳实践**：使用 Jackson 将对象序列化为 JSON 字符串存储在 Redis 中，以解决上述所有问题



#### 反序列化时的类型信息

当你使用 `redisTemplate.opsForValue().set("user:1", user)` 存储一个 `User` 对象时，它被转为 JSON 存入

但当你 `get` 时，`redisTemplate.opsForValue().get("user:1")` 返回的是 `Object` 类型
`RedisTemplate` **不知道**这个 JSON 字符串应该被反序列化回 `User` 类还是 `Product` 类

为了解决这个问题，序列化器**必须**在 JSON 中额外存储类型信息（例如，添加一个 `@class` 字段）



#### 两种序列化配置方式

这里先提及一个概念：`RedisConnectionFactory` 是 Spring Data Redis 框架中的一个**核心接口**

- 它的唯一职责就是：**创建和管理与 Redis 服务器之间的物理网络连接**

```java
//下面这两行代码完成了 RedisTemplate Bean 的创建和 连接工厂 的装配
//RedisTemplate必须持有 RedisConnectionFactory 才能与 Redis 服务器通信
RedisTemplate<String, Object> template = new RedisTemplate<>();
template.setConnectionFactory(factory);
```



##### 方式1:`Jackson2JsonRedisSerializer`

> 手动配置

这是"老"方法，需要你 **手动配置** `ObjectMapper` 来开启"默认类型" (`Default Typing`)

```java
// 这是你笔记中的第一个示例
@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);

        // 1. 创建 JSON 序列化器
        Jackson2JsonRedisSerializer<Object> serializer = 
            new Jackson2JsonRedisSerializer<>(Object.class);

        // 2. 创建并配置 ObjectMapper
        ObjectMapper mapper = new ObjectMapper();
        
        // [关键] 允许 Jackson 访问所有字段 (包括 private)
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        
        // [核心] 开启默认类型。
        // 这会使 Jackson 在序列化时向 JSON 中添加一个 "@class" 属性，
        // 值为类的全限定名, e.g., "com.example.User"。
        // 反序列化时, Jackson 会读取这个 "@class" 属性来确定目标类。
        mapper.activateDefaultTyping(
            mapper.getPolymorphicTypeValidator(),
            ObjectMapper.DefaultTyping.NON_FINAL,
            JsonTypeInfo.As.PROPERTY
        );
        // 注册 Java 8 时间模块 (必备)
        // 支持 LocalDate, LocalDateTime 等
        serializer.setObjectMapper(mapper);
        serializer.setObjectMapper(mapper);

        // 3. 设置 Key/Value/HashKey/HashValue 的序列化器
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);
        
        //用于在所有属性设置完成后进行初始化检查和默认值设置，手动创建Bean时,需要手动调用
        //Spring自动管理时，它由Spring自动调用
        template.afterPropertiesSet();
        return template;
    }
}
```

- **缺点**：

  - 配置极其繁琐，

    且 `activateDefaultTyping` 如果配置不当（例如使用 `ObjectMapper.DefaultTyping.EVERYTHING`）可能存在安全漏洞



##### 方式2:`GenericJackson2JsonRedisSerializer` (推荐)

`spring-data-redis` 提供了这个类，它是一个 **预先配置好** 的、**更安全** 的 `Jackson2JsonRedisSerializer`

它在内部**自动处理了类型信息的添加**，你不需要关心 `ObjectMapper` 的复杂配置

```java
// Spring Boot 环境下的【最佳实践】
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(factory);

        // 1. 使用【通用】JSON 序列化器
        // 这个序列化器在内部已经配置好了, 会自动添加 "@class" 属性
        GenericJackson2JsonRedisSerializer serializer = 
            new GenericJackson2JsonRedisSerializer();
        
        //内部已经默认注册了 Java 8 时间模块

        // 2. 设置序列化器
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);
        
        //用于在所有属性设置完成后进行初始化检查和默认值设置，手动创建Bean时,需要手动调用
        //Spring自动管理时，它由Spring自动调用
        template.afterPropertiesSet();
        return template;
    }
}
```

- **结论**：始终优先使用 `GenericJackson2JsonRedisSerializer`，它简单、安全且能正确处理类型信息





### 5.4. 全局异常处理

当 Jackson 在 Spring MVC 流程中（即处理 `@RequestBody`）失败时，它会抛出异常

我们必须在全局捕获这些异常，以防止向用户显示不友好的错误页面或原始 JSON 错误

`@RestControllerAdvice` (或 `@ControllerAdvice` + `@ResponseBody`) 是 Spring 推荐的标准做法



###  5.5. application.yml 声明式配置

除了使用 `@Bean` 进行编程式配置外，Spring Boot 还提供了第三种，也是最简单的配置方式：在 `application.yml` (或 `application.properties`) 中直接声明

- **工作原理**：
  -  Spring Boot 在自动配置 `Jackson2ObjectMapperBuilder` 时，会读取 `spring.jackson` 命名空间下的所有属性，并将它们设置到**默认**的 `Builder` 中



#### a. 常用配置示例

```java
spring:
  jackson:
    # 1. 默认包含策略 (最常用)
    # 对应: mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL)
    default-property-inclusion: non_null

    # 2. 属性命名策略
    # 对应: mapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE)
    property-naming-strategy: SNAKE_CASE

    # 3. 日期格式
    # [注意]: 这会同时影响 Date 和 Java 8 的时间类型
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8

    # 4. 序列化 (SerializationFeature)
    serialization:
      # 格式化输出 (INDENT_OUTPUT: true)
      indent-output: true
      # 日期不使用时间戳 (WRITE_DATES_AS_TIMESTAMPS: false)
      write-dates-as-timestamps: false
      # 空 POJO 不报错 (FAIL_ON_EMPTY_BEANS: false)
      fail-on-empty-beans: false

    # 5. 反序列化 (DeserializationFeature)
    deserialization:
      # 忽略未知属性 (FAIL_ON_UNKNOWN_PROPERTIES: false)
      fail-on-unknown-properties: false

    # 6. 解析器特性 (JsonParser.Feature)
    parser:
      # 允许 JSON 中带注释 (ALLOW_COMMENTS: true)
      allow-comments: true
      # 允许使用单引号 (ALLOW_SINGLE_QUOTES: true)
      allow-single-quotes: true

    # 7. 生成器特性 (JsonGenerator.Feature)
    # generator:
    #   write-numbers-as-strings: true # 示例：数字转字符串
```



#### 配置优先级

**高优先级 (覆盖一切)** ↓ **低优先级 (被覆盖)**

1. **`@Bean` - `ObjectMapper` (最高)**

   - **方式**：`public ObjectMapper objectMapper() { ... }`

   - **效果**：**完全接管**

     - Spring Boot 的所有 Jackson 自动配置（包括 `yml`）**全部失效**

       你需要手动配置所有东西（如 `JavaTimeModule`）

   

2. **`@Bean` - `Jackson2ObjectMapperBuilder`**

   - **方式**：`public Jackson2ObjectMapperBuilder builder() { ... }`

   - **效果**：**部分接管**

     `application.yml` 中的配置**仍然会**被加载，但**会被你的 `Builder` Bean 中的配置所覆盖**

     这是 `Builder` 和 `yml` 协同工作的方式

   

3. **`application.yml` (最低)**

   - **方式**：`spring.jackson.default-property-inclusion: non_null`
   - **效果**：
     - **仅当**你**没有**提供上述任何 `@Bean` 时，`yml` 中的配置才会**完全生效**，用于配置 Spring Boot **默认**的 `Jackson2ObjectMapperBuilder`



## 6. 配置项参考手册

### 简述

Jackson 核心配置项的速查手册。每一个特性我都会展示其在三种不同配置方式下的等价写法：

1. **[编程式] **

   - **`ObjectMapper`**：

     - `mapper.configure(Feature, state)` 

       或 `mapper.enable(Feature)` / `mapper.disable(Feature)`

2. **[构建器] **

   - **`Builder`**：`builder.feature(...)`

3. **[声明式] **

   - **`yml`**：`spring.jackson.serialization.*`



⭐ 这里原本想整理一下，但是又意识到这里应该**利用 AI 工具**在使用中边用边查，这样子最好，就没整理，但是下面还是简单列举了列举



### SerializationFeature（序列化特性）

```java
// 日期时间
WRITE_DATES_AS_TIMESTAMPS          // 日期写为时间戳（默认 true）
WRITE_DATE_KEYS_AS_TIMESTAMPS      // Map 的日期 key 写为时间戳
WRITE_DATES_WITH_ZONE_ID          // 包含时区信息

// 枚举
WRITE_ENUMS_USING_TO_STRING       // 枚举使用 toString()
WRITE_ENUMS_USING_INDEX           // 枚举使用索引

// 数字
WRITE_NUMBERS_AS_STRINGS          // 数字写为字符串
WRITE_BIGDECIMAL_AS_PLAIN         // BigDecimal 不使用科学计数法

// 集合
WRITE_EMPTY_JSON_ARRAYS           // 空数组写为 []
WRITE_SINGLE_ELEM_ARRAYS_UNWRAPPED // 单元素数组展开

// 其他
INDENT_OUTPUT                      // 美化输出（格式化）
FAIL_ON_EMPTY_BEANS               // 空对象抛异常（默认 true）
WRAP_ROOT_VALUE                   // 添加根节点
CLOSE_CLOSEABLE                   // 自动关闭 Closeable 对象
FLUSH_AFTER_WRITE_VALUE           // 写入后刷新
```



### DeserializationFeature（反序列化特性）

```java
// 未知属性
FAIL_ON_UNKNOWN_PROPERTIES        // 遇到未知属性抛异常（默认 true）
FAIL_ON_IGNORED_PROPERTIES        // 遇到忽略属性抛异常

// 空值处理
ACCEPT_EMPTY_STRING_AS_NULL_OBJECT    // 空字符串转 null
ACCEPT_EMPTY_ARRAY_AS_NULL_OBJECT     // 空数组转 null

// 数字处理
ACCEPT_FLOAT_AS_INT               // 浮点数可以转整数
ACCEPT_SINGLE_VALUE_AS_ARRAY      // 单个值可以转数组

// 枚举
READ_ENUMS_USING_TO_STRING        // 使用 toString() 匹配枚举
READ_UNKNOWN_ENUM_VALUES_AS_NULL  // 未知枚举值转 null

// 日期
ADJUST_DATES_TO_CONTEXT_TIME_ZONE // 调整日期到上下文时区

// 其他
FAIL_ON_NULL_FOR_PRIMITIVES       // 基本类型为 null 时抛异常
FAIL_ON_NUMBERS_FOR_ENUMS         // 数字转枚举失败时抛异常
UNWRAP_ROOT_VALUE                 // 解开根节点
```



### JsonInclude.Include（属性包含策略）

```java
ALWAYS          // 总是包含（默认）
NON_NULL        // 非 null
NON_ABSENT      // 非 null 且非 Optional.empty
NON_EMPTY       // 非空（null、空字符串、空集合、空数组）
NON_DEFAULT     // 非默认值
CUSTOM          // 自定义过滤器
USE_DEFAULTS    // 使用默认配置
```



### PropertyNamingStrategies（属性命名策略）

```java
LOWER_CAMEL_CASE      // userName (默认)
UPPER_CAMEL_CASE      // UserName (Pascal)
SNAKE_CASE            // user_name
UPPER_SNAKE_CASE      // USER_NAME
LOWER_CASE            // username
KEBAB_CASE            // user-name
LOWER_DOT_CASE        // user.name
```

------



## 7. 最佳实践

### 1. 单例 `ObjectMapper`

这是 Jackson 的**第一条黄金法则**。`ObjectMapper` 是**线程安全**的（前提是配置完成后不再修改），但创建它的开销非常大

- **❌ 错误做法**：在需要时 `new ObjectMapper()`。这会严重影响性能，因为每次创建都需要 JRE 查找并加载所有注册的模块
- **✅ 正确做法**：在 Spring 容器中注册为**全局单例**，并在所有地方 `@Autowired` 注入这同一个实例

```java
// ✅ 推荐：在 @Configuration 中将其注册为全局单例
@Configuration
public class JacksonConfig {
    
    @Bean
    @Primary // 确保 Spring 优先使用这个自定义的 mapper
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        // ... 在这里进行所有配置 ...
        return mapper;
    }
}

// ❌ 避免：在业务逻辑中即时创建
@Service
public class MyService {
    public void someMethod(Object data) {
        // 极差的性能！
        ObjectMapper localMapper = new ObjectMapper(); 
        // String json = localMapper.writeValueAsString(data);
    }
}
```



### 2. 全局异常处理

API 的健壮性体现在它如何处理"坏"数据。永远不要让 Jackson 的解析异常（`JsonParseException` 或其父类）直接抛给前端用户

- **实践**：使用 `@RestControllerAdvice` 捕获 Jackson 在反序列化（`@RequestBody`）时最常抛出的两个异常

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    /**
     * 1. 处理 JSON 格式错误、类型不匹配
     * (例如：JSON 语法错、"age": "abc")
     */
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public Result<?> handleJsonParseError(HttpMessageNotReadableException e) {
        // log.error("JSON 解析错误: {}", e.getMessage());
        return Result.error(400, "请求参数格式错误或类型不匹配");
    }

    /**
     * 2. 处理 JSR-303 Validation 校验失败
     * (例如：@NotBlank, @NotNull, @Email 校验失败)
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<?> handleValidationError(MethodArgumentNotValidException e) {
        // String errorMsg = e.getBindingResult().getFieldErrors()... (提取错误)
        // log.error("参数校验失败: {}", errorMsg);
        return Result.error(400, "参数校验失败，请检查输入");
    }
}

```



### 3. 统一响应格式

这是 API 设计的最佳实践，Jackson 注解在这里发挥着关键作用，确保所有 API 的 JSON 响应结构一致、可预测且干净

```java
/**
 * 全局统一 API 响应类
 * @param <T> data 字段的泛型
 */
// @Data // from lombok

// 核心注解：仅序列化非 null 字段
@JsonInclude(JsonInclude.Include.NON_NULL) 
public class Result<T> {
    
    // 明确指定 JSON 字段名
    @JsonProperty("code") 
    private Integer code;
    
    @JsonProperty("message")
    private String message;
    
    // data 字段：当 T 为 null 时，此字段不参与序列化
    @JsonProperty("data")
    private T data;
    
    // 格式化时间戳，确保输出为标准字符串
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss") 
    private LocalDateTime timestamp = LocalDateTime.now();
    
    // (构造函数、静态工厂方法...)

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        result.setData(data);
        return result;
    }
    
    public static <T> Result<T> error(Integer code, String message) {
        Result<T> result = new Result<>();
        result.setCode(code);
        result.setMessage(message);
        return result;
    }
}
```





### 4. 性能与资源管理

`ObjectMapper` 的创建成本很高，而 `TypeReference` 每次 `new` 也会有开销

```java
// ✅ 推荐 1: 缓存 TypeReference
// 每次 new TypeReference(){} 都会创建一个匿名内部类实例，在高性能场景下应避免。
// 将其定义为静态常量 (static final) 以复用。
private static final TypeReference<List<User>> USER_LIST_TYPE = new TypeReference<List<User>>() {};

// 在业务方法中使用缓存的 Type
List<User> users = mapper.readValue(json, USER_LIST_TYPE);


// ✅ 推荐 2: 注入并复用 ObjectMapper (见 7.1 节)
// 绝不在方法内部 `new ObjectMapper()`。


// ✅ 推荐 3: 流式 API 处理超大 JSON 文件
// 当 JSON 文件达到 GB 级别时，`readValue()` 或 `readTree()` 会导致内存溢出 (OOM)。
// 此时必须使用 jackson-core 提供的流式 API (JsonParser) 逐个 Token 读取和处理。
public void processLargeJsonFile(File largeFile) throws IOException {
    // 1. 从 ObjectMapper (或 JsonFactory) 获取 JsonParser
    // 2. 确保 ObjectMapper 实例是被注入的，而不是 new 的
    @Autowired
    private ObjectMapper mapper; 
    
    try (JsonParser parser = mapper.getFactory().createParser(largeFile)) {
        // 假设 JSON 结构是: { "data": [ ...海量 User 对象... ] }
        
        // 移动到 "data" 字段
        while (parser.nextToken() != JsonToken.FIELD_NAME || !"data".equals(parser.getCurrentName())) {
            // 寻找 data 字段
        }
        
        // 移动到数组的开始 `[`
        if (parser.nextToken() != JsonToken.START_ARRAY) {
            throw new IOException("Expected an array for 'data' field");
        }

        // 循环读取数组中的每个对象 `User`
        while (parser.nextToken() == JsonToken.START_OBJECT) {
            // 使用 readValueAs 将当前的 Token 指针解析为一个 User 对象
            User user = parser.readValueAs(User.class);
            
            // 逐条处理 user 对象，而不是全部加载到内存
            userService.processUser(user);
        }
        
        // 循环结束，parser.currentToken() 应该是 END_ARRAY `]`
    }
}
```



### 5. Java 8 日期时间处理

Jackson 默认不直接支持 Java 8 的 `LocalDateTime`、`LocalDate` 等，需要注册 `jackson-datatype-jsr310` 模块。

**强烈推荐**在 Spring Boot 中通过 `@Bean` 来配置 `JavaTimeModule`，而不是使用 `@JsonFormat` 注解，这样更易于全局统一管理

```java
@Configuration
public class JacksonConfig {

    // 推荐的日期格式
    private static final String DATETIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    private static final String DATE_FORMAT = "yyyy-MM-dd";
    private static final String TIME_FORMAT = "HH:mm:ss";

    @Bean
    public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder() {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();
        
        // 1. 创建 JavaTimeModule
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        
        // 2. (可选) 为特定类型自定义格式化器
        // 如果不自定义，模块默认使用 ISO 格式 (例如: 2025-11-11T15:30:00)
        javaTimeModule.addSerializer(LocalDateTime.class, 
            new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DATETIME_FORMAT)));
        javaTimeModule.addSerializer(LocalDate.class, 
            new LocalDateSerializer(DateTimeFormatter.ofPattern(DATE_FORMAT)));
        javaTimeModule.addSerializer(LocalTime.class, 
            new LocalTimeSerializer(DateTimeFormatter.ofPattern(TIME_FORMAT)));

        javaTimeModule.addDeserializer(LocalDateTime.class, 
            new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DATETIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDate.class, 
            new LocalDateDeserializer(DateTimeFormatter.ofPattern(DATE_FORMAT)));
        javaTimeModule.addDeserializer(LocalTime.class, 
            new LocalTimeDeserializer(DateTimeFormatter.ofPattern(TIME_FORMAT)));
            
        // 3. 注册模块
        builder.modules(javaTimeModule);
        
        // 4. 禁用将日期序列化为时间戳 (默认行为)
        builder.featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

        return builder;
    }
    
    // Spring Boot 会自动使用上面的 Builder 来构建一个 ObjectMapper Bean
    // 如果你没有使用 Builder，而是直接提供了 ObjectMapper Bean，做法如下：
    /*
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        
        // (配置 JavaTimeModule 的代码同上...)
        
        mapper.registerModule(javaTimeModule);
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        return mapper;
    }
    */
}
```



### 6. 枚举处理

默认情况下，Jackson 使用枚举的 `name()` (即枚举项的名称，如 "ACTIVE") 进行序列化

在实践中，我们通常希望使用自定义的 `code` (如 1) 或 `desc` (如 "激活")

最佳实践是使用 `@JsonValue` (序列化) 和 `@JsonCreator` (反序列化) 组合

```java
@Getter
@AllArgsConstructor // Lombok: 自动生成包含所有参数的构造器
public enum UserStatus {
    
    ACTIVE(1, "激活"),
    INACTIVE(0, "未激活"),
    BANNED(-1, "封禁");
    
    /**
     * @JsonValue
     * 核心功能：告诉 Jackson，在序列化此枚举实例时，
     * 不要使用默认的 name()，而是调用此方法，并使用其返回值。
     */
    @JsonValue
    private final Integer code;
    
    private final String desc;
    
    /**
     * @JsonCreator
     * 核心功能：告诉 Jackson，在反序列化时 (例如从 JSON 中读取到数字 1)，
     * 调用此静态工厂方法来构造枚举实例。
     */
    @JsonCreator
    public static UserStatus fromCode(Integer code) {
        if (code == null) {
            return null;
        }
        for (UserStatus status : values()) {
            if (status.code.equals(code)) {
                return status;
            }
        }
        throw new IllegalArgumentException("未知的状态码: " + code);
    }
    
    // (可选) 如果你想根据 desc ("激活") 来反序列化
    /*
    @JsonCreator
    public static UserStatus fromDesc(String desc) {
        if (desc == null) { return null; }
        for (UserStatus status : values()) {
            if (status.desc.equals(desc)) {
                return status;
            }
        }
        throw new IllegalArgumentException("未知的状态描述: " + desc);
    }
    */
}
```

```java
// === 使用效果 ===
// 序列化 new User(UserStatus.ACTIVE) -> JSON: { ..., "status": 1 }
// 反序列化 JSON: { "status": 1 } -> Java: User.status 值为 UserStatus.ACTIVE
```



### 7. 泛型处理

Jackson 在反序列化泛型（如 `List<User>` 或 `Result<User>`）时，会因为 Java 的**类型擦除**而丢失泛型信息，导致其可能将 `data` 反序列化为 `LinkedHashMap` 而不是 `User`

**`TypeReference`** 是 Jackson 解决此问题的标准方案。`JavaType` 是更底层、更强大的替代方案

```java
/**
 * 封装一个基于 Spring Boot 自动配置的 ObjectMapper 的工具类
 * 注意：这里不再 static final new ObjectMapper()，而是通过注入获取。
 */
@Component // 注册为 Spring Bean
public class JsonUtils {

    private static ObjectMapper mapper;

    // 通过构造器注入 Spring Boot 配置好的单例 ObjectMapper
    @Autowired
    public JsonUtils(ObjectMapper objectMapper) {
        JsonUtils.mapper = objectMapper;
    }

    // ... (其他方法如 writeValueAsString) ...

    /**
     * 反序列化为简单对象
     */
    public static <T> T parseObject(String json, Class<T> clazz) {
        try {
            return mapper.readValue(json, clazz);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("JSON 解析失败", e);
        }
    }

    /**
     * 反序列化为 List (使用 JavaType)
     * 这是 TypeReference 的替代方案，在构建泛型工具类时更灵活。
     */
    public static <T> List<T> parseArray(String json, Class<T> elementClass) {
        try {
            // 1. 获取类型工厂
            TypeFactory typeFactory = mapper.getTypeFactory();
            // 2. 构建集合类型
            JavaType collectionType = typeFactory.constructCollectionType(List.class, elementClass);
            // 3. 反序列化
            return mapper.readValue(json, collectionType);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("JSON 数组解析失败", e);
        }
    }

    /**
     * 反序列化为复杂泛型 (TypeReference)
     * 例如：Result<User>, Map<String, List<User>>
     *
     * 用法:
     * Result<User> result = JsonUtils.parseObject(json, new TypeReference<Result<User>>() {});
     */
    public static <T> T parseObject(String json, TypeReference<T> typeReference) {
        try {
            return mapper.readValue(json, typeReference);
        } catch (JsonProcessingException e) {
            throw new RuntimeException("JSON 泛型解析失败", e);
        }
    }
}
```



------



## 8. 常见问题

### 1. Long 精度丢失

JavaScript 的 `Number` 遵循 IEEE 754 双精度浮点数标准，其能**安全**表示的最大整数是 `Number.MAX_SAFE_INTEGER` (即 2^53 - 1, 约 9007 万亿)

当一个 Long 值超过这个数时，JavaScript 的 `Number` 类型无法精确表示它，导致精度丢失

**解决方案（推荐：全局配置）：** 最好的办法是将所有 `Long` 类型（包括 `long`）在序列化时都转换为 JSON **字符串**

**全局配置**：这是最推荐的一劳永逸的方案

通过注册一个 `SimpleModule`，将 `Long` 和 `long` 类型的序列化器指定为 `ToStringSerializer`

```java
@Configuration
public class JacksonConfig {

    @Bean
    public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder() {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();
        
        // 创建一个模块
        SimpleModule longModule = new SimpleModule("LongToStringModule");
        
        // 添加 Long 和 long 的序列化器
        longModule.addSerializer(Long.class, ToStringSerializer.instance);
        longModule.addSerializer(Long.TYPE, ToStringSerializer.instance); // Long.TYPE 即 long.class
        
        // 注册模块
        builder.modules(longModule);
        
        return builder;
    }
}
```

**或者**

```java
@Bean
public ObjectMapper objectMapper() {
    ObjectMapper mapper = new ObjectMapper();
    SimpleModule longModule = new SimpleModule();
    longModule.addSerializer(Long.class, ToStringSerializer.instance);
    longModule.addSerializer(Long.TYPE, ToStringSerializer.instance);
    mapper.registerModule(longModule);
    return mapper;
}
```



**局部注解 (不推荐)** 只在特定字段上使用 `@JsonFormat`。这只适用于你知道某个特定 ID 可能会超长的情况，但维护性差

```java
public class User {
    /**
     * @JsonFormat(shape = JsonFormat.Shape.STRING)
     * 告诉 Jackson 将这个字段序列化为 JSON 字符串，而不是 JSON 数字。
     */
    @JsonFormat(shape = JsonFormat.Shape.STRING)
    private Long id;
    
    private Long otherId; // 这个 ID 依然是数字，可能丢失精度
}
```



### 2. 日期格式问题

现象：

- `java.util.Date` 默认被序列化为了一个**数字时间戳**（如 `1678886400000`）

  或

  `java.time.LocalDateTime` (JSR-310) 默认被序列化为**ISO 格式**（如 `[2023, 3, 15, 12, 0, 0]` 或 `2023-03-15T12:00:00`）

  - 这都不是前端想要的 "yyyy-MM-dd HH:mm:ss"

**原因：** 这是 Jackson 的默认配置。

- 对于 `Date`，`SerializationFeature.WRITE_DATES_AS_TIMESTAMPS` 默认 `true`
- 对于 `LocalDateTime`，需要 `jackson-datatype-jsr310` 模块，且该模块默认使用 ISO 标准格式

**解决方案（推荐：全局配置）**

**全局配置**

```java
@Configuration
public class JacksonConfig {
    @Bean
    public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder() {
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        
        // 为关键的 Java 8 时间类型设置统一格式
        javaTimeModule.addSerializer(LocalDateTime.class, 
            new LocalDateTimeSerializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
        javaTimeModule.addDeserializer(LocalDateTime.class, 
            new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
            
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder();
        builder.modules(javaTimeModule);
        builder.featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS); // 顺便禁用 Date 的时间戳
        return builder;
    }
}
```

**局部配置**

```java
// 方案2：局部注解 (仅用于临时覆盖)
public class Order {
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private LocalDateTime createTime; // 注意：LocalDateTime 默认不带时区，配置 timezone 可能引发偏移
    
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date legacyTime; // Date 必须指定 timezone
}
```



### 3. 空值处理

**现象：** `{ "name": "张三", "email": null, "tags": [] }`。`email` 为 `null` 和 `tags` 为空集合 `[]` 都在 JSON 中显示了，希望将它们隐藏

**原因：** Jackson 的默认包含策略 `JsonInclude.Include.ALWAYS` 决定了它会序列化所有字段

**解决方案（推荐：全局配置）：**

**1. 全局配置 (application.yml)** 最简单的 Spring Boot 方式：

```java
spring:
  jackson:
    # 推荐：NON_NULL (只忽略 null) 或 NON_EMPTY (忽略 null、空字符串、空集合/Map)
    default-property-inclusion: non_null 
```

**2. 全局配置 (@Bean)**

```java
@Bean
public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder() {
    // 效果等同于 yml 中的 non_null
    builder.serializationInclusion(JsonInclude.Include.NON_NULL);
    return builder;
}
```

**3. 类级别注解** 覆盖全局配置，只对 `User` 类生效。

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class User { ... }
```

**4. 字段级别注解** 最高优先级，只对 `description` 字段生效。

```java
public class User {
    @JsonInclude(JsonInclude.Include.NON_NULL)
    private String description;
}
```



### 4. 循环依赖

**现象：** 序列化 `User` 对象时抛出 `StackOverflowError` 异常

**原因：** 两个对象之间存在**双向引用**

- 例如：`User` 拥有一个 `Order` 列表 (`List<Order>`)；同时，`Order` 对象又持有对 `User` 的引用 (`User user`)

  - Jackson 序列化时

     `User` -> 序列化 `orders` 列表 -> 序列化第一个 `Order` -> 序列化 `Order` 的 `user` 属性 -> `User` -> 序列化 `orders` 列表 -> ... 无限循环



#### **解决方案**

##### **方案1 （推荐）**

**`@JsonManagedReference` / `@JsonBackReference` **

- 这是 Jackson 官方推荐的、用于**父子关系**（如一对多）的解决方案

  - `@JsonManagedReference`：用在 "父" (One) 的一方（持有列表的一方）。它会**正常序列化**

  - `@JsonBackReference`：

    - 用在 "子" (Many) 的一方（持有父引用的一方）

      它会**在序列化时被忽略**，从而打破循环。但在反序列化时，Jackson 会尝试自动注入父对象

```java
// 父 (One)
public class User {
    public Long id;
    public String name;
    
    @JsonManagedReference
    public List<Order> orders;
}

// 子 (Many)
public class Order {
    public Long id;
    public String orderNumber;
    
    @JsonBackReference
    public User user;
}
```

**序列化 `User` 结果：**

-  `{"id":1, "name":"张三", "orders": [{"id":101, "orderNumber":"A001"}]}` (Order 中的 user 字段被忽略) 

**序列化 `Order` 结果：**

​	 `{"id":101, "orderNumber":"A001"}` (Order 中的 user 字段被忽略)



##### **方案2：`@JsonIgnore` (简单粗暴)** 

- 在 "子" 的一方（`Order`）上直接忽略对 "父"（`User`）的引用

```
public class Order {
    public Long id;
    public String orderNumber;
    
    @JsonIgnore
    public User user;
}
```

**优点：** 简单。 **缺点：** `Order` 对象在任何情况下序列化都不会带 `user` 信息



##### **方案3：`@JsonIdentityInfo` (图结构)**

- 如果对象关系不是严格的父子，而是复杂的图（例如 "朋友" 列表），可以使用此注解

  Jackson 会为每个对象生成一个 ID，并在后续引用中只使用该 ID

```java
@JsonIdentityInfo(
  generator = ObjectIdGenerators.PropertyGenerator.class, 
  property = "id")
public class User {
    public Long id;
    public String name;
    public List<User> friends;
}

// 序列化结果会像：
// [ {"id":1, "name":"张三", "friends": [2]}, {"id":2, "name":"李四", "friends": [1]} ]
```

