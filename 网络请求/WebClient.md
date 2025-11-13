# SpringBoot WebClient 全面学习笔记

## 一、WebClient 概述

### 1.1 什么是 WebClient

WebClient 是 Spring WebFlux 模块提供的一个非阻塞、响应式的 HTTP 客户端,用于执行 HTTP 请求。它是 RestTemplate 的现代化替代品,支持同步、异步和流式场景。

**核心特点:**

- 非阻塞式 I/O,基于 Reactor 项目
- 支持响应式编程(Reactive Programming)
- 函数式流式 API 设计
- 支持同步和异步调用
- 更高效的资源利用
- 支持流式数据传输

### 1.2 WebClient vs RestTemplate

| 特性     | RestTemplate         | WebClient                  |
| -------- | -------------------- | -------------------------- |
| 编程模型 | 同步阻塞             | 异步非阻塞/响应式          |
| API 风格 | 传统命令式           | 函数式流式                 |
| 性能     | 每个请求占用一个线程 | 少量线程处理大量请求       |
| 状态     | 维护模式(不再更新)   | 主动维护和推荐             |
| 适用场景 | 简单同步调用         | 高并发、微服务、响应式系统 |

### 1.3 应用场景

- 微服务之间的 HTTP 通信
- 调用第三方 REST API
- 需要高并发处理的场景
- 响应式 Web 应用
- 需要流式数据处理
- 对性能有较高要求的应用

------

## 二、环境配置

### 2.1 Maven 依赖

```xml
<!-- Spring WebFlux (包含 WebClient) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

<!-- 如果只需要 WebClient 而不需要 WebFlux 服务端功能 -->
<!-- 在 Spring MVC 项目中也可以使用 -->
```

### 2.2 Gradle 依赖

```gradle
implementation 'org.springframework.boot:spring-boot-starter-webflux'
```

**注意:** 即使你的项目是基于 Spring MVC(spring-boot-starter-web),也可以添加 webflux 依赖来使用 WebClient,两者可以共存。

------

## 三、WebClient 创建和配置

### 3.1 基本创建方式

#### 方式一: 快速创建(默认配置)

```java
WebClient webClient = WebClient.create();
```

#### 方式二: 指定 Base URL

```java
WebClient webClient = WebClient.create("https://api.example.com");
```

#### 方式三: 使用 Builder(推荐)

```java
WebClient webClient = WebClient.builder()
    .baseUrl("https://api.example.com")
    .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
    .defaultHeader(HttpHeaders.USER_AGENT, "MyApp/1.0")
    .build();
```

### 3.2 详细配置 Builder

```java
@Configuration
public class WebClientConfig {
    
    @Bean
    public WebClient webClient() {
        return WebClient.builder()
            // 1. 基础 URL
            .baseUrl("https://api.example.com")
            
            // 2. 默认请求头
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .defaultHeader(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE)
            .defaultHeader("Custom-Header", "value")
            
            // 3. 默认 Cookie
            .defaultCookie("sessionId", "abc123")
            
            // 4. 默认 URI 变量
            .defaultUriVariables(Collections.singletonMap("version", "v1"))
            
            // 5. 过滤器(拦截器)
            .filter(logRequest())
            .filter(logResponse())
            
            // 6. 自定义 HttpClient (超时、连接池等)
            .clientConnector(new ReactorClientHttpConnector(
                HttpClient.create()
                    .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
                    .doOnConnected(conn -> 
                        conn.addHandlerLast(new ReadTimeoutHandler(5))
                            .addHandlerLast(new WriteTimeoutHandler(5))
                    )
                    .responseTimeout(Duration.ofSeconds(5))
            ))
            
            // 7. 编解码器配置
            .codecs(configurer -> {
                configurer.defaultCodecs().maxInMemorySize(2 * 1024 * 1024); // 2MB
            })
            
            .build();
    }
    
    // 请求日志过滤器
    private ExchangeFilterFunction logRequest() {
        return ExchangeFilterFunction.ofRequestProcessor(clientRequest -> {
            log.info("Request: {} {}", clientRequest.method(), clientRequest.url());
            clientRequest.headers().forEach((name, values) -> 
                values.forEach(value -> log.info("{}={}", name, value))
            );
            return Mono.just(clientRequest);
        });
    }
    
    // 响应日志过滤器
    private ExchangeFilterFunction logResponse() {
        return ExchangeFilterFunction.ofResponseProcessor(clientResponse -> {
            log.info("Response Status: {}", clientResponse.statusCode());
            return Mono.just(clientResponse);
        });
    }
}
```

### 3.3 连接超时配置详解

```java
import io.netty.channel.ChannelOption;
import io.netty.handler.timeout.ReadTimeoutHandler;
import io.netty.handler.timeout.WriteTimeoutHandler;
import reactor.netty.http.client.HttpClient;

@Bean
public WebClient webClientWithTimeout() {
    HttpClient httpClient = HttpClient.create()
        // 连接超时
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000)
        
        // 响应超时
        .responseTimeout(Duration.ofSeconds(5))
        
        // 读写超时
        .doOnConnected(conn -> 
            conn.addHandlerLast(new ReadTimeoutHandler(5, TimeUnit.SECONDS))
                .addHandlerLast(new WriteTimeoutHandler(5, TimeUnit.SECONDS))
        );
    
    return WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
}
```

### 3.4 连接池配置

```java
import reactor.netty.resources.ConnectionProvider;

@Bean
public WebClient webClientWithConnectionPool() {
    ConnectionProvider provider = ConnectionProvider.builder("custom")
        .maxConnections(500)                    // 最大连接数
        .maxIdleTime(Duration.ofSeconds(20))    // 最大空闲时间
        .maxLifeTime(Duration.ofSeconds(60))    // 连接最大生命周期
        .pendingAcquireTimeout(Duration.ofSeconds(60)) // 获取连接超时
        .evictInBackground(Duration.ofSeconds(120))    // 后台清理间隔
        .build();
    
    HttpClient httpClient = HttpClient.create(provider);
    
    return WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
}
```

------

## 四、HTTP 请求方法详解

