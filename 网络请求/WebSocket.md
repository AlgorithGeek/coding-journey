# WebSocket 全面技术指南

## 目录

- [1. WebSocket 概述](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#1-websocket-概述)
- [2. WebSocket 协议详解](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#2-websocket-协议详解)
- [3. WebSocket vs HTTP](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#3-websocket-vs-http)
- [4. WebSocket 生命周期](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#4-websocket-生命周期)
- [5. Spring WebSocket 集成](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#5-spring-websocket-集成)
- [6. STOMP 协议](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#6-stomp-协议)
- [7. 实战案例](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#7-实战案例)
- [8. 安全性考虑](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#8-安全性考虑)
- [9. 性能优化](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#9-性能优化)
- [10. 常见问题与解决方案](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c#10-常见问题与解决方案)

------

## 1. WebSocket 概述

### 1.1 什么是WebSocket

WebSocket是一种在单个TCP连接上进行**全双工通信**的协议，它为客户端和服务器之间的数据交换提供了一种实时、双向的通信渠道。

**核心特点：**

- **全双工通信**：客户端和服务器可以同时发送和接收数据
- **持久连接**：一旦建立连接，会保持开放状态直到任一方关闭
- **低延迟**：没有HTTP请求/响应的开销
- **轻量级**：数据帧格式简单，协议开销小

### 1.2 WebSocket的诞生背景

传统的HTTP协议是**请求-响应**模式，存在以下限制：

- 单向请求：只能由客户端发起请求
- 无状态：每次请求都是独立的
- 开销大：每次请求都需要完整的HTTP头部

**传统的实时通信解决方案：**

```markdown
1. 轮询（Polling）
   - 客户端定期发送HTTP请求
   - 缺点：延迟高、服务器压力大、带宽浪费

2. 长轮询（Long Polling）
   - 服务器保持请求开放直到有数据
   - 缺点：连接管理复杂、仍有HTTP开销

3. SSE（Server-Sent Events）
   - 服务器向客户端单向推送
   - 缺点：只支持单向通信
```

### 1.3 WebSocket应用场景

- **即时通讯**：聊天应用、在线客服
- **实时协作**：协同编辑、白板应用
- **金融交易**：股票行情、交易推送
- **游戏开发**：多人在线游戏、实时对战
- **物联网**：设备监控、实时数据采集
- **直播互动**：弹幕、点赞、实时评论
- **系统监控**：服务器监控、日志实时推送

------

## 2. WebSocket 协议详解

### 2.1 WebSocket URL

WebSocket使用不同的URL方案：

```
ws://example.com:80/path    # 非加密连接
wss://example.com:443/path   # 加密连接（WebSocket over TLS）
```

### 2.2 握手过程

WebSocket连接建立通过HTTP升级机制实现：

**客户端请求：**

```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Sec-WebSocket-Protocol: chat, superchat
Origin: http://example.com
```

**服务器响应：**

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

**关键头部字段说明：**

- `Upgrade: websocket`：请求协议升级
- `Sec-WebSocket-Key`：客户端生成的随机密钥
- `Sec-WebSocket-Accept`：服务器基于Key计算的响应值
- `Sec-WebSocket-Protocol`：子协议协商
- `Sec-WebSocket-Version`：WebSocket协议版本

### 2.3 数据帧格式

WebSocket数据传输的基本单位是**帧（Frame）**：

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

**帧类型（Opcode）：**

- `0x0`：延续帧
- `0x1`：文本帧
- `0x2`：二进制帧
- `0x8`：关闭连接
- `0x9`：Ping帧
- `0xA`：Pong帧

### 2.4 心跳机制

WebSocket使用Ping/Pong帧维持连接：

```java
// 服务端发送Ping
websocket.ping(ByteBuffer.wrap("ping".getBytes()));

// 客户端自动响应Pong
// 或手动处理
@OnMessage
public void onPong(PongMessage pong, Session session) {
    // 处理Pong消息
}
```

------

## 3. WebSocket vs HTTP

### 3.1 详细对比

| 特性         | HTTP                | WebSocket              |
| ------------ | ------------------- | ---------------------- |
| **通信模式** | 请求-响应           | 全双工                 |
| **连接状态** | 无状态              | 有状态                 |
| **方向性**   | 客户端发起          | 双向                   |
| **协议标识** | http:// 或 https:// | ws:// 或 wss://        |
| **默认端口** | 80/443              | 80/443（共用）         |
| **数据格式** | 文本（HTTP头+正文） | 二进制帧               |
| **开销**     | 较大（HTTP头部）    | 较小（帧头部2-14字节） |
| **实时性**   | 低（需要轮询）      | 高                     |
| **跨域**     | 受限（需要CORS）    | 支持                   |
| **代理支持** | 普遍支持            | 需要特殊配置           |

### 3.2 适用场景对比

**HTTP适合：**

- RESTful API
- 文档下载
- 表单提交
- 传统Web页面

**WebSocket适合：**

- 实时聊天
- 实时通知
- 协同编辑
- 流媒体
- 实时数据推送

------

## 4. WebSocket 生命周期

### 4.1 连接建立（Opening Handshake）

```javascript
// 浏览器端
const ws = new WebSocket('ws://localhost:8080/websocket');

ws.onopen = function(event) {
    console.log('连接已建立');
    ws.send('Hello Server!');
};
```

### 4.2 数据传输（Data Transfer）

```javascript
// 发送文本
ws.send("Hello WebSocket!");

// 发送JSON
ws.send(JSON.stringify({
    type: 'message',
    content: 'Hello',
    timestamp: Date.now()
}));

// 发送二进制
const buffer = new ArrayBuffer(8);
ws.send(buffer);

// 接收消息
ws.onmessage = function(event) {
    if (event.data instanceof Blob) {
        // 处理二进制数据
    } else {
        // 处理文本数据
        console.log('收到消息:', event.data);
    }
};
```

### 4.3 连接关闭（Closing Handshake）

```javascript
// 主动关闭
ws.close(1000, "Normal closure");

// 监听关闭事件
ws.onclose = function(event) {
    console.log('连接已关闭:', event.code, event.reason);
};

// 关闭状态码
// 1000: 正常关闭
// 1001: 端点离开
// 1002: 协议错误
// 1003: 数据类型错误
// 1006: 异常关闭
// 1007: 数据不一致
// 1008: 策略违规
// 1009: 消息过大
// 1011: 服务器错误
```

### 4.4 错误处理

```javascript
ws.onerror = function(error) {
    console.error('WebSocket错误:', error);
    // 实现重连逻辑
};
```

------

## 5. Spring WebSocket 集成

### 5.1 依赖配置

**Maven依赖：**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>

<!-- 如果使用STOMP -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-messaging</artifactId>
</dependency>
```

### 5.2 基础WebSocket配置

**配置类：**

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new MyWebSocketHandler(), "/ws")
                .setAllowedOrigins("*")  // 配置跨域
                .addInterceptors(new HttpSessionHandshakeInterceptor());
    }
}
```

**处理器实现：**

```java
@Component
public class MyWebSocketHandler extends TextWebSocketHandler {
    
    private final Map<String, WebSocketSession> sessions = new ConcurrentHashMap<>();
    
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        sessions.put(session.getId(), session);
        System.out.println("连接建立: " + session.getId());
    }
    
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) 
            throws Exception {
        String payload = message.getPayload();
        System.out.println("收到消息: " + payload);
        
        // 回复消息
        session.sendMessage(new TextMessage("Echo: " + payload));
        
        // 广播给所有连接
        broadcast(new TextMessage("Broadcast: " + payload));
    }
    
    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) 
            throws Exception {
        System.err.println("传输错误: " + exception.getMessage());
        session.close();
    }
    
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) 
            throws Exception {
        sessions.remove(session.getId());
        System.out.println("连接关闭: " + session.getId());
    }
    
    private void broadcast(TextMessage message) throws IOException {
        for (WebSocketSession session : sessions.values()) {
            if (session.isOpen()) {
                session.sendMessage(message);
            }
        }
    }
}
```

### 5.3 使用@ServerEndpoint（Java WebSocket API）

```java
@ServerEndpoint(value = "/websocket/{userId}")
@Component
public class WebSocketServer {
    
    private static Map<String, Session> sessionMap = new ConcurrentHashMap<>();
    private String userId;
    
    @OnOpen
    public void onOpen(Session session, @PathParam("userId") String userId) {
        this.userId = userId;
        sessionMap.put(userId, session);
        System.out.println("用户连接: " + userId);
    }
    
    @OnMessage
    public void onMessage(String message, Session session) {
        System.out.println("收到消息: " + message);
        // 处理消息
        sendMessageToUser(userId, "收到: " + message);
    }
    
    @OnClose
    public void onClose() {
        sessionMap.remove(userId);
        System.out.println("用户断开: " + userId);
    }
    
    @OnError
    public void onError(Session session, Throwable error) {
        System.err.println("错误: " + error.getMessage());
    }
    
    // 发送消息给指定用户
    public static void sendMessageToUser(String userId, String message) {
        Session session = sessionMap.get(userId);
        if (session != null && session.isOpen()) {
            try {
                session.getBasicRemote().sendText(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    
    // 广播消息
    public static void broadcast(String message) {
        sessionMap.values().forEach(session -> {
            try {
                session.getBasicRemote().sendText(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }
}
```

**配置ServerEndpointExporter：**

```java
@Configuration
public class WebSocketConfiguration {
    
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

------

## 6. STOMP 协议

### 6.1 STOMP简介

STOMP（Simple Text Oriented Messaging Protocol）是一个简单的文本消息协议，为WebSocket提供了更高级的消息语义。

**STOMP帧格式：**

```
COMMAND
header1:value1
header2:value2

Body^@
```

**常用命令：**

- CONNECT：建立连接
- SEND：发送消息
- SUBSCRIBE：订阅
- UNSUBSCRIBE：取消订阅
- ACK：确认
- DISCONNECT：断开连接

### 6.2 Spring STOMP配置

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketStompConfig implements WebSocketMessageBrokerConfigurer {
    
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        // 启用简单的内存消息代理
        config.enableSimpleBroker("/topic", "/queue");
        // 设置应用目标前缀
        config.setApplicationDestinationPrefixes("/app");
        // 设置用户目标前缀
        config.setUserDestinationPrefix("/user");
    }
    
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws-stomp")
                .setAllowedOrigins("*")
                .withSockJS();  // 启用SockJS后备选项
    }
    
    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registry) {
        registry.setMessageSizeLimit(128 * 1024);  // 消息大小限制
        registry.setSendTimeLimit(20 * 1000);      // 发送超时
        registry.setSendBufferSizeLimit(512 * 1024); // 发送缓冲区大小
    }
}
```

### 6.3 STOMP消息处理

```java
@Controller
@RequiredArgsConstructor
public class WebSocketController {
    
    private final SimpMessagingTemplate messagingTemplate;
    
    // 处理客户端发送的消息
    @MessageMapping("/chat/send")
    @SendTo("/topic/messages")
    public ChatMessage sendMessage(@Payload ChatMessage message, 
                                  SimpMessageHeaderAccessor headerAccessor) {
        // 可以从header中获取session属性
        String username = (String) headerAccessor.getSessionAttributes().get("username");
        message.setSender(username);
        message.setTimestamp(LocalDateTime.now());
        return message;
    }
    
    // 发送给特定用户
    @MessageMapping("/chat/private")
    public void sendPrivateMessage(@Payload PrivateMessage message, Principal principal) {
        message.setSender(principal.getName());
        messagingTemplate.convertAndSendToUser(
            message.getReceiver(), 
            "/queue/private", 
            message
        );
    }
    
    // 处理订阅事件
    @EventListener
    public void handleSubscribeEvent(SessionSubscribeEvent event) {
        StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(event.getMessage());
        String destination = headerAccessor.getDestination();
        System.out.println("用户订阅: " + destination);
    }
    
    // 处理连接事件
    @EventListener
    public void handleWebSocketConnectListener(SessionConnectedEvent event) {
        System.out.println("新的WebSocket连接建立");
    }
    
    // 处理断开事件
    @EventListener
    public void handleWebSocketDisconnectListener(SessionDisconnectEvent event) {
        StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(event.getMessage());
        String username = (String) headerAccessor.getSessionAttributes().get("username");
        if (username != null) {
            // 广播用户离开消息
            ChatMessage chatMessage = new ChatMessage();
            chatMessage.setType(ChatMessage.MessageType.LEAVE);
            chatMessage.setSender(username);
            messagingTemplate.convertAndSend("/topic/messages", chatMessage);
        }
    }
}
```

### 6.4 前端STOMP客户端

```javascript
// 引入SockJS和STOMP
// <script src="/webjars/sockjs-client/sockjs.min.js"></script>
// <script src="/webjars/stomp-websocket/stomp.min.js"></script>

let stompClient = null;

function connect() {
    const socket = new SockJS('/ws-stomp');
    stompClient = Stomp.over(socket);
    
    stompClient.connect({}, function(frame) {
        console.log('Connected: ' + frame);
        
        // 订阅公共消息
        stompClient.subscribe('/topic/messages', function(message) {
            const chatMessage = JSON.parse(message.body);
            displayMessage(chatMessage);
        });
        
        // 订阅私人消息
        stompClient.subscribe('/user/queue/private', function(message) {
            const privateMessage = JSON.parse(message.body);
            displayPrivateMessage(privateMessage);
        });
    });
}

function sendMessage() {
    const message = {
        content: document.getElementById('message').value,
        type: 'CHAT'
    };
    
    stompClient.send("/app/chat/send", {}, JSON.stringify(message));
}

function sendPrivateMessage(receiver, content) {
    const message = {
        receiver: receiver,
        content: content
    };
    
    stompClient.send("/app/chat/private", {}, JSON.stringify(message));
}

function disconnect() {
    if (stompClient !== null) {
        stompClient.disconnect();
    }
    console.log("Disconnected");
}
```

------

## 7. 实战案例

### 7.1 在线聊天室

**后端核心代码：**

```java
@Component
@RequiredArgsConstructor
public class ChatRoomWebSocketHandler extends TextWebSocketHandler {
    
    private final ObjectMapper objectMapper;
    private final Map<String, ChatUser> users = new ConcurrentHashMap<>();
    
    @Data
    public static class ChatUser {
        private String id;
        private String username;
        private WebSocketSession session;
        private LocalDateTime joinTime;
    }
    
    @Data
    public static class ChatMessage {
        private String type;  // JOIN, CHAT, LEAVE
        private String userId;
        private String username;
        private String content;
        private LocalDateTime timestamp;
    }
    
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        String userId = session.getId();
        ChatUser user = new ChatUser();
        user.setId(userId);
        user.setSession(session);
        user.setJoinTime(LocalDateTime.now());
        
        // 从session attributes获取用户名
        user.setUsername((String) session.getAttributes()
            .getOrDefault("username", "User" + userId.substring(0, 6)));
        
        users.put(userId, user);
        
        // 发送欢迎消息
        ChatMessage welcomeMsg = new ChatMessage();
        welcomeMsg.setType("SYSTEM");
        welcomeMsg.setContent("欢迎加入聊天室！当前在线: " + users.size());
        session.sendMessage(new TextMessage(objectMapper.writeValueAsString(welcomeMsg)));
        
        // 广播用户加入
        ChatMessage joinMsg = new ChatMessage();
        joinMsg.setType("JOIN");
        joinMsg.setUsername(user.getUsername());
        joinMsg.setTimestamp(LocalDateTime.now());
        broadcast(joinMsg, userId);
    }
    
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) 
            throws Exception {
        ChatMessage chatMessage = objectMapper.readValue(message.getPayload(), ChatMessage.class);
        ChatUser user = users.get(session.getId());
        
        if ("CHAT".equals(chatMessage.getType())) {
            chatMessage.setUserId(user.getId());
            chatMessage.setUsername(user.getUsername());
            chatMessage.setTimestamp(LocalDateTime.now());
            
            // 广播消息
            broadcast(chatMessage, null);
        }
    }
    
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) 
            throws Exception {
        String userId = session.getId();
        ChatUser user = users.remove(userId);
        
        if (user != null) {
            // 广播用户离开
            ChatMessage leaveMsg = new ChatMessage();
            leaveMsg.setType("LEAVE");
            leaveMsg.setUsername(user.getUsername());
            leaveMsg.setTimestamp(LocalDateTime.now());
            broadcast(leaveMsg, userId);
        }
    }
    
    private void broadcast(ChatMessage message, String excludeUserId) throws IOException {
        String messageJson = objectMapper.writeValueAsString(message);
        TextMessage textMessage = new TextMessage(messageJson);
        
        for (ChatUser user : users.values()) {
            if (!user.getId().equals(excludeUserId) && user.getSession().isOpen()) {
                user.getSession().sendMessage(textMessage);
            }
        }
    }
}
```

### 7.2 实时通知系统

```java
@Service
@RequiredArgsConstructor
public class NotificationService {
    
    private final SimpMessagingTemplate messagingTemplate;
    private final NotificationRepository notificationRepository;
    
    // 发送通知给特定用户
    public void sendNotification(String userId, NotificationDTO notification) {
        // 保存到数据库
        Notification entity = new Notification();
        entity.setUserId(userId);
        entity.setTitle(notification.getTitle());
        entity.setContent(notification.getContent());
        entity.setType(notification.getType());
        entity.setRead(false);
        entity.setCreatedAt(LocalDateTime.now());
        notificationRepository.save(entity);
        
        // 通过WebSocket推送
        messagingTemplate.convertAndSendToUser(
            userId,
            "/queue/notifications",
            notification
        );
    }
    
    // 批量发送通知
    public void broadcastNotification(NotificationDTO notification) {
        messagingTemplate.convertAndSend("/topic/broadcast", notification);
    }
    
    // 发送给特定角色的用户
    public void sendToRole(String role, NotificationDTO notification) {
        List<String> userIds = getUserIdsByRole(role);
        userIds.forEach(userId -> sendNotification(userId, notification));
    }
    
    // 标记已读
    @MessageMapping("/notification/read/{notificationId}")
    public void markAsRead(@DestinationVariable Long notificationId, Principal principal) {
        notificationRepository.markAsRead(notificationId, principal.getName());
    }
}
```

### 7.3 实时数据监控

```java
@Component
@RequiredArgsConstructor
public class MetricsWebSocketHandler {
    
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
    private final Set<WebSocketSession> sessions = ConcurrentHashMap.newKeySet();
    
    @PostConstruct
    public void startMetricsBroadcast() {
        scheduler.scheduleAtFixedRate(this::broadcastMetrics, 0, 5, TimeUnit.SECONDS);
    }
    
    private void broadcastMetrics() {
        if (sessions.isEmpty()) return;
        
        SystemMetrics metrics = collectSystemMetrics();
        String metricsJson = convertToJson(metrics);
        
        sessions.parallelStream().forEach(session -> {
            try {
                if (session.isOpen()) {
                    session.sendMessage(new TextMessage(metricsJson));
                }
            } catch (Exception e) {
                sessions.remove(session);
            }
        });
    }
    
    private SystemMetrics collectSystemMetrics() {
        SystemMetrics metrics = new SystemMetrics();
        
        // CPU使用率
        metrics.setCpuUsage(getCpuUsage());
        
        // 内存使用
        Runtime runtime = Runtime.getRuntime();
        metrics.setMemoryUsed(runtime.totalMemory() - runtime.freeMemory());
        metrics.setMemoryTotal(runtime.totalMemory());
        
        // JVM线程数
        metrics.setThreadCount(Thread.activeCount());
        
        // 自定义业务指标
        metrics.setActiveUsers(getActiveUserCount());
        metrics.setRequestsPerSecond(getRequestRate());
        
        return metrics;
    }
}
```

------

## 8. 安全性考虑

### 8.1 认证与授权

**JWT Token认证：**

```java
@Component
public class WebSocketAuthInterceptor implements HandshakeInterceptor {
    
    @Autowired
    private JwtTokenProvider tokenProvider;
    
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response,
                                  WebSocketHandler wsHandler, Map<String, Object> attributes) 
                                  throws Exception {
        // 从查询参数获取token
        String token = extractTokenFromRequest(request);
        
        if (token != null && tokenProvider.validateToken(token)) {
            String username = tokenProvider.getUsernameFromToken(token);
            attributes.put("username", username);
            attributes.put("authorities", tokenProvider.getAuthorities(token));
            return true;
        }
        
        response.setStatusCode(HttpStatus.UNAUTHORIZED);
        return false;
    }
    
    private String extractTokenFromRequest(ServerHttpRequest request) {
        String query = request.getURI().getQuery();
        if (query != null) {
            String[] params = query.split("&");
            for (String param : params) {
                String[] keyValue = param.split("=");
                if ("token".equals(keyValue[0]) && keyValue.length == 2) {
                    return keyValue[1];
                }
            }
        }
        return null;
    }
}
```

**Spring Security集成：**

```java
@Configuration
@EnableWebSecurity
public class WebSocketSecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/ws/**").authenticated()
                .anyRequest().permitAll()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );
        
        return http.build();
    }
}
```

### 8.2 防止跨站WebSocket劫持（CSWSH）

```java
@Component
public class OriginCheckInterceptor implements HandshakeInterceptor {
    
    private final List<String> allowedOrigins = Arrays.asList(
        "http://localhost:3000",
        "https://example.com"
    );
    
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response,
                                  WebSocketHandler wsHandler, Map<String, Object> attributes) 
                                  throws Exception {
        HttpHeaders headers = request.getHeaders();
        String origin = headers.getOrigin();
        
        if (origin == null || !allowedOrigins.contains(origin)) {
            response.setStatusCode(HttpStatus.FORBIDDEN);
            return false;
        }
        
        return true;
    }
}
```

### 8.3 消息验证与过滤

```java
@Component
public class MessageValidator {
    
    private static final int MAX_MESSAGE_SIZE = 10 * 1024; // 10KB
    private static final Pattern XSS_PATTERN = Pattern.compile(
        "<script>|</script>|<iframe>|javascript:", 
        Pattern.CASE_INSENSITIVE
    );
    
    public boolean validateMessage(String message) {
        // 检查消息大小
        if (message.length() > MAX_MESSAGE_SIZE) {
            return false;
        }
        
        // XSS防护
        if (XSS_PATTERN.matcher(message).find()) {
            return false;
        }
        
        // SQL注入防护（如果消息会存入数据库）
        if (containsSqlInjection(message)) {
            return false;
        }
        
        return true;
    }
    
    public String sanitizeMessage(String message) {
        // HTML转义
        return StringEscapeUtils.escapeHtml4(message);
    }
}
```

### 8.4 速率限制

```java
@Component
public class WebSocketRateLimiter {
    
    private final Map<String, RateLimiter> limiters = new ConcurrentHashMap<>();
    
    public boolean tryAcquire(String userId) {
        RateLimiter limiter = limiters.computeIfAbsent(userId, 
            k -> RateLimiter.create(10.0)); // 每秒10条消息
        
        return limiter.tryAcquire();
    }
    
    @Scheduled(fixedDelay = 300000) // 5分钟清理一次
    public void cleanup() {
        limiters.entrySet().removeIf(entry -> 
            entry.getValue().getRate() == 0);
    }
}
```

------

## 9. 性能优化

### 9.1 连接池管理

```java
@Configuration
public class WebSocketPoolConfig {
    
    @Bean
    public TaskScheduler heartbeatScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(1);
        scheduler.setThreadNamePrefix("wss-heartbeat-");
        scheduler.initialize();
        return scheduler;
    }
    
    @Bean
    public TaskExecutor webSocketTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("websocket-");
        executor.initialize();
        return executor;
    }
}
```

### 9.2 消息批处理

```java
@Component
public class MessageBatcher {
    
    private final Map<String, Queue<Message>> userMessageQueues = new ConcurrentHashMap<>();
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
    
    @PostConstruct
    public void init() {
        // 每100ms批量发送一次
        scheduler.scheduleAtFixedRate(this::flushAllQueues, 0, 100, TimeUnit.MILLISECONDS);
    }
    
    public void queueMessage(String userId, Message message) {
        userMessageQueues.computeIfAbsent(userId, k -> new ConcurrentLinkedQueue<>())
                        .offer(message);
    }
    
    private void flushAllQueues() {
        userMessageQueues.forEach((userId, queue) -> {
            if (!queue.isEmpty()) {
                List<Message> batch = new ArrayList<>();
                Message msg;
                int count = 0;
                
                while ((msg = queue.poll()) != null && count++ < 50) {
                    batch.add(msg);
                }
                
                if (!batch.isEmpty()) {
                    sendBatch(userId, batch);
                }
            }
        });
    }
    
    private void sendBatch(String userId, List<Message> messages) {
        BatchMessage batchMessage = new BatchMessage(messages);
        // 发送批量消息
        sendToUser(userId, batchMessage);
    }
}
```

### 9.3 消息压缩

```java
@Component
public class MessageCompressor {
    
    public byte[] compress(String message) throws IOException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try (GZIPOutputStream gzipOut = new GZIPOutputStream(baos)) {
            gzipOut.write(message.getBytes(StandardCharsets.UTF_8));
        }
        return baos.toByteArray();
    }
    
    public String decompress(byte[] compressed) throws IOException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try (GZIPInputStream gzipIn = new GZIPInputStream(
                new ByteArrayInputStream(compressed))) {
            byte[] buffer = new byte[1024];
            int len;
            while ((len = gzipIn.read(buffer)) != -1) {
                baos.write(buffer, 0, len);
            }
        }
        return baos.toString(StandardCharsets.UTF_8);
    }
    
    public boolean shouldCompress(String message) {
        // 只压缩大于1KB的消息
        return message.length() > 1024;
    }
}
```

### 9.4 负载均衡与集群

```java
@Configuration
@EnableRedisWebSocketMessageBroker
public class WebSocketClusterConfig {
    
    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer(
            RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        return container;
    }
    
    @Bean
    public MessagePublisher redisPublisher(RedisTemplate<String, Object> redisTemplate) {
        return new RedisMessagePublisher(redisTemplate);
    }
    
    @Component
    public class RedisMessagePublisher implements MessagePublisher {
        
        private final RedisTemplate<String, Object> redisTemplate;
        private final String CHANNEL_PREFIX = "websocket:";
        
        public RedisMessagePublisher(RedisTemplate<String, Object> redisTemplate) {
            this.redisTemplate = redisTemplate;
        }
        
        @Override
        public void publish(String channel, Object message) {
            redisTemplate.convertAndSend(CHANNEL_PREFIX + channel, message);
        }
    }
}
```

------

## 10. 常见问题与解决方案

### 10.1 连接断开问题

**问题：** WebSocket连接频繁断开

**解决方案：**

```java
// 1. 实现心跳机制
@Component
public class HeartbeatHandler {
    
    @Scheduled(fixedDelay = 30000) // 每30秒发送一次心跳
    public void sendHeartbeat() {
        sessions.forEach((id, session) -> {
            if (session.isOpen()) {
                try {
                    session.sendMessage(new PingMessage());
                } catch (IOException e) {
                    // 处理异常
                }
            }
        });
    }
}

// 2. 配置合理的超时时间
@Override
public void configureWebSocketTransport(WebSocketTransportRegistration registry) {
    registry.setSendTimeLimit(60 * 1000);     // 60秒
    registry.setSendBufferSizeLimit(512 * 1024);
}

// 3. 实现自动重连（前端）
class WebSocketClient {
    constructor(url) {
        this.url = url;
        this.reconnectInterval = 5000;
        this.maxReconnectAttempts = 10;
        this.reconnectAttempts = 0;
    }
    
    connect() {
        this.ws = new WebSocket(this.url);
        
        this.ws.onclose = (event) => {
            if (!event.wasClean) {
                this.reconnect();
            }
        };
        
        this.ws.onerror = (error) => {
            console.error('WebSocket error:', error);
            this.reconnect();
        };
    }
    
    reconnect() {
        if (this.reconnectAttempts < this.maxReconnectAttempts) {
            this.reconnectAttempts++;
            console.log(`Reconnecting... Attempt ${this.reconnectAttempts}`);
            setTimeout(() => this.connect(), this.reconnectInterval);
        }
    }
}
```

### 10.2 跨域问题

**问题：** WebSocket跨域请求被拒绝

**解决方案：**

```java
@Override
public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
    registry.addHandler(myHandler(), "/ws")
            .setAllowedOrigins("*")  // 开发环境
            // 生产环境指定具体域名
            .setAllowedOrigins("https://example.com", "https://app.example.com")
            .withSockJS();
}

// 或使用动态配置
@Override
public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
    registry.addHandler(myHandler(), "/ws")
            .setAllowedOriginPatterns("https://*.example.com")
            .withSockJS();
}
```

### 10.3 消息顺序问题

**问题：** 消息到达顺序不一致

**解决方案：**

```java
@Component
public class OrderedMessageHandler {
    
    private final Map<String, MessageSequencer> sequencers = new ConcurrentHashMap<>();
    
    public static class MessageSequencer {
        private long expectedSequence = 1;
        private final TreeMap<Long, Message> pendingMessages = new TreeMap<>();
        
        public synchronized List<Message> addMessage(Message message) {
            List<Message> toProcess = new ArrayList<>();
            
            if (message.getSequence() == expectedSequence) {
                toProcess.add(message);
                expectedSequence++;
                
                // 处理缓存的后续消息
                while (pendingMessages.containsKey(expectedSequence)) {
                    toProcess.add(pendingMessages.remove(expectedSequence));
                    expectedSequence++;
                }
            } else if (message.getSequence() > expectedSequence) {
                // 缓存乱序消息
                pendingMessages.put(message.getSequence(), message);
            }
            // 忽略重复或过期消息
            
            return toProcess;
        }
    }
    
    public void handleMessage(String sessionId, Message message) {
        MessageSequencer sequencer = sequencers.computeIfAbsent(
            sessionId, k -> new MessageSequencer()
        );
        
        List<Message> messagesToProcess = sequencer.addMessage(message);
        messagesToProcess.forEach(this::processMessage);
    }
}
```

### 10.4 内存泄漏问题

**问题：** 长时间运行后内存占用过高

**解决方案：**

```java
@Component
public class SessionManager {
    
    private final Map<String, SessionInfo> sessions = new ConcurrentHashMap<>();
    private final ScheduledExecutorService cleanupExecutor = 
        Executors.newSingleThreadScheduledExecutor();
    
    @PostConstruct
    public void init() {
        // 定期清理无效session
        cleanupExecutor.scheduleAtFixedRate(
            this::cleanup, 0, 30, TimeUnit.MINUTES
        );
    }
    
    private void cleanup() {
        long now = System.currentTimeMillis();
        sessions.entrySet().removeIf(entry -> {
            SessionInfo info = entry.getValue();
            // 清理超过1小时未活动的session
            return !info.getSession().isOpen() || 
                   (now - info.getLastActivityTime()) > 3600000;
        });
    }
    
    @PreDestroy
    public void destroy() {
        cleanupExecutor.shutdown();
        sessions.clear();
    }
}
```

### 10.5 大量连接处理

**问题：** 无法处理大量并发WebSocket连接

**解决方案：**

```java
// 1. 优化服务器配置
@Configuration
public class WebSocketServerConfig {
    
    @Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = 
            new ServletServerContainerFactoryBean();
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
        container.setMaxSessionIdleTimeout(300000L);
        container.setAsyncSendTimeout(60000L);
        return container;
    }
}

// 2. 使用Netty实现
@Configuration
@EnableWebSocketMessageBroker
public class NettyWebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
        registration.setMessageSizeLimit(128 * 1024);
        registration.setSendBufferSizeLimit(512 * 1024);
        registration.setSendTimeLimit(60 * 1000);
    }
    
    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(10);
        scheduler.setThreadNamePrefix("websocket-");
        scheduler.initialize();
        return scheduler;
    }
}

// 3. 系统层面优化（Linux）
// 修改 /etc/sysctl.conf
// net.core.somaxconn = 65535
// net.ipv4.tcp_max_syn_backlog = 65535
// fs.file-max = 1000000
```

------

## 11. 测试与调试

### 11.1 单元测试

```java
@SpringBootTest
@AutoConfigureMockMvc
public class WebSocketTest {
    
    @Test
    public void testWebSocketConnection() throws Exception {
        WebSocketClient client = new StandardWebSocketClient();
        WebSocketSession session = client.doHandshake(
            new TextWebSocketHandler() {
                @Override
                public void handleTextMessage(WebSocketSession session, 
                                            TextMessage message) {
                    System.out.println("Received: " + message.getPayload());
                }
            },
            "ws://localhost:8080/ws"
        ).get(5, TimeUnit.SECONDS);
        
        assertTrue(session.isOpen());
        session.sendMessage(new TextMessage("Hello"));
        Thread.sleep(1000);
        session.close();
    }
    
    @Test
    public void testStompConnection() throws Exception {
        StompSessionHandler sessionHandler = new DefaultStompSessionHandler();
        
        WebSocketStompClient stompClient = new WebSocketStompClient(
            new StandardWebSocketClient()
        );
        stompClient.setMessageConverter(new MappingJackson2MessageConverter());
        
        StompSession session = stompClient.connect(
            "ws://localhost:8080/ws-stomp", sessionHandler
        ).get(5, TimeUnit.SECONDS);
        
        session.subscribe("/topic/messages", new StompFrameHandler() {
            @Override
            public Type getPayloadType(StompHeaders headers) {
                return ChatMessage.class;
            }
            
            @Override
            public void handleFrame(StompHeaders headers, Object payload) {
                ChatMessage message = (ChatMessage) payload;
                assertNotNull(message);
            }
        });
        
        session.send("/app/chat/send", new ChatMessage("Test", "Hello"));
        Thread.sleep(1000);
        session.disconnect();
    }
}
```

### 11.2 调试工具

**Chrome DevTools:**

```javascript
// 在Console中调试WebSocket
const ws = new WebSocket('ws://localhost:8080/ws');

ws.onopen = () => console.log('Connected');
ws.onmessage = (e) => console.log('Message:', e.data);
ws.onerror = (e) => console.error('Error:', e);
ws.onclose = () => console.log('Closed');

// 发送消息
ws.send('Hello Server');

// 查看连接状态
console.log('ReadyState:', ws.readyState);
// 0: CONNECTING, 1: OPEN, 2: CLOSING, 3: CLOSED
```

**使用wscat工具:**

```bash
# 安装
npm install -g wscat

# 连接
wscat -c ws://localhost:8080/ws

# 发送消息
> {"type":"chat","message":"Hello"}

# 带认证连接
wscat -c "ws://localhost:8080/ws?token=your-jwt-token"
```

------

## 12. 最佳实践总结

### 12.1 设计原则

1. **优雅降级**：提供HTTP轮询作为WebSocket的后备方案
2. **消息格式标准化**：使用统一的消息格式（如JSON）
3. **错误处理**：完善的异常处理和错误恢复机制
4. **安全第一**：始终验证和清理用户输入
5. **监控和日志**：记录关键事件和性能指标

### 12.2 代码规范

```java
// ✅ 好的实践
@Component
@Slf4j
public class WebSocketHandler extends TextWebSocketHandler {
    
    private static final int MAX_MESSAGE_SIZE = 64 * 1024;
    private static final int SESSION_TIMEOUT = 30 * 60 * 1000;
    
    @Override
    protected void handleTextMessage(WebSocketSession session, 
                                   TextMessage message) throws Exception {
        try {
            // 验证消息
            if (message.getPayloadLength() > MAX_MESSAGE_SIZE) {
                session.close(CloseStatus.NOT_ACCEPTABLE);
                return;
            }
            
            // 处理消息
            processMessage(session, message);
            
        } catch (Exception e) {
            log.error("Error handling message", e);
            sendErrorMessage(session, "Processing error");
        }
    }
}

// ❌ 避免的做法
public class BadWebSocketHandler extends TextWebSocketHandler {
    
    protected void handleTextMessage(WebSocketSession session, 
                                   TextMessage message) {
        // 没有异常处理
        // 没有消息验证
        // 没有日志记录
        broadcast(message.getPayload());
    }
}
```

### 12.3 部署建议

1. **使用反向代理**（Nginx配置）：

```nginx
location /ws {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    
    # 超时设置
    proxy_connect_timeout 7d;
    proxy_send_timeout 7d;
    proxy_read_timeout 7d;
}
```

1. **容器化部署**（Docker）：

```dockerfile
FROM openjdk:17-jdk-slim
EXPOSE 8080
COPY target/websocket-app.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar", 
            "--server.port=8080",
            "--spring.profiles.active=prod"]
```

1. **性能调优参数**：

```yaml
server:
  tomcat:
    max-connections: 10000
    max-threads: 200
    accept-count: 100
  compression:
    enabled: true
    mime-types: application/json,text/html,text/xml,text/plain

spring:
  websocket:
    servlet:
      max-text-message-size: 64KB
      max-binary-message-size: 64KB
      max-session-idle-timeout: 30m
```

------

## 13. 参考资源

### 官方文档

- [RFC 6455 - The WebSocket Protocol](https://tools.ietf.org/html/rfc6455)
- [Spring WebSocket Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket)
- [MDN WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

### 相关库

- [Socket.IO](https://socket.io/) - 实时应用框架
- [SockJS](https://github.com/sockjs/sockjs-client) - WebSocket后备方案
- [STOMP.js](https://stomp-js.github.io/stomp-websocket/) - STOMP客户端
- [Netty](https://netty.io/) - 高性能网络框架

### 工具

- [wscat](https://github.com/websockets/wscat) - WebSocket命令行客户端
- [Chrome WebSocket Frame Inspector](https://claude.ai/chat/992407d1-7f8e-40fe-a3b6-a4f2f909ae0c) - Chrome调试工具
- [Wireshark](https://www.wireshark.org/) - 网络协议分析

------

## 总结

WebSocket作为实现实时双向通信的重要协议，在现代Web应用中扮演着关键角色。通过本笔记，你应该掌握了：

1. **基础概念**：理解WebSocket的工作原理和适用场景
2. **协议细节**：掌握握手过程、数据帧格式等底层机制
3. **Spring集成**：熟练使用Spring WebSocket和STOMP
4. **实战应用**：能够开发聊天室、通知系统等实时应用
5. **优化技巧**：了解性能优化、安全防护的最佳实践

WebSocket技术仍在不断发展，建议持续关注新的标准和实践，在实际项目中根据具体需求选择合适的实现方案。