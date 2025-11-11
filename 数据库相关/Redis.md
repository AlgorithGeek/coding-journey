# 基础入门

## 1. Redis 简介

### 什么是 Redis？

- Redis 的全称是 **RE**mote **DI**ctionary **S**erver (远程字典服务)

- 从本质上讲，Redis 是一个开源的、高性能的、基于**内存**的**键值** 存储系统
  - 它通常被用作数据库、缓存和消息中间件



### Redis 的特性

- Redis 之所以被广泛应用，主要得益于其以下几个核心特性：

  1. **内存存储**

     - Redis 的核心数据存储在服务器的物理内存中。这是其实现极高读写性能的根本原因（避免了磁盘 I/O 的瓶颈）
     - 它也支持**持久化**，即将内存中的数据异步写入磁盘，以确保在服务重启后数据不丢失

  2. **高性能**

     - 由于数据在内存中，读写操作的速度非常快。在标准硬件上，Redis 的 每秒查询率 通常能达到 10 万级别
     - 它采用**单线程模型**（处理网络请求和执行命令）结合 **I/O 多路复用** 技术，避免了多线程上下文切换和锁竞争带来的性能开销

  3. **丰富的数据结构**

     - Redis 不仅仅是存储简单的字符串键值对。它内置支持多种复杂的数据结构，并且对这些结构的操作是**原子性**的

     - 主要包括：

       - 字符串 (String)
       - 哈希 (Hash)
       - 列表 (List)
       - 集合 (Set)
       - 有序集合 (Sorted Set / ZSet)

       此外，还包括 Bitmaps (位图), HyperLogLogs (基数统计), Geospatial (地理空间) 和 Streams (流) 等高级结构

  4. **高可用与扩展性**

     - 支持主从复制 (Replication)，实现数据备份和读写分离

       > 简单来说，就是让一个 Redis 服务器（主节点）成为数据的主要来源，而其他服务器（从节点）则去“订阅”这个主节点的数据变更

     - 支持哨兵 (Sentinel) 机制，实现主节点的自动故障转移

       > 哨兵专门用来解决主从复制中的一个核心问题：**如果主节点宕机了，谁来负责自动地把一个从节点提升为新的主节点**

     - 支持集群 (Cluster)，实现数据的自动分片 (Sharding)，突破单机内存和性能限制



### Redis 的典型应用场景

基于上述特性，Redis 在实际业务中有非常广泛的应用：

1. **缓存 (Cache)**

   - **场景**：

     - 这是 Redis 最常见的用途

       用于缓存热点数据（如商品信息、新闻详情、用户资料），大幅降低后端数据库的访问压力，提升应用响应速度

2. **计数器 (Counter)**

   - **场景**：
     - 利用 `INCR` / `DECR` 命令的原子性（单线程保证），用于实现如文章阅读量、视频播放量、用户点赞数、库存扣减等功能

3. **消息队列 (Message Queue)**

   - **场景**：
     - 利用 List 结构的 `LPUSH` 和 `BRPOP` (阻塞式弹出) 命令，可以实现简单的生产者-消费者模式，用于异步任务处理或服务解耦。Redis 5.0 后的 Stream 提供了更专业的消息队列功能

4. **排行榜 (Leaderboard)**

   - **场景**：
     - 利用 Sorted Set (ZSet) 自动排序的特性。ZSet 中的每个成员都关联一个分数 (score)，非常适合实现游戏积分榜、热门商品榜、贡献榜等

5. **分布式锁 (Distributed Lock)**

   - **场景**：在分布式系统中，利用 `SETNX` (SET if Not eXists) 命令的特性，确保在多个服务实例中，同一时间只有一个实例可以执行某个临界区代码

6. **会话存储 (Session Store)**

   - **场景**：存储 Web 应用的用户登录状态（Session），实现分布式会话共享



## 2. NoSQL 数据库概述

### 什么是 NoSQL？

- NoSQL 的意思是 **"Not Only SQL"** (不仅仅是 SQL)，它是对所有 **非关系型数据库** 的统称

### NoSQL 与 SQL 的对比

- 为了理解 NoSQL 的价值，我们通常将其与传统的 SQL 数据库（如 MySQL, SQL Server）进行对比

- 先说说二者的适用场景
  - **SQL** 适用于需要高度一致性、结构化数据和复杂事务查询的场景（如金融、ERP 系统）
  - **NoSQL** 适用于需要高吞吐量、高扩展性、灵活数据结构和海量数据的场景（如社交网络、物联网、大数据分析）
    - Redis 是 NoSQL 家族中高性能键值存储的杰出代表



#### 数据模型

##### SQL

- 结构化数据。基于**表 (Table)** 和行 (Row)，数据之间通过外键建立严格的关系

##### NoSQL

- 多样化模型
  - 例如：
    - **键值 (Key-Value)** (如 Redis)
    - **文档 (Document)** (如 MongoDB)
    - **列簇 (Wide-Column)** (如 Cassandra)
    - **图 (Graph)** (如 Neo4j)



#### Schema(结构)

##### SQL

- **强 Schema** (Schema-on-Write)
  - 数据在写入前，必须严格符合预先定义的表结构



##### NoSQL

- **动态 Schema** (Schema-on-Read)
  - 数据结构灵活，可以随时添加新字段，不需要预先定义



#### 事务

##### SQL

- **严格支持 ACID** (原子性, 一致性, 隔离性, 持久性)
  - 优先保证数据一致性



##### NoSQL

- **放宽 ACID 要求**
  - 通常只保证单文档/单键的原子性。转而追求 **BASE 理论** (基本可用, 软状态, 最终一致性)



#### 扩展性

##### SQL

- 通常优先**垂直扩展** (Scale-Up)，即提升单台服务器的硬件性能
  - **水平扩展** (Scale-Out) 相对复杂（如分库分表）



##### NoSQL

-  设计之初就为**水平扩展** (Scale-Out) 而生
  - 可以更容易地将数据分散到多台服务器上，构建大规模集群



## 3. 环境搭建

### Windows

### Docker



## 4. Redis 核心概念

### Redis 为什么快

Redis 的主要网络服务（命令处理）是基于**单线程模型**构建的

> 但这并不意味着 Redis 内部完全没有其他线程（例如，后台的持久化、异步删除会使用辅助线程）

- “单线程”是指 Redis 服务器使用**一个主线程**来接收客户端连接、处理命令请求、读取和写入数据

  - 之所以采用单线程模型仍能实现极高性能，主要有三个原因：

    1. **纯内存操作**：Redis 的所有数据都存储在内存中。内存的读写速度远快于磁盘，这消除了大部分性能瓶颈

    2. **I/O 多路复用 (I/O Multiplexing)**：

       - 这是关键。单线程不意味着“一个一个排队处理”

         Redis 使用了 I/O 多路复用技术（如 Linux 上的 `epoll`），使其能够**同时监听**多个客户端连接

         当某个连接准备好（例如，数据已到达），该线程才去处理它。它避免了为每个连接创建一个线程的开销，也避免了在等待 I/O 时发生阻塞

    3. **避免线程切换和锁开销**：由于只有一个主线程，就不存在多线程环境下的几个主要性能问题：

       - **上下文切换 (Context Switching)**：多线程在 CPU 核心上切换执行时会产生性能消耗
       - **锁竞争 (Lock Contention)**：多线程访问共享资源时需要加锁，这会引入复杂性和性能开销

> **总结**：Redis 的高性能，本质上是**内存存储**和**高效的 I/O 模型**共同作用的结果



### 键和值

> Key and Value

Redis 是一个**键值存储系统**

- **键**

  - 键是 Redis 中数据的唯一标识符

  - **Redis** 的键是**二进制安全**的字符串

    > 这意味着你可以使用任何二进制序列作为键，例如一个普通的 ASCII 字符串 "user:1000"，或者一个 JPEG 图像的二进制内容

  - 不过，为了便于管理和阅读，键通常使用可读的字符串来表示

- **值**

  - 值是与键相关联的数据

  - 值同样是二进制安全的

  - Redis 的“值”并不仅限于简单的字符串，而是支持多种复杂的数据结构，如：

    - 字符串 (String)
    - 哈希 (Hash)
    - 列表 (List)
    - 集合 (Set)
    - 有序集合 (Sorted Set)

    这就是 Redis 被称为“远程字典**服务**”而不是“远程字符串服务”的原因



### 键的良好命名规范

良好的键命名是 Redis 可维护性的重要组成部分

**最佳实践：**

1. **使用统一的格式和分隔符**

   - 推荐使用冒号 (`:`) 作为分隔符，将键的层级结构或不同部分分开
   - **格式**：`object-type:id:field`
   - **示例**：
     - 存储用户 ID 为 1000 的信息：`user:1000:profile`
     - 存储文章 ID 为 985 的评论：`post:985:comments`
     - 存储某日活跃用户：`stats:20251102:daily_active_users`

   

2. **保持描述性和简洁性**

   - **描述性**：键名应能清晰地反映其存储的内容，例如 `u:1000` 不如 `user:1000` 清晰
   - **简洁性**：键名也不应过长
     - Redis 的键和值都会占用内存。例如 `user:1000:profile:bio` 就比 `user_profile_for_user_with_id_1000_bio` 要好

   

3. **遵循统一的大小写规范**

   - Redis 的键是区分大小写的（`user:1000` 和 `User:1000` 是两个不同的键）
   - 建议全部使用小写字母，以避免混淆



### 大小写差异

#### 命令不区分大小写

Redis 的命令是**不区分大小写**的，以下写法完全等效：

- `GET key`

- `get key`

- `Get key`

- `gEt key`

不过按照惯例，文档和示例中通常使用**大写**来表示命令，以便于阅读和区分



#### **键区分大小写**

Redis 的键名是**严格区分大小写**的：

```bash
SET mykey "value1"
SET MyKey "value2"
SET MYKEY "value3"
```

- 上面的命令会创建三个**不同的键**



#### **值区分大小写**

存储的数据值也是**区分大小写**的，会完全按照你存储的内容保留：

```bash
SET name "Alice"
GET name  # 返回 "Alice"，不是 "alice"
```



## 5. Redis 多数据库

### 1. 核心概念: 一个实例, 多个库

- **一个 Redis 实例：** 

  - 当您启动一个 Redis 服务时，您就启动了一个 Redis 实例

    这个实例会占用操作系统的内存、CPU 等物理资源

- **16 个数据库：** 

  - 在这一个单独的实例内部，Redis 默认会创建 16 个逻辑上的数据库

- **编号：**

  - 这些数据库的编号从 `0` 到 `15`



### 2. 默认行为

- **默认连接 DB 0：** 
  - 任何客户端（如 `redis-cli` 或代码中的库）连接到 Redis 实例时，默认会自动选择 `0` 号数据库



### 3. 修改数据库数量 (配置)

- **可以修改：** 默认的 16 个数据库这个数量是可以修改的

- **如何修改：** 在 Redis 配置文件 `redis.conf` 中，找到 `databases 16` 这一行

- **修改配置：** 将 `16` 修改为您想要的数量（例如 `databases 32`）

- **重启生效：** 修改配置后，必须**重启 Redis 服务**才能生效

- **注意事项：** 

  - 此修改仅在独立实例或哨兵模式下有效

    并且，增加数量并不能解决"资源不隔离"问题



### 4. 切换数据库：`SELECT` 命令

- 您可以使用 `SELECT <db_index>` 命令来切换当前连接正在操作的数据库
- **示例：**
  - `SELECT 0`：切换到 0 号数据库 ( 默认 )
  - `SELECT 15`：切换到 15 号数据库
- **注意：** `SELECT` 命令只对当前客户端的**当前连接**生效。其他客户端连接不受影响，新建立的连接也仍然默认连接到 DB 0



### 5. 关键特性

#### A. 命名空间隔离 (数据隔离)

- **含义：** 不同数据库中的 Key 是**完全隔离**的

- **示例：**

  - 可以在 `DB 0` 中设置一个 `SET mykey "hello"`,

    然后切换到 `DB 7`（使用 `SELECT 7`），再设置一个 `SET mykey "world"`,

    这两个 `mykey` 是**不同**的数据，互不干扰.

    在 `DB 0` 中 `GET mykey` 得到的是 "hello"，在 `DB 7` 中 `GET mykey` 得到的是 "world"



#### B. 资源不隔离 (共享资源)

- **⭐：** 所有的数据库**共享**同一个 Redis 实例的所有物理资源

- **共享内存：** 

  - 如果 `DB 0` 存了 10GB 数据，`DB 1` 存了 2GB 数据，那么这个 Redis 实例总共就占用了 12GB 内存（加上一些额外开销）

- **共享 CPU：** 

  - Redis 主要是单线程处理命令。如果 `DB 0` 上的一个慢查询（如 `KEYS *`）阻塞了 Redis 主线程，

    那么**所有其他数据库**上的操作也会被同时阻塞

- **共享配置：**

  - `maxmemory`（最大内存限制）是针对整个实例的，而不是平分给每个数据库的



## 6. 例外: 集群模式

> Redis Cluster

- 如果您使用的是 Redis Cluster（哨兵模式 Sentinel 不是集群模式），那么**不支持**多数据库这个概念
- 在集群模式下，**只有一个 `DB 0`**
- 如果您在集群模式下尝试执行 `SELECT` 命令，会收到一个错误



## 7. 生产环境使用建议

- **不推荐使用多数据库来隔离应用**

- **原因：** 

  - 正是因为"资源不隔离"

    如果一个"不重要"的应用（在 `DB 1`）占用了所有内存或阻塞了线程，它会严重影响到您的"核心应用"（在 `DB 0`）

- **最佳实践：** 

  - 为每个需要隔离的应用 (如开发、测试、生产环境，或 App1、App2等) 启动 **各自独立 的 Redis 实例** (使用不同的端口或不同的服务器)

    这样就可以做到真正的资源隔离



## 8. 其它概念

### a. 用户名

**Redis 6.0 之前**

- ❌ **没有用户名**,只有密码
- 所有客户端使用同一个密码连接
- 所有连接拥有相同的权限

**Redis 6.0 及以后**

- ✅ **支持多用户(ACL系统)**
- 每个用户有独立的用户名和密码
- 可以为不同用户设置不同的权限



### b. 内存特性

#### 1. 默认内存大小

- **Linux系统**: 默认**无限制**,可以使用系统所有可用内存
- **Windows系统**: 默认也是无限制(但Windows版本是非官方的)
- **没有预设的固定大小**



#### 2. 内存上限配置

**可以通过配置设置上限:**

```bash
# 配置文件 redis.conf
maxmemory 2gb

# 或运行时设置
CONFIG SET maxmemory 2147483648  # 单位是字节
```

**查看当前配置:**

```bash
CONFIG GET maxmemory
# 返回 0 表示无限制
```

**常用单位:**

- `1k` → 1000 字节
- `1kb` → 1024 字节
- `1m` → 1000000 字节
- `1mb` → 1024*1024 字节
- `1g` → 1000000000 字节
- `1gb` → 1024*1024*1024 字节



#### 3. 内存分配方式

**动态分配,非预分配:**

- Redis 启动时**不会预先分配内存**
- 随着数据写入**动态增长**
- 删除数据后,内存**不一定立即释放给操作系统**(取决于内存分配器)

**查看内存使用情况:**

```bash
INFO memory
```

关键指标:

- `used_memory`: Redis实际使用的内存
- `used_memory_rss`: 操作系统分配给Redis的物理内存
- `maxmemory`: 配置的最大内存限制





# Redis 核心数据结构与常用命令

- ⭐注：无论值是什么类型（String、List、Hash、Set、Sorted Set），**键本身始终是字符串**
  - 下面所说的所有数据结构，指的**都是“值”的数据结构**



## X--通用命令

- 通用命令适用于**所有数据类型**（String、Hash、List、Set、ZSet 等）



#### 1. Key 的基本操作

| 命令                   | 功能                                   | 示例                      |
| ---------------------- | -------------------------------------- | ------------------------- |
| `DEL key [key ...]`    | 删除一个或多个 key                     | `DEL user:1000`           |
| `EXISTS key [key ...]` | 检查 key 是否存在                      | `EXISTS user:1000`        |
| `TYPE key`             | 查看 key 的数据类型                    | `TYPE user:1000` → `hash` |
| `RENAME key newkey`    | 重命名 key（覆盖，原来的`newkey`消失） | `RENAME oldkey newkey`    |
| `RENAMENX key newkey`  | 重命名 key（`newkey`不存在时才执行）   | `RENAMENX oldkey newkey`  |



#### 2. Key 的过期时间管理

| 命令                       | 功能                     | 示例                          |
| -------------------------- | ------------------------ | ----------------------------- |
| `EXPIRE key seconds`       | 设置过期时间（秒）       | `EXPIRE session:123 3600`     |
| `EXPIREAT key timestamp`   | 设置过期时间戳（秒）     | `EXPIREAT key 1699999999`     |
| `PEXPIRE key milliseconds` | 设置过期时间（毫秒）     | `PEXPIRE key 60000`           |
| `PEXPIREAT key timestamp`  | 设置过期时间戳（毫秒）   | `PEXPIREAT key 1699999999000` |
| `TTL key`                  | 查看剩余生存时间（秒）   | `TTL session:123` → `3599`    |
| `PTTL key`                 | 查看剩余生存时间（毫秒） | `PTTL session:123`            |
| `PERSIST key`              | 移除过期时间（永久）     | `PERSIST session:123`         |

- ##### TTL 返回值说明

  - 正数：剩余秒数

  - `-1`：key 存在但没有设置过期时间

  - `-2`：key 不存在



#### 3. Key 的查找和遍历