### 4.1 GET 请求

#### 基础 GET 请求

```java
// 无参数
String result = webClient.get()
    .uri("/users")
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 带路径参数
String result = webClient.get()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 带查询参数
String result = webClient.get()
    .uri(uriBuilder -> uriBuilder
        .path("/users")
        .queryParam("page", 1)
        .queryParam("size", 10)
        .queryParam("sort", "name")
        .build())
    .retrieve()
    .bodyToMono(String.class)
    .block();
```

#### 获取对象

```java
// 单个对象
User user = webClient.get()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(User.class)
    .block();

// 列表
List<User> users = webClient.get()
    .uri("/users")
    .retrieve()
    .bodyToFlux(User.class)
    .collectList()
    .block();

// 泛型类型
ParameterizedTypeReference<List<User>> typeRef = 
    new ParameterizedTypeReference<List<User>>() {};
    
List<User> users = webClient.get()
    .uri("/users")
    .retrieve()
    .bodyToMono(typeRef)
    .block();
```

### 4.2 POST 请求

#### 发送 JSON

```java
// 发送对象
User newUser = new User("张三", "zhangsan@example.com");

User createdUser = webClient.post()
    .uri("/users")
    .contentType(MediaType.APPLICATION_JSON)
    .bodyValue(newUser)  // 直接传递对象
    .retrieve()
    .bodyToMono(User.class)
    .block();

// 使用 Mono
Mono<User> userMono = Mono.just(newUser);

User createdUser = webClient.post()
    .uri("/users")
    .contentType(MediaType.APPLICATION_JSON)
    .body(userMono, User.class)
    .retrieve()
    .bodyToMono(User.class)
    .block();
```

#### 发送表单

```java
MultiValueMap<String, String> formData = new LinkedMultiValueMap<>();
formData.add("username", "zhangsan");
formData.add("password", "123456");

String result = webClient.post()
    .uri("/login")
    .contentType(MediaType.APPLICATION_FORM_URLENCODED)
    .bodyValue(formData)
    .retrieve()
    .bodyToMono(String.class)
    .block();
```

#### 文件上传

```java
// 单文件上传
MultipartBodyBuilder builder = new MultipartBodyBuilder();
builder.part("file", new FileSystemResource("/path/to/file.pdf"));
builder.part("description", "文件描述");

String result = webClient.post()
    .uri("/upload")
    .contentType(MediaType.MULTIPART_FORM_DATA)
    .bodyValue(builder.build())
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 多文件上传
MultipartBodyBuilder builder = new MultipartBodyBuilder();
builder.part("files", new FileSystemResource("/path/to/file1.pdf"));
builder.part("files", new FileSystemResource("/path/to/file2.pdf"));
builder.part("userId", "123");

String result = webClient.post()
    .uri("/upload/multiple")
    .contentType(MediaType.MULTIPART_FORM_DATA)
    .bodyValue(builder.build())
    .retrieve()
    .bodyToMono(String.class)
    .block();
```

### 4.3 PUT 请求

```java
User updateUser = new User("李四", "lisi@example.com");

User updatedUser = webClient.put()
    .uri("/users/{id}", 123)
    .contentType(MediaType.APPLICATION_JSON)
    .bodyValue(updateUser)
    .retrieve()
    .bodyToMono(User.class)
    .block();
```

### 4.4 DELETE 请求

```java
// 删除并获取响应
String result = webClient.delete()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 只关心状态码
webClient.delete()
    .uri("/users/{id}", 123)
    .retrieve()
    .toBodilessEntity()
    .block();
```

### 4.5 PATCH 请求

```java
Map<String, Object> updates = new HashMap<>();
updates.put("email", "newemail@example.com");

User patchedUser = webClient.patch()
    .uri("/users/{id}", 123)
    .contentType(MediaType.APPLICATION_JSON)
    .bodyValue(updates)
    .retrieve()
    .bodyToMono(User.class)
    .block();
```

------

## 五、请求配置详解

### 5.1 设置请求头

```java
String result = webClient.get()
    .uri("/api/data")
    // 单个请求头
    .header(HttpHeaders.AUTHORIZATION, "Bearer " + token)
    .header("Custom-Header", "value")
    // 多个请求头
    .headers(headers -> {
        headers.set(HttpHeaders.AUTHORIZATION, "Bearer " + token);
        headers.set("X-Request-ID", UUID.randomUUID().toString());
        headers.add("Accept-Language", "zh-CN");
        headers.add("Accept-Language", "en-US");
    })
    .retrieve()
    .bodyToMono(String.class)
    .block();
```

### 5.2 设置 Cookie

```java
String result = webClient.get()
    .uri("/api/data")
    .cookie("sessionId", "abc123")
    .cookies(cookies -> {
        cookies.add("token", "xyz789");
        cookies.add("preference", "dark-mode");
    })
    .retrieve()
    .bodyToMono(String.class)
    .block();
```

### 5.3 URI 构建

```java
// 方式一: 简单路径变量
webClient.get()
    .uri("/users/{id}/orders/{orderId}", 123, 456)
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 方式二: Map 传递路径变量
Map<String, Object> uriVariables = new HashMap<>();
uriVariables.put("userId", 123);
uriVariables.put("orderId", 456);

webClient.get()
    .uri("/users/{userId}/orders/{orderId}", uriVariables)
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 方式三: UriBuilder(最灵活)
webClient.get()
    .uri(uriBuilder -> uriBuilder
        .path("/users/{userId}/orders")
        .queryParam("status", "pending")
        .queryParam("page", 1)
        .queryParam("size", 10)
        .build(123))
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 方式四: 复杂 URI 构建
webClient.get()
    .uri(uriBuilder -> uriBuilder
        .scheme("https")
        .host("api.example.com")
        .port(8080)
        .path("/api/v1/users")
        .queryParam("name", "张三")
        .queryParam("tags", "tag1", "tag2")
        .build())
    .retrieve()
    .bodyToMono(String.class)
    .block();
```

### 5.4 Accept 和 ContentType

