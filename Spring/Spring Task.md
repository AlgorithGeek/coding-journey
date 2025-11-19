# Spring Task

## 1. Spring Task 概述

### 1.1 什么是 Spring Task？

Spring Task 是 Spring Framework 核心框架 (`spring-context` 模块) 自带的轻量级任务调度框架。它允许开发者在 Spring 应用程序中便捷地执行定时任务和异步任务

它的主要特点是 **轻量级** 和 **内嵌性** ：

1. **无需额外依赖**：只要引入了 `spring-context`，就具备了任务调度的能力
2. **内嵌于应用**：任务的调度和执行都在当前 Spring 应用程序的进程内完成，不需要额外部署独立的调度中心或代理



### 1.2 核心概念与组件

Spring Task 的核心抽象主要有两个接口，理解它们的职责划分至关重要：

- **`TaskExecutor` (任务执行器)**

  - **职责**：定义任务 **“如何被执行”**

  - **说明**：

    - 这是 Spring 对 Java `java.util.concurrent.Executor` 接口的抽象

      它封装了线程池，决定了任务是提交到哪个线程池、以什么策略（例如，并发、串行）来运行

  - **主要应用**：`@Async` 异步方法默认会寻找一个 `TaskExecutor` 类型的 Bean 来执行

  

- **`TaskScheduler` (任务调度器)**

  - **职责**：定义任务 **“何时被执行”**
  - **说明**：这是 Spring 提供的用于“定时”和“周期性”执行任务的抽象。它负责管理定时触发器（Triggers）和待执行的任务（Runnables）
  - **主要应用**：`@Scheduled` 注解的任务默认由一个 `TaskScheduler` 来调度和执行



**总结**：

- `TaskExecutor` 只关心“执行”这个动作（通常是异步的），而 `TaskScheduler` 关心“定时”这个策略

  在 `@Scheduled` 场景中，`TaskScheduler` 不仅要负责“定时”，还要负责提供线程来“执行”，因此它内部通常会持有一个 `TaskExecutor`（或等价的线程池）



### 1.3 Spring Task 的适用场景(单机、轻量级)

Spring Task 的“简单”既是优点也是局限。它非常适合以下场景：

- **单机应用**：在单个服务器实例上运行的简单应用，如 定时清理临时文件、发送日报邮件
- **轻量级调度**：对任务的执行时效性、失败重试、高可用性要求不高的场景
- **辅助性任务**：例如定期的缓存刷新、数据同步、应用健康检查等

**重要局限**： 

- Spring Task 本质上是一个 **单机调度器**。如果将同一个 Spring Boot 应用部署为多个实例（集群）， **每个实例都会独立触发和执行相同的定时任务** ，这通常会导致数据错乱或重复执行。它本身不具备集群感知、分布式协调或持久化任务的能力



### 1.4 与 Quartz、XXL-Job 等框架的对比

| 特性            | Spring Task (spring-context) | Quartz                             | XXL-Job                                 |
| --------------- | ---------------------------- | ---------------------------------- | --------------------------------------- |
| **定位**        | 轻量级、单机、内嵌式         | 专业、功能全面、可内嵌             | 分布式任务调度平台                      |
| **部署**        | 内嵌于应用，无额外组件       | 可内嵌于应用                       | 独立的 **调度中心** + 应用侧 **执行器** |
| **集群/分布式** | **不支持** (会导致重复执行)  | **支持** (需DB持久化)              | **核心特性** (高可用、分片)             |
| **监控管理**    | 较弱 (Actuator)              | 一般 (需JMX或DB)                   | **强大** (自带Web管理界面)              |
| **动态管理**    | 较弱 (需编程式)              | 支持                               | **核心特性** (界面化CRUD)               |
| **依赖**        | 无额外依赖                   | 需引入 `quartz` 依赖               | 需部署调度中心，应用引入SDK             |
| **选型建议**    | 简单的单机任务               | 复杂的、需要持久化和集群的单体应用 | 企业级的、海量任务、分布式环境          |



## 2. 快速入门与基础配置

### 2.1 依赖说明 (Maven/Gradle)

Spring Task 的核心功能包含在 `spring-context` 模块中

- **对于 Spring Boot 项目 (推荐)**

  你 **不需要** 添加任何额外的依赖

  spring-boot-starter` (或 `spring-boot-starter-web` 等) 已经自动包含了 `spring-context`，Spring Task 的所有功能开箱即用

  ```xml
  <!-- spring-boot-starter 已包含 spring-context -->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
  </dependency>
  ```



### 2.2 启用调度 (`@EnableScheduling`)

要让 Spring 发现并执行 `@Scheduled` 注解的任务，必须显式地“启用”这个功能

你需要在 Spring (或 Spring Boot) 的主配置类上添加 `@EnableScheduling` 注解

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling // 关键：启用 Spring Task 的定时任务功能
public class YourApplication {

    public static void main(String[] args) {
        SpringApplication.run(YourApplication.class, args);
    }
}
```

- **工作原理**：`@EnableScheduling` 注解会触发 Spring 容器去扫描所有 Bean，查找被 `@Scheduled` 注解标记的方法，并将它们注册到任务调度器中



### 2.3 编写第一个定时任务 (`@Scheduled`)

启用调度后，我们可以在任何 Spring Bean (如 `@Component`, `@Service` 等) 中编写定时任务方法

1. 创建一个组件类 (例如 `ScheduledTasks`)
2. 编写一个公共 (`public`)、无返回值 (`void`)、无参的方法
3. 在该方法上添加 `@Scheduled` 注解，并指定一种调度策略



**示例：一个每5秒执行一次的任务**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import java.time.LocalDateTime;

@Component
public class ScheduledTasks {

    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

    /**
     * 演示 @Scheduled 注解
     * 使用 fixedRate 策略：表示每 5000 毫秒（5秒）执行一次。
     * 计时器从任务 *开始* 执行时计算。
     * * (我们将在下一章详细讲解 fixedRate, fixedDelay 和 cron 的区别)
     */
    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("当前时间：{}", LocalDateTime.now());
        // 假设这里执行一些业务逻辑
    }
}
```



**运行与验证**：

启动 Spring Boot 应用。无需做任何其他操作，在控制台的日志中，会看到 `ScheduledTasks` 每隔5秒打印一次当前时间，证明第一个定时任务已经成功运行

```
... [           main] c.y.YourApplication            : Started YourApplication in 1.234 seconds
... [   scheduling-1] c.y.ScheduledTasks             : 当前时间：2023-10-27T10:00:05.123
... [   scheduling-1] c.y.ScheduledTasks             : 当前时间：2023-10-27T10:00:10.124
... [   scheduling-1] c.y.ScheduledTasks             : 当前时间：2023-10-27T10:00:15.125
```

- **注意日志中的 `[ scheduling-1]`**：这是执行任务的线程名

  默认情况下，Spring Task 使用一个名为 `scheduling-1` 的单线程池来执行所有任务。后续章节将深入探讨这一点



## 3. `@Scheduled` 注解

`@Scheduled` 注解是 Spring Task 最核心的注解。它提供了三种主要的调度方式：`fixedRate`, `fixedDelay`, 和 `cron`



### 3.1 `fixedRate`:固定速率(按开始时间计时)

- **定义**：`fixedRate` (固定速率) 属性用于指定任务 **“上一次开始”** 到 **“下一次开始”** 之间的时间间隔
- **计时方式**：计时器在任务 **开始执行** 的那一刻启动
- **适用场景**：适用于那些不关心上一次任务是否执行完毕，只要求“固定频率”触发的场景（例如：每秒钟刷新一次缓存，即使刷新动作本身需要100毫秒）



**时序图** (假设 `fixedRate = 5000`，任务执行耗时 `taskTime`):

```
|---- task (耗时 2s) ----|
|<---------- 5s --------->|
                        |---- task (耗时 2s) ----|
                        |<---------- 5s --------->|
                                                |---- task (耗时 2s) ----|
