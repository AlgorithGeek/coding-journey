# JUnit相关概念

- **JUnit** 是一个为 Java 语言编写的、开源的**单元测试框架**。它被广泛用于编写和运行可重复的自动化测试

- **单元测试（Unit Testing）** 是指对软件中的最小可测试单元（通常是一个方法或一个类）进行检查和验证。其目的是确保代码的每个部分都按预期工作

- 在 JavaWeb 开发中，你可以用 JUnit 测试：

  - **DAO 层**：数据库的增删改查操作是否正确。

  - **Service 层**：业务逻辑是否符合预期。

  - **工具类**：各种辅助方法的功能是否正常。

  - ...以及任何不直接依赖 Web 容器（如 Tomcat）的独立代码模块。



# JUnit的作用

1. **保证代码质量**：通过测试可以及早发现代码中的 Bug。
2. **让重构更安全**：当你修改或重构现有代码时，只需重新运行测试，就能快速验证是否破坏了原有功能。
3. **提供活文档**：测试用例本身就是代码功能和使用方式的最佳说明文档。
4. **提升开发效率**：相比于启动整个 Web 应用来调试一小段逻辑，单元测试要快得多。
5. **驱动设计（TDD）**：可以采用“测试驱动开发”模式，先写测试再写实现，从而写出更简洁、更易于测试的代码。



# JUnit 5 中的核心注解

- JUnit 5 提供了丰富的注解来满足不同的测试场景

## 注解 `@Test`

- **`@Test` 是基础注解**：它是最基本的测试注解，用于标记一个**标准的、无参数的测试方法**
- **可替代 `@Test` 的注解**：当你使用以下注解时，它们本身就定义了一个测试，因此**不需要**再额外添加 `@Test` 注解
  - `@ParameterizedTest` (参数化测试)
  - `@RepeatedTest` (重复测试)
  - `@TestFactory` (测试工厂)
- **辅助注解**：以下注解用于修饰测试或定义测试的生命周期。它们有的**直接修饰**测试方法（如 `@DisplayName`），有的则**独立标记辅助方法来服务于测试**（如 `@BeforeAll`, `@AfterAll`, `@BeforeEach`, `@AfterEach`）。



## 注解详解

### 基本测试与生命周期注解

| 注解                       | 说明                                                         | 使用场景                                                     |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`@Test`**                | 标记一个方法为**测试方法**。这是最核心的注解。               | `public void testAdd() { ... }`                              |
| **`@DisplayName("描述")`** | 为测试类或测试方法提供一个自定义的、更易读的显示名称。       | `@DisplayName("测试加法功能")`                               |
| **`@BeforeEach`**          | 标记一个方法，该方法在**每个** `@Test` 方法运行**之前**执行。 | 初始化测试对象、准备测试数据。                               |
| **`@AfterEach`**           | 标记一个方法，该方法在**每个** `@Test` 方法运行**之后**执行。 | 清理资源、恢复环境。                                         |
| **`@BeforeAll`**           | 标记一个**静态(static)方法，该方法在当前测试类中所有**测试方法运行**之前**执行一次。 | 建立数据库连接、启动昂贵的外部资源。                         |
| **`@AfterAll`**            | 标记一个**静态(static)方法，该方法在当前测试类中所有**测试方法运行**之后**执行一次。 | 关闭数据库连接、释放外部资源。                               |
| **`@Disabled`**            | 暂时**禁用**一个测试类或测试方法。被禁用的测试不会被执行。   | 当某个功能暂时不可用或正在重构时。                           |
| **`@Nested`**              | 用于创建一个**内嵌测试类**，可以将相关的测试分组，使结构更清晰。 | `class UserAuthenticationTests { @Nested class LoginTests { ... } }` |



### 高级测试注解 (可替代 `@Test`)