```java
// 接受 JSON 响应
webClient.get()
    .uri("/api/data")
    .accept(MediaType.APPLICATION_JSON)
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 发送 JSON,接受 XML
webClient.post()
    .uri("/api/data")
    .contentType(MediaType.APPLICATION_JSON)
    .accept(MediaType.APPLICATION_XML)
    .bodyValue(data)
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 多种 Accept 类型
webClient.get()
    .uri("/api/data")
    .accept(MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML)
    .retrieve()
    .bodyToMono(String.class)
    .block();
```

### 5.5 请求属性

```java
webClient.get()
    .uri("/api/data")
    .attribute("requestId", UUID.randomUUID())
    .attributes(attrs -> {
        attrs.put("startTime", System.currentTimeMillis());
        attrs.put("userId", 123);
    })
    .retrieve()
    .bodyToMono(String.class)
    .block();
```

------

## 六、响应处理

### 6.1 retrieve() vs exchange()

#### retrieve() - 简化的响应处理(推荐)

```java
// 只关心响应体
String body = webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .block();

// 自动处理 4xx 和 5xx 错误
```

#### exchange() - 完整的响应访问(需要手动处理资源)

```java
// 可以访问完整的响应信息
ClientResponse response = webClient.get()
    .uri("/api/data")
    .exchange()
    .block();

// 必须手动释放资源
String body = response.bodyToMono(String.class).block();

// ⚠️ 注意: exchange() 已弃用,推荐使用 exchangeToMono/exchangeToFlux
```

#### exchangeToMono/exchangeToFlux - exchange() 的替代品

```java
String result = webClient.get()
    .uri("/api/data")
    .exchangeToMono(response -> {
        if (response.statusCode().is2xxSuccessful()) {
            return response.bodyToMono(String.class);
        } else {
            return response.createException().flatMap(Mono::error);
        }
    })
    .block();
```

### 6.2 获取响应体

```java
// Mono - 单个对象
Mono<User> userMono = webClient.get()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(User.class);

User user = userMono.block(); // 阻塞获取

// Flux - 多个对象流
Flux<User> usersFlux = webClient.get()
    .uri("/users")
    .retrieve()
    .bodyToFlux(User.class);

List<User> users = usersFlux.collectList().block();

// String
String text = webClient.get()
    .uri("/api/text")
    .retrieve()
    .bodyToMono(String.class)
    .block();

// byte[]
byte[] bytes = webClient.get()
    .uri("/api/file")
    .retrieve()
    .bodyToMono(byte[].class)
    .block();
```

### 6.3 获取响应头和状态码

```java
ResponseEntity<User> responseEntity = webClient.get()
    .uri("/users/{id}", 123)
    .retrieve()
    .toEntity(User.class)
    .block();

// 状态码
HttpStatus statusCode = responseEntity.getStatusCode();
int statusCodeValue = responseEntity.getStatusCodeValue();

// 响应头
HttpHeaders headers = responseEntity.getHeaders();
String contentType = headers.getContentType().toString();
List<String> customHeaders = headers.get("Custom-Header");

// 响应体
User user = responseEntity.getBody();

// 使用 exchangeToMono 获取更详细的响应信息
webClient.get()
    .uri("/users/{id}", 123)
    .exchangeToMono(response -> {
        HttpStatus status = response.statusCode();
        HttpHeaders headers = response.headers().asHttpHeaders();
        
        // 根据状态码处理
        if (status.is2xxSuccessful()) {
            return response.bodyToMono(User.class);
        } else {
            return Mono.error(new RuntimeException("Request failed"));
        }
    })
    .block();
```

### 6.4 空响应处理

```java
// 响应无 body
Mono<Void> voidMono = webClient.delete()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(Void.class);

voidMono.block();

// 获取 ResponseEntity(可以检查状态码)
ResponseEntity<Void> response = webClient.delete()
    .uri("/users/{id}", 123)
    .retrieve()
    .toBodilessEntity()
    .block();

if (response.getStatusCode().is2xxSuccessful()) {
    System.out.println("删除成功");
}
```

------

## 七、错误处理

### 7.1 基础错误处理

```java
try {
    String result = webClient.get()
        .uri("/api/data")
        .retrieve()
        .bodyToMono(String.class)
        .block();
} catch (WebClientResponseException e) {
    // 4xx 和 5xx 错误
    HttpStatus statusCode = e.getStatusCode();
    String statusText = e.getStatusText();
    String responseBody = e.getResponseBodyAsString();
    
    log.error("请求失败: {} {}", statusCode, statusText);
    log.error("响应体: {}", responseBody);
}
```

### 7.2 onStatus() - 自定义状态码处理

```java
String result = webClient.get()
    .uri("/api/data")
    .retrieve()
    // 处理 4xx 错误
    .onStatus(HttpStatus::is4xxClientError, response -> {
        return response.bodyToMono(String.class)
            .flatMap(body -> Mono.error(
                new CustomClientException("客户端错误: " + body)
            ));
    })
    // 处理 5xx 错误
    .onStatus(HttpStatus::is5xxServerError, response -> {
        return Mono.error(
            new CustomServerException("服务器错误: " + response.statusCode())
        );
    })
    // 处理特定状态码
    .onStatus(status -> status.value() == 404, response -> {
        return Mono.error(new ResourceNotFoundException("资源未找到"));
    })
    .bodyToMono(String.class)
    .block();
```

### 7.3 onErrorResume() - 错误恢复

```java
// 错误时返回默认值
String result = webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .onErrorResume(error -> {
        log.error("请求失败,返回默认值", error);
        return Mono.just("默认值");
    })
    .block();

// 根据错误类型处理
String result = webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .onErrorResume(WebClientResponseException.class, error -> {
        if (error.getStatusCode() == HttpStatus.NOT_FOUND) {
            return Mono.just("未找到资源");
        }
        return Mono.error(error);
    })
    .block();
```

### 7.4 doOnError() - 错误日志记录

```java
String result = webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .doOnError(error -> {
        log.error("请求失败", error);
        // 发送告警、记录监控等
    })
    .block();
```

### 7.5 retry() - 重试机制

