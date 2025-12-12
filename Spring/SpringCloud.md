# 微服务

> 先说一个比较有意思的点：**微服务架构天生支持用多种编程语言开发同一个项目的不同部分**

## 概念

**微服务**是一种**软件架构风格**，它将一个大型复杂应用，拆分为一组**小型的、松散耦合的、可独立部署的服务的集合**



## 理解

### 概念理解

对于初学者，一个 **“服务” **就可以理解为**一个独立运行的、只专注于做一件特定业务的小项目**

- 整个系统**不再是一个庞大的单体项目**，而是由许多**独立的“小项目”**  **协同工作**  来提供完整的业务能力

> 其实吧，要是需要说说“项目”和“服务”的区别
>
> - **“项目”** 更倾向于**物理静态层面**，比如代码文件夹什么的
> - **“服务”** 更倾向于**灵动层面**，比如进行啊，ing(英语中的进行时)啊，叽里咕噜什么什么的



### 举例理解

**在以前的单体架构里**

- 整个电商网站是一个巨大的“项目”

  用户管理、商品展示、下订单、支付所有功能都混在一个代码库里，打包成一个程序运行



**而在微服务架构里**：我们会按业务边界进行拆分：

- **用户服务**
  - 一个独立的**项目/应用**
  - **单一职责**：只处理用户相关的业务（注册、登录、资料修改等）
  - **数据隔离**：有自己独立的数据库，只存放用户信息
  - **可独立部署**：可以随时更新上线，而不影响商品或订单功能

- **商品服务**
  
  - 另一个独立的**项目/应用**
  - **单一职责**：只管理商品信息（上下架、价格、库存等）
  - **数据隔离**：有自己的数据库
  
- **订单服务**：

  - 又一个独立的**项目/应用**

  - **单一职责**：只负责处理订单流程

  - **通过API通信**：

    - 当它需要用户信息时（比如获取收货地址），

      它不会直接访问用户数据库，而是通过网络去**调用“用户服务”提供的API**，

      说：“请把 ID 为 123 的用户信息告诉我”



## 单体VS微服务

### 以前的单体架构

**单体架构**：

- 将应用程序的所有功能都打包在一个单元中

  > 比如一个电商网站，用户管理、商品管理、订单处理、支付功能等所有模块都在同一个代码库里，最终部署为单个可执行文件或 Web 应用

- **优点**：开发简单，易于测试和部署

- **缺点**：

  - 随着功能增多，代码库变得臃肿，难以维护；
  - 技术栈单一，无法灵活选择；
  - 任何微小的修改都需要重新部署整个应用；
  - 可扩展性差，只能整体扩展，造成资源浪费



### 全新的微服务架构

#### 核心特性

- **服务组件化**：每个微服务都是一个可独立替换和升级的组件

- **围绕业务能力组织**：
  - 团队不再按技术分层（如前端、后端、数据库团队），而是围绕业务功能组建跨职能团队
  - 例如，一个“订单团队”会负责所有与订单相关的服务

- **去中心化治理**：
  - 每个服务团队可以自由选择最适合其业务场景的技术栈（编程语言、数据库、框架等），不受限于其他服务

- **去中心化数据管理**：
  - 每个服务通常有自己独立的数据库
  - 这避免了单体应用中“一库天下”的模式，保证了服务的自治性，但也带来了数据一致性的挑战

- **基础设施自动化**：微服务架构的复杂性要求高度自动化的持续集成（CI）和持续部署（CD）流程，以及强大的运维监控体系

- **高容错性设计**：

  - 在分布式系统中，任何服务调用都可能失败

    因此，架构设计必须考虑到服务故障、网络延迟等问题，并具备服务降级、熔断等机制，以确保一个服务的失败不会导致整个系统崩溃



#### 微服务的挑战与缺点

- **分布式系统复杂性**：开发人员需要处理网络延迟、服务间通信、数据一致性、容错等分布式系统固有的问题

- **运维开销**：

  - 需要部署、管理、监控大量的服务实例，这对自动化运维（DevOps）能力提出了极高的要求

    你需要强大的服务发现、配置管理、日志聚合、监控告警等工具

- **数据一致性**：
  - 跨多个服务的事务处理变得非常复杂。通常需要采用最终一致性和Saga模式等方案来解决，这比传统的 ACID 事务更难实现

- **测试复杂性**：
  - 对单个服务的测试很简单，但进行跨服务的端到端集成测试则非常困难

- **服务间的依赖管理**：需要仔细管理服务之间的调用关系和版本兼容性，避免出现循环依赖或版本冲突



## 微服务解决方案

微服务解决方案有很多，但是 **Spring Cloud** 毫无疑问是 **Java 生态中最主流、最全面**的**微服务解决方案**



# Spring Cloud

## 基本概念

Spring Cloud 并非一个单一的技术，它也并不是一个框架，而是 **Spring官方 提供的一整套完整的微服务解决方案**




## 组件

**“组件”** 是 Spring Cloud 中的一部分，它是 Spring Cloud 中 **某个特定领域的解决方案**



## BOM 引入

- 这里使用 **BOM** 对 **Spring Cloud 生态依赖**进行**统一版本管理**，**消除版本冲突风险**

  引入具体依赖时**不再填写 `<version>`**，版本将 **由 BOM 自动对齐**为**经过验证的兼容组合**

- 记得 BOM 要引入到 `<dependencyManagement>` 中哦，不要直接把 BOM 放到引入依赖的地方了，也就是 `<dependencies>` 那边

- 引入示例

  ```xml
  <dependencyManagement>
    	<dependencies>
      	<dependency>
        		<groupId>org.springframework.cloud</groupId>
        		<artifactId>spring-cloud-dependencies</artifactId>
        		<version>2024.0.2</version>
        		<type>pom</type>
        		<scope>import</scope>
      	</dependency>
  	</dependencies>
  </dependencyManagement>
  ```

  - 注意这里一定要加上 **`<scope> import </scope>`**

- **对于版本的选择**

  - 先以你的 **Spring Boot 版本**为锚点 → 选对应的 **Spring Cloud 版本**（看官方兼容表）

  - 再选与该 **Spring Cloud 版本**匹配的 **Spring Cloud Alibaba (SCA) 版本**

    > 这个见下文中 Spring Cloud Alibaba 中的 “BOM 引入“



# Spring Cloud Alibaba

> 背景：
>
> - 早期的 Spring Cloud 生态主要依赖 Netflix OSS 全家桶，如 Eureka、Ribbon、Hystrix 等，但这些组件后来陆续停止维护。与此同时，阿里巴巴基于其在分布式系统的实践经验推出了 Spring Cloud Alibaba，为微服务架构提供了性能更高、功能更完善的替代方案。随着 Netflix 体系逐步退出主流，Spring Cloud Alibaba 成为了社区中广泛采用的方案之一。

## 基本概念

前面我们说过：

**Spring Cloud** 是 **Spring官方 提供的 一整套完整的微服务解决方案**

而 **Spring Cloud Alibaba** 则是**这套** **解决“方案”** 的 一套 **功能强大、性能卓越的 “具体方案实现”**

> Spring Cloud Alibaba 是对 Spring Cloud 体系在国内场景的增强实现，兼容 Spring Cloud 的标准接口，同时提供更多阿里云生态的集成能力



**理解**

> 就相当于对于一道数学题，
>
> - Spring Cloud 可类比于老师给的一个**解决方案**，比如：“这道题**要用一元二次方程来解**“
>
>   > 这里给的只是**解决方案**，并没有给**具体的方案实现**，
>   >
>   > 也就是说它并没有强制你必须用“配方法”或“因式分解法”这种 **具体的方案实现方式** 来一步一步地算
>   >
>   > 你可以自己根据这个**解决方案**，也就是 **“要用一元二次方程来解”** 这个**解决方案**，
>   >
>   > 来自行决定用什么样的 **具体的实现方案** 来解，比如“配方法”或“因式分解法”
>
> - 而 Spring Cloud Alibaba 就相当于一种非常强大，并且各方面都超级优秀的**“具体方案实现”**，
>
>   比如公式 $$x = \frac{-b \pm \sqrt{b^2-4ac}}{2a}$$ (假设在本案例中这是当前的最优选之一，假设它在本案例中远比“配方法”和“因式分解法”等要更好) 
>
>   使用这个公式可以非常完美地解决这个数学题，这个公式就是一个**强大并且卓越的 “具体的方案实现”**
>
>   > 这个公式也并没有违反 **“要用一元二次方程来解”** 的这个 **Spring Cloud “解决方案”**



## 组件

前面我们说过：**“组件”**是 Spring Cloud 中 **某个特定领域的解决方案**

而**”Spring Cloud Alibaba 的组件“** 则是 **这个** **特定领域的 解决 “方案”** 的 **功能强大、性能卓越的 “具体方案实现”**



**理解**

> 可去参考 Spring Cloud Alibaba 的 “基本概念” 中的 “理解” 去进行理解



## BOM 引入

- 这里使用 **BOM** 对 **Spring Cloud Alibaba生态依赖**进行**统一版本管理**，**消除版本冲突风险**

  引入具体依赖时**不再填写 `<version>`**，版本将 **由 BOM 自动对齐**为**经过验证的兼容组合**

- 记得 BOM 要引入到 `<dependencyManagement>` 中哦，不要直接把 BOM 放到引入依赖的地方了，也就是 `<dependencies>` 那边

- 引入示例

  ```xml
  <dependencyManagement>
    	<dependencies>
  		<dependency>
        		<groupId>com.alibaba.cloud</groupId>
        		<artifactId>spring-cloud-alibaba-dependencies</artifactId>
        		<version>2023.0.3.3</version>
        		<type>pom</type>
        		<scope>import</scope>	<!-- ⭐ 这里注意一定要写 -->
  		</dependency>
      </dependencies>
  </dependencyManagement>
  ```

  - ⭐注意这里一定要加上**`<scope> import </scope>`**，这里 **MVN Repository** 的网站里面没有写，我真服了，图片证据如下👇

    ![image-20251008075538973](./assets/image-20251008075538973.png)



- **对于版本的选择**

  - 先以你的 **Spring Boot 版本**为锚点 → 选对应的 **Spring Cloud 版本**（看官方兼容表）

  - 再选与该 **Spring Cloud 版本**匹配的 **Spring Cloud Alibaba (SCA) 版本**

    > 这个见前文中 Spring Cloud 中的 “BOM 引入“

  



# 组件

## 服务治理

### Nacos

> 名字来源：**Na**ming 和 **Co**nfiguration **S**ervice

![image-20251015003845338](./assets/image-20251015003845338.png)

#### 简介

##### 概述

官网：https://nacos.io/

**Nacos** 是阿里巴巴开源的一个 **动态服务发现、配置管理和服务管理平台**

在 **Spring Cloud Alibaba** 生态中，Nacos 是默认的 **注册中心** 和 **配置中心**



##### 功能

功能概括下来，就下面这**两个**

1. **服务注册与发现	**

   > 模块：**Naming Module（注册中心）**

2. **配置管理**

   > 这个在非微服务项目中也可以使用哦
   >
   > 配置实时更新，实时生效，还有一堆好处，超级好

   > 模块： **Config Module（配置中心）**



##### 背景

> 在现代微服务架构中，应用被拆分为众多独立的服务，例如订单服务、支付服务、用户服务等
>
> 随着系统的不断扩展，这些服务的实例数量、部署节点以及网络地址（IP 和端口）会因为 **弹性伸缩、故障转移、版本发布** 等操作而频繁动态变化
>
> 与此同时，系统中每个服务都依赖大量的配置项（如数据库连接、限流规则、功能开关等），这些配置还需要根据不同环境（开发、测试、生产）进行**隔离与切换**，这使得服务管理和配置维护变得愈加复杂
>
> **Nacos** 的诞生，正是为了 **优雅地解决微服务体系中 “服务动态注册与发现” 以及 “配置集中化管理” 这两大核心难题**



#### 核心概念

---

##### 服务注册与发现相关概念

###### **服务**

>Service

- 作用：表示一个对外提供功能的模块

- 举例：用户服务、订单服务、库存服务等

- 说明：
  - 在 Nacos 中，每个服务会注册自己的名字（Service Name）， 方便其他服务通过服务名发现并调用它



###### **服务实例**

> Service Instance

- 作用：

  - 提供该服务的**具体运行节点**
  - 一个服务往往有多个实例，用来实现**负载均衡和高可用**

- 举例：

  - `user-service` 可能有以下三个实例：

    ```makefile
    192.168.1.10:8080  
    192.168.1.11:8080  
    192.168.1.12:8080
    ```

---

##### 配置管理相关概念

###### **命名空间**

> Namespace

- 作用：

  - 用于**环境级别或租户级别的隔离**
  - 不同的命名空间下，可以存在相同的 `Group` 和 `Data ID`，互不影响

- 举例：

  - 在开发环境（dev）、测试环境（test）、生产环境（prod）中，

    同一个配置文件名 `application.yaml` 可以同时存在，但内容不同



###### **分组**

> Group

- 作用：
  - 在 **同一个命名空间** 下，用来对 **配置文件** 进行 **逻辑分组** 管理
  - 默认分组名是 `DEFAULT_GROUP`
- 举例：
  - 可以按照项目、模块或业务类型来区分，比如：
    - `order-group`（订单相关配置）
    - `user-group`（用户相关配置）



###### **数据 ID**

> Data ID

- 作用：
  - 是 Nacos 中 **组织配置的最小单位**，**一般对应一个配置文件**

- 举例：

  - `user-service.yaml`

  - `application-prod.properties`

- 说明：
  -  程序启动时会通过 `Data ID` 去拉取配置内容

---

#### 服务注册与发现

> 注册中心

##### 为什么需要?

在微服务架构中，一个完整的应用被拆分为多个独立的服务，这些服务部署在不同的进程或服务器上

这就带来了一个核心问题：**服务之间如何互相找到对方并进行通信 ？**

- 在传统的单体应用或者服务数量固定的情况下，我们可以在配置文件中硬编码服务的 IP 地址和端口

- 但是微服务**肯定不行**

  - 在微服务架构中，服务的数量和运行位置经常会发生变化：
    - 系统扩容时，可能新增多个实例
    - 某个实例宕机或重启后，**IP 地址可能改变**
    - 服务升级发布时，旧版本下线、新版本上线，**端口号也可能不同**
    - **......**

  - 如果我们在配置文件中 **手动写死每个服务的 IP 和端口** ，那每次变动都要去修改配置、重新部署，**非常麻烦，而且极容易出错**

所以，为了解决 **服务实例动态变化导致的调用不稳定问题**，

我们 **引入** 一个统一的机制，用来  **自动注册服务实例并提供动态发现能力**

这个机制就是 **服务注册与发现**

- **Naming Module（注册中心）** 负责实现这个机制

> ⭐emmmm...,说的还是有点书面，可能有点不好get，简单来说的话，可以把它理解成：他的核心职责是 **“维护一张动态的 IP端口 地址表”**
>
> - 然后这句：**自动注册服务实例并提供动态发现能力**，可以这样理解：
>   - **自动注册服务实例**：**自动记录上线服务的 IP**
>   - **动态发现能力**：**实时查询 IP 列表**

---

##### 工作原理

- **服务注册与发现** 的工作流程主要围绕三个核心动作：**服务注册**、**健康检查** 和 **服务发现(订阅与更新)**

###### 服务注册

1. 当一个**服务提供者**（例如 `user-service` 的某个实例）启动时，会向 **Nacos Server** 发起注册请求，携带关键信息：

   - **服务名**：如 `user-service`（通常来源于 `spring.application.name`）。
   - **IP 地址**：实例所在的服务器 IP
   - **端口号**：实例监听的端口
   - **元数据（metadata）**：如 `version`、`region`、`cluster`、`weight` 等

   

2. **Nacos Server** 收到注册请求后，会将这些信息记录在一个内部的服务**注册表**中



###### 健康检查

为了确保注册表中的服务实例都是“可用的”，Nacos Server 会与已注册的服务实例之间会维护健康状态，是一种“心跳”机制
- **临时实例（默认实例）**
  - 会 **周期性发送心跳**（默认间隔为数秒，可配置）
  - 若在**超时时间窗口**内未收到心跳，Nacos 会将实例标记为不健康
  - 继续超时则**自动摘除**，避免被调用到坏节点
- **持久实例**
  - 不依赖心跳自动过期，适合由外部/人工流程控制健康与上下线

具体心跳间隔与超时阈值为**可配置项**，不同版本/环境可能不同



###### 服务发现

- 当一个**服务消费者**（如 `order-service`）需要调用 `user-service`：

  1. 它向 Nacos Server 发起请求，来查询 `user-service` 的 **健康实例列表**

  2. `order-service` 会将获取到的实例列表 **缓存到本地** 以提高性能，避免每次调用都去查询 Nacos

  3. 同时，`order-service` 通过客户端 SDK **订阅** `user-service` 的实例变更

     当实例增删或状态变化时，Nacos Server 会**通知客户端**，`order-service` 的本地监听器收到事件后**更新本地缓存**（客户端也会定期刷新以兜底）

- **选择具体哪个实例** 进行调用由 **客户端侧的负载均衡组件** 决定（例如 OpenFeign + Spring Cloud LoadBalancer 的轮询/权重/同机房优先等策略），**Nacos 只负责提供健康实例列表**



#### 动态配置管理

##### 为什么需要？

- 在传统的开发模式中，我们通常将配置信息（如数据库连接、线程池大小、功能开关等）写在项目本地的配置文件中，例如 `application.properties` 或 `application.yml`

  这种方式简单直接，但在微服务架构下会暴露出一系列痛点：

  - **静态性与硬编码**：
    - 配置被打包在代码中。一旦需要修改配置，就必须经历“修改 → 重新打包 → 部署 → 重启服务”的流程，效率低下，且无法应对紧急的线上变更需求
  - **环境管理复杂**：
    - 一套代码需要部署到开发、测试、生产等多个环境中，每个环境的配置都不同。管理这些不同版本的配置文件非常繁琐，且容易出错
  - **管理不集中**：
    - 配置分散在各个微服务项目中，无法统一审计与治理；服务越多，配置管理成本越高
  - **无法实时生效**：
    - 对需要根据线上负载动态调整的参数（如熔断阈值、限流规则），传统方式做不到实时变更

- 所以，为了解决 **配置分散、变更低效、难以及时生效** 的问题，我们**引入**了**动态配置管理**，

  它能够在 **不重启应用** 的前提下**将配置 动态 下发到各服务实例**，并支持**多环境隔离、版本管理、灰度发布与回滚......**

  **Config Module（配置中心）** 负责实现这个机制



##### 核心能力

- Nacos 支持将**大部分易变配置**从应用中剥离，集中存放在 Nacos Server 上统一治理（保留少量启动级配置用于定位与引导）

  为解决传统方式的痛点，提供以下能力：

  - **集中管理（UI + OpenAPI/SDK）**：
    - 在控制台统一维护各服务/各环境配置，也可通过 OpenAPI 接口接入流水线与自动化流程

  - **动态刷新（热更新）**：
    - 客户端订阅配置变更，配置发布后可**无需重启**使部分配置实时生效（通常需 `@RefreshScope` 或相应刷新机制；个别资源类参数仍可能需要重建/重启）

  - **多维组织与环境隔离**：
    - 通过 **Namespace / Group / Data ID** 管理配置，dev/test/prod 等环境相互隔离；可结合标签/元数据做更细粒度划分

  - **版本管理与回滚**：
    - 每次发布形成历史版本，出现问题可一键回滚到稳定版本，降低变更风险

  - **灰度发布**：
    - 支持按实例/集群/标签**定向下发**配置，验证无误后再全量发布，提升发布安全性与可控性

  - **权限与审计**：
    - 支持权限控制与变更审计，记录“谁在何时改了什么”，满足合规与追溯需求

  - **客户端容灾**：
    - 客户端保留**本地快照/最后一次有效配置**，在短时网络异常或服务端不可达时仍可按既往配置运行（降级）




##### 动态刷新的原理:长轮询

1. **首次拉取**
   - 客户端（微服务）启动后，从 Nacos Server 拉取**自己关心的配置项**（按 namespace/group/dataId），并写入本地缓存与快照
   - 若首启时服务端不可达，客户端可优先使用**本地快照（最后一次有效配置）**启动
2. **建立长轮询（或订阅通道）**
   1. 拉取成功后，客户端会发起**监听请求**，这个请求携带当前各配置项的 **MD5 列表（按 *namespace/group/dataId* 维度）**
   2. 服务端比较 MD5：
      - 若有不一致（表示有变更），**立即返回变更的 dataId 列表**；
      - 若一致，则将请求**挂起等待**直到有变更或超时（例如默认约 30s，可配置），随后客户端会**立刻重发**下一次监听请求（形成持续订阅）
   3. 在 Nacos 2.x，通知通道也可能通过 **gRPC** 实现，但语义与长轮询一致：**变更发生 → 客户端被唤醒**
3. **服务端变更**
   - 当你在控制台或 OpenAPI 发布了配置，服务端会记录新版本并**唤醒相关监听**（与该配置匹配的订阅者）
4. **变更响应与增量拉取**
   - 客户端收到**“哪些配置发生了变化”的响应**后，**仅对这些 dataId 再次发起拉取**，获取最新内容
5. **本地更新与再次监听**
   1. 客户端拿到最新配置后，更新内存中的配置值，并触发应用侧的**刷新流程**（如 `@RefreshScope` Bean 重建、`@ConfigurationProperties` 重新绑定等）
   2. 之后立即 **发起新的监听请求**，继续订阅后续的变更（或保持 gRPC 订阅通道）
6. **容灾（快照）**
   - 客户端保留本地**快照文件**，用来兜底。当服务器暂不可达或网络抖动时，可临时使用快照中的“最后一次有效配置”以保证服务可用



#### 如何引入

##### 1. 引入依赖

- 我们先需要明白一点，那就是 nacos 的依赖，是**“客户端型依赖**”，也就是它只提供“客户端”，必须连接**外部服务端**才能发挥作用

- nacos 的依赖有两个，我们按需引入，这里的 **版本号我们不写**，我们**由前文中引入的BOM进行管理**

  - **”服务注册与发现“**依赖：

    - `spring-cloud-starter-alibaba-nacos-discovery`

      ```xml
      <dependency>
          <groupId>com.alibaba.cloud</groupId>
          <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
      </dependency>
      ```

      

  - **”配置管理”**依赖：

    - `spring-cloud-starter-alibaba-nacos-config`

      ```xml
      <dependency>
          <groupId>com.alibaba.cloud</groupId>
          <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
      </dependency>
      ```



##### 2. 安装部署启动 Nacos Server

###### 概述

- 我们前面说了，Nacos 的依赖属于**“客户端型依赖”**，它必须连接**外部服务端**才能发挥作用，所以我们还需要**对服务端进行安装**

- 安装方式这里只说两种，**Docker 安装** 和 **Windows 安装**，因为我目前的电脑是 Windows 哈哈哈



###### 两种模式：单机与集群

- **单机模式**

  > 适合：个人学习、功能开发、本地测试

  - 这是 Nacos 的基础运行模式，所有数据都存在单个 Nacos 实例中

    它 **默认使用内置嵌入式数据库（Apache Derby）**，开箱即用、启动最快

    > 这个是默认行为，不过它也**支持改成外部 MySQL**（改 `application.properties`），便于后续平滑迁移到集群



- **集群模式**

  > 生产环境、准生产环境、任何对可用性有要求的场景

  - 在生产环境中，为了保证高可用，通常会部署至少3个 Nacos 节点组成一个集群

    这些节点之间会同步数据，通常接入一个 **外部、共享的数据库（常用 MySQL）**做统一持久化存储，这是官方生产指引

    > Nacos 2.x 也支持 **集群内置存储（embedded storage）**，无需 MySQL，但需要显式配置，**不作为主流生产做法**



###### Windows 版

- **前提**：
  
  - 确保已经安装好了 Java 8 或 以上的 JDK，并配置好了 `JAVA_HOME` 环境变量
  - 下载一个 Nacos Server，大多数团队现在用 Nacos 2.x，稳妥一点选 `2.2.3` 比较好
  
- **⭐单机模式**

  1. 打开 `cmd` 之后进入 `nacos\bin`
  2. 执行命令

     ```cmd
     # -m standalone 参数明确告诉 Nacos 以单机模式启动
     startup.cmd -m standalone
     ```
     
     > nacos 默认是以集群模式启动的，但是如果没有进行配置集群，是会报错的，
     >
     > 所以在本地开发时，我们需要手动添加 `-m standalone` 参数，**显式** 指定以 **单机模式** 启动
  3. 执行命令后会展示弹窗
  
  4. 启动日志滚动完成后，浏览器中进入上一步弹窗中显式的网址，进入nacos控制台
  
     - **默认用户名**：`nacos`
     - **默认密码**：`nacos`
  
  5. **如何关闭服务**：在同一个 `cmd` 窗口中按 `Ctrl+C`，或者新开一个 `cmd` 窗口进入 `nacos\bin` 目录，执行 `shutdown.cmd`




- **⭐集群模式**

  > 前提：**需要一个可用的 MySQL 数据库 (5.7+ 版本)**

  - **数据库初始化**
    1. 在 MySQL 中创建一个专门给 Nacos 使用的数据库，例如 `nacos_config`
    2. 找到 Nacos 解压目录下的 `conf/nacos-mysql.sql` 文件
    3. 将这个 SQL 脚本在 `nacos_config` 数据库中执行，这会创建 Nacos 所需的全部数据表

  
  
  - **修改 Nacos 配置文件**
  
    所有参与集群的 Nacos 节点都需要修改此文件，将其从默认的嵌入式数据库切换为外部 MySQL
  
    - 打开 `nacos\conf\application.properties`
  
    - 找到 `Count of DB` 部分，取消注释并修改如下配置：
  
      ```properties
      # 开启外部数据库支持
      spring.datasource.platform=mysql
      
      # 配置数据库数量
      db.num=1
      
      # 配置数据库连接信息 (请根据实际情况修改 url, user, password)
      # 注意：连接地址后的参数通常是必需的，尤其是时区和字符集
      db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
      db.user.0=root
      db.password.0=123456
      ```
  
  
  
  - **配置集群节点列表**
  
    在 `nacos\conf` 目录下找到 `cluster.conf.example`,将其重命名为 `cluster.conf`
  
    > `cluster.conf`:**静态列表**。每一个 Nacos 节点启动时，都会读取这个文件, 让 Nacos 服务端彼此“看见”对方
  
    编辑该文件，配置所有节点的 **IP:Port** (每行一个)
  
    > **重要提示**：
    >
    > 在本地模拟集群时，Nacos 内部通信通常要求使用**真实 IP**
    >
    > 请避免使用 `127.0.0.1` 或 `localhost`，请通过 `ipconfig` 查询本机局域网 IP（例如 `192.168.x.x`）
  
    ```conf
    # 格式：IP:端口
    192.168.31.200:8848
    192.168.31.200:8858
    192.168.31.200:8868
    ```
  
  
  
  - **部署多节点实例**
  
    > 即使是集群模式，Nacos 本身也是一个独立的 Java 进程。在 Windows 本地模拟时，我们需要复制 3 份 Nacos 文件夹来代表 3 个服务器
  
    1. 将配置好 `application.properties` 和 `cluster.conf` 的 `nacos` 文件夹完整复制三份
    2. 分别重命名文件夹为 `nacos-8848`, `nacos-8858`, `nacos-8868` 以示区分
    3. **关键步骤**：修改后两个文件夹中 `conf/application.properties` 的 `server.port` 属性，防止端口冲突
       - `nacos-8848` 文件夹：保持 `server.port=8848`
       - `nacos-8858` 文件夹：修改为 `server.port=8858`
       - `nacos-8868` 文件夹：修改为 `server.port=8868`
  
    
  
  - **启动集群**
  
    - 分别进入三个文件夹的 `bin` 目录
  
    - 打开 `cmd`，执行启动命令：
  
      > 注意：此时**不需要** `-m standalone` 参数，因为 Nacos 默认就是以集群模式启动
  
      ```cmd
      startup.cmd
      ```
  
    - 观察三个窗口的日志，当看到 `Nacos started successfully in cluster mode.` 即表示该节点启动成功
  
  
  
  - **验证与使用**
    - 访问任意一个节点的控制台（如 `http://192.168.31.200:8848/nacos`）。
    - 在左侧菜单栏点击 **“集群管理” -> “节点列表”**。
    - 此时应该能看到 3 个节点的信息（IP和端口），且状态栏均显示为 `UP`



###### Docker 版

- 前提：Docker 已安装并启动

- **单机模式**

  - 输入下面的命令。这条命令会**基于镜像创建并启动一个容器**；如果本机**还没有**这个镜像，Docker 会**先自动拉取**它

    ```bash
    docker run -d \
    --name nacos-standalone \
    -e MODE=standalone \
    -p 8848:8848 \
    -p 9848:9848 \
    -p 9849:9849 \
    nacos/nacos-server:v2.2.3
    ```

  - **命令参数解析**：

    - `-e MODE=standalone`：**核心参数**
      - 明确指定以“单机模式”启动。如果不写，Nacos 镜像默认以集群模式启动，在没有配置集群的情况下会报错退出
    - `-p 8848:8848`：主端口，用于 HTTP 协议（控制台页面、OpenAPI）
    - `-p 9848:9848`：**Nacos 2.x 新增**，用于客户端 gRPC 通信（必须映射）
    - `-p 9849:9849`：**Nacos 2.x 新增**，用于节点间 gRPC 通信（必须映射）

    

  - **查看运行状态**：

    - 查看日志：`docker logs -f nacos-standalone`
    - 当看到日志输出 `Nacos started successfully in stand alone mode` 时，即表示启动成功。
    - 访问地址：`http://localhost:8848/nacos` (账号密码默认均为 `nacos`)



- **集群模式**
  - 前提：准备好一个可用的 **MySQL (5.7+)** 数据库（可以是容器内的，也可以是外部的），并已导入 `nacos-mysql.sql`



#### 如何使用

⭐按照之前所说，先引入依赖

##### A. 服务注册与发现

**核心目标**：通过配置和注解，使 Spring Boot 应用在启动时自动向 Nacos Server 发起注册请求，上报自身的 IP、端口和服务名称

###### **1.修改配置文件 (`application.yml`)**

Nacos 客户端必须知道两件事：**我是谁（服务名）** 和 **我要向谁汇报（服务端地址）**

```yaml
server:
  port: 8081 # 当前服务的端口

spring:
  application:
    name: order-service # 【关键】服务名称。这是 Nacos 区分不同微服务的唯一标识
  cloud:
    nacos:
      discovery:
        # Nacos Server 地址
        # 如果是单机，写 IP:8848
        # 如果是集群，写 IP1:8848,IP2:8858,IP3:8868
        server-addr: 127.0.0.1:8848
```



###### **2.开启客户端功能**

在 Spring Boot 的启动类上添加注解，显式声明开启服务发现功能

> *注：在 Spring Cloud Alibaba 的部分新版本中，此注解已变为可选（Auto-Configuration 会自动生效），但建议显式添加以保证语义清晰*

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient // 【关键】开启服务发现与注册客户端
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```



###### **3.启动与验证**

1. 启动该 Spring Boot 项目
2. 观察 IDEA 控制台日志，若出现 `[REGISTER-SERVICE] ... register service order-service success` 字样，说明客户端注册逻辑执行成功
3. 打开浏览器访问 Nacos 控制台（如 `http://127.0.0.1:8848/nacos`）
4. 点击左侧菜单 **“服务管理” -> “服务列表”**
5. 若列表中出现了 `order-service`，且 **“实例数”** 显示为 1，**“健康实例数”** 显示为 1，即表示注册成功



###### **4.服务发现 (即：服务间调用)**

当服务注册成功后，其他微服务可以通过 Nacos 获取该服务的 IP 列表进行调用。 *(此处仅展示最基础的 RestTemplate 整合方式)*

```java
@Bean
@LoadBalanced // 【关键】让 RestTemplate 具备通过“服务名”查找 IP 的能力（集成 Ribbon/LoadBalancer）
public RestTemplate restTemplate() {
    return new RestTemplate();
}

// 使用代码：
// 直接使用服务名 "order-service" 代替具体的 "IP:端口"
// restTemplate.getForObject("http://order-service/orders/1", String.class);
```



##### B. 动态配置管理

**核心目标**：将微服务的配置（如数据库连接、业务开关等）从本地代码中剥离，托管到 Nacos Server，并实现配置修改后服务 **无需重启即可实时生效**（热更新）



###### **a. 前置准备 (版本说明)**

Spring Boot 2.4+ 改变了配置文件的加载机制。针对不同版本，我们有两种主流的配置方式：

- **方案 A (Bootstrap 模式)**：经典、稳健，逻辑清晰，适合习惯旧版本 Spring Cloud 的用户（**推荐**）
- **方案 B (Import 模式)**：SpringBoot 2.4+ 原生支持，无需额外依赖，配置更简洁



###### **b. 配置文件设置 (二选一)**

**👉 方案 A：经典 Bootstrap 模式 (推荐)**

1. **引入依赖**：在 `pom.xml` 中引入 bootstrap 支持

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-bootstrap</artifactId>
   </dependency>
   ```

   

2. **创建 `bootstrap.yml`**：在 `src/main/resources` 下新建

   ```yaml
   spring:
     application:
       name: order-service     # 【核心】服务名 -> Data ID 前缀
     profiles:
       active: dev             # 【核心】环境名 -> Data ID 后缀
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848
           file-extension: yaml
   ```



**👉 方案 B：新潮 Import 模式 (SpringBoot 2.4+ 原生)**

1. **无需引入** `spring-cloud-starter-bootstrap` 依赖

2. **无需创建** `bootstrap.yml` 文件

3. **直接修改 `application.yml`**： 使用 `spring.config.import` 语法直接导入 Nacos 配置源

   ```yaml
   spring:
     application:
       name: order-service
     profiles:
       active: dev
     cloud:
       nacos:
         config:
           server-addr: 127.0.0.1:8848
           file-extension: yaml
     # 【核心变化】在这里直接导入 Nacos 配置
     config:
       import:
       # 这里是直接明确指定了 Data ID (见后文⭐)
         # 写法 1：完全引用变量（适合理解原理）
         - optional:nacos:${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
         
         # 写法 2：生产推荐（后缀直接写死，更稳定）
         # - optional:nacos:${spring.application.name}-${spring.profiles.active}.yaml
   ```



###### **c. 在 Nacos 控制台创建配置**

无论使用哪种方案，Nacos 客户端最终都会去服务端寻找 **Data ID**

**1. Data ID 匹配公式 (黄金法则)**

Nacos 客户端在启动时，会把本地配置文件中的 **三个关键属性** 拼凑在一起，形成一个文件名（Data ID），然后去 Nacos 服务端找这个文件

> **公式**：`${spring.application.name}`-`${spring.profiles.active}`.`${file-extension}`

**举例推导**：

- `spring.application.name` = `order-service`
- `spring.profiles.active` = `dev`
- `file-extension` = `yaml`
- **最终匹配的 Data ID** = `order-service-dev.yaml`



**2. 创建步骤**

1. 进入 Nacos 控制台 -> **配置管理** -> **配置列表**

2. 确保顶部的 **命名空间** 选对了（与配置文件保持一致）

3. 点击右上角 **“+”** 号：

   - **Data ID**: `order-service-dev.yaml`

   - **Group**: `DEFAULT_GROUP` (保持默认)

   - **配置内容**:

     ```yaml
     order:
       timeout: 3000
       desc: "这是远程 Nacos 的配置"
     ```

4. 点击 **“发布”**



###### **d. 代码读取与热更新 (两种方式)**

**方式一：@Value + @RefreshScope (简单直观)**

> **适用场景**：测试 Demo、配置项极少（1-2个）、或者非结构化的简单配置

这种方式就像我们在 `application.yml` 里读取配置一样，只是多加了一个注解来实现“热更新”

```java
@RestController
@RefreshScope // 【核心】动态刷新开关
// 原理：当 Nacos 配置变更时，Spring 会销毁当前 Bean，并在下次请求时重新创建，从而读取到最新值
public class OrderConfigController {

    // 1. 基础用法
    @Value("${order.timeout}")
    private String orderTimeout;

    // 2. 进阶用法：设置默认值
    // 格式：${key:default_value}
    // 作用：如果 Nacos 上没配这个 key，应用也不会报错启动失败，而是使用冒号后的默认值
    @Value("${order.desc:这是默认描述信息}") 
    private String desc;

    @GetMapping("/config/info")
    public String getConfig() {
        return "超时时间: " + orderTimeout + ", 描述: " + desc;
    }
}
```



**方式二：@ConfigurationProperties (生产推荐 ⭐️)**

> **适用场景**：企业级开发、配置项较多、需要分组管理、强类型检查

这种方式将一组配置（如 `order.timeout`, `order.desc`, `order.max-count`）映射为一个 Java 对象

**第一步：定义配置类 (Properties Bean)**

```java
@Component
@ConfigurationProperties(prefix = "order") // 【核心】自动匹配配置文件中以 "order" 开头的属性
@RefreshScope // 【核心】赋予该 Bean 动态刷新能力
@Data // 使用 Lombok 自动生成 Getter/Setter (必须要有 Setter 才能注入值)
public class OrderProperties {
    
    // 对应 order.timeout
    // 优势：自动进行类型转换 (String -> Integer)
    private Integer timeout;
    
    // 对应 order.desc
    private String desc;
    
    // 对应 order.auto-confirm (驼峰命名自动匹配 kebab-case)
    private Boolean autoConfirm;
}
```



**第二步：在业务代码中使用**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class OrderController {

    // 直接注入上面的配置对象
    @Autowired
    private OrderProperties orderProperties;

    @GetMapping("/order/check")
    public String checkOrder() {
        // 使用对象 get 方法获取配置，代码更整洁
        return "当前超时设置：" + orderProperties.getTimeout() + 
               "，自动确认：" + orderProperties.getAutoConfirm();
    }
}
```





## 服务调用

- 这里讨论的服务调用特指 **“同步调用”**（关于 **“异步调用”**，见后文“消息队列”章节）

- **核心认知**：无论是何种类型的调用，微服务的调用 **一定是通过”网络"进行调用的** ，这区别于我们以前的单体架构的那种代码风格

  > 在单体应用中（比如一个巨大的 Spring Boot 项目），当你需要调用另一个模块的功能时，你只需要引入对象，之后用对象直接调用就行
  >
  > 比如：
  >
  > ```java
  > @Autowired
  > private UserService userService;
  > 
  > ......
  >     
  > userService.getUserById(1);
  > ```
  >
  > 但是微服务中，`OrderService`（订单服务）和 `UserService`（用户服务）可能部署在不同的机器里，它们不仅内存不共享，甚至可能相隔十万八千里
  >
  > 你就需要使用一系列的手段，通过网络，进行调用另一个服务中的东西，麻烦死了，这里就不给示例了
  >
  > 其实本质上就是 **写一段代码，模拟浏览器发送一个 HTTP 请求** 嘛

  但是用原生方式进行网络调用实在太麻烦了，为了简化开发，屏蔽底层繁琐的 HTTP 通信细节，Spring Cloud 引入了 OpenFeign 等组件，方便我们开发



### OpenFeign

> Feign 读音: [feɪn]

#### 核心定义

OpenFeign 是一个 **声明式** 的 Web Service 客户端。它的主要作用是简化 HTTP API 客户端的开发过程

- OpenFeign 能够让你像调用本地 Java 方法一样，调用远程的 HTTP API

  > **补充理解**：你只需要定义一个接口，并在上面加点注解告诉它“发给谁”和“发什么”，剩下的脏活累活（建连接、拼参数、解析结果）OpenFeign 全包了

- OpenFeign还默认集成了 **客户端负载均衡器**（Spring Cloud LoadBalancer），它还会默认的帮你实现负载均衡



#### 快速入门

在 Spring Cloud 环境下，集成 OpenFeign 非常简单，核心流程只需要三步：**引入依赖 -> 开启注解 -> 定义接口**



##### 1. 引入依赖

在项目的 `pom.xml` 中添加 OpenFeign 的起步依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

如果后面报错显式没有loadbalancer，就把这个也加上，默认其实是已经集成了的

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```



##### 2. 开启功能

在 Spring Boot 的 **启动类** 上添加 `@EnableFeignClients` 注解

- 这个注解的作用是告诉 Spring 容器：“启动时请扫描项目中所有的 `@FeignClient` 接口，并为它们创建代理对象”

```java
@SpringBootApplication
@EnableFeignClients // <--- 关键注解：开启 Feign 扫描
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```



##### 3. 定义客户端接口 (Interface)

创建一个 Java 接口，并使用 `@FeignClient` 注解标记它

```java
// name: 指定要调用的远程服务名称（对应注册中心，如 Nacos/Eureka 里的服务名）
// 注意：不需要写 http://localhost:8080，OpenFeign 会自动通过负载均衡查找服务 IP
@FeignClient(name = "user-service") 
public interface UserClient {

    // 声明要调用的具体 API 路径和方法
    // 这里的 @GetMapping 是 Spring MVC 的标准注解，完全兼容
    @GetMapping("/users/{id}")
    UserDTO getUserById(@PathVariable("id") Long id);
}
```

- 关于这里我有几点要说：

  - OpenFeign 是通过 HTTP 协议沟通的，而不是 Java 语言沟通的，不一定要和另一个服务中的方法签名完全相同，

    有很多注意点，这里详见后面的笔记“关于定义接口”



##### 4. 在业务中使用

完成以上三步后，OpenFeign 已经在 Spring 容器中生成了 `UserClient` 的代理 Bean

你可以在任何 Service 或 Controller 中通过 `@Autowired` 注入并直接使用

```java
@Service
public class OrderService {

    @Autowired
    private UserClient userClient; // 注入接口

    public void createOrder(Long userId) {
        // 像调用本地方法一样调用远程服务
        UserDTO user = userClient.getUserById(userId);
        
        System.out.println("查询到的用户信息: " + user.getName());
    }
}
```







#### OpenFeign接口的定义与管理

##### 1. 核心原则：HTTP 契约 > Java 语法

OpenFeign 只关心 HTTP 协议层面的“对上”，不关心 Java 语法层面的“对上”



###### a. 必须“一模一样”的地方 (HTTP 契约)

这些地方如果不对，请求一定发不出去，或者服务端收不到参数

1. **URL 路径**：`@GetMapping("/users/{id}")` 里的路径必须和服务端完全一致
2. **HTTP 方法**：服务端是 POST，你就必须用 `@PostMapping`
3. **参数注解的 Key**：
   - **错误写法**：`@RequestParam String name` （在 OpenFeign 中，必须指定别名！）
   - **正确写法**：`@RequestParam("name") String name`
   - **原因**：Java 编译后可能会丢失参数名信息，OpenFeign 需要你显式告诉它这个参数在 HTTP URL 里叫什么名字



###### b. 建议“保持一致”的地方 (最佳实践)

虽然技术上允许不一样，但为了 **可读性** 和 **维护性** ，我们强烈建议保持一致

1. **Java 方法名**：
   - 服务端：`public User queryUserById(Long id)`
   - 客户端：`public UserDTO getUser(Long id)` (技术上可行)
   - **最佳实践**：客户端最好也叫 `queryUserById`。这样排查日志时，两边能迅速对应上
2. **DTO 类名**：
   - 服务端：`UserEntity`
   - 客户端：`UserDTO`
   - **最佳实践**：类名可以不同（因为服务端通常混用数据库实体，客户端只想要数据对象），但 **字段名** （JSON key）必须完全一致



##### 2. 避坑

###### 陷阱一：GET 请求变成了 POST?

**场景**：你要调用一个 GET 接口，参数是对象

```java
// 错误写法
@GetMapping("/users/search")
User findUser(UserSearchDTO searchParam); 
```

- **结果**：OpenFeign 会强制把这个请求变成 **POST**！

- **原因**：
  - 如果参数没有加注解，OpenFeign 默认把它当作 Request Body 处理，而 GET 请求通常不带 Body
  - **解决**：加上 `@SpringQueryMap`（如果是对象）或者用 `@RequestParam`



###### 陷阱二：路径参数报错

**场景**：

```java
// 错误写法
@GetMapping("/users/{id}")
User getUser(@PathVariable Long id);
```

- **结果**：
  - 报错 `PathVariable annotation was empty on param 0`
- **解决**：必须指定 value，即 `@PathVariable("id")`。Spring MVC Controller 里或许有时可以省，但 OpenFeign 里 **别省**



###### 陷阱三：返回值接不住

- **场景**：
  - 服务端返回了 `{ "data": { "name": "..." }, "code": 200 }`，但你的接口返回值定义的是 `UserDTO`（直接对应 data 内部结构）
  - **结果**：反序列化失败，或者字段全为 null
- **解决**：看清服务端返回的最外层结构。如果服务端包了一层 `Result<T>`，你的 Feign 接口返回值也必须是 `Result<UserDTO>`



##### 一些最佳实践总结

针对你问的“怎么写最好”，以下是经过实战检验的标准姿势：

1. **复制粘贴是美德**： 
   - 不要凭空手写 Feign 接口
   - 打开服务端的 Controller 代码，把方法签名复制过来，然后把复杂的实体类改成你的 DTO，把 `@RequestBody` 等注解保留
2. **注解显式化**： 
   - 永远明确写出 `@RequestParam("xx")` 和 `@PathVariable("xx")` 中的名字，不要依赖编译器的参数名推断
3. **保持简洁**： 
   - Feign 接口只保留需要的字段。如果服务端返回一个包含 50 个字段的 User 对象，而你只需要 `name` 和 `phone`，你的 DTO 里就只写这两个字段。这不会报错，反而能减少内存消耗



##### ※ 接口管理策略

###### 1. 核心问题

在大型微服务架构中，会有几十甚至上百个服务互相调用。比如 `Order-Service` 需要调用 `User-Service`

- **难道 `UserClient` 这个 Feign 接口代码，每个服务都要写一个一模一样的，然后如果原来的逻辑变了之后，再每一个都修改？**
  - 对于这个接口定义的问题，其实业界主要有两种解决方案：
    1. **调用方自己写**
    2. **服务方提供 Jar 包**

###### 2. 模式一：调用方自己写

这是最原始、也最松散的模式



**a. 工作流程**

1. **用户服务** 开发人员写好 Controller 接口，并发布 Swagger 文档
2. **订单服务** 开发人员打开文档，把需要的接口（如 `getUserById`）手动抄写到自己的项目里，定义为 Feign 接口
3. **支付服务** 开发人员也打开文档，把自己需要的接口手动抄写一遍



**b.优缺点分析**

| 维度     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| **优点** | **完全解耦**。调用方和服务方互不依赖，服务方随便改代码（只要 核心调用点 不变），调用方不用升级 Jar 包 |
| **优点** | **按需定义**。服务方有 100 个接口，我只需要其中 2 个，我就只写这两个，清爽干净 |
| **缺点** | **重复劳动**。如果有 10 个服务都要查用户，这段代码就被抄了 10 遍 |
| **缺点** | **维护成本高**。一旦服务方修改了 URL 或参数名，这 10 个调用方必须一个个去改代码，容易漏改导致线上故障 |



###### 3. 模式二：发布方写公共 API 模块

这是目前大型互联网公司（如阿里、美团等）最主流的做法

**核心理念**：即使项目文件夹不在一起，通过 **Maven 私服** 将它们连接起来



**a. 工程结构设计**

服务方（User-Service）的项目结构不再是一个单体模块，而是拆分为 **父工程 + 子模块**

```
user-service-project (父工程)
├── user-api (子模块 / JAR)
│   ├── com.example.user.api.UserFeignClient.java  (Feign 接口定义)
│   └── com.example.user.dto.UserDTO.java          (传输对象定义)
│   └── (这个模块会被推送到公司 Maven 私服)
|
└── user-server (子模块 / Application)
    ├── com.example.user.controller.UserController.java (Controller 实现)
    └── com.example.user.service.UserService.java       (业务逻辑)
```

- 注意，调用方是没有这个 `user-api` 模块的，他们只能通过 maven 引入，调用方他们只有他们自己的 `api` 模块，比如 `order-api`



**b. 跨仓库协作流程**

假设 `User-Service` 和 `Order-Service` 是两个完全独立的 Git 仓库，物理隔离。它们通过 **Maven 私服** 联系

- **核心流向图：**

  ```
  [User-Service 开发者]                 [Order-Service 开发者]
         |                                       |
     (git push)                              (git pull)
         ↓                                       ↓
  [Git 仓库 A (源码)]                     [Git 仓库 B (源码)]
         |                                       |
         |                                 (读取 pom.xml 依赖)  
         ↓                                       ↓
  [ 公司 Maven 私服]  <-------- (自动下载 Jar 包)
         ↑
  (存放 user-api-1.0.jar)
  ```



- **详细步骤：**

  1. **发布方动作**：

     - `User-Service` 开发完接口后，

       将编译好的 `user-api.jar` 被上传到公司的 Maven 私服

       - 注意：源码依然在 Git 里，私服里只有编译后的 class 文件

  2. **消费方动作**：

     - `Order-Service` 不需要知道对方源码在哪里，只需要在 `pom.xml` 里写上坐标
     - **结果**：Maven 自动从私服下载 jar 包到本地仓库，你的代码就可以 `import` 对方的类了

     ```xml
     <!-- 订单服务 (Git 仓库 B) 的 pom.xml -->
     <dependency>
         <groupId>com.example</groupId>
         <artifactId>user-api</artifactId> <!-- 引用的是 Nexus 里的 jar -->
         <version>1.0.0</version>
     </dependency>
     ```



**c. API 变更与版本迭代**

- 当**用户服务**新增了接口（例如 `updateUser`）或修改了 DTO 字段时，**必须**遵循以下更新流程：
  1. **服务端升级版本**： 在 `user-api` 模块中修改代码后，必须修改 `pom.xml` 中的版本号
     - *开发阶段*：使用快照版，如 `1.0.1-SNAPSHOT`（允许频繁覆盖）
     - *发布阶段*：使用正式版，如 `1.0.1`（Maven 私服通常禁止覆盖已存在的正式版）
  2. **服务端重新发布**： 再次执行 `mvn deploy`。这会将新版本 `user-api-1.0.1.jar` 推送到私服
  3. **客户端升级依赖**： `Order-Service` 的开发人员收到通知，在自己的 `pom.xml` 中将版本号修改为 `1.0.1`，Maven 会自动下载新包

> **注意**：如果服务端只改了代码却 **没有** 改版本号（且不是 SNAPSHOT），客户端 Maven 可能会因为本地缓存而拉取不到最新的代码





**c. 优缺点分析**

| 维度     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| **优点** | **开发效率极高**。调用方通过 Maven 引入即可使用，一行代码都不用写 |
| **优点** | **契约统一**。所有调用方使用的都是同一份代码，避免了手动抄写错误（如参数名拼错） |
| **缺点** | **强耦合**。调用方强依赖了服务方的 Jar 包                    |
| **风险** | **依赖冲突**。如果 `user-api` 引用了 `fastjson 1.0`，而调用方项目用的是 `fastjson 2.0`，可能会发生 Jar 包冲突 |



#### 连接池性能优化

##### 1. 为什么默认配置性能差？

在未做任何配置时，OpenFeign 使用的是 JDK 原生的 `java.net.HttpURLConnection` 来发送 HTTP 请求

###### a. 短连接机制

- **HttpURLConnection** 是基于“短连接”的
  - 这意味着每一次 HTTP 请求（比如调用一次 `getUserById`），都会完整地经历以下过程：
    - 建立 TCP 连接（三次握手） 🤝
    - 传输数据 📦
    - 断开 TCP 连接（四次挥手） 👋

###### b. 性能瓶颈

- 在高并发场景下（例如每秒几千次调用），这种机制会带来两个严重问题：
  1. **时间损耗**：TCP 握手和挥手的时间可能比实际传输数据的时间还要长
  2. **资源耗尽**：频繁创建和销毁连接会占用大量操作系统端口，甚至导致“端口耗尽”错误



##### 2. 解决方案：引入连接池

为了解决上述问题，我们需要将底层的 HTTP 客户端替换为支持 **连接池（Connection Pool）** 的组件



**推荐组件**

业界主流的两个替代方案是：

- **Apache HttpClient (HttpClient 5)**：老牌、稳定、功能丰富
- **OkHttp**：轻量级、性能优异，Android 和微服务中都很常用

**连接池的好处**：

- **长连接**：建立一次 TCP 连接后不立即断开，后续请求复用该连接
- **减少开销**：省去了握手和挥手的时间，显著降低延迟



##### 3. 优化步骤

下面以 **Apache HttpClient 5** 为例，演示如何替换默认客户端

###### 1. 引入依赖

在 `pom.xml` 中添加 `feign-hc5` 依赖。这个 jar 包的作用是把 OpenFeign 的请求转发给 Apache HttpClient 执行

```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-hc5</artifactId>
</dependency>
```



###### 2. 修改 YAML 配置

在 `application.yml` 中开启 HttpClient 功能

```yaml
# 方案 A：Spring Boot 3.x + Spring Cloud 2022.x (新版标准)
# 专门针对 HttpClient 5 的配置
spring:
  cloud:
    openfeign:
      httpclient:
        hc5:
          enabled: true

# 方案 B：Spring Boot 2.x 或 编译器报错时的通用配置
# 这是旧版本或通用的全局开关
feign:
  httpclient:
    enabled: true
```

> **注意**：方案 B (`feign.httpclient.enabled: true`) 在很多版本中是通用的。如果你的项目中引入了 `feign-hc5` 包且配置了该项，OpenFeign 通常也会自动识别并启用 HttpClient



###### 3. 连接池参数调优 (可选)

虽然开启 `enabled: true` 后已经有了默认连接池，但在高并发下通常需要调整默认参数（默认值通常较小）

你可以在 YAML 中直接配置（Spring Cloud 2022+ 新版支持），或者通过 Java Config 配置 `CloseableHttpClient` Bean

**YAML 配置示例（简单版）：**

```yaml
feign:
  httpclient:
  	enabled: true
    max-connections: 200           # 连接池最大连接数（默认可能只有 50，建议调大）
    max-connections-per-route: 50  # 每个路由（目标域名）的最大连接数
```

> - `max-connections`：整个连接池最多能存多少个连接（比如你同时调 10 个不同的服务，总共加起来不能超过这个数）
> - `max-connections-per-route`：对 **同一个服务**（例如 `user-service`）最多能同时建立多少个连接



##### 4. 验证是否生效

如何确认 OpenFeign 真的切换到了 HttpClient？

1. **看日志**：启动时，Spring Boot 的控制台通常会打印 `HttpClient` 相关的 Bean 初始化日志
2. **Debug**：在代码中断点调试 `FeignClient` 的调用处，深入查看 `Client` 接口的实现类
   - **优化前**：实现类是 `Client.Default`
   - **优化后**：实现类是 `ApacheHttp5Client`



#### 请求拦截器

> 核心作用：统一参数处理

在微服务调用链中，我们经常需要把上游服务的一些“隐形参数”传递给下游，最典型的就是 **Token（身份认证信息）** 或者 **TraceId（链路追踪 ID）**

- 如果不配置拦截器，每次调用 Feign 接口时，你都得手动在方法参数里加一个 `@RequestHeader("Authorization") String token`，这太蠢了



##### a. 核心原理

OpenFeign 提供了一个 `RequestInterceptor` 接口

- 只要你 **实现了这个接口并注册为 Bean** ，OpenFeign 在发送每一个 HTTP 请求之前，都会先执行这个接口的 `apply` 方法
- `RequestInterceptor` 的执行时机位于 **Feign 接口被调用之后**，**HTTP 客户端（如 HttpClient/OkHttp）发出请求之前**



##### b. 核心对象详解：RequestTemplate

在 `RequestInterceptor` 接口中，核心参数 `RequestTemplate` 是 OpenFeign 对 HTTP 请求的 **可变（Mutable）抽象模型**

- 它是 OpenFeign 准备发送请求时的 **数据容器**

  当 OpenFeign 解析完接口上的注解后，会将生成的请求信息（URL、Header、Body）全部装入这个对象，并交给拦截器

  - 通过操作该对象，你可以在请求正式发出前，**动态注入 Header、重写 URL 或修正 Body**，从而实现对最终网络请求报文的精确控制



**1. 常用方法速查表**

| 方法签名                     | 核心作用        | 场景示例                                 | 注意事项                                                     |
| ---------------------------- | --------------- | ---------------------------------------- | ------------------------------------------------------------ |
| **`header(k, v...)`**        | **追加** 请求头 | `template.header("Token", "123")`        | 如果 Key 已存在，**不会覆盖**，而是追加（形成 `Key: [v1, v2]`） |
| **`replaceHeader(k, v...)`** | **覆盖** 请求头 | `template.replaceHeader("Token", "new")` | 如果想彻底替换原有的 Header，必须用这个，否则会重叠          |
| **`query(k, v...)`**         | 追加 URL 参数   | `template.query("ts", "16888")`          | 这里的参数会被 OpenFeign 自动进行 URL 编码                   |
| **`uri(path)`**              | 修改/追加路径   | `template.uri("/v2" + template.path())`  | 可以用来动态改变请求的 API 版本前缀                          |
| **`body(byte[], charset)`**  | 修改请求体      | `template.body(jsonBytes, UTF_8)`        | 仅在 POST/PUT 等包含 Body 的请求中有效。常用于统一加密 Body  |
| **`method()`**               | 获取请求方法    | `template.method()`                      | 返回 String，如 "GET"、"POST"。常用于判断是否需要加 Body     |



**2. 进阶操作与避坑**

- **Header 的追加与覆盖**

  - `header("A", "1")` + `header("A", "2")` -> 结果：`A: 1, 2` (多值)
  - `header("A", "1")` + `replaceHeader("A", "2")` -> 结果：`A: 2` (单值)
  - **最佳实践**：鉴权 Token 通常建议用 `header`（防止覆盖了某些框架自带的 Token），除非你明确知道要重置它

  

- **Query 参数的编码问题**

  - `template.query("name", "张三")` -> 实际发出：`name=%E5%BC%A0%E4%B8%89`
  - OpenFeign 默认会处理编码。如果你传入的已经是编码过的字符串，可能会被**二次编码**，导致服务端解码乱码。请传入原始字符串

  

- **Body 的处理**

  - `template.body()` 接收的是 `byte[]` 或 `String`

  - **注意**：

    - 一旦你在拦截器里调用了 `template.body(...)`，就会覆盖掉原本由 `@RequestBody` 注解生成的 JSON 数据

      这通常用于 **全链路加密** 场景（把原本的明文 JSON 替换为加密后的字节流）



**3. 代码示例：全能型修改**

```java
@Override
public void apply(RequestTemplate template) {
    // 1. 动态追加路径前缀 (如 /users/1 -> /api/v1/users/1)
    if (!template.path().startsWith("/api")) {
        template.uri("/api/v1" + template.path());
    }

    // 2. 强制覆盖 Content-Type (防止上游传错)
    template.replaceHeader("Content-Type", "application/json;charset=UTF-8");

    // 3. 只有 POST 请求才加特定的签名参数
    if ("POST".equalsIgnoreCase(template.method()) && template.body() != null) {
         String bodyStr = new String(template.body(), StandardCharsets.UTF_8);
         String sign = SignUtil.sign(bodyStr); // 对 Body 签名
         template.query("sign", sign);
    }
}
```



##### 2. 实战场景：Token 传递

这是开发中最常见的需求：A 服务调用 B 服务，需要把当前用户的 JWT Token 透传过去，否则 B 服务会报 401 未授权

**代码实现：**

```java
@Configuration
public class FeignConfig {

    @Bean
    public RequestInterceptor requestInterceptor() {
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate template) {
                // 1. 获取当前请求的上下文（即谁调用的我）
                	//这个不认识的类的作用：SpringMVC提供的可在任何地方帮你拿到当前线程的 HTTP 请求对象的类
                ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                
                if (attributes != null) {
                    HttpServletRequest request = attributes.getRequest();
                    // 2. 从 Header 中提取 Authorization
                    String token = request.getHeader("Authorization");
                    
                    // 3. 如果有 Token，就塞入 Feign 的请求头中
                    if (token != null) {
                        // template 就是 Feign 即将发出的请求
                        template.header("Authorization", token); //⭐⭐重点
                    }
                }
            }
        };
    }
}
```

> **注意**：
>
> - 这里使用了 `RequestContextHolder`，所以必须在 Web 环境下（也就是主线程中）才能获取到 Header
> - 如果你在子线程或异步线程中调用 Feign，这种写法会拿不到 Token，需要做额外的上下文传递处理



##### 3. 自定义鉴权逻辑

有时候不仅仅是传递 Token，你可能需要给所有的请求加上一个统一的“暗号”或者 API Key，防止外部直接攻击内部接口

```java
@Component
public class CustomAuthInterceptor implements RequestInterceptor {
    @Override
    public void apply(RequestTemplate template) {
        // 给所有发出的请求加一个固定的内部密钥
        template.header("X-Internal-Secret", "my-super-secret-password");
        // 或者统一加时间戳
        template.query("timestamp", String.valueOf(System.currentTimeMillis()));
    }
}
```



#### 日志与超时

##### 1. 为什么需要日志配置

默认情况下，OpenFeign 是“沉默”的

哪怕你请求失败了，或者参数传错了，控制台也不会打印具体的 HTTP 请求细节（比如 URL 是什么、Header 里有没有 Token）

为了能看清 OpenFeign 到底发了什么，我们需要 **手动开启日志**



##### **2. 四个日志级别 (Logger.Level)**

OpenFeign 定义了四种详细程度，由 `feign.Logger.Level` 枚举控制。你需要根据不同的环境选择合适的级别

| 级别        | 描述                                         | 适用场景                                     |
| ----------- | -------------------------------------------- | -------------------------------------------- |
| **NONE**    | 不记录任何日志（**默认值**）                 | **生产环境** (追求极致性能，减少 IO 开销)    |
| **BASIC**   | 仅记录请求方法、URL、响应状态码及执行时间    | **生产环境推荐** (仅用于监控慢请求)          |
| **HEADERS** | 在 BASIC 基础上，额外记录请求和响应的 Header | 调试 Token、Cookie 或自定义头缺失问题        |
| **FULL**    | 记录请求和响应的 Header、Body 和元数据       | **开发环境推荐** (调试 Bug 时看参数和响应体) |



##### 3. 核心原理：双层控制机制

要让日志显示在控制台，必须 **同时满足** 以下两个维度的配置。

OpenFeign 的日志输出依赖于底层日志框架（SLF4J）的 **DEBUG 级别**，因此存在两层控制：

**a. 第一层：OpenFeign `Logger.Level`（决定生成日志的详细程度）**

- 该配置控制 OpenFeign 在运行时是否记录请求细节（如 URL、Header、Body）
- **关键点**：OpenFeign 记录的所有日志，在底层都会被标记为 `DEBUG` 级别



**b. 第二层：Spring Boot `logging.level`（决定是否输出日志）**

- 该配置控制 Spring Boot（SLF4J/Logback）允许输出的日志最低级别
- Spring Boot 默认级别是 `INFO`，会拦截并丢弃所有 `DEBUG` 级别的消息
- **结论**：即使 OpenFeign 生成了详细日志（FULL），如果 Spring Boot 的日志级别未放行 `DEBUG`，控制台依然什么都不会显示



##### 4. 配置步骤

###### 第一步：设置日志详细度

你需要向 Spring 容器注入一个 `Logger.Level` Bean，明确指定 OpenFeign 生成日志的详细程度

```java
@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel() {
        // 开发环境建议用 FULL，看清楚每一个字段，生产环境不建议用，会消耗大量 CPU 和 磁盘 IO，导致吞吐量急剧下降
        return Logger.Level.FULL; 
    }

}
```



###### 第二步：开启 DEBUG 输出

你需要显式配置 Spring Boot 日志系统，将 **OpenFeign 接口所在包** 的日志级别设置为 `DEBUG`，以允许底层输出

```yaml
logging:
  level:
    # ⚠️ 核心坑点：必须精确到 Feign 接口所在的包名
    # 假设你的 UserClient 定义在 com.example.project.client 包下
    com.example.project.client: DEBUG
```

> **检查方法**：启动项目发起调用。如果控制台看到了类似 `[UserClient#getUser] ---> GET http://...` 的日志，说明配置成功



## API网关

### Gateway

#### 0. 为什么需要网关?

在微服务架构中，后端服务往往会被拆分成多个微小的服务（如订单服务、用户服务、库存服务）。如果没有网关，客户端（App、Web、小程序）需要直接与各个微服务进行交互，会产生严重的耦合与安全隐患

网关主要解决以下四个核心问题：

- **统一接入**
  - **问题**：后端服务（如订单、用户、库存）的IP和端口可能动态变化，且数量众多
  - **解决**：客户端只需连接唯一的网关地址，由网关根据路由规则反向代理到具体的后端服务
- **解耦与抽象**
  - **问题**：客户端（Web/Mobile）往往需要根据业务聚合多个服务的数据，直接调用会导致客户端逻辑复杂
  - **解决**：网关屏蔽了后端微服务的拆分细节，客户端无需感知微服务的具体存在
- **横切关注点统一处理**
  - **问题**：鉴权（Auth）、限流（Rate Limiting）、日志（Logging）、跨域（CORS）是每个服务都需要的通用逻辑。如果在每个微服务中重复实现，会导致代码冗余且维护困难
  - **解决**：将这些逻辑下沉到网关层统一实现（AOP思想），微服务只需专注于业务逻辑
- **安全防护**
  - **问题**：核心业务接口直接暴露在公网极其危险
  - **解决**：网关作为隔离层，隐藏内部服务架构，仅暴露必要的API

**Spring Cloud Gateway** 的出现就是为了解决这些问题。它是整个微服务系统的 **统一入口** ，负责接收所有请求，并根据规则 **转发** 到目标服务



#### 1. 转发的概念

前文中，我们提到了 **转发** 。在这里，我们必须先明确一个词语的概念： **转发**

- 在 Gateway 的语境下，它就是一个实实在在的 **HTTP 请求中转** 过程
- 定义：
  - **转发 (Forwarding)**，其实就是网关（Gateway）作为 **中间人**，收到你的请求后，**代替你** 向后端的微服务发起一个新的请求，拿到数据后再 **转交** 给你的过程
    - 其实在技术术语中，这叫做 **反向代理 (Reverse Proxy)**



#### 2. 架构与技术栈

Spring Cloud Gateway 是 Spring 官方推出的第二代网关框架，其底层架构与传统Servlet容器有本质区别



##### a. 核心技术栈组成

Spring Cloud Gateway 的高性能源于其底层的 **响应式编程**模型。它的技术栈由下至上包含：

- **Netty**：底层网络通信框架。不同于 Tomcat 的“一线程一请求”同步阻塞模型，Netty 基于 NIO（非阻塞 I/O），能够使用少量线程处理海量并发连接
- **Project Reactor**：Spring 的反应式编程库，实现了 Reactive Streams 规范，提供了 `Mono` 和 `Flux` 异步序列流
- **Spring WebFlux**：Spring 5.0 推出的非阻塞 Web 框架，它是 Gateway 的运行基础



##### b. 阻塞与非阻塞的区别（技术原理）

- **传统架构 (Spring MVC)**：

  - 基于 **Servlet API**

    当请求进入时，线程会被阻塞，直到业务处理完成（如等待数据库查询）。在高并发下，线程池容易耗尽，导致系统崩溃

- **Gateway 架构**：

  - 基于 **WebFlux**

    采用**异步非阻塞**模型。当涉及 I/O 操作时，线程不会等待，而是通过回调或事件驱动机制处理后续逻辑

    这使得 Gateway 能够以极低的资源消耗处理高吞吐量请求



##### c. 环境搭建与依赖管理

###### 核心依赖坐标

在构建网关服务时，核心依赖为 `spring-cloud-starter-gateway`



###### **依赖说明**

| Group ID                    | Artifact ID                    | 作用                                                         |
| --------------------------- | ------------------------------ | ------------------------------------------------------------ |
| `org.springframework.cloud` | `spring-cloud-starter-gateway` | 引入 Gateway 核心功能，自动包含 `spring-boot-starter-webflux` 和 `Netty` |



###### **Maven 坐标**

```xml
<dependencies>
    <!-- Gateway 核心依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    
    <!-- 服务注册中心依赖（Gateway 通常配合 Nacos 或 Eureka 等使用） -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    
    <!-- Spring Cloud 负载均衡器 (替代 Ribbon) -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
</dependencies>
```



##### d. Servlet 与 WebFlux 的兼容性冲突

###### **1. 场景描述**

- 你创建了一个新的 Spring Boot 模块作为网关，习惯性地引入了 `spring-boot-starter-web`，或者你的父工程（Parent POM）默认引入了该依赖



###### 2. 异常现象

启动应用时，当项目中存在依赖冲突时，会抛出如下异常：

```cmd
Description:
Spring MVC found on classpath, which is incompatible with Spring Cloud Gateway.

Action:
Please include spring-boot-starter-webflux or remove spring-boot-starter-web.
```



###### 3. 根本原因

Spring Cloud Gateway 基于 **WebFlux**，而 `spring-boot-starter-web` 基于 **Spring MVC**。 这两者在同一个应用中是 **互斥** 的，不能共存

> Spring Boot 在启动时会进行自动配置
>
> 如果 Classpath 中检测到 `spring-boot-starter-web`，它会尝试初始化 Servlet 容器（Tomcat）
> 而 Gateway 强依赖于 WebFlux 环境，**两者无法在同一个 Spring Boot 应用中共存**



###### 4. 解决方式

在 Gateway 模块的 `pom.xml` 中，**绝对不要**引入 `spring-boot-starter-web`

如果你的父工程为了方便统一管理，全局引入了 `spring-boot-starter-web`，那么你必须在 Gateway 模块中 **显式地** 将其 **排除**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 显式排除导致冲突的传递依赖 -->
    <exclusions>
        <!-- 1. 排除内嵌的 Tomcat 容器 -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
        <!-- 2. 排除 Spring MVC 框架 -->
        <exclusion>
            <groupId>org.springframework.web</groupId>
            <artifactId>spring-webmvc</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```





#### 3. 核心概念

Spring Cloud Gateway 的处理流程完全围绕着 **“路由”** 这一核心概念构建，最重要的三个术语：**路由、断言、过滤器**



$$\text{Route (路由)} = \text{ID (唯一标识)} + \text{URI (目的地)} + \underbrace{\text{Predicates (断言集合)}}_{\text{判断条件}} + \underbrace{\text{Filters (过滤器集合)}}_{\text{处理逻辑}}$$



##### AA. Route (路由)

这是网关的基本构建模块。本质上就是一条 **“转发规则”**(对于转发是什么，我们在前文已经介绍过了)

> 当前端请求访问到 gateway 的时候，gateway 会去利用这些 **”转发规则“**(路由) 去处理这些一个又一个的请求

- 一个 **路由** 由以下几部分组成：

  - **ID**：路由的唯一标识符（String类型）

    - *建议*：使用具有业务含义的名称（如 `order_service_route`），便于日志追踪和管理
  
      
  
  - **URI**：**路由断言匹配成功后**，请求最终要被转发到的目标地址
  
    - **Http 方式**：`http://localhost:8080` (直接指向特定地址)
  
    - **LoadBalancer 方式**：`lb://service-name` (指向服务注册中心的服务名，启用负载均衡)
  
      
  
  - **Predicate (断言)**：一组 **匹配规则**，**仅** 用于 **筛选** 请求
  
    - *作用*：负责判断请求是否符合条件（比如：判断请求路径是否以 `/order` 开头）
    - *注意*：它是“只读”的，绝对不会修改请求内容
  
    
  
  - **Filter (过滤器)**：**处理器**，它和断言的“纯筛选”不同，它负责对请求或响应进行 **加工**，这里容易被这个名字误导......
  
    - *生命周期*：在 **网关将请求转发给下游服务之前 (Pre)**，或 **从下游服务接收到响应之后 (Post)** 执行
    - *作用*：执行 **数据处理** 的逻辑链（如：修改请求头、鉴权拦截、统计耗时）



- 一个 Gateway 服务可以配置成百上千个路由，管理通往后端几十个甚至上百个微服务的所有入口

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
          # ---------------------------------------------------
          # [路由 1] 静态路由示例 (指向具体 IP)
          # ---------------------------------------------------
          - id: payment_route_static          # 1. ID: 唯一且有语义
            uri: http://localhost:8001        # 2. URI: 直接指向目标地址
            predicates:                       # 3. 断言: 必须同时满足以下条件
              - Path=/payment/** 					#    条件A: 路径匹配
              - Method=GET                    	#    条件B: 且必须是 GET 请求
            filters:                          # 4. 过滤器: 对请求进行加工
              - AddRequestHeader=X-Source, Gw #    动作: 添加请求头
  
          # ---------------------------------------------------
          # [路由 2] 动态路由示例 (生产推荐，配合注册中心)
          # ---------------------------------------------------
          - id: order_route_dynamic
            uri: lb://order-service           # 2. URI: 使用 lb:// 协议启用负载均衡
            predicates:
              - Path=/order/** 				#    只要路径匹配 /order/**
            filters:
              - StripPrefix=1                 #    动作: 去掉路径的第一层 (如 /order/create -> /create)
  ```

  - 在 `application.yml` 中，`routes` 本质上是一个**数组（List）**。你只需要像写清单一样，把路由一条接一条地列出来即可



##### b. Predicate (断言)

> 一定在路由内部

###### 技术定义

- Predicate 是 Java 8 `java.util.function.Predicate<T>` 的一种实现

  在 Gateway 中，输入参数 `T` 是 `ServerWebExchange`（封装了 HTTP 请求和响应的上下文对象）

  - **接口定义(简化版)**:

    ```java
    @FunctionalInterface
    public interface Predicate<T> {
        // 评估输入参数，返回 true (匹配) 或 false (不匹配)
        boolean test(T t);
    }
    ```



###### 作用与行为

- **匹配逻辑**：开发人员通过 Predicate 匹配 HTTP 请求的各种属性（Request Path, Method, Header, Host, Cookie, Query Param 等）
- **逻辑组合**：一个路由可以定义多个 Predicate，它们之间默认是 **AND (逻辑与)** 的关系。只有当所有断言都返回 `true` 时，该路由才会被匹配



###### 常用断言工厂

Spring Cloud Gateway 内置了许多工厂类来生成断言，以下是开发中最常用的几种：

| 断言工厂   | 说明                 | 示例配置                                           |
| ---------- | -------------------- | -------------------------------------------------- |
| **Path**   | 匹配请求路径         | `- Path=/order/**`                                 |
| **Method** | 匹配 HTTP 方法       | `- Method=GET,POST`                                |
| **After**  | 在指定时间之后       | `- After=2025-01-01T00:00:00+08:00[Asia/Shanghai]` |
| **Header** | 匹配请求头(支持正则) | `- Header=X-Request-Id, \d+`                       |
| **Query**  | 匹配请求参数         | `- Query=token` (是否有token参数)                  |



##### c. Filter (过滤器)

###### 技术定义

- Filter 是 Gateway 处理请求和响应的核心逻辑组件

  - **作用**：在请求被路由之前（Pre）或之后（Post）修改请求和响应

  - **场景**：添加请求头、记录日志、鉴权、限流等



###### 生命周期

从逻辑上我们将其分为两个阶段：

1. **Pre Filter (前置处理)**：

   - **执行时机**：请求被路由匹配成功后，发送给目标服务 **之前**

   - **常见作用**：参数校验、权限校验、流量监控、日志埋点、修改请求头

     

2. **Post Filter (后置处理)**：

   - **执行时机**：目标服务处理完毕，响应返回给网关 **之后**，再返回给客户端之前
   - **常见作用**：修改响应头、修改响应体、统计请求耗时、统一异常处理



###### 过滤器分类

- **GatewayFilter (局部过滤器)**：只作用于当前配置的 **特定路由**
- **GlobalFilter (全局过滤器)**：不需要在配置文件中显式配置，作用于 **所有路由**（如全局鉴权、全局LoadBalancer）



##### ee. 配置方式

- 配置 Gateway 有两种主流方式：**YAML 配置文件**（常用）和 **Java Fluent API**（灵活，适合动态配置）

###### 方式一：YAML 配置（推荐，声明式）

这是最直观的配置方式，易于阅读和维护

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: payment_route               # 1. 路由 ID
          uri: http://localhost:8001      # 2. 目标 URI
          predicates:                     # 3. 断言集合 (AND关系)
            - Path=/payment/** # 匹配路径
            - Method=GET                  # 且必须是 GET 请求
          filters:                        # 4. 过滤器集合
            - AddRequestHeader=X-Source, Gateway # 添加请求头
            - StripPrefix=1               # 去除路径前缀的第一部分
```



###### 方式二：Java Fluent API（编程式）

这种方式通常用于需要根据代码逻辑动态构建路由的场景

**核心类：`RouteLocatorBuilder`**

- **作用**：用于构建 `RouteLocator` Bean，定义路由规则
- **常用方法**：
  - `routes()`: 开始构建路由流
  - `route(id, fn)`: 定义单个路由，`fn` 是一个函数式接口，用于配置 Predicate 和 Filter
  - `path(String)`: 定义路径断言
  - `uri(String)`: 定义目标 URI
  - `filters(fn)`: 定义过滤器链

**代码示例：**

```java
@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("code_route", r -> r
                // 1. 断言：匹配路径 /guonei
                .path("/guonei")
                // 2. 过滤器：添加参数 name=test
                .filters(f -> f.addRequestParameter("name", "test"))
                // 3. 目标 URI
                .uri("[http://news.baidu.com/guonei](http://news.baidu.com/guonei)")
            )
            .build();
    }
}
```



#### 4. 快速上手：Hello World

- 我们通过一个简单的需求来体验 Gateway 的配置：**当用户访问网关的 `/guonei` 路径时，自动跳转到百度新闻国内版**

##### 方式一：YAML 配置（推荐，生产主流）

在 Spring Cloud 开发中，**约定大于配置**。我们首选 `application.yml` 来管理路由，因为它支持动态刷新，且运维人员无需修改代码即可调整规则

**配置示例：**

```yaml
server:
  port: 9527 # 网关服务端口

spring:
  cloud:
    gateway:
      routes:
        - id: baidu_news_route           # 1. 路由 ID (唯一标识)
          uri: [http://news.baidu.com](http://news.baidu.com)     # 2. 目标 URI (最终要去的地方)
          predicates:
            - Path=/guonei/** 				# 3. 断言 (匹配规则)
```

**测试效果：** 访问 `http://localhost:9527/guonei`，网关会匹配到 `baidu_news_route`，并将请求转发给百度新闻。



##### 方式二：Java Fluent API（编程式）

虽然 YAML 是主流，但在某些需要 **动态构建路由** 或 **复杂逻辑判断** 的场景下，Java API 非常有用

**核心类：`RouteLocatorBuilder`** Spring Boot 会自动注入这个构建器，可以通过链式调用来定义路由

**代码示例：**

```java
@Configuration
public class GatewayConfig {

    /**
     * 编程式路由配置
     * @param builder 由 Spring 容器自动注入
     * @return RouteLocator 路由规则定位器
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // 定义一条路由，ID 为 "path_route_baidu"
            .route("path_route_baidu", r -> r
                // 1. 断言：路径匹配 /guonei
                .path("/guonei")
                // 2. 目标：转发地址
                .uri("[http://news.baidu.com/guonei](http://news.baidu.com/guonei)")
            )
            // .route(...) 可以继续链式添加其他路由
            .build();
    }
}
```

**实战建议**：初学者请专注掌握 **YAML 配置**。Java API 通常用于开发自定义的路由管理后台



#### 5. 断言

在 Spring Cloud Gateway 中，**Predicate (断言)** 是决定请求“去向”的核心判断逻辑

- 在 YAML 配置中，`predicates` 下面的每一行（如 `- Path=/abc`）其实都对应着一个 Spring 内置的 **Factory 类**

  它们负责提取 HTTP 请求的各种属性（如发送时间、Cookie、Header、Host 等），并与配置的规则进行匹配
  
- **断言** 使用的是 **短路匹配**

  - **先到先得，找到即止**

    Gateway 会把你配置的所有路由按顺序排成一个长队,当请求进来时，Gateway 从 **第一条** 路由开始尝试匹配

    - 如果不匹配 -> 继续看下一条
    - **一旦匹配成功** -> **立刻停止** 后续的所有检查，直接 **利用** 当前这条路由进行转发




##### XX. 核心逻辑：逻辑与 (AND)

一个路由可以配置多个断言，它们之间默认是 **AND (逻辑与)** 的关系

> **规则**：只有当配置的 **所有** 断言都返回 `True` 时，该路由才算匹配成功，请求才会被转发

如果任何一个断言失败，网关会忽略该路由，继续尝试匹配列表中的下一个路由



##### a. 基础路径断言 (Path) [最常用]

这是网关中最常见、几乎必配的断言。它用于匹配请求的 URI 路径

- **工厂类**：`PathRoutePredicateFactory`
- **参数**：支持 Spring 的 `AntPathMatcher` 风格通配符

###### 通配符语法详解

| 符号 | 含义                         | 示例          | 匹配情况 (True)            | 不匹配情况 (False)         |
| ---- | ---------------------------- | ------------- | -------------------------- | -------------------------- |
| `?`  | 匹配 1 个字符                | `/api/v?`     | `/api/v1`, `/api/v2`       | `/api/v10` (字符数不对)    |
| `*`  | 匹配 0 或多个字符 **(单层)** | `/api/*.json` | `/api/user.json`           | `/api/a/b.json` (跨层级了) |
| `**` | 匹配 0 或多个目录 **(多层)** | `/user/**`    | `/user/get`, `/user/a/b/c` | `/other/get`               |



###### 单路径匹配 (基础)

这是最简单的用法，指定唯一的一个路径模板。只有请求路径符合该模板时，才会应用路由

**YAML 配置示例：**

```yaml
predicates:
  # 匹配 /order 下的所有子路径，如 /order/create
  - Path=/order/**
```



###### 多路径匹配

你可以在一行配置中指定多个路径模板，用逗号分隔。它们之间是 **OR (或)** 的关系。只要请求路径满足其中 **任意一个**，断言就会通过

**YAML 配置示例：**

```yaml
predicates:
  # 只要路径是以 /red 开头 OR 以 /blue 开头，都算匹配成功，走这个路由
  - Path=/red/**, /blue/**
```



###### 进阶用法：URI 模板变量 (占位符)

`Path` 断言支持使用 `{name}` 的形式来 **捕获** 路径中的特定片段

- 这些捕获到的值会被提取出来，供后续的过滤器（如 `SetPath`）直接使用，这在 RESTful 风格的 API 中非常强大

**配置示例：**

```yaml
predicates:
  # 捕获中间的 segment 作为变量 {segment}
  - Path=/product/{segment}/detail
```

- **请求**：`/product/12345/detail`
- **结果**：匹配成功，并且网关会自动提取出 `segment = 12345`



###### 注意

- **路由顺序 (Order)**：如果你配置了两个路由，一个匹配 `/product/detail` (精确)，一个匹配 `/product/**` (模糊)
  - **原则**：请务必把 **精确匹配** 的路由配置在 **模糊匹配** 的路由 **之前**
  - **原因**：Gateway 采用“短路匹配”，如果模糊匹配在前面，请求会被拦截，永远走不到精确匹配的路由里



##### b. 时间类断言

这类断言通常用于 **秒杀**、**预售**、**维护期** 或 **限时活动** 等对时间敏感的场景

| 断言类型    | 作用                       | 场景示例                                   |
| ----------- | -------------------------- | ------------------------------------------ |
| **After**   | 在指定时间 **之后** 生效   | 2025年双11零点开始抢购，之前的请求会被拒绝 |
| **Before**  | 在指定时间 **之前** 生效   | 优惠券领取截止时间，过期后路由失效         |
| **Between** | 在指定时间段 **之间** 生效 | 仅在活动期间（如 1号到 3号）开放入口       |



**YAML 配置示例：**

```yaml
predicates:
  # 只有在 2025年1月1日 凌晨1点之后，这个接口才通
  - After=2025-01-01T01:00:00.000+08:00[Asia/Shanghai]
```



**🚨 预警：时间格式**

Gateway 对配置中的时间字符串格式要求极其严格，必须遵循 **Java 8 ZonedDateTime (ISO-8601)** 标准，包含时区信息

- ❌ **错误写法**：`2025-01-01 01:00:00` (常见错误，启动直接报错)
- ✅ **正确写法**：`2025-01-01T01:00:00.000+08:00[Asia/Shanghai]`

**最佳实践：如何获取这个字符串？** 千万不要手写！请运行下面这段简单的 Java 代码，复制控制台的输出结果：

```java
public class TimeUtils {
    public static void main(String[] args) {
        // 获取当前时区的标准格式时间字符串
        System.out.println(java.time.ZonedDateTime.now());
        // 输出示例：2023-11-23T15:30:11.456+08:00[Asia/Shanghai]
    }
}
```



##### c. 请求属性类断言

这类断言用于提取 HTTP 请求的特征进行精细化路由，常用于 **灰度发布 (Canary Release)**、**API 版本控制** 或 **特定来源限制**

| 断言工厂   | 说明           | 示例配置                                        |
| ---------- | -------------- | ----------------------------------------------- |
| **Header** | 检查请求头     | `- Header=X-Request-Id, \d+` (Value 必须是数字) |
| **Cookie** | 检查 Cookie    | `- Cookie=userType, vip` (必须包含 vip cookie)  |
| **Method** | 检查 HTTP 方法 | `- Method=GET,POST` (限制请求动作)              |
| **Host**   | 检查 Host 域名 | `- Host=**.baidu.com` (支持 Ant 风格通配符)     |



**🚨 陷阱预警：正则匹配**

`Header` 和 `Cookie` 断言接收两个参数：`key` 和 `regexp` (正则表达式)

- **场景**：你想匹配 Header 中 `X-Name` 包含 "admin" 的请求
- **错误配置**：`- Header=X-Name, admin`
  - *原因*：这表示值必须**完全等于** "admin"
- **正确配置**：`- Header=X-Name, .*admin.*`
  - *原因*：第二个参数是正则，`.*` 表示匹配任意字符



##### d. 参数类断言

用于检查 URL 中的查询参数

**YAML 配置示例：**

```yaml
predicates:
  # 1. 简单匹配：请求必须包含名为 token 的参数 (例如 /api?token=123)
  - Query=token
  
  # 2. 值匹配：参数值必须满足正则表达式
  # 要求：token 参数的值必须以 jwt 开头
  # - Query=token, jwt.*
```



##### ee. 实战演练：组合拳

假设我们有一个“新版本内测服务”，准入规则极其严格，必须 **同时满足** 以下 3 点：

1. **时间限制**：必须在 2024 年之后
2. **路径限制**：访问路径以 `/beta` 开头
3. **身份限制**：请求头必须携带 `X-Beta-Test: true`

**Application.yml 完整配置：**

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: beta_service_route
          uri: http://localhost:8088
          predicates:
            # 多个断言同时存在，必须全部满足 (AND 关系)
            - Path=/beta/**
            - After=2024-01-01T00:00:00.000+08:00[Asia/Shanghai]
            - Header=X-Beta-Test, true
```

**测试结果分析：**

- **场景 A**：`curl http://gateway/beta/api`

  - **结果**：❌ **404 Not Found**
  - **原因**：虽然路径和时间满足，但缺少必要的 Header，断言链匹配失败

  

- **场景 B**：`curl -H "X-Beta-Test: true" http://gateway/beta/api`

  - **结果**：✅ **200 OK**
  - **原因**：所有条件均满足，请求成功转发至 `localhost:8088`



#### 6. 过滤器

在 Spring Cloud Gateway 中，**Filter (过滤器)** 负责对请求和响应进行“加工”。所有的过滤器都遵循 **责任链模式**，请求在链条中依次传递

##### a. 过滤器的生命周期

虽然在代码层面是一个链条，但在逻辑上，我们通常根据“执行时机”将过滤器分为两个阶段：

| 阶段                | 时机                                                    | 作用                                              |
| ------------------- | ------------------------------------------------------- | ------------------------------------------------- |
| **Pre (前置处理)**  | 请求被路由匹配成功，但转发给下游服务 **之前**           | 参数校验、鉴权 (Auth)、限流、修改请求头、重写路径 |
| **Post (后置处理)** | 下游微服务返回响应 **之后**，网关将响应发回给客户端之前 | 修改响应头、统计接口耗时、统一异常处理、日志记录  |



##### b. 内置过滤器

Spring Cloud Gateway 内置了 30 多种过滤器工厂，用于通过 YAML 配置快速修改请求。其中最常用的是 **Header 处理** 和 **Path 处理**

###### 请求头与响应头处理

这类过滤器用于在请求转发的 **Pre 阶段** 或响应返回的 **Post 阶段**，对 HTTP Header 进行增删改查

**核心语法**

Spring Cloud Gateway 采用 `工厂名=参数1, 参数2` 的简写格式

| 过滤器工厂名             | 作用                         | 语法格式     | 示例                        |
| ------------------------ | ---------------------------- | ------------ | --------------------------- |
| **AddRequestHeader**     | **添加** 请求头 (发给微服务) | `Key, Value` | `X-Source, Gateway`         |
| **RemoveRequestHeader**  | **移除** 请求头 (发给微服务) | `Key`        | `Cookie` (防止敏感信息泄露) |
| **SetRequestHeader**     | **覆盖** 请求头 (发给微服务) | `Key, Value` | `X-User-Id, 1001`           |
| **AddResponseHeader**    | **添加** 响应头 (发给客户端) | `Key, Value` | `X-Response-Time, 100ms`    |
| **RemoveResponseHeader** | **移除** 响应头 (发给客户端) | `Key`        | `Server` (隐藏服务器信息)   |



**YAML 配置实战**：

假设我们要实现以下需求：

1. 告诉微服务请求来自网关（添加 `X-Request-Source`）
2. 担心 Cookie 里的敏感信息传给微服务不安全，把它删掉
3. 告诉客户端这个请求是谁处理的（响应头添加 `X-Powered-By`）

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: header_route
          uri: lb://user-service
          predicates:
            - Path=/api/user/**
          filters:
            # --- 请求头处理 (Request) ---
            # 1. 添加：如果 Header 已存在，会追加在后面（变成数组）
            - AddRequestHeader=X-Request-Source, Gateway
            
            # 2. 覆盖/设置：不管原来有没有，强制改为这个值
            - SetRequestHeader=X-Role, Admin
            
            # 3. 移除：通常用于清洗敏感数据
            - RemoveRequestHeader=Cookie
            
            # --- 响应头处理 (Response) ---
            # 4. 响应返回给前端时，自动带上这个头
            - AddResponseHeader=X-Powered-By, SpringCloudGateway

```



###### 路径处理 (StripPrefix)⭐

这是微服务网关中最常用的过滤器，也是初学者最容易踩坑的地方

- **场景**：前端为了区分服务，请求地址通常包含服务名前缀（如 `/order-service/create`），但后端微服务的真实接口路径往往不包含该前缀（如 `/create`）
- **作用**：在转发前，从请求路径中 **截断** 指定数量的层级



**原理图解：**

假设配置为 `StripPrefix=1`：

| 客户端请求 Path | 过滤器操作             | 最终转发给微服务的 Path | 结果说明       |
| --------------- | ---------------------- | ----------------------- | -------------- |
| `/order/create` | 去掉第 1 层 (`/order`) | `/create`               | ✅ 正常         |
| `/a/b/c`        | 去掉第 1 层 (`/a`)     | `/b/c`                  | ✅ 正常         |
| `/api`          | 去掉第 1 层 (`/api`)   | `/`                     | ⚠️ 变成了根路径 |

**配置实战：**

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order_route
          uri: http://localhost:8002
          predicates:
            # 1. 匹配以 /order-service 开头的路径
            - Path=/order-service/**
          filters:
            # 2. 截断第 1 层前缀，否则微服务会报 404
            - StripPrefix=1
```

**请求流程追踪**：

1. **客户端请求**：`http://gateway:9527/order-service/create`
2. **断言匹配**：`/order-service/**` 匹配成功
3. **过滤器执行**：`StripPrefix=1` 将路径切割为 `/create`
4. **转发**：请求被发送到 `http://localhost:8002/create`



**🚨 巨坑预警：** `StripPrefix` 是按 **"/" 分隔的层级** 计数的，不是按字符数

- 如果配置 `StripPrefix=2`，访问 `/a/b/c` 会变成 `/c`
- 如果路径层级不够切（例如访问 `/a` 却配置了 `StripPrefix=2`），通常会导致转发异常



##### c. 默认过滤器 (`default-filters`)

在实际开发中，我们经常遇到这种需求：**给所有的微服务接口都加上同一个请求头**（例如 `X-Source: Gateway`），或者给所有响应都加上时间戳

如果我们在每个 Route 下面都写一遍 `filters`，配置会极其臃肿。这时候就需要 **`default-filters`**

- **作用**：对 **所有** 配置在 YAML 中的路由生效（相当于 YAML 版的全局过滤器）
- **位置**：配置在 `spring.cloud.gateway` 下，与 `routes` 平级

**YAML 配置示例：**

```YAML
spring:
  cloud:
    gateway:
      # --- 核心配置：默认过滤器 ---
      # 这里的过滤器会对下面 routes 列表中的所有路由生效
      default-filters:
        # 1. 给所有请求添加来源标识
        - AddRequestHeader=X-Request-Source, Gateway
        # 2. 给所有响应添加处理时间
        - AddResponseHeader=X-Powered-By, SpringCloudGateway
        # 3. 甚至可以全局开启限流 (如果用自带限流器的话)
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20

      # --- 具体的路由列表 ---
      routes:
        - id: order_route
          uri: lb://order-service
          predicates:
            - Path=/order/**
          # 这里只需要配置 order 特有的过滤器 (如 StripPrefix)
          # 上面的 default-filters 会自动合并进来
          filters:
            - StripPrefix=1
```

**对比总结：**

| 过滤器类型         | 作用范围 | 配置方式                | 典型场景                             |
| ------------------ | -------- | ----------------------- | ------------------------------------ |
| **Route Filter**   | 单个路由 | `routes` 下的 `filters` | 去前缀 (`StripPrefix`)、特定服务限流 |
| **Default Filter** | 所有路由 | `default-filters`       | 统一加请求头/响应头、全局日志        |
| **Global Filter**  | 所有请求 | Java 代码实现接口       | 统一鉴权 (Token)、复杂逻辑           |





##### d. 自定义全局过滤器

- 虽然 YAML 配置方便，但复杂的业务逻辑（如 **统一鉴权**、**黑名单校验**）必须通过 Java 代码实现

- 要实现一个全局过滤器，通常需要实现 `GlobalFilter` 和 `Ordered` 这两个接口

###### `GlobalFilter` 接口

```java
public interface GlobalFilter {
    /**
     * @param exchange  网关上下文（包含 Request 和 Response）
     * @param chain     过滤器链（用于放行请求到下一个环节）
     * @return Mono<Void>  代表一个异步任务
     */
    Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain);
}
```

- **`exchange` (网关上下文)**：

  - 它是封装了 HTTP **请求** 和 **响应** 的核心容器

  - **获取数据**：调用 `exchange.getRequest()` 可读取请求参数、Header 等信息

  - **控制响应**：调用 `exchange.getResponse()` 可设置状态码或结束请求

  - **传递数据**：使用 `exchange.getAttributes()` 可在不同过滤器间共享变量

    

- **`chain` (过滤器链条)**：

  - 它是控制请求 **流转** 的关键对象
  - **放行**：必须手动调用 `return chain.filter(exchange)`，请求才会传递给下一个过滤器或微服务
  - **拦截**：如果不调用它，而是直接操作 response 并返回（如 `response.setComplete()`），请求链就会在此终止（常用于鉴权失败）

  

- **`Mono<Void>` (异步任务返回值)**：

  - Gateway 基于 WebFlux (Reactor) 响应式编程，因此方法无法直接返回结果
  - 这个返回值代表了 **当前过滤逻辑的执行状态** 。它告诉框架：“这里定义了一个任务，请在未来执行它，并在完成后通知我”



###### `Ordered` 接口

```java
public interface Ordered {
    int getOrder();
}
```

在 Spring Cloud Gateway 中，所有的过滤器（Global、Route、Default）最终都会被合并到一条 **过滤器链 (Filter Chain)** 中执行

- **作用**：决定过滤器的执行顺序

- **规则**：数字越 **小**，优先级越 **高**

  - **Pre 阶段**：Order 小的先执行
  - **Post 阶段**：Order 小的后执行（类似于栈的“先进后出”）

- **为什么要注意顺序**

  - 你写的过滤器并不是在“裸奔”，Gateway 内部已经内置了很多 **默认过滤器** 在运行，它们也有自己的 `Order`

    如果你的 Order 设置不当，可能会导致业务失效：

    - **典型冲突场景 1：请求转发前修改**

      - 网关内置了一个核心过滤器 `NettyRoutingFilter`（负责把请求通过网络发给微服务），

        它的 Order 是 `Integer.MAX_VALUE`（最大值，最后执行）

      - **后果**：

        - 这保证了你自定义的过滤器（只要 Order < MAX_VALUE）都在它之前执行

          但如果你不小心把 Order 设得太大，可能请求都已经发出去了，你的 Pre 逻辑才执行，那就没用了

    - **典型冲突场景 2：响应写回前修改**

      - 网关内置了 `NettyWriteResponseFilter`（负责把响应数据写回给客户端），它的 Order 是 `-1`（非常小）
      - **后果**：这就意味着它在 Post 阶段是 **最后执行** 的（Pre 阶段最先执行）
      - 你的自定义过滤器如果想在 Post 阶段修改响应头，Order 最好大于 -1

- **最佳实践**

  - **鉴权/安全类**：建议设置为 **负数**（如 `-100`）
    - **原因**：越小越靠前。确保在系统处理请求参数之前，先完成身份校验。如果校验失败，直接拒绝，省得浪费资源去跑后面的逻辑
  - **统计/日志类**：建议设置为 **极小的负数**（如 `-10000`）
    - **原因**：这样它能包裹住几乎所有的过滤器，统计出的“耗时”才最接近网关处理的总耗时
  - **一般业务类**：设置为 `0` 或正整数即可



###### ⭐补充：YAML 配置中过滤器的执行顺序

我们知道 `GlobalFilter` (Java代码) 可以通过 `getOrder()` 显式指定优先级

- 那么问题来了：**我们在 `application.yml` 中配置的 `default-filters` 和路由专属 `filters`，它们的优先级是多少？谁先执行？**



**aaa. 自动排序规则**

Spring Cloud Gateway 在解析 YAML 配置文件时，会按照以下规则自动为过滤器分配 `Order` 值：

1. **起始值**：从 **1** 开始
2. **递增规则**：按照配置在文件中的 **先后顺序**，依次递增
3. **合并逻辑**：Gateway 会先加载 `default-filters`，再加载具体 Route 的 `filters`，将它们合并为一个列表

**结论公式**： **最终列表 = [default-filters] + [route-filters]** （`default-filters` 总是排在前面，优先级更高）



**bbb. 计算示例**

假设你的配置文件如下：

```yaml
spring:
  cloud:
    gateway:
      # --- 全局默认过滤器 ---
      default-filters:
        - FilterA  # 它是第 1 个 -> Order = 1
        - FilterB  # 它是第 2 个 -> Order = 2

      routes:
        - id: my_route
          uri: lb://my-service
          # --- 路由专属过滤器 ---
          filters:
            - FilterC  # 接在 default 后面 -> Order = 3
            - FilterD  # 继续递增 -> Order = 4
```

**最终执行顺序 (Pre 阶段)**： `FilterA (1)` -> `FilterB (2)` -> `FilterC (3)` -> `FilterD (4)`



**ccc. 对自定义开发的影响**

理解这个顺序对于编写自定义 **GlobalFilter** 至关重要：

- **如果你想在所有 YAML 过滤器之前执行**：
  - 请将你的 GlobalFilter 的 Order 设为 **负数** (如 `-1`)
  - *场景*：统一鉴权、黑名单校验（先拦下来，别浪费时间去跑后面的逻辑）
- **如果你想在所有 YAML 过滤器之后执行**：
  - 请将你的 GlobalFilter 的 Order 设为 **较大的正数** (如 `100`)
  - *场景*：统一日志记录（等大家都处理完了再记）



###### 一些比较关键的对象

- **`ServerWebExchange`**：这是 WebFlux 的核心容器，替代了 Servlet 中的 `HttpServletRequest` 和 `HttpServletResponse`

  | 作用              | 代码示例                                                     | 说明                                   |
  | ----------------- | ------------------------------------------------------------ | -------------------------------------- |
  | **获取 Request**  | `ServerHttpRequest request = exchange.getRequest();`         | 获取只读的请求对象                     |
  | **获取 Response** | `ServerHttpResponse response = exchange.getResponse();`      | 获取响应对象                           |
  | **共享数据**      | `exchange.getAttributes().put("startTime", System.currentTimeMillis());` | 在不同的过滤器之间传递参数（类似 Map） |
  | **获取共享数据**  | `long startTime = exchange.getAttribute("startTime");`       | 取出之前存的数据                       |

  - `ServerHttpRequest ` (请求对象)

    > **注意**：这个对象是 **只读** 的！如果你想修改它（比如加 Header），必须使用 `mutate()` 方法创建一个新的 Request 副本，然后重新构建 Exchange

    - **a. 获取请求信息 (查)**

      这是鉴权逻辑中最常用的部分

      ```JAVA
      ServerHttpRequest request = exchange.getRequest();
      
      // 1. 获取请求路径
      String path = request.getPath().value();        // 例如: /order/create
      String rawPath = request.getURI().getRawPath(); // 原始路径
      
      // 2. 获取请求方法
      HttpMethod method = request.getMethod();        // GET, POST...
      String methodName = method.name();
      
      // 3. 获取请求头 (Header) - 最常用！
      HttpHeaders headers = request.getHeaders();
      String token = headers.getFirst("Authorization"); // 获取单个值
      List<String> values = headers.get("my-header");   // 获取多个值
      
      // 4. 获取查询参数 (Query Param) - 最常用！
      // 例如: /api?name=zhangsan&age=18
      MultiValueMap<String, String> queryParams = request.getQueryParams();
      String name = queryParams.getFirst("name");
      
      // 5. 获取客户端 IP 地址
      InetSocketAddress remoteAddress = request.getRemoteAddress();
      String clientIp = remoteAddress.getAddress().getHostAddress();
      ```

    - **b. 修改请求信息💡**

      由于 `ServerHttpRequest` 是不可变的，你不能直接 `request.getHeaders().add(...)`，这会报错

      **正确做法**：使用 `mutate()` 创建一个新的 Request 副本，然后重新构建 Exchange

      ```JAVA
      // 场景：在网关鉴权通过后，往 Header 里塞一个 UserId 传给下游微服务
      ServerHttpRequest newRequest = exchange.getRequest().mutate()
          .header("X-User-Id", "10086") // 添加/覆盖 Header
          .path("/new/path")            // 重写路径 (可选)
          .build();
      
      // 【关键一步】必须把新的 Request 塞回 Exchange，否则下游拿不到
      ServerWebExchange newExchange = exchange.mutate()
          .request(newRequest)
          .build();
      
      // 放行时传入新的 Exchange
      return chain.filter(newExchange);
      ```

      

  - `ServerHttpResponse `(响应对象)

    >主要用于 **拒绝请求**（直接返回错误码）或 **修改响应头**

    - **a. 拒绝请求 (拦截)**

      当鉴权失败时，我们需要直接结束请求，不让它转发给微服务

      ```JAVA
      ServerHttpResponse response = exchange.getResponse();
      
      // 1. 设置状态码 (例如 401 或 403)
      response.setStatusCode(HttpStatus.UNAUTHORIZED);
      
      // 2. 添加响应头 (可选)
      response.getHeaders().add("Content-Type", "application/json");
      
      // 3. 结束请求 (关键！)
      // setComplete() 会发送响应并关闭连接，不再执行后续过滤器
      return response.setComplete();
      ```

    - **b. 返回自定义 JSON (进阶)**

      如果你想返回 `{"code": 401, "msg": "No Token"}` 这种 JSON，操作比较繁琐（因为是流式 IO），通常建议只返状态码。如果非要写：

      ```JAVA
      // 这是一个通用的写入 JSON 的模板代码
      String msg = "{\"code\": 401, \"msg\": \"非法访问\"}";
      DataBuffer buffer = response.bufferFactory().wrap(msg.getBytes(StandardCharsets.UTF_8));
      response.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
      return response.writeWith(Mono.just(buffer));
      ```

    

  - `GatewayFilterChain `

    > 用于控制流转

    - **放行**：`return chain.filter(exchange);`

      - 这里的 `exchange` 如果是修改过的，下游就能拿到修改后的数据

    - **后置处理 (Post Logic)**：

      - 如果你想在微服务返回 **之后** 做事（比如统计耗时），不能写在 `return` 后面，而是要利用 Reactive 的链式编程：

      ```java
      long start = System.currentTimeMillis();
      
      // 先去执行后面的逻辑 (chain.filter)
      return chain.filter(exchange).then(
          // 等都执行完了，再回来执行这里的逻辑 (Mono.fromRunnable)
          Mono.fromRunnable(() -> {
              long end = System.currentTimeMillis();
              System.out.println("耗时: " + (end - start) + "ms");
          })
      );
      ```





###### 实战场景1：统一鉴权过滤器

**场景**：在请求转发给微服务之前，拦截所有请求，检查 Header 中是否包含合法的 Token

```java
@Component
public class AuthGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 1. 获取 Request 对象
        ServerHttpRequest request = exchange.getRequest();

        // 2. 获取 Header 中的 Token
        String token = request.getHeaders().getFirst("Authorization");

        // 3. 校验逻辑 (模拟)
        if (token == null || token.isEmpty()) {
            System.out.println("❌ 鉴权失败：未携带 Token");
            
            // 4. 拒绝请求逻辑
            ServerHttpResponse response = exchange.getResponse();
            response.setStatusCode(HttpStatus.UNAUTHORIZED); // 设置 401 状态码
            
            // 5. 【关键】直接结束请求，不再向下传递
            return response.setComplete();
        }

        // 6. 放行逻辑
        System.out.println("✅ 鉴权成功");
        // 将请求交给链条中的下一个过滤器
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        // 设置为 -1，保证在大多数系统过滤器之前执行
        return -1;
    }
}
```



###### 实战场景2：接口耗时统计

- **场景**：我们需要统计每个请求从进入网关到微服务返回响应，总共花费了多少时间，并打印日志。这就需要用到 **Post（后置）** 逻辑

- **难点**：WebFlux 中没有显式的 "return" 后的代码块，必须使用 `.then()` 方法

**代码实现：**

```java
@Component
public class ExecutionTimeFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // --- Pre 阶段 (请求发出去之前) ---
        
        // 1. 记录开始时间
        long startTime = System.currentTimeMillis();
        
        // 2. 放行，并定义 "回来之后" 要做的事
        return chain.filter(exchange).then(
            // --- Post 阶段 (微服务响应回来之后) ---
            Mono.fromRunnable(() -> {
                long endTime = System.currentTimeMillis();
                long executeTime = endTime - startTime;
                
                // 获取请求路径
                String path = exchange.getRequest().getURI().getPath();
                
                System.out.println("📊 接口 [" + path + "] 耗时: " + executeTime + "ms");
            })
        );
    }

    @Override
    public int getOrder() {
        // 这里的 Order 设定很有讲究
        // Order 越小，Pre 越先执行，Post 越后执行
        // 放在最外层包裹，才能统计到最完整的耗时
        return -100;
    }
}
```

**原理图解 (责任链的回调)：**

```
Filter A (Order -100) -> Pre逻辑: 记录开始时间
    Filter B (Order -1) -> Pre逻辑: 鉴权
        ... 转发给微服务 ...
    Filter B (Order -1) -> Post逻辑 (无)
Filter A (Order -100) -> Post逻辑: 计算结束时间 (then)
```



###### 实际开发中的“巨坑”指南

**坑一：千万别用 Thread.sleep()**

- **错误示范**：

  ```java
  // ❌ 绝对禁止！！
  Thread.sleep(1000); 
  ```



- **后果**：

  - Spring Cloud Gateway 底层是 Netty，默认只有 CPU 核数个线程（IO 线程）

    如果你让线程睡眠，整个网关的处理能力会瞬间归零，所有请求都会卡死

    

- **正确做法**：如果需要模拟延迟，使用 Reactor 的 API：

  ```java
  // ✅ 正确做法
  return Mono.delay(Duration.ofSeconds(1)).then(chain.filter(exchange));
  ```



**坑二：DataBuffer 的读写问题**

- 如果你想在过滤器中 **读取 Request Body（请求体）** 或 **修改 Response Body（响应体）**，这在 WebFlux 中非常复杂

  - 因为 Body 是数据流（Flux），可能分多次传输

    一旦你读取了流，流就断了，后续的服务就读不到了

- **最佳实践**：尽量避免在全局过滤器中读取 Body。如果必须读取（如验签），建议使用 Spring Cloud Gateway 提供的高级工厂 `ModifyRequestBodyGatewayFilterFactory`，不要自己手写



**坑三：Order 的优先级冲突**

- 如果你发现鉴权过滤器没生效，或者 Header 没加上，检查一下 `getOrder()` 的返回值
- 系统内置的过滤器 Order 通常在 `0` 到 `20000` 之间，或者非常小
- **建议**：自定义的安全类过滤器，Order 设为负数（如 -1, -10），确保最先执行



#### 7. Gateway 与 Nacos 整合：动态路由

在之前的章节中，我们的很多路由配置是写死 IP 的（如 `http://localhost:8001`）

- 这种 **静态路由** 在微服务架构中是不可接受的，因为微服务的 IP 是动态变化的，且会有多个实例

  我们将让 Gateway 与 **Nacos** 整合，实现基于服务名的 **动态路由** 和 **负载均衡**



##### a. 核心原理：LB 协议与负载均衡

Gateway 提供了一个特殊的协议头：**`lb://` (Load Balance)**，它是实现动态路由的关键

- **旧写法 (静态/直连)**：

  - `uri: http://localhost:8001`
  - **缺点**：单点依赖，IP 写死。一旦服务迁移或扩容，必须修改网关配置并重启

  

- **新写法 (动态/负载均衡)**：

  - `uri: lb://product-service`
  - **优点**：走注册中心，解耦 IP，支持负载均衡。网关只认识“服务名”，不关心具体 IP



**工作原理**： 

- 当 Gateway 接收到请求并匹配到路由后，如果发现 URI 是以 `lb://` 开头的，它会委托内置的 `LoadBalancerClient` 进行处理
  1. **解析服务名**：Gateway 提取 `lb://` 后面的主机名（例如 `product-service`）
  2. **服务发现**：去注册中心（Nacos）查询该服务名下当前所有 **健康实例** 的列表（`List<Instance>`）
  3. **负载均衡**：根据配置的算法（默认是轮询 RoundRobin），从列表中选择一个具体的 IP 和端口（例如 `192.168.1.5:8081`）
  4. **请求转发**：将 HTTP 请求转发给选中的那个具体实例



##### b. 环境搭建与依赖陷阱

要使用动态路由，Gateway 必须拥有“发现服务”和“选择服务”的能力，因此必须引入服务发现组件和负载均衡器



###### 依赖陷阱：Missing LoadBalancer

这是一个从 Spring Cloud 2020 版本开始，99% 的初学者都会踩的坑：

- **背景**：在旧版本中，`spring-cloud-starter-alibaba-nacos-discovery` 默认集成了 Netflix Ribbon 作为负载均衡器，开箱即用
- **变化**：从 **Spring Cloud 2020 (Ilford)** 版本开始，官方正式 **移除**了 Ribbon
- **后果**：如果你只引入了 Nacos Discovery，启动时不会报错，但在访问 `lb://xxx` 时，Gateway 会抛出 `503 Service Unavailable` 或 `Unable to find instance` 错误
- **解决**：必须手动引入 Spring Cloud 官方推荐的替代品 **Spring Cloud LoadBalancer**



###### 正确的依赖配置

在 Gateway 项目的 `pom.xml` 中，显式添加以下依赖：

```xml
<dependencies>
    <!-- 1. Nacos 服务发现 (让 Gateway 能找到微服务) -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <!-- 2. 【关键】Spring Cloud 负载均衡器 (替代 Ribbon) -->
    <!-- 如果不加这个，lb:// 协议无法解析，报 503 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
</dependencies>
```



##### c. 手动配置动态路由 (推荐)

这是生产环境最常用的模式：**在网关明确定义路由规则，但目标地址使用服务名动态解析**

> 它既保留了配置的灵活性（可以精细控制每个路由的断言、过滤器），又享受了服务发现的动态性

###### 标准配置示例 (application.yml)

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 地址
    gateway:
      routes:
        - id: product_route       # 路由 ID，保持唯一即可
          # 【核心】使用 lb:// 协议 + 服务名称
          # lb = LoadBalancer (负载均衡器)，product-service = Nacos里的服务名
          uri: lb://product-service
          predicates:
            # 断言：匹配所有以 /product 开头的请求
            - Path=/product/**
          filters:
            # 【伴侣配置】StripPrefix=1 (去前缀)
            # 作用：在转发前，自动去掉 Path 中的第一层 (/product)
            # 场景：网关暴露的是 /product/1，但微服务接口实际是 /1
            - StripPrefix=1
```



###### 效果演示与流程

假设 `product-service` 在 Nacos 中注册了两个实例：`192.168.1.5:8081` 和 `192.168.1.6:8081`

1. **客户端请求**：用户访问网关地址 `http://gateway-ip:9527/product/1`
2. **路由匹配**：网关发现路径 `/product/1` 符合 `product_route` 的断言规则
3. **解析与发现**：Gateway 识别到 `lb://` 协议，向 Nacos 查询 `product-service` 的健康实例列表
4. **处理过滤器 (StripPrefix)**：
   - 网关将路径 `/product/1` 切割掉第一层 `/product`
   - 最终转发给微服务的路径变为：`/1`
5. **负载均衡转发**：
   - 第 1 次请求 -> 转发给 `192.168.1.5:8081/product/1`
   - 第 2 次请求 -> 转发给 `192.168.1.6:8081/product/1` (默认轮询)

> **生产建议**： 
>
> - 为什么推荐这种方式，而不是后面会提到的“自动路由”？ 
>
>   因为这种方式 **控制力最强**。你可以在 `routes` 下面继续添加 `filters`（比如限流、去前缀、鉴权），而自动路由很难做到针对性配置



##### d. DiscoveryLocator (自动路由)

Spring Cloud Gateway 提供了一种“偷懒”模式，可以 **一行路由都不写**，自动根据 Nacos 里的服务名为所有服务创建路由

###### 开启配置

这种模式默认是关闭的，需要在 `application.yml` 中开启：

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启自动路由
          lower-case-service-id: true # 强制将服务名转为小写 (强烈推荐开启)
```



###### 自动生成的规则

开启后，网关会自动为 Nacos 中的每个服务创建一条路由规则：

- **断言**：`Path=/服务名/**`
- **转发**：`lb://服务名`

**访问示例**：

- 假设服务名为 `order-service`，接口为 `/create`
- 访问地址：`http://gateway-ip:9527/order-service/create`



###### 为什么生产环境不推荐？

虽然它很方便，但在正规项目中 **几乎不用**，原因如下：

1. **URL 丑陋且暴露**：必须把内部服务名（如 `user-center-service`）暴露在 URL 中给前端，不优雅且不安全
2. **控制力弱**：你无法针对某个特定的服务单独配置限流、重试、跨域或 `StripPrefix`。所有服务都是“一刀切”的配置
3. **命名冲突**：如果服务名叫 `order`，而你正好有个接口路径也叫 `/order`，容易导致 404 或死循环

> **结论**：建议仅在开发阶段调试使用。生产环境请使用 **“手动配置路由”** 模式



##### e. 实际开发中的“巨坑”指南

###### 坑一：服务名的大小写敏感

- **现象**：

  - Nacos 控制台里显示服务名是 `Product-Service` (大写)，你在路由里写 `lb://product-service` (小写)，

    或者访问自动路由 `/product-service/xxx`，结果报 503 或 404

- **原因**：Gateway 对服务名匹配有时非常严格（取决于版本和配置）

- **最佳实践**：

  - 所有微服务的 `spring.application.name` **统一使用小写中划线格式** (kebab-case)，例如 `order-service`, `user-center`。不要使用驼峰或大写
  - 如果在自动路由模式下，务必开启 `lower-case-service-id: true`




###### 坑二：`lb://` 协议的使用范围

- **误区**：很多人想在 Predicate（断言）里判断服务名
- **纠正**：`lb://` 只是一个 **URI 协议**，它只能写在 `uri:` 字段里。
  - ✅ `uri: lb://order-service` (正确)
  - ❌ `predicates: - Path=lb://order-service/**` (错误，Predicate 只能判断 HTTP 请求本身，如路径、Header)



##### f. 进阶:实现路由配置的完全热更新

###### 1. 背景与痛点

我们在前文中配置的 `lb://product-service` 虽然完美解决了 **服务 IP 变动** 的问题（服务上线/下线会自动感知），但它无法解决 **“路由规则本身变动”** 的问题

- **现状**：我们的路由规则（如 `Path=/product/**`，`StripPrefix=1`）目前是写死在 `application.yml` 中的
- **痛点**：一旦业务发生变化，比如：
  - 需要 **新增** 一个路由（如上线了新的 `payment-service`）
  - 需要 **修改** 现有规则（如临时增加一个限流过滤器）
  - 此时， **必须重启 Gateway 网关服务** 才能让新规则生效。这在生产环境中是不够优雅的
- **目标**：我们需要一种机制，通过 Nacos 下发 JSON 格式的路由配置，网关自动监听到变更并刷新内存中的路由表，实现 **无需重启，立即生效**



###### 2. 核心思路

Spring Cloud Gateway 原生并不支持 Nacos 路由配置的自动刷新（默认只能刷新普通属性变量，无法驱动路由表重建）

- 我们需要手动实现一个 **连接 Nacos 配置变更与 Gateway 路由引擎的监听适配器**，逻辑如下：

  1. **配置分离**：不再把路由规则混在 `application.yml` 里，而是单独存为一个 Nacos 里的 `JSON` 配置文件

     > 选择JSON的原因是为了方便快捷

  2. **监听变更**：在网关项目中编写 Java 代码，监听这个 JSON 文件的变化

  3. **动态刷新**：一旦监听到 JSON 发生改变，立即调用 Gateway 的底层接口 (`RouteDefinitionWriter`)，将旧路由删除，新路由写入



###### 3. 步骤一：在 Nacos 创建路由配置文件

我们需要在 Nacos 控制台新建一个独立的配置文件，专门用来存放路由规则

**新建配置**

- **Data ID**: `gateway-routes.json` (建议单独命名，不要和 application.yml 混淆)
- **Group**: `DEFAULT_GROUP`
- **配置格式**: **JSON** (选择 JSON，配合代码解析，十分方便)



**配置内容示例**

这里我们将原本 YAML 格式的路由规则，转换为 JSON 对象数组。 例如，配置一个 `order-service` 的路由：

```js
[
    {
        "id": "order-route",
        "uri": "lb://order-service",
        "predicates": [
            {
                "name": "Path",
                "args": {
                    "pattern": "/order/**"
                }
            }
        ],
        "filters": []
    },
    {
        "id": "user-route",
        "uri": "lb://user-service",
        "predicates": [
            {
                "name": "Path",
                "args": {
                    "pattern": "/user/**"
                }
            }
        ]
    }
]
```

> **注意**： JSON 格式对符号非常敏感，编写时容易出错。建议先在本地编辑器写好，用 JSON 校验工具检查无误后再粘贴到 Nacos



###### 4. 步骤二：编写监听器代码

> 见后文**”附.监听器详解"**



###### 5. 效果验证

1. **启动**：启动 Gateway 服务，观察日志，应看到 `"项目启动，首次加载路由配置..."`
2. **测试旧路由**：访问配置在 JSON 中的接口（如 `/order/1`），应能正常通
3. **动态修改**：
   - **不重启 Gateway**
   - 在 Nacos 控制台修改 `gateway-routes.json`，例如：把 `order-service` 的断言从 `/order/**` 改为 `/new-order/**`
   - 点击发布
4. **观察日志**：IDEA 控制台瞬间打印 `"监听到路由配置变更..."`
5. **验证生效**：
   - 访问 `/order/1` -> 404 (旧规则失效)
   - 访问 `/new-order/1` -> 200 (新规则生效)
   - **结论**：完全热更新实现成功！



##### 附. 监听器详解

在这里，我们将深入剖析实现动态路由所需的 Java 组件与逻辑。本示例将采用 **Fastjson2** 作为 JSON 解析工具（这也是目前性能极佳且流行的选择)



###### 1. 核心组件拆解

要实现“监听 Nacos 并刷新网关”，我们需要三个核心组件通力合作。在看代码之前，必须先理解它们各自的职责

**A. `RouteDefinitionWriter` (路由写入器)**

- **身份**：Spring Cloud Gateway 官方提供的核心接口

- **职责**：它是网关内存路由表的 **“管理员”**

- **为什么需要它**： 

  - 普通配置（如超时时间）只需要改个值；但路由规则是一个复杂的对象（包含 ID、断言、过滤器等）

    我们要往网关里 **新增** 或 **删除** 一个路由，不能直接操作 List，必须通过这个“管理员”提供的 `save()` 和 `delete()` 方法来安全地修改



**B. `NacosConfigManager` (配置管理器)**

- **身份**：Spring Cloud Alibaba 提供的 Nacos 客户端 SDK 封装

- **职责**：它是我们与 Nacos 服务端通信的 **“连接器”**

- **为什么需要它**： 

  - 我们需要利用它获取 `ConfigService`，从而调用 Nacos 原生的 API（如 `getConfigAndSignListener`）来注册监听器

    如果没有它，我们就无法感知到 Nacos 上 JSON 文件的变化



**C. Fastjson2 (数据翻译官)**

- **身份**：阿里巴巴开源的高性能 JSON 处理库

- **职责**：负责 **“反序列化”**

- **为什么需要它**： 

  - Nacos 推送给我们的配置仅仅是一串 **“纯文本字符串”** (String)

    网关看不懂字符串，它只认 Java 对象 (`RouteDefinition`)

    因此，我们需要用 Fastjson2 将这串字符串转换成网关能识别的 Java 对象列表



###### 2. 逻辑链路：组件如何协作

在代码运行过程中，RouteDefinitionWriter、NacosConfigManager 和 Fastjson2 这三个组件是按照 **"监听 -> 解析 -> 更新"** 的流水线进行协作的



我们将其拆解为四个关键阶段：

**第一阶段：注册监听 (初始化)**

- **动作**：程序启动时，通过 `NacosConfigManager` 获取 `ConfigService`，从而调用 Nacos 原生的 API（如 `getConfigAndSignListener`）来注册监听器
- **逻辑**：向 Nacos 服务端发送一个请求，注册一个监听器 (Listener)
- **目的**：告诉 Nacos：“请帮我盯着 `gateway-routes.json` 这个 Data ID，一旦内容有变，马上通知我”



**第二阶段：变更触发 (运行中)**

- **动作**：你在 Nacos 控制台修改了 JSON 配置并发布
- **逻辑**：Nacos 服务端通过长轮询机制，将最新的配置内容（一串 JSON **字符串**）推送给我们的 Java 程序
- **结果**：触发了我们在第一阶段注册的 `Listener` 的回调方法 (`receiveConfigInfo`)



**第三阶段：数据翻译 (解析)**

- **动作**：调用 `Fastjson2` 工具类
- **逻辑**：`JSON.parseArray(configInfo, RouteDefinition.class)`
- **目的**：将那串网关看不懂的 **JSON 字符串**，转换成网关内存能识别的 **`List<RouteDefinition>` (路由定义对象列表)**



**第四阶段：内存刷新 (更新)**

- **动作**：调用 `RouteDefinitionWriter`
- **逻辑**：这是一个 **"先破后立"** 的过程
  1. **清理旧路由**：遍历上一次缓存的路由 ID，调用 `delete(id)` 方法，把旧的规则从内存中抹去
  2. **写入新路由**：遍历刚刚解析出来的路由对象，调用 `save(route)` 方法，把新的规则加载进内存
- **最终效果**：网关内部的路由表被重置，新规则立即生效，无需重启服务



###### 3. 预备知识：核心 API 方法速查

在即将编写的监听器代码中，会涉及到 **Nacos SDK**、**Gateway 内核** 以及 **响应式编程** 这三个领域的 API

提前熟悉一下



**A. Nacos SDK 相关 (负责连接)**

| 方法名                           | 所属类               | 作用                 | 通俗解释                                                     |
| -------------------------------- | -------------------- | -------------------- | ------------------------------------------------------------ |
| **`getConfigService()`**         | `NacosConfigManager` | 获取操作入口         | 拿到 Nacos 的操作核心对象，之后所有的配置增删改查都基于它    |
| **`getConfigAndSignListener()`** | `ConfigService`      | **拉取** 并 **监听** | 组合拳：<br />1. 获取当前最新的配置内容；<br />2. 注册一个监听器，确保后续配置变更时能自动触发回调 |
| **`receiveConfigInfo(String)`**  | `Listener` (接口)    | 变更回调             | Nacos 客户端监听到服务端配置变更后，会自动调用此方法，并将最新的配置内容以字符串形式传入 |



**B. Gateway 路由相关 (负责干活)**

| 方法名                            | 所属类                  | 作用          | 通俗解释                                    |
| --------------------------------- | ----------------------- | ------------- | ------------------------------------------- |
| **`save(Mono<RouteDefinition>)`** | `RouteDefinitionWriter` | 保存/更新路由 | 将一个路由定义对象写入 Gateway 的内存路由表 |
| **`delete(Mono<String>)`**        | `RouteDefinitionWriter` | 删除路由      | 根据 ID 将内存路由表中对应的路由移除        |



**C. 响应式编程 (WebFlux) 相关 (负责包装)**

Gateway 底层基于 Reactor 框架，其 API 参数风格与传统 Java 不同，以下两个操作是 **必选项**：

| 方法名                | 作用               | 为什么代码里必须写？                                         |
| --------------------- | ------------------ | ------------------------------------------------------------ |
| **`Mono.just(对象)`** | **构建异步数据流** | Gateway 的底层接口（如 `save`）要求的参数类型是 `Mono<T>` 而不是普通的 Java 对象 `T`<br />该方法的作用是将一个普通对象（如 `RouteDefinition`）包装成一个**包含单个元素的异步数据流**，以符合接口规范 |
| **`.subscribe()`**    | **触发任务执行**   | **这是最容易被遗漏的关键点**<br />响应式编程具有**惰性（Lazy）特征。仅仅调用 `writer.save()` 只是声明**了一个保存任务，但并没有启动它<br />只有调用了 `.subscribe()`，这个任务才会被真正提交给线程池去**执行**。如果不写，代码运行后没有任何效果 |



###### 4. 完整代码实现

以下是基于 **Fastjson2** 实现的完整代码。此代码是一个标准的 Spring Bean，启动后会自动开始工作



**依赖引入 (pom.xml)**

首先确保引入了 Fastjson2 依赖：

```xml
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2</artifactId>
    <version>2.0.35</version> <!-- 建议使用较新稳定版 -->
</dependency>
```



**Java 类实现 (DynamicRouteLoader.java)**

```java
/**
 * 动态路由加载器
 * 负责监听 Nacos 路由配置变更，并同步更新到 Gateway 内存中
 *
 */
@Slf4j
@Component
@RequiredArgsConstructor
public class DynamicRouteLoader {

    // 1. Gateway 提供的路由存储接口 (核心)
    private final RouteDefinitionWriter writer;
    
    // 2. Nacos 配置管理接口 (核心)
    private final NacosConfigManager nacosConfigManager;

    // 监听的 Data ID 和 Group (必须与 Nacos 控制台一致)
    private final String dataId = "gateway-routes.json";
    private final String group = "DEFAULT_GROUP";

    // 缓存已加载的路由 ID，用于在更新时清理旧路由
    private final Set<String> routeIds = new HashSet<>();

    /**
     * 初始化监听器
     * @PostConstruct: Spring 容器启动并初始化 Bean 后，立即执行此方法
     */
    @PostConstruct
    public void initRouteConfigListener() throws NacosException {
        // 获取 Nacos 配置服务
        ConfigService configService = nacosConfigManager.getConfigService();

        // 1. 注册监听器并首次拉取配置
        // 参数说明: dataId, group, 超时时间, 监听器实现
        String configInfo = configService.getConfigAndSignListener(dataId, group, 5000, new Listener() {
            @Override
            public Executor getExecutor() {
                // 返回 null 表示使用 Nacos 默认的内部线程池来执行回调
                return null;
            }

            @Override
            public void receiveConfigInfo(String configInfo) {
                // 【回调逻辑】一旦监听到配置变更，Nacos 会自动调用此方法
                log.info("监听到 Nacos 路由配置变更，准备刷新...");
                updateConfigInfo(configInfo);
            }
        });

        // 2. 项目首次启动时，手动加载一次配置，确保网关刚启动就有路由可用
        log.info("项目启动，首次加载 Nacos 路由配置...");
        updateConfigInfo(configInfo);
    }

    /**
     * 路由刷新逻辑 (核心业务)
     * 流程：解析 JSON -> 清理旧路由 -> 写入新路由
     */
    private void updateConfigInfo(String configInfo) {
        // 1. 解析配置 (String -> List<RouteDefinition>)
        // 使用 Fastjson2 的 parseArray 方法直接转换
        List<RouteDefinition> routeDefinitions = JSON.parseArray(configInfo, RouteDefinition.class);

        if (routeDefinitions == null) {
            log.warn("Nacos 路由配置为空或解析失败，跳过更新");
            return;
        }

        // 2. 清理旧路由 (先删)
        // 遍历缓存的旧 ID，调用 writer.delete() 删除
        for (String routeId : routeIds) {
            // Mono.just() 是响应式编程的包装，subscribe() 触发执行
            writer.delete(Mono.just(routeId)).subscribe(
                null, // 成功回调 (此处忽略)
                e -> log.error("删除旧路由失败: {}", routeId, e) // 失败回调
            );
        }
        routeIds.clear(); // 清空缓存

        // 3. 写入新路由 (后增)
        routeDefinitions.forEach(routeDefinition -> {
            // 调用 writer.save() 保存
            writer.save(Mono.just(routeDefinition)).subscribe();
            // 记录 ID 到缓存，方便下次删除
            routeIds.add(routeDefinition.getId());
        });

        log.info("路由刷新完成，共加载 {} 条路由规则", routeIds.size());
    }
}
```







#### 8. 高级特性（跨域配置与限流）

##### a. 跨域配置 (CORS)

###### 为什么需要 CORS？

- **背景**：在前后端分离架构中，前端通常运行在 `localhost:8080`，而网关运行在 `localhost:9527`
- **冲突**：当浏览器发现前端试图向不同端口（或域名）的后端发起请求时，出于 **同源策略 (Same-Origin Policy)** 的安全考虑，会默认拦截响应
- **现象**：控制台报错 `Access to XMLHttpRequest has been blocked by CORS policy`



###### 解决方案

- 在微服务架构中，**最佳实践** 是：**只在网关层配置 CORS，下游微服务不做任何处理** 
- **重复配置灾难**：
  - 如果网关配置了 CORS，下游微服务（如 OrderService）也加了 `@CrossOrigin`，浏览器会收到两个 `Access-Control-Allow-Origin` 头，导致请求直接报错




###### YAML 配置方式 (推荐)

Spring Cloud Gateway 提供了非常简单的全局 CORS 配置，直接修改 `application.yml` 即可

```yaml
spring:
  cloud:
    gateway:
      # 全局跨域配置
      globalcors:
        cors-configurations:
          '[/**]': # 匹配所有请求
            allowedOrigins: "*" # 允许所有的源 (生产环境建议指定具体域名，如 [https://my-app.com](https://my-app.com))
            # allowedOriginPatterns: "*" # 如果 Spring Boot 版本较高，可能需要用这个代替 allowedOrigins
            allowedMethods: # 允许的 HTTP 方法
              - GET
              - POST
              - PUT
              - DELETE
              - OPTIONS
            allowedHeaders: "*" # 允许所有的请求头
            allowCredentials: true # 允许携带 Cookie
            maxAge: 3600 # 预检请求(OPTIONS)的缓存时间，单位秒
```



##### b. 请求限流

###### 核心原理：令牌桶算法

Gateway 内置了基于 Redis 的限流过滤器 `RequestRateLimiter`。它使用的是 **令牌桶算法**：

1. **桶**：系统有一个存放令牌的容器
2. **放令牌**：系统以固定的速率往桶里放入令牌（比如每秒放 10 个）
3. **拿令牌**：请求进来时，必须从桶里拿走一个令牌才能被处理
4. **结果**：如果桶空了（令牌拿完了），请求就会被拒绝（HTTP 429 Too Many Requests）



###### 环境准备

限流依赖 **Redis** 来存储令牌桶的状态。请确保 `pom.xml` 中引入了 Redis 响应式依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```



###### 第一步：定义限流规则

我们需要告诉网关：**按什么维度来限流？** 是按 IP 限流？还是按用户 ID 限流？还是按接口 URL 限流？

这就需要编写一个 `KeyResolver` 的 Bean

**Java 代码示例 (按 IP 限流)**：

```java
@Configuration
public class RateLimitConfig {

    /**
     * 按请求 IP 限流
     * 返回的 String 就是 Redis 中的 Key
     */
    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> {
            // 获取请求的 HostAddress (IP)
            String ip = exchange.getRequest().getRemoteAddress().getAddress().getHostAddress();
            System.out.println("限流检测 IP: " + ip);
            return Mono.just(ip);
        };
    }
    
    // 如果想按用户 ID 限流 (假设参数中有 userId)
    /*
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("userId"));
    }
    */
}
```



###### 第二步：配置 YAML

在具体的路由中，应用 `RequestRateLimiter` 过滤器

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    database: 0

  cloud:
    gateway:
      routes:
        - id: rate_limit_route
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/limit/**
          filters:
            - name: RequestRateLimiter
              args:
                # 1. 引用我们在 JavaConfig 中定义的 Bean 名称
                key-resolver: "#{@ipKeyResolver}"
                # 2. 令牌填充速率 (每秒放多少个令牌) -> 限制平均 QPS
                redis-rate-limiter.replenishRate: 1
                # 3. 令牌桶容量 (允许在一秒内突发的最大请求数)
                redis-rate-limiter.burstCapacity: 10
                # 4. 每个请求消耗多少个令牌 (默认为 1)
                redis-rate-limiter.requestedTokens: 1
```

**配置解读**：

- `replenishRate: 1`：每秒产生 1 个令牌。这意味着稳态下，每秒只能通过 1 个请求
- `burstCapacity: 10`：桶最大能存 10 个令牌。这意味着如果前几秒没有请求，桶满了，下一秒突然来了 10 个请求，这 10 个都能瞬间通过（突发流量）。第 11 个会被拒绝



###### 测试效果

1. 启动 Redis
2. 快速刷新访问 `http://localhost:9527/payment/limit/test`
3. **结果**：正常情况下返回 200。当你刷新速度极快（超过 1次/秒）时，网关会返回 **HTTP 429 Too Many Requests**



## 负载均衡

LoadBalancer



## 流量治理与容错

### Sentinel

#### 1. Sentinel 全景概览

##### 1.1 背景与痛点

在深入代码之前，我们需要先明确一个核心问题：

- **在微服务架构已经非常成熟的今天，为什么我们还需要引入 Sentinel 这样的流量治理框架？它究竟解决了什么“至暗时刻”的问题？**



###### A. 微服务架构的阿喀琉斯之踵：雪崩效应

在单体架构向微服务架构演进的过程中，系统被拆分成了许多细小的服务单元

- 虽然这提升了开发效率和可扩展性，但也带来了一个致命的副作用 —— **服务调用链路的脆弱性**



**什么是雪崩效应？**

假设我们有一个电商系统，链路如下： `客户端 -> 订单服务 -> 库存服务 -> 数据库`

1. **初始状态**：一切正常
2. **故障发生**：**库存服务** 因数据库慢查询或网络波动，响应时间（RT）从 20ms 飙升到 2000ms，甚至超时
3. **资源耗尽**：**订单服务** 不断向库存服务发起请求。由于库存服务响应极慢，订单服务的 **Tomcat 线程** 被迫长时间等待
4. **级联失效**：随着请求量的涌入，订单服务的线程池被迅速占满，无法处理新的请求
5. **全盘崩溃**：此时，依赖订单服务的 **网关** 或 **前端** 也因为订单服务无响应而耗尽资源
6. 最终，一个底层服务的微小故障，沿着调用链路逆向传导，导致整个系统瘫痪

**这就是雪崩效应：基础服务的故障导致上层服务不可用，最终致使整个分布式系统崩溃**



###### B. 时代的眼泪：Hystrix 的局限性

为了解决雪崩问题，Netflix 开源了 Hystrix。它是第一代断路器模式的集大成者，但随着技术的发展，它的局限性日益凸显：

| 对比维度     | Netflix Hystrix            | 痛点分析                                                     |
| ------------ | -------------------------- | ------------------------------------------------------------ |
| **维护状态** | **已停止维护**             | 官方已不再开发新功能，Spring Cloud 2020.0 版本后已将其从核心组件中移除 |
| **隔离策略** | **线程池隔离**             | Hystrix 为每个依赖服务创建一个独立的线程池。虽然隔离性好，但带来了**巨大的上下文切换开销**<br />在 QPS 高的场景下，性能损耗严重 |
| **实时监控** | 延迟较高，聚合粒度粗       | Hystrix Dashboard 的数据展示往往有数秒延迟，难以应对秒杀级的瞬时流量分析 |
| **规则配置** | 硬编码或通过 Archaius 配置 | 动态修改规则比较繁琐，缺乏统一、可视化的动态规则管理控制台   |



###### C. Sentinel 的核心优势

Sentinel (Distributed System Guard) 是阿里巴巴开源的，面向分布式服务架构的流量控制组件。

- 它在设计之初就考虑到了 Hystrix 的不足，并针对“云原生”和“高并发”场景做了深度优化



**核心优势解析：**

1. **轻量级，高性能**

   - **信号量隔离**：Sentinel 默认不使用线程池隔离，而是通过 **并发线程数计数**（信号量机制）来实现隔离

   - **优势**：

     - 这种方式没有线程切换的开销，仅需对计数器进行原子操作，性能损耗几乎可以忽略不计

       这使得 Sentinel 非常适合 QPS 极高的场景（如阿里双十一零点流量）

   

2. **多样化的流量整形**

   - 不仅能“切断”流量（限流），还能“削峰填谷”
   - 支持 **Warm Up (预热)**：让系统流量缓慢增加，防止冷系统被瞬间压垮
   - 支持 **匀速排队**：将突发流量变成均匀的流量输出，保护下游服务

   

3. **极其丰富的应用场景**

   - 近乎完美地覆盖了国内互联网的复杂场景：秒杀限流、消息削峰填谷、集群流量控制、实时熔断等

   

4. **实时的监控与动态规则管理**

   - Sentinel Dashboard 提供秒级的实时监控（QPS、RT、拒绝 QPS 等）
   - 支持接入 Nacos、Apollo 等配置中心，实现规则的 **动态推送**，无需重启服务



##### 1.2 核心概念

Sentinel 的官方定义是：“以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性”

要实现这个目标，Sentinel 建立了一套抽象的模型

- 我们必须掌握四个最核心的关键词：**Resource (资源)**、**Rule (规则)**、**Entry (凭证)** 和 **Slot Chain (插槽链)**



###### A. 资源 (Resource) —— 核心

**定义**： 

- 资源是 Sentinel 的保护对象，是 Sentinel 能够识别的最小单位

  在代码中，任何需要保护的代码片段、方法、接口，甚至是一段 SQL，都可以被定义为一个“资源”

**它的本质**： 在 Sentinel 内部，每一个资源都对应唯一的名称（Identity）



**常见的资源形态**：

- **API 接口**：如 `/order/create`（这是最常见的）
- **普通方法**：Service 层的一个 Java 方法 `doSomething()`
- **后端服务**：如通过 Feign 调用的第三方 API



**代码映射 (伪代码预览)**：

```java
// "HelloWorld" 就是资源名称
try (Entry entry = SphU.entry("HelloWorld")) {
    // 这里是受保护的业务逻辑
    System.out.println("hello world");
} catch (BlockException e) {
    // 这里是流量被拦截后的处理逻辑 (被限流、被熔断等)
}
```



###### B. 规则 (Rules) —— 控制策略

**定义**： 规则定义了如何保护资源。 Sentinel 的设计亮点在于：**资源与规则是完全解耦的**

**解耦的意义**： 

- 你可以在代码中先定义好“这个方法是一个资源”，但此时不需要指定“限流阈值是多少”

  规则可以在 **运行时** 动态配置（通过控制台或配置中心），并实时生效。这解决了硬编码配置无法动态调整的痛点

**规则体系**：

- `FlowRule`：流量控制规则（QPS 超过 100 就拒绝）
- `DegradeRule`：熔断降级规则（响应时间超过 1s 就熔断）
- `SystemRule`：系统保护规则（CPU 使用率超过 80% 就全站限流）
- `ParamFlowRule`：热点参数规则
- `AuthorityRule`：来源访问控制规则



###### C. Entry (凭证) —— 运行时的令牌

**定义**： Entry 是 Sentinel 每一个请求的“令牌”或“凭证”

**工作原理**： 当你调用 `SphU.entry("资源名")` 时，Sentinel 会去申请一个 Entry

- **申请成功**：返回一个正常的 Entry 对象，代表当前请求 **通过** 了所有规则检查，可以执行业务逻辑
- **申请失败**：抛出 `BlockException` 异常，代表当前请求被 **拦截**（限流或熔断）



**Context (上下文)**： 

- 与 Entry 伴生的是 Context。它贯穿一次调用链路，用于记录调用来源（Origin）和调用链入口（EntranceNode），是链路流控模式的基础



###### D. 插槽链 (Slot Chain) —— 引擎

这是 Sentinel 最核心、最底层的架构设计，也是它高性能的秘密

- **Sentinel 的工作流程本质上就是一个责任链模式**

当一个请求进入 Sentinel 时，它会按顺序经过一系列的“插槽（Slot）”，每个插槽各司其职



**核心插槽解析 (按执行顺序)**：

1. **NodeSelectorSlot (节点选择器)**：

   - **作用**：负责收集资源的路径，并将这些资源的调用路径以树状结构存储起来，用于根据调用链路来限流
   - *简单说：构建调用链树*

   

2. **ClusterBuilderSlot (集群节点构建器)**：

   - **作用**：用于存储资源的统计信息以及调用者信息，例如该资源的 RT, QPS, thread count 等等，这些信息将用作为多维度限流，降级的依据
   - *简单说：准备好统计数据的容器*

   

3. **StatisticSlot (统计插槽) —— 【关键】**：

   - **作用**：这是 Sentinel 的 **数据统计中心**。它使用 **滑动时间窗口算法** 实时统计资源的运行指标（QPS、RT、线程数、异常数）
   - *注意*：它是后续所有校验规则的数据源头

   

4. **FlowSlot (流控插槽)**：

   - **作用**：根据 `FlowRule` 和 `StatisticSlot` 统计的数据，判断是否触发限流

   

5. **DegradeSlot (熔断插槽)**：

   - **作用**：根据 `DegradeRule` 和统计数据，判断是否触发熔断

   

6. **SystemSlot (系统保护插槽)**：

   - **作用**：校验系统整体负载（如 Load, CPU）

**架构图示**：

```
Request 
   ↓
[ NodeSelectorSlot ]   -> 构建调用链
   ↓
[ ClusterBuilderSlot ] -> 构建统计集群节点
   ↓
[ StatisticSlot ]      -> 实时指标计数 (Pass/Block/RT/Exception)
   ↓
[ FlowSlot ]           -> 检查流控规则 (QPS/Thread)
   ↓
[ DegradeSlot ]        -> 检查熔断规则 (RT/Error)
   ↓
[ SystemSlot ]         -> 检查系统规则 (Load/CPU)
   ↓
Business Logic (业务代码)
```



##### 1.3 架构设计：控制台与核心库的分离

Sentinel 在架构设计上采用了 **控制台 (Dashboard)** 与 **核心库 (Core)** 分离的模式

- 这与 Hystrix 强依赖于应用内部不同，Sentinel 提供了一个独立的控制台服务

###### A. 两大组件

Sentinel 的完整生态由两部分组成：

**a. 核心库 (Java Client)**

- **位置**：集成在微服务应用（如订单服务、库存服务）中，以 Jar 包形式存在

- **依赖**：`spring-cloud-starter-alibaba-sentinel`

- **职责**：

  - **拦截请求**：执行 1.2 节中提到的“插槽链”逻辑
  - **统计数据**：实时统计 QPS、RT 等指标
  - **执行规则**：判断请求是 Pass 还是 Block
  - **对外暴露 API**：默认监听 `8719` 端口，等待控制台的查询和指令

  

**b. 控制台 (Dashboard)**

- **位置**：一个独立运行的 Java Web 项目（SpringBoot 应用），通常以 `java -jar` 方式运行

- **职责**：

  - **可视化**：展示所有微服务的机器列表、实时 QPS 监控曲线
  - **规则管理**：提供图形化界面，允许运维人员动态增删改查限流/熔断规则

  

###### B. 交互原理

很多同学会好奇：*“我在控制台上点了一个按钮，我的微服务是怎么知道规则变了的？”*

这涉及到两者之间的通信机制：

1. **心跳与注册**：

   - 微服务启动后，会通过 `Transport` 模块，定时向控制台发送心跳包
   - **内容**：报告自己的 IP、端口、应用名称
   - **结果**：控制台的“机器列表”中就会出现这个服务

   

2. **监控数据拉取 (Pull)**：

   - 控制台不会一直存数据。当你打开“实时监控”页面时，控制台会通过 HTTP 请求向微服务的 `8719` 端口发起查询
   - 微服务返回当前内存中的统计数据

   

3. **规则推送 (Push)**：

   - 当你在控制台配置了一条限流规则，控制台会调用微服务的 API，将规则推送到微服务的 **内存** 中
   - **注意**：微服务接收到新规则后，会立即更新内存中的规则缓存，**即时生效**



**架构简图**：

```
+---------------------+           HTTP (Push Rules)          +----------------------+
|  Sentinel Dashboard |  --------------------------------->  |   MicroService A     |
|      (Server)       |                                      |      (Client)        |
|                     |  <---------------------------------  |                      |
|  [GUI: Rule Config] |           HTTP (Heartbeat)           |  [Sentinel Core]     |
|  [GUI: Metric View] |  <---------------------------------  |  [Command Center]    |
+---------------------+           HTTP (Pull Metrics)        |  (Listen Port: 8719) |
                                                             +----------------------+
```



###### C. 生产环境的“阿喀琉斯之踵”

这里必须着重讲一个 **巨坑**，也是初学者最容易产生误解的地方



**默认模式下的规则是“临时”的！**

- **现象**：你在控制台辛辛苦苦配了 10 条限流规则，运行得很完美。但是，一旦你 **重启了微服务**（或者重启了控制台），你会发现 **所有规则都消失了**
- **原因**：默认情况下，Sentinel Dashboard 推送的规则是保存在微服务的 **内存**（`MemoryRuleRepository`）中的。内存是易失的，重启即丢



**生产环境解决方案 (Preview)**： 在生产环境中，我们 **绝对不会** 直接使用默认模式。我们需要引入“配置中心”（如 Nacos, Apollo, Zookeeper）

- **流程**：控制台 -> Nacos -> 微服务
- **持久化**：规则持久化存储在 Nacos 数据库中，微服务重启后去 Nacos 拉取，从而保证规则不丢失
- *(这一块内容非常硬核，我们会在后面专门实战讲解，目前只需知道有这个问题即可)*



###### D. 关键端口：8719

- **作用**：Sentinel 在微服务本地启动的 `Command Center` 服务端口
- **逻辑**：默认使用 `8719`。如果 8719 被占用（比如一台机器部署了多个微服务），它会依次扫描 `8720`, `8721`... 直到找到可用端口
- **避坑**：如果你的微服务部署在 Docker 或 K8s 容器中，必须确保 **Dashboard 能访问到容器内的 8719+ 端口**，否则你会看到“实时监控”页面一片空白





#### 2. 环境搭建与 Hello World

##### 2.1 服务端启动：部署 Sentinel 控制台

在上一章我们了解到，Sentinel 控制台（Dashboard）是一个独立的 Java Web 应用。这一节我们将亲手把它运行起来



###### a. 获取控制台组件

Sentinel 控制台本质上是一个标准的 Spring Boot Jar 包

- **官方地址**：GitHub Releases 页面 (alibaba/Sentinel)
- **版本选择**：建议与你的 Spring Cloud Alibaba 版本对应的 Sentinel 版本保持一致（通常下载最新稳定版即可，如 1.8.6 或 1.8.8）
- **文件名示例**：`sentinel-dashboard-1.8.6.jar`



###### b. 标准启动

下载完成后，打开终端（CMD 或 Shell），进入 Jar 包所在目录。 由于它是基于 Spring Boot 开发的，直接使用 `java -jar` 命令即可启动

**基础启动命令**：

```bash
java -jar sentinel-dashboard-1.8.6.jar
```

- **默认端口**：`8080`
- **默认账号**：`sentinel`
- **默认密码**：`sentinel`



###### c. 进阶配置启动

在实际开发或生产环境中，默认的 8080 端口经常被占用，或者我们需要修改默认密码。我们可以通过 JVM 参数（`-D`）来覆盖默认配置

**常用启动参数清单**：

| 参数名                               | 默认值   | 作用说明                      |
| ------------------------------------ | -------- | ----------------------------- |
| `-Dserver.port`                      | 8080     | 指定控制台 Web 服务的运行端口 |
| `-Dsentinel.dashboard.auth.username` | sentinel | 指定登录用户名                |
| `-Dsentinel.dashboard.auth.password` | sentinel | 指定登录密码                  |
| `-Dserver.servlet.session.timeout`   | 30m      | Session 过期时间              |

**生产环境启动示例**： (假设我们要把端口改为 `8858`，并将密码修改为 `admin123`)

```bash
java -Dserver.port=8858 \
-Dsentinel.dashboard.auth.username=admin \
-Dsentinel.dashboard.auth.password=admin123 \
-jar sentinel-dashboard-1.8.6.jar
```



###### d. 验证是否成功

1. 启动命令执行后，看到 Spring Boot 的 Banner 和 `Started DashboardApplication` 日志即表示启动成功
2. 打开浏览器访问：`http://localhost:8858` (如果你修改了端口)
3. 输入账号密码登录

**初次登录界面**： 你会发现界面左侧菜单栏空空如也，只有“机器列表”等少数几项

- **原因**：Sentinel 是 **懒加载** 的。因为目前还没有任何微服务接入，也没有任何流量产生，所以控制台是空的。这是正常现象，不要惊慌



###### 5. 避坑指南

- **坑点一：端口冲突 (Port 8080)**
  - **现象**：启动报错 `Web server failed to start. Port 8080 was already in use.`
  - **解决**：8080 是 Web 开发最常用的端口，极易冲突
    - **强烈建议 **养成习惯，启动 Sentinel 时总是指定一个特殊端口（如 `8858`），避免与你正在开发的本地微服务冲突
- **坑点二：JDK 版本**
  - Sentinel Dashboard 1.8.x 需要 **JDK 1.8** 或以上版本。
    - 如果你的环境是 JDK 17 或 21，通常也能兼容运行，但如果启动报错，请检查 Java 环境变量
- **坑点三：VM 内存设置**
  - 控制台本身也是一个 Java 进程。如果你的服务器内存较小，建议限制一下堆内存大小，防止 OOM
  - 示例：`java -Xms256m -Xmx516m -Dserver.port=8858 -jar ...`



##### 2.2 客户端接入：微服务集成 Sentinel

服务端跑起来了，但它现在还是个“光杆司令”。我们需要配置微服务，让它主动向控制台“报到”

###### a. 引入 Maven 依赖

在你的 Spring Cloud 微服务项目（例如 `order-service`）的 `pom.xml` 中，引入 Sentinel 的核心 Starter

**核心依赖**：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

**版本兼容性警告 —— 巨坑预警**： 

- Spring Cloud Alibaba (SCA) 对 Spring Boot 和 Spring Cloud 的版本有严格要求
  - 乱搭版本会导致 `ClassNotFoundException` 或 `MethodNotFoundException`

- **SCA 2021.x** -> Spring Boot 2.6.x
- **SCA 2.2.x** -> Spring Boot 2.3.x
- **推荐**：严格遵循 [Spring Cloud Alibaba 版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/版本说明) 进行搭配



###### b. YAML 配置

引入依赖后，我们需要在 `application.yml` (或 `application.properties`) 中告诉微服务三件事：

1. 控制台在哪里？
2. 我自己是谁（应用名称）？
3. 我通过哪个端口与控制台交互？



**配置示例**：

```yaml
server:
  port: 8001  # 微服务本身的端口

spring:
  application:
    name: order-service  # 【关键】应用名称，将显示在控制台左侧列表中

  cloud:
    sentinel:
      transport:
        # 【关键】控制台的地址 (IP:Port)
        dashboard: localhost:8858
        # 【关键】客户端监控 API 的端口 (默认 8719)
        # 如果被占用，Sentinel 会自动 +1 扫描 (8720, 8721...)
        port: 8719
        # 客户端 IP，如果部署在虚拟机/容器中，需指定宿主机能访问的 IP
        # client-ip: 192.168.1.10 
      # 是否开启饥饿加载 (默认 false)
      eager: true
```

**配置项深度解析**：

- **`dashboard`**：
  - 指定 Sentinel 控制台的地址。客户端启动后，会向这个地址发送心跳包
- **`port` (Client IP Port)**：
  - **作用**：这是微服务本地开启的一个 HTTP Server 端口
  - **Why?** 控制台需要从微服务拉取监控数据（Pull 模式），或者推送规则（Push 模式），都需要通过这个端口进行通信
  - **注意**：这不是你的业务端口（8001），而是一个管理端口（默认 8719）



###### c. 为什么启动后控制台还是空的？

当你配置好依赖和 YAML，成功启动微服务后，兴奋地刷新控制台，通常会发现 **依然什么都没有**

**这是正常现象！**

- **机制**：Sentinel 默认采用 **懒加载机制 (Lazy Loading)**
- **原因**：只有当微服务 **接收到流量**（即有接口被调用）时，Sentinel 才会正式初始化，并向控制台发送心跳注册
- **解决**：
  1. 手动调用一次微服务的任意接口（如 `http://localhost:8001/hello`）
  2. 或者在 YAML 中配置 `spring.cloud.sentinel.eager: true`（取消懒加载，启动即注册，**推荐在测试环境开启**）



###### d. 避坑指南

- **坑点一：依赖冲突**
  - 如果你同时引入了 `spring-boot-starter-actuator`，请确保没有禁用 Sentinel 的端点
- **坑点二：循环依赖 (Circular Dependency)**
  - 在 Spring Boot 2.6+ 版本中，SCA 的某些低版本可能存在 Bean 循环依赖问题
  - **解决**：在 `application.yml` 中添加 `spring.main.allow-circular-references: true`
- **坑点三：Docker/K8s 环境下的 IP 问题**
  - 如果微服务运行在容器里，控制台运行在宿主机。微服务上报的 IP 是容器内部 IP（如 `172.17.0.x`），控制台无法访问该 IP 的 8719 端口
  - **解决**：需要通过 `spring.cloud.sentinel.transport.client-ip` 显式指定宿主机可访问的 IP，或者使用 K8s 的 Service 发现机制



##### 2.3 初体验：编写第一个被保护的接口

在 Spring Cloud Alibaba 中，Sentinel 已经对 Spring MVC 做了极好的集成。**默认情况下，所有的 HTTP 接口（URL）都会自动被定义为资源**

这意味着你不需要写任何一行 Sentinel 相关的代码，就可以直接开始流控



###### a. 编写测试接口

我们在 `order-service` 中创建一个标准的 Controller

**代码示例**：

```java
package com.example.order.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.concurrent.TimeUnit;

@RestController
public class OrderController {

    // 资源名默认为请求路径："/order/hello"
    @GetMapping("/order/hello")
    public String hello() {
        return "Hello, Sentinel!";
    }

    // 模拟一个稍微耗时的接口，用于后续测试熔断
    @GetMapping("/order/sleep")
    public String sleep() throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(200); // 模拟业务处理耗时 200ms
        return "Sleep finished.";
    }
}
```



###### b. 触发流量与控制台验证

1. **启动服务**：启动 `order-service`
2. **产生流量**：打开浏览器或使用 Postman，快速刷新访问几次 `http://localhost:8001/order/hello`
3. **查看控制台**：
   - 回到 Sentinel Dashboard (`localhost:8858`)
   - 刷新左侧菜单，你应该能看到 `order-service` 出现了（懒加载生效）
   - 点击 **“实时监控”**，你应该能看到刚才访问的 QPS 曲线图（蓝色代表通过的请求）



###### c. 动手：配置第一条流控规则

光看监控不过瘾，我们来试着“拦截”它

1. **找到资源**：在左侧菜单点击 **“簇点链路” (Cluster Node)**。这是最常用的页面，它列出了所有被扫描到的资源
2. **新增流控**：在 `/order/hello` 这一行的右侧，点击 **“+流控”** 按钮
3. **填写参数**：
   - **针对来源**：`default`
   - **阈值类型**：`QPS`
   - **单机阈值**：`1` (表示每秒只允许 1 个请求通过)
   - 点击“新增”



###### d. 验证限流效果

现在，回到浏览器，疯狂刷新 `http://localhost:8001/order/hello`

**预期结果**：

- 第 1 次请求：显示 `Hello, Sentinel!`
- 第 2 次请求（1秒内）：页面显示 `Blocked by Sentinel (flow limiting)`

**恭喜你！** 你已经成功完成了 Sentinel 的“Hello World”



###### e. 避坑指南

- **现象：簇点链路是空的**

  - **原因 1**：你还没访问接口。请先狂刷几次接口，再刷新控制台页面
  - **原因 2**：端口不通。检查微服务日志，看是否有 `Registration failed` 或者无法连接 `localhost:8719` 的报错。确保 Docker/防火墙配置正确

- **现象：默认拦截提示太丑了**

  - Sentinel 默认返回的 `Blocked by Sentinel ...` 只是一个纯字符串

    在实际项目中，我们肯定需要返回 JSON 格式的统一响应（如 `{"code": 429, "msg": "系统繁忙"}`）

    这需要实现 `BlockExceptionHandler` 接口进行全局异常处理（我们会在后续高级章节详细讲解）



#### 3. 流量控制 —— 流量整形

##### 3.1 流控模式：谁惹事，谁负责？

在 Sentinel 控制台添加流控规则时，你会看到一个不起眼的选项叫 **“流控模式”**

- 它决定了 Sentinel 在统计 QPS 时，到底统计的是 **谁** 的数据，以及触发限流后要制裁 **谁**

Sentinel 支持三种流控模式：**直接 (Direct)**、**关联 (Relational)**、**链路 (Link)**



###### 1. 直接模式 (Direct) —— 默认

**含义**： 这是最简单、最直观的模式

- **统计对象**：当前资源本身
- **限流对象**：当前资源本身
- **逻辑**：当 **资源 A** 的 QPS 超过阈值，直接拒绝 **资源 A** 的请求

**应用场景**： 90% 的日常限流场景

- 例如：`/order/create` 接口承载能力有限，配置 QPS > 100 直接拒绝



###### 2. 关联模式 (Relational) —— 围魏救赵

**含义**：

- **统计对象**：**关联资源** (Ref Resource)
- **限流对象**：当前资源本身
- **逻辑**：当 **资源 B (关联资源)** 的 QPS 超过阈值时，**限流 资源 A**



**为什么要这么做？(Why)** 这通常用于处理存在 **资源争抢** 或 **优先级区分** 的场景



**核心场景：支付与下单** 假设你的“支付接口”和“下单接口”共用同一个数据库连接池

- **痛点**：当“支付接口”流量激增（例如双11 0点），数据库压力巨大。此时如果允许大量新的“下单请求”进来，会进一步压垮数据库，导致已经下单的用户无法支付
- **策略**：优先保“支付”（高优先级），牺牲“下单”（低优先级）
- **配置**：
  - **当前资源**：`/order/create` (下单)
  - **流控模式**：关联
  - **关联资源**：`/order/pay` (支付)
  - **阈值**：10
- **效果**：一旦 `/order/pay` 的 QPS 超过 10，Sentinel 就会限制 `/order/create` 的访问，为支付业务预留资源



###### 3. 链路模式 (Link) —— 精准打击

**含义**：

- **统计对象**：当前资源，但只统计 **特定入口** 进来的流量
- **限流对象**：当前资源
- **逻辑**：只有从 **入口 A** 进入 **资源 C** 的 QPS 超过阈值时，才限流，从入口 B 进来的请求不受影响



**应用场景：公共服务限流** 假设你有一个公共服务 `getUserInfo()`，它被两个 Controller 调用：

1. `OrderController` -> `getUserInfo()`
2. `PayController`   -> `getUserInfo()`

- **需求**：`PayController` 是核心业务，不能限流；但 `OrderController` 查询太频繁，需要限制它对 `getUserInfo()` 的调用
- **配置**：
  - **当前资源**：`getUserInfo`
  - **流控模式**：链路
  - **入口资源**：`/order/get`
- **效果**：只有走 `/order/get` 这条路过来的请求会被统计和限流



###### 4. 巨坑指南

**关于链路模式的失效问题**： 这是 Spring Cloud Alibaba 中最著名的 **坑** 之一

**现象**：你配置了链路模式，指定了入口，但无论怎么测，限流都不生效

**原因**：

- Spring Cloud Alibaba 默认将所有的 HTTP 请求上下文入口 (Context Name) 统一聚合为了一个名称：`sentinel_spring_web_context`

  这导致 Sentinel 无法区分请求究竟是从哪个 Controller 进来的，链路模式因此失效

**解决方案**： 你需要在 `application.yml` 中关闭这种聚合功能：

```yaml
spring:
  cloud:
    sentinel:
      web-context-unify: false  # 【关键】默认为 true，改为 false 才能使用链路流控
```



##### 3.2 流控效果：流量整形的核心算法

当 QPS 超过了我们设定的阈值，Sentinel 会怎么做？ 很多人以为只有“直接拒绝”这一种结果

- 其实 Sentinel 提供了三种核心流控效果，分别对应不同的底层算法和业务场景



###### 1. 快速失败—— 默认效果

- **对应算法**：计数器 / 令牌桶 (Token Bucket) 的简化版
- **行为**：当 QPS 超过阈值，直接抛出 `FlowException`
- **场景**：
  - 最通用的场景
  - 对于那些承载能力是硬性的（如 CPU 跑满、线程池满）的系统，超过就必须拒绝，否则系统会挂
- **代码表现**：客户端收到异常，通常需要捕获并返回友好提示



###### 2. Warm Up (预热/冷启动) —— 慢启动防猝死

- **对应算法**：**令牌桶算法** 的变体

- **痛点**： 

  - 当系统长时间处于低水位（比如大促前的深夜），各种缓存（Redis 缓存、JVM JIT 编译代码、数据库连接池）都处于“冷”状态

    如果此时瞬间涌入海量流量（例如秒杀开始），系统可能在几秒钟内就被压垮，因为缓存还没热起来，请求全打到了数据库

- **行为**：

  - 设置一个 **冷启动时长 (`warmUpPeriodSec`)**
  - 流量刚来时，初始阈值 = `QPS阈值 / coldFactor` (默认为 3)
  - 在 `warmUpPeriodSec` 时间内，阈值会**线性攀升**，直到达到设定的最大 QPS

- **图解**：

  ```
  QPS 阈值
     ^
  100|                 /----------- (最终阈值)
     |                /
   33|  -------------/ (初始阈值 = 100/3)
     |
     +-----------------------------> 时间
           <--预热期-->
  ```

- **场景**：秒杀系统、应用重启后的瞬间保护



###### 3. 排队等待 —— 削峰填谷

- **对应算法**：**漏桶算法 (Leaky Bucket)**

- **痛点 (Why)**： 有些场景下，流量是 **脉冲式** 的（一会有一会无）

  - **例子**：

    - 消息队列（MQ）消费。假设每秒 1000 个消息推过来，但你的数据库每秒只能处理 200 个

      如果直接拒绝多余的 800 个，消息就丢失了（或者要重试），这不合理

  - **期望**：我希望能把这 1000 个请求排个队，以每秒 200 个的匀速处理

  

- **行为**：

  - 严格控制请求通过的间隔时间
  - 如果当前请求来得太快，就让它在队列里 **排队等待**
  - 如果排队时间超过了 `超时时间 (maxQueueingTimeMs)`，这部分请求才会被丢弃

- **场景**：**削峰填谷**。常用于 MQ 消费端限流，或者打向第三方不稳定 API 的请求（防止把对方打挂）

- **限制**：此模式下，**阈值类型必须是 QPS**，不能是线程数



##### 3.3 实战：如何应对突发流量与脉冲流量

在生产环境中，流量永远不会像心电图那样平滑。我们将面临两类最头疼的流量形态：**突发激增** 和 **脉冲式锯齿 (Pulse)**

- 下面通过两个真实案例，教你如何“对症下药”



###### 场景：秒杀大促:应对“冷系统”的突发流量

- **背景描述**： 

  - 假设你的微服务刚刚**重启**上线，或者在深夜低负载运行了很久（JVM 已经缩容，缓存已经过期）

    突然，双十一零点到了，秒杀活动开始，流量从 0 瞬间飙升到 10,000 QPS

  

- **问题分析**： 如果使用默认的 **“快速失败”** 模式：
  1. 系统刚启动，数据库连接池还没初始化满，代码还没被 JIT 编译成机器码，热点数据还没进 Redis
  2. 此时系统极其脆弱，原本能抗 10,000 QPS，现在可能只能抗 1,000 QPS
  3. 瞬间涌入的流量会直接把“冷系统”打死，导致 **开局即宕机**



- **解决方案**： 使用 **Warm Up (预热)** 模式，给系统一个“热身”的机会



- **配置实战**：

  - **资源名**：`/seckill/order`

  - **阈值类型**：QPS

  - **单机阈值**：`1000` (系统的长期稳定水位)

  - **流控效果**：`Warm Up`

  - **预热时长**：`10` (秒)

  

- **效果演示**：
  1. **第 0-1 秒**：Sentinel 将阈值压低到 `1000 / 3 = 333` QPS。超过 333 的请求直接拒绝，保护脆弱的系统
  2. **第 1-10 秒**：阈值从 333 缓慢线性爬升。系统随着流量增加，逐步建立连接、预热缓存、编译代码
  3. **第 10 秒后**：阈值恢复到满血的 `1000` QPS，系统已经完全热身完毕，满负载运行



###### 场景:MQ削峰填谷:应对“脉冲”流量

- **背景描述**： 你的系统是一个**下游服务**，专门负责把 MQ (RocketMQ/Kafka) 里的订单消息写入 MySQL 数据库

  - **上游**：MQ 推送速度极快，有时每秒推 2000 条，有时每秒 0 条（脉冲式）

  - **自身**：MySQL 的写入瓶颈是每秒 500 TPS



- **问题分析 (The Problem)**： 如果使用默认的 **“快速失败”** 模式：
  1. 当 MQ 推送 2000 条时，Sentinel 限制 500 条通过	
  2. **剩下的 1500 条被拒绝**（抛出异常）
  3. MQ 消费者捕获异常，触发 **重试机制**
  4. 这会导致消息大量积压，且重试流量会和新流量叠加，引发“重试风暴”，甚至导致数据重复消费或乱序



- **解决方案**： 使用 **排队等待 (Rate Limiter / 漏桶)** 模式。我们不希望丢弃请求，而是希望它们“排队慢点走”



- **配置实战**：

  - **资源名**：`handle_mq_message`

  - **阈值类型**：QPS

  - **单机阈值**：`500` (数据库的写入极限)

  - **流控效果**：`排队等待`

  - **超时时间**：`2000` (毫秒)



- **效果演示**：
  1. **瞬时流量**：MQ 突然推过来 1000 个请求
  2. **Sentinel 干预**：
     - 第 1-500 个请求：匀速通过，每 2ms 放行一个（1000ms / 500 = 2ms）
     - 第 501-1000 个请求：**不会被拒绝**，而是进入队列沉睡等待
  3. **结果**：对于数据库来说，它感受到的是连绵不断的、稳定的 500 QPS 流量，没有任何尖刺，也没有任何消息丢失（除非排队太久超时）



###### 避坑指南：排队等待的副作用

- 虽然“排队等待”看起来很美好，但有两个 **致命限制**：

  1. **内存风险**： 

     - 排队是在内存中进行的。如果超时时间 (`maxQueueingTimeMs`) 设置得太大（例如 10秒），且请求对象很大，会导致内存暴涨，甚至 OOM

  2. **RT (响应时间) 变长**： 

     - 对于排在队尾的请求，它的响应时间 = `业务执行时间` + `排队等待时间`

       如果你的上游系统（例如前端页面）对超时很敏感（如 1秒超时），而你在 Sentinel 让它排队等了 2秒，那么上游会直接报 ReadTimeout，此时后端的处理就变成了无效计算

- **最佳实践**： 
  - `排队等待` 模式最适合 **异步调用** 场景（如 MQ 消费、定时任务批处理），通常 **不建议** 用于直接面向用户的实时 HTTP 接口（除非用户能接受等待转圈圈）



#### 4. 熔断降级

##### 4.1 Sentinel 与 Hystrix 熔断策略对比

在微服务领域，提到熔断，大家的第一反应往往是 Netflix Hystrix。虽然 Hystrix 已经停止维护，但它的“线程池隔离”思想影响深远

- Sentinel 作为后来者，在熔断降级的设计上选择了完全不同的路线。理解这两者的差异，是你正确使用 Sentinel 的前提



###### 1. 什么是熔断降级？

想象一下你家里的 **电路保险丝**

- **正常情况**：电流通过，电器工作
- **异常情况**：某个电器短路，电流过载
- **熔断**：保险丝自动烧断，切断电流，防止整个房子的线路起火烧毁



在微服务中：

- **资源**：微服务之间的调用（如 `OrderService` -> `InventoryService`）

- **熔断**：

  - 当 `InventoryService` 响应过慢或报错率过高时，Sentinel 会 **主动切断** 对它的调用，后续请求直接返回失败（或兜底数据），

    不再去等待那个“半死不活”的服务，从而释放调用方的线程资源



###### 2. Hystrix 的核心：线程池隔离

Hystrix 崇尚“舱壁模式” (Bulkhead Pattern)，它通过 **资源隔离** 来实现熔断

- **实现方式**： Hystrix 为每一个依赖服务（Dependency）创建一个独立的 **线程池**
  - `ServiceA` 线程池：10个线程
  - `ServiceB` 线程池：20个线程
- **工作流程**： 请求 -> Hystrix 命令 -> **提交到特定线程池** -> 执行业务
- **优点**：
  - **强隔离**：`ServiceA` 的线程池满了，绝不会影响 `ServiceB` 的调用。即使 `ServiceA` 挂死，主业务线程也能立即返回（因为是异步提交）
- **缺点 (痛点)**：
  - **上下文切换 (Context Switch) 开销巨大**：
    - 每次请求都要把任务从 Tomcat 线程切换到 Hystrix 线程，执行完再切回来。在高并发场景（QPS > 1000）下，这种损耗是不可接受的
  - **线程池管理复杂**：你需要预估每个依赖服务需要多少线程，配多了浪费，配少了容易被拒绝



###### 3. Sentinel 的核心：并发线程数控制 & 响应时间

Sentinel 放弃了线程池隔离，采用了更轻量级的方案：**信号量隔离 + 响应时间检测**

- **实现方式**： 

  - Sentinel 不创建额外的线程池，直接在 **当前调用线程** 中执行统计和判断

    它通过统计 **当前资源正在执行的线程数**（即并发数）是否超过阈值，来决定是否拒绝新的请求

- **工作流程**： 请求 -> Sentinel 检查 (当前并发数 < 阈值?) -> **直接在当前线程执行业务**

- **优点**：

  - **极致性能**：没有线程切换，损耗几乎为零。这使得 Sentinel 能支撑阿里双十一级别的高流量
  - **简单**：不需要预估复杂的线程池参数

- **缺点**：

  - **隔离性稍弱**：

    - 虽然限制了总并发数，但如果下游服务响应非常慢，调用方的线程还是会被阻塞住（直到超时）

      但 Sentinel 通过“慢调用比例”熔断策略完美补足了这一点（下一节讲）



###### 4. 核心对比总结

| 特性         | Hystrix                  | Sentinel                      |
| ------------ | ------------------------ | ----------------------------- |
| **隔离策略** | 线程池隔离 (Thread Pool) | 信号量隔离 (并发线程数)       |
| **熔断触发** | 异常比例                 | 慢调用比例、异常比例、异常数  |
| **性能损耗** | 高 (线程切换)            | 极低 (原子计数)               |
| **异步调用** | 支持 (CompletableFuture) | 支持                          |
| **流量整形** | 不支持 (只有拒绝)        | 支持 (WarmUp, 排队等)         |
| **系统负载** | 不感知                   | 支持系统自适应保护 (Load/CPU) |



###### 5. 为什么 Sentinel 是更好的选择？

- Hystrix 的线程池隔离在早期的分布式系统中非常有用，但在现在的云原生、Mesh 化趋势下，Sidecar 模式或 Java Agent 模式往往无法承受过多的线程开销
- Sentinel 的设计哲学是：**不假设你会怎么执行业务（同步/异步），只关心你的业务执行结果（RT、异常）**
  - 这种轻量级的设计让它不仅能做熔断，还能做流量控制，成为更通用的流量治理框架



##### 4.2 三大熔断策略

在 Sentinel 控制台的“降级规则”弹窗中，**“熔断策略”** 是必选的第一项。它决定了 Sentinel 依据什么指标来判断服务是否不可用

- Sentinel 提供三种策略：**慢调用比例**、**异常比例**、**异常数**

###### 1. 慢调用比例 —— 性能守护神

这是 Sentinel 最推荐、也是最常用的策略，因为它直接反映了用户体验（响应慢）

- **核心逻辑**： 

  - 以 **RT (响应时间)** 为参考依据。 如果请求的响应时间超过了设定的阈值，这个请求就被定义为 **“慢调用”**

    当单位统计时长内的 **慢调用比例** 超过阈值，就会触发熔断

  

- **关键配置参数**：

  1. **最大 RT (Response Time)**：例如 `200ms`。超过这个时间的请求才算“慢调用”
  2. **比例阈值 (Threshold)**：范围 `[0.0, 1.0]`。例如 `0.8` 代表 80%
  3. **最小请求数 (Min Request Amount)**：例如 `5`
  4. **熔断时长 (Duration)**：例如 `10s`

  

- **触发条件 (AND 关系)**：

  1. 当前统计窗口内的请求总数 >= **最小请求数**
  2. (慢调用数 / 总请求数) > **比例阈值**

- **场景实战**： 

  - 你的接口通常 50ms 返回。如果数据库死锁导致接口耗时飙升到 5秒，你希望系统能快速切断调用，防止线程池被这些 5秒 的请求占满。此时就用此策略

###### 2. 异常比例 —— 错误感知

- **核心逻辑**： 以 **Java 异常** 为参考依据。 当单位统计时长内的 **异常比例** 超过阈值，触发熔断
- **关键配置参数**：
  1. **比例阈值**：范围 `[0.0, 1.0]`。例如 `0.5` 代表 50%
  2. **最小请求数**：例如 `5`
  3. **熔断时长**：例如 `10s`
- **触发条件**：
  1. 当前请求总数 >= **最小请求数**
  2. (异常数 / 总请求数) > **比例阈值**
- **场景实战**： 调用第三方支付接口，如果对方挂了，返回全是 500 或者 `SocketTimeoutException`。此时应该快速熔断，不再尝试调用
- **注意**：`BlockException` (Sentinel 自身的限流异常) **不** 会计算在内，只有业务代码抛出的异常才算



###### 3. 异常数 —— 零容忍

- **核心逻辑**： 以 **异常的绝对数量** 为参考依据。 只要异常的数量达到指定的值，就触发熔断

- **关键配置参数**：

  1. **异常数阈值**：例如 `5`
  2. **熔断时长**：例如 `10s`

- **差异点 (VS 异常比例)**： 

  - “异常比例”受 QPS 影响。如果 QPS 极低（例如 1分钟只有 1个请求），且这个请求报错了，那么异常比例就是 100%，会立即触发熔断

    有时候我们不希望因为偶尔 1 个请求失败就熔断整个服务。

    但是，**“异常数”策略** 正好相反，它适合 QPS 较小的场景，或者对错误非常敏感的场景



###### 4. 巨坑指南：为什么我的熔断不生效？

这是新手最常遇到的问题：*“我配置了异常比例 50%，我故意抛异常，为什么没熔断？”*

**罪魁祸首：最小请求数 (MinRequestAmount)**

Sentinel 为了防止“刚启动还没几个请求就把自己熔断了”这种误判情况，设置了一个 **静默保护机制**

- **默认值**：Sentinel 1.8.0 之前默认是 5，在控制台如果不显式配置，很容易忽略

- **现象**： 

  - 你手速不够快，每秒只点了一下接口（QPS=1）

    即使你每次都报错（异常比例 100%），但因为 **当前总请求数 (1) < 最小请求数 (5)**，Sentinel 认为样本不足，**绝不会触发熔断**

- **解决方案**： 测试时，使用 Postman 的 Runner 或 JMeter 进行高并发测试，确保 QPS 超过最小请求数



##### 4.3 熔断状态机

- 熔断器的核心价值不仅在于“切断风险”，更在于“自动恢复”。Sentinel 的熔断机制由三种状态组成，它们之间的流转逻辑构成了 Sentinel 稳定性的基石

###### 1. 三种状态详解

- **Closed (关闭状态)**

  - **含义**：这是 **正常状态**
  - **行为**：保险丝是连通的，请求正常通过。Sentinel 在后台默默统计 QPS、RT 和异常数，随时准备应对突发状况
  - **比喻**：家里电路正常，灯火通明

  

- **Open (开启状态)**

  - **含义**：这是 **熔断状态**
  - **行为**：保险丝已烧断。请求进入时，**不再调用后端服务**，而是直接抛出 `DegradeException` 异常（或执行 fallback 逻辑）
  - **持续时间**：取决于你在规则中配置的 **“熔断时长”** (timeWindow)
  - **比喻**：跳闸了，全屋停电，防止电器烧毁

  

- **Half-Open (半开启状态)**

  - **含义**：这是一个**“试探”状态**，也是 Sentinel 最机智的设计
  - **行为**：熔断时长结束后，Sentinel 不会立即让所有流量涌入（万一后端还没好呢？），而是切换到 Half-Open 状态，**仅允许一个请求**通过作为“侦察兵”
  - **比喻**：电工小心翼翼地合上闸，先开一盏灯试试看还会不会短路



###### 2. 状态流转全过程

让我们通过一次完整的故障模拟，来看清状态是如何流转的：

**第一阶段：触发熔断 (Closed -> Open)**

1. 系统处于 **Closed** 状态，一切正常
2. 突然，数据库崩溃，接口响应时间从 50ms 飙升到 5秒
3. Sentinel 监测到“慢调用比例”超过阈值
4. **动作**：将状态置为 **Open**
5. **结果**：后续请求直接被拦截，不再去请求那个卡死的数据库，从而释放了线程资源



**第二阶段：熔断计时 (Open)**

1. 系统保持在 **Open** 状态
2. 计时器开始倒数（例如你配置了 `熔断时长 = 10s`）
3. 在这 10秒 内，无论发多少请求，都会直接返回失败



**第三阶段：尝试恢复 (Open -> Half-Open)**

1. 10秒倒计时结束
2. **动作**：Sentinel 自动将状态切换为 **Half-Open**
3. **结果**：此时系统 **静默等待** 下一个请求的到来



**第四阶段：生死时刻 (Half-Open -> ?)** 这时候，一个新的请求（侦察兵）进来了：

- **情况 A：侦察兵“牺牲”了 (Half-Open -> Open)**
  - 如果这个请求依然很慢，或者依然抛出异常（说明后端还没恢复）
  - **动作**：Sentinel 判定恢复失败，状态立刻切回 **Open**，并 **重新重置** 10秒 的计时器
  - **循环**：系统继续熔断，等待下一个 10秒
- **情况 B：侦察兵“凯旋”了 (Half-Open -> Closed)**
  - 如果这个请求响应很快，且没有抛异常（说明后端已经恢复健康）
  - **动作**：Sentinel 判定恢复成功，状态切回 **Closed**，结束熔断
  - **结果**：后续的所有请求重新开始正常通行



###### 3. 图解状态机

```
                  (1) 达到阈值 (慢/异常)
       +---------------------------------------+
       |                                       |
       v                                       |
  [ Closed ] <-------(3) 探测成功-------- [ Half-Open ]
  (正常通行)                                (仅放行1个)
       |                                       ^
       |                                       |
       +---------> [ Open ] -------------------+
                  (拒绝所有)    (2) 熔断时长结束
                      |
                      +----(4) 探测失败 (重置计时)
```



###### 4. 核心 API 与监控

在代码层面，我们可以通过 Sentinel 提供的 API 获取当前资源的状态（通常用于监控报警）

```java
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeRuleManager;
import com.alibaba.csp.sentinel.slots.block.degrade.circuitbreaker.CircuitBreaker;

// 获取某个资源名称下的所有断路器实例
List<CircuitBreaker> breakers = DegradeRuleManager.getCircuitBreakers("my-resource");

for (CircuitBreaker breaker : breakers) {
    // 获取当前状态
    CircuitBreaker.State state = breaker.currentState();
    System.out.println("当前状态: " + state.name()); 
    // 输出: OPEN, CLOSED, or HALF_OPEN
}
```



###### 5. 避坑指南：Half-Open 的并发问题

- **问题**：在高并发下，`Half-Open` 真的只放行一个请求吗？

- **答案**：是的。

  - Sentinel 底层使用 CAS (Compare And Swap) 乐观锁机制来保证并发安全

    即便一瞬间来了 1000 个请求，也 **绝对只有 1 个** 能拿到“探测令牌”去执行业务，剩下的 999 个依然会被拦截（抛出 `DegradeException`）

    这有效防止了“试探请求”本身压垮刚恢复的服务



#### 5. 热点参数限流 —— 精细化防护

##### 5.1 场景分析：为何需要针对“参数”限流

在前面的章节中，我们的限流维度都是“资源”（接口）级别的。例如，限制 `/goods/detail` 接口的总 QPS 不能超过 1000

- 但在实际的大促场景（如双11）中，这种粗粒度的限流往往是不够的。我们需要一种能深入到 **“请求参数内容”** 的精细化流量控制



###### 1. 什么是“热点”？

**热点** 就是经常被访问的数据

- **商品维度**：某一款爆款手机（iPhone 15），某一个超低价的秒杀商品
- **用户维度**：某个疯狂刷接口的恶意用户（爬虫），或者某个被盗号的异常账户
- **IP 维度**：某个特定的来源 IP



###### 2. 传统限流的痛点

假设你的电商系统有一个查询商品详情的接口： `GET /goods/detail?id={skuId}`

你配置了流控规则：该接口总 QPS 限制为 **100**

**极端场景演示**： 库里有 1000 款商品

- **正常情况**：大家访问分布很均匀，iPhone、华为、小米都有人看，总 QPS 50，系统很稳
- **突发情况**：突然，**ID=1001 (iPhone 18)** 发布了，99% 的流量都涌向了这个 ID
- **结果**：
  1. 总流量瞬间打满 100 QPS
  2. Sentinel 开始无差别限流
  3. **误杀**：因为 iPhone 18 把 100 QPS 的配额全占了，导致想买“诺基亚”用户（访问 ID=2001）的请求也被拦截了

**这就是痛点**：因为一个热点资源（iPhone），导致其他非热点资源（诺基亚）陪葬。我们希望的是：**限制 iPhone 的访问，但不影响诺基亚的访问**



###### 3. 热点参数限流的解决方案

Sentinel 的 **热点参数限流** 就是为了解决这个问题。它能实时统计传入参数的值

**核心逻辑**： 它可以针对 `GET /goods/detail` 接口的 **第 0 个参数**（即 `id`）进行统计

- 如果参数值是 `1001` (iPhone)，限制 QPS = 50
- 如果参数值是 `2001` (诺基亚)，限制 QPS = 200 (或者不限制)
- 其他参数值，走默认阈值



**对比图示**：

- **普通流控**：
  - 阀门关在门口
  - 不管你是买 iPhone 还是买牙膏，只要进门的人多了，统统拦住
- **热点参数流控**：
  - 阀门关在货架前
  - 买 iPhone 的人排队（限流），买牙膏的人随便拿（不限流）



###### 4. 典型应用场景

1. **爆款防刷**：
   - 防止某个热门商品占用整个数据库连接池
2. **防恶意爬虫/刷子**：
   - 针对 `userId` 参数限流
   - 规定：每个 User ID 每秒只能调用 5 次接口。超过的可能是脚本在刷，直接拒绝
3. **热点键 (Hot Key) 探测**：
   - Redis 缓存热点 Key 探测与保护



##### 5.2 配置详解：索引下标与参数例外项

要实现热点参数限流，我们不能仅依靠默认的 URL 资源名（如 `/order/get`），因为默认的 Web 适配层不一定能将 HTTP 参数完美透传给 Sentinel

- **最佳实践是：必须配合 `@SentinelResource` 注解使用**

###### 1. 代码准备：埋点

我们需要在 Controller 方法上显式添加 `@SentinelResource`，并定义一个唯一的资源名称

**代码示例**：

```java
@RestController
public class HotSpotController {

    // 1. 定义资源名为 "deal_hot_product"
    // 2. blockHandler 用于处理限流后的逻辑 (必须是 public，返回类型相同，参数最后多一个 BlockException)
    @SentinelResource(value = "deal_hot_product", blockHandler = "handleHotBlock")
    @GetMapping("/product/query")
    public String queryProduct(@RequestParam(value = "p1", required = false) String p1, 
                               @RequestParam(value = "p2", required = false) String p2) {
        // p1 代表商品 ID，p2 代表用户 ID
        return "查询商品成功：ID=" + p1;
    }

    // 兜底方法
    public String handleHotBlock(String p1, String p2, BlockException exception) {
        return "不好意思，该商品访问过于火爆，请稍后再试 (Blocked by HotParam)";
    }
}
```



###### 2. 基础配置：参数索引与默认阈值

启动服务，访问几次接口后，进入 Sentinel 控制台的 **“热点规则”** 菜单

点击 **“新增热点规则”**，填写如下：

- **资源名**：`deal_hot_product` (必须与注解中的 value 一致)
- **参数索引 (Param Index)**：`0`
  - **含义**：对应 Java 方法参数列表里的**第 0 个参数**（即代码中的 `p1` / 商品ID）
  - *注意*：这里不看 HTTP 参数名（?p1=xx），只看 Java 方法签名的顺序
- **单机阈值**：`1`
  - **含义**：**默认情况**下，任何一个 `p1` 的值，QPS 都不能超过 1
- **统计窗口时长**：`1` (秒)

**效果验证**：

- 访问 `http://.../product/query?p1=100`，QPS > 1 时，会被限流
- 访问 `http://.../product/query?p1=200`，QPS > 1 时，也会被限流
- (大家一视同仁，都只能每秒访问 1 次)



###### 3. 进阶配置：参数例外项 —— VIP 通道

现在需求变了：

- **普通商品 (ID=任意)**：QPS 限制为 1
- **爆款商品 (ID=5)**：如果不让它火，会影响 GMV。我们需要给它开绿灯，允许 QPS 达到 **200**

这就是 **“参数例外项”** 的用武之地



**配置步骤**：

1. 在刚才规则的编辑界面，打开 **“高级选项”**
2. 在 **“参数例外项”** 表格中添加一行：
   - **参数类型**：`String` (必须与代码中 `p1` 的类型严格一致)
   - **参数值**：`5`
   - **限流阈值**：`200`
3. 点击“保存”



**效果验证**：

- 狂刷 `p1=5` (爆款)：你会发现 QPS 飙到 100 都没有问题（走 VIP 通道）
- 狂刷 `p1=100` (普通)：QPS 超过 1 直接报错（走默认通道）

**这就是“精准隔离”：** 只有 `p1=5` 的请求享受高阈值，其他值依然受限



###### 4. 避坑指南

- **巨坑一：参数类型不匹配**

  - 在“参数例外项”里，如果你代码里的 `p1` 是 `Integer` 类型，但配置里选了 `String`，**规则将失效**。Sentinel 不会自动做类型转换
  - *建议*：尽量使用 `String` 或基本数据类型

- **巨坑二：未配置 `blockHandler`**

  - 热点限流触发时，抛出的是 `ParamFlowException` (继承自 `BlockException`)
  - 如果不配置 `blockHandler`，用户会看到一个极其不友好的 HTTP 500 错误页面（Whitelabel Error Page），前端也无法处理。务必编写兜底逻辑返回 JSON

- **巨坑三：方法参数顺序**

  - 索引下标 `0` 永远指向方法签名的**第一个参数**

    如果你在重构代码时，调整了参数顺序（把 `p2` 挪到了 `p1` 前面），记得一定要去控制台修改规则的索引，否则限流对象就变了！



#### 6. 系统自适应保护

##### 6.1 系统负载保护：最后的防线

**系统自适应保护** 是 Sentinel 提供的一种全局兜底机制

###### 1. 核心理念：从“单点”到“全局”

- **常规流控 (FlowRule)**：是针对某个具体的 API（如 `/order/create`）进行限制
  - *局限性*：如果有 100 个接口，每个接口 QPS 都不高，但加在一起把 CPU 吃满了，常规流控可能无法触发，导致机器卡死
- **系统保护 (SystemRule)**：是针对 **整个 Java 进程** 所在的服务器（或者容器）进行保护
  - *作用*：它不关心是哪个接口进来的流量，只要机器整体指标（如 CPU、Load）超过阈值，它就无差别地拦截入口流量，直到机器负载恢复正常

**比喻**：

- **流控**：像是大楼里每个房间的限流（这个会议室只能进 10 个人）
- **系统保护**：像是大楼的总闸。如果大楼要塌了（地震了），直接封锁大门，谁都别进来了



###### 2. 五大核心指标

Sentinel 支持从五个维度来感知系统的负载情况。当达到阈值时，触发系统保护

1. **Load (系统负载)** —— **【最核心】**

   - **指标**：Linux 系统的 `Load1` (1分钟平均负载)
   - **含义**：代表当前正在等待 CPU 时间片的线程数
   - **阈值建议**：通常设置为 `CPU 核数 * 2.5`。例如 4核机器，阈值设为 10
   - **注意**：此指标仅在 Linux/Unix 下生效，Windows 下始终为 0（无法触发）

   

2. **CPU Usage (CPU 使用率)**

   - **指标**：当前 Java 进程所在的系统 CPU 使用率
   - **范围**：`[0.0, 1.0]`。例如 `0.8` 代表 80% 使用率
   - **场景**：当 CPU 飙升时，自动拒绝流量，防止 CPU 100% 卡死

   

3. **RT (平均响应时间)**

   - **指标**：所有入口流量的平均响应时间
   - **场景**：当系统整体变慢（可能是数据库挂了，导致所有业务都慢），自动限流

   

4. **Thread (入口线程数)**

   - **指标**：Sentinel 统计的当前全局并行处理线程数
   - **场景**：防止线程池耗尽

   

5. **QPS (入口 QPS)**

   - **指标**：整个应用的总 QPS
   - **场景**：极少单独使用（因为总 QPS 很难评估准确），通常配合 CPU 使用



###### 3. 启发式流控算法 (BBR 思想)

Sentinel 的系统规则借鉴了 TCP BBR 拥塞控制算法的思想

**逻辑是这样的**： 系统在运行过程中，Sentinel 会实时采样 `CPU` 和 `Load`

- 如果 `Load` > 阈值，Sentinel 不会立刻一刀切拒绝所有请求
- 它会判断当前的 **并发线程数 (Thread)** 是否超过了 **系统最大的处理能力**
- 只有当“负载高”且“并发也高”时，才会进行限流

这是一种 **自适应** 的机制：它试图在系统崩溃的边缘，找到一个最大的吞吐量平衡点



###### 4. 避坑指南

- **不要乱配**：系统规则是 **全局生效** 的！一旦触发，**所有** 接口（包括健康检查接口、核心支付接口）都会被拦截，报 `SystemBlockException`。这可能导致误杀
- **Windows 开发环境**：
  - 在 Windows 本地开发时，`Load` 指标永远获取不到（因为 JVM 在 Windows 下不支持 `getSystemLoadAverage`），所以不要在本地测 Load 规则
- **建议**：生产环境通常配置 **CPU 使用率 (0.8)** 或 **Load** 作为最后的兜底即可，不要配得太复杂



##### 6.2 最佳实践：作为兜底防御的最后一道防线

系统自适应保护 (System Rule) 是 Sentinel 独有的“核武器”。因为它一刀切的特性，我们在生产环境中使用时必须格外小心



###### 1. 定位：保命而非治病

首先要摆正心态：**系统规则不是用来做精细化限流的，而是用来“保命”的**

- **常规流控 (FlowRule)**：是“治病”。针对某个具体的业务接口（如秒杀接口）进行限制，让它不要太嚣张
- **系统规则 (SystemRule)**：是“保命”。当机器快要挂掉（Load 飙高、CPU 跑满）时，不管你是谁，统统拦在门外，先让机器缓一口气

**最佳实践原则**： 永远优先配置 **热点参数限流** 和 **接口流控**。只有当这些手段都失效（或者漏配）了，系统规则才作为 **最后一道防线** 生效



###### 2. 生产环境配置建议

在实际生产中，我们通常建议只配置以下两个指标作为兜底：

**A. Load (系统负载)**

- **适用环境**：仅限 Linux/Unix 服务器
- **推荐阈值**：`CPU 核数 * 2.5`
- **理由**：这是业界公认的 Load 警戒线。超过这个值，说明排队等待 CPU 的线程过多，系统处理能力已经饱和
- **例子**：4核 8G 的容器，阈值设为 `10.0`



**B. CPU Usage (CPU 使用率)**

- **适用环境**：Linux / Windows / Mac 通用
- **推荐阈值**：`0.75` ~ `0.8` (即 75% - 80%)
- **理由**：预留 20% 的 CPU 给系统内核进程、GC 线程以及 SSH 运维操作。如果 CPU 飙到 100%，你连 SSH 登录服务器去 `kill` 进程的机会都没有



###### 3. 致命坑点：健康检查误杀

这是 Kubernetes (K8s) 环境下最容易发生的 **级联灾难**



**场景复现**：

1. 你配置了系统规则：`CPU > 80%` 时触发限流
2. 业务高峰期，机器 CPU 飙升到了 90%
3. Sentinel 触发保护，开始拦截 **所有** 入口流量
4. **关键点来了**：K8s 的 **Readiness Probe (就绪探针)** 或 **Liveness Probe (存活探针)** 本质上也是 HTTP 请求（例如 `GET /actuator/health`）
5. Sentinel 把健康检查接口也给拦截了（抛出 `SystemBlockException`）
6. K8s 发现接口返回 500 或不通，判定该 Pod “已死”
7. K8s **重启** 该 Pod
8. Pod 重启后，流量再次涌入，CPU 再次飙高，再次重启……进入 **死循环**

**解决方案**： 虽然 Sentinel 官方声称系统规则是针对入口流量，但在某些框架整合中，健康检查接口确实可能被拦截

- **方案 1**：确保健康检查接口不在 Sentinel 的拦截范围内（配置白名单，但这比较麻烦）
- **方案 2 (推荐)**：不要把系统规则阈值设得太低，给健康检查留出余地



#### 7. 注解 `@SentinelResource` 与 API

##### 7.1 API 清单

在 Sentinel 的世界里，`@SentinelResource` 注解其实只是一个 AOP 的“糖衣”。剥开这层糖衣，内部运行的全是原生的 API 调用

掌握这些 API，意味着你拥有了 **脱离 Spring 框架独立使用 Sentinel** 的能力



###### 1. `SphU`

>Sentinel Protection Handler - User

这是 Sentinel **最核心** 的入口类。所有的资源定义、规则检查，最终都会汇聚到这个类的方法上

**常用 API 清单**：

| 方法签名                                                     | 返回值       | 作用说明                                                     | 备注                            |
| ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ | ------------------------------- |
| `entry(String name)`                                         | `Entry`      | **最常用**。定义一个资源。如果被限流/熔断，抛出 `BlockException` | 必须配合 `entry.exit()` 使用    |
| `entry(String name, EntryType type)`                         | `Entry`      | 定义资源并指定流量类型（`IN` 入口流量 / `OUT` 出口流量）     | 用于区分系统规则统计            |
| `entry(String name, EntryType type, int count, Object... args)` | `Entry`      | 定义资源，并传入 **热点参数**                                | `args` 对应热点参数限流中的索引 |
| `asyncEntry(...)`                                            | `AsyncEntry` | 定义一个 **异步** 资源                                       | 用于主线程和回调线程分离的场景  |

**注意**：`SphU` 的全称是 "Sentinel Protection Handler for User"，意为“面向用户的防护处理器”



###### 2. `Tracer` (追踪器)

在原生 API 模式下，Sentinel **不会自动感知** 你的业务异常

- 如果不显式调用 `Tracer`，即使你的业务抛出了 `RuntimeException`，Sentinel 的控制台里“异常数”依然是 0，导致 **异常熔断降级失效**

**常用 API 清单**：

| 方法签名                        | 返回值 | 作用说明                            |
| ------------------------------- | ------ | ----------------------------------- |
| `trace(Throwable e)`            | `void` | 将异常记录到当前 Entry 的统计槽中。 |
| `trace(Throwable e, int count)` | `void` | 记录异常（支持自定义计数）。        |



###### 3. `ContextUtil` (上下文工具)

用于维护当前调用链路的上下文。主要用于 **链路流控模式** 和 **来源访问控制 (授权规则)**



**常用 API 清单**：

| 方法签名                                   | 返回值    | 作用说明                                 |
| ------------------------------------------ | --------- | ---------------------------------------- |
| `enter(String contextName)`                | `Context` | 进入一个调用上下文                       |
| `enter(String contextName, String origin)` | `Context` | 进入上下文，并标记 **调用来源 (origin)** |
| `exit()`                                   | `void`    | 退出当前上下文                           |



###### 4. 标准代码模板 —— 背诵级

如果你不使用注解，想手动保护一段代码，**必须且只能 **严格遵守以下格式。顺序错一步，可能导致 **统计不准**、**内存泄漏 **或 **规则失效**

```JAVA
import com.alibaba.csp.sentinel.SphU;
import com.alibaba.csp.sentinel.Tracer;
import com.alibaba.csp.sentinel.Entry;
import com.alibaba.csp.sentinel.context.ContextUtil;
import com.alibaba.csp.sentinel.slots.block.BlockException;

public void doSomethingRaw() {
    // 1. (可选) 定义上下文，常用于标示调用来源，例如 "app-A"
    // 如果不写这一行，默认 Context 为 "sentinel_default_context"
    ContextUtil.enter("my_context", "app-A");

    Entry entry = null;
    try {
        // 2. 【核心】申请资源凭证
        // 资源名 "resourceName" 必须与控制台规则一致
        entry = SphU.entry("resourceName");

        // 3. 【业务逻辑】被保护的代码
        System.out.println("Hello Sentinel Raw API");
        
        // 模拟业务异常
        if (System.currentTimeMillis() % 2 == 0) {
            throw new RuntimeException("Business Error");
        }

    } catch (BlockException e) {
        // 4. 【限流处理】被 Sentinel 拒绝（流控/熔断/热点）
        // 相当于注解模式下的 blockHandler
        System.out.println("Blocked!");
    } catch (Throwable t) {
        // 5. 【异常统计】关键！手动记录异常
        // 如果不写这一行，异常比例/异常数降级规则将失效
        Tracer.trace(t); 
        
        // 抛出业务异常给上层
        throw t;
    } finally {
        // 6. 【资源释放】必须确保执行
        if (entry != null) {
            entry.exit();
        }
        // 7. (可选) 退出上下文
        ContextUtil.exit();
    }
}
```



###### 5. 为什么要懂这个？

你可能会问：“注解那么方便，我为什么要写这种又臭又长的 try-catch？”

- **场景一：第三方库封装** 

  - 你正在使用一个第三方 HTTP 客户端（如 OkHttp），你想对所有的外部请求做熔断。你无法去修改 OkHttp 的源码加注解

    此时，你可以在拦截器（Interceptor）中使用 `SphU` 包裹请求

- **场景二：复杂的热点参数** 

  - 注解模式下，参数必须是方法的参数。

    如果你想对方法内部生成的一个局部变量（比如解析 JSON 后的某个字段）做热点限流，注解无能为力，只能用 `SphU.entry(name, EntryType.IN, 1, jsonField)`



##### 7.2 注解属性:`@SentinelResource`

`@SentinelResource` 用于定义资源，并指定当资源被限流、熔断时该执行什么逻辑

- 它是基于 AspectJ 切面实现的，因此它的生效必须遵循 Spring AOP 的基本原则（如类内调用失效问题，将在后面讨论）



###### 1. 核心属性一览表

| 属性名                   | 类型      | 必填   | 默认值 | 作用说明                                                    |
| ------------------------ | --------- | ------ | ------ | ----------------------------------------------------------- |
| **`value`**              | String    | **是** | 无     | **资源名称**。控制台配置规则时必须与此值完全一致            |
| **`entryType`**          | EntryType | 否     | OUT    | 流量类型（IN/OUT）。主要用于系统规则保护，通常保持默认即可  |
| **`blockHandler`**       | String    | 否     | ""     | **处理 Sentinel 规则拦截**（如限流、熔断）的方法名          |
| **`blockHandlerClass`**  | Class     | 否     | 无     | 指定 blockHandler 所在的类（用于代码解耦）                  |
| **`fallback`**           | String    | 否     | ""     | **处理 Java 运行时异常**（如 NullPointerException）的方法名 |
| **`fallbackClass`**      | Class     | 否     | 无     | 指定 fallback 所在的类                                      |
| **`exceptionsToIgnore`** | Class[]   | 否     | {}     | 指定哪些异常 **不计入** 异常熔断统计                        |

本节我们将重点攻克 `blockHandler`，下一节专门讲解 `fallback` 及其对比



###### 2. `value` 属性：资源命名的艺术

- **规范**：资源名必须全局唯一
- **建议**：虽然可以随意起名（如 "aaa"），但为了可维护性，建议遵循 `动词:名词` 或 `路径风格`
  - 推荐：`"order:create"`, `"/order/create"`, `"getUserInfo"`
- **避坑**：如果你在 Controller 上使用，且希望复用 URL 作为资源名，**请务必显式写出**
  - *错误*：`@SentinelResource` (空) -> 默认不会自动取 URL
  - *正确*：`@SentinelResource(value = "/order/create")`



###### 3. `blockHandler`属性详解(重中之重)

**定义**： 当资源请求被 Sentinel 的规则（流控、熔断、热点参数、系统规则）拦截时，**不会** 抛出异常给用户，而是 **自动执行** `blockHandler` 指定的方法

**方法签名要求 (Strict Signature Rules)**： 这是最容易出错的地方。`blockHandler` 方法必须满足以下严苛条件：

1. **修饰符**：必须是 `public`
2. **返回值**：必须与原方法 **完全一致**
3. **参数**：
   - 前 N 个参数：必须与原方法**完全一致**（包括顺序、类型）
   - **最后一个参数**：必须是 `BlockException` 类型（这是 Sentinel 传给你的异常详情）
4. **位置**：默认必须写在同一个类中（除非使用了 `blockHandlerClass`）

**代码示例 (同类处理)**：

```java
@RestController
public class OrderController {

    @SentinelResource(value = "createOrder", blockHandler = "handleBlock")
    @GetMapping("/order/create")
    public String createOrder(Long orderId, String type) {
        return "Order Created: " + orderId;
    }

    // 【关键】方法签名必须严格匹配
    // 1. 返回值 String 对 String
    // 2. 参数 (Long, String) 对 (Long, String)
    // 3. 尾部追加 BlockException
    public String handleBlock(Long orderId, String type, BlockException ex) {
        // 在这里可以判断 ex 的具体类型，返回不同的提示
        // if (ex instanceof FlowException) ...
        // if (ex instanceof DegradeException) ...
        return "很抱歉，当前访问人数过多，请稍后重试 (Blocked)";
    }
}
```



###### 4. `blockHandlerClass`：解决代码臃肿

如果每个 Controller 方法下面都挂一个 `handleBlock` 方法，代码会变得非常丑陋且难以复用。 我们可以将降级逻辑抽取到一个单独的类中

**使用步骤**：

1. 创建一个普通的 Java 类（不需要 `@Component`）
2. 编写处理方法，**必须是 `static` (静态)**
3. 在注解中指定 `blockHandlerClass`

**代码示例 (解耦处理)**：

```java
// 1. 独立的降级处理类
public class OrderBlockHandlers {
    
    // 【关键】必须是 static 方法
    public static String globalBlockHandler(Long orderId, String type, BlockException ex) {
        return "Global Handler: System Busy for order " + orderId;
    }
}

// 2. 业务类
@RestController
public class OrderController {

    @SentinelResource(value = "createOrder", 
                      blockHandler = "globalBlockHandler", 
                      blockHandlerClass = OrderBlockHandlers.class) // 指定类
    @GetMapping("/order/create")
    public String createOrder(Long orderId, String type) {
        return "Success";
    }
}
```



###### 5. 常见错误排查

如果你配置了 `blockHandler` 但不生效（依然报 500 错误或 Whitelabel Error Page）：

1. **检查方法签名**：这是 99% 的原因。检查参数类型是否一一对应？是否漏了最后的 `BlockException`？

2. **检查 static**：如果用了 `blockHandlerClass`，方法忘加 `static` 了吗？

3. **检查 public**：方法必须是 public 的

4. **检查异常类型**：

   - `blockHandler` **只处理** Sentinel 产生的 `BlockException`

     如果你的业务代码里抛出了 `NullPointerException`，`blockHandler` 是不会管的（那是 `fallback` 的事，下一节讲）



##### 7.3 区分：`blockHandler` vs `fallback`

在 `@SentinelResource` 中，`blockHandler` 和 `fallback` 就像是两道安全门。理解它们的本质区别，是你能够写出健壮代码的关键

###### 1. 核心差异对比

| 维度          | blockHandler (限流处理)                                      | fallback (降级/异常处理)                                     |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **触发条件**  | **违规**：触发了 Sentinel 的规则（限流、熔断、热点、系统规则） | **出错**：业务代码抛出了 Java 运行时异常（如 NPE、SQL Error、Timeout） |
| **捕获异常**  | 仅捕获 `BlockException` 及其子类                             | 捕获所有 `Throwable` (除了 `exceptionsToIgnore` 中排除的)    |
| **设计目的**  | 告诉用户：“你点太快了”或“系统太忙了”                         | 告诉用户：“我们内部出错了”或“启用备用方案”                   |
| **对应 HTTP** | 类似于 429 Too Many Requests                                 | 类似于 500 Internal Server Error                             |

**一句话总结**：

- **Sentinel 拦住的**，走 `blockHandler`
- **Java 报错的**，走 `fallback`



###### 2. `fallback` 属性

`fallback` 专门用于处理业务逻辑执行过程中发生的意外

**方法签名要求**： 与 `blockHandler` 类似，但更灵活：

1. **返回值**：必须与原方法一致
2. **参数**：
   - 与原方法参数一致
   - (可选) 最后一个参数可以是 `Throwable` (或具体异常类型)，用于接收报错信息
3. **static**：如果使用了 `fallbackClass`，方法必须是 `static`

**特殊的 `defaultFallback`**： 为了避免为每个方法都写一个对应的 fallback，Sentinel 支持配置 `defaultFallback`

- **作用**：当前类中所有未指定 fallback 的方法，统一使用这个默认方法
- **要求**：**参数列表必须为空** (或者只能有一个 `Throwable`)，不能有原方法的业务参数（因为它是通用的，不知道原方法有哪些参数）



###### 3. 双剑合璧：执行流程与优先级

在实际生产中，我们通常会 **同时配置** 这两个属性，以确保无论发生什么情况，用户都能得到友好的反馈

**执行流程图**：

```
Request 进来
    ↓
[ Sentinel 规则检查 ]
    ↓ ❌ 不通过 (如 QPS > 10)
    ↓ ---------------------------> 执行 blockHandler
    ↓
    ↓ ✅ 通过
    ↓
[ 执行业务代码 (try-catch) ]
    ↓ ❌ 抛异常 (如 NullPoint)
    ↓ ---------------------------> 执行 fallback
    ↓
    ↓ ✅ 正常
    ↓
Return Result
```

**实战代码示例**：

```java
@RestController
public class DoubleSafeController {

    @SentinelResource(
        value = "demo_resource",
        blockHandler = "handleBlock",    // 应对限流
        fallback = "handleFallback"      // 应对异常
    )
    @GetMapping("/demo")
    public String demo(@RequestParam Integer id) {
        if (id == 0) {
            throw new RuntimeException("业务参数错误"); // 触发 fallback
        }
        return "Success: " + id;
    }

    // 1. 处理 Sentinel 限流 (参数必须匹配 + BlockException)
    public String handleBlock(Integer id, BlockException ex) {
        return "被限流了，请稍后";
    }

    // 2. 处理 Java 异常 (参数必须匹配 + Throwable)
    public String handleFallback(Integer id, Throwable t) {
        return "业务出错了，启动兜底逻辑：" + t.getMessage();
    }
}
```



###### 4. 忽略异常 (`exceptionsToIgnore`)

有些异常是业务逻辑的一部分，我们 **不希望** 它触发 Sentinel 的异常熔断统计

- **场景**：

  - `AccountNotFoundException`：这是告诉前端“查无此人”，是正常的业务结果，不是服务挂了
  - 如果不忽略，Sentinel 会记录异常数，可能导致服务被错误熔断

- **配置**：

  ```JAVA
  @SentinelResource(
      value = "findUser",
      fallback = "fallbackHandler",
      // 下面这个异常抛出时，Sentinel 不会统计到异常比例中，也不会触发 fallback
      exceptionsToIgnore = { AccountNotFoundException.class } 
  )
  public User findUser(String id) {
      throw new AccountNotFoundException("无此用户"); // 直接抛给前端
  }
  ```



###### 5. 避坑指南：Spring AOP 的限制

在使用 `blockHandler` 和 `fallback` 时，必须时刻牢记它们是基于 Spring AOP 实现的

- **私有方法无效**：`private` 方法上的注解不生效

- **类内部调用失效 (最经典的大坑)**：

  ```JAVA
  @Service
  public class OrderService {
  
      public void methodA() {
          // 直接调用 methodB，AOP 切面无法介入
          methodB(); 
      }
  
      @SentinelResource("b") // 【这里不会生效】
      public void methodB() { ... }
  }
  ```

  - **原因**：Spring AOP 基于代理模式。类内部调用是 `this.methodB()`，绕过了代理对象
  - **解决**：下一节我们会专门讲这个问题的解决方案



##### 7.4 避坑指南：类内调用失效与返回值陷阱

在使用 `@SentinelResource` 时，我们必须时刻铭记：**Sentinel 是基于 Spring AOP（动态代理）实现的**

这意味着，凡是会导致 AOP 失效的场景，都会导致 Sentinel 失效。其中最经典、最坑人的就是 **“类内部调用”** 问题



###### 1. 鬼故事：类内部调用失效

**场景复现**： 你写了一个 Service，里面有两个方法：`methodA` 和 `methodB`

- `methodA`：普通方法，作为入口
- `methodB`：加了 `@SentinelResource`，希望能被限流
- **调用链路**：Controller -> Service.methodA() -> Service.methodB()

**失效代码**：

```java
@Service
public class OrderService {

    public void methodA() {
        System.out.println("进入 A");
        // 【关键点】这里直接调用了内部的 methodB
        // 本质上等价于 this.methodB()
        methodB(); 
    }

    @SentinelResource("resource_b") // 【这里绝对不会生效】
    public void methodB() {
        System.out.println("执行 B");
    }
}
```



**原因深度解析**： Spring AOP 的工作原理是生成一个 **代理对象 (Proxy)**

1. 当 Controller 调用 `OrderService` 时，实际上拿到的是 `OrderServiceProxy`
2. `Proxy.methodA()` 内部调用了 `Target.methodA()`
3. 在 `Target.methodA()` 内部，调用 `methodB()` 时，使用的是 `this.methodB()`
4. **`this` 代表的是目标对象本身，而不是代理对象**
5. 因此，代码直接绕过了代理对象（以及代理对象中的 Sentinel 切面逻辑），直接执行了业务代码。**没有经过代理，就没有限流**



###### 2. 解决方案：如何打破僵局？

要解决这个问题，核心在于：**必须通过代理对象来调用 methodB**

**方案一：自注入 (Self Injection) —— 最推荐** 在类内部注入自己。Spring 支持这种循环依赖（Setter 注入或字段注入）

```java
@Service
public class OrderService {

    // 1. 注入自己（拿到的是代理对象）
    @Autowired
    private OrderService self;

    public void methodA() {
        // 2. 通过代理对象调用 methodB
        self.methodB(); 
    }

    @SentinelResource("resource_b") // 【现在生效了】
    public void methodB() { ... }
}
```



**方案二：拆分 Service (Refactor) —— 最规范** 

- 如果 `methodA` 和 `methodB` 逻辑差异很大，建议将 `methodB` 拆分到另一个 Service 类中。跨类调用天然经过 AOP 代理

**方案三：AopContext (高级)** 

- 不推荐，因为需要修改 Spring 全局配置 `exposeProxy=true`，侵入性较强



###### 3. 陷阱二：Private 方法失效

**规则**： `@SentinelResource` **必须**添加在 `public` 方法上

- 如果加在 `private` 或 `protected` 方法上，AOP 切面无法切入，配置将直接被忽略，且不会报错（静默失效）



###### 4. 陷阱三：返回值不匹配

我们在 7.2 节提到过 `blockHandler` 的签名要求，这里再次强调一个容易忽视的细节：**泛型擦除后的匹配**

**场景**： 原方法返回 `List<User>`，兜底方法也返回 `List<User>`。 但在某些极端情况下（特别是使用了自定义泛型类 `Result<T>`），可能会报错



**严格原则**：

- 兜底方法的返回值类型，必须与原方法 **完全一致**
- 如果原方法是 `void`，兜底方法也必须是 `void`
- 建议尽量避免使用复杂的嵌套泛型作为返回值，或者确保你的兜底方法签名在编译后与原方法一致



**调试技巧**： 如果你收到 `java.lang.NoSuchMethodException`，请仔细对比原方法和 `blockHandler` 的：

1. 方法名拼写
2. 参数类型顺序
3. **返回值类型**
4. `static` 关键字（如果是跨类调用）



###### 5. 陷阱四：BlockException 漏传

**现象**： `blockHandler` 逻辑没报错，但无法获取到限流的详情（比如到底是触发了 QPS 限流还是熔断）

**原因**： 你在 `blockHandler` 的参数列表中，忘记加最后一个 `BlockException` 参数，或者参数位置放错了

- *错误*：`handleBlock(BlockException ex, Long id)`
- *正确*：`handleBlock(Long id, BlockException ex)`



#### 8. Sentinel 规则持久化

##### 8.1 原生模式痛点：重启后规则丢失

在生产环境中，服务的重启、扩容、发布是家常便饭。如果每次重启都需要运维人员手动去 Sentinel 控制台重新配一遍限流规则，那绝对是场灾难

- 然而，Sentinel 的 **默认模式** 正是如此

###### 1. 现象复现

我们可以做一个简单的实验：

1. 启动 `order-service` 和 Sentinel Dashboard
2. 在控制台为 `/order/create` 接口配置一条流控规则：QPS Limit = 1
3. 测试一下，发现限流生效
4. **操作**：停止 `order-service`，然后重新启动它
5. **结果**：刷新 Sentinel 控制台，你会发现刚才配置的规则 **消失得无影无踪**



###### 2. 根源分析：内存存储

为什么会这样？我们需要回顾一下 Sentinel 的架构

- **Dashboard 的角色**：它只是一个规则的“展示者”和“推送者”
- **Client (微服务) 的角色**：它是规则的“接收者”和“执行者”



**默认模式下的数据流向**：

1. 你在 Dashboard 点击“保存”
2. Dashboard 调用微服务的 API，将规则推送到微服务
3. 微服务接收到规则，将其保存在 **内存 (JVM Memory)** 中
   - 具体类：`com.alibaba.csp.sentinel.property.InMemorySentinelProperty`
4. **并没有写入任何文件或数据库**

- 一旦微服务重启，JVM 内存被清空，规则自然就丢失了

  同样的，如果 Dashboard 重启，Dashboard 也会丢失所有机器列表和聚合数据（虽然这不影响微服务的运行，但影响查看）



###### 3. 生产环境的需求

在真正的生产环境中，我们需要满足以下标准：

1. **持久化**：规则必须保存到物理介质（数据库/文件/配置中心）中，重启不丢失
2. **动态推送**：修改规则后，必须实时推送到所有微服务实例，无需重启服务
3. **统一管理**：不能去每台机器上改文件，必须有统一的管控台



###### 4. 解决方案预览

为了解决这个问题，Sentinel 提供了两种扩展模式：

- **Pull 模式 (拉模式)**：
  - 微服务定时去文件或某个地方拉取规则
  - *缺点*：实时性差，很难保证数据一致性
- **Push 模式 (推模式)** —— **【生产推荐】**
  - 引入第三方 **配置中心 (Configuration Center)**，如 **Nacos**, Apollo, Zookeeper
  - **流程**：控制台 -> Nacos -> 微服务
  - **优点**：规则持久化在 Nacos 数据库中；Nacos 支持毫秒级的动态推送



##### 8.2 持久化模式：Pull vs Push

- 为了解决规则丢失问题，Sentinel 提供了 `DataSource` 扩展接口。根据数据源的不同，衍生出了两种截然不同的持久化模式

###### 1. Pull 模式 (拉模式) —— 原始而简单

**工作原理**： 客户端（微服务）主动向某个数据源（通常是本地文件或远程 URL）定期轮询，一旦发现数据有变化，就更新内存中的规则

**典型实现**：**FileRefreshableDataSource** (基于本地文件)

**流程图解**：

```
[ Sentinel Dashboard ] --(1. 推送)--> [ 内存 ] --(2. 写入)--> [ 本地 JSON 文件 ]
                                          ^
                                          | (3. 定时轮询读取)
                                       [ 微服务 ]
```

**优点**：

- **简单**：不需要额外的中间件（如 Nacos、Zookeeper），只要有文件系统就能用
- **低成本**：适合单机或极小规模的测试环境



**缺点 (致命)**：

1. **非实时**：因为是定时轮询（默认间隔几秒），规则更新会有延迟
2. **数据不一致**：Dashboard 推送到内存，内存再写文件，这个过程容易出现并发冲突
3. **管理困难**：
   - 如果是集群部署（100 个微服务实例），你需要保证这 100 台机器本地都有那个规则文件，且内容一致，这在容器化（Docker/K8s）环境下几乎是不可维护的

**结论**：**生产环境不推荐使用 Pull 模式**



###### 2. Push 模式 (推模式) —— 生产级标准

**工作原理**： 

- 客户端与**配置中心**（如 Nacos, Apollo, Zookeeper）保持长连接（或监听机制）

  一旦配置中心的数据发生变化，配置中心会 **第一时间推送** 给客户端，客户端立即更新内存

**典型实现**：**NacosDataSource** (基于 Nacos)

**流程图解**：

```
[ Sentinel Dashboard ] --(1. 发布/修改)--> [ Nacos 配置中心 ]
                                                |
                                                | (2. 实时推送 / 监听变更)
                                                v
                                           [ 微服务 A ]
                                           [ 微服务 B ]
                                           [ 微服务 ... ]
```

**优点**：

- **实时性高**：毫秒级推送，几乎无延迟
- **一致性强**：所有微服务实例共享 Nacos 中的同一份配置，保证集群规则统一
- **持久化**：Nacos 本身支持数据持久化（MySQL），重启后数据不丢

**缺点**：

- **架构复杂**：需要引入并维护一个配置中心中间件

- **改造成本**：**Sentinel Dashboard 默认不支持直接推送到 Nacos**

  - *解释*：原生 Dashboard 只能推送到微服务内存

    要实现“Dashboard -> Nacos”这一步，我们需要 **修改 Dashboard 的源码** 并重新打包（或者使用第三方修改好的 Jar 包）

###### 3. 核心对比

| 特性         | Pull 模式 (拉)       | Push 模式 (推)               |
| ------------ | -------------------- | ---------------------------- |
| **时效性**   | 低 (秒级/分钟级延迟) | **极高 (毫秒级)**            |
| **一致性**   | 差 (容易冲突)        | **强 (集中管理)**            |
| **复杂度**   | 低 (依赖本地文件)    | 中 (依赖 Nacos/Apollo)       |
| **适用场景** | 本地开发、演示       | **生产环境、大规模集群**     |
| **典型代表** | File, HTTP           | **Nacos**, Apollo, Zookeeper |



##### 8.3 基于 Nacos 配置中心实现规则持久化

- 在生产环境中，我们通常采用 **Push 模式**。本节将演示如何配置微服务，使其能够从 Nacos 读取流控规则

###### 1. 准备工作 (Prerequisites)

- 确保你已经启动了 **Nacos Server** (默认端口 8848)
- 确保你已经启动了 **Sentinel Dashboard**
- 确保你的微服务 (`order-service`) 能正常运行



###### 2. 引入依赖 (Maven)

在你的微服务 `pom.xml` 中，除了 `sentinel-core`，还需要引入 Sentinel 对 Nacos 的适配包

```xml
<!-- Sentinel Nacos 数据源适配 -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```



###### 3. 客户端配置 (YAML)

这是最关键的一步。我们需要在 `application.yml` 中告诉 Sentinel：“去 Nacos 的哪个 DataID 读取规则？”

**配置示例**：

```yaml
spring:
  application:
    name: order-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 # Nacos 地址
    sentinel:
      transport:
        dashboard: localhost:8858
      # 【核心配置】配置数据源
      datasource:
        # 自定义的数据源名称 (可随便起，如 flow-ds)
        flow-ds:
          nacos:
            server-addr: localhost:8848
            dataId: order-service-flow-rules
            groupId: SENTINEL_GROUP
            # 规则类型：flow (流控), degrade (熔断), param-flow (热点), system (系统), authority (授权)
            rule-type: flow
            # 数据格式
            dataType: json
```

**配置详解**：

- **`dataId`**：Nacos 中的文件名。推荐命名规范：`${spring.application.name}-flow-rules`
- **`groupId`**：Nacos 分组，默认常用 `SENTINEL_GROUP`
- **`rule-type`**：必须明确告诉 Sentinel 这里面存的是流控规则 (`flow`) 还是熔断规则 (`degrade`)



###### 4. 在 Nacos 中创建规则

打开 Nacos 控制台 (`localhost:8848/nacos`)，新建一个配置：

- **Data ID**: `order-service-flow-rules`
- **Group**: `SENTINEL_GROUP`
- **配置格式**: `JSON`
- **配置内容**：

```js
[
    {
        "resource": "/order/create",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
```

**JSON 字段解析** (对照 `FlowRule` 类)：

- `resource`: 资源名
- `grade`: 阈值类型 (1=QPS, 0=线程数)
- `count`: 单机阈值 (这里设为 1，方便测试)
- `strategy`: 流控模式 (0=直接)
- `controlBehavior`: 流控效果 (0=快速失败)



###### 5. 验证效果

1. 启动微服务 `order-service`
2. 查看微服务启动日志，如果看到类似 `[NacosDataSource] properties loaded`，说明连接成功
3. 打开 Sentinel Dashboard，查看“流控规则”页面
   - **神奇的事情发生了**：虽然你没有在 Dashboard 里配规则，但列表里已经自动出现了一条 `/order/create` 的限流规则
   - **来源**：这是微服务从 Nacos 读到内存后，同步汇报给 Dashboard 的
4. **测试**：快速访问 `/order/create` 两次，发现确实被限流了
5. **持久化测试**：重启微服务。再次刷新 Dashboard，规则依然存在！(因为是从 Nacos 拉取的)



###### 6. 巨坑：Dashboard 的“单向失效”问题

做到这里，你通过 Nacos 修改规则，微服务能收到，这没问题。 但是，如果你**在 Sentinel Dashboard 界面上修改规则**，会发生什么？

- **现象**：
  1. Dashboard 上修改 QPS = 10，点击保存
  2. 微服务内存生效了，QPS 变成了 10
  3. **但是！Nacos 里的配置文件依然是 QPS = 1**
  4. **后果**：一旦微服务重启，它重新去读 Nacos，QPS 又变回了 1。你在 Dashboard 做的修改全丢了
- **原因**：
  - 官方开源的 Dashboard **默认只支持推送到微服务内存**，不支持推送到 Nacos
- **生产级解决方案**：
  - 必须对 Sentinel Dashboard 进行 **源码改造**
  - 改造逻辑：拦截 Dashboard 的“发布”按钮请求，改为调用 Nacos 的 Open API 将配置写入 Nacos
  - 社区有很多已经改好的 jar 包（搜索 `Sentinel-Dashboard-Nacos`），直接下载使用即可，无需自己重写



#### 9. 网关层集成

##### 9.1 网关流控：全局流量清洗

普通的 Sentinel 是运行在微服务内部（Servlet 容器）的，而网关流控是运行在 Spring Cloud Gateway（Netty/WebFlux 容器）层面的

- 由于底层架构完全不同（Servlet vs Reactor），**网关流控的依赖、API 和规则配置都与普通微服务完全不同**

###### 1. 核心架构与优势 (Architecture)

- **位置**：部署在 Spring Cloud Gateway 应用中

- **原理**：Sentinel 提供了一个 `GlobalFilter`，在请求进入网关路由（Route）之前进行拦截

- **核心优势**：

  - **Route 维度限流**：直接针对 YAML 里配置的 `id` (如 `order-route`) 进行限流
  - **API 分组维度**：可以将 `/order/get` 和 `/order/create` 归为一组，统一限流
  - **高性能**：专门针对 Reactor 异步模型优化，性能损耗极低

  

###### 2. 引入依赖 (Dependencies)

**注意**：网关应用 **不需要** `spring-boot-starter-web` (它是 Servlet)，而是依赖 `spring-cloud-starter-gateway`。 Sentinel 侧也需要引入专门的适配器

```xml
<!-- 1. Spring Cloud Gateway 核心 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

<!-- 2. Sentinel 适配 Gateway 的专属依赖 (关键) -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>

<!-- 3. Sentinel 核心库 (如果父工程没引的话) -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```



###### 3. 快速实战：Route 维度流控

假设你的 Gateway YAML 配置如下：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order_route  # 【资源名】就是这个 id
          uri: lb://order-service
          predicates:
            - Path=/order/**
```

**步骤**：

1. 启动 Gateway 和 Sentinel Dashboard
2. 进入 Dashboard，你会发现多了一个 **“网关流控”** 的菜单（普通微服务没有这个菜单）
3. 点击 **“请求链路”**，找到 `order_route`
4. 点击 **“+流控”**：
   - **API 类型**：Route ID
   - **API 名称**：order_route
   - **阈值**：QPS = 1
5. **验证**：快速访问网关地址 `http://localhost:9000/order/hello`，超过阈值后，会返回默认的拦截提示



###### 4. 定制异常处理

**痛点**： 

- 默认情况下，网关拦截后返回的是一段非常简单的文本：`Blocked by Sentinel: FlowException`

  前端无法解析，我们需要返回标准的 JSON（如 `{"code": 429, "message": "网关忙"}`）

**实现难点**： 

- Gateway 基于 **WebFlux**，不能使用 Servlet 的 `HttpServletRequest`，必须使用 `ServerWebExchange`

  所以不能用第 7 章讲的 `BlockExceptionHandler`

**代码示例**：

```java
import com.alibaba.csp.sentinel.adapter.gateway.sc.callback.BlockRequestHandler;
import com.alibaba.csp.sentinel.adapter.gateway.sc.callback.GatewayCallbackManager;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.web.reactive.function.BodyInserters;
import org.springframework.web.reactive.function.server.ServerResponse;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import javax.annotation.PostConstruct;

@Configuration
public class GatewayConfiguration {

    @PostConstruct
    public void doInit() {
        // 设置自定义的限流异常处理器
        GatewayCallbackManager.setBlockHandler(new BlockRequestHandler() {
            @Override
            public Mono<ServerResponse> handleRequest(ServerWebExchange exchange, Throwable t) {
                // 自定义 JSON 返回体
                String jsonBody = "{\"code\": 429, \"message\": \"网关流量过大，请稍后重试\"}";

                return ServerResponse.status(HttpStatus.TOO_MANY_REQUESTS) // HTTP 429
                        .contentType(MediaType.APPLICATION_JSON)
                        .body(BodyInserters.fromValue(jsonBody));
            }
        });
    }
}
```



###### 5. 避坑指南

- **坑点一：依赖冲突**
  - 千万不要在 Gateway 项目里引入 `spring-boot-starter-web`，否则会启动报错（Tomcat 和 Netty 冲突）
- **坑点二：规则对象变了**
  - 在代码中手动配置规则时，普通微服务用 `FlowRule`，但网关要用 **`GatewayFlowRule`**
  - 如果不小心用了 `FlowRule`，网关流控是不会生效的
- **坑点三：Dashboard 版本**
  - 确保 Dashboard 版本 >= 1.6.0，否则没有“网关流控”界面



##### 9.2 API 分组与参数限流：精细化网关治理

- 在网关层，我们不仅需要控制“路宽”（Route 维度限流），还需要精细控制“谁在走这条路”（API 分组与参数限流）

###### 1. API 分组 —— 自定义资源维度

**痛点**： 

- 默认的 Route ID 维度太粗了。

  假设 `order-service` 有 50 个接口，你只想对其中的 **“写操作”**（如 `/create`, `/update`）进行统一限流，限制总 QPS < 100，而不影响“读操作”

  用 Route ID 没法做，因为它们都在同一个 Route 下

**解决方案**： 使用 **API 分组**。我们可以把一组 URL 匹配模式定义为一个 API Group，然后针对这个 Group 配规则

**配置方式**：

1. 进入 **“网关流控”** -> **“API 管理”**
2. 点击 **“+新增 API 分组”**
3. **API 名称**：`order_write_group`
4. **匹配模式**：
   - **精确匹配**：`/order/create`
   - **前缀匹配**：`/order/update/**`
   - **正则匹配**：`^/order/.*write.*$`
5. 保存后，回到 **“请求链路”** 页面，点击 **“+流控”**
   - **API 类型**：选择 **API 分组**
   - **API 名称**：选择刚才创建的 `order_write_group`



**代码方式 (Init)**：

```java
private void initCustomAPIs() {
    Set<ApiDefinition> definitions = new HashSet<>();
    
    // 定义一个分组：my_api_group
    ApiDefinition api1 = new ApiDefinition("my_api_group")
        .setPredicateItems(new HashSet<ApiPredicateItem>() {{
            // 规则 1：精确匹配 /order/create
            add(new ApiPathPredicateItem().setPattern("/order/create")
                .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_EXACT));
            
            // 规则 2：前缀匹配 /order/update/**
            add(new ApiPathPredicateItem().setPattern("/order/update/**")
                .setMatchStrategy(SentinelGatewayConstants.URL_MATCH_STRATEGY_PREFIX));
        }});
    
    definitions.add(api1);
    // 加载定义
    GatewayApiDefinitionManager.loadApiDefinitions(definitions);
}
```



###### 2. 网关热点参数限流

- 这是网关层最强大的功能之一
- **场景**：你需要限制 **每个客户端 IP** 每秒只能访问 10 次网关，防止恶意爬虫或 DDOS 攻击
  - **不同点**： 网关的热点参数限流，**不需要**像普通微服务那样配 `@SentinelResource` 和索引下标。它内置了多种参数解析策略



- **支持的参数类型**：

  - **Client IP** (内置最常用)

  - **Remote Host**

  - **Header** (例如限制某个 `token` 的访问频率)

  - **URL Param** (例如 `?userId=xxx`)

  - **Cookie**



- **配置实战 (Dashboard 暂不支持完全配置)**： 
  - 目前 Sentinel Dashboard (1.8.x) 对网关热点参数的可视化支持较弱，通常建议直接通过 **代码** 或 **Nacos 配置** 来加载规则



**代码示例：针对 IP 限流**

```java
private void initGatewayRules() {
    Set<GatewayFlowRule> rules = new HashSet<>();
    
    // 针对 "order_route" 这个路由配置规则
    rules.add(new GatewayFlowRule("order_route")
        .setCount(10) // 限流阈值 (QPS 10)
        .setIntervalSec(1) // 统计窗口 1秒
        // 【关键】配置参数限流项
        .setParamItem(new GatewayParamFlowItem()
            // 策略：解析客户端 IP
            .setParseStrategy(SentinelGatewayConstants.PARAM_PARSE_STRATEGY_CLIENT_IP)
        )
    );
    
    GatewayRuleManager.loadRules(rules);
}
```

- **效果解析**： 上述代码生效后，Sentinel 会自动提取请求中的 Client IP

  - IP `1.1.1.1` 访问 QPS > 10 -> **被拦截**

  - IP `2.2.2.2` 访问 QPS < 10 -> **通过**

  - 实现了“千人千面”的流量控制，互不影响



###### 3. 全局过滤器与跨域配置 (Tips)

在集成 Sentinel Gateway 时，经常会遇到 **CORS (跨域)** 问题被拦截器吞掉的情况

- **执行顺序**：Sentinel 的 `GlobalFilter` 优先级非常高
- **现象**：如果请求被限流，Sentinel 直接返回了 JSON，导致前端收不到 `Access-Control-Allow-Origin` 头，浏览器报跨域错误，而不是业务错误
- **解决**：在自定义异常处理器（`BlockRequestHandler`）中，手动把 CORS 头加上，或者调整 Filter 的 Order



## 分布式事务

### seata

#### 1. 分布式事务综述

##### 1.1 痛点：本地事务的失效

在学习 Seata 之前，我们必须先彻底搞清楚一个问题：**我们在单体应用里用得好好的 `@Transactional`，为什么一到微服务架构就崩了？**

- 这不仅仅是“配置”变了，而是底层的 **物理边界** 变了



###### A. 传统单体项目

在传统的单体应用中，Spring 的声明式事务（`@Transactional`）是基于 **本地事务** 实现的

- **物理基础**：同一个 JVM 进程，同一个数据库实例
- **核心机制**：Spring 的 `DataSourceTransactionManager` 利用了 `java.sql.Connection` 对象
  1. 开启事务：`connection.setAutoCommit(false)`
  2. 业务执行：所有 SQL 都在这一条连接上执行
  3. 结束事务：如果无异常，执行 `connection.commit()`；如果有异常，执行 `connection.rollback()`

**结论**：只要是在同一个 JVM 内，Spring 就能通过 `ThreadLocal` 绑定同一个数据库连接，从而保证 ACID（原子性、一致性、隔离性、持久性）



###### B. 微服务下的“崩塌”

当我们把单体拆分为微服务（如：订单服务、库存服务、账户服务）后，情况发生了本质变化：

- **跨进程**：服务之间通过 RPC（Feign/Dubbo）通信，不再是本地方法调用
- **跨资源**：每个微服务通常连接自己的数据库



**场景复现：电商下单流程**

假设我们有一个下单方法，逻辑如下：

```java
// 伪代码：OrderService (订单微服务)

@Transactional // 这里开启的是 OrderDB 的本地事务
public void placeOrder(OrderDTO order) {

    // 1. 本地操作：插入订单 (操作 OrderDB)
    orderMapper.insert(order); 
    
    // 2. 远程调用：扣减库存 (RPC 调用 StockService)
    // 注意：这里是跨网调用，StockService 会开启它自己的本地事务
    stockClient.deduct(order.getProductId(), order.getCount()); 
    
    // 3. 模拟异常：比如计算金额时发生了除以零，或者空指针
    int i = 10 / 0; 
    
    //...其它逻辑
}
```



###### C. 事故现场分析

当上述代码执行到第 3 步发生异常时，Spring 的事务管理器会捕获这个异常，并试图回滚

- **OrderService 的视角**：

  - 捕获到异常
  - 拿到 `OrderDB` 的数据库连接，执行 `rollback()`
  - **结果**：刚才插入的“订单记录”被成功回滚，数据库里查不到这条订单**（符合预期）**

  

- **StockService 的视角**：

  - 它在第 2 步被调用
  - 它开启了自己的 `StockDB` 事务，扣减库存，提交事务（`commit`）
  - 它的任务已经完成，连接已经归还给连接池
  - **结果**：库存已经被扣减，且数据已经持久化到磁盘（无法回滚）



###### D. 核心矛盾总结

这个场景下，导致了严重的**数据不一致**：**没有生成订单，但是库存却被扣了**

本地事务失效的根本原因在于：

- **Spring 的 `TransactionManager` 的管辖范围仅限于当前 JVM 内部持有的数据库连接**

  **它无法感知、更无法控制运行在另一个 JVM（库存服务）里的数据库事务提交与否**



这就是 **分布式事务** 要解决的核心痛点：我们需要一个“上帝视角”的协调者，来统筹管理这些分布在不同 JVM、不同数据库里的本地事务，让它们 **“要么一起成功，要么一起失败”**



##### 1.2 理论基石：CAP 与 BASE

- 上一节我们明确了本地事务在微服务中失效的物理原因。本节我们将探讨分布式系统的 **核心边界**
  - 不理解这个边界，就无法理解 Seata 不同模式（AT/XA/TCC）的存在意义



###### a. CAP 定理的严格定义

2000 年 Eric Brewer 提出的 CAP 定理是分布式系统的物理法则。它指出在分布式系统中，以下三个指标 **不可能同时达成**：

- **C (Consistency) 一致性**

  - **严格定义**：线性一致性
  - **技术指标**：在分布式系统中的所有数据副本，在同一时刻的值必须完全相同
  - **验证标准**：写入操作一旦返回成功，所有节点在随后的读取操作中都必须能读到这个新值

  

- **A (Availability) 可用性**

  - **严格定义**：高可用性
  - **技术指标**：服务一直可用，对任何非故障节点的请求，必须在有限时间内返回 **非错误** 的响应
  - **验证标准**：不能返回“系统繁忙”或无限等待，必须返回结果（哪怕是旧数据）

  

- **P (Partition tolerance) 分区容错性**

  - **严格定义**：网络分区耐受能力
  - **技术指标**：分布式系统在遇到任何网络分区故障（节点之间无法通信）时，仍然能够对外提供满足 C 或 A 的服务



###### b. 为什么三者不可兼得？

很多初学者的困惑在于：“为什么我不能设计一个完美的 CAP 系统？” 我们可以通过一个 **逻辑推导** 来证明其不可能性

- **前提条件（P 是必然的）：** 

  - 在分布式系统中，网络是不靠谱的（光纤被挖断、路由抖动）

    假设我们有两个节点 **Node A** 和 **Node B**，它们之间同步数据。一旦网络断开（**P 发生**），系统面临如下场景：

- **场景演练：**
  1. 客户端向 **Node A** 发送写请求：`set balance = 100`
  2. **Node A** 接收请求，但在尝试同步给 **Node B** 时，发现网络不通



**此时，架构师面临唯一的两难抉择：**

- **抉择一：死保一致性 (Choose C, Drop A)**

  - **逻辑**：为了保证 Node A 和 Node B 数据绝对一致

  - **行动**：Node A 检测到无法同步，于是 **拒绝** 或 **阻塞** 客户端的写请求

  - **结果**：数据没乱（C 满足），但系统报错不可用了（A 丢失）

  - **架构模式**：**CP 系统**（如Zookeeper、Seata XA）

    > CP 的意思是 “**在容忍网络分区的前提下**，死保一致性”

  

- **抉择二：死保可用性 (Choose A, Drop C)**

  - **逻辑**：无论发什么事，必须让用户能用

  - **行动**：Node A **接受** 写请求，更新本地数据为 100

  - **结果**：系统可用（A 满足），但此时 Node B 还是旧值，两边数据不一致（C 丢失）

  - **架构模式**：**AP 系统**（如 Eureka、Seata AT/TCC）

    >AP 的意思是“**在容忍网络分区的前提下**，死保可用性”



**推导结论：** **P（分区）是前提，当 P 发生时，我们只能在 C 和 A 之间做“二选一”的零和博弈**





###### c. 工业界的解决方案：BASE 理论

鉴于 **CP 系统（强一致性）**通常伴随着性能低下和可用性丧失，互联网业务（如电商、社交）通常**无法接受 CP 架构**

- 于是，eBay 的架构师提出了 **BASE 理论** , 可以说，**BASE 理论就是 AP 架构的“生存法则”**



**核心思想**：既然无法做到强一致性（Strong Consistency），我们可以追求 **最终一致性**，用时间换空间

- **BA (Basically Available) 基本可用**

  - 分布式系统在出现故障时，允许损失部分可用性（例如：响应时间变长、功能降级），但核心功能依然可用
  - **实战**：双11期间，为了保下单（核心），暂时关闭评价功能（非核心）

  

- **S (Soft state) 软状态**

  - 允许系统存在中间状态，该状态不影响系统整体可用性
  - **实战**：Seata AT 模式中，一阶段提交后，数据库数据已经修改，但全局事务并未结束。此时的数据状态就是“软状态”

  

- **E (Eventually consistent) 最终一致性**

  - 系统保证在没有新的更新操作下，经过一段时间（可能是毫秒级，也可能是分钟级），所有数据副本最终会达到一致
  - **实战**：Seata 发现异常后，通过 Undo Log 异步回滚，将数据修正回原始状态



###### 4. Seata 的架构定位

理解了 CAP 和 BASE，我们就看懂了 Seata 的版图：

1. **Seata AT / TCC 模式**：

   - **定位**：**AP 模型**，遵循 **BASE 理论**
   - **特征**：允许短暂的数据不一致（软状态），通过补偿机制实现最终一致性
   - **适用**：绝大多数高并发互联网业务

   

2. **Seata XA 模式**：

   - **定位**：**CP 模型**，遵循 **ACID 理论**
   - **特征**：利用数据库锁机制，强一致，但性能差
   - **适用**：对一致性要求极高且并发低的金融核心业务





#### 2. Seata 架构

Seata 的架构设计参考了 DTP (Distributed Transaction Processing) 模型，但对其进行了针对微服务的改造

- 它由三个核心角色组成：**TC (事务协调者)**、**TM (事务管理器)**、**RM (资源管理器)**

  > 可以把它们看作是一个 **“指挥中心 + 发起人 + 执行者”** 的协作团队



##### 2.1 核心组件详解 (The Trinity)

###### A. TC - 事务协调者

> Transaction Coordinator

- **角色定位**：**服务端 (Server)**。它是独立的中间件服务（类似 Nacos、Redis），需要单独部署
- **核心职责**：
  - **维护全局事务状态**：记录哪个事务开始了，哪个分支事务提交了，哪个失败了
  - **决策者**：接收 TM 的提交/回滚指令，然后指挥所有的 RM 进行提交或回滚
  - **全局锁管理**：管理全局行锁（Global Lock），防止脏写
- **物理形态**：就是下载的 `seata-server.jar` 包，运行在独立的服务器上



###### B. TM - 事务管理器

> Transaction Manager

- **角色定位**：**客户端 (Client)**。它嵌入在业务微服务中（通常是发起方）
- **核心职责**：
  - **发起人**：它是全局事务的起点
  - **边界定义**：决定全局事务在哪里开始，在哪里结束
  - **指令下达**：向 TC 申请开启全局事务，根据业务执行结果，向 TC 发起全局提交（Commit）或全局回滚（Rollback）的决议
- **代码体现**：通常通过注解 `@GlobalTransactional` 标记的方法，就是 TM 的活动范围



###### C. RM - 资源管理器

> Resource Manager

- **角色定位**：**客户端 (Client)**。也嵌入在业务微服务中（通常是参与方/被调用方）
- **核心职责**：
  - **执行者**：管理本地数据库资源
  - **注册分支**：当它被调用执行 SQL 时，它会自动向 TC 注册一个“分支事务”，告诉 TC：“我也参与了这个事务”
  - **汇报状态**：向 TC 汇报本地事务是成功了还是失败了
  - **执行指令**：接收 TC 的指令，完成二阶段的提交或回滚（例如执行 Undo Log 反向补偿）
- **代码体现**：Seata 自动代理的数据源



##### 2.2 核心纽带：XID (Transaction ID)

这三个组件分布在不同的网络节点上，它们靠什么识别彼此属于同一个“团伙”？ 答案是 **XID (Global Transaction ID)**

- **定义**：全局事务的唯一标识符（例如：`192.168.1.5:8091:12345678`）
- **传递机制**：
  1. **生成**：TM 向 TC 申请开启事务时，TC 生成 XID 并返回给 TM
  2. **传播**：TM 在调用下游微服务（RPC）时，会将 XID 放入请求头（Header）中传给下游
  3. **绑定**：下游的 RM 从请求头拿到 XID，把它和本地执行的 SQL 绑定在一起，向 TC 注册

**上下文理解**： 如果没有 XID 的传递，下游服务就不知道自己处于一个全局事务中，它就会按照普通的本地事务执行（立马提交），一旦需要回滚就完了



##### 2.3 全局事务的生命周期

一个典型的分布式事务生命周期，可以被拆解为 **4 个关键步骤**

- 假设场景：**用户下单**（TM），调用 **库存服务**（RM1）扣库存，调用 **订单服务**（RM2）创建订单



###### 第一阶段：执行与决议

这是“干活”的阶段。所有的业务 SQL 都在这个阶段执行完毕，并且 **本地事务已经提交**（注意：数据库里的数据已经改了）

1. **TM 开启全局事务**

   - **动作**：用户请求进入下单接口（带有 `@GlobalTransactional`）。TM 向 TC 发起请求：“我要开启一个全局事务”
   - **结果**：TC 生成一个全局唯一的 **XID**，并返回给 TM

   

2. **XID 传播**

   - **动作**：TM 调用库存服务（RPC）
   - **细节**：Seata 的拦截器会自动把 **XID** 塞到 RPC 请求的 Header 中，传给库存服务

   

3. **RM 注册分支与执行**

   - **动作**：
     1. 库存服务的 RM 拦截到 SQL
     2. **解析 SQL**：识别出是 `UPDATE stock ...`
     3. **生成镜像**：查询更新前的数据（Before Image），执行更新，查询更新后的数据（After Image），把这些信息打包成 **Undo Log**
     4. **注册分支**：RM 拿着 XID 向 TC 汇报：“我是库存服务，我参与了 XID 为 xxx 的事务，我的分支 ID 是 yyy”
     5. **本地提交**：RM 在同一个本地事务中，**同时提交** 业务 SQL 和 Undo Log

   

4. **TM 决议 (Decide)**

   - **场景 A（一切顺利）**：所有微服务调用都成功返回。TM 决定：**提交 (Commit)**
   - **场景 B（发生异常）**：任何一个环节抛出了异常。TM 决定：**回滚 (Rollback)**



###### 第二阶段：全局提交或回滚

这是“收尾”的阶段。TC 根据 TM 的决议，指挥所有 RM 进行清理或还原

**场景 A：全局提交 (Global Commit)**

- **触发**：TM 向 TC 发送 `GlobalCommit` 指令

- **TC 动作**：TC 发现所有分支都成功了

- **RM 动作**：

  - 因为一阶段本地事务已经真的提交了（数据是对的），所以二阶段非常快
  - **异步清理**：TC 通知 RM：“完事了，把刚才记的 Undo Log 删了吧，留着占地方”
  - *注：AT 模式的提交是**异步**的，性能极高*

  

**场景 B：全局回滚 (Global Rollback)**

- **触发**：TM 捕获异常，向 TC 发送 `GlobalRollback` 指令
- **TC 动作**：TC 查到该 XID 下所有的分支记录
- **RM 动作**：
  - **同步回滚**：TC 立即通知所有 RM：“出事了，快回滚！”
  - **反向补偿**：RM 收到 XID，去数据库查 Undo Log
  - **还原数据**：根据 After Image 和 Before Image，生成反向 SQL（例如把 `stock=99` 改回 `stock=100`）并执行
  - **完成**：回滚成功后，向 TC 汇报



##### 2.4 Seata 的四种模式概览

理解了生命周期，我们需要知道 Seata 不止有一种玩法。虽然 90% 的场景用 AT，但了解其他模式能让你在架构选型时更专业

| 模式     | 全称                  | 核心特点                                  | 侵入性 | 适用场景                                                    |
| -------- | --------------------- | ----------------------------------------- | ------ | ----------------------------------------------------------- |
| **AT**   | Automatic Transaction | **无侵入**、两阶段提交、自动生成 Undo Log | 低     | **首选**。绝大多数业务场景（Web应用、电商）                 |
| **TCC**  | Try-Confirm-Cancel    | **高性能**、手动实现三个方法              | 高     | 核心系统，对性能要求极高，或不支持 SQL 的数据库（如 Redis） |
| **Saga** | Saga                  | **长事务**、基于状态机、无锁              | 中     | 业务流程极长、跨机构调用的场景（如银行跨行转账）            |
| **XA**   | eXtended Architecture | **强一致**、数据库原生支持、阻塞等待      | 低     | 金融核心，不能容忍任何中间状态，并发要求不高                |



#### 3. Seata Server (TC) 部署与配置

理论部分翻篇了。现在的任务是：**把 Seata 的服务端（TC）跑起来**

- 在 Spring Cloud Alibaba 生态中，Seata Server 通常不单打独斗，而是和 **Nacos** 深度绑定（用 Nacos 做注册中心和配置中心）



##### 3.1 TC 的存储模式

- 在启动 Seata Server 之前，你必须做一个架构决策：**Seata 运行过程中的数据存哪里？**

  - Seata Server (TC) 在运行时，需要记录所有 **正在进行中** 的全局事务状态

    - 比如：事务 `XID:1001` 正在进行中，它下面挂了 3 个分支事务，其中 2 个已经提交了，还有 1 个没跑完

      这些数据如果不持久化，一旦 TC 重启或宕机，正在运行的事务就会丢失（导致数据不一致）

      Seata 提供了三种存储模式，由 `store.mode` 参数控制



###### a. File 模式 (文件存储)

- **原理**：TC 将事务会话数据序列化后，直接写入本地磁盘文件（默认路径为 `sessionStore/root.data`）
- **优点**：
  - **零依赖**：不需要连接数据库或 Redis，解压即用
  - **快速上手**：最适合本地开发调试
- **缺点 (致命)**：
  - **单点故障**：数据在本地文件里，意味着你没法部署多个 TC 节点组成集群（因为文件没法共享）。如果这台机器挂了，事务状态就丢了
  - **性能瓶颈**：受限于本地磁盘 IO
- **适用场景**：本地开发 (Local Development)、功能验证



###### b. DB 模式 (数据库存储) —— **生产**

- **原理**：TC 将会话数据存入共享的数据库（MySQL/Oracle/PostgreSQL）。需要提前建三张表：
  - `global_table`：存储全局事务信息（XID, 状态, 超时时间）
  - `branch_table`：存储分支事务信息（分支 ID, 资源 ID）
  - `lock_table`：存储全局锁信息（用于隔离性控制）
- **优点**：
  - **高可用 (HA)**：多个 TC 节点连接同一个数据库，天然支持集群部署。任何一个 TC 挂了，别的 TC 可以接手继续处理
  - **数据安全**：依赖成熟数据库的持久化能力
- **缺点**：
  - **依赖性**：需要维护一个额外的数据库
  - **性能上限**：受限于数据库的写性能（TPS）
- **适用场景**：**生产环境**



###### c. Redis 模式 (缓存存储)

- **原理**：TC 将会话数据存入 Redis
- **优点**：
  - **高性能**：基于内存操作，吞吐量极高
  - **支持集群**：多个 TC 可以共享 Redis 数据
- **缺点**：
  - **数据风险**：虽然 Redis 有 AOF/RDB，但在极端断电或 Redis 集群故障时，仍有丢失少量事务数据的风险（相比 DB 模式）
- **适用场景**：对吞吐量要求极高、且能容忍极低概率事务丢失的场景



###### d. 选型对比总结

| 特性           | File 模式          | DB 模式        | Redis 模式     |
| -------------- | ------------------ | -------------- | -------------- |
| **部署架构**   | 单机 (Stand-alone) | 集群 (Cluster) | 集群 (Cluster) |
| **读写性能**   | 中                 | 低 (受限于 DB) | **极高**       |
| **数据可靠性** | 低 (单机风险)      | **高** (ACID)  | 中 (AOF/RDB)   |
| **外部依赖**   | 无                 | 数据库         | Redis          |
| **推荐环境**   | 开发/测试          | **生产环境**   | 高并发生产     |



##### 3.2 关键配置文件

进入 Seata Server 的 `conf` 目录，你会看到两个核心配置文件。很多初学者容易混淆它们的职责，导致连不上 Nacos 或者改了配置不生效

###### a. registry.conf (入口文件:寻址录)

这是 Seata Server 启动时 **第一个** 读取的文件。它决定了 Seata Server 的“外交关系”。它包含两大核心配置块：

- **registry (注册中心)**：

  - **作用**：决定 TC 把自己注册到哪里（Nacos, Eureka, Zookeeper 等）
  - **目的**：让微服务（TM/RM）能发现并连接上 TC

  

- **config (配置中心)**：

  - **作用**：决定 TC 去哪里读取详细的运行参数（比如数据库连接 URL、存储模式、超时时间等）
  - **重要逻辑**：
    - 如果 `type = "file"`：读取本地的 `file.conf`
    - 如果 `type = "nacos"`：忽略本地 `file.conf`，直接去 Nacos 读取配置

- **生产环境推荐配置 (基于 Nacos):**

  ```js
  registry {
    # 注册中心类型：nacos, eureka, redis, zk, consul, etcd3, sofa
    type = "nacos"
  
    nacos {
      application = "seata-server"
      serverAddr = "127.0.0.1:8848"
      group = "SEATA_GROUP"
      namespace = ""
      cluster = "default"
    }
  }
  
  config {
    # 配置中心类型：nacos, consul, apollo, etcd3, zk
    type = "nacos"
  
    nacos {
      serverAddr = "127.0.0.1:8848"
      namespace = ""
      group = "SEATA_GROUP"
    }
  }
  ```

  



###### b. file.conf (参数表：详细配置)

这个文件包含了 Seata Server 运行所需的所有详细参数（如 `store.mode`, `transport.threadFactory` 等）

**生效规则：** 

- 只有当 `registry.conf` 中配置了 `config.type = "file"` 时，这个文件才生效。如果你选择了 Nacos 作为配置中心，**修改这个文件是没有任何用的**



**关键参数 (以 DB 模式为例):**

```js
store {
  # 1. 存储模式：file, db, redis
  mode = "db"

  # 2. 数据库连接配置 (仅在 mode = "db" 时生效)
  db {
    datasource = "druid"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true"
    user = "root"
    password = "password"
    minConn = 5
    maxConn = 30
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
  }
}

service {
  # 3. 事务分组映射 (非常重要)
  # 格式: service.vgroupMapping.事务分组名 = SeataServer集群名
  # default 是 Seata Server 默认的集群名
  vgroupMapping.my_test_tx_group = "default"
  
  # 4. 降级开关
  enableDegrade = false
  # 5. 是否禁用全局事务 (用于故障紧急切断)
  disableGlobalTransaction = false
}
```



###### **c. 避坑指南**

- 在生产环境下，既然我们通常用 Nacos 做配置中心，那么你需要把 `file.conf` 里的内容（如 `store.mode`, `store.db.url` 等）**全部迁移** 到 Nacos 的配置列表中，而不是在服务器上改文件

- Seata 官方提供了一个脚本 `nacos-config.sh`，可以一键将 `config.txt`（`file.conf` 的扁平化版本）推送到 Nacos



##### 3.3 部署实战步骤 (DB 模式 + Nacos)

这是 Seata 最经典、最稳定的部署方案。请严格按照顺序执行，任何一步跳过都会导致启动报错



###### 第一步：初始化数据库

TC 需要数据库来存储全局事务会话。我们需要先建表

1. **创建数据库**： 在你的 MySQL 中创建一个名为 `seata` 的数据库（字符集推荐 `utf8mb4`）
2. **执行建表脚本**： 去 Seata 官方 GitHub 仓库下载对应版本的脚本（`script/server/db/mysql.sql`）
   - **核心表说明**：
     - `global_table`: 存储全局事务（谁开启了事务，状态是 Begin 还是 Committing）
     - `branch_table`: 存储分支事务（谁参与了事务，Resource ID 是多少）
     - `lock_table`: **全局锁** 表（用于校验脏写，非常关键）



###### 第二步：修改 registry.conf

找到 Seata Server 的 `conf/registry.conf`，告诉 TC 去连 Nacos

```js
registry {
  type = "nacos"
  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = ""
    cluster = "default"
  }
}

config {
  type = "nacos"
  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = ""
  }
}
```



###### 第三步：推送到 Nacos 配置中心

**这是最容易出错的一步！** 

- 因为我们在第二步设置了 `config.type = "nacos"`，所以 Seata 启动时 **不会** 去读本地的 `file.conf`，而是去 Nacos 找配置

  此时 Nacos 里是空的，TC 启动就会报错。我们需要把配置参数推上去



**操作方法：**

1. **准备配置文本**：你可以把 `file.conf` 的内容转化为 Nacos 的 `Properties` 格式，或者直接使用 Seata 提供的 `config.txt` 模板
2. **核心配置项 (必须确保 Nacos 里有这些 key)**：
   - `store.mode` = `db`
   - `store.db.driverClassName` = `com.mysql.jdbc.Driver`
   - `store.db.url` = `jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true`
   - `store.db.user` = `root`
   - `store.db.password` = `123456`
   - `service.vgroupMapping.my_test_tx_group` = `default` (这是事务分组，后面客户端要用)

**便捷神器**： Seata 源码包里提供了一个脚本 `script/config-center/nacos/nacos-config.sh`

```bash
# 示例命令：把 config.txt 推送到 Nacos
sh nacos-config.sh -h 127.0.0.1 -p 8848 -g SEATA_GROUP -t "" -u nacos -w nacos
```



###### 第四步：启动 Seata Server

配置完成后，启动服务

- **Docker 方式 (推荐)**：

  ```bash
  docker run -d --name seata-server \
  -p 8091:8091 \
  -e SEATA_IP=192.168.x.x \
  -e SEATA_CONFIG_NAME=file:/root/seata-config/registry.conf \
  seataio/seata-server
  ```

  *(注意：挂载 registry.conf 且确保容器能连上宿主机的 Nacos 和 MySQL)*

- **脚本方式**：

  - Linux/Mac: `sh bin/seata-server.sh`
  - Windows: 点击`bin/seata-server.bat`



###### 第五步：验证部署

1. **检查日志**： 观察启动日志，看到类似 `Server started ... listening on 8091` 字样
2. **检查 Nacos 服务列表**： 登录 Nacos 控制台 -> 服务管理。你应该能看到一个名为 `seata-server` 的服务，且实例数 > 0
3. **检查数据库**： 虽然此时表是空的，但在后续事务运行时，`global_table` 会有数据进出



#### 4. Spring Cloud 整合 Seata Client

服务端（TC）搭建完毕后，接下来回到微服务代码中（OrderService, StorageService）

- 我们需要做三件事：加依赖、写配置、建表



##### 4.1 核心依赖与版本避坑

在 Spring Cloud Alibaba 生态中，整合 Seata 看似只需要引入一个 Starter，但这里藏着一个极易导致项目启动失败的 **版本冲突陷阱**

###### a. 为什么需要依赖管理？

- **接入能力**：我们需要引入 Seata 的客户端 SDK，让微服务拥有连接 TC、注册分支事务、拦截 SQL 的能力
- **版本对齐**：Spring Cloud Alibaba 内部集成的 Seata SDK 版本通常滞后于 Seata 官方发布的 Server 版本。如果不手动调整，会导致 **通信协议不兼容**



###### b. 依赖引入最佳实践

请打开你所有需要参与分布式事务的微服务（如 `order-service`）的 `pom.xml`

**标准配置代码：**

```xml
<!-- 1. 引入 Spring Cloud Alibaba Seata Starter -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <!-- 【关键动作】排除内置的 seata-all，防止版本冲突 -->
    <exclusions>
        <exclusion>
            <groupId>io.seata</groupId>
            <artifactId>seata-spring-boot-starter</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- 2. 手动引入与 Server 端一致的 Seata 版本 -->
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <!-- 【铁律】必须与你部署的 Seata Server 版本完全一致！-->
    <!-- 例如：服务端是 1.4.2，这里就必须写 1.4.2 -->
    <version>1.4.2</version> 
</dependency>
```



###### c. 避坑指南

- **现象**：启动报错 `java.lang.AbstractMethodError` 或者连接 TC 时报序列化异常

- **原因**：

  - `spring-cloud-starter-alibaba-seata` 默认依赖的可能是 `seata-all 1.3.0`，而你部署的 Server 是 `1.4.2`

    不同版本的 RPC 协议或序列化机制可能不兼容

- **解决**：如上代码所示，**排除默认依赖，显式指定版本**。这是生产环境最稳妥的做法



##### 4.2 Client 端核心配置

引入依赖后，微服务还不知道 TC 在哪。我们需要在 `application.yml`（推荐在 `bootstrap.yml` 中，如果你用了 Nacos 配置中心）添加 Seata 的专属配置



###### 1. 核心配置清单

打开你的微服务配置文件，加入以下内容。这是最简、最标准的 Nacos 集成模式

```yaml
seata:
  # 1. 开关：是否启用 Seata 分布式事务（默认 true）
  enabled: true
  
  # 2. 【核心】事务分组 (Transaction Service Group)
  # 这是一个逻辑名称，代表当前微服务属于哪个“事务组”
  # 它的值通常是：${spring.application.name}-group
  # 例如：order-service-group
  tx-service-group: my_test_tx_group 
  
  # 3. 注册中心配置：告诉 Client 去 Nacos 找 TC
  registry:
    type: nacos
    nacos:
      application: seata-server # 必须与 TC 在 Nacos 注册的服务名一致
      server-addr: 127.0.0.1:8848
      group: SEATA_GROUP # TC 注册时的分组 (registry.conf 里的)
      namespace: "" # 如果 TC 在特定命名空间，这里要填 ID
      
  # 4. 配置中心配置：告诉 Client 去 Nacos 拉取详细配置
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      group: SEATA_GROUP 
      namespace: ""
```



###### 2. 关键参数

- **`seata.tx-service-group` (重中之重)**

  - **定义**：客户端的 **逻辑分组标识**
  - **作用**：它 **不是** TC 的服务名，也不是集群名。它只是一个 **标签**。客户端拿着这个标签，去配置中心问：“我这个组应该连哪个 TC 集群？”
  - **避坑**：很多初学者随便填一个名字，结果报错 `no available service 'null' found`。这是因为它缺少了 **下一步的映射关系**（4.3 节）

  

- **`seata.registry.nacos.application`**

  - **定义**：Seata Server 在 Nacos 上的服务名

  - **默认值**：`seata-server`

  - **注意**：

    - 该值必须与 Seata Server 启动参数或 `registry.conf` 中配置的 `application` 保持严格一致

      如果服务端自定义了名称（如 `seata-tc-cluster`），此处必须同步修改，否则服务发现机制将失效

  

- **`seata.config.type`**

  - **作用**：

    - 决定了客户端启动时，去哪里读取诸如 `transport.heartbeat`、`client.undo.logTable` 等底层参数

      设置成 `nacos` 可以实现配置动态刷新

  - **最佳实践**：生产环境推荐设置为 `nacos`
  - **作用**：启用后，客户端将从 Nacos 拉取配置。这允许运维人员在不重启微服务的情况下，动态调整 Seata 的运行时参数（如全局锁重试间隔）



配置完成后，客户端具备了以下能力：

1. 拥有了一个逻辑标识 `my_test_tx_group`
2. 晓了去 `127.0.0.1:8848` 寻找名为 `seata-server` 的服务实例

**遗留问题：** 客户端持有的逻辑组名 `my_test_tx_group` 是如何解析并指向具体的 Seata Server 集群的？

- 这涉及到 Seata 核心的**事务分组映射机制**



##### 4.3 事务分组与映射关系

在上一节的配置中，我们定义了一个核心参数 `seata.tx-service-group = my_test_tx_group`

- 初学者常有的疑问是：*为什么不直接填 Seata Server 的 IP 地址？为什么要绕这么大一个弯子搞个“分组名”？*
  - 这背后的设计哲学是：**解耦**



###### 1. 映射机制链路

当微服务启动并尝试连接 Seata Server 时，会经历一个**“三级查找”**的过程

- **第一级：应用层**

  - **配置**：`seata.tx-service-group`
  - **值**：`my_test_tx_group` (逻辑组名)
  - **作用**：微服务只知道自己属于哪个逻辑组，不知道具体的服务器在哪

  

- **第二级：配置中心层**

  - **配置**：Client 去 Nacos 配置中心查找 key 为 `service.vgroupMapping.my_test_tx_group` 的配置项
  - **值**：`default` (Seata Server 的 **物理集群名称**)
  - **作用**：将“逻辑组名”映射为具体的“后端集群名”

  

- **第三级：服务发现层**

  - **配置**：Client 拿到集群名 `default` 后，去 Nacos 注册中心查找服务名为 `seata-server` 且 **cluster** 属性为 `default` 的实例列表
  - **值**：`192.168.1.100:8091` (真实 IP)
  - **作用**：最终定位到物理机器



###### 2. 为什么要这么设计？

这种设计看似繁琐，实则是为了**高可用 (HA)** 和 **故障隔离**

**场景演练：TC 集群故障切换** 

假设你现在的生产环境连接的是 `beijing-cluster` (机房 A)。突然机房 A 断电了，你需要紧急切到 `shanghai-cluster` (机房 B)

- **如果没有分组映射**：你需要修改所有微服务的 `application.yml`，把 IP 改成上海机房的 IP，然后 **重启所有微服务**。这在生产环境是灾难
- **有了分组映射**：你只需要在 Nacos 配置中心，把 `service.vgroupMapping.my_test_tx_group` 的值从 `beijing-cluster` 改为 `shanghai-cluster`
  - **结果**：所有微服务监听配置变化，自动断开旧连接，连上新集群。**无需重启，秒级切换**



###### 3. 避坑指南：常见报错排查

**报错信息**： `no available service 'null' found, please make sure registry config correct`

**排查步骤**：

1. **检查本地配置**：确认 `application.yml` 里 `tx-service-group` 的值（例如叫 `my_group`）
2. **检查 Nacos 配置**：确认 Nacos 配置列表里是否真的有一个 DataID 包含 `service.vgroupMapping.my_group`
3. **检查映射值**：确认这个 Key 对应的值（例如 `default`），是否和 Seata Server 启动时注册的集群名一致（默认是 `default`）



**逻辑组名 -> (通过配置中心) -> 集群名 -> (通过注册中心) -> 实例 IP**



##### 4.4 数据库表结构准备 (`undo_log` 表)

在 AT 模式下，Seata 需要记录数据的“前置镜像”和“后置镜像”来实现自动回滚。这些数据不是存在内存里，而是必须持久化到 **业务数据库** 中

这就要求：**每一个参与分布式事务的业务数据库（如 OrderDB, StorageDB），都必须创建这张 `undo_log` 表**



###### 1. 核心作用

- **存什么？**：存储 SQL 执行前的数据快照（Before Image）和执行后的数据快照（After Image）
- **谁来存？**：Seata 的代理数据源 (`DataSourceProxy`) 会在业务 SQL 执行的同一个本地事务中，自动插入这条 Log
- **怎么用？**：
  - **提交时**：异步删除这条 Log
  - **回滚时**：读取这条 Log，生成反向 SQL 执行补偿



###### 2. 建表脚本 (SQL Script)

在你的 **每一个微服务对应的数据库** 中执行以下 SQL

> **注意**：这不是在 Seata Server 的数据库里建，而是在你的业务库（如 `order_db`, `stock_db`）里建！

```sql
-- 注意：这是 MySQL 的脚本
-- 每个业务库（OrderDB, StockDB, AccountDB）都要执行
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL COMMENT '分支事务ID',
  `xid` varchar(100) NOT NULL COMMENT '全局事务ID',
  `context` varchar(128) NOT NULL COMMENT '上下文',
  `rollback_info` longblob NOT NULL COMMENT '回滚信息(存镜像数据)',
  `log_status` int(11) NOT NULL COMMENT '状态',
  `log_created` datetime NOT NULL COMMENT '创建时间',
  `log_modified` datetime NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```



###### 3. 避坑指南

1. **表名大小写**：

   - Seata 默认查找的表名是 `undo_log`（小写）

     如果你的数据库配置了大小写敏感，或者你想改表名，需要在 `application.yml` 中配置 `seata.client.undo.log-table`

     但 **强烈建议保持默认**

2. **序列化问题**：

   - `rollback_info` 字段存储的是二进制数据（默认用 Jackson 序列化）

     请确保数据库该字段类型是 `longblob` 或足够大的 `blob`，否则复杂对象的镜像数据可能会截断导致无法回滚

3. **多数据源**：如果你的微服务使用了多数据源（Dynamic Datasource），**每一个数据源** 对应的物理库里，都要有这张表





所有的准备工作彻底完成：

1. **TC (Server)**：部署好了，配置进 Nacos 了
2. **TM/RM (Client)**：依赖加了，配置连上 TC 了
3. **DB (Resource)**：`undo_log` 表建好了



#### 5. AT 模式原理

- AT (Automatic Transaction) 模式是 Seata 独创的模式，它的核心理念是：**对业务无侵入**

  - 你只需要在代码上加一个 `@GlobalTransactional` 注解，Seata 就能在底层自动帮你完成分布式事务的协调

  - 它本质上是一个 **改进版的两阶段提交 (2PC)** 协议



##### 5.1 第一阶段：核心魔法

在第一阶段，Seata 的代理数据源 (`DataSourceProxy`) 会“劫持”你的业务 SQL，在本地事务提交前，悄悄做了一堆事情



**执行流程（以 `UPDATE product SET stock = stock - 1 WHERE id = 1` 为例）：**

1. **解析 SQL (Parse)**：

   - Seata 拦截到 SQL，解析出要更新的表是 `product`，条件是 `id = 1`

     

2. **前置镜像 (Before Image)**：

   - Seata 生成一条查询语句：`SELECT * FROM product WHERE id = 1 FOR UPDATE`
   - 执行查询，保存当前数据（假设 stock=100）。这叫“前置镜像”

   

3. **执行业务 SQL (Execute)**：

   - 执行你的业务 SQL：`stock` 变成了 99

   

4. **后置镜像 (After Image)**：

   - Seata 再次查询：`SELECT * FROM product WHERE id = 1`
   - 保存更新后的数据（stock=99）。这叫“后置镜像”

   

5. **插入回滚日志 (Undo Log)**：

   - Seata 将 前置镜像、后置镜像、SQL 语句信息 打包成一条二进制数据
   - 插入到 `undo_log` 表中

   

6. **提交前卫士：全局锁检查 (Global Lock Check)**：

   - **关键点**：在提交本地事务前，RM 会拿着 `product` 表 `id=1` 这行数据的“主键值”，去 TC 申请 **全局锁**
   - *如果不加锁会怎样？* 别的分布式事务可能会脏写这行数据
   - 如果拿不到锁，RM 会重试；如果一直拿不到，抛异常回滚

   

7. **本地提交 (Local Commit)**：

   - 业务数据的更新 + `undo_log` 的插入，在同一个本地事务中提交
   - **注意**：此时数据库里的数据 **已经变了**（stock=99），且锁已经释放了



##### 5.2 第二阶段：决议与执行

第一阶段结束后，TM 向 TC 发起最终决议。根据第一阶段的汇报情况，TC 会指挥所有 RM 走两条截然不同的路



###### 情况 A：全局提交

如果所有微服务的一阶段都成功了，TM 发起提交请求

1. **指令下达**：TC 向所有 RM 发送 `Branch Commit` 请求

2. **响应**：RM 收到请求后，直接把这个请求放入一个 **异步队列**，然后立马给 TC 返回“成功”

   - *为什么这么快？* 因为一阶段本地事务已经真的提交了，数据已经是 99 了，不需要再做任何数据库操作来“确认”

   

3. **异步清理**：RM 的后台线程会批量消费这个队列，找到对应的 `undo_log` 记录，直接 **删除**

   - *逻辑*：既然事务成功了，就不需要后悔药了，删了省空间

   

**特点**：**异步、极快**。这是 Seata 高性能的核心原因

###### 情况 B：全局回滚

如果任何一个微服务报错了（抛出异常），TM 发起回滚请求

1. **指令下达**：TC 向所有 RM 发送 `Branch Rollback` 请求
2. **查找日志**：RM 通过 XID 和 BranchID，去 `undo_log` 表里找到对应的记录
3. **数据校验 (脏写检查)**：
   - RM 拿出 `undo_log` 里的 `After Image`（后置镜像，记录了当时改成 99 的样子）
   - RM 查询数据库当前的实际值
   - **对比**：如果 `当前值 == After Image`，说明中间没人动过数据，可以安全回滚
4. **反向补偿**：
   - RM 根据 `Before Image`（前置镜像，stock=100）和 `After Image` 自动生成反向 SQL
   - SQL 逻辑类似：`UPDATE product SET stock = 100 WHERE id = 1`
5. **执行回滚**：执行这条反向 SQL，把数据改回去
6. **清理与提交**：删除 `undo_log`，提交本地事务，向 TC 汇报“回滚完毕”

**特点**：**同步、补偿**。



##### 5.3 AT 模式 vs XA 模式

| 特性         | Seata AT (AP 变种)            | Seata XA (传统 CP)           |
| ------------ | ----------------------------- | ---------------------------- |
| **数据库锁** | **短**。一阶段提交后立马释放  | **长**。直到二阶段结束才释放 |
| **并发性能** | **高**。不阻塞其他业务        | **低**。强一致但阻塞         |
| **回滚原理** | **反向 SQL 补偿** (Undo Log)  | **数据库原生回滚**           |
| **一致性**   | **最终一致性** (中间有软状态) | **强一致性** (ACID)          |



#### 6. 核心注解：`@GlobalTransactional`

这是 Seata 最核心的注解，它的作用是 **定义全局事务的边界**。 一旦方法上加了这个注解，当前微服务就变成了 **TM (事务发起者)**



##### 6.1 注解参数速查

请像背诵 `@Transactional` 一样熟悉这些参数，因为生产环境出的问题（如锁超时、不回滚），往往就是参数没配对

**全路径**：`io.seata.spring.annotation.GlobalTransactional`

| 参数名            | 类型          | 默认值      | 作用详解                                                     | 生产建议                                                     |
| ----------------- | ------------- | ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **timeoutMills**  | `int`         | 60000 (60s) | **全局事务超时时间**。<br />如果整个链路（所有微服务调用加起来）超过这个时间，TC 会强制触发全局回滚 | **务必设置**。<br />建议根据业务链路长度设定（如 5000ms）。默认 60s 太长，一旦网络卡顿，会长时间占用全局锁，导致高并发下其他事务排队卡死 |
| **name**          | `String`      | ""          | **全局事务名称**<br />用于在 Seata 控制台或日志中追踪事务。  | **建议格式**：`应用名-业务动作`（如 `order-create-tx`）。当线上出 bug 时，拿着这个名字去日志里搜，效率翻倍 |
| **rollbackFor**   | `Class[]`     | `{}`        | **指定回滚异常**<br />指定哪些异常类触发全局回滚。默认情况下，任何 `Throwable` 都会触发回滚 | 建议显式指定 `Exception.class` 或自定义业务异常基类，防止遗漏。 |
| **noRollbackFor** | `Class[]`     | `{}`        | **指定不回滚异常**<br />即使抛出这些异常，也不回滚           | 用于某些允许失败的非核心业务场景（例如：发送积分通知失败，但不影响下单主流程）。 |
| **propagation**   | `Propagation` | `REQUIRED`  | **传播行为**<br />目前 Seata 仅支持 `REQUIRED`（默认）、`REQUIRES_NEW`、`NOT_SUPPORTED` 等几种 | 绝大多数场景使用默认值即可。不要与 Spring 的传播行为混淆，Seata 的传播主要控制是否开启新的全局 XID。 |



##### 6.2 标准代码示例

这是一个最经典的分布式事务场景：**用户下单**

- **OrderService (TM)**：事务发起者
- **StorageService (RM)**：扣减库存
- **AccountService (RM)**：扣减余额

只要在 `OrderService` 的入口方法加上注解，剩下的一切（XID 传递、分支注册、提交回滚）都由 Seata 自动完成

###### 1. 业务代码实现

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Autowired
    private StorageClient storageClient; // Feign 客户端，远程调用库存
    @Autowired
    private AccountClient accountClient; // Feign 客户端，远程调用账户
    @Autowired
    private OrderMapper orderMapper;     // 本地 DAO，操作订单库

    /**
     * 下单核心接口：开启全局事务
     * * @GlobalTransactional 参数解析：
     * 1. name: "create-order-tx" -> 给这个事务起个名，排查日志用。
     * 2. timeoutMills: 300000 -> 设置超时为 5分钟 (调试期设长一点，生产环境建议 5000ms)。
     * 3. rollbackFor: Exception.class -> 只要抛出 Exception 就回滚 (含 Checked Exception)。
     */
    @Override
    @GlobalTransactional(name = "create-order-tx", timeoutMills = 300000, rollbackFor = Exception.class)
    public void create(Order order) {
        
        LOGGER.info("-----> [1] 开始新建订单");
        // 1. 本地数据库操作 (RM1)
        // 在 undo_log 中记录：插入订单的前后镜像
        orderMapper.create(order);

        LOGGER.info("-----> [2] 订单微服务开始调用库存，做扣减 Count");
        // 2. 远程调用库存服务 (RM2)
        // Seata 拦截器会自动把 XID 放入 Feign 请求头：Header map.put("TX_XID", "xxx")
        storageClient.decrease(order.getProductId(), order.getCount());

        LOGGER.info("-----> [3] 订单微服务开始调用账户，做扣减 Money");
        // 3. 远程调用账户服务 (RM3)
        // 同样传递 XID，账户服务作为 RM3 加入事务
        accountClient.decrease(order.getUserId(), order.getMoney());

        LOGGER.info("-----> [4] 订单结束，准备提交");
        
        // 4. 模拟异常场景：触发全局回滚
        // 如果商品ID是 "ErrorProduct"，手动抛出异常
        if ("ErrorProduct".equals(order.getProductId())) {
             throw new RuntimeException("模拟业务异常，触发 Seata 全局回滚！");
        }
        
        // 如果代码顺利走到这里，没有任何异常抛出
        // TM (Transaction Manager) 会向 TC 发送 GlobalCommit 请求
    }
}
```



###### 2. 下游服务需要加注解吗？(FAQ)

这是一个新手最高频的问题：*“StorageService 和 AccountService 的方法上，需要加 `@GlobalTransactional` 吗？”*

**答案：不需要，也不建议加。**

- **原因**：
  - `@GlobalTransactional` 的作用是 **开启一个新的全局事务**（生成新的 XID）
  - 下游服务是 **参与者**，它们只需要通过 XID 加入已有的事务即可。Seata 的自动配置（`DataSourceProxy`）会自动识别上下文中的 XID 并注册分支
- **例外**：除非下游服务本身也需要作为一个独立的入口被其他系统调用，且需要独立开启事务。但在当前链路中，它们只是 RM



###### 3. 事务传播机制

上述代码执行时，XID 是如何流转的？

1. **OrderService**：`@GlobalTransactional` 拦截 -> 向 TC 申请 XID（例如 `1001`）
2. **OrderService -> StorageService**：Feign 拦截器将 `XID=1001` 放入 HTTP Header
3. **StorageService**：
   - Spring MVC 拦截器从 Header 取出 XID，绑定到本地 `RootContext`
   - 执行 SQL 时，发现 `RootContext` 有 XID，于是向 TC 注册分支事务（属于 `1001`）
4. **异常回滚**：
   - OrderService 抛出异常
   - TM 捕获异常，向 TC 发送 `GlobalRollback(XID=1001)`
   - TC 通知 StorageService 和 AccountService 进行回滚



##### 6.3 避坑指南

Seata 的 `@GlobalTransactional` 基于 Spring AOP 实现，如果你不了解 AOP 的局限性，很容易写出“无效代码”

###### 1. 巨坑一：AOP 动态代理失效

- **现象**：在同一个类中，方法 A（无注解）调用方法 B（有 `@GlobalTransactional`），事务不生效

- **错误代码示例**：

  ```java
  @Service
  public class OrderServiceImpl {
  
      public void createOrder() {
          // 错误！这里相当于 this.doTransaction()
          // 绕过了 Spring 的代理对象，直接调用了原始对象的方法
          // 导致切面逻辑没执行，XID 根本没生成
          doTransaction(); 
      }
  
      @GlobalTransactional // 这个注解完全废了
      public void doTransaction() {
          // ...
      }
  }
  ```

- **原理**：Spring AOP 是基于代理模式的。只有通过代理对象调用方法，拦截器才会生效。`this.xxx()` 是对象内部调用，不走代理

- **解决方案**：

  1. 将带有 `@GlobalTransactional` 的方法抽离到另一个 Service 类中
  2. （不推荐）注入自己 (`@Autowired private OrderService self`)，然后 `self.doTransaction()`



###### 2. 巨坑二：异常被“吃掉”

- **现象**：代码里明明报错了，控制台也打印异常堆栈了，但 Seata 就是不回滚，数据库里产生脏数据

- **错误代码示例**：

  ```java
  @GlobalTransactional
  public void createOrder() {
      try {
          storageClient.decrease(); // 这里抛出了异常
      } catch (Exception e) {
          e.printStackTrace(); // 只是打印了日志，吞掉了异常
          // TM 认为方法正常执行结束，于是发起 Global Commit
      }
  }
  ```

- **原理**：Seata 的 TM（事务管理器）是靠捕获方法抛出的异常来感知失败的。如果你把异常捕获并消化了，TM 会认为“一切安好”，从而发起提交

- **解决方案**：

  - **原则**：要么别 `catch`，要么 `catch` 之后必须 `throw`

  - **修正**：

    ```java
    catch (Exception e) {
        e.printStackTrace();
        throw e; // 必须再次抛出！
    }
    ```

  - 或者使用 `GlobalTransactionContext.reload(RootContext.getXID()).rollback()` 手动回滚（不推荐，代码侵入性高）



###### 3. 巨坑三：全局锁等待超时

- **现象**：高并发场景下，频繁报错 `GlobalLockWaitTimeoutException`，系统吞吐量急剧下降
- **原因**：
  - **热点数据竞争**：多个全局事务同时尝试修改同一行数据（例如秒杀扣减同一商品的库存）
  - **锁持有时间过长**：
    - 你的 `@GlobalTransactional` 方法里执行了耗时的操作（如发邮件、高延迟的 RPC），导致一直占着全局锁不释放，后面的事务拿不到锁只能排队，超时就报错
- **解决方案**：
  - **优化链路**：把非数据库操作（发邮件、计算密集型逻辑）移出事务范围，尽量缩短持有锁的时间
  - **调整参数**：适当增加 `seata.service.client.global-lock-retry-interval`（重试间隔）和 `retries`（重试次数），但这只是治标
  - **架构优化**：对于极致热点（如秒杀），不要直接用数据库+Seata，建议上 Redis



#### 7. AT 模式下的隔离性与全局锁

在 AT 模式中，Seata 为了高性能，做了一个大胆的决定：**一阶段本地事务提交后，直接释放数据库锁**

- 这带来了一个巨大的风险：如果二阶段还没执行，另一个线程（可能是别的全局事务，也可能是普通的本地事务）来修改这行数据，会发生什么？
  - 这就是 **隔离性 (Isolation)** 问题



##### 7.1 写隔离 —— 防止脏写

###### 1. 什么是脏写？

假设我们有两个全局事务：事务 A 和 事务 B，它们都想修改 `id=1` 的余额（初始值 100）

1. **事务 A (XID=1001)**：
   - 执行 `UPDATE balance = 90`
   - **一阶段提交**，释放数据库锁。此时数据库里是 90
2. **事务 B (XID=1002)**：
   - 执行 `UPDATE balance = 80`
   - **一阶段提交**，释放数据库锁。此时数据库里是 80
3. **事务 A 回滚**：
   - 事务 A 发生异常，需要回滚
   - 它拿出自己的 Undo Log（Before Image 是 100）
   - 它执行反向 SQL：`UPDATE balance = 100`
   - **结果**：数据库变成了 100。**事务 B 的更新（80）彻底丢失了！**

这就是脏写。它会导致数据永久性错误



###### 2. Seata 的解决方案：全局锁

为了防止这种情况，Seata 引入了 **全局锁** 机制。这是一把由 TC（服务端）维护的逻辑锁

**核心流程：**

1. **申请锁**：

   - RM 在**一阶段提交本地事务之前**，必须拿到 `table:pk`（表名+主键）的信息
   - RM 去 TC 申请该行数据的全局锁

   

2. **持有与等待**：

   - **如果拿到了**：说明没有人在操作这行数据，允许提交本地事务
   - **如果没拿到**（说明有别的全局事务正在占着）：RM 会进行重试（默认重试 30 次，间隔 10ms）
   - **如果超时**：抛出 `GlobalLockWaitTimeoutException`，回滚本地事务

   

3. **释放锁**：

   - 只有等到 **二阶段（全局提交或全局回滚）彻底完成** 后，TC 才会释放这把锁



**结论**： 通过“提交前抢锁”机制，Seata 保证了：**两个全局事务之间，绝对不会发生脏写。** 因为谁先抢到全局锁，谁才能提交本地事务



##### 7.2 读隔离

在数据库层面，ACID 中的 I (Isolation) 通常指“读已提交”或“可重复读”。 但在 Seata AT 模式中，情况变得特殊

###### 1. 默认状态：读未提交

**现状**：

- 全局事务 A 修改了数据（100 -> 90），并在 **一阶段提交了本地事务**
- 此时，全局事务 A 还没结束（可能在等其他分支，或者正在二阶段处理中）
- 数据库里的数据 **已经是 90 了 **

**脏读风险**：

- 如果此时有一个普通请求（或者另一个全局事务 B）来查询余额
- 它会读到 **90**
- **万一** 全局事务 A 后来回滚了（90 -> 100），那么刚才读到的 90 就是 **脏数据**

**Seata 的选择**：

- **默认情况** 下，Seata 是 **允许脏读** 的
- **理由**：为了极致的性能。绝大多数业务场景（如查看商品详情、用户信息）是可以容忍短暂的数据不一致的



###### 2. 进阶需求：如何实现“读已提交”？

如果你的业务场景（比如资金核算）要求 **必须读到最终提交的数据**，不能读中间状态，怎么办？

Seata 提供了基于 `SELECT FOR UPDATE` 的代理机制

**代码写法**：

```java
@GlobalTransactional // 必须开启全局事务，或者使用 @GlobalLock
public void checkBalance() {
    // 必须使用 FOR UPDATE 语句
    // 只有这种特定的 SQL，Seata 才会去检查全局锁
    Account account = accountMapper.selectByIdForUpdate(1L);
}
```

**底层原理**：

1. **拦截**：Seata 代理数据源拦截到 `SELECT ... FOR UPDATE`
2. **抢锁检查**：RM 会拿着主键去 TC 检查：**“这行数据有没有被加全局锁？”**
   - **有锁**：说明有一个全局事务正在改它（还没结束）。RM 会**等待**，直到锁释放
   - **无锁**：说明没有全局事务在改它，或者已经结束了。RM 执行查询
3. **结果**：通过等待全局锁释放，保证了读出来的数，一定是全局事务提交后的最终结果



**代价**：

- **性能损耗**：查询也要去 TC 查锁，会有网络开销
- **阻塞风险**：如果写事务很慢，读请求也会被卡住



###### 3. 什么是 @GlobalLock 注解？

有时候，我们想查询数据，或者想执行一个不涉及分布式事务的本地更新，但又想 **避开** 正在执行的全局事务（利用全局锁互斥）

这时可以使用 `@GlobalLock`

- **作用**：它 **不开启** 全局事务（不生成 XID，不写 Undo Log），但在执行 SQL 前会去 **申请/检查全局锁**
- **适用场景**：
  - **安全读**：配合 `SELECT FOR UPDATE`，实现高性能的“读已提交”
  - **安全写**：一个普通的本地 update，加上这个注解，就能防止跟正在跑的 Seata 全局事务发生脏写

```java
// 这是一个普通的本地业务方法，不参与全局事务
@GlobalLock(retryInterval = 100, retries = 50)
public void safeUpdate() {
    // 执行前会去检查全局锁。
    // 如果有别的全局事务在改这行数据，我会等它完事了再改。
    // 从而避免了“由于我没加全局锁，导致我覆盖了它的回滚”的问题。
    accountMapper.updateById(...);
}
```



#### 8. TCC 模式原理与接口设计

AT 模式虽然好用（自动挡），但它并不是万能的。在某些特殊场景下，我们必须切换到 TCC 模式（手动挡）

##### 8.1 为什么需要 TCC？(Why)

AT 模式的核心依赖于 **JDBC 数据源代理** 和 **SQL 解析**。它的工作原理是拦截你的 SQL，去数据库查镜像，然后生成 Undo Log

这就注定了 AT 模式在以下三个场景会 **彻底失效**，必须使用 TCC：

###### 1. 非关系型数据库 (NoSQL)

- **场景**：你的业务不是操作 MySQL/Oracle，而是操作 **Redis**、**MongoDB**、**Elasticsearch** 甚至 **HBase**
- **痛点**：AT 模式根本解析不了 Redis 的 `set key value` 命令，也不知道怎么去 Redis 里查“前置镜像”和“后置镜像”，更无法自动生成回滚命令
- **解决**：TCC 允许你手动写 `cancel` 方法。比如 Try 阶段 `set key value`，Cancel 阶段你可以手动写 `del key`

###### 2. 追求极致性能 (High Performance)

- **场景**：大促秒杀、核心高频交易系统

- **痛点**：AT 模式虽然快，但它毕竟有“黑盒”开销：

  1. SQL 解析（CPU 消耗）
  2. 查询前置镜像（一次 DB Select）
  3. 查询后置镜像（一次 DB Select）
  4. 插入 Undo Log（一次 DB Insert）
  5. 申请全局锁（网络交互）

  - 这些额外步骤会导致写性能下降 30%~50%

- **解决**：TCC 没有任何“黑盒”操作。Try 就是一个普通的方法调用，性能完全取决于你的代码写得有多快（比如直接操作内存）



###### 3. 跨系统/API 调用 (Cross-System API)

- **场景**：你的业务逻辑是 **调用第三方接口**
  - 例如：调用支付宝的支付接口、调用物流公司的发货接口、调用遗留系统的 API
- **痛点**：你连人家的数据库都连不上，更别提让 Seata 去代理人家的数据源了
- **解决**：TCC 是唯一解。
  - Try：调用 `pay()` 接口
  - Confirm：不做操作（或调用确认接口）
  - Cancel：调用 `refund()` 退款接口



##### 8.2 TCC 三阶段模型

TCC 是 **Try - Confirm - Cancel** 的缩写。这也是 DTP 模型中两阶段提交（2PC）在业务层的具体实现

你需要为每一个业务功能（比如“扣减余额”）手动编写三个方法



###### 1. 一阶段：Try (尝试/预留)

这是 TCC 最核心的一步。它的职责是：**检测** + **预留**

- **业务检查：

  - 检查前置条件是否满足。例如：账户余额是否充足？商品库存够不够？

- **资源预留：

  - **关键点**：不要直接把钱扣掉，而是把它“冻结”起来
  - 这样做的目的是为了**隔离性**。一旦冻结，别的事务就无法使用这笔钱，但原本的余额看起来减少了（避免了超卖）

- **SQL 示例 (扣减 100 元)**：

  - 假设表结构：`id`, `balance` (可用余额), `frozen` (冻结金额)
  - **逻辑**：余额 -100，冻结 +100

  ```java
  UPDATE account 
  SET balance = balance - 100, 
      frozen = frozen + 100 
  WHERE id = 1 AND balance >= 100; -- 顺便做了余额检查
  ```



###### 2. 二阶段：Confirm (确认/提交)

如果全局事务中所有的 Try 阶段都成功了，TM 会通知执行 Confirm

- **职责**：真正执行业务，不做任何业务检查

- **逻辑**：使用 Try 阶段预留的资源（冻结金额）

- **幂等性要求**：**极高**。因为网络中断时 TC 会重试调用 Confirm，必须保证执行 1 次和执行 10 次结果一样

- **SQL 示例**：

  - **逻辑**：把冻结的那 100 元真正扣掉

  ```sql
  UPDATE account 
  SET frozen = frozen - 100 
  WHERE id = 1;
  ```

  - *注：这里不需要判断 `balance` 了，因为 Try 阶段已经锁定资源了*



###### 3. 二阶段：Cancel (取消/回滚)

如果任何一个 Try 阶段失败了，TM 会通知所有成功的分支执行 Cancel

- **职责**：释放 Try 阶段预留的资源

- **逻辑**：把“冻结”的钱还给“余额”

- **幂等性要求**：**极高**

- **SQL 示例**：

  - **逻辑**：余额 +100，冻结 -100

  ```sql
  UPDATE account 
  SET balance = balance + 100, 
      frozen = frozen - 100 
  WHERE id = 1;
  ```



###### 4. 总结：数据视角的变化

通过 TCC，我们将原本的一个动作拆解了：

| 阶段        | 动作 | 可用余额 (balance) | 冻结金额 (frozen) | 实际总资产                  |
| ----------- | ---- | ------------------ | ----------------- | --------------------------- |
| **初始**    | -    | 1000               | 0                 | 1000                        |
| **Try**     | 挪用 | 900                | 100               | 1000 (钱还在，只是不能用了) |
| **Confirm** | 消耗 | 900                | 0                 | 900 (真正扣款)              |
| **Cancel**  | 归还 | 1000               | 0                 | 1000 (恢复原状)             |



##### 8.3 核心注解与接口定义 (API)

在 Seata TCC 模式中，你不能随意写三个方法。你需要定义一个接口，并通过特定的注解告诉 Seata：“这是一个 TCC 资源，Try 是谁，Confirm 是谁，Cancel 是谁”

###### 1. 核心注解清单

- **`@LocalTCC`**

  - **位置**：加在 **接口**上（推荐）或实现类上
  - **作用**：告诉 Seata，这个类是一个 TCC 参与者，需要被代理拦截

  

- **`@TwoPhaseBusinessAction`**

  - **位置**：加在 **Try 方法** 上

  - **作用**：定义 TCC 的元数据

  - **核心属性**：

    - `name`: **全局唯一的 TCC Bean 名称**。千万不能重复，TC 靠它来定位资源
    - `commitMethod`: 指定 Confirm 方法的方法名（字符串）
    - `rollbackMethod`: 指定 Cancel 方法的方法名（字符串）

    

- **`BusinessActionContext`**

  - **位置**：作为 Confirm 和 Cancel 方法的 **第一个参数**
  - **作用**：上下文传递。因为 Confirm/Cancel 是由 TC 异步调用的，它们无法直接获取 Try 方法的入参

  

- **`@BusinessActionContextParameter`**

  - **位置**：加在 **Try 方法的入参** 上
  - **作用**：将该参数放入上下文（Context）中，以便在二阶段（Confirm/Cancel）中通过 `actionContext.getActionContext("paramName")` 取出来



###### 2. 标准接口定义示例

假设我们要实现一个“扣减库存”的 TCC 接口

```java
import io.seata.rm.tcc.api.BusinessActionContext;
import io.seata.rm.tcc.api.BusinessActionContextParameter;
import io.seata.rm.tcc.api.LocalTCC;
import io.seata.rm.tcc.api.TwoPhaseBusinessAction;

@LocalTCC // 1. 标记这是一个 TCC 接口
public interface StorageTccService {

    /**
     * 一阶段方法：Try
     * * @param productId 商品ID
     * @param count     扣减数量
     * @return boolean  是否成功
     */
    @TwoPhaseBusinessAction(
        name = "StorageTccAction", // 必须全局唯一！
        commitMethod = "commit",   // 指向下面的 commit 方法
        rollbackMethod = "rollback" // 指向下面的 rollback 方法
    )
    boolean prepare(BusinessActionContext actionContext,
                    @BusinessActionContextParameter(paramName = "productId") Long productId,
                    @BusinessActionContextParameter(paramName = "count") int count);

    /**
     * 二阶段方法：Confirm (确认)
     * * @param actionContext 上下文，可以从中取出 Try 阶段传进来的 productId 和 count
     * @return boolean
     */
    boolean commit(BusinessActionContext actionContext);

    /**
     * 二阶段方法：Cancel (取消)
     * * @param actionContext 上下文
     * @return boolean
     */
    boolean rollback(BusinessActionContext actionContext);
}
```



###### 3. 参数传递的奥秘

初学者最容易懵的是：*“Confirm 方法里怎么知道要扣哪个商品的库存？”*

- **Try 阶段**： 

  - 当你调用 `prepare(ctx, 1001L, 5)` 时，Seata 会看到 `@BusinessActionContextParameter` 注解，

    于是把 `productId=1001L` 和 `count=5` 存入 `actionContext`（本质是一个 Map），并持久化到 TC

- **Confirm/Cancel 阶段**： TC 调用 `commit` 方法时，会将这个 `actionContext` 还原并传回来。 你只需要：

  ```java
  Long productId = ((Number) actionContext.getActionContext("productId")).longValue();
  int count = ((Number) actionContext.getActionContext("count")).intValue();
  ```



#### 9. TCC 三大异常控制

TCC 模式最大的难点不在于写业务逻辑，而在于 **处理 RPC 带来的网络混沌**

- TC（服务端）是通过 RPC 远程调用你的 Try、Confirm、Cancel 方法的，所有的异常都源于 **网络延迟** 或 **丢包**

##### 9.1 异常现象复现与危害

###### 1. 异常一：空回滚

- **现象**：TC 调用了 `Cancel` 方法，但是 `Try` 方法根本没执行（或者没执行成功）
- **场景复现**：
  1. TM 请求 TC 开启全局事务
  2. TM 调用分支服务 A 的 Try 接口
  3. **异常**：RPC 网络中断，Try 请求没发出去；或者 Try 方法刚进去就抛异常了
  4. TM 感知到失败，向 TC 发起全局回滚
  5. TC 命令所有分支执行 `Cancel`
  6. **结果**：分支 A 收到了 Cancel 指令，但它从来没收到过 Try。这就叫“空回滚”
- **危害**：如果不做判断直接执行 Cancel（比如 `UPDATE balance = balance + 100`），就会**凭空给用户加钱**



###### 2. 异常二：幂等控制

- **现象**：TC 重复调用了 `Confirm` 或 `Cancel` 方法
- **场景复现**：
  1. TC 发送 Confirm 指令
  2. 分支 A 执行成功，返回结果
  3. **异常**：结果返回时网络丢包，TC 没收到响应
  4. TC 认为超时，触发 **重试机制**，再次发送 Confirm 指令
- **危害**：如果 Confirm 是扣款操作，重复调用会导致 **用户被扣两次钱**



###### 3. 异常三：悬挂

这是最烧脑的一个，也是空回滚的“变种”，非常容易被忽略

- **现象**：`Cancel` 比 `Try` 先执行
- **场景复现**：
  1. TM 调用 Try，**网络拥堵**，请求卡在路上（比如 GC 停顿了 10秒）
  2. TM 等不到响应，认为超时失败，向 TC 发起全局回滚
  3. TC 调用 Cancel。分支服务执行 Cancel（此时由于还没 Try，逻辑上判定为空回滚，返回成功）
  4. **异常**：这时候，那个卡在路上的 Try 请求终于到了！
  5. 分支服务执行 Try，预留了资源（冻结了钱）
  6. **结果**：全局事务在 TC 那边已经结束了（回滚了），但分支服务这里却刚刚冻结了一笔钱。这笔钱永远不会被 Confirm 或 Cancel，像吊死鬼一样挂在那里
- **危害**：资源被**永久冻结**，导致资金泄露（钱没丢，但拿不出来用了）



##### 9.2 终极解决方案：TCC Fence 机制

要同时解决空回滚、幂等、悬挂这三个问题，最稳妥的办法是利用 **数据库的唯一性约束** 和 **状态机**。Seata 将这种机制命名为 **TCC Fence (围栏)**

###### 1. 物理基础：tcc_fence_log 表

首先，你需要在**业务数据库**中创建一张特殊的表。Seata 会利用这张表来记录每个分支事务的状态

```sql
-- Seata TCC Fence 专用表 (MySQL)
CREATE TABLE `tcc_fence_log` (
    `xid`           VARCHAR(128)  NOT NULL COMMENT '全局事务ID',
    `branch_id`     BIGINT(20)    NOT NULL COMMENT '分支事务ID',
    `action_name`   VARCHAR(64)   NOT NULL COMMENT 'TCC Bean Name',
    `status`        TINYINT(4)    NOT NULL COMMENT '状态: 1=Tried, 2=Committed, 3=RolledBack, 4=Suspended',
    `gmt_create`    DATETIME(3)   NOT NULL COMMENT '创建时间',
    `gmt_modified`  DATETIME(3)   NOT NULL COMMENT '修改时间',
    PRIMARY KEY (`xid`, `branch_id`),
    KEY `idx_gmt_modified` (`gmt_modified`),
    KEY `idx_status` (`status`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;
```



###### 2. 自动化开启

在 Seata 1.5.0+ 版本中，你不需要自己写 `if-else` 去查这张表。只需要在注解上开启一个开关：

```java
@LocalTCC
public interface StorageTccService {

    @TwoPhaseBusinessAction(
        name = "StorageTccAction", 
        commitMethod = "commit", 
        rollbackMethod = "rollback",
        useTccFence = true  // <--- 开启 TCC Fence 机制！
    )
    boolean prepare(BusinessActionContext actionContext, ...);
}
```

开启后，Seata 代理对象会在调用你的 Try/Confirm/Cancel 方法 **之前** 和 **之后**，自动执行以下的防御逻辑





###### 3. 底层原理

即使 Seata 帮我们做了，作为架构师也必须理解它到底干了什么：

**A. 防悬挂 (Anti-Suspension) —— Try 阶段**

- **逻辑**：在执行 Try 业务 SQL 之前，先开启本地事务，插入一条记录 `status = TRIED`

- **防御**：

  - 如果插入成功 -> 正常执行业务

  - 如果插入失败（主键冲突） -> 检查已存在的记录

    - 如果发现状态是 `ROLLED_BACK` 或 `SUSPENDED`，说明 Cancel 已经执行过了（悬挂发生了）
    - **动作**：直接抛出异常，阻止 Try 方法执行资源预留

    

**B. 防幂等 (Idempotency) —— Confirm/Cancel 阶段**

- **逻辑**：在执行 Confirm/Cancel 之前，先查询记录

- **防御**：

  - 如果状态已经是 `COMMITTED`（在 Confirm 时）或 `ROLLED_BACK`（在 Cancel 时）
  - **动作**：直接返回成功，不再执行业务逻辑

  

**C. 防空回滚 (Anti-Null Rollback) —— Cancel 阶段**

- **逻辑**：在执行 Cancel 之前，通过 `xid` + `branch_id` 查询记录
- **防御**：
  - 如果记录**不存在**（说明 Try 没执行成功）
  - **动作**：插入一条 `status = SUSPENDED` 的记录（为了防止后续 Try 又来了导致悬挂），然后直接返回成功



###### 4. 总结

TCC Fence 机制用一张表实现了完美的逻辑闭环：

| 场景              | Fence 的处理动作                  | 结果                    |
| ----------------- | --------------------------------- | ----------------------- |
| **正常 Try**      | 插入 `TRIED` 记录                 | 成功                    |
| **重复 Confirm**  | 查到 `COMMITTED` 记录             | 直接返回成功 (幂等)     |
| **空回滚 Cancel** | 查不到记录 -> 插入 `SUSPENDED`    | 直接返回成功 (防空回滚) |
| **悬挂 Try**      | 查到 `SUSPENDED` 记录 -> 插入失败 | 抛异常 (防悬挂)         |



#### 10. XA 与 Saga 模式

在 Seata 的版图中，AT 和 TCC 占据了 95% 的互联网应用场景。但在金融、账务等对 **强一致性** 要求极高的领域，XA 模式依然是不可替代的



##### 10.1 XA 模式

###### 1. 什么是 XA ?

XA 是 X/Open 组织定义的一套 **分布式事务标准**

- **核心特征**：它利用 **数据库本身**（MySQL, Oracle）提供的 2PC（两阶段提交）机制，而不是像 AT 模式那样在应用层模拟 2PC
- **角色**：Seata 的 RM 此时不再是一个复杂的代理，它只是数据库 XA 接口的“搬运工”



###### 2. XA vs AT

很多同学容易混淆 XA 和 AT，因为它们都是 **2PC**（两阶段），且**代码都无侵入**（都用 `@GlobalTransactional`）

但它们的底层逻辑完全相反：

| 特性         | AT 模式 (默认)                                               | XA 模式                                                      |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **锁机制**   | **应用层逻辑锁** (全局锁)。数据库本地锁在一阶段提交后 **立即释放** | **数据库物理锁**。数据库本地锁一直 **持有**，直到二阶段结束才释放 |
| **隔离性**   | **读未提交** (默认)。中间状态对外部可见                      | **序列化/强一致**。中间状态对外不可见 (阻塞)                 |
| **并发性能** | **高**。锁持有时间短                                         | **低**。长事务会阻塞数据库连接，导致 TPS 骤降                |
| **回滚方式** | **补偿** (Undo Log 反向 SQL)                                 | **原生 Rollback** (数据库机制)                               |



###### 3. 如何切换到 XA 模式？

Seata 的设计非常优雅，从 AT 切换到 XA，**不需要改哪怕一行 Java 代码**，只需要改配置

**步骤一：修改数据源代理模式** 在 `application.yml` 中：

```yaml
seata:
  data-source-proxy-mode: XA  # 默认是 AT，改为 XA 即可
```

**步骤二：代码保持不变** 依然使用 `@GlobalTransactional` 注解

```java
@GlobalTransactional // 此时它开启的就是 XA 分布式事务
public void transfer() {
    // 数据库操作...
}
```

###### 4. 适用场景与避坑

- **适用场景**：
  - **金融核心**：如银行转账、清结算系统
  - **旧系统迁移**：原本就是基于 XA 协议的旧系统，迁移到 Spring Cloud 架构
  - **并发量低**：TPS 要求不高，但数据绝对不能错
- **避坑指南**：
  - **性能陷阱**：千万不要在电商下单等高并发场景用 XA，数据库连接池会被瞬间打满（因为锁一直不释放）
  - **数据库支持**：必须确保你使用的数据库（如 MySQL 5.7+）和驱动程序支持 XA 协议



##### 10.2 Saga 模式：长事务解决方案

Saga 模式源于 1987 年的一篇学术论文，它的核心思想非常简单粗暴：**“一往无前，错了再退”**



###### 1. 核心机制

Saga 模式把一个长事务拆分成多个 **本地事务**（T1, T2, T3...）。 每个本地事务都有一个对应的 **补偿事务**（C1, C2, C3...）

- **正向执行**：T1 -> T2 -> T3。如果都成功，事务结束
- **失败回滚**：如果 T1 成功，T2 成功，但 **T3 失败** 了。Seata 会自动执行 C2 -> C1 来消除影响

**Saga vs TCC 的区别（核心）：**

- **TCC**：有 `Try` 阶段（预留资源）。先冻结，再扣款
- **Saga**：**没有 `Try` 阶段**。直接扣款！
  - T1 (扣款) -> C1 (退款)
  - 因为没有预留动作，所以 Saga 对业务的侵入性比 TCC 低（不需要改数据库表结构加冻结字段），但隔离性也更差



###### 2. 实现方式：状态机引擎

在 Seata 中，Saga 模式不需要写 Java 代码来编排流程，而是通过 **JSON 文件** 定义一个状态机

**示例 JSON (State Diagram)：**

```js
{
    "Name": "buyGoods",
    "StartState": "ReduceInventory",
    "States": {
        "ReduceInventory": {
            "Type": "ServiceTask",
            "ServiceName": "inventoryService",
            "ServiceMethod": "reduce",
            "CompensateState": "CompensateReduceInventory", // 指定回滚方法
            "Next": "ReduceBalance"
        },
        "ReduceBalance": {
            "Type": "ServiceTask",
            "ServiceName": "balanceService",
            "ServiceMethod": "reduce",
            "CompensateState": "CompensateReduceBalance",
            "Next": "Succeed"
        },
        "CompensateReduceInventory": {
            "Type": "ServiceTask",
            "ServiceName": "inventoryService",
            "ServiceMethod": "compensateReduce"
        }
        // ... 其他补偿节点
    }
}
```



###### 3. 优缺点与适用场景

| 特性         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| **优点**     | 1. **高并发**：完全无锁。T1 提交了就是真提交了，不会锁表<br />2. **长流程友好**：适合那种耗时几分钟甚至几天的业务 |
| **缺点**     | **隔离性极差**<br />因为 T1 提交后，其他事务立马能读到修改后的数据。如果 T1 后来回滚了，其他事务读到的就是脏数据。必须在业务层处理“脏读”后果 |
| **适用场景** | 1. **老系统改造**：无法改造数据库接口，只能调 API<br />2. **长事务**：跨行转账、复杂的审批流、旅游OTA（订机票+订酒店+买门票） |



#### 11. 生产环境最佳实践

代码写得再好，如果服务端挂了，一切都是白搭。 Seata Server (TC) 作为分布式事务的协调核心，一旦单点故障，所有涉及分布式事务的业务都会阻塞甚至回滚

##### 11.1 Seata 高可用部署方案

在生产环境中，**绝对禁止 **使用 `file` 模式部署单节点 TC。你必须部署 TC 集群



###### 1. 核心架构原理

Seata 的高可用依赖于两个组件：

- **共享存储**：所有的 TC 节点都连接 **同一个数据库**。这样，即使 TC-1 挂了，TC-2 也能从数据库里读到未完成的事务状态，继续处理
- **注册中心**：所有的 TC 节点都注册到 **同一个 Nacos**。客户端（微服务）通过 Nacos 的服务发现机制，自动实现负载均衡和故障转移



###### 2. 部署步骤

**Step 1: 准备共享数据库**

- 确保你的 MySQL 本身是高可用的（如主从复制、MGR）
- 初始化 `seata` 库（`global_table`, `branch_table`, `lock_table`）



**Step 2: 修改 TC 配置 (registry.conf)**

- 所有 TC 节点的配置必须一致
- `store.mode = "db"`
- `registry.type = "nacos"`



**Step 3: 启动多个 TC 实例**

- 假设你有三台机器（或 Docker 容器）：
  - `192.168.1.101:8091`
  - `192.168.1.102:8091`
  - `192.168.1.103:8091`
- 启动命令：确保它们都注册到同一个 Nacos Namespace 和 Group



**Step 4: 客户端验证**

- 在 Nacos 控制台 -> 服务列表
- 你应该看到 `seata-server` 服务的 **实例数** 变成了 **3**
- 微服务启动时，Seata Client 会自动轮询这三个 IP



###### 3. 异地多活与集群分组

如果你的业务跨机房（北京、上海），为了减少网络延迟，你需要使用 **TC 集群分组**

- **北京机房**：部署 TC 集群 `beijing-cluster`
- **上海机房**：部署 TC 集群 `shanghai-cluster`
- **微服务配置**：
  - 北京的微服务：`service.vgroupMapping.my_tx_group = beijing-cluster`
  - 上海的微服务：`service.vgroupMapping.my_tx_group = shanghai-cluster`

这样，流量就被物理隔离了，实现了 **就近访问**



##### 11.2 性能调优：让 Seata 飞起来

Seata AT 模式虽然高性能，但相比于本地事务，它依然增加了网络 RTT（往返时延）和数据库 IO。 以下是生产环境中必须关注的优化点

###### 1. 序列化优化

Seata 需要将 Undo Log（包含数据镜像）序列化后存入数据库

- **默认配置**：`jackson`。优点是可读性好（JSON），缺点是体积大、CPU 消耗高
- **优化方案**：生产环境建议切换为 **kryo** 或 **protobuf**
  - **体积对比**：Kryo 只有 Jackson 的 1/10 左右
  - **性能对比**：序列化速度快 5-10 倍



**配置方法 (application.yml)**：

```yaml
seata:
  client:
    undo:
      log-serialization: kryo # 推荐使用 kryo
```

*(注意：服务端和客户端都需要引入对应的 kryo 依赖 jar 包)*





###### 2. Undo Log 表优化

`undo_log` 表是 IO 的热点

- **索引优化**：
  - 确保 `branch_id` 和 `xid` 上有联合唯一索引 `ux_undo_log`。Seata 二阶段回滚或删除日志全靠它
- **表分区 (Partitioning)**：
  - 如果 TPS 极高，`undo_log` 表会瞬间膨胀。建议根据时间或 ID 对该表进行数据库表分区
- **清理策略**：
  - 正常情况下，Seata 会异步自动删除已提交的 Undo Log
  - **异常兜底**：如果发现表数据持续堆积（百万级），说明有事务卡死或清理线程失效。需要监控该表的行数，必要时配置 `log_status=1` 的自动清理脚本



###### 3. 全局锁重试策略

在热点数据（如秒杀库存）场景下，获取全局锁是最大的瓶颈

- **默认配置**：
  - `retry-interval`: 10ms (重试间隔)
  - `retry-times`: 30 (重试次数)
- **优化思路**：
  - 如果你的业务持有锁的时间较长（比如 SQL 慢），应该 **增大间隔**，减少无意义的频繁网络请求
  - **建议**：`retry-interval` 调整为 20-50ms，`retry-times` 调整为 20

```yaml
seata:
  client:
    rm:
      lock:
        retry-interval: 20
        retry-times: 20
```



###### 4. 网络压缩

TC 和 RM 之间有大量的交互（尤其是 Undo Log 数据传输）。启用压缩可以显著降低带宽压力

**配置方法**：

```yaml
seata:
  transport:
    compressor: gzip # 开启 GZIP 压缩
```



##### 11.3 常见错误排查与故障处理

在使用 Seata 的过程中，报错信息通常比较隐晦。以下是 **Top 3** 高频故障及其解决方案

###### 1. 榜首恶魔：`no available service 'null' found`

- **现象**：微服务启动或调用时报错，提示找不到服务
- **根本原因**：**事务分组映射（Mapping）配置错误**。客户端拿着“事务分组名”去配置中心查，结果查了个寂寞，或者查到了集群名但注册中心里没有对应的 TC 实例
- **排查链路**（按顺序检查）：
  1. **检查 Client 配置**：`application.yml` 里的 `seata.tx-service-group` 值是什么？（假设是 `my_group`）
  2. **检查 Nacos 配置**：去 Nacos 配置列表里搜，有没有一个 Key 叫 `service.vgroupMapping.my_group`？
  3. **检查映射值**：这个 Key 的值（假设是 `default`），是不是 Seata Server 启动时注册的集群名？
  4. **检查 TC 实例**：去 Nacos 服务列表，看 `seata-server` 服务下的实例，其 `cluster` 元数据是不是 `default`？



###### 2. 性能杀手：`GlobalLockWaitTimeoutException`

- **现象**：系统吞吐量骤降，日志疯狂刷屏 `Could not get global lock within 5000ms`
- **根本原因**：**热点数据竞争**。多个全局事务（或加了 `@GlobalLock` 的本地事务）都在抢同一行记录的全局锁
- **解决方案**：
  1. **治标**：调大 `retry-times`（重试次数）和 `retry-interval`（重试间隔），给等待多一点时间
  2. **治本**：优化业务 SQL
     - 检查 SQL 是否没有走索引？导致锁住了整个表（Gap Lock）？
     - 检查事务内是否包含了 HTTP 请求等耗时操作？（锁持有时间 = SQL 执行时间 + 其它业务耗时）。**务必把耗时操作移出事务外！**



###### 3. 诡异回滚：`UndoLogDeleteException` 或 回滚失败

- **现象**：二阶段回滚时报错，提示无法删除 Undo Log，或者反向补偿失败
- **根本原因**：**脏数据**。
  - 有人（DBA 或其他未接入 Seata 的服务）绕过 Seata 直接修改了数据库，导致 `After Image` 和当前数据库值对不上
- **解决方案**：
  - **人工介入**：Seata 为了保护数据，一旦发现数据不一致，会停止回滚并报警。你需要去数据库对比数据，手动修复，然后删除该条异常的 Undo Log。
  - **规范**：确保所有涉及该表的写操作，都必须通过 Seata 体系（加 `@GlobalTransactional` 或 `@GlobalLock`）



## 消息队列

mq







Dubbo？