# JWT (JSON Web Token) 

## 1. 什么是 JWT？

JWT (JSON Web Token) 是一个开放标准 ([RFC 7519](https://tools.ietf.org/html/rfc7519))，它定义了一种紧凑且自包含的方式，用于在各方之间安全地传输信息。这些信息会经过数字签名，因此可以被验证和信任。

**核心特点：**

- **紧凑 (Compact):** JWT 的体积很小，可以通过 URL、POST 参数或者 HTTP Header 发送。
- **自包含 (Self-contained):** JWT 的载荷（Payload）部分包含了所有需要的信息（例如用户信息、权限等），避免了多次查询数据库。
- **安全 (Secure):** JWT 可以被签名（例如使用 HMAC 算法）或加密，以保证其完整性和保密性。服务器可以通过验证签名来确认 Token 未被篡改。

在 Web 开发中，JWT 最常见的应用场景就是**身份认证**。一旦用户登录成功，服务器就会生成一个 JWT 并返回给客户端。客户端在后续的每次请求中，都需要携带这个 JWT，服务器通过验证 JWT 来确认用户的身份。

## 2. JWT 的组成部分

一个标准的 JWT 看上去像这样的一长串字符，由两上 `.` 分隔成三个部分：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

这三个部分分别是：

1. **Header (头部)**
2. **Payload (载荷)**
3. **Signature (签名)**

### 第一部分：Header (头部)

Header 部分通常由两部分组成：

- `typ` (Type): 令牌的类型，固定为 `JWT`。
- `alg` (Algorithm): 使用的签名算法，例如 `HS256` (HMAC SHA256) 或 `RS256` (RSA SHA256)。

这个 JSON 对象会经过 **Base64Url** 编码，形成 JWT 的第一部分。

**示例：**

```
{
  "alg": "HS256",
  "type": "JWT"
}
```

编码后得到 `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`。

### 第二部分：Payload (载荷)

Payload 部分是 JWT 的主体，用于存放实际需要传输的数据，也称为**声明 (Claims)**。声明是关于实体（通常是用户）和其他数据的陈述。

声明有三种类型：

1. **Registered Claims (注册声明):** 这些是预定义的一组声明，虽然不是强制性的，但建议使用，以提供一组有用的、可互操作的声明。
   - `iss` (Issuer): 签发者
   - `sub` (Subject): 主题，通常是用户的唯一标识符 (ID)
   - `aud` (Audience): 接收方
   - `exp` (Expiration Time): 过期时间戳
   - `nbf` (Not Before): 生效时间戳
   - `iat` (Issued At): 签发时间戳
   - `jti` (JWT ID): JWT 的唯一标识
2. **Public Claims (公共声明):** 可以随意定义，但为了避免冲突，应该在 [IANA JSON Web Token Registry](https://www.google.com/search?q=https://www.iana.org/assignments/json-web-token/json-web-token.xhtml) 中定义，或者定义为包含命名空间的 URI。
3. **Private Claims (私有声明):** 这是在通信双方之间共享的自定义声明，既不是注册声明也不是公共声明。例如，你可以添加 `username`、`roles` 等自定义字段。

**注意：** Payload 默认是不加密的，只是经过 Base64Url 编码。**因此，绝对不要在 Payload 中存放任何敏感信息，例如用户的密码。**

**示例：**

```
{
  "sub": "user123",
  "username": "Alice",
  "roles": ["USER", "VIEWER"],
  "iat": 1662541800,
  "exp": 1662545400
}
```

编码后得到 JWT 的第二部分。

### 第三部分：Signature (签名)

签名部分是对前两部分（Header 和 Payload）的签名，用于验证消息在传递过程中没有被篡改。

签名的生成过程如下：

1. 获取编码后的 Header 和编码后的 Payload，并用 `.` 连接起来。
2. 使用 Header 中指定的签名算法 (`alg`) 和一个**密钥 (Secret)** 对连接后的字符串进行签名。

**公式：**

```
Signature = HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

这个 `secret` **必须保存在服务器端**，绝对不能泄露给客户端。正是因为这个密钥的存在，服务器才能验证收到的 JWT 是否合法且未被篡改。

## 3. JWT 的工作流程

下面是基于 JWT 的认证流程：

1. **用户登录：** 用户使用用户名和密码发起登录请求。

2. **服务器验证：** 服务器验证用户的凭据是否正确。

3. **生成 JWT：** 如果验证通过，服务器会创建一个包含用户标识和权限信息的 Payload，然后用一个密钥生成签名，最终组装成一个完整的 JWT。

4. **返回 JWT：** 服务器将生成的 JWT 返回给客户端。

5. **客户端存储：** 客户端（例如浏览器）收到 JWT 后，通常会将其存储在 `localStorage`、`sessionStorage` 或 HTTP Header (`Authorization`) 中。

6. **后续请求：** 在之后的每一次请求中，客户端都需要将 JWT 附加到请求中，最常见的方式是放在 HTTP Header 的 `Authorization` 字段里，并使用 `Bearer` 方案：

   ```
   Authorization: Bearer <token>
   ```

7. **服务器验证请求：** 服务器收到请求后，会从 Header 中获取 JWT。然后使用**相同的密钥**验证 JWT 的签名。

   - 如果签名验证通过，服务器就可以信任这个 JWT，并从中解析出用户信息，执行相应的操作。
   - 如果签名验证失败，或者 JWT 已过期，服务器会拒绝该请求，返回 `401 Unauthorized` 或 `403 Forbidden` 错误。

## 4. JWT 的优缺点

### 优点

- **无状态与可扩展性 (Stateless & Scalable):**
  - 服务器端不需要存储 Session 信息。用户的状态信息完全包含在 JWT 中。
  - 这使得应用非常容易扩展。任何一台服务器只要拥有相同的密钥，就能处理用户的请求，非常适合分布式系统和微服务架构。
- **解耦 (Decoupling):**
  - 后端服务和认证服务可以解耦。只要能验证 JWT 的签名，任何服务都可以信任这个请求。
- **性能：**
  - 由于 Payload 中可以包含用户信息，减少了频繁查询数据库获取用户信息的需要。
- **适用于多客户端 (Mobile & Web):**
  - JWT 不依赖于 Cookie，因此在 Web、移动端（iOS, Android）等不同平台上都能很好地工作。

### 缺点

- **Token 体积较大：** 如果在 Payload 中存放过多信息，会导致 JWT 体积增大，增加网络传输的开销。
- **无法主动失效 (Revocation):**
  - 一旦 JWT 签发，在它过期之前都会有效。服务器无法主动使其失效。
  - **解决方案：**
    1. **设置较短的过期时间：** 这是最简单的方式，但会影响用户体验（需要频繁登录）。
    2. **维护一个黑名单：** 在服务器端（例如用 Redis）维护一个已失效的 JWT 列表。每次收到请求时，先检查 JWT 是否在黑名单中。
- **安全性依赖于密钥：** 签名的密钥一旦泄露，攻击者就可以伪造任意的 JWT，后果非常严重。必须妥善保管密钥。
- **续签问题 (Renewal):**
  - 为了平衡安全性和用户体验，通常会使用**双 Token 机制**：
    - **Access Token:** 过期时间很短（例如 15 分钟），用于访问受保护的资源。
    - **Refresh Token:** 过期时间很长（例如 7 天），专门用于在 Access Token 过期后获取新的 Access Token。Refresh Token 只在请求新 Token 时才发送，并且需要安全地存储。

## 5. 在 Spring Boot 中使用 JWT

在 Spring Boot 项目中集成 JWT 通常需要以下步骤：

1. **添加依赖：** 在 `pom.xml` 中添加 JWT 相关的库，例如 `jjwt`。

   ```
   <!-- Java JWT: A library for creating and parsing JWTs -->
   <dependency>
       <groupId>io.jsonwebtoken</groupId>
       <artifactId>jjwt-api</artifactId>
       <version>0.11.5</version>
   </dependency>
   <dependency>
       <groupId>io.jsonwebtoken</groupId>
       <artifactId>jjwt-impl</artifactId>
       <version>0.11.5</version>
       <scope>runtime</scope>
   </dependency>
   <dependency>
       <groupId>io.jsonwebtoken</groupId>
       <artifactId>jjwt-jackson</artifactId>
       <version>0.11.5</version>
       <scope>runtime</scope>
   </dependency>
   ```

2. **创建 JWT 工具类 (JwtUtils):** 创建一个工具类，封装生成和解析 JWT 的逻辑。

   - **生成 Token:** 提供一个方法，接收用户信息（如用户 ID、用户名），生成 JWT 字符串。
   - **解析 Token:** 提供一个方法，接收 JWT 字符串，验证签名并从中提取 Claims。

3. **与 Spring Security 集成：**

   - **配置 Spring Security:** 禁用默认的 Session 管理 (`STATELESS`)。
   - **创建 JWT 认证过滤器:** 自定义一个过滤器 (例如 `JwtAuthenticationFilter`)，它继承自 `OncePerRequestFilter`。
   - **过滤器的逻辑:**
     1. 在 `doFilterInternal` 方法中，从请求的 `Authorization` Header 中获取 Token。
     2. 如果 Token 存在且有效，调用 JWT 工具类进行解析和验证。
     3. 如果验证成功，根据 Token 中的用户信息构建一个 `Authentication` 对象（例如 `UsernamePasswordAuthenticationToken`）。
     4. 将这个 `Authentication` 对象设置到 `SecurityContextHolder` 中，这样 Spring Security 的后续授权逻辑就能正常工作。
   - **将过滤器添加到过滤器链中:** 在 Spring Security 的配置类中，使用 `addFilterBefore()` 方法将你的自定义过滤器添加到 `UsernamePasswordAuthenticationFilter` 之前。

4. **创建登录接口：** 提供一个 `/login` 接口，接收用户名和密码。在认证成功后，调用 JWT 工具类生成 Token 并返回给客户端。

## 6. 安全注意事项

1. **始终使用 HTTPS:** 确保所有通信都通过 HTTPS 进行，防止 JWT 在传输过程中被窃听。
2. **密钥安全:**
   - 密钥必须足够复杂且随机。
   - 绝对不要硬编码在代码中，最好通过环境变量或配置文件管理。
   - 定期更换密钥。
3. **设置合理的过期时间 (`exp`):**
   - 不要设置永不过期的 Token。
   - 根据业务场景设置合理的过期时间。
4. **不要在 Payload 中存放敏感信息:** 再次强调，Payload 是公开可见的。
5. **验证算法 (`alg`):** 在解析 Token 时，务必指定并验证签名算法，防止攻击者将算法篡改为 `none` 来绕过签名验证。
6. **考虑引入 Refresh Token 机制:** 对于需要长时间保持登录状态的应用，这是最佳实践。

希望这份笔记能帮助你系统地掌握 JWT！