```java
// 简单重试
String result = webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .retry(3) // 失败后重试 3 次
    .block();

// 条件重试
String result = webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .retryWhen(Retry.backoff(3, Duration.ofSeconds(2))
        .maxBackoff(Duration.ofSeconds(10))
        .filter(throwable -> throwable instanceof WebClientResponseException)
        .onRetryExhaustedThrow((retryBackoffSpec, retrySignal) -> {
            throw new RuntimeException("重试次数已用尽");
        })
    )
    .block();
```

### 7.6 timeout() - 超时处理

```java
String result = webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .timeout(Duration.ofSeconds(5)) // 5秒超时
    .onErrorResume(TimeoutException.class, e -> {
        log.error("请求超时");
        return Mono.just("请求超时");
    })
    .block();
```

### 7.7 综合错误处理示例

```java
public Mono<User> getUserWithErrorHandling(Long userId) {
    return webClient.get()
        .uri("/users/{id}", userId)
        .retrieve()
        .onStatus(HttpStatus::is4xxClientError, response -> 
            response.bodyToMono(ErrorResponse.class)
                .flatMap(errorBody -> {
                    if (response.statusCode() == HttpStatus.NOT_FOUND) {
                        return Mono.error(new UserNotFoundException(
                            "用户不存在: " + userId
                        ));
                    }
                    return Mono.error(new ClientException(
                        "客户端错误: " + errorBody.getMessage()
                    ));
                })
        )
        .onStatus(HttpStatus::is5xxServerError, response ->
            Mono.error(new ServerException("服务器错误"))
        )
        .bodyToMono(User.class)
        .timeout(Duration.ofSeconds(5))
        .retryWhen(Retry.backoff(3, Duration.ofSeconds(1))
            .filter(throwable -> !(throwable instanceof UserNotFoundException))
        )
        .doOnError(error -> 
            log.error("获取用户失败 userId={}", userId, error)
        )
        .onErrorResume(TimeoutException.class, e -> {
            log.error("请求超时 userId={}", userId);
            return Mono.error(new TimeoutException("获取用户信息超时"));
        });
}
```

------

## 八、响应式编程

### 8.1 Mono 操作

```java
// 基本操作
Mono<User> userMono = webClient.get()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(User.class);

// 订阅(异步执行)
userMono.subscribe(
    user -> System.out.println("获取用户: " + user.getName()),
    error -> System.err.println("错误: " + error.getMessage()),
    () -> System.out.println("完成")
);

// 转换
Mono<String> nameMono = userMono.map(User::getName);

// 链式调用
Mono<OrderDTO> orderDTOMono = webClient.get()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(User.class)
    .flatMap(user -> 
        webClient.get()
            .uri("/users/{id}/orders", user.getId())
            .retrieve()
            .bodyToMono(Order.class)
    )
    .map(order -> new OrderDTO(order));

// 默认值
Mono<User> userWithDefault = userMono
    .defaultIfEmpty(new User("默认用户", "default@example.com"));

// 过滤
Mono<User> filteredUser = userMono
    .filter(user -> user.getAge() > 18);
```

### 8.2 Flux 操作

```java
// 基本操作
Flux<User> usersFlux = webClient.get()
    .uri("/users")
    .retrieve()
    .bodyToFlux(User.class);

// 订阅
usersFlux.subscribe(
    user -> System.out.println("用户: " + user.getName()),
    error -> System.err.println("错误: " + error.getMessage()),
    () -> System.out.println("所有用户处理完成")
);

// 转换
Flux<String> namesFlux = usersFlux.map(User::getName);

// 过滤
Flux<User> adultsFlux = usersFlux.filter(user -> user.getAge() >= 18);

// 收集为 List
Mono<List<User>> userListMono = usersFlux.collectList();
List<User> users = userListMono.block();

// 计数
Mono<Long> countMono = usersFlux.count();

// 限制数量
Flux<User> first10Users = usersFlux.take(10);

// 跳过
Flux<User> skipFirst10 = usersFlux.skip(10);

// 去重
Flux<User> distinctUsers = usersFlux.distinct();

// 排序
Flux<User> sortedUsers = usersFlux.sort((u1, u2) -> 
    u1.getName().compareTo(u2.getName())
);
```

### 8.3 并发请求

```java
// 并行执行多个请求
Mono<User> userMono = webClient.get()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(User.class);

Mono<List<Order>> ordersMono = webClient.get()
    .uri("/orders?userId={userId}", 123)
    .retrieve()
    .bodyToFlux(Order.class)
    .collectList();

Mono<Account> accountMono = webClient.get()
    .uri("/accounts/{userId}", 123)
    .retrieve()
    .bodyToMono(Account.class);

// zip - 组合多个 Mono
Mono<UserProfile> profileMono = Mono.zip(userMono, ordersMono, accountMono)
    .map(tuple -> {
        User user = tuple.getT1();
        List<Order> orders = tuple.getT2();
        Account account = tuple.getT3();
        return new UserProfile(user, orders, account);
    });

UserProfile profile = profileMono.block();

// zipWith - 组合两个 Mono
Mono<UserWithOrders> result = userMono.zipWith(ordersMono)
    .map(tuple -> new UserWithOrders(tuple.getT1(), tuple.getT2()));
```

### 8.4 顺序执行

```java
// flatMap - 串行执行
Mono<OrderDetail> orderDetailMono = webClient.get()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(User.class)
    .flatMap(user -> 
        webClient.get()
            .uri("/orders?userId={userId}", user.getId())
            .retrieve()
            .bodyToMono(Order.class)
    )
    .flatMap(order -> 
        webClient.get()
            .uri("/orders/{id}/details", order.getId())
            .retrieve()
            .bodyToMono(OrderDetail.class)
    );

// then - 忽略前一个结果
Mono<String> result = webClient.post()
    .uri("/users")
    .bodyValue(newUser)
    .retrieve()
    .bodyToMono(User.class)
    .then(webClient.post()
        .uri("/notifications")
        .bodyValue(notification)
        .retrieve()
        .bodyToMono(String.class)
    );
```

### 8.5 错误处理和重试

