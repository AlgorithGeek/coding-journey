# Elasticsearch

## 01. Elasticsearch 架构原理与核心概念

### 1.1 为什么我们需要 Elasticsearch？

在 Java Web 开发中，MySQL 是我们最熟悉的各种业务数据（订单、用户、商品）的存储主力。但在面对海量数据的 **“搜索”** 需求时，MySQL 往往力不从心

- 我们通过一个具体的电商场景来推演这个痛点

#### 场景设定

假设你正在维护一个电商系统的核心数据库，其中有一张 `product`（商品）表，数据量已经增长到了 **1 亿条**。 表结构简化如下：

- `id` (主键)
- `name` (商品名称，例如：“华为 Mate 60 Pro 5G 手机”)
- `description` (商品详情描述，长文本)



#### 痛点

##### 痛点一：模糊查询导致的全表扫描

**需求**：用户在搜索框输入关键词 **“华为手机”**

- **MySQL 方案**： 通常我们会使用 SQL 的 `LIKE` 语句：

  ```sql
  SELECT * FROM product WHERE name LIKE '%华为手机%';
  ```



**问题分析**：

1. **索引失效**：在 MySQL 的 B+Tree 索引机制中，**左模糊匹配**（即 `%` 在开头的 `%xx`）会导致索引失效
2. **全表扫描**：因为无法利用索引，数据库必须把这 **1 亿条** 数据从磁盘加载到内存，逐行对比 `name` 字段是否包含“华为手机”这四个字
3. **后果**：
   - **I/O 爆炸**：磁盘 I/O 瞬间打满
   - **响应极慢**：查询耗时可能高达几十秒甚至超时
   - **服务雪崩**：高并发下，大量慢查询会直接拖垮整个数据库，导致正常下单业务也无法进行



##### 痛点二：无法理解自然语言

**需求**：用户输入关键词 **“华为5G手机”**（注意中间加了“5G”）

**MySQL 方案**：

```sql
SELECT * FROM product WHERE name LIKE '%华为5G手机%';
```

**问题分析**：

1. **机械匹配**：MySQL 的 `LIKE` 是绝对精准的字符串子串匹配
2. **漏查数据**：
   - 如果数据库里有一条商品叫 **“华为 Mate 60 Pro 5G 智能手机”**，虽然它包含“华为”、“5G”、“手机”，但因为它的语序或者中间夹杂了其他字符，`LIKE '%华为5G手机%'` 根本匹配不到这条记录
3. **分词缺失**：MySQL 不知道“华为5G手机”其实应该拆解为 `华为` + `5G` + `手机` 三个关键词来分别匹配



##### 痛点三：缺乏相关度排名

**需求**：搜索结果应该按照“匹配程度”排序，名字里出现关键词次数多的、位置靠前的应该排在前面

**MySQL 方案**： MySQL 只能按照字段（如价格、创建时间）排序，无法计算文本的 **相关度分数**。它无法告诉你哪条记录更“符合”用户的搜索意图



#### 结论

MySQL 擅长 **事务处理 (ACID)** 和 **精确查询**（如 `WHERE id = 1`），但在 **全文检索**、**模糊匹配** 和 **海量数据分析** 场景下，它是极其低效的

这也是为什么我们需要引入 **Elasticsearch** —— 它不是要取代 MySQL，而是作为 MySQL 在“搜索能力”上的 **外挂**





### 1.2 核心原理：倒排索引 (Inverted Index)

Elasticsearch 之所以快，根本原因在于它存储数据的方式与传统数据库截然不同。它使用的是一种被称为 **倒排索引** 的设计

要理解“倒排”，我们先看看什么是“正排”

#### 1. 正排索引 (Forward Index)

> MySQL 的方式

- **结构**：`Document ID -> Content`
- **逻辑**：通过 ID 找内容
- **类比**：就像你拿着一本书，从第一页（ID=1）开始读，看这一页里有没有写“Java”。如果书有 1000 页，你就得翻 1000 次
- **局限**：这就是 MySQL `LIKE` 查询慢的根本原因——必须遍历所有内容



#### 2. 倒排索引 (Inverted Index)

> ES 的方式

- **结构**：`Keyword (Term) -> List<Document ID>`

- **逻辑**：通过 关键词 找 ID

- **类比**：

  - 就像书本末尾的 **“关键词索引页”** 

    你想找“Java”，直接翻到索引页的 `J` 栏，上面写着：“Java 出现在第 5、12、108 页”。你不需要读完整本书，直接翻到这三页即可



#### 3. 图解构建过程

假设我们有两行原始数据（Document）：

| Doc ID | Content (Text) |
| ------ | -------------- |
| **1**  | "Java is good" |
| **2**  | "PHP is good"  |

当这两条数据存入 ES 时，会经历以下两个步骤：

**Step 1: Analysis (分词)** ES 会使用分词器（Analyzer）将文本拆解成独立的单词（Term）

- Doc 1 -> `Java`, `is`, `good`
- Doc 2 -> `PHP`, `is`, `good`



**Step 2: Indexing (建立索引)** ES 会建立一张映射表，记录每个单词出现在哪些文档里：

| Term (词条) | Posting List (倒排表 - 记录 ID) |
| ----------- | ------------------------------- |
| **Java**    | `[1]`                           |
| **PHP**     | `[2]`                           |
| **is**      | `[1, 2]`                        |
| **good**    | `[1, 2]`                        |



#### 4. 搜索流程对比

当用户搜索 **"Java"** 时：

1. **MySQL**：遍历 Doc 1，判断包含 Java？是。遍历 Doc 2，判断包含 Java？否。遍历 Doc 3...
2. **Elasticsearch**：
   - 直接去 **Term** 列查找 "Java"
   - 瞬间定位到 "Java" 对应的 Posting List 是 `[1]`
   - **直接返回** ID 为 1 的文档

无论你有 1 亿条数据还是 10 亿条数据，查找 "Java" 这个词的时间几乎是恒定的（取决于该词对应的 ID 列表长度，而不是数据总量）



#### 💡 关键特性与代价

1. **Immutable (不可变性)**：

   - 倒排索引一旦写入磁盘，就是 **不可变** 的。这带来了极高的读取并发性能（不需要锁），但也意味着 ES **不适合频繁修改**

     > *注：ES 的更新操作（Update）本质上是“标记删除旧文档 + 插入新文档”，成本较高*

2. **空间换时间**：

   - 为了支持极速搜索，ES 需要建立大量的索引文件，通常索引文件的大小会接近甚至超过原始数据的大小（这也是为什么 ES 比较吃磁盘空间）



### 1.3 逻辑概念与物理架构

在学习 ES 时，初学者最容易晕头转向的就是各种名词。为了快速上手，我们可以先将其与关系型数据库（MySQL）进行类比，但必须注意它们本质上的区别



#### 1. 核心数据模型 (逻辑层)

这是开发过程中每天都会接触到的概念

| Elasticsearch 概念  | 对应 MySQL 概念     | 说明                                                         |
| ------------------- | ------------------- | ------------------------------------------------------------ |
| **Index (索引)**    | **Table (表)**      | 数据的逻辑容器。例如 `product_index`<br />*注：在 6.x 之前的旧版本中，Index 常被比喻为 Database，但现在更接近 Table。* |
| **Document (文档)** | **Row (行)**        | 一条具体的 JSON 数据。例如 `{"id":1, "name":"Phone"}`。ES 是面向文档的存储 |
| **Field (字段)**    | **Column (列)**     | JSON 中的 Key。例如 `name`、`price`                          |
| **Mapping (映射)**  | **Schema (表结构)** | 定义字段名称及其类型(Text, Keyword, Long, Date 等).虽然 ES 支持自动推断，但生产环境 **强烈建议手动定义** |
| **Type (类型)**     | **(已废弃)**        | **⚠️ 重要避坑**：在 ES 6.x 中一个 Index 可以有多个 Type，但 ES 7.x 之后彻底废弃了 Type 概念（默认为 `_doc`），ES 8.x 已完全移除。**请不要在设计中再引入 Type** |



#### 2. 分布式架构 (物理层)

ES 天生就是分布式的，这是它处理 PB 级数据、实现高可用的物理基础

##### (1) Node (节点)

- **定义**：一台运行 Elasticsearch 进程的服务器（或者 Docker 容器）
- **唯一标识**：每个节点都有一个名字，默认随机生成，生产环境建议配置有意义的名字
- **角色**：
  - **Master Node**：大脑。负责管理集群状态（创建/删除索引、分配分片、维护元数据）
  - **Data Node**：苦力。负责真正存储数据，执行 CRUD、搜索和聚合操作。**这部分对 CPU、内存和 I/O 要求极高**



##### (2) Cluster (集群)

- 由一个或多个 Node 组成，它们配置了相同的 `cluster.name`
- 集群会自动协同工作，对外提供统一的服务接口



##### (3) Shards (分片) —— ⭐ 核心难点

为了解决单机存储容量的上限（比如硬盘只有 1TB，但数据有 10TB），ES 将一个 Index 切分成多个部分，每一部分叫一个 **Shard**

- **Primary Shard (主分片)**：

  - 数据的“本体”
  - **不可变规则**：索引创建时必须指定主分片数量（默认为 1），且 **创建后无法修改**
  - *为什么不能改？* 因为文档到分片的路由算法是 `hash(id) % 主分片数`，如果分片数变了，路由规则就乱了，数据就找不到了

  

- **Replica Shard (副本分片)**：

  - 主分片的“备份”
  - **作用**：
    1. **高可用 (HA)**：如果存放主分片的节点宕机，集群会自动将某个副本提升为新主，数据不丢失
    2. **提升读性能**：搜索请求可以在主分片或任意副本上并行执行，分担查询压力（Scale Out）
  - **灵活规则**：副本数量可以随时动态修改



