# Gson

## 1. 什么是 Gson？

- Gson 是一个由 Google 开发的 Java 库，用于在 Java 对象和 JSON 格式数据之间进行相互转换。这个过程主要包括两个核心操作：

  - **序列化 (Serialization)**：将 Java 对象转换成 JSON 字符串。这在你需要通过 API 将数据发送到前端或其他服务时非常有用

  - **反序列化 (Deserialization)**：将 JSON 字符串转换成 Java 对象。这在你接收到来自外部的 JSON 数据并需要在 Java 程序中使用时非常方便

- Gson 以其**高性能**、**高可靠性**和**易用性**而著称，是 Java 开发中处理 JSON 的主流工具之一



## 2. 为什么在 SpringBoot 中需要 Gson？

- SpringBoot 默认使用的 JSON 处理库是 **Jackson**。Jackson 功能强大，生态完善，在大多数情况下都表现得很好。但是，在某些特定场景下，你可能会更倾向于使用 Gson：

  - **性能要求**：在某些基准测试中，Gson 在处理大规模或结构简单的 JSON 数据时可能表现出比 Jackson 更好的性能

  - **处理非标准 JSON**：Gson 对于一些不那么规范的 JSON 格式容忍度更高，处理起来更简单

  - **特定功能需求**：Gson 提供了一些独特的特性，例如 `TypeAdapter`，可以让你对序列化和反序列化的过程进行非常精细的控制

  - **团队技术栈**：如果你的团队成员对 Gson 更熟悉，那么在项目中统一使用 Gson 可以降低学习成本

- 简单来说，学习 Gson 不仅是多掌握一个工具，更是让你在面对不同业务需求时，能够有更多、更合适的选择



## 3. Gson 的核心功能和用法

### 3.1 序列化：Java 对象 -> JSON

- 这是最常见的用法。假设我们有一个 `User` 类：

  ```java
  public class User {
      private String name;
      private int age;
      private String email;
  
      // 构造函数、Getter 和 Setter 省略
  }
  ```

  

- 使用 Gson 将其实例化对象转换为 JSON 字符串非常简单：

  ```java
  // 1. 创建 Gson 实例
  Gson gson = new Gson();
  
  // 2. 创建一个 User 对象
  User user = new User("Cortex", 30, "cortex@example.com");
  
  // 3. 将对象转换为 JSON
  String jsonString = gson.toJson(user);
  
  // 输出结果
  System.out.println(jsonString);
  // ==> {"name":"Cortex","age":30,"email":"cortex@example.com"}
  ```

  

### 3.2 反序列化：JSON -> Java 对象

- 同样，将 JSON 字符串转换回 Java 对象也一样直观：

  ```java
  // 1. 准备一个 JSON 字符串
  String json = "{\"name\":\"Gemini\",\"age\":2,\"email\":\"gemini@example.com\"}";
  
  // 2. 创建 Gson 实例
  Gson gson = new Gson();
  
  // 3. 将 JSON 转换为 User 对象
  User user = gson.fromJson(json, User.class);
  
  // 访问对象的属性
  System.out.println(user.getName()); // ==> Gemini
  System.out.println(user.getAge());  // ==> 2
  ```

  

### 3.3 处理泛型

- 当你要处理像 `List<User>` 这样的泛型集合时，需要借助 `TypeToken` 来辅助 Gson 准确识别类型：

  ```java
  // 准备一个包含多个用户的 JSON 数组
  String jsonArray = "[{\"name\":\"User1\",\"age\":25},{\"name\":\"User2\",\"age\":28}]";
  
  Gson gson = new Gson();
  
  // 使用 TypeToken 定义要转换的目标类型
  Type userListType = new TypeToken<ArrayList<User>>(){}.getType();
  
  // 反序列化
  List<User> userList = gson.fromJson(jsonArray, userListType);
  
  // 遍历结果
  for(User user : userList) {
      System.out.println(user.getName());
  }
  // ==> User1
  // ==> User2
  ```



## 4. 在 SpringBoot 中集成和使用 Gson

- 要在 SpringBoot 项目中用 Gson 替换掉默认的 Jackson，操作非常简单。

### 步骤 1: 添加 Gson 依赖

- 首先，在你的 `pom.xml` 文件中添加 Gson 的 Maven 依赖。

  ```xml
  <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <!-- 使用较新的稳定版本 -->
  </dependency>
  ```

- *注意：SpringBoot 的 Parent POM 通常已经管理了 Gson 的版本，所以你可能不需要指定 `<version>` 标签*



### 步骤 2: 配置 SpringBoot 使用 Gson

- 最简单的方式是在 `application.properties` 或 `application.yml` 文件中进行配置。

- **对于 `application.properties` 文件:**

  ```properties
  # 设置 Spring Boot 的首选 JSON 处理器为 GSON
  spring.http.converters.preferred-json-mapper=gson
  ```

- **对于 `application.yml` 文件:**

  ```yaml
  spring:
    http:
      converters:
        preferred-json-mapper: gson
  ```

- 完成这两步后，你的 SpringBoot 应用在处理 HTTP 请求和响应中的 JSON 数据时，就会自动使用 Gson 了。
  - 例如，你在 Controller 中返回一个对象，SpringBoot 会调用 Gson 将其序列化为 JSON 字符串。



### 步骤 3: (可选) 自定义 Gson 配置

- 有时候，你可能需要对 Gson 的行为进行更细致的控制，比如日期格式、空值处理等。

  - 你可以通过创建一个 `GsonBuilder` Bean 来实现

  - 在你的配置类（一个带有 `@Configuration` 注解的类）中添加如下代码：

  ```java
  import com.google.gson.Gson;
  import com.google.gson.GsonBuilder;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.http.converter.json.GsonHttpMessageConverter;
  
  @Configuration
  public class GsonConfig {
  
      @Bean
      public GsonHttpMessageConverter gsonHttpMessageConverter() {
          GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
          GsonBuilder builder = new GsonBuilder();
  
          // 1. 设置日期格式
          builder.setDateFormat("yyyy-MM-dd HH:mm:ss");
  
          // 2. 配置序列化时，包含 null 值的字段
          //    默认情况下，Gson 会忽略 null 值的字段
          builder.serializeNulls();
  
          // 3. 配置输出的 JSON 格式化（易于阅读）
          builder.setPrettyPrinting();
  
          Gson gson = builder.create();
          converter.setGson(gson);
          return converter;
      }
  }
  ```



## 5. Gson 常用注解

- Gson 提供了一些注解来处理字段命名不一致或特定字段不需要序列化等问题。

  - **`@SerializedName`**: 当 Java 对象的字段名和 JSON 中的 key 不匹配时使用。

    ```java
    public class Product {
        // JSON 中的 key 是 "product_name"，而 Java 字段是 "productName"
        @SerializedName("product_name")
        private String productName;
        private double price;
        // ...
    }
    ```

  - **`@Expose`**: 用于精确控制哪些字段应该被序列化或反序列化。需要与 `GsonBuilder` 的 `excludeFieldsWithoutExposeAnnotation()` 方法配合使用。

    ```java
    public class User {
        @Expose
        private String username;
    
        // password 字段将不会被序列化
        private String password;
    }
    
    // 配置 Gson
    Gson gson = new GsonBuilder()
            .excludeFieldsWithoutExposeAnnotation()
            .create();
    ```

  - **`transient` 关键字**: Java 原生的 `transient` 关键字也可以告诉 Gson 在序列化时忽略某个字段。

    ```java
    public class Account {
        private String id;
        // 临时密码，不需要被序列化存储
        private transient String temporaryPassword;
    }
    ```