```java
Mono<User> userMono = webClient.get()
    .uri("/users/{id}", 123)
    .retrieve()
    .bodyToMono(User.class)
    .doOnSuccess(user -> log.info("成功获取用户: {}", user.getName()))
    .doOnError(error -> log.error("获取用户失败", error))
    .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)))
    .onErrorResume(error -> {
        log.error("重试失败,返回默认用户", error);
        return Mono.just(new User("默认", "default@example.com"));
    });
```

------

## 九、过滤器(Filter)

### 9.1 ExchangeFilterFunction

过滤器用于拦截和修改请求和响应,类似于 Servlet 的 Filter。

#### 基本过滤器结构

```java
public class CustomFilter implements ExchangeFilterFunction {
    @Override
    public Mono<ClientResponse> filter(ClientRequest request, ExchangeFunction next) {
        // 请求前处理
        ClientRequest modifiedRequest = ClientRequest.from(request)
            .header("Custom-Header", "value")
            .build();
        
        // 继续执行
        return next.exchange(modifiedRequest)
            .flatMap(response -> {
                // 响应后处理
                return Mono.just(response);
            });
    }
}
```

### 9.2 常用过滤器示例

#### 日志过滤器

```java
public class LoggingFilter implements ExchangeFilterFunction {
    
    private static final Logger log = LoggerFactory.getLogger(LoggingFilter.class);
    
    @Override
    public Mono<ClientResponse> filter(ClientRequest request, ExchangeFunction next) {
        long startTime = System.currentTimeMillis();
        
        log.info("请求开始: {} {}", request.method(), request.url());
        request.headers().forEach((name, values) -> 
            log.debug("请求头: {}={}", name, values)
        );
        
        return next.exchange(request)
            .doOnSuccess(response -> {
                long duration = System.currentTimeMillis() - startTime;
                log.info("请求完成: {} {} - 状态码: {} - 耗时: {}ms",
                    request.method(), request.url(), 
                    response.statusCode(), duration);
            })
            .doOnError(error -> {
                long duration = System.currentTimeMillis() - startTime;
                log.error("请求失败: {} {} - 耗时: {}ms",
                    request.method(), request.url(), duration, error);
            });
    }
}
```

#### 认证过滤器

```java
public class AuthenticationFilter implements ExchangeFilterFunction {
    
    private final TokenService tokenService;
    
    public AuthenticationFilter(TokenService tokenService) {
        this.tokenService = tokenService;
    }
    
    @Override
    public Mono<ClientResponse> filter(ClientRequest request, ExchangeFunction next) {
        String token = tokenService.getToken();
        
        ClientRequest authenticatedRequest = ClientRequest.from(request)
            .header(HttpHeaders.AUTHORIZATION, "Bearer " + token)
            .build();
        
        return next.exchange(authenticatedRequest);
    }
}
```

#### Token 刷新过滤器

```java
public class TokenRefreshFilter implements ExchangeFilterFunction {
    
    private final TokenService tokenService;
    
    @Override
    public Mono<ClientResponse> filter(ClientRequest request, ExchangeFunction next) {
        return next.exchange(request)
            .flatMap(response -> {
                if (response.statusCode() == HttpStatus.UNAUTHORIZED) {
                    // Token 过期,刷新 token 后重试
                    return tokenService.refreshToken()
                        .flatMap(newToken -> {
                            ClientRequest retryRequest = ClientRequest.from(request)
                                .header(HttpHeaders.AUTHORIZATION, "Bearer " + newToken)
                                .build();
                            return next.exchange(retryRequest);
                        });
                }
                return Mono.just(response);
            });
    }
}
```

#### 限流过滤器

```java
public class RateLimitFilter implements ExchangeFilterFunction {
    
    private final RateLimiter rateLimiter;
    
    public RateLimitFilter(RateLimiter rateLimiter) {
        this.rateLimiter = rateLimiter;
    }
    
    @Override
    public Mono<ClientResponse> filter(ClientRequest request, ExchangeFunction next) {
        if (!rateLimiter.tryAcquire()) {
            return Mono.error(new RateLimitException("请求过于频繁"));
        }
        return next.exchange(request);
    }
}
```

### 9.3 使用过滤器

```java
@Configuration
public class WebClientConfig {
    
    @Bean
    public WebClient webClient(TokenService tokenService) {
        return WebClient.builder()
            .baseUrl("https://api.example.com")
            .filter(new LoggingFilter())
            .filter(new AuthenticationFilter(tokenService))
            .filter(new TokenRefreshFilter(tokenService))
            .filter(new RateLimitFilter(RateLimiter.create(10.0)))
            .build();
    }
}
```

### 9.4 函数式过滤器

```java
// 简单的函数式过滤器
ExchangeFilterFunction addHeaderFilter = (request, next) -> {
    ClientRequest filtered = ClientRequest.from(request)
        .header("X-Custom-Header", "value")
        .build();
    return next.exchange(filtered);
};

WebClient webClient = WebClient.builder()
    .filter(addHeaderFilter)
    .build();

// 使用 ExchangeFilterFunction 工厂方法
ExchangeFilterFunction logRequest = ExchangeFilterFunction.ofRequestProcessor(
    clientRequest -> {
        log.info("Request: {} {}", clientRequest.method(), clientRequest.url());
        return Mono.just(clientRequest);
    }
);

ExchangeFilterFunction logResponse = ExchangeFilterFunction.ofResponseProcessor(
    clientResponse -> {
        log.info("Response: {}", clientResponse.statusCode());
        return Mono.just(clientResponse);
    }
);
```

------

## 十、实战案例

### 10.1 完整的 Service 层实现