#### 💡 架构图解 (示例)

假设有一个集群（3 个节点），索引 `product_index` 配置了 **3 个主分片，1 个副本**（每个主分片有 1 个备份）

- **分布情况**：
  - 总共有 6 个分片块（3 主 + 3 副）
  - ES 会自动将这 6 个块尽可能均匀地分散在 3 个节点上
  - **强制规则**：同一个分片的主分片和副本分片，**绝对不会** 出现在同一个节点上（否则节点挂了，主副全没，备份就失效了）



#### 🚀 生产环境最佳实践

1. **分片大小**：单个分片的大小建议控制在 **30GB - 50GB**。太小会导致元数据管理开销大，太大会导致恢复时间极长
2. **规划前置**：因为主分片无法修改，上线前必须根据预估数据量做好容量规划





## 02. 环境搭建与 IK 分词器配置 (1/N)

### 2.1 基于 Docker Compose 快速搭建

在学习阶段，我们不建议在本机直接安装庞大的 ES 安装包，那样会污染环境且难以管理。Docker 是最佳选择

#### 1. 准备工作

- 确保电脑已安装 **Docker** 和 **Docker Compose**
- 我们选用的版本是 **Elasticsearch 8.12.2** (8.x 是当前主流稳定版)



#### 2. 编写 docker-compose.yml

请在一个空文件夹下新建 `docker-compose.yml` 文件，并粘贴以下内容：

```yaml
version: '3.8'

services:
  # Elasticsearch 节点
  es:
    image: elasticsearch:8.12.2
    container_name: es-node
    environment:
      - node.name=es-node
      - cluster.name=es-docker-cluster
      - discovery.type=single-node  # 开发环境运行单节点模式
      - bootstrap.memory_lock=true  # 禁止内存交换，保证性能
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" # 限制内存，防止电脑卡死
      - xpack.security.enabled=false    # 学习阶段关闭安全认证(HTTPS/密码)，简化操作
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./es-data:/usr/share/elasticsearch/data # 数据持久化挂载
      - ./es-plugins:/usr/share/elasticsearch/plugins # 插件挂载(后续装IK用)
    ports:
      - "9200:9200" # HTTP 端口，用于 Rest API
      - "9300:9300" # TCP 端口，集群通信用
    networks:
      - es-net

  # Kibana 可视化界面
  kibana:
    image: kibana:8.12.2
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://es-node:9200 # 指向 ES 容器
      - I18N_LOCALE=zh-CN # 开启 Kibana 中文汉化
    networks:
      - es-net
    depends_on:
      - es # 确保 ES 先启动

networks:
  es-net:
    driver: bridge
```



#### 3. 关键配置详解 (避坑指南)

1. **`ES_JAVA_OPTS=-Xms512m -Xmx512m`**:
   - **为什么？** ES 默认非常吃内存（默认可能占用 1GB+）。在个人电脑上学习，必须手动限制堆内存大小，否则可能导致电脑卡顿
2. **`xpack.security.enabled=false`**:
   - **为什么？** ES 8.x 默认开启强安全模式（HTTPS + 用户密码）。为了降低学习门槛，专注于 DSL 语法本身，我们暂时关闭它，避免处理繁琐的 SSL 证书问题
3. **`discovery.type=single-node`**:
   - **为什么？** 告诉 ES “我只是一个单机节点，别去尝试寻找其他队友了”，能加快启动速度，避免报错



#### 4. 启动与验证

在终端进入该文件所在目录，执行：

```bash
docker-compose up -d
```

等待约 30 秒后（ES 启动较慢），进行验证：

- **验证 ES**: 浏览器访问 `http://localhost:9200`
  - 如果你看到一段包含 `"tagline": "You Know, for Search"` 的 JSON，说明 ES 活了
- **验证 Kibana**: 浏览器访问 `http://localhost:5601`
  - 加载完成后，点击左侧菜单的 **Dev Tools (开发者工具)**，这里就是我们接下来敲 DSL 代码的主战场



#### 5. 常见错误排查

如果启动失败（ES 容器自动退出），通常是因为 Linux/Mac/Windows WSL 的 **虚拟内存映射限制** 太小。 **解决方法**（Linux/WSL 用户需执行）：

```bash
sysctl -w vm.max_map_count=262144
```





### 2.2 解决中文搜索痛点：IK 分词器

Elasticsearch 自带一个 **Standard Analyzer**（标准分词器），它对英文支持很好，但对中文非常“傻”

#### 1. 为什么必须装插件？(Why)

我们要知其然，知其所以然。在安装之前，先在 Kibana Dev Tools 里运行这个测试，看看原生 ES 是怎么处理中文的：

```js
# 测试原生分词器
POST /_analyze
{
  "analyzer": "standard",
  "text": "华为手机"
}
```

**原生结果**： 它会将“华为手机”拆解成 4 个独立的字：`华`、`为`、`手`、`机`

- **痛点**：当你搜“华为”时，ES 可能会给你返回包含“中华有为”或者“手办”的文档，因为它只是匹配了单字，根本不懂“华为”是一个词



#### 2. 安装 IK Analysis 插件 (Action)

**IK 分词器** 是目前 Java 社区最主流的 ES 中文分词插件

由于我们是 Docker 环境，安装步骤如下：

**Step 1: 进入容器内部** 在终端执行：

```bash
docker exec -it es-node bash
```



**Step 2: 在线安装插件** 在容器内部终端执行（注意版本必须与 ES 版本严格一致，我们是 8.12.2）：

```bash
./bin/elasticsearch-plugin install [https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.12.2/elasticsearch-analysis-ik-8.12.2.zip](https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.12.2/elasticsearch-analysis-ik-8.12.2.zip)
```

*看见 `-> Installed analysis-ik` 提示即表示成功*



**Step 3: 退出并重启容器** 执行 `exit` 退出容器，然后重启 ES 节点让插件生效：

```bash
docker restart es-node
```

*(重启需要几十秒，请耐心等待)*



#### 3. IK 的两种核心模式 (API)

安装好后，IK 提供了两个全新的分词器。这是面试和实战中最高频的考点：

1. **`ik_smart` (智能最少切分)**
   - **特点**：颗粒度粗，只切分出最常用的词，尽量不重复
   - **场景**：通常用于 **搜索框的输入** （用户搜什么，就切成什么）
2. **`ik_max_word` (最细粒度切分)**
   - **特点**：颗粒度细，会穷尽文本中所有可能的词汇组合
   - **场景**：通常用于 **构建索引**（为了让文档能被更多关键词搜到）



#### 4. 实战对比 (Code)

请在 Kibana Dev Tools 中分别运行以下两段 DSL：

**测试 A：使用 ik_smart**

```js
POST /_analyze
{
  "analyzer": "ik_smart",
  "text": "华为5G智能手机"
}
```

- **结果**：`[华为, 5G, 智能手机]`
- **评价**：干净利落，符合人类直觉



**测试 B：使用 ik_max_word**

```js
POST /_analyze
{
  "analyzer": "ik_max_word",
  "text": "华为5G智能手机"
}
```

- **结果**：`[华为, 5G, 智能手机, 智能, 手机]`
- **评价**：它把“智能手机”拆解成了“智能”和“手机”，这样用户搜“智能”也能搜到这条数据



#### 5. 扩展词库

- **痛点**：如果在这个月发售了新词（比如“遥遥领先”），IK 默认词库里没有，它还是会被切成单字
- **解决**：IK 支持配置扩展词典（`IKAnalyzer.cfg.xml`），你可以维护一份 `ext.dic` 文件，将公司业务的专有名词加进去，确保分词准确



## 03. 索引与文档的 CRUD 操作

### 3.1 索引管理与 Mapping 设计 (核心)

在 MySQL 中，我们建表时必须指定字段类型

- 在 ES 中，虽然它支持“无模式”（Schema-less，即不建表直接存数据），但在生产环境中，**严禁** 依赖自动 Mapping

  **必须**手动设计 Mapping，否则后期会遇到无法排序、无法聚合或类型冲突的灾难性后果

#### 1. 创建索引 (Create Index)

我们来创建一个电商商品的索引 `products`

- **API**: `PUT /<index_name>`
- **Context**: 设定主分片数、副本数，并详细定义每个字段的类型

请在 Kibana Dev Tools 中执行：

```js
PUT /products
{
  "settings": {
    "number_of_shards": 1,    // 学习环境设为 1，生产环境根据数据量规划
    "number_of_replicas": 0   // 单机环境设为 0，否则索引状态会变黄(Yellow)
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "long"
      },
      "title": {
        "type": "text",             // 用于全文检索（会被分词）
        "analyzer": "ik_max_word",  // 写入时：最细粒度分词
        "search_analyzer": "ik_smart" // 搜索时：智能粗粒度分词（最佳实践）
      },
      "category": {
        "type": "keyword"           // 用于精确匹配、聚合、排序（不分词）
      },
      "price": {
        "type": "double"
      },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||strict_date_optional_time||epoch_millis"
      },
      "is_active": {
        "type": "boolean"
      }
    }
  }
}
```

**返回结果**：

```js
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "products"
}
```



#### 2. Mapping 核心数据类型 (API)

| 类型             | 说明           | 适用场景               | 避坑指南                                                     |
| ---------------- | -------------- | ---------------------- | ------------------------------------------------------------ |
| **text**         | **文本类型**   | 文章内容、商品标题     | **会被分词**。不支持聚合、排序。必须指定 `analyzer` (如 `ik_max_word`) |
| **keyword**      | **关键字类型** | 手机号、状态枚举、分类 | **不分词**，原样存储。用于 `term` 查询、`aggs` 聚合、`sort` 排序 |
| **long/integer** | **数值类型**   | ID、库存               | 尽量选择够用的最小类型以节省空间                             |
| **date**         | **日期类型**   | 创建时间               | **巨坑**：ES 底层存的是毫秒级时间戳。建议显式指定 `format`，否则传 "2023-01-01" 可能会报错 |
| **boolean**      | **布尔类型**   | 是否上架               | 接受 `true`/`false` 或 `"true"`/`"false"` 字符串             |



