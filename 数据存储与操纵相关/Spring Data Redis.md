# 一些核心概念

## 1. Spring Data Redis (SDR) 简介

### 1.1 SDR 是什么？

- Spring Data Redis (简称 SDR) 是 Spring Data 项目家族中的一个核心成员
  - 它提供了一个统一的、高级的编程模型，用于在 Spring 框架（特别是 Spring Boot）中访问和操作 Redis 数据库

- SDR 的核心目标是**简化 Spring 应用与 Redis 之间的数据访问**

- 需要明确的是，SDR **不是**一个新的 Redis 客户端

  它是一个构建在现有 Redis 客户端（如 Lettuce 或 Jedis）之上的 **抽象层**

  它让开发者可以专注于业务逻辑，而不用处理底层的连接管理、数据转换和异常处理等繁琐细节



### 1.2 SDR 解决了什么核心问题？

- SDR 通过封装底层的 Redis 客户端，主要解决了以下几个在 Java 开发中非常棘手的问题：


#### 1. 连接管理

###### 使用Jedis或Lettuce

- 直接使用 Jedis 或 Lettuce 时，开发者需要手动配置和管理连接池（例如 `GenericObjectPoolConfig`）

  这包括设置最大连接数、最小空闲数、连接超时、连接测试等，配置复杂且容易出错

- 示例

  ```java
  import io.lettuce.core.RedisClient;
  import io.lettuce.core.api.StatefulRedisConnection;
  import io.lettuce.core.api.sync.RedisCommands;
  import com.fasterxml.jackson.databind.ObjectMapper;
  
  public class UserService {
      
      public void saveUserSession(User user) throws Exception {
          // 1. 手动创建连接
          RedisClient client = RedisClient.create("redis://localhost:6379");
          StatefulRedisConnection<String, String> connection = client.connect();
          RedisCommands<String, String> commands = connection.sync();
          
          // 2. 手动序列化对象
          ObjectMapper mapper = new ObjectMapper();
          String userJson = mapper.writeValueAsString(user);
          
          // 3. 存储数据
          commands.set("user:session:" + user.getId(), userJson);
          commands.expire("user:session:" + user.getId(), 1800);  // 30分钟过期
          
          // 4. 手动关闭连接（忘记就泄漏！）
          connection.close();
          client.shutdown();
      }
      
      public User getUserSession(Long userId) throws Exception {
          RedisClient client = RedisClient.create("redis://localhost:6379");
          StatefulRedisConnection<String, String> connection = client.connect();
          RedisCommands<String, String> commands = connection.sync();
          
          // 手动反序列化
          String json = commands.get("user:session:" + userId);
          ObjectMapper mapper = new ObjectMapper();
          User user = mapper.readValue(json, User.class);
          
          connection.close();
          client.shutdown();
          
          return user;
      }
  }
  ```

  - **问题清单：**

    - ⚠️ 每次操作都要创建/关闭连接（**性能差**）

    - ⚠️ 忘记关闭连接会导致**连接泄漏**

    - ⚠️ 手动序列化/反序列化，代码**繁琐且容易出错**

    - ⚠️ 异常处理复杂（`throws Exception`）

    - ⚠️ 代码重复度高，难以维护



###### **SDR 解决**

- SDR（特别是配合 Spring Boot）实现了连接的**自动化配置**

  它会根据 `application.properties` 中的配置（如 `spring.data.redis.lettuce.pool.*`）自动创建和管理底层的连接池，并确保连接的正确获取和释放，开发者基本无需关心这个过程
  
- 示例

  ```java
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.data.redis.core.RedisTemplate;
  import org.springframework.stereotype.Service;
  import java.util.concurrent.TimeUnit;
  
  @Service
  public class UserService {
      
      @Autowired
      private RedisTemplate<String, Object> redisTemplate;
      
      public void saveUserSession(User user) {
          // 一行搞定！自动连接、序列化、释放资源
          redisTemplate.opsForValue().set(
              "user:session:" + user.getId(), 
              user, 
              30, 
              TimeUnit.MINUTES
          );
      }
      
      public User getUserSession(Long userId) {
          // 一行搞定！自动反序列化
          return (User) redisTemplate.opsForValue().get("user:session:" + userId);
      }
  }
  ```



#### 2. 模板抽象

- **问题**：

  - Redis 的命令非常多（如 `SET`, `HGET`, `LPUSH`）

    直接使用客户端 API，虽然灵活，但代码会显得繁琐，且 API 风格与 Spring 应用的其他部分（如 `JdbcTemplate`）不统一

  

- **SDR 解决**：

  - SDR 提供了核心工具 `RedisTemplate` 和 `StringRedisTemplate`
    - 它将 Redis 的命令按数据结构进行了分类封装（例如 `redisTemplate.opsForValue()` 操作 String，`redisTemplate.opsForHash()` 操作 Hash），API 调用更符合 Java 开发者的直觉
      - 这极大简化了日常的数据操作代码量



#### 3. 序列化

- **问题**：

  - Redis 内部存储的是字节数组

    当你想存一个 Java 对象（如一个 `User` 对象）时，必须手动将其转换为字节（序列化）；当取回数据时，又必须手动将字节转回对象（反序列化）

- **SDR 解决**：

  - `RedisTemplate` 内置了强大的序列化机制

    开发者可以配置不同的序列化器（Serializer），SDR 会在数据存取时**自动执行**序列化和反序列化过程

    - **Key的序列化**：通常使用 `StringRedisSerializer`（字符串）
    - **Value的序列化**：推荐使用 `Jackson2JsonRedisSerializer`（JSON格式）



#### 4. 异常转译

- **问题**：

  - 底层的 Redis 客户端（Lettuce/Jedis）会抛出它们自己的特定异常（如连接超时异常、IO异常）

    如果这些异常直接暴露给业务层，会导致业务代码与底层技术实现（例如“我们用的是Lettuce”）产生耦合

- **SDR 解决**：

  - SDR 会捕获所有底层的特定异常（Checked/Unchecked Exceptions），并将它们**转译**为 Spring 框架统一的、与技术无关的数据访问异常，即 `DataAccessException`（这是一个 RuntimeException）
    - 这使得业务代码可以只关心 Spring 的异常体系，实现了更好的解耦，也让事务管理（如果需要）成为可能



### 1.3 SDR在技术栈中的位置

```
┌───────────────────────────────────────────┐
│   你的业务代码（UserService）                │  ← 只关心业务逻辑
│   - saveUser()                            │     写出的代码简洁优雅
│   - getUser()                             │
├───────────────────────────────────────────┤
│   Spring Data Redis (抽象层)               │  ← SDR 在这一层
│   - RedisTemplate                         │     提供统一API
│   - StringRedisTemplate                   │     自动序列化
│   - 连接工厂、序列化器、事务管理               │     异常转译
├───────────────────────────────────────────┤
│   Lettuce/Jedis (驱动层)                   │  ← 底层客户端
│   - 网络通信                                │     负责与Redis Server通信
│   - 连接池管理                              │     管理TCP连接
│   - 协议解析（RESP）                        │
├───────────────────────────────────────────┤
│   Redis Server                            │  ← 数据存储
│   - 内存数据库                              │     真正的数据存储位置
│   - 持久化                                 │
└───────────────────────────────────────────┘
```



## 2. Redis 客户端选择

> 就是从Jedis 和 Lettuce中进行选择

在 Spring Data Redis (SDR) 出现之前，Java 开发者需要直接选择和使用 Redis 客户端库，Jedis 和 Lettuce 是最主流的两个选择

- SDR 是构建在这些客户端之上的抽象层，因此，底层的客户端性能和特性，会直接影响到整个 Spring 应用

- 这边建议直接 **无脑选 Lettuce**



# 代码入门

## 30秒快速上手

想立即开始使用？只需三步：

### 步骤1：添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```



### 步骤2：配置Redis地址

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
```



### 步骤3：注入使用

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

// 存储
stringRedisTemplate.opsForValue().set("user:token:123", "abc123xyz");

// 读取
String token = stringRedisTemplate.opsForValue().get("user:token:123");

// 设置过期时间（10分钟）
stringRedisTemplate.opsForValue().set("verify:code:456", "8888", 10, TimeUnit.MINUTES);
```

✅ **完成！**

---

## Starter 依赖解析

### 1. 这一个依赖做了什么？

`spring-boot-starter-data-redis` 会自动帮你引入：

| 组件                     | 说明                                                        |
| ------------------------ | ----------------------------------------------------------- |
| `spring-data-redis`      | 核心库                                                      |
| `lettuce-core`           | 默认客户端（推荐，支持异步）                                |
| `commons-pool2`          | 连接池实现                                                  |
| `RedisAutoConfiguration` | 自动配置类（创建 ConnectionFactory、RedisTemplate 等 Bean） |



### 2. 为什么不需要手动添加 Lettuce？

Spring Boot 的 "约定优于配置" 原则已经帮你选好了：

- ✅ **默认使用 Lettuce**（高性能、支持异步、更现代）
- ❌ 不推荐 Jedis（老旧、同步阻塞）

> **💡 提示**：如果你坚持要用 Jedis，需要排除 Lettuce 依赖并手动添加 Jedis，但这通常没有必要

---



## 配置详解

### 1. 单机模式配置（最常用）

```yaml
spring:
  data:
    redis:
      # 基础连接
      host: 127.0.0.1
      port: 6379
      password: your_password  # 没有密码可以不配置
      database: 0              # 使用的数据库索引（0-15）
      
      # 超时和SSL
      timeout: 5s              # 连接超时
      ssl: false               # 是否启用SSL
      
      # Lettuce 连接池配置（重要！）
      lettuce:
        pool:
          max-active: 20       # 最大连接数（根据并发量调整）
          max-idle: 10         # 最大空闲连接数
          min-idle: 5          # 最小空闲连接数（保持预热）
          max-wait: 2s         # 获取连接最大等待时间
```



### 2. 哨兵模式配置（高可用）

```yaml
spring:
  data:
    redis:
      # 注意：使用哨兵时，不配置 host 和 port
      password: your_password
      sentinel:
        master: my-master-name  # 主节点名称
        nodes:                   # 哨兵节点列表
          - 192.168.1.10:26379
          - 192.168.1.11:26379
          - 192.168.1.12:26379
```



### 3. 集群模式配置（分片）

```yaml
spring:
  data:
    redis:
      # 注意：使用集群时，不配置 host、port 和 sentinel
      password: your_password
      cluster:
        nodes:                   # 集群节点列表
          - 192.168.1.20:7001
          - 192.168.1.21:7002
          - 192.168.1.22:7003
        max-redirects: 3         # 集群重定向次数
```



### 4. 连接池参数调优指南

| 参数         | 推荐值              | 说明                           |
| ------------ | ------------------- | ------------------------------ |
| `max-active` | 20-50               | 根据压测结果调整，过大浪费资源 |
| `max-idle`   | `max-active` 的 50% | 保持足够的空闲连接             |
| `min-idle`   | 5-10                | 预热连接，避免冷启动           |
| `max-wait`   | 2s-5s               | 超时时间不宜过长               |



## 序列化方案选择⭐

### 1. 三种方案对比

| **方案**                 | **Key序列化** | **Value序列化** | **支持的Redis结构**      | **可读性** | 性能   | **推荐度**   |
| ------------------------ | ------------- | --------------- | ------------------------ | ---------- | ------ | ------------ |
| **默认 RedisTemplate**   | Jdk           | Jdk             | 全支持                   | ❌ 乱码     | ⚠️ 一般 | ⭐ **不推荐** |
| **StringRedisTemplate**  | String        | String          | **全支持 (Hash/List等)** | ✅ 纯文本   | ✅ 高   | ⭐⭐⭐⭐         |
| **自定义 RedisTemplate** | String        | JSON            | 全支持                   | ✅ JSON     | ✅ 高   | ⭐⭐⭐⭐⭐        |

> **注：** `StringRedisTemplate` 的 Value 类型为 String，指的是 **传入 Java 方法的数据** 必须是 String，但存入 Redis 时依然可以使用 List、Hash 等复杂结构



### 2. 快速决策树

```
需要存储对象（User、Order等）？
├─ 是 → 使用自定义 RedisTemplate<String, Object>  （方案3）
└─ 否 → 只存简单数据（Token、计数器）？
       └─ 是 → 使用 StringRedisTemplate  （方案2）
```

------



### 3. 方案一：默认 RedisTemplate

>❌ 不推荐

#### 3.1 定位

`RedisTemplate` 是 SDR 中最核心的类，它是 Spring Data Redis 提供的一个 **通用** 操作模板，它封装了所有 Redis 连接和数据操作

- 它提供了对 Redis 五种基本数据结构（String, Hash, List, Set, ZSet）以及其他高级数据结构（如 Geo, HyperLogLog）的便捷操作方法（通过 `opsForValue()`, `opsForHash()` 等）
- 它的设计目标是让你能够将任意 Java 对象（如 `User` 对象、`Product` 对象）作为 Key 或 Value 存入 Redis



#### 3.2 **问题所在**

Spring Boot 自动配置的这个`RedisTemplate<Object, Object>` 默认使用序列化器:  `JdkSerializationRedisSerializer`

- 它使用 Java 的标准序列化机制，将你的 Key 和 Value 转换成字节数组。会带来问题

  - **可读性差**

    - 当你使用 Redis 客户端（如 `redis-cli`）查看时，你看到的不是明文字符串，而是一长串以 `\xac\xed\x00\x05...` 开头的二进制数据

      - 示例

        ```JAVA
        // 存储一个User对象
        redisTemplate.opsForValue().set("user:1", new User("Alice", 25));
        ```

        在 redis-cli 中看到的是：

        ```bash
        > GET user:1
        "\xac\xed\x00\x05t\x00\x04..."  # 完全不可读的二进制数据！
        ```

  - **跨语言性差**: 只有 Java 应用（并且需要有对应的类定义）才能反序列化这些数据



#### 3.3 **为什么不推荐**

1. ❌ **可读性差** - 无法调试和排查问题
2. ❌ **不跨语言** - Python、Node.js 无法读取
3. ❌ **安全风险** - 存在反序列化漏洞
4. ❌ **体积大** - 序列化后的数据较大

> **结论：生产环境禁止使用默认 RedisTemplate！**

------



### 4. 方案二：StringRedisTemplate

>✅ 适合简单场景

- **`RedisTemplate` 的特化子类**：
  - **`StringRedisTemplate`** 继承自 `RedisTemplate`
  - **设计初衷**：它预设了 Key 和 Value 的序列化器均为 `StringRedisSerializer`



#### 4.1 核心特点

- **序列化策略**：Key 和 Value 都会被当作 **字符串** 处理
- 序列化器：固定使用 `StringRedisSerializer`
- 存储格式：UTF-8 编码的纯文本字符串
- **工作方式**: 
  - **序列化 (Java -> Redis)**：它使用指定的字符集（默认为 UTF-8）将 Java 的 `String` 对象转换为字节数组
  - **反序列化 (Redis -> Java)**：它使用相同的字符集将从 Redis 读取的字节数组转回 Java `String` 对象
- **⭐ 核心误区纠正：** 很多人误以为 `StringRedisTemplate` 只能操作 Redis 的 String（字符串）类型。 **这是错误的！**
  - 它 **支持 Redis 的所有数据类型**（Hash, List, Set, ZSet）
  - **区别在于**：它要求你存入 List 或 Hash 等 **容器中** 的 **具体元素内容** 必须是字符串（String）




#### 4.2 优点和局限性

##### 优点

1. **可读性极高**：
   - 存入 Redis 的数据就是 UTF-8 编码的纯文本字符串。你可以直接在 `redis-cli` 中查看到明文数据，非常利于调试
2. **跨语言通用**：任何编程语言（Python, Node.js, PHP）都可以轻松读写 UTF-8 字符串，实现了完美的跨平台兼容
3. **零配置**：Spring Boot 默认提供，开箱即用
4. **安全且高效**：没有 Java 序列化漏洞，且字符串转换的性能很高



##### 局限性

`StringRedisTemplate` 的局限性在于它 **无法自动** 处理 Java 非字符串对象（POJO）与字符串（特别是 JSON）之间的转换

- 它**不仅限于 String 结构，但仅限于 String 内容**：
  - 如果要存一个 `User` 对象到 List 中，你必须先手动把 User 转成 JSON 字符串
  - 相比 `自定义 RedisTemplate`，多了一步手动序列化的代码



#### 4.3 适用场景

**✅ 推荐使用：**

- 存储 Token、Session ID
- 计数器、排行榜
- 验证码、配置项
- 手动序列化的 JSON 字符串

**❌ 不适合：**

- 需要自动序列化对象的场景（太繁琐）



#### 4.4 代码示例

**场景1：存储简单字符串**

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

// 存储Token
stringRedisTemplate.opsForValue().set("user:token:123", "abc123xyz", 30, TimeUnit.MINUTES);

// 计数器
stringRedisTemplate.opsForValue().increment("page:view:count");
Long views = Long.parseLong(stringRedisTemplate.opsForValue().get("page:view:count"));
```



**场景2：存储JSON（需手动序列化）**

```java
@Autowired
private ObjectMapper objectMapper; // Jackson

@Autowired
private StringRedisTemplate stringRedisTemplate;

// 存储
User user = new User("Alice", 25);
String userJson = objectMapper.writeValueAsString(user);
stringRedisTemplate.opsForValue().set("user:1", userJson);

// 读取
String json = stringRedisTemplate.opsForValue().get("user:1");
User user = objectMapper.readValue(json, User.class);
```

**redis-cli 中查看：**

```bash
> GET user:token:123
"abc123xyz"

> GET user:1
"{\"name\":\"Alice\",\"age\":25}"  # 可读的JSON！
```



**场景3：使用 StringRedisTemplate 操作复杂结构（Hash/List）**

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

public void complexStructureTest() {
    // 1. 操作 Hash (存对象属性，所有值必须转为 String)
    stringRedisTemplate.opsForHash().put("user:1001", "name", "ZhangSan");
    stringRedisTemplate.opsForHash().put("user:1001", "age", "25"); // 数字也要转 String
    
    // 2. 操作 List (做消息队列)
    stringRedisTemplate.opsForList().leftPush("task:queue", "task_1");
    stringRedisTemplate.opsForList().leftPush("task:queue", "task_2");
    
    // 3. 操作 Set (做标签)
    stringRedisTemplate.opsForSet().add("product:tags:1", "hot", "sale");
}
```



### 5. 方案三：自定义 RedisTemplate⭐

>⭐ 推荐

- 我们发现`RedisTemplate` 默认使用 JDK 序列化（可读性差、不安全），`StringRedisTemplate` 只能处理字符串（对象操作繁琐）
  - 为了解决这些问题，最佳实践是**自定义一个 `RedisTemplate` Bean**，使其兼具强大的功能和良好的可读性



#### 5.1 设计目标

目标是创建一个 `RedisTemplate<String, Object>` 类型的 Bean，并为其配置：

1. **Key 序列化器 (KeySerializer)**: 使用 `StringRedisSerializer`
   - 确保所有 Key 都是可读的字符串，方便调试
   
   
   
2. **Value 序列化器 (ValueSerializer)**: 使用 `Jackson2JsonRedisSerializer`
   - 确保所有非字符串的 Value（如 `User` 对象）都被自动序列化为 **JSON 字符串** 存储
   - JSON 格式可读性强、跨语言通用
   
   
   
3. **Hash Key 序列化器 (HashKeySerializer)**: 使用 `StringRedisSerializer`
   - 确保 Hash 结构中的 Field 是可读的字符串
   
   
   
4. **Hash Value 序列化器 (HashValueSerializer)**: 使用 `Jackson2JsonRedisSerializer`
   - 确保 Hash 结构中的 Value 是 JSON 字符串



#### 5.2 配置类代码

这里先提及一个概念：`RedisConnectionFactory` 是 Spring Data Redis 框架中的一个**核心接口**

- 它的唯一职责就是：**创建和管理与 Redis 服务器之间的物理网络连接**

```java
//下面这两行代码完成了 RedisTemplate Bean 的创建和 连接工厂 的装配
//RedisTemplate必须持有 RedisConnectionFactory 才能与 Redis 服务器通信
RedisTemplate<String, Object> template = new RedisTemplate<>();
template.setConnectionFactory(factory);
```



##### 示例1 (手动配置版)

```java
package com.example.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.JsonTypeInfo;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.jsontype.impl.LaissezFaireSubTypeValidator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 * Redis 配置类
 */
@Configuration
public class RedisConfig {

    /**
     * 自定义 RedisTemplate<String, Object>
     *
     * @param connectionFactory Spring Boot 自动配置好的连接工厂
     * @return RedisTemplate
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        // --- 1. 创建序列化器 ---

        // Key 序列化器：String 序列化器
        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        
        // Value 序列化器：Jackson JSON 序列化器
        Jackson2JsonRedisSerializer<Object> jsonSerializer = 
            										new Jackson2JsonRedisSerializer<>(Object.class);
        
        // --- 1.1 配置 Jackson ObjectMapper (关键步骤) ---
        ObjectMapper om = new ObjectMapper();
        // 指定要序列化的字段、可见性等
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // 关键：指定序列化输入的类型，必须是 NON_FINAL
        // 这会在 JSON 中加入 "@class" 属性，例如: "@class": "com.example.User"
        // 使得反序列化时能准确地将 JSON 转回对应的 Java 对象
        om.activateDefaultTyping(
                LaissezFaireSubTypeValidator.instance,
                ObjectMapper.DefaultTyping.NON_FINAL,
                JsonTypeInfo.As.PROPERTY
        );
        // 注册 Java 8 时间模块 (必备)
        // 支持 LocalDate, LocalDateTime 等
        jsonSerializer.setObjectMapper(om);
        jsonSerializer.setObjectMapper(om);
        
        // --- 2. 设置序列化器到 Template ---

        // Key
        template.setKeySerializer(stringSerializer);
        // Hash Key
        template.setHashKeySerializer(stringSerializer);

        // Value
        template.setValueSerializer(jsonSerializer);
        // Hash Value
        template.setHashValueSerializer(jsonSerializer);

        // --- 3. 初始化 Template ---
        //用于在所有属性设置完成后进行初始化检查和默认值设置，手动创建Bean时,需要手动调用
        //Spring自动管理时，它由Spring自动调用
        template.afterPropertiesSet();
        
        return template;
    }
}
```



##### 示例2 (推荐版,自动版)

> 他俩的实现效果几乎相同，但是第一个 会序列化私有字段，因为
> `om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);`

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



