# Rabbit MQ

## 一：核心概念与架构

在写下第一行代码之前，我们必须先从“上帝视角”理解它的运行机制。

- RabbitMQ 是一套基于 **AMQP** 协议构建的复杂路由系统

### 1.1 为什么需要消息中间件

在分布式系统架构中，引入 RabbitMQ 这样重量级的中间件是有代价的。我们必须明确它到底解决了什么核心痛点，才能在技术选型时做出正确判断

- 通常，我们引入 MQ 主要为了解决三个核心问题：**异步处理**、**应用解耦**、**流量削峰**



#### 1. 异步处理

**痛点：** 

- 传统的同步调用会导致响应时间线性叠加

  如果一个核心业务操作需要调用多个下游服务（如发邮件、发短信、入积分），用户必须等待所有操作完成后才能收到响应，体验极差



**场景示例：用户注册** 假设用户注册涉及三个步骤：

1. 写入用户数据库 (50ms)
2. 调用邮件服务发送激活码 (100ms)
3. 调用短信服务发送通知 (100ms)

- **同步模式（串行）：** 总耗时 = 50 + 100 + 100 = **250ms**

- **同步模式（并行/多线程）：** 虽然可以并发，但仍占用线程资源，且需等待最慢的服务返回

- **MQ 异步模式：**

  1. 写入用户数据库 (50ms)
  2. 写入消息队列 (5ms) -> **立即返回给用户**
  3. 邮件服务和短信服务作为消费者，异步从队列读取消息并处理

  - **用户感知耗时：** 约 **55ms**

**价值：** 显著提升系统的吞吐量和用户响应速度



#### 2. 应用解耦

**痛点：** 

- 在微服务架构中，服务之间如果通过 HTTP/RPC 直接调用，会产生强耦合

  如果下游服务宕机或接口变更，上游服务会直接报错或阻塞，导致级联故障



**场景示例：订单系统 -> 库存系统**

- **强耦合：** 用户下单，订单系统直接调用库存系统的 RPC 接口扣减库存

  - *风险：* 如果库存系统升级或宕机，用户无法下单，订单系统崩溃

  

- **MQ 解耦：**

  1. 订单系统：用户下单 -> 写入数据库 -> **发布“订单已创建”消息到 MQ** -> 返回下单成功
  2. 库存系统：订阅该消息 -> 收到消息后执行扣减库存逻辑

  - *优势：* 即使库存系统宕机，订单系统依然可以正常接单。消息会堆积在 MQ 中，待库存系统恢复后，继续消费处理，数据不丢失

**价值：** 提高系统的容错性和可维护性，实现“由于下游不可用导致的非核心业务损失最小化”



#### 3. 流量削峰

**痛点：** 

- 互联网业务中常有突发流量场景（如秒杀、抢购）

  瞬间的高并发请求（如 10,000 QPS）如果直接打到数据库（MySQL 通常只能抗 2,000 QPS 左右），会导致数据库直接挂掉，系统全线崩溃



**场景示例：秒杀活动**

- **无保护：** 10,000 个请求同时涌入 -> DB 宕机
- **MQ 削峰：**
  1. **网关/上游：** 接收 10,000 个请求，全部快速写入 MQ（RabbitMQ 单机吞吐量远高于 DB）
  2. **消费者（下游）：** 根据数据库的实际承受能力（如 1,000 QPS），**定速** 从 MQ 中拉取消息进行处理
  3. **结果：** 此时 MQ 充当了一个巨大的“缓冲区”或“漏斗”。虽然会有大量消息积压，但数据库始终以安全的速度运行

**价值：** 保护后端核心资源不被流量洪峰击垮，以空间（消息积压）换时间



#### 4. 生产避坑指南

引入 MQ 并不是“银弹”，它会带来显著的架构复杂度和风险。在决定使用 RabbitMQ 前，必须清楚以下代价：

1. **系统可用性降低：**
   - 原本是 A -> B，现在是 A -> MQ -> B
   - 如果 MQ 挂了，整个链路就断了。因此，**RabbitMQ 的高可用（HA）集群搭建是生产环境的必选项**（后续章节会讲镜像队列）
2. **系统复杂度大幅提升：**
   - **消息丢失：** 网络抖动导致消息没发出去？消费者处理失败消息丢了？（需要可靠性投递机制）
   - **消息重复：** 网络超时导致重发，A 给 B 转账两次？（需要幂等性设计）
   - **顺序性：** 怎么保证“先创建订单”比“扣减库存”先执行？（需要顺序消费方案）
3. **一致性问题：**
   - A 系统处理完了，B 系统消费失败怎么办？这导致了分布式事务问题（最终一致性）

**总结：** 只有当你的业务确实面临上述三个痛点（异步、解耦、削峰），且你准备好处理上述风险时，才引入 RabbitMQ。不要为了用技术而用技术



### 1.2 RabbitMQ 核心架构

RabbitMQ 是 **AMQP (Advanced Message Queuing Protocol)** 协议的一个开源实现。要理解 RabbitMQ，本质上就是要理解 AMQP 的模型

请务必打破一个传统的思维误区：

- **“生产者直接把消息发给队列，消费者从队列取消息”** **❌ 错！** 
  - 在 RabbitMQ 的设计中，生产者（Producer）**从不** 直接将消息发送到队列（Queue）



#### 1. 核心组件

我们将 RabbitMQ 内部拆解为以下几个关键组件，务必记清楚它们的职责：

##### 1. Virtual Host (虚拟主机)

- **核心概念**：
  - **类比理解**：它之于 RabbitMQ，就像 **Database** 之于 **MySQL**
  - RabbitMQ 本身是一个 Server，而 VHost 是 Server 内部划分出的一个个“迷你 RabbitMQ”
  - 每个 VHost 内部拥有自己独立的 **Exchange**、**Queue**、**Binding** 和 **权限机制**
- **作用：** 
  - **解决命名冲突**：
    - 假设电商组和物流组共用同一个 RabbitMQ 集群
    - 如果没有 VHost，他们如果都想创建一个叫 `order_status` 的队列，就会报错
    - **有了 VHost**：电商组用 `/shop`，物流组用 `/logistics`。哪怕队列重名，也互不干扰，完全隔离
  - **权限控制的边界**：
    - 你可以创建一个用户 `user_shop`，只给他访问 `/shop` 的权限。这样他就绝对无法误删物流组的队列，保障了数据安全
- **生产避坑**：
  - **严禁裸奔**：生产环境 **绝对不要** 使用默认的 `/` (root) 路径存放业务数据
  - **环境隔离**：建议按照环境（如 `/dev`, `/test`, `/prod`）或者按照业务域（如 `/payment`, `/order`）来创建独立的 VHost





##### 2. Exchange —— 核心路由器

- **角色定位：** 核心路由器 (The Router)

  - **1. 概念**：它是消息进入消息中间件后的 **第一个接收站**

    - *注意：在 AMQP 架构中，生产者直接对接 Exchange，而非直接对接 Queue*

      

  - **2. 核心职责**

    - **接收**：接收生产者发送的消息
    - **路由**：根据消息携带的 **路由键 (Routing Key)** 和预定义的 **绑定规则 (Binding)**，通过算法决定将消息投递给哪一个队列，或者同时广播给多个队列

    

  - **3. 关键特性 —— 生产级避坑**

    - **Exchange 不存储消息！**
    - Exchange 本质上是无状态的路由逻辑组件
    - **风险警告**：如果一条消息被发送到了一个 **没有任何队列绑定** 的 Exchange，这条消息将会直接 **丢失**，且默认不会报错
    - *解决方案：生产环境中通常需配置备份交换机或开启消息回退来感知此类路由失败*



##### 3. Queue —— 消息缓冲区

- **角色定位：** 消息最终存储容器

  - **1. 概念** 它是 AMQP 模型中 **唯一** 实际存储消息的数据容器

    - *对比：Exchange 只是负责路由（路标），Queue 才是负责落地存储（仓库）*

    

  - **2. 核心职责**

    - **缓冲**：保存消息，直到消费者连接并准备好接收它
    - **削峰填谷**：作为系统的“蓄水池”，在下游服务处理不过来时，承担消息堆积的压力，保护后端服务

    

  - **3. 关键特性**

    - **队列必须要和交换机绑定**，如果没有绑定，**没有任何消息能进到这个队列里**。队列就会变成一座“孤岛”

      - 你可能会疑惑，你疑惑的原因是因为 RabbitMQ 有一个**“隐形自动绑定”**机制
  
        当你创建一个队列（比如叫 `work.queue`）但 **不手动绑定** 任何交换机时：

        - RabbitMQ 会 **自动** 把它绑定到 **默认交换机 (Default Exchange)** 上
        - **绑定规则**：Routing Key = 队列名字
  
      
  
    - **竞争消费者模式**
  
      - **机制**：如果多个消费者监听 **同一个** 队列，RabbitMQ 默认采用 **轮询** 策略分发消息
      - **结果**：同一条消息 **只会被其中一个** 消费者处理，**绝对不会** 被广播给该队列的所有消费者（除非使用了不同的队列）
      - *应用场景：这是实现服务横向扩展和负载均衡的基础*
  
      
  
    - **持久化机制**
  
      - 消息是存在内存中（速度快但易丢）还是磁盘中（速度慢但安全），取决于以下配置：
        1. **Queue Durable**: 队列元数据是否持久化
        2. **Message DeliveryMode**: 消息内容是否标记为持久化
      - *生产建议：核心业务必须双重持久化，否则重启 MQ 服务会导致数据丢失*



##### 4. Binding —— 路由规则表

> 逻辑上它属于 Exchange 的“路由表”，一个 **Binding** 就是这个路由表中的一条数据
>
> - 它定义了三要素 —— **“源头（Exchange）”** + **“条件（Binding Key）”** + **“去向（目标 Queue）”**
>
> - 唯一作用就是 **把 Exchange和 Queue 关联起来**，并设定 **过滤规则**
>   - 如果后续 **生产者发送的消息** 中的 **Routing Key 匹配这个过滤规则** ，就 **转发一份该消息** 到 **关联起来的这个 Queue**
>   - 如果后续 **生产者发送的消息** 中的 **Routing Key 没有找到匹配的过滤规则**，就直接 **丢掉该消息** 

- **角色定位：** 路由映射关系表

  - **1. 概念**： 它是连接 **Exchange** (交换机) 与 **Queue** (队列) 的 **逻辑桥梁**
    - *本质：它定义了 RabbitMQ 内部的“网络拓扑结构”。如果没有绑定，Exchange 就像一个拔了网线的路由器，收到消息后无法转发*

  

  - **2. 核心职责**

    - **规则定义**：明确告诉 Exchange：“当收到 Routing Key 符合 **路由表中的 Binding 的 Binding Key** 的消息时，请复制一份放入 **该 Queue**”
    - **多对多映射**：
      - 一个 Exchange 可以绑定多个 Queue（实现广播/多播）
      - 一个 Queue 也可以被多个 Exchange 绑定（接收多来源数据）

    

  - **3. 核心难点**

    - **Routing Key vs Binding Key 的区别**

      - **Routing Key (动态指令)**：

        - 来源：由 **生产者** 在发送每一条消息时指定
        - 性质：动态的字符串，例如 `order.create` 或 `user.login`

        

      - **Binding Key (静态规则)**：

        - 来源：在 **绑定** 队列到交换机时指定
        
        - 性质：静态的配置参数，可能是精确匹配（如 `order.create`）也可能是通配符匹配（如 `order.#`）
        
        - 注意：在这个管理的 UI 界面，交换机这里的 Routing Key ，其实就是 Binding Key ，它这个名字写的有问题
        
          ![image-20251214211119260](./assets/image-20251214211119260.png)



##### 5. Routing Key —— 地址

> 它是 **生产者 **写的: 代表这条消息的“具体内容”或“去向指令”

- **角色定位：** 动态分发指令

  - **1. 概念**： 它是 **生产者** 在 **发送消息时** 附带的一个字符串标识

    - *本质：就像快递包裹上填写的“具体收件地址”。它是消息“头部信息”的一部分*

    

  - **2. 核心作用**

    - **路由依据**：它是 Exchange 进行路由计算的 **输入变量**
    - **匹配过程**：Exchange 接收到消息后，会提取这个 Routing Key，将其与所有绑定的 **Binding Key** 进行比对
      - *完全匹配 (Direct)*：RoutingKey 必须等于 BindingKey
      - *模式匹配 (Topic)*：RoutingKey 符合 BindingKey 的通配符规则（如 `order.*`）

    

  - **3. 生产最佳实践**

    - **层级命名法**

      - 强烈建议使用 **点号 (.)** 分隔的字符串格式
      - **推荐模版**：`<业务域>.<对象>.<动作>`
      - **示例**：
        - `order.payment.success` (订单支付成功)
        - `user.login.failed` (用户登录失败)
        - `stock.item.deducted` (库存扣减)

      

    - **设计哲学**

      - Routing Key 应该代表消息的 **本质特征** 或 **业务意图**
      - 设计良好的 Routing Key 可以让下游消费者灵活地选择订阅“所有订单消息”(`order.#`) 还是只订阅“支付成功消息”(`order.payment.success`)，从而极大提升系统的扩展性



#### 2. 消息流转的标准生命周期

- **Step 1: 发布 (Publish)**
  - **动作**：生产者连接到 RabbitMQ 的具体 **Virtual Host**
  - **核心参数**：
    1. **Payload**：实际的消息体（业务数据）
    2. **Exchange Name**：指定发给哪个交换机
    3. **Routing Key**：指定路由指令（如 `order.created`）



- **Step 2: 路由 (Route)**
  - **动作**：Exchange 收到消息后，扮演“路由器”角色
  - **逻辑**：它读取消息头的 `Routing Key`，并遍历自己内部维护的 **Binding (绑定表)**
  - **匹配算法**：根据交换机类型（Direct/Topic/Fanout）进行精确匹配或模糊匹配



- **Step 3: 投递 (Dispatch)**

  - **场景 A：匹配成功**
    - 消息被写入对应的 **Queue**
    - *注意：如果匹配到了多个 Queue（例如 Fanout 广播），RabbitMQ 会将消息 **复制** 多份，分别放入这些 Queue 中*

  

  - **场景 B：匹配失败 (无路可走)**
    - 如果没有任何 Binding 规则匹配该 Routing Key
    - **结果**：消息默认被 **直接丢弃**，且不会通知生产者
    - *避坑：生产环境通常需开启 `Mandatory` 参数或配置 `Dead Letter Exchange` 来捕获这些“迷路”的消息*

  

- **Step 4: 消费 (Consume)**

  - **动作**：消费者监听 Queue
  - **模式**：
    - **Push (推模式)**：MQ 主动将消息推送给消费者（Spring AMQP 默认）
    - **Pull (拉模式)**：消费者主动轮询拉取（即 `Get` 操作，性能较差，不推荐）



#### 3. 避坑指南：初学者常犯的架构错误

**陷阱一：误以为 Exchange 会存消息**

- **现象：** 生产者发了消息，但消费者还没启动（或队列还没创建绑定），结果消费者启动后收不到之前的消息
- **原因：** Exchange 是无状态的转发器。如果消息到达 Exchange 时没有队列接收它，它就“蒸发”了
- **解决：** 必须先创建 Queue 并 Binding 到 Exchange，然后再发消息。或者使用 `Mandatory` 参数让 MQ 通知你路由失败



**陷阱二：混淆 RoutingKey 与 BindingKey**

- **RoutingKey：** 是 **生产者** 发送消息时带的“动作指令”（例如：`order.create`）
- **BindingKey：** 是 **消费者/管理员** 将队列绑定到 Exchange 时设定的“接收规则”（例如：`order.#`）
- 交换机的作用就是判断：`RoutingKey` 是否符合 `BindingKey` 的规则



**陷阱三：未做持久化**

- **现象：** RabbitMQ 重启后，Exchange 和 Queue 全都不见了，消息也没了
- **解决：** 生产环境必须将 Exchange、Queue、Message 三者都设置为 **Durable（持久化）**（后续 API 章节会讲如何设置）



#### 4. 消息、交换机、队列关系!⭐⭐⭐

**请先记住：它们之间不是物理直连，而是靠“Binding 表”进行逻辑关联的**⭐

##### a. 核心三角关系

请把它们想象成数据库里的三张表：