#### 3. 查看与删除 (Read & Delete)

- **查看索引结构**： 当你接手一个新项目，想知道某个索引怎么设计的，用这个：

  ```js
  GET /products
  ```

- **删除索引**： **高危操作！** 这相当于 MySQL 的 `DROP TABLE`，数据瞬间清空且无法恢复

  ```js
  DELETE /products
  ```



#### 4. 避坑指南：Mapping 的不可变性

这是 ES 新手最容易踩的 **天坑**

- **规则**：**索引一旦创建，已有的字段定义（Mapping）绝对不能修改！**
- **Why?** 假如 `title` 字段原本是 `keyword` 类型，倒排索引里存的是 "华为手机" 整个词。如果你把它改成 `text` 类型，它应该被拆成 "华为"、"手机"。但倒排索引已经生成了，ES 不可能去重构已有的底层文件
- **场景**： 如果你发现字段类型定义错了（比如 `price` 定义成了 `text`），或者需要修改分词器
- **解决方案 (Reindex)**：
  1. 创建一个配置正确的新索引 `products_v2`
  2. 使用 `_reindex` API 把数据从 `products` 迁移到 `products_v2`
  3. 删除旧索引
  4. （可选）设置别名（Alias）让应用层无感知

```js
# Reindex 示例（了解即可）
POST /_reindex
{
  "source": { "index": "products" },
  "dest": { "index": "products_v2" }
}
```



### 3.2 文档的增删改查 (CRUD)

有了索引 `products`，我们现在开始对数据进行操作

#### 1. 新增文档 (Create)

在 ES 中，写入数据被称为 **Indexing（索引化）**

- **场景 A：指定 ID 写入 (PUT)**

  - **适用**：你自己的业务表已有主键（如 MySQL 的 `id`），希望 ES 的 `_id` 与之保持一致
  - **语法**：`PUT /<index>/_doc/<id>`

  ```js
  PUT /products/_doc/1001
  {
    "id": 1001,
    "title": "华为 Mate 60 Pro 5G 手机",
    "category": "手机",
    "price": 6999.00,
    "created_at": "2023-09-01 12:00:00",
    "is_active": true
  }
  ```



- **场景 B：自动生成 ID 写入 (POST)**
  - **适用**：日志数据、流水记录，不关心 ID 是什么
  - **语法**：`POST /<index>/_doc` (注意这里没有 ID)
  - **结果**：ES 会生成一个类似 `wM0q44kB...` 的随机字符串作为 `_id`



#### 2. 查询文档 (Read)

- **根据 ID 精确查询**： 这是最快的查询方式（类似于 MySQL 的 `SELECT * FROM ... WHERE id = ...`）

  ```js
  GET /products/_doc/1001
  ```

- **验证是否存在 (HEAD)**： 如果你只想检查 ID 是否存在，不需要看内容（节省带宽）：

  ```js
  HEAD /products/_doc/1001
  ```

  - **返回 200 OK**：存在
  - **返回 404 Not Found**：不存在



#### 3. 更新文档 (Update) —— 重点与坑

ES 的更新操作有两种完全不同的模式，混用会导致严重的性能问题或数据丢失

- **模式 A：全量替换 (PUT)**

  - **原理**：和“新增”的语法一模一样。如果 ID 1001 已存在，ES 会 **先删除旧文档，再创建新文档**
  - **坑**：如果你只传了 `price` 字段，其他字段（如 `title`）会 **全部丢失**！它不是修改，是 **替换**

  

- **模式 B：局部更新 (POST _update)**

  - **推荐**：这是真正的“修改”
  - **语法**：`POST /<index>/_update/<id>`
  - **结构**：必须包裹在 `doc` 字段内

  ```js
  POST /products/_update/1001
  {
    "doc": {
      "price": 6499.00,
      "is_active": false
    }
  }
  ```

  - **优势**：只会修改 `price` 和 `is_active`，其他字段保持不变



#### 4. 删除文档 (Delete)

- **语法**：

  ```js
  DELETE /products/_doc/1001
  ```

- **逻辑**：

  - ES 的删除不是物理删除，而是 **逻辑删除**

    它会在 `.del` 文件中标记该文档为“deleted”。在后续的 Segment Merge（段合并）阶段，物理空间才会被真正回收



#### 5. 批量操作 (Bulk) —— 生产环境必备

如果你要一次性插入 1000 条数据，千万不要循环调用上面的 `PUT` 接口，网络开销会死人

- **语法**：NDJSON 格式（一行指令，一行数据）

  ```js
  POST /_bulk
  { "index": { "_index": "products", "_id": "1002" } }
  { "id": 1002, "title": "小米 14", "price": 3999 }
  { "index": { "_index": "products", "_id": "1003" } }
  { "id": 1003, "title": "iPhone 15", "price": 5999 }
  ```

- **最佳实践**：Java 客户端通常会自动封装 Bulk 操作。每次 Bulk 的大小建议在 5MB - 15MB 之间



### 3.3 生产环境的大坑：并发修改与乐观锁

在实际的 Java 高并发场景下（如秒杀、库存扣减），直接使用简单的 `UPDATE` 是非常危险的

#### 1. 场景重现：丢失更新 (Lost Update)

假设商品 ID `1001` 当前库存 `stock = 10`。 有两个运营人员（或两个线程）几乎同时操作：

1. **线程 A** 读取数据：`stock = 10`
2. **线程 B** 读取数据：`stock = 10`
3. **线程 A** 将库存 -1，写入 `9`
4. **线程 B** 将库存 -5，写入 `5`
5. **结果**：线程 B 的写入覆盖了线程 A 的操作，实际库存变成了 `5`，而正确的逻辑应该是 `10 - 1 - 5 = 4`

**MySQL 解决方案**：

- 通常利用行锁（`SELECT ... FOR UPDATE`）或 Update 语句的原子性（`UPDATE table SET stock = stock - 1`）
- **ES 痛点**：ES 是分布式的，且没有 ACID 事务，无法加锁。它采用的是 **乐观锁 (Optimistic Locking)** 机制



#### 2. 核心机制：_seq_no 与 _primary_term

当你执行 `GET /products/_doc/1001` 时，注意观察返回的元数据：

```js
{
  "_index": "products",
  "_id": "1001",
  "_version": 1,
  "_seq_no": 0,       // 关键字段：序列号
  "_primary_term": 1, // 关键字段：主分片任期号
  "_source": { ... }
}
```

- **`_seq_no` (Sequence Number)**：
  - 文档每被修改（增删改）一次，该数字就会**自增 1**
  - 类似于 Java `AtomicInteger` 或数据库的 Version 字段
- **`_primary_term`**：
  - 代表当前主分片（Primary Shard）是第几代
  - 如果集群发生故障转移，主分片切换了，这个数字会变
  - 它的作用是防止“脑裂”或旧主分片的数据覆盖新主分片



#### 3. 方案: CAS (Compare And Swap) 操作

ES 允许我们在更新时传入我们**“期望”**的 `seq_no` 和 `primary_term`

- 如果 ES 发现当前的序号和传进来的不一样，说明数据已经被别人改过了，它会拒绝写入并抛出异常



**正确的并发更新流程 (API 演示)**

**Step 1: 先查询，获取当前状态**

```js
GET /products/_doc/1001
```

*假设返回：`"_seq_no": 5, "_primary_term": 1, "stock": 10`*



**Step 2: 带条件的更新 (CAS)** 线程 A 想把库存改为 9，它必须告诉 ES：“我基于 seq_no=5 进行修改，如果变成 6 了就别让我改”

```js
POST /products/_update/1001?if_seq_no=5&if_primary_term=1
{
  "doc": {
    "stock": 9
  }
}
```

- **如果成功**：返回 200 OK，`_seq_no` 变为 6
- **如果失败**（被线程 B 抢先改成了 6）：
  - ES 返回 **409 Conflict** 状态码
  - 报错信息：`version conflict, required seqNo [5], primary term [1]. but no document was found, or current seqNo [6]...`



#### 4. Java 端的处理逻辑

作为 Java 程序员，你不能只知道抛了异常，必须在代码层面处理这个异常

```js
// 伪代码示例
public void updateStock(String id, int count) {
    int maxRetries = 3; // 最大重试次数
    for (int i = 0; i < maxRetries; i++) {
        try {
            // 1. 查询当前 seq_no
            Product product = esClient.get(id);
            
            // 2. 尝试更新 (带上 seq_no)
            esClient.update(id, product.getSeqNo(), product.getPrimaryTerm(), count);
            
            // 3. 成功则退出循环
            return;
        } catch (ElasticsearchStatusException e) {
            // 4. 捕获 409 Conflict
            if (e.status() == RestStatus.CONFLICT) {
                // log.info("发生并发冲突，准备重试...");
                continue; // 重新进入循环，重新查询最新 seq_no
            }
            throw e; // 其他异常直接抛出
        }
    }
    throw new RuntimeException("系统繁忙，重试失败");
}
```



#### 5. 总结

1. **ES 没有悲观锁**（不能像 MySQL 那样锁表）
2. 必须使用 **`if_seq_no`** 和 **`if_primary_term`** 来实现乐观锁控制
3. Java 代码中必须捕获 **409 Conflict** 异常并结合**重试机制 (Retry Loop)** 来保证数据一致性





## 04. DSL 高级查询语法 (1/N)

### 4.1 DSL 世界观：查询上下文 vs 过滤上下文