| 注解                     | 说明                                               | 备注                                                         |
| ------------------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| **`@RepeatedTest(N)`**   | 重复测试。将一个方法重复执行 N 次。                | **替代 `@Test`**。                                           |
| **`@ParameterizedTest`** | 参数化测试。使用不同的参数多次运行同一个测试方法。 | **替代 `@Test`**。需要与参数源注解（如 `@ValueSource`）配合。 |
| **`@TestFactory`**       | 测试工厂。用于在运行时动态生成测试。               | **替代 `@Test`**。方法必须返回一个动态测试的集合或流。       |



### 参数源注解 (配合 `@ParameterizedTest` 使用)

| 注解                    | 说明                                                      |
| ----------------------- | --------------------------------------------------------- |
| **`@ValueSource(...)`** | 提供一组简单的字面量值（如 `int`, `String`）。            |
| **`@CsvSource(...)`**   | 提供一组以逗号分隔的值（CSV），可为每次测试提供多个参数。 |



# 断言（Assertions类）

- 断言是测试的灵魂，它用于**验证**代码的行为是否符合预期。如果断言失败，测试就会失败，并给出明确的错误信息

- JUnit 5 的断言方法都位于 `org.junit.jupiter.api.Assertions` 类中，所有方法都是静态的，可以直接导入使用



## 常用断言方法表

| 方法签名                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `assertEquals(Object expected, Object actual, String message?)` | 验证**预期值**和**实际值**是否相等（通过 `equals()` 方法比较）。如果不想等，测试失败。这是最常用的断言。 |
| `assertNotEquals(Object unexpected, Object actual, String message?)` | 验证**非预期值**和**实际值**不相等。如果相等，测试失败。     |
| `assertTrue(boolean condition, String message?)`             | 验证提供的条件是否为 `true`。如果为 `false`，测试失败。      |
| `assertFalse(boolean condition, String message?)`            | 验证提供的条件是否为 `false`。如果为 `true`，测试失败。      |
| `assertNull(Object actual, String message?)`                 | 验证对象是否为 `null`。如果不为 `null`，测试失败。           |
| `assertNotNull(Object actual, String message?)`              | 验证对象是否不为 `null`。如果为 `null`，测试失败。           |
| `assertThrows(Class<T> expectedType, Executable executable, String message?)` | 验证执行 `executable` 中的代码块时，是否会抛出 `expectedType` 类型的异常。如果没有抛出或类型不匹配，测试失败。 |
| `assertAll(String heading?, Executable... executables)`      | 分组断言。它会执行所有传入的断言，并**一次性报告所有失败**，而不是在第一个失败时就停止。非常适合用于验证一个对象的多个属性。 |
| `assertTimeout(Duration timeout, Executable executable, String message?)` | 验证某个操作能否在指定的**超时时间**内完成。如果超时，测试失败。 |
| `assertArrayEquals(Object[] expected, Object[] actual, String message?)` | 验证两个**数组**的内容和顺序是否完全相等。**注意**：不能用 `assertEquals` 来比较数组内容，因为它只会比较数组对象的引用地址。 |
| `fail(String message)`                                       | 直接让测试**失败**。通常用在某些不应该被执行到的代码分支中，以确保逻辑的正确性。 |



## 关于 `message?` 参数的说明

- 你可能注意到了断言方法表中的 `String message?`，这里解释一下：

  - **`?` 的含义**：这是一个文档标记，表示该参数是**可选的 (Optional)**。在写代码时**不能**带上问号。JUnit 为绝大多数断言都提供了两个版本（通过方法重载实现）：一个带 `message`，一个不带

  - **`String message` 的作用**：这个参数用于提供一个**自定义的失败信息**。当断言失败时，你写的这条信息会清晰地显示在测试报告中，帮助你快速定位问题。如果测试通过，该信息会被完全忽略

- **示例：**

  ```java
  // 当测试失败时，默认信息比较通用
  assertEquals(5, 4); 
  // -> org.opentest4j.AssertionFailedError: expected: <5> but was: <4>
  
  // 使用自定义信息，错误一目了然
  assertEquals(5, 4, "用户总数计算错误"); 
  // -> org.opentest4j.AssertionFailedError: 用户总数计算错误 ==> expected: <5> but was: <4>
  ```

  