#### 5.3  `activateDefaultTyping`

##### 为什么需要它？

- `Jackson2JsonRedisSerializer` 默认在序列化 POJO 时，**不会**存储对象的类型信息

  - **不配置的后果**：

    - 当你尝试反序列化时（`GET` 操作），Jackson 不知道这个 JSON 字符串应该转回 `User` 对象还是 `Product` 对象，

      它默认会将其转为一个 `LinkedHashMap`，导致你的程序出现 `ClassCastException`

  - **配置后的效果**：

    - `om.activateDefaultTyping(...)` 这行代码告诉 Jackson，

      在序列化时，**额外存入一个 `@class` 字段**来标记原始类型

      当反序列化时，Jackson 会读取这个 `@class` 字段，从而准确地将 JSON 转回原始的 Java 对象（如 `User` 对象）



##### 示例

```java
// 存入 User 对象
redisTemplate.opsForValue().set("user:1", new User("Alice", 25));

// 读取时变成 LinkedHashMap！
Object obj = redisTemplate.opsForValue().get("user:1");
User user = (User) obj;  // ❌ ClassCastException!
```

- **解决：** 配置后，JSON 中会包含 `@class` 字段：

  ```json
  {
    "@class": "com.example.model.User",
    "name": "Alice",
    "age": 25
  }
  ```

  - 反序列化时，Jackson 根据 `@class` 准确还原为 `User` 对象




#### 5.4 完整使用示例

```java
@Service
public class UserService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 存储对象（自动序列化）
    public void saveUser(User user) {
        redisTemplate.opsForValue().set("user:" + user.getId(), user, 1, TimeUnit.HOURS);
    }
    
    // 读取对象（自动反序列化）
    public User getUser(Long userId) {
        Object obj = redisTemplate.opsForValue().get("user:" + userId);
        return (User) obj;  // ✅ 类型安全
    }
    
    // Hash 操作
    public void saveUserProfile(Long userId, String field, Object value) {
        redisTemplate.opsForHash().put("user:profile:" + userId, field, value);
    }
    
    // List 操作
    public void addToRecentViews(Long userId, Long productId) {
        redisTemplate.opsForList().leftPush("user:recent:" + userId, productId);
        redisTemplate.opsForList().trim("user:recent:" + userId, 0, 9);  // 保留最近10条
    }
}
```

**redis-cli 中查看：**

```bash
> GET user:1
"{\"@class\":\"com.example.model.User\",\"id\":1,\"name\":\"Alice\",\"age\":25}"
```



#### 5.5 优缺点总结

| 优点                       | 缺点                    |
| -------------------------- | ----------------------- |
| ✅ 自动序列化对象，代码简洁 | ⚠️ 需要额外配置          |
| ✅ JSON格式，可读性好       | ⚠️ `@class` 字段略增体积 |
| ✅ 跨语言通用               |                         |
| ✅ 类型安全，无需手动转换   |                         |



#### 5.6 关于配置的其它问题

###### a1. 问题：LocalDateTime 序列化失败

在使用方案三（自定义 `RedisTemplate`）时，如果 POJO 对象中包含 `java.time.LocalDateTime`、`LocalDate` 或 `LocalTime` 等 Java 8 日期时间类型的字段，序列化时会抛出 `InvalidDefinitionException` 异常

**错误日志示例：**

```
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: 
Java 8 date/time type `java.time.LocalDateTime` not supported by default: 
add Module "com.fasterxml.jackson.datatype:jackson-datatype-jsr310" to enable handling
```



**问题根源：**

`Jackson2JsonRedisSerializer` 依赖 `ObjectMapper` 来完成 JSON 转换
默认情况下，`ObjectMapper` 并不知道如何将 `LocalDateTime` 这种 Java 8 特有的对象转换为 JSON 字符串（例如 "2025-11-16T14:30:00"）

在 `5.2 节` 的配置示例中，我们只配置了 `activateDefaultTyping` 来解决 `@class` 类型丢失的问题，但没有配置对 Java 8 时间类型的支持

```java
// 5.2 节中的配置（缺少时间模块）
ObjectMapper om = new ObjectMapper();
om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
om.activateDefaultTyping(
    LaissezFaireSubTypeValidator.instance,
    ObjectMapper.DefaultTyping.NON_FINAL,
    JsonTypeInfo.As.PROPERTY
);
jsonSerializer.setObjectMapper(om); // <-- 此时 om 还不支持 LocalDateTime
```



###### a2. 解决方案：注册 `JavaTimeModule`

Jackson 官方提供了 `jackson-datatype-jsr310` 模块来专门解决 Java 8 时间类型的序列化和反序列化问题

**1. 确保依赖存在**

如果你的项目（如 Spring Boot 项目）已经引入了 `spring-boot-starter-web`，那么这个依赖通常已经被间接引入。如果没有，请在 `pom.xml` 中手动添加：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
</dependency>
```



**2. 在 `RedisConfig` 中注册模块**

修改 `5.2 节` 中的 `RedisTemplate` 配置，在 `ObjectMapper` 上注册 `JavaTimeModule`

**修正后的 `RedisConfig` (基于 5.2 节示例1)：**

```java
package com.example.config;

// ... (其他 import) ...
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule; // 1. 导入 JavaTimeModule

@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer<Object> jsonSerializer = 
            new Jackson2JsonRedisSerializer<>(Object.class);
        
        // --- 配置 Jackson ObjectMapper ---
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        
        // 解决 @class 类型问题
        om.activateDefaultTyping(
            LaissezFaireSubTypeValidator.instance,
            ObjectMapper.DefaultTyping.NON_FINAL,
            JsonTypeInfo.As.PROPERTY
        );
        
        // 2. 注册 JavaTimeModule，解决 LocalDateTime 序列化问题
        om.registerModule(new JavaTimeModule()); 
        
        jsonSerializer.setObjectMapper(om);
        
        // --- 设置序列化器到 Template ---
        template.setKeySerializer(stringSerializer);
        template.setHashKeySerializer(stringSerializer);
        template.setValueSerializer(jsonSerializer);
        template.setHashValueSerializer(jsonSerializer);

        template.afterPropertiesSet();
        
        return template;
    }
}
```





## 常见问题 ❓

### Q1：redis-cli 中看到的 Key 是乱码怎么办？

**原因：** 使用了 JDK 序列化（默认 `RedisTemplate`）

**解决：**

1. 使用 `StringRedisTemplate`（Key 自动为字符串）
2. 或自定义 `RedisTemplate`，设置 Key 序列化器为 `StringRedisSerializer`

------

### Q2：反序列化报错 `Cannot deserialize...`

**原因：** 没有配置 `activateDefaultTyping`，Jackson 不知道目标类型

**解决：** 在 `RedisConfig` 中添加：

```java
om.activateDefaultTyping(
    LaissezFaireSubTypeValidator.instance,
    ObjectMapper.DefaultTyping.NON_FINAL,
    JsonTypeInfo.As.PROPERTY
);
```

------

### Q3：`StringRedisTemplate` 和自定义 `RedisTemplate` 能同时用吗？

**答：** 可以！根据场景选择：

- 简单字符串 → `StringRedisTemplate`
- 复杂对象 → `RedisTemplate<String, Object>`

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;  // 存Token

@Autowired
private RedisTemplate<String, Object> redisTemplate;  // 存User对象
```

------

### Q4：如何解决 Redis 连接池耗尽？

**排查步骤：**

1. 检查是否有连接泄漏（未释放）
2. 查看 `max-active` 是否过小
3. 监控实际并发量，适当调大连接池

**临时方案：**

```yaml
lettuce:
  pool:
    max-active: 50  # 增大最大连接数
    max-wait: 5s    # 增加等待时间
```

------

### Q5：如何处理大Key问题？

**识别：**

```bash
redis-cli --bigkeys  # 扫描大Key
```

**优化：**

1. 拆分为多个小Key
2. 使用 Hash 结构代替大 String
3. 设置合理的过期时间
4. 考虑使用压缩（如 Gzip）



# SDR 基础数据结构操作

## 1. 操作 String

- String 是 Redis 中最基本的数据类型

  - 在 SDR 中，所有对 String 类型的操作都是通过 `redisTemplate.opsForValue()` 来获取操作器完成的

  - 假设我们已经在 Spring 容器中注入了配置好的 `RedisTemplate<String, Object>`：

    ```java
    @Autowiredprivate RedisTemplate<String, Object> redisTemplate;
    ```



### 1.1 常用方法

- 以下是 `ValueOperations` 提供的主要方法及其对应的 Redis 命令

#### 1. 存入 (SET / SETEX / SETNX)

- **`void set(K key, V value)`**

  - **对应命令**：`SET key value`

  - **说明**：设置指定 key 的值。如果 key 已经存在，则覆盖

  - **示例**：

    ```java
    // V value 会被配置的序列化器（如 Jackson）序列化后存储
    User user = new User(1L, "Alice");
    redisTemplate.opsForValue().set("user:1", user); 
    ```

  

- **`void set(K key, V value, Duration timeout)`**

  - **对应命令**：`SETEX key seconds value` (SET with EXpire)

  - **说明**：设置 key 的值，并指定过期时间

  - **示例**：

    ```java
    // 存入验证码，设置10分钟过期
    redisTemplate.opsForValue().set("sms:13800001111", "123456", Duration.ofMinutes(10));
    ```

  

- **`Boolean setIfAbsent(K key, V value)`**

  - **对应命令**：`SETNX key value` (SET if Not eXists)

  - **说明**：当 key 不存在时才设置。常用于实现分布式锁

  - **返回值**：`true` 表示设置成功，`false` 表示 key 已存在

  - **示例**：

    ```java
    // 尝试获取锁
    Boolean lockAcquired = redisTemplate.opsForValue().setIfAbsent("lock:myTask", "processing");
    if (lockAcquired) {    
        // ... 执行业务 ...
    }
    ```

  

- **`Boolean setIfAbsent(K key, V value, Duration timeout)`**

  - **对应命令**：`SET key value NX EX seconds`

  - **说明**：当 key 不存在时才设置，并指定过期时间。推荐用于分布式锁，防止死锁

  - **示例**：

    ```java
    // 获取锁，30秒自动释放
    Boolean lockAcquired = redisTemplate.opsForValue()    
        .setIfAbsent("lock:myTask", "processing", Duration.ofSeconds(30));
    ```

  

- **`Boolean setIfPresent(K key, V value)`**

  - **对应命令**：`SET key value XX`

  - **说明**：当 key 存在时才更新值

  - **示例**：

    ```java
    // 只有用户存在时才更新
    Boolean updated = redisTemplate.opsForValue().setIfPresent("user:1", newUser);
    ```



#### 2. 读取 (GET / GETSET)

- **`V get(Object key)`**

  - **对应命令**：`GET key`

  - **说明**：获取指定 key 的值。如果 key 不存在，返回 `null`

  - **示例**：

    ```java
    // SDR 会自动将存储的数据反序列化为对象
    User user = (User) redisTemplate.opsForValue().get("user:1");
    ```

  

- **`V getAndSet(K key, V value)`**

  - **对应命令**：`GETSET key value`

  - **说明**：设置新值并返回旧值。常用于计数器重置等场景

  - **示例**：

    ```java
    // 获取旧的访问计数并重置为0
    Long oldCount = (Long) redisTemplate.opsForValue().getAndSet("page:view:1001", 0);
    ```



#### 3. 增减 (INCR / DECR / INCRBY / DECRBY)

- **`Long increment(K key)`**

  - **对应命令**：`INCR key`

  - **说明**：将 key 中存储的数字值增加 1

  - **示例**：

    ```java
    // 访问量 +1
    redisTemplate.opsForValue().increment("page:view:1001");
    ```

  

- **`Long increment(K key, long delta)`**

  - **对应命令**：`INCRBY key delta`

  - **说明**：将 key 中存储的数字值增加指定的增量 `delta`。如果 key 不存在，会先初始化为 0

  - **注意**：要求 value 必须是数字或数字字符串，推荐使用 `StringRedisTemplate`

  - **示例**：

    ```java
    // 积分增加 10
    redisTemplate.opsForValue().increment("user:1:points", 10);
    ```

  

- **`Double increment(K key, double delta)`**

  - **对应命令**：`INCRBYFLOAT key delta`

  - **说明**：将 key 中的浮点数值增加指定的增量

  - **示例**：

    ```java
    // 账户余额增加 9.99
    redisTemplate.opsForValue().increment("user:1:balance", 9.99);
    ```

  

- **`Long decrement(K key)`**

  - **对应命令**：`DECR key`
  - **说明**：将 key 中存储的数字值减少 1

  

- **`Long decrement(K key, long delta)`**

  - **对应命令**：`DECRBY key delta`

  - **说明**：将 key 中存储的数字值减少指定的减量

  - **示例**：

    ```java
    // 库存 -1
    redisTemplate.opsForValue().decrement("stock:product:1001", 1);
    ```



#### 4. 批量操作 (MSET / MGET)

- **`void multiSet(Map<? extends K, ? extends V> map)`**

  - **对应命令**：`MSET key1 value1 key2 value2 ...`

  - **说明**：同时设置一个或多个 key-value 对，原子操作

  - **示例**：

    ```java
    Map<String, Object> data = new HashMap<>();
    data.put("user:1", new User(1L, "Alice"));
    data.put("user:2", new User(2L, "Bob"));
    redisTemplate.opsForValue().multiSet(data);
    ```

  

- **`Boolean multiSetIfAbsent(Map<? extends K, ? extends V> map)`**

  - **对应命令**：`MSETNX key1 value1 key2 value2 ...`
  - **说明**：当且仅当所有给定 key 都不存在时，才设置所有 key-value 对
  - **返回值**：`true` 表示全部设置成功，`false` 表示至少有一个 key 已存在（此时不会设置任何值）

  

- **`List<V> multiGet(Collection<K> keys)`**

  - **对应命令**：`MGET key1 key2 ...`

  - **说明**：获取所有给定 key 的值。不存在的 key 对应位置返回 `null`

  - **示例**：

    ```java
    List<String> keys = Arrays.asList("user:1", "user:99");
    List<Object> results = redisTemplate.opsForValue().multiGet(keys);// results 将会是 [User(1L, "Alice"),null]
    ```



#### 5. 其他实用操作

- **`Long size(K key)`**
  - **对应命令**：`STRLEN key`
  - **说明**：返回 key 所存储的字符串值的长度



- **`Integer append(K key, String value)`**

  - **对应命令**：`APPEND key value`

  - **说明**：将 value 追加到原值的末尾，返回追加后的字符串长度

  - **示例**：

    ```java
    redisTemplate.opsForValue().set("msg", "Hello");
    redisTemplate.opsForValue().append("msg", " World"); // 存储变为 "Hello World"
    ```



## 2. 操作 Hash

**Hash** 是一种在 Redis 中**存储对象的理想结构**

- 它允许你将一个"大 Key"（例如 `user:1`）拆分为多个 field-value 对（例如 `name`, `age` 等）
- 在 SDR 中，所有对 Hash 类型的操作都是通过 `redisTemplate.opsForHash()` 来获取操作器 (HashOperations) 完成的



**对比 String 存 JSON**：

- **String 存 JSON**：`SET "user:1" "{\"name\":\"Alice\",\"age\":30}"`
  - **优点**：一次性获取所有数据，结构简单
  - **缺点**：修改单个字段需要先 GET 整个 JSON，反序列化、修改后再序列化、SET 回去，操作成本高
- **Hash 存储**：`HSET "user:1" "name" "Alice"`，`HSET "user:1" "age" 30`
  - **优点**：可以对单个字段进行独立读写（`HGET`, `HSET`），灵活高效，节省网络传输
  - **缺点**：获取整个对象需要 `HGETALL`，在字段数量少时可能比 `GET` 稍慢



**序列化说明**：

- `opsForHash()` 使用三种序列化器：
  - `KeySerializer (StringRedisSerializer)`：处理 Redis 主 Key（例如 `user:1`）
  - `HashKeySerializer (StringRedisSerializer)`：处理 Hash 的 Field（例如 `name`, `age`）
  - `HashValueSerializer (Jackson2JsonRedisSerializer)`：处理 Hash 的 Value（例如 `"Alice"`, `30`, 或复杂对象）



### 2.1 常用方法

- 假设已注入 `RedisTemplate<String, Object> redisTemplate`



#### 1. 存入单个字段 (HSET)

- **`void put(H key, HK hashKey, HV value)`**

  - **对应命令**：`HSET key field value`

  - **说明**：设置 Hash 表中指定字段的值。如果字段已存在则覆盖

  - **示例**：

    ```java
    // HSET user:1 name "Alice"
    redisTemplate.opsForHash().put("user:1", "name", "Alice"); // HSET user:1 age 30
    redisTemplate.opsForHash().put("user:1", "age", 30);
    							// 存储复杂对象（会被序列化为 JSON）Address address = new Address("Main St", "123");
    redisTemplate.opsForHash().put("user:1", "address", address);
    ```

​	

- **`Boolean putIfAbsent(H key, HK hashKey, HV value)`**

  - **对应命令**：`HSETNX key field value`

  - **说明**：当字段不存在时才设置值

  - **返回值**：`true` 表示设置成功，`false` 表示字段已存在

  - **示例**：

    ```java
    // 只有在字段不存在时才设置默认值
    Boolean isSet = redisTemplate.opsForHash().putIfAbsent("user:1", "status", "active");
    ```



#### 2. 存入多个字段 (HSET)

- **`void putAll(H key, Map<? extends HK, ? extends HV> map)`**

  - **对应命令**：`HSET key field1 value1 field2 value2 ...`（Redis 4.0+ 支持多字段）

  - **说明**：同时将多个 field-value 对设置到 Hash 表中

  - **示例**：

    ```java
    Map<String, Object> fields = new HashMap<>();
    fields.put("name", "Bob");
    fields.put("age", 42);
    fields.put("email", "bob@example.com");
    redisTemplate.opsForHash().putAll("user:2", fields);
    ```



#### 3. 读取单个字段 (HGET)

- **`HV get(H key, Object hashKey)`**

  - **对应命令**：`HGET key field`

  - **说明**：获取 Hash 表中指定字段的值。如果字段不存在，返回 `null`

  - **示例**：

    ```java
    String name = (String) redisTemplate.opsForHash().get("user:1", "name");
    Integer age = (Integer) redisTemplate.opsForHash().get("user:1", "age");
    Address address = (Address) redisTemplate.opsForHash().get("user:1", "address");
    ```



#### 4. 批量读取字段 (HMGET)

- **`List<HV> multiGet(H key, Collection<HK> hashKeys)`**

  - **对应命令**：`HMGET key field1 field2 ...`

  - **说明**：批量获取多个字段的值。不存在的字段对应位置返回 `null`

  - **示例**：

    ```java
    List<Object> fields = Arrays.asList("name", "age", "nonexistent");
    List<Object> values = redisTemplate.opsForHash().multiGet("user:1", fields);
    															// values 可能是 ["Alice", 30, null]
    ```



#### 5. 读取所有字段 (HGETALL)

- **`Map<HK, HV> entries(H key)`**

  - **对应命令**：`HGETALL key`

  - **说明**：获取 Hash 表中所有的 field-value 对

  - **注意**：返回的是 `Map<Object, Object>`，需要手动类型转换

  - **示例**：

    ```java
    Map<Object, Object> entries = redisTemplate.opsForHash().entries("user:2");
    												// entries.get("name") -> "Bob"// entries.get("age") -> 42
    ```



#### 6. 删除字段 (HDEL)

- **`Long delete(H key, Object... hashKeys)`**

  - **对应命令**：`HDEL key field1 [field2 ...]`

  - **说明**：删除一个或多个 Hash 字段

  - **返回值**：成功删除的字段数量

  - **示例**：

    ```java
    Long deletedCount = redisTemplate.opsForHash().delete("user:1", "age", "address");
    ```



#### 7. 判断字段存在 (HEXISTS)

- **`Boolean hasKey(H key, Object hashKey)`**

  - **对应命令**：`HEXISTS key field`

  - **说明**：检查 Hash 表中是否存在指定字段

  - **示例**：

    ```java
    Boolean exists = redisTemplate.opsForHash().hasKey("user:1", "email");
    if (exists) {    // 字段存在
    }
    ```



#### 8. 获取字段和值列表

- **`Set<HK> keys(H key)`**

  - **对应命令**：`HKEYS key`

  - **说明**：获取 Hash 表中所有的字段名

  - **示例**：

    ```java
    Set<Object> fields = redisTemplate.opsForHash().keys("user:1");	// 例如：["name", "age", "address"]
    ```

  

- **`List<HV> values(H key)`**

  - **对应命令**：`HVALS key`

  - **说明**：获取 Hash 表中所有的值

  - **示例**：

    ```java
    List<Object> values = redisTemplate.opsForHash().values("user:1");	// 例如：["Alice", 30, Address(...)]
    ```



#### 9. 获取字段数量 (HLEN)

- **`Long size(H key)`**

  - **对应命令**：`HLEN key`

  - **说明**：获取 Hash 表中字段的数量

  - **示例**：

    ```java
    Long fieldCount = redisTemplate.opsForHash().size("user:1");
    ```



#### 10. 增减操作 (HINCRBY / HINCRBYFLOAT)

- **`Long increment(H key, HK hashKey, long delta)`**

  - **对应命令**：`HINCRBY key field delta`

  - **说明**：为 Hash 表中指定字段的整数值增加 `delta`。如果字段不存在，初始化为 0

  - **注意**：字段值必须是数字或数字字符串

  - **示例**：

    ```java
    // 用户年龄 +1
    redisTemplate.opsForHash().increment("user:1", "age", 1);				// 商品库存 -5
    redisTemplate.opsForHash().increment("product:1001", "stock", -5);
    ```

  

- **`Double increment(H key, HK hashKey, double delta)`**

  - **对应命令**：`HINCRBYFLOAT key field delta`

  - **说明**：为 Hash 表中指定字段的浮点数值增加 `delta`

  - **示例**：

    ```java
    // 用户余额增加 9.99
    redisTemplate.opsForHash().increment("user:1", "balance", 9.99);
    ```



## 3. 操作 List

**List** 是一个**有序的字符串元素集合**

- 允许元素重复，严格按照插入顺序排序
- 底层实现是 quicklist（快速列表，Redis 3.2+），结合了 ziplist 和 linkedlist 的优点
- 在头部和尾部进行 `PUSH`/`POP` 操作的时间复杂度为 O(1)
- 适合用作**消息队列**（FIFO）或**任务栈**（LIFO）