在学习具体的 `match` 或 `term` 查询之前，你必须先理解 ES 执行搜索的两种不同“环境”：**Query Context** 和 **Filter Context**

- 很多新手写出的慢查询，都是因为混淆了这两者，导致 ES 做了大量无用的计算



#### 1. Query Context (查询上下文) —— "匹配度是多少？"

- **核心问题**：这个文档与查询语句的**匹配程度 (How well does it match)** 是多少？
- **行为**：
  1. 查找匹配的文档
  2. **计算相关度得分 (`_score`)**：根据算法（BM25）计算文档与搜索词的关联度
  3. 结果默认按 `_score` 降序排列（得分高的排前面）
- **场景**：全文检索。例如：“搜索包含‘华为’的商品”，包含次数越多的应该排越前
- **关键字**：`query` 参数中直接使用的语句（如 `match`）

**示例 (Query)**：

```js
GET /products/_search
{
  "query": {
    "match": {
      "title": "华为手机"
    }
  }
}
```

*ES 会思考：文档 A 包含“华为”吗？包含。包含“手机”吗？包含。它俩挨得近吗？近。好，给它打 8.5 分！*



#### 2. Filter Context (过滤上下文) —— "是否匹配？"

- **核心问题**：这个文档**匹配吗 (Does it match)**？（Yes or No）
- **行为**：
  1. 查找匹配的文档
  2. **不计算得分**：`_score` 默认为 0 或 1
  3. **自动缓存 (Caching)**：ES 会自动缓存频繁使用的 Filter 结果（BitSet），下次再查瞬间返回
- **场景**：结构化筛选。例如：“价格大于 5000”、“状态为上架”、“创建时间在 2023 年”
- **关键字**：`bool` 查询下的 `filter` 参数

**示例 (Filter)**：

```js
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "is_active": true } },
        { "range": { "price": { "gte": 5000 } } }
      ]
    }
  }
}
```

*ES 会思考：文档 A 的 `is_active` 是 true 吗？是。价格大于 5000 吗？是。好，保留它（不关心它具体是多少钱，也不打分）*



#### 3. 性能对比与最佳实践

| 特性         | Query Context     | Filter Context       |
| ------------ | ----------------- | -------------------- |
| **计算得分** | ✅ 是 (消耗 CPU)   | ❌ 否                 |
| **结果缓存** | ❌ 否              | ✅ 是 (BitSet 缓存)   |
| **执行速度** | 较慢              | **极快**             |
| **适用场景** | 全文搜索 (Search) | 精确筛选 (Filtering) |

**🚀 黄金法则**： **只要不需要算分（Relevance Score），统统使用 Filter！** 

- 如果你只是想找“用户 ID = 1001”的订单，请务必用 Filter，不要用 Query，这能显著提升集群性能



**组合使用示例 (Query + Filter)**： 通常我们会组合使用：先用 Filter 过滤掉不相关的（高性能），再用 Query 对剩下的进行算分（高精度）

```js
GET /products/_search
{
  "query": {
    "bool": {
      "filter": { "term": { "category": "手机" } },  // 先过滤：只要手机
      "must": { "match": { "title": "华为" } }      // 再算分：标题含“华为”的排前面
    }
  }
}
```



### 4.2 全文检索详解 (Full Text Queries)

全文检索查询（Full Text Queries）是搜索引擎最核心的功能。它的特点是**“先分词，再搜索”**



#### 1. 核心查询：match (匹配查询)

这是最常用的查询，没有之一。它是 **分词感知** 的

- **原理**：
  1. ES 会先用该字段配置的分词器（如 `ik_smart`）分析你的搜索词
  2. 例如搜 "华为手机"，会被拆成 "华为" 和 "手机"
  3. ES 会去倒排索引里找包含 "华为" **OR** "手机" 的文档
  4. **注意**：默认逻辑是 `OR`（只要命中其中一个词就算匹配）
- **API 示例**：

```js
GET /products/_search
{
  "query": {
    "match": {
      "title": "华为手机"
    }
  }
}
```

- **进阶技巧：控制精度 (operator)** 如果你希望必须**同时**包含 "华为" **AND** "手机" 才能搜出来：

```js
GET /products/_search
{
  "query": {
    "match": {
      "title": {
        "query": "华为手机",
        "operator": "and" // 默认为 "or"
      }
    }
  }
}
```



#### 2. 多字段查询：multi_match

- **场景**：用户在搜索框输入了一个词，你希望同时去“标题(title)”、“描述(description)”、“品牌(brand)”里找
- **API 示例**：

```js
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "华为 5G",
      "fields": ["title", "description", "brand"] // 在这三个字段里搜
    }
  }
}
```

- **权重控制 (Boosting)**： 

  - 通常我们认为命中“标题”比命中“描述”更重要。

    可以使用 `^` 符号提权： `"fields": ["title^10", "description"]` —— 命中 title 的得分权重是 description 的 10 倍



#### 3. 短语匹配：match_phrase (精确短语)

- **痛点**：`match` 查询不关心词的顺序。搜 "手机华为"，也能搜出 "华为手机"
- **场景**：你需要内容必须 **完全包含** 搜索词，且 **顺序一致**，且 **中间没有其他干扰词**
- **原理**：它不仅看倒排索引，还看词在文档中的 **位置 (Position)** 信息
- **API 示例**：

```js
GET /products/_search
{
  "query": {
    "match_phrase": {
      "title": "华为手机"
    }
  }
}
```

- **结果**：
  - ✅ "华为手机 Pro" (匹配)
  - ❌ "华为 5G 手机" (不匹配，因为中间夹了 5G)
  - ❌ "手机华为" (不匹配，顺序反了)
- **容错控制：slop** 如果你允许中间夹杂几个词（比如允许 "华为 5G 手机" 也能被搜到）：

```js
GET /products/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "华为手机",
        "slop": 2 // 允许中间最多间隔 2 个词
      }
    }
  }
}
```

#### 4. 总结与 API 速查表

| 查询类型     | API 关键字     | 作用                               | 核心参数            |
| ------------ | -------------- | ---------------------------------- | ------------------- |
| **标准匹配** | `match`        | 先分词，再搜索。默认 OR 逻辑。     | `query`, `operator` |
| **多字段**   | `multi_match`  | 一个词搜多个字段。                 | `fields`, `type`    |
| **短语匹配** | `match_phrase` | 精确匹配短语，顺序和位置必须一致。 | `slop` (容错步数)   |
| **所有匹配** | `match_all`    | 匹配所有文档（常用于测试或导出）。 | 无                  |

**最佳实践**： 在电商搜索中，通常会组合使用：

1. 用 `match` 保证召回率（能搜出来的都搜出来）
2. 用 `match_phrase` 提高精确度（匹配度高的排前面）
3. 用 `multi_match` 覆盖标题和描述



### 4.3 精确查询详解 (Term Level Queries)

与 `match` 查询不同，Term Level Queries **不进行分词**。它直接拿你输入的搜索词，去倒排索引里**精准匹配**。

> **⚠️ 核心原则**：此类查询通常用于结构化数据（如数字、日期、枚举、Keyword），且**强烈建议在 `filter` 上下文中执行**（利用缓存，不计算得分）。

#### 1. 单值精确匹配：term

- **场景**：类似于 MySQL 的 `WHERE category = '手机'`
- **API 示例**：

```js
GET /products/_search
{
  "query": {
    "term": {
      "category": "手机"
    }
  }
}
```

- **🚨 巨坑预警：对 Text 字段使用 Term 查询** 很多新手会犯这个错误：

  ```js
  // 错误示范：title 是 text 类型（分词了）
  "term": { "title": "华为手机" }
  ```

  - **结果**：搜不到！
  - **原因**：
    1. 索引库里，`title` 被拆成了 "华为"、"手机"（没有 "华为手机" 这个词）
    2. `term` 查询是不分词的，它拿着 "华为手机" 去找，当然找不到
  - **结论**：**永远不要用 `term` 查询 `text` 类型的字段**。如果非要精准匹配 `text` 字段，请使用它的 `.keyword` 子字段（前提是 Mapping 里配了）



#### 2. 多值匹配：terms

- **场景**：类似于 MySQL 的 `WHERE category IN ('手机', '电脑')`
- **API 示例**：

```js
GET /products/_search
{
  "query": {
    "terms": {
      "category": ["手机", "电脑", "平板"]
    }
  }
}
```



#### 3. 范围查询：range

- **场景**：价格区间、时间范围
- **参数**：
  - `gte` (Greater Than or Equal): `>=`
  - `lte` (Less Than or Equal): `<=`
  - `gt`: `>`
  - `lt`: `<`
- **API 示例 (价格区间)**：

```js
GET /products/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 1000,
        "lte": 5000
      }
    }
  }
}
```



- **进阶：日期计算 (Date Math)** ES 对日期范围支持极好，支持数学运算。
  - `now`: 当前时间。
  - `now-1d`: 过去 24 小时。
  - `now/d`: 当前日期的 00:00:00。

```js
GET /products/_search
{
  "query": {
    "range": {
      "created_at": {
        "gte": "now-7d/d", // 最近 7 天（取整到天）
        "lte": "now"
      }
    }
  }
}
```



#### 4. 存在性检测：exists

- **场景**：类似于 MySQL 的 `WHERE field IS NOT NULL`
- **作用**：查找包含指定字段（且值不为 null）的文档

```js
GET /products/_search
{
  "query": {
    "exists": {
      "field": "description" // 查找有描述的商品
    }
  }
}
```

*注：ES 中没有 `missing` 查询，如果想查“空值”，请使用 `must_not` + `exists` 组合*



#### 5. ID 查询：ids

- **场景**：类似于 MySQL 的 `WHERE id IN (1, 2, 3)`
- **注意**：这里查的是 `_id` 元数据字段

