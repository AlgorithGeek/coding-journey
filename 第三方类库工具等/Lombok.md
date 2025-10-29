# Lombok 简介

- **Lombok** 是一个第三方 Java工具库

  - 它通过注解的方式，在编译时自动为 Java类 生成那些冗长乏味的“样板代码”，

    > 例如 `getter/setter` 方法、构造函数、`toString()`、`equals()` 和 `hashCode()` 等

- **核心作用：**是消除代码冗余，让代码库更加简洁、可读和易于维护，从而显著提升开发效率



# 如何配置和使用 Lombok

## 步骤

### 1.**在项目中添加 Lombok 依赖**

```xml
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>XXX</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

- **注意**: `scope`被设置为`provided`，因为Lombok主要在编译期间起作用，运行时并不需要它



### 2.在IDEA中安装 Lombok 插件

- **为什么都引入依赖了，还需要IDE插件？**

  -  如果没有插件，IDE将无法识别 Lombok 在编译期生成的那些方法（如`getName()`），因此会在你的代码中显示编译错误，尽管项目本身可以成功构建

    插件能让IDE理解这些注解并提供正确的代码提示和语法检查

- **插件图**

  ![image-20251021171016909](./assets/image-20251021171016909.png)



# Lombok 常见注解

## 空

### `@NonNull`

- `@NonNull` 是一个契约式编程注解，它的主要作用是在方法或构造函数的开头自动生成一个**空值检查**

  如果被注解的参数、字段或变量被赋予了一个 `null` 值，代码会立即抛出一个 `NullPointerException`，从而防止 `null` 值在程序中进一步传递

- Lombok 的 `@NonNull` 只会在你通过代码“入口”**主动为字段赋予 `null` 值的时候报错**

- 这有助于开发者遵循 “快速失败” 的设计原则，即让问题在发生的第一时间就暴露出来，而不是等到 `null` 值在代码深处被使用时才引发难以追踪的错误



#### 1. 它解决了什么问题？

- 在 Java 中，`NullPointerException` (NPE) 是最常见的运行时异常之一

  为了避免它，开发者通常需要在方法或构造函数的开头手动编写防御性代码：

  ```java
  public class UserProfile {
      private String username;
  
      public UserProfile(String username) {
          if (username == null) {
              throw new NullPointerException("username is marked non-null but is null");
          }
          this.username = username;
      }
      
      public void setUsername(String username) {
          if (username == null) {
              throw new NullPointerException("username is marked non-null but is null");
          }
          this.username = username;
      }
  }
  ```

  - 这种代码非常重复、冗长，且容易被遗忘。`@NonNull` 可以自动为我们生成这些检查



#### 2. 作用与核心思想

- `@NonNull` 的核心思想是 **在代码中插入一个非空断言**

  - 当用在 **方法或构造函数的参数** 上时

    - 它会在方法体的最开始插入一条 `if (param == null) throw new NullPointerException("param is marked non-null but is null");` 语句

    

  - 当用在 **字段**上时

    - 它会自动作用于 Lombok 生成的**构造函数参数**和 **setter 方法参数**上，为它们添加空值检查

    

  - 当用在 **局部变量**上时

    - 它会在该变量声明的下一行立即插入一条 `if (variable == null) throw new NullPointerException(...)` 语句



#### 3. 位置

- **方法参数**
- **构造函数参数**
- **字段**: 
  - 当与 `@Setter`, `@RequiredArgsConstructor`, `@AllArgsConstructor`, `@Data` 等注解一起使用时，空值检查会被添加到生成的对应方法中
- **局部变量**: 也可以用在局部变量上



#### 4. 常用属性

- `@NonNull` 本身是一个标记注解，它**没有任何可配置的属性**。它的存在就代表着一个非空契约



#### 5. 示例代码

##### 示例 1: 在构造函数参数上使用

```java
import lombok.NonNull;

public class Task {
    private final String description;

    public Task(@NonNull String description) {
        this.description = description;
    }
}

// 如何触发:
// new Task("Do something"); // OK
// new Task(null);          // Throws NullPointerException
```

- **生成的代码:**

  ```java
  public class Task {
      private final String description;
  
      public Task(String description) {
          if (description == null) {
              throw new NullPointerException("description is marked non-null but is null");
          }
          this.description = description;
      }
  }
  ```



##### 示例 2: 在字段上使用 (结合 `@Setter` 和 `@RequiredArgsConstructor`)

```java
import lombok.Getter;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@RequiredArgsConstructor
public class UserAccount {
    @NonNull
    private String userId; // 这个字段的构造器和 setter 都会有 null-check

    private String displayName;
}
```

- **生成的代码:**

  ```java
  public class UserAccount {
      @NonNull
      private String userId;
      private String displayName;
  
      // 由 @RequiredArgsConstructor 生成，并加入了 null-check
      public UserAccount(String userId) {
          if (userId == null) {
              throw new NullPointerException("userId is marked non-null but is null");
          }
          this.userId = userId;
      }
  
      // 由 @Setter 生成，并加入了 null-check
      public void setUserId(String userId) {
          if (userId == null) {
              throw new NullPointerException("userId is marked non-null but is null");
          }
          this.userId = userId;
      }
      // ... 其他 getter 和 setter ...
  }
  ```



## 构造器

### `@NoArgsConstructor`

#### 1. 作用与核心思想

- 自动在类中生成一个不接受任何参数的构造方法

  - 如果类中没有 `final` 字段，生成的构造函数将是空的

  - 如果类中有 `final` 字段，直接使用 `@NoArgsConstructor` 会导致编译错误，因为 `final` 字段必须在构造函数中被初始化

    这时，你需要使用 `force = true` 属性来强制生成（但需格外小心）



#### 2. 位置

- **类 (Class)**



#### 3. 常用属性（详细讲解）

##### `access`

- **类型**: `lombok.AccessLevel`
- **作用**: 
  - 这个属性用来精确控制生成的无参构造函数的**访问级别**。`AccessLevel` 是一个枚举，包含以下几个值：
    - `PUBLIC`: `public` (默认值)，公开的，任何地方都可以访问
    - `PROTECTED`: `protected`，受保护的，仅限本包和子类访问
    - `PACKAGE`: 包级私有（默认），仅限本包内访问
    - `PRIVATE`: `private`，私有的，仅限本类内部访问
- **使用场景**: 
  - 当你为了满足框架要求而需要一个无参构造函数，但又不希望它在业务代码中被随意调用时，可以将其访问级别设为 `PROTECTED` 或 `PRIVATE`



##### `force`

- **类型**: `boolean`

- **作用**: 

  - 这是一个“强制”开关，默认值为 `false`。当你的类中包含**未初始化的 `final` 字段**时，Java 编译器会要求构造函数必须对这些字段进行初始化

    因此，Lombok 默认情况下会拒绝生成无参构造函数并报错

  - 将 `force` 设为 `true` 后，Lombok 会“强行”生成一个无参构造函数，并将所有 `final` 字段初始化为默认值（`0`, `false`, `null`）

    > 设置为`true`后，在生成的无参构造函数里，会主动把 `final` 字段初始化为它的默认值



##### `staticName`

- **类型**: `String`
- **作用**: 如果为此属性提供一个字符串名称（例如 `"newInstance"`），Lombok 的行为会发生改变：
  1. 它会将生成的无参构造函数设置为 `private`
  2. 它会额外生成一个 `public static` 的**静态工厂方法**，方法名就是你提供的 `staticName`，这个方法会调用那个私有的无参构造函数来创建实例
- **使用场景**: 
  - 这是一种推荐的实例化对象的方式，因为静态工厂方法有名称，可读性更好，并且可以隐藏具体的实现细节



##### `onConstructor_` (注意下划线)

- **类型**: `lombok.experimental.Tolerate`

- **作用**: 

  - 这是一个实验性功能，允许你将其他注解添加到 Lombok 生成的构造函数上

    这在与某些依赖注入框架（如 Guice 的 `@Inject`）或校验框架集成时非常有用

- **使用方法**: 

  - 属性的格式是 `onConstructor_ = @YourAnnotation`

    如果你需要添加多个注解，可以使用花括号 `{}` 包裹起来，例如 `onConstructor_ = {@Inject, @Deprecated}`



#### 4. 示例代码

#### 示例 1: 基础用法

```java
import lombok.NoArgsConstructor;

@NoArgsConstructor
public class Credentials {
    private String username;
    private String password;
}
```

- **生成代码:**

  ```java
  public class Credentials {
      private String username;
      private String password;
  
      public Credentials() {
      }
  }
  ```



#### 示例 2: 设置访问级别为 `private`

```java
import lombok.AccessLevel;
import lombok.NoArgsConstructor;

@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class SingletonService {
    // ...
}
```

- **生成代码:**

  ```java
  public class SingletonService {
      private SingletonService() { // 构造函数是 private
      }
      // ...
  }
  ```



#### 示例 3: 强制生成 (危险操作)

```java
import lombok.NoArgsConstructor;

@NoArgsConstructor(force = true)
public class UserEntity {
    private final Long id; // final 字段
    private String name;
}
```

**生成代码:**

```java
public class UserEntity {
    private final Long id;
    private String name;