- 在 SDR 中，所有对 List 类型的操作都是通过 `redisTemplate.opsForList()` 来获取操作器 (ListOperations) 完成的
- **序列化说明**：存入对象时会使用配置的序列化器（如 Jackson）将对象序列化为 JSON 字符串



### 3.1 常用方法 (SDR vs Redis 命令)

- 假设已注入 `RedisTemplate<String, Object> redisTemplate`



#### 1. 推入元素 (LPUSH / RPUSH)

- **`Long leftPush(K key, V value)`**

  - **对应命令**：`LPUSH key value`

  - **说明**：将值推入到列表**头部**（左侧）

  - **返回值**：推入后列表的长度

  - **示例**：（实现栈 LIFO）

    ```java
    redisTemplate.opsForList().leftPush("tasks:stack", "Task:3");
    redisTemplate.opsForList().leftPush("tasks:stack", "Task:2");// 此时列表为 ["Task:2", "Task:3"]，后进先出
    ```



- **`Long rightPush(K key, V value)`**

  - **对应命令**：`RPUSH key value`

  - **说明**：将值推入到列表**尾部**（右侧）

  - **返回值**：推入后列表的长度

  - **示例**：（实现队列 FIFO）

    ```java
    redisTemplate.opsForList().rightPush("tasks:queue", "Task:1");
    redisTemplate.opsForList().rightPush("tasks:queue", "Task:2");// 此时列表为 ["Task:1", "Task:2"]，先进先出
    ```



- **`Long leftPushAll(K key, V... values)`**

  - **对应命令**：`LPUSH key v1 v2 v3`

  - **说明**：一次性从头部推入多个值

  - **注意**：多个值是**逐个**从左侧推入，所以顺序会反转

  - **示例**：

    ```java
    redisTemplate.opsForList().leftPushAll("tasks", "A", "B", "C");// 列表为 ["C", "B", "A"]
    ```



- **`Long rightPushAll(K key, V... values)`**

  - **对应命令**：`RPUSH key v1 v2 v3`

  - **说明**：一次性从尾部推入多个值，顺序保持

  - **示例**：

    ```java
    redisTemplate.opsForList().rightPushAll("tasks", "A", "B", "C");// 列表为 ["A", "B", "C"]
    ```



- **`Long leftPushIfPresent(K key, V value)`**
  - **对应命令**：`LPUSHX key value`
  - **说明**：只有当列表存在时，才从头部推入值
  - **返回值**：推入后列表的长度，如果列表不存在返回 0



- **`Long rightPushIfPresent(K key, V value)`**
  - **对应命令**：`RPUSHX key value`
  - **说明**：只有当列表存在时，才从尾部推入值



#### 2. 弹出元素 (LPOP / RPOP)

- **`V leftPop(K key)`**

  - **对应命令**：`LPOP key`

  - **说明**：从列表**头部**弹出一个元素并返回。如果列表为空，返回 `null`

  - **示例**：（配合 `leftPush` 实现栈 LIFO）

    ```java
    Object task = redisTemplate.opsForList().leftPop("tasks:stack"); // 返回 "Task:2"
    ```



- **`V rightPop(K key)`**

  - **对应命令**：`RPOP key`

  - **说明**：从列表**尾部**弹出一个元素并返回。如果列表为空，返回 `null`

  - **示例**：（配合 `leftPush` 实现队列 FIFO）

    ```java
    Object task = redisTemplate.opsForList().rightPop("tasks:queue"); // 返回 "Task:2"
    ```



- **`List<V> leftPop(K key, long count)`**

  - **对应命令**：`LPOP key count`（Redis 6.2+）

  - **说明**：从头部一次性弹出多个元素

  - **示例**：

    ```java
    List<Object> tasks = redisTemplate.opsForList().leftPop("tasks", 3);
    ```



- **`List<V> rightPop(K key, long count)`**
  - **对应命令**：`RPOP key count`（Redis 6.2+）
  - **说明**：从尾部一次性弹出多个元素



#### 3. 阻塞式弹出 (BLPOP / BRPOP)

- **`V leftPop(K key, long timeout, TimeUnit unit)`**

  - **对应命令**：`BLPOP key timeout_in_seconds`

  - **说明**：

    - 阻塞式地从列表头部弹出元素
    - 如果列表有元素，立即返回
    - 如果列表为空，阻塞等待直到超时或有新元素被推入
    - `timeout` 设为 0 表示永久阻塞

  - **示例**：（消息队列消费者）

    ```java
    // 阻塞等待最多 10 秒
    Object task = redisTemplate.opsForList().leftPop("tasks:queue", 10, TimeUnit.SECONDS);
    if (task != null) {    
        // 处理任务
    } else {
        // 超时，未等到任务
    }
    ```



- **`V rightPop(K key, long timeout, TimeUnit unit)`**
  - **对应命令**：`BRPOP key timeout_in_seconds`
  - **说明**：阻塞式地从列表尾部弹出元素



#### 4. 列表间移动 (RPOPLPUSH / BRPOPLPUSH)

- **`V rightPopAndLeftPush(K sourceKey, K destinationKey)`**

  - **对应命令**：`RPOPLPUSH source destination`

  - **说明**：从源列表尾部弹出元素，推入目标列表头部。原子操作，常用于可靠队列

  - **示例**：

    ```java
    // 从待处理队列移到处理中队列Object task = redisTemplate.opsForList()    .rightPopAndLeftPush("queue:pending", "queue:processing");
    ```



- **`V rightPopAndLeftPush(K sourceKey, K destinationKey, long timeout, TimeUnit unit)`**
  - **对应命令**：`BRPOPLPUSH source destination timeout`
  - **说明**：阻塞版本的列表间移动操作



#### 5. 查看与遍历 (LRANGE / LINDEX / LLEN)

- **`List<V> range(K key, long start, long end)`**

  - **对应命令**：`LRANGE key start end`

  - **说明**：获取列表指定范围内的所有元素

  - **注意**：

    - 索引从 0 开始，负数索引表示从尾部计数（-1 表示最后一个元素）
    - `start=0, end=-1` 表示获取所有元素
    - 对于大列表（百万级），全量获取可能阻塞 Redis

  - **示例**：

    ```java
    // 获取所有元素
    List<Object> allTasks = redisTemplate.opsForList().range("tasks:queue", 0, -1);	// 获取前 10 个元素
    List<Object> top10 = redisTemplate.opsForList().range("tasks:queue", 0, 9);
    ```



- **`V index(K key, long index)`**

  - **对应命令**：`LINDEX key index`

  - **说明**：获取列表中指定索引位置的元素

  - **示例**：

    ```java
    // 获取第一个元素（不弹出）
    Object first = redisTemplate.opsForList().index("tasks", 0);// 获取最后一个元素
    Object last = redisTemplate.opsForList().index("tasks", -1);
    ```



- **`Long size(K key)`**

  - **对应命令**：`LLEN key`

  - **说明**：获取列表的长度（元素个数）

  - **示例**：

    ```java
    Long count = redisTemplate.opsForList().size("tasks:queue");
    ```



#### 6. 修改元素 (LSET / LREM / LTRIM)

- **`void set(K key, long index, V value)`**

  - **对应命令**：`LSET key index value`

  - **说明**：设置列表中指定索引位置的元素值

  - **注意**：索引必须在有效范围内，否则报错

  - **示例**：

    ```java
    // 修改第一个任务
    redisTemplate.opsForList().set("tasks", 0, "Updated Task");
    ```



- **`Long remove(K key, long count, Object value)`**

  - **对应命令**：`LREM key count value`

  - **说明**：从列表中移除指定数量的匹配元素

  - **参数说明**：

    - `count > 0`：从头部开始，移除 count 个等于 value 的元素
    - `count < 0`：从尾部开始，移除 |count| 个等于 value 的元素
    - `count = 0`：移除所有等于 value 的元素

  - **返回值**：实际移除的元素数量

  - **示例**：

    ```java
    // 移除所有值为 "Task:1" 的元素
    Long removed = redisTemplate.opsForList().remove("tasks", 0, "Task:1");
    ```



- **`void trim(K key, long start, long end)`**

  - **对应命令**：`LTRIM key start end`

  - **说明**：修剪列表，只保留指定范围内的元素，其余删除

  - **示例**：

    ```java
    // 只保留最新的 100 条消息
    redisTemplate.opsForList().trim("messages", 0, 99);	// 删除前 5 个元素（保留第 6 个及之后的）
    redisTemplate.opsForList().trim("tasks", 5, -1);
    ```



## 4. 操作 Set

Set (集合) 是一个**无序**的字符串元素集合，**不允许元素重复**

- 适合用于**去重**和**关系计算**（交集、并集、差集）的场景，例如：
  - 存储帖子的点赞用户 ID（天然去重）
  - 存储用户的标签
  - 计算共同好友（交集）
  - 推荐系统中的标签匹配



- 在 SDR 中，所有对 Set 类型的操作都是通过 `redisTemplate.opsForSet()` 来获取操作器 (SetOperations) 完成的

- **序列化说明**：Set 通常用于存储 ID 或简单字符串，而非复杂对象



### 4.1 常用方法 (SDR vs Redis 命令)

- 假设已注入 `RedisTemplate<String, Object> redisTemplate`



#### 1. 添加元素 (SADD)

- **`Long add(K key, V... values)`**

  - **对应命令**：`SADD key member [member ...]`

  - **说明**：向集合中添加一个或多个元素。已存在的元素会被忽略

  - **返回值**：成功添加的新元素数量

  - **示例**：

    ```java
    // 添加点赞用户
    Long count = redisTemplate.opsForSet().add("likes:post:1001", "user:1", "user:2", "user:3");// count = 3
    // 尝试再次添加已存在的元素
    count = redisTemplate.opsForSet().add("likes:post:1001", "user:2");	// count = 0（user:2 已存在）
    ```



#### 2. 移除元素 (SREM / SPOP)

- **`Long remove(K key, Object... values)`**

  - **对应命令**：`SREM key member [member ...]`

  - **说明**：从集合中移除一个或多个指定元素

  - **返回值**：成功移除的元素数量

  - **示例**：

    ```java
    // 用户取消点赞
    Long removed = redisTemplate.opsForSet().remove("likes:post:1001", "user:2");
    ```



- **`V pop(K key)`**

  - **对应命令**：`SPOP key`

  - **说明**：随机从集合中移除并返回一个元素

  - **示例**：

    ```java
    // 抽奖：随机抽取一个用户
    Object winner = redisTemplate.opsForSet().pop("lottery:participants");
    ```



- **`List<V> pop(K key, long count)`**

  - **对应命令**：`SPOP key count`

  - **说明**：随机从集合中移除并返回多个元素

  - **示例**：

    ```java
    // 抽取 3 个中奖用户
    List<Object> winners = redisTemplate.opsForSet().pop("lottery:participants", 3);
    ```



#### 3. 查看元素 (SMEMBERS / SRANDMEMBER)

- **`Set<V> members(K key)`**

  - **对应命令**：`SMEMBERS key`

  - **说明**：返回集合中的所有元素

  - **警告**：这是一个 O(N) 重操作。如果集合非常大（百万级），会阻塞 Redis。生产环境慎用

  - **示例**：

    ```java
    Set<Object> userIds = redisTemplate.opsForSet().members("likes:post:1001");
    														// 例如：["user:1", "user:2", "user:3"]（顺序不固定）
    ```



- **`V randomMember(K key)`**

  - **对应命令**：`SRANDMEMBER key`

  - **说明**：随机返回集合中的一个元素（不删除）

  - **示例**：

    ```java
    // 随机推荐一个用户
    Object randomUser = redisTemplate.opsForSet().randomMember("users:active");
    ```



- **`List<V> randomMembers(K key, long count)`**

  - **对应命令**：`SRANDMEMBER key count`

  - **说明**：随机返回多个元素（可能重复）

  - **注意**：

    - `count > 0`：返回不重复的元素，最多返回集合大小个
    - `count < 0`：返回可能重复的元素，数量为 |count|

  - **示例**：

    ```java
    // 随机推荐 5 个商品（不重复）
    List<Object> products = redisTemplate.opsForSet().randomMembers("products:hot", 5);
    ```



- **`Set<V> distinctRandomMembers(K key, long count)`**

  - **对应命令**：`SRANDMEMBER key count`（count > 0）

  - **说明**：随机返回多个不重复的元素

  - **示例**：

    ```java
    // 确保返回的是不重复的元素
    Set<Object> uniqueProducts = redisTemplate.opsForSet().distinctRandomMembers("products:hot", 5);
    ```



#### 4. 判断与统计 (SISMEMBER / SCARD)

- **`Boolean isMember(K key, Object o)`**

  - **对应命令**：`SISMEMBER key member`

  - **说明**：判断指定元素是否存在于集合中。O(1) 时间复杂度

  - **示例**：

    ```java
    // 检查用户是否点赞了帖子
    Boolean hasLiked = redisTemplate.opsForSet().isMember("likes:post:1001", "user:2");
    ```



- **`List<Boolean> isMember(K key, Object... objects)`**

  - **对应命令**：`SMISMEMBER key member [member ...]`（Redis 6.2+）

  - **说明**：批量判断多个元素是否存在

  - **示例**：

    ```java
    List<Boolean> results = redisTemplate.opsForSet()
        								.isMember("likes:post:1001", "user:1", "user:2", "user:999");
    // 例如：[true, true, false]
    ```



- **`Long size(K key)`**

  - **对应命令**：`SCARD key`

  - **说明**：获取集合的元素数量

  - **示例**：

    ```java
    // 获取点赞数
    Long likeCount = redisTemplate.opsForSet().size("likes:post:1001");
    ```



#### 5. 集合运算 - 计算 (SDIFF / SINTER / SUNION)

- **`Set<V> difference(K key, K otherKey)`**

  - **对应命令**：`SDIFF key otherKey`

  - **说明**：返回差集，即存在于第一个集合但不存在于其他集合中的元素

  - **示例**：

    ```java
    // A 关注了但 B 没关注的人
    Set<Object> diff = redisTemplate.opsForSet().difference("user:alice:following", "user:bob:following");
    							// 如果 alice 关注 [1,2,3]，bob 关注 [2,3,4]，结果为 [1]
    ```



- **`Set<V> difference(K key, Collection<K> otherKeys)`**

  - **对应命令**：`SDIFF key otherKey1 otherKey2 ...`

  - **说明**：计算多个集合的差集

  - **示例**：

    ```java
    Set<Object> diff = redisTemplate.opsForSet()    
        							.difference("setA", Arrays.asList("setB", "setC"));
    ```



- **`Set<V> intersect(K key, K otherKey)`**

  - **对应命令**：`SINTER key otherKey`

  - **说明**：返回交集，即同时存在于所有集合中的元素

  - **示例**：

    ```java
    // A 和 B 的共同好友
    Set<Object> common = redisTemplate.opsForSet()    
        								.intersect("user:alice:following", "user:bob:following");
    						// 如果 alice 关注 [1,2,3]，bob 关注 [2,3,4]，结果为 [2,3]
    ```



- **`Set<V> intersect(K key, Collection<K> otherKeys)`**

  - **对应命令**：`SINTER key otherKey1 otherKey2 ...`

  - **说明**：计算多个集合的交集

  - **示例**：

    ```java
    // 三个用户的共同好友
    Set<Object> common = redisTemplate.opsForSet()    
                        .intersect("user:alice:following",
                                   Arrays.asList("user:bob:following", "user:charlie:following")
                                  );
    ```



- **`Set<V> union(K key, K otherKey)`**

  - **对应命令**：`SUNION key otherKey`

  - **说明**：返回并集，即所有集合的元素合并后去重

  - **示例**：

    ```java
    // A 和 B 关注的所有人
    Set<Object> all = redisTemplate.opsForSet()    
        							.union("user:alice:following", "user:bob:following");
    						// 如果 alice 关注 [1,2,3]，bob 关注 [2,3,4]，结果为 [1,2,3,4]
    ```



- **`Set<V> union(K key, Collection<K> otherKeys)`**
  - **对应命令**：`SUNION key otherKey1 otherKey2 ...`
  - **说明**：计算多个集合的并集



#### 6. 集合运算 - 存储 (SDIFFSTORE / SINTERSTORE / SUNIONSTORE)

- **`Long differenceAndStore(K key, K otherKey, K destKey)`**

  - **对应命令**：`SDIFFSTORE destination key otherKey`

  - **说明**：计算差集并将结果存储到新集合中

  - **返回值**：结果集合的元素数量

  - **示例**：

    ```java
    // 计算差集并存储
    Long count = redisTemplate.opsForSet()    
        					.differenceAndStore("setA", "setB", "result:diff");
    ```



- **`Long differenceAndStore(K key, Collection<K> otherKeys, K destKey)`**
  - **对应命令**：`SDIFFSTORE destination key otherKey1 otherKey2 ...`
  - **说明**：计算多个集合的差集并存储



- **`Long intersectAndStore(K key, K otherKey, K destKey)`**

  - **对应命令**：`SINTERSTORE destination key otherKey`

  - **说明**：计算交集并将结果存储到新集合中

  - **示例**：

    ```java
    // 将共同好友存储到新集合
    redisTemplate.opsForSet()    
        .intersectAndStore("user:alice:following", "user:bob:following","common:alice:bob");
    ```



- **`Long intersectAndStore(K key, Collection<K> otherKeys, K destKey)`**
  - **对应命令**：`SINTERSTORE destination key otherKey1 otherKey2 ...`
  - **说明**：计算多个集合的交集并存储



- **`Long unionAndStore(K key, K otherKey, K destKey)`**
  - **对应命令**：`SUNIONSTORE destination key otherKey`
  - **说明**：计算并集并将结果存储到新集合中



- **`Long unionAndStore(K key, Collection<K> otherKeys, K destKey)`**
  - **对应命令**：`SUNIONSTORE destination key otherKey1 otherKey2 ...`
  - **说明**：计算多个集合的并集并存储



#### 7. 移动元素 (SMOVE)

- **`Boolean move(K key, V value, K destKey)`**

  - **对应命令**：`SMOVE source destination member`

  - **说明**：将元素从源集合移动到目标集合（原子操作）

  - **返回值**：`true` 表示移动成功，`false` 表示源集合中不存在该元素

  - **示例**：

    ```java
    // 将用户从待审核集合移到已通过集合
    Boolean moved = redisTemplate.opsForSet().move("users:pending", "user:123", "users:approved");
    ```



#### 8. 迭代扫描 (SSCAN)

- **`Cursor<V> scan(K key, ScanOptions options)`**

  - **对应命令**：`SSCAN key cursor [MATCH pattern] [COUNT count]`

  - **说明**：迭代扫描集合，适用于大集合，避免阻塞

  - **示例**：

    ```java
    // 使用游标扫描大集合
    ScanOptions options = ScanOptions.scanOptions().match("user:*").count(100).build();
    Cursor<Object> cursor = redisTemplate.opsForSet().scan("large:set", options);
    while (cursor.hasNext()) {    
        Object member = cursor.next();    // 处理元素
    }
    cursor.close();
    ```



## 5. 操作 ZSet (Sorted Set)

ZSet (有序集合，也叫 Sorted Set) 是 Set 的升级版

- 同样**不允许元素重复**
- 每个元素都关联一个 `double` 类型的 **分数 (Score)**
- Redis 通过分数自动对元素进行排序

ZSet 非常适合用于实现**排行榜**相关的业务，例如：

- 游戏积分排行榜（Score: 积分，Value: 玩家ID）
- 文章热度排行榜（Score: 点击数，Value: 文章ID）
- 微博热搜（Score: 搜索热度，Value: 关键词）
- 延时任务队列（Score: 执行时间戳，Value: 任务ID）

在 SDR 中，所有对 ZSet 类型的操作都是通过 `redisTemplate.opsForZSet()` 来获取操作器 (ZSetOperations) 完成的

- **序列化说明**：ZSet 的 Value 通常是 ID 或字符串，Score 是 `double` 类型的数值



### 5.1 常用方法 (SDR vs Redis 命令)

- 假设已注入 `RedisTemplate<String, Object> redisTemplate`

#### 1. 添加/更新元素 (ZADD)

- **`Boolean add(K key, V value, double score)`**

  - **对应命令**：`ZADD key score member`

  - **说明**：向 ZSet 中添加元素并指定分数

  - **返回值**：`true` 表示添加成功，`false` 表示元素已存在且更新了分数

  - **示例**：

    ```java
    // 添加玩家及其分数
    redisTemplate.opsForZSet().add("game:ranking", "player:1", 100);
    redisTemplate.opsForZSet().add("game:ranking", "player:2", 150);	// 更新 player:1 的分数为 110
    Boolean isNew = redisTemplate.opsForZSet().add("game:ranking", "player:1", 110);
    																	// isNew = false（元素已存在）
    ```



- **`Long add(K key, Set<TypedTuple<V>> tuples)`**

  - **对应命令**：`ZADD key score1 member1 score2 member2 ...`

  - **说明**：批量添加多个元素及其分数

  - **返回值**：成功添加的新元素数量

  - **示例**：

    ```java
    Set<ZSetOperations.TypedTuple<Object>> tuples = new HashSet<>();
    tuples.add(new DefaultTypedTuple<>("player:1", 100.0));
    tuples.add(new DefaultTypedTuple<>("player:2", 150.0));
    tuples.add(new DefaultTypedTuple<>("player:3", 120.0));
    Long count = redisTemplate.opsForZSet().add("game:ranking", tuples);
    ```



- **`Boolean addIfAbsent(K key, V value, double score)`**
  - **对应命令**：`ZADD key NX score member`（Redis 6.2+）
  - **说明**：只有当元素不存在时才添加
  - **返回值**：`true` 表示添加成功，`false` 表示元素已存在



#### 2. 增加分数 (ZINCRBY)

- **`Double incrementScore(K key, V value, double delta)`**

  - **对应命令**：`ZINCRBY key delta member`

  - **说明**：为指定元素的分数增加 `delta`。如果元素不存在，先添加（分数为 0）再增加

  - **返回值**：增加后的新分数

  - **示例**：

    ```java
    // 玩家获得 30 分
    Double newScore = redisTemplate.opsForZSet().incrementScore("game:ranking", "player:1", 30);
    // newScore = 140.0 (110 + 30)
    
    // 负数表示减分
    redisTemplate.opsForZSet().incrementScore("game:ranking", "player:1", -10);
    ```



#### 3. 按排名范围查询 (ZRANGE / ZREVRANGE)

