# WebSocket

## 1. WebSocket 概述

### 1.1 什么是 WebSocket

#### 简述

WebSocket 是一种在**单个 TCP (Transmission Control Protocol) 连接**上进行**全双工通信**的标准化网络协议。它被定义在 [RFC 6455](https://tools.ietf.org/html/rfc6455) 中

- 其核心设计目标是提供一个低开销、低延迟的渠道，允许客户端和服务器之间进行**实时、双向的数据交换**

- 与传统的 HTTP 协议不同，WebSocket 连接一旦建立，就会保持持久打开状态，允许数据在两个方向上 **同时**、**主动** 地流动，而不需要像 HTTP 那样依赖严格的“请求-响应”循环

#### 核心特点详解

- **全双工通信**

  - **定义**：指在通信的任意时刻，通信双方可以 **同时** 发送和接收数据，就像一个双向车道

  - **对比 (HTTP)**：

    - 传统的 HTTP 1.x 是“半双工”的。在一个连接上，必须是客户端先发起请求（Request），服务器才能回送响应（Response）。

      在这个周期完成之前，服务器不能主动向客户端发送数据，客户端也不能在发送请求后、收到响应前发送第二个请求（注：HTTP Pipelining 尝试解决但未普及，HTTP/2 通过多路复用在单个连接上实现了并发，但本质上仍是请求-响应模型）



- **持久连接**

  - **机制**：WebSocket 通过一个初始的 HTTP "Upgrade" 握手来建立连接。一旦握手成功，底层的 TCP 连接就从 HTTP 协议切换到了 WebSocket 协议

  - **优势**：

    - 这个 TCP 连接会一直保持开放，直到客户端或服务器任意一方明确地关闭它

      这完全避免了 HTTP 模式下为获取新数据而频繁建立和断开 TCP 连接（或发送 keep-alive 心跳）所带来的巨大开销



- **低延迟**
  - **原因 1 (无请求开销)**：数据交换不再需要等待客户端发起“请求”。服务器一旦有新数据，就可以立即将其推送给客户端
  - **原因 2 (协议开销小)**：WebSocket 的数据帧（Frame）头部非常轻量。一个数据帧的头部最小可以只有 2 字节（相比之下，一个典型的 HTTP 请求头部动辄几百字节）。在需要频繁发送小块数据的场景（如在线游戏、协同编辑）下，这种效率提升是极其显著的



- **协议切换**
  - WebSocket 并非一个全新的独立协议，它依赖于 HTTP 来发起初始连接。它使用 HTTP 的 `Upgrade` 头部来“请求升级”协议
  - 这种设计允许 WebSocket 服务和 HTTP 服务共享同一个端口（如 80 或 443），便于穿透防火墙和代理服务器

### 1.2 WebSocket 的诞生背景

WebSocket 的出现是为了解决 Web 应用中日益增长的 **实时双向通信** 需求，而传统的 HTTP 协议在该方面存在根本性的设计缺陷

#### HTTP 协议的限制

1. **客户端驱动的请求-响应模式**

   - HTTP 协议规定，通信必须由客户端（浏览器）发起。服务器在收到请求之前，无法主动向客户端推送任何信息
   - 这对于需要服务器实时更新数据的应用（如股票行情、聊天室）来说，是一个巨大的障碍

   

2. **高昂的头部开销 (Header Overhead)**

   - HTTP 是无状态协议

     为了在每次请求中识别用户和传递上下文，每个 HTTP 请求都需要携带一个完整的头部，包含 Cookies、User-Agent、Accept 等大量冗余信息

   - 在实时通信中，消息通常很小（例如一条聊天消息或一个坐标点），但头部可能比消息体大几十倍，造成了极大的带宽浪费

   

3. **连接的短暂性**

   - 尽管 `HTTP Keep-Alive` 允许在一段时间内复用 TCP 连接，但它本质上还是为“请求-响应”服务的

     长时间的空闲连接仍会被中间设备（如代理、防火墙）断开



#### 传统的“伪实时”解决方案及其缺陷

在 WebSocket 出现之前，开发者们使用各种技术来模拟服务器推送，这些技术被称为 **Comet**（彗星技术）：

1. **轮询 (Polling)**

   - **机制**：客户端使用 JavaScript 定时器（如 `setInterval`），每隔一个固定的时间（例如 1 秒）就向服务器发送一次 HTTP 请求，询问“是否有新数据？”
   - **缺点**：
     - **高延迟**：数据的实时性取决于轮询间隔。间隔太长，延迟高；间隔太短，请求量剧增
     - **服务器压力**：服务器需要处理海量的、绝大多数时间都是“没有新数据”的无效请求
     - **带宽浪费**：大量的 HTTP 头部开销

   

2. **长轮询 (Long Polling)**

   - **机制**：

     1. 客户端发送一个 HTTP 请求
     2. 服务器 **不立即响应** ，而是保持（Hold）这个连接
     3. 当服务器 **有新数据** 时，它才将数据作为响应发送给客户端，并 **关闭** 当前连接
     4. 客户端收到响应后， **立即** 发起一个新的长轮询请求，重复此过程
     5. 如果长时间没有数据（例如 30 秒超时），服务器会发送一个空响应来关闭连接，客户端收到后同样立即发起新请求

   - **缺点**：

     - **时序问题**：在服务器响应和客户端发起新请求的短暂间隙中，可能会有新数据到达，导致延迟
     - **连接开销**：
       - 虽然减少了无效请求，但每次数据交换**仍然需要一次 TCP 连接的建立和 HTTP 握手**（如果 Keep-Alive 超时）或至少一次完整的 HTTP 请求-响应周期，头部开销依然存在
     - **服务器资源**：在服务器端长时间保持大量挂起的 HTTP 连接，对服务器的连接管理能力是一个挑战

     

3. **SSE (Server-Sent Events)**

   - **机制**：也称为 "EventSource"。客户端发起一个 HTTP 请求，服务器响应一个特定的 `Content-Type (text/event-stream)` 并保持连接开放。服务器可以通过这个连接**单向**地向客户端推送数据
   - **优点**：基于标准 HTTP，实现简单，支持自动重连
   - **缺点**：它是**单向**的。服务器可以推给客户端，但客户端**不能**通过此连接向服务器发送数据（客户端发送数据仍需发起新的 HTTP Post/Put 请求）。这使它不适用于聊天室、游戏等需要双向交互的场景

**总结**：WebSocket 的出现，正是为了用一个标准化的、高效的、真正的全双工方案，来取代上述所有存在缺陷的“模拟”技术



### 1.3 WebSocket 应用场景

WebSocket 的核心价值在于 **低延迟** 和 **双向通信**。任何符合这两个特点的场景都非常适合使用它：

- **即时通讯 (IM)**
  - *需求*：聊天消息、在线状态（上线/下线/输入中...）的实时发送和接收
  - *示例*：Web 聊天室、在线客服系统、移动端 App 的消息推送
- **实时协作**
  - *需求*：多人同时对一份数据进行操作，操作信令需要被毫秒级地广播给所有参与者
  - *示例*：在线协同文档（如 Google Docs）、共享电子白板、Trello/Jira 等工具的状态实时同步
- **金融交易与数据推送**
  - *需求*：股票价格、期货行情、汇率等数据瞬息万变，必须在数据产生时立即推送到客户端
  - *示例*：股票行情软件、在线交易平台、实时数据大屏
- **游戏开发**
  - *需求*：玩家的操作（移动、开火）需要低延迟地发送给服务器，服务器的计算结果（游戏状态、其他玩家位置）需要实时广播给所有玩家
  - *示例*：多人在线网页游戏 (MMO)、棋牌类游戏、实时对战
- **物联网 (IoT)**
  - *需求*：大量传感器设备需要持续不断地向服务器上报数据，同时服务器也需要能随时向特定设备下发控制指令
  - *示例*：智能家居控制、工业设备状态监控、车联网数据采集
- **直播互动**
  - *需求*：视频流本身可能使用 RTMP/HLS 协议，但直播间的 **互动功能** （如弹幕、礼物、点赞、实时评论）需要 WebSocket 来保证所有观众的即时参与感
- **系统监控与日志**
  - *需求*：实时展示服务器的性能指标（CPU、内存）、业务指标，或实时滚动显示应用日志
  - *示例*：`htop`/`top` 命令的 Web 版、ELK/Grafana 的实时数据展示、CI/CD 流水线的实时构建日志



------

## 2. WebSocket 协议详解

### 2.1 WebSocket URL 方案

WebSocket 定义了两种新的 URL 方案，用于标识 WebSocket 端点：

- **`ws://` (WebSocket)**
  - **描述**：用于 **非加密** 的 WebSocket 连接
  - **默认端口**：`80`
  - **类比**：对应 HTTP 协议中的 `http://`
- **`wss://` (WebSocket Secure)**
  - **描述**：用于 **加密** 的 WebSocket 连接，它在 TLS (SSL) 层之上运行
  - **默认端口**：`443`
  - **类比**：对应 HTTPS 协议中的 `https://`

这种设计（`ws` 对应 `http`，`wss` 对应 `https`）使得 WebSocket 能够与现有的 Web 基础设施（如防火墙、代理服务器）兼容，因为它们使用了与 HTTP/HTTPS 相同的默认端口

### 2.2 握手过程 (Handshake)

WebSocket 协议并非一个完全独立的协议，它的连接建立过程 **复用** 了 HTTP 协议。客户端必须发起一个特殊的 HTTP 请求（称为“升级请求”），服务器在验证通过后，双方才会将底层的 TCP 连接从 HTTP 协议“切换”到 WebSocket 协议

这个过程被称为 **WebSocket 握手**

#### 2.2.1 客户端请求 (Client Request)

客户端（通常是浏览器）发起一个标准的 HTTP GET 请求，但包含了几个特定的“升级”头部字段：

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: [http://example.com](http://example.com)
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Extensions: permessage-deflate
```



**关键头部字段详解：**

- `GET /chat HTTP/1.1`
  - 请求行。这表明 WebSocket 连接是向 `ws://server.example.com/chat` 发起的。它必须是一个 `GET` 请求



- `Upgrade: websocket`
  - **核心意图**。客户端明确告知服务器：“我希望将这个连接从 HTTP 升级到 WebSocket 协议”



- `Connection: Upgrade`
  - 一个标准的 HTTP 头部，用来配合 `Upgrade`。它告诉服务器（以及中间的任何代理）这个连接将被用于一种不同的协议，而不是保持为 HTTP



- `Sec-WebSocket-Key`
  - **安全密钥**。这是 WebSocket 握手安全机制的核心
  - **生成**：由客户端 **随机** 生成一个 16 字节的值，然后对其进行 Base64 编码后得到这个字符串
  - **用途**：用于服务器计算配对的 `Sec-WebSocket-Accept` 响应头，以向客户端证明服务器确实“理解” WebSocket 协议，而不是一个碰巧支持 `Upgrade` 头部的老旧 HTTP 服务器



- `Sec-WebSocket-Version: 13`
  - **协议版本**。客户端声明它希望使用的 WebSocket 协议版本。`13` 是 [RFC 6455](https://tools.ietf.org/html/rfc6455) 定义的最终标准版本，目前**所有**现代实现都使用此版本。如果服务器不支持此版本，它会响应一个错误



- `Origin` (可选，浏览器通常会发送)
  - **来源**。浏览器会自动添加此头部，用于声明这个 WebSocket 请求发起的来源页面（域）
  - **安全**：服务器可以使用这个值来进行 **跨域安全检查**（CORS），决定是否接受来自该来源的 WebSocket 连接



- `Sec-WebSocket-Protocol` (可选)

  - **子协议协商**。客户端声明它能够支持的“应用层子协议”列表，按优先级排序

  - **定义**：

    - 子协议是在 WebSocket 之上运行的、由应用自定义的协议（例如 `STOMP`、`MQTT`，或者自定义的 `chat` 协议）

      这允许双方约定一种结构化的数据交换格式



- `Sec-WebSocket-Extensions` (可选)
  - **扩展协商**。客户端声明希望使用的 WebSocket 协议扩展，例如 `permessage-deflate`（帧压缩扩展）

#### 2.2.2 服务器响应 (Server Response)

如果服务器理解并同意升级请求，它将返回一个 `101 Switching Protocols` 状态码，并包含匹配的 `Upgrade` 头部和计算出的 `Sec-WebSocket-Accept` 头部

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

**关键头部字段详解：**

- `HTTP/1.1 101 Switching Protocols`
  - **状态码**。一个明确的信号，表示服务器已同意客户端的协议升级请求。从这一刻起，这个 TCP 连接上的通信协议将发生变更



- `Upgrade: websocket`
  - **确认升级**。服务器确认将协议升级到 `websocket`



- `Connection: Upgrade`
  - **确认切换**。服务器确认连接类型已变更



- `Sec-WebSocket-Accept`
  - **安全响应**。这是服务器对客户端 `Sec-WebSocket-Key` 的“答案”
  - **用途**：客户端会验证这个值，以确认服务器的合法性
  - **计算过程（核心）**：
    1. 获取客户端请求中的 `Sec-WebSocket-Key` 的值（例如：`dGhlIHNhbXBsZSBub25jZQ==`）
    2. 将其与一个固定的“魔术字符串”（GUID）相连接（拼接）：`258EAFA5-E914-47DA-95CA-C5AB0DC85B11`
    3. 拼接后的字符串变为：`dGhlIHNhbXBsZSBub25jZQ==258EAFA5-E914-47DA-95CA-C5AB0DC85B11`
    4. 对这个拼接后的字符串计算 **SHA-1** 哈希值（得到一个 20 字节的二进制值）
    5. 将这个 20 字节的 SHA-1 哈希值进行 **Base64** 编码
    6. 最终得到的 Base64 字符串（例如：`s3pPLMBiTxaQ9kYGzzhZRbK+xOo=`）就是 `Sec-WebSocket-Accept` 的值



- `Sec-WebSocket-Protocol` (如果协商成功)
  - **确认子协议**。如果服务器从客户端的 `Sec-WebSocket-Protocol` 列表中选择了一个它也支持的协议，它就会返回这个头部，并且**只包含那一个**被选中的协议（例如 `chat`）。如果服务器不支持客户端请求的任何子协议，它将**不包含**此头部，连接仍可建立，但双方后续需要自行协调数据格式



**握手总结：**

一旦客户端收到了 `101` 响应，并成功验证了 `Sec-WebSocket-Accept` 的值，握手就成功了

此时，这个 TCP 连接的“身份”就从 HTTP 永久地转变为 WebSocket。之后，双方将开始使用 WebSocket 的 **数据帧**（Data Frame）格式进行全双工通信，而不再使用 HTTP 的请求/响应报文



### 2.3 数据帧格式 (Data Frame Format)

握手成功后，HTTP 协议切换完成。之后所有的数据传输都以 **帧 (Frame)** 的形式进行。帧是 WebSocket 协议中数据传输的最小单位。一个完整的“消息”（Message）可能由一个或多个帧组成

#### 2.3.1 帧结构

WebSocket 帧的二进制结构定义如下：

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-------+-+-------------+-------------------------------+
     |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
     |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
     |N|V|V|V|       |S|             |   (if payload len==126/127)   |
     | |1|2|3|       |K|             |                               |
     +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
     |     Extended payload length continued, if payload len == 127  |
     + - - - - - - - - - - - - - - - +-------------------------------+
     |                               |Masking-key, if MASK set to 1  |
     +-------------------------------+-------------------------------+
     | Masking-key (continued)       |          Payload Data         |
     +-------------------------------- - - - - - - - - - - - - - - - +
     :                     Payload Data continued ...                :
     + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
     |                     Payload Data continued ...                |
     +---------------------------------------------------------------+
```



#### 2.3.2 关键字段详解

- **FIN (1 bit)**

  - **含义**："Final frame" 标记
  - **`1`**：表示这是消息（Message）的**最后一帧**
  - **`0`**：表示这不是最后一帧，后续还会有“延续帧”
  - **用途**：用于 **消息分片 (Message Fragmentation)**。一个大的消息（如一个 10MB 的文件）可以被拆分成多个帧来发送

  

- **RSV1, RSV2, RSV3 (各 1 bit)**

  - **含义**："Reserved" 保留位
  - **用途**：必须为 `0`，除非协商了某个 WebSocket 扩展（如 `permessage-deflate` 压缩扩展）来使用它们

  

- **Opcode (4 bits)**

  - **含义**："Operation code"，操作码，定义了此帧的 **类型**
  - **`0x0` (%x0)**：**延续帧 (Continuation Frame)**。如果消息被分片，除第一帧外的所有后续帧的 Opcode 都必须是 `0x0`
  - **`0x1` (%x1)**：**文本帧 (Text Frame)**。Payload Data 必须是 UTF-8 编码的文本
  - **`0x2` (%x2)**：**二进制帧 (Binary Frame)**。Payload Data 是任意的二进制数据（如图片、音频）
  - **`0x8` (%x8)**：**关闭连接帧 (Connection Close Frame)**。用于发起连接关闭
  - **`0x9` (%x9)**：**Ping 帧**。用作心跳检测
  - **`0xA` (%xA)**：**Pong 帧**。作为对 Ping 帧的响应
  - **注**：`0x8`, `0x9`, `0xA` 是 **控制帧 (Control Frames)**。控制帧不能被分片（FIN 必须为 `1`）且载荷长度（Payload len）有最大限制（<= 125 字节）

  

- **MASK (1 bit)**

  - **含义**："Masked" 掩码标记
  - **用途**：一个**强制的安全特性**
  - **`1`**：表示此帧的 Payload Data 经过了“掩码”处理。**所有**从客户端发送到服务器的帧，此位**必须**为 `1`
  - **`0`**：表示此帧的 Payload Data 未经掩码处理。**所有**从服务器发送到客户端的帧，此位**必须**为 `0`

  

- **Payload len (7 bits) / Extended payload length (16 or 64 bits)**

  - **含义**：Payload Data 的长度（字节）
  - **读取逻辑**：
    1. 如果这个 7 bit 值为 `0` 到 `125`，那么它 **就是** 真实的长度
    2. 如果值为 `126` (二进制 `1111110`)，则**后续 2 个字节 (16 bits)** 才代表真实的长度（最大 65535 字节）
    3. 如果值为 `127` (二进制 `1111111`)，则**后续 8 个字节 (64 bits)** 代表真实的长度（用于超大数据）

  

- **Masking-key (0 or 4 bytes)**

  - **含义**：掩码密钥
  - **存在性**：**仅当** MASK 位为 `1` 时（即客户端发往服务器时），这 4 个字节（32 bits）的掩码密钥才会存在
  - **用途**：Payload Data 中的每个字节都与这个密钥进行了按位**异或 (XOR)** 运算

  

- **Payload Data (N bytes)**

  - **含义**：有效载荷
  - **内容**：如果是文本帧，这里是 UTF-8 字符串；如果是二进制帧，这里是原始字节
  - **掩码**：如果 MASK 位为 `1`，这里存储的是被掩码（XOR）**之后**的数据。服务器收到后需要用 Masking-key 再次进行 XOR 运算来还原原始数据



#### 2.3.3 核心概念：消息分片

- **为什么需要分片？**
  - 允许发送一个“未知总长度”的消息（例如实时生成的视频流）
  - 允许在有限的缓冲区内处理大消息，避免一次性占用过多内存
  - 允许大消息和高优先级的“控制帧”（如 Ping/Pong）在网络中交错传输，防止控制帧被大消息阻塞
- **分片示例**：发送一个大的文本消息
  1. **第一帧**：`FIN=0`, `Opcode=0x1` (文本帧), `Payload=消息的第一部分`
  2. **第二帧**：`FIN=0`, `Opcode=0x0` (延续帧), `Payload=消息的第二部分`
  3. ...
  4. **最后N帧**：`FIN=1`, `Opcode=0x0` (延续帧), `Payload=消息的最后部分`



#### 2.3.4 核心概念：掩码 (Masking)

- **为什么需要掩码？(且仅限客户端到服务器)**
  - 这**不是为了加密**（XOR 运算是可逆的，密钥也在同一个帧里），而是为了**防止“代理缓存中毒攻击” (Cache Poisoning)**
  - 某些中间代理服务器（特别是透明代理）可能会缓存它们认为是 HTTP 的通信内容。如果一个恶意客户端构造了特定的、未经掩码的 WebSocket 帧，其内容可能被代理误认为是一个 HTTP 响应（例如 `HTTP/1.1 200 OK ...`），并将其缓存
  - 当一个正常用户通过该代理访问某个网站时，代理可能会错误地返回这个被“投毒”的缓存，导致安全问题
  - 通过强制客户端对每一帧数据使用一个**随机**的 4 字节 Masking-key 进行 XOR 运算，可以确保从客户端发出的数据对于中间代理来说是“不可预测的”的随机字节流，从而防止它们被误解和缓存



### 2.4 心跳机制 (Heartbeat)

#### 2.4.1 为什么需要心跳？

1. **连接保活 (Keep-Alive)**
   - 许多网络中间设备（如 NAT 网关、企业防火墙、负载均衡器）会自动关闭它们认为“空闲”的 TCP 连接（例如 300 秒内没有数据包通过）
   - WebSocket 连接虽然是持久的，但如果长时间没有业务数据交换（例如用户在聊天室挂机），连接也可能被这些中间设备强行断开
2. **“僵尸连接”检测 (Zombie Connection Detection)**
   - 有时一方（如客户端）可能因断电、网络切换、崩溃等原因异常掉线，导致它没来得及发送 `0x8` 关闭帧
   - 此时，另一方（服务器）并不知道连接已失效，它持有一个“半开放”的 TCP 连接（也称“僵尸连接”），这会白白消耗服务器资源



#### 2.4.2 Ping/Pong 帧

WebSocket 协议内置了**控制帧** `Ping` (0x9) 和 `Pong` (0xA) 来解决上述问题

- **机制**：

  1. 一方（通常是服务器）可以向另一方发送一个 `Ping` 帧
  2. 接收方（客户端）在收到 `Ping` 帧后，**必须**（协议规定）尽快回复一个 `Pong` 帧
  3. `Ping` 帧可以携带可选的 Payload Data（例如，发送方的时间戳）
  4. `Pong` 帧 **必须** 原封不动地返回 `Ping` 帧中携带的 Payload Data（这可以用来计算网络延迟 RTT）

- **实现**：

  - **服务器端 (Spring)**：

    - 可以定时（例如每 30 秒）向所有连接的客户端发送 `Ping` 帧

      如果在一个设定的超时时间（例如 10 秒）内没有收到对应的 `Pong` 响应，服务器就可以认为该连接已失效，并主动关闭它，释放资源

  - **客户端 (Browser)**：

    - 现代浏览器在收到 `Ping` 帧时，会自动在底层回复 `Pong` 帧，这个过程通常**不会**暴露给 JavaScript 应用层（即你无法在 JS 的 `onmessage` 中收到 Ping）

  - **客户端 (Java 等)**：非浏览器的客户端（如 Java 编写的 WebSocket 客户端）通常可以完全控制 Ping/Pong 的发送和接收，如你的示例代码所示

```java
// Java WebSocket API (JSR 356) 示例：
// 服务器主动发送 Ping
// "ping" 字符串会被编码为 bytes
session.getBasicRemote().sendPing(ByteBuffer.wrap("ping-payload".getBytes()));

// 监听 Pong 消息
@OnMessage
public void onPong(PongMessage pong, Session session) {
    // 检查 Pong 的载荷是否与之前 Ping 的载荷匹配
    // ByteBuffer payload = pong.getApplicationData();
    System.out.println("Pong received from client: " + session.getId());
}
```

**心跳总结**：通过定期发送 Ping 并期望收到 Pong，服务器既能确保连接不被中间设备视为空闲而断开，也能及时发现并清理那些已经失效的客户端连接



---

## 3. WebSocket vs HTTP

WebSocket 和 HTTP 都是构建在 TCP/IP 协议栈之上的应用层协议，但它们的设计哲学、通信模式和适用场景截然不同

### 3.1 核心差异对比

| 特性         | HTTP (HyperText Transfer Protocol)                           | WebSocket (WebSocket Protocol)                               |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **通信模式** | **请求-响应 (Request-Response)**                             | **全双工 (Full-Duplex)**                                     |
|              | 严格的客户端发起，服务器响应<br />服务器不能主动向客户端发送数据 | 连接建立后，客户端和服务器可以 **同时**、**主动** 地向对方发送数据 |
| **连接状态** | **无状态 (Stateless)**                                       | **有状态 (Stateful)**                                        |
|              | 每个请求都是独立的<br />服务器默认不保留客户端的连接信息（`Keep-Alive` 只是复用 TCP，协议层仍是无状态的） | 基于**持久连接**<br />服务器和客户端在整个连接生命周期内都“知道”对方的存在，可以维持会话状态 |
| **数据流向** | **单向发起**                                                 | **双向**                                                     |
|              | 只能由客户端发起请求                                         | 双方地位平等，均可主动发起数据传输                           |
| **协议标识** | `http://` (非加密), `https://` (加密)                        | `ws://` (非加密), `wss://` (加密)                            |
| **默认端口** | `80` (http), `443` (https)                                   | `80` (ws), `443` (wss)                                       |
|              | WebSocket 巧妙地复用了 HTTP/HTTPS 的端口，以实现更好的防火墙穿透性 |                                                              |
| **协议开销** | **高 (High Overhead)**                                       | **极低 (Very Low Overhead)**                                 |
|              | 每个请求都包含完整的 HTTP 头部（几百到几千字节），包含 Cookies, User-Agent 等大量冗余信息 | 握手阶段使用 HTTP 头部。握手成功后，数据帧头部最小仅 **2 字节** |
| **数据格式** | **MIME 类型描述**                                            | **帧 (Frame)**                                               |
|              | 头部定义 `Content-Type`，正文可以是 `text/html`, `application/json`, `image/png` 等 | 协议层面原生支持 **文本帧 (UTF-8)** 和 **二进制帧**          |
| **实时性**   | **低 (通过模拟实现)**                                        | **高 (原生支持)**                                            |
|              | 依赖轮询、长轮询等“拉”(Pull)模式来模拟实时性，延迟高且浪费资源 | 服务器可主动“推”(Push)数据，延迟可达毫秒级                   |
| **跨域策略** | **CORS (Cross-Origin Resource Sharing)**                     | **Origin 头部验证**                                          |
|              | 浏览器有严格的同源策略限制，跨域请求需要服务器在 HTTP 响应中包含 `Access-Control-Allow-Origin` 等头部 | **同样受同源策略限制**<br />但它在**握手阶段**通过 `Origin` 头部进行验证<br />服务器在握手时检查 `Origin` 值，决定是否接受连接<br />它**不使用** HTTP CORS 的 `OPTIONS` 预检请求 |
| **代理支持** | **普遍支持**                                                 | **需要特定配置**                                             |
|              | 几乎所有的代理服务器（正向、反向）都原生理解 HTTP 协议       | 许多老旧的代理服务器只懂 HTTP，它们可能无法正确处理 WebSocket 的持久连接和非 HTTP 格式的数据帧，导致连接被中断。现代的代理服务器（如 Nginx, HAProxy）需要**显式配置**以支持 `Upgrade` 头部和 WebSocket 流量的转发 |



### 3.2 适用场景与选择

#### 3.2.1 HTTP 的主场

HTTP 协议的核心优势在于其 **简单性、无状态性、可缓存性 **和 **广泛的生态支持**

**适用场景：**

- **资源获取**：加载 HTML 页面、CSS、JavaScript 文件、图片、视频等静态资源
- **RESTful API / CRUD 操作**：执行定义清晰的增、删、改、查操作。例如，用户注册、提交表单、发布文章、获取文章列表
- **搜索引擎优化 (SEO)**：HTTP 页面可以被搜索引擎爬虫轻松索引
- **一次性数据请求**：当客户端只需要“一次性”地从服务器获取数据时（例如，查看天气预报）

#### 3.2.2 WebSocket 的主场

WebSocket 的核心优势在于 **低延迟、高频率的双向数据交换** 和 **有状态的长连接**

**适用场景：**

- **高频实时更新**：股票行情、体育比分、服务器性能监控
- **即时通讯**：聊天室、在线客服、"输入中..." 状态显示
- **实时协作**：协同文档编辑、共享白板
- **多人在线游戏**：玩家位置、状态的实时同步
- **需要服务器主动推送的场景**：系统告警、Web Push 通知



### 3.3 现代 Web 架构：HTTP + WebSocket 共存

在绝大多数现代 Web 应用中，HTTP 和 WebSocket **不是互斥的，而是互补的**

一个典型的架构模式是：

1. **使用 HTTP/HTTPS (REST or GraphQL API)**:
   - 处理应用的 **初始化加载**（加载页面、JS/CSS）
   - 处理所有 **“事务性”**、**“状态变更”** 的操作（如登录、注册、提交订单、发布评论）
   - 处理**“低频”**的数据拉取（如加载用户资料页）
2. **使用 WebSocket/WSS**:
   - 在页面加载完成后建立连接
   - 专门用于处理**“实时更新”**的部分
   - **示例**：
     - 你使用 HTTP POST 提交了一篇新博客
     - 服务器处理完 HTTP 请求后，通过 WebSocket 向 **所有** 在线用户（或订阅了该频道的用户） **推送** 一条“有新博客发布”的通知
     - 这样，其他用户的页面无需刷新或轮询，就可以实时看到新内容的提示



**思考：是否总需要 WebSocket？**

- 不一定

  如果你的需求 **只是** 服务器向客户端的 **单向推送**（例如，新闻网站的突发新闻推送），使用 **SSE (Server-Sent Events)** 可能是一个更轻量、更简单的选择

  SSE 运行在标准 HTTP/2 之上，具有断线重连等特性，且无需处理 WebSocket 复杂的协议和代理配置

------

## 4. WebSocket 生命周期

WebSocket 的生命周期指的是从连接发起、建立、数据交换到最终关闭的整个过程。在浏览器端，`WebSocket` 对象通过四个核心事件 `onopen`, `onmessage`, `onerror`, `onclose` 来管理和响应这个周期

### 4.1 连接建立 (Opening)

这是生命周期的第一个阶段，通过 `WebSocket` 构造函数发起，并由 `onopen` 事件确认完成

#### 4.1.1 发起连接

通过调用 `WebSocket` 构造函数并传入 `ws://` 或 `wss://` URL 来**启动**连接过程

```js
// 1. 创建 WebSocket 实例，握手开始
const ws = new WebSocket('ws://localhost:8080/websocket');

// 我们可以设置接收二进制数据的格式
// 'blob' (默认) 或 'arraybuffer'
ws.binaryType = 'arraybuffer';
```

**注意**：`new WebSocket(...)` 只是**开始**了第 2.2 节中描述的 HTTP 握手过程。在这一刻，连接**尚未**建立

#### 4.1.2 onopen：连接成功

当 WebSocket 握手成功（服务器返回 `HTTP 101` 响应）后，`onopen` 事件被触发

```js
// 2. 握手成功后，onopen 事件被触发
ws.onopen = function(event) {
    console.log('连接已建立 (WebSocket Handshake successful)');
    console.log('Event:', event);
    
    // 只有在 onopen 触发后，才代表连接可用
    // 这是发送第一条消息最安全的地方
    ws.send('Hello Server! (Sent from onopen)');
};
```

**关键点**：`onopen` 是你**唯一**可以 100% 确定连接已准备就绪的信号。在 `onopen` 触发之前尝试 `ws.send()` 会导致错误



### 4.2 数据传输 (Data Transfer)

连接建立后，生命周期进入数据传输阶段。这由 `onmessage` (接收) 和 `send()` (发送) 方法主导

#### 4.2.1 `send()`：发送数据

`WebSocket.send()` 方法负责将数据发送到服务器。浏览器 API 会自动根据你传入的数据类型，将其打包成正确的帧类型（Opcode）



**1. 发送文本 (Opcode 0x1)**

- `send()` 会自动将 JavaScript 字符串编码为 UTF-8

```js
ws.send("Hello WebSocket!");
```



**2. 发送结构化数据 (JSON)**

- WebSocket **没有** 原生的 JSON 格式
- **最佳实践**：将 JavaScript 对象 `JSON.stringify()` 转换为字符串，然后作为 **文本帧** 发送。服务器端需要进行反向的 JSON 解析

```js
ws.send(JSON.stringify({
    type: 'message',
    content: 'Hello',
    timestamp: Date.now()
}));
```



**3. 发送二进制数据 (Opcode 0x2)**

- `send()` 可以直接接收 `Blob` 或 `ArrayBuffer` (以及 `TypedArray` 视图) 作为参数

```js
// 发送一个 8 字节的空缓冲区
const buffer = new ArrayBuffer(8);
ws.send(buffer);

// 发送一个包含数据的视图
const view = new Uint8Array(2);
view[0] = 65; // 'A'
view[1] = 66; // 'B'
ws.send(view);
```



#### 4.2.2 `onmessage`：接收数据

当客户端收到来自服务器的 **完整消息** 时（可能是一个或多个数据帧），`onmessage` 事件被触发

`event.data` 属性包含了消息的有效载荷，其 **数据类型** 取决于服务器发送的帧类型：

- 服务器发送 **文本帧 (Opcode 0x1)** -> `event.data` 是一个 `String`
- 服务器发送 **二进制帧 (Opcode 0x2)** -> `event.data` 是一个 `Blob` 或 `ArrayBuffer` (取决于 `ws.binaryType` 的设置)

```js
ws.onmessage = function(event) {
    
    // event.data 包含了服务器发送的有效载荷
    
    if (typeof event.data === 'string') {
        // --- 处理文本数据 ---
        console.log('收到文本消息:', event.data);
        
        // 尝试解析 JSON
        try {
            const jsonMessage = JSON.parse(event.data);
            console.log('解析 JSON 成功:', jsonMessage);
            // handleJsonMessage(jsonMessage);
        } catch (e) {
            // 不是 JSON 格式的纯文本
            // console.warn('Received non-JSON text message.');
        }

    } else if (event.data instanceof ArrayBuffer) {
        // --- 处理 ArrayBuffer (二进制) ---
        // (前提是 ws.binaryType = 'arraybuffer')
        console.log('收到 ArrayBuffer 二进制数据:', event.data);
        const view = new Uint8Array(event.data);
        console.log('Binary data as bytes:', view);
        
    } else if (event.data instanceof Blob) {
        // --- 处理 Blob (二进制) ---
        // (前提是 ws.binaryType = 'blob' (默认))
        console.log('收到 Blob 二进制数据:', event.data);
        
        // 可以使用 FileReader 异步读取 Blob 内容
        // const reader = new FileReader();
        // reader.onload = function() {
        //     console.log('Blob content:', reader.result);
        // };
        // reader.readAsArrayBuffer(event.data); // 或 readAsText 等
    }
};
```



### 4.3 `readyState` 状态（重要）

`WebSocket` 实例拥有一个 `readyState` 属性，用于精确描述当前处于生命周期的哪个阶段。在 `onopen` 回调之外发送数据时，**必须**检查此状态

```js
// WebSocket.readyState 属性值：
// ---------------------------------
// WebSocket.CONNECTING (0): 正在建立连接 (调用构造函数后)
// WebSocket.OPEN       (1): 连接已建立 (onopen 触发后)
// WebSocket.CLOSING    (2): 正在关闭连接
// WebSocket.CLOSED     (3): 连接已关闭 (onclose 触发后)

console.log(ws.readyState); // 可能打印 0 (CONNECTING) 或 1 (OPEN)

/**
 * 一个安全发送数据的包装函数
 */
function safeSend(data) {
    if (ws.readyState === WebSocket.OPEN) {
        ws.send(data);
    } else if (ws.readyState === WebSocket.CONNECTING) {
        console.warn('WebSocket 仍在连接中，稍后重试...');
        // 可以考虑排队，或在 onopen 中发送
    } else {
        console.error('WebSocket 连接已关闭或正在关闭，无法发送。');
    }
}
```



### 4.3 连接关闭 (Closing)

WebSocket 的关闭是一个“双向握手”过程，使用 **关闭帧 (Opcode 0x8)** 来实现的

#### 4.3.1 `close()`：主动关闭

客户端或服务器都可以通过发送一个“关闭帧”来**发起**关闭过程。在浏览器端，我们调用 `ws.close()` 方法

`ws.close()` 方法是**异步**的。调用它会使 `readyState` 变为 `CLOSING (2)`

```js
// 语法: ws.close(code, reason);
// - code: (可选) 一个数字状态码，解释关闭原因 (见 4.3.3)
// - reason: (可选) 一个字符串，提供更详细的关闭原因 (UTF-8, <= 123 字节)

// 示例：客户端主动正常关闭
ws.close(1000, "User logged out");
```



#### 4.3.2 `onclose`：关闭完成

`onclose` 事件在连接**完全关闭**（`readyState` 变为 `CLOSED (3)`）时触发。无论关闭是由客户端发起、服务器发起还是由网络错误引起的，`onclose` **总是**会被触发

`onclose` 事件对象 `event` 包含三个重要属性：

- `event.code`：服务器或客户端提供的关闭状态码（数字）
- `event.reason`：关闭原因的描述（字符串）
- `event.wasClean`：一个**非常重要**的布尔值
  - `true`：表示连接是通过正常的“关闭握手”（双方都发送了关闭帧）完成的。这通常对应 `code 1000` 或 `1005`
  - `false`：表示连接是**异常**中断的（例如网络断开、服务器进程崩溃、`wss://` 证书无效）。此时 `code` 通常是 `1006`

```js
ws.onclose = function(event) {
    console.log('连接已关闭 (Connection closed)');
    console.log(`- Status Code: ${event.code}`);
    console.log(`- Reason: ${event.reason}`);
    console.log(`- Was Clean: ${event.wasClean}`);

    if (event.wasClean) {
        // 正常关闭，例如用户退出登录
        console.log('WebSocket cleanly closed.');
    } else {
        // 异常关闭 (code === 1006)，例如网络丢失
        console.error('WebSocket connection dropped (Abnormal closure).');
        // 可以在这里触发重连逻辑
        // handleReconnect();
    }
};
```



#### 4.3.3 常用关闭状态码 (Close Code)

| Code     | 含义                            | 解释                                                         |
| -------- | ------------------------------- | ------------------------------------------------------------ |
| **1000** | **正常关闭 (Normal Closure)**   | **最常见的状态码**。表示连接已成功完成其目的（例如，`ws.close(1000)`） |
| 1001     | 端点离开 (Going Away)           | 服务器或客户端即将下线或重启（例如服务器正在部署新版本）     |
| 1002     | 协议错误 (Protocol Error)       | 一方检测到协议错误（例如收到了非法的帧数据）                 |
| 1003     | 不可接受的数据类型              | 一方收到了它无法处理的数据类型（例如，只支持文本的端点收到了二进制数据） |
| **1005** | *无状态码*                      | （保留值）表示**没有**在关闭帧中提供状态码。浏览器中 `wasClean` 仍为 `true` |
| **1006** | **异常关闭 (Abnormal Closure)** | （保留值）**绝不会**在关闭帧中发送。这是 `onclose` 事件用来通知你连接**未**通过正常握手关闭的特殊代码（`wasClean` 为 `false`）。这通常意味着网络丢失、TCP 连接被强行终止 |
| 1007     | 数据不一致                      | 收到的消息包含与消息类型不符的数据（例如，文本帧包含非 UTF-8 数据） |
| 1008     | 策略违规 (Policy Violation)     | 通用的策略错误。例如服务器因安全策略拒绝了某个消息           |
| 1009     | 消息过大 (Message Too Big)      | 收到的消息超出了其能处理的最大尺寸                           |
| 1011     | 服务器内部错误                  | 服务器遇到了意外情况导致无法处理请求                         |



### 4.5 错误处理 (Error Handling)

#### 4.5.1 `onerror` 事件

`onerror` 事件在 WebSocket 通信过程中发生 **错误** 时触发

```js
ws.onerror = function(event) {
    // onerror 事件对象通常**不包含**详细的错误信息
    // 只有一个通用的 "Event" 对象
    console.error('WebSocket 错误发生:', event);
};
```



**`onerror` 和 `onclose` 的关系（非常重要）：**

1. WebSocket 的 `onerror` 事件 **非常** 基础，它本身几乎不提供任何有用的调试信息（`event` 对象里基本是空的）
2. `onerror` 的触发 **几乎总是** 伴随着连接的关闭
3. 当发生一个导致连接中断的错误时（例如，初始 `wss://` 握手时证书无效、网络连接突然断开），`onerror` 事件会 **首先** 被触发
4. 紧接着，`onclose` 事件会 **立即** 被触发

因此，**不要**在 `onerror` 中实现重连逻辑。所有的连接状态判断和重连逻辑都应该放在 `onclose` 事件中，通过检查 `event.wasClean` 和 `event.code` 来执行

```js
// 推荐的错误/关闭处理模式
ws.onclose = function(event) {
    if (event.wasClean) {
        // 正常关闭
        console.log(`[close] Connection closed cleanly, code=${event.code} reason=${event.reason}`);
    } else {
        // 异常关闭 (e.g. process killed or network down)
        // event.code 此时通常是 1006
        console.error(`[close] Connection died (Abnormal closure). Code=${event.code}`);
        
        // --- 在这里实现重连逻辑 ---
        // setTimeout(function() {
        //   connect(); // 尝试重新连接
        // }, 5000); // 5秒后重试
    }
};

ws.onerror = function(error) {
    // onerror 只是一个信号，真正的处理在 onclose
    console.error(`[error] WebSocket Error: ${error.message || 'Unknown error'}`);
};
```

------

## 5. STOMP 协议

### 5.1 为什么需要 STOMP？

我们首先要理解一个核心问题：**WebSocket 协议本身是不够的**

- **WebSocket (RFC 6455)**：它只是一种 **通信协议** ，定义了如何在客户端和服务器之间建立一个双向的“管道”。它只规定了如何传输“文本帧”和“二进制帧”
- **问题在于**：WebSocket 协议本身 **没有定义** 消息的 **语义**。例如：
  - 一个文本帧 `{"user":"A", "message":"hello"}` 应该被发送给谁？
  - 客户端如何告诉服务器它只想“订阅”关于“股票代码: GOOG”的更新？
  - 服务器如何实现“发布/订阅” (Pub/Sub) 模式，将一条消息广播给所有订阅者？



**STOMP (Simple Text Oriented Messaging Protocol)** 就是来解决这个问题的

- **STOMP 是一种“应用层”协议**，它运行在 WebSocket (或其他传输方式) **之上**
- 它模仿了消息队列 (MQ) 的模式，定义了一套简单的、基于文本的**命令**，如 `CONNECT` (连接)、`SUBSCRIBE` (订阅)、`SEND` (发送) 等
- 它引入了“**目的地 (Destination)**”的概念，让消息有了明确的去处（例如 `"/topic/news"` 或 `"/app/chat"`）

**总结：WebSocket 是“传输管道”，而 STOMP 是在管道里跑的“消息规范”**



### 5.2 STOMP 协议帧

STOMP 是基于“帧” (Frame) 的。每个帧都是纯文本，格式如下：

```
COMMAND
header1:value1
header2:value2

Body^@
```

- `COMMAND`：大写的命令 (如 `CONNECT`, `SEND`)
- `Headers`：键值对，提供关于消息的元数据（例如 `destination`）
- `Body`：消息体（例如 JSON 字符串）
- `^@`：帧的结束符 (一个 NULL 字符)



**常用命令 (Commands)：**

| 命令           | 发送方 | 目的                                  | 关键头部 (Header)                           |
| -------------- | ------ | ------------------------------------- | ------------------------------------------- |
| `CONNECT`      | Client | 请求连接到 STOMP 服务器。             | `accept-version`, `host`                    |
| `CONNECTED`    | Server | 响应 `CONNECT`，表示连接成功。        | `version`                                   |
| `SEND`         | Client | 向某个“目的地”发送消息。              | `destination`                               |
| `SUBSCRIBE`    | Client | 订阅某个“目的地”，以便接收消息。      | `destination`, `id` (订阅ID)                |
| `UNSUBSCRIBE`  | Client | 取消订阅。                            | `id` (要取消的订阅ID)                       |
| `MESSAGE`      | Server | 服务器将消息推送给已订阅的客户端。    | `destination`, `subscription`, `message-id` |
| `ACK` / `NACK` | Client | （高级特性）用于客户端确认/拒绝消息。 | `id` (要确认的消息ID)                       |
| `DISCONNECT`   | Client | 请求断开连接。                        |                                             |

---

## 6. Spring 与 WebSocket 集成

### 起步与依赖

#### 1 为什么要用 WebSocket？

在传统的 HTTP 协议中，通信只能由客户端发起（“请求-响应”模式）。服务器想主动给客户端发消息（比如股票变动、新消息通知），非常困难，通常只能靠客户端不断地“轮询”询问

**WebSocket** 解决了这个问题：它在客户端和服务器之间建立了一条 **全双工（Full-Duplex）** 的长连接

- **全双工**：服务器和客户端都可以随时向对方发送消息，就像打电话一样，而不是像对讲机（说完一句还得说“Over”）

#### 2 核心依赖配置 ⭐

在 Spring Boot 中集成 WebSocket 非常简单，通常只需要引入**一个**核心依赖

##### 1 引入依赖 (Maven)

请在你的 `pom.xml` 中加入以下依赖：

```xml
<!-- Spring Boot WebSocket 核心启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```



##### 2 这个 Starter 干了什么？

很多初学者会问：“我需要加 `spring-messaging` 吗？我需要加 Tomcat 支持吗？” 答案是：**通常不需要，这个 Starter 是“一站式”解决方案**

它自动包含了以下关键组件：

1. **`spring-websocket`**：提供底层的 WebSocket API（处理握手、连接、原始消息）
2. **`spring-messaging`**：提供高层的消息协议支持（这是后续使用 STOMP 协议的基础）
3. **Servlet 容器支持**：它会自动适配你当前的容器（Tomcat, Jetty 或 Undertow），开启它们对 WebSocket 的支持



#### 3 🚫 常见陷阱与最佳实践

##### 💥 陷阱 1：依赖重复添加

**错误做法**： 有些旧教程会教你同时添加 `spring-boot-starter-websocket` 和 `spring-boot-starter-messaging`

**正确做法**： 

- **只加 `spring-boot-starter-websocket` 即可** 

  在 Spring Boot 2.x 及后续版本中，前者已经传递依赖了后者。重复添加虽然通常不会报错，但会让你的依赖树变得臃肿且难以维护



##### 💥 陷阱 2：版本不兼容

- 如果你是将 WebSocket 集成到现有的老旧 Spring (非 Boot) 项目中，可能会遇到 Servlet 版本过低不支持 WebSocket 的情况

- **最佳实践**：

  - 确保你的 Servlet 容器（如 Tomcat）支持 **Servlet 3.1+** 规范，这是 Java WebSocket 的最低要求

    当然，如果你用的是 Spring Boot 内置容器，这完全不用担心



#### 📝 总结

- WebSocket 是为了解决 **服务器无法主动推送** 的痛点
- 你只需要记住一个依赖：**`spring-boot-starter-websocket`**
- 它既支持“原生”的 WebSocket 开发，也支持高级的 STOMP 协议开发



### 原生 WebSocket 开发

在这一节，我们不使用任何高级协议，直接操作最底层的 WebSocket 管道

#### 1 什么是“原生”支持？

Spring 提供了 `WebSocketHandler` 接口，它让我们能够直接处理 WebSocket 的生命周期事件：

1. **连接建立** (Connection Established)
2. **收到消息** (Message Received)
3. **连接关闭** (Connection Closed)



**适用场景**：

- 你需要极致的性能，且消息格式很简单（比如只是推送一个数字）
- 你的客户端不支持 STOMP 协议（如某些物联网设备）
- 你想完全控制二进制数据的传输



#### 2 第一步：编写处理器 (Handler)

我们需要写一个类来决定“收到消息后做什么”。通常我们继承 `TextWebSocketHandler`，因为它帮我们处理了大部分通用逻辑

```java
import org.springframework.web.socket.*;
import org.springframework.web.socket.handler.TextWebSocketHandler;
import java.io.IOException;
import java.util.concurrent.ConcurrentHashMap;

public class MyNativeHandler extends TextWebSocketHandler {

    /**
     * 🧠 核心难点：会话管理
     * HTTP 是无状态的，但 WebSocket 是长连接。
     * 我们必须自己用一个集合（Map）把所有连上来的用户存起来，
     * 否则服务器一会儿就忘了谁是谁，也无法主动给他们发消息
     */
    // Key: session ID, Value: Session
    private static final ConcurrentHashMap<String, WebSocketSession> SESSIONS = new ConcurrentHashMap<>();

    /**
     * 1. 连接建立成功时调用
     * 这里是初始化的最佳时机
     */
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        // 必须存起来！不然之后就找不到这个用户了
        SESSIONS.put(session.getId(), session);
        System.out.println("用户上线了: " + session.getId());
    }

    /**
     * 2. 收到消息时调用
     * 客户端发什么，这里就收到什么
     */
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String payload = message.getPayload();
        System.out.println("收到来自 " + session.getId() + " 的消息: " + payload);

        // 示例：原样弹回 (Echo)
        // 注意：发送消息是阻塞的，高并发下建议放到线程池处理
        if (session.isOpen()) {
            session.sendMessage(new TextMessage("服务器收到了: " + payload));
        }
    }

    /**
     * 3. 连接关闭时调用
     * 不管是用户主动断开，还是网络异常断开，都会触发
     */
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        // 💥 重要：必须移除，否则 Map 会越来越大，导致内存泄漏！
        SESSIONS.remove(session.getId());
        System.out.println("用户下线了: " + session.getId());
    }
}
```



#### 3 第二步：注册端点 (Configuration)

写好了处理器，我们需要告诉 Spring：“当有人访问 `/ws/native` 这个地址时，请用上面的 `MyNativeHandler` 来接待”

我们需要创建一个配置类，实现 `WebSocketConfigurer` 接口

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.*;

@Configuration
@EnableWebSocket // 👈 开启 WebSocket 支持
public class WebSocketNativeConfig implements WebSocketConfigurer {

    // 将处理器注册为 Bean
    @Bean
    public MyNativeHandler myNativeHandler() {
        return new MyNativeHandler();
    }

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myNativeHandler(), "/ws/native")
                // 💥 巨坑预警：跨域设置
                // 默认情况下 Spring 为了安全会拒绝跨域连接。
                // 开发环境设为 "*" (允许所有)，生产环境请指定具体域名！
                .setAllowedOrigins("*");
    }
}
```



#### 4 🚫 常见陷阱与避坑指南

这是一个非常重要的部分，原生 WebSocket 开发看似简单，实则暗藏杀机



##### 💥 陷阱 1：多实例部署导致的“消息丢失”

**场景**：

- 你的后端部署了 2 台服务器（A 和 B）,用户小明连上了 A，用户小红连上了 B

  - **问题**：你想让小明给小红发消息

    你在 A 服务器的代码里遍历 `SESSIONS` Map，发现根本找不到小红的 Session

    **原因**：**`ConcurrentHashMap` 是内存级别的，它只存在于当前的 JVM 中。** 服务器 A 根本不知道服务器 B 上有哪些用户

  - **最佳实践**：

    - 如果你的应用需要多节点部署（集群），**原生 WebSocket 几乎不可用**，或者你需要引入 Redis/MQ 做非常复杂的“广播中继”

      这就是为什么我们后面推荐用 STOMP 的原因之一

##### 💥 陷阱 2：并发发送异常

**问题**：

- `WebSocketSession` **不是线程安全的**

  如果你在多个线程中同时尝试向同一个 Session 发送消息（`session.sendMessage(...)`），可能会抛出异常或导致数据错乱

- **最佳实践**：在这个底层模式下，你需要自己通过 `synchronized` 关键字或者队列来保证对同一个 Session 的写入是串行的



##### 💥 陷阱 3：忘记检查 isOpen()

**问题**：

- 网络波动时，连接可能已经断开，但你的 Map 里还没来得及移除

  此时直接调用 `sendMessage` 会抛出 `IOException`

-  **最佳实践**：发送前永远建议加上 `if (session.isOpen())` 判断，并用 `try-catch` 包裹发送逻辑



#### 📝 总结

- **原生 WebSocket** 让你像操作 Socket 一样操作 Web 实时通信
- 你需要自己实现 `WebSocketHandler` 来处理连接、消息和关闭事件
- **最大的痛点**：你需要自己手写 Map 来管理用户会话，且难以处理集群环境下的消息同步



### STOMP 协议开发⭐

- 在上一节的原生开发中，我们需要手动管理 Session 集合，极其繁琐

  而在 STOMP (Simple Text Oriented Messaging Protocol) 模式下，Spring 会接管这些底层细节，我们只需要关注业务逻辑

#### 1 核心配置

这是 STOMP 开发的第一步，也是最关键的一步。我们需要**创建一个配置类**，告诉 Spring 两件事：

1. **连接入口**：客户端通过哪个 URL 连上来（握手）
2. **路由规则**：消息连上来后，该怎么转发（是直接广播，还是给 Controller 处理）



##### 1.1 配置类代码详解

我们需要创建一个新的配置类，使用 `@EnableWebSocketMessageBroker` 注解

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.*;

@Configuration
@EnableWebSocketMessageBroker // ⭐ 关键注解：开启 STOMP 消息代理功能
public class WebSocketStompConfig implements WebSocketMessageBrokerConfigurer {

    /**
     * 🔧 方法 1：配置消息代理 (Message Broker)
     * 作用：定义消息在系统内部的流向（路由规则）
     */
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // A. 启用内置的 "内存消息代理" (Simple Broker)
        // 作用：定义 "广播" 通道的前缀
        // 逻辑：当客户端订阅 (SUBSCRIBE) 的路径以 /topic 或 /queue 开头时，
        //       Spring 会自动由 Broker 处理，不再经过 Controller，直接广播给所有订阅者
        // 规范：
        //   - /topic: 用于 "一对多" 广播 (如：全服公告)
        //   - /queue: 用于 "一对一" 私信 (如：点对点聊天)
        config.enableSimpleBroker("/topic", "/queue");

        // B. 设置 "应用目的地" 前缀 (Application Destination Prefix)
        // 作用：定义 "业务处理" 通道的前缀
        // 逻辑：当客户端发送 (SEND) 的消息路径以 /app 开头时，
        //       Spring 会把这个消息路由给 @Controller 中的 @MessageMapping 方法
        config.setApplicationDestinationPrefixes("/app");
    }

    /**
     * 🚪 方法 2：注册 STOMP 端点 (Endpoint)
     * 作用：定义客户端建立连接时的 "握手地址"
     */
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // 1. 添加端点：/ws-stomp
        //    客户端代码示例：let socket = new SockJS('/ws-stomp');
        registry.addEndpoint("/ws-stomp")
                
                // 2. 允许跨域设置
                //    注意：Spring Boot 2.4+ 建议使用 setAllowedOriginPatterns("*")
                .setAllowedOrigins("*")
                
                // 3. 开启 SockJS 支持 (强烈建议)
                //    作用：如果浏览器不支持 WebSocket，自动降级为 HTTP 轮询 (Polling)
                .withSockJS();
    }
}

```



##### 1.2 消息走向

以上一节的配置为例

**场景 1：客户端要发业务请求（如：发送聊天内容）**

- **路径**：`/app/chat`

- **流向**：客户端 `SEND` -> 匹配 `/app` 前缀 -> **进入 Spring Controller** (`@MessageMapping` 方法) -> 业务逻辑处理 -> (可选) 返回结果给 Broker

  > 这里的 Spring Controller 详见下一节



**场景 2：客户端要订阅消息（如：监听新公告）**

- **路径**：`/topic/news`
- **流向**：客户端 `SUBSCRIBE` -> 匹配 `/topic` 前缀 -> **进入 Message Broker** (内存队列) -> 注册订阅关系 -> 等待服务器推送



##### 1.3 🚫 常见误区与避坑

1. **端点 (Endpoint) vs 目的地 (Destination)**

   - `registerStompEndpoints` 配置的是 **“门”** (`/ws-stomp`)。客户端连接时只用这一个地址
   - `configureMessageBroker` 配置的是 **“房间号”** (`/topic`, `/app`)。连接成功后，发消息和订消息都用这些地址
   - **切记**：千万不要在 `new SockJS(...)` 里写 `/topic/...`

   

2. **为什么有了 `/topic` 还要 `/queue`？**

   - 其实在 `enableSimpleBroker` 看来，它俩没区别，都是内存里的队列
   - **约定俗成**：这是 STOMP 协议的语义规范。看到 `/topic` 就知道是广播，看到 `/queue` 就知道是私信。遵守这个规范能让代码更易读

   

3. **跨域问题**

   - 如果在浏览器控制台看到 `Access-Control-Allow-Origin` 错误，检查 `setAllowedOrigins("*")`
   - 如果项目集成了 Spring Security，还需要在 Security配置链中放行这个端点 (`/ws-stomp/**`)



#### 2 消息控制器 (Controller)

在 STOMP 开发中，`Controller` 是核心的消息处理中心。它的工作模式非常像 Spring MVC 处理 HTTP 请求，但底层机制完全不同



##### 2.1 核心注解详解

我们需要掌握三个最关键的注解，它们决定了消息的“来”与“去”

**1. `@Controller`**

- **定义**：标记在类上。声明该类是一个 Spring 组件，会被扫描并注册到 Spring 容器中

- **原理**：在 WebSocket 场景下，它负责接收由 `SimpAnnotationMethodMessageHandler` 分发的 STOMP 消息

- **关键区别**：

  - **在 HTTP 中**：`@Controller` 默认配合视图解析器（找 HTML 页面），返回数据需加 `@ResponseBody`

  - **在 STOMP 中**：

    - **不需要** `@ResponseBody`

      Spring 默认认为你的返回值就是要写入 WebSocket 消息帧（Frame）的数据负载（Payload），会自动进行 JSON 序列化

  - **建议**：虽然用 `@RestController` 也能跑通，但为了语义清晰（我们在处理消息，不是 REST API），官方强烈推荐使用 `@Controller`



**2. `@MessageMapping` (入站路由)**

- **定义**：标记在方法上。定义消息的 **接收路径**
- **工作流程**：当客户端发送 (`SEND`) 消息且目的地匹配该路径时：
  1. Spring 拦截消息
  2. 利用 Jackson 库将消息体（JSON）反序列化为 Java 对象
  3. 调用该方法，传入对象
- **路径规则**：实际触发路径 = **配置的 Application 前缀** (`/app`) + **注解路径** (`/chat`) = `/app/chat`



**3. `@SendTo` (出站广播)**

- **定义**：标记在方法上。定义消息的 **广播目的地**
- **工作流程**：当方法执行完毕并 `return` 一个对象时：
  1. Spring 获取返回值。
  2. 利用 Jackson 库将 Java 对象序列化为 JSON 字符串
  3. 将消息广播发送给 **所有** 订阅了该路径的客户端⭐



##### 2.2 代码实战

```java
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

@Controller
public class ChatController {

    /**
     * 接收并广播聊天消息
     * * 1. 入口 (收): 客户端发送到 /app/chat
     * (前缀 /app 来自配置类 config.setApplicationDestinationPrefixes)
     * * 2. 出口 (发): 广播给 /topic/messages
     * (凡是订阅了 client.subscribe('/topic/messages') 的客户端都会收到)
     */
    @MessageMapping("/chat")
    @SendTo("/topic/messages")
    public ChatMessage handleChatMessage(ChatMessage message) {
        // 业务处理：比如敏感词过滤、保存到数据库
        System.out.println("收到用户 " + message.getSender() + " 的消息: " + message.getContent());
        
        // 这个返回值会被自动序列化为 JSON，发送到 /topic/messages
        return message;
    }
    
    /**
     * 实体类 (POJO)
     * ⚠️ 警告：必须有 Getter/Setter 方法，否则 Jackson 序列化会失败
     */
    public static class ChatMessage {
        private String content;
        private String sender;
        
        // 省略构造器...
        
        public String getContent() { return content; }
        public void setContent(String content) { this.content = content; }
        public String getSender() { return sender; }
        public void setSender(String sender) { this.sender = sender; }
    }
}
```



##### 2.3 🚫 常见误区与避坑

1. **前端发送路径错误**

   - **错误**：前端直接发给 `/chat`

   - **正确**：

     - 必须加上配置的前缀，发给 `/app/chat`

       如果不加前缀，Spring 会认为这是直接发给 Broker 的（如果 Broker 没配置这个路径，消息就丢了），根本不会进 Controller

   

2. **Jackson 序列化空对象**

   - **现象**：后端没报错，但前端收到的消息是 `{}` (空 JSON)
   - **原因**：你的实体类（`ChatMessage`）没有 `public` 的 Getter 方法。Jackson 默认靠 Getter 读取属性值，没有 Getter 就读不到数据
   - **解决**：加 `@Data` (Lombok) 或手动写 Getter

   

3. **逻辑处理耗时**

   - `@MessageMapping` 的方法是在 Spring 的消息处理线程池中运行的

     如果这里面有极其耗时的操作（如复杂的数据库查询、长时间计算），会阻塞后续消息的处理。建议使用 `@Async` 或扔给线程池处理



#### 3 主动推送消息 (SimpMessagingTemplate)

在 `@Controller` 模式下，消息的流向是 **Request-Reply（请求-响应）** 模式

而在本节，我们将学习 **Fire-and-Forget（发送即忘）** 模式，即服务器在任意时间、任意地点主动向客户端推送数据



##### 3.1 核心工具：`SimpMessagingTemplate`

Spring 提供了一个极其强大的 Bean：**`SimpMessagingTemplate`**

- **定义**：它是 Spring Messaging 模块提供的通用消息发送模板
- **地位**：它在 WebSocket 中的地位，等同于 HTTP 开发中的 `RestTemplate` 或数据库开发中的 `JdbcTemplate`
- **特性**：
  1. **线程安全**：你可以在多线程环境下（如定时任务线程池）安全使用
  2. **随处注入**：它可以被注入到 Service、Controller、Component、Scheduler 等任何 Spring 管理的 Bean 中
  3. **自动转换**：它接受 Java 对象作为参数，并利用 Jackson 库自动将其序列化为 JSON



##### 3.2 核心方法详解 (API)

这个类里有很多方法，但作为开发者，你只需要死记硬背以下 **两个核心方法**，就能应对 99% 的场景

> 这两个方法是服务独单主动用来推送消息的



**1. 广播发送：`convertAndSend`**

这是最常用的方法，用于向某个“公有频道”推送消息，所有订阅了该频道的客户端都能收到

> `void convertAndSend(String destination, Object payload)`

```java
/**
 * @param destination 目的地 (String)。对应前端订阅的地址
 * @param payload     消息体 (Object)。在这个方法内部，它会被转成 JSON
 */
void convertAndSend(String destination, Object payload);

// ✅ 示例：
// 假设前端订阅了：client.subscribe('/topic/news', ...)
// 后端代码：
template.convertAndSend("/topic/news", new News("重磅新闻"));
```



**2. 点对点发送：`convertAndSendToUser`**

这是专门用于“私信”的方法。Spring 会自动把消息路由给指定用户的 Session

> `void convertAndSendToUser(String user, String destination, Object payload)`

```java
/**
 * @param user        目标用户 (String)。通常是 userId 或 username (取决于你的 Principal 怎么定义的)
 * @param destination 目的地 (String)。注意：这里只需要写后缀
 * @param payload     消息体 (Object)
 */
void convertAndSendToUser(String user, String destination, Object payload);

// ✅ 示例：
// 假设前端订阅了：client.subscribe('/user/queue/notify', ...)
// 后端代码 (发给用户 "1001")：
// 注意：中间的 "/user" 是 Spring 自动加的，这里不需要写
template.convertAndSendToUser("1001", "/queue/notify", "您的订单已发货");
```



**⚠️ 关键区别总结：**

| 方法                       | 目标受众            | 目的地写法 (后端) | 目的地写法 (前端订阅) |
| -------------------------- | ------------------- | ----------------- | --------------------- |
| **`convertAndSend`**       | **所有人** (广播)   | `/topic/abc`      | `/topic/abc`          |
| **`convertAndSendToUser`** | **特定用户** (私信) | `/queue/abc`      | **`/user`/queue/abc** |



##### 3.3 代码实战：构建一个通知服务

我们通常会封装一个 Service 来专门负责发消息，这样业务代码就不需要关心 WebSocket 的细节

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.simp.SimpMessagingTemplate;
import org.springframework.stereotype.Service;

import java.util.Date;

@Service
public class WebSocketNotificationService {

    // 1. 注入核心模板
    @Autowired
    private SimpMessagingTemplate messagingTemplate;

    /**
     * 场景 A：系统全员广播
     * 例如：停机维护通知、实时公告
     * * @param message 通知内容
     */
    public void sendGlobalNotification(String message) {
        // 构造消息体 (可以是普通 String，也可以是复杂对象)
        SystemNotification notification = new SystemNotification("系统公告", message, new Date());

        // 2. 核心方法：convertAndSend
        // 参数 1: 目的地 (Destination) -> 对应配置中的 /topic/...
        // 参数 2: 消息负载 (Payload) -> 会被转成 JSON
        messagingTemplate.convertAndSend("/topic/global-notify", notification);
        
        System.out.println("已发送全员广播: " + message);
    }

    // 简单的 DTO 对象
    public static class SystemNotification {
        public String title;
        public String content;
        public Date timestamp;
        
        public SystemNotification(String title, String content, Date timestamp) {
            this.title = title;
            this.content = content;
            this.timestamp = timestamp;
        }
    }
}
```



##### 3.4 常见触发场景

有了上面的 Service，你就可以在任何业务逻辑中触发推送了：

**场景 1：定时任务 (Scheduled Task)** 比如每隔 5 秒推送一次服务器监控数据

```java
@Component
public class ServerMonitorTask {

    @Autowired
    private WebSocketNotificationService notificationService;

    @Scheduled(fixedRate = 5000)
    public void pushMetrics() {
        // 模拟获取数据
        String metric = "CPU Load: " + Math.random() * 100 + "%";
        notificationService.sendGlobalNotification(metric);
    }
}
```



**场景 2：HTTP 接口触发 (Rest Controller)** 比如外部系统调用了一个回调接口，通知我们支付成功

```java
@RestController
public class PaymentCallbackController {

    @Autowired
    private WebSocketNotificationService notificationService;

    @PostMapping("/api/payment/callback")
    public String onPaymentSuccess(@RequestBody PaymentResult result) {
        // 业务逻辑...
        
        // 触发 WebSocket 推送
        notificationService.sendGlobalNotification("订单 " + result.getOrderId() + " 支付成功！");
        
        return "success";
    }
}
```



##### 3.5 🚫 常见误区与避坑

1. **不要手动拼接 JSON**

   - **错误**：`template.convertAndSend("/topic/x", "{\"name\":\"" + name + "\"}")`
   - **正确**：直接传对象 `template.convertAndSend("/topic/x", new User(name))`。Spring 会处理转义和序列化，手动拼接极其容易出错且不安全

2. **目的地前缀问题**

   - 在使用 `SimpMessagingTemplate` 时，发送的目的地必须是完整的路径（如 `/topic/news`）
   - **注意**：这里不需要加 `/app` 前缀。`/app` 是给 `@MessageMapping` 用的（入站），而这里是直接发给 Broker 的（出站）

3. **消息顺序与并发**

   - 虽然 `SimpMessagingTemplate` 是线程安全的，但 TCP 传输本身有网络延迟

     如果你在极短时间内连续调用两次发送（例如先发“状态A”，再发“状态B”），客户端收到顺序通常是有保证的（TCP特性），但在高并发多线程环境下，发出的顺序取决于线程调度的顺序，不一定严格按照代码书写顺序





### 用户认证与私信

在前面的章节中，我们实现的都是“广播”——一个人说话，所有人都能听到。但在真实业务中，我们通常需要“私信”：

- 用户 A 支付成功，只通知用户 A
- 客服 B 回复用户 C，只有用户 C 能看到

要实现这个功能，前提是服务器必须知道 **每一个 WebSocket 连接背后是谁**，需要把 **WebSocket 连接** 与 **具体用户 (User)** 绑定起来



#### 1. 核心概念：Principal (身份主体)

在 Spring STOMP 的标准体系下，如果要实现”发私信“，必须使用 Principal

在 Java 安全体系（包括 Spring Security）中，`Principal` 是代表“当前登录用户”的标准接口

- **在 HTTP 中**：`Principal` 通常由 Session 或 JWT 解析而来
- **在 WebSocket 中**：每一个 `WebSocketSession` 也可以关联一个 `Principal` 对象



**绑定的意义**： 

- 一旦连接关联了 `Principal`，Spring 就知道这个 Session 属于 "User1001"

  当你调用“给 User1001 发消息”的方法时，Spring 就能自动找到对应的 Session 并把消息推过去



**获取身份的两种路径**：

1. **自动绑定 (Spring Security)**：

   - 如果你的项目集成了 Spring Security，且用户已经登录（Session 或 Cookie），Spring 会自动把用户绑定到 WebSocket

     **不需要** 写任何额外代码

2. **手动绑定 (HandshakeInterceptor)**：

   - **这是 JWT 的主流方案**

     由于浏览器 WebSocket 不支持自定义 Header，我们无法直接传递 `Authorization: Bearer ...`
     因此，必须手动在 **握手阶段** 拦截 URL 中的 Token，解析用户，并告诉 Spring “这个人是谁”



#### 2. 场景一：手动实现身份绑定

> 适用于 JWT / 自定义 Token

在前后端分离的架构中，我们通常使用 JWT 或 Redis Token 来标识用户

但遇到的核心痛点是：**WebSocket 握手请求无法自定义 Header**。前端无法像调用 REST 接口那样发送 `Authorization: Bearer xxx`



**唯一的解决方案**： 

- 将 Token 放在 URL 参数中传递，例如：`ws://domain.com/ws-stomp?token=eyJhb...`

  然后后端在握手拦截器中读取该参数，验证身份，并绑定到 Session



##### a. 第一步：定义 Principal 类

Spring WebSocket 需要一个实现了 `java.security.Principal` 接口的对象来代表“当前用户”

这个对象很简单，只要能存一个唯一的 ID（比如 userId 或 username）即可

```java
import java.security.Principal;

public class StompPrincipal implements Principal {
    private final String name; // 用户唯一标识 (通常是 userId)

    public StompPrincipal(String name) {
        this.name = name;
    }

    @Override
    public String getName() {
        return this.name;
    }
}
```



##### b. 第二步：编写握手拦截器 (HandshakeInterceptor)

这是鉴权的核心逻辑所在。拦截器会在连接建立之前（Before Handshake）执行。 在这里，我们需要完成三件事：

1. 从 URL 解析出 Token
2. 验证 Token 是否合法（查 Redis 或解密 JWT）
3. 如果合法，将解析出的 UserID 暂时存入 `attributes` Map 中

```java
public class AuthHandshakeInterceptor implements HandshakeInterceptor {

    /**
     * 握手前调用
     * @return true=允许连接; false=拒绝连接
     */
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response,
                                   WebSocketHandler wsHandler, Map<String, Object> attributes) {
        
        // 1. 将 ServerHttpRequest 转换为 ServletServerHttpRequest 以便获取 HttpServletRequest
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) request;
            
            // 2. 从 URL 参数中获取 token
            // 对应前端代码: new WebSocket("ws://xxx/ws-stomp?token=xxx")
            String token = servletRequest.getServletRequest().getParameter("token");
            
            // 3. 执行 Token 校验逻辑
            if (token == null || token.isEmpty()) {
                System.out.println("握手失败: 未携带 Token");
                return false;
            }

            try {
                // --- 模拟 JWT 解析逻辑 ---
                // String userId = JwtUtil.getUserId(token);
                // 如果 Token 过期或非法，这里应抛出异常或返回 null
                String userId = "user_" + token; // 模拟解析结果
                
                // 4. ⭐ 关键步骤：将用户 ID 放入 attributes
                // 注意：这里还不能直接生成 Principal，因为 beforeHandshake 方法签名不支持返回 Principal
                // 我们先把 ID 存到 Map 里，传给下一步的 Handler 使用
                attributes.put("user_id", userId);
                
                System.out.println("握手成功，用户: " + userId);
                return true; 
            } catch (Exception e) {
                System.out.println("握手失败: Token 无效");
                return false;
            }
        }
        return false;
    }

    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response,
                               WebSocketHandler wsHandler, Exception exception) {
        // 握手后处理，通常留空
    }
}
```



##### c. 第三步：配置握手处理器 (HandshakeHandler)

拦截器只是把 ID 存到了 Map 里，Session 此时还是匿名的

我们需要配置 `DefaultHandshakeHandler`，重写 `determineUser` 方法，利用 Map 里的 ID 生成真正的 `Principal` 对象，完成最后的“身份证绑定”

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketStompConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws-stomp")
                .setAllowedOriginPatterns("*") // 允许跨域
                
                // 1. 注册我们写的拦截器 (负责校验 Token)
                .addInterceptors(new AuthHandshakeInterceptor())
                
                // 2. 自定义握手处理器 (负责生成 Principal)
                .setHandshakeHandler(new DefaultHandshakeHandler() {
                    @Override
                    protected Principal determineUser(ServerHttpRequest request, WebSocketHandler wsHandler, Map<String, Object> attributes) {
                        // 从 attributes 中取出拦截器存入的 user_id
                        String userId = (String) attributes.get("user_id");
                        
                        // 如果拦截器通过了但 ID 为空，说明逻辑有漏洞，返回 null (匿名)
                        if (userId == null) return null;
                        
                        // 生成 Principal 对象，Spring 内部会自动把它绑定到当前的 WebSocket Session
                        return new StompPrincipal(userId);
                    }
                })
                .withSockJS();
    }
    
    // configureMessageBroker 方法略...
}
```





#### 3. 场景二：发送“点对点”私信

完成了上一节的身份绑定后，Spring 现在已经知道每一个 WebSocket 连接背后是哪个 `user_id` 了

现在，我们终于可以实现精准的**“私信推送”**：只把消息发给特定的人，其他人收不到



##### **方式 A：在 Controller 中回复发送者**

**适用场景**： “请求-响应”模式。用户发了一条指令给服务器，服务器处理完后，只给 **这个用户** 回一条确认消息

**核心注解**：`@SendToUser`

```java
@Controller
public class PrivateChatController {