| 命令                                        | 功能               | 示例                            |
| ------------------------------------------- | ------------------ | ------------------------------- |
| `KEYS pattern`                              | 查找匹配模式的 key | `KEYS user:*`                   |
| `SCAN cursor [MATCH pattern] [COUNT count]` | 迭代 key（推荐）   | `SCAN 0 MATCH user:* COUNT 100` |
| `RANDOMKEY`                                 | 随机返回一个 key   | `RANDOMKEY`                     |

- ⚠️ **注意**：`KEYS` 会阻塞 Redis，**生产环境禁用**，应使用 `SCAN` 代替



##### SCAN 示例

```bash
# 第一次扫描
SCAN 0 MATCH user:* COUNT 10
# 返回: 1) "17"  2) 1) "user:100" 2) "user:200" ...

# 继续扫描（使用返回的游标）
SCAN 17 MATCH user:* COUNT 10
```



#### 4. Key 的其他操作

| 命令                            | 功能                  | 示例                           |
| ------------------------------- | --------------------- | ------------------------------ |
| `DUMP key`                      | 序列化 key 的值       | `DUMP user:1000`               |
| `RESTORE key ttl value`         | 反序列化并创建 key    | `RESTORE newkey 0 "\x00\x..."` |
| `MOVE key db`                   | 移动 key 到其他数据库 | `MOVE mykey 1`                 |
| `TOUCH key [key ...]`           | 更新最后访问时间      | `TOUCH user:1000`              |
| `OBJECT subcommand [arguments]` | 查看 key 的内部信息   | `OBJECT ENCODING mykey`        |



#### 5. 数据库操作

| 命令           | 功能                | 示例       |
| -------------- | ------------------- | ---------- |
| `SELECT index` | 切换数据库          | `SELECT 1` |
| `DBSIZE`       | 当前数据库 key 数量 | `DBSIZE`   |
| `FLUSHDB`      | 清空当前数据库      | `FLUSHDB`  |
| `FLUSHALL`     | 清空所有数据库      | `FLUSHALL` |



#### 6. 实用示例

```bash
# 创建一个带过期时间的 hash
HSET session:abc123 user_id 1000 login_time "2025-11-06"
EXPIRE session:abc123 1800  # 30分钟后过期

# 检查 key 是否存在
EXISTS session:abc123  # 返回: 1

# 查看剩余时间
TTL session:abc123  # 返回: 1795

# 查看数据类型
TYPE session:abc123  # 返回: hash

# 移除过期时间，变成永久
PERSIST session:abc123

# 重命名
RENAME session:abc123 session:xyz789

# 删除
DEL session:xyz789
```



#### 7. 常用模式匹配

```bash
KEYS *           # 所有 key（危险！）
KEYS user:*      # user: 开头
KEYS user:?00    # user:100, user:200 等
KEYS user:[12]*  # user:1xxx 或 user:2xxx
```

**最佳实践**：

- ✅ 使用 `SCAN` 代替 `KEYS`
- ✅ 为 key 设置合理的过期时间
- ✅ 使用命名规范（如 `类型:ID:字段`）
- ❌ 避免在生产环境使用 `KEYS`
- ❌ 慎用 `FLUSHDB`/`FLUSHALL`



## 1. 字符串 (String)

### 简介

- 字符串 (String) 是 Redis 中最基本的数据类型。它可以存储任何形式的二进制安全数据，包括：
  - 普通的字符串

  - JSON 格式的字符串

  - 序列化后的对象

  - 数字（整数或浮点数）

  - 二进制数据（如图像或音频文件内容）

  - 一个 String 类型的键，其值最大可以存储 **512MB** 的数据



### 常用命令

- 以下是 String 类型的核心命令，`key` 代表键名，`value` 代表值



#### 基础操作

- `SET <key> <value>`

  - **作用**：

    - **设置一个键值对**

      **如果 `key` 已经存在，`SET` 会覆盖旧的值**

  - **代码示例**：

    ```bash
    # 设置用户名为 alice
    127.0.0.1:6379> SET user:1000:username "alice"
    OK
    
    # 覆盖旧值
    127.0.0.1:6379> SET user:1000:username "alice_new"
    OK
    ```



- `GET <key>`

  - **作用**：获取指定 `key` 的值。如果 `key` 不存在，返回 `nil` (空)

  - **代码示例**：

    ```bash
    # 获取用户名
    127.0.0.1:6379> GET user:1000:username
    "alice_new"
    
    # 获取不存在的键，返回 nil
    127.0.0.1:6379> GET nonexistent_key
    (nil)
    ```



#### 高级设置

- `SETEX <key> <seconds> <value>`

  > SET with EXpiration

  - **作用**：设置一个带**过期时间**（秒）的键值对。这是一个原子操作

  - **场景**：常用于缓存和会话管理

  - **代码示例**：

    ```bash
    # 设置 token，1小时后过期
    127.0.0.1:6379> SETEX auth:token:xyz 3600 "user_session_data"
    OK
    
    # 查看剩余过期时间
    127.0.0.1:6379> TTL auth:token:xyz
    (integer) 3598  # 剩余秒数
    ```



- `SETNX <key> <value>`

  > SET if Not eXists

  - **作用**：

    - 当且仅当 `key` **不存在**时，才设置该 `key` 的值

      如果 `key` 已存在，则此命令不执行任何操作

  - **返回值**：设置成功返回 `1`，设置失败（已存在）返回 `0`

  - **场景**：常用于实现**分布式锁**

  - **代码示例**：

    ```bash
    # 第一次设置，成功
    127.0.0.1:6379> SETNX lock:order:1001 "process_1"
    (integer) 1  # 返回1表示设置成功
    
    # 第二次设置，失败（key已存在）
    127.0.0.1:6379> SETNX lock:order:1001 "process_2"
    (integer) 0  # 返回0表示key已存在，设置失败
    ```



- `SET <key> <value> [EX seconds|PX milliseconds] [NX|XX]`

  - **作用**：`SET` 命令的增强版本，支持多种选项组合

    - `EX seconds`：设置过期时间（秒）
    - `PX milliseconds`：设置过期时间（毫秒）
    - `NX`：仅当 key 不存在时才设置（等同于 SETNX）
    - `XX`：仅当 key 已存在时才设置

  - **代码示例**：

    ```bash
    # 设置分布式锁，30秒后自动释放
    127.0.0.1:6379> SET lock:payment EX 30 NX "locked"
    OK
    
    # NX选项：key已存在，设置失败
    127.0.0.1:6379> SET lock:payment EX 30 NX "locked2"
    (nil)  # 返回nil表示设置失败
    
    # XX选项：仅当key存在时才更新
    127.0.0.1:6379> SET cache:data XX "new_value"
    (nil)  # key不存在，设置失败
    ```



#### 原子计数

- 当 String 类型的值是一个可以被解释为整数的字符串时，可以使用原子增减命令

  - `INCR <key>` (INCRement)

    - **作用**：将 `key` 中存储的数字值**原子性地加 1**。如果 `key` 不存在，会先初始化为 `0` 再执行加 1

    - **代码示例**：

      ```bash
      # 文章123的阅读量+1（key不存在，初始化为0再+1）
      127.0.0.1:6379> INCR post:123:views
      (integer) 1
      
      # 再次+1
      127.0.0.1:6379> INCR post:123:views
      (integer) 2
      
      # 查看当前值
      127.0.0.1:6379> GET post:123:views
      "2"
      ```

  

  - `DECR <key>`

    >DECRement

    - **作用**：将 `key` 中存储的数字值**原子性地减 1**

    - **代码示例**：
    
      ```bash
      # 设置商品99的库存为100
      127.0.0.1:6379> SET stock:item:99 100
      OK
      
      # 商品99的库存-1
      127.0.0.1:6379> DECR stock:item:99
      (integer) 99
      ```

  

  - `INCRBY <key> <increment>`

    - **作用**：将 `key` 的值原子性地增加指定的 `increment` (增量)

    - **代码示例**：
  
      ```bash
      # 设置用户1000的分数为50
      127.0.0.1:6379> SET user:1000:score 50
      OK
      
      # 用户1000的分数+10
      127.0.0.1:6379> INCRBY user:1000:score 10
      (integer) 60
      ```

  

  - `DECRBY <key> <decrement>`

    - **作用**：将 `key` 的值原子性地减少指定的 `decrement` (减量)

    - **代码示例**：
  
      ```bash
      # 商品99的库存-5
      127.0.0.1:6379> DECRBY stock:item:99 5
      (integer) 94
      ```

  

  - `INCRBYFLOAT <key> <increment>`

    - **作用**：将 `key` 的值原子性地增加指定的浮点数增量

    - **代码示例**：
  
      ```bash
      # 设置商品价格
      127.0.0.1:6379> SET product:price 99.99
      OK
      
      # 价格上涨0.01
      127.0.0.1:6379> INCRBYFLOAT product:price 0.01
      "100"
      
      # 价格下降5.50
      127.0.0.1:6379> INCRBYFLOAT product:price -5.50
      "94.5"
      ```



#### 批量操作

- `MSET <key1> <value1> [<key2> <value2> ...]`

  >Multiple SET

  - **作用**：一次性设置一个或多个键值对。这是一个原子操作

  - **代码示例**：

    ```bash
    # 批量设置用户1和用户2的姓名
    127.0.0.1:6379> MSET user:1:name "Alice" user:2:name "Bob"
    OK
    
    # 批量设置多个用户信息
    127.0.0.1:6379> MSET user:1:age "25" user:1:city "Beijing" user:2:age "30"
    OK
    ```

- `MGET <key1> [<key2> ...]`

  >Multiple GET

  - **作用**：一次性获取一个或多个 `key` 的值

  - **代码示例**：

    ```bash
    # 批量获取用户1和用户2的姓名
    127.0.0.1:6379> MGET user:1:name user:2:name
    1) "Alice"
    2) "Bob"
    
    # 批量获取用户1的多个字段
    127.0.0.1:6379> MGET user:1:name user:1:age user:1:city
    1) "Alice"
    2) "25"
    3) "Beijing"
    ```



#### 其他操作

- `APPEND <key> <value>`

  - **作用**：如果 `key` 存在并且是一个字符串，`APPEND` 会将 `value` 追加到该 `key` 原有值的末尾

  - **代码示例**：

    ```bash
    # 设置初始值
    127.0.0.1:6379> SET mykey "Hello"
    OK
    
    # 追加字符串
    127.0.0.1:6379> APPEND mykey " World"
    (integer) 11  # 返回追加后的字符串长度
    
    # 查看结果
    127.0.0.1:6379> GET mykey
    "Hello World"
    ```



- `STRLEN <key>`

  >STRing LENgth

  - **作用**：返回 `key` 所存储的字符串值的字节长度

  - **代码示例**：
  
    ```bash
    # 获取字符串长度
    127.0.0.1:6379> STRLEN mykey
    (integer) 11
    
    # key不存在时返回0
    127.0.0.1:6379> STRLEN nonexistent_key
    (integer) 0
    ```



- `GETSET <key> <value>`

  - **作用**：设置 `key` 的新值，并返回 `key` 的旧值。这是一个原子操作

  - **场景**：常用于需要获取旧值同时更新新值的场景

  - **代码示例**：

    ```bash
    # 设置计数器初始值
    127.0.0.1:6379> SET counter 100
    OK
    
    # 重置计数器为0，同时返回旧值
    127.0.0.1:6379> GETSET counter 0
    "100"  # 返回旧值100
    
    # 查看新值
    127.0.0.1:6379> GET counter
    "0"
    ```



### 应用场景

1. **缓存（页面、对象）**

   - **描述**：

     - 这是 String 最广泛的用途

       将需要频繁读取的数据（如用户信息、商品详情）序列化为 JSON 字符串或特定格式，并存储在 Redis 中（通常使用 `SETEX` 设置一个过期时间）

   - **代码示例**：

     ```bash
     # 缓存商品信息，5分钟过期
     127.0.0.1:6379> SETEX cache:product:1001 300 "{\"id\":1001,\"name\":\"iPhone\",\"price\":5999}"
     OK
     
     # 获取缓存的商品信息
     127.0.0.1:6379> GET cache:product:1001
     "{\"id\":1001,\"name\":\"iPhone\",\"price\":5999}"
     
     # 5分钟后自动删除
     127.0.0.1:6379> TTL cache:product:1001
     (integer) -2  # -2表示key已过期
     ```

   

2. **计数器**

   - **描述**：利用 `INCR` 系列命令的原子性，安全地实现高并发场景下的计数功能

   - **代码示例**：

     ```bash
     # 网站总访问量+1
     127.0.0.1:6379> INCR page:view:count
     (integer) 1
     127.0.0.1:6379> INCR page:view:count
     (integer) 2
     
     # 用户1的点赞数+1
     127.0.0.1:6379> INCR user:1:like_count
     (integer) 1
     
     # 文章阅读量+1
     127.0.0.1:6379> INCR article:1001:views
     (integer) 1
     
     # 取消点赞
     127.0.0.1:6379> DECR user:1:like_count
     (integer) 0
     ```

   

3. **分布式锁**

   - **描述**：

     - 利用 `SETNX` 命令（或者 `SET key value EX seconds NX` 复合命令），确保在分布式环境中，多个服务节点只有一个能成功设置某个特定的 `key`，从而获得执行临界区代码的"锁"

   - **代码示例**：

     ```bash
     # 进程1尝试获取订单1的支付锁，并设置60秒超时
     127.0.0.1:6379> SET lock:payment:order1 "process_id_1" EX 60 NX
     OK  # 成功获取锁
     
     # 进程2尝试获取同一个锁
     127.0.0.1:6379> SET lock:payment:order1 "process_id_2" EX 60 NX
     (nil)  # 获取失败，锁已被占用
     
     # 处理完成后释放锁
     127.0.0.1:6379> DEL lock:payment:order1
     (integer) 1# 或者等待60秒自动过期释放
     ```

   

4. **限流（简单场景）**

   - **描述**：利用带过期时间的计数器实现简单的 API 限流

   - **代码示例**：

     ```bash
     # 限制用户1000每分钟最多访问10次
     # 首次访问，创建计数器，60秒后过期
     127.0.0.1:6379> SET rate:limit:user:1000 0 EX 60 NX
     OK
     
     # 每次访问计数+1
     127.0.0.1:6379> INCR rate:limit:user:1000
     (integer) 1
     
     # 获取当前访问次数
     127.0.0.1:6379> GET rate:limit:user:1000
     "1"
     
     # 如果返回值 > 10，则拒绝请求
     ```



## 2. 哈希 (Hash)

### 简介

- Redis 的哈希是一个 **键值对集合**

  ```tex
  Redis键（字符串）: Hash对象
            			├─> 字段1: 值1
            			├─> 字段2: 值2
            			├─> 字段3: 值3
            			└─> ...
  ```

- 它是一个 String 类型的**键 (Key)** 和**值 (Value)** 之间的映射表

  特殊之处在于，这个**值 (Value)** 本身也是一个集合，由多个**字段 (Field)** 和**字段值 (Value)** 组成

  - 你可以将其理解为：一个 `Key` 对应了一个"小型 Key-Value 数据库"

    > 在其他编程语言中，这类似于 `Map`、`Dictionary` 或 `Struct` (结构体)



**结构对比：**

- **String 类型**：

  - `Key -> Value`
  - `user:1000:name -> "Alice"`
  - `user:1000:age -> 30`

  

- **Hash 类型**：

  - `Key -> { Field1 -> Value1, Field2 -> Value2, ... }`

    `user:1000 -> { "name" -> "Alice", "age" -> 30, "email" -> "alice@example.com" }`

  

- 使用 Hash 结构的主要优点是 **节省内存** 和 **便于组织**

  - 当存储一个对象（如用户信息）时，使用一个 Hash 键 `user:1000` 显然比使用多个 String 键（如 `user:1000:name`, `user:1000:age`）更规整，并且在内部存储上更紧凑



### 常用命令

- 以下是 Hash 类型的核心命令。`key` 代表主键，`field` 代表字段名，`value` 代表字段值



#### 设置与获取

- `HSET <key> <field> <value>`

  - **作用**：

    - 为一个 `key` 中的指定 `field` 设置 `value`

      如果 `field` 已存在，则覆盖

  - **代码示例**：

    ```bash
    # 为用户1000设置姓名
    127.0.0.1:6379> HSET user:1000 name "Alice"
    (integer) 1  # 返回1表示新字段
    
    # 更新已存在的字段
    127.0.0.1:6379> HSET user:1000 name "Alice Wang"
    (integer) 0  # 返回0表示字段已存在，执行了更新
    ```

  

- `HGET <key> <field>`

  - **作用**：获取 `key` 中指定 `field` 的 `value`

  - **代码示例**：

    ```bash
    # 获取用户1000的姓名
    127.0.0.1:6379> HGET user:1000 name
    "Alice"
    
    # 获取不存在的字段，返回 nil
    127.0.0.1:6379> HGET user:1000 address
    (nil)
    ```

  

- `HSETNX <key> <field> <value>`

  > **HSE**t if **N**ot e**X**ists

  - **作用**：只有在 `field` 不存在时，才会为该字段设置值。如果 `field` 已存在，则不做任何操作

  - **代码示例**：

    ```bash
    # 仅当字段不存在时才设置
    127.0.0.1:6379> HSETNX user:1000 email "alice@example.com"
    (integer) 1  # 返回1表示设置成功
    
    # 字段已存在，设置失败
    127.0.0.1:6379> HSETNX user:1000 email "new@example.com"
    (integer) 0  # 返回0表示字段已存在，未设置
    ```

  