- **`Set<V> range(K key, long start, long end)`**

  - **对应命令**：`ZRANGE key start end`

  - **说明**：按分数**从小到大**（升序）获取指定排名范围的元素

  - **注意**：索引从 0 开始，`-1` 表示最后一个元素

  - **示例**：

    ```java
    // 获取分数最低的 3 人（第 0-2 名）
    Set<Object> bottom3 = redisTemplate.opsForZSet().range("game:ranking", 0, 2);
    
    // 获取所有元素
    Set<Object> all = redisTemplate.opsForZSet().range("game:ranking", 0, -1);
    ```



- **`Set<V> reverseRange(K key, long start, long end)`**

  - **对应命令**：`ZREVRANGE key start end`

  - **说明**：按分数**从大到小**（降序）获取指定排名范围的元素

  - **示例**：

    ```java
    // 获取积分榜 Top 3
    Set<Object> top3 = redisTemplate.opsForZSet().reverseRange("game:ranking", 0, 2);
    ```



- **`Set<TypedTuple<V>> rangeWithScores(K key, long start, long end)`**

  - **对应命令**：`ZRANGE key start end WITHSCORES`

  - **说明**：升序获取元素及其分数

  - **示例**：

    ```java
    Set<ZSetOperations.TypedTuple<Object>> results = 
        								redisTemplate.opsForZSet().rangeWithScores("game:ranking", 0, 2);
    results.forEach(tuple -> {
        	System.out.println("玩家: " + tuple.getValue() + ", 分数: " + tuple.getScore());
    });
    ```



- **`Set<TypedTuple<V>> reverseRangeWithScores(K key, long start, long end)`**

  - **对应命令**：`ZREVRANGE key start end WITHSCORES`

  - **说明**：降序获取元素及其分数

  - **示例**：

    ```java
    // 获取 Top 10 及其分数
    Set<ZSetOperations.TypedTuple<Object>> top10 =     		
        							redisTemplate.opsForZSet().reverseRangeWithScores("game:ranking", 0, 9);
    ```



#### 4. 按分数范围查询 (ZRANGEBYSCORE / ZREVRANGEBYSCORE)

- **`Set<V> rangeByScore(K key, double min, double max)`**

  - **对应命令**：`ZRANGEBYSCORE key min max`

  - **说明**：按分数**从小到大**获取指定分数区间 `[min, max]` 内的元素

  - **示例**：

    ```java
    // 获取分数在 100 到 150 之间的玩家
    Set<Object> players = redisTemplate.opsForZSet().rangeByScore("game:ranking", 100, 150);
    ```



- **`Set<V> rangeByScore(K key, double min, double max, long offset, long count)`**

  - **对应命令**：`ZRANGEBYSCORE key min max LIMIT offset count`

  - **说明**：按分数升序获取，并支持分页

  - **示例**：

    ```java
    // 跳过前 10 个，取 20 个
    Set<Object> page = redisTemplate.opsForZSet()    
        							.rangeByScore("game:ranking", 100, 200, 10, 20);
    ```



- **`Set<TypedTuple<V>> rangeByScoreWithScores(K key, double min, double max)`**

  - **对应命令**：`ZRANGEBYSCORE key min max WITHSCORES`

  - **说明**：按分数升序获取元素及其分数

  - **示例**：

    ```java
    Set<ZSetOperations.TypedTuple<Object>> results =     
        						redisTemplate.opsForZSet()
        									.rangeByScoreWithScores("game:ranking", 100, 150);
    ```



- **`Set<V> reverseRangeByScore(K key, double min, double max)`**
  - **对应命令**：`ZREVRANGEBYSCORE key max min`
  - **说明**：按分数**从大到小**获取指定分数区间内的元素
  - **注意**：虽然方法参数是 `min, max`，但结果是降序的



- **`Set<TypedTuple<V>> reverseRangeByScoreWithScores(K key, double min, double max)`**
  - **对应命令**：`ZREVRANGEBYSCORE key max min WITHSCORES`
  - **说明**：按分数降序获取元素及其分数



#### 5. 查询元素信息 (ZSCORE / ZRANK / ZREVRANK)

- **`Double score(K key, Object o)`**

  - **对应命令**：`ZSCORE key member`

  - **说明**：获取指定元素的分数。如果元素不存在，返回 `null`

  - **示例**：

    ```java
    Double score = redisTemplate.opsForZSet().score("game:ranking", "player:2");
    ```



- **`List<Double> score(K key, Object... o)`**

  - **对应命令**：`ZMSCORE key member [member ...]`（Redis 6.2+）

  - **说明**：批量获取多个元素的分数

  - **示例**：

    ```java
    List<Double> scores = redisTemplate.opsForZSet()    
        								.score("game:ranking", "player:1", "player:2", "player:3");
    ```



- **`Long rank(K key, Object o)`**

  - **对应命令**：`ZRANK key member`

  - **说明**：获取指定元素的**升序**排名（从 0 开始，分数最低的排名为 0）

  - **示例**：

    ```java
    Long rank = redisTemplate.opsForZSet().rank("game:ranking", "player:1");// 如果 player:1 分数最低，rank = 0
    ```



- **`Long reverseRank(K key, Object o)`**

  - **对应命令**：`ZREVRANK key member`

  - **说明**：获取指定元素的**降序**排名（从 0 开始，分数最高的排名为 0）

  - **示例**：

    ```java
    Long topRank = redisTemplate.opsForZSet().reverseRank("game:ranking", "player:1");
    							// 如果 player:1 分数最高，topRank = 0
    ```



#### 6. 统计 (ZCARD / ZCOUNT)

- **`Long size(K key)`**

  - **对应命令**：`ZCARD key`

  - **说明**：获取 ZSet 中的元素总数

  - **示例**：

    ```java
    Long totalPlayers = redisTemplate.opsForZSet().size("game:ranking");
    ```



- **`Long count(K key, double min, double max)`**

  - **对应命令**：`ZCOUNT key min max`

  - **说明**：统计分数在 `[min, max]` 区间内的元素数量

  - **示例**：

    ```java
    // 统计分数在 100-200 之间的玩家数量
    Long count = redisTemplate.opsForZSet().count("game:ranking", 100, 200);
    ```



#### 7. 删除元素 (ZREM / ZREMRANGEBYRANK / ZREMRANGEBYSCORE)

- **`Long remove(K key, Object... values)`**

  - **对应命令**：`ZREM key member [member ...]`

  - **说明**：移除一个或多个指定元素

  - **返回值**：成功移除的元素数量

  - **示例**：

    ```java
    Long removed = redisTemplate.opsForZSet().remove("game:ranking", "player:1", "player:2");
    ```



- **`Long removeRange(K key, long start, long end)`**

  - **对应命令**：`ZREMRANGEBYRANK key start end`

  - **说明**：按排名范围删除元素（升序排名）

  - **示例**：

    ```java
    // 删除排名最低的 10 个玩家（清理不活跃用户）
    redisTemplate.opsForZSet().removeRange("game:ranking", 0, 9);
    ```



- **`Long removeRangeByScore(K key, double min, double max)`**

  - **对应命令**：`ZREMRANGEBYSCORE key min max`

  - **说明**：按分数范围删除元素

  - **示例**：

    ```java
    // 删除分数低于 50 的玩家
    redisTemplate.opsForZSet().removeRangeByScore("game:ranking", 0, 50);
    ```



#### 8. 弹出元素 (ZPOPMIN / ZPOPMAX)

- **`TypedTuple<V> popMin(K key)`**

  - **对应命令**：`ZPOPMIN key`（Redis 5.0+）

  - **说明**：弹出并返回分数最低的元素及其分数

  - **示例**：

    ```java
    // 处理优先级最低的任务
    ZSetOperations.TypedTuple<Object> task = redisTemplate.opsForZSet()
        											.popMin("tasks:queue");
    if (task != null) { 
        System.out.println("任务: " + task.getValue() + ", 优先级: " + task.getScore());
    }
    ```



- **`Set<TypedTuple<V>> popMin(K key, long count)`**
  - **对应命令**：`ZPOPMIN key count`
  - **说明**：弹出多个分数最低的元素



- **`TypedTuple<V> popMax(K key)`**

  - **对应命令**：`ZPOPMAX key`（Redis 5.0+）

  - **说明**：弹出并返回分数最高的元素及其分数

  - **示例**：

    ```java
    // 处理优先级最高的任务
    ZSetOperations.TypedTuple<Object> urgentTask = redisTemplate.opsForZSet().popMax("tasks:queue");
    ```



- **`Set<TypedTuple<V>> popMax(K key, long count)`**
  - **对应命令**：`ZPOPMAX key count`
  - **说明**：弹出多个分数最高的元素



- **`TypedTuple<V> popMin(K key, long timeout, TimeUnit unit)`**
  - **对应命令**：`BZPOPMIN key timeout`（Redis 5.0+）
  - **说明**：阻塞式弹出分数最低的元素



- **`TypedTuple<V> popMax(K key, long timeout, TimeUnit unit)`**
  - **对应命令**：`BZPOPMAX key timeout`（Redis 5.0+）
  - **说明**：阻塞式弹出分数最高的元素



#### 9. 集合运算 (ZUNION / ZINTER)

- **`Long unionAndStore(K key, K otherKey, K destKey)`**

  - **对应命令**：`ZUNIONSTORE destination numkeys key [key ...]`

  - **说明**：计算并集并存储到新集合。分数相加

  - **返回值**：结果集合的元素数量

  - **示例**：

    ```java
    // 合并两个服务器的排行榜
    redisTemplate.opsForZSet()    
        		.unionAndStore("server1:ranking", "server2:ranking", "global:ranking");
    ```



- **`Long unionAndStore(K key, Collection<K> otherKeys, K destKey)`**
  - **对应命令**：`ZUNIONSTORE destination numkeys key [key ...]`
  - **说明**：计算多个 ZSet 的并集



- **`Long intersectAndStore(K key, K otherKey, K destKey)`**

  - **对应命令**：`ZINTERSTORE destination numkeys key [key ...]`

  - **说明**：计算交集并存储。只保留同时存在于所有集合中的元素，分数相加

  - **示例**：

    ```java
    // 找出在两个榜单中都出现的玩家
    redisTemplate.opsForZSet()    
        			.intersectAndStore("ranking:week", "ranking:month", "ranking:both");
    ```



- **`Long intersectAndStore(K key, Collection<K> otherKeys, K destKey)`**
  - **对应命令**：`ZINTERSTORE destination numkeys key [key ...]`
  - **说明**：计算多个 ZSet 的交集



#### 10. 迭代扫描 (ZSCAN)

- **`Cursor<TypedTuple<V>> scan(K key, ScanOptions options)`**

  - **对应命令**：`ZSCAN key cursor [MATCH pattern] [COUNT count]`

  - **说明**：迭代扫描 ZSet，适用于大集合

  - **示例**：

    ```java
    ScanOptions options = ScanOptions.scanOptions()   
        								.match("player:*")    
        								.count(100)    
        								.build();
    Cursor<ZSetOperations.TypedTuple<Object>> cursor = redisTemplate.opsForZSet()
        															.scan("game:ranking", options);
    while (cursor.hasNext()) {
        ZSetOperations.TypedTuple<Object> tuple = cursor.next();    
        System.out.println(tuple.getValue() + ": " + tuple.getScore());
    }
    
    cursor.close();
    ```



## 6. 通用 Key 操作🔑

- 通用 Key 操作适用于**所有数据类型**（String、Hash、List、Set、ZSet 等）
- 这些方法直接通过 `redisTemplate` 调用，而不是通过 `opsForValue()` 等操作器
- 假设已注入 `RedisTemplate<String, Object> redisTemplate`

### 6.1 Key 的基本操作

#### 1. 删除 Key (DEL)

- **`Boolean delete(K key)`**

  - **对应命令**：`DEL key`

  - **说明**：删除单个 key

  - **返回值**：`true` 表示删除成功，`false` 表示 key 不存在

  - **示例**：

    ```java
    Boolean deleted = redisTemplate.delete("user:1000");
    ```



- **`Long delete(Collection<K> keys)`**

  - **对应命令**：`DEL key1 key2 ...`

  - **说明**：批量删除多个 key

  - **返回值**：成功删除的 key 数量

  - **示例**：

    ```java
    List<String> keys = Arrays.asList("user:1", "user:2", "user:3");
    Long count = redisTemplate.delete(keys);
    ```



- **`Boolean unlink(K key)`**

  - **对应命令**：`UNLINK key`（Redis 4.0+）

  - **说明**：异步删除 key，不阻塞 Redis。推荐用于删除大 key

  - **示例**：

    ```java
    // 异步删除大集合
    redisTemplate.unlink("large:set:key");
    ```



- **`Long unlink(Collection<K> keys)`**
  - **对应命令**：`UNLINK key1 key2 ...`
  - **说明**：批量异步删除



#### 2. 检查 Key 存在 (EXISTS)

- **`Boolean hasKey(K key)`**

  - **对应命令**：`EXISTS key`

  - **说明**：检查 key 是否存在

  - **示例**：

    ```java
    Boolean exists = redisTemplate.hasKey("user:1000");
    if (exists) {    // key 存在
    }
    ```



- **`Long countExistingKeys(Collection<K> keys)`**

  - **对应命令**：`EXISTS key1 key2 ...`

  - **说明**：统计存在的 key 数量

  - **示例**：

    ```java
    List<String> keys = Arrays.asList("user:1", "user:2", "user:999");
    Long existingCount = redisTemplate.countExistingKeys(keys);	// 例如返回 2（只有 user:1 和 user:2 存在）
    ```



#### 3. 查看 Key 类型 (TYPE)

- **`DataType type(K key)`**

  - **对应命令**：`TYPE key`

  - **说明**：获取 key 的数据类型

  - **返回值**：`DataType` 枚举（NONE, STRING, LIST, SET, ZSET, HASH, STREAM）

  - **示例**：

    ```java
    DataType type = redisTemplate.type("user:1000");
    if (type == DataType.HASH) {    // 这是一个 Hash 类型
    }
    ```



#### 4. 重命名 Key (RENAME)

- **`void rename(K oldKey, K newKey)`**

  - **对应命令**：`RENAME oldKey newKey`

  - **说明**：重命名 key。如果 newKey 已存在，会被覆盖

  - **注意**：如果 oldKey 不存在，会抛出异常

  - **示例**：

    ```java
    redisTemplate.rename("old:key", "new:key");
    ```



- **`Boolean renameIfAbsent(K oldKey, K newKey)`**

  - **对应命令**：`RENAMENX oldKey newKey`

  - **说明**：只有当 newKey 不存在时才重命名

  - **返回值**：`true` 表示重命名成功，`false` 表示 newKey 已存在

  - **示例**：

    ```java
    Boolean renamed = redisTemplate.renameIfAbsent("old:key", "new:key");
    ```



### 6.2 过期时间管理

#### 1. 设置过期时间 (EXPIRE)

- **`Boolean expire(K key, long timeout, TimeUnit unit)`**

  - **对应命令**：`EXPIRE key seconds` 或 `PEXPIRE key milliseconds`

  - **说明**：设置 key 的过期时间

  - **返回值**：`true` 表示设置成功，`false` 表示 key 不存在

  - **示例**：

    ```java
    // 设置 30 分钟过期
    redisTemplate.expire("session:abc123", 30, TimeUnit.MINUTES);// 设置 5 秒过期
    redisTemplate.expire("verification:code", 5, TimeUnit.SECONDS);
    ```



- **`Boolean expire(K key, Duration timeout)`**

  - **对应命令**：`EXPIRE key seconds`

  - **说明**：使用 `Duration` 设置过期时间（推荐，更清晰）

  - **示例**：

    ```java
    redisTemplate.expire("session:abc123", Duration.ofMinutes(30));
    ```



#### 2. 设置过期时间戳 (EXPIREAT)

- **`Boolean expireAt(K key, Date date)`**

  - **对应命令**：`EXPIREAT key timestamp`

  - **说明**：设置 key 在指定日期时间过期

  - **示例**：

    ```java
    // 设置在明天凌晨过期
    Calendar calendar = Calendar.getInstance();
    calendar.add(Calendar.DAY_OF_MONTH, 1);
    calendar.set(Calendar.HOUR_OF_DAY, 0);
    calendar.set(Calendar.MINUTE, 0);
    calendar.set(Calendar.SECOND, 0);
    redisTemplate.expireAt("daily:stats", calendar.getTime());
    ```



- **`Boolean expireAt(K key, Instant expireAt)`**

  - **对应命令**：`EXPIREAT key timestamp`

  - **说明**：使用 `Instant` 设置过期时间戳

  - **示例**：

    ```java
    Instant tomorrow = Instant.now().plus(1, ChronoUnit.DAYS);
    redisTemplate.expireAt("temp:data", tomorrow);
    ```



#### 3. 查看剩余时间 (TTL)

- **`Long getExpire(K key)`**

  - **对应命令**：`TTL key`

  - **说明**：获取 key 的剩余生存时间（秒）

  - **返回值**：

    - 正数：剩余秒数
    - `-1`：key 存在但未设置过期时间
    - `-2`：key 不存在

  - **示例**：

    ```java
    Long ttl = redisTemplate.getExpire("session:abc123");
    if (ttl > 0) {    
        System.out.println("剩余 " + ttl + " 秒");
    } else if (ttl == -1) {    
        System.out.println("永久有效");
    } else {    
        System.out.println("key 不存在");
    }
    ```



- **`Long getExpire(K key, TimeUnit timeUnit)`**

  - **对应命令**：`TTL key` 或 `PTTL key`

  - **说明**：以指定时间单位获取剩余时间

  - **示例**：

    ```java
    // 获取剩余毫秒数
    Long ttlMillis = redisTemplate.getExpire("key", TimeUnit.MILLISECONDS);
    
    // 获取剩余分钟数
    Long ttlMinutes = redisTemplate.getExpire("key", TimeUnit.MINUTES);
    ```



#### 4. 移除过期时间 (PERSIST)

- **`Boolean persist(K key)`**

  - **对应命令**：`PERSIST key`

  - **说明**：移除 key 的过期时间，使其永久有效

  - **返回值**：`true` 表示移除成功，`false` 表示 key 不存在或未设置过期时间

  - **示例**：

    ```java
    // 将临时 session 转为永久
    redisTemplate.persist("session:abc123");
    ```



### 6.3 Key 的查找和遍历

#### 1. 模式匹配查找 (KEYS)

- **`Set<K> keys(K pattern)`**

  - **对应命令**：`KEYS pattern`

  - **说明**：查找匹配指定模式的所有 key

  - **警告**：这是一个 O(N) 的阻塞操作，**生产环境禁用**，应使用 `scan()` 代替

  - **示例**：

    ```java
    // 查找所有 user 相关的 
    keySet<String> userKeys = redisTemplate.keys("user:*");// 查找所有 key（非常危险！）
    Set<String> allKeys = redisTemplate.keys("*");
    ```

- **模式说明**：

  - `*`：匹配任意多个字符（`user:*` 匹配 `user:1`, `user:100` 等）
  - `?`：匹配单个字符（`user:?` 匹配 `user:1`, `user:a` 等）
  - `[abc]`：匹配括号内的任意一个字符（`user:[12]*` 匹配 `user:1xxx` 或 `user:2xxx`）



#### 2. 迭代扫描 (SCAN)

- **`Cursor<K> scan(ScanOptions options)`**

  - **对应命令**：`SCAN cursor [MATCH pattern] [COUNT count]`

  - **说明**：迭代扫描 key，不阻塞 Redis。**生产环境推荐**

  - **示例**：

    ```java
    // 扫描所有 user 相关的 
    keyScanOptions options = ScanOptions.scanOptions()    
        								.match("user:*")    
        								.count(100)  // 每次迭代的建议数量    
        								.build();
    Cursor<String> cursor = redisTemplate.scan(options);
    try {    
        while (cursor.hasNext()) {        
            String key = cursor.next();        
            System.out.println("找到 key: " + key);    
        }} finally {    
        cursor.close();  // 必须关闭游标
    }
    ```

  

  - **批量处理示例**：

    ```java
    // 批量删除匹配的 key
    ScanOptions options = ScanOptions.scanOptions()
        .match("cache:*")
        .count(1000)
        .build();
    
    Cursor<String> cursor = redisTemplate.scan(options);
    List<String> keysToDelete = new ArrayList<>();
    
    try {
        while (cursor.hasNext()) {
            keysToDelete.add(cursor.next());
            
            // 每 500 个批量删除一次
            if (keysToDelete.size() >= 500) {
                redisTemplate.delete(keysToDelete);
                keysToDelete.clear();
            }
        }
        
        // 删除剩余的 key
        if (!keysToDelete.isEmpty()) {
            redisTemplate.delete(keysToDelete);
        }
    } finally {
        cursor.close();
    }
    ```



#### 3. 随机 Key (RANDOMKEY)

- **`K randomKey()`**

  - **对应命令**：`RANDOMKEY`

  - **说明**：从当前数据库中随机返回一个 key

  - **示例**：

    ```java
    String randomKey = redisTemplate.randomKey();
    ```



### 6.4 序列化与持久化

#### 1. 序列化 Key (DUMP)

- **`byte[] dump(K key)`**

  - **对应命令**：`DUMP key`

  - **说明**：序列化 key 的值，返回字节数组

  - **示例**：

    ```java
    byte[] serialized = redisTemplate.dump("user:1000");// 可以保存到文件或传输到其他 Redis 实例
    ```



#### 2. 反序列化并创建 Key (RESTORE)

- **`void restore(K key, byte[] value, long timeToLive, TimeUnit unit)`**

  - **对应命令**：`RESTORE key ttl serialized-value`

  - **说明**：从序列化的值创建 key

  - **示例**：

    ```java
    // 从序列化数据恢复 key，设置 1 小时过期
    redisTemplate.restore("user:backup", serializedData, 1, TimeUnit.HOURS);// 创建永久 key（ttl = 0）
    redisTemplate.restore("user:backup", serializedData, 0, TimeUnit.SECONDS);
    ```



- **`void restore(K key, byte[] value, long timeToLive, TimeUnit unit, boolean replace)`**

  - **说明**：增加 `replace` 参数，如果为 `true`，会覆盖已存在的 key

  - **示例**：

    ```java
    // 强制覆盖已存在的 
    keyredisTemplate.restore("user:1000", serializedData, 0, TimeUnit.SECONDS, true);
    ```



### 6.5 其他实用操作

#### 1. 移动 Key 到其他数据库 (MOVE)