    public UserEntity() {
        this.id = null; // final 字段被初始化为 null!
    }
}
```



#### 示例 4: 使用静态工厂方法

```java
import lombok.NoArgsConstructor;

@NoArgsConstructor(staticName = "create")
public class Task {
    // ...
}
```

**生成代码:**

```java
public class Task {
    private Task() { // 构造函数变为 private
    }

    public static Task create() { // 提供了 public 静态工厂方法
        return new Task();
    }
    // ...
}
```



### `@AllArgsConstructor`

#### 1. 作用与核心思想

- 自动生成一个**包含类中所有字段的构造函数**

  - 如果类中有 `final` 字段，这些字段也会被包含在构造函数中，并在此处进行初始化

  - 如果类是空的（没有任何字段），它会生成一个功能上等同于默认无参构造函数的构造器



#### 2. 位置

- **类 (Class)**



#### 3. 常用属性

##### `access`

- **类型**: `lombok.AccessLevel`
- **作用**: 与 `@NoArgsConstructor` 和 `@RequiredArgsConstructor` 中的 `access` 属性完全相同。它用于精确控制生成的全参构造函数的**访问级别**。
  - `PUBLIC`: `public` (默认值)，公开的
  - `PROTECTED`: `protected`，受保护的
  - `PACKAGE`: 包级私有
  - `PRIVATE`: `private`，私有的
- **使用场景**: 当你希望强制通过其他方式（如 `@Builder` 或静态工厂方法）来创建对象，但又想保留一个私有的全参构造函数供内部使用时，这个属性非常有用



##### `staticName`

- **类型**: `String`
- **作用**: 
  - 这个属性的功能也和在其他构造器注解中一样。当你提供一个字符串名称（例如 `"of"`) 时，Lombok 会：
    1. 将生成的全参构造函数设置为 `private`
    2. 额外生成一个 `public static` 的**静态工厂方法**，其参数与私有构造函数完全相同，方法名就是你提供的 `staticName`
- **使用场景**: 
  - 使用静态工厂方法可以让代码更具可读性，例如 `UserDto.of("testUser", "test@example.com")` 比 `new UserDto(...)` 意图更清晰



##### `suppressConstructorProperties`

- **类型**: `boolean`
- **作用**: 这是一个用来控制注解生成的开关，默认值为 `false`
  - 默认情况下 (`false`)，Lombok 会在生成的构造函数上添加一个 `@java.beans.ConstructorProperties` 注解
    - 这个注解对于需要反射进行对象创建的框架（如 Jackson）很有帮助
  - 当你将此属性设为 `true` 时，Lombok 将**不会**生成 `@ConstructorProperties` 注解
- **使用场景**: 在某些极少数情况下，该注解可能与特定环境或工具冲突，此时可以将其禁用



##### `onConstructor_`

- **类型**: `lombok.experimental.Tolerate`
- **作用**: 允许你将一个或多个其他注解（例如 Spring 的 `@Autowired` 或 JSR 303 的校验注解）添加到 Lombok 生成的构造函数上
- **使用方法**: `onConstructor_ = @Autowired`。多个注解用花括号包裹：`onConstructor_ = {@Autowired, @Valid}`



#### 4. 示例代码

##### 示例 1: 基础用法

```java
import lombok.AllArgsConstructor;

@AllArgsConstructor
public class UserDto {
    private String username;
    private final String email;
    private int age;
}
```

- **生成代码:**

  ```java
  public class UserDto {
      private String username;
      private final String email;
      private int age;
  
      // @ConstructorProperties({"username", "email", "age"}) // 默认会生成
      public UserDto(String username, String email, int age) {
          this.username = username;
          this.email = email;
          this.age = age;
      }
  }
  ```



##### 示例 2: 使用静态工厂方法并设为私有构造器

```java
import lombok.AccessLevel;
import lombok.AllArgsConstructor;