```js
GET /products/_search
{
  "query": {
    "ids": {
      "values": ["1001", "1002"]
    }
  }
}
```

**本节总结**：

1. 查 **文本内容**（文章、标题），用 `match`
2. 查 **精确值**（价格、ID、状态），用 `term`/`range`
3. 切记：**Term 查询不分词**，千万别对 `text` 字段用 `term`



### 4.4 复合查询：bool 查询详解

`bool` 查询是 ES 中用来组合多个查询子句的容器。它类似于 SQL 中的 `AND` / `OR` / `NOT` 逻辑，但功能更强大，因为它还包含“算分”与“过滤”的控制。

#### 1. bool 查询的四大护法

一个 `bool` 查询可以包含以下四种子句（可以混合使用）：

| 子句关键字   | 逻辑含义 | 上下文 (Context) | 是否算分 | 缓存 | 作用详解                                                     |
| ------------ | -------- | ---------------- | -------- | ---- | ------------------------------------------------------------ |
| **must**     | **AND**  | Query            | ✅ 是     | ❌ 否 | 必须匹配。贡献算分                                           |
| **must_not** | **NOT**  | Filter           | ❌ 否     | ✅ 是 | 必须**不**匹配。**不计算分数**（直接排除）                   |
| **filter**   | **AND**  | Filter           | ❌ 否     | ✅ 是 | 必须匹配。**不计算分数**。用于硬性筛选                       |
| **should**   | **OR**   | Query            | ✅ 是     | ❌ 否 | **可选**匹配。如果匹配了，分数变高；不匹配也不会被剔除（除非没有 `must`） |



#### 2. 实战演练：构建复杂查询

**业务需求**： 搜索“华为手机”，要求：

1. 价格必须大于 2000
2. 必须有库存（`is_active: true`）
3. **不能**是二手货（`tags` 不包含 "second_hand"）
4. （加分项）如果是“5G”手机，排在前面



**DSL 编写思路**：

- “华为手机” -> 全文匹配 -> `must` (match)
- 价格 > 2000 -> 硬性指标 -> `filter` (range)
- 有库存 -> 硬性指标 -> `filter` (term)
- 不是二手 -> 排除 -> `must_not` (term)
- 5G -> 加分项 -> `should` (term)



**最终代码**：

```js
GET /products/_search
{
  "query": {
    "bool": {
      // 1. 必须包含 (Query Context)
      "must": [
        { "match": { "title": "华为手机" } }
      ],
      // 2. 必须过滤 (Filter Context - 高性能)
      "filter": [
        { "range": { "price": { "gt": 2000 } } },
        { "term":  { "is_active": true } }
      ],
      // 3. 必须排除 (Filter Context)
      "must_not": [
        { "term": { "tags": "second_hand" } }
      ],
      // 4. 加分项 (Query Context)
      "should": [
        { "term": { "title": "5G" } }
      ]
    }
  }
}
```



#### 3. 深入理解 should (OR 的两种面孔)

`should` 的行为比较特殊，它取决于 `bool` 查询中是否包含 `must` 或 `filter`

- **场景 A：只有 should**

  - **逻辑**：至少满足其中一个（OR）
  - **例子**：`should: [ {A}, {B} ]` -> 必须满足 A 或 B，否则无结果

  

- **场景 B：should 混搭 must**

  - **逻辑**：`should` 变成了**“加分项”**
  - **例子**：`must: [A], should: [B]`
  - **结果**：只要满足 A 就一定返回。如果同时满足了 B，得分更高，排在更前面；如果不满足 B，也照样返回，只是分低一点

- **控制精度：minimum_should_match** 如果你希望在混搭时 `should` 也是必须满足一部分，可以使用此参数：

  ```js
  "bool": {
    "must": [ ... ],
    "should": [ {A}, {B}, {C} ],
    "minimum_should_match": 1 // A/B/C 中至少要满足 1 个
  }
  ```



#### 4. 嵌套查询 (Nested Logic)

`bool` 查询是可以无限嵌套的。比如实现 `(A OR B) AND C`：

```js
{
  "bool": {
    "must": [
      { "term": { "field_C": "value" } }, // AND C
      {
        "bool": { // 嵌套一个 bool 实现 (A OR B)
          "should": [
            { "term": { "field_A": "value" } },
            { "term": { "field_B": "value" } }
          ]
        }
      }
    ]
  }
}
```



#### 5. 性能优化建议

1. **Filter 优先**：把不需要算分的条件（价格、状态、ID、时间）全部扔进 `filter` 和 `must_not`。只有真正需要按内容相关度排序的（关键词搜索），才放进 `must` 和 `should`
2. **结构清晰**：虽然 JSON 可以乱序，但建议按照 `must` -> `filter` -> `should` -> `must_not` 的顺序编写，便于人类阅读



## 05. Aggregations 聚合分析 (1/N)

### 5.1 聚合分析基础与 Metric 聚合

Elasticsearch 的聚合（Aggregations，简称 `aggs`）功能极其强大，它能在毫秒级时间内对亿级数据进行实时统计。这使得 ES 不仅是一个搜索引擎，更是一个**实时 OLAP（联机分析处理）引擎**

#### 1. 核心概念：Bucket vs Metric

聚合主要分为两大类，理解它们的区别是构建复杂报表的基础：

| 概念     | 英文       | 对应 SQL 概念         | 作用                                         | 例子                             |
| -------- | ---------- | --------------------- | -------------------------------------------- | -------------------------------- |
| **分桶** | **Bucket** | `GROUP BY`            | **分组**。把文档按照某种条件分成不同的“桶”。 | 按“品牌”分组、按“价格区间”分组。 |
| **指标** | **Metric** | `COUNT`, `AVG`, `SUM` | **计算**。在分好的桶里进行数值计算。         | 计算平均价格、最高销量。         |



#### 2. 聚合的 DSL 语法结构

聚合查询是和 `query` 平级的。

**关键语法结构**：

```js
GET /products/_search
{
  "size": 0,           // 🚀 重点：我们只看统计结果，不需要看具体的文档内容，设为 0 可提升性能
  "query": { ... },    // 可选：先限定统计范围（例如：只统计“华为手机”）
  "aggs": {            // 聚合块
    "my_agg_name": {   // 自定义名字（随便起，比如 avg_price）
      "agg_type": {    // 聚合类型（比如 avg, terms）
        "field": "price" // 针对哪个字段
      }
    }
  }
}
```



#### 3. Metric 聚合详解 (数值计算)

Metric 聚合通常用于计算数字类型的字段

##### (1) 单值分析：min, max, avg, sum

计算商品的平均价格、最高价、最低价

```js
GET /products/_search
{
  "size": 0,
  "aggs": {
    "max_price": { "max": { "field": "price" } },
    "min_price": { "min": { "field": "price" } },
    "avg_price": { "avg": { "field": "price" } }
  }
}
```

**返回结果**（关注 `aggregations` 字段）：

```js
{
  ...
  "aggregations": {
    "max_price": { "value": 6999.0 },
    "min_price": { "value": 2999.0 },
    "avg_price": { "value": 4500.0 }
  }
}
```



##### (2) 统计全家桶：stats

如果你想一次性获取 min, max, avg, sum, count，不用写 5 个聚合，直接用 `stats`

```js
GET /products/_search
{
  "size": 0,
  "aggs": {
    "price_stats": {
      "stats": {
        "field": "price"
      }
    }
  }
}
```



##### (3) 唯一值统计：cardinality (去重计数)

- **对应 SQL**：`COUNT(DISTINCT category)`
- **场景**：统计有多少个不同的品牌、有多少个独立访客（UV）
- **注意**：这是一个**近似值**（基于 HyperLogLog++ 算法），在数据量极大时（亿级），它会有 5% 左右的误差，但速度极快且内存占用极低

```js
GET /products/_search
{
  "size": 0,
  "aggs": {
    "distinct_brands": {
      "cardinality": {
        "field": "brand" // 字段通常要是 keyword 类型
      }
    }
  }
}
```



#### 4. 结合 Query 进行范围统计

聚合分析是可以和 `query` 配合使用的。ES 会**先执行 Query 过滤数据，再对剩下的数据进行 Aggs 统计**

**场景**：只想统计“手机”这个分类下的平均价格

```js
GET /products/_search
{
  "size": 0,
  "query": {
    "term": { "category": "手机" } // 先圈定范围
  },
  "aggs": {
    "avg_price_of_phones": {
      "avg": { "field": "price" }  // 只算手机的平均价
    }
  }
}
```

#### 💡 避坑指南

1. **size: 0**：做聚合分析时，务必将 `size` 设为 0。否则 ES 既要帮你算统计数据，又要从磁盘读取文档详情（`_source`），网络和 IO 开销会增加，且没有任何意义
2. **字段类型**：Metric 聚合通常针对 **Numeric (数值)** 类型。如果对 `keyword` 类型求 `avg` 会直接报错



### 5.2 Bucket 聚合 (分桶) 与 多维数据分析

Bucket 聚合的作用是**将文档“归类”到不同的桶中**。这就好比把一堆散乱的衣服，按照“颜色”或者“尺码”分别扔进不同的篮子里

#### 1. Terms Aggregation (按词条分桶)

这是最常用的分桶方式，对应 SQL 的 `GROUP BY field`

- **场景**：统计每个分类（category）下有多少个商品。
- **语法**：

```js
GET /products/_search
{
  "size": 0,
  "aggs": {
    "group_by_category": {
      "terms": {
        "field": "category", // 必须是 keyword 类型
        "size": 10           // 只返回数量最多的前 10 个分类
      }
    }
  }
}
```

- **返回结构**： ES 会返回一个 `buckets` 数组，每个对象包含 `key` (分类名) 和 `doc_count` (文档数)