- **`Boolean move(K key, int dbIndex)`**

  - **对应命令**：`MOVE key db`

  - **说明**：将 key 移动到指定的数据库

  - **返回值**：`true` 表示移动成功，`false` 表示 key 不存在或目标数据库已有同名 key

  - **注意**：仅适用于单机 Redis，不适用于集群模式

  - **示例**：

    ```java
    // 将 key 移动到数据库 1
    Boolean moved = redisTemplate.move("temp:key", 1);
    ```



#### 2. 批量操作工具方法

- **`List<Object> executePipelined(SessionCallback<?> session)`**

  - **说明**：使用管道批量执行命令，提高性能

  - **示例**：

    ```java
    List<Object> results = redisTemplate.executePipelined(
        new SessionCallback<Object>() {    
            @Override    
            public Object execute(RedisOperations operations) throws DataAccessException {        
                for (int i = 0; i < 1000; i++) {            
                    operations.opsForValue().set("key:" + i, "value:" + i);
                }        
                return null;  // 管道中返回值无意义
            }});
    ```

- **`Object execute(RedisCallback<T> action)`**

  - **说明**：执行原生 Redis 命令

  - **示例**：

    ```java
    // 执行原生命令
    Object result = redisTemplate.execute(new RedisCallback<Object>() {    
        @Override    
        public Object doInRedis(RedisConnection connection) {        
            return connection.ping();    
        }});
    ```



### 6.6 实用示例

```java
// 完整的 Session 管理示例
public class SessionManager {
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 创建 Session
    public void createSession(String sessionId, User user) {
        String key = "session:" + sessionId;
        redisTemplate.opsForHash().put(key, "userId", user.getId());
        redisTemplate.opsForHash().put(key, "username", user.getUsername());
        redisTemplate.expire(key, Duration.ofMinutes(30));
    }
    
    // 续期 Session
    public void renewSession(String sessionId) {
        String key = "session:" + sessionId;
        if (redisTemplate.hasKey(key)) {
            redisTemplate.expire(key, Duration.ofMinutes(30));
        }
    }
    
    // 检查 Session 是否即将过期
    public boolean isSessionExpiringSoon(String sessionId) {
        String key = "session:" + sessionId;
        Long ttl = redisTemplate.getExpire(key, TimeUnit.MINUTES);
        return ttl != null && ttl > 0 && ttl < 5; // 剩余时间少于 5 分钟
    }
    
    // 删除 Session
    public void deleteSession(String sessionId) {
        redisTemplate.delete("session:" + sessionId);
    }
    
    // 清理过期的 Session（使用 SCAN）
    public void cleanupExpiredSessions() {
        ScanOptions options = ScanOptions.scanOptions()
            .match("session:*")
            .count(100)
            .build();
        
        Cursor<String> cursor = redisTemplate.scan(options);
        try {
            while (cursor.hasNext()) {
                String key = cursor.next();
                Long ttl = redisTemplate.getExpire(key);
                if (ttl == -2) {  // key 不存在
                    // 已过期，无需处理
                } else if (ttl == -1) {  // 永久 key，异常情况
                    // 设置默认过期时间
                    redisTemplate.expire(key, Duration.ofMinutes(30));
                }
            }
        } finally {
            cursor.close();
        }
    }
}
```



### 6.7 最佳实践

- ✅ **使用 `scan()` 代替 `keys()`**：避免阻塞 Redis
- ✅ **合理设置过期时间**：防止内存泄漏
- ✅ **使用命名规范**：如 `类型:业务:ID`（`user:profile:1000`）
- ✅ **删除大 key 时使用 `unlink()`**：避免阻塞
- ✅ **使用管道批量操作**：提高性能
- ✅ **定期清理无效 key**：使用 `scan()` 配合 TTL 检查
- ❌ **避免在生产环境使用 `keys()`**：会阻塞整个 Redis
- ❌ **避免存储过大的 key**：单个 key 不要超过 10MB
- ❌ **避免使用不带过期时间的 key**：除非确实需要永久存储



# SDR 高级数据结构操作

## 1. 操作 BitMap (位图)

在 SDR 中操作 BitMap，新手最容易碰壁的第一点就是：**你在 `RedisTemplate` 的 API 里，根本找不到 `opsForBitMap()` 这个方法**



### 1.1 SDR 中的 BitMap 定位

#### 1. 归属 `opsForValue()` 管辖

- **BitMap 的底层数据结构其实就是 String**

  因此，SDR 将所有针对 BitMap 的基础操作（如 `SETBIT`, `GETBIT`）全部“隐藏”在了 **`ValueOperations`** 中，也就是你需要通过 `redisTemplate.opsForValue()` 来调用它



#### 2. ⭐必须使用 `StringRedisTemplate`

- **大坑预警**： 

  - 在前面的基础篇中，我们强烈推荐大家自定义 `RedisTemplate<String, Object>`，并使用 Jackson JSON 作为 Value 的序列化器，这对于存储普通的 Java 对象来说非常完美。 **但是，这个配置好的模板，绝对不能用来操作 BitMap！**

  

- **为什么会踩坑？**

  1. 假设你用 JSON 序列化器去执行了 `setBit(key, offset, true)`
  2. SDR 会在底层直接修改该 Key 对应的纯字节数组
  3. 如果这个 Key 是新创建的，它将是一个纯粹的二进制串，**这完全不符合 JSON 的格式规范**
  4. 之后如果你手滑，或者业务里其他人用 `get(key)` 去获取这个数据，Jackson 序列化器在尝试把它反序列化为 JSON/Object 时，会抛出**解析异常 (JsonParseException)**！
  5. 如果你使用的是 Spring Boot 最原始默认的 JDK 序列化器，情况更糟：它会在数据头部自动加上一段魔法字符（如 `\xac\xed\x00\x05...`），这会导致你在代码里算好的 Offset 偏移量在 Redis 底层**彻底错乱**，存取的值完全不对

  

- **最佳实践**：

  - 在你的 Service 类中，**专门注入 `StringRedisTemplate` 来处理所有的 BitMap 业务**
  - 因为它的 Key 和 Value 序列化器都是纯粹的 `StringRedisSerializer`，它对底层字节串的操作是完全透明、干净且安全的

  ```java
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.data.redis.core.StringRedisTemplate;
  import org.springframework.stereotype.Service;
  
  @Service
  public class CheckInService {
  
      // ⭐ 操作 BitMap，请认准且只认准 StringRedisTemplate
      @Autowired
      private StringRedisTemplate stringRedisTemplate; 
  
      // ...
  }
  ```



### 1.2 基础读写操作 (setBit / getBit)

为了更好地理解 API 的应用，我们统一假设一个最经典的业务场景：**用户每日签到 (Check-in)**

- **Key**：`sign:20260226`（代表某一天的签到总表）
- **Offset**：用户的唯一短数字 ID（例如 `1001`）
- **Value**：`true` 代表 1（已签到），`false` 代表 0（未签到）



#### 1. 设置位值 (setBit)

- **`Boolean setBit(K key, long offset, boolean value)`**

  - **对应命令**：`SETBIT key offset value`

  - **说明**：将指定 `key` 的底层字符串在 `offset` 偏移量上的位，设置为 `value`

  - **返回值 (⭐极其重要)**：

    - SDR 中 `setBit` 的返回值，代表的是该位置被修改 **之前** 的 **旧值**！
    - 返回 `false`：表示该位之前是 0（或者该位压根没被初始化过）。这通常意味着用户是**首次操作**
    - 返回 `true`：表示该位之前已经是 1 了。这通常意味着用户在**重复操作**
    - 这个特性天然适合用来做**幂等性判断**（例如：防止用户重复点击签到按钮而多次获得积分）

  - **实战代码示例**：

    ```java
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    
    public void userCheckIn(Long userId) {
        String signKey = "sign:20260226";
    
        // 执行 SETBIT，将 offset(userId) 设为 true(1)
        // isOldValue 接收的是修改前的旧状态
        Boolean isOldValue = stringRedisTemplate.opsForValue().setBit(signKey, userId, true);
    
        // 注意：由于 isOldValue 可能是 null（极少数异常情况），推荐使用 Boolean.FALSE.equals
        if (Boolean.FALSE.equals(isOldValue)) {
            System.out.println("签到成功！您是今天首次签到，增加 10 积分。");
            // ... 执行加积分、写流水等业务逻辑
        } else {
            System.out.println("您今天已经签到过了，请勿重复点击！");
            // ... 直接返回，不给积分（幂等拦截）
        }
    }
    ```



#### 2. 获取位值 (getBit)

- **`Boolean getBit(K key, long offset)`**

  - **对应命令**：`GETBIT key offset`

  - **说明**：获取指定 `key` 在 `offset` 偏移量上的二进制位的值

  - **返回值**：

    - 返回 `true`：表示该位的值为 1
    - 返回 `false`：表示该位的值为 0

  - **⭐ 越界与空值安全**：

    - 与数组下标越界会抛异常不同，Redis 的 `GETBIT` 极其安全
    - 如果你查询的 `key` 压根**不存在**，或者你查询的 `offset` 远远**超出了**当前字符串实际分配的长度，SDR 都**不会报错**，而是会统一温柔地返回 `false`（即认为该位是 0）

  - **实战代码示例**：

    ```java
    public boolean checkUserSignedIn(Long userId) {
        String signKey = "sign:20260226";
    
        // 查询用户今天是否已签到
        Boolean hasSignedIn = stringRedisTemplate.opsForValue().getBit(signKey, userId);
    
        // 同样推荐使用 Boolean.TRUE.equals 防范 NullPointerException
        if (Boolean.TRUE.equals(hasSignedIn)) {
            System.out.println("前端展示：高亮的【已签到】大拇指图标 👍");
            return true;
        } else {
            System.out.println("前端展示：灰色的【未签到】大拇指图标");
            return false;
        }
    
        /* * 越界安全测试：
         * 假设今天根本没人签到（sign:20260226 不存在）
         * 或者查一个极其离谱的用户 ID
         * Boolean result = stringRedisTemplate.opsForValue().getBit(signKey, 999999999L);
         * 此时 result 安全地等于 false，代码绝对不会抛异常。
         */
    }
    ```



### 1.3 高级位域操作(BITFIELD)——内存压榨利器

如果说 `SETBIT` 是按“单个位(1 bit)”来操作，那么 `BITFIELD` 就是按“块(多 bits)”来操作。 

当我们需要为海量用户存储多个小数值属性（例如：游戏玩家的等级、VIP 标识、当前经验值）时，如果用 Hash 或者 JSON 存储，除了数据本身，Key 和 Field 的额外开销会非常大。

而 `BITFIELD` 可以将这些属性“紧紧地塞”在同一个 String 的二进制串中，将内存利用率推向极致

Spring Data Redis 提供了非常优雅的链式构建器 **`BitFieldSubCommands`** 来实现这一高级功能



#### 0. SDR 核心 API 解析

在 SDR 中执行 `BITFIELD`，统一使用以下方法：

- **`List<Long> bitField(K key, BitFieldSubCommands subCommands)`**

这里的核心是构建 `BitFieldSubCommands` 对象，它是一个链式构建器，允许你一次性打包多个操作（GET/SET/INCR）发送给 Redis

下面我们将极其详细地拆解这个构建器的每一个核心组件



#### 1. `BitFieldSubCommands` 的三大核心基石

在使用构建器发号施令之前，必须先搞懂它依赖的三个核心参数对象：**类型 (Type)**、**偏移量 (Offset)** 和 **溢出控制 (Overflow)**



##### 数据类型与位宽 (`BitFieldType`)

- **核心概念：打破数据类型的边界** 

  - 在传统的 Java 语言中，数字类型的位宽是死板固定的（如 `byte` 占 8 位，`short` 占 16 位，`int` 占 32 位）

    但在 Redis 的 `BITFIELD` 中，你可以**随心所欲地定义任何位宽**（例如只占 3 位、7 位或 11 位的数字）

    这就是它被称为“内存压榨利器”的根本原因—— **按需分配，绝不浪费哪怕 1 个 bit 的空间**

  

- **SDR API 详解**： 

  SDR 提供了两个极其直观的静态方法来定义数据类型，主要区分**“是否带负数”**：

  1. **无符号整数：`BitFieldType.unsigned(int bits)`**

     - **含义**：没有负数，所有的二进制位全部用于表示数值的绝对大小。对应 Redis 原生命令中的前缀 `u`（如 `u8`）
     - **硬性限制 (防坑)**：
       - 最大只能支持到 **63 位**（即 `u63`）。这是 Redis 底层 C 语言实现的一个限制，为了防止无符号 64 位整数在转换时发生符号位溢出混淆
     - **业务选型参考**：
       - `.unsigned(1)`：`u1`，取值 `0 ~ 1`。相当于极度压缩的 `boolean`（比如存性别：0女 1男）
       - `.unsigned(4)`：`u4`，取值 `0 ~ 15`。适合存 VIP 等级（目前市面上大部分游戏的 VIP 很少超过 15 级）
       - `.unsigned(8)`：`u8`，取值 `0 ~ 255`。适合存角色的等级（Level）
       - `.unsigned(32)`：`u32`，取值 `0 ~ 42.9亿`。适合存金币、积分等较大的非负数值

     

  2. **有符号整数：`BitFieldType.signed(int bits)`**

     - **含义**：最高位被强行征用充当符号位（`0` 代表正数，`1` 代表负数）。对应 Redis 原生命令中的前缀 `i`（如 `i8`）
     - **硬性限制**：最大可以支持满血的 **64 位**（即 `i64`）
     - **业务选型参考**：
       - `.signed(8)`：`i8`，取值 `-128 ~ 127`。适合存储温度变化值、简单的坐标偏移量，或者允许透支的小额信用分
       - `.signed(16)`：`i16`，取值 `-32768 ~ 32767`



##### 偏移量 (`Offset`) 

- **作用**：告诉 Redis 从二进制串的第几个位置开始读写数据(可以简单理解成一个全是0和1组成的字节数组，这个偏移量就是数组索引或下标)

- **SDR API** (提供了两种截然不同的定位方式)：

  1. **绝对位偏移**：

     - `BitFieldSubCommands.Offset.offset(long index)`
     - **说明**：直接指定从第几个 bit 开始。例如 `.offset(8)` 就是精确地从第 8 个二进制位开始操作

     

  2. **基于类型的偏移**：

     - `BitFieldSubCommands.Offset.typeBasedOffset(long index)`
     - **说明**：对应原生 Redis 命令里的 `#` 符号。它会根据你定义的 `BitFieldType` 自动帮你算乘法！
     - **举例**：如果你操作的类型是 `u8` (占8位)，使用 `.typeBasedOffset(2)`，Redis 会自动算出实际的 bit 偏移量是 `2 * 8 = 16`，即从第 16 个位开始操作。这在存储结构统一的数组时极其好用



##### 溢出控制策略 (`Overflow`)

- **作用**：仅用于 `INCRBY` (加减) 操作。当加减的结果超出了该类型所能表示的最大/最小值时，决定 Redis 该如何处理
- **SDR API** (对应 `BitFieldSubCommands.BitFieldIncrBy.Overflow` 枚举)：
  1. **`WRAP` (折返 / 默认)**：
     - 类似 C 语言的溢出处理。例如 `u8` 的最大值是 255。如果当前值是 255，再加 1，就会“折返”变成 `0`
  2. **`SAT` (饱和)**：
     - **极其常用的业务策略**。到达最大值后，继续加依然是最大值，不再变化；减到最小值后，继续减还是最小值
     - **场景**：游戏满级了（不能再升了）、血量扣到 0 了（不能变成负数）
  3. **`FAIL` (失败)**：
     - 拒绝执行可能导致溢出的操作，对应的返回结果将是 `null`



#### 2. 构建器链式操作

了解了三大基石（类型、偏移量、溢出策略）后，我们来看看 `BitFieldSubCommands` 提供了哪些具体的操作命令

- **创建构建器 (入口)**：

  ```java
  BitFieldSubCommands commands = BitFieldSubCommands.create();
  ```

  *说明*：每次执行 `bitField` 操作前，都需要先 `create()` 一个空的命令集合，然后像拼积木一样把多个子命令（GET/SET/INCR）拼接到一起，最后一次性发给 Redis



##### 1. 读取操作 (GET)

- **API 语法**：

  ```
  commands.get(BitFieldType type).valueAt(Offset offset);
  ```

- **详细解剖**：

  - `.get(type)`：给 Redis 提供解码规则，声明要读取的位宽和符号类型
  - `.valueAt(offset)`：定位读取的起始绝对或相对偏移量

  

- **业务示例**： 获取偏移量为 0 开始的 4位无符号整数（如 VIP 等级）

  ```java
  commands.get(BitFieldType.unsigned(4)).valueAt(BitFieldSubCommands.Offset.offset(0));
  ```



> **为什么获取 (GET) 的时候，也要指定有符号/无符号和位宽？**
>
> **答案核心：因为 Redis 的底层是“无模式”的纯原始二进制流**
>
> 在 Java 中，如果你声明了一个 `int a = 255;`，Java 虚拟机在内存里清清楚楚地记着“这是一个 32 位的整数”。 
>
> 但是！当你把数据存进 Redis 的 String (BitMap底层) 时，Redis 只看到了一长串毫无感情的 `0` 和 `1`，比如：`11111111`。
>
> 它**并没有额外保存**任何关于“这段数据是什么类型”的元数据
>
> 当你想去读这串二进制时：
>
> - 如果你不告诉 Redis 怎么读，它根本不知道该读几位
> - 如果你告诉它读 8 位，但**不告诉它符号**，当它读到 `11111111` 时，它无法独立判断数据的真实含义：
>   - 如果你想要的是**无符号整数 (`u8`)**，这串数字翻译过来应该是 **`255`**
>   - 如果你想要的是**有符号整数 (`i8`)**，第一位 `1` 会被当成负号，翻译过来就是 **`-1`**
>
> **总结**：在 `GET` 时指定 `BitFieldType`，本质上是给 Redis 递过去一个**“模具（或叫解码器）”**，告诉它：“请从 offset 的位置开始，框出 8 个位，并且按照无符号的规则，把它翻译成十进制数字返回给我”



##### 2. 写入操作 (SET)

- **API 语法**：

  ```java
  commands.set(BitFieldType type).valueAt(Offset offset).to(long value);
  ```



- **详细解剖**：
  - `.set(type)`：定义要写入的位段区间大小
  - `.valueAt(offset)`：定义写入的起始点
  - `.to(value)`：你要写入的具体十进制数值。Redis 会自动把你传入的十进制数转成二进制，然后塞进那个区块里
- **返回值注意**：在 SDR 中执行 `bitField` 后，SET 操作返回的是覆盖前那个区块里的 **旧值**



##### 3. 增减操作 (INCRBY)

这是 `BITFIELD` 最强大的指令，它保证了多线程下加减操作的**绝对原子性**

- **API 语法 (基础版 - 默认 WRAP 溢出)**：

  ```java
  commands.incr(BitFieldType type).valueAt(Offset offset).by(long increment);
  ```

  *说明*：`.by(increment)` 传入的是增量。如果是减法，传入负数即可（例如 `.by(-10)`）。如果不特别指定，默认到达最大值后会变成 0（折返）



- **API 语法 (高级版 - 手动指定溢出策略)**：

  *说明*：通过 `.overflow(...)` 可以明确指定当加减结果超出当前数据类型所能表示的极限（最大/最小值）时的处理行为。SDR 提供了三个枚举值：

  - `Overflow.WRAP`：默认行为，溢出后折返
  - `Overflow.SAT`：饱和策略，触碰极限值后不再变化（加不上去，也扣不下来），非常适合处理不能为负数的金币、不能超过上限的经验值
  - `Overflow.FAIL`：失败策略，拒绝执行可能导致溢出的操作，该条子命令对应的返回值为 `null`

  ```java
  commands.incr(BitFieldType type)
          .valueAt(Offset offset)
          .by(long increment)
          .overflow(BitFieldSubCommands.BitFieldIncrBy.Overflow overflowStrategy);
  ```



- **业务示例 (防止透支的扣减)**： 

  - 假设玩家用 500 金币买道具。金币存放在第 12 位开始的 32位无符号整数 (`u32`) 里。我们不希望扣除后变成天文数字（WRAP 导致的下溢），也不希望代码报错。我们希望如果不够扣，就扣到 0 为止（SAT 策略）

  ```java
  commands.incr(BitFieldType.unsigned(32))
          .valueAt(BitFieldSubCommands.Offset.offset(12))
          .by(-500) // 扣减 500
          .overflow(BitFieldSubCommands.BitFieldIncrBy.Overflow.SAT); // 饱和策略：扣到底线 0 就不再扣了
  ```



##### **关于返回值 `List<Long>`** 

在 SDR 中执行 `bitField` 方法后，它的返回值总是长这样

```java
List<Long> results = stringRedisTemplate.opsForValue().bitField(key, commands);
```

很多初学者看到这个 `List` 会感到懵：我明明对不同的字段执行了不同的操作（有的是查，有的是改，有的是加），为什么它只给我返回了一个单纯的数字集合？我该怎么知道哪个数字对应什么结果？

- **核心定律（顺序强绑定）**： 

  在 `BitFieldSubCommands` 构建器中，自上而下**串联了 N 个子命令**（无论它是 GET、SET 还是 INCRBY），返回的 `List<Long>` 中就**绝对且严格地包含 N 个元素**

  List 中的第 `0` 个元素，绝对对应你写的第 `1` 条子命令的执行结果；第 `1` 个元素对应第 `2` 条子命令的结果……依此类推

  

- **不同子命令的返回值含义**：

  - **GET 子命令**：返回读取到的实际值
  - **SET 子命令**：返回该位段被你的新值覆盖**之前**的**旧值**
  - **INCRBY 子命令**：返回执行完加减操作**之后**的**最新值**（如果触发了 `FAIL` 溢出策略，可能返回 `null`）



**实战代码示例 (一次性包含所有操作类型的解析)**：

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