1. **Producer (生产者)**：**“发信人”**

   - **职责**：在发送代码中 **显式指定** 消息要发给哪个 **Exchange (交换机)**
   - **代码体现**：`rabbitTemplate.convertAndSend("Exchange名字", "RoutingKey", 消息)`

   

2. **Exchange (交换机)**：相当于 **“路由器”**(持有整张路由表)

   - **职责**：接收生产者发来的消息，遍历查询内部维护的路由表中的每一个 **Binding**，转发消息
   - **注意**：它不存储消息（除非是死信队列那种特殊情况的中转），它只是一个运行逻辑的组件

   

3. **Queue (队列)**：相当于 **“存储仓库”**

   - **职责**：落地存储消息，等待消费者来取
   - **地位**：它是完全被动的，不知道消息是从哪个交换机来的，只管收

   

4. **Binding (绑定)**：相当于 “**路由表中的一行记录“**

   - **职责**：它定义了三要素 —— **“源头（Exchange）”** + **“条件（Binding Key）”** + **“去向（目标 Queue）”**
   - **本质**：**一个 Binding = 一行路由规则记录，明确了匹配成功后消息要去哪个队列**
   - **集合概念**：一个 Exchange 内部通常会有成百上千个 Binding，这些 Binding 合在一起，才构成了完整的“路由策略表”



##### 2. Routing Key vs Binding Key

- **Routing Key (路由键)**：

  - **来源**：**生产者** 发送消息时带的
  - **性质**：**动态参数**（就像 SQL 查询里的 `WHERE id = ?` 的那个值）
  - **作用**：用来去 Binding 表里做匹配

  

- **Binding Key (绑定键)**：

  - **来源**：**运维/开发** 在代码里配置的 (`BindingBuilder.with("...")`)。
  - **存储位置**：**存在 Exchange (交换机) 内部，路由表中有非常多的 Binding，Binding 中又有 Binding Key**
  - **性质**：**静态规则**
  - **作用**：定义匹配的条件



**匹配过程文字演示：**

假设 Exchange 内部维护了这样一张 Binding 表：

- **规则1**：Binding Key = `info`，目标队列 -> **Queue A**
- **规则2**：Binding Key = `error`，目标队列 -> **Queue B**
- **规则3**：Binding Key = `error`，目标队列 -> **Queue C**

**场景还原**：生产者执行代码 `convertAndSend("此Exchange", "error", msg)`

1. **定位交换机**：消息首先找到了代码中指定的 **“此Exchange”**
2. **提取钥匙**：Exchange 拿到消息里的 Routing Key —— **`'error'`**
3. **遍历查表**：Exchange 开始遍历自己的 Binding 表：
   - 对比 **规则1** (`info`)：不匹配，跳过
   - 对比 **规则2** (`error`)：**匹配成功！** 查看 Binding 中的目标 -> **复制一份发给 Queue B**
   - 对比 **规则3** (`error`)：**匹配成功！** 查看 Binding 中的目标 -> **复制一份发给 Queue C**
4. **最终结果**：Queue B 和 Queue C 都收到了消息



##### 3. 复杂映射关系详解 (N:M)

很多初学者以为“一个队列只能有一个 Key”，这是错的。它们是完全自由的 **多对多** 关系

| 关系模式               | 绑定配置 (Binding)                                           | 效果                                                         | 典型场景                                 |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------- |
| **1 对 1**(精准)       | Exchange A -> `error` -> Queue A                             | **单播**。只有 `error` 消息能进 Queue A                      | 错误日志归档                             |
| **1 对 多**(多重绑定)  | Exchange A -> `info` -> Queue AExchange A -> `warn` -> Queue A | **多名**。Queue A 有两个别名。发 `info` 或 `warn` 它都能收到 | 一个队列处理多种类型的业务               |
| **多 对 1**(广播/多播) | Exchange A -> `pay` -> Queue AExchange A -> `pay` -> Queue B | **群发**。发一条 `pay`，复制两份，A 和 B 各收一份            | 支付成功，同时通知“积分系统”和“短信系统” |



##### 4. 灵魂拷问：如果没绑定会怎样？

- **场景**：你往 Exchange 发了消息，Routing Key 是 `abc`
- **过程**：Exchange 查 Binding 表，发现没有任何一行记录匹配 `abc`
- **结果**：
  - **默认情况**：**直接丢弃 (Drop)**。RabbitMQ 不会帮你暂存，消息瞬间消失
  - **开启 Mandatory**：Exchange 会调用 `ReturnsCallback` 把消息退回给生产者（参考 4.1 节）
  - **配置了备份交换机**：转发给备用交换机



##### 5. 总结

- **生产者** 决定消息发给哪个 **Exchange**
- **Binding** 决定了匹配成功后消息去哪个 **Queue**
- **Routing Key** 是用来在 Binding 表里进行“通关验证”的口令



### 1.3 安装与控制台概览

**3分钟内启动 RabbitMQ**，并学会通过 Management UI 监控核心指标

#### 1. 极速安装

作为开发人员，不要在本地 Windows/Mac 上安装笨重的安装包。我们使用 Docker 一键启动，这也是生产环境容器化部署的标准姿势

**启动命令 (直接复制执行)：**

```bash
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin \
  rabbitmq:3-management
```

**关键参数解析：**

- **`rabbitmq:3-management`**：**必须带 `-management` 后缀！** 否则启动后没有 Web 控制台界面
- **`-p 5672:5672`**：**通信端口**。Java 代码连接 MQ 时填这个端口
- **`-p 15672:15672`**：**管理端口**。浏览器访问控制台时用这个端口
- **`-e ..._USER/PASS`**：设置默认账号密码（这里设为 `admin/admin`）

**验证启动：** 打开浏览器访问 `http://localhost:15672`，输入 `admin/admin` 即可登录



#### 2. Management UI 核心面板

登录后，你看到的是 RabbitMQ 的“驾驶舱”。请重点关注以下几个 Tab 页，它们对应了我们之前学的架构组件

##### A. Overview

> **总览 —— 集群健康状态**

- **Totals**:
  - 当前集群的消息总吞吐量（按秒统计），包括生产速率和消费速率
- **Nodes**:
  - 节点的健康状态（CPU、内存、磁盘水位）
  - **生产避坑**：
    - 如果 **Disk space** 爆红（磁盘空间不足），RabbitMQ 会触发 **流控机制**，也就是强行“刹车”，**拒绝接收生产者的新消息**，直到磁盘空间释放

- **图**

  ![image-20251214124159247](./assets/image-20251214124159247.png)





##### B. Connections & Channels

> **连接与信道 —— 网络层模型**

- **Connections (TCP连接)**：
  
  - **定义**：物理层面的 TCP 长连接
  
  - **现状**：建立 TCP 连接非常消耗资源（三次握手、鉴权）
  
  - **图**
  
    ![image-20251214124223245](./assets/image-20251214124223245.png)
  
  
  
- **Channels (虚拟信道)**：

  - **定义**：复用在 Connection 之上的“逻辑连接”

  - **最佳实践**：Spring Boot 程序通常只维持 **一个** 长连接，但在内部通过 **多线程创建多个 Channel** 来并发发送/接收消息。这叫 **“多路复用”**

  - **图**

    ![image-20251214124240167](./assets/image-20251214124240167.png)

##### C. Exchanges (交换机)

- **列表展示**：这里列出了当前 VHost 下所有的交换机

- **默认交换机**：你会发现系统默认创建了一些 `amq.*` 开头的交换机（如 `amq.direct`, `amq.topic`）

  - **建议**：**不要删，也不要直接用**。生产环境请务必自己新建业务专用的交换机，以便于管理和监控

- **图**

  ![image-20251214124300114](./assets/image-20251214124300114.png)



##### D. Queues (队列)

> **排查问题的核心战场**

点击具体的队列名称，可以查看详情。这里有两个 **至关重要** 的指标:

| 指标        | 含义                                                         | 潜在风险 / 排查方向                                          |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Ready**   | **待消费消息数**<br />消息已安全存入队列，正在等待投递给消费者 | 如果该值持续飙高，说明 **生产速度 > 消费速度**，也就是发生了 **消息积压**<br />**对策**：需要扩容消费者实例，或优化消费逻辑 |
| **Unacked** | **待确认消息数**。消息已推给消费者，但消费者 **还没回传 ACK** | 如果值很高，说明消费者 **处理太慢**，或者消费者 **卡死/报错** 导致无法提交 ACK<br/>**对策**：检查消费者日志，看是否有死锁或异常未捕获 |
| **Total**   | Ready + Unacked 的总和                                       | 队列当前的总负载                                             |

> **ACK (Acknowledge)** 的意思是 **“确认应答”**,可以把它理解为**“快递签收单”**
>
> - 简单来说，ACK 就是 **“确认收到并处理完成”** 的 **信号**



#### 3. 在控制台模拟“发布/订阅”

不写一行代码，先体验一下消息流转：

##### **建队列**

去 `Queues` 页 -> `Add a new queue` -> **Name** 填 `test.q` -> 点击 `Add queue`

![image-20251214125131464](./assets/image-20251214125131464.png)



##### **建交换机**

去 `Exchanges` 页 -> `Add a new exchange` -> Name 填 `test.ex` -> Type 选 `direct` -> 点击 `Add exchange`

![image-20251214125216894](./assets/image-20251214125216894.png)



##### **做绑定 (Binding)**

- 点击刚才建的 `test.ex` 进入详情页

  ![image-20251214125332175](./assets/image-20251214125332175.png)

- 找到 `Bindings` 栏 -> `To queue` 填 `test.q` -> `Routing key` 填 `key1` -> 点击 `Bind`

  ![image-20251214125704795](./assets/image-20251214125704795.png)





##### **发消息**

- 依然在 `test.ex` 详情页
- 找到 `Publish message` 栏 -> `Routing key` 填 `key1` -> `Payload` 填 `Hello RabbitMQ` -> 点击 `Publish message`

![image-20251214125927755](./assets/image-20251214125927755.png)



##### **验结果**

- 去 `Queues` 页，你会发现 `test.q` 的 **Ready** 变成了 `1`

  ![image-20251214130020600](./assets/image-20251214130020600.png)

- 点击 `test.q` -> 展开 `Get messages` -> 点击 `Get Message` -> 你就能看到刚才发的消息了！

  ![image-20251214130118134](./assets/image-20251214130118134.png)





## 二：五大核心消息模型 (原生 API 视角)

### 2.0 环境准备与依赖引入

在开始编写 Java 原生 API 代码之前，我们需要搭建一个纯净的 Java 开发环境

**注意**：本阶段不使用 Spring Boot，而是直接使用 RabbitMQ 官方提供的 Java 客户端 (`amqp-client`)，目的是为了让你看清 RabbitMQ 底层的真实面貌

#### 1. 核心依赖

请创建一个新的 **Maven** 项目，并在配置文件中添加以下依赖

###### Maven (`pom.xml`)

```xml
<dependencies>
    <!-- RabbitMQ Java Client 官方驱动核心包 -->
    <!-- 就像 JDBC 驱动之于 MySQL，这是连接 RabbitMQ 必不可少的桥梁 -->
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.20.0</version> <!-- 推荐使用 5.x 稳定版 -->
    </dependency>

    <!-- 可选：简单的日志依赖 (便于查看连接日志) -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>1.7.36</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```



#### 2. 版本兼容性说明

- **Client 5.x**：支持 JDK 8 及以上版本，支持 RabbitMQ 3.x 服务器。这是目前最主流的选择
- **Client 4.x**：较旧版本，通常用于 JDK 7 或更老的系统（不推荐）



#### 3. 为什么不直接用 Spring Boot?

- **原生 API**：你可以清晰地看到 `Connection` 是如何建立的，`Channel` 是如何创建的，以及 `ACK` 是如何手动提交的
- **Spring Boot**：它封装了太多的细节（自动重连、自动序列化、自动 ACK）
  - **如果跳过原生 API 直接学 Spring Boot，一旦生产环境出现“消息积压”或“连接泄露”，你将根本不知道去哪里调整参数**



### 2.1 简单队列与工作队列

#### 1. 核心 API 速查手册

在看代码前，必须先死磕这几个方法的参数。后面全靠它们

**类：`com.rabbitmq.client.Channel`** (所有核心操作都在 Channel 上，而不是 Connection 上)

##### A. 声明队列 (`queueDeclare`)

**作用**：定义一个队列。如果队列不存在则创建；如果存在但参数不一致则报错

```java
Queue.DeclareOk queueDeclare
    				(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments)
```

- `queue`: **队列名称**
- `durable`: **持久化**
  - `true` = 写入磁盘（RabbitMQ 重启后队列还在）
  - `false` = 仅存内存（RabbitMQ 重启后队列消失）
- `exclusive`: **排他性**
  - `true` = 只允许当前 Connection 访问，且连接断开后自动删除队列（通常用于临时队列）
- `autoDelete`: **自动删除**
  - `true` = 当最后一个消费者断开连接后，自动删除该队列
- `arguments`: **扩展参数**（如 TTL 过期时间、最大长度、关联死信队列等）



##### B. 发送消息 (`basicPublish`)

**作用**：将消息投递到 Exchange

```java
void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body)
```

- `exchange`: **交换机名称**
  - **重要**：如果传空字符串 `""`，代表使用 **Default Exchange**（直接投递模式）
- `routingKey`: **路由键**
  - 在使用默认交换机时，该值必须等于 **队列名称**
- `props`: **消息属性**
  - 常用 `MessageProperties.PERSISTENT_TEXT_PLAIN` 设置消息为持久化文本
- `body`: **消息体**（字节数组）



##### C. 消费消息 (`basicConsume`)

**作用**：启动一个消费者监听队列

```java
String basicConsume(String queue, boolean autoAck, Consumer callback)
```

- `queue`: 监听的 **队列名**
- `autoAck`: **自动应答机制**（非常关键）
  - `true`: 消息一发给消费者，MQ 就认为由消费者“签收”了，直接从内存删除（**风险极大，处理失败会丢数据**）
  - `false`: 需要消费者手动调用 `basicAck` 确认（**生产环境标准做法**）
- `callback`: 回调接口，处理业务逻辑



##### D. 流量控制 (`basicQos`)

**作用**：设置“预取数量”（Prefetch Count）

```
void basicQos(int prefetchCount)
```

- `prefetchCount`: 告诉 MQ，“在我没确认（ACK）之前，不要给我发送超过 X 条消息”。
- **意义**：这是解决“长尾效应”和实现“能者多劳”的关键配置。通常设置为 `1`。





#### 2. 消息模型解析

在写代码之前，必须先看懂这两个模型

##### 模型一：简单队列 (Simple Queue)

- **拓扑结构**：`P (Producer) -> Queue -> C (Consumer)`
- **逻辑**：一对一。一个生产者对应一个消费者
- **幕后黑手 —— Default Exchange**：
  - 在简单队列中，我们通常直接把消息发给队列。但你是否记得 1.2 节说过“生产者不能直接发给队列”？
  - **真相**：当你调用 `channel.basicPublish("", "hello_q", ...)` 时，第一个参数 `""` (空字符串) 实际上是告诉 RabbitMQ 使用 **默认交换机**
  - **默认交换机规则**：它会自动将消息投递到 **RoutingKey 指定名称的队列** 中



##### 模型二：工作队列 (Work Queue)

- **拓扑结构**：`P (Producer) -> Queue -> C1, C2, C3... (Consumers)`

- **场景**：

  - 生产者发送任务的速度（例如 1000条/秒）远大于消费者处理的速度（例如 100条/秒）
  - 这时通过增加消费者（C2, C3）来 **横向扩展** 处理能力

- **核心机制对比**：

  | 分发机制                | 描述                                                         | API 配置                        | 缺点/优点                                                    |
  | ----------------------- | ------------------------------------------------------------ | ------------------------------- | ------------------------------------------------------------ |
  | 轮询分发(Round-Robin)   | **(默认行为)** RabbitMQ 按顺序把消息平分给每个消费者<br />例：C1 拿第 1,3,5 条，C2 拿第 2,4,6 条 | `autoAck = true`或者未设置 QoS  | **缺点**：不考虑消费者的实际处理能力。如果 C1 处理很慢，C2 很快，C2 会闲死，C1 会累死，且整体效率低（长尾效应） |
  | 公平分发(Fair Dispatch) | **(能者多劳)** 消费者告诉 MQ：“我都还没处理完（没回 ACK），别给我发新的”<br/>MQ 就会把消息发给其他空闲的消费者 | `basicQos(1)` `autoAck = false` | **优点**：能够充分利用集群性能，防止慢节点拖垮整体进度。这是生产环境的 **最佳实践** |