- `HMSET <key> <field1> <value1> [<field2> <value2> ...]`

  > Multiple SET

  - **作用**：一次性为一个 `key` 设置多个 `field-value` 对

  - **代码示例**：

    ```bash
    # 批量设置用户1000的多个字段
    127.0.0.1:6379> HMSET user:1000 age 30 email "alice@example.com" city "Beijing"
    OK
    ```

  

- `HMGET <key> <field1> [<field2> ...]`

  > Multiple GET

  - **作用**：一次性获取 `key` 中一个或多个 `field` 的 `value`

  - **代码示例**：

    ```bash
    # 批量获取用户1000的姓名和年龄
    127.0.0.1:6379> HMGET user:1000 name age
    1) "Alice"
    2) "30"
    
    # 包含不存在的字段
    127.0.0.1:6379> HMGET user:1000 name age address
    1) "Alice"
    2) "30"
    3) (nil)  # 不存在的字段返回nil
    ```

  

- `HGETALL <key>`

  > GET ALL

  - **作用**：获取 `key` 中**所有**的 `field` 和 `value`

  - **返回值**：一个包含字段和值的列表（例如：["name", "Alice", "age", "30"]）

  - **注意**：当 Hash 中字段非常多时（成千上万），`HGETALL` 可能成为性能瓶颈，应谨慎使用

  - **代码示例**：

    ```bash
    # 获取用户1000的所有字段和值
    127.0.0.1:6379> HGETALL user:1000
    1) "name"
    2) "Alice"
    3) "age"
    4) "30"
    5) "email"
    6) "alice@example.com"
    7) "city"
    8) "Beijing"
    ```



#### 其他操作

- `HEXISTS <key> <field>`

  > EXists

  - **作用**：检查 `key` 中指定的 `field` 是否存在

  - **返回值**：存在返回 `1`，不存在返回 `0`

  - **代码示例**：

    ```bash
    # 检查字段是否存在
    127.0.0.1:6379> HEXISTS user:1000 name
    (integer) 1  # 字段存在
    
    127.0.0.1:6379> HEXISTS user:1000 phone
    (integer) 0  # 字段不存在
    ```

  

- `HLEN <key>`

  > LENgth

  - **作用**：获取 `key` 中字段的数量

  - **代码示例**：

    ```bash
    # 获取用户1000有多少个字段
    127.0.0.1:6379> HLEN user:1000
    (integer) 4
    ```

  

- `HKEYS <key>`

  - **作用**：获取 `key` 中所有的 `field` (字段名)

  - **代码示例**：

    ```bash
    # 获取用户1000的所有字段名
    127.0.0.1:6379> HKEYS user:1000
    1) "name"
    2) "age"
    3) "email"
    4) "city"
    ```

  

- `HVALS <key>`

  - **作用**：获取 `key` 中所有的 `value` (字段值)

  - **代码示例**：

    ```bash
    # 获取用户1000的所有字段值
    127.0.0.1:6379> HVALS user:1000
    1) "Alice"
    2) "30"
    3) "alice@example.com"
    4) "Beijing"
    ```

  

- `HDEL <key> <field1> [<field2> ...]`

  > DELete

  - **作用**：删除 `key` 中一个或多个指定的 `field`

  - **代码示例**：

    ```bash
    # 删除用户1000的邮箱字段
    127.0.0.1:6379> HDEL user:1000 email
    (integer) 1  # 返回删除的字段数量
    
    # 批量删除多个字段
    127.0.0.1:6379> HDEL user:1000 city phone
    (integer) 1  # phone不存在，只删除了city
    ```

  

- `HINCRBY <key> <field> <increment>`

  > INCRement BY

  - **作用**：为 `key` 中指定 `field` 的值（必须是数字）**原子性地**增加 `increment`

  - **代码示例**：

    ```bash
    # 设置用户1000的发帖数为20
    127.0.0.1:6379> HSET user:1000 posts 20
    (integer) 1
    
    # 发帖数+1
    127.0.0.1:6379> HINCRBY user:1000 posts 1
    (integer) 21
    
    # 发帖数+5
    127.0.0.1:6379> HINCRBY user:1000 posts 5
    (integer) 26
    
    # 使用负数实现减少
    127.0.0.1:6379> HINCRBY user:1000 posts -3
    (integer) 23
    ```

  

- `HINCRBYFLOAT <key> <field> <increment>`

  > INCRement BY FLOAT

  - **作用**：为 `key` 中指定 `field` 的值原子性地增加浮点数增量

  - **代码示例**：

    ```bash
    # 设置商品价格
    127.0.0.1:6379> HSET product:1001 price 99.99
    (integer) 1
    
    # 价格上涨0.5
    127.0.0.1:6379> HINCRBYFLOAT product:1001 price 0.5
    "100.49"
    
    # 价格下降10.0
    127.0.0.1:6379> HINCRBYFLOAT product:1001 price -10.0
    "90.49"
    ```

  

- `HSTRLEN <key> <field>`

  > STRing LENgth

  - **作用**：获取 `key` 中指定 `field` 的值的字符串长度

  - **代码示例**：

    ```bash
    # 获取用户1000的姓名长度
    127.0.0.1:6379> HSTRLEN user:1000 name
    (integer) 5
    
    # 字段不存在返回0
    127.0.0.1:6379> HSTRLEN user:1000 address
    (integer) 0
    ```



### 应用场景

1. **存储对象信息**

   - **描述**：Hash 是存储对象（如用户信息、商品信息、配置信息）的理想选择

   - **代码示例**：

     ```bash
     # 创建用户1001对象
     127.0.0.1:6379> HSET user:1001 username "bob"
     (integer) 1
     127.0.0.1:6379> HMSET user:1001 email "bob@example.com" age 25 points 100
     OK
     
     # 用户获得积分
     127.0.0.1:6379> HINCRBY user:1001 points 5
     (integer) 105
     
     # 查看用户完整信息
     127.0.0.1:6379> HGETALL user:1001
     1) "username"
     2) "bob"
     3) "email"
     4) "bob@example.com"
     5) "age"
     6) "25"
     7) "points"
     8) "105"
     ```

2. **购物车**

   - **描述**：一个用户的购物车可以用 Hash 结构来存储

     - `cart:{user_id}` 作为 `key`
     - `field` 是**商品 ID** (如 `product:105`)
     - `value` 是该**商品的数量**

   - **代码示例**：

     ```bash
     # 用户1001添加商品105到购物车，数量+1
     127.0.0.1:6379> HINCRBY cart:1001 product:105 1
     (integer) 1
     
     # 再次添加相同商品，数量+1
     127.0.0.1:6379> HINCRBY cart:1001 product:105 1
     (integer) 2
     
     # 添加其他商品
     127.0.0.1:6379> HINCRBY cart:1001 product:208 3
     (integer) 3
     
     # 查看商品105的数量
     127.0.0.1:6379> HGET cart:1001 product:105
     "2"
     
     # 查看购物车所有商品
     127.0.0.1:6379> HGETALL cart:1001
     1) "product:105"
     2) "2"
     3) "product:208"
     4) "3"	
     
     # 删除商品105
     127.0.0.1:6379> HDEL cart:1001 product:105
     (integer) 1
     
     # 清空购物车（删除整个key）
     127.0.0.1:6379> DEL cart:1001
     (integer) 1
     ```

3. **商品详情页缓存**

   - **描述**：存储商品的各项属性，比 JSON 字符串更灵活，可以单独更新某个字段

   - **代码示例**：

     ```bash
     # 缓存商品详情
     127.0.0.1:6379> HMSET product:2001 name "iPhone 15" price 5999 stock 100 category "手机"
     OK
     
     # 仅更新库存
     127.0.0.1:6379> HINCRBY product:2001 stock -1
     (integer) 99
     
     # 获取商品名称和价格
     127.0.0.1:6379> HMGET product:2001 name price
     1) "iPhone 15"
     2) "5999"
     ```

4. **会话状态管理**

   - **描述**：存储用户会话信息，如登录状态、上次活动时间、权限等

   - **代码示例**：

     ```bash
     # 创建会话
     127.0.0.1:6379> HMSET session:abc123 user_id 1001 login_time 1699999999 last_active 1699999999 role "admin"
     OK
     
     # 更新最后活跃时间
     127.0.0.1:6379> HSET session:abc123 last_active 1700000100
     (integer) 0
     
     # 检查会话是否存在
     127.0.0.1:6379> HEXISTS session:abc123 user_id
     (integer) 1
     
     # 获取用户ID和角色
     127.0.0.1:6379> HMGET session:abc123 user_id role
     1) "1001"
     2) "admin"
     ```



## 3. 列表 (List)

### 1. 简介

#### 概念

- Redis 列表 (List) 是一种**有序**的字符串集合，其元素**允许重复**
  - 它按照元素**插入的顺序**进行排序。你可以从列表的头部（左侧）或尾部（右侧）添加或移除元素



#### 底层实现

- 在 Redis 3.2 之前，List 底层是"双向链表"(LinkedList) 和"压缩列表"(ZipList) 的结合
- 在 Redis 3.2 之后，统一升级为"快速列表"(QuickList)，它是由 ZipList 组成的双向链表，是空间和时间效率的平衡



#### 关键特性

1. **有序性**：按插入顺序排序
2. **可重复**：可以存储多个相同的 "hello"
3. **两端操作**：头部和尾部操作的效率都非常高 (O(1))



### 2. 常用命令

- 假设我们操作的键是 `mylist`

#### (1) 添加/推送 (Push)

- `LPUSH key element [element ...]`

  - **描述**：将一个或多个元素**推入**列表的**头部（左侧）**

  - **代码示例**：

    ```bash
    # 从左侧推入单个元素
    127.0.0.1:6379> LPUSH mylist "A"
    (integer) 1  # 返回列表长度
    
    # 从左侧推入多个元素（注意：C、B、A的插入顺序）
    127.0.0.1:6379> LPUSH mylist "C" "B" "A"
    (integer) 4  # 现在列表是 ["A", "B", "C", "A"]
    
    # 查看列表内容
    127.0.0.1:6379> LRANGE mylist 0 -1
    1) "A"
    2) "B"
    3) "C"
    4) "A"
    ```

  

- `RPUSH key element [element ...]`

  - **描述**：将一个或多个元素**推入**列表的**尾部（右侧）**

  - **代码示例**：

    ```bash
    # 创建新列表并从右侧推入
    127.0.0.1:6379> RPUSH mylist2 "A" "B"
    (integer) 2
    
    # 从右侧继续推入
    127.0.0.1:6379> RPUSH mylist2 "X" "Y"
    (integer) 4
    
    # 查看列表内容
    127.0.0.1:6379> LRANGE mylist2 0 -1
    1) "A"
    2) "B"
    3) "X"
    4) "Y"
    ```

  

- `LPUSHX key element [element ...]`

  - **描述**：仅当 `key` 已存在且是列表时，才从左侧推入元素

  - **代码示例**：

    ```bash
    # 列表不存在，推入失败
    127.0.0.1:6379> LPUSHX nonexist "value"
    (integer) 0
    
    # 列表存在，推入成功
    127.0.0.1:6379> LPUSH mylist3 "A"
    (integer) 1
    127.0.0.1:6379> LPUSHX mylist3 "B"
    (integer) 2
    ```

  

- `RPUSHX key element [element ...]`

  - **描述**：仅当 `key` 已存在且是列表时，才从右侧推入元素



#### (2) 弹出/移除 (Pop)

- `LPOP key [count]`

  - **描述**：从列表的**头部（左侧）**弹出一个或多个元素并返回

  - **代码示例**：

    ```bash
    127.0.0.1:6379> RPUSH mylist "A" "B" "C" "D"
    (integer) 4
    
    # 从左侧弹出一个元素
    127.0.0.1:6379> LPOP mylist
    "A"
    
    # 从左侧弹出两个元素（Redis 6.2+）
    127.0.0.1:6379> LPOP mylist 2
    1) "B"
    2) "C"
    
    # 查看剩余元素
    127.0.0.1:6379> LRANGE mylist 0 -1
    1) "D"
    ```

  

- `RPOP key [count]`

  - **描述**：从列表的**尾部（右侧）**弹出一个或多个元素并返回

  - **代码示例**：

    ```bash
    127.0.0.1:6379> RPUSH mylist "A" "B" "C" "D"
    (integer) 4
    
    # 从右侧弹出一个元素
    127.0.0.1:6379> RPOP mylist
    "D"
    
    # 从右侧弹出两个元素
    127.0.0.1:6379> RPOP mylist 2
    1) "C"
    2) "B"
    ```



#### (3) 查看/范围

- `LRANGE key start stop`

  - **描述**：获取列表指定范围内的所有元素。这是**查看**列表内容的主要方式

  - **索引**：`0` 是头部第一个元素，`-1` 是尾部第一个元素

  - **代码示例**：

    ```bash
    127.0.0.1:6379> RPUSH mylist "A" "B" "C" "D" "E"
    (integer) 5
    
    # 获取所有元素
    127.0.0.1:6379> LRANGE mylist 0 -1
    1) "A"
    2) "B"
    3) "C"
    4) "D"
    5) "E"
    
    # 获取前3个元素
    127.0.0.1:6379> LRANGE mylist 0 2
    1) "A"
    2) "B"
    3) "C"
    
    # 获取最后2个元素
    127.0.0.1:6379> LRANGE mylist -2 -1
    1) "D"
    2) "E"
    ```

  

- `LLEN key`

  - **描述**：获取列表的长度（元素的数量）

  - **代码示例**：

    ```bash
    # 获取列表长度
    127.0.0.1:6379> LLEN mylist
    (integer) 5
    
    # 不存在的列表返回0
    127.0.0.1:6379> LLEN nonexist
    (integer) 0
    ```

  

- `LINDEX key index`

  - **描述**：获取指定索引位置的元素（效率较低，O(N)）

  - **代码示例**：

    ```bash
    # 获取索引0的元素（头部元素）
    127.0.0.1:6379> LINDEX mylist 0
    "A"
    
    # 获取索引-1的元素（尾部元素）
    127.0.0.1:6379> LINDEX mylist -1
    "E"
    
    # 获取索引2的元素
    127.0.0.1:6379> LINDEX mylist 2
    "C"
    
    # 索引超出范围返回nil
    127.0.0.1:6379> LINDEX mylist 100
    (nil)
    ```



#### (4) 修改操作

- `LSET key index element`

  - **描述**：设置列表指定索引位置的元素值

  - **代码示例**：

    ```bash
    # 将索引1的元素设置为"X"
    127.0.0.1:6379> LSET mylist 1 "X"
    OK
    
    # 查看修改结果
    127.0.0.1:6379> LRANGE mylist 0 -1
    1) "A"
    2) "X"  # 原来的"B"被替换
    3) "C"
    4) "D"
    5) "E"
    ```

  

- `LINSERT key BEFORE|AFTER pivot element`

  - **描述**：在列表中指定元素（pivot）的前面或后面插入新元素

  - **代码示例**：

    ```bash
    # 在"C"前面插入"B2"
    127.0.0.1:6379> LINSERT mylist BEFORE "C" "B2"
    (integer) 6  # 返回列表长度
    
    # 在"C"后面插入"C2"
    127.0.0.1:6379> LINSERT mylist AFTER "C" "C2"
    (integer) 7
    
    # 查看结果
    127.0.0.1:6379> LRANGE mylist 0 -1
    1) "A"
    2) "X"
    3) "B2"
    4) "C"
    5) "C2"
    6) "D"
    7) "E"
    ```

  

- `LREM key count element`

  - **描述**：从列表中移除指定数量的匹配元素

    - `count > 0`：从头到尾移除 count 个元素
    - `count < 0`：从尾到头移除 count 个元素
    - `count = 0`：移除所有匹配的元素

  - **代码示例**：

    ```bash
    127.0.0.1:6379> RPUSH mylist "A" "B" "A" "C" "A" "D"
    (integer) 6
    
    # 从头部移除2个"A"
    127.0.0.1:6379> LREM mylist 2 "A"
    (integer) 2  
    
    # 返回实际移除的数量
    
    # 查看结果
    127.0.0.1:6379> LRANGE mylist 0 -1
    1) "B"
    2) "C"
    3) "A"
    4) "D"
    
    # 移除所有的"A"
    127.0.0.1:6379> LREM mylist 0 "A"(integer) 1
    ```

  

- `LTRIM key start stop`

  - **描述**：修剪列表，只保留指定范围内的元素，其他元素全部删除

  - **代码示例**：

    ```bash
    127.0.0.1:6379> RPUSH mylist "A" "B" "C" "D" "E" "F"
    (integer) 6
    
    # 只保留索引1到3的元素
    127.0.0.1:6379> LTRIM mylist 1 3
    OK
    
    # 查看结果
    127.0.0.1:6379> LRANGE mylist 0 -1
    1) "B"
    2) "C"
    3) "D"
    ```



#### (5) 阻塞式操作 (Blocking)

- 这是 List 实现消息队列的关键

  - `BLPOP key [key ...] timeout`

    - **描述**：**阻塞式**地从列表**头部（左侧）**弹出一个元素

    - **工作机制**：如果列表为空，`BLPOP` 会使客户端连接**阻塞**（等待），直到 `timeout` (秒) 超时或有新元素被 `LPUSH` 进来

    - **`timeout = 0`** 表示无限期阻塞

    - **代码示例**：

      ```bash
      # 列表不为空，立即返回
      127.0.0.1:6379> RPUSH queue "task1" "task2"
      (integer) 2
      127.0.0.1:6379> BLPOP queue 0
      1) "queue"  # 返回键名
      2) "task1"  # 返回元素值
      
      # 列表为空，阻塞等待（在另一个客户端推入数据后才返回）
      127.0.0.1:6379> BLPOP empty_queue 5
      (nil)  # 5秒后超时，返回nil
      
      # 可以同时监听多个列表
      127.0.0.1:6379> BLPOP queue1 queue2 queue3 10
      # 从第一个非空列表弹出元素
      ```

    

  - `BRPOP key [key ...] timeout`

    - **描述**：**阻塞式**地从列表**尾部（右侧）**弹出一个元素

    - **代码示例**：

      ```bash
      # 从右侧阻塞弹出
      127.0.0.1:6379> RPUSH queue "task1" "task2" "task3"
      (integer) 3
      127.0.0.1:6379> BRPOP queue 0
      1) "queue"
      2) "task3"  # 从尾部弹出
      ```



