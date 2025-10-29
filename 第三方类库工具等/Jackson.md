# Jackson

## 1. 什么是 Jackson？

>Jackson 是一个功能极其强大的库，作为 SpringBoot 的默认选择，它能满足绝大多数应用场景的需求。
>
>通过其丰富的配置选项和注解，你可以实现几乎任何你想要的 JSON 操作。

Jackson 是一个高性能的 Java JSON 处理库，也是 Spring Boot **默认集成**的 JSON 解决方案。它功能极其强大，生态系统非常完善，被广泛认为是 Java 世界中处理 JSON 的事实标准。

Jackson 的核心功能是在 Java 对象和 JSON 数据之间进行转换：

- **序列化 (Serialization)**：将 Java 对象转换为 JSON 字符串。
- **反序列化 (Deserialization)**：将 JSON 字符串恢复为 Java 对象。

由于是 SpringBoot 的“亲儿子”，Jackson 与 Spring 框架的集成是无缝的，开箱即用，无需任何额外配置。



## 2. Jackson 的核心优势

- **功能全面**：除了基本的 JSON 转换，Jackson 还支持多种数据格式，如 XML、CSV、YAML 等，具有很强的扩展性。
- **性能优异**：Jackson 在性能上经过了大量优化，通常是 Java 中最快的 JSON 处理库之一。
- **强大的注解系统**：提供了丰富的注解，可以非常精细地控制序列化和反序列化的每一个细节。
- **与 SpringBoot 完美集成**：作为默认选项，你不需要添加任何依赖或配置就可以直接在 SpringBoot 项目中使用它。



## 3. Jackson 的核心功能和用法

### 3.1 序列化：Java 对象 -> JSON

- 假设我们有一个 `User` 类：

  ```java
  public class User {
      private String name;
      private int age;
      private String email;
  
      // 构造函数、Getter 和 Setter 省略
  }
  ```

  

- 使用 Jackson 进行序列化操作如下：

  ```java
  // 1. 创建 ObjectMapper 实例，这是 Jackson 的核心类
  ObjectMapper objectMapper = new ObjectMapper();
  
  // 2. 创建一个 User 对象
  User user = new User("Cortex", 30, "cortex@example.com");
  
  // 3. 将对象转换为 JSON
  String jsonString = objectMapper.writeValueAsString(user);
  
  // 输出结果
  System.out.println(jsonString);
  // ==> {"name":"Cortex","age":30,"email":"cortex@example.com"}
  ```



### 3.2 反序列化：JSON -> Java 对象

- 反向操作同样简单：

  ```java
  // 1. 准备一个 JSON 字符串
  String json = "{\"name\":\"Gemini\",\"age\":2,\"email\":\"gemini@example.com\"}";
  
  // 2. 创建 ObjectMapper 实例
  ObjectMapper objectMapper = new ObjectMapper();
  
  // 3. 将 JSON 转换为 User 对象
  User user = objectMapper.readValue(json, User.class);
  
  // 访问对象的属性
  System.out.println(user.getName()); // ==> Gemini
  System.out.println(user.getAge());  // ==> 2
  ```

  

### 3.3 处理泛型

- 处理像 `List<User>` 这样的集合时，Jackson 提供了 `TypeReference`：

  ```java
  // 准备一个包含多个用户的 JSON 数组
  String jsonArray = "[{\"name\":\"User1\",\"age\":25},{\"name\":\"User2\",\"age\":28}]";
  
  ObjectMapper objectMapper = new ObjectMapper();
  
  // 使用 TypeReference 定义要转换的目标类型
  List<User> userList = objectMapper.readValue(jsonArray, new TypeReference<List<User>>(){});
  
  // 遍历结果
  for(User user : userList) {
      System.out.println(user.getName());
  }
  // ==> User1
  // ==> User2
  ```

  



## 4. 在 SpringBoot 中配置 Jackson

- 在 SpringBoot 中，你几乎不需要为 Jackson 做任何事情。但如果你想自定义其行为，可以通过 `application.properties` 或 `application.yml` 文件进行配置。

- **对于 `application.properties` 文件:**

  ```properties
  # 1. 格式化输出 JSON
  spring.jackson.serialization.indent-output=true
  
  # 2. 配置日期格式
  spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
  
  # 3. 设置时区
  spring.jackson.time-zone=Asia/Shanghai
  
  # 4. 序列化时忽略 null 值的字段
  spring.jackson.default-property-inclusion=non_null
  
  # 5. 反序列化时，如果 JSON 中的属性在 Java 对象中不存在，不抛出异常
  spring.jackson.deserialization.fail-on-unknown-properties=false
  ```

  

- **对于 `application.yml` 文件:**

  ```yaml
  spring:
    jackson:
      serialization:
        indent-output: true
      date-format: yyyy-MM-dd HH:mm:ss
      time-zone: Asia/Shanghai
      default-property-inclusion: non_null
      deserialization:
        fail-on-unknown-properties: false
  ```



## 5. Jackson 常用注解

- Jackson 的注解非常丰富，这里列举几个最常用的。

  - **`@JsonProperty`**: 用于指定字段在 JSON 中对应的 key 名称

    ```java
    public class Product {
        @JsonProperty("product_name")
        private String productName;
        // ...
    }
    ```

  - **`@JsonIgnore`**: 在序列化和反序列化时完全忽略某个字段

    ```java
    public class User {
        private String username;
        @JsonIgnore
        private String password; // password 字段将不会出现在 JSON 中
    }
    ```

  - **`@JsonFormat`**: 用于格式化日期和时间类型的输出

    ```java
    import java.util.Date;
    import com.fasterxml.jackson.annotation.JsonFormat;
    
    public class Order {
        private String orderId;
        @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd", timezone = "GMT+8")
        private Date createDate;
    }
    ```

  - **`@JsonInclude`**: 精细化控制序列化时字段的包含策略（例如，仅包含非空、非默认值的字段）

    ```java
    import com.fasterxml.jackson.annotation.JsonInclude;
    
    @JsonInclude(JsonInclude.Include.NON_NULL)
    public class User {
        private String name;
        private String email; // 如果 email 为 null，则不会被序列化
    }
    ```

  - **`@JsonCreator` 和 `@JsonProperty`**: 结合使用，可以指定用于反序列化的构造函数

    ```java
    public class Circle {
        private double radius;
    
        @JsonCreator
        public Circle(@JsonProperty("r") double radius) {
            this.radius = radius;
        }
    }
    // 可以反序列化 JSON: {"r": 10.0}
    ```