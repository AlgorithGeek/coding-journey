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

Sentinel 





## 分布式事务

seata



## 消息队列

mq







Dubbo？