#### (6) 原子性移动操作

- `RPOPLPUSH source destination`

  - **描述**：原子性地从 source 列表的尾部弹出一个元素，并推入 destination 列表的头部

  - **场景**：实现安全的消息队列（处理中队列）

  - **代码示例**：

    ```bash
    127.0.0.1:6379> RPUSH source "A" "B" "C"
    (integer) 3
    
    # 从source尾部弹出，推入destination头部
    127.0.0.1:6379> RPOPLPUSH source destination
    "C"
    
    # 查看两个列表
    127.0.0.1:6379> LRANGE source 0 -1
    1) "A"
    2) "B"
    127.0.0.1:6379> LRANGE destination 0 -1
    1) "C"
    ```

  

- `BRPOPLPUSH source destination timeout`

  - **描述**：`RPOPLPUSH` 的阻塞版本

  - **代码示例**：

    ```bash
    # 如果source为空，会阻塞等待
    127.0.0.1:6379> BRPOPLPUSH empty_source destination 5
    (nil)  # 5秒后超时
    ```



### 3. 典型应用场景

1. **消息队列 (Message Queue)**

   - **描述**：

     - 这是 List 最经典的应用场景 生产者使用 `LPUSH` 向列表推送任务，一个或多个消费者使用 `BRPOP` (或 `BLPOP`) 等待并获取任务

   - **代码示例**：

     ```bash
     # 生产者：推送任务到队列
     127.0.0.1:6379> LPUSH task_queue "task_1"
     (integer) 1
     127.0.0.1:6379> LPUSH task_queue "task_2"
     (integer) 2
     127.0.0.1:6379> LPUSH task_queue "task_3"
     (integer) 3
     
     # 消费者：阻塞式获取任务（从右侧弹出）
     127.0.0.1:6379> BRPOP task_queue 0
     1) "task_queue"
     2) "task_1"
     
     127.0.0.1:6379> BRPOP task_queue 0
     1) "task_queue"
     2) "task_2"
     
     # 队列为空时，消费者会阻塞等待新任务
     127.0.0.1:6379> BRPOP task_queue 0
     # 等待中...直到有新任务被LPUSH进来
     ```

   - **局限性**：

     - 这是一个简易的消息队列 它不支持"消费组"（一个消息只能被一个消费者处理），也不支持"消息确认"(ACK)

       > Redis 5.0 后的 Stream 类型是更专业的消息队列

   

2. **最新动态列表 (e.g., 朋友圈 Feed)**

   - **描述**：存储用户发布的最新动态

   - **代码示例**：

     ```bash
     # 用户1001发布新动态（总是插入到头部）
     127.0.0.1:6379> LPUSH user:feed:1001 "post_id_123"
     (integer) 1
     127.0.0.1:6379> LPUSH user:feed:1001 "post_id_124"
     (integer) 2
     127.0.0.1:6379> LPUSH user:feed:1001 "post_id_125"
     (integer) 3
     
     # 获取最新的5条动态
     127.0.0.1:6379> LRANGE user:feed:1001 0 4
     1) "post_id_125"  # 最新的
     2) "post_id_124"
     3) "post_id_123"
     
     # 只保留最新的50条动态（删除旧动态）
     127.0.0.1:6379> LTRIM user:feed:1001 0 49
     OK
     ```

   - **优势**：插入快，并且天然保持了时间顺序

   

3. **可靠消息队列（带处理中队列）**

   - **描述**：使用 `RPOPLPUSH` 实现更安全的消息队列，防止消息丢失

   - **代码示例**：

     ```bash
     # 生产者：推送任务
     127.0.0.1:6379> LPUSH task_queue "task_1"
     (integer) 1
     
     # 消费者：从task_queue弹出，同时推入processing队列
     127.0.0.1:6379> RPOPLPUSH task_queue processing
     "task_1"
     
     # 此时task_1在processing队列中，表示正在处理
     127.0.0.1:6379> LRANGE processing 0 -1
     1) "task_1"
     
     # 处理完成后，从processing队列移除
     127.0.0.1:6379> LREM processing 1 "task_1"
     (integer) 1
     
     # 如果处理失败，可以将任务重新推回task_queue
     127.0.0.1:6379> RPOPLPUSH processing task_queue
     "task_1"  # 重新进入待处理队列
     ```

   

4. **文章评论列表**

   - **描述**：存储文章的评论，按时间顺序展示

   - **代码示例**：

     ```bash
     # 用户发表评论（新评论插入头部）
     127.0.0.1:6379> LPUSH article:1001:comments "comment_id_1"
     (integer) 1
     127.0.0.1:6379> LPUSH article:1001:comments "comment_id_2"
     (integer) 2
     
     # 获取最新的10条评论
     127.0.0.1:6379> LRANGE article:1001:comments 0 9
     1) "comment_id_2"
     2) "comment_id_1"
     
     # 获取评论总数
     127.0.0.1:6379> LLEN article:1001:comments
     (integer) 2
     
     # 分页获取评论（第2页，每页10条）
     127.0.0.1:6379> LRANGE article:1001:comments 10 19
     (empty array)
     ```

   

5. **排行榜/时间线（简单场景）**

   - **描述**：按时间顺序记录用户行为或事件

   - **代码示例**：

     ```bash
     # 记录用户登录历史（最多保留100条）
     127.0.0.1:6379> LPUSH user:1001:login_history "2024-11-07 10:30:00"
     (integer) 1
     127.0.0.1:6379> LPUSH user:1001:login_history "2024-11-07 15:20:00"
     (integer) 2
     
     # 只保留最近100条记录
     127.0.0.1:6379> LTRIM user:1001:login_history 0 99
     OK
     
     # 查看最近5次登录
     127.0.0.1:6379> LRANGE user:1001:login_history 0 4
     1) "2024-11-07 15:20:00"
     2) "2024-11-07 10:30:00"
     ```



## 4. 集合 (Set)

### 简介

- Redis 的集合 (Set) 是一个 **无序的**、**不允许重复** 的字符串元素集合

- Set 类型的核心特性是 **唯一性** 和 **无序性**

  - ⭐当你向一个 Set 中添加一个已经存在的元素时，该命令会被忽略，Set 中的成员保持不变

    > 非覆盖

- ⭐注意：

  - Set 的"无序"是指：
    - 不保证插入顺序
    - 不应该依赖任何特定顺序
  - 不是指
    - 每次返回的顺序都随机改变（通常不会）

- 常见场景：

  - 由于 Set 提供了高效的添加、删除、查找以及集合间的运算（交集、并集、差集）能力，

    它非常适合用于去重、标签系统和社交网络中的关系计算



### 常用命令

- 以下是 Set 类型的核心命令。`key` 代表集合的键名，`member` 代表元素



#### 添加、移除与查看

- `SADD <key> <member1> [<member2> ...]`

  > Set ADD

  - **作用**：

    - 向集合 `key` 中添加一个或多个 `member`

      如果 `member` 已存在，则被忽略

  - **代码示例**：

    ```bash
    # 添加文章1的标签
    127.0.0.1:6379> SADD tags:article:1 "redis" "database" "nosql"
    (integer) 3  # 返回成功添加的成员数量
    
    # 尝试添加已存在的成员（会被忽略）
    127.0.0.1:6379> SADD tags:article:1 "redis"
    (integer) 0  # 返回0表示没有新成员被添加
    
    # 查看集合中的所有成员
    127.0.0.1:6379> SMEMBERS tags:article:1
    1) "redis"
    2) "database"
    3) "nosql"
    ```

- `SREM <key> <member1> [<member2> ...]`

  > Set REMove

  - **作用**：从集合 `key` 中移除一个或多个 `member`

  - **代码示例**：

    ```bash
    # 移除单个成员
    127.0.0.1:6379> SREM tags:article:1 "database"
    (integer) 1  # 返回成功移除的成员数量
    
    # 移除多个成员
    127.0.0.1:6379> SREM tags:article:1 "nosql" "cache"
    (integer) 1  # cache不存在，只移除了nosql
    
    # 查看剩余成员
    127.0.0.1:6379> SMEMBERS tags:article:1
    1) "redis"
    ```

- `SMEMBERS <key>`

  - **作用**：获取集合 `key` 中的 **所有** 成员

  - **注意**：返回的成员是无序的。同 `HGETALL` 一样，当集合非常大时，此命令可能阻塞 Redis，应谨慎使用

  - **代码示例**：

    ```bash
    127.0.0.1:6379> SADD myset "a" "b" "c" "d" "e"
    (integer) 5
    
    # 获取所有成员（顺序可能与插入顺序不同）
    127.0.0.1:6379> SMEMBERS myset
    1) "a"
    2) "c"
    3) "e"
    4) "b"
    5) "d"
    ```

- `SPOP <key> [count]`

  > Set POP

  - **作用**：从集合中随机移除并返回一个或多个成员

  - **场景**：适用于抽奖、随机任务分配等场景

  - **代码示例**：

    ```bash
    127.0.0.1:6379> SADD lottery "user1" "user2" "user3" "user4" "user5"
    (integer) 5
    
    # 随机弹出1个成员
    127.0.0.1:6379> SPOP lottery
    "user3"
    
    # 随机弹出2个成员
    127.0.0.1:6379> SPOP lottery 2
    1) "user1"
    2) "user5"
    
    # 查看剩余成员
    127.0.0.1:6379> SMEMBERS lottery
    1) "user2"
    2) "user4"
    ```



#### 判断与统计

- `SISMEMBER <key> <member>`

  > Set Is MEMBER

  - **作用**：判断 `member` 是否是集合 `key` 中的成员

  - **返回值**：是返回 `1`，不是（或 `key` 不存在）返回 `0`

  - **代码示例**：

    ```bash
    127.0.0.1:6379> SADD tags:article:1 "redis" "database"
    (integer) 2
    
    # 判断成员是否存在
    127.0.0.1:6379> SISMEMBER tags:article:1 "redis"
    (integer) 1  # 存在
    
    127.0.0.1:6379> SISMEMBER tags:article:1 "mongodb"
    (integer) 0  # 不存在
    ```

  

- `SMISMEMBER <key> <member1> [<member2> ...]`

  > Set Multiple Is MEMBER

  - **作用**：批量判断多个成员是否在集合中（Redis 6.2+）

  - **返回值**：返回一个数组，每个元素对应一个成员的判断结果（1或0）

  - **代码示例**：

    ```bash
    # 批量判断多个成员
    127.0.0.1:6379> SMISMEMBER tags:article:1 "redis" "mongodb" "database"
    1) (integer) 1  # redis存在
    2) (integer) 0  # mongodb不存在
    3) (integer) 1  # database存在
    ```

  

- `SCARD <key>`

  > Set CARDinality

  - **作用**：获取集合 `key` 的基数（即成员的数量）

  - **代码示例**：

    ```bash
    # 获取集合成员数量
    127.0.0.1:6379> SCARD tags:article:1
    (integer) 2
    
    # 不存在的集合返回0
    127.0.0.1:6379> SCARD nonexist
    (integer) 0
    ```

  

- `SRANDMEMBER <key> [count]`

  > Set RANDom MEMBER

  - **作用**：从集合 `key` 中随机返回一个或多个（由 `count` 决定）成员，但**不删除**它们

  - **代码示例**：

    ```bash
    127.0.0.1:6379> SADD lottery_pool "user1" "user2" "user3" "user4" "user5"
    (integer) 5
    
    # 随机获取1个成员（不删除）
    127.0.0.1:6379> SRANDMEMBER lottery_pool
    "user3"
    
    # 随机获取5个成员
    127.0.0.1:6379> SRANDMEMBER lottery_pool 5
    1) "user1"
    2) "user4"
    3) "user2"
    4) "user5"
    5) "user3"
    
    # count为负数时，可能返回重复元素
    127.0.0.1:6379> SRANDMEMBER lottery_pool -10
    1) "user2"
    2) "user2"
    3) "user5"
    4) "user1"
    # ... 可能有重复
    
    # 集合仍然完整（未删除）
    127.0.0.1:6379> SCARD lottery_pool
    (integer) 5
    ```



#### 移动操作

- `SMOVE <source> <destination> <member>`

  > Set MOVE

  - **作用**：将 `member` 从 `source` 集合移动到 `destination` 集合

  - **返回值**：成功返回 `1`，如果 `member` 不在 `source` 中返回 `0`

  - **代码示例**：

    ```bash
    127.0.0.1:6379> SADD set1 "a" "b" "c"
    (integer) 3
    127.0.0.1:6379> SADD set2 "x" "y"
    (integer) 2
    
    # 将"b"从set1移动到set2
    127.0.0.1:6379> SMOVE set1 set2 "b"
    (integer) 1
    
    # 查看结果
    127.0.0.1:6379> SMEMBERS set1
    1) "a"
    2) "c"
    127.0.0.1:6379> SMEMBERS set2
    1) "b"
    2) "x"
    3) "y"
    ```



#### 集合运算

- 这是 Set 类型的强大功能，用于计算多个集合之间的关系

- `SINTER <key1> [<key2> ...]`

  > Set INTERsection

  - **作用**：返回 **所有给定集合的交集**（共同成员）

    - 如果只写了一个集合，含义为：自己和自己的交集 = 自己

      > 所有给定的集合，只有自己

  - **代码示例**：

    ```bash
    # 用户1的好友
    127.0.0.1:6379> SADD user:1:friends "alice" "bob" "charlie" "david"
    (integer) 4
    
    # 用户2的好友
    127.0.0.1:6379> SADD user:2:friends "bob" "charlie" "eve"
    (integer) 3
    
    # 计算共同好友
    127.0.0.1:6379> SINTER user:1:friends user:2:friends
    1) "bob"
    2) "charlie"
    ```

  

- `SUNION <key1> [<key2> ...]`

  > Set UNION

  - **作用**：返回 **所有给定集合的并集**（所有成员，自动去重）

    - 如果只写了一个集合，含义为：自己和自己的并集 = 自己

      > **所有的给定集合**，只有自己

  - **代码示例**：

    ```bash
    # 查看用户1和用户2的所有好友（去重）
    127.0.0.1:6379> SUNION user:1:friends user:2:friends
    1) "alice"
    2) "bob"
    3) "charlie"
    4) "david"
    5) "eve"
    ```

  

- `SDIFF <key1> [<key2> ...]`

  > Set DIFFerence

  - **作用**：返回 **第一个集合与后续所有集合的差集**（只在第一个集合中，但不在其他集合中的成员）

    - 如果只写了一个集合，含义为：自己减去"无" = 自己

      > "后续所有集合"为"无"

  - **代码示例**：

    ```bash
    # 查看用户1的好友中，哪些不是用户2的好友
    127.0.0.1:6379> SDIFF user:1:friends user:2:friends
    1) "alice"
    2) "david"
    
    # 反过来：用户2独有的好友
    127.0.0.1:6379> SDIFF user:2:friends user:1:friends
    1) "eve"
    ```

  

- `SINTERSTORE <destination> <key1> [<key2> ...]`

  > Set INTERsection STORE

  - **作用**：计算交集并将结果存储到 `destination` 集合中

  - **代码示例**：

    ```bash
    # 计算交集并存储
    127.0.0.1:6379> SINTERSTORE common_friends user:1:friends user:2:friends
    (integer) 2  # 返回交集的成员数量
    
    # 查看存储的结果
    127.0.0.1:6379> SMEMBERS common_friends
    1) "bob"
    2) "charlie"
    ```

  

- `SUNIONSTORE <destination> <key1> [<key2> ...]`

  > Set UNION STORE

  - **作用**：计算并集并将结果存储到 `destination` 集合中

  - **代码示例**：

    ```bash
    # 计算并集并存储
    127.0.0.1:6379> SUNIONSTORE all_friends user:1:friends user:2:friends
    (integer) 5  # 返回并集的成员数量
    
    # 查看存储的结果
    127.0.0.1:6379> SMEMBERS all_friends
    1) "alice"
    2) "bob"
    3) "charlie"
    4) "david"
    5) "eve"
    ```

  

- `SDIFFSTORE <destination> <key1> [<key2> ...]`

  > Set DIFFerence STORE

  - **作用**：计算差集并将结果存储到 `destination` 集合中

  - **代码示例**：

    ```bash
    # 计算差集并存储
    127.0.0.1:6379> SDIFFSTORE unique_friends user:1:friends user:2:friends
    (integer) 2  # 返回差集的成员数量
    
    # 查看存储的结果
    127.0.0.1:6379> SMEMBERS unique_friends
    1) "alice"
    2) "david"
    ```



### 应用场景

1. **标签 (Tags)**

   - **描述**：Set 的唯一性使其非常适合存储标签

   - **代码示例**：

     ```bash
     # 为文章100添加标签
     127.0.0.1:6379> SADD tags:article:100 "Java" "Redis" "Database"
     (integer) 3
     
     # 反向索引：记录使用"Redis"标签的所有文章
     127.0.0.1:6379> SADD tag:Redis 100 101 102
     (integer) 3
     
     # 查询文章100的所有标签
     127.0.0.1:6379> SMEMBERS tags:article:100
     1) "Java"
     2) "Redis"
     3) "Database"
     
     # 查询所有带"Redis"标签的文章
     127.0.0.1:6379> SMEMBERS tag:Redis
     1) "100"
     2) "101"
     3) "102"
     
     # 找出同时具有"Redis"和"Java"标签的文章
     127.0.0.1:6379> SADD tag:Java 100 103 104
     (integer) 3
     127.0.0.1:6379> SINTER tag:Redis tag:Java
     1) "100"
     ```

   