```java
@Service
@Slf4j
public class UserService {
    
    private final WebClient webClient;
    
    public UserService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder
            .baseUrl("https://api.example.com")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();
    }
    
    // 同步方式获取用户
    public User getUserById(Long id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .onStatus(HttpStatus::is4xxClientError, response -> 
                Mono.error(new UserNotFoundException("用户不存在: " + id))
            )
            .bodyToMono(User.class)
            .block();
    }
    
    // 异步方式获取用户
    public Mono<User> getUserByIdAsync(Long id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(User.class)
            .timeout(Duration.ofSeconds(5))
            .doOnError(error -> log.error("获取用户失败 id={}", id, error));
    }
    
    // 获取用户列表(分页)
    public Mono<Page<User>> getUsers(int page, int size) {
        ParameterizedTypeReference<Page<User>> typeRef = 
            new ParameterizedTypeReference<Page<User>>() {};
        
        return webClient.get()
            .uri(uriBuilder -> uriBuilder
                .path("/users")
                .queryParam("page", page)
                .queryParam("size", size)
                .build())
            .retrieve()
            .bodyToMono(typeRef);
    }
    
    // 创建用户
    public Mono<User> createUser(UserCreateRequest request) {
        return webClient.post()
            .uri("/users")
            .bodyValue(request)
            .retrieve()
            .bodyToMono(User.class)
            .doOnSuccess(user -> log.info("用户创建成功: {}", user.getId()));
    }
    
    // 更新用户
    public Mono<User> updateUser(Long id, UserUpdateRequest request) {
        return webClient.put()
            .uri("/users/{id}", id)
            .bodyValue(request)
            .retrieve()
            .bodyToMono(User.class);
    }
    
    // 删除用户
    public Mono<Void> deleteUser(Long id) {
        return webClient.delete()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(Void.class)
            .doOnSuccess(v -> log.info("用户删除成功: {}", id));
    }
    
    // 批量获取用户
    public Mono<List<User>> getUsersByIds(List<Long> ids) {
        return Flux.fromIterable(ids)
            .flatMap(id -> getUserByIdAsync(id)
                .onErrorResume(error -> {
                    log.warn("获取用户失败 id={}", id, error);
                    return Mono.empty();
                })
            )
            .collectList();
    }
}
```

### 10.2 文件下载

```java
@Service
public class FileDownloadService {
    
    private final WebClient webClient;
    
    // 下载文件到内存
    public byte[] downloadFile(String fileUrl) {
        return webClient.get()
            .uri(fileUrl)
            .accept(MediaType.APPLICATION_OCTET_STREAM)
            .retrieve()
            .bodyToMono(byte[].class)
            .block();
    }
    
    // 下载文件到本地
    public Mono<Void> downloadFileToPath(String fileUrl, Path targetPath) {
        return webClient.get()
            .uri(fileUrl)
            .accept(MediaType.APPLICATION_OCTET_STREAM)
            .retrieve()
            .bodyToFlux(DataBuffer.class)
            .flatMap(dataBuffer -> {
                try {
                    byte[] bytes = new byte[dataBuffer.readableByteCount()];
                    dataBuffer.read(bytes);
                    DataBufferUtils.release(dataBuffer);
                    return Mono.fromCallable(() -> {
                        Files.write(targetPath, bytes, StandardOpenOption.CREATE, 
                                  StandardOpenOption.APPEND);
                        return bytes;
                    });
                } catch (IOException e) {
                    return Mono.error(e);
                }
            })
            .then();
    }
    
    // 流式下载大文件
    public Flux<DataBuffer> streamDownload(String fileUrl) {
        return webClient.get()
            .uri(fileUrl)
            .accept(MediaType.APPLICATION_OCTET_STREAM)
            .retrieve()
            .bodyToFlux(DataBuffer.class);
    }
}
```

### 10.3 第三方 API 集成

```java
@Service
@Slf4j
public class WeatherService {
    
    private final WebClient webClient;
    
    @Value("${weather.api.key}")
    private String apiKey;
    
    public WeatherService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder
            .baseUrl("https://api.weather.com")
            .defaultHeader("Accept-Language", "zh-CN")
            .filter((request, next) -> {
                log.info("调用天气API: {}", request.url());
                return next.exchange(request);
            })
            .build();
    }
    
    public Mono<WeatherResponse> getCurrentWeather(String city) {
        return webClient.get()
            .uri(uriBuilder -> uriBuilder
                .path("/v1/current")
                .queryParam("city", city)
                .queryParam("key", apiKey)
                .build())
            .retrieve()
            .onStatus(HttpStatus::isError, response -> 
                response.bodyToMono(String.class)
                    .flatMap(body -> Mono.error(
                        new WeatherApiException("天气API调用失败: " + body)
                    ))
            )
            .bodyToMono(WeatherResponse.class)
            .timeout(Duration.ofSeconds(10))
            .retryWhen(Retry.backoff(2, Duration.ofSeconds(1))
                .filter(throwable -> !(throwable instanceof WeatherApiException))
            )
            .doOnError(error -> log.error("获取天气失败 city={}", city, error));
    }
    
    public Mono<List<WeatherForecast>> getForecast(String city, int days) {
        return webClient.get()
            .uri(uriBuilder -> uriBuilder
                .path("/v1/forecast")
                .queryParam("city", city)
                .queryParam("days", days)
                .queryParam("key", apiKey)
                .build())
            .retrieve()
            .bodyToFlux(WeatherForecast.class)
            .collectList()
            .cache(Duration.ofMinutes(30)); // 缓存30分钟
    }
}
```

### 10.4 微服务间调用