public void parseBitFieldResults() {
    String key = "test:bitfield:results";
    
    // 假设预设前提：当前 key 里有一段 u8 数据，值为 10

    // 1. 疯狂串联三种完全不同的子命令
    BitFieldSubCommands mixedCommands = BitFieldSubCommands.create()
            // 第 1 条子命令：查一下当前值是多少 (GET)
            .get(BitFieldType.unsigned(8)).valueAt(BitFieldSubCommands.Offset.offset(0))
            
            // 第 2 条子命令：给它强制赋值为 50 (SET)
            .set(BitFieldType.unsigned(8)).valueAt(BitFieldSubCommands.Offset.offset(0)).to(50)
            
            // 第 3 条子命令：在刚才的基础上加 20 (INCRBY)
            .incr(BitFieldType.unsigned(8)).valueAt(BitFieldSubCommands.Offset.offset(0)).by(20);

    // 2. 执行并接收返回值
    List<Long> results = stringRedisTemplate.opsForValue().bitField(key, mixedCommands);
    
    // 3. 严格按照子命令的编写顺序去提取并解析结果
    // results 集合大小必定为 3
    System.out.println("完整的执行结果列表: " + results);

    // 解析第 1 条 GET 的结果（返回的是实际读取到的值）
    Long getResult = results.get(0); 
    System.out.println("GET 子命令的结果 (当前真实值): " + getResult); // 输出 10
    
    // 解析第 2 条 SET 的结果（返回的是被覆盖前的旧值！）
    Long setResult = results.get(1); 
    System.out.println("SET 子命令的结果 (覆盖前的旧值): " + setResult); // 输出 10
    
    // 解析第 3 条 INCRBY 的结果（返回的是加完之后的最新值！）
    Long incrResult = results.get(2); 
    System.out.println("INCRBY 子命令的结果 (加完后的最新值): " + incrResult); // 输出 70 (即前面的50 + 20)
}
```



#### 3. 实战代码示例：硬核玩家属性存储

假设我们要在一个 Key (`player:1001:attrs`) 的二进制串中，紧凑地存下三个属性：

1. **VIP 等级**：占 4 位二进制（`u4`，0~15），存放在第 **0~3** 位
2. **玩家等级**：占 8 位二进制（`u8`，0~255），存放在第 **4~11** 位
3. **玩家金币**：占 32 位二进制（`u32`，0~42亿），存放在第 **12~43** 位

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

public void playerBitFieldDemo() {
    String playerKey = "player:1001:attrs";

    // ==========================================
    // 动作 1：初始化玩家属性 (批量 SET)
    // VIP=1级 (u4), 等级=10级 (u8), 金币=500个 (u32)
    // ==========================================
    BitFieldSubCommands initCommands = BitFieldSubCommands.create()
        .set(BitFieldType.unsigned(4)).valueAt(BitFieldSubCommands.Offset.offset(0)).to(1)
        .set(BitFieldType.unsigned(8)).valueAt(BitFieldSubCommands.Offset.offset(4)).to(10)
        .set(BitFieldType.unsigned(32)).valueAt(BitFieldSubCommands.Offset.offset(12)).to(500);

    // 返回值是一个 List，里面包含了每一条子命令执行前的【旧值】
    List<Long> initResults = stringRedisTemplate.opsForValue().bitField(playerKey, initCommands);
    System.out.println("初始化结果(旧值): " + initResults); // [0, 0, 0]

    
    // ==========================================
    // 动作 2：玩家升级打怪 (批量 INCRBY，并演示溢出控制)
    // 等级 +250 级 (故意超出 u8 的 255 限制，使用 SAT 饱和策略防止变为 0)
    // 金币 +1000 个
    // ==========================================
    BitFieldSubCommands incrCommands = BitFieldSubCommands.create()
        .incr(BitFieldType.unsigned(8))
            .valueAt(BitFieldSubCommands.Offset.offset(4))
            .by(250)
            .overflow(BitFieldSubCommands.BitFieldIncrBy.Overflow.SAT) // ⭐ 关键：到达255后不再增加
        .incr(BitFieldType.unsigned(32))
            .valueAt(BitFieldSubCommands.Offset.offset(12))
            .by(1000)
            .overflow(BitFieldSubCommands.BitFieldIncrBy.Overflow.WRAP);

    // 返回值是执行 INCRBY 之后的【最新结果】
    List<Long> incrResults = stringRedisTemplate.opsForValue().bitField(playerKey, incrCommands);
    System.out.println("升级后等级(被限制在255): " + incrResults.get(0)); // 255
    System.out.println("当前金币数量: " + incrResults.get(1)); // 1500

    
    // ==========================================
    // 动作 3：查询玩家属性 (批量 GET)
    // ==========================================
    BitFieldSubCommands getCommands = BitFieldSubCommands.create()
        .get(BitFieldType.unsigned(4)).valueAt(BitFieldSubCommands.Offset.offset(0))  // 查 VIP
        .get(BitFieldType.unsigned(8)).valueAt(BitFieldSubCommands.Offset.offset(4))  // 查 等级
        .get(BitFieldType.unsigned(32)).valueAt(BitFieldSubCommands.Offset.offset(12)); // 查 金币

    List<Long> getResults = stringRedisTemplate.opsForValue().bitField(playerKey, getCommands);
    System.out.println("---- 玩家最新属性看板 ----");
    System.out.println("VIP 等级: " + getResults.get(0));
    System.out.println("玩家等级: " + getResults.get(1));
    System.out.println("玩家金币: " + getResults.get(2));
}
```



### 1.4 统计与底层调用 (BITCOUNT / BITOP)

#### 1. 核心痛点：`opsForValue()` 的 API 缺失

- **大坑预警**：当你开开心心地用 `setBit` 和 `getBit` 完成了用户的签到功能后，产品经理提出一个需求：“统计一下今天总共有多少人签到？” 你习惯性地敲出 `stringRedisTemplate.opsForValue().`，却绝望地发现，**里面根本没有 `bitCount` 和 `bitOp` 这两个极其重要的方法！**
- **为什么？** 因为 Spring 官方在设计高级模板 API 时，没有把这两个略显底层的命令暴露在 `ValueOperations` 接口中
- **终极解法**：
  - 我们必须绕过高层的操作模板，使用 **`redisTemplate.execute(RedisCallback)`** 方法，直接获取底层的 Redis 连接 (`RedisConnection`) 来执行原生的位图命令



#### 2. 统计指令：BITCOUNT

- **业务场景**：统计 `sign:20260226` 这个 Key 中，有多少个位被设置为了 `1`（即统计今天的总签到人数）

- **底层方法**：`connection.bitCount(byte[] key)`

- **序列化细节**：

  - 由于底层 `RedisConnection` 操作的都是纯字节数组 (`byte[]`)，我们需要使用 `StringRedisTemplate` 自带的序列化器将 String 类型的 Key 转换成 `byte[]`

- **实战代码示例**：

  ```java
  @Autowired
  private StringRedisTemplate stringRedisTemplate;
  
  public Long countTodaySignIns() {
      String signKey = "sign:20260226";
  
      // 使用 execute 传入 RedisCallback 回调接口
      Long totalSignIns = stringRedisTemplate.execute(new RedisCallback<Long>() {
          @Override
          public Long doInRedis(RedisConnection connection) throws DataAccessException {
              // 1. 获取 String 序列化器
              RedisSerializer<String> serializer = stringRedisTemplate.getStringSerializer();
              // 2. 将 Key 序列化为 byte[]
              byte[] rawKey = serializer.serialize(signKey);
  
              // 3. 调用底层的 bitCount 命令
              return connection.bitCount(rawKey);
  
              // 💡 补充：如果想限制统计范围，可以调用 connection.bitCount(rawKey, start, end)
              // ！！！千万记住：这里的 start 和 end 是【字节(Byte)】级别的索引，不是 Bit 级别！
          }
      });
  
      return totalSignIns != null ? totalSignIns : 0L;
  }
  
  // ==========================================
  // 🌟 使用 Java 8 Lambda 表达式的优雅简写版：
  // ==========================================
  public Long countTodaySignInsLambda() {
      return stringRedisTemplate.execute((RedisCallback<Long>) connection -> {
          byte[] rawKey = stringRedisTemplate.getStringSerializer().serialize("sign:20260226");
          return connection.bitCount(rawKey);
      });
  }
  ```



#### 3. 批量按位运算：BITOP

- **业务场景**：我们需要统计最近三天（26号、27号、28号）**“连续每天都签到”**的铁杆用户有多少人？

- **实现思路**：对这三天的 BitMap 求**交集 (AND)**，将结果存入一个新的目标 Key 中，然后再对这个目标 Key 执行 `BITCOUNT`

- **底层方法**：`connection.bitOp(BitOperation op, byte[] destination, byte[]... keys)`

  - `BitOperation` 是一个枚举类，包含：`AND`(交集), `OR`(并集), `XOR`(异或), `NOT`(取反)

- **实战代码示例**：

  ```java
  public Long countContinuousSignIns3Days() {
      String key1 = "sign:20260226";
      String key2 = "sign:20260227";
      String key3 = "sign:20260228";
      String destKey = "sign:continuous:3days"; // 存放交集结果的目标 Key
  
      // 1. 执行 BITOP AND 运算
      stringRedisTemplate.execute((RedisCallback<Long>) connection -> {
          RedisSerializer<String> serializer = stringRedisTemplate.getStringSerializer();
  
          byte[] rawDestKey = serializer.serialize(destKey);
          byte[] rawKey1 = serializer.serialize(key1);
          byte[] rawKey2 = serializer.serialize(key2);
          byte[] rawKey3 = serializer.serialize(key3);
  
          // 执行底层的 BITOP 命令
          // 参数1: 运算类型 (AND)
          // 参数2: 目标 Key
          // 参数3: 参与运算的多个数据 Key
          return connection.bitOp(
              RedisStringCommands.BitOperation.AND, 
              rawDestKey, 
              rawKey1, rawKey2, rawKey3
          );
      });
  
      // 2. 对结果目标 Key 执行 BITCOUNT，得出连续签到总人数
      return stringRedisTemplate.execute((RedisCallback<Long>) connection -> {
          byte[] rawDestKey = stringRedisTemplate.getStringSerializer().serialize(destKey);
          return connection.bitCount(rawDestKey);
      });
  }
  ```





# Spring Cache

> Spring Data Redis (SDR) **不是** Spring Cache 的**发明者**，而是**“实现者”**

- 这是 **Spring Framework**（`spring-context` 模块）提供的最强大的功能之一，它定义了一套**缓存抽象 API 和规范**
  - 它允许开发者通过 **统一的注解**（如 `@Cacheable`, `@CachePut`），将缓存逻辑 **声明式** 地集成到业务代码中，而无需手动编写 `redisTemplate.get()` 或 `set()` 这样重复的“样板代码”
  
    SDR 提供了 `RedisCacheManager`，它 **实现** 了 Spring Framework 定义的 `CacheManager` **接口**，
    这个 `RedisCacheManager` 内部封装了 `RedisTemplate`，负责执行**实际**的 Redis GET/SET/DEL 操作
  
    - 这种方式极大地简化了缓存的使用，并使业务代码保持简洁，专注于业务逻辑本身



## 4.1 启用缓存功能：@EnableCaching

- 要使用 Spring 的缓存抽象，第一步是在 Spring Boot 的主启动类（或任何 `@Configuration` 配置类）上添加 `@EnableCaching` 注解

  ```java
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.cache.annotation.EnableCaching;
  
  @SpringBootApplication
  @EnableCaching // <-- 启动缓存功能
  public class Application {
      public static void main(String[] args) {
          SpringApplication.run(Application.class, args);
      }
  }
  ```



- **作用**：这个注解会触发 Spring 容器去查找所有 `@Cacheable`、`@CacheEvict` 等相关的注解，并为它们创建代理（Proxy），以此来拦截方法调用，实现自动的缓存读写



## 4.2 `@Cacheable`（读）

- `@Cacheable` 是最核心、最常用的注解
  - 它被放置在 **“读数据”** 的方法上（例如查询数据库的方法），用于自动执行 **“旁路缓存” (Cache-Aside Pattern) **的 **读逻辑**



- **`@Cacheable` 行为简述**⭐⭐
  - **行为依据**：`value` + `key`
  - **行为**：
    1. 根据 `value` + `key` **查** 缓存
       - 如果 **有**：直接返回缓存值，**方法体不执行**
       - 如果 **没有**：**执行方法体**，拿 **返回值** 去 **存** 缓存，再返回



### 4.2.1 基本用法

- 假设我们有一个根据 ID 查询用户的方法：

  ```java
  @Service
  public class UserServiceImpl implements UserService {
  
      @Autowired
      private UserRepository userRepository; // 模拟数据库操作
  
      // 每当调用此方法时，Spring 会自动处理缓存
      @Cacheable(value = "users", key = "#id")
      @Override
      public User getUserById(Long id) {
          // 这行日志只会在“缓存未命中”时打印
          System.out.println("=== 正在从数据库查询用户: " + id + " ===");
          // 模拟数据库查询
          return userRepository.findById(id).orElse(null);
      }
  }
  ```



### 4.2.2 `@Cacheable` 的工作流程

- 当一个被 `@Cacheable` 标记的方法（如 `getUserById(1L)`）被调用时，Spring 的缓存切面会执行以下流程：
  1. **生成 Key**：
     - Spring 会根据注解中的 `value` 和 `key` 属性来生成一个最终存储在 Redis 中的 Key
     - `value = "users"`：指定了缓存的“命名空间”（Cache Name），它会作为 Key 的 **前缀**，例如 `users::`
     - 形如：`key = "#id"`：这是 Spring 表达式语言 (SpEL)
       - `#id` 表示“使用方法参数中名为 id 的值”作为 Key 的 **后缀**
     - 最终生成的 Redis Key 可能是：`users::1`
  2. **检查缓存 (GET)**：
     - Spring 自动向 Redis 发送 `GET "users::1"` 命令
  3. **判断结果**：
     - **情况 A：缓存命中 (Cache Hit)**
       - Redis 返回了 `users::1` 对应的数据（例如一个 JSON 字符串）
       - Spring 自动将其反序列化为 `User` 对象
       - **方法体（`getUserById` 内部的代码）根本不会被执行**
       - Spring 直接将反序列化后的 `User` 对象返回给调用方
     - **情况 B：缓存未命中 (Cache Miss)**
       - Redis 返回 `nil` (空)
       - Spring **会执行方法体**（即 `System.out.println(...)` 和 `userRepository.findById(...)`）
       - 方法体从数据库查询到了 `User` 对象
       - 在方法返回之前，Spring 会自动将这个 `User` 对象序列化（例如转为 JSON），并执行 `SET "users::1" "..."` 命令，将其 **自动存入 Redis 缓存**
       - 最后，将 `User` 对象返回给调用方



### 4.2.3 关键属性说明

- **`value`** (或 **`cacheNames`**)
  - **必需**。指定缓存的名称，用于隔离不同类型的缓存
  - 例如：`@Cacheable("users")`，`@Cacheable("products")`



- **`key`**
  - **必需**。用于指定 Key 的生成策略，支持 SpEL
  - `key = "#id"`：使用名为 `id` 的参数
  - `key = "#user.id"`：使用 `user` 对象的 `id` 属性
  - `key = "#p0"`：使用第一个参数（`p0` 代表第0个参数）
  - 如果不指定，Spring 会使用所有参数的 `hashCode()` 组合，可读性差，**不推荐**



- **`sync`**
  - **重要**。用于**防止缓存击穿**
  - `@Cacheable(value = "users", key = "#id", sync = true)`
  - 当 `sync = true` 时，如果一个热点 Key 缓存失效（例如 `users::1` 过期），此时有 1000 个并发请求同时调用 `getUserById(1L)`：
    - **只有一个**线程会真正去执行方法体（查数据库）
    - 其他 999 个线程会被**阻塞等待**，直到第一个线程完成数据库查询、将数据放入缓存
    - 随后，这 999 个线程会直接从缓存中获取数据并返回
  - **原理**：它在 JVM 内部对同一个 Key 的加载操作加了锁



- **`unless`**
  - 用于否决缓存。在方法执行**之后**判断，如果表达式为 `true`，则**不缓存**结果
  - `unless = "#result == null"`：如果方法返回 `null`，则不缓存这个 `null` 结果（这可以防止缓存穿透，但 `@Cacheable` 默认会缓存 `null`）



- **`condition`**
  - 用于否决缓存
    - 在方法执行**之前**判断，如果表达式为 `true`，**才**会走缓存逻辑；
    - 如果为 `false`，则**总是执行**方法体（不读也不写缓存）
  - `condition = "#id > 100"`：只有当 ID 大于 100 时，才启用缓存



## 4.3 `@CacheEvict`（删）

- `@Cacheable` 解决了“读”的问题，而 `@CacheEvict` 则解决了“写”的问题
  - 当数据库中的数据发生 **变更**（更新或删除）时，缓存中的数据必须被 **移除**（Evict），否则用户就会读到过时的数据（脏数据）

- `@CacheEvict` 负责自动执行“旁路缓存” (Cache-Aside Pattern) 的 **写逻辑**，即： **先更新数据库，再删除缓存**



- **`@CacheEvict` 行为简述**⭐⭐

  - **行为依据**：`value` + `key`

  - **行为**：
    1. **执行方法体**（默认）
    2. 方法成功后，根据 `value` + `key` 去 **删除** 对应的缓存
    3. （它 **不关心** 方法返回值是什么）



### 4.3.1 基本用法

- `@CacheEvict` 通常被放置在 **“写数据”** 的方法上（如 `updateUser` 或 `deleteUser`）

#### 1. 用于更新操作

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository; // 模拟数据库操作

    // ... (省略之前的 getUserById 方法) ...

    @Override
    @CacheEvict(value = "users", key = "#user.id")
    public void updateUser(User user) {
        // 1. 先更新数据库
        System.out.println("=== 正在更新数据库: " + user.getId() + " ===");
        userRepository.save(user);
        
        // 2. 方法执行成功后，Spring 会自动删除缓存
    }
}
```



#### 2. 用于删除操作

```java
    @Override
    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        // 1. 先删除数据库
        System.out.println("=== 正在删除数据库: " + id + " ===");
        userRepository.deleteById(id);
        
        // 2. 方法执行成功后，Spring 会自动删除缓存
    }
```



### 4.3.2 `@CacheEvict` 的工作流程

- 当一个被 `@CacheEvict` 标记的方法（如 `updateUser(user)`）被调用时：
  1. **执行方法体**：
     - Spring **首先会执行方法体**，即 `userRepository.save(user)`（更新数据库）
     - **注意**：
       - 这是 `@CacheEvict` 的默认行为 (`beforeInvocation = false`)
       - 如果数据库操作失败（例如抛出异常），方法会在此处终止，**缓存删除操作也不会被执行**。这保证了操作的原子性
     
  2. **生成 Key**：
     - 方法体成功执行完毕后，Spring 会根据注解中的 `value` 和 `key` 属性来生成要删除的 Key
     
       - `value = "users"`，`key = "#user.id"`
     
       > 形如：`key = "#id"`：这是 Spring 表达式语言 (SpEL)
       >
       > - `#id` 表示“使用方法参数中名为 id 的值”作为 Key 的 **后缀**
       > - `#result`引用方法调用的结果
     
     - 假设 `user.getId()` 为 `1L`，则生成的 Redis Key 为：`users::1`
     
  3. **删除缓存 (DEL)**：
     - Spring 自动向 Redis 发送 `DEL "users::1"` 命令
     - 无论这个 Key 是否存在，Redis 都会执行删除



### 4.3.3 为什么是“删除缓存”，而不是“更新缓存”？

- 这是一个经典的缓存一致性策略问题：

  - **更新缓存**：先更新数据库，再更新缓存
    - **缺点1（性能）**：数据库更新后，缓存可能在短时间内不再被访问，此时“更新缓存”是一个无效操作，浪费性能
    - **缺点2（并发问题）**：如果两个线程同时更新，线程A更新DB -> 线程B更新DB -> 线程B更新缓存 -> 线程A更新缓存。这会导致缓存中是A的数据（旧数据），而数据库中是B的数据（新数据），造成数据不一致

  - **删除缓存**（`@CacheEvict` 采用的策略）：
    - **策略**：先更新数据库，再删除缓存
    - **优点**：操作简单、高效。当下次“读”请求（调用 `@Cacheable` 方法）进来时，发现缓存未命中，会自动去数据库加载最新数据，并回填到缓存中。这是一种“懒加载”思想

  **结论**：在“旁路缓存”模式中，**“写操作”推荐删除缓存**，而不是更新缓存



### 4.3.4 关键属性说明

- **`value`** (或 **`cacheNames`**)
  - **必需**。必须与你要清除的缓存 `@Cacheable` 的 `value` **完全一致**



- **`key`**
  - **必需**。必须与 `@Cacheable` 的 `key` 生成策略 **完全一致**
  - 例如，`getUserById(Long id)` 使用 `key = "#id"`，那么 `deleteUser(Long id)` 也必须使用 `key = "#id"`



- **`allEntries`**⭐
  - **危险但有用**。是否清除该 `value`（命名空间）下的 **所有** 缓存
  - `@CacheEvict(value = "users", allEntries = true)`
  - **作用**：执行此方法后，所有以 `users::` 开头的 Key 都会被（批量）删除
  - **场景**：适用于“清空用户列表缓存”等批量操作，但要小心使用，避免误删



- **`beforeInvocation`**
  - **重要**。是否在方法执行 **之前** 清除缓存
  
  - `beforeInvocation = false` (默认)：方法执行 **之后** 清除。如果方法执行失败（抛异常），缓存不会被清除**（推荐）**
  
  - `beforeInvocation = true`：方法执行 **之前** 清除
    
    - **风险**：
    
      如果方法执行前清除了缓存，但方法体（如更新数据库）执行失败了，那么缓存被清了，数据库却是旧数据
    
      此时“读”请求进来，会把旧数据重新加载到缓存中，导致不一致**（不推荐）**
    
      > 不推荐的核心：它允许一个失败的写操作（更新）对缓存产生了“删除”的副作用，这破坏了原子性



## 4.4 `@CachePut` (强制更新缓存)

- `@CachePut` 是三个注解中用法最特殊的一个

  它通常也标记在 **“写数据”** 的方法上（例如 `updateUser`），但它的目的 **不是“删除”缓存** ，而是 **“更新”缓存**

  - 它的核心机制是：**方法体总会被执行**，然后将方法的 **返回值** 强制放入缓存



- **`@CachePut` 行为简述**⭐⭐

  - **行为依据**：`value` + `key` + **方法返回值**

  - **行为**：
    1. **总是执行方法体**（它不查缓存）
    2. 方法成功后，拿 **方法的返回值**
    3. 根据 `value` + `key` 去 **覆盖** 对应的缓存



### 4.4.1 基本用法