@AllArgsConstructor(staticName = "create", access = AccessLevel.PRIVATE)
public class Product {
    private final Long id;
    private String name;
    private double price;
}
```

- **生成代码:**

  ```java
  public class Product {
      private final Long id;
      private String name;
      private double price;
  
      // 构造函数变为 private
      private Product(Long id, String name, double price) {
          this.id = id;
          this.name = name;
          this.price = price;
      }
  
      // 提供了 public 静态工厂方法
      public static Product create(Long id, String name, double price) {
          return new Product(id, name, price);
      }
  }
  ```



### `@RequiredArgsConstructor`

#### 1. 作用与核心思想

- 自动生成一个构造函数，其参数只包含符合以下条件的字段：

  > 构造函数中参数的顺序与字段在类中声明的顺序一致

  1. 所有未被初始化的 `final` 字段
  2. 所有被 `@NonNull` 注解标记的字段
     - 这些字段在构造函数中会自动进行 null-check，如果传入 `null` 会抛出 `NullPointerException`



#### 2. 位置

- **类**



#### 3. 常用属性

##### `access`

- **类型**: `lombok.AccessLevel`
- **作用**: 与 `@NoArgsConstructor` 中的 `access` 属性完全相同。它用于精确控制生成的构造函数的**访问级别**
  - `PUBLIC`: `public` (默认值)，公开的
  - `PROTECTED`: `protected`，受保护的
  - `PACKAGE`: 包级私有
  - `PRIVATE`: `private`，私有的
- **使用场景**: 
  - 当你希望通过静态工厂方法来控制对象的创建时，可以将构造函数的访问级别设为 `PRIVATE`，然后配合 `staticName` 属性使用



##### `staticName`

- **类型**: `String`
- **作用**: 
  - 这个属性的功能也和在 `@NoArgsConstructor` 中一样。当你提供一个字符串名称（例如 `"of"` 或 `"from"`）时，Lombok 会：
    1. 将生成的构造函数设置为 `private`
    2. 额外生成一个 `public static` 的**静态工厂方法**，其参数与私有构造函数完全相同，方法名就是你提供的 `staticName`
- **使用场景**: 
  - 使用静态工厂方法可以让代码更具可读性，例如 `Order.of(customer, product)` 比 `new Order(customer, product)` 意图更清晰



##### `suppressConstructorProperties`

- **类型**: `boolean`

- **作用**: 这是一个用来控制注解生成的开关，默认值为 `false`

  - 默认情况下 (`false`)，Lombok 会在生成的构造函数上添加一个 `@java.beans.ConstructorProperties` 注解

    这个注解会以字符串形式列出所有构造参数的名称，某些框架（如 Jackson）可以利用它来更好地进行反序列化，即使参数名在编译后丢失了也能正确对应

  - 当你将此属性设为 `true` 时，Lombok 将**不会**生成 `@ConstructorProperties` 注解

- **使用场景**: 

  - 在某些极少数情况下，`@ConstructorProperties` 注解可能会与特定框架或旧版本的 Java 环境产生冲突，或者你希望减小一点点编译后字节码的大小，此时可以将其禁用



##### `onConstructor_` (注意下划线)

- **类型**: `lombok.experimental.Tolerate`

- **作用**: 

  - 与 `@NoArgsConstructor` 中的 `onConstructor_` 属性完全相同

    它允许你将一个或多个其他注解（例如 Spring 的 `@Autowired`）添加到 Lombok 生成的构造函数上

- **使用方法**: 

  - `onConstructor_ = @Autowired`。多个注解用花括号包裹：`onConstructor_ = {@Autowired, @Inject}`



#### 4. 示例代码

##### 示例 1: 仅包含 final 字段

- 这是最常见的用法，尤其是在进行构造器注入时

  ```java
  import lombok.RequiredArgsConstructor;
  import org.springframework.stereotype.Service;
  
  @Service
  @RequiredArgsConstructor
  public class UserService {
      private final UserRepository userRepository; // 'final' 字段
      private final String serviceName = "UserService"; // 'final' 但已初始化，不会被包含
      private int maxUsers; // 非 final, 非 @NonNull, 不会被包含
  }
  ```

  - **生成代码:**

    ```java
    @Service
    public class UserService {
        private final UserRepository userRepository;
        private final String serviceName = "UserService";
        private int maxUsers;
    
        // @ConstructorProperties({"userRepository"}) // 默认会生成
        public UserService(UserRepository userRepository) {
            this.userRepository = userRepository;
        }
    }
    ```



##### 示例 2: 包含 `@NonNull` 字段

- `@NonNull` 标记的字段也会被认为是“必需”的

  ```java
  import lombok.NonNull;
  import lombok.RequiredArgsConstructor;
  
  @RequiredArgsConstructor
  public class Order {
      @NonNull // @NonNull 字段
      private Customer customer;
      private final String orderId; // final 字段
  }
  ```

  - **生成代码:**

    ```java
    import java.util.Objects;
    
    public class Order {
        private Customer customer;
        private final String orderId;
    
        public Order(Customer customer, String orderId) {
            // @NonNull 会生成 null-check
            if (customer == null) {
                throw new NullPointerException("customer is marked non-null but is null");
            }
            this.customer = customer;
            this.orderId = orderId;
        }
    }
    ```

    

##### 示例 3: 使用静态工厂方法

```java
import lombok.AccessLevel;
import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor(staticName = "of", access = AccessLevel.PRIVATE)
public class Pair<K, V> {
    private final K key;
    private final V value;
}
```

- **生成代码:**

  ```java
  public class Pair<K, V> {
      private final K key;
      private final V value;
  
      // 构造函数变为 private
      private Pair(K key, V value) {
          this.key = key;
          this.value = value;
      }
  
      // 提供了 public 静态工厂方法
      public static <K, V> Pair<K, V> of(K key, V value) {
          return new Pair<K, V>(key, value);
      }
  }
  ```



### `@Builder`

- `@Builder` 是一个实现了“构建者设计模式”的注解

  当你需要创建一个拥有多个字段（尤其是可选字段）的对象时，使用构建者模式可以让你的代码变得极其优雅、可读性高且不易出错。它避免了使用冗长的构造函数或者繁琐的 setter 方法链



#### 1. 作用与核心思想

- Lombok 会为你的类生成一套完整的构建者 API，通常包括：

  1. 一个名为 `YourClassBuilder` 的**内部静态类**（即构建者类）
  2. 构建者类中包含与目标类字段对应的**设置方法**（例如 `username(String username)`），这些方法返回构建者自身，以便进行链式调用
  3. 一个名为 `build()` 的方法，用于根据设置的值**创建并返回**目标类的实例
  4. 一个名为 `builder()` 的**静态方法**在目标类中，作为获取构建者实例的入口



#### 2. 位置

- **类**: 这是最常见的用法，Lombok 会为类中所有的非静态字段生成构建者。
- **构造函数**: 当你希望构建者只包含类中部分字段时，可以手写一个包含这些字段的构造函数，并将 `@Builder` 注解放在这个构造函数上
- **静态方法**: 与构造函数类似，你可以将 `@Builder` 放在一个返回类实例的静态方法上，Lombok 会根据该方法的参数来生成构建者



#### 3. 常用属性

##### `builderMethodName`

- **类型**: `String`

- **作用**: 用于自定义获取构建者实例的**静态方法**的名称

- **默认值**: `"builder"`

- **使用场景**: 

  - 默认生成的入口方法是 `User.builder()`

    如果你想换个名字，比如 `User.create()`，就可以设置 `builderMethodName = "create"`



##### `buildMethodName`

- **类型**: `String`

- **作用**: 用于自定义构建者类中**最终构建对象**的方法的名称

- **默认值**: `"build"`

- **使用场景**: 

  - 默认的构建方法是 `builder.build()`

    如果你希望用一个更具业务含义的词，比如 `builder.execute()` 或 `builder.done()`，就可以通过这个属性来修改



##### `builderClassName`

- **类型**: `String`

- **作用**: 

  - 用于自定义生成的**构建者内部类**的名称

- **默认值**: 

  - `"<你的类名>+NameBuilder"` (例如，`User` 类的构建者是 `UserBuilder`)

- **使用场景**: 

  - 在大多数情况下，默认名称已经足够清晰

    但在某些特殊情况下，比如为了避免命名冲突，或者想让构建者类的名称更符合特定领域的术语，你可以自定义它



##### `toBuilder`

- **类型**: `boolean`

- **作用**: 

  - 这是一个非常有用的开关，默认值为 `false`

    当设置为 `true` 时，Lombok 会在你的类中额外生成一个名为 `toBuilder()` 的成员方法，返回值为一个新的 `Builder` 实例

    这个 `Builder` 内部的字段值已经用当前对象的所有字段值给填充好了

    > 你可以基于这个预填充的 `Builder` 进行链式调用，只修改你感兴趣的字段，然后调用 `build()` 创建一个新对象

    方法大概是这样的

    ```java
    public OrderBuilder toBuilder() {
            // 返回一个新的、已填充所有字段的构建者实例
            return new OrderBuilder()
                    .orderId(this.orderId)
                    .product(this.product)
                    .quantity(this.quantity)
                    .status(this.status);
        }
    ```

- **核心功能**: 

  - 调用 `toBuilder()` 方法会返回一个**包含了当前对象所有字段值的新构建者实例**

    这使得“复制并修改”一个不可变对象变得非常简单

- **使用场景**: 

  - 当你有一个不可变对象（例如用 `@Value` 注解的类），并且想创建一个与它大部分属性相同，只有一两个属性不同的新对象时，这个功能是最佳选择



##### `access`

- **类型**: `lombok.AccessLevel`
- **作用**: 用于控制生成的 `builder()` 静态方法的访问级别
- **默认值**: `PUBLIC`
- **使用场景**: 如果你希望对象的创建入口不直接暴露给外部，可以将其级别降低为 `PROTECTED` 或 `PRIVATE`



##### `setterPrefix`

- **类型**: `String`

- **作用**: 用于为构建者中的字段设置方法添加或移除前缀

- **使用场景**: 

  - 假设你有一个字段 `_name`，Lombok 默认生成的设置方法是 `_name()`

    如果你希望方法是 `withName()`，可以设置 `setterPrefix = "with"`，Lombok 会自动移除字段的下划线前缀并添加你指定的前缀



#### 4. 示例代码

##### 示例 1: 基础用法

```java
import lombok.Builder;

@Builder
public class UserProfile {
    private final String userId;
    private String username;
    private String email;
    private int age;
}

// 如何使用:
// UserProfile profile = UserProfile.builder()
//         .userId("u123")
//         .username("LombokFan")
//         .email("contact@lombok.com")
//         .age(30)
//         .build();
```

- **生成的部分核心代码 (简化版):**

  ```java
  public class UserProfile {
      // ... 字段 ...
  
      // 隐藏的全参构造函数
      UserProfile(String userId, String username, String email, int age) { ... }
  
      // 静态入口方法
      public static UserProfileBuilder builder() {
          return new UserProfileBuilder();
      }
  
      // 内部构建者类
      public static class UserProfileBuilder {
          private String userId;
          // ... 其他字段 ...
  
          // 链式设置方法
          public UserProfileBuilder userId(String userId) {
              this.userId = userId;
              return this;
          }
          // ... 其他方法 ...
  
          // 构建方法
          public UserProfile build() {
              return new UserProfile(userId, username, email, age);
          }
      }
  }
  ```

  

##### 示例 2: 使用 `toBuilder = true`

```java
import lombok.Builder;
import lombok.Value;

@Value // 通常与不可变类一起使用
@Builder(toBuilder = true)
public class ImmutableTask {
    String id;
    String description;
    boolean completed;
}