```java
@Service
@Slf4j
public class OrderService {
    
    private final WebClient userServiceClient;
    private final WebClient productServiceClient;
    private final WebClient paymentServiceClient;
    
    public OrderService(
            @Qualifier("userServiceWebClient") WebClient userServiceClient,
            @Qualifier("productServiceWebClient") WebClient productServiceClient,
            @Qualifier("paymentServiceWebClient") WebClient paymentServiceClient) {
        this.userServiceClient = userServiceClient;
        this.productServiceClient = productServiceClient;
        this.paymentServiceClient = paymentServiceClient;
    }
    
    public Mono<OrderDetail> createOrder(CreateOrderRequest request) {
        // 1. 验证用户
        Mono<User> userMono = userServiceClient.get()
            .uri("/users/{id}", request.getUserId())
            .retrieve()
            .bodyToMono(User.class);
        
        // 2. 验证商品库存
        Mono<Product> productMono = productServiceClient.get()
            .uri("/products/{id}", request.getProductId())
            .retrieve()
            .bodyToMono(Product.class)
            .filter(product -> product.getStock() >= request.getQuantity())
            .switchIfEmpty(Mono.error(new OutOfStockException("库存不足")));
        
        // 3. 并行验证用户和商品
        return Mono.zip(userMono, productMono)
            .flatMap(tuple -> {
                User user = tuple.getT1();
                Product product = tuple.getT2();
                
                // 4. 创建订单
                Order order = new Order(user, product, request.getQuantity());
                
                // 5. 扣减库存
                return productServiceClient.post()
                    .uri("/products/{id}/reduce-stock", product.getId())
                    .bodyValue(Map.of("quantity", request.getQuantity()))
                    .retrieve()
                    .bodyToMono(Void.class)
                    .then(Mono.just(order));
            })
            .flatMap(order -> {
                // 6. 调用支付服务
                return paymentServiceClient.post()
                    .uri("/payments")
                    .bodyValue(new PaymentRequest(order))
                    .retrieve()
                    .bodyToMono(Payment.class)
                    .map(payment -> new OrderDetail(order, payment));
            })
            .doOnSuccess(orderDetail -> 
                log.info("订单创建成功: {}", orderDetail.getOrderId())
            )
            .doOnError(error -> 
                log.error("订单创建失败", error)
            );
    }
}
```

------

## 十一、最佳实践

### 11.1 WebClient Bean 配置

```java
@Configuration
public class WebClientConfig {
    
    // 通用 WebClient Builder
    @Bean
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder()
            .codecs(configurer -> 
                configurer.defaultCodecs().maxInMemorySize(2 * 1024 * 1024)
            );
    }
    
    // 特定服务的 WebClient
    @Bean
    @Qualifier("userServiceWebClient")
    public WebClient userServiceWebClient(
            WebClient.Builder builder,
            @Value("${services.user.url}") String baseUrl) {
        return builder
            .baseUrl(baseUrl)
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .filter(loggingFilter())
            .build();
    }
    
    @Bean
    @Qualifier("productServiceWebClient")
    public WebClient productServiceWebClient(
            WebClient.Builder builder,
            @Value("${services.product.url}") String baseUrl) {
        return builder
            .baseUrl(baseUrl)
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();
    }
    
    private ExchangeFilterFunction loggingFilter() {
        return (request, next) -> {
            log.info("Request: {} {}", request.method(), request.url());
            return next.exchange(request);
        };
    }
}
```

### 11.2 异常处理策略

```java
@ControllerAdvice
public class WebClientExceptionHandler {
    
    @ExceptionHandler(WebClientResponseException.class)
    public ResponseEntity<ErrorResponse> handleWebClientResponseException(
            WebClientResponseException ex) {
        
        ErrorResponse error = new ErrorResponse(
            ex.getStatusCode().value(),
            "外部服务调用失败: " + ex.getMessage(),
            ex.getResponseBodyAsString()
        );
        
        return ResponseEntity
            .status(ex.getStatusCode())
            .body(error);
    }
    
    @ExceptionHandler(WebClientRequestException.class)
    public ResponseEntity<ErrorResponse> handleWebClientRequestException(
            WebClientRequestException ex) {
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.SERVICE_UNAVAILABLE.value(),
            "服务不可用",
            ex.getMessage()
        );
        
        return ResponseEntity
            .status(HttpStatus.SERVICE_UNAVAILABLE)
            .body(error);
    }
}
```

### 11.3 性能优化

```java
// 1. 使用连接池
@Bean
public WebClient optimizedWebClient() {
    ConnectionProvider provider = ConnectionProvider.builder("custom")
        .maxConnections(500)
        .maxIdleTime(Duration.ofSeconds(20))
        .maxLifeTime(Duration.ofSeconds(60))
        .pendingAcquireTimeout(Duration.ofSeconds(60))
        .evictInBackground(Duration.ofSeconds(120))
        .build();
    
    HttpClient httpClient = HttpClient.create(provider)
        .responseTimeout(Duration.ofSeconds(5))
        .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000);
    
    return WebClient.builder()
        .clientConnector(new ReactorClientHttpConnector(httpClient))
        .build();
}

// 2. 缓存策略
@Service
public class CachedApiService {
    
    private final WebClient webClient;
    private final Cache<String, User> userCache;
    
    public Mono<User> getUserWithCache(Long id) {
        String cacheKey = "user:" + id;
        
        // 先查缓存
        User cachedUser = userCache.getIfPresent(cacheKey);
        if (cachedUser != null) {
            return Mono.just(cachedUser);
        }
        
        // 缓存未命中,调用 API
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(User.class)
            .doOnSuccess(user -> userCache.put(cacheKey, user))
            .cache(Duration.ofMinutes(10)); // Reactor 缓存
    }
}

// 3. 批量请求优化
public Mono<List<User>> getUsersBatch(List<Long> ids) {
    return Flux.fromIterable(ids)
        .buffer(10) // 每批 10 个
        .flatMap(batch -> 
            webClient.post()
                .uri("/users/batch")
                .bodyValue(batch)
                .retrieve()
                .bodyToFlux(User.class)
                .collectList()
        )
        .flatMap(Flux::fromIterable)
        .collectList();
}
```

### 11.4 测试

```java
@SpringBootTest
class UserServiceTest {
    
    private MockWebServer mockWebServer;
    private WebClient webClient;
    private UserService userService;
    
    @BeforeEach
    void setUp() throws IOException {
        mockWebServer = new MockWebServer();
        mockWebServer.start();
        
        webClient = WebClient.builder()
            .baseUrl(mockWebServer.url("/").toString())
            .build();
        
        userService = new UserService(webClient);
    }
    
    @AfterEach
    void tearDown() throws IOException {
        mockWebServer.shutdown();
    }
    
    @Test
    void testGetUser() {
        // 模拟响应
        String mockResponse = """
            {
                "id": 123,
                "name": "张三",
                "email": "zhangsan@example.com"
            }
            """;
        
        mockWebServer.enqueue(new MockResponse()
            .setBody(mockResponse)
            .setHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .setResponseCode(200));
        
        // 执行测试
        User user = userService.getUserById(123L);
        
        // 验证
        assertThat(user).isNotNull();
        assertThat(user.getId()).isEqualTo(123L);
        assertThat(user.getName()).isEqualTo("张三");
        
        // 验证请求
        RecordedRequest request = mockWebServer.takeRequest();
        assertThat(request.getMethod()).isEqualTo("GET");
        assertThat(request.getPath()).isEqualTo("/users/123");
    }
    
    @Test
    void testGetUserNotFound() {
        mockWebServer.enqueue(new MockResponse()
            .setResponseCode(404)
            .setBody("Not Found"));
        
        assertThatThrownBy(() -> userService.getUserById(999L))
            .isInstanceOf(UserNotFoundException.class);
    }
}
```