#### 2. Histogram & Date Histogram (直方图/时间分桶)

如果字段是连续的数值或时间，我们不能用 `terms`（因为值太多了），而是要用区间来切分

- **Histogram (数值间隔)**：

  - **场景**：价格分布图。每 1000 元一个区间（0-1000, 1000-2000...）

  - **语法**：

    ```js
    "aggs": {
      "price_histogram": {
        "histogram": {
          "field": "price",
          "interval": 1000
        }
      }
    }
    ```

  

- **Date Histogram (时间间隔)**：

  - **场景**：统计每月的销量趋势。

  - **语法**：

    ```js
    "aggs": {
      "sales_over_time": {
        "date_histogram": {
          "field": "created_at",
          "calendar_interval": "month", // day, hour, year
          "format": "yyyy-MM"
        }
      }
    }
    ```



#### 3. Sub-Aggregations (嵌套聚合) —— 核心杀手锏

ES 的强大之处在于：**聚合是可以无限嵌套的**。 你可以在一个“桶”里，再进行“分桶”或者“指标计算”。

**实战场景**： “我想看每个分类（Category）下，价格最高的商品是多少钱（Max Price）？”

- **逻辑**：先按 `category` 分桶 -> 在每个桶内计算 `max(price)`。

```js
GET /products/_search
{
  "size": 0,
  "aggs": {
    // 第一层：按分类分桶
    "group_by_category": {
      "terms": { "field": "category" },
      "aggs": {
        // 第二层：在当前分类桶内，计算最高价
        "max_price_in_category": {
          "max": { "field": "price" }
        },
        // 甚至可以再加一个：计算平均价
        "avg_price_in_category": {
          "avg": { "field": "price" }
        }
      }
    }
  }
}
```

**返回结果示意**：

```js
"buckets": [
  {
    "key": "手机",
    "doc_count": 100,
    "max_price_in_category": { "value": 6999 },
    "avg_price_in_category": { "value": 3500 }
  },
  {
    "key": "电脑",
    "doc_count": 50,
    "max_price_in_category": { "value": 12999 },
    "avg_price_in_category": { "value": 8000 }
  }
]
```



#### 4. 排序规则 (Order)

默认情况下，`terms` 聚合是按文档数量 (`doc_count`) 降序排列的。但有时我们需要按子聚合的结果排序

- **需求**：按“平均价格”降序排列分类（即：平均价最贵的分类排前面，而不是数量最多的排前面）

```js
"aggs": {
  "group_by_category": {
    "terms": {
      "field": "category",
      "order": {
        "avg_price_calc": "desc" // 指向子聚合的名字
      }
    },
    "aggs": {
      "avg_price_calc": { "avg": { "field": "price" } }
    }
  }
}
```



#### 避坑指南

1. **Text 字段无法分组**：
   - 如果你尝试对 `title` (text 类型) 进行 `terms` 聚合，ES 会报错 `Fielddata is disabled...`
   - **解决**：必须使用 `title.keyword`，或者在 Mapping 中开启 `fielddata: true`（极度消耗内存，**生产环境严禁开启**）。永远优先使用 Keyword 字段进行聚合
2. **数据不准确 (Approximate Counts)**：
   - `terms` 聚合在分布式环境下是**有损**的。如果你的数据分散在多个分片上，且 `size` 设置得较小（例如 top 5），那么返回的 top 5 可能不是全局真正的 top 5
   - **解决**：在请求中增加 `shard_size` 参数，或者提高 `size` 的值
3. **深度嵌套风险**：
   - 聚合嵌套层级不要太深（建议不超过 3 层）。每一层嵌套都会导致计算量呈指数级增长，容易引发 OOM（内存溢出）



## 06. Java 客户端选型与环境集成 (1/N)

### 6.1 客户端的“乱世演义”：我们该选谁？

在开始写代码前，必须先选对“武器”。ES 的 Java 客户端经历了三次大的变革，选错版本会导致代码完全跑不起来

#### 1. 历史版本回顾 (避坑必读)

| 客户端名称                    | 状态                    | 说明                                                         |
| ----------------------------- | ----------------------- | ------------------------------------------------------------ |
| **TransportClient**           | **已淘汰 (Dead)**       | 使用 TCP 9300 端口。依赖严重耦合 ES 内核版本，ES 7.0 废弃，ES 8.0 彻底移除。**千万别学了** |
| **RestHighLevelClient**       | **已废弃 (Deprecated)** | 曾经的王者。基于 HTTP 9200 端口。但在 ES 7.15.0 宣布废弃，ES 8.x 官方不再建议新项目使用。**网上教程最多，但已过时，黑马程序员的课就讲的这个** |
| **Java API Client**           | **新标准**              | ES 8.0 推出的全新官方客户端。代码风格更现代（Builder 模式，Lambda 表达式），与 HTTP 协议解耦。**我们要学的就是这个** |
| **Spring Data Elasticsearch** | **最强辅助**            | Spring 对上述客户端的封装。**Spring Boot 3.x 底层默认使用的就是 Java API Client** |



#### 2. 我们的技术选型

- **框架**：Spring Boot 3.2+ (JDK 17+)
- **ES 版本**：Elasticsearch 8.12.2
- **集成方式**：**Spring Data Elasticsearch** (既能用 Repository 的便利，又能用 Native Client 的灵活)



### 6.2 Spring Boot 3 项目搭建

#### 1. 引入依赖 (pom.xml)

创建一个新的 Spring Boot 项目，引入 `spring-boot-starter-data-elasticsearch`。

> **⚠️ 关键点**：Spring Boot 版本必须与 ES 版本兼容。Spring Boot 3.2.x 默认适配 ES 8.10+。

```xml
<dependencies>
    <!-- Web 模块 (方便写 Controller 测试) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Data Elasticsearch (核心依赖) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    </dependency>
    
    <!-- Lombok (减少样板代码) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```



#### 2. 配置连接信息 (application.yml)

在 `src/main/resources/application.yml` 中配置 ES 地址。 由于我们在 Docker 搭建时禁用了 SSL 和账号密码，配置非常简单：

```yaml
spring:
  elasticsearch:
    uris: http://localhost:9200
    # 如果开启了账号密码 (xpack)
    # username: elastic
    # password: your_password
    # socket-timeout: 30s
    # connection-timeout: 5s
```



#### 3. 开启详细日志 (Optional)

在开发阶段，为了看清 Spring 到底帮我们要生成了什么样的 DSL，建议开启 Trace 日志：

```yaml
logging:
  level:
    org.springframework.data.elasticsearch.client.WIRE: trace
```



### 6.3 验证环境是否连通

我们写一个简单的单元测试来验证 Spring 是否成功连上了 ES

```java
package com.example.demo;

import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.transport.Version;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class DemoApplicationTests {

    // 注入新版原生客户端
    @Autowired
    private ElasticsearchClient esClient;

    @Test
    void testConnection() throws Exception {
        // 获取 ES 服务端版本信息
        // 这里的 DSL 相当于: GET /
        var response = esClient.info();
        
        System.out.println("ES 连接成功！");
        System.out.println("ES 版本: " + response.version().number());
        System.out.println("Lucene 版本: " + response.version().luceneVersion());
    }
}
```

**运行结果预期**： 控制台打印出 `ES 版本: 8.12.2`。如果你看到了版本号，说明 Java 代码与 Docker 里的 ES 握手成功！



### 💡 避坑指南：Spring Boot 与 ES 的版本地狱

- **现象**：启动报错 `ClassNotFoundException` 或 `NoSuchMethodError`

- **原因**：Spring Boot 内置的 ES 依赖版本与你服务器运行的 ES 版本跨度太大（比如 Boot 用了 7.x 的客户端去连 8.x 的服务端）

- **解决**：强制指定版本。在 `pom.xml` 中添加属性覆盖：

  ```xml
  <properties>
      <elasticsearch.version>8.12.2</elasticsearch.version>
  </properties>
  ```





## 07. Spring Data Elasticsearch

### 7.1 实体映射：把 Java 对象变成 ES 文档

在 Spring Data 中，我们需要定义一个 POJO 类来描述索引的结构（Schema）。ES 会读取这些注解，在启动时（默认情况下）自动创建索引和 Mapping

#### 1. 核心注解清单 (API)

这些注解都位于 `org.springframework.data.elasticsearch.annotations` 包下

| 注解              | 作用                                    | 关键属性                                                     |
| ----------------- | --------------------------------------- | ------------------------------------------------------------ |
| **`@Document`**   | **类级别**。标记这是一个 ES 实体类。    | `indexName`: 对应索引名。`createIndex`: 是否自动创建索引（生产环境建议 false） |
| **`@Id`**         | **字段级别**。标记主键。                | 无。对应 ES 中的 `_id` 元数据                                |
| **`@Field`**      | **字段级别**。定义字段的 Mapping 属性。 | `type`: 字段类型 (Text, Keyword, Date...)。`analyzer`: 指定分词器。`name`: 指定 ES 中的字段名（如果和 Java 属性名不同） |
| **`@DateFormat`** | **配合 @Field**。定义日期格式。         | 解决 Java `LocalDateTime` 与 ES 时间戳的转换问题             |



#### 2. 代码实战：定义 Product 实体

请在项目中创建一个 `model` 或 `entity` 包，并新建 `Product.java`

```java
package com.example.demo.entity;

import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.DateFormat;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.time.LocalDateTime;

@Data
// 1. 标记为文档，指定索引名为 "products"
@Document(indexName = "products")
public class Product {

    // 2. 映射 _id。建议使用 String 类型，因为 ES 的 ID 本质是字符串。
    @Id
    private String id;

    // 3. 定义 Text 类型，并指定分词器
    // 写入时用 ik_max_word (细粒度)，搜索时用 ik_smart (粗粒度)
    @Field(type = FieldType.Text, analyzer = "ik_max_word", searchAnalyzer = "ik_smart")
    private String title;

    // 4. Keyword 类型：不分词，用于精确匹配、聚合、排序
    @Field(type = FieldType.Keyword)
    private String category;

    // 5. 数值类型
    @Field(type = FieldType.Double)
    private Double price;

    // 6. 日期类型 (坑点高发区 🚨)
    // 必须指定 format，否则 Java 的 LocalDateTime 序列化到 ES 时会报错
    @Field(type = FieldType.Date, format = DateFormat.date_hour_minute_second_millis)
    private LocalDateTime createdAt;

    // 7. 布尔类型
    @Field(type = FieldType.Boolean)
    private Boolean isActive;
}
```



#### 3. 避坑指南

1. **日期类型的坑**：
   - ES 底层默认存储的是 UTC 时间戳（Long）
   - Java 的 `LocalDateTime` 没有时区概念
   - **建议**：在 `@Field` 中显式指定 `format`，或者干脆在 Java 侧用 `Long` 类型存储时间戳，避免繁琐的时间格式转换问题
2. **自动创建索引 (createIndex)**：
   - `@Document(createIndex = true)` 是默认值。Spring 启动时会检查索引是否存在，不存在则创建
   - **生产环境建议**：设置为 `false`。索引的创建和 Mapping 的维护应该通过运维脚本或专门的迁移工具（Review 机制）来管理，而不是由应用程序启动时静默触发，防止代码修改意外覆盖了线上配置
3. **主键生成策略**：
   - 如果在保存时 `id` 字段为 `null`，ES 会自动生成一个随机 UUID
   - 如果 `id` 有值，则是“新增或覆盖（Upsert）”



### 7.2 Repository 接口开发：零 SQL 实现 CRUD

#### 1. 定义接口 (Interface Definition)

在 `repository` 包下创建 `ProductRepository` 接口

- **关键点**：继承 `ElasticsearchRepository<T, ID>`
  - `T`: 实体类类型 (`Product`)
  - `ID`: 主键类型 (`String`)

```java
package com.example.demo.repository;

import com.example.demo.entity.Product;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.annotations.Query;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import java.util.List;

public interface ProductRepository extends ElasticsearchRepository<Product, String> {

    // ---------------------------------------------------------
    // 第一层：继承父接口获得的方法 (无需声明，直接用)
    // ---------------------------------------------------------
    // save(entity), saveAll(entities) -> 增/改
    // findById(id), findAll()         -> 查
    // deleteById(id), delete(entity)  -> 删
    
    // ---------------------------------------------------------
    // 第二层：方法名衍生查询 (Derived Queries)
    // ---------------------------------------------------------
    // Spring 会解析方法名，自动生成 DSL。
    // 类似于: SELECT * FROM product WHERE category = ?
    List<Product> findByCategory(String category);

    // 类似于: SELECT * FROM product WHERE price BETWEEN ? AND ?
    List<Product> findByPriceBetween(Double min, Double max);

    // 类似于: SELECT * FROM product WHERE title LIKE ?
    // 注意：在 ES 中这通常对应 "match" 查询
    List<Product> findByTitleMatches(String title);

    // 带分页的查询
    // 调用方式: findByTitleMatches("华为", PageRequest.of(0, 10))
    Page<Product> findByTitleMatches(String title, Pageable pageable);

    // ---------------------------------------------------------
    // 第三层：@Query 自定义 DSL (终极武器)
    // ---------------------------------------------------------
    // 当方法名太长写不下，或者逻辑太复杂时，直接写 JSON DSL。
    // ?0 代表第一个参数，?1 代表第二个参数
    @Query("{\"match\": {\"title\": {\"query\": \"?0\"}}}")
    List<Product> findByNameCustom(String title);
}
```



#### 2. 单元测试实战 (Unit Test)

我们直接在测试类中注入这个接口，看看它是怎么工作的

```java
@SpringBootTest
class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @Test
    void testCrud() {
        // 1. 新增 (Create)
        Product p = new Product();
        p.setId("1001");
        p.setTitle("华为 Mate 60 Pro");
        p.setCategory("手机");
        p.setPrice(6999.0);
        p.setIsActive(true);
        
        productRepository.save(p); // 自动生成 index 请求
        System.out.println("保存成功");

        // 2. 查询 (Read)
        // Optional<T> 是 Java 8 标准
        productRepository.findById("1001").ifPresent(product -> {
            System.out.println("查到商品: " + product.getTitle());
        });

        // 3. 按价格区间查询 (Custom Method)
        List<Product> cheapPhones = productRepository.findByPriceBetween(1000.0, 5000.0);
        System.out.println("5000元以下的手机有: " + cheapPhones.size() + " 个");

        // 4. 删除 (Delete)
        productRepository.deleteById("1001");
    }
}
```



#### 3. 避坑指南：Repository 的局限性

虽然 Repository 很爽，但它有明显的**天花板**：

1. **复杂聚合不支持**：如果你要做类似“按月统计每个品牌的平均销量”这种报表分析，Repository 的方法名无法表达（或者写出来极其变态）
2. **动态查询困难**：如果你的搜索条件是动态的（比如用户有时候选价格，有时候选品牌，有时候两个都选），用 Repository 需要写大量的 `if-else` 来调用不同的方法，非常丑陋
3. **高亮显示 (Highlight)**：虽然 Spring Data 只有有限的支持，但处理起来比较麻烦，不如原生 Client 灵活

**结论**：

- 简单的 CRUD、根据 ID 查、根据单个字段查 -> **用 Repository**
- 复杂搜索、动态过滤、聚合分析、高亮 -> **需要用 ElasticsearchClient（下一节的内容）**



## 08. Elasticsearch Java API Client 实战

### 8.1 为什么我们需要原生 Client？

Spring Data Repository 虽然方便，但在面对以下场景时会显得力不从心：

1. **复杂嵌套查询**：比如 4 层嵌套的 Bool 查询
2. **聚合分析 (Aggregations)**：Repository 方法名无法表达复杂的聚合逻辑
3. **高亮显示 (Highlighting)**：虽然 Repository 支持，但灵活性较差
4. **精确控制**：需要手动控制 `track_total_hits`、`min_score` 或 `search_after` 参数

Elasticsearch 8.x 推出的 **Java API Client** 采用了全新的 **Builder + Lambda** 设计模式，代码结构与 JSON DSL 高度对应，是解决复杂问题的终极武器



### 8.2 基础查询实战

#### 1. 注入客户端

在 Service 或 Test 类中注入 `ElasticsearchClient`

```java
@Autowired
private ElasticsearchClient esClient;
```



#### 2. 构建查询 (Fluent API)

我们要实现这个 DSL：

```js
GET /products/_search
{
  "query": {
    "match": { "title": "华为" }
  }
}
```



**Java 代码实现**：

```js
@Test
void testSimpleSearch() throws IOException {
    // 1. 发起搜索请求
    SearchResponse<Product> response = esClient.search(s -> s
            .index("products") // 指定索引
            .query(q -> q      // 构建查询
                .match(m -> m  // match 查询
                    .field("title")
                    .query("华为")
                )
            ),
        Product.class // 结果映射的目标类
    );

    // 2. 解析结果
    long total = response.hits().total().value();
    System.out.println("共搜索到 " + total + " 条数据");

    for (Hit<Product> hit : response.hits().hits()) {
        Product p = hit.source();
        System.out.println("商品名: " + p.getTitle() + ", 分数: " + hit.score());
    }
}
```



### 8.3 复杂 Bool 查询实战 (核心)

还记得我们在 4.4 节写的那个复杂 DSL 吗？

- **Must**: 标题包含 "华为"
- **Filter**: 价格 > 2000
- **Filter**: 有库存
- **Must Not**: 标签包含 "second_hand"
- **Should**: 标题包含 "5G"

**Java 代码实现**：

```js
@Test
void testBoolQuery() throws IOException {
    SearchResponse<Product> response = esClient.search(s -> s
        .index("products")
        .query(q -> q
            .bool(b -> b
                // 1. Must
                .must(m -> m
                    .match(t -> t
                        .field("title")
                        .query("华为")
                    )
                )
                // 2. Filter (多个条件用 add)
                .filter(f -> f
                    .range(r -> r
                        .field("price")
                        .gt(JsonData.of(2000)) // 注意: 数值需包装
                    )
                )
                .filter(f -> f
                    .term(t -> t
                        .field("isActive")
                        .value(true)
                    )
                )
                // 3. Must Not
                .mustNot(mn -> mn
                    .term(t -> t
                        .field("tags")
                        .value("second_hand")
                    )
                )
                // 4. Should
                .should(sh -> sh
                    .match(m -> m
                        .field("title")
                        .query("5G")
                    )
                )
            )
        ),
        Product.class
    );
    
    // ... 解析结果同上
}
```



### 8.4 聚合分析实战 (Aggregations)

**需求**：统计每个分类下的商品数量，并按数量降序排列。

```java
@Test
void testAggregation() throws IOException {
    String aggName = "group_by_category";

    SearchResponse<Void> response = esClient.search(s -> s
        .index("products")
        .size(0) // 只需要聚合结果，不需要文档列表
        .aggregations(aggName, a -> a
            .terms(t -> t
                .field("category")
                .size(10)
            )
        ),
        Void.class // 泛型填 Void 因为不解析 hits
    );

    // 解析聚合结果
    // 1. 获取聚合体 (注意类型强转)
    Aggregate aggregate = response.aggregations().get(aggName);
    
    // 2. 只有 Terms 类型的聚合才能转成 StringTermsAggregate
    StringTermsAggregate termsAgg = aggregate.sterms();

    // 3. 遍历桶
    for (StringTermsBucket bucket : termsAgg.buckets().array()) {
        String categoryName = bucket.key().stringValue();
        long docCount = bucket.docCount();
        System.out.println("分类: " + categoryName + ", 数量: " + docCount);
    }
}
```