// 如何使用:
// ImmutableTask originalTask = ImmutableTask.builder().id("t1").description("Initial task").build();
// ImmutableTask completedTask = originalTask.toBuilder().completed(true).build();
// 'completedTask' 拥有 'originalTask' 的所有属性，只是 'completed' 状态被更新了。
```

- **生成的额外方法:**

  ```java
  public class ImmutableTask {
      // ... 其他生成的方法 ...
  
      public ImmutableTaskBuilder toBuilder() {
          return new 	
              ImmutableTaskBuilder().id(this.id).description(this.description).completed(this.completed);
      }
  }
  ```

  



### `@SuperBuilder`

- **标准 `@Builder` 不支持继承**

  如果你在一个继承体系中的子类上使用 `@Builder`，它生成的构建者将无法设置父类中的字段

  `@SuperBuilder` 通过生成一套更复杂的、支持继承的构建者 API 来完美解决这个问题



#### 1. 作用与核心思想

- `@SuperBuilder` 的核心思想是让类内部的**构建者内部类也形成继承关系**

  当你在一个继承体系中使用它时，Lombok 会：

  1. 为继承链中的**每一个类**（父类和子类）都生成一个对应的抽象构建者类
  2. 子类的构建者会继承父类的构建者
  3. 通过泛型和一系列精巧的方法，确保链式调用时返回的是正确的子类构建者实例，从而让你可以在一次链式调用中设置从顶层父类到当前子类的所有字段



- **关键规则**: 
  - 如果一个类使用了 `@SuperBuilder`，那么它的所有父类（直到 `Object`）也必须使用 `@SuperBuilder`



#### 2. 位置

- **类**: 

  - 必须用在继承链中**所有**相关的类上

    > 如果



#### 3. 常用属性

- `@SuperBuilder` 支持 `@Builder` 的大部分常用属性，并且它们的作用是完全相同的

##### `builderMethodName`

- **类型**: `String`
- **作用**: 自定义获取构建者实例的静态方法的名称。
- **默认值**: `"builder"`

##### `buildMethodName`

- **类型**: `String`
- **作用**: 自定义构建者类中最终构建对象的方法的名称。
- **默认值**: `"build"`

##### `toBuilder`

- **类型**: `boolean`
- **作用**: 设置为 `true` 时，会生成一个 `toBuilder()` 实例方法。这个方法非常强大，它返回的构建者实例会正确地包含从父类到子类的所有字段值，让你能够轻松地复制和修改整个继承链上的属性。
- **默认值**: `false`



#### 4. 示例代码

- ##### 场景: 父类 `Person` 和子类 `Student`

  - **父类 `Person`:**

    ```java
    import lombok.experimental.SuperBuilder;
    
    @SuperBuilder
    public class Person {
        private String name;
        private int age;
    }
    ```

    

  - **子类 `Student`:**

    ```java
    import lombok.experimental.SuperBuilder;
    
    @SuperBuilder
    public class Student extends Person {
        private String school;
        private String studentId;
    }
    ```

    

  - **如何使用:** 你可以直接在子类上调用 `builder()`，然后像普通 `@Builder` 一样设置所有字段，包括从父类继承的字段

    ```java
    // 正确用法: 在一次调用中设置父类和子类的字段
    Student student = Student.builder()
            .name("Li Hua")      // 来自父类 Person
            .age(18)             // 来自父类 Person
            .school("High School") // 来自子类 Student
            .studentId("S12345") // 来自子类 Student
            .build();
    ```

  

  - **生成代码:** Lombok 生成的代码比这个复杂得多，但核心思想如下：

    ```java
    // 父类 Person 的构建者 (抽象)
    public abstract static class PersonBuilder<C extends Person, B extends PersonBuilder<C, B>> {
        private String name;
        private int age;
    
        public B name(String name) { this.name = name; return self(); }
        public B age(int age) { this.age = age; return self(); }
    
        protected abstract B self(); // 关键方法，返回子类构建者实例
        public abstract C build();
    }
    
    // 子类 Student 的构建者 (继承自 PersonBuilder)
    public static class StudentBuilder extends Person.PersonBuilder<Student, StudentBuilder> {
        private String school;
        private String studentId;
    
        public StudentBuilder school(String school) { ...; return this; }
        public StudentBuilder studentId(String studentId) { ...; return this; }
    
        @Override
        protected StudentBuilder self() { return this; } // 返回自己
    
        @Override
        public Student build() {
            return new Student(this); // 使用构建者创建实例
        }
    }
    ```

    



### `@Jacksonized`

- `@Jacksonized` 是一个集成注解，它的唯一作用是让 Lombok 生成的 `@Builder` 或 `@SuperBuilder` 与流行的 JSON 库 Jackson 的**反序列化**（JSON 字符串转换为 Java 对象）过程兼容



#### 1. 它解决了什么问题？

- 默认情况下，当你尝试用 Jackson 反序列化一个 JSON 到一个使用 `@Builder` 的类时，会失败并抛出异常

  这是因为 Jackson 默认需要一个**无参构造函数**或者一个明确用 `@JsonCreator` 标注的构造函数/静态工厂方法来实例化对象

- 而 `@Builder` 生成的类，其构造函数通常是 `private` 或 `package-private` 的，并且参数是构建者类，Jackson 并不认识这个模式

  > `@Jacksonized` 就是来解决这个问题的



#### 2. 作用与核心思想

- `@Jacksonized` 会指示 Lombok 在生成构建者代码的同时，额外生成一个**私有的、静态的工厂方法**，并在这个方法上添加 Jackson 的 `@JsonCreator` 注解
  - 这个生成的工厂方法会接收一个由 Jackson 创建的、填满了 JSON 数据的构建者实例，然后调用其 `build()` 方法来返回最终的对象实例



#### 3. 位置

- **在 `@Builder` 或 `@SuperBuilder` 注解的上方**，不能写到下方

  它的位置很重要，必须直接修饰这两个构建者注解之一



#### 4. 常用属性（详细讲解）

- `@Jacksonized` 本身是一个标记注解，它**没有任何可配置的属性**。它的存在本身就是一个开关，用来告诉 Lombok 生成与 Jackson 兼容的代码



#### 5. 示例代码

##### 场景: 将 JSON 反序列化为一个使用 `@Builder` 的不可变对象

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.Builder;
import lombok.ToString;
import lombok.Value;
import lombok.extern.jackson.Jacksonized;

// 注意 @Jacksonized 在 @Builder 的上方
@Jacksonized
@Builder
@Value // 创建一个不可变的类
@ToString
public class User {
    String name;
    int age;

    @Builder.Default
    boolean active = true; // 演示与 @Builder.Default 的兼容性
}

class Main {
    public static void main(String[] args) throws Exception {
        // 假设我们有这样一个 JSON 字符串，注意它没有 'active' 字段
        String jsonString = "{\"name\":\"Alice\",\"age\":30}";

        ObjectMapper objectMapper = new ObjectMapper();

        // 进行反序列化
        User user = objectMapper.readValue(jsonString, User.class);

        // 打印结果，可以看到 active 字段正确地使用了默认值
        System.out.println(user);
        // 输出: User(name=Alice, age=30, active=true)
    }
}
```

- **如果没有 `@Jacksonized` 会发生什么？** 

  程序会抛出类似 `JsonMappingException: Cannot construct instance of 'User'` 的异常，因为 Jackson 找不到创建 `User` 对象的途径。

- **生成的部分核心代码:** 

  - Lombok 看到 `@Jacksonized` 后，会在 `User` 类中生成类似下面的代码：

    ```java
    public class User {
        // ... 其他字段和由 @Value 生成的方法 ...
    
        // 这是由 @Jacksonized 触发生成的关键方法
        @com.fasterxml.jackson.annotation.JsonCreator
        private static User fromJson(UserBuilder builder) {
            return builder.build();
        }
        
        // ... 由 @Builder 生成的 UserBuilder 内部类 ...
    }
    ```





### `@Singular`

- 是一个**专门用来处理集合类型字段**的注解

- `@Singular` 是一个辅助注解，它**不能**单独使用，**必须与 `@Builder` 或 `@SuperBuilder` 配合**

  它的作用是为被注解的**集合类型字段**生成更友好、更符合直觉的“添加入口”

- 通常情况下，对于一个集合字段 ( 如 `List<String> items`) ，

  `@Builder` 只会生成一个接受整个集合的方法(`items(List<String> items)`)

  而 `@Singular` 会在此基础上，额外生成一个一次只添加**单个元素**的方法，以及一个清空集合的方法



#### 1. 作用与核心思想

- **生成单数添加方法**: 

  - 为集合生成一个 "singular"（单数）形式的添加方法，让你可以在链式调用中逐个添加元素

    > 方法名也会体现“单数”或“复数”

- **生成复数添加方法**: 

  - 保留一个 "plural"（复数）形式的方法，用于一次性设置整个集合

    > 方法名也会体现“单数”或“复数”

- **生成清空方法**: 

  - 提供一个 `clear` 方法，用于清空已添加的所有元素

- **自动初始化**:

  - 它能确保集合被正确地初始化，避免了 `NullPointerException`



#### 2. 位置

- **字段**: 只能用在集合类型的字段上，并且该字段所在的类必须使用了 `@Builder` 或 `@SuperBuilder`



#### 3. 常用属性（详细讲解）

##### `value`

- **类型**: `String`

- **作用**: 用于自定义生成的**单数添加方法**的名称

- **默认行为**: 

  - Lombok 会尝试根据字段名自动推断单数形式

    例如，字段 `items` -> 方法 `item()`;  字段 `tasks` -> 方法 `task()`;  字段 `people` -> 方法 `person()`

    Lombok 内置了一个推断规则库，但它并不总是完美

- **使用场景**: 

  - 当 Lombok 的自动推断不符合你的预期时，或者你想使用一个更具表现力的名称时，就可以用这个属性来强制指定

    例如，字段 `memberList`，Lombok 可能会推断为 `memberList()`，此时你可以通过 `@Singular("member")` 将其指定为 `member()`



#### 4. 注意

- Lombok 的 `@Singular` 注解在工作时，会尝试自动推断你集合字段名的“单数”形式，但它无法识别 `a1` 这种非标准英文名词的单数是什么，如果命名不好，编译器会爆红

  当你的集合字段名不是一个 Lombok 能识别的标准英文复数单词时，就需要用 `@Singular("你想要的方法名")` 的方式来帮助它



#### 5. 示例代码

##### 示例 1: 基础用法 (List)

```java
import lombok.Builder;
import lombok.Singular;
import java.util.List;
import java.util.Set;

@Builder
public class Project {
    private String name;

    @Singular
    private Set<String> members;

    @Singular("task") // 使用 value 属性自定义单数名称
    private List<String> tasks;
}
```

- **如何使用:**

  ```java
  Project project = Project.builder()
          .name("Lombok Project")
          .member("Alice")      // 使用生成的单数方法
          .member("Bob")        // 可以多次调用
          .task("Design API")   // 使用自定义的单数方法
          .task("Implement feature")
          .build();
  
  // 你仍然可以使用复数方法一次性设置
  // List<String> initialTasks = Arrays.asList("Task A", "Task B");
  // Project project2 = Project.builder().name("Another Project").tasks(initialTasks).build();
  ```