- 假设我们有一个 `updateUser` 方法，并且我们希望在更新数据库后，**立刻** 将 **最新的 `User` 对象**（可能包含数据库生成的默认值或时间戳）更新到缓存中，而不是删除它

  ```JAVA
  @Service
  public class UserServiceImpl implements UserService {
  
      @Autowired
      private UserRepository userRepository; // 模拟数据库操作
  
      // ... (省略 getUserById 和 deleteUser 方法) ...
  
      /**
       * 注意：此方法的返回值必须是 User 对象，
       * 因为 Spring 会将这个返回值放入缓存。
       */
      @Override
      @CachePut(value = "users", key = "#user.id")
      public User updateUserAndRefreshCache(User user) {
          // 1. 方法体总会执行，更新数据库
          System.out.println("=== 正在更新数据库 (CachePut): " + user.getId() + " ===");
          User updatedUser = userRepository.save(user);
          
          // 2. 方法执行成功后，Spring 会自动将 updatedUser 放入缓存
          return updatedUser;
      }
  }
  ```



### 4.4.2 `@CachePut` 的工作流程

1. **方法体总被执行**：
   - `updateUserAndRefreshCache` 方法被调用
   - Spring **不会** 在方法执行前检查缓存
   - 方法体（`userRepository.save(user)`）**总是会被执行**
   
2. **获取返回值**：
   - 方法执行完毕，返回 `updatedUser` 对象
   
3. **生成 Key**：
   - Spring 根据 `value = "users"` 和 `key = "#user.id"` 生成 Key（例如 `users::1`）
   
     > 形如：`key = "#id"`：这是 Spring 表达式语言 (SpEL)
     >
     > - `#id` 表示“使用方法参数中名为 id 的值”作为 Key 的 **后缀**
     > - `#result`引用方法调用的结果
   
4. **更新缓存 (SET)**：
   - Spring 将 `updatedUser` 对象序列化，然后自动向 Redis 发送 `SET "users::1" "..."` 命令，**强制覆盖** 该 Key 对应的缓存数据



### 4.4.3 `@CachePut` vs `@CacheEvict`

| 特性         | `@CachePut` (更新缓存)     | `@CacheEvict` (删除缓存)           |
| ------------ | -------------------------- | ---------------------------------- |
| **策略**     | 先更新DB，再 **更新** 缓存 | 先更新DB，再 **删除** 缓存         |
| **方法体**   | 总是执行                   | 总是执行                           |
| **缓存操作** | `SET` (覆盖)               | `DEL` (删除)                       |
| **返回值**   | **必须** 返回要缓存的对象  | 返回值类型 **随意** (e.g., `void`) |



### 4.4.4 `@CachePut` 的使用场景

- `@CachePut` 在标准的“旁路缓存”模式中 **不常用**，因为 `CacheEvict`（删除缓存）通常是更简单、更健壮的策略

- 它的主要使用场景是：
  1. **数据强一致性要求**：希望在更新数据库后，缓存 **立即** 被更新为最新数据，而不是等待下一次“读”请求来回填
  2. **缓存预热**：方法体不是从数据库查，而是通过复杂计算得到一个结果，希望 **总是执行计算**，并 **总是把最新结果** 放入缓存

- **注意**：

  - 使用 `@CachePut` 必须确保方法的 **返回值** 就是你要存入缓存的那个对象

    如果你把方法写成 `public void updateUser(...)`，`@CachePut` 会尝试把 `null` 存入缓存（取决于序列化器配置），导致缓存数据被破坏



## 4.5 @CacheConfig (类级别公共配置)

`@CacheConfig` 是一个 **类级别** 的辅助注解，它本身不触发任何缓存操作，其主要目的是 **提取公共的缓存配置** ，以减少重复代码，提高可维护性。

- **使用场景**： 

  - 在一个 Service 实现类中（例如 `UserServiceImpl`），通常所有的方法（`@Cacheable`, `@CachePut`, `@CacheEvict`）操作的都是 **同一个缓存空间**（例如 `value = "users"`）

    这会导致每个注解上都必须重复声明 `value = "users"`，`@CacheConfig` 就是为了解决这个问题



### 4.5.1 使用 @CacheConfig 前后对比

#### 1. 未使用 @CacheConfig (配置冗余)

在 `UserServiceImpl` 中，`value = "users"` 在每个注解中都出现了

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    @Cacheable(value = "users", key = "#id")
    @Override
    public User getUserById(Long id) {
        System.out.println("=== 正在从数据库查询用户: " + id + " ===");
        return userRepository.findById(id).orElse(null);
    }

    @CacheEvict(value = "users", key = "#id")
    @Override
    public void deleteUser(Long id) {
        System.out.println("=== F 正在删除数据库: " + id + " ===");
        userRepository.deleteById(id);
    }
    
    @CachePut(value = "users", key = "#user.id")
    @Override
    public User updateUserAndRefreshCache(User user) {
        System.out.println("=== 
        正在更新数据库 (CachePut): " + user.getId() + " ===");
        return userRepository.save(user);
    }
}
```



#### 2. 使用 @CacheConfig (配置简洁)

我们将公共的 `cacheNames`（`value` 属性的别名）提取到类声明上

```java
@Service
@CacheConfig(cacheNames = "users") // <-- 提取公共的缓存名称
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    @Cacheable(key = "#id") // <-- 无需再写 value
    @Override
    public User getUserById(Long id) {
        System.out.println("=== 
        正在从数据库查询用户: " + id + " ===");
        return userRepository.findById(id).orElse(null);
    }

    @CacheEvict(key = "#id") // <-- 无需再写 value
    @Override
    public void deleteUser(Long id) {
        System.out.println("=== 
        正在删除数据库: " + id + " ===");
        userRepository.deleteById(id);
    }
    
    @CachePut(key = "#user.id") // <-- 无需再写 value
    @Override
    public User updateUserAndRefreshCache(User user) {
        System.out.println("=== 
        正在更新数据库 (CachePut): " + user.getId() + " ===");
        return userRepository.save(user);
    }
}
```



### 4.5.2 关键属性说明

`@CacheConfig` 注解允许您配置多个公共属性，最常用的就是 `cacheNames`

- **`cacheNames`** (或 **`value`**)
  - 指定该类中所有缓存操作的 **默认** 缓存空间名称
  - **注意**：`@Cacheable` 和 `@CacheEvict` 上的 `value` 属性是 `cacheNames` 属性的别名，因此 `@CacheConfig(value = "users")` 和 `@CacheConfig(cacheNames = "users")` 是等效的
- **`keyGenerator`**
  - 指定一个**默认**的 `KeyGenerator` Bean 的名称
- **`cacheManager`**
  - 指定一个**默认**的 `CacheManager` Bean 的名称（在拥有多个 `CacheManager` 时有用）



### 4.5.3 配置覆盖规则

`@CacheConfig` 提供的只是**默认值**

如果在方法级别的注解（如 `@Cacheable`）上**也**指定了相同的属性（例如 `value`），那么**方法级别的配置会覆盖类级别的配置**

```java
@Service
@CacheConfig(cacheNames = "users") // 默认使用 "users" 缓存
public class UserServiceImpl implements UserService {

    @Cacheable(key = "#id") // <-- 继承 @CacheConfig，使用 "users" 缓存
    public User getUserById(Long id) {
        // ...
    }

    @Cacheable(value = "admins", key = "#id") // <-- 覆盖 @CacheConfig
    public User getAdminById(Long id) {
        // 这个方法会使用 "admins" 缓存，而不是 "users" 缓存
        // ...
    }
}
```



## 4.6 缓存配置 (RedisCacheManager)

- `@EnableCaching` 开启缓存功能后，Spring Boot 会自动配置一个 `RedisCacheManager` (缓存管理器) 的 Bean

  - 但是，这个默认的 Bean 存在两个主要问题：

    1. **默认使用 JDK 序列化**：它会使用 `JdkSerializationRedisSerializer`，导致存入 Redis 的数据可读性极差
    2. **默认 TTL 为 -1 (永久有效)**：缓存永不过期，这在生产环境中是灾难性的，会导致内存溢出

    因此，我们 **必须** 自定义一个 `RedisCacheManager` Bean，来覆盖 Spring Boot 的默认配置



### A 自定义 RedisCacheManager

- 我们回到 `RedisConfig`配置类，在里面添加一个新的 `@Bean`

```java
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;
import com.fasterxml.jackson.databind.ObjectMapper; 
import com.fasterxml.jackson.annotation.JsonTypeInfo; 
import com.fasterxml.jackson.annotation.PropertyAccessor; 
import com.fasterxml.jackson.annotation.JsonAutoDetect; 
import com.fasterxml.jackson.databind.jsontype.impl.LaissezFaireSubTypeValidator; 
import java.time.Duration;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableCaching // <-- @EnableCaching 也可以放在这里
public class RedisConfig {

    // ... (这里是前文配置的 RedisTemplate Bean) ...
    // @Bean
    // public RedisTemplate<String, Object> redisTemplate(LettuceConnectionFactory connectionFactory) { ... }

    /**
     * 自定义缓存管理器 RedisCacheManager
     */
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        
        // 1. 配置序列化器
        // Key 使用 String 序列化
        StringRedisSerializer stringSerializer = new StringRedisSerializer();
        // Value 使用 JSON 序列化
        Jackson2JsonRedisSerializer<Object> jsonSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        
        // (配置 ObjectMapper 以支持反序列化复杂的泛型对象)
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, 
                                 ObjectMapper.DefaultTyping.NON_FINAL, 
                                 JsonTypeInfo.As.PROPERTY);
        jsonSerializer.setObjectMapper(om);

        // 2. 配置默认的缓存行为
        RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig()
            // 2.1 设置 Key 的序列化方式
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(stringSerializer))
            // 2.2 设置 Value 的序列化方式
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jsonSerializer))
            // 2.3 设置默认过期时间 (例如：1小时)
            .entryTtl(Duration.ofHours(1))
            // 2.4 (可选) 禁止缓存 null 值。默认是允许的。
            // .disableCachingNullValues() 
            // 2.5 (可选) Key 的前缀，默认是 cacheName::
            .computePrefixWith(cacheName -> cacheName + ":");


        // 3. (可选) 配置特定 CacheName 的过期时间
        // 比如 "users" 缓存 30 分钟, "products" 缓存 2 小时
        Map<String, RedisCacheConfiguration> specificCacheConfigs = new HashMap<>();
        specificCacheConfigs.put("users", 
            defaultCacheConfig.entryTtl(Duration.ofMinutes(30)));
        specificCacheConfigs.put("products", 
            defaultCacheConfig.entryTtl(Duration.ofHours(2)));

        // 4. 构建 CacheManager
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(defaultCacheConfig) // 设置默认配置
            .withInitialCacheConfigurations(specificCacheConfigs) // (可选) 设置特定配置
            .build();
    }
}
```

- **总结**：配置 `RedisCacheManager` 主要就是为了做两件事：
  1. 把 Value 的序列化方式从 `JDK` 改成 `JSON`
  2. 设置一个合理的**默认 TTL** (过期时间)



## 4.7 缓存实战：应对“三兄弟”问题

- 在生产环境中，高并发下的缓存使用必须考虑以下三个经典问题：

### 4.7.1 缓存穿透

>Cache Penetration

- **现象**：攻击者或恶意请求，**大量查询一个数据库中根本不存在的数据** (例如 `id = -1`)
- **流程**：
  1. 请求 `id = -1`
  2. `@Cacheable` 去 Redis 查 `users::-1`，未命中 (Cache Miss)
  3. 执行方法体，去数据库查 `id = -1`，未命中 (DB Miss)
  4. 方法返回 `null`
  5. `@Cacheable` (默认) **不会缓存**这个 `null` 结果
- **后果**：所有对不存在数据的查询，都会绕过 Redis，直接打到数据库，导致数据库压力剧增甚至崩溃
- **SDR 对策**：
  1. **缓存 `null` 值** (推荐)：
     - Spring `RedisCacheManager` **默认是允许缓存 `null` 值**的
       - 当 `getUserById(-1)` 返回 `null` 时，Spring 会把 `null` 存入 Redis (`SET "users::-1" "null"`)，并设置一个**较短的过期时间** (例如 5 分钟，这个 TTL 需要在 `specificCacheConfigs` 中单独为 `null` 值配置，或者接受默认的1小时)
       - 在 5 分钟内，后续对 `id = -1` 的请求都会命中缓存（`null`），直接返回，从而保护了数据库
  2. **`unless` 属性**：
     - `@Cacheable(value="users", key="#id", unless="#result == null")`
     - 这反而**会导致缓存穿透**，因为它明确告诉 Spring：“如果结果是 `null`，不要缓存”（**不推荐**，除非业务明确不允许缓存 `null`）



### 4.7.2 缓存击穿

>Cache Breakdown

- **现象**：某一个**热点 Key** (例如一个爆款商品 `products::1001`) 承载着极高的并发访问

- **流程**：

  1. `products::1001` 这个 Key 突然过期了
  2. 在它过期的**瞬间**，假如有 5000 个并发请求同时访问这个 Key
  3. 这 5000 个请求全部“未命中”(Cache Miss)
  4. (默认情况下) 这 5000 个请求**全部**去执行方法体，**同时**打到数据库

- **后果**：数据库在瞬时压力剧增，可能导致卡顿或崩溃

- **SDR 对策**：使用 `@Cacheable` 的 `sync = true` 属性

  ```JAVA
  @Cacheable(value = "products", key = "#id", sync = true)
  public Product getProductById(Long id) {
      // ... 查询数据库 ...
  }
  ```

  - **原理**：
    1. 当 5000 个请求同时未命中时，`sync = true` 会起作用
    2. Spring 会在 JVM 内部对这个 Key (`products::1001`) 的加载操作**加锁**
    3. **只有一个**线程被允许去执行方法体（查询数据库）
    4. 其他 4999 个线程被**阻塞等待**
    5. 第一个线程查完数据库，将数据放入缓存后，释放锁
    6. 其他 4999 个线程被唤醒，此时它们会**再次检查缓存**（而不是去查数据库），发现缓存已命中，直接返回数据
  - **代价**：牺牲了其他线程的等待时间，换取了数据库的安全



### 4.7.3 缓存雪崩

- **现象**：**大量**的 Key 在**同一时刻**集体失效 (过期)

- **流程**：

  1. 例如，你在0点0分通过定时任务，预热了 100 万个 Key，并都设置了 1 小时过期
  2. 到了1点0分，这 100 万个 Key **同时过期**
  3. 在这之后的一小段时间内，大量的请求（无论是不是热点 Key）都会“未命中”，全部涌向数据库

- **后果**：数据库发生瞬时雪崩，无法处理海量请求，导致整个系统瘫痪

- **SDR 对策**：**设置随机化的过期时间**

  - **核心思想**：避免所有 Key 在同一时间点过期，把过期时间“打散”
  - **实现**：在 `RedisCacheManager` 配置中，为 `entryTtl` 设置一个随机范围

  ```JAVA
  RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig()
      // ... 其他配置 ...
      // 设置一个基础 TTL + 一个随机偏移量
      // 例如：基础 60 分钟 + 随机 (0-10) 分钟
      .entryTtl(Duration.ofMinutes(60 + new Random().nextInt(10)));
  
  // 或者在特定配置中
  specificCacheConfigs.put("users", 
      defaultCacheConfig.entryTtl(Duration.ofMinutes(30 + new Random().nextInt(5))));
  ```

  - 这样，`users` 缓存的过期时间就会在 30-35 分钟之间随机分布，避免了同时失效



## 4.8 Spring Cache 常见的问题

在使用 Spring Cache 时，有一些常见的“陷阱”是由 AOP 代理机制或与其他注解（如 `@Transactional`）的交互引起的。

### a. 陷阱一：注解的“自调用失效”问题

- **问题描述**： 在一个 Service 类中，一个 **没有** 缓存注解的方法 `A`，去调用同一个类中 **有** 缓存注解的方法 `B`，会导致方法 `B` 的缓存 **不生效**

  ```java
  @Service
  public class UserServiceImpl implements UserService {
  
      @Autowired
      private UserRepository userRepository;
  
      @Cacheable(value = "users", key = "#id")
      @Override
      public User getUserById(Long id) {
          System.out.println("=== 正在从数据库查询用户: " + id + " ===");
          return userRepository.findById(id).orElse(null);
      }
  
      /**
       * 外部调用这个方法
       */
      public User getAndProcessUser(Long id) {
          // ... 假设有一些前置处理 ...
  
          // ！！！问题所在：内部调用 this.getUserById(id) ！！！
          // ！！！@Cacheable 会失效，导致每次都查数据库 ！！！
          User user = this.getUserById(id); 
  
          // ... (其他处理) ...
          return user;
      }
  }
  ```

- **原因分析**： Spring Cache（以及 `@Transactional` 等）是基于 Spring AOP（面向切面编程）实现的，它通过**动态代理**来工作

  1. 当外部调用 `userService.getAndProcessUser(1L)` 时，它访问的是 Spring 创建的 `UserService` **代理对象**

  2. `getAndProcessUser` 方法本身没有注解，所以代理对象直接执行了它的方法体

  3. 当方法体内部执行 `this.getUserById(id)` 时，它使用的是 `this` 关键字，`this` 指向的是**原始的 `UserServiceImpl` 对象**，而不是代理对象

  4. 调用原始对象的方法会**完全绕过** Spring AOP 代理，导致缓存切面没有机会介入，`@Cacheable` 注解自然就失效了

     

- **解决方案**（任选其一）：

  1. **（推荐）重构代码**：将需要缓存的方法（如 `getUserById`）移到另一个单独的 Service（例如 `UserCacheService`）中，然后在 `UserServiceImpl` 中注入并调用 `userCacheService.getUserById(id)`

  2. **（不常用）注入自己**：在 `UserServiceImpl` 中注入 `UserService` 代理对象自身（Spring 会自动处理依赖注入，确保注入的是代理）

     ```java
     @Service
     public class UserServiceImpl implements UserService {
         @Autowired
         private UserService selfProxy; // 注入代理对象
     
         public User getAndProcessUser(Long id) {
             // 使用代理对象调用，AOP 会生效
             User user = selfProxy.getUserById(id); 
             return user;
         }
         // ... other methods ...
     }
     ```

  3. **（不常用）使用 `AopContext`**：通过 `AopContext.currentProxy()` 获取当前代理对象，但需要额外配置（`@EnableAspectJAutoProxy(exposeProxy = true)`）



### b. 陷阱二：@CacheEvict 与 @Transactional 的时序问题

- **问题描述**： 

  - 当 `@CacheEvict`（默认 `beforeInvocation = false`）和 `@Transactional` 标记在同一个方法上时（例如 `updateUser`），由于二者都依赖 AOP，会产生一个危险的执行时序问题

- **危险流程**：

  -  `@CacheEvict` 的默认行为是在 **方法执行完毕后** 删除缓存。而 `@Transactional` 是在 **方法执行完毕后** 才 **提交事务**

    假设 `updateUser` 方法同时使用了这两个注解：

    1. `updateUser` 方法开始执行（事务开启）	
    2. 方法体执行数据库 `UPDATE` 操作（此时事务 **尚未提交**）
    3. `updateUser` 方法体执行完毕，返回
    4. AOP 拦截器介入，首先执行 `@CacheEvict` 的逻辑（因为它在 `@Transactional` 之前或顺序相近）
    5. `@CacheEvict` 成功执行 `DEL "users::1"`，**缓存被删除**
    6. 接着，AOP 拦截器执行 `@Transactional` 的提交逻辑
    7. **！！！此时，如果数据库事务提交失败**（例如因为唯一键冲突、网络故障或数据库锁超时）
    8. 事务 **回滚**，数据库中的数据 **恢复到了旧状态**

    

- **最终结果**： **缓存被删了，但数据库回滚了旧数据**

  当下一个 `getUserById(1L)` 请求进来时：

  1. `@Cacheable` 检查缓存，发现 `users::1` 不存在（缓存未命中）
  2. 执行方法体，去数据库查询
  3. 查到了被回滚的 **旧数据**
  4. 将这个 **旧数据** 重新写入了 `users::1` 缓存

  这导致了严重的 **数据不一致** 问题

- **解决方案**： 核心思想是：**必须在事务（Transaction）成功提交（Commit）之后，才去执行缓存删除（Evict）操作**

  1. **（推荐）使用事务同步管理器**： 
     - 通过 `TransactionSynchronizationManager.registerSynchronization` 注册一个回调，在事务 `afterCommit()` 事件中再执行缓存删除
  2. **（推荐）使用 Spring Event**： 
     - 在 `updateUser` 方法中（事务内）发布一个“用户更新事件”
       然后创建一个 **事务性事件监听器**（使用 `@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)`），
       这个监听器只会在事务成功提交后被触发，由它来负责执行缓存删除





# SDR 进阶主题

## 5.1 事务与管道

- 当你需要一次执行多个 Redis 命令时，SDR 提供了 `execute` 和 `executePipelined` 方法，它们允许你获取一个底层的 Redis 连接，并在该连接上执行多个操作

- 这两个操作都依赖于一个回调接口：`SessionCallback<T>`

### 5.1.1 事务 (Transaction)

- Redis 的事务是通过 `MULTI`（开启事务）、`EXEC`（执行事务）、`DISCARD`（取消事务）和 `WATCH`（乐观锁）来实现的
  - SDR 将这个过程封装在了 `redisTemplate.execute(SessionCallback<T> callback)` 方法中

- **工作机制**： 当你使用 `execute(SessionCallback<T> callback)` 时，SDR 会自动帮你：
  1. 获取一个连接
  2. 调用 `MULTI` 命令（标记事务开始）
  3. 执行你在 `callback` 中定义的所有 Redis 操作（例如 `opsForValue().set()`）
     - **注意：** 此时命令只是被**放入队列**，并**不会立即执行**
  4. 调用 `EXEC` 命令（原子性地执行队列中的所有命令）
  5. 释放连接

- **代码示例：**

  - 假设有一个场景：用户A给用户B转账100元

    1. 用户A的余额 (Key: `balance:A`) 减 100
    2. 用户B的余额 (Key: `balance:B`) 加 100。 这两个操作必须是原子的

    ```java
    import org.springframework.data.redis.core.SessionCallback;
    import org.springframework.data.redis.core.RedisOperations;
    import org.springframework.dao.DataAccessException;
    import java.util.List;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public boolean transferMoney(String fromUser, String toUser, double amount) {
        
        // 使用 SessionCallback 来执行事务
        List<Object> results = redisTemplate.execute(new SessionCallback<List<Object>>() {
            
            @Override
            public List<Object> execute(RedisOperations operations) throws DataAccessException {
                // 1. 开启事务 (这一步由 SDR 自动完成，即调用 MULTI)
                
                // 2. 将命令加入队列
                operations.opsForValue().increment("balance:" + fromUser, -amount);
                operations.opsForValue().increment("balance:" + toUser, amount);
                
                // 3. 执行事务 (SDR 会自动调用 EXEC)
                // execute 方法会返回一个 List，包含队列中每个命令的执行结果
                return operations.exec(); 
            }
        });
    
        // Redis 的 EXEC 要么返回 null (如果事务被 DISCARD 或 WATCH 失败)
        // 要么返回一个包含所有命令结果的 List
        if (results == null) {
            System.out.println("转账失败，事务被回滚 (可能是 WATCH 失败)");
            return false;
        }
    
        System.out.println("事务执行结果: " + results); // [800, 1200] (假设A原有900, B原有1100)
        return true;
    }
    
    // 使用 Lambda 表达式 (更简洁)
    public boolean transferMoneyLambda(String fromUser, String toUser, double amount) {
        List<Object> results = redisTemplate.execute(operations -> {
            // operations.multi(); // 如果需要显式开启，但SDR在execute中已默认开启
            operations.opsForValue().increment("balance:" + fromUser, -amount);
            operations.opsForValue().increment("balance:" + toUser, amount);
            return operations.exec(); 
        });
        
        return results != null;
    }
    ```



- **注意**：Redis 事务**不支持回滚**

  - 如果在 `EXEC` 时，队列中的某个命令执行失败（例如对一个 String 使用了 `HSET`），其他命令**仍会继续执行**

    它只保证原子性（不会被其他客户端插队）



### 5.1.2 管道 (Pipeline)

- **问题**：

  - 如果你需要连续执行 100 次 `SET` 命令，

    SDR 默认会： `获取连接 -> SET -> 释放连接` (第1次) `获取连接 -> SET -> 释放连接` (第2次) ... `获取连接 -> SET -> 释放连接` (第100次) 这会产生 100 次网络往返时间 (RTT)，性能极差

- **管道 (Pipeline)**：

  - 允许客户端一次性将 100 个命令 **全部发送** 给 Redis 服务器，而不需要等待每个命令的回复

    Redis 服务器执行完这 100 个命令后，再**一次性**将 100 个结果打包返回给客户端

  - 这极大地减少了网络 RTT，是**批量操作**的首选方案

  - SDR 封装：`redisTemplate.executePipelined(SessionCallback<T> callback)`

- **代码示例：** 批量 `MSET` 1000 个 Key

  ```java
  public void batchSetUsingPipeline() {
      long startTime = System.currentTimeMillis();
      
      // executePipelined 会自动开启 Pipeline
      // 注意：Callback 的返回值类型通常是 Void，因为结果会在外层的 List<Object> 中返回
      List<Object> results = redisTemplate.executePipelined(new SessionCallback<Void>() {
          
          @Override
          public Void execute(RedisOperations operations) throws DataAccessException {
              for (int i = 0; i < 1000; i++) {
                  String key = "pipeline_key:" + i;
                  String value = "pipeline_value:" + i;
                  
                  // 只是将命令发送到本地缓冲区，不会立即发送
                  operations.opsForValue().set(key, value); 
              }
              // execute 方法返回 null，因为结果由 executePipelined 统一处理
              return null; 
          }
      });
      
      long endTime = System.currentTimeMillis();
      System.out.println("管道执行耗时: " + (endTime - startTime) + " ms");
      
      // results 列表包含了 Pipeline 中所有命令的执行结果
      // [true, true, true, ..., true] (1000 个 true，代表 SET 成功)
      System.out.println("管道执行结果数量: " + results.size()); 
  }
  ```



### 5.1.3 事务 vs 管道

| 特性         | 事务 (`execute`)                 | 管道 (`executePipelined`)                         |
| ------------ | -------------------------------- | ------------------------------------------------- |
| **目的**     | **原子性**。保证一组命令不被打断 | **高性能**。减少网络 RTT                          |
| **封装**     | `MULTI` ... `EXEC`               | 一次性发送所有命令                                |
| **原子性**   | **是**                           | **否** (管道中的命令执行时，可能被其他客户端插队) |
| **返回时机** | `exec()` 时才返回结果            | 所有命令执行完后，一次性返回所有结果              |



- **实践**：

  - 如果你的操作**必须保证原子性**（如转账），使用 `execute` (事务)

  - 如果你的操作只是**大批量的数据写入或读取**（如缓存预热），并且对原子性没要求，**优先使用 `executePipelined` (管道)**



## 5.2 执行 Lua 脚本

- 当 Redis 提供的标准命令无法满足你的需求时（特别是你需要**原子性**地执行多个命令的组合逻辑时），就需要使用 Lua 脚本
  - Redis 会保证 Lua 脚本在执行期间**不会被其他命令打断**，它提供了比事务（`MULTI`/`EXEC`）更强大、更灵活的原子性保障

-  Lua 脚本是解决 Redis **并发原子性**问题的终极方案
  - SDR 提供了 `redisTemplate.execute(RedisScript, ...)` 接口，让 Java 代码可以方便、高效地调用这些脚本



### 5.2.1 为什么需要 Lua？(原子性)

- **场景**：实现一个“比较并删除”的原子操作

  - **需求**：检查 Key `my_lock` 的值是否等于 "thread_id_1"。如果是，就删除它；如果不是，就不做任何操作

  - **非原子实现 (错误)**：

    ```java
    // 1. GET
    String value = (String) redisTemplate.opsForValue().get("my_lock"); 
    if ("thread_id_1".equals(value)) {
        // 2. DEL
        // ！！！ 非原子 ！！！
        // 在 1 和 2 之间，my_lock 可能被其他线程释放或抢占
        redisTemplate.delete("my_lock"); 
    }
    ```

  - **原子实现 (正确)**：必须将 `GET` 和 `DEL` 组合在一个 Lua 脚本中执行



### 5.2.2 SDR 如何执行 Lua 脚本

- SDR 通过 `redisTemplate.execute(RedisScript<T> script, List<K> keys, Object... args)` 方法来执行 Lua 脚本

- 你需要两个核心组件：
  1. **`RedisScript<T>` 对象**：用于封装你的 Lua 脚本字符串，并指定返回类型
  2. **参数**：
     - `List<K> keys`：传递给 Lua 脚本的 **Key 列表** (在 Lua 中通过 `KEYS[1]`, `KEYS[2]` ... 访问)
     - `Object... args`：传递给 Lua 脚本的 **Value 参数列表** (在 Lua 中通过 `ARGV[1]`, `ARGV[2]` ... 访问)



### 5.2.3 代码示例：实现“比较并删除”

**1. 定义 Lua 脚本**

```lua
-- 这是一个 Lua 脚本
-- 检查 KEYS[1] (第一个 Key) 的值是否等于 ARGV[1] (第一个参数)