2. **共同好友/共同关注**

   - **描述**：利用 `SINTER` 命令，可以极快地计算出两个用户的共同好友列表

   - **代码示例**：

     ```bash
     # 用户1关注的人
     127.0.0.1:6379> SADD user:1:following "userA" "userB" "userC"
     (integer) 3
     
     # 用户2关注的人
     127.0.0.1:6379> SADD user:2:following "userC" "userD"
     (integer) 2
     
     # 计算共同关注
     127.0.0.1:6379> SINTER user:1:following user:2:following
     1) "userC"
     
     # 推荐好友：用户1可能认识的人（用户2关注但用户1未关注）
     127.0.0.1:6379> SDIFF user:2:following user:1:following
     1) "userD"
     
     # 统计共同关注数量
     127.0.0.1:6379> SINTERSTORE temp user:1:following user:2:following
     (integer) 1
     127.0.0.1:6379> SCARD temp
     (integer) 1
     ```

   

3. **抽奖（去重）**

   - **描述**：如果一个活动中，一个用户只能中奖一次，可以使用 `SADD` 自动去重

   - **代码示例**：

     ```bash
     # 用户参与抽奖（自动去重）
     127.0.0.1:6379> SADD activity:101:participants "user_888" "user_777" "user_666"
     (integer) 3
     
     # 用户888重复参与（被自动忽略）
     127.0.0.1:6379> SADD activity:101:participants "user_888"
     (integer) 0  # 返回0表示该用户已参与
     
     # 随机抽取10个中奖用户（不重复）
     127.0.0.1:6379> SRANDMEMBER activity:101:participants 10
     1) "user_888"
     2) "user_777"
     3) "user_666"
     
     # 将中奖用户移动到获奖名单（确保不会重复中奖）
     127.0.0.1:6379> SPOP activity:101:participants 2
     1) "user_777"
     2) "user_666"
     
     # 记录中奖用户
     127.0.0.1:6379> SADD activity:101:winners "user_777" "user_666"
     (integer) 2
     
     # 检查用户是否已中奖
     127.0.0.1:6379> SISMEMBER activity:101:winners "user_888"
     (integer) 0  # 未中奖
     127.0.0.1:6379> SISMEMBER activity:101:winners "user_777"
     (integer) 1  # 已中奖
     ```

   

4. **用户签到/打卡**

   - **描述**：记录每天签到的用户，自动去重

   - **代码示例**：

     ```bash
     # 用户1001在2024-11-07签到
     127.0.0.1:6379> SADD checkin:2024-11-07 1001
     (integer) 1
     
     # 用户1001重复签到（被忽略）
     127.0.0.1:6379> SADD checkin:2024-11-07 1001
     (integer) 0
     
     # 更多用户签到
     127.0.0.1:6379> SADD checkin:2024-11-07 1002 1003 1004
     (integer) 3
     
     # 统计今天签到人数
     127.0.0.1:6379> SCARD checkin:2024-11-07
     (integer) 4
     
     # 检查用户1001是否签到
     127.0.0.1:6379> SISMEMBER checkin:2024-11-07 1001
     (integer) 1
     
     # 计算连续两天都签到的用户
     127.0.0.1:6379> SADD checkin:2024-11-08 1001 1003 1005
     (integer) 3
     127.0.0.1:6379> SINTER checkin:2024-11-07 checkin:2024-11-08
     1) "1001"
     2) "1003"
     ```

   

5. **黑名单/白名单**

   - **描述**：快速判断用户是否在黑名单或白名单中

   - **代码示例**：

     ```bash
     # 添加用户到黑名单
     127.0.0.1:6379> SADD blacklist:ip "192.168.1.100" "10.0.0.50"
     (integer) 2
     
     # 检查IP是否在黑名单中
     127.0.0.1:6379> SISMEMBER blacklist:ip "192.168.1.100"
     (integer) 1  # 在黑名单中
     
     127.0.0.1:6379> SISMEMBER blacklist:ip "192.168.1.200"
     (integer) 0  # 不在黑名单中
     
     # 从黑名单移除
     127.0.0.1:6379> SREM blacklist:ip "192.168.1.100"
     (integer) 1
     
     # 查看黑名单中的所有IP
     127.0.0.1:6379> SMEMBERS blacklist:ip
     1) "10.0.0.50"
     ```

   

6. **权限管理**

   - **描述**：存储用户拥有的权限集合

   - **代码示例**：

     ```bash
     # 为用户1001分配权限
     127.0.0.1:6379> SADD user:1001:permissions "read" "write" "delete"
     (integer) 3
     
     # 检查用户是否有删除权限
     127.0.0.1:6379> SISMEMBER user:1001:permissions "delete"
     (integer) 1  # 有权限
     
     # 检查用户是否有管理员权限
     127.0.0.1:6379> SISMEMBER user:1001:permissions "admin"
     (integer) 0  # 无权限
     
     # 撤销写权限
     127.0.0.1:6379> SREM user:1001:permissions "write"
     (integer) 1
     
     # 查看用户所有权限
     127.0.0.1:6379> SMEMBERS user:1001:permissions
     1) "read"
     2) "delete"
     ```



## 5. 有序集合 (Sorted Set / ZSet)

### 简介

- Redis 的有序集合 (ZSet) 是一种非常强大的数据结构
  - **与 Set (集合) 的共同点**：ZSet 也是一个 **不允许重复** 的字符串元素集合（`member` 唯一）
  - **与 Set (集合) 的不同点**：
    - ZSet 中的每个成员（`member`）**都会关联一个分数 (score)**，这个分数是一个 `double` 类型的浮点数，是一定有的
      - **核心特性**：
        - ZSet 通过这个 `score` 来为集合中的成员**自动进行排序**
        - ZSet 的 **默认排序** 是根据你为每个成员指定的 **`score` (分数)**，进行**升序（从小到大）排列**的
- ZSet 内部实现（跳跃表 SkipList）保证了它在添加、删除、更新成员时，依然能保持 O(log N) 的高效时间复杂度，这使得它在需要动态排序的场景下（如排行榜）表现极其出色



### 常用命令

- 以下是 ZSet 类型的核心命令。`key` 代表 ZSet 键名，`score` 代表分数，`member` 代表成员



#### 添加、移除与增加分数

- `ZADD <key> <score1> <member1> [<score2> <member2> ...]`

  > ZSet ADD

  - **作用**：向有序集合 `key` 中添加一个或多个成员，并指定它们的分数

  - **特性**：如果 `member` 已存在，`ZADD` 会**更新**该成员的 `score`，并重新排序

  - **代码示例**：

    ```bash
    # 创建游戏1的排行榜，添加Alice和Bob
    127.0.0.1:6379> ZADD leaderboard:game1 100 "Alice" 200 "Bob"
    (integer) 2  # 返回成功添加的成员数量
    
    # 更新已存在成员的分数
    127.0.0.1:6379> ZADD leaderboard:game1 150 "Alice"
    (integer) 0  # 返回0表示成员已存在，执行了更新
    
    # 添加更多成员
    127.0.0.1:6379> ZADD leaderboard:game1 300 "Charlie" 250 "David"
    (integer) 2
    
    # 查看所有成员（按分数升序）
    127.0.0.1:6379> ZRANGE leaderboard:game1 0 -1 WITHSCORES
    1) "Alice"
    2) "150"
    3) "Bob"
    4) "200"
    5) "David"
    6) "250"
    7) "Charlie"
    8) "300"
    ```

  

- `ZREM <key> <member1> [<member2> ...]`

  > ZSet REMove

  - **作用**：从有序集合 `key` 中移除一个或多个成员

  - **代码示例**：

    ```bash
    # 移除单个成员
    127.0.0.1:6379> ZREM leaderboard:game1 "Alice"
    (integer) 1  # 返回成功移除的成员数量
    
    # 移除多个成员
    127.0.0.1:6379> ZREM leaderboard:game1 "Bob" "NonExist"
    (integer) 1  # NonExist不存在，只移除了Bob
    
    # 查看剩余成员
    127.0.0.1:6379> ZRANGE leaderboard:game1 0 -1
    1) "David"
    2) "Charlie"
    ```

  

- `ZINCRBY <key> <increment> <member>`

  > ZSet INCRement BY

  - **作用**：**原子性地**为 `key` 中指定 `member` 的分数增加 `increment`（可以是负数）

  - **代码示例**：

    ```bash
    127.0.0.1:6379> ZADD leaderboard:game1 100 "Alice"
    (integer) 1
    
    # Alice的分数+50
    127.0.0.1:6379> ZINCRBY leaderboard:game1 50 "Alice"
    "150"  # 返回增加后的分数
    
    # Alice的分数-20
    127.0.0.1:6379> ZINCRBY leaderboard:game1 -20 "Alice"
    "130"
    
    # 如果成员不存在，会自动创建并设置分数
    127.0.0.1:6379> ZINCRBY leaderboard:game1 100 "NewPlayer"
    "100"
    ```



#### 按排名范围获取 (Rank)

- 这是 ZSet 最常用的功能之一。排名 (Rank) 是基于分数的顺序（**排名的索引默认从 0 开始**）

  - `ZRANGE <key> <start> <stop> [WITHSCORES]`

    - **作用**：按分数**升序**（从小到大）获取指定排名范围的成员

    - **代码示例**：

      ```bash
      127.0.0.1:6379> ZADD scores 60 "Alice" 80 "Bob" 90 "Charlie" 70 "David"
      (integer) 4
      
      # 获取所有成员（按分数从低到高）
      127.0.0.1:6379> ZRANGE scores 0 -1
      1) "Alice"
      2) "David"
      3) "Bob"
      4) "Charlie"
      
      # 获取分数最低的前3名，并带上分数
      127.0.0.1:6379> ZRANGE scores 0 2 WITHSCORES
      1) "Alice"
      2) "60"
      3) "David"
      4) "70"
      5) "Bob"
      6) "80"
      
      # 获取排名1-2的成员（索引从0开始）
      127.0.0.1:6379> ZRANGE scores 1 2
      1) "David"
      2) "Bob"
      ```

    

  - `ZREVRANGE <key> <start> <stop> [WITHSCORES]` (REVerse RANGE)

    - **作用**：按分数**降序**（从大到小）获取指定排名范围的成员

    - **代码示例**：

      ```bash
      # 获取分数最高的Top 3
      127.0.0.1:6379> ZREVRANGE scores 0 2 WITHSCORES
      1) "Charlie"
      2) "90"
      3) "Bob"
      4) "80"
      5) "David"
      6) "70"
      
      # 获取分数最高的Top 10（常用于排行榜）
      127.0.0.1:6379> ZREVRANGE leaderboard:game1 0 9
      1) "Player1"
      2) "Player2"
      # ...
      ```



#### 按分数范围获取 (Score)

- `ZRANGEBYSCORE <key> <min> <max> [WITHSCORES] [LIMIT offset count]`

  - **作用**：按分数**升序**获取指定分数区间 `[min, max]` 的所有成员

  - **技巧**：`min` 和 `max` 可以使用 `(` 来表示"不包含"，例如 `(100 500` 表示大于100且小于等于500

  - **代码示例**：

    ```bash
    127.0.0.1:6379> ZADD scores 50 "A" 100 "B" 150 "C" 200 "D" 250 "E"
    (integer) 5
    
    # 获取分数在100到500之间的所有成员
    127.0.0.1:6379> ZRANGEBYSCORE scores 100 500
    1) "B"
    2) "C"
    3) "D"
    4) "E"
    
    # 获取分数在100到200之间的成员，带分数
    127.0.0.1:6379> ZRANGEBYSCORE scores 100 200 WITHSCORES
    1) "B"
    2) "100"
    3) "C"
    4) "150"
    5) "D"
    6) "200"
    
    # 使用开区间：大于100（不包含100），小于等于200
    127.0.0.1:6379> ZRANGEBYSCORE scores (100 200
    1) "C"
    2) "D"
    
    # 使用LIMIT分页：跳过前2个，获取2个
    127.0.0.1:6379> ZRANGEBYSCORE scores 0 300 LIMIT 2 2
    1) "C"
    2) "D"
    
    # 使用-inf和+inf表示负无穷和正无穷
    127.0.0.1:6379> ZRANGEBYSCORE scores -inf +inf
    1) "A"
    2) "B"
    3) "C"
    4) "D"
    5) "E"
    ```

  

- `ZREVRANGEBYSCORE <key> <max> <min> [WITHSCORES] [LIMIT offset count]`

  > REVerse RANGE BY SCORE

  - **作用**：按分数**降序**获取指定分数区间的成员（注意：参数顺序是 max 在前，min 在后）

  - **代码示例**：

    ```bash
    # 按分数降序获取100到200之间的成员
    127.0.0.1:6379> ZREVRANGEBYSCORE scores 200 100 WITHSCORES
    1) "D"
    2) "200"
    3) "C"
    4) "150"
    5) "B"
    6) "100"
    
    # 获取分数大于150的所有成员（降序）
    127.0.0.1:6379> ZREVRANGEBYSCORE scores +inf 150
    1) "E"
    2) "D"
    3) "C"
    ```



#### 获取排名

- `ZRANK <key> <member>`

  > ZSet RANK

  - **作用**：获取成员在有序集合中的**升序排名**（分数从小到大，排名从0开始）

  - **代码示例**：

    ```bash
    127.0.0.1:6379> ZADD scores 60 "Alice" 80 "Bob" 90 "Charlie" 70 "David"
    (integer) 4
    
    # 获取Alice的排名（分数最低，排名为0）
    127.0.0.1:6379> ZRANK scores "Alice"
    (integer) 0
    
    # 获取Bob的排名
    127.0.0.1:6379> ZRANK scores "Bob"
    (integer) 2
    
    # 成员不存在返回nil
    127.0.0.1:6379> ZRANK scores "NonExist"
    (nil)
    ```

  

- `ZREVRANK <key> <member>`

  > ZSet REVerse RANK

  - **作用**：获取成员在有序集合中的**降序排名**（分数从大到小，排名从0开始）

  - **代码示例**：

    ```bash
    # 获取Charlie的降序排名（分数最高，排名为0）
    127.0.0.1:6379> ZREVRANK scores "Charlie"
    (integer) 0
    
    # 获取Alice的降序排名（分数最低，排名为3）
    127.0.0.1:6379> ZREVRANK scores "Alice"
    (integer) 3
    
    # 获取Bob的降序排名
    127.0.0.1:6379> ZREVRANK scores "Bob"
    (integer) 1
    ```



#### 获取分数与统计

- `ZSCORE <key> <member>`

  - **作用**：获取 `key` 中指定 `member` 的分数

  - **代码示例**：

    ```bash
    # 获取Bob的分数
    127.0.0.1:6379> ZSCORE leaderboard:game1 "Bob"
    "200"
    
    # 成员不存在返回nil
    127.0.0.1:6379> ZSCORE leaderboard:game1 "NonExist"
    (nil)
    ```

  

- `ZCARD <key>` (ZSet CARDinality)

  - **作用**：获取 `key` 中成员的数量

  - **代码示例**：

    ```bash
    # 获取有序集合的成员数量
    127.0.0.1:6379> ZCARD leaderboard:game1
    (integer) 4
    
    # 不存在的key返回0
    127.0.0.1:6379> ZCARD nonexist
    (integer) 0
    ```

  

- `ZCOUNT <key> <min> <max>`

  > ZSet COUNT

  - **作用**：统计分数在指定区间 `[min, max]` 内的成员数量

  - **代码示例**：

    ```bash
    127.0.0.1:6379> ZADD scores 60 "A" 80 "B" 90 "C" 70 "D" 100 "E"
    (integer) 5
    
    # 统计分数在70到90之间的成员数量
    127.0.0.1:6379> ZCOUNT scores 70 90
    (integer) 3  # D, B, C
    
    # 使用开区间
    127.0.0.1:6379> ZCOUNT scores (70 (90
    (integer) 1  # 只有B（80）
    
    # 统计所有成员
    127.0.0.1:6379> ZCOUNT scores -inf +inf
    (integer) 5
    ```



#### 删除操作

- `ZREMRANGEBYRANK <key> <start> <stop>`

  > ZSet REMove RANGE BY RANK

  - **作用**：移除有序集合中指定排名区间的所有成员（按升序排名）

  - **代码示例**：

    ```bash
    127.0.0.1:6379> ZADD scores 60 "A" 80 "B" 90 "C" 70 "D" 100 "E"
    (integer) 5
    
    # 删除排名0-1的成员（分数最低的两个）
    127.0.0.1:6379> ZREMRANGEBYRANK scores 0 1
    (integer) 2  # 删除了A和D
    
    # 查看剩余成员
    127.0.0.1:6379> ZRANGE scores 0 -1
    1) "B"
    2) "C"
    3) "E"
    ```

  

- `ZREMRANGEBYSCORE <key> <min> <max>`

  > ZSet REMove RANGE BY SCORE

  - **作用**：移除有序集合中分数在指定区间的所有成员

  - **代码示例**：

    ```bash
    127.0.0.1:6379> ZADD scores 60 "A" 80 "B" 90 "C" 70 "D" 100 "E"
    (integer) 5
    
    # 删除分数在70到90之间的成员
    127.0.0.1:6379> ZREMRANGEBYSCORE scores 70 90
    (integer) 3  # 删除了D, B, C
    
    # 查看剩余成员
    127.0.0.1:6379> ZRANGE scores 0 -1 WITHSCORES
    1) "A"
    2) "60"
    3) "E"
    4) "100"
    ```