- **生成的部分核心代码:** 对于 `members` 字段:

  ```java
  public class Project {
      // ...
      public static class ProjectBuilder {
          private java.util.ArrayList<String> members;
          // ...
  
          // 单数添加方法
          public ProjectBuilder member(String member) {
              if (this.members == null) this.members = new java.util.ArrayList<String>();
              this.members.add(member);
              return this;
          }
  
          // 复数设置方法
          public ProjectBuilder members(java.util.Collection<? extends String> members) {
              if (this.members == null) this.members = new java.util.ArrayList<String>();
              this.members.addAll(members);
              return this;
          }
  
          // 清空方法
          public ProjectBuilder clearMembers() {
              if (this.members != null) this.members.clear();
              return this;
          }
  
          // ... build() 方法 ...
      }
  }
  ```

  - *注意：`tasks` 字段会生成类似的代码，只是单数方法名为 `task()`*



##### 示例 2: 用于 Map

- `@Singular` 同样支持 `Map` 类型，它会生成一个接受 `key` 和 `value` 的 `put` 方法

  ```java
  import lombok.Builder;
  import lombok.Singular;
  import java.util.Map;
  
  @Builder
  public class Report {
      @Singular
      private Map<String, Integer> metrics;
  }
  
  // 如何使用:
  // Report report = Report.builder()
  //         .metric("users", 1500)
  //         .metric("sales", 200)
  //         .build();
  ```



## 数据类

### `@Getter` / `@Setter`

#### 作用

- 为类中的字段自动生成标准的 `getter` 和 `setter` 方法



#### 位置

- 用在**字段**上：只为当前字段生成
- 用在**类**上：为类中所有非静态字段生成



#### 常用属性

- `value` 或 `AccessLevel`: 

  - 用于指定生成方法的访问级别（`PUBLIC`, `PROTECTED`, `PACKAGE`, `PRIVATE`）
  - 默认是 `PUBLIC`

- `onMethod_`: 

  - 在生成的**方法**上添加其他注解
  - 例如，`@Getter(onMethod_ = @Deprecated)` 会在生成的 `getX()` 方法上添加 `@Deprecated` 注解。

- `onParam_`: 

  - 在生成的 `setter` **方法参数**上添加注解

    > 只有`@Setter`有这个属性

  - 例如，`@Setter(onParam_ = @NonNull)` 会在 `setX(T x)` 的参数 `x` 前添加 `@NonNull` 注解。



#### Lombok 生成的等效代码

```java
public class User {

    private String username;
    private int age;

    // @Getter 为 username 生成
    public String getUsername() {
        return this.username;
    }

    // @Setter 为 username 生成
    public void setUsername(String username) {
        this.username = username;
    }

    // @Getter 为 age 生成
    public int getAge() {
        return this.age;
    }

    // @Setter(AccessLevel.PROTECTED) 为 age 生成
    protected void setAge(int age) {
        this.age = age;
    }
}
```



#### 使用示例

```java
@Getter // 为所有字段生成 public getter
@Setter // 为所有字段生成 public setter
public class User {

    private String username;

    @Setter(AccessLevel.PROTECTED) // 单独为 age 字段的 setter 指定为 protected
    private int age;
}
```



### `@ToString`

#### 作用

- 自动重写 `toString()` 方法，输出类名和所有**非静态字段**的名称及值，方便调试和日志记录



#### 位置

- 是一个类级注解



#### 常用属性

##### `exclude`

> 黑名单，从`toString()`中排除出去指定字段

- **类型/默认**：`String[]`，默认空
- **作用**：把指定字段**排除**出 `toString`
- **使用场景**：少数字段是敏感/臃肿（如 `password`、大数组）
- **注意**：与 `of` **互斥**
- **示例**：`@ToString(exclude = {"password", "token"})`



##### `of`

> 白名单：`toString()`只包含这些字段

- **类型/默认**：`String[]`，默认空
- **作用**：只输出这些字段，其它一律不打印
- **使用场景**：类很大，只想展示“身份特征”那几项（如 `id`、`email`）
- **注意**：与 `exclude` **互斥**；后续新增字段**不会自动**出现在 `toString` 里
- **示例**：`@ToString(of = {"id", "email"})`



##### `callSuper`

> 是否在最终字符串的最后拼接父类的 `toString()`生成的字符串

- **类型/默认**：`boolean`，默认 `false`
- **作用**：为 `true` 时，把 `super.toString()` 拼接在**字符串的前面**
- **使用场景**：继承体系里，父类 `toString()` 提供了有价值的信息（如审计字段）
- **权衡**：父类输出若过长/含敏感信息，可能不适合开启
- **示例**：`@ToString(callSuper = true)`



##### `includeFieldNames`

> 是否在字符串中拼进去 "字段名="

- **类型/默认**：`boolean`，默认 `true`
- **作用**：`true` 打印 `id=1`；`false` 只打印 `1`
  - `true`对应的`toString()`字符串示例：`User(id=1, email=alice@example.com, age=18)`
  - `false`对应的`toString()`字符串示例：`User(1, alice@example.com, 18)`
- **使用场景**：日志对长度敏感、且字段顺序已经固定时可设为 `false`
- **权衡**：去掉字段名可读性下降，排序尤为重要
- **示例**：`@ToString(of = {"id", "email"}, includeFieldNames = false)`



##### `doNotUseGetters`

> 是否绕过 getter，个人感觉不太常用

- **类型/默认**：`boolean`，默认 `false`（**默认会调用 getter**）
- **作用**：`true` 时直接读字段值，不调用 getter
- **使用场景**：避免**懒加载**、**副作用**或**昂贵计算**（例如 JPA 的 LAZY、缓存命中统计等）在 `toString()` 中被意外触发
- **权衡**：如果 getter 做了格式化/脱敏，绕过它会丢失这些处理
- **示例**：`@ToString(doNotUseGetters = true)`



##### `onlyExplicitlyIncluded`

> 显式包含模式

- **类型/默认**：`boolean`，默认 `false`

- **作用**：`true` 时进入“只认标注”的模式：**只有**带 `@ToString.Include` 的成员会被输出

- **使用场景（推荐现代用法）**：需要**精确控制**输出项、显示名与排序；团队希望“显式即文档”

- **注意**：开启后**不要再用** `of/exclude`；新增字段默认不输出，需手动加 `@ToString.Include`

- **示例**：

  ```java
  @ToString(onlyExplicitlyIncluded = true)
  class User {
    @ToString.Include Long id;
    @ToString.Include(name = "mail") String email;
    @ToString.Exclude String password;
  }
  ```



#### Lombok 生成的等效代码

```java
public class Account extends BaseEntity {
    private Long id;
    private String email;
    private String password;

    @Override
    public String toString() {
        // 因为 callSuper = true, 所以会调用父类的 toString()
        return "Account(super=" + super.toString() + ", id=" + this.id + ", email=" + this.email + ")";
    }
}
```



#### 使用示例

```java
@ToString(exclude = "password", callSuper = true)
public class Account extends BaseEntity {
    private Long id;
    private String email;
    private String password; // 此字段将被排除
}
```



### `@EqualsAndHashCode`

#### 作用

- 自动重写 `equals(Object other)` 和 `hashCode()` 方法。默认使用所有非静态、非瞬态（`transient`）的字段进行比较



#### 位置

- **类**



#### 常用属性

##### `exclude`

> 黑名单：把这些字段**排除**在相等性与哈希计算之外

- **类型/默认**：`String[]`，默认空
- **作用**：这些字段**不参与** `equals`/`hashCode`
- **使用场景**：体积大/易变/无关身份的字段（如 `password`、`lastUpdated`、缓存/统计类字段）
- **注意**：与 `of` **互斥**；更稳的方式是直接在字段上标 `@EqualsAndHashCode.Exclude`（重命名字段更安全）
- **示例**：`@EqualsAndHashCode(exclude = {"password", "lastUpdated"})`



##### `of`

> 白名单：**只**用这些字段来判断相等与计算哈希

- **类型/默认**：`String[]`，默认空
- **作用**：仅列出的字段参与 `equals`/`hashCode`，其余一律忽略
- **使用场景**：领域对象很多字段，但“身份”只由少数键决定（例如业务主键 `id`、`email`）
- **注意**：与 `exclude` **互斥**；后续新增字段**不会自动**纳入，需要你手动维护白名单
- **示例**：`@EqualsAndHashCode(of = {"id", "email"})`



##### `callSuper`

> 是否把**父类**也纳入相等/哈希计算（会调用 `super.equals` 与 `super.hashCode`）

- **类型/默认**：`boolean`，默认 `false`
- **作用**：`true` 时，先比较父类，再比较当前类字段
- **使用场景**：继承体系里，父类的状态**确实**是对象身份的一部分
- **权衡**：只有当父类**正确实现**了 `equals`/`hashCode` 时才建议开启；否则可能破坏对称性/传递性
- **示例**：`@EqualsAndHashCode(callSuper = true)`



##### `onlyExplicitlyIncluded`

> 显式包含模式：**只**计算被标注 `@EqualsAndHashCode.Include` 的成员

- **类型/默认**：`boolean`，默认 `false`

- **作用**：开启后进入“只认标注”的严格模式，未标注的字段一律不参与

- **使用场景（推荐现代用法）**：需要**精确**声明“相等性边界”，让代码审阅更直观、更不易出错

- **注意**：开启后**不要再用** `of/exclude`；新增字段默认不参与，需要你手动贴 `@Include`