# 在 Maven 项目中使用 JUnit 5

- 在标准的 Maven 项目中集成并使用 JUnit 5，核心是配置 `pom.xml` 文件

## 配置依赖：两种方式

- 可以根据需要选择以下任意一种方式来添加依赖

  - **方式一 (推荐学习)：分开引入 `api` 和 `engine`**

    - 这种方式能更清晰地理解 JUnit 5 的模块化架构

      ```xml
      <dependencies>
          <!-- 编写测试所需的核心API -->
          <dependency>
              <groupId>org.junit.jupiter</groupId>
              <artifactId>junit-jupiter-api</artifactId>
              <version>5.10.2</version>
              <scope>test</scope>
          </dependency>
          <!-- 运行测试所需的引擎 -->
          <dependency>
              <groupId>org.junit.jupiter</groupId>
              <artifactId>junit-jupiter-engine</artifactId>
              <version>5.10.2</version>
              <scope>test</scope>
          </dependency>
      </dependencies>
      ```

      

  - **方式二 (推荐项目)：使用聚合依赖 `junit-jupiter`**

    - 这是一个“全家桶”依赖，它会自动帮你引入 `api`, `engine`, `params` 等最常用的核心组件。这种方式更简洁，能避免手动引入时可能出现的版本不一致问题

      ```xml
      <dependencies>
          <dependency>
              <groupId>org.junit.jupiter</groupId>
              <artifactId>junit-jupiter</artifactId>
              <version>5.10.2</version>
              <scope>test</scope>
          </dependency>
      </dependencies>
      ```

      

## 配置插件：最佳实践