### 8.5 避坑指南

1. **Lambda 层级过深**：
   - Java Client 的 Lambda 写法虽然还原了 JSON 结构，但层级一旦深了（超过 5 层），代码可读性会急剧下降
   - **建议**：将复杂的 Query 构建逻辑抽取成独立的方法（例如 `buildBoolQuery()`），保持主流程清晰
2. **JsonData 包装**：
   - 在构建 `range` 查询或某些参数时，数值和对象需要用 `JsonData.of(value)` 进行包装，直接传 `int` 有时会报错
3. **结果解析类型安全**：
   - 聚合结果解析时，必须非常清楚你的聚合类型是 Terms、Histogram 还是 Avg。如果类型强转错误（比如把 `sterms` 转成了 `lterms`），会抛出异常



## 09. 生产环境常见问题与数据同步 (1/N)

### 9.1 MySQL 与 ES 的数据同步方案

Elasticsearch 是一个“从数据源”，它的数据通常是从 MySQL（主数据源）流转过来的。根据业务对**实时性**和**一致性**要求的不同，主要有三种同步策略

#### 1. 方案一：同步双写

这是最直观、最简单的方案。在代码层面，更新数据库的同时，顺便更新 ES

- **代码逻辑**：

  ```java
  @Transactional
  public void updateProduct(Product product) {
      // 1. 写 MySQL
      productMapper.updateById(product);
  
      // 2. 写 ES
      // ⚠️ 风险点：如果 MySQL 成功，ES 失败（比如网络超时），数据就不一致了！
      esClient.index(product); 
  }
  ```

- **优点**：

  - 简单粗暴，实时性极高（基本是毫秒级）

- **缺点 (巨坑)**：

  - **硬耦合**：业务代码和 ES 搜索代码绑死在一起
  - **数据不一致**：无法保证 MySQL 和 ES 的分布式事务。如果 ES 写失败，虽然可以回滚 MySQL，但如果 MySQL 提交成功后 ES 挂了，数据就丢了
  - **性能损耗**：用户请求的响应时间 = MySQL 耗时 + ES 耗时



#### 2. 方案二：异步 MQ 通知 (Asynchronous MQ)

为了解决耦合和性能问题，我们可以引入消息队列（RabbitMQ / RocketMQ / Kafka）

- **流程**：
  1. 业务系统只写 MySQL
  2. 业务系统发送一条“商品变更消息”到 MQ（例如：`product_id: 1001, action: UPDATE`）
  3. 单独起一个“搜索服务”消费 MQ，拿到 ID 后去查 MySQL，然后写入 ES
- **优点**：
  - **解耦**：核心业务不再依赖 ES 的稳定性
  - **削峰填谷**：在流量洪峰时，MQ 可以缓冲写入压力，保护 ES
- **缺点**：
  - **延迟**：数据可能有秒级延迟
  - **代码侵入**：还是需要在业务代码里发 MQ 消息。如果忘了发，数据就不同步了



#### 3. 方案三：监听 Binlog (CDC - Change Data Capture) —— 👑 推荐方案

这是目前大厂最主流的 **无侵入** 同步方案。它把 ES 当作 MySQL 的一个“从库”

- **工具选型**：阿里巴巴 **Canal** (最成熟)、Debezium
- **流程**：
  1. MySQL 开启 Binlog（二进制日志）
  2. 部署 Canal Server，它伪装成 MySQL 的 Slave 节点，向 MySQL Master 发送 dump 协议
  3. MySQL 收到请求，推送 Binlog 给 Canal
  4. Canal 解析 Binlog（变成 JSON 格式：`{before:..., after:..., type: UPDATE}`），发送到 MQ
  5. 消费者服务消费 MQ，写入 ES
- **优点**：
  - **完全解耦**：业务代码一行都不用改，随便你怎么写 SQL，只要数据变了，ES 就会自动变
  - **零丢失**：Canal 支持断点续传
- **缺点**：
  - 架构复杂，维护成本高（需要维护 Canal、MQ、Consumer 组件）

#### 💡 选型建议

| 场景                          | 推荐方案              | 理由                                                     |
| ----------------------------- | --------------------- | -------------------------------------------------------- |
| **初创公司 / 小项目**         | **方案一 (同步双写)** | 还没遇到高并发，先活下来最重要。代码简单，出了问题好排查 |
| **中型项目 / 对实时性要求高** | **方案二 (MQ)**       | 业务代码改动可控，利用 MQ 保证最终一致性                 |
| **大型架构 / 遗留系统改造**   | **方案三 (Canal)**    | 业务代码不能动，数据量大，必须完全解耦                   |



### 9.2 生产环境避坑：脑裂

这是一个导致集群数据永久丢失的恐怖问题

- **现象**： 一个集群中突然出现了**两个 Master 节点**

  - 节点 A 认为自己是老大，节点 B 也认为自己是老大
  - 结果：索引数据一部分写入了 A 管理的分片，一部分写入了 B 管理的分片。数据四分五裂，无法合并

- **原因**： 网络波动导致节点间通信中断。A 以为 B 挂了，自己当了主；B 以为 A 挂了，自己也当了主

- **解决方案 (Elasticsearch 7.x 以前)**： 必须配置 `discovery.zen.minimum_master_nodes = (N/2) + 1`

  - 比如 3 个节点，设为 2。只有得到 2 票才能当 Master

- **现状 (Elasticsearch 7.x+ / 8.x)**： 

  - 🎉 **好消息**：ES 7.0 以后引入了全新的集群协调子系统，**自动处理了脑裂问题**

    只要你配置好 `cluster.initial_master_nodes`，ES 会自动通过 Raft 算法选举，不再需要人工计算这个公式

    但**网络分区**依然是集群最大的杀手，保证内网的稳定性至关重要



## 10. 进阶实战：性能优化与零停机运维

### 10.1 写入性能优化

ES 默认的配置是为了“搜索实时性”优先（数据写入 1 秒后就能搜到）。但在海量数据导入场景下，这会严重拖慢写入速度

如果你的 ES 写入遇到瓶颈（CPU 飙升、拒绝连接），请尝试以下三大优化手段：



#### 1. 调整刷新间隔 (Refresh Interval) —— 效果最立竿见影

- **原理**： ES 默认 `refresh_interval: "1s"`。这意味着每秒钟 ES 都要把内存里的数据写到文件系统缓存中，并生成一个新的 Segment 文件。这会消耗大量的 CPU 和 I/O
- **优化策略**：
  - **日常运行**：如果业务允许 30 秒延迟，可以设为 `"30s"`
  - **批量导入时**：直接设为 `"-1"`（关闭自动刷新）。等导完几亿条数据后，再改回来

```js
PUT /products/_settings
{
  "index": {
    "refresh_interval": "30s" 
  }
}
```



#### 2. 也是老生常谈：Bulk 批处理

- **原理**：减少网络开销和线程切换
- **建议**：
  - 单次 Bulk 大小建议在 **5MB - 15MB** 之间
  - Spring Data Elasticsearch 默认的 `saveAll()` 底层就是 Bulk



#### 3. 异步 Translog (Translog Durability)

- **原理**： 

  - 为了保证数据不丢，ES 默认每写一条数据，都会把操作记录 fsync 到磁盘的 Translog 文件（`request` 模式）

    这和 MySQL 的“双一”策略类似，非常安全但非常慢

- **优化策略**： 如果允许在极端宕机情况下丢失最近 5 秒的数据（例如日志场景），可以改为 **异步刷盘 (async)**

```js
PUT /products/_settings
{
  "index.translog.durability": "async",
  "index.translog.sync_interval": "5s"
}
```



### 10.2 零停机索引重构：别名 (Alias) 的妙用

**痛点**： 

- ES 的 Mapping 是 **不可变** 的。如果你把 `price` 字段定义成了 `integer`，后来发现不够用，想改成 `double`，或者想修改分词器，必须**重建索引**

  但在生产环境，你不能把服务停了来做这件事

**解决方案：永远使用别名 (Always use Alias)**

#### 1. 什么是别名？

别名就像是数据库的“视图”或者域名的 CNAME

- **物理索引**：`products_v1`（真实存储数据的地方）
- **逻辑别名**：`products`（应用层代码使用的名字）

**应用层代码**：

```java
@Document(indexName = "products") // 永远只指向别名
public class Product { ... }
```



#### 2. 零停机迁移实战步骤 (The Blue-Green Deployment)

假设当前应用正在使用 `products_v1`

**Step 1: 创建新索引 (Create V2)** 创建一个新的索引 `products_v2`，修改 Mapping（比如把 price 改成 double）

```js
PUT /products_v2
{
  "mappings": { ...新结构... }
}
```

**Step 2: 数据迁移 (Reindex)** 使用 `_reindex` API 把 V1 的数据抄到 V2。这个过程是后台进行的，不影响 V1 的正常读写（只要不是并发写入极高）。

```js
POST /_reindex
{
  "source": { "index": "products_v1" },
  "dest":   { "index": "products_v2" }
}
```

**Step 3: 原子切换 (Atomic Switch) —— 关键** 这是最骚的一步。我们需要在一个原子操作中，把别名 `products` 从 V1 身上撕下来，贴到 V2 身上。 **用户完全无感知。**

```js
POST /_aliases
{
  "actions": [
    { "remove": { "index": "products_v1", "alias": "products" } },
    { "add":    { "index": "products_v2", "alias": "products" } }
  ]
}
```

**Step 4: 验证与清理** 现在应用查 `products`，实际上查的是 `products_v2`。确认无误后，可以删除 `products_v1` 回收空间。

```js
DELETE /products_v1
```