- **示例**：

  ```java
  @EqualsAndHashCode(onlyExplicitlyIncluded = true)
  class Product {
    @EqualsAndHashCode.Include Long id;
    @EqualsAndHashCode.Include String email;
    @EqualsAndHashCode.Exclude String displayName;
  }
  ```



#### Lombok 生成的等效代码

```java
public class Product {
    private Long id;
    private String name;
    private String email;
    private double price;

    @Override
    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Product)) return false;
        Product other = (Product) o;
        if (!Objects.equals(this.id, other.id)) return false;
        if (!Objects.equals(this.email, other.email)) return false;
        return true;
    }

    @Override
    public int hashCode() {
        final int PRIME = 59;
        int result = 1;
        result = result * PRIME + (this.id == null ? 43 : this.id.hashCode());
        result = result * PRIME + (this.email == null ? 43 : this.email.hashCode());
        return result;
    }
}
```



#### 使用示例

```java
// 只使用 id 和 email 字段来判断对象是否相等
@EqualsAndHashCode(of = {"id", "email"})
public class Product {
    private Long id;
    private String name; // 不参与比较
    private String email;
    private double price; // 不参与比较
}
```



### `@Data`

#### 作用

- **组合注解**，它集成了多个常用注解的功能
  - 使用 `@Data` 相当于同时使用了以下5个注解：
    - `@Getter`
    - `@Setter`
    - `@ToString`
    - `@EqualsAndHashCode`
    - `@RequiredArgsConstructor` 



#### 位置

- **类**



#### 常用属性

##### `staticConstructor`

> 生成“私有构造器 + 静态工厂方法”

- **类型**: `String`
- **默认值**: `""` (空字符串，表示不启用)
- **作用**: 这个属性用于触发**静态工厂方法**的生成。当你为它提供一个名称（例如 `"of"` 或 `"newInstance"`) 时，`@Data` 会改变其构造函数的生成策略：
  1. 原本将要生成的 `public` 的 `RequiredArgsConstructor` 会变为 `private`
  2. 额外生成一个 `public static` 的工厂方法，其名称就是你提供的 `staticConstructor` 的值，方法参数与私有构造函数完全相同
- **使用场景**:
  - **封装创建逻辑**: 
    - 强制所有使用者通过一个统一的入口（工厂方法）来创建对象，便于未来在创建过程中添加校验、缓存等逻辑。
  - **改善泛型推断**: 
    - 在某些旧版 Java 中，使用静态工厂方法（如 `User.of("name")`）比构造函数（如 `new User<>("name")`) 能更好地进行泛型类型推断
  - **提升可读性**:
    - `User.of(...)` 在语义上可能比 `new User(...)` 更清晰

- **注意**：

  - 如果类里**已有你自己写的构造器**，`@Data` 就**不会再生成**构造器，相应地**也不会生成静态工厂**
    - 这种情况请改用显式的 `@RequiredArgsConstructor(staticName="of")` / `@AllArgsConstructor(staticName="of")`
  - `@Data` 只打包 `@ToString/@EqualsAndHashCode/@Getter/@Setter/@RequiredArgsConstructor` 的**默认行为**
    - 想自定义这些注解的参数（比如 `@ToString(callSuper=true)`），需要**额外显式标注**相应注解

- **示例**：

  ```java
  import lombok.Data;
  import lombok.NonNull;
  
  @Data(staticConstructor = "of")
  public class Article<T> {
      private final long id;
      @NonNull private String title;
      private T payload;
  }
  
  // 使用
  var a = Article.of(1L, "Hello");  // 构造器是 private，走静态工厂；泛型可被推断
  ```



#### 示例代码

```JAVA
@Data
public class Article {
    private final long id; // final 字段
    @NonNull private String title; // @NonNull 字段
    private String content;
}
```



### `@Value`

#### 作用

- 用于快速创建**不可变类**，不可变类一旦被创建，其内部状态就永远不能被修改，这使得它们在并发编程和函数式编程中非常有用，能有效避免副作用

  它也是一个组合注解，但会强制应用一些不可变特性。

  - **所有字段自动设为 `private final`**: 你无需手动添加这些修饰符，Lombok 会强制执行
  - **类本身自动设为 `final`**: 防止其他类通过继承来改变其行为
  - **`@Getter`**: 为所有字段生成 `public` 的 `getter` 方法
    - **注意：不会生成 `setter`**，因为字段是 `final` 的，不能被修改
  - **`@AllArgsConstructor`**: 
    - 生成一个包含所有字段的公共构造函数，用于在创建对象时一次性初始化所有数据
  - **`@ToString`**: 
    - 生成一个包含所有字段的 `toString()` 方法，方便调试和日志输出
  - **`@EqualsAndHashCode`**: 
    - 生成 `equals()` 和 `hashCode()` 方法，用于正确地比较对象和在哈希集合（如 `HashMap`, `HashSet`）中使用



#### 位置

- **类**



#### 常用属性

- `staticConstructor`: `String` 类型

  这是一个非常有用的属性，如果为其指定一个名称（例如 `"of"`），Lombok 会将自动生成的构造函数变为 `private`，并提供一个指定名称的 `public` 静态工厂方法来创建实例

  这是一种推荐的实践，因为静态工厂方法比构造函数有更明确的名称，可读性更好



#### 示例代码

```java
@Value(staticConstructor = "of")
public class UserProfile {
    String userId;
    String nickname;
}
```



#### Lombok 生成的等效代码

```java
public final class UserProfile {
    private final String userId;
    private final String nickname;

    // 构造函数变为 private
    private UserProfile(String userId, String nickname) {
        this.userId = userId;
        this.nickname = nickname;
    }

    // 提供了 public 的静态工厂方法
    public static UserProfile of(String userId, String nickname) {
        return new UserProfile(userId, nickname);
    }
    
    // ... 此处省略 getters, equals, hashCode, toString ...
}
```





### `@With`

#### 作用

- 是处理**不可变对象**的一个关键工具
  - 它的主要作用是提供一种优雅且安全的方式来“更新”不可变对象的字段

- `@With` 注解会为它所标记的字段生成一个名为 `with<FieldName>()` 的方法

  > `@With` 的工作依赖于一个包含所有字段的构造函数，所以通常会和`@AllArgsConstructor`等能提供全参构造器的注解一起使用

  - 当你调用这个方法时，它并**不会修改当前对象**，而是会返回一个**全新的对象副本**
    - 这个副本的大部分字段值都与原始对象相同，只有你通过 `with` 方法指定的那个字段被更新为了新的值



#### 位置

- **类**: 在类级别使用时，会为该类中所有**非静态**字段生成对应的 `with...()` 方法
- **字段**: 在字段级别使用时，只会为当前这一个字段生成 `with...()` 方法



#### 常用属性

##### `value / AccessLevel`

> 控制生成的 `withXxx(...)` 方法的可见性

- **类型/默认**：`AccessLevel`，默认 `PUBLIC`，可以手动设置`PROTECTED`, `PRIVATE` ......
- **作用**：把 `with` 方法设为 `public/protected/package/private` 等
- **使用场景**：
  - 只允许类内/同包内使用“派生”能力；
  - 对外暴露不可变对象，但限制外部随意创建派生实例。



#### 示例

- 与 `@Value` 结合使用

  > 这是最常见的用法，因为 `@Value` 创建的正是不可变对象

  ```java
  @AllArgsConstructor // @With 需要这个构造器来创建新实例
  @Value
  @With
  public class FlightTicket {
      String passengerName;
      String flightNumber;
      String seat;
      Status status;
  
      public enum Status { BOOKED, CHECKED_IN, BOARDED }
  }
  ```





### `@FieldNameConstants`

#### 作用

- `@FieldNameConstants` 会在你的类中生成一个**内部类**（默认为 `Fields`），这个内部类包含了所有字段的名称常量



#### 位置

- **类**
- **枚举**



#### 常用属性

- **`innerTypeName`**:
  -  `String` 类型，用于自定义生成的内部类的名称
  - 默认值是 `"Fields"`
- **`level`**:
  -  `AccessLevel` 类型，用于指定生成的内部类及其常量的访问级别（如 `PUBLIC`, `PROTECTED`, `PRIVATE`）
  - 默认是 `PUBLIC`
- `asEnum`: 
  - `boolean` 类型，默认为 `false`
  - 如果设为 `true`，则会生成一个枚举（Enum）而不是包含 `public static final String` 的类



#### 示例代码

```java
@FieldNameConstants
public class Student {
    private String studentId;
    private String firstName;
    private String lastName;
}
```



#### Lombok 生成的等效代码

```java
// 编译后生成的 .class 文件反编译后的样子
public class Student {
    private String studentId;
    private String firstName;
    private String lastName;

    // @FieldNameConstants 生成的内部类
    public static final class Fields {
        public static final String studentId = "studentId";
        public static final String firstName = "firstName";
        public static final String lastName = "lastName";
        
        private Fields() {} // 私有构造函数，防止实例化
    }
}
// 使用方式: String field = Student.Fields.studentId;
```



## 访问

### `@Accessors`

- `@Accessors` 是一个用来配置 `getter` 和 `setter` 生成方式的注解

  它允许你偏离标准的 JavaBeans 规范（即 `getX()` 和 `setX(value)`），从而创建出更具可读性和流畅性的 “流式 API”

- 它主要通过三个属性来控制方法的样式：`fluent`、`chain` 和 `prefix`


#### 1. 它解决了什么问题？

- 标准 JavaBean 风格的 `getter` 和 `setter` 方法虽然通用，但在某些场景下显得有些冗长，尤其是当你需要连续设置多个属性时：

  ```java
  User user = new User();
  user.setUsername("test");
  user.setEmail("test@example.com");
  user.setAge(25);
  ```

  - `@Accessors` 可以将上述代码简化成更优雅的链式调用，提高代码的可读性



#### 2. 作用与核心思想

- `@Accessors` 的核心思想是**改变访问器（`getter`/`setter`）的默认行为**

  - **`fluent`**: 让 getter 和 setter 的方法名就是字段名本身，去掉 `get`/`set` 前缀

  - **`chain`**: 让 setter 方法返回 `this`（当前对象实例），而不是 `void`，从而可以进行链式调用

  - **`prefix`**: 在生成方法名时，自动去掉字段名的指定前缀




#### 3. 位置

- **类**: 将配置应用于类中所有的字段
- **字段**: 只将配置应用于当前字段



#### 4. 常用属性

##### `chain`

- **类型**: `boolean`
- **默认值**: `false`
- **作用**: 当设置为 `true` 时，生成的 `setter` 方法的返回类型将是当前对象实例（`this`），而不是 `void`
- **使用场景**: 这是实现链式调用的关键。它允许你写出 `user.setUsername("test").setEmail(...)` 这样的代码



##### `fluent`

- **类型**: `boolean`

- **默认值**: `false`

- **作用**: 当设置为 `true` 时：
  - `getter` 的方法名将直接是字段名，例如 `username()` 而不是 `getUsername()`
  
  - `setter` 的方法名也将直接是字段名，例如 `username("newName")` 而不是 `setUsername("newName")`
  
  - 如果该属性为 `true` 且属性 `chain` 省略，`chain` 则默认为 `true`
  
    > 因为这个机制
  
- **使用场景**: 当你希望创建一个非常简洁的 API，完全隐藏 `get/set` 的概念时使用



##### `prefix`

- **类型**: `String[]` (字符串数组)

- **默认值**: `{}` (空数组)

- **作用**: 

  - 用于指定一个或多个需要从字段名开头移除的前缀
  - Lombok 在生成 `getter`/`setter` 方法名时，会先检查字段名是否以这些前缀开头，如果是，则移除前缀后再生成方法名

- **使用场景**: 

  - 在处理一些有特定命名规范的遗留系统或数据库映射时非常有用

    例如，如果所有字段都以 `f_` 或 `_` 开头（如 `_name`, `f_email`），你可以通过这个属性将它们映射到 `name()`/`setName()` 和 `email()`/`setEmail()`




#### 5. 示例代码

##### 示例 : `fluent = true` 

```JAVA
import lombok.Getter;
import lombok.Setter;
import lombok.experimental.Accessors;

@Getter
@Setter
@Accessors(fluent = true)//这里如果没有指定chain，会默认会让chain=true
public class UserFluentChain {
    private String username;
    private String email;
}

// 如何使用 (最流畅的 API):
// UserFluentChain user = new UserFluentChain()
//         .username("fluentAndChained")
//         .email("fluentchain@example.com");
```

- **生成的 Setter 代码:**

  ```JAVA
  public UserFluentChain username(String username) {
      this.username = username;
      return this; // 方法名是字段名，且返回 this
  }
  ```



##### 示例 : 使用 `prefix`

```JAVA
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import lombok.experimental.Accessors;

@Getter
@Setter
@Accessors(prefix = {"m", "_"}) // 移除 "m" 或 "_" 前缀
@ToString
public class LegacyUser {
    private String mUsername;
    private String _email;
    private int age; // 没有前缀，不受影响
}

// 如何使用:
// LegacyUser user = new LegacyUser();
// user.setUsername("legacy"); // 对应 mUsername 字段
// user.setEmail("legacy@example.com"); // 对应 _email 字段
// System.out.println(user.getUsername());
```



### `@FieldDefaults`

- `@FieldDefaults` 是一个代码风格注解，它的主要作用是为类中的字段设置一个统一的默认访问级别和 `final` 修饰符

  当你希望类中的大部分或所有字段都具有相同的修饰符（例如，都是 `private final`）时，这个注解可以帮你消除大量重复的样板代码，让实体类或数据类的定义更加简洁

#### 1. 它解决了什么问题？

- 在一个典型的 Java 类中，你通常需要为每个字段都显式地声明访问修饰符，比如 `private`。如果字段是不可变的，还需要加上 `final`。当字段很多时，代码会显得非常重复和冗长：

  ```java
  public class User {
      private final String userId;
      private String username;
      private String email;
      private int age;
      // ... 大量重复的 'private'
  }
  ```

- `@FieldDefaults` 可以让你只声明一次，然后将其应用于所有未显式指定修饰符的字段



#### 2. 作用与核心思想

- `@FieldDefaults` 的核心思想是**为字段设置默认的修饰符**。它通过两个属性来控制：

  - **`level`**: 为所有没有显式访问修饰符（`public`, `protected`, `private`）的字段设置一个默认的访问级别

  - **`makeFinal`**: 将所有未标注 `@NonFinal` 的的字段都设置为 `final`


- **重要规则**: `@FieldDefaults` **不会**覆盖字段上已经存在的显式修饰符



#### 3. 位置

- **类**



#### 4. 常用属性

##### `level`

- **类型**: `lombok.AccessLevel` (枚举)
- **默认值**: `AccessLevel.NONE` (不做任何改动)
- **作用**: 这个属性用来设定类中字段的**默认访问级别**。Lombok 只会为那些你**没有**手动添加访问修饰符的字段应用这个级别
- **可选值**:
  - `PUBLIC`: `public`，公开的
  - `PROTECTED`: `protected`，受保护的
  - `PACKAGE`: 包级私有（即没有任何访问修饰符）
  - `PRIVATE`: `private`，私有的。（这是最常用的值）
- **使用场景**: 在编写实体类、DTO 或任何数据容器类时，字段通常都应该是 `private` 的。使用 `level = AccessLevel.PRIVATE` 可以让你免去为每个字段都写 `private` 的麻烦



##### `makeFinal`

- **类型**: `boolean`

- **默认值**: `false`

- **作用**: 这是一个开关。

  - 当设置为 `true` 时，Lombok 会将类中所有**非 `final`** 的字段（无论这些字段是你手动声明的还是通过 `@FieldDefaults(level=...)` 设置的）都强制设置为 `final`

- **使用场景**: 

  - 当你致力于编写**不可变对象**时，这个属性非常有用

    通常与 `level = AccessLevel.PRIVATE` 结合使用，可以轻松创建出所有字段都是 `private final` 的类




#### 5. 示例代码

##### 示例 1: 设置所有字段默认为 `private`

```java
import lombok.experimental.FieldDefaults;
import lombok.AccessLevel;

@FieldDefaults(level = AccessLevel.PRIVATE)
public class PrivateUser {
    // 这些字段都会被设置为 private
    String username;
    String email;
    int age;
    
    // 这个字段已经显式声明为 public，所以不受影响
    public String publicProfileUrl;
}
```

- **生成的代码:**

  ```java
  public class PrivateUser {
      private String username;
      private String email;
      private int age;
      public String publicProfileUrl;
  }
  ```



##### 示例 2: 设置所有字段默认为 `private` 和 `final` (创建不可变类)

```java
import lombok.Getter;
import lombok.AllArgsConstructor;
import lombok.experimental.FieldDefaults;
import lombok.AccessLevel;

@AllArgsConstructor // 需要一个构造函数来初始化 final 字段
@Getter
@FieldDefaults(level = AccessLevel.PRIVATE, makeFinal = true)
public class ImmutableProduct {
    String id;
    String name;
    double price;

    // 这个字段已经显式声明为非 final，但 makeFinal=true 会覆盖它吗？
    // 不会，但是如果一个字段是 final，它必须被初始化。
    // 如果 makeFinal = true，所有字段都将是 final。
}
```

- **生成的代码:**

  ```java
  public class ImmutableProduct {
      private final String id;
      private final String name;
      private final double price;
  
      // ... getter 方法 ...
      // ... AllArgsConstructor ...
  }
  ```



## 日志

### `@Slf4j`

- `@Slf4j` 是 Lombok 日志注解系列中**最常用、最推荐**的一个

- 它的作用是自动在类中注入一个 **SLF4J** 的 `Logger` 实例，

  从而免去开发者手动编写 `private static final Logger log = LoggerFactory.getLogger(...)` 这行样板代码的麻烦。

#### 1. 它解决了什么问题？