    /**
     * 接收用户消息，并回私信
     * 1. 入口: /app/private
     * 2. 出口: /user/queue/reply (注意：这是发给发送者的私有队列)
     * 3. 参数: Principal 会自动注入，代表当前发消息的人
     */
    @MessageMapping("/private")
    @SendToUser("/queue/reply") 
    public String handlePrivateMessage(Principal principal, String msg) {
        // principal.getName() 会自动获取我们在拦截器里绑定的 userId
        System.out.println("收到用户 " + principal.getName() + " 的私信: " + msg);
        
        return "服务器已收到你的秘密消息: " + msg;
    }
}
```

**前端订阅规则**： 客户端必须订阅以 `/user` 开头的特殊前缀： `client.subscribe('/user/queue/reply', ...)`



##### **方式 B：主动推送 (Active) ⭐ 最常用**

**适用场景**： “触发-通知”模式。用户没有给服务器发消息，但服务器的业务逻辑（如订单系统、后台审核）触发了事件，需要主动通知特定用户

**核心方法**：`messagingTemplate.convertAndSendToUser`

```java
@Service
public class PrivateNotificationService {

    @Autowired
    private SimpMessagingTemplate messagingTemplate;

    /**
     * 发送私信给指定用户
     * @param userId  目标用户的唯一标识 (要和 Principal.getName() 一致)
     * @param message 消息内容
     */
    public void sendPrivateNotification(String userId, String message) {
        // 核心方法: convertAndSendToUser
        // 参数 1: user (目标用户 ID)
        // 参数 2: destination (目的地后缀，不带 /user 前缀)
        // 参数 3: payload (消息体)
        messagingTemplate.convertAndSendToUser(
            userId, 
            "/queue/notifications", 
            "您有一条新通知: " + message
        );
        
        System.out.println("已向用户 " + userId + " 发送通知");
    }
}

```







#### 4 🚫 常见陷阱与避坑指南

##### 💥 陷阱 1：客户端订阅路径的混淆

这是新手最容易晕的地方

- **服务器发送代码**：`messagingTemplate.convertAndSendToUser("alice", "/queue/notify", msg)`
- **错误的客户端订阅**：`/user/alice/queue/notify` ❌
- **正确的客户端订阅**：`/user/queue/notify` ✅

**原理**： 

- 客户端只需要订阅 `/user/queue/notify`

  Spring 内部会自动把它转换为 `/user/{session-id}/queue/notify`，从而实现只把消息发给当前连接的这个会话

  客户端不需要（也不应该）知道自己的 UserID 在 WebSocket URL 里的具体形式



##### 💥 陷阱 2：Principal 为空

- **问题**：在 Controller 的方法参数里注入 `Principal principal`，结果是 `null`

- **原因**：你没有配置 Spring Security，也没有像 6.13 那样手动配置 `HandshakeInterceptor` 和 `DefaultHandshakeHandler`
- **解决**：WebSocket 是基于 TCP 的，它天然不知道“谁是谁”，必须通过拦截器在握手阶段手动赋值



##### 💥 陷阱 3：Token 放在 Header 还是 URL？

- **问题**：很多开发者想把 Token 放在 WebSocket 的 Header 里（比如 `Authorization: Bearer...`）

-  **现实**：**浏览器标准的 WebSocket API 不支持自定义 Header**

-  **解决**：

  1. **放在 URL 参数里**：`ws://xxx?token=abc`（最常用，虽然有 URL 泄露风险，但 WebSocket 建立后 URL 不会暴露在请求体外）

  2. **先建立连接，再发认证帧**：

     连接时不认证，连接建立后客户端发送的第一条 STOMP 消息带上 Token Header，服务器在 `ChannelInterceptor` 里拦截验证。这种方式更安全，但代码复杂度高很多



### Spring WebSocket 其它注解补充

很多同学在看别人代码时，会发现一些没见过的注解

#### 0. 最核心的注解



#### 1. 参数获取类注解

我们在 `@MessageMapping` 方法中，除了可以接收实体对象，还可以接收很多“元数据”

##### a. `@DestinationVariable` (路径参数)

- **对标 HTTP**：`@PathVariable`
- **作用**：从订阅或发送的**路径中**提取参数
- **场景**：聊天室 ID、特定的动态 Topic

```java
@Controller
public class RoomController {

    // 客户端发送到: /app/chat/room/101
    @MessageMapping("/chat/room/{roomId}")
    @SendTo("/topic/room/{roomId}")
    public ChatMessage speak(@DestinationVariable String roomId, ChatMessage message) {
        System.out.println("收到 " + roomId + " 房间的消息");
        return message;
    }
}
```



##### b. `@Header` (获取头部)

- **对标 HTTP**：`@RequestHeader`
- **作用**：获取 STOMP 帧头部的特定字段（如 session-id, native-headers）。
- **场景**：获取客户端传来的特殊 Token（如果没在拦截器处理），或者获取 `simpSessionId`。

```java
@MessageMapping("/test")
public void test(@Header("simpSessionId") String sessionId, 
                 @Header(value = "my-token", required = false) String token) {
    System.out.println("当前 Session ID: " + sessionId);
}
```



##### c. `@Payload` (消息体)

- **对标 HTTP**：`@RequestBody`
- **作用**：明确指定哪个参数是“消息体”。
- **注意**：通常可以**省略不写**。Spring 默认会把非 Header 的对象参数当作 Payload。但如果你想配合 `@Valid` 做参数校验，就必须写上。

```java
@MessageMapping("/register")
public void register(@Payload @Valid UserDto user) {
    // 如果 user 字段校验失败，会抛出异常
}
```



##### d. `@Headers` (所有头部)

- **作用**：一次性拿到所有的头信息，返回一个 `Map`。
- **场景**：调试时打印所有头信息。

```java
@MessageMapping("/debug")
public void debug(@Headers Map<String, Object> headers) {
    System.out.println("所有头信息: " + headers);
}
```



#### 2. 异常处理类注解

##### a. `@MessageExceptionHandler`

- **对标 HTTP**：`@ExceptionHandler`
- **作用**：当 `@MessageMapping` 方法抛出异常时，这个方法会被触发
- **场景**：统一处理业务异常，并给用户返回错误提示

```java
@Controller
public class GlobalWsExceptionHandler {

    @MessageMapping("/trade")
    public void trade() {
        throw new RuntimeException("余额不足");
    }

    /**
     * 专门捕获当前 Controller 里的异常
     * 结合 @SendToUser，只把错误信息发给该用户
     */
    @MessageExceptionHandler
    @SendToUser("/queue/errors")
    public String handleException(Throwable exception) {
        return "操作失败: " + exception.getMessage();
    }
}
```





#### 3. 订阅回调类

##### a. `@SubscribeMapping`

这是一个比较特殊的注解，容易和 `@MessageMapping` 搞混

- **触发时机**：当客户端发起 `SUBSCRIBE` (订阅) 请求时触发
- **作用**：**“订阅即加载”**
- **场景**：用户刚点开股票页面（发起订阅），你想立刻给他返回当前的最新股价，而不需要等下一次变动
- **区别**：
  - `@MessageMapping` 处理 `SEND` 指令
  - `@SubscribeMapping` 处理 `SUBSCRIBE` 指令

```java
@Controller
public class StockController {

    // 客户端订阅: /app/stocks/init
    // 注意：这里不需要 @SendTo，返回值直接回传给订阅者
    @SubscribeMapping("/stocks/init")
    public List<Stock> initStocks() {
        // 用户一订阅，立马给他返回这一刻的初始数据
        return stockService.getCurrentPrices();
    }
}
```





#### 📝 对照表：HTTP vs WebSocket

| HTTP 注解 (Web)     | WebSocket 注解 (STOMP)            | 作用                   |
| ------------------- | --------------------------------- | ---------------------- |
| `@RequestMapping`   | **`@MessageMapping`**             | 路由映射 (收消息)      |
| `@ResponseBody`     | **`@SendTo`** / **`@SendToUser`** | 返回数据 (发消息)      |
| `@RequestBody`      | **`@Payload`**                    | 获取消息体 (Body)      |
| `@PathVariable`     | **`@DestinationVariable`**        | 获取路径参数           |
| `@RequestHeader`    | **`@Header`**                     | 获取头部信息           |
| `@ExceptionHandler` | **`@MessageExceptionHandler`**    | 捕获异常               |
| (无直接对标)        | **`@SubscribeMapping`**           | 订阅时立即返回初始数据 |

