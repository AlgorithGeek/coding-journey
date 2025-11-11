# JWT (JSON Web Token) 

## 目录

1. [JWT 是什么](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#jwt-是什么)
2. [为什么需要 JWT](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#为什么需要-jwt)
3. [JWT 的结构详解](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#jwt-的结构详解)
4. [JWT 的工作原理](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#jwt-的工作原理)
5. [JWT vs Session 对比](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#jwt-vs-session-对比)
6. [Spring Boot 集成 JWT](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#spring-boot-集成-jwt)
7. [JWT 实战代码](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#jwt-实战代码)
8. [JWT 安全性](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#jwt-安全性)
9. [最佳实践](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#最佳实践)
10. [常见问题与解决方案](https://claude.ai/chat/cf1ea3c2-d48d-4a71-b7a2-43acba0118e2#常见问题与解决方案)

------

## JWT 是什么

### 基本定义

JWT (JSON Web Token) 是一个开放标准 (RFC 7519),它定义了一种紧凑且自包含的方式,用于在各方之间以 JSON 对象的形式安全地传输信息。

### 核心特点

1. **紧凑性(Compact)**: 体积小,可以通过 URL、POST 参数或 HTTP Header 发送
2. **自包含(Self-contained)**: 包含了关于用户的所有必要信息,避免多次查询数据库
3. **跨语言**: JSON 格式,各种语言都支持
4. **无状态**: 服务器不需要保存会话信息

### 应用场景

- **身份认证(Authentication)**: 用户登录后,后续请求都携带 JWT
- **信息交换(Information Exchange)**: 安全地在各方之间传输信息
- **单点登录(SSO)**: 因为开销小且跨域能力强

------

## 为什么需要 JWT

### 传统 Session 认证的问题

```
客户端                    服务器
  |                         |
  |----1. 登录请求--------->|
  |                         |---- 验证用户
  |                         |---- 创建Session
  |                         |---- 存储到内存/Redis
  |<---2. 返回SessionID-----|
  |                         |
  |----3. 携带SessionID---->|
  |                         |---- 查询Session
  |                         |---- 验证身份
  |<---4. 返回数据----------|
```

**Session 的缺点:**

1. **服务器压力大**: 需要存储所有用户的 Session
2. **扩展性差**: 分布式环境下需要 Session 共享(Redis)
3. **跨域问题**: Cookie 不能跨域
4. **移动端不友好**: 原生 APP 不支持 Cookie

### JWT 认证的优势

```
客户端                    服务器
  |                         |
  |----1. 登录请求--------->|
  |                         |---- 验证用户
  |                         |---- 生成JWT
  |<---2. 返回JWT-----------|
  |                         |
  |----3. 携带JWT---------->|
  |                         |---- 验证JWT签名
  |                         |---- 解析用户信息
  |<---4. 返回数据----------|
```

**JWT 的优点:**

1. **无状态**: 服务器不存储任何会话数据
2. **可扩展**: 天然支持分布式和微服务
3. **跨域友好**: 不依赖 Cookie
4. **性能好**: 不需要查询数据库或 Redis

------

## JWT 的结构详解

JWT 由三部分组成,用点(`.`)分隔:

```
Header.Payload.Signature
```

### 完整示例

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### 1. Header (头部)

**作用**: 描述 JWT 的元数据

**结构**:

```json
{
  "alg": "HS256",     // 签名算法
  "typ": "JWT"        // 令牌类型
}
```

**常见算法**:

- `HS256`: HMAC SHA-256 (对称加密)
- `RS256`: RSA SHA-256 (非对称加密)
- `ES256`: ECDSA SHA-256

**Base64URL 编码后**:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

### 2. Payload (载荷)

**作用**: 存放实际需要传递的数据

**声明类型**:

#### 注册声明 (Registered Claims) - 预定义的

```json
{
  "iss": "issuer",           // 签发者
  "sub": "subject",          // 主题(用户ID)
  "aud": "audience",         // 接收方
  "exp": 1516239022,         // 过期时间(时间戳)
  "nbf": 1516239022,         // 生效时间
  "iat": 1516239022,         // 签发时间
  "jti": "jwt-id"            // JWT ID(唯一标识)
}
```

#### 公共声明 (Public Claims)

可以自定义,但建议使用 IANA JSON Web Token Registry 定义的名称

#### 私有声明 (Private Claims)

自定义的用户信息

**完整示例**:

```json
{
  "sub": "1234567890",          // 用户ID
  "name": "张三",               // 用户名
  "role": "ADMIN",              // 角色
  "permissions": ["read", "write"],  // 权限
  "iat": 1516239022,            // 签发时间
  "exp": 1516242622             // 过期时间(1小时后)
}
```

**Base64URL 编码后**:

```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
```

**⚠️ 重要提示**:

- Payload 只是 Base64 编码,**不是加密**!
- 不要在 Payload 中放敏感信息(密码、身份证号等)
- 任何人都可以解码 Payload 查看内容

### 3. Signature (签名)

**作用**: 验证消息未被篡改,如果使用私钥签名,还可以验证发送者身份

**生成方式**:

```javascript
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

**完整过程**:

```
1. 将 Header 和 Payload 进行 Base64URL 编码
2. 用点(.)连接两个编码后的字符串
3. 使用 Header 中指定的算法和密钥对其进行签名
4. 将签名也进行 Base64URL 编码
```

**示例**:

```
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**签名的安全保障**:

- 如果有人修改了 Header 或 Payload,签名就会失效
- 没有密钥无法伪造签名
- 服务器可以验证 Token 的真实性

------

## JWT 的工作原理

### 完整认证流程

```
┌─────────┐                                    ┌─────────┐
│         │  1. POST /login                    │         │
│         │  {username, password}              │         │
│ 客户端  │ ---------------------------------> │ 服务器  │
│         │                                    │         │
│         │  2. 验证用户名密码                  │         │
│         │     └─> 查询数据库                 │         │
│         │     └─> 验证成功                   │         │
│         │                                    │         │
│         │  3. 生成 JWT                       │         │
│         │     └─> 创建 Payload               │         │
│         │     └─> 使用密钥签名                │         │
│         │                                    │         │
│         │  4. 返回 JWT                       │         │
│         │ <--------------------------------- │         │
│         │  {token: "eyJhbGc..."}            │         │
│         │                                    │         │
│         │  5. 存储 JWT                       │         │
│         │     └─> LocalStorage/Cookie       │         │
│         │                                    │         │
│         │  6. 后续请求携带 JWT               │         │
│         │  GET /api/user                     │         │
│         │  Header: Authorization: Bearer eyJ...│       │
│         │ ---------------------------------> │         │
│         │                                    │         │
│         │  7. 验证 JWT                       │         │
│         │     └─> 验证签名                   │         │
│         │     └─> 检查过期时间                │         │
│         │     └─> 解析用户信息                │         │
│         │                                    │         │
│         │  8. 返回数据                       │         │
│         │ <--------------------------------- │         │
└─────────┘                                    └─────────┘
```

### 详细步骤说明

#### 1. 用户登录

```java
// 前端发送
POST /api/auth/login
Content-Type: application/json

{
  "username": "zhangsan",
  "password": "123456"
}
```

#### 2. 服务器验证并生成 JWT

```java
// 后端处理
1. 验证用户名密码
2. 查询用户信息(ID、角色、权限等)
3. 创建 JWT:
   - 设置 Payload(用户ID、角色等)
   - 设置过期时间
   - 使用密钥签名
4. 返回 JWT
```

#### 3. 客户端存储 JWT

```javascript
// 前端存储
// 方式1: LocalStorage
localStorage.setItem('token', response.data.token);

// 方式2: SessionStorage
sessionStorage.setItem('token', response.data.token);

// 方式3: Cookie (设置 HttpOnly 更安全)
document.cookie = `token=${response.data.token}; path=/`;
```

#### 4. 后续请求携带 JWT

```javascript
// 前端请求
GET /api/user/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// 或者在请求头中
fetch('/api/user/profile', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
});
```

#### 5. 服务器验证 JWT

```java
// 后端验证流程
1. 从请求头获取 Token
2. 验证 Token 格式
3. 验证签名是否正确
4. 检查是否过期
5. 解析 Payload 获取用户信息
6. 继续业务逻辑
```

------

## JWT vs Session 对比

### 详细对比表

| 特性           | Session              | JWT                         |
| -------------- | -------------------- | --------------------------- |
| **存储位置**   | 服务器端(内存/Redis) | 客户端(LocalStorage/Cookie) |
| **服务器压力** | 大(需要存储所有会话) | 小(无状态,不存储)           |
| **扩展性**     | 差(需要会话共享)     | 优(天然支持分布式)          |
| **跨域**       | 不友好(Cookie限制)   | 友好(不依赖Cookie)          |
| **移动端**     | 不友好               | 友好                        |
| **安全性**     | 较高(可以主动销毁)   | 中等(无法主动失效)          |
| **存储容量**   | 无限制               | 有限制(Header大小)          |
| **网络开销**   | 小(只传SessionID)    | 大(每次传整个Token)         |
| **续期**       | 简单(延长过期时间)   | 复杂(需要刷新Token机制)     |

### 使用场景建议

#### 适合使用 Session 的场景

- 单体应用
- 需要精确控制会话(如强制下线)
- 对安全性要求极高的场景
- 传统 Web 应用

#### 适合使用 JWT 的场景

- 分布式系统/微服务
- 移动端 APP
- 跨域的单页应用(SPA)
- RESTful API
- 需要横向扩展的系统

------

## Spring Boot 集成 JWT

### 1. 添加依赖

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!-- JWT 依赖 -->
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
    
    <!-- Lombok (可选) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

### 2. 配置文件

```yaml
# application.yml
jwt:
  secret: mySecretKeyForJWTTokenGenerationAndValidation2024!@#  # 密钥(生产环境要复杂且保密)
  expiration: 86400000  # 过期时间(毫秒) 24小时
  header: Authorization  # Token 在 Header 中的名称
  prefix: Bearer  # Token 前缀

spring:
  application:
    name: jwt-demo
```

### 3. JWT 配置类

```java
@Data
@Component
@ConfigurationProperties(prefix = "jwt")
public class JwtConfig {
    private String secret;
    private Long expiration;
    private String header;
    private String prefix;
}
```

------

## JWT 实战代码

### 1. JWT 工具类

```java
package com.example.jwt.util;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@Slf4j
@Component
public class JwtUtil {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private Long expiration;

    /**
     * 生成 JWT Token
     * @param username 用户名
     * @return Token 字符串
     */
    public String generateToken(String username) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("username", username);
        return createToken(claims, username);
    }

    /**
     * 生成 JWT Token (带额外信息)
     * @param username 用户名
     * @param userId 用户ID
     * @param role 用户角色
     * @return Token 字符串
     */
    public String generateToken(String username, Long userId, String role) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("username", username);
        claims.put("userId", userId);
        claims.put("role", role);
        return createToken(claims, username);
    }

    /**
     * 创建 Token
     * @param claims 自定义声明
     * @param subject 主题(通常是用户名)
     * @return Token 字符串
     */
    private String createToken(Map<String, Object> claims, String subject) {
        Date now = new Date();
        Date expirationDate = new Date(now.getTime() + expiration);

        return Jwts.builder()
                .setClaims(claims)              // 自定义声明
                .setSubject(subject)            // 主题
                .setIssuedAt(now)               // 签发时间
                .setExpiration(expirationDate)  // 过期时间
                .signWith(getSignKey(), SignatureAlgorithm.HS256)  // 签名算法
                .compact();
    }

    /**
     * 从 Token 中提取用户名
     * @param token JWT Token
     * @return 用户名
     */
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    /**
     * 从 Token 中提取用户ID
     * @param token JWT Token
     * @return 用户ID
     */
    public Long extractUserId(String token) {
        Claims claims = extractAllClaims(token);
        return claims.get("userId", Long.class);
    }

    /**
     * 从 Token 中提取角色
     * @param token JWT Token
     * @return 角色
     */
    public String extractRole(String token) {
        Claims claims = extractAllClaims(token);
        return claims.get("role", String.class);
    }

    /**
     * 从 Token 中提取过期时间
     * @param token JWT Token
     * @return 过期时间
     */
    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    /**
     * 从 Token 中提取指定声明
     * @param token JWT Token
     * @param claimsResolver 声明解析器
     * @return 声明值
     */
    public <T> T extractClaim(String token, java.util.function.Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    /**
     * 从 Token 中提取所有声明
     * @param token JWT Token
     * @return 所有声明
     */
    private Claims extractAllClaims(String token) {
        try {
            return Jwts.parserBuilder()
                    .setSigningKey(getSignKey())
                    .build()
                    .parseClaimsJws(token)
                    .getBody();
        } catch (ExpiredJwtException e) {
            log.error("Token 已过期: {}", e.getMessage());
            throw new RuntimeException("Token 已过期");
        } catch (UnsupportedJwtException e) {
            log.error("不支持的 Token: {}", e.getMessage());
            throw new RuntimeException("不支持的 Token");
        } catch (MalformedJwtException e) {
            log.error("Token 格式错误: {}", e.getMessage());
            throw new RuntimeException("Token 格式错误");
        } catch (SignatureException e) {
            log.error("Token 签名无效: {}", e.getMessage());
            throw new RuntimeException("Token 签名无效");
        } catch (IllegalArgumentException e) {
            log.error("Token 为空: {}", e.getMessage());
            throw new RuntimeException("Token 为空");
        }
    }

    /**
     * 检查 Token 是否过期
     * @param token JWT Token
     * @return true-已过期, false-未过期
     */
    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    /**
     * 验证 Token
     * @param token JWT Token
     * @param username 用户名
     * @return true-有效, false-无效
     */
    public Boolean validateToken(String token, String username) {
        final String tokenUsername = extractUsername(token);
        return (tokenUsername.equals(username) && !isTokenExpired(token));
    }

    /**
     * 验证 Token(不需要用户名)
     * @param token JWT Token
     * @return true-有效, false-无效
     */
    public Boolean validateToken(String token) {
        try {
            extractAllClaims(token);
            return !isTokenExpired(token);
        } catch (Exception e) {
            return false;
        }
    }

    /**
     * 获取签名密钥
     * @return 密钥
     */
    private SecretKey getSignKey() {
        byte[] keyBytes = secret.getBytes(StandardCharsets.UTF_8);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    /**
     * 刷新 Token(生成新的 Token)
     * @param token 旧 Token
     * @return 新 Token
     */
    public String refreshToken(String token) {
        Claims claims = extractAllClaims(token);
        return createToken(claims, claims.getSubject());
    }
}
```

### 2. JWT 过滤器

```java
package com.example.jwt.filter;

import com.example.jwt.util.JwtUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * JWT 认证过滤器
 * 拦截所有请求,验证 JWT Token
 */
@Slf4j
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtUtil jwtUtil;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                    HttpServletResponse response, 
                                    FilterChain filterChain) throws ServletException, IOException {
        
        // 1. 从请求头中获取 Token
        String token = getTokenFromRequest(request);

        // 2. 验证 Token
        if (token != null && jwtUtil.validateToken(token)) {
            try {
                // 3. 从 Token 中提取用户名
                String username = jwtUtil.extractUsername(token);

                // 4. 加载用户信息
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                // 5. 创建认证对象
                UsernamePasswordAuthenticationToken authentication = 
                    new UsernamePasswordAuthenticationToken(
                        userDetails, 
                        null, 
                        userDetails.getAuthorities()
                    );
                
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

                // 6. 将认证对象设置到 Security 上下文中
                SecurityContextHolder.getContext().setAuthentication(authentication);
                
                log.debug("用户 {} 通过 JWT 认证", username);
            } catch (Exception e) {
                log.error("JWT 认证失败: {}", e.getMessage());
            }
        }

        // 7. 继续过滤器链
        filterChain.doFilter(request, response);
    }

    /**
     * 从请求头中提取 Token
     * @param request HTTP 请求
     * @return Token 字符串
     */
    private String getTokenFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        
        // 检查 Token 格式: Bearer <token>
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);  // 去掉 "Bearer " 前缀
        }
        
        return null;
    }
}
```

### 3. Security 配置类

```java
package com.example.jwt.config;

import com.example.jwt.filter.JwtAuthenticationFilter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)  // 启用方法级别的权限控制
public class SecurityConfig {

    @Autowired
    private JwtAuthenticationFilter jwtAuthenticationFilter;

    /**
     * 配置 Security 过滤器链
     */
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // 禁用 CSRF(JWT 不需要)
            .csrf().disable()
            
            // 配置请求授权
            .authorizeRequests()
                .antMatchers("/api/auth/**").permitAll()  // 登录接口放行
                .antMatchers("/api/public/**").permitAll()  // 公开接口放行
                .anyRequest().authenticated()  // 其他请求需要认证
            
            .and()
            
            // 配置 Session 管理为无状态
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            
            .and()
            
            // 添加 JWT 过滤器
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    /**
     * 密码编码器
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * 认证管理器
     */
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
}
```

### 4. 用户实体类

```java
package com.example.jwt.entity;

import lombok.Data;
import javax.persistence.*;
import java.util.Date;

@Data
@Entity
@Table(name = "sys_user")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String username;
    
    @Column(nullable = false)
    private String password;
    
    private String email;
    
    private String phone;
    
    private String role;  // USER, ADMIN
    
    private Integer status;  // 0-禁用, 1-启用
    
    @Column(name = "create_time")
    private Date createTime;
    
    @Column(name = "update_time")
    private Date updateTime;
}
```

### 5. UserDetailsService 实现

```java
package com.example.jwt.service;

import com.example.jwt.entity.User;
import com.example.jwt.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.Collections;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 查询用户
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("用户不存在: " + username));

        // 检查用户状态
        if (user.getStatus() == 0) {
            throw new RuntimeException("用户已被禁用");
        }

        // 构建 UserDetails 对象
        return org.springframework.security.core.userdetails.User.builder()
                .username(user.getUsername())
                .password(user.getPassword())
                .authorities(Collections.singletonList(new SimpleGrantedAuthority("ROLE_" + user.getRole())))
                .build();
    }
}
```

### 6. 认证控制器

```java
package com.example.jwt.controller;

import com.example.jwt.dto.LoginRequest;
import com.example.jwt.dto.LoginResponse;
import com.example.jwt.entity.User;
import com.example.jwt.service.UserService;
import com.example.jwt.util.JwtUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;

@Slf4j
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private JwtUtil jwtUtil;

    @Autowired
    private UserService userService;

    /**
     * 用户登录
     */
    @PostMapping("/login")
    public ResponseEntity<?> login(@Valid @RequestBody LoginRequest loginRequest) {
        try {
            // 1. 使用 Spring Security 进行认证
            Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    loginRequest.getUsername(), 
                    loginRequest.getPassword()
                )
            );

            // 2. 认证成功,获取用户信息
            User user = userService.findByUsername(loginRequest.getUsername());

            // 3. 生成 JWT Token
            String token = jwtUtil.generateToken(
                user.getUsername(), 
                user.getId(), 
                user.getRole()
            );

            // 4. 返回 Token
            LoginResponse response = LoginResponse.builder()
                .token(token)
                .username(user.getUsername())
                .role(user.getRole())
                .message("登录成功")
                .build();

            log.info("用户 {} 登录成功", user.getUsername());
            return ResponseEntity.ok(response);

        } catch (AuthenticationException e) {
            log.error("登录失败: {}", e.getMessage());
            return ResponseEntity.status(401).body("用户名或密码错误");
        }
    }

    /**
     * 用户注册
     */
    @PostMapping("/register")
    public ResponseEntity<?> register(@Valid @RequestBody User user) {
        try {
            User registeredUser = userService.register(user);
            return ResponseEntity.ok("注册成功");
        } catch (Exception e) {
            log.error("注册失败: {}", e.getMessage());
            return ResponseEntity.badRequest().body(e.getMessage());
        }
    }

    /**
     * 刷新 Token
     */
    @PostMapping("/refresh")
    public ResponseEntity<?> refresh(@RequestHeader("Authorization") String oldToken) {
        try {
            // 去掉 "Bearer " 前缀
            String token = oldToken.substring(7);
            
            // 验证旧 Token
            if (jwtUtil.validateToken(token)) {
                // 生成新 Token
                String newToken = jwtUtil.refreshToken(token);
                return ResponseEntity.ok(LoginResponse.builder()
                    .token(newToken)
                    .message("Token 刷新成功")
                    .build());
            } else {
                return ResponseEntity.status(401).body("Token 无效");
            }
        } catch (Exception e) {
            log.error("刷新 Token 失败: {}", e.getMessage());
            return ResponseEntity.status(401).body("刷新失败");
        }
    }

    /**
     * 登出(客户端删除 Token 即可)
     */
    @PostMapping("/logout")
    public ResponseEntity<?> logout() {
        // JWT 是无状态的,服务器端不需要做任何操作
        // 客户端删除 Token 即可
        return ResponseEntity.ok("登出成功");
    }
}
```

### 7. DTO 类

```java
// LoginRequest.java
package com.example.jwt.dto;

import lombok.Data;
import javax.validation.constraints.NotBlank;

@Data
public class LoginRequest {
    @NotBlank(message = "用户名不能为空")
    private String username;
    
    @NotBlank(message = "密码不能为空")
    private String password;
}

// LoginResponse.java
package com.example.jwt.dto;

import lombok.Builder;
import lombok.Data;

@Data
@Builder
public class LoginResponse {
    private String token;
    private String username;
    private String role;
    private String message;
}
```

### 8. 测试控制器

```java
package com.example.jwt.controller;

import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class TestController {

    /**
     * 公开接口
     */
    @GetMapping("/public/hello")
    public String publicHello() {
        return "Hello, this is a public endpoint!";
    }

    /**
     * 需要认证的接口
     */
    @GetMapping("/user/profile")
    public String userProfile(Authentication authentication) {
        return "Hello " + authentication.getName() + ", this is your profile!";
    }

    /**
     * 只有 ADMIN 角色才能访问
     */
    @GetMapping("/admin/dashboard")
    @PreAuthorize("hasRole('ADMIN')")
    public String adminDashboard() {
        return "Welcome to Admin Dashboard!";
    }

    /**
     * 只有 USER 角色才能访问
     */
    @GetMapping("/user/data")
    @PreAuthorize("hasRole('USER')")
    public String userData() {
        return "Here is your user data!";
    }
}
```

------

## JWT 安全性

### 1. 常见安全威胁

#### (1) Token 被窃取

**威胁**: XSS 攻击窃取 LocalStorage 中的 Token

**防护措施**:

```javascript
// ❌ 不安全: 存储在 LocalStorage
localStorage.setItem('token', token);

// ✅ 更安全: 使用 HttpOnly Cookie
// 后端设置
response.setHeader("Set-Cookie", 
    "token=" + token + "; HttpOnly; Secure; SameSite=Strict; Path=/");
```

#### (2) 中间人攻击(MITM)

**威胁**: Token 在传输过程中被拦截

**防护措施**:

- **必须使用 HTTPS**
- 设置 Cookie 的 `Secure` 标志
- 使用 HSTS(HTTP Strict Transport Security)

#### (3) Token 过期问题

**威胁**: Token 永不过期,泄露后一直有效

**防护措施**:

```java
// 设置合理的过期时间
private Long accessTokenExpiration = 3600000;  // 1小时
private Long refreshTokenExpiration = 604800000;  // 7天

// 实现双 Token 机制
public TokenPair generateTokenPair(String username) {
    String accessToken = generateAccessToken(username);   // 短期
    String refreshToken = generateRefreshToken(username); // 长期
    return new TokenPair(accessToken, refreshToken);
}
```

#### (4) 密钥泄露

**威胁**: 密钥泄露导致可以伪造 Token

**防护措施**:

```yaml
# ❌ 不要硬编码密钥
jwt:
  secret: 123456

# ✅ 使用环境变量或密钥管理系统
jwt:
  secret: ${JWT_SECRET}

# ✅ 使用足够复杂的密钥(至少256位)
jwt:
  secret: ${random.value}
```

### 2. 最佳安全实践

```java
/**
 * JWT 安全增强工具类
 */
@Component
public class SecureJwtUtil extends JwtUtil {
    
    /**
     * 1. 添加用户指纹(防止 Token 被盗用)
     */
    public String generateTokenWithFingerprint(String username, String userAgent, String ip) {
        String fingerprint = DigestUtils.md5DigestAsHex((userAgent + ip).getBytes());
        
        Map<String, Object> claims = new HashMap<>();
        claims.put("username", username);
        claims.put("fingerprint", fingerprint);
        
        return createToken(claims, username);
    }
    
    /**
     * 2. 验证用户指纹
     */
    public boolean validateFingerprint(String token, String userAgent, String ip) {
        String tokenFingerprint = extractClaim(token, 
            claims -> claims.get("fingerprint", String.class));
        String currentFingerprint = DigestUtils.md5DigestAsHex((userAgent + ip).getBytes());
        
        return tokenFingerprint.equals(currentFingerprint);
    }
    
    /**
     * 3. 添加 JWT ID(防止 Token 重放)
     */
    public String generateTokenWithJti(String username) {
        String jti = UUID.randomUUID().toString();
        
        Map<String, Object> claims = new HashMap<>();
        claims.put("username", username);
        claims.put("jti", jti);
        
        // 将 JTI 存储到 Redis,用于验证
        redisTemplate.opsForValue().set("jti:" + jti, "valid", expiration, TimeUnit.MILLISECONDS);
        
        return createToken(claims, username);
    }
    
    /**
     * 4. 验证 JTI(防止重放攻击)
     */
    public boolean validateJti(String token) {
        String jti = extractClaim(token, claims -> claims.get("jti", String.class));
        String status = redisTemplate.opsForValue().get("jti:" + jti);
        return "valid".equals(status);
    }
    
    /**
     * 5. Token 黑名单机制(实现登出功能)
     */
    public void addToBlacklist(String token) {
        String jti = extractClaim(token, claims -> claims.get("jti", String.class));
        long expiration = extractExpiration(token).getTime() - System.currentTimeMillis();
        
        // 将 Token 加入黑名单,过期时间与 Token 一致
        redisTemplate.opsForValue().set("blacklist:" + jti, "true", 
            expiration, TimeUnit.MILLISECONDS);
    }
    
    /**
     * 6. 检查 Token 是否在黑名单中
     */
    public boolean isInBlacklist(String token) {
        String jti = extractClaim(token, claims -> claims.get("jti", String.class));
        return redisTemplate.hasKey("blacklist:" + jti);
    }
}
```

### 3. Token 刷新机制

```java
/**
 * 双 Token 刷新机制
 */
@Service
public class TokenRefreshService {
    
    @Autowired
    private JwtUtil jwtUtil;
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    /**
     * 生成 Access Token 和 Refresh Token
     */
    public TokenPair generateTokenPair(String username) {
        // Access Token: 短期(1小时)
        String accessToken = jwtUtil.generateToken(username, 3600000L);
        
        // Refresh Token: 长期(7天)
        String refreshToken = UUID.randomUUID().toString();
        
        // 将 Refresh Token 存储到 Redis
        redisTemplate.opsForValue().set(
            "refresh_token:" + username, 
            refreshToken, 
            7, 
            TimeUnit.DAYS
        );
        
        return new TokenPair(accessToken, refreshToken);
    }
    
    /**
     * 使用 Refresh Token 刷新 Access Token
     */
    public String refreshAccessToken(String refreshToken) {
        // 从 Redis 中查找 Refresh Token
        Set<String> keys = redisTemplate.keys("refresh_token:*");
        for (String key : keys) {
            String storedToken = redisTemplate.opsForValue().get(key);
            if (refreshToken.equals(storedToken)) {
                String username = key.replace("refresh_token:", "");
                // 生成新的 Access Token
                return jwtUtil.generateToken(username);
            }
        }
        
        throw new RuntimeException("无效的 Refresh Token");
    }
}

/**
 * Token 对
 */
@Data
@AllArgsConstructor
public class TokenPair {
    private String accessToken;
    private String refreshToken;
}
```

### 4. 防止常见攻击

```java
/**
 * JWT 安全过滤器(增强版)
 */
@Component
public class SecureJwtFilter extends OncePerRequestFilter {
    
    @Autowired
    private SecureJwtUtil secureJwtUtil;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                    HttpServletResponse response, 
                                    FilterChain filterChain) throws ServletException, IOException {
        
        String token = getTokenFromRequest(request);
        
        if (token != null) {
            try {
                // 1. 基本验证(签名、过期时间)
                if (!secureJwtUtil.validateToken(token)) {
                    throw new RuntimeException("Token 无效");
                }
                
                // 2. 检查黑名单(已登出的 Token)
                if (secureJwtUtil.isInBlacklist(token)) {
                    throw new RuntimeException("Token 已失效");
                }
                
                // 3. 验证用户指纹(防止 Token 被盗用)
                String userAgent = request.getHeader("User-Agent");
                String ip = getClientIp(request);
                if (!secureJwtUtil.validateFingerprint(token, userAgent, ip)) {
                    throw new RuntimeException("Token 指纹不匹配");
                }
                
                // 4. 验证 JTI(防止重放攻击)
                if (!secureJwtUtil.validateJti(token)) {
                    throw new RuntimeException("Token 已被使用");
                }
                
                // 5. 设置认证信息
                String username = secureJwtUtil.extractUsername(token);
                // ... 设置到 Security Context
                
            } catch (Exception e) {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                response.getWriter().write("认证失败: " + e.getMessage());
                return;
            }
        }
        
        filterChain.doFilter(request, response);
    }
    
    /**
     * 获取客户端真实 IP
     */
    private String getClientIp(HttpServletRequest request) {
        String ip = request.getHeader("X-Forwarded-For");
        if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("X-Real-IP");
        }
        if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}
```

------

## 最佳实践

### 1. Token 存储位置选择

| 存储位置              | 优点             | 缺点             | 适用场景           |
| --------------------- | ---------------- | ---------------- | ------------------ |
| **LocalStorage**      | 简单易用,容量大  | 易受 XSS 攻击    | 安全要求不高的应用 |
| **SessionStorage**    | 关闭页面自动清除 | 易受 XSS 攻击    | 临时会话           |
| **Cookie (HttpOnly)** | 防止 XSS 攻击    | 可能受 CSRF 攻击 | 安全要求高的应用   |
| **内存(变量)**        | 最安全           | 刷新页面丢失     | 极高安全要求       |

**推荐方案**: Cookie + HttpOnly + Secure + SameSite

### 2. Token 过期时间设置

```java
// 推荐配置
public class TokenConfig {
    // Access Token: 15分钟 - 1小时
    private Long accessTokenExpiration = 3600000;  // 1小时
    
    // Refresh Token: 7天 - 30天
    private Long refreshTokenExpiration = 604800000;  // 7天
    
    // 敏感操作 Token: 5-15分钟
    private Long sensitiveTokenExpiration = 900000;  // 15分钟
}
```

### 3. Token 内容设计

```java
// ✅ 好的做法
{
  "sub": "user123",           // 用户ID
  "username": "zhangsan",     // 用户名
  "role": "USER",             // 角色
  "iat": 1516239022,          // 签发时间
  "exp": 1516242622,          // 过期时间
  "jti": "unique-id"          // Token ID
}

// ❌ 不要放敏感信息
{
  "password": "123456",        // ❌ 永远不要放密码
  "idCard": "110101199001011234",  // ❌ 不要放身份证
  "bankCard": "6222021234567890",  // ❌ 不要放银行卡号
  "privateKey": "xxx"          // ❌ 不要放私钥
}
```

### 4. 性能优化

```java
@Component
public class OptimizedJwtUtil {
    
    // 1. 使用缓存避免重复验证
    @Cacheable(value = "jwt_cache", key = "#token")
    public Claims parseToken(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSignKey())
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
    
    // 2. 批量验证 Token
    public Map<String, Boolean> batchValidate(List<String> tokens) {
        return tokens.parallelStream()
            .collect(Collectors.toMap(
                token -> token,
                this::validateToken
            ));
    }
    
    // 3. 使用更快的算法(RS256 比 HS256 验证更快)
    private Key getSignKey() {
        // 使用 RSA 公钥验证,私钥签名
        return Keys.hmacShaKeyFor(secret.getBytes());
    }
}
```

### 5. 监控和日志

```java
@Aspect
@Component
@Slf4j
public class JwtAuditAspect {
    
    /**
     * 记录 Token 生成
     */
    @After("execution(* com.example.jwt.util.JwtUtil.generateToken(..))")
    public void afterTokenGeneration(JoinPoint joinPoint) {
        Object[] args = joinPoint.getArgs();
        log.info("生成 Token - 用户: {}", args[0]);
    }
    
    /**
     * 记录 Token 验证失败
     */
    @AfterThrowing(pointcut = "execution(* com.example.jwt.util.JwtUtil.validateToken(..))", 
                   throwing = "ex")
    public void afterValidationFailure(JoinPoint joinPoint, Exception ex) {
        log.warn("Token 验证失败 - 原因: {}", ex.getMessage());
    }
}
```

### 6. 错误处理

```java
/**
 * 全局异常处理器
 */
@RestControllerAdvice
public class JwtExceptionHandler {
    
    @ExceptionHandler(ExpiredJwtException.class)
    public ResponseEntity<?> handleExpiredJwt(ExpiredJwtException e) {
        return ResponseEntity.status(401).body(ErrorResponse.builder()
            .code("TOKEN_EXPIRED")
            .message("Token 已过期,请重新登录")
            .build());
    }
    
    @ExceptionHandler(SignatureException.class)
    public ResponseEntity<?> handleInvalidSignature(SignatureException e) {
        return ResponseEntity.status(401).body(ErrorResponse.builder()
            .code("INVALID_SIGNATURE")
            .message("Token 签名无效")
            .build());
    }
    
    @ExceptionHandler(MalformedJwtException.class)
    public ResponseEntity<?> handleMalformedJwt(MalformedJwtException e) {
        return ResponseEntity.status(401).body(ErrorResponse.builder()
            .code("MALFORMED_TOKEN")
            .message("Token 格式错误")
            .build());
    }
}

@Data
@Builder
class ErrorResponse {
    private String code;
    private String message;
    private Long timestamp = System.currentTimeMillis();
}
```

------

## 常见问题与解决方案

### 1. JWT 无法主动失效怎么办?

**问题**: JWT 一旦签发,在过期前都是有效的,无法像 Session 那样主动销毁

**解决方案**:

#### 方案 1: Token 黑名单

```java
@Service
public class TokenBlacklistService {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    /**
     * 登出时将 Token 加入黑名单
     */
    public void logout(String token) {
        String jti = extractJti(token);
        long expiration = getTokenExpiration(token);
        
        // 存储到 Redis,过期时间与 Token 一致
        redisTemplate.opsForValue().set(
            "blacklist:" + jti, 
            "true", 
            expiration, 
            TimeUnit.MILLISECONDS
        );
    }
    
    /**
     * 验证时检查黑名单
     */
    public boolean isBlacklisted(String token) {
        String jti = extractJti(token);
        return redisTemplate.hasKey("blacklist:" + jti);
    }
}
```

#### 方案 2: 短期 Token + 刷新机制

```java
// Access Token 设置较短的过期时间(15分钟)
// Refresh Token 设置较长的过期时间(7天)
// 用户主动操作时使用 Access Token
// Access Token 过期后用 Refresh Token 获取新的 Access Token
```

### 2. Token 被盗用怎么办?

**解决方案**:

#### 方案 1: 添加设备指纹

```java
public String generateToken(String username, HttpServletRequest request) {
    String userAgent = request.getHeader("User-Agent");
    String ip = request.getRemoteAddr();
    String fingerprint = MD5(userAgent + ip);
    
    Map<String, Object> claims = new HashMap<>();
    claims.put("username", username);
    claims.put("fingerprint", fingerprint);
    
    return createToken(claims);
}

public boolean validate(String token, HttpServletRequest request) {
    String tokenFingerprint = extractFingerprint(token);
    String currentFingerprint = MD5(request.getHeader("User-Agent") + request.getRemoteAddr());
    
    return tokenFingerprint.equals(currentFingerprint);
}
```

#### 方案 2: 单点登录检测

```java
// 同一用户只允许一个 Token 有效
@Service
public class SingleTokenService {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    public String generateToken(String username) {
        String token = jwtUtil.generateToken(username);
        String jti = extractJti(token);
        
        // 存储最新的 JTI
        redisTemplate.opsForValue().set("user_token:" + username, jti);
        
        return token;
    }
    
    public boolean validate(String token) {
        String username = extractUsername(token);
        String jti = extractJti(token);
        String latestJti = redisTemplate.opsForValue().get("user_token:" + username);
        
        // 只有最新的 Token 才有效
        return jti.equals(latestJti);
    }
}
```

### 3. Token 太长导致请求头过大

**解决方案**:

#### 方案 1: 减少 Payload 内容

```java
// ❌ Payload 太大
{
  "userId": 12345,
  "username": "zhangsan",
  "email": "zhangsan@example.com",
  "phone": "13800138000",
  "address": "...",
  "permissions": ["read", "write", "delete", ...],  // 很多权限
  ...  // 更多字段
}

// ✅ 只保留必要信息
{
  "sub": "12345",           // 用户ID
  "role": "ADMIN",          // 角色
  "exp": 1516242622         // 过期时间
}
// 其他信息在需要时从数据库查询
```

#### 方案 2: 使用 Token ID 方案

```java
// Token 中只存储一个 ID
{
  "tid": "unique-token-id"
}

// 完整信息存储在 Redis
redis.set("token:unique-token-id", {
  userId: 12345,
  username: "zhangsan",
  permissions: [...]
});
```

### 4. 跨域问题

**解决方案**:

```java
@Configuration
public class CorsConfig {
    
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        
        // 允许的域名
        config.addAllowedOrigin("http://localhost:8080");
        config.addAllowedOrigin("https://example.com");
        
        // 允许的请求头
        config.addAllowedHeader("*");
        
        // 允许的请求方法
        config.addAllowedMethod("*");
        
        // 允许携带凭证(Cookie)
        config.setAllowCredentials(true);
        
        // 暴露的响应头(让前端可以访问)
        config.addExposedHeader("Authorization");
        
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        
        return new CorsFilter(source);
    }
}
```

### 5. 前端使用示例

```javascript
// Axios 拦截器配置
import axios from 'axios';

// 请求拦截器 - 自动添加 Token
axios.interceptors.request.use(
  config => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);

// 响应拦截器 - 处理 Token 过期
axios.interceptors.response.use(
  response => {
    return response;
  },
  async error => {
    const originalRequest = error.config;
    
    // Token 过期
    if (error.response.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      try {
        // 使用 Refresh Token 获取新的 Access Token
        const refreshToken = localStorage.getItem('refreshToken');
        const response = await axios.post('/api/auth/refresh', { refreshToken });
        const newToken = response.data.token;
        
        // 保存新 Token
        localStorage.setItem('token', newToken);
        
        // 重试原请求
        originalRequest.headers.Authorization = `Bearer ${newToken}`;
        return axios(originalRequest);
      } catch (refreshError) {
        // Refresh Token 也过期,跳转到登录页
        localStorage.removeItem('token');
        localStorage.removeItem('refreshToken');
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }
    
    return Promise.reject(error);
  }
);

// 登录
async function login(username, password) {
  try {
    const response = await axios.post('/api/auth/login', {
      username,
      password
    });
    
    const { token, refreshToken } = response.data;
    
    // 保存 Token
    localStorage.setItem('token', token);
    localStorage.setItem('refreshToken', refreshToken);
    
    console.log('登录成功');
  } catch (error) {
    console.error('登录失败:', error);
  }
}

// 登出
async function logout() {
  try {
    await axios.post('/api/auth/logout');
    
    // 清除 Token
    localStorage.removeItem('token');
    localStorage.removeItem('refreshToken');
    
    // 跳转到登录页
    window.location.href = '/login';
  } catch (error) {
    console.error('登出失败:', error);
  }
}

// 获取用户信息
async function getUserProfile() {
  try {
    const response = await axios.get('/api/user/profile');
    console.log('用户信息:', response.data);
  } catch (error) {
    console.error('获取用户信息失败:', error);
  }
}
```

### 6. 性能优化建议

```java
/**
 * JWT 性能优化配置
 */
@Configuration
public class JwtPerformanceConfig {
    
    /**
     * 1. 使用缓存减少重复验证
     */
    @Bean
    public CacheManager cacheManager() {
        RedisCacheManager cacheManager = RedisCacheManager.builder(redisConnectionFactory)
            .cacheDefaults(RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))  // 缓存10分钟
                .serializeValuesWith(SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer())))
            .build();
        return cacheManager;
    }
    
    /**
     * 2. 使用线程池处理 Token 验证
     */
    @Bean
    public Executor jwtValidationExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("jwt-validation-");
        executor.initialize();
        return executor;
    }
    
    /**
     * 3. 批量操作优化
     */
    @Service
    public class BatchJwtService {
        
        @Async("jwtValidationExecutor")
        public CompletableFuture<Map<String, Boolean>> batchValidate(List<String> tokens) {
            return CompletableFuture.completedFuture(
                tokens.parallelStream()
                    .collect(Collectors.toMap(
                        token -> token,
                        jwtUtil::validateToken
                    ))
            );
        }
    }
}
```