- 无论你用哪种方式添加依赖，**最佳实践**都是在 `build` 中**明确声明** `maven-surefire-plugin`

  ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-surefire-plugin</artifactId>
              <version>3.2.5</version>
          </plugin>
      </plugins>
  </build>
  ```

  

- **为什么建议明确声明插件？**

  - **依赖** 提供了写测试和运行测试的“能力”（如同工具库）

  - **插件** 则是 Maven 在 `test` 阶段真正调用的“执行者”

  - 虽然 Maven 会默认绑定一个 `surefire` 插件，但明确声明是为了将这个默认的测试行为**“显式化”、“固定化”和“可配置化”**，这是保证项目构建稳定、可靠、可复现的关键一步

- **如何选择插件版本？**

  - **追求兼容性，而非版本号一致**：你不需要寻找一个和 JUnit 版本号完全匹配的 `surefire` 版本。

  - **选择现代版本**：`maven-surefire-plugin` 从 `2.22.0` 版本开始就原生支持 JUnit 5。

  - **最佳实践**：选择一个较新的、稳定的版本（如 `3.x` 系列）是最佳实践。它能很好地兼容所有现代的 JUnit 5 版本。



# JUnit 编写规范与要求

## 命名与存放位置 (推荐)

- **测试类命名**：通常是 **`被测试的类名Test`** 或 **`被测试的类名Tests`**。
  - 例如：测试 `UserService` 类，测试类应命名为 `UserServiceTest`。
- **测试方法命名**：推荐采用能够清晰描述测试行为的风格。
  - **`test功能描述()`**：例如 `testLoginWithInvalidPassword()`。
  - **`should_期望行为_when_特定条件()`**：可读性极佳，如 `should_throwException_when_userIdIsNull()`。
- **测试类的存放位置 (约定优于配置)**：
  - **源代码存放于**：`src/main/java`
  - **测试代码存放于**：`src/test/java`
  - 最重要的是，**测试类的包路径应该和被测试类的包路径保持一致**。这能让 Maven 自动识别，并方便测试代码访问被测类的包级私有成员。



## 强制要求 (方法修饰符详解)

- 这些是编写 JUnit 5 测试时必须遵守的规则，特别是关于方法的访问修饰符 (`public`, `private`) 和 `static` 修饰符

  - **访问修饰符 (`public` vs. `private`)**

    - 在老的 **JUnit 4** 中，测试类和测试方法**必须**声明为 `public`。
    - 在 **JUnit 5** 中，这个限制被放宽了。测试方法可以是 `public` 或 **包级私有 (package-private)**（即不写任何访问修饰符）。
    - **但绝不能是 `private`**！因为测试引擎需要从外部调用你的测试方法，`private` 会阻止这种访问，导致测试无法被发现和执行

  - **`static` 修饰符** `static` 的使用与否，完全取决于注解所关联的生命周期

    | 注解                 | `static` 要求        | 为什么？                                         |
    | -------------------- | -------------------- | ------------------------------------------------ |
    | `@BeforeAll`         | ⭐**必须是** `static` | 作用于整个测试类，在任何测试实例创建前执行。     |
    | `@AfterAll`          | ⭐**必须是** `static` | 作用于整个测试类，在所有测试实例销毁后执行。     |
    | `@BeforeEach`        | **不能是** `static`  | 为每个测试实例做准备，需要访问实例级别的成员。   |
    | `@Test`              | **不能是** `static`  | 每个测试都在一个独立的实例上运行。               |
    | `@RepeatedTest`      | **不能是** `static`  | 同 `@Test`，在实例上运行。                       |
    | `@ParameterizedTest` | **不能是** `static`  | 同 `@Test`，在实例上运行。                       |
    | `@AfterEach`         | **不能是** `static`  | 清理每个测试实例的资源，需要访问实例级别的成员。 |

- **其他强制要求**

  - **方法必须返回 `void`**：测试的成功与否由断言决定，不需要返回值
  - **标准 `@Test` 方法不能有参数**：参数化测试（`@ParameterizedTest`）是例外



# 单元测试最佳实践

1. **快**：单元测试应该运行得非常快
2. **独立**：测试之间不应有任何依赖关系，可以以任意顺序运行
3. **可重复**：每次运行测试都应该得到相同的结果，不受外部环境影响
4. **自我验证**：测试应该能自动判断成功或失败，无需人工检查结果
5. **及时**：测试应该与业务代码一同编写
6. **一个测试只测一件事**：保持测试方法的简单和专注
7. **使用 Mock/Stub 技术**：当测试对象依赖于其他复杂组件（如数据库、网络服务）时，使用 **Mockito** 等框架创建“假”对象来模拟这些依赖，从而实现真正的“单元”隔离



# 为什么 API 和 Engine 是分开的？

- 初次接触时，很多人会觉得奇怪：“为什么编写测试的 `api` 和运行测试的 `engine` 要分成两个依赖？合在一起不是更方便吗？”

  - 这其实是 JUnit 5 最核心的设计思想——**关注点分离（Separation of Concerns）**，它带来了巨大的灵活性和可扩展性

    - **`junit-jupiter-api` (API - 应用程序接口)**
      - **角色**：一套**规范和工具集**，供**开发者**使用。
      - **包含**：`@Test`、`@DisplayName` 等注解和 `assertEquals()` 等断言方法。
      - **目的**：让你能够按照一套稳定的标准来**编写测试**。

    - **`junit-jupiter-engine` (Engine - 引擎)**
      - **角色**：一个**具体的执行者**，供 **Maven、IDE 等工具**使用。
      - **包含**：发现和执行测试的底层逻辑。
      - **目的**：让工具能够**运行你写的测试**。

  - 这种分离的好处在于，JUnit 5 不仅仅是 JUnit 自身的升级，它致力于成为 JVM 上的一个**开放的、统一的测试平台**。其他测试框架（如 Spock）也可以开发自己的 `Engine`，只要它们都接入 JUnit Platform 这个统一的“插座”，我们开发者就能用同样的方式（`mvn test`）来运行所有类型的测试，IDE 也能用同样的方式来展示结果。

  - 这个设计也使得**向后兼容**成为可能：通过引入 `junit-vintage-engine`，JUnit 5 可以无缝运行老的 JUnit 4 测试，极大地便利了项目的升级和迁移