#### 3. 生产级代码演示

场景模拟：生产者发送 10 个耗时任务，两个消费者（Worker1 和 Worker2）争抢处理。

##### 3.1 生产者 (TaskProducer.java)

生产者负责源源不断地生产消息。注意资源释放的标准写法

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

public class TaskProducer {
    // 定义队列名称
    private static final String QUEUE_NAME = "work_queue_demo";

    public static void main(String[] argv) throws Exception {
        // 1. 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");

        // 2. 建立连接 (Connection) 和 信道 (Channel)
        // 使用 try-with-resources 语法确保资源自动关闭 (Connection 和 Channel 都实现了 AutoCloseable)
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {

            // 3. 声明队列 (双保险：生产者和消费者都建议声明，防止谁先启动报错)
            // 参数: queue, durable(持久化), exclusive, autoDelete, arguments
            boolean durable = true; 
            channel.queueDeclare(QUEUE_NAME, durable, false, false, null);

            // 4. 模拟发送 10 条消息
            for (int i = 0; i < 10; i++) {
                String message = "Task No." + i;
                
                // 5. 发送消息
                // 参数: exchange, routingKey, props, body
                // MessageProperties.PERSISTENT_TEXT_PLAIN: 让消息持久化存储到磁盘
                channel.basicPublish("", QUEUE_NAME, 
                        MessageProperties.PERSISTENT_TEXT_PLAIN, 
                        message.getBytes("UTF-8"));
                
                System.out.println(" [x] Sent '" + message + "'");
            }
        }
    }
}
```



##### 3.2 消费者 (WorkerConsumer.java)

消费者需要长驻运行，且必须配置 QoS

```java
import com.rabbitmq.client.*;
import java.io.IOException;

public class WorkerConsumer {
    private static final String QUEUE_NAME = "work_queue_demo";

    public static void main(String[] argv) throws Exception {
        // 1. 建立连接 (同生产者)
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("admin");

        // 消费者通常不需要关闭连接，因为需要一直监听
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        // 2. 声明队列
        boolean durable = true;
        channel.queueDeclare(QUEUE_NAME, durable, false, false, null);

        // ================== 核心配置开始 ==================
        
        // 3. 设置 QoS (Quality of Service) - 公平分发关键
        // prefetchCount = 1: 告诉 RabbitMQ，在我回传 ACK 之前，不要给我发新的消息。
        // 这样 RabbitMQ 就会寻找其他空闲的消费者。
        int prefetchCount = 1;
        channel.basicQos(prefetchCount);

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        // 4. 定义消息处理的回调逻辑
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + message + "'");

            try {
                // 模拟业务处理耗时 (例如 Worker1 休眠 1s，Worker2 休眠 2s)
                doWork(message);
            } finally {
                // 5. 手动确认 (Manual ACK) - 必须在 finally 块中执行
                // deliveryTag: 消息的唯一标签 (ID)
                // multiple: false (仅确认当前这一条，不批量确认)
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
                System.out.println(" [x] Done");
            }
        };

        // 6. 启动监听
        // autoAck = false: 关闭自动确认，开启手动确认模式
        // 如果这里设为 true，上面的 basicQos 将失效！
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, consumerTag -> { });
        
        // ================== 核心配置结束 ==================
    }

    private static void doWork(String task) {
        try {
            Thread.sleep(1000); // 模拟耗时操作
        } catch (InterruptedException _ignored) {
            Thread.currentThread().interrupt();
        }
    }
}
```



#### 4. 避坑指南

1. **忘记 `basicAck` 的灾难**：
   - **现象**：如果你设置了 `autoAck = false`，但在代码里的 `finally` 块忘记调用 `channel.basicAck`
   - **后果**：
     - RabbitMQ 认为你一直在处理中，消息状态一直是 **Unacked**。MQ 内存会逐渐撑爆，且当你重启消费者时，这些旧消息会像洪水一样重新涌入，导致死循环
2. **`basicQos` 不生效**：
   - **原因**：`basicQos` 必须配合 `autoAck = false` 使用。如果 `autoAck = true`，MQ 会无视 QoS 设置，瞬间把队列里的消息全部推送到你的客户端缓冲区
3. **持久化不彻底**：
   - 要保证重启不丢数据，必须满足三点：
     1. **队列持久化** (`durable=true`)
     2. **消息持久化** (`MessageProperties.PERSISTENT_TEXT_PLAIN`)
     3. **交换机持久化** (这里用的是默认交换机，自带持久化)



### 2.2 发布订阅模型

> Publish/Subscribe - Fanout



#### 1. 模型原理解析

在之前的“工作队列”中，一条消息只能被一个消费者抢到。 而在“发布/订阅”模型中，我们要实现：**一条消息能被所有订阅者收到**

- **拓扑结构**： `P (Producer) -> X (Exchange: Fanout) -> Q1, Q2... -> C1, C2...`
- **核心机制**：
  1. 生产者不再直接把消息发给队列，而是发给 **Fanout 类型的交换机**
  2. 交换机收到消息后，会 **忽略 Routing Key**，直接把消息 **复制 (Copy)** 并转发给所有绑定到它的队列
  3. 每个消费者都有自己 **独立** 的队列，因此都能收到完整的消息副本



#### 2. 核心 API 速查

要实现这个模型，我们需要用到几个新方法：

##### A. 声明交换机 (`exchangeDeclare`)

**作用**：在 MQ 中创建一个交换机

```java
Exchange.DeclareOk exchangeDeclare(String exchange, BuiltinExchangeType type)
```

- `exchange`: 交换机名称
- `type`: 交换机类型。本节使用 `BuiltinExchangeType.FANOUT` (即 "fanout")



##### B. 创建临时队列 (`queueDeclare().getQueue()`)

**场景**：在广播模式下，消费者通常只关心“当前最新”的消息，不关心以前的。而且每个消费者都需要一个独立的队列

**最佳实践**：让 MQ 自动生成一个随机命名的、排他的、自动删除的队列

```java
// 生成一个随机名字（如 amq.gen-JzTY2...），
// 连接断开后自动删除 (Exclusive + AutoDelete)
String queueName = channel.queueDeclare().getQueue();
```



##### C. 绑定队列 (`queueBind`)

**作用**：把队列接到交换机上（插网线）

```java
Queue.BindOk queueBind(String queue, String exchange, String routingKey)
```

- `queue`: 队列名
- `exchange`: 交换机名
- `routingKey`: **在 Fanout 模式下无效**，通常传空字符串 `""`



#### 3. 生产级代码实战

场景模拟：**日志广播系统**

- 生产者：`EmitLog`，发出日志消息
- 消费者1：`SaveToDisk`，把日志存入磁盘（绑定到一个队列）
- 消费者2：`PrintConsole`，把日志打印到屏幕（绑定到另一个队列）

##### 3.1 生产者 (EmitLog.java)

注意：生产者 **只声明交换机**，不声明队列。因为生产者根本不知道（也不关心）有多少个消费者在听

```java
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class EmitLog {
    private static final String EXCHANGE_NAME = "logs_fanout";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("admin");

        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {

            // 1. 声明 Fanout 交换机
            // 必须声明，否则如果先启动生产者，交换机不存在会报错
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);

            String message = "Critical System Info: Database is down!";

            // 2. 发送消息
            // 这里的 routingKey 填空字符串，因为 Fanout 模式不看 Key
            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
```



##### 3.2 消费者 (ReceiveLogs.java)

消费者需要：声明交换机 -> 创建临时队列 -> 绑定 -> 消费

```java
import com.rabbitmq.client.*;

public class ReceiveLogs {
    private static final String EXCHANGE_NAME = "logs_fanout";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("admin");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        // 1. 声明交换机 (消费者也声明，防止先启动消费者报错)
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);

        // 2. 获取临时队列
        // 这是一个非持久化、排他、自动删除的随机队列
        String queueName = channel.queueDeclare().getQueue();
        System.out.println(" [*] Temporary Queue Name: " + queueName);

        // 3. 绑定 (Binding)
        // 核心步骤：告诉交换机，把消息转发到这个临时队列来
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        // 4. 消费消息
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + message + "'");
        };

        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }
}
```



#### 4. 避坑指南

1. **启动顺序陷阱 (Lost Messages)**
   - **现象**：如果你先启动生产者发消息，再启动消费者
   - **结果**：**消息会全部丢失！**
   - **原因**：Fanout 交换机不存消息。如果你发消息时没有任何队列绑定到它（消费者还没启动），消息就被丢进黑洞了
   - *区别*：这与 Work Queue 不同，Work Queue 的消息会暂存在队列里等待
2. **临时队列的特性**
   - 一旦消费者断开连接（Ctrl+C），这个随机生成的队列会 **立即自动删除**。里面的数据也会消失。这非常适合“实时日志”这种不需要持久化的场景





### 2.3 路由模型 (Routing - Direct)

#### 1. 模型原理解析

如果说 Fanout 是“无脑群发”，那么 Direct 就是“精准投递”

- **拓扑结构**： `P -> X (Exchange: Direct) -> Binding(Key) -> Queue -> C`
- **核心机制 (精确匹配)**：
  - **规则**：消息的 **Routing Key** 必须 **完全等于** 队列绑定的 **Binding Key**，消息才会被转发到该队列
  - **多重绑定**：一个队列可以用不同的 Binding Key 多次绑定到同一个交换机
    - *例如*：队列 Q1 绑定了 `black` 和 `green`。那么路由键是 `black` 或 `green` 的消息它都能收到
- **图解场景 (日志系统)**：
  - **Exchange**: `logs_direct`
  - **Queue A (磁盘)**: 绑定 key = `error`
  - **Queue B (屏幕)**: 绑定 key = `info`, `warning`, `error`
  - **结果**：
    - 发送 `info` -> 只有 B 收到
    - 发送 `error` -> A 和 B 都收到



#### 2. 核心 API 速查

跟 Fanout 的代码非常像，唯一的区别在于 **Binding 的时候不能传空字符串了**

##### A. 声明 Direct 交换机

```java
channel.exchangeDeclare("logs_direct", BuiltinExchangeType.DIRECT);
```



##### B. 带 Key 绑定 (`queueBind`)

这是路由模型的核心。你需要指定第三个参数

```java
// 意思是：这个队列只接收路由键为 "error" 的消息
channel.queueBind(queueName, "logs_direct", "error");
```



#### 3. 生产级代码实战

场景模拟：**智能日志路由**

- **生产者**：发送不同级别 (`info`, `error`, `warning`) 的日志
- **消费者1 (Disk)**：只把 `error` 存盘
- **消费者2 (Console)**：把所有级别的日志都打印出来



##### 3.1 生产者 (DirectEmitLog.java)

生产者发送消息时，必须指定具体的 `Routing Key`

```java
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class DirectEmitLog {
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("admin");

        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {

            // 1. 声明 Direct 交换机
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

            // 模拟发送三种级别的日志
            String[] severities = {"error", "info", "warning"};

            for (String severity : severities) {
                String message = "Log info: This is a [" + severity + "] message";
                
                // 2. 发送消息
                // 关键点：第二个参数 routingKey 必须填具体的值
                channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
                
                System.out.println(" [x] Sent '" + severity + "':'" + message + "'");
            }
        }
    }
}
```



##### 3.2 消费者 (DirectReceiveLogs.java)

为了演示灵活绑定，我们把绑定的 Keys 提取为参数

```java
import com.rabbitmq.client.*;

public class DirectReceiveLogs {
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("admin");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        // 1. 声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

        // 2. 获取临时队列
        String queueName = channel.queueDeclare().getQueue();

        // ================== 核心配置：动态绑定 ==================
        
        // 模拟：我们想监听哪些级别的日志？
        // 如果是 Consumer1 (Disk)，这里只写 new String[]{"error"};
        // 如果是 Consumer2 (Console)，这里写 new String[]{"info", "warning", "error"};
        String[] bindingKeys = new String[]{"info", "warning", "error"};

        for (String key : bindingKeys) {
            // 3. 绑定 (可以多次绑定！)
            // 对每一个感兴趣的 Key，都执行一次 bind
            channel.queueBind(queueName, EXCHANGE_NAME, key);
            System.out.println(" [*] Bind to key: " + key);
        }

        // ======================================================

        System.out.println(" [*] Waiting for messages...");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            String routingKey = delivery.getEnvelope().getRoutingKey();
            System.out.println(" [x] Received '" + routingKey + "':'" + message + "'");
        };

        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }
}
```



#### 4. 避坑指南

1. **完全匹配原则**
   - Direct 模式是非常“死板”的
   - 如果绑定的是 `error`，发送的是 `error.critical`，**收不到！**
   - 它不做任何模糊匹配（这是下一节 Topic 的工作）
2. **没有匹配的消息去哪了？**
   - 如果你发了一个 `debug` 级别的日志，但没有任何队列绑定了 `debug` 这个 Key
   - 结果：**消息直接被丢弃**。RabbitMQ 不会帮你暂存它



### 2.4 通配符模型 (Topics) —— 最强路由

#### 1. 模型原理解析

Topic 模型是 RabbitMQ 中 **最灵活、最常用** 的模型。它兼具了 Direct (精确) 和 Fanout (广播) 的能力

- **核心规则**：

  1. **Routing Key 必须是“点分字符串”**：
     - 单词之间用 `.` 隔开
     - 格式推荐：`<域>.<对象>.<动作>`
     - *例子*：`user.order.create`, `sys.cron.1min`, `quick.orange.rabbit`
  2. **Binding Key 支持通配符**：
     - **`\*` (星号)**：只能匹配 **一个单词**
     - **`#` (井号)**：可以匹配 **零个或多个单词**

- **图解算法**： 假设消息的 Routing Key 是 `quick.orange.rabbit`

  | Binding Key                | 结果     | 原因                                         |
  | -------------------------- | -------- | -------------------------------------------- |
  | `quick.orange.rabbit`      | ✅ 匹配   | 完全匹配 (等同于 Direct)                     |
  | `*.orange.*`               | ✅ 匹配   | 中间是 orange，前后各一个单词                |
  | `*.*.rabbit`               | ✅ 匹配   | 最后一个单词是 rabbit                        |
  | `quick.#`                  | ✅ 匹配   | quick 开头，后面不管有几个单词               |
  | `#.rabbit`                 | ✅ 匹配   | rabbit 结尾，前面不管有几个单词              |
  | `quick.orange.male.rabbit` | ❌ 不匹配 | `*.orange.*` 无法匹配，因为 `*` 只能占一个坑 |



#### 2. 核心 API 速查

代码结构与 Direct 几乎一致，唯一的区别是交换机类型

##### A. 声明 Topic 交换机

```java
channel.exchangeDeclare("topic_logs", BuiltinExchangeType.TOPIC);
```



#### 3. 生产级代码实战

场景模拟：**生态系统监控**。 我们发送的消息 Routing Key 格式为：`<设施>.<严重性>` (例如: `kern.critical`, `auth.info`)

- **消费者1 (核心报警)**：只关心所有设施的 `critical` 错误
- **消费者2 (内核日志)**：关心 `kern` 设施下的所有日志



##### 3.1 生产者 (TopicEmitLog.java)

```java
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.util.HashMap;
import java.util.Map;

public class TopicEmitLog {
    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("admin");

        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {

            // 1. 声明 Topic 交换机
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

            // 模拟发送不同设施、不同级别的日志
            Map<String, String> logs = new HashMap<>();
            logs.put("kern.critical", "A critical kernel error");
            logs.put("kern.info", "Kernel updated");
            logs.put("auth.info", "User login success");
            logs.put("cron.critical", "Job failed");

            for (Map.Entry<String, String> entry : logs.entrySet()) {
                String routingKey = entry.getKey();
                String message = entry.getValue();

                // 2. 发送消息
                channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes("UTF-8"));
                
                System.out.println(" [x] Sent '" + routingKey + "':'" + message + "'");
            }
        }
    }
}
```

##### 3.2 消费者 (TopicReceiveLogs.java)

```java
import com.rabbitmq.client.*;

public class TopicReceiveLogs {
    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setUsername("admin");
        factory.setPassword("admin");

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        String queueName = channel.queueDeclare().getQueue();

        // ================== 核心配置：通配符绑定 ==================
        
        // 场景 A: 我想接收所有 "critical" (严重) 的日志
        // Binding Key: "#.critical"
        // 解释: 无论前面是 kern 还是 cron，只要结尾是 critical 就行
        // channel.queueBind(queueName, EXCHANGE_NAME, "#.critical");

        // 场景 B: 我想接收所有 "kern" (内核) 的日志
        // Binding Key: "kern.*"
        // 解释: kern 开头，后面只能跟一个单词 (如果是 kern.a.b 则不匹配)
         channel.queueBind(queueName, EXCHANGE_NAME, "kern.*");

        // 场景 C (组合): 既要 kern又要 critical
        String[] bindingKeys = new String[]{"#.critical", "kern.*"};
        for (String key : bindingKeys) {
            channel.queueBind(queueName, EXCHANGE_NAME, key);
            System.out.println(" [*] Binding key: " + key);
        }

        // ======================================================

        System.out.println(" [*] Waiting for messages...");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            String routingKey = delivery.getEnvelope().getRoutingKey();
            System.out.println(" [x] Received '" + routingKey + "':'" + message + "'");
        };

        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }
}
```



#### 4. 避坑指南

1. **`\*` 与 `#` 的致命误区**
   - `*` 是 **占位符**（必须占 1 个坑）
     - 路由键 `user` **不能** 匹配 `user.*`（因为 `.` 后面缺一个词）
     - 路由键 `user.login` **能** 匹配 `user.*`
   - `#` 是 **万能符**（0 或 N 个坑）
     - 路由键 `user` **能** 匹配 `user.#`（后面 0 个词）
     - 路由键 `user.login.fail` **能** 匹配 `user.#`（后面 N 个词）
2. **退化现象**
   - 如果 Binding Key 是 `#` ——> 这就是 **Fanout**（接收所有）
   - 如果 Binding Key 不含任何 `*` 或 `#` ——> 这就是 **Direct**（精确匹配）



## 三：Spring Boot 整合与实战 (核心开发) ⭐

**阶段综述**： 在原生 API 中，我们必须手动处理 Connection 的建立与关闭、Channel 的复用、异常的捕获。而在 Spring Boot 中，这一切都被 `spring-boot-starter-amqp` 封装在黑盒之下。 本阶段的目标是：**既要享受 Spring 的便利，又要懂得黑盒里的机制，防止生产环境出现“连接泄露”或“配置失效”。**

### 3.1 核心配置与自动配置原理

#### 1. 环境搭建

在 Spring Boot 项目中，我们不再需要像第二阶段那样单独引入 `amqp-client`

- Spring Boot 提供了一个“开箱即用”的 Starter，它不仅包含了客户端驱动，还包含了自动化配置模块

##### Maven 配置 (`pom.xml`)

请在 `dependencies` 标签中添加：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <!-- 
      注意：通常无需指定版本号 (<version>)。
      版本会由 spring-boot-starter-parent 统一管理，
      确保与你的 Spring Boot 版本兼容。
    -->
</dependency>
```

- ##### 依赖解析

  - 引入这个 Starter 后，Maven 会自动拉取以下核心 jar 包

    1. **`spring-amqp`**：Spring 对 AMQP 协议的抽象层
    2. **`spring-rabbit`**：Spring 对 RabbitMQ 的具体实现（包含 `RabbitTemplate`, `RabbitAdmin` 等核心类）
    3. **`amqp-client`**：RabbitMQ 官方提供的 Java 底层驱动（就是我们第二阶段用的那个）

    

**开发建议**： 建议创建一个全新的 Spring Boot 工程（勾选 "Spring for RabbitMQ" 模块）来开始本阶段的练习，保持环境纯净



#### 2. 生产级配置 (`application.yml`)

不要只配 host 和 port，以下是生产环境的标准配置模板：

```yaml
spring:
  rabbitmq:
    # === 1. 基础连接信息 ===
    host: localhost
    port: 5672
    username: admin
    password: admin
    # 关键：生产环境请务必指定具体的 vhost，实现业务隔离
    virtual-host: /order_vhost
    
    # 连接超时设置 (防止网络波动导致长时间阻塞)
    connection-timeout: 10s 
    
    # === 2. CachingConnectionFactory 缓存配置 (性能关键) ===
    # Spring Boot 默认使用 CachingConnectionFactory
    cache:
      connection:
        # 模式：CHANNEL (默认)。缓存 Channel，复用 Connection
        # 只有在极端高并发下才考虑 CONNECTION 模式
        mode: channel 
      channel:
        # 关键：通道缓存数量 (默认 25)
        # 如果你的业务并发很高（如瞬时几百个线程同时发消息），默认值可能不够用，
        # 会导致线程阻塞等待 Channel，建议调大至 50-100
        size: 50 
        # 获取通道的最长等待时间
        checkout-timeout: 5s 

    # === 3. 消息投递可靠性配置 (第四阶段细讲，此处先预留) ===
    # 开启发布确认 (Publisher Confirms)
    publisher-confirm-type: correlated 
    # 开启消息回退 (Publisher Returns)
    publisher-returns: true            

    # === 4. 消费者监听配置 (Listener) ===
    # 这里配置的是 @RabbitListener 的默认行为
    listener:
      simple:
        # 关键：开启手动 ACK (Manual Acknowledge)
        # 对应原生 API 的 autoAck = false
        # 如果不配这一行，默认是自动确认，一旦报错消息就丢了！
        acknowledge-mode: manual       
        
        # 关键：预取数量 (Prefetch Count)
        # 对应原生 API 的 basicQos(1)。实现“能者多劳”和“公平分发”
        prefetch: 1                    
        
        # 关键：并发消费者数量
        # 决定了每个 @RabbitListener 启动时会在后台开几个线程去消费
        # concurrency: 初始线程数
        # max-concurrency: 最大线程数 (自动伸缩)
        concurrency: 5                 
        max-concurrency: 10            
```

**配置核心点解析**

1. **`virtual-host`**:
   - 新手最容易漏掉的配置。如果不配，默认连到 `/`。一定要显式指定，防止连错环境
2. **`cache.channel.size`**:
   - Spring 的 `RabbitTemplate` 发送消息时需要从池子里借一个 `Channel`。如果不调大这个值，高并发发送时这里会成为瓶颈
3. **`acknowledge-mode: manual`**:
   - **这是保命配置**。把它设为 manual，Spring 才会等待你的业务逻辑执行完（或抛出异常）后，自动帮你决定是 Ack 还是 Nack
4. **`concurrency`**:
   - 这是 Spring 封装的黑科技
   - 在原生 API 中，如果你想开启 5 个消费者，你得运行 5 次 `main` 方法或者写 5 个线程
   - 在 Spring 中，只需配置 `concurrency: 5`，Spring 就会自动为同一个队列启动 5 个消费者线程并行处理，吞吐量直接翻倍



#### 3. Spring 是如何管理连接的？

这是从原生 API 转到 Spring 后最大的思维转变

##### A.`CachingConnectionFactory `连接工厂

- **原生 API 的痛点**： 我们需要手动调用 `factory.newConnection()`，并且要小心翼翼地维护它。一旦网络波动断开，还得写一堆代码去重连

- **Spring AMQP 的解法**：

  - **单例连接机制**：Spring 默认只创建一个 **单例的 Connection**（这也符合 RabbitMQ 官方的性能建议）
  - **缓存机制**：它默认 **不缓存 Connection**，而是 **缓存 Channel (信道)**

  

- **工作流程详解**：

  1. 当 `RabbitTemplate` 需要发送消息时，它向 Factory 申请一个 Channel
  2. Factory 检查缓存池（`cache.channel.size`，默认25）里有没有空闲的？
  3. **有** -> 直接复用
  4. **没有** -> 基于那条唯一的 Connection 创建一个新的 Channel
  5. **用完后** -> 不会关闭 Channel，而是把它重置后放回缓存池

- **性能瓶颈预警**： 如果你的业务并发极高（例如每秒几千次发送），默认的 25 个 Channel 缓存可能瞬间被耗尽，导致线程阻塞等待 Channel

  - *对策*：在 `application.yml` 中调大 `spring.rabbitmq.cache.channel.size`

  

##### B.`RabbitAdmin `自动化管理员

- **作用**： 它是 Spring AMQP 的运维管家。它负责扫描 Spring 容器中的 `@Bean`，自动去 RabbitMQ Server 上创建对应的 Exchange、Queue 和 Binding
- **Lazy Declaration (懒加载声明)**：
  - **现象**：你定义好了 Queue 的 Bean，启动项目，去 MQ 控制台看，发现队列 **并没有出现** ！你以为配置挂了
  - **真相**：Spring 采用的是 **懒加载** 策略。`RabbitAdmin` 只有在 **第一次与 MQ 建立连接时** 才会去执行声明操作
  - **触发点**：
    1. 第一次发送消息时
    2. 第一次监听器启动时
  - *验证方法*：写个单元测试随便发一条消息，队列自然就出现了



#### 4. 实战：声明式结构定义

在 Spring Boot 中，我们将 AMQP 的拓扑结构定义为配置类中的 Bean

- `RabbitAdmin` 会自动扫描这些 Bean 并同步到 RabbitMQ Server

创建一个配置类 `RabbitConfig.java`：

```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitConfig {

    // ==========================================
    // 1. 定义队列 (Queue)
    // ==========================================
    @Bean
    public Queue orderQueue() {
        // 参数说明：
        // 1. name: 队列名称 "boot.order.q"
        // 2. durable: 是否持久化 (true = 重启还在)
        // 3. exclusive: 是否排他 (false = 所有连接都能访问)
        // 4. autoDelete: 是否自动删除 (false = 哪怕没消费者也不删)
        return new Queue("boot.order.q", true, false, false);
        
        // 或者使用 Builder 模式 (更易读)：
        // return QueueBuilder.durable("boot.order.q").build();
    }

    // ==========================================
    // 2. 定义交换机 (Exchange)
    // ==========================================
    @Bean
    public DirectExchange orderExchange() {
        // 定义一个 Direct 类型的交换机
        // 参数: name, durable, autoDelete
        return new DirectExchange("boot.order.ex", true, false);
    }

    // ==========================================
    // 3. 定义绑定关系 (Binding)
    // ==========================================
    @Bean
    public Binding orderBinding() {
        // 核心逻辑：把 orderQueue 绑定到 orderExchange
        // 路由键 (RoutingKey) 为 "order.create"
        // 翻译：只有 RoutingKey = "order.create" 的消息，才会进 boot.order.q
        return BindingBuilder
                .bind(orderQueue())
                .to(orderExchange())
                .with("order.create");
    }
}
```



#### 5. 避坑指南

这是新手最容易遇到的 **启动失败** 原因

- **场景还原**：
  1. 你第一次定义 Bean 时，写的是 `durable=false`（非持久化）
  2. 启动项目，RabbitMQ 创建了这个队列
  3. 后来你觉得不对，修改代码为 `durable=true`（持久化）
  4. 再次启动项目
- **结果**：
  - **报错**：`Channel shutdown: channel error; protocol method: #method<channel.close>(reply-code=406, reply-text=PRECONDITION_FAILED - inequivalent arg 'durable'...)`
  - **原因**：RabbitMQ 不允许重新定义已存在的队列，除非所有参数完全一致
- **解决方案**：
  - **粗暴法**：去 RabbitMQ 管理控制台，手动删除那个旧队列，然后重启项目
  - **代码法**：给队列换个新名字（生产环境不推荐）



### 3.2 RabbitTemplate API（发送）

#### 1. 核心 API 速查手册

`RabbitTemplate` 是线程安全的，可以在 Service 中直接注入使用

##### A. 基础发送 (`convertAndSend`)

最常用的方法。Spring 会自动将你的 Java 对象（Object）序列化为二进制（byte[]），然后封装成 AMQP 消息发送

```java
// 1. 发送到默认交换机 (RoutingKey = 队列名)
// routingKey: 路由键
//    * 由于没指定交换机，默认使用 Default Exchange
//    * 此时 RoutingKey 必须完全等于 目标队列名 才能被收到
// object: 消息体
//    * 你要发的实际业务数据 (String, User, Map 等)
void convertAndSend(String routingKey, Object object);

// 2. 发送到指定交换机 (最常用)
// exchange: 交换机名称
//    * 指定消息要发给哪个交换机 (例如 "boot.order.ex")
// routingKey: 路由键
//    * 交换机根据这个 Key 决定转发给哪个队列 (例如 "order.create")
// object: 消息体
void convertAndSend(String exchange, String routingKey, Object object);
```



##### B. 进阶发送 (`MessagePostProcessor`)

如果你需要在发送时设置 **消息过期时间 (TTL)**、**优先级** 或 **自定义 Header**，必须使用这个重载方法

```java
// 前三个参数同上...
// mpp (MessagePostProcessor): 消息后置处理器
//    * 这是一个函数式接口
//    * Spring 在把对象转成 Message 后，发给 RabbitMQ 前，会回调这个接口
//    * 你可以在这里拿到 Message 对象，随意修改它的 Header 和 Properties
void convertAndSend(String exchange, String routingKey, Object object, MessagePostProcessor mpp);
```



#### 2. 生产级代码实战

我们在 `OrderService` 中演示不同场景的发送逻辑

```java
import org.springframework.amqp.AmqpException;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessagePostProcessor;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

@Service
public class OrderService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    // 定义交换机和路由键常量 (与 3.1 节 RabbitConfig 对应)
    private static final String EXCHANGE_NAME = "boot.order.ex";
    private static final String ROUTING_KEY = "order.create";

    /**
     * 场景 1: 发送简单的字符串或对象
     */
    public void sendStringMessage() {
        String msg = "New Order Created: " + UUID.randomUUID();
        
        // 参数: 交换机, 路由键, 消息体
        rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, msg);
        
        System.out.println(" [x] Sent String: " + msg);
    }

    /**
     * 场景 2: 发送 Java Map (模拟 JSON)
     * 注意：默认使用 JDK 序列化，消费者接收时会看到乱码，下节课我们解决这个问题
     */
    public void sendMapMessage() {
        Map<String, Object> orderMap = new HashMap<>();
        orderMap.put("orderId", "1001");
        orderMap.put("price", 99.9);
        orderMap.put("user", "admin");

        rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, orderMap);
        
        System.out.println(" [x] Sent Map: " + orderMap);
    }

    /**
     * 场景 3: 发送带有过期时间 (TTL) 的消息
     * 使用 MessagePostProcessor 在消息发出前"加工"一下
     */
    public void sendExpirationMessage() {
        String msg = "This message will die in 5 seconds";

        // Lambda 表达式写法
        rabbitTemplate.convertAndSend(EXCHANGE_NAME, ROUTING_KEY, msg, message -> {
            // 设置消息属性
            // setExpiration 单位是字符串类型的毫秒
            message.getMessageProperties().setExpiration("5000");
            
            // 还可以设置 Header
            message.getMessageProperties().setHeader("source", "mobile-app");
            
            return message;
        });
        
        System.out.println(" [x] Sent TTL Message: " + msg);
    }
}
```



#### 3. 避坑指南

1. **JDK 序列化的坑 (The Serialization Trap)**
   - **现象**：你发了一个 String 或 Map，去 RabbitMQ 控制台看消息体，发现是一堆看不懂的乱码（如 `\xAC\xED\x00\x05t...`）
   - **原因**：`RabbitTemplate` 默认使用 `SimpleMessageConverter`，它基于 Java 原生的 `ObjectOutputStream` 进行序列化
   - **后果**：
     1. 消息体积大
     2. 跨语言不兼容（Python 消费者解不开 Java 的序列化数据）
     3. 具有安全风险
   - **预告**：我们将在 **3.3 节** 引入 Jackson，把消息变成通用的 JSON 格式
2. **RoutingKey 填错**
   - Spring 不会校验你的 RoutingKey 是否正确。如果你填了一个不存在的 Key，消息发给交换机后，因为匹配不到队列，会被直接 **丢弃**（除非配置了回退）





### 3.3 监听序列化器替换

#### 1. 为什么必须换掉默认的序列化器？

Spring AMQP 默认使用的 `SimpleMessageConverter` 基于 JDK 的 `ObjectOutputStream`

- **缺点 1**：生成的消息体是二进制乱码（如 `rO0ABXNy...`），在 RabbitMQ 控制台无法阅读
- **缺点 2**：对非 Java 消费者（如 Python 脚本）完全不可读
- **缺点 3**：容易引发 Java 反序列化安全漏洞

**生产标准**：使用 **JSON** 作为消息的通用载体



#### 2. 实战：配置 Jackson 消息转换器

我们需要修改（或新增）配置类，向 Spring 容器注入一个 `MessageConverter` Bean

- Spring Boot 的自动配置机制发现这个 Bean 后，会自动将其注入到 `RabbitTemplate` 中

**修改 `RabbitConfig.java`：**

```JAVA
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitConfig {

    // ... 之前定义的 Queue, Exchange, Binding 代码保持不变 ...

    /**
     * 核心配置：将消息序列化机制改为 JSON
     * 	1. 发送消息时：RabbitTemplate 会自动把 Java 对象转成 JSON 字符串
     * 	2. 接收消息时：@RabbitListener 会自动把 JSON 字符串转成 Java 对象
     */
    @Bean
    public MessageConverter jsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```



#### 3. 依赖检查

`Jackson2JsonMessageConverter` 依赖于 Jackson 核心包。通常 `spring-boot-starter-web` 已经包含了它

如果你是纯后端微服务（没有 Web 模块），可能需要单独引入：

```XML
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```



#### 4. 效果验证

配置好这个 Bean 后，你可以重新运行 3.2 节的 `sendMapMessage()` 方法

**变化**：

- **以前**：控制台 Payload 是 `\xAC\xED\x00\x05sr...`

- **现在**：控制台 Payload 变成了清晰的 JSON：

  ```
  {"orderId":"1001", "price":99.9, "user":"admin"}
  ```

- **Content-Type**：会自动变为 `application/json`





### 3.3 注解式监听 (`@RabbitListener`)



#### 0. `@RabbitListener` 注解 (核心基础)

##### 简述

- `@RabbitListener` 是 Spring AMQP 提供的魔法注解，它的底层是一个 **异步消息监听容器**
  - 一旦你在方法上加了这个注解，Spring 就会在后台启动线程去 RabbitMQ 拉取消息

##### **常用参数速查**

- **`queues`** (最常用)

  - **含义**: 指定监听的队列名称（支持数组）
  - **注意**: 这里填写的队列 **必须预先存在** ！如果队列在 MQ 中不存在，启动时会直接报错
  - **示例**: `queues = "boot.order.q"` 或 `queues = {"q1", "q2"}`

  

- **`concurrency`** (性能调优)

  - **含义**: 指定当前这个监听器的 **并发消费者线程数**
  - **优先级**: **高于** `application.yml` 中的全局配置
  - **格式**: `"固定值"` 或 `"初始值-最大值"`
  - **示例**: `concurrency = "5-10"`（表示该队列最少起 5 个线程消费，忙不过来时自动扩容到 10 个）

  

- **`ackMode`** (覆盖配置)

  - **含义**: 允许你针对某个特定的队列，覆盖全局的 ACK 模式
  - **值**: `"MANUAL"` (手动), `"AUTO"` (自动), `"NONE"` (无)
  - **示例**: `ackMode = "MANUAL"`

  

- **`containerFactory`** (高级配置)

  - **含义**: 当你有多个配置不同的工厂时（比如一个工厂处理 JSON，一个工厂处理二进制），用来指定使用哪一个工厂



##### **补充参数：`bindings` (高级声明式绑定)**

这是一个 **“懒人神器”**。它和 `queues` 参数是 **二选一** 的关系

###### **核心区别**

- **`queues`**：**“只负责监听”**

  - 前提：队列必须 **已经存在** 于 RabbitMQ 中（或者你已经在 `@Configuration` 类里定义了 `@Bean`）
  - 如果队列不存在，启动报错

  

- **`bindings`**：**“连建带连再监听” (三合一)**

  - 前提：**不依赖** 任何前置配置
  - 效果：容器启动时，Spring 会自动去 MQ 检查：
    1. 队列有吗？没有我给你建
    2. 交换机有吗？没有我给你建
    3. 绑定关系有吗？没有我给你绑
    4. 最后开始监听



###### **属性全解 (API 速查)**

`@QueueBinding` 注解本身包含三个核心属性，分别对应 AMQP 的三大组件：

1. **`value` (目标队列)**

   - **类型**: `@Queue`
   - **作用**: 定义要监听的队列
   - **常用内部属性**:
     - `name`: 队列名称。如果留空 `""`，MQ 会生成一个随机名称（通常用于临时队列）
     - `durable`: 是否持久化 (默认 `true`)。重启后队列是否还在
     - `autoDelete`: 是否自动删除 (默认 `false`)。没有消费者连接时是否删除
     - `exclusive`: 是否排他 (默认 `false`)

2. **`exchange` (源头交换机)**

   - **类型**: `@Exchange`
   - **作用**: 定义消息来源的交换机
   - **常用内部属性**:
     - `name`: 交换机名称
     - `type`: 交换机类型。推荐使用常量 `ExchangeTypes.DIRECT`, `TOPIC`, `FANOUT`
     - `delayed`: 是否支持延迟消息 (默认 `false`)。需要安装插件才生效
     - `ignoreDeclarationExceptions`: 是否忽略声明异常 (默认 `false`)。如果交换机已存在且属性不一致，设为 `true` 可防止启动报错

3. **`key` (路由规则)**

   - **类型**: `String[]` (字符串数组)
   - **作用**: 定义 Binding Key
   - **说明**: 支持绑定多个 Key。例如 `key = {"red", "blue"}` 表示只要是红色或蓝色的消息都接收

   

###### **使用建议**

- **不要混用**：虽然语法允许，但强烈建议 **不要** 同时使用 `queues` 和 `bindings`
- **场景**：
  - 用 `bindings`：适合小型项目、测试代码，或者消费者逻辑非常独立，不想写繁琐的 `@Configuration` 配置类时
  - 用 `queues`：适合大型规范化项目，队列结构统一在配置类管理，消费者只负责干活



###### **代码示例**

```java
// 这是一个"一站式"写法
// 自动创建队列 "direct.queue1"
// 自动创建交换机 "hmall.direct"
// 自动绑定 RoutingKey "red" 和 "blue"
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"), // 1. 声明队列
    exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT), // 2. 声明交换机
    key = {"red", "blue"} // 3. 绑定规则
))
public void listenDirectQueue1(String msg){
    System.out.println("消费者1接收到消息：【" + msg + "】");
}
```



#### 1. 前置检查

在写代码前，请务必确认 `application.yml` 中已开启手动确认模式（除非你在注解里单独配置了 `ackMode`）：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: manual  # 必须配置！开启手动 ACK
        prefetch: 1               # 关键：设置为 1 实现"能者多劳"
```



#### 2. 生产级代码实战：订单消费者

我们将创建一个监听器，模拟处理订单逻辑。请注意 `try-catch-finally` 的结构，这是标准模版

```JAVA
import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Map;

@Component
public class OrderListener {

    /**
     * 核心注解 @RabbitListener
     * queues: 监听的队列名 (必须已存在，否则报错)
     */
    @RabbitListener(queues = "boot.order.q")
    public void listenOrder(Map<String, Object> orderMap, Message message, Channel channel) throws IOException {
        
        // 1. 获取消息的唯一标识 (DeliveryTag)
        // 它是 Channel 内自增的数字，用于确认消息
        long deliveryTag = message.getMessageProperties().getDeliveryTag();

        try {
            // 2. 执行业务逻辑
            System.out.println("收到订单消息：" + orderMap);
            System.out.println("当前线程：" + Thread.currentThread().getName());
            
            // 模拟业务耗时
            Thread.sleep(1000);
            
            // 模拟业务异常 (测试 Nack 用)
            if (orderMap.get("price") == null) {
                throw new RuntimeException("价格不能为空");
            }

            // 3. 手动确认 (ACK)
            // 参数1: deliveryTag
            // 参数2: multiple (false=只确认当前一条, true=批量确认所有小于tag的消息)
            channel.basicAck(deliveryTag, false);
            System.out.println("订单处理成功，已ACK");

        } catch (Exception e) {
            System.err.println("业务处理失败: " + e.getMessage());
            
            // 4. 异常处理 (NACK / Reject)
            // 生产环境通常有两种策略：
            
            // 策略 A: 失败重回队列 (requeue = true)
            // 风险：如果是因为代码 Bug 导致的报错，消息会无限循环重试，卡死队列！
            // channel.basicNack(deliveryTag, false, true);
            
            // 策略 B: 拒绝并丢弃 (requeue = false) -> 进入死信队列 (DLX)
            // 推荐：配合死信队列使用，防止消息丢失
            channel.basicNack(deliveryTag, false, false);
        }
    }
}
```



#### 3. 核心机制：方法参数注入

Spring AMQP 非常智能，它会利用 **反射机制** 检查你定义的方法参数类型，然后自动注入对应的内容。你不需要死记顺序，只需要知道它支持哪些类型

##### A. 支持的参数类型清单

| 参数类型                                    | 注入内容           | 必须性 (手动ACK模式) | 用途                                                     |
| ------------------------------------------- | ------------------ | -------------------- | -------------------------------------------------------- |
| **`com.rabbitmq.client.Channel`**           | 原生信道对象       | **必须**             | 只有拿到它，才能调用 `basicAck` 执行签收                 |
| **`org.springframework.amqp.core.Message`** | 原生消息包装类     | **必须**             | 只有拿到它，才能获取 `DeliveryTag` (快递单号)            |
| **`Map` / `String` / `User`/ 其它**         | 业务数据 (Payload) | 可选                 | Spring 会自动调用 Jackson 将 JSON 反序列化为你指定的类型 |
| **`@Header("xxx") String val`**             | 指定的消息头       | 可选                 | 获取某个特定的 Header 值                                 |



##### B. 常见写法组合

**写法 1：最全写法 (生产标准)**

- 既要处理业务，又要手动 ACK

```java
@RabbitListener(queues = "q1")
public void handle(Map<String, Object> data, Channel channel, Message msg) {
    // 1. 处理 data
    // 2. msg.getMessageProperties().getDeliveryTag()
    // 3. channel.basicAck(...)
}
```



**写法 2：偷懒写法 (仅限自动 ACK 模式)**

- 如果 `acknowledge-mode: auto`，你可以只写业务对象。Spring 会在方法无异常执行完后自动 ACK

```java
@RabbitListener(queues = "q1")
public void handle(User user) {
    System.out.println("收到用户：" + user.getName());
}
```



##### C. 致命陷阱：参数缺失导致“假死”

- **场景**：你在 `application.yml` 里配了 `manual` (手动确认)，但是方法签名里**只写了** `void handle(User user)`，**忘记写** `Channel` 和 `Message`
- **后果**：
  1. 你拿不到 `Channel`，代码里就没法写 `basicAck()`
  2. 方法执行完了，但 MQ 没收到 ACK，消息状态一直是 `Unacked`
  3. 因为 `prefetch=1`，MQ 以为你还没忙完，**彻底停止 **给你发新消息
  4. **结果**：消费者 **假死**，队列 **积压**



#### 4. 核心 API

在 catch 块中，我们有三种方式处理失败的消息：

| 方法            | 参数                  | 含义 (通俗版)                                | 适用场景        |
| --------------- | --------------------- | -------------------------------------------- | --------------- |
| **basicAck**    | `tag, false`          | **签收**。东西没问题，我收下了               | 业务成功        |
| **basicNack**   | `tag, false, requeue` | **拒收(强)**。东西坏了，我不要。支持批量拒绝 | 业务报错 (推荐) |
| **basicReject** | `tag, requeue`        | **拒收(弱)**。功能同上，但 **不支持批量**    | 较少使用        |



#### 5. 避坑指南

1. **无限重试风暴**
   - **场景**：你在 `catch` 块中写了 `channel.basicNack(tag, false, true)`（重回队列）
   - **后果**：如果报错是因为数据格式错误（代码 Bug），这条消息会：取出 -> 报错 -> 放回 -> 取出 -> 报错...
   - **导致**：CPU 飙升，日志打满，其他消息被堵塞
   - **解法**：重回队列一定要谨慎！通常只针对网络抖动等临时异常。对于代码逻辑错误，必须 `requeue=false` (扔进死信队列人工处理)
2. **重复 ACK**
   - **场景**：Spring 可能会在某些配置下自动帮你 ACK，如果你代码里又写了一遍 `basicAck`
   - **后果**：报错 `Double Ack`，Channel 会被关闭
   - **解法**：确保 `application.yml` 中 `acknowledge-mode: manual`



### 3.4 交换机

#### A Direct Exchange (直连交换机)

这是 RabbitMQ 默认且最简单的模式，也是我们学习的起点

> 在 RabbitMQ 中，空字符串名称的交换机是一个 **特殊的系统内置交换机**，被称为 **Default Exchange**



##### 1. 概念解析：精准打击

**核心逻辑**：**“完全匹配，一字不差”**

它的规则非常“死板”：

- **发送方**：发送消息时，会在快递单上写一个具体的地址（**Routing Key**，例如 `"error"`）
- **交换机**：它手里有一张路由表（Binding），上面写着：“Routing Key 是 `"error"` 的消息，送给 Queue A”
- **结果**：只有当 **Routing Key** 字符串 **完全等于** 队列绑定的 **Binding Key** 时，消息才会被投递

**场景**：

- 消息 A (Key=`"info"`) ---> 交换机 ---> 没找到 `"info"` 的绑定 ---> **丢弃**
- 消息 B (Key=`"error"`) ---> 交换机 ---> 找到 `"error"` 绑定 ---> **投递给 Queue A**

**适用场景**：

- 只需要简单的点对点通信
- 或者需要根据明确的标识（如日志级别、支付渠道）进行分流



##### 2. Spring Boot 配置实战

在 Spring 中，我们需要用 `@Bean` 把这套“路由规则”注册到容器里

- 在配置类中，我们定义“Exchange”、“Queue” 以及 “Binding”

###### **新建配置类 `DirectConfig.java`**

```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DirectConfig {

    // ==========================================
    // 1. 定义队列 (收件箱)
    // ==========================================
    @Bean
    public Queue directErrorQueue() {
        // 【核心 API】new Queue(name, durable, exclusive, autoDelete)
        // 参数1: name (队列名) -> "boot.direct.error"
        // 参数2: durable (持久化) -> true (推荐! 重启后队列还在)
        // 参数3: exclusive (排他性) -> false (推荐! 设为 true 则只有创建它的连接能连，断开即删)
        // 参数4: autoDelete (自动删除) -> false (推荐! 设为 true 则没消费者监听时自动删除)
        return new Queue("boot.direct.error", true, false, false);
        
        // 也可以用 Builder 模式 (更易读)：
        // return QueueBuilder.durable("boot.direct.error").build();
    }

    // ==========================================
    // 2. 定义 Direct 交换机 (分拣中心)
    // ==========================================
    @Bean
    public DirectExchange directExchange() {
        // 【核心 API】new DirectExchange(name, durable, autoDelete)
        // 参数1: name (交换机名) -> "boot.direct.ex"
        // 参数2: durable (持久化) -> true (推荐! 重启后交换机还在)
        // 参数3: autoDelete (自动删除) -> false (推荐! 设为 true 则没有队列绑定时自动删除)
        return new DirectExchange("boot.direct.ex", true, false);
    }

    // ==========================================
    // 3. 定义绑定关系 (路由规则)
    // ==========================================
    @Bean
    public Binding directBinding() {
        // 链式编程逻辑：
        // 1. bind(队列): 指定要绑哪个队列
        // 2. to(交换机): 指定绑到哪个交换机
        // 3. with("key"): 指定 Binding Key (锁芯形状)
        return BindingBuilder.bind(directErrorQueue())
                .to(directExchange())
                .with("error"); // <---【关键】只有 RoutingKey="error" 才能进
    }
}

```



###### 核心 API 参数速查表 (Ref)

不管是 `Queue` 还是 `Exchange`，构造函数里那几个布尔值非常容易搞混，请参考下表：

| 参数名         | 含义         | 生产建议  | 后果说明                                                     |
| -------------- | ------------ | --------- | ------------------------------------------------------------ |
| **Durable**    | **持久化**   | **True**  | 设为 `false`，RabbitMQ 重启后该队列/交换机会 **直接消失**    |
| **Exclusive**  | **排他性**   | **False** | 设为 `true`，则 **只有当前连接** 能访问该队列，且连接断开队列 **立即自删**。通常用于临时队列 |
| **AutoDelete** | **自动删除** | **False** | 设为 `true`，当最后一个消费者断开连接（或最后一个绑定解绑）后，队列/交换机 **立即自删** |



##### 3. 收发代码实战

配置好了结构，我们来看看怎么用代码来触发这个流程

**生产者发送 (精准投递):**

在 Service 中注入 `RabbitTemplate` 发送消息

```java
@Autowired
private RabbitTemplate rabbitTemplate;

public void sendLog() {
    // 1. 指定交换机 (必须与 Config 中定义的一致)
    String exchangeName = "boot.direct.ex";

    // 场景 A：发送 "error" 日志 -> 成功
    // 原因：RoutingKey "error" 与 BindingKey "error" 完全相等
    rabbitTemplate.convertAndSend(exchangeName, "error", "这是报错日志，请查收");
    System.out.println("发送 error 日志成功");

    // 场景 B：发送 "info" 日志 -> 失败 (丢失)
    // 原因：RoutingKey "info" 在 Binding 表里找不到匹配记录
    // 结果：Exchange 收到后直接丢弃 (Drop)
    rabbitTemplate.convertAndSend(exchangeName, "info", "这是普通信息，将被丢弃");
    System.out.println("发送 info 日志成功(但其实丢了)");
}
```

**消费者监听:**

消费者只需要监听队列即可，不需要关心交换机是啥逻辑

```java
@Component
public class DirectListener {

    // queues: 监听的队列名 (对应 Config 中的 queue name)
    @RabbitListener(queues = "boot.direct.error")
    public void receiveError(String msg) {
        System.out.println("【报警服务】收到报错日志：" + msg);
    }
}
```



##### 4. 避坑指南 (Direct 特性)

1. **一字之差，千里之别**
   - 如果你绑定的 Key 是 `"error"`，发送时写成了 `"Error"` (大写) 或者 `"error "` (多个空格)
   - **结果**：匹配失败，消息丢失。Direct 模式是 **区分大小写** 且 **完全匹配** 的
2. **多重绑定 (1对多)**
   - Direct 也可以实现群发，只要你让**多个不同的队列**都绑定**同一个 Key**（比如都绑定 `"error"`）
   - 这样一来，发一条 `"error"` 消息，这几个队列都能收到（类似于广播，但仅限于该 Key）



##### 5. 本节小结

- **Direct** = **`==`** (字符串相等判断)
- 它是最严谨的模式，RoutingKey 必须和 BindingKey 一模一样
- 如果找不到匹配的 Key，消息默认会被 **丢弃**



#### B Fanout Exchange (扇形交换机)

这是 RabbitMQ 中最简单粗暴，但 **性能最好** 的模式

##### 1. 概念解析：全量广播

**核心逻辑**：**“我全都要，无视路由键 (Binding Key)”**

- **发送方**：给代码指定的这个 Fanout 交换机发送消息时，虽然协议要求填 Routing Key，但你可以随便填（通常填空字符串 `""`），因为交换机根本不看
- **交换机**：它像一个 **大喇叭**。只要收到消息，它会立即 **复制（Copy）** 多份，分别投递给 **所有** 绑定到它(就是这个Fanout Exchange)的 **所有** 队列
- **结果**：绑定了多少个队列，就有多少个消费者能收到这条消息



**场景**：

- 生产者发送一条公告：“服务器将在 10 分钟后维护”
- 交换机 Fanout ---> 复制 ---> Queue A (邮件服务) ---> 发邮件
- 交换机 Fanout ---> 复制 ---> Queue B (短信服务) ---> 发短信
- 交换机 Fanout ---> 复制 ---> Queue C (站内信) ---> 弹窗



**适用场景**：

- **系统通知**：用户下单成功，同时通知库存、积分、营销系统
- **实时刷新**：游戏全服公告、股票价格变动推送



##### 2. Spring Boot 配置实战

配置 Fanout 的关键在于 **Binding（绑定）** 环节 —— **不需要指定 Key**

- 广播模式通常意味着 **“一个交换机 -> 多个队列”**。所以我们在配置类里至少要定义两个队列，才能看出广播的效果

###### **新建配置类 `FanoutConfig.java`**

```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FanoutConfig {

    // ==========================================
    // 1. 定义队列 (模拟两个下游系统)
    // ==========================================
    
    // 队列 A：模拟邮件服务
    @Bean
    public Queue emailQueue() {
        // 【核心 API】new Queue(name, durable, exclusive, autoDelete)
        // 参数1: name (队列名) -> "boot.fanout.email"
        // 参数2: durable (持久化) -> true (推荐! 重启后队列还在)
        // 参数3: exclusive (排他性) -> false (推荐! 设为 true 则只有创建它的连接能连，断开即删)
        // 参数4: autoDelete (自动删除) -> false (推荐! 设为 true 则没消费者监听时自动删除)
        return new Queue("boot.fanout.email", true, false, false);
    }

    // 队列 B：模拟短信服务
    @Bean
    public Queue smsQueue() {
        // 参数配置同上，生产环境建议保持一致
        return new Queue("boot.fanout.sms", true, false, false);
    }

    // ==========================================
    // 2. 定义 Fanout 交换机 (大喇叭)
    // ==========================================
    
    @Bean
    public FanoutExchange fanoutExchange() {
        // 【核心 API】new FanoutExchange(name, durable, autoDelete)
        // 参数1: name (交换机名) -> "boot.fanout.ex"
        // 参数2: durable (持久化) -> true (推荐! 重启后交换机还在)
        // 参数3: autoDelete (自动删除) -> false (推荐! 设为 true 则没有队列绑定时自动删除)
        return new FanoutExchange("boot.fanout.ex", true, false);
    }

    // ==========================================
    // 3. 定义绑定关系 (插网线)
    // ==========================================
    
    @Bean
    public Binding bindingEmail() {
        // 【关键差异】注意这里没有 .with("key") ！
        // Fanout 模式下，BindingBuilder.bind(q).to(ex) 之后就结束了
        // 语义：只要插上这根网线，Exchange 里的所有消息无脑转发给 Queue
        // 链式编程逻辑：
        // 1. bind(队列): 指定要绑哪个队列
        // 2. to(交换机): 指定绑到哪个交换机
        // 3. with("key"): 指定 Binding Key (锁芯形状)
        return BindingBuilder.bind(emailQueue()).to(fanoutExchange());
    }

    @Bean
    public Binding bindingSms() {
        // 把第二个队列也挂上去
        // 链式编程逻辑：
        // 1. bind(队列): 指定要绑哪个队列
        // 2. to(交换机): 指定绑到哪个交换机
        // 3. with("key"): 指定 Binding Key (锁芯形状)
        return BindingBuilder.bind(smsQueue()).to(fanoutExchange());
    }
}

```



###### 核心 API 参数速查表 (Ref)

不管是 `Queue` 还是 `Exchange`，构造函数里那几个布尔值非常容易搞混，请参考下表：

| 参数名         | 含义         | 生产建议  | 后果说明                                                     |
| -------------- | ------------ | --------- | ------------------------------------------------------------ |
| **Durable**    | **持久化**   | **True**  | 设为 `false`，RabbitMQ 重启后该队列/交换机会 **直接消失**    |
| **Exclusive**  | **排他性**   | **False** | 设为 `true`，则 **只有当前连接** 能访问该队列，且连接断开队列 **立即自删**。通常用于临时队列 |
| **AutoDelete** | **自动删除** | **False** | 设为 `true`，当最后一个消费者断开连接（或最后一个绑定解绑）后，队列/交换机 **立即自删** |



##### 3. 收发代码实战

**生产者发送 (广播):**

```java
@Autowired
private RabbitTemplate rabbitTemplate;

public void sendNotification() {
    String exchangeName = "boot.fanout.ex";
    String msg = "紧急通知：今晚 12 点停机维护！";

    // 【核心细节】RoutingKey 填什么？
    // 虽然协议要求这里必须填一个字符串，但在 Fanout 模式下，
    // 交换机直接无视这个值。
    // 最佳实践：填空字符串 ""。
    // (如果你填了 "abc"，代码不会报错，但也没任何实际意义)
    rabbitTemplate.convertAndSend(exchangeName, "", msg);
    
    System.out.println("广播消息已发出：" + msg);
}
```



**消费者监听 (两个消费者):**

这里我们需要两个监听器，分别监听上面的两个队列，验证是否都能收到

```java
@Component
public class FanoutListener {

    // 消费者 1：监听邮件队列
    @RabbitListener(queues = "boot.fanout.email")
    public void handleEmail(String msg) {
        System.out.println("【邮件服务】收到广播：" + msg);
        // TODO: 调用邮件发送接口...
    }

    // 消费者 2：监听短信队列
    @RabbitListener(queues = "boot.fanout.sms")
    public void handleSms(String msg) {
        System.out.println("【短信服务】收到广播：" + msg);
        // TODO: 调用短信网关...
    }
}
```

**预期运行结果：** 当你调用 `sendNotification()` 一次，控制台会 **同时** 打印出：

- `【邮件服务】收到广播：紧急通知...`
- `【短信服务】收到广播：紧急通知...`



##### 4. 避坑指南 (Fanout 特性)

1. **性能之王**
   - 为什么它最快？因为 Direct 和 Topic 都要在内部执行字符串匹配算法（RoutingKey vs BindingKey），而 Fanout 只需要查一下“我有几个绑定”，然后无脑转发即可
   - 如果你的业务不需要路由过滤，优先选 Fanout
2. **不要试图用 Key 过滤**
   - 很多新手配置了 Fanout，又习惯性地在发送时加了 RoutingKey，指望消费者能过滤。这是徒劳的
   - **Fanout 下，RoutingKey 参数是摆设**



##### 5. 本节小结

- **Fanout** = **广播** (One to Many)
- 配置绑定时，**不需要** `.with()` 指定 Key
- 它是实现 **应用解耦** 的最典型模式（上游发消息，下游随便加消费者，上游完全不知情）



#### C Topic Exchange (主题交换机)

这是 RabbitMQ 的“瑞士军刀”。如果你在设计系统时拿不准用哪个交换机，**选 Topic 准没错**



##### 1. 概念解析：通配符路由

**核心逻辑**：**“模式匹配，模糊路由”**

- **Routing Key 规范**：必须是 **点分字符串** (单词用 `.` 隔开)
  - 推荐格式：`<业务域>.<对象>.<动作>`
  - 例如：`user.login.success`、`order.pay.failed`、`log.app.error`
- **Binding Key 通配符**：
  - **`*` (星号)**：匹配 **正好一个** 单词
    - `*.error` -> 能匹配 `app.error`，**不能** 匹配 `app.user.error`
  - **`#` (井号)**：匹配 **零个或多个** 单词
    - `order.#` -> 能匹配 `order.pay`，也能匹配 `order.pay.success`

**图解算法**： 假设消息的 Routing Key 是 `quick.orange.rabbit`：

| Binding Key                | 结果     | 原因                                         |
| -------------------------- | -------- | -------------------------------------------- |
| `quick.orange.rabbit`      | ✅ 匹配   | 完全匹配 (退化为 Direct)                     |
| `*.orange.*`               | ✅ 匹配   | 中间是 orange，前后各一个单词                |
| `*.*.rabbit`               | ✅ 匹配   | 最后一个单词是 rabbit                        |
| `quick.#`                  | ✅ 匹配   | quick 开头，后面不管有几个单词               |
| `#.rabbit`                 | ✅ 匹配   | rabbit 结尾，前面不管有几个单词              |
| `quick.orange.male.rabbit` | ❌ 不匹配 | `*.orange.*` 无法匹配，因为 `*` 只能占一个坑 |



##### 2. Spring Boot 配置实战

场景模拟：**多级日志系统**。 我们有三个队列，分别关注不同的日志：

1. `allQueue`: 接收所有日志 (`#`)
2. `userQueue`: 只接收用户模块的日志 (`user.#`)
3. `errorQueue`: 接收所有模块的报错日志 (`#.error`)

**新建配置类 `TopicConfig.java`：**

```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TopicConfig {

    // ==========================================
    // 1. 定义队列 (准备三个收件箱)
    // ==========================================
    
    // 队列 A：接收所有日志 (海纳百川)
    @Bean 
    public Queue allQueue() { 
        // 【核心 API】new Queue(name, durable, exclusive, autoDelete)
        // 参数1: name (队列名) -> "boot.direct.error"
        // 参数2: durable (持久化) -> true (推荐! 重启后队列还在)
        // 参数3: exclusive (排他性) -> false (推荐! 设为 true 则只有创建它的连接能连，断开即删)
        // 参数4: autoDelete (自动删除) -> false (推荐! 设为 true 则没消费者监听时自动删除)
        return new Queue("boot.topic.all", true, false, false); 
    }
    
    // 队列 B：只接收用户模块的日志 (按业务域过滤)
    @Bean 
    public Queue userQueue() { 
        return new Queue("boot.topic.user", true, false, false); 
    }
    
    // 队列 C：接收所有模块的报错日志 (按级别过滤)
    @Bean 
    public Queue errorQueue() { 
        return new Queue("boot.topic.error", true, false, false); 
    }

    // ==========================================
    // 2. 定义 Topic 交换机 (通配符路由器)
    // ==========================================
    
    @Bean
    public TopicExchange topicExchange() {
        // 【核心 API】new TopicExchange(name, durable, autoDelete)
        // 参数1: name (交换机名) -> "boot.direct.ex"
        // 参数2: durable (持久化) -> true (推荐! 重启后交换机还在)
        // 参数3: autoDelete (自动删除) -> false (推荐! 设为 true 则没有队列绑定时自动删除)
        return new TopicExchange("boot.topic.ex", true, false);
    }

    // ==========================================
    // 3. 定义绑定 (核心通配符规则)
    // ==========================================
    
    // 绑定规则 1: 接收所有 ( # )
    @Bean
    public Binding bindAll() {
        // 规则解释："#" 匹配 0 个或多个单词
        // 效果：只要发给这个交换机，无论 RoutingKey 是什么，都会进这个队列
        // (此时退化为 Fanout 效果)
        return BindingBuilder.bind(allQueue()).to(topicExchange()).with("#");
    }

    // 绑定规则 2: 接收用户模块所有操作 ( user.# )
    @Bean
    public Binding bindUser() {
        // 规则解释：
        // "user" 开头
        // "." 作为分隔符
        // "#" 匹配后面所有内容
        // 匹配：user.login, user.pay.success, user.a.b.c
        // 不匹配：order.user, user (因为#虽然匹配0个词，但格式通常隐含了层级)
        return BindingBuilder.bind(userQueue()).to(topicExchange()).with("user.#");
    }

    // 绑定规则 3: 接收所有模块的报错 ( #.error )
    @Bean
    public Binding bindError() {
        // 规则解释：
        // 前面不管有几个单词 (#)
        // 最后一个单词必须是 error
        // 匹配：user.error, order.pay.error, system.error
        // 不匹配：user.error.info (因为 error 不是最后一个词)
        return BindingBuilder.bind(errorQueue()).to(topicExchange()).with("#.error");
    }
}

```



##### 3. 收发代码实战

**生产者发送 (灵活路由):**

```java
@Autowired
private RabbitTemplate rabbitTemplate;

public void sendTopicLog() {
    String exchange = "boot.topic.ex";

    // 场景 A: 用户登录成功 -> "user.login.info"
    // 路由分析：
    // 1. 匹配 "#" (allQueue) ? -> YES
    // 2. 匹配 "user.#" (userQueue) ? -> YES (user开头)
    // 3. 匹配 "#.error" (errorQueue) ? -> NO (info结尾)
    // 结果：allQueue, userQueue 收到
    rabbitTemplate.convertAndSend(exchange, "user.login.info", "用户张三登录了");

    // 场景 B: 支付服务报错 -> "pay.error"
    // 路由分析：
    // 1. 匹配 "#" ? -> YES
    // 2. 匹配 "user.#" ? -> NO (pay开头)
    // 3. 匹配 "#.error" ? -> YES (error结尾)
    // 结果：allQueue, errorQueue 收到
    rabbitTemplate.convertAndSend(exchange, "pay.error", "支付服务炸了");
    
    // 场景 C: 用户模块报错 -> "user.login.error"
    // 路由分析：
    // 1. 匹配 "#" ? -> YES
    // 2. 匹配 "user.#" ? -> YES (user开头)
    // 3. 匹配 "#.error" ? -> YES (error结尾)
    // 结果：三个队列都收到！
    rabbitTemplate.convertAndSend(exchange, "user.login.error", "用户登录报错");
}
```

**消费者监听:**

```java
@Component
public class TopicListener {

    @RabbitListener(queues = "boot.topic.user")
    public void listenUser(String msg) {
        System.out.println("【用户服务】收到用户相关日志：" + msg);
        // 此时收到的可能是 "user.login.info" 也可能是 "user.login.error"
    }
}
```



##### 4. 避坑指南 (Topic 特性)

1. **Routing Key 格式错误**
   - Topic 交换机要求 Key 必须是单词列表。如果你发的消息 Key 是 `"user_login_info"`（下划线连接），这被视为 **一个单词**
   - 此时 `user.*` 无法匹配它，必须用 `user.#` 或者完全匹配
   - **建议**：始终使用点号 `.` 分隔
2. **效率误区**
   - 虽然 Topic 很强大，但它的路由匹配算法复杂度比 Direct 高
   - 如果你的系统 QPS 极高（十万级），且路由逻辑很简单，请优先使用 Direct。但在 99% 的业务系统中，Topic 的性能损耗可以忽略不计



##### 5. 本节小结

- **Topic** = **`\*` (1个词)** 和 **`#` (N个词)**
- 它是 RabbitMQ 灵活性的集大成者
- **架构建议**：在系统设计初期，即使你觉得只需要 Direct，也建议 **默认使用 Topic 交换机** 。因为一旦业务变复杂，Topic 可以直接扩展，不需要改架构





## 四：消息可靠性保障

### 4.1 保证消息不丢失 (全链路分析)

#### 1. 消息到底丢在哪了？

不要只盯着代码看，要把视线拉长到整个数据流。一条消息从产生到最终被消费，会经过**三个关键环节**，**任何一个环节出错都会导致丢数据**

- **防线 1：发送端可靠性 (Producer -> Broker)**

  - **风险**：网络抖动、交换机名称写错、MQ 磁盘满拒绝接收
  - **结果**：消息根本没发出去，或者发出去 MQ 没收到
  - **对策**：**Publisher Confirms (发布确认)** + **Publisher Returns (回退)**（*下节讲*）

  

- **防线 2：存储端可靠性 (Broker Internal)**

  - **风险**：RabbitMQ 服务宕机、服务器断电、Docker 崩溃
  - **结果**：内存里的消息瞬间蒸发
  - **对策**：**Persistence (持久化机制)**（*本节重点*）

  

- **防线 3：消费端可靠性 (Broker -> Consumer)**

  - **风险**：消费者刚收到消息还没处理（或处理报错）就宕机了，但 MQ 以为你处理完了
  - **结果**：消息丢失
  - **对策**：**Manual ACK (手动确认)**。（*3.3节已讲，是防线3的核心*）



#### 2. 防线 2：持久化机制

**核心原则**：要想 RabbitMQ 重启后数据还在，必须同时满足三个条件的持久化。缺一不可！

##### A. 交换机持久化

- **作用**：保证交换机的元数据（名称、类型、属性）不丢失

- **代码**：

  ```java
  // new DirectExchange(name, durable, autoDelete)
  // 第二个参数 true 代表持久化
  new DirectExchange("boot.order.ex", true, false);
  ```



##### B. 队列持久化

- **作用**：保证队列的元数据（名称、属性）不丢失

- **注意**：**队列持久化不代表消息持久化！** 它只保证“盒子”不丢，不保证“盒子里的东西”不丢

- **代码**：

  ```java
  // new Queue(name, durable, exclusive, autoDelete)
  // 第二个参数 true 代表持久化
  new Queue("boot.order.q", true, false, false);
  ```



##### C. 消息持久化

- **作用**：保证消息内容被写入磁盘

- **原生 API**：

  ```java
  channel.basicPublish(..., MessageProperties.PERSISTENT_TEXT_PLAIN, ...);
  ```

- **Spring Boot**：

  - **好消息**：`RabbitTemplate` 默认发送的消息 **就是持久化的** (DeliveryMode = 2)
  - *验证*：你可以查看 `RabbitTemplate` 源码或 Debug 消息属性，默认 `deliveryMode` 确实是 `PERSISTENT`



##### D. 避坑指南

1. **性能损耗**
   - 持久化需要写磁盘（Disk I/O）
   - 虽然 RabbitMQ 有缓冲区优化，但持久化消息的吞吐量通常比非持久化低一个数量级
   - **建议**：核心业务（订单、支付）必须全链路持久化；像“临时日志”、“系统状态采样”这种丢了也没事的数据，可以关闭持久化以换取高性能
2. **Queue 属性冲突（老生常谈）**
   - 如果你把一个原本是非持久化的队列改成持久化，启动会报错。**必须删了重建！**



#### 3. 防线 1：发送端确认机制

生产者发消息其实涉及两个步骤，RabbitMQ 提供了两种不同的回调机制来监控这两个步骤：

- **关卡 A：到达交换机了吗？ (ConfirmCallback)**

  - **对应机制**：`Publisher Confirms`
  - **成功 (Ack)**：消息成功到达 Exchange
  - **失败 (Nack)**：Exchange 不存在，或网络中断

  

- **关卡 B：入队了吗？ (ReturnsCallback)**

  - **对应机制**：`Publisher Returns`
  - **触发条件**：消息到达了 Exchange，但是 **没有匹配到任何队列**（RoutingKey 写错或 Binding 没建好）
  - **结果**：MQ 会把这条无法路由的消息 **“退回”** 给生产者



#### 4. 生产级配置 (`application.yml`)

要开启这两个机制，必须先修改配置：

```yaml
spring:
  rabbitmq:
    # 1. 开启发布确认 (Confirm)
    # simple: 同步确认 (性能差，不推荐)
    # correlated: 异步回调确认 (生产标准模式，通过 CorrelationData 关联)
    publisher-confirm-type: correlated
    
    # 2. 开启消息回退 (Returns)
    publisher-returns: true
    
    # 3. 强制把退回的消息发给回调方法 (必须配！否则退回的消息直接丢弃)
    template:
      mandatory: true
```



#### 5. 核心代码实战：配置全局回调

我们需要配置 `RabbitTemplate`，给它安上两个“监听器”。通常在配置类中完成

**修改 `RabbitConfig.java`：**

```java
import jakarta.annotation.PostConstruct; // Spring Boot 3.x (2.x用 javax)
import org.springframework.amqp.core.ReturnedMessage;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitConfig {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 初始化 RabbitTemplate 的回调逻辑
     * @PostConstruct 确保在 Bean 属性注入完成后执行
     */
    @PostConstruct
    public void initRabbitTemplate() {

        // =================================================================
        // 1. 设置 ConfirmCallback (消息是否到达交换机)
        // =================================================================
        rabbitTemplate.setConfirmCallback((CorrelationData correlationData, boolean ack, String cause) -> {
            // correlationData: 消息的唯一标识 (需要我们在发送时自己指定)
            // ack: true=交换机收到了, false=失败了
            // cause: 失败原因
            
            String msgId = (correlationData != null) ? correlationData.getId() : "Unknown";
            
            if (ack) {
                System.out.println("✅ 消息发送成功 (Ack), ID: " + msgId);
            } else {
                System.err.println("❌ 消息发送失败 (Nack), ID: " + msgId + ", 原因: " + cause);
                // TODO: 可以在这里执行重试逻辑，或者记录到"发送失败表"中后续人工补偿
            }
        });

        // =================================================================
        // 2. 设置 ReturnsCallback (消息是否路由到队列)
        // =================================================================
        // 注意：只有消息"无法路由"时，才会触发这个回调。如果正常入队，这里不会响。
        rabbitTemplate.setReturnsCallback((ReturnedMessage returned) -> {
            System.err.println("⚠️ 消息被退回 (Return) !");
            System.err.println("   Exchange: " + returned.getExchange());
            System.err.println("   RoutingKey: " + returned.getRoutingKey());
            System.err.println("   ReplyText: " + returned.getReplyText()); // 退回原因
            System.err.println("   Message: " + returned.getMessage());
            
            // TODO: 这种情况通常是配置错误 (RoutingKey写错)，需要报警通知开发人员修改代码
        });
    }
}
```



#### 6. 发送端配合

为了在回调中知道是 **哪条消息** 成功或失败了，我们在发送时必须带上“身份证号” (`CorrelationData`)

**Service 层代码：**

```java
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.UUID;

@Service
public class OrderProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendOrder() {
        // 1. 生成唯一 ID (关联数据)
        String msgId = UUID.randomUUID().toString();
        CorrelationData correlationData = new CorrelationData(msgId);

        // 2. 发送消息 (带上 ID)
        rabbitTemplate.convertAndSend(
            "boot.order.ex", 
            "order.create", 
            "New Order Data...", 
            correlationData // <--- 关键参数！
        );
        
        System.out.println("发送消息中... ID: " + msgId);
    }
    
    // 模拟一个 RoutingKey 写错的场景，测试 ReturnsCallback
    public void sendWrongMessage() {
        rabbitTemplate.convertAndSend(
            "boot.order.ex", 
            "wrong.key.123", // 这个 Key 没绑定任何队列
            "Data...", 
            new CorrelationData(UUID.randomUUID().toString())
        );
    }
}
```



#### 7. 避坑指南

1. **回调是异步的**
   - `setConfirmCallback` 是异步执行的。千万不要指望在 `convertAndSend` 后面直接拿到结果
2. **ReturnsCallback 不触发？**
   - 检查 `application.yml` 里 `mandatory: true` 配了吗？没配的话，MQ 发现路由不到，直接就丢弃了，不会退回
3. **ConfirmCallback 里的 `correlationData` 是 null？**
   - 原因：你在调用 `convertAndSend` 时，忘记传第 4 个参数 `CorrelationData` 对象了



### 4.2 死信队列

#### 1. 什么是死信？

死信就是“无法被正常消费的消息”。在 RabbitMQ 中，消息变成死信有且仅有以下 **三种情况**：

1. **消息被拒绝 (Rejection)**：
   - 消费者调用了 `basicNack` 或 `basicReject`
   - 并且设置了 **`requeue = false`** (不重回队列)
   - *这是最常见的业务报错处理方式*
2. **消息过期 (TTL Expired)**：
   - 消息在队列中存活的时间超过了设置的 TTL (Time To Live)，且还没被消费
3. **队列达到最大长度 (Queue Length Limit)**：
   - 队列满了，最早入队的消息会被挤出来，变成死信

#### 2. DLX 架构

DLX 本质上就是一个 **普通的交换机**，只是我们在定义 **正常队列** 时，给它加了一个属性：“如果我有死信，请发给这个交换机”

**死信流转过程详解：**

1. **正常流程**：生产者将消息发送给“正常交换机”，消息进入“正常队列”
2. **触发死信**：当消息在“正常队列”中过期、被拒收或因队列满被挤出时，它不会被立即丢弃
3. **二次转发**：正常队列会根据配置，自动将这条死信转发给绑定的 **“死信交换机 (DLX)”**
4. **最终归宿**：死信交换机将消息路由到 **“死信队列”**
5. **后续处理**：专门的报警消费者监听死信队列，进行记录或通知人工处理



#### 3. 生产级代码实战

我们需要定义两套结构：**正常业务结构** 和 **死信备份结构**

**修改 `RabbitConfig.java`：**

```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DlxConfig {

    // ==========================================
    // 1. 定义死信结构 (垃圾回收站)
    // ==========================================
    
    @Bean
    public DirectExchange dlxExchange() {
        return new DirectExchange("boot.dlx.ex");
    }

    @Bean
    public Queue dlxQueue() {
        return new Queue("boot.dlx.q");
    }

    @Bean
    public Binding dlxBinding() {
        return BindingBuilder.bind(dlxQueue())
                .to(dlxExchange())
                .with("dlx.key"); // 死信的 RoutingKey
    }

    // ==========================================
    // 2. 定义正常业务结构 (指定 DLX)
    // ==========================================
    
    @Bean
    public DirectExchange bizExchange() {
        return new DirectExchange("boot.biz.ex");
    }

    @Bean
    public Queue bizQueue() {
        // 关键点：给正常队列绑定死信交换机
        return QueueBuilder.durable("boot.biz.q")
                // A. 声明死信交换机是谁
                .deadLetterExchange("boot.dlx.ex")
                // B. 声明死信发过去时用什么 RoutingKey
                .deadLetterRoutingKey("dlx.key")
                // (可选) C. 设置队列最大长度，测试"队列满"产生死信
                // .maxLength(5) 
                // (可选) D. 设置消息过期时间 (10秒)，测试 TTL 死信
                // .ttl(10000) 
                .build();
    }

    @Bean
    public Binding bizBinding() {
        return BindingBuilder.bind(bizQueue())
                .to(bizExchange())
                .with("biz.order");
    }
}
```



#### 4. 消费者配合 (拒收消息)

在消费者中，我们需要模拟“业务失败拒收”的场景，触发死信流程

```java
@Component
public class BizConsumer {

    @RabbitListener(queues = "boot.biz.q") // 监听正常队列
    public void listen(String msg, Channel channel, Message message) throws IOException {
        long tag = message.getMessageProperties().getDeliveryTag();

        try {
            System.out.println("收到业务消息: " + msg);
            
            if (msg.contains("dead")) {
                throw new RuntimeException("触发死信！");
            }
            
            channel.basicAck(tag, false);
            
        } catch (Exception e) {
            System.err.println("消费失败，进入死信队列...");
            
            // 关键：requeue = false
            // 只有设为 false，消息才会从 biz.q 移出，转投给 dlx.ex
            channel.basicNack(tag, false, false);
        }
    }
}
```



#### 5. 避坑指南

1. **参数无法动态修改 (Queue Immutable)**
   - **场景**：`boot.biz.q` 之前已经存在了，现在你修改 Java 代码加了 `.deadLetterExchange(...)`。
   - **结果**：启动报错！`inequivalent arg 'x-dead-letter-exchange'`。
   - **原因**：RabbitMQ 的队列一旦创建，其参数（Arguments）就不能修改。
   - **解决**：必须先**删除旧队列**，再启动项目重新创建。
2. **死信死循环**
   - 千万不要让死信队列又把消息转发回正常队列（除非你有计数器控制），否则会形成无限循环。



### 4.3 延迟队列

#### 1. 为什么需要延迟队列？

在开发中，我们经常遇到这种需求：**“消息发出去后，不要立即消费，而是过一段时间（如 30分钟）再消费”**

**经典场景：**

- **订单超时**：用户下单后，如果 30 分钟没支付，系统自动取消订单，回滚库存
- **重试机制**：接口调用失败，等 1 分钟后再重试
- **智能家居**：设定 2 小时后关闭空调



**传统方案 vs MQ 方案**：

- ❌ **定时任务 (Cron)**：每分钟扫一次数据库，查出所有超时的订单
  - *缺点*：数据库压力大，时间不精准（有扫描间隔）
- ✅ **RabbitMQ 延迟队列**：发送一条“30分钟后才送达”的消息
  - *优点*：无数据库压力，时间精准



#### 2. 实现原理：TTL + DLX 组合拳

RabbitMQ 原生**并没有**“延迟队列”这个功能（除非装插件）

- 我们通常利用 **TTL (Time To Live)** 和 **DLX (Dead Letter Exchange)** 的特性来“模拟”出延迟效果

**架构逻辑图：**

1. **缓冲阶段 (Waiting)**：

   - 创建一个 **TTL 队列**（死信队列的前身）
   - **关键点 1**：给这个队列设置过期时间（如 10秒），或者给消息设置过期时间
   - **关键点 2**：**不要**给这个队列配置任何消费者（让消息在里面“等死”）
   - **关键点 3**：给这个队列配置 **DLX**（指向实际处理业务的交换机）

   

2. **执行阶段 (Processing)**：

   - 消息在 TTL 队列里躺了 10 秒，过期了
   - RabbitMQ 把它踢出队列，转发给 DLX
   - DLX 把它路由到 **业务队列**
   - 消费者监听业务队列，收到消息**（此时距离发送刚好过去了 10 秒）**



#### 3. 生产级代码实战 (Java Config)

我们要实现：发送一条消息，**10秒后** 被消费者收到

**修改 `RabbitConfig.java`：**

```java
import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DelayConfig {

    // ==========================================
    // 1. 定义缓冲部分 (TTL Queue) - 没有消费者
    // ==========================================
    
    @Bean
    public DirectExchange ttlExchange() {
        return new DirectExchange("boot.ttl.ex");
    }

    @Bean
    public Queue ttlQueue() {
        return QueueBuilder.durable("boot.ttl.q")
                // 关键设置 A: 设置死信交换机 (指向实际业务交换机)
                .deadLetterExchange("boot.process.ex")
                // 关键设置 B: 设置死信 RoutingKey
                .deadLetterRoutingKey("process.key")
                // 关键设置 C: 设置队列统一过期时间 (10000ms = 10s)
                // 在这个队列里的消息，10秒后自动变成死信
                .ttl(10000) 
                .build();
    }

    @Bean
    public Binding ttlBinding() {
        return BindingBuilder.bind(ttlQueue())
                .to(ttlExchange())
                .with("ttl.key");
    }

    // ==========================================
    // 2. 定义处理部分 (Process Queue) - 有消费者
    // ==========================================
    
    @Bean
    public DirectExchange processExchange() {
        return new DirectExchange("boot.process.ex");
    }

    @Bean
    public Queue processQueue() {
        return new Queue("boot.process.q");
    }

    @Bean
    public Binding processBinding() {
        return BindingBuilder.bind(processQueue())
                .to(processExchange())
                .with("process.key");
    }
}
```

**发送与消费：**

- **生产者**：发给 `boot.ttl.ex` (缓冲交换机)
- **消费者**：监听 `boot.process.q` (业务队列)
- **结果**：生产者发出后，控制台静默 10 秒，然后消费者打印收到消息



#### 4. 避坑指南

既然可以在队列设置 TTL，那能不能在 **发送消息时** 设置 TTL 呢？ 比如：

- 发消息 A：TTL = 10秒。
- 发消息 B：TTL = 5秒。

**代码：**

```java
rabbitTemplate.convertAndSend(..., msgA, m -> { m.getMessageProperties().setExpiration("10000"); return m; });
rabbitTemplate.convertAndSend(..., msgB, m -> { m.getMessageProperties().setExpiration("5000"); return m; });
```

**陷阱：** RabbitMQ 的原生队列是一个 FIFO (先进先出) 的链表。它 **只检查队头消息** 是否过期

**后果：**

1. 消息 A (10s) 先进队，排在第一个
2. 消息 B (5s) 后进队，排在第二个
3. 即使 5 秒过去了，B 该过期了，但因为 A 还在前面堵着（A 还没过期），RabbitMQ 根本扫描不到 B
4. 结果：**B 必须等 A 过期（10s）后，才能跟着一起过期**

**结论：**

- 如果你的延迟时间是固定的（如所有订单都是 30 分钟），用 **队列 TTL**（代码演示的那种）
- 如果你的延迟时间是动态的（有的 5秒，有的 10分钟），**严禁使用原生 TTL！**
- **解决方案**：安装 RabbitMQ 官方插件 `rabbitmq_delayed_message_exchange`（它将消息存在 Mnesia 数据库里，解决了排队阻塞问题）



## 五：高级业务场景与分布式难题

### 5.1 消息积压与紧急扩容

#### 1. 事故现场：如何判断积压？

不要等到用户投诉才发现。请死死盯着监控面板（Grafana 或 RabbitMQ UI）：

- **现象 A：消费慢 (Slow Consumer)**
  - `Ready` 持续上涨（例如 10w -> 20w）
  - `Unacked` 有数值且稳定（说明消费者还在干活，只是干得太慢）
  - **结论**：生产速度 > 消费速度。**需要扩容**
- **现象 B：消费死锁**
  - `Ready` 飙升
  - `Unacked` = `prefetch count` * 消费者数量（例如 5个消费者，prefetch=1，则 Unacked 卡在 5 不动）
  - **结论**：消费者拿到消息后卡在半路（比如数据库锁死、外部接口超时未设置），既不 Ack 也不 Nack。**需要重启或修复 Bug**



#### 2. Plan A：常规扩容 (增加并发)

如果仅仅是因为促销活动导致流量激增，且下游数据库（MySQL/Redis）还能扛得住，最快的方法就是 **加消费者线程**

**Spring Boot 极速扩容法：**

不需要改代码，只需要修改 `application.yml` 或在启动命令中覆盖参数：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        # 初始并发线程数 (默认1) -> 改为 10
        concurrency: 10
        # 最大并发线程数 -> 改为 20
        max-concurrency: 20
        # 预取数量 (关键) -> 适当调大，减少网络交互
        prefetch: 10
```

**操作步骤**：

1. 修改配置
2. 重启服务（或者滚动更新）
3. **效果**：单台机器的处理能力瞬间提升 10-20 倍（前提是 CPU 和 数据库撑得住）



#### 3. Plan B：紧急搬运 (拆库/泄洪)

**场景**： MQ 里积压了 1000 万条消息。即使你把消费者扩容到 100 个，但 **数据库挂了**（MySQL 写入瓶颈）。此时单纯加消费者只会让数据库死得更快

**目标**： 先保证 MQ 不被撑爆（磁盘满导致整个集群宕机），把消息 **先挪出来**，等高峰期过了再慢慢处理

**实战步骤**：

1. **停掉旧消费者**：

   - 防止它们继续轰炸不堪重负的数据库

   

2. **上线“搬运工”消费者 (Dummy Consumer)**：

   - 编写一个 **不做任何业务逻辑** 的消费者
   - **逻辑**：收到消息 -> **不做处理** -> 这里的“处理”是指：
     - 方案一：写入高性能存储（如 Kafka、Redis List、甚至写本地文件）
     - 方案二：如果只是为了测试或不重要数据，直接空跑 Ack
   - **特点**：速度极快（只受网络限制）

   

3. **临时建立新队列 (可选)**：

   - 如果需要后续处理，可以将“搬运工”设计为 **转发器**
   - 搬运工收到消息 -> 轮询转发给 10 个新的临时 Topic/Queue
   - 这相当于把一个大湖的水分流到 10 个小池子里

   

4. **夜间重放**：

   - 等到流量高峰过去，数据库恢复正常
   - 写一个脚本，把刚才缓存在 Redis/Kafka/临时队列里的数据，重新读取出来，按照正常的业务逻辑慢慢消费



#### 4. 预防措施

1. **上线前压测**：
   - 一定要知道你的消费者 **最高 TPS** 是多少
   - *计算公式*：如果是 1000 TPS，面对 10万 QPS 的秒杀，你需要 100 个消费者
2. **设置 TTL (过期丢弃)**：
   - 对于非核心数据（如日志、验证码），设置队列 TTL。如果积压超过 10 分钟没处理，直接丢弃（或进死信），保住 MQ 不死
3. **限流 (QoS Prefetch)**：
   - 再次强调 `prefetch: 1`。防止消费者一次拿太多处理不完，导致消息卡在 Unacked 状态，重启还会导致消息重新入队造成的流量反扑

#### 5. 本节小结

- **积压** = **生产 > 消费**
- **Plan A**：改 `concurrency` 加线程（前提是 DB 能抗）
- **Plan B**：写一个**不操作 DB** 的临时消费者，先把消息从 MQ 挪到 Redis/文件 里存起来，稍后处理



### 5.2 消息幂等性设计

> 防止重复消费

#### 1. 为什么会重复消费？

很多新手认为 RabbitMQ 像 TCP 一样可靠，发一次就收一次。**错！**

- RabbitMQ 为了保证消息 **绝对不丢失**，采用的是 **"At Least Once" (至少投递一次)** 策略

**灾难场景还原**：

1. 消费者收到了消息 (扣款 100 元)
2. 消费者扣款成功，写入数据库
3. **就在这时，网络抖动了！**
4. 消费者发送的 `ACK` 确认包在半路丢失，MQ 没收到
5. **MQ 的反应**：MQ 认为消费者没收到（超时未 ACK），于是触发 **重试机制**，把这条消息重新发给了另一个消费者
6. **结果**：第二个消费者再次扣款 100 元。**用户炸毛了**



#### 2. 什么是幂等性？

**数学定义**：`f(f(x)) = f(x)`。无论执行多少次，结果都和执行一次一样

**业务定义**：对于同一条消息（同一个业务 ID），无论消费者收到了多少次，**数据库里的结果只能变一次**



#### 3. 解决方案 A：数据库唯一键 (强一致性)

这是最简单、最兜底的方案。利用数据库的 **Unique Constraint** 来防重

- **原理**：
  - 在数据库中建一张 `message_log` 表，将 `message_id` 设为唯一索引
  - 消费时，先 `INSERT INTO message_log ...`。
  - 如果报错 `DuplicateKeyException`，说明已经消费过了，直接 ACK 并忽略
- **优点**：实现简单，数据绝对安全
- **缺点**：数据库是系统的瓶颈，高并发下性能差



#### 4. 解决方案 B：Redis 原子性去重(高性能)

生产环境通常使用 Redis 的 **SETNX (Set If Not Exists)** 命令来实现

**逻辑流程**：

1. 消费者收到消息，获取唯一 ID (如 `order_id` 或 `message_id`)
2. 请求 Redis：`SETNX key value` (例如 `SETNX consumed:order:1001 1`)
3. **返回 True**：说明第一次来 -> **执行业务** -> (可选)更新 Redis 状态
4. **返回 False**：说明键已存在 -> **重复消费** -> 直接 ACK 并丢弃

##### 生产级代码实战 (Spring Boot + Redis)

**UserListener.java**：

```java
import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.concurrent.TimeUnit;

@Component
public class IdempotentListener {

    @Autowired
    private StringRedisTemplate redisTemplate;

    @RabbitListener(queues = "boot.order.q")
    public void listen(String orderJson, Message message, Channel channel) throws IOException {
        
        // 1. 获取消息唯一标识 (业务ID 或 MessageID)
        // 建议让生产者在 Header 里带上全局唯一的 MessageID
        String msgId = message.getMessageProperties().getMessageId();
        
        // 兜底：如果生产者没传 MessageId，尝试从 JSON 里解析 orderId (伪代码)
        if (msgId == null) {
            msgId = "fallback_id_" + System.currentTimeMillis(); 
        }

        String key = "mq:consumed:" + msgId;

        try {
            // 2.【核心步骤】Redis 原子判断
            // setIfAbsent 等同于 SETNX
            // 关键：必须设置过期时间！防止业务报错导致 Key 永远无法删除(如果逻辑是执行完删Key的话)，或者防止缓存无限膨胀
            // 这里设置 10 分钟，假设 10 分钟后重试机制早已停止
            Boolean isNew = redisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.MINUTES);

            if (!Boolean.TRUE.equals(isNew)) {
                // isNew = false，说明 Key 已存在
                System.out.println("🛑 重复消息，直接丢弃: " + msgId);
                // 也要 ACK，否则 MQ 会一直发
                channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
                return;
            }

            // 3. 执行正常业务
            System.out.println("✅ 处理业务逻辑: " + orderJson);
            // doBusiness(orderJson);

            // 4. 手动 ACK
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);

        } catch (Exception e) {
            // 业务异常，需要删除 Redis Key，以便让重试机制能再次进来 (根据业务情况决定)
            // 或者直接 Nack 进入死信
            System.err.println("❌ 处理失败: " + e.getMessage());
            
            // 这里的策略要小心：
            // 如果是代码 bug，删了 key 也会一直报错。
            // 建议：redisTemplate.delete(key); 
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
        }
    }
}
```



#### 5. 避坑指南

1. **先写 Redis 还是后写 Redis？**
   - **先写 (锁机制)**：上面代码演示的就是“先抢锁，再干活”。适合防止并发重复
   - **后写 (日志机制)**：干完活了，往 Redis 写一条记录。**缺点**：如果干完活 Redis 挂了，没写进去，下次还会重复干活
   - **结论**：**推荐先写 (SETNX)**，把它当锁用
2. **Key 的过期时间 (TTL)**
   - 必须设置！
   - 如果不设 TTL，Redis 内存会随着时间推移被撑爆
   - 时间设置多久？通常设置为 **“大于 MQ 的最大重试时间”** 即可（例如 1 小时）
3. **Redis 挂了怎么办？**
   - 如果 Redis 宕机，SETNX 会报错
   - **降级策略**：catch 到 Redis 连接异常，降级去查 **数据库** 的去重表

#### 6. 本节小结

- RabbitMQ 保证的是 **“至少一次”**，不是“恰好一次”
- 消费者端必须设计 **“幂等性”** 逻辑
- **Redis SETNX** 是实现幂等性最高效的方案



### 5.3 集群架构与高可用

#### 1. 为什么 RabbitMQ 的集群这么特殊？

RabbitMQ 是基于 Erlang 语言编写的，Erlang 天生支持分布式。但 RabbitMQ 的集群模式和 Kafka/RocketMQ 不太一样

它主要有三种模式：

| 模式                   | 数据同步程度                        | 可用性                 | 吞吐量             | 评价                                     |
| ---------------------- | ----------------------------------- | ---------------------- | ------------------ | ---------------------------------------- |
| **普通集群**(Standard) | **元数据同步** 消息内容 **不** 同步 | 低(节点挂了数据不可见) | 高                 | 适合对数据丢失不敏感、只追求扩展性的场景 |
| **镜像队列**(Mirrored) | **全量同步** 所有节点都存一份数据   | 高(节点挂了自动切主)   | 低(网络带宽消耗大) | **经典 HA 方案** (但即将被废弃)          |
| **仲裁队列**(Quorum)   | **Raft 协议同步** 基于日志复制      | 极高(数据强一致性)     | 中                 | **新一代 HA 方案** (推荐)                |



#### 2. 深入解析：三种架构模型

##### A. 普通集群 —— "默认模式"

- **机制**：
  - 假设有 Node A 和 Node B
  - 你在 Node A 创建了一个队列 `Q1`
  - **元数据**（队列的名字、属性）会被同步到 Node B
  - **真实数据**（消息体）**只存在于 Node A**
- **现象**：
  - 消费者连接到 Node B 也可以消费 `Q1` 的消息，但是 Node B 会在内部偷偷转发请求给 Node A
- **致命弱点**：
  - 如果 **Node A 宕机**，`Q1` 里的数据就彻底无法访问了（直到 A 恢复）。这就不是高可用



##### B. 镜像队列 —— "经典 HA"

- **机制**：
  - 通过配置策略 (Policy)，让 Queue 的数据在集群节点间自动复制
  - 包含一个 **Master** 和多个 **Slave**
  - **写操作**：都发给 Master，然后同步给 Slaves
  - **读操作**：都找 Master（为了保证一致性）
- **故障转移**：
  - 如果 Master 挂了，集群会自动选在一个 Slave 晋升为 Master
- **缺点**：
  - **惊群效应**：数据同步量巨大，网络带宽压力大



##### C. 仲裁队列 —— "未来之星"

- **机制**：
  - 抛弃了传统的镜像同步，改用 **Raft 一致性协议**（和 Redis Sentinel、Etcd 类似）
  - 只同步操作日志，效率更高，数据更安全
- **现状**：
  - 官方强烈推荐用 Quorum Queues 替代镜像队列



#### 3. 接入层设计

不管是哪种集群，客户端（Java 代码）都不应该把 IP 写死

**错误写法**：

```yaml
spring:
  rabbitmq:
    host: 192.168.1.101 # 直接连某个节点
```

- **风险**：如果 101 挂了，你的服务就崩了

**正确架构**： `Client -> HAProxy / Nginx (VIP) -> RabbitMQ Cluster`

1. **搭建 HAProxy**：配置 TCP 代理，监听 5672 端口

2. **配置轮询**：将请求轮询转发给后端的 RabbitMQ 节点（Node A, Node B, Node C）

3. **Java 配置**：

   ```yaml
   spring:
     rabbitmq:
       # 填写 HAProxy 的 VIP 地址
       host: 192.168.1.200 
   ```



#### 4. 本节小结

- **普通集群** 不保数据，只分担流量
- **镜像队列**（或 **仲裁队列**）才是真正的高可用方案
- 生产环境代码连接的应该是 **Load Balancer (VIP)** 的地址，而不是具体物理机的 IP