if redis.call('GET', KEYS[1]) == ARGV[1] then
    -- 如果相等，就执行 DEL 命令删除它
    return redis.call('DEL', KEYS[1])
else
    -- 如果不相等，返回 0
    return 0
end
```

- `redis.call(...)`：在 Lua 中调用 Redis 命令
- `return 1` 代表 `DEL` 成功（删除了1个Key）
- `return 0` 代表 `DEL` 未执行



**2. 在 Spring 中定义 `RedisScript` Bean**

- 通常我们会把 Lua 脚本放在 `resources` 目录下的 `.lua` 文件中（例如 `resources/lua/compare_and_delete.lua`），但为了简单，这里我们使用内联字符串

  ```java
  import org.springframework.data.redis.core.script.RedisScript;
  import org.springframework.data.redis.core.script.DefaultRedisScript;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  // 假设在 RedisConfig.java 中
  @Configuration
  public class RedisConfig {
      
      // ... (之前的 RedisTemplate, CacheManager Bean) ...
  
      @Bean
      public RedisScript<Long> compareAndDeleteScript() {
          String scriptText = 
              "if redis.call('GET', KEYS[1]) == ARGV[1] then " +
              "    return redis.call('DEL', KEYS[1]) " +
              "else " +
              "    return 0 " +
              "end";
          
          // DefaultRedisScript 是 RedisScript 的标准实现
          DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
          redisScript.setScriptText(scriptText);
          
          // 必须指定脚本的返回类型，这里 DEL 返回的是数字 (Long)
          redisScript.setResultType(Long.class); 
          return redisScript;
      }
  }
  ```



**3. 在 Service 中执行脚本**

```java
import org.springframework.data.redis.core.script.RedisScript;
import java.util.Collections;

@Service
public class LockService {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private RedisScript<Long> compareAndDeleteScript; // 注入脚本 Bean

    /**
     * 尝试安全地释放锁
     * @param lockKey 锁的 Key
     * @param expectedValue 期望的值 (例如线程ID)
     * @return true 如果释放成功, false 如果锁不存在或值不匹配
     */
    public boolean safeReleaseLock(String lockKey, String expectedValue) {
        
        // 执行脚本
        Long result = redisTemplate.execute(
            compareAndDeleteScript,          // 1. 要执行的脚本
            Collections.singletonList(lockKey), // 2. KEYS 列表 (KEYS[1] = lockKey)
            expectedValue                      // 3. ARGV 列表 (ARGV[1] = expectedValue)
        );

        // Lua 脚本返回 1 (DEL 成功) 或 0 (GET 失败或值不匹配)
        return result != null && result == 1L;
    }
    
    // 假设先加锁
    public void lock(String lockKey, String value) {
        redisTemplate.opsForValue().set(lockKey, value);
    }
}

// === 测试 ===
// lockService.lock("my_lock", "thread_id_1");
// boolean success = lockService.safeReleaseLock("my_lock", "thread_id_1"); // true
// boolean fail = lockService.safeReleaseLock("my_lock", "thread_id_2");    // false
```



## 5.3 发布/订阅 (Pub/Sub)

- Redis 提供了发布/订阅功能，允许客户端作为消息的发布者 (Publisher) 或订阅者 (Subscriber)，实现进程间的消息通信

- SDR 对此进行了封装，核心组件有两个：

  1. **发布者 (Publisher)**：通过 `redisTemplate.convertAndSend(String channel, Object message)` 方法来发布消息
  2. **订阅者 (Subscriber)**：
     - 需要配置一个 `RedisMessageListenerContainer` (消息监听器容器)，并将实现了 `MessageListener` 接口的 Bean 注册给它

- SDR 的 Pub/Sub 功能提供了一种**配置驱动**的方式来实现消息订阅，

  > (`RedisMessageListenerContainer` + `MessageListenerAdapter`)

  并提供了简单的 API 来发布消息 (`convertAndSend`)



### 5.3.1 配置订阅者 (Listener)

- 订阅消息是一个**阻塞**操作（需要一个线程专门等待消息）

  - SDR 提供的 `RedisMessageListenerContainer` 会自动管理连接和线程池，我们只需要配置它

    这通常在 `RedisConfig` 配置类中完成

    ```java
    import org.springframework.data.redis.connection.RedisConnectionFactory;
    import org.springframework.data.redis.listener.RedisMessageListenerContainer;
    import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;
    import org.springframework.data.redis.listener.Topic;
    import org.springframework.data.redis.listener.ChannelTopic;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor; // (需要配置线程池)
    import java.util.concurrent.ThreadPoolExecutor;
    
    
    // 假设在 RedisConfig.java 中
    @Configuration
    public class RedisConfig {
    
        // ... (之前的 RedisTemplate, CacheManager, RedisScript Bean) ...
    
        /**
         * 配置消息监听器容器 (RedisMessageListenerContainer)
         * 这个 Bean 会自动启动一个线程池，用于接收 Redis 的 Pub/Sub 消息
         */
        @Bean
        public RedisMessageListenerContainer redisMessageListenerContainer(
                RedisConnectionFactory connectionFactory,
                MessageListenerAdapter userUpdateListenerAdapter, // 1. 注入我们稍后定义的监听器
                ThreadPoolTaskExecutor redisListenerExecutor // 2. 注入一个专用的线程池
        ) {
            
            RedisMessageListenerContainer container = new RedisMessageListenerContainer();
            container.setConnectionFactory(connectionFactory);
            
            // 设置用于执行监听器的线程池
            container.setTaskExecutor(redisListenerExecutor);
    
            // 3. 将监听器添加到容器，并指定监听的频道 (Topic)
            // 我们可以监听多个频道
            // 监听 "user:update" 频道
            container.addMessageListener(userUpdateListenerAdapter, new ChannelTopic("user:update")); 
            // 监听 "order:create" 频道
            // container.addMessageListener(orderListenerAdapter, new ChannelTopic("order:create")); 
            
            return container;
        }
    
        /**
         * (可选, 但推荐) 为 Redis 监听器配置一个专用的线程池
         */
        @Bean
        public ThreadPoolTaskExecutor redisListenerExecutor() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            executor.setCorePoolSize(10);
            executor.setMaxPoolSize(20);
            executor.setQueueCapacity(1000);
            executor.setThreadNamePrefix("RedisListener-");
            executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
            executor.initialize();
            return executor;
        }
    }
    ```



### 5.3.2 实现消息监听器 (MessageListener)

- 我们有两种方式实现监听器：

  1. (推荐) 任意 POJO (Plain Old Java Object)，结合 `MessageListenerAdapter`
  2. (底层) 直接实现 `MessageListener` 接口

  这里我们使用更简单的 POJO 方式

  ```java
  import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;
  import org.springframework.stereotype.Component;
  
  // 1. 定义一个 POJO 监听器 (一个普通的 Bean)
  @Component
  public class UserUpdateReceiver {
  
      // 定义一个方法，用于接收消息
      // 方法名默认为 "handleMessage"，也可以在 MessageListenerAdapter 中指定
      public void handleMessage(String message, String channel) {
          // 注意：这里的 message 类型取决于发布时使用的序列化器
          // 因为 RedisTemplate<String, Object> 默认使用 JSON，
          // 如果发布的是一个对象，这里可能需要接收一个 String (JSON串) 或 Map
          
          System.out.println("--- 收到消息 ---");
          System.out.println("频道 (Channel): " + channel);
          System.out.println("消息 (Message): " + message);
          
          // 假设 message 是 JSON 字符串，我们可以反序列化
          // ObjectMapper om = new ObjectMapper();
          // User user = om.readValue(message, User.class);
          // System.out.println("更新用户缓存: " + user.getId());
      }
  }
  
  // 2. 在配置类 (如 RedisConfig) 中，将 POJO 包装成 Adapter
  @Configuration
  public class RedisConfig {
      // ... (上面的 Container Bean) ...
      
      /**
       * 创建消息监听器适配器 (MessageListenerAdapter)
       * @param receiver 注入我们上面定义的 POJO (UserUpdateReceiver)
       */
      @Bean
      public MessageListenerAdapter userUpdateListenerAdapter(UserUpdateReceiver receiver) {
          // 1. 传入 POJO 实例
          // 2. (可选) 指定 POJO 中用于接收消息的方法名 (默认是 handleMessage)
          MessageListenerAdapter adapter = new MessageListenerAdapter(receiver, "handleMessage");
          
          // (重要) 设置序列化器
          // 我们需要告诉 Adapter 如何反序列化消息体 (message)
          // 这里我们假设消息体是字符串
          adapter.setSerializer(new StringRedisSerializer()); 
          return adapter;
      }
  }
  ```



### 5.3.3 发布消息 (Publisher)

- 发布消息非常简单，直接使用 `RedisTemplate` 即可

  ```java
  @Service
  public class UserService {
  
      @Autowired
      private RedisTemplate<String, Object> redisTemplate;
  
      public void updateUser(User user) {
          // 1. 更新数据库
          // db.update(user);
          
          // 2. (可选) 删除缓存
          // redisTemplate.delete("user:" + user.getId());
  
          // 3. 发布 "用户已更新" 的消息
          // 其他订阅了 "user:update" 频道的服务 (包括自己) 都会收到这个消息
          // 我们可以只发送 ID，也可以发送整个 user 对象 (它会被 JSON 序列化)
          
          String message = "User " + user.getId() + " updated";
          // redisTemplate.convertAndSend("user:update", user); // 发送对象
          
          redisTemplate.convertAndSend("user:update", message); // 发送字符串
          
          System.out.println("发布消息: " + message);
      }
  }
  ```





# 实战：整合 Redisson

- 在实际项目中，我们经常将 Spring Data Redis (SDR) 和 Redisson 两个框架**混合使用**

- 这种模式的**核心思想**是：

  - **Spring Data Redis**：利用其强大的 `RedisTemplate` 和 `@Cacheable` 缓存抽象，专注于**数据访问**和**应用缓存**

  - **Redisson**：利用其专业、健壮的**分布式锁 (`RLock`)** 实现，专注于解决**并发安全**问题



## 6.1 为什么 SDR 的锁不够用？

- SDR 本身可以通过 `redisTemplate.opsForValue().setIfAbsent(key, value, timeout)` (即 `SETNX`) 来实现一个基础的分布式锁

- 但这个基础实现有**致命缺陷**：

  1. **无法自动续期 (看门狗)**：
     - 锁设置了过期时间（例如 30 秒），如果业务线程执行超过了 30 秒（例如 `try-finally` 块中的业务逻辑卡顿），锁会自动释放。此时，其他线程会拿到锁，导致多个线程同时执行临界区代码，**锁失效**
  2. **实现复杂**：要实现一个生产可用的锁（可重入、公平、可续期、阻塞/非阻塞），需要编写非常复杂的 Lua 脚本

  Redisson 是一个功能更强大的 Redis 客户端，它**内置**了上述所有高级锁功能



## 6.2 整合 Redisson

### 6.2.1 添加依赖

- 在 `pom.xml` 中，除了 `spring-boot-starter-data-redis` 之外，再添加 Redisson 的 Spring Boot Starter 依赖：

  ```java
  <dependency>
      <groupId>org.redisson</groupId>
      <artifactId>redisson-spring-boot-starter</artifactId>
      <!-- 查找并使用最新的稳定版本 -->
      <version>3.17.7</version> 
  </dependency>
  ```



### 6.2.2 澄清：SDR 与 Redisson 的关系

- **SDR (基于 Lettuce)** 和 **Redisson (基于 Netty)** 是两个**完全独立**的 Redis 客户端
- 它们**不能共享**连接池（SDR 用 `LettuceConnectionFactory`，Redisson 用自己的 `RedissonClient`）
- “整合”的含义是：它们各自配置连接到**同一个** Redis 服务器实例（使用相同的主机、端口、密码），各自管理自己的连接池，各司其职



### 6.2.3 配置 Redisson

- `redisson-spring-boot-starter` 允许我们在 `application.yml` 中配置 Redisson

  ```yaml
  spring:
    data:
      redis:
        # --- SDR (Lettuce) 的配置 (保持不变) ---
        host: 127.0.0.1
        port: 6379
        database: 0
        lettuce:
          pool:
            max-active: 8
  
  # --- Redisson 的独立配置 ---
  redisson:
    # Redisson 默认使用 "redis://" 协议
    # "singleServerConfig" 用于配置单机模式
    singleServerConfig:
      address: "redis://127.0.0.1:6379"
      database: 1 # (可选) Redisson 可以使用不同的 DB
      password: null # (如果有密码)
      # (重要) Redisson 的连接池配置
      connectionPoolSize: 64
      connectionMinimumIdleSize: 10
  ```



- Spring Boot 会自动读取这个配置，并创建一个 `RedissonClient` Bean 供我们注入



## 6.3 使用 Redisson 的 RLock

- 现在，我们可以在业务代码中同时注入 `RedisTemplate` 和 `RedissonClient`

  - **场景**：防止缓存击穿的另一种方法，或者防止库存超卖

    ```java
    import org.redisson.api.RLock;
    import org.redisson.api.RedissonClient;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.data.redis.core.RedisTemplate;
    import org.springframework.stereotype.Service;
    import java.util.concurrent.TimeUnit;
    
    @Service
    public class ProductService {
    
        @Autowired
        private RedisTemplate<String, Object> redisTemplate; // 来自 SDR
        
        @Autowired
        private RedissonClient redissonClient; // 来自 Redisson
    
        public Product getProduct(Long productId) {
            String cacheKey = "product:" + productId;
            
            // 1. (SDR) 尝试从缓存获取
            Product product = (Product) redisTemplate.opsForValue().get(cacheKey);
            if (product != null) {
                return product;
            }
    
            // 2. (Redisson) 缓存未命中，获取分布式锁
            // 使用一个专门的锁 Key
            String lockKey = "lock:product:" + productId;
            RLock lock = redissonClient.getLock(lockKey);
    
            try {
                // 3. 尝试加锁 (阻塞式，带租约时间)
                // 尝试在 10 秒内获取锁，获取后，锁的有效期为 30 秒
                boolean isLocked = lock.tryLock(10, 30, TimeUnit.SECONDS);
    
                if (isLocked) {
                    // 4. (关键) Double-Check，再次检查缓存
                    // 防止在获取锁的过程中，已有其他线程完成了缓存重建
                    product = (Product) redisTemplate.opsForValue().get(cacheKey);
                    if (product != null) {
                        return product; // 其他线程已重建，直接返回
                    }
                    
                    // 5. (SDR) 缓存中确实没有，查询数据库
                    System.out.println("加锁成功，查询数据库: " + productId);
                    product = database.queryById(productId); // 模拟查库
                    
                    if (product != null) {
                        // 查到了，放入缓存 (设置过期时间)
                        redisTemplate.opsForValue().set(cacheKey, product, 1, TimeUnit.HOURS);
                    } else {
                        // 数据库也没有 (防止缓存穿透)
                        redisTemplate.opsForValue().set(cacheKey, null, 5, TimeUnit.MINUTES);
                    }
                    
                    return product;
                    
                } else {
                    // 未获取到锁 (例如 10 秒超时)
                    System.out.println("未获取到锁，服务降级或重试...");
                    return null; // 或者抛出异常
                }
    
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                return null;
            } finally {
                // 6. 释放锁
                // (重要) 必须检查锁是否仍被当前线程持有
                if (lock.isHeldByCurrentThread()) { 
                    lock.unlock();
                }
            }
        }
    }
    ```



- **Redisson RLock 的优势 (看门狗)**： 如果你使用 `lock.lock()` (不带租约时间参数)，Redisson 会启用“看门狗”机制

  - 它会给锁一个默认的 30 秒过期时间

  - Redisson 会启动一个后台线程，**每隔 10 秒**检查一次：如果持有锁的业务线程还在运行，就**自动将锁的过期时间重置为 30 秒**

  - 这就是“自动续期”，它保证了只要业务线程不宕机，锁就不会在业务执行完之前过期