#### 弹出操作

- `ZPOPMIN <key> [count]`

  > ZSet POP MINimum

  - **作用**：移除并返回分数最小的一个或多个成员（Redis 5.0+）

  - **代码示例**：

    ```bash
    127.0.0.1:6379> ZADD scores 60 "A" 80 "B" 90 "C" 70 "D"
    (integer) 4
    
    # 弹出分数最小的成员
    127.0.0.1:6379> ZPOPMIN scores
    1) "A"
    2) "60"
    
    # 弹出分数最小的2个成员
    127.0.0.1:6379> ZPOPMIN scores 2
    1) "D"
    2) "70"
    3) "B"
    4) "80"
    
    # 查看剩余成员
    127.0.0.1:6379> ZRANGE scores 0 -1
    1) "C"
    ```

  

- `ZPOPMAX <key> [count]`

  > ZSet POP MAXimum

  - **作用**：移除并返回分数最大的一个或多个成员（Redis 5.0+）

  - **代码示例**：

    ```bash
    127.0.0.1:6379> ZADD scores 60 "A" 80 "B" 90 "C" 70 "D"
    (integer) 4
    
    # 弹出分数最大的成员
    127.0.0.1:6379> ZPOPMAX scores
    1) "C"
    2) "90"
    
    # 弹出分数最大的2个成员
    127.0.0.1:6379> ZPOPMAX scores 2
    1) "B"
    2) "80"
    3) "D"
    4) "70"
    ```

  

- `BZPOPMIN <key> [key ...] <timeout>`

  > Blocking ZSet POP MINimum

  - **作用**：阻塞版本的 `ZPOPMIN`，当集合为空时会阻塞等待（Redis 5.0+）

  - **代码示例**：

    ```bash
    # 集合不为空，立即返回
    127.0.0.1:6379> ZADD queue 1 "task1" 2 "task2"
    (integer) 2
    127.0.0.1:6379> BZPOPMIN queue 5
    1) "queue"  # 键名
    2) "task1"  # 成员
    3) "1"      # 分数
    
    # 集合为空，阻塞等待5秒
    127.0.0.1:6379> BZPOPMIN empty_queue 5
    (nil)  # 5秒后超时
    ```

  

- `BZPOPMAX <key> [key ...] <timeout>`

  > Blocking ZSet POP MAXimum

  - **作用**：阻塞版本的 `ZPOPMAX`（Redis 5.0+）



#### 集合运算

- `ZUNIONSTORE <destination> <numkeys> <key1> [<key2> ...] [WEIGHTS <weight1> [<weight2> ...]] [AGGREGATE SUM|MIN|MAX]`

  > ZSet UNION STORE

  - **作用**：计算多个有序集合的并集，并将结果存储到 `destination`

  - **参数**：

    - `WEIGHTS`：为每个集合设置权重，分数会乘以权重
    - `AGGREGATE`：聚合方式（SUM求和、MIN取最小、MAX取最大）

  - **代码示例**：

    ```bash
    # 创建两个有序集合
    127.0.0.1:6379> ZADD zset1 10 "A" 20 "B" 30 "C"
    (integer) 3
    
    127.0.0.1:6379> ZADD zset2 15 "A" 25 "B" 35 "D"
    (integer) 3
    
    # 计算并集（默认SUM聚合）
    127.0.0.1:6379> ZUNIONSTORE result 2 zset1 zset2
    (integer) 4  # 返回结果集合的成员数
    
    # 查看结果（A和B的分数是两个集合中分数的和）
    127.0.0.1:6379> ZRANGE result 0 -1 WITHSCORES
    1) "A"
    2) "25"   # 10 + 15
    3) "C"
    4) "30"
    5) "D"
    6) "35"
    7) "B"
    8) "45"   # 20 + 25
    
    # 使用权重和MIN聚合
    127.0.0.1:6379> ZUNIONSTORE result2 2 zset1 zset2 WEIGHTS 1 2 AGGREGATE MIN
    (integer) 4
    
    127.0.0.1:6379> ZRANGE result2 0 -1 WITHSCORES
    1) "A"
    2) "10"   # min(10*1, 15*2) = 10
    3) "B"
    4) "20"   # min(20*1, 25*2) = 20
    5) "C"
    6) "30"
    7) "D"
    8) "70"   # 35*2
    ```

  

- `ZINTERSTORE <destination> <numkeys> <key1> [<key2> ...] [WEIGHTS <weight1> [<weight2> ...]] [AGGREGATE SUM|MIN|MAX]`

  > ZSet INTERsection STORE

  - **作用**：计算多个有序集合的交集，并将结果存储到 `destination`

  - **代码示例**：

    ```bash
    # 计算交集（只有同时存在于两个集合的成员）
    127.0.0.1:6379> ZINTERSTORE result 2 zset1 zset2
    (integer) 2  # 只有A和B
    
    # 查看结果
    127.0.0.1:6379> ZRANGE result 0 -1 WITHSCORES
    1) "A"
    2) "25"   # 10 + 15
    3) "B"
    4) "45"   # 20 + 25
    
    # 使用权重和MAX聚合
    127.0.0.1:6379> ZINTERSTORE result2 2 zset1 zset2 WEIGHTS 2 1 AGGREGATE MAX
    (integer) 2
    127.0.0.1:6379> ZRANGE result2 0 -1 WITHSCORES
    1) "A"
    2) "20"   # max(10*2, 15*1) = 20
    3) "B"
    4) "40"   # max(20*2, 25*1) = 40
    ```



### 应用场景

1. **排行榜 (Leaderboard)**

   - **描述**：这是 ZSet 最经典的应用场景

   - **代码示例**：

     ```bash
     # 初始化每日积分榜
     127.0.0.1:6379> ZADD leaderboard:daily:2025-11-07 100 "player_1" 200 "player_2" 150 "player_3"
     (integer) 3
     
     # 玩家5完成任务，增加10分
     127.0.0.1:6379> ZINCRBY leaderboard:daily:2025-11-07 10 "player_5"
     "10"
     
     # 玩家2继续得分
     127.0.0.1:6379> ZINCRBY leaderboard:daily:2025-11-07 50 "player_2"
     "250"
     
     # 显示Top 10（降序）
     127.0.0.1:6379> ZREVRANGE leaderboard:daily:2025-11-07 0 9 WITHSCORES
     1) "player_2"
     2) "250"
     3) "player_3"
     4) "150"
     5) "player_1"
     6) "100"
     7) "player_5"
     8) "10"
     
     # 查看player_5的排名（降序排名，0是第一名）
     127.0.0.1:6379> ZREVRANK leaderboard:daily:2025-11-07 "player_5"
     (integer) 3  # 第4名（索引从0开始）
     
     # 查看player_5的分数
     127.0.0.1:6379> ZSCORE leaderboard:daily:2025-11-07 "player_5"
     "10"
     
     # 查看排名在50-100之间的玩家
     127.0.0.1:6379> ZREVRANGE leaderboard:daily:2025-11-07 49 99
     # 返回排名50-100的玩家
     
     # 统计分数大于100的玩家数量
     127.0.0.1:6379> ZCOUNT leaderboard:daily:2025-11-07 100 +inf
     (integer) 3
     ```

   

2. **带权重的消息队列 / 延迟队列**

   - **描述**：与 List 实现的消息队列不同，ZSet 可以实现**带优先级**或**延迟执行**的队列

   - **代码示例**：

     ```bash
     # 添加延迟任务（score是执行时间戳）
     127.0.0.1:6379> ZADD delayed_tasks 1678886400 "task_id_1"
     (integer) 1
     127.0.0.1:6379> ZADD delayed_tasks 1678886500 "task_id_2"
     (integer) 1
     127.0.0.1:6379> ZADD delayed_tasks 1678886600 "task_id_3"
     (integer) 1
     
     # 假设当前时间戳为1678886450
     # 获取所有到期的任务（分数小于等于当前时间戳）
     127.0.0.1:6379> ZRANGEBYSCORE delayed_tasks 0 1678886450 WITHSCORES
     1) "task_id_1"
     2) "1678886400"
     
     # 移除已获取的任务（原子性很重要）
     127.0.0.1:6379> ZREM delayed_tasks "task_id_1"
     (integer) 1
     
     # 或者使用ZPOPMIN直接弹出分数最小的任务
     127.0.0.1:6379> ZPOPMIN delayed_tasks
     1) "task_id_2"
     2) "1678886500"
     
     # 优先级队列示例（分数越小优先级越高）
     127.0.0.1:6379> ZADD priority_queue 1 "urgent_task"
     (integer) 1
     127.0.0.1:6379> ZADD priority_queue 5 "normal_task"
     (integer) 1
     127.0.0.1:6379> ZADD priority_queue 10 "low_priority_task"
     (integer) 1
     
     # 获取优先级最高的任务
     127.0.0.1:6379> ZPOPMIN priority_queue
     1) "urgent_task"
     2) "1"
     ```

   

3. **热门文章/商品排序**

   - **描述**：根据点击量、销量等指标动态排序

   - **代码示例**：

     ```bash
     # 初始化文章热度
     127.0.0.1:6379> ZADD hot_articles 100 "article:1" 200 "article:2" 150 "article:3"
     (integer) 3
     
     # 用户点击文章1，热度+1
     127.0.0.1:6379> ZINCRBY hot_articles 1 "article:1"
     "101"
     
     # 获取热度最高的10篇文章
     127.0.0.1:6379> ZREVRANGE hot_articles 0 9 WITHSCORES
     1) "article:2"
     2) "200"
     3) "article:3"
     4) "150"
     5) "article:1"
     6) "101"
     
     # 清理热度低于50的文章
     127.0.0.1:6379> ZREMRANGEBYSCORE hot_articles 0 50
     (integer) 0
     ```

   

4. **时间序列数据**

   - **描述**：使用时间戳作为分数，存储带时间属性的数据

   - **代码示例**：

     ```bash
     # 记录用户登录历史（时间戳作为分数）
     127.0.0.1:6379> ZADD user:1001:login_history 1699000000 "login_1"
     (integer) 1
     127.0.0.1:6379> ZADD user:1001:login_history 1699010000 "login_2"
     (integer) 1
     127.0.0.1:6379> ZADD user:1001:login_history 1699020000 "login_3"
     (integer) 1
     
     # 查询最近3次登录
     127.0.0.1:6379> ZREVRANGE user:1001:login_history 0 2 WITHSCORES
     1) "login_3"
     2) "1699020000"
     3) "login_2"
     4) "1699010000"
     5) "login_1"
     6) "1699000000"
     
     # 查询某个时间段的登录记录
     127.0.0.1:6379> ZRANGEBYSCORE user:1001:login_history 1699000000 1699015000
     1) "login_1"
     2) "login_2"
     
     # 删除30天前的登录记录
     127.0.0.1:6379> ZREMRANGEBYSCORE user:1001:login_history 0 1696408000
     (integer) 0
     ```

   

5. **限流（滑动窗口）**

   - **描述**：使用时间戳作为分数，实现滑动窗口限流

   - **代码示例**：

     ```bash
     # 记录用户访问请求（当前时间戳为1699000000）
     127.0.0.1:6379> ZADD rate_limit:user:1001 1699000000 "req_1"
     (integer) 1
     127.0.0.1:6379> ZADD rate_limit:user:1001 1699000001 "req_2"
     (integer) 1
     127.0.0.1:6379> ZADD rate_limit:user:1001 1699000002 "req_3"
     (integer) 1
     
     # 删除60秒前的请求记录（滑动窗口）
     127.0.0.1:6379> ZREMRANGEBYSCORE rate_limit:user:1001 0 1698999940
     (integer) 0
     
     # 统计最近60秒的请求数量
     127.0.0.1:6379> ZCOUNT rate_limit:user:1001 1698999940 1699000000
     (integer) 3
     
     # 如果请求数量超过限制（如10次），则拒绝请求
     # 实际使用中，结合ZADD和ZCOUNT实现完整的限流逻辑
     ```

   

6. **范围查询（如附近的人）**

   - **描述**：使用地理位置编码作为分数，实现简单的范围查询

   - **代码示例**：

     ```bash
     # 使用经纬度转换的score存储用户位置（简化示例）
     127.0.0.1:6379> ZADD locations 116.404 "user:1001"
     (integer) 1
     127.0.0.1:6379> ZADD locations 116.405 "user:1002"
     (integer) 1
     127.0.0.1:6379> ZADD locations 116.410 "user:1003"
     (integer) 1
     
     # 查询位置在116.403到116.407之间的用户
     127.0.0.1:6379> ZRANGEBYSCORE locations 116.403 116.407
     1) "user:1001"
     2) "user:1002"
     
     # 注意：实际地理位置应使用Redis的GEO类型，这里仅作演示
     ```



# Redis 进阶特性

## 持久化

>Persistence

### 1. 什么是持久化？

- Redis 是一个基于内存的数据库。当 Redis 服务器进程退出时，存储在内存中的数据将会丢失

- **持久化** 是一种机制，它允许 Redis 将内存中的数据集**写入到磁盘**，以便在服务器重启时能够从磁盘加载数据，实现数据恢复，确保数据不会因为进程退出而丢失
  - Redis 提供了两种主要的持久化方式：**RDB (Redis Database)** 和 **AOF (Append Only File)**



### 2. RDB

>Redis Database

#### 工作原理：快照

> Snapshotting

- RDB 是 Redis 默认的持久化方式。它通过**创建快照**来工作

- RDB 会在**指定的时间间隔**内，自动执行一次“快照”操作，将**某一时刻**内存中的**所有数据**生成一个二进制文件，保存到磁盘上。这个文件默认名为 `dump.rdb`

- Redis 重启时，会优先查找 `dump.rdb` 文件，如果找到，就会将文件中的数据一次性全部加载到内存中



#### 优点

1. **性能高**：

   - RDB 通过 `fork` 一个子进程来执行持久化，

     主进程（单线程）不需要执行磁盘 I/O 操作，可以继续处理客户端请求，对性能影响极小

2. **恢复速度快**：

   - `dump.rdb` 文件是一个紧凑的二进制文件，存储的是数据的实际内容
     - 在恢复时，Redis 只需要解析并加载这个文件，速度远快于 AOF



#### 缺点

1. **数据丢失风险**：
   - RDB 是**间隔性**保存快照的
     - 如果在两次快照之间（例如设置了5分钟保存一次），Redis 意外宕机，那么从上次快照到宕机时刻这期间所有写入的数据都将**全部丢失**
2. **数据量大时耗时**：
   - 如果数据集非常大，`fork` 子进程和生成快照的过程可能会消耗一定时间（秒级）



#### 配置

- 在 `redis.conf` 文件中通过 `save` 指令配置：

  - `save 900 1`：在 900 秒（15分钟）内，如果至少有 1 个 key 发生了变化，则执行一次快照

  - `save 300 10`：在 300 秒（5分钟）内，如果至少有 10 个 key 发生了变化，则执行一次快照

  - `save 60 10000`：在 60 秒内，如果至少有 10000 个 key 发生了变化，则执行一次快照



### 3. AOF

>Append Only File

#### 工作原理: 追加日志

>Append Only

- AOF 持久化会**记录**（追加）**每一条**导致数据发生变化的**写命令**（例如 `SET`, `INCR`, `ZADD`）到一个日志文件末尾（默认 `appendonly.aof`）
  - 当 Redis 重启时，它会重新执行 AOF 文件中保存的所有写命令，从头到尾执行一遍，从而恢复到宕机前的最后状态



#### 优点

1. **数据安全性高**：
   - AOF 提供了更高的数据安全性
     - 根据 `appendfsync` 配置的不同，最多只会丢失 1 秒的数据（默认配置 `everysec`）
2. **可读性**：
   - AOF 文件以 Redis 命令协议的格式存储，内容是可读的（尽管是二进制安全的文本）
     - 在极端情况下，可以手动编辑 AOF 文件来修复错误（例如 `FLUSHALL` 误操作）



#### 缺点

1. **文件体积大**：对于相同的数 据集，AOF 文件通常远大于 RDB 文件，因为它存储的是操作日志，而不是实际数据
2. **恢复速度慢**：数据恢复时，Redis 需要逐条重新执行 AOF 文件中的所有命令，速度通常慢于 RDB 的加载
3. **对性能有一定影响**：
   - 虽然 AOF 也是在后台线程中执行刷盘，但在高并发写入时，其性能（尤其是 `appendfsync` 设置为 `always` 时）通常略低于 RDB



#### 配置

- `appendonly yes`：启用 AOF（默认是 `no`）
- `appendfsync`：配置 AOF 何时将数据同步（`fsync`）到磁盘：
  - `always`：每条写命令都立即同步到磁盘。**最安全，但性能最差**
  - `everysec`（默认）：每秒同步一次。**性能和安全的平衡点**。如果宕机，最多丢失 1 秒的数据
  - `no`：完全依赖操作系统的刷盘策略。**最快，但最不安全**



### 4. RDB 与 AOF 的选择与混合使用

#### 如何选择？

- **如果能接受分钟级的数据丢失**：
  - 只使用 RDB。它提供了良好的性能和快速的恢复
- **如果追求最高的数据安全性**：
  - 只使用 AOF（并设置为 `everysec` 或 `always`）
- **不推荐**：
  - **关闭所有持久化**。这**只适用于数据完全可以丢失**的纯缓存场景



#### 混合使用（Redis 4.0+ 推荐）

- Redis 4.0 以后支持 RDB 和 AOF 混合使用，这是目前 **推荐** 的方式

  - **工作方式**：
    - 当 Redis 触发 AOF 重写时（AOF 文件过大时会触发压缩），它不再是把命令重写到 AOF 文件，而是**将当前时刻的 RDB 快照内容**写入到新的 AOF 文件开头，然后再追加 AOF 重写期间产生的增量写命令

  - **优点**：
    1. **数据安全**：保留了 AOF 的高安全性（增量日志）
    2. **恢复快速**：恢复时，Redis 先加载文件开头的 RDB 部分，然后再加载增量的 AOF 命令，恢复速度大幅提升，接近 RDB

  - **如何开启**：
    1. `appendonly yes` (开启 AOF)
    2. `aof-use-rdb-preamble yes` (开启混合持久化)





## 事务

>Transaction

- Redis 的事务提供了一种将**多个命令打包**、**一次性**、**按顺序**执行的机制
  - 在事务执行期间，Redis 会保证**不会中途插入**其他客户端的命令

### 基本命令

- `MULTI` (Multiple)
  - **作用**：标记一个事务块的开始
  - **响应**：OK
  - **后续**：此命令之后的所有命令都会被放入一个**命令队列**中，而**不会立即执行**



- `EXEC` (Execute)
  - **作用**：原子性地执行所有在 `MULTI` 之后入队的命令
  - **响应**：返回一个数组，数组中的每个元素是队列中对应命令的执行结果
  - **注意**：一旦调用 `EXEC`，事务块结束



- `DISCARD` (Discard)
  - **作用**：放弃事务
  - **后续**：清空已入队的命令，并退出事务状态
  - **响应**：OK



- `WATCH <key1> [<key2> ...]`

  - **作用**：监视一个或多个 `key`
  - **机制**：
    - 如果在 `MULTI` 执行之前，被 `WATCH` 监视的 `key` **被其他客户端修改了**，那么当 `EXEC` 被调用时，整个事务将**不会被执行**（事务失败）

  - **响应**：OK



### 示例

```bash
# 客户端 A
127.0.0.1:6379> SET balance 100
OK
127.0.0.1:6379> SET debt 0
OK

127.0.0.1:6379> MULTI       		# 1. 开始事务
OK
127.0.0.1:6379> DECRBY balance 20  	# 2. 命令入队
QUEUED
127.0.0.1:6379> INCRBY debt 20     	# 3. 命令入队
QUEUED
127.0.0.1:6379> EXEC       			# 4. 执行事务
1) (integer) 80  					# DECRBY 的结果
2) (integer) 20  					# INCRBY 的结果
```



### Redis 事务的“原子性”

- Redis 事务的原子性与传统 SQL 数据库（如 MySQL）的事务**不同**

  - **SQL 事务（强原子性）**：
    - **回滚 (Rollback)**：
      - 在一个事务中，如果某条 SQL 语句执行失败（例如违反了唯一约束），整个事务会**全部回滚**，所有已执行的操作都会被撤销，数据库恢复到事务开始前的状态

  - **Redis 事务（弱原子性）**：
    - **批量执行**：Redis 只保证 `MULTI` 和 `EXEC` 之间的所有命令会被**连续执行**，中间不会被其他客户端打断
    
    - **没有回滚**：
      1. **语法错误**：
      
         - 如果入队时命令存在语法错误（例如 `SET` 写成了 `SETT`），Redis 会在 `EXEC` 时**拒绝执行**整个事务，所有命令都不执行
      
      2. **运行时错误**：
      
         - 如果入队时命令语法正确，但在 `EXEC` 执行时出错（例如对一个 String 类型的 key 执行 `HSET` 操作），
      
           Redis **不会停止**，它会跳过这条出错的命令，**继续执行事务中剩余的其他命令**
      
    - **总结**：Redis 事务保证了“**批量执行**”的原子性，但不保证“**逻辑正确**”的原子性，它**没有回滚机制**



### CAS

>Check-And-Set

> 使用 WATCH 实现乐观锁

- 在并发环境中，我们经常遇到“Read-Modify-Write”的场景（例如：读取A=10，计算A+5，写回A=15）
  - 如果在我们“读取”和“写回”之间，有其他人把 A 修改成了 20，那么我们的“写回”就会覆盖掉别人的修改，导致数据不一致

- `WATCH` 命令就是用来解决这个问题的，它实现了一种**乐观锁 (Optimistic Locking)**

  > WATCH` 提供了“**检查并设置**”（Check-And-Set, CAS）的能力，它保证了事务的执行是基于一个**未被修改过的**初始值，从而确保了并发操作的正确性

- **工作原理**：
  1. 客户端 A 在 `MULTI` 之前 `WATCH balance`
  2. 客户端 A 读取 `balance` 为 100
  3. **此时**，客户端 B 修改了 `balance`，执行了 `SET balance 300`
  4. 客户端 A 开始事务 `MULTI`
  5. 客户端 A 执行 `SET balance 120`（基于它读到的 100 计算得到）
  6. 客户端 A 执行 `EXEC`
  7. **触发检查**：Redis 在 `EXEC` 时发现被 `WATCH` 的 `balance` 已经被客户端 B 修改过了
  8. **事务失败**：Redis **拒绝执行**客户端 A 的整个事务（`SET balance 120` 不会执行），并向客户端 A 返回 `(nil)`
  9. **重试**：
     - 客户端 A 收到 `(nil)` 后，就知道操作失败了，它需要**从头开始重试**整个流程（重新 `WATCH`，重新 `GET`，重新计算，重新 `MULTI`...）



## 发布/订阅

>Pub/Sub

### 简介：一种消息通信模式

- Redis 的发布/订阅 (Pub/Sub) 功能是一种**消息通信模式**

  > 这种模式实现了发送者和订阅者之间的**解耦**

  - 它允许客户端订阅任意数量的“**频道 (Channel)**”，当其他客户端向这些频道“**发布 (Publish)**”消息时，所有订阅了该频道的客户端都会收到这条消息

- **发送者 (Publisher)**：负责向特定频道发布消息。发送者不需要知道谁在监听
- **订阅者 (Subscriber)**：负责订阅一个或多个频道。订阅者不需要知道谁在发消息
- **频道 (Channel)**：消息的传输媒介



### 常用命令

#### 订阅 (Subscription)

- `SUBSCRIBE <channel1> [<channel2> ...]`

  - **作用**：订阅一个或多个指定的频道
  - **注意**：
    - 一旦客户端执行 `SUBSCRIBE`，它就会**进入“订阅模式”**
    - 在这种模式下，客户端**不能执行**除 `SUBSCRIBE`, `UNSUBSCRIBE`, `PSUBSCRIBE`, `PUNSUBSCRIBE`, `PING` 和 `QUIT` 之外的其他命令（例如 `SET`, `GET`）
  - **响应**：客户端会**阻塞**并等待该频道的消息

  

- `PSUBSCRIBE <pattern1> [<pattern2> ...]` (Pattern SUBSCRIBE)

  - **作用**：**模式订阅**。订阅所有匹配指定模式 (`pattern`) 的频道
  - **模式匹配**：使用 `*` (匹配任意多个字符) 和 `?` (匹配单个字符)
  - **示例**：`PSUBSCRIBE news.*` 会订阅所有以 `news.` 开头的频道 (如 `news.tech`, `news.sports`)



#### 发布 (Publication)

- `PUBLISH <channel> <message>`
  - **作用**：向指定的频道 `channel` 发布一条消息 `message`
  - **响应**：返回一个整数，表示收到这条消息的订阅者数量



#### 取消订阅 (Unsubscription)

- `UNSUBSCRIBE [<channel1> ...]`
  - **作用**：取消订阅指定的频道。如果不指定频道，则取消所有订阅
- `PUNSUBSCRIBE [<pattern1> ...]`
  - **作用**：取消订阅指定的模式



### 示例

- 打开**两个** `redis-cli` 窗口来模拟

  - **窗口 1 (订阅者)**：

    ```bash
    127.0.0.1:6379> SUBSCRIBE channel:chat_room:1
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "channel:chat_room:1"
    3) (integer) 1
    # 此时，窗口 1 处于阻塞状态，等待消息
    ```

    

  - **窗口 2 (发布者)**：

    ```bash
    127.0.0.1:6379> PUBLISH channel:chat_room:1 "Hello, anyone here?"
    (integer) 1  # 告诉发布者，有 1 个客户端收到了消息
    
    127.0.0.1:6379> PUBLISH channel:chat_room:1 "This is message 2"
    (integer) 1
    ```

  

  - **窗口 1 (订阅者) 会立即收到消息**：

    ```bash
    ...
    1) "message"
    2) "channel:chat_room:1"
    3) "Hello, anyone here?"
    
    1) "message"
    2) "channel:chat_room:1"
    3) "This is message 2"
    ```



### 应用场景

1. **实时聊天室**

   - **描述**：

     - 每个聊天室可以是一个频道 (如 `chat_room:888`)

       所有加入该聊天室的用户都 `SUBSCRIBE` 这个频道

       当一个用户发言时，服务器就 `PUBLISH` 消息到该频道，所有订阅者都会收到

   

2. **事件通知与系统解耦**

   - **描述**：

     - 微服务架构中，一个服务（例如“订单服务”）在完成某个动作（例如“订单支付成功”）后，可以向一个频道 (如 `events:order_paid`) 发布一条消息

   - **解耦**：

     - 其他服务（例如“积分服务”、“物流服务”、“邮件服务”）可以 `SUBSCRIBE` 这个频道

       它们收到消息后各自执行自己的逻辑（增加积分、通知仓库、发送邮件），而“订单服务”完全不需要知道它们的存在



### 局限性

- Redis 的 Pub/Sub 是一种“**发后即忘 (Fire and Forget)**”的模式

  - **没有持久化**：如果消息发布时，某个订阅者不在线，那么这条消息对它来说就**永远丢失了**

  - **没有ACK机制**：发布者不知道订阅者是否成功处理了消息

- 如果需要可靠的消息传递（确保消息至少被消费一次），应优先使用 Redis 5.0 引入的 **Stream** 数据结构，或专业的**消息队列**（如 RabbitMQ, Kafka）



## 管道

>Pipeline

### 概念：减少网络往返时延 (RTT)

- Redis 是一个基于“**请求-响应**”模型的 TCP 服务

  - **标准执行**：客户端发送一个命令（如 `SET`），等待服务器响应（`OK`），然后再发送下一个命令（如 `GET`），再等待响应

  - **网络开销**：
    - 如果执行大量命令，这种“一来一回”的网络延迟（RTT - Round-Trip Time）会成为主要的性能瓶颈，而不是 Redis 服务器本身的处理速度

- **管道 (Pipeline)** 是一种客户端技术，它允许客户端将**多个命令一次性打包**发送给服务器，然后在最后**一次性接收**所有命令的响应

  - 这种方式通过一次网络往返（RTT）执行了 N 个命令，极大地提高了批量操作的执行效率



### 示例

- 假设我们要连续执行 3 个 `SET` 命令：

  - **1. 不使用 Pipeline（3 次 RTT）:**

    ```cmd
    Client -> Server: SET a 1
    Server -> Client: OK
    Client -> Server: SET b 2
    Server -> Client: OK
    Client -> Server: SET c 3
    Server -> Client: OK
    ```

    

  - **2. 使用 Pipeline（1 次 RTT）：**

    ```cmd
    Client -> Server: SET a 1
    Client -> Server: SET b 2
    Client -> Server: SET c 3
    (客户端将命令打包在输出缓冲区，一次性发送)
    
    Server -> Client: OK
    Server -> Client: OK
    Server -> Client: OK
    (服务器执行完所有命令后，一次性返回所有结果)
    ```



### 与事务的区别

- Pipeline 和 Redis 事务（`MULTI`/`EXEC`）虽然都能批量执行命令，但它们的目的和机制完全不同

  - **管道 (Pipeline)**

    - **目的**：**网络性能优化**。减少网络往返 (RTT)
    - **原子性**：**不保证原子性**。服务器在执行 Pipeline 中的命令时，可能会被其他客户端的命令打断
    - **服务器端**：服务器不知道客户端在使用 Pipeline，它只是按顺序处理接收到的命令
    - **命令执行**：命令是**边接收边执行**（或在缓冲区满时执行），而不是等待所有命令都到达*（注：不同客户端实现略有差异，但核心是批量发送/接收）*

    

  - **事务 (Transaction)**

    - **目的**：**原子性保证**。确保一组命令连续执行，不被插队
    - **原子性**：**保证原子性**（批量执行的原子性）。使用 `MULTI` 和 `EXEC` 包裹，命令会连续执行，不会被打断
    - **服务器端**：服务器通过 `MULTI` 知道这是一个事务，会将命令放入队列，等待 `EXEC` 触发
    - **命令执行**：命令是**先入队**（返回 `QUEUED`），在 `EXEC` 时才**一次性执行**

  - 在实际应用中，两者也**可以组合使用**：
    - 即在 `MULTI` 和 `EXEC` 之间，通过 Pipeline 一次性发送所有事务内的命令（`INCRBY`, `DECRBY` 等），这样既保证了原子性，又优化了网络性能



# Redis 高可用与集群

## **主从复制**

> Replication

### 概念：高可用的基础

- **主从复制** 是一种异步复制机制，

  - 它允许一个 Redis 服务器（称为 **从节点/副本 (Slave / Replica)**）成为另一个 Redis 服务器（称为 **主节点 (Master)**）的精确副本

    - **主节点 (Master)**：负责处理**写**命令（`SET`, `INCR` 等），并将数据变更同步给所有连接的从节点

    - **从节点 (Slave/Replica)**：负责接收主节点同步过来的数据。默认情况下，从节点是**只读**的



### 作用

1. **数据冗余备份**

   - **描述**：
     - 这是主从复制最核心的功能。当主节点的数据丢失或损坏时（例如磁盘损坏），从节点上依然保留着一份完整的数据副本，确保数据安全

   

2. **读写分离**

   - **描述**：在“读多写少”的场景下，这是最常用的性能扩展方案
   - **Master 负责写**：所有写操作都发往主节点
   - **Slave 负责读**：所有读操作（`GET`, `LRANGE` 等）都可以分发到多个从节点上执行
   - **效果**：通过将读负载分散到多个从节点，极大地提高了应用的整体QPS（每秒查询率），减轻了主节点的压力

   

3. **高可用性**

   - **描述**：

     - 主从复制本身**不能**自动实现高可用

       但是，它是实现高可用（如哨兵模式）的**基石**

       当主节点宕机时，必须有一个从节点来接替它，这个“接替”的过程需要主从复制作为前提



### 配置

- 配置主从关系非常简单

**方式一：使用命令 (临时生效)**

- 在**从节点**（例如 `127.0.0.1:6380`）的 `redis-cli` 中执行：

  ```cmd
  # 假设主节点 IP 为 192.168.1.100，端口为 6379
  127.0.0.1:6380> REPLICAOF 192.168.1.100 6379
  
  # (旧版命令为 SLAVEOF 192.168.1.100 6379)
  ```



**方式二：修改配置文件 (永久生效)**

- 在**从节点**的 `redis.conf` 文件中添加一行：

  ```cmd
  # 格式: replicaof <masterip> <masterport>
  replicaof 192.168.1.100 6379
  ```

​	修改配置后重启从节点服务



### 工作原理

- 主从复制的过程分为两个阶段：**全量复制 (Full Resync)** 和 **增量复制 (Incremental Sync)**

#### 1. 全量复制 (Full Resync)

- 当一个从节点首次连接到主节点时（或者复制中断后重连），会触发全量复制
  1. **SYNC/PSYNC**：从节点发送 `PSYNC ? -1` 命令给主节点，请求同步数据（这是 Redis 2.8+ 的命令，旧版是 `SYNC`）
  2. **RDB 快照**：主节点收到请求后，执行 `BGSAVE` 在后台**生成一个 RDB 快照文件**
  3. **缓冲区**：在生成 RDB 期间，主节点会将所有新执行的**写命令**缓存到一个“**复制缓冲区 (Replication Buffer)**”中
  4. **发送 RDB**：主节点将生成好的 RDB 文件发送给从节点
  5. **加载 RDB**：从节点接收 RDB 文件，**清空**自己的旧数据，然后**加载** RDB 文件到内存中
  6. **发送缓冲区**：主节点将复制缓冲区中缓存的写命令发送给从节点
  7. **应用命令**：从节点执行这些写命令，完成数据同步



#### 2. 增量复制 (Incremental Sync)

- 全量复制完成后，主从进入**增量复制**阶段，也称为**命令传播 (Command Propagation)**

  - 在此阶段，主节点每执行一个**写命令**，都会**异步**地将这个命令发送给所有从节点

  - 从节点接收并执行这些命令，从而保持与主节点的数据实时（或最终）一致

​	

#### 复制中断与重连 (Partial Resync)

- 如果主从之间的网络连接短暂中断（例如网络抖动），Redis 2.8+ 引入了**部分重同步 (Partial Resynchronization)** 机制
- **复制积压缓冲区 (Replication Backlog)**：主节点维护一个固定大小的环形缓冲区，存储最近发送过的写命令
- **重连**：从节点重连后，会发送 `PSYNC <runid> <offset>` (自己的复制ID和偏移量)
- **判断**：主节点检查从节点的偏移量是否还在积压缓冲区内
  - **是**：执行**增量复制**。主节点只发送中断期间积Z压缓冲区内的命令
  - **否**（中断时间太长，偏移量已不在缓冲区）：触发**全量复制** (Full Resync)



## 哨兵 (Sentinel)

### 概念：高可用“巡逻兵”

- 在“主从复制”模式下，如果主节点 (Master) 发生故障宕机，整个系统将失去**写**能力

  同时，所有从节点 (Slave) 将无法获取新的数据同步

  - 这个故障必须由人工介入，手动将一个从节点提升（promote）为新的主节点，并通知所有其他从节点和客户端切换目标

- **哨兵 (Sentinel)** 是一个独立的进程，它的出现就是为了解决这个“**自动故障转移 (Automatic Failover)**”的问题。它充当一个“巡逻兵”和“决策者”，持续监控整个 Redis 主从系统的健康状况



### 哨兵的作用

- 哨兵系统通常由**多个**哨兵进程组成（为了防止哨兵本身单点故障），它们共同协作完成以下三个核心任务：
  1. **监控 (Monitoring)**
     - 哨兵会持续地向它所监控的主节点和从节点发送 `PING` 命令
     - 如果某个实例在规定时间（`down-after-milliseconds`）内未响应，哨兵会将其标记为“**主观下线 (SDOWN)**”
  2. **选举/通知 (Notification)**
     - 当一个哨兵认为主节点“主观下线”后，它会询问其他哨兵进程对该主节点的看法
     - 如果**超过指定数量**（`quorum`，即法定人数）的哨兵都认为该主节点下线了，该主节点会被标记为“**客观下线 (ODOWN)**”
     - 此时，哨兵们会**选举**出一个“领导者 (Leader)”哨兵，由它来负责执行下一步的故障转移
  3. **自动故障转移 (Automatic Failover)**
     - 这是哨兵**最核心**的功能
     - **(a) 选举新主节点**：领导者哨兵会按照一定的策略（例如：优先级、复制偏移量等）从所有从节点中选举出一个最合适的从节点
     - **(b) 提升为新主**：哨兵向被选中的从节点发送 `REPLICAOF NO ONE` 命令，使其转变成为新的主节点 (Master)
     - **(c) 指向新主**：哨兵向所有其他从节点发送 `REPLICAOF` 命令，让它们转而复制新的主节点
     - **(d) 监控旧主**：哨兵会继续监控那个已经宕机的旧主节点。如果它某天恢复了，哨兵会向它发送 `REPLICAOF` 命令，使其降级为新主节点的从节点



### 实现高可用：主从 + 哨兵

- 一个典型的高可用架构是：**1个 Master + N个 Slave + M个 Sentinel**（M通常 >= 3）
  - **为什么哨兵至少要 3 个？**
    - 这是为了保证“选举”的 `quorum`（法定人数）机制能正常工作
    - 如果只有 1 个哨兵，它自己宕机了，系统就瘫痪了
    - 如果只有 2 个哨兵，当网络分区（脑裂）时，两个哨兵可能各自持有一票，无法达成共识（例如 `quorum=2`，两个哨兵互相认为对方掉线）
    - **推荐 3 个或 5 个**这样的奇数个哨兵实例。例如 3 个哨兵，`quorum` 设置为 2
      - 即使有 1 个哨兵宕机，剩下的 2 个哨兵依然可以达成共识（2 >= 2），完成故障转移



### 客户端如何连接？

- 在这种模式下，客户端**不应该**直接连接 Redis 主节点的固定 IP

  - 客户端应该连接到**哨兵的地址**

    客户端启动时，会向哨兵询问“当前的主节点是谁？”

    哨兵会将最新的主节点地址（例如 `192.168.1.100:6379`）返回给客户端

    - 当发生故障转移后，哨兵会检测到主节点地址发生了变化（例如变成了 `192.168.1.101:6379`）

      客户端（或哨兵）会收到这个变更通知，自动将后续的写操作切换到新的主节点上，实现对应用层的透明切换



## 集群 (Cluster)

### 概念：分布式解决方案

- Redis 集群 (Redis 3.0+) 是 Redis 官方提供的**分布式 (Distributed)** 数据库方案
  - 它通过**数据分片 (Sharding)** 将数据自动分散存储在多个 Redis 节点上，同时提供了高可用性（主从复制和故障转移）

- 集群解决了单机 Redis 在**内存容量**和**并发性能**上的瓶颈



### 核心作用

1. **数据分片 (Sharding)**
   - **描述**：集群不再是将所有数据存在一个主节点上，而是将整个数据集自动分割成多片，存储在多个不同的主节点上
   - **例如**：你有 30GB 数据，集群可以将它分散到 3 个主节点上，每个主节点只负责存储 10GB 的数据
2. **高可用 (High Availability)**
   - **描述**：集群模式**内置了主从复制和故障转移**（类似哨兵的功能）
   - 集群中的每个主节点 (Master) 都可以有自己的一个或多个从节点 (Slave)
   - 当某个主节点宕机时，它的一个从节点会自动被提升为新的主节点，继续提供服务。集群不需要依赖外部的哨兵进程



### 核心原理：哈希槽 (Hash Slot)

- 为了实现数据分片，集群引入了“哈希槽”的概念
  1. **16384 个哈希槽**
     - Redis 集群将整个数据集（所有的 Key）映射到了一个固定大小的 **16384** 个“槽 (Slot)”中
     - 启动集群时，这 16384 个槽会被平均分配给集群中所有的**主节点**
     - 例如：3 个主节点（M1, M2, M3）
       - M1 负责槽：0 - 5460
       - M2 负责槽：5461 - 10922
       - M3 负责槽：10923 - 16383
  2. **Key 如何映射到槽？**
     - 当客户端需要操作一个 Key（例如 `SET name "gemini"`）时，集群会使用一个算法来计算这个 Key 应该属于哪个槽：
     - `slot = CRC16(key) % 16384`
     - 例如，`CRC16("name") % 16384` 的结果可能是 `5110`
     - 集群会查找 `5110` 号槽由哪个主节点（M1）负责，然后将这个命令转发到 M1 上去执行
  3. **客户端重定向 (MOVED)**
     - 客户端**可以连接到集群中的任意一个节点**
     - 如果客户端连接的节点**不是**该 Key 对应的正确节点，服务器会返回一个 `MOVED` 错误，告诉客户端这个 Key 应该由哪个 IP 和端口（正确的主节点）来处理
     - `MOVED 5110 192.168.1.100:6379`
     - 智能的客户端（如 Jedis, Redisson）会自动处理这个重定向，并缓存槽与节点的映射关系，下次直接访问正确的节点



### 节点通信：Gossip 协议

- 集群中的多个节点（例如 6 个节点：3 主 3 从）是如何知道彼此的状态（谁存了哪些槽？谁宕机了？）

  - **Gossip 协议**：节点之间通过一种点对点的通信方式（称为 Gossip 协议）来交换集群的状态信息

  - **节点心跳 (Ping/Pong)**：每个节点都会定期向其他节点发送 `PING` 包，接收方回复 `PONG`

  - **共享信息**：在 `PING/PONG` 包中，节点会携带自己知道的集群状态信息（例如它认为谁下线了，谁的槽位变更了）

  - **最终一致**：通过这种方式，信息会在集群中“流传”开来，最终所有节点都会对整个集群的状态达成一致。这也包括了故障检测和新主节点的选举



### 搭建和管理

- **最小集群**：搭建一个可用的 Redis 集群，至少需要 **3 个主节点**（才能保证高可用和选举）
- **推荐配置**：官方推荐使用 **6 个节点**，即 **3 个主节点 (M1, M2, M3)** 和 **3 个从节点 (S1, S2, S3)**，S1 复制 M1，S2 复制 M2，依此类推
- **管理**：使用 `redis-cli --cluster` 命令来创建、检查、添加节点或重新分片



# 缓存设计与实战

## 1. 缓存的意义

在现代应用架构中，数据通常存储在磁盘上的数据库中 (如 MySQL)。磁盘 I/O 的速度远慢于内存
- **缓存 (Cache)** 是一种高速的数据存储层，它存储了数据的子集（通常是访问最频繁的数据）



### 为什么需要缓存？

1. **加速读写 (高性能)**

   - **描述**：

     - 内存的读写速度比磁盘快几个数量级

       Redis 这种内存数据库的 QPS（每秒查询率）可以达到数万甚至十万以上，而传统关系型数据库（如 MySQL）通常在几千 QPS

   - **效果**：将热点数据（如首页信息、热门商品）放入 Redis 缓存，可以极大提高应用的响应速度，降低延迟

2. **降低后端负载 (高并发)**

   - **描述**：

     - 在高并发场景下（如秒杀、热点新闻），绝大多数请求都是读请求

       如果这些请求全部直接访问数据库，数据库会因为连接数过多或 I/O 压力过大而崩溃

   - **效果**：

     - 缓存层（如 Redis）可以承受远高于数据库的并发读请求，它作为应用和数据库之间的屏障，只将缓存中没有的请求（少量）透传给数据库，保护了后端系统



## 2. 缓存更新策略

- 当数据发生变化时（例如用户修改了个人信息），我们必须同时更新数据库和缓存，以保证数据一致性。以下是几种主流的策略

### (1) Cache-Aside (旁路缓存) - 最常用

- 这是实际开发中**使用最广泛**的策略。应用层（你的代码）需要同时维护数据库和缓存

  - **读操作 (Read)**：

    1. 应用先从缓存中读取数据
    2. 如果缓存命中 (Hit)，则直接返回数据
    3. 如果缓存未命中 (Miss)，则应用**从数据库中读取**数据
    4. 读取成功后，将数据**写入缓存**
    5. 返回数据

    

  - **写操作 (Write)**：

    - **策略：先更新数据库，再删除缓存 (Delete Cache)**

    1. 应用将新数据**写入数据库**
    2. **成功**后，应用向 Redis 发送命令**删除**对应的缓存 Key

    

  - **为什么是“删除”缓存，而不是“更新”缓存？**
    - **懒加载 (Lazy Loading)**：
      - 删除缓存可以保证下次读取该数据时，一定会从数据库加载最新的数据（见“读操作”第3步），确保数据被动更新
    - **性能开销**：如果每次更新都去更新缓存，而这个数据可能短期内不会被访问，这就造成了不必要的写缓存开销。



### (2) Read-Through (穿透缓存)

- **描述**：应用只与缓存交互。由缓存层（例如 Redis 插件或特定的缓存服务）负责在未命中时从数据库加载数据
- **优点**：应用层代码简单
- **缺点**：Redis 原生不支持，需要额外组件或自定义实现，不常用



### (3) Write-Through (写穿透)

- **描述**：应用只与缓存交互。当应用更新缓存时，由缓存层负责**同步**将数据写入数据库
- **优点**：保证缓存和数据库的强一致性
- **缺点**：写入性能较低（必须等待数据库写入完成）



### (4) Write-Back (写回)

- **描述**：应用只更新缓存。缓存层会**异步**、批量地将数据刷回数据库（例如每 5 秒刷一次）
- **优点**：写入性能极高（因为只操作内存）
- **缺点**：
  - 数据一致性最差。如果 Redis 宕机，会丢失最后一次刷盘前的所有数据。常用于对数据一致性要求不高的场景（如统计访问量）



## 3. 缓存常见问题与解决方案

### (1) 缓存穿透 (Cache Penetration)

- **现象**：
  - 客户端（或恶意攻击者）**查询一个数据库中根本不存在的数据**（例如查询 `userID = -1`）
  - 由于缓存中也没有这个数据（缓存未命中）
  - 导致该请求**每次**都会穿过缓存，直接访问数据库
  - 如果大量此类请求涌入，数据库压力剧增，可能导致崩溃
- **解决方案**：
  1. **缓存空对象 (Cache Nulls)**
     - **描述**：当数据库查询不到数据时，仍然将一个“空值”（例如 `null` 或一个特定的空字符串）**写入缓存**
     - **设置较短的过期时间**（例如 60 秒），防止这个空值永久存在
     - **优点**：实现简单，效果好
     - **缺点**：需要额外占用缓存空间；可能存在短期的数据不一致（60秒内如果数据库添加了该数据）
  2. **布隆过滤器 (Bloom Filter)**
     - **描述**：在缓存层前再加一道屏障。布隆过滤器是一种高效的数据结构，用于**快速判断一个元素是否“一定不存在”**
     - **流程**：将数据库中所有合法的 Key 提前加载到布隆过滤器中。当请求到来时：
       1. 先问布隆过滤器：“这个 Key 是否存在？”
       2. 如果过滤器说“**一定不存在**”，则**直接拒绝请求**，不再查询缓存和数据库
       3. 如果过滤器说“**可能存在**”，则放行请求，继续走后续的缓存查询流程
     - **优点**：内存占用极小，查询效率高，能有效拦截恶意攻击
     - **缺点**：实现复杂；有极低的误判率（Bloom Filter 的特性）（通常使用 Redis 模块 `RedisBloom` 来实现）



### (2) 缓存击穿 (Cache Breakdown)

- **现象**：
  - 某一个**热点 Key**（例如爆款商品的详情页）承载着极高的并发访问
  - 在这个 Key **过期失效的瞬间**，成千上万的并发请求同时涌入
  - 由于缓存未命中，这些请求**全部**穿透到数据库，导致数据库瞬间崩溃
  - 这就像在一个点上（热点 Key）打穿了缓存
- **解决方案**：
  1. **互斥锁 (Mutex Lock)**
     - **描述**：在缓存失效时，只允许**第一个**获取到数据的线程去查询数据库，其他线程**原地等待**
     - **流程**（使用 `SETNX` 实现分布式锁）：
       1. 线程A发现缓存未命中，尝试 `SETNX lock:key "1" EX 60` 获取锁
       2. 线程A获取成功，立即去数据库查询数据，然后写回缓存，最后释放锁（`DEL lock:key`）
       3. 在线程A执行期间，线程B、C、D也来访问，它们获取锁失败（`SETNX` 返回 0）
       4. 线程B、C、D 不去查数据库，而是进入等待（例如 `sleep(50ms)`）后**重试**（重新查询缓存）
       5. 当线程A释放锁后，线程B、C、D 再次查询缓存时，就能命中数据
     - **优点**：严格保证一致性，有效防止数据库被冲垮
     - **缺点**：实现复杂；引入了锁竞争，可能导致部分线程等待时间变长，吞吐量下降
  2. **热点数据永不过期（逻辑过期）**
     - **描述**：物理上不给热点 Key 设置 TTL（`EXPIRE`）
     - **在 Value 中存储一个逻辑过期时间**（例如 `{"data": {...}, "expire_at": 1678888888}`）
     - **流程**：
       1. 线程A发现缓存命中，但检查到 `expire_at` 已过期
       2. 线程A**立刻返回旧数据**（保证用户能看到内容），同时**异步**开启一个新线程去数据库加载新数据并写回缓存（更新 `expire_at`）
     - **优点**：性能极好，没有锁等待
     - **缺点**：数据不是最新的（返回的是旧数据）；实现也较复杂



### (3) 缓存雪崩 (Cache Avalanche)

- **现象**：
  - **同一时间，大量的 Key 集体失效**（例如：为图方便，给所有 Key 设置了相同的过期时间 `EXPIRE 3600`）
  - 或者 **Redis 服务自身宕机**
  - 导致在某一瞬间，绝大多数请求都无法命中缓存，所有请求如洪水般涌向数据库，导致数据库崩溃
- **解决方案**：
  1. **设置随机过期时间 (针对情况一)**
     - **描述**：在设置 Key 的过期时间时，**添加一个随机数**，防止集体失效
     - **例如**：
       - 原定过期时间 1 小时 (3600 秒)，改为 `EXPIRE key (3600 + random(0, 300))`，即在 3600 到 3900 秒之间随机过期
     - **效果**：将过期时间点打散，避免了瞬时压力
  2. **使用高可用集群 (针对情况二)**
     - **描述**：使用**主从+哨兵**或 **Redis Cluster** 来保证 Redis 服务的健壮性
     - **效果**：即使一个 Redis 节点宕机，其他节点（从节点或集群其他主节点）可以立即接管服务，避免整个缓存层失效
  3. **多级缓存 / 熔断限流**
     - **描述**：
       - 在应用层增加本地缓存（如 Caffeine, Guava Cache）；或者在网关层设置限流（如 Sentinel），当数据库压力过大时，直接熔断或拒绝部分请求，保证核心服务可用



## 4. 内存淘汰策略 (Eviction Policies)

### 什么是内存淘汰？

- Redis 是基于内存的，当 Redis 占用的内存达到了 `maxmemory` 配置的上限时，必须选择一些数据进行**淘汰**（删除），以便为新写入的数据腾出空间

### 策略 (如何选择？)

- 在 `redis.conf` 中通过 `maxmemory-policy` 配置：

  1. **noeviction (默认策略)**

     - **描述**：不淘汰任何数据。当内存达到上限时，所有**写命令**都会返回错误 (OOM error)
     - **适用**：适用于数据不能丢失的场景（例如 Redis 作为主数据库）

     

  2. **allkeys-lru (最常用)**

     - **描述**：**LRU (Least Recently Used)** 算法。从**所有** Key 中，找到**最近最少使用**的 Key 进行淘汰
     - **适用**：**最推荐的策略**。符合缓存的“热点数据”特性（最近访问多的数据，未来被访问的概率也高）

     

  3. **volatile-lru**

     - **描述**：LRU 算法。但只从**设置了过期时间 (EXPIRE)** 的 Key 中进行淘汰
     - **适用**：如果你希望那些“永久”数据（没设置过期时间）永远不被淘汰时使用

     

  4. **allkeys-random**

     - **描述**：从**所有** Key 中**随机**淘汰
     - **适用**：如果你的数据访问没有明显的热点区分

     

  5. **volatile-random**

     - **描述**：只从**设置了过期时间**的 Key 中**随机**淘汰

     

  6. **volatile-ttl**

     - **描述**：只从**设置了过期时间**的 Key 中，找到**即将过期**（TTL 最小）的 Key 进行淘汰