### 11.5 监控和指标

```java
@Configuration
public class WebClientMetricsConfig {
    
    @Bean
    public WebClient monitoredWebClient(MeterRegistry meterRegistry) {
        return WebClient.builder()
            .filter((request, next) -> {
                Timer.Sample sample = Timer.start(meterRegistry);
                
                return next.exchange(request)
                    .doOnSuccess(response -> {
                        sample.stop(Timer.builder("webclient.requests")
                            .tag("method", request.method().name())
                            .tag("uri", request.url().getPath())
                            .tag("status", String.valueOf(response.statusCode().value()))
                            .register(meterRegistry));
                    })
                    .doOnError(error -> {
                        sample.stop(Timer.builder("webclient.requests")
                            .tag("method", request.method().name())
                            .tag("uri", request.url().getPath())
                            .tag("status", "error")
                            .register(meterRegistry));
                    });
            })
            .build();
    }
}
```

------

## 十二、常见问题和注意事项

### 12.1 阻塞调用的注意事项

```java
// ❌ 错误: 在响应式上下文中使用 block()
@GetMapping("/users/{id}")
public Mono<User> getUser(@PathVariable Long id) {
    // 不要这样做!
    User user = webClient.get()
        .uri("/users/{id}", id)
        .retrieve()
        .bodyToMono(User.class)
        .block();  // 这会阻塞响应式流
    
    return Mono.just(user);
}

// ✅ 正确: 保持响应式
@GetMapping("/users/{id}")
public Mono<User> getUser(@PathVariable Long id) {
    return webClient.get()
        .uri("/users/{id}", id)
        .retrieve()
        .bodyToMono(User.class);
}
```

### 12.2 内存泄漏风险

```java
// ❌ 使用 exchange() 但未正确处理响应
webClient.get()
    .uri("/api/data")
    .exchange()
    .subscribe(response -> {
        // 如果没有消费响应体,可能导致内存泄漏
    });

// ✅ 使用 retrieve() 或正确处理 exchange()
webClient.get()
    .uri("/api/data")
    .exchangeToMono(response -> {
        if (response.statusCode().is2xxSuccessful()) {
            return response.bodyToMono(String.class);
        } else {
            // 必须消费或释放响应
            return response.releaseBody()
                .then(Mono.error(new RuntimeException("错误")));
        }
    })
    .subscribe();
```

### 12.3 超时配置

```java
// 不同级别的超时
WebClient webClient = WebClient.builder()
    .clientConnector(new ReactorClientHttpConnector(
        HttpClient.create()
            .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 5000) // 连接超时
            .responseTimeout(Duration.ofSeconds(5))              // 响应超时
            .doOnConnected(conn -> 
                conn.addHandlerLast(new ReadTimeoutHandler(5))  // 读超时
                    .addHandlerLast(new WriteTimeoutHandler(5)) // 写超时
            )
    ))
    .build();

// 单次请求超时
webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .timeout(Duration.ofSeconds(3)) // 覆盖默认超时
    .block();
```

### 12.4 错误传播

```java
// WebClient 的错误会传播到订阅者
webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .subscribe(
        data -> log.info("成功: {}", data),
        error -> log.error("失败", error),  // 必须处理错误
        () -> log.info("完成")
    );
```

### 12.5 线程模型

```java
// WebClient 使用非阻塞 I/O
// 默认使用 Reactor Netty 的事件循环线程
// 不要在回调中执行阻塞操作

// ❌ 错误
webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .map(data -> {
        Thread.sleep(1000); // 阻塞操作!
        return data;
    });

// ✅ 正确: 将阻塞操作移到其他线程池
webClient.get()
    .uri("/api/data")
    .retrieve()
    .bodyToMono(String.class)
    .publishOn(Schedulers.boundedElastic()) // 切换到可阻塞的线程池
    .map(data -> {
        // 现在可以执行阻塞操作
        someBlockingOperation(data);
        return data;
    });
```

------

## 十三、总结

### 核心要点

1. **WebClient 是 Spring 推荐的 HTTP 客户端**,用于替代 RestTemplate
2. **基于响应式编程**,支持非阻塞异步调用
3. **函数式 API 设计**,链式调用清晰易读
4. **灵活配置**,支持超时、连接池、拦截器等
5. **完善的错误处理**,支持重试、降级等策略

### 使用建议

- 新项目优先使用 WebClient
- 高并发场景充分利用响应式特性
- 合理配置超时和连接池
- 善用过滤器实现横切关注点
- 注意避免在响应式流中使用 block()
- 做好错误处理和监控

### 学习路径

1. 掌握基础用法(GET、POST 等)
2. 理解 Mono 和 Flux
3. 学习配置和优化
4. 实践错误处理和重试
5. 探索高级特性(过滤器、指标等)

------

## 附录

### 常用依赖

```xml
<!-- WebFlux -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>

<!-- Reactor Test -->
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- MockWebServer (测试) -->
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>mockwebserver</artifactId>
    <scope>test</scope>
</dependency>
```

### 参考资源

- [Spring WebFlux 官方文档](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)
- [WebClient 官方文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/reactive/function/client/WebClient.html)
- [Project Reactor 文档](https://projectreactor.io/docs/core/release/reference/)
- [Reactive Streams 规范](https://www.reactive-streams.org/)