```

- **代码示例**：

  ```JAVA
  /**
   * fixedRate = 5000:
   * 任务在 10:00:00 开始，耗时 2s (在 10:00:02 结束)。
   * 下一次任务将在 10:00:05 (10:00:00 + 5s) 开始。
   */
  @Scheduled(fixedRate = 5000)
  public void taskWithFixedRate() throws InterruptedException {
      log.info("Fixed Rate Task - START");
      Thread.sleep(2000); // 模拟任务耗时 2 秒
      log.info("Fixed Rate Task - END");
  }
  ```

- **重要陷阱 (fixedRate 与任务耗时)**：

  如果任务的执行时间 **大于** `fixedRate` 的间隔时间，会发生什么？ 

  - **答案**：Spring 会等待当前任务**执行完毕之后**，**立即** 启动下一次任务，**不会** 发生并发执行（在默认单线程模型下）

  **时序图** (假设 `fixedRate = 3000`，任务执行耗时 `taskTime = 4000ms`):

  ```
  |---- task (耗时 4s) ----|
  |<------ 3s ---->| (本应在第3s启动，但任务未结束)
                          |---- task (耗时 4s) ----| (上一个刚结束，下一个立即开始)
                          |<------ 3s ---->|
                                                  |---- task (耗时 4s) ----|
  ```

  - **日志表现**：

    - 你会发现任务在 10:00:00 开始，10:00:04 结束

      下一次任务 **立即** 在 10:00:04 开始，在 10:00:08 结束。任务的实际执行间隔变成了 4s，而不是 3s



### 3.2 `fixedDelay`:固定延迟(按结束时间计时)

- **定义**：`fixedDelay` (固定延迟) 属性用于指定任务 **“上一次结束”** 到 **“下一次开始”** 之间的时间间隔
- **计时方式**：计时器在任务 **执行完毕** 的那一刻启动
- **适用场景**：适用于那些必须保证两次任务之间有“休息时间”或“安全间隔”的场景（例如：数据轮询，每次轮询结束后必须等待 10 秒再进行下一次）

**时序图** (假设 `fixedDelay = 5000`，任务执行耗时 `taskTime = 2000ms`):

```
|---- task (耗时 2s) ----|
                        |<---------- 5s --------->|
                                                |---- task (耗时 2s) ----|
                                                                        |<---------- 5s --------->|
                                                                                                |---- task (耗时 2s) ----|
```

- **代码示例**：

  ```java
  /**
   * fixedDelay = 5000:
   * 任务在 10:00:00 开始，耗时 2s (在 10:00:02 结束)。
   * 计时器在 10:00:02 启动。
   * 下一次任务将在 10:00:07 (10:00:02 + 5s) 开始。
   */
  @Scheduled(fixedDelay = 5000)
  public void taskWithFixedDelay() throws InterruptedException {
      log.info("Fixed Delay Task - START");
      Thread.sleep(2000); // 模拟任务耗时 2 秒
      log.info("Fixed Delay Task - END");
  }
  ```

- **对比 `fixedRate`**： 

  - 使用 `fixedDelay` 时，任务的“总周期”是 **不固定** 的，它等于 `(任务执行时间 + fixedDelay) `

    而 `fixedRate` 的周期是固定的（除非任务执行时间超过了 `fixedRate`）



### 3.3 `initialDelay`: 初始延迟

- **定义**：`initialDelay` (初始延迟) 属性用于指定**应用启动后，第一次执行任务之前**的延迟时间

- **说明**：它会且仅会生效一次。它可以和 `fixedRate` 或 `fixedDelay` 配合使用

- **适用场景**：

  1. **防止应用启动拥堵**：避免应用一启动就立即执行耗时任务，影响启动速度或抢占CPU/IO资源
  2. **等待依赖资源**：等待其他资源（如数据库连接池、消息队列）完全初始化后再开始执行任务

- **代码示例**：

  ```java
  /**
   * initialDelay = 10000:
   * 应用启动成功后，等待 10 秒钟。
   *
   * fixedDelay = 5000:
   * 10秒钟后，开始第一次执行。
   * 第一次执行结束后，再等待 5 秒，开始第二次执行，以此类推。
   */
  @Scheduled(initialDelay = 10000, fixedDelay = 5000)
  public void taskWithInitialDelay() {
      log.info("Initial Delay Task - Executed after 10s delay");
  }
  ```



### 3.4 `cron`: Cron 表达式

- **定义**：`cron` 属性允许你使用 Cron 表达式来定义复杂的、基于 **绝对时间** 的调度规则

- **适用场景**：`fixedRate` 和 `fixedDelay` 无法满足的复杂调度。例如：“每个工作日的上午10点”、“每个月最后一天”等

- **优先级**：`cron` 属性的优先级最高。如果你同时设置了 `cron` 和 `fixedRate` (或 `fixedDelay`)，只有 `cron` 会生效

- **代码示例**：

  ```java
  /**
   * "0 15 10 * * ?" 含义：
   * 每天上午 10:15:00 执行。
   * (下一章将详细学习 Cron 表达式的语法)
   */
  @Scheduled(cron = "0 15 10 * * ?")
  public void taskWithCronExpression() {
      log.info("Cron Task");
  }
  ```

- **`zone` 属性 (配合 cron 使用)**

  - **用途**：`zone` 属性用于指定在哪个 **时区** 来解释 `cron` 表达式

  - **说明**：

    - 默认情况下，Spring 使用服务器的本地时区

      如果你的应用部署在不同时区的服务器上，或者你需要按特定时区（如 "Asia/Shanghai"）的时间执行任务，`zone` 属性就非常重要

  - **示例**：

    ```java
    // 无论服务器在哪个时区，都在 "GMT+8" (上海时间) 的 10:15 执行
    @Scheduled(cron = "0 15 10 * * ?", zone = "Asia/Shanghai")
    public void taskWithZone() {}
    ```



- **`...String` 属性 (支持占位符)**

  - **用途**：`fixedRateString`, `fixedDelayString`, `initialDelayString` 和 `cron` 属性都允许你使用占位符 (`${...}`) 从配置文件 (如 `application.properties` 或 `application.yml`) 中动态读取调度参数，这提供了更大的灵活性

  - **示例** (`application.yml`):

    ```yaml
    my:
      task:
        rate: 5000
        delay: 10000
        cron: "0 0 2 * * ?"
    ```

  - **代码**：

    ```java
    @Scheduled(fixedRateString = "${my.task.rate}")
    public void taskWithConfigRate() {}
    
    @Scheduled(initialDelayString = "${my.task.delay}", fixedDelayString = "${my.task.delay}")
    public void taskWithConfigDelay() {}
    
    @Scheduled(cron = "${my.task.cron}", zone = "Asia/Shanghai")
    public void taskWithConfigCron() {}
    ```



## 4. Cron 表达式详解

Cron 表达式是一种用于定义复杂、基于日历的调度规则的字符串。它是 `@Scheduled` 注解最强大、最常用的配置方式

### 4.1 语法结构 (秒 分 时 日 月 周)

Spring Task 中最常用的 Cron 表达式包含 **6 个** 字段，用空格分隔

```
┌───────────── 1. 秒 (0-59)
│ ┌───────────── 2. 分钟 (0-59)
│ │ ┌───────────── 3. 小时 (0-23)
│ │ │ ┌───────────── 4. 日 (1-31)
│ │ │ │ ┌───────────── 5. 月 (1-12 或 JAN-DEC)
│ │ │ │ │ ┌───────────── 6. 星期 (0-7 或 SUN-SAT; 0 和 7 都代表周日)
│ │ │ │ │ │
* * * * * *
```

**示例**：`0 15 10 * * ?` 表示 **每天10点15分**

> 理解的时候，我的想法是：用第一个 * 匹配脑海中的 “每” ，比如这里的第一个 * 在 “日” 上，就是 每天 10点15分，理解的时候其实挺松散的，多看示例



**关键规则：日期 vs 星期**

在 Cron 表达式中，"日 (第4位)" 和 "星期 (第6位)" 这两个字段是 **互斥** 的。你必须在两者中选择一个来指定，另一个则必须设置为 `?` (不指定值)

- **错误示例**：`0 0 12 15 * MON` (同时指定了 15号 和 星期一❌)
- **正确示例 (1)**：`0 0 12 15 * ?` (每月15号的12点)
- **正确示例 (2)**：`0 0 12 ? * MON` (每周一的12点)

(我们将在下一节 "特殊字符" 中详细讲解 `?` 的含义)



**关于 7 字段 (年份)**

你可能也会看到 7 个字段的 Cron 表达式，第 7 位是 "年份 (可选, 1970-2099)"

- Spring 5.3+ 版本开始支持 7 字段格式
- 在 Spring 5.3 之前，Spring 仅支持 6 字段格式 (不含年份)

**建议**：除非你有“在特定年份才执行”的罕见需求，否则**一律使用 6 字段格式**。这是最通用、兼容性最好的写法



### 4.2 特殊字符详解

| 字符     | 名称                 | 含义与示例                                                   |
| -------- | -------------------- | ------------------------------------------------------------ |
| **`\*`** | **任意值 (每)**      | 匹配该字段的任意值。• `*` 在“分钟”字段，表示“每分钟”。• `* * * * * ?` 表示“每秒钟” |
| **`?`**  | **不指定 (互斥)**    | 仅用于“日期”和“星期”字段。用于在这两者之间进行互斥选择<br/>• `0 0 12 15 * ?` - `?` 在“星期”字段，表示不关心是星期几，只要日期是15号<br/>• `0 0 12 ? * MON` - `?` 在“日期”字段，表示不关心是几号，只要是星期一 |
| **`-`**  | **范围**             | 指定一个范围<br/>• `9-17` 在“小时”字段，表示 9 点到 17 点 (包含 9 和 17)<br/>• `MON-FRI` 在“星期”字段，表示周一到周五 |
| **`,`**  | **列表 (枚举)**      | 指定多个离散的值<br/>• `1,15` 在“日期”字段，表示1号和15号<br/>• `MON,WED,FRI` 在“星期”字段，表示周一、周三、周五 |
| **`/`**  | **增量 (步长)**      | 指定一个起始值和一个增量<br/>• `0/15` 在“秒”字段，表示从第0秒开始，每隔15秒执行一次 (0s, 15s, 30s, 45s)<br/>• `5/10` 在“分钟”字段，表示从第5分钟开始，每隔10分钟执行一次 (5m, 15m, 25m, ...) |
| **`L`**  | **最后 (Last)**      | 仅用于“日期”和“星期”字段。含义随字段变化<br/>• `L` 在“日期”字段，表示“本月的最后一天” (例如 1月31日, 2月28日或29日)<br/>• `L` 在“星期”字段，表示“本周的最后一天”，即星期六 (SAT 或 6)<br/>• `5L` 在“星期”字段，表示“本月的最后一个星期四” (5=THU) |
| **`W`**  | **工作日 (Weekday)** | 仅用于“日期”字段。指定离给定日期最近的工作日 (周一至周五)<br />• `15W`：如果15号是工作日,则在15号执行;如果15号是周六,则在14号(周五)执行;如果15号是周日,则在16号(周一)执行<br/>• **注意**：`L` 和 `W` 可以组合为 `LW`，表示“本月最后一个工作日” |
| **`#`**  | **第几个 (Nth)**     | 仅用于“星期”字段。指定“本月的第几个星期X”<br />• `6#3`：表示“本月的第三个星期五” (6=FRI, #3=第三个)<br />• `2#1`：表示“本月的第一个星期一” (2=MON, #1=第一个) |



### 4.3 常见示例与在线生成工具

掌握了特殊字符后，**最快学习 Cron 的方式就是看示例**

| 需求                    | Cron 表达式            | 解释                                            |
| ----------------------- | ---------------------- | ----------------------------------------------- |
| **每分钟**              | `0 * * * * ?`          | 在每分钟的第0秒执行                             |
| **每5分钟**             | `0 0/5 * * * ?`        | 从0分开始，每5分钟执行一次                      |
| **每小时**              | `0 0 * * * ?`          | 在每小时的0分0秒执行                            |
| **每天上午10:15**       | `0 15 10 * * ?`        | 每天的10点15分0秒执行                           |
| **每天凌晨2点**         | `0 0 2 * * ?`          | 每天的2点0分0秒执行                             |
| **工作日 (周一至周五)** | `0 0 9-17 ? * MON-FRI` | 每周一至周五的9点到17点之间，每小时的0分0秒执行 |
| **每月1号和15号**       | `0 0 10 1,15 * ?`      | 每月1号和15号的上午10点整执行                   |
| **每月最后一天**        | `0 59 23 L * ?`        | 每月最后一天的23点59分0秒执行                   |
| **每月第三个周五**      | `0 15 10 ? * 6#3`      | 每月的第三个星期五 (6=FRI) 的上午10点15分执行   |

**重要提示：在线工具**⭐⭐

Cron 表达式非常容易写错。在生产环境中使用之前，**强烈建议** 使用在线工具来生成和验证你的表达式

- 搜索 "Cron 在线生成器" 可以找到很多有用的工具，它们可以让你直观地看到表达式的执行时间

> 这个东西我认为多看看就能看的非常熟练，就能看懂，之后我个人不建议写，我建议去用生成工具去生成



### 4.4 代码示例

下面是一个 Spring Boot 组件 (`@Component`)，它展示了如何在 `@Scheduled` 注解中实际使用 4.3 节中的 Cron 表达式

**注意**：为了演示，这里的方法都是空方法。在实际项目中，它们将包含你的业务逻辑

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class CronExamples {

    private static final Logger log = LoggerFactory.getLogger(CronExamples.class);

    // 示例 1: 每分钟执行一次 (在第0秒)
    @Scheduled(cron = "0 * * * * ?")
    public void taskEveryMinute() {
        log.info("taskEveryMinute - 每分钟执行");
    }

    // 示例 2: 每5分钟执行一次
    @Scheduled(cron = "0 0/5 * * * ?")
    public void taskEvery5Minutes() {
        log.info("taskEvery5Minutes - 每5分钟执行");
    }

    // 示例 3: 每天上午10:15执行
    @Scheduled(cron = "0 15 10 * * ?")
    public void taskDailyAt10_15AM() {
        log.info("taskDailyAt10_15AM - 每天上午10:15执行");
    }

    // 示例 4: 工作日 (周一至周五) 的上午9点到下午5点，每小时执行
    // 9-17 表示 9, 10, 11, 12, 13, 14, 15, 16, 17
    @Scheduled(cron = "0 0 9-17 ? * MON-FRI")
    public void taskWorkingHours() {
        log.info("taskWorkingHours - 工作日9点到17点每小时执行");
    }

    // 示例 5: 每月1号和15号的凌晨4点
    @Scheduled(cron = "0 0 4 1,15 * ?")
    public void taskTwiceMonthly() {
        log.info("taskTwiceMonthly - 每月1号和15号凌晨4点执行");
    }

    // 示例 6: 每月最后一天23点执行
    @Scheduled(cron = "0 0 23 L * ?")
    public void taskLastDayOfMonth() {
        log.info("taskLastDayOfMonth - 每月最后一天23点执行");
    }

    // 示例 7: 每月第三个周五的下午2点
    // 6#3 = 第三个(3)星期五(6)
    @Scheduled(cron = "0 0 14 ? * 6#3")
    public void taskThirdFridayOfMonth() {
        log.info("taskThirdFridayOfMonth - 每月第三个周五下午2点执行");
    }

    // 示例 8: (7字段带年份) 2025年和2026年的圣诞节中午12点
    // 仅在 Spring 5.3+ 支持
    // @Scheduled(cron = "0 0 12 25 12 ? 2025,2026")
    // public void taskChristmas2025And2026() {
    //     log.info("Merry Christmas!");
    // }
}
```





## 5. 任务执行与线程模型

理解 Spring Task 的默认线程模型，是避免在生产环境中出现任务阻塞、“假死”的先决条件

### 5.1 默认线程池 (重要：默认单线程)

当你在 Spring Boot 应用中添加了 `@EnableScheduling` 并编写了 `@Scheduled` 任务，你可能认为 Spring 会智能地为每个任务分配线程

**但事实是：**

默认情况下，Spring Task 会创建一个**大小为 1 的单线程池** (`SingleThreadScheduledExecutor`) 来执行 **所有** 的 `@Scheduled` 任务

- **线程名**：这个唯一的线程通常名为 `scheduling-1`
- **行为**：所有任务，无论你有多少个 `@Scheduled` 方法，都会被提交到这同一个线程中，**排队串行执行**

**为什么 Spring 这么设计？**

1. **轻量级**：作为 `spring-context` 的一部分，它旨在提供最基础的调度能力，而不引入多线程的复杂性
2. **资源开销**：单线程池的资源开销最小
3. **“安全”的默认值**：对于不知道并发编程的开发者，单线程至少能保证任务不会“同时执行”，避免了线程安全问题

但这种“安全”的默认值，也带来了巨大的风险。我们将在下一节探讨它的"局限性"



### 5.2 单线程模型的局限性与风险 (任务阻塞)

单线程模型最大的风险就是 **“任务阻塞”**

如果你的多个 `@Scheduled` 任务中，**任何一个** 任务执行时间过长（例如：网络超时、大量数据处理），或者进入了死循环，它将会：

1. **阻塞** 队列中所有其他的任务
2. **“饿死”** 那些执行频率高、但耗时短的任务

**场景演示：长任务“饿死”短任务**

假设我们有两个任务：

1. **短任务 (Task A)**：`fixedRate = 2000` (每 2 秒执行一次)，用于打印心跳
2. **长任务 (Task B)**：`fixedRate = 5000` (每 5 秒执行一次)，但它需要 `sleep(10000)` (耗时 10 秒)

**代码示例**：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class DefaultSchedulerProblems {

    private static final Logger log = LoggerFactory.getLogger(DefaultSchedulerProblems.class);

    // 任务A: 预期每 2 秒执行一次
    @Scheduled(fixedRate = 2000)
    public void taskA_heartbeat() {
        // 打印日志，并显示当前线程名
        log.info("Task A (Heartbeat) - Executing... Thread: {}", 
                 Thread.currentThread().getName());
    }

    // 任务B: 预期每 5 秒执行一次，但自身耗时 10 秒
    @Scheduled(fixedRate = 5000)
    public void taskB_longRunning() throws InterruptedException {
        log.info("Task B (Long Task) - START. Thread: {}", 
                 Thread.currentThread().getName());
        
        Thread.sleep(10000); // 模拟一个 10 秒的耗时操作
        
        log.info("Task B (Long Task) - END");
    }
}
```

**预期的执行结果 (如果是并发的)**： Task A 会稳定地每 2 秒打印一次。Task B 会在另一个线程上慢慢执行

**实际的执行结果 (默认单线程)**：

```
... [scheduling-1] Task A (Heartbeat) - Executing... Thread: scheduling-1
... [scheduling-1] Task B (Long Task) - START. Thread: scheduling-1
(--- scheduling-1 线程被 Task B 占用了 10 秒 ---)
(--- 在这 10 秒内, Task A 的 (2s, 4s, 6s, 8s, 10s) 5次触发机会全部错过了 ---)
... [scheduling-1] Task B (Long Task) - END
... [scheduling-1] Task A (Heartbeat) - Executing... Thread: scheduling-1
... [scheduling-1] Task A (Heartbeat) - Executing... Thread: scheduling-1
... [scheduling-1] Task B (Long Task) - START. Thread: scheduling-1
(--- scheduling-1 线程再次被 Task B 占用了 10 秒 ---)
...
```

**结论**：

- 所有任务都在 `scheduling-1` 线程上执行
- 当 Task B 开始执行时，它会**独占** `scheduling-1` 线程长达 10 秒
- 在这 10 秒内，Task A (心跳任务) 即使到了触发时间点，也无法被执行，因为它在排队等待 `scheduling-1` 线程被释放
- **后果**：你的“心跳”任务完全失效了，应用的监控可能会误报“应用已死”

这就是默认单线程模型最大的陷阱



### 5.3 如何验证默认是单线程？

验证 Spring Task 默认是否为单线程，最简单、最直观的方法就是：**查看日志输出**

1. **配置你的日志格式**： 

   - 确保你的日志（无论是 Logback, Log4j2 还是 JUL）配置为可以输出 **线程名**

     在 Spring Boot 的默认 `logback-spring.xml` 配置中，`[%thread]` 通常都已包含

     一个典型的日志 `pattern` 可能是： `%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n`

2. **运行 5.2 中的代码**： 运行上一节 (5.2) 那个包含 `taskA_heartbeat` 和 `taskB_longRunning` 的 `DefaultSchedulerProblems` 类

3. **观察控制台**： 你不需要做任何其他事情，只需要看日志。你会清晰地看到：

   ```
   ... [scheduling-1] Task A (Heartbeat) - Executing... Thread: scheduling-1
   ... [scheduling-1] Task B (Long Task) - START. Thread: scheduling-1
   ...
   ... [scheduling-1] Task B (Long Task) - END
   ... [scheduling-1] Task A (Heartbeat) - Executing... Thread: scheduling-1
   ```

4. **得出结论**：

   - **关键证据**：日志中方括号 `[]` 里的 `scheduling-1` 就是线程名
   - **事实**： 无论是 `Task A` 还是 `Task B`，它们在打印日志时，`Thread.currentThread().getName()` 返回的 **都是 "scheduling-1"**
   - **证明**：这无可辩驳地证明了，这两个（甚至所有）`@Scheduled` 任务都是在 **同一个线程** 中排队执行的



## 6. 自定义任务执行器 (TaskExecutor)

在第 5 章，我们发现了 Spring Task 默认的“单线程陷阱”：

- 所有 `@Scheduled` 任务默认都在同一个 `scheduling-1` 线程中串行执行，一个长任务会阻塞所有其他任务

本章的核心目的，就是**彻底解决这个问题**，让我们所有的定时任务能够**并发**执行



### 6.1 为什么需要自定义？(实现并发)

默认的 `TaskScheduler` 内部只使用了一个单线程

我们要做的，就是替换掉这个“小水管”，换上一个“多车道的高速公路”—— **即一个配置了多线程的 `ThreadPoolTaskScheduler`**

自定义任务执行器主要有以下三个目的：

1. **实现并发执行**： 

   - 这是最主要的目的。配置一个大小为 10 (或更多) 的线程池，`Task A` 就可以在 `thread-1` 上执行，`Task B` 可以在 `thread-2` 上执行

     它们**互不阻塞**，`Task B` 即使耗时 10 秒，`Task A` 的心跳任务也能在 `thread-3` 上稳定地每 2 秒执行一次

     

2. **线程隔离与命名**： 

   - 我们可以给定时任务的线程池起一个专属的名字（例如 `my-scheduled-`），这样在排查日志或OOM问题时，可以清晰地知道是哪个线程池出了问题，避免和 `@Async` 或 Tomcat 的 `http-nio-` 线程混淆

     

3. **精细化资源控制**： 

   - 自定义线程池后，我们可以设置它的核心线程数、最大线程数、队列容量、拒绝策略等（就像你之前笔记里写的那样），对服务器资源进行精细化控制



接下来，我们将介绍两种最主流的配置方式



### 6.2 方法1:实现`SchedulingConfigurer`接口(推荐)

这是 Spring 官方推荐的、用于**替换默认调度器**（那个单线程的）的标准方式

- **工作原理**： 

  - `@EnableScheduling` 注解在启动时，会主动查找 Spring 容器中所有实现了 `SchedulingConfigurer` 接口的 Bean

    如果找到了，Spring 就会调用这个 Bean 的 `configureTasks` 方法来进行配置，**而不是使用它自己的默认单线程调度器**

- **核心**： `configureTasks` 方法提供了一个 `ScheduledTaskRegistrar` (任务注册器) 对象，你只需要调用它的 `setScheduler` 方法，把你自定义的、包含多线程的 `TaskScheduler` 线程池传给它。

**代码示例**： (这个配置类解决了第 5 章的“单线程陷阱”)

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;

import java.util.concurrent.ThreadPoolExecutor;

@Configuration
@EnableScheduling
public class ScheduleMultiThreadConfig implements SchedulingConfigurer {

    /**
     * 1. 实现 SchedulingConfigurer 接口，重写 configureTasks 方法
     */
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        // 2. 核心：将我们自定义的多线程 TaskScheduler 注入到任务注册器中
        //    这样，Spring 就会使用这个线程池来执行所有 @Scheduled 任务
        taskRegistrar.setScheduler(myTaskScheduler());
    }

    /**
     * 3. 声明一个多线程的 TaskScheduler Bean
     * 注意：这里的 Bean 名称可以随意，Spring 是通过 configureTasks 
     * 方法来获取调度器的，而不是通过 @Bean 的名称
     */
    @Bean
    public TaskScheduler myTaskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        
        // 4. 配置线程池大小
        scheduler.setPoolSize(10);
        
        // 5. 配置线程名前缀，方便排查问题
        scheduler.setThreadNamePrefix("my-scheduled-");
        
        // 6. 配置拒绝策略 (可选：当线程池和队列都满了之后的处理策略)
        //    CallerRunsPolicy：由调用者线程（通常是主线程）自己来执行任务
        scheduler.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        
        // 7. 配置等待任务完成 (可选：应用关闭时等待任务执行完毕)
        scheduler.setWaitForTasksToCompleteOnShutdown(true);
        scheduler.setAwaitTerminationSeconds(60);
        
        // 8. 初始化
        scheduler.initialize();
        return scheduler;
    }
}
```

**验证**： 当你使用这个 `ScheduleMultiThreadConfig` 配置类替换了 `@EnableScheduling` 的简单配置后，再次运行第 5.2 节的“长短任务”示例

你会从日志中发现：

1. 线程名不再是 `scheduling-1`，而是变成了 `my-scheduled-1`, `my-scheduled-2`...
2. `taskB_longRunning` (长任务) 在 `my-scheduled-1` 上执行
3. `taskA_heartbeat` (心跳任务) 会在 `my-scheduled-2` (或其他) 线程上 **正常地、每2秒** 执行一次

**`taskA` 不再被 `taskB` 阻塞。问题解决！**



### 6.3 方法二:声明自定义TaskScheduler Bean(Spring Boot 方式)

这种方法利用了 Spring Boot 的自动配置机制，代码更少，更简洁

- **工作原理**： Spring Boot 的 `TaskSchedulingAutoConfiguration` 在自动配置时，会检查容器中是否 **已经存在** 一个 `TaskScheduler` 类型的 Bean
  1. 如果 **不存在**：Spring Boot 会创建一个 **单线程** 的 `ThreadPoolTaskScheduler` 作为默认值 (这就是 5.1 节中“陷阱”的来源)
  2. 如果 **存在**：Spring Boot 会 **跳过** 自己的默认配置，转而使用你提供的那个 Bean
- **核心**： 我们只需要在配置类中声明一个 `TaskScheduler` 类型的 Bean，并将其配置为多线程即可。Spring Boot 会自动检测到它并使用它

**代码示例**： (这个配置类同样解决了第 5 章的“单线程陷阱”)

```JAVA
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;

import java.util.concurrent.ThreadPoolExecutor;

@Configuration
@EnableScheduling
public class ScheduleMultiThreadConfigSimple {

    /**
     * 1. 声明一个 TaskScheduler 类型的 Bean
     * 
     * 2. @Primary (推荐)：
     *    如果容器中有多个 TaskScheduler Bean，这个注解确保 Spring 
     *    优先选择这一个用于 @Scheduled 任务
     *
     * 3. Bean 名称 (可选)：
     *    在 Spring Boot 中，如果 Bean 的名称恰好是 "taskScheduler"，
     *    它也会被优先选中，甚至可以不用 @Primary
     *    但使用 @Primary 是更通用的做法
     */
    @Bean
    @Primary
    public TaskScheduler myPrimaryTaskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        
        scheduler.setPoolSize(10);
        scheduler.setThreadNamePrefix("primary-scheduled-");
        scheduler.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        scheduler.setWaitForTasksToCompleteOnShutdown(true);
        scheduler.setAwaitTerminationSeconds(60);
        scheduler.initialize();
        
        return scheduler;
    }
}
```



**对比 6.2 和 6.3**：

- **方法一 (6.2 `SchedulingConfigurer`)**：
  - **优点**：功能最强。它能拿到 `ScheduledTaskRegistrar` 对象，因此不仅可以设置调度器，还能**动态添加任务**（第 8 章会讲）。这是官方推荐的“标准”方式
  - **缺点**：多写几行代码
- **方法二 (6.3 `TaskScheduler` Bean)**：
  - **优点**：代码最简洁
  - **缺点**：只能替换调度器，但拿不到 `ScheduledTaskRegistrar`，无法动态添加任务

**结论**：如果只需要解决“单线程陷阱”问题，方法二 (6.3) 足够了。如果未来可能需要动态注册任务，请使用方法一 (6.2)



### 6.4 方法三:使用`application.yml`(最简单)

这是在 Spring Boot 环境下配置 `@Scheduled` 线程池 **最简单、最推荐** 的方法。它不需要你编写任何 Java `Configuration` 类

- **工作原理**： 

  - Spring Boot 的 `TaskSchedulingAutoConfiguration` 会自动读取 `application.yml` (或 `application.properties`) 中以 `spring.task.scheduling` 为前缀的属性

    它会使用这些属性来**自动**为你创建一个 `ThreadPoolTaskScheduler`

- **核心**： 你只需要在 `application.yml` 中提供 `pool.size` 即可

**代码示例**： (这个配置**同样**解决了第 5 章的“单线程陷阱”)

**`application.yml`**

```YAML
spring:
  task:
    scheduling:
      # 关键配置：线程池大小
      # 只要这个值 > 1，Spring Boot 就会自动创建一个多线程的 TaskScheduler
      # 来替换掉默认的单线程调度器。
      pool:
        size: 10
      
      # 线程名前缀
      thread-name-prefix: yml-scheduled-
      
      # (可选) 应用关闭时的配置
      shutdown:
        # 等待所有任务执行完毕后再关闭
        await-termination: true
        # 最大等待时间
        await-termination-period: 30s
```

**`application.properties` (等效配置)**

```PROPERTIES
spring.task.scheduling.pool.size=10
spring.task.scheduling.thread-name-prefix=properties-scheduled-
spring.task.scheduling.shutdown.await-termination=true
spring.task.scheduling.shutdown.await-termination-period=30s
```

**验证**：

- 不需要任何 Java 配置类 (删除 6.2 和 6.3 的 `Configuration` 类)，只需要 `@EnableScheduling` 和上面的 `yml` 配置

  再次运行 5.2 的“长短任务”示例，你会发现日志中的线程名变成了 `yml-scheduled-1`, `yml-scheduled-2`...，并且任务并发执行，不再阻塞



## 7. 异步任务 (`@Async`)

`@Async` 注解的核心作用是**将同步方法调用转换为异步方法调用**

`@Scheduled` (定时任务)   

`@Async` (异步任务)

- **`@Scheduled` vs `@Async`**
  - **`@Scheduled`**：关心 **“何时”** 执行 (Timing)。它负责按 `cron`, `fixedRate` 等策略 **定时触发** 任务
  - **`@Async`**：关心 **“如何”** 执行 (Concurrency)。它负责将一个方法的执行体 **提交到线程池中** ，使调用者 **立即返回** ，而不需要等待方法执行完毕

`@Async` 通常用于：

- Web 接口中耗时操作的异步化（例如：下单后异步发送邮件、记录日志）
- 并行执行多个独立操作
- (我们稍后会讲) 与 `@Scheduled` 结合，实现更复杂的并发控制



### 7.1 启用异步 (`@EnableAsync`)

与 `@EnableScheduling` 类似，要让 Spring 识别并处理 `@Async` 注解，你必须显式地“启用”它

你需要在 Spring (或 Spring Boot) 的主配置类（或任何 `@Configuration` 类）上添加 `@EnableAsync` 注解

```JAVA
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling // 启用 @Scheduled
@EnableAsync      // 关键：启用 @Async
public class YourApplication {

    public static void main(String[] args) {
        SpringApplication.run(YourApplication.class, args);
    }
}
```

- **工作原理**： 

  - `@EnableAsync` 会触发 Spring 容器去扫描所有 Bean，为被 `@Async` 注解标记的方法创建代理

    当你调用这个方法时，代理会拦截调用，将实际的执行逻辑打包成一个任务，并提交到一个 `TaskExecutor` (任务执行器) 线程池中去执行，从而实现异步

- **独立的线程池**：

  -  `@EnableAsync` 默认会寻找一个 `TaskExecutor` 类型的线程池

     `@EnableScheduling` 默认会寻找一个 `TaskScheduler` 类型的线程池

     **重要的是**：这两个是**独立**的。在 Spring Boot 中，你可以（也应该）分别为它们配置不同的线程池



### 7.2 在 `@Scheduled` 方法上结合使用 `@Async`

这是一个非常强大的高级用法，它提供了最精细的并发控制，并能 **完美解决 `fixedRate` 的陷阱**

当你将 `@Scheduled` 和 `@Async` 放在 **同一个方法** 上时：

- **`@Scheduled`** 负责“定时触发”
- **`@Async`** 负责将“方法体”的执行 **抛到另一个线程池**

**执行流程** (假设 `@Scheduled` 线程池为 `scheduling-pool`，`@Async` 线程池为 `async-pool`)：

1. `scheduling-pool` (例如 `scheduling-1` 线程) 按时触发 `@Scheduled` 方法
2. `@Async` 的代理立即介入，将 **真正的方法逻辑** 打包成一个新任务
3. `@Async` 将这个新任务提交到 `async-pool` (例如 `async-1` 线程) 中去执行
4. `scheduling-1` 线程 **立即返回** ，几乎没有任何耗时。它马上就空闲了，可以去触发其他定时任务
5. `async-1` 线程开始 **真正执行** 那个耗时的方法体

**代码示例**：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ScheduledWithAsync {

    private static final Logger log = LoggerFactory.getLogger(ScheduledWithAsync.class);

    /**
     * @Scheduled: 确保该方法每 3 秒被触发一次。
     * @Async:     确保每次触发时，方法体都在 @Async 的线程池中执行。
     */
    @Async // (我们将在 7.3 节配置它所使用的线程池)
    @Scheduled(fixedRate = 3000)
    public void fixedRateTaskWithAsync() throws InterruptedException {
        log.info("Task START - Thread: {}", Thread.currentThread().getName());
        Thread.sleep(5000); // 模拟一个 5 秒的耗时操作
        log.info("Task END - Thread: {}", Thread.currentThread().getName());
    }
}
```

**日志表现 (对比 3.1 节的陷阱)**：

- **没有 `@Async`** (3.1节的陷阱)：
  - 任务耗时 5s > `fixedRate` 3s。任务会在 `scheduling-1` 上执行，5s 后结束。下一次任务**立即**在 `scheduling-1` 上开始。实际周期变成 5s
- **有了 `@Async`** (本节示例)：
  - `00:00:00`: `scheduling-1` 触发任务，`@Async` 将其抛给 `async-1` 执行。`scheduling-1` 立即释放
  - `00:00:00`: `async-1` 开始执行，耗时 5s (到 00:00:05 结束)
  - `00:00:03`: `scheduling-1` **准时**再次触发 `fixedRate`。`@Async` 将其抛给 `async-2` 执行
  - `00:00:03`: `async-2` 开始执行，耗时 5s (到 00:00:08 结束)
  - `00:00:06`: `scheduling-1` **准时**再次触发 `fixedRate`。`@Async` 将其抛给 `async-3` 执行

**结论**： 

- `@Scheduled + @Async` 组合拳，实现了 `fixedRate` 的**真正并发执行**

  调度器线程 (`scheduling-*`) 永远不会被阻塞，它只负责“发布命令”；而执行器线程 (`async-*`) 负责“干苦活”



### 7.3 @Async 与自定义 TaskExecutor 的关系

`@Async` 注解的方法需要一个 **`TaskExecutor`** (任务执行器) 类型的线程池来执行

#### 1. 默认的 TaskExecutor (Spring Boot)

- 如果你 **只** 使用了 `@EnableAsync` 而 **没有** 提供任何自定义的 `TaskExecutor` Bean，Spring Boot 会自动创建一个默认的 `ThreadPoolTaskExecutor`
- 这个默认线程池的配置通常是：核心线程 8 个，最大线程 `Integer.MAX_VALUE`，队列 `Integer.MAX_VALUE`
- **警告**：这是一个 **无界** 的线程池，在搞并发下可能导致 OOM (内存溢出)。因此， **强烈建议** 你总是自定义它

#### 2. 自定义 TaskExecutor (推荐)

与 `@Scheduled` (它使用 `TaskScheduler`) 不同，`@Async` 对应的是 `TaskExecutor`。

我们有两种方式来配置它（这和你之前提供的笔记完全一致）：

**方式一：使用 `application.yml` (简单)**

Spring Boot 提供了 `spring.task.execution` 前缀来自动配置 `@Async` 的线程池。

```yaml
spring:
  task:
    # ---------------------------------------------
    # (复习) 6.4 节：这是给 @Scheduled 用的
    scheduling:
      pool:
        size: 5
      thread-name-prefix: scheduled-
    # ---------------------------------------------
    
    # ---------------------------------------------
    # 7.3 节：这是给 @Async 用的
    execution:
      pool:
        # 核心线程数
        core-size: 8
        # 最大线程数
        max-size: 16
        # 队列容量
        queue-capacity: 100
      # 线程名前缀
      thread-name-prefix: async-exec-
      shutdown:
        await-termination: true
        await-termination-period: 30s
    # ---------------------------------------------
```

- **最佳实践**：通过 `yml` 分别配置 `scheduling` 和 `execution`，将“调度”和“执行”的线程池**彻底隔离**，这是最简单、最清晰的做法。



**方式二：使用 Java Bean (灵活)**

如果你需要更高级的配置 (如自定义 `RejectedExecutionHandler` 或 `AsyncUncaughtExceptionHandler`)，你可以使用 Java Bean。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;
import java.util.concurrent.ThreadPoolExecutor;

@Configuration
@EnableAsync
public class AsyncConfig {

    /**
     * 定义 @Async 使用的线程池
     * Bean 的名称 "asyncTaskExecutor" 是 Spring Boot 
     * @EnableAsync 自动配置时会寻找的默认名称之一。
     * 你也可以自定义名称，例如 "myExecutor"。
     */
    @Bean(name = "asyncTaskExecutor")
    public Executor asyncTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("bean-async-");
        // 配置拒绝策略
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        // (可选) 配置异步异常处理器
        // executor.setAsyncUncaughtExceptionHandler(...)
        executor.initialize();
        return executor;
    }
    
    // (可以定义多个 Executor Bean)
    @Bean(name = "anotherExecutor")
    public Executor anotherExecutor() {
        // ... 其他配置
        return new ThreadPoolTaskExecutor();
    }
}
```



#### 3. 如何指定 @Async 使用哪个线程池？

- **不指定**：`@Async`

  - Spring 会寻找一个名为 `taskExecutor` 的 Bean。
  - 如果找不到，则使用 `asyncTaskExecutor` (如上例)。
  - 如果都找不到，则使用 Spring Boot 自动创建的（或默认的）`TaskExecutor`。

- **指定名称**：`@Async("beanName")`

  - 你可以通过在 `@Async` 注解中传递 Bean 的名称，来精确控制这个异步方法在**哪个**线程池中执行。

  ```java
  @Async("asyncTaskExecutor") // 使用我们配置的 bean-async- 线程池
  public void task1() {
      log.info("Thread: {}", Thread.currentThread().getName());
  }
  
  @Async("anotherExecutor") // 使用另一个线程池
  public void task2() {
      log.info("Thread: {}", Thread.currentThread().getName());
  }
  ```





## 8. 动态任务调度 (编程式调度)

到目前为止，我们使用的 `@Scheduled` 注解是一种 **静态调度**。它的 `cron` 或 `fixedRate` 表达式必须在 **编译时** 就固定在代码中

**静态调度的局限性**： 

- 如果你的需求是 "每天凌晨2点执行"，那么 `@Scheduled(cron = "0 0 2 * * ?")` 非常完美

- 但如果你的需求是 "让用户在 Web 界面上动态设置一个任务的执行时间"，那么 `@Scheduled` 就无能为力了，因为你无法在运行时修改注解的属性



### 8.1 使用 `TaskScheduler` 接口

为了实现动态调度，我们必须抛弃 `@Scheduled` 注解，转而使用它底层的核心：**`TaskScheduler` 接口**

- `TaskScheduler` (通常是 `ThreadPoolTaskScheduler` 的实例) 是真正负责执行定时任务的组件。它提供了直接的 API 让我们在**运行时**提交任务

**核心 API**： `TaskScheduler` 接口提供了多种 `schedule(...)` 方法，最常用的有两个：

1. `schedule(Runnable task, Trigger trigger)`

   - **最强大** 的方法。它允许你传入一个 `Trigger` (触发器)

   - `Trigger` 会在 **每次** 任务执行完毕后，被调用来 **重新计算下一次** 的执行时间

   - 这使得“动态修改 Cron”变得非常容易：你只需要让 `Trigger` 下次从数据库或配置中心读取最新的 Cron 表达式即可

     

2. `schedule(Runnable task, Instant startTime)`

   - 在某个**绝对时间点**，只执行一次

     

3. `scheduleAtFixedRate(Runnable task, Duration period)`

   - 编程式的 `fixedRate`

     

4. `scheduleWithFixedDelay(Runnable task, Duration period)`

   - 编程式的 `fixedDelay`



**如何获取 `TaskScheduler` 实例？** 你只需要在你的 Spring Bean (如 `@Service`) 中，像注入其他 Bean 一样，`@Autowired` 注入它即可

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.TaskScheduler;
import org.springframework.stereotype.Service;

@Service
public class DynamicTaskService {

    // 注入在第 6 章配置的那个 TaskScheduler
    @Autowired
    private TaskScheduler taskScheduler;

    public void startDynamicTask() {
        // ... 我们将在 8.2 节中填充这里的逻辑 ...
        
        // taskScheduler.schedule(...);
    }
}
```

- **注意**：

  - 这里注入的 `taskScheduler` **就是** 我们在第 6 章配置的那个 **多线程** 的 Bean (无论是通过 `yml` 还是 `SchedulingConfigurer`)

    我们现在只是直接使用它来动态提交任务，而不是让 `@Scheduled` 注解来替我们提交



### 8.2 动态添加与修改任务 (实现 `Trigger`)

实现动态调度的**核心**是 `schedule(Runnable task, Trigger trigger)` 方法

- `Runnable task`：这是你的业务逻辑，即任务“做什么”
- `Trigger trigger`：这是你的调度逻辑，即任务“何时执行”

`Trigger` 是一个接口，它只有一个方法：`nextExecutionTime(TriggerContext context)`

**最简单**的动态 Cron 实现方式，就是使用 Spring 提供的 `CronTrigger`

**示例：从数据库动态读取 Cron 来执行任务**

我们来改造 8.1 节的 `DynamicTaskService`

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.Trigger;
import org.springframework.scheduling.TriggerContext;
import org.springframework.scheduling.support.CronTrigger;
import org.springframework.stereotype.Service;
import jakarta.annotation.PostConstruct; // (或 javax.annotation.PostConstruct)
import java.util.Date;

@Service
public class DynamicTaskService {

    @Autowired
    private TaskScheduler taskScheduler;

    // 假设这个变量模拟了数据库中存储的 Cron 表达式
    private volatile String cronFromDatabase = "0/10 * * * * ?"; // 默认每 10 秒

    // 1. 任务的业务逻辑 (做什么)
    private Runnable getTaskRunnable() {
        return () -> {
            // 这里是你的业务逻辑
            System.out.println("动态任务执行... 当前 Cron 是: " + cronFromDatabase);
            System.out.println("执行线程: " + Thread.currentThread().getName());
        };
    }

    // 2. 任务的触发器 (何时做)
    private Trigger getTaskTrigger() {
        return new Trigger() {
            @Override
            public Date nextExecutionTime(TriggerContext triggerContext) {
                // 每次执行完，都从 "数据库" 重新获取 Cron 表达式
                String newCron = getCronFromDatabase();
                
                // 使用 CronTrigger 来解析表达式并计算下一次执行时间
                CronTrigger trigger = new CronTrigger(newCron);
                return trigger.nextExecutionTime(triggerContext);
            }
        };
    }

    // 3. 在应用启动时，提交这个动态任务
    @PostConstruct
    public void scheduleDynamicTask() {
        taskScheduler.schedule(getTaskRunnable(), getTaskTrigger());
        System.out.println("动态任务已提交...");
    }

    // 模拟一个 HTTP 接口，用于在运行时修改 Cron
    public void updateCron(String newCron) {
        System.out.println("Cron 已被修改为: " + newCron);
        this.cronFromDatabase = newCron;
    }

    // 模拟从数据库读取
    private String getCronFromDatabase() {
        return this.cronFromDatabase;
    }
}
```



### 8.3 动态取消任务 (管理 `ScheduledFuture`)

在 8.2 节中，我们学会了如何动态 *添加* 和 *修改* 任务。但同样重要的，我们还需要知道如何 *取消* 一个已经提交的任务

**核心：`ScheduledFuture`**

`TaskScheduler` 的 `schedule(...)` 方法在提交任务后，会返回一个 `ScheduledFuture<?>` 对象

- **`ScheduledFuture`**：它代表了一个尚未完成的、被调度的任务
- **关键方法**：`future.cancel(boolean mayInterruptIfRunning)`

**实现思路**： 要取消一个任务，你 **必须** 在创建它时，将返回的 `ScheduledFuture` 对象 **保存** 起来。一个常见的做法是使用一个 `Map` 来存储它

**示例：实现动态添加、修改、取消**

我们来升级 8.2 节的 `DynamicTaskService`，让它支持取消

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.Trigger;
import org.springframework.scheduling.TriggerContext;
import org.springframework.scheduling.support.CronTrigger;
import org.springframework.stereotype.Service;
import java.util.Date;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ScheduledFuture;

@Service
public class DynamicTaskService {

    @Autowired
    private TaskScheduler taskScheduler;

    // 1. 关键：使用 Map 来存储任务和它对应的 Future
    private final Map<String, ScheduledFuture<?>> scheduledTasks = new ConcurrentHashMap<>();
    
    // 2. 模拟数据库中的 Cron 表达式
    private final Map<String, String> cronDatabase = new ConcurrentHashMap<>();

    public DynamicTaskService() {
        // 启动时，给一个默认任务
        cronDatabase.put("task-1", "0/10 * * * * ?");
    }

    /**
     * 启动 (或重启) 一个任务
     * @param taskId 任务的唯一 ID
     */
    public void startTask(String taskId) {
        // 2.1 先尝试取消已存在的同名任务 (实现 "重启" 逻辑)
        cancelTask(taskId);

        // 2.2 定义任务逻辑
        Runnable task = () -> {
            String cron = cronDatabase.get(taskId);
            System.out.printf("执行动态任务 [ %s ] (Cron: %s) - 线程: %s%n",
                    taskId, cron, Thread.currentThread().getName());
        };

        // 2.3 定义触发器 (从 "数据库" 动态获取 Cron)
        Trigger trigger = triggerContext -> {
            String cron = cronDatabase.get(taskId);
            if (cron == null) {
                // 如果任务被删除了，返回 null 停止执行
                return null;
            }
            CronTrigger cronTrigger = new CronTrigger(cron);
            return cronTrigger.nextExecutionTime(triggerContext);
        };

        // 2.4 提交任务，并保存 Future
        ScheduledFuture<?> future = taskScheduler.schedule(task, trigger);
        scheduledTasks.put(taskId, future);
        System.out.printf("任务 [ %s ] 已启动...%n", taskId);
    }

    /**
     * 停止一个任务
     * @param taskId 任务的唯一 ID
     */
    public void cancelTask(String taskId) {
        ScheduledFuture<?> future = scheduledTasks.get(taskId);
        if (future != null) {
            // 3. 关键：调用 cancel() 来停止任务
            // true: 尝试中断正在执行中的任务
            // false: 不中断，仅停止下一次调度
            future.cancel(true); 
            scheduledTasks.remove(taskId);
            System.out.printf("任务 [ %s ] 已停止...%n", taskId);
        }
    }

    /**
     * 修改任务的 Cron (通过 Trigger 自动生效)
     * @param taskId 任务 ID
     * @param newCron 新的 Cron 表达式
     */
    public void updateTaskCron(String taskId, String newCron) {
        System.out.printf("任务 [ %s ] 的 Cron 已被修改为: %s%n", taskId, newCron);
        cronDatabase.put(taskId, newCron);
        // 注意：我们不需要重启任务。
        // 在下一次 Trigger 被调用时，它会自动读到 newCron。
    }

    /**
     * 删除一个任务 (停止 + 移除 Cron)
     * @param taskId 任务 ID
     */
    public void deleteTask(String taskId) {
        cancelTask(taskId);
        cronDatabase.remove(taskId);
        System.out.printf("任务 [ %s ] 已被彻底删除...%n", taskId);
    }
}
```

**总结**： `TaskScheduler.schedule(Runnable, Trigger)` + `ScheduledFuture.cancel()` 是实现动态任务管理 (增、删、改) 的标准组合。



## 9. 错误处理与异常管理

在任务调度中，异常（例如 `NullPointerException`, `IOException`）是不可避免的

Spring Task 对异常的处理方式，尤其是 **默认** 的处理方式，是另一个必须注意的“陷阱”

### 9.1 默认的异常处理机制 (任务停止)

如果你在 `@Scheduled` 方法中没有手动 `try-catch` 异常，会发生什么？

**答案**：

1. 异常会被 Spring 的 `TaskScheduler` 捕获
2. 异常日志 **会被打印** 到控制台
3. **最重要 (陷阱)**：这个任务的 **后续调度将会被** **停止**！

是的，你没看错。对于一个 `@Scheduled` 任务（无论是 `fixedRate`, `fixedDelay` 还是 `cron`），只要它有一次未被捕获的异常抛出，Spring 默认会**“静默地”**将其从调度池中移除

- **后果**：这个任务就“死”了。它既不会在下个周期执行，也不会在应用重启前执行。你的数据同步、定时心跳等任务将 **永久失效**，而你可能毫不知情（除非你仔细检查日志）

**示例：让任务“死掉”**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class TaskExceptionDemo {

    private static final Logger log = LoggerFactory.getLogger(TaskExceptionDemo.class);
    private int counter = 0;

    /**
     * 这个任务会在第 3 次执行时抛出异常
     */
    @Scheduled(fixedRate = 2000) // 每 2 秒执行
    public void taskThatFails() {
        log.info("任务执行中... (第 {} 次)", ++counter);

        if (counter == 3) {
            log.warn("即将抛出异常...");
            // 模拟一个未捕获的运行时异常
            throw new RuntimeException("模拟任务执行失败！");
        }
    }
}
```

**日志表现**：

```log
... [my-scheduled-1] 任务执行中... (第 1 次)
... [my-scheduled-2] 任务执行中... (第 2 次)
... [my-scheduled-3] 任务执行中... (第 3 次)
... [my-scheduled-3] 即将抛出异常...
... [my-scheduled-3] o.s.s.s.TaskUtils$LoggingErrorHandler : Unexpected error occurred in scheduled task
java.lang.RuntimeException: 模拟任务执行失败！
	at c.y.TaskExceptionDemo.taskThatFails(TaskExceptionDemo.java:23)
	... (省略堆栈)
(--- 之后，控制台再也不会有 "任务执行中..." 的日志了 ---)
(--- 这个任务已经停止调度 ---)
```

**结论**：在 `@Scheduled` 方法中 **绝对不能** “裸奔”——即绝对不能让任何受检或非受检异常“逃逸”出方法



### 9.2 方法一：自定义 `ErrorHandler`

Spring 官方提供了一种“全局”捕获 `@Scheduled` 异常的机制：`ErrorHandler`

`ErrorHandler` 是一个简单的接口，只有一个 `handleError(Throwable t)` 方法

你可以创建一个自定义的 `ErrorHandler` Bean，并在配置 `TaskScheduler` 时将其注入。这样，**所有** 由该 `TaskScheduler` 调度的任务所抛出的异常，都会被这个 handler 接收，而不是由默认的 `LoggingErrorHandler`（它会停止任务）来处理



**步骤 1：创建一个自定义的 ErrorHandler**

这个 Handler 只负责记录错误，**但它不会停止任务的后续调度**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.util.ErrorHandler;

@Component
public class CustomTaskErrorHandler implements ErrorHandler {

    private static final Logger log = LoggerFactory.getLogger(CustomTaskErrorHandler.class);

    @Override
    public void handleError(Throwable t) {
        // 核心：只记录日志，不抛出异常
        // 这样 Spring 调度器就会认为异常已经被 "处理" 了
        // 并会继续安排下一次的调度
        log.error("定时任务发生未捕获异常", t);

        // 在这里你可以添加
        // 1. 告警通知（邮件、钉钉、企业微信）
        // 2. 错误日志上报
    }
}
```



**步骤 2：在配置 `TaskScheduler` 时注入它**

还记得**第 6 章**我们如何配置多线程的吗？我们需要在那个配置上做一点小小的修改

**如果你在使用 `SchedulingConfigurer` (6.2 节的方法)：**

```java
// 这是 6.2 节的 ScheduleMultiThreadConfig
@Configuration
@EnableScheduling
public class ScheduleMultiThreadConfig implements SchedulingConfigurer {

    // 1. 注入你刚刚创建的 ErrorHandler
    @Autowired
    private CustomTaskErrorHandler customTaskErrorHandler;

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.setScheduler(myTaskScheduler());
    }

    @Bean
    public TaskScheduler myTaskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(10);
        scheduler.setThreadNamePrefix("my-scheduled-");
        
        // 2. 关键：将 ErrorHandler 设置给调度器
        scheduler.setErrorHandler(customTaskErrorHandler);
        
        scheduler.initialize();
        return scheduler;
    }
}
```

**如果你在使用 `application.yml` (6.4 节的方法)：** `application.yml` **不支持**直接配置 `error-handler`。在这种情况下，你必须退回到使用 Java Bean 的配置方式（如上所示），或者使用 9.3 节的 `try-catch` 方法

**验证**： 再次运行 9.1 节的 `taskThatFails()` 示例。你会看到：

1. 第 3 次执行时，`CustomTaskErrorHandler` 打印了错误日志
2. **但任务没有停止！**
3. 第 4 次、第 5 次... 任务会**继续执行**（当然，它们会继续抛出异常并被 `ErrorHandler` 捕获）



### 9.3 方法二：`try-catch` 手动处理 (最推荐)

这是解决“任务停止”陷阱最简单、最灵活，也是 **最推荐** 的方式

- **原则**：永远不要让任何 **未经处理** 的异常（无论是 `RuntimeException` 还是 `CheckedException`）“逃逸”出你的 `@Scheduled` 方法
- **做法**：在 `@Scheduled` 方法的**最顶层**，使用一个 `try-catch(Throwable)` 来包裹你所有的业务逻辑

**示例：改造 9.1 节的“会死掉”的任务**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class TaskExceptionDemo {

    private static final Logger log = LoggerFactory.getLogger(TaskExceptionDemo.class);
    private int counter = 0;

    /**
     * 这个任务会在第 3 次执行时抛出异常，但会被 try-catch 捕获。
     */
    @Scheduled(fixedRate = 2000) // 每 2 秒执行
    public void taskThatCatches() {
        try {
            log.info("任务执行中... (第 {} 次)", ++counter);

            if (counter == 3) {
                log.warn("即将抛出异常...");
                // 模拟一个未捕获的运行时异常
                throw new RuntimeException("模拟任务执行失败！");
            }
            
            // ... 你所有的正常业务逻辑 ...

        } catch (Throwable t) {
            // 1. 关键：捕获所有异常 (包括 Error 和 Exception)
            // 2. 记录详细错误日志，用于后续排查
            log.error("定时任务[taskThatCatches]执行异常", t);
            
            // 3. (可选) 发送告警通知
            // sendAlert("定时任务失败", t);

            // 4. 重点：不要在这里 re-throw 异常！
            //    一旦 re-throw，任务还是会停止。
        }
    }
}
```

**日志表现**：

```
... [my-scheduled-1] 任务执行中... (第 1 次)
... [my-scheduled-2] 任务执行中... (第 2 次)
... [my-scheduled-3] 任务执行中... (第 3 次)
... [my-scheduled-3] 即将抛出异常...
... [my-scheduled-3] c.y.TaskExceptionDemo : 定时任务[taskThatCatches]执行异常
java.lang.RuntimeException: 模拟任务执行失败！
	at c.y.TaskExceptionDemo.taskThatCatches(TaskExceptionDemo.java:23)
... (--- 第 3 次执行结束 ---)
... [my-scheduled-4] 任务执行中... (第 4 次)
... [my-scheduled-1] 任务执行中... (第 5 次)
```

**结论**：

- 异常被 `catch` 块成功捕获并记录
- `taskThatCatches` 方法正常返回（没有抛出异常）
- Spring 调度器认为任务“成功”完成，并会 **继续调度** 下一次的执行
- **任务不会停止！**



**`try-catch` vs `ErrorHandler`**

| 对比       | 9.3 `try-catch` (推荐)                         | 9.2 `ErrorHandler` (不常用)            |
| ---------- | ---------------------------------------------- | -------------------------------------- |
| **粒度**   | **方法级别**。可针对不同任务做不同处理。       | **全局级别**。所有任务共用一个处理器。 |
| **灵活性** | **高**。可决定是否重试、是否告警。             | **低**。只能做统一的日志记录/告警。    |
| **侵入性** | 高。每个 `@Scheduled` 方法都要写 `try-catch`。 | 低。Java 配置一次，一劳永逸。          |
| **推荐度** | ⭐⭐⭐⭐⭐ (健壮性首选)                             | ⭐⭐ (仅用于简单日志记录)                |



### 9.4 结合Spring Retry实现重试

> `@Retryable`

`try-catch` (9.3) 只能保证任务不“死掉”，但如果任务失败了（例如，调用第三方 API 超时），`try-catch` 无法自动帮你重试

Spring Retry 是一个独立但常与 Spring Task 结合的框架，它允许你通过注解的方式，为“可能会失败”的方法添加强大的重试逻辑

#### 步骤 1：添加依赖

Spring Retry 不是 `spring-context` 的一部分。你可能需要手动添加它（尽管很多 `spring-boot-starter` 已经包含了 `spring-boot-starter-aop`）

```xml
<!-- 确保 spring-boot-starter-aop 存在 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
<!-- 添加 Spring Retry -->
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```



#### 步骤 2：启用重试

在你的主配置类或任何 `@Configuration` 类上添加 `@EnableRetry`

```java
@SpringBootApplication
@EnableScheduling
@EnableAsync
@EnableRetry // 关键：启用 Spring Retry
public class YourApplication {
    // ...
}
```



#### 步骤 3：分离业务逻辑 (AOP 代理陷阱)

这是一个 **非常重要** 的陷阱。

`@Retryable` (和 `@Async` 一样) 依赖 AOP 代理。你 **不能** 在一个 Bean 中，从一个 `@Scheduled` 方法  **内部调用**同个 Bean 的另一个 `@Retryable` 方法（`this.doWork()`），因为这会绕过代理

**最佳实践**：将你的重试逻辑放在一个**单独的** `@Service` Bean 中，然后让 `@Scheduled` 任务去**调用**它

**示例：`Scheduled` 调用 `Retryable`**

**1. `RetryableService.java` (负责干活的 Bean)**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Recover;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Service;

@Service
public class RetryableService {
    
    private static final Logger log = LoggerFactory.getLogger(RetryableService.class);

    /**
     * @Retryable 注解：
     * - value = RuntimeException.class: 只有在抛出 RuntimeException 时才重试。
     * - maxAttempts = 3: 最多尝试 3 次 (1 次初次 + 2 次重试)。
     * - backoff = @Backoff(delay = 2000, multiplier = 2): 
     * - delay = 2000: 第一次重试前等待 2 秒。
     * - multiplier = 2: 之后每次重试的等待时间 x2 (即第 2 次重试等 4 秒)。
     */
    @Retryable(
        value = { RuntimeException.class }, 
        maxAttempts = 3, 
        backoff = @Backoff(delay = 2000, multiplier = 2)
    )
    public void retryableWork() {
        log.warn("正在执行 [retryableWork]... (可能会失败)");
        
        // 模拟一个 50% 概率失败的操作
        if (Math.random() > 0.5) {
            log.error("[retryableWork] 失败，抛出异常！ (将触发重试)");
            throw new RuntimeException("模拟网络抖动");
        }
        
        log.info("[retryableWork] 成功执行！");
    }

    /**
     * @Recover 注解：
     * - 这是一个 "恢复" 方法。
     * - 它会在 @Retryable 尝试了所有次数 (3次) 之后，如果 *仍然* 失败，
     * Spring Retry 就会调用这个方法。
     * - 方法的参数 (Throwable t) 必须与 @Retryable 捕获的异常一致。
     */
    @Recover
    public void recover(RuntimeException t) {
        log.error("[recover] 所有重试均告失败！已触发恢复逻辑。 异常: {}", t.getMessage());
        // 在这里，你可以
        // 1. 发送严重告警
        // 2. 将失败的任务记录到数据库，以便人工处理
    }
}
```

**2. `ScheduledTasks.java` (负责调度的 Bean)**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component
public class ScheduledTasks {
    
    private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

    @Autowired
    private RetryableService retryableService;

    @Scheduled(fixedDelay = 30000) // 每 30 秒触发一次
    public void myScheduledTask() {
        // 我们在 9.3 节学到的 try-catch 仍然是必要的！
        // 为什么？因为 @Recover 方法本身也可能抛出异常，
        // 或者 @Retryable 配置不当。
        // try-catch 是保证 @Scheduled 不 "死掉" 的最后一道防线。
        try {
            // 调用另一个 Bean 的 @Retryable 方法
            retryableService.retryableWork();
            
        } catch (Throwable t) {
            // 如果 retryableWork 失败了，并且 recover 方法也失败了，
            // 最终会在这里被捕获。
            log.error("定时任务[myScheduledTask]的重试逻辑彻底失败", t);
        }
    }
}
```