- 在几乎所有的 Java 应用中，我们都需要记录日志来追踪程序行为和排查问题。

  标准的做法是在每个需要日志的类顶部定义一个 `Logger` 字段：

  ```java
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  
  public class DataProcessor {
      //下面这行代码
      private static final Logger log = LoggerFactory.getLogger(DataProcessor.class);
  
      public void process() {
          log.info("Starting data processing...");
      }
  }
  ```

  

- 这行代码虽然简单，但在项目中每个类里都重复一遍，不仅繁琐，而且在复制粘贴时很容易忘记修改 `DataProcessor.class`，导致日志来源混乱
- `@Slf4j` 注解就是为了彻底解决这个问题



#### 2. 作用与核心思想

- `@Slf4j` 的核心思想是**依赖注入一个日志记录器**

  - 当你给一个类添加 **`@Slf4j`** 注解时，Lombok 会在编译期间自动为这个类生成如下字段：

    ```JAVA
    private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(YourClassName.class);
    ```

  - 这样，你就可以在类的任何（非静态）方法中直接使用名为 `log` 的变量来记录日志了




#### 3. 位置

- **类**



#### 4. 常用属性

##### `topic`

- **类型**:  `String`

- **默认值**:  当前类的完全限定名 (e.g., `"com.example.DataProcessor"`)

- **作用**: 

  - 这个属性允许你自定义生成的 `Logger` 实例的名称

    默认情况下，`Logger` 的名称与它所在的类名绑定，这是最佳实践，因为它允许你在日志配置文件中对每个类进行精细的日志级别控制

- **使用场景**: 

  - 仅在特殊情况下使用

    例如，当你有多个类共同完成一个特定的业务功能（如“支付模块”），并且希望它们都使用同一个名为 `PAYMENT_LOG` 的 `Logger` 时，可以通过 `@Slf4j(topic = "PAYMENT_LOG")` 来实现




#### 5. 示例代码

```JAVA
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class DataProcessor {

    public void process(String data) {
        // 'log' 变量可以直接使用，无需手动定义
        log.info("Received data: {}", data);

        if (data == null || data.isEmpty()) {
            log.warn("Processing aborted: data is empty.");
            return;
        }

        try {
            // ... 复杂的处理逻辑 ...
            log.debug("Data successfully processed.");
        } catch (Exception e) {
            log.error("An error occurred during data processing for data: {}", data, e);
        }
    }
}
```



#### 6. 依赖配置 (重要)

- 为了让 `@Slf4j` 正常工作，你的项目构建工具中必须包含以下依赖：

  1. **SLF4J API**: 这是 SLF4J 的核心接口
  2. **一个 SLF4J 绑定**: SLF4J 只是一个门面，你需要一个具体的日志实现来真正地记录日志，例如 Logback（推荐）或 Log4j2

  **Maven 示例 (使用 Logback):**

```XML
<dependencies>
    <!-- SLF4J API -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>x.x.x</version>
    </dependency>
    <!-- Logback Binding -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>x.x.x</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```



### `@Log4j2`

- `@Log4j2` 是 Lombok 日志注解系列中的一员，专门为使用 **Apache Log4j 2日志框架**的项目服务
  - 它的作用与 `@Slf4j` 类似，都是为了自动在类中注入一个 `Logger` 实例，从而消除手动编写样板代码的繁琐



#### 1. 它解决了什么问题？

- 如果你的项目选择 Log4j 2 作为日志记录工具，那么在每个需要日志的类中，你都需要手动定义一个 `Logger` 字段：

  ```JAVA
  import org.apache.logging.log4j.LogManager;
  import org.apache.logging.log4j.Logger;
  
  public class ReportGenerator {
      //下面这行代码
      private static final Logger log = LogManager.getLogger(ReportGenerator.class);
  
      public void generate() {
          log.info("Generating report...");
      }
  }
  ```

  - 这行 `private static final Logger log = ...` 的代码在每个类中都是重复的

    `@Log4j2` 注解就是为了让你不再需要手动编写它



#### 2. 作用与核心思想

- `@Log4j2` 的核心思想是**自动注入一个 Log4j 2 的日志记录器**

  - 当你给一个类添加 `@Log4j2` 注解时，Lombok 会在编译期间自动为这个类生成如下字段：

    ```java
    private static final org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(YourClassName.class);
    ```

    - 之后，你就可以在类的方法中直接使用名为 `log` 的变量来记录日志了




#### 3. 位置

- **类**



#### 4. 常用属性

##### `topic`

- **类型**: `String`

- **默认值**: 

  - 当前类的完全限定名 (e.g., `"com.example.ReportGenerator"`)

- **作用**: 

  - 这个属性允许你自定义生成的 `Logger` 实例的名称

    默认情况下，`Logger` 的名称与它所在的类名绑定，这使得在 `log4j2.xml` 等配置文件中可以方便地对每个类进行独立的日志级别控制

- **使用场景**: 

  - 仅在特殊情况下使用

    例如，当你有多个类共同负责“数据导出”功能，并希望它们使用同一个名为 `DATA_EXPORT_LOG` 的 `Logger` 时，可以通过 `@Log4j2(topic = "DATA_EXPORT_LOG")` 来实现




#### 5. 示例代码

```java
import lombok.extern.log4j.Log4j2;
import org.springframework.stereotype.Service;

@Log4j2
@Service
public class ReportGenerator {

    public boolean generateReport(String reportName) {
        // 'log' 变量可以直接使用，无需手动定义
        log.info("Starting generation for report: {}", reportName);

        if (reportName == null || reportName.trim().isEmpty()) {
            log.warn("Report generation skipped: report name is blank.");
            return false;
        }

        try {
            // ... 生成报告的复杂逻辑 ...
            log.debug("Report '{}' generated successfully.", reportName);
            return true;
        } catch (Exception e) {
            // Log4j 2 支持在最后一个参数传入 Throwable
            log.error("Failed to generate report '{}'", reportName, e);
            return false;
        }
    }
}
```



#### 6. 依赖配置

- 为了让 `@Log4j2` 正常工作，你的项目构建工具（如 Maven 或 Gradle）中必须包含 Log4j 2 的核心依赖：

  - **Maven 示例:**

    ```xml
    <dependencies>
        <!-- Log4j 2 API -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>X.X.X</version>
        </dependency>
        <!-- Log4j 2 Core Implementation -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>X.X.X</version>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
    ```

    



### `@Log`

- `@Log` 是 Lombok 日志注解系列中的一员，它专门为使用 Java 标准库自带的日志框架 **`java.util.logging` (JUL)** 的项目服务

  与系列中其他日志注解一样，它的作用是自动在类中注入一个 `Logger` 实例，以消除手动编写样板代码的繁琐

- 我讨厌这个，我最喜欢 **Logback** 嘻嘻



#### 1. 它解决了什么问题？

- 如果你的项目决定使用 `java.util.logging`，那么在每个需要日志的类中，你都需要手动定义一个 `Logger` 字段：

  ```java
  import java.util.logging.Logger;
  
  public class LegacyService {
      private static final Logger log = Logger.getLogger(LegacyService.class.getName());
  
      public void execute() {
          log.info("Executing legacy service...");
      }
  }
  ```

  - 这行 `private static final Logger log = ...` 的代码在每个类中都是重复的，并且获取 `Logger` 的方式（使用类名字符串 `class.getName()`）也略显不同

    `@Log` 注解就是为了让你不再需要手动编写它



#### 2. 作用与核心思想

- `@Log` 的核心思想是**自动注入一个 `java.util.logging.Logger` 实例**

  - 当你给一个类添加 `@Log` 注解时，Lombok 会在编译期间自动为这个类生成如下字段：

    ```java
    private static final java.util.logging.Logger log = 
        java.util.logging.Logger.getLogger(YourClassName.class.getName());
    ```

    - 之后，你就可以在类的方法中直接使用名为 `log` 的变量来记录日志了	




#### 3. 位置

- **类**



#### 4. 常用属性

##### `topic`

- **类型**: `String`

- **默认值**: 当前类的完全限定名字符串 (e.g., `"com.example.LegacyService"`)

- **作用**: 

  - 这个属性允许你自定义生成的 `Logger` 实例的名称

    默认情况下，`Logger` 的名称与它所在的类名绑定，这使得在 JUL 的配置文件中可以方便地对每个 Logger 进行独立的日志级别控制。

- **使用场景**: 

  - 仅在特殊情况下使用

    例如，当你有多个类共同负责一个功能，并希望它们使用同一个 Logger 名称时，可以通过 `@Log(topic = "com.example.mymodule")` 来实现




#### 5. 示例代码

```java
import lombok.extern.java.Log;

@Log
public class LegacyService {

    public void execute(String taskId) {
        // 'log' 变量可以直接使用，无需手动定义
        log.info("Executing task: " + taskId);

        if (taskId == null) {
            // JUL 的日志级别和 SLF4J 等略有不同
            log.warning("Task ID is null, execution may fail.");
        }

        try {
            // ... 业务逻辑 ...
            log.fine("Task " + taskId + " completed successfully.");
        } catch (Exception e) {
            // 记录异常
            log.log(java.util.logging.Level.SEVERE, "Failed to execute task " + taskId, e);
        }
    }
}
```