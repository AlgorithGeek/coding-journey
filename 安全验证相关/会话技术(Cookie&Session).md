# 会话技术

## 1. 什么是会话技术？

- **会话 (Session)** 可以理解为用户与网站服务器之间的一次“交流”或“访问过程”。它从用户打开你的网站（打开第一个页面）开始，到用户关闭浏览器或离开网站一段时间后结束

- **会话技术 (Session Technology)** 就是一套用来在这次“交流”中，持续跟踪和识别同一个用户的机制。它解决了“服务器如何知道多个请求都来自同一个人”的问题



> 我理解的会话技术，就是 : **让服务器能够在同一个会话中的多个请求中建立关联的技术**



## 2. 为什么需要会话技术：HTTP的无状态性

- HTTP协议是浏览器和服务器之间通信的基础协议。它的“无状态”意味着服务器不会记录任何关于之前请求的信息。

  - 每一次HTTP请求（比如刷新页面、点击链接）都是完全独立的，服务器看待它们时，就像看待一个全新的、从未见过的客户端发来的请求一样

  - 这种设计**在早期很简单高效**，但在现代Web应用中会带来**很多麻烦**：

    - **购物车**：
      - 你把商品加入购物车（发送请求1），然后去结算（发送请求2）。
      - 如果服务器是无状态的，它在处理结算请求时，根本不知道你之前往购物车里加了什么东西

    - **用户登录**：
      - 你在登录页面输入了用户名和密码（请求1），登录成功后跳转到个人中心（请求2）。
      - 如果服务器是无状态的，它在处理访问个人中心的请求时，会忘记你刚刚已经登录过了，很可能会把你踢回登录页

  - 为了解决这个问题，会话技术应运而生。它的核心任务就是在这些独立的请求之间建立关联，让服务器能够“记住”用户的状态



## 3. 主流的会话跟踪技术

- 主要有两种会话技术来实现会话跟踪：**Cookie** 和 **Session**。它们通常是协同工作的



# Cookie技术

- Cookie 是 HTTP 协议的一部分，用于在客户端（通常是浏览器）存储少量数据，以便服务器能够“记住”关于用户或会话的信息。



## 什么是 Cookie

- **本质**：

  - Cookie 本质 上 是**服务器发送到用户浏览器**并保存在本地的一小块数据。它会在浏览器下次向同一服务器发起请求时被携带并发送到服务器上

    > HTTP协议里面没有Cookie，但是对Cookie提供了支持

- **主要用途：**
  1. **会话管理 (Session Management):** 记录用户的登录状态、购物车内容、游戏得分等。这是 Cookie 最核心的用途。
  2. **个性化 (Personalization):** 存储用户偏好，如网站主题、语言选择、显示设置等。
  3. **追踪 (Tracking):** 记录和分析用户行为，常见于广告和数据分析领域。



## Cookie 的工作原理

- Cookie 的工作流程是一个简单的“客户端-服务器”交互模型：

  1. **服务器发送 Cookie:** 

     - 如果应用程序的业务逻辑需要（例如，为了开始一个会话或记录用户信息），服务器会在其 HTTP 响应头中包含一个 `Set-Cookie` 字段来创建 Cookie

     ```http
     HTTP/1.1 200 OK
     Content-Type: text/html
     Set-Cookie: sessionId=38afes7a8; HttpOnly; Path=/
     ```

  2. **浏览器存储 Cookie:** 浏览器收到响应后，会解析 `Set-Cookie` 头，并将这个 Cookie 保存起来。

  3. **浏览器发送 Cookie:** 在之后的每一次对该网站的请求中，浏览器都会通过 HTTP 请求头中的 `Cookie` 字段，将之前保存的 Cookie 发送回服务器。

     ```http
     GET /profile HTTP/1.1
     Host: example.com
     Cookie: sessionId=38afes7a8
     ```

- 服务器通过读取请求头中的 `sessionId`，就能识别出这是哪个用户，从而提供个性化的服务或维持登录状态。



## Cookie 的核心属性详解

- 一个 Cookie 不仅仅是一个简单的键值对，它还包含多个控制其行为的属性

  | 属性             | 描述                                                         | 示例                                                   |
  | ---------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
  | **`Name=Value`** | Cookie 的名称和值（必需）。这是实际存储的数据。              | `username=JohnDoe`                                     |
  | **`Expires`**    | **过期时间（绝对时间）**。一个具体的 GMT 格式日期。若不设置，则为会话 Cookie，浏览器关闭后失效 | `Expires=Wed, 21 Oct 2025 07:28:00 GMT`                |
  | **`Max-Age`**    | **过期时间（相对时间）**。从现在开始的秒数。比 `Expires` 更现代，优先级更高。设置为 `0` 表示立即删除 | `Max-Age=3600` (1 小时后过期)                          |
  | **`Domain`**     | 指定可以接收该 Cookie 的域名。默认是当前域名                 | `Domain=.example.com` (所有子域名都可访问)             |
  | **`Path`**       | 指定可以接收该 Cookie 的 URL 路径。只有当请求路径匹配时，才会发送 Cookie。 | `Path=/admin` (只有 `/admin` 及其子路径下的请求会携带) |
  | **`Secure`**     | 一个布尔标志。如果设置了，Cookie 只会在 HTTPS 连接中被发送，可以防止中间人攻击窃取。 | `Secure`                                               |
  | **`HttpOnly`**   | 一个布尔标志。如果设置了，**客户端脚本（如 JavaScript）将无法访问该 Cookie**。这是防止跨站脚本（XSS）攻击的关键措施。 | `HttpOnly`                                             |
  | **`SameSite`**   | 控制浏览器如何处理跨站请求中的 Cookie，是防止跨站请求伪造（CSRF）攻击的重要屏障。 | `SameSite=Strict` 或 `SameSite=Lax` 或 `SameSite=None` |

  - **`SameSite` 属性详解:**

    - **`Strict`**: 最严格。完全禁止第三方请求携带 Cookie。例如，从其他网站链接点过来，也不会携带 Cookie。

    - **`Lax`** (默认值): 允许一些安全的跨站请求携带 Cookie，如 `GET` 请求(导航链接、预加载)。`POST` 等不安全的方法则会被阻止

    - **`None`**: 允许任何跨站请求携带 Cookie，但**必须同时设置 `Secure` 属性**（即只能在 HTTPS 中使用）



## SpringBoot中操作Cookie

- 在 SpringBoot 的 Controller 中，我们可以很方便地通过 `HttpServletRequest` 和 `HttpServletResponse` 来操作 Cookie。

### 1. 创建和发送 Cookie

```JAVA
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CookieController {

    @GetMapping("/set-cookie")
    public String setCookie(HttpServletResponse response) {
        // 1. 创建一个新的 Cookie 对象
        Cookie cookie = new Cookie("username", "testUser");

        // 2. 设置 Cookie 的属性
        cookie.setMaxAge(60 * 60 * 24); // 设置过期时间为 1 天 (单位: 秒)
        cookie.setPath("/"); // 设置在整个应用内有效
        cookie.setHttpOnly(true); // 设置 HttpOnly，防止脚本访问
        // cookie.setSecure(true); // 在生产环境中，推荐设置为 true，只在 HTTPS 下发送

        // 3. 将 Cookie 添加到 HTTP 响应中
        response.addCookie(cookie);

        return "Cookie has been set!";
    }
}
```



### 2. 读取 Cookie

- SpringBoot 提供了非常便捷的方式来读取 Cookie

**方式一：使用 `@CookieValue` 注解（推荐）**

- 这是最简单直接的方式，可以直接将 Cookie 的值注入到方法参数中。

  ```JAVA
  import org.springframework.web.bind.annotation.CookieValue;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  public class CookieController {
  
      @GetMapping("/read-cookie")
      public String readCookie(@CookieValue(value = "username", defaultValue = "Guest") String username) {
          // 如果名为 "username" 的 cookie 存在，它的值会被赋给 username 变量
          // 如果不存在，则使用 defaultValue 指定的默认值 "Guest"
          return "Hello, " + username;
      }
  }
  ```

  

**方式二：通过 `HttpServletRequest`**

- 这种方式更传统，可以获取到所有的 Cookie 并进行遍历

  ```JAVA
  import javax.servlet.http.Cookie;
  import javax.servlet.http.HttpServletRequest;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RestController;
  import java.util.Arrays;
  
  @RestController
  public class CookieController {
  
      @GetMapping("/read-all-cookies")
      public String readAllCookies(HttpServletRequest request) {
          Cookie[] cookies = request.getCookies();
          if (cookies != null) {
              StringBuilder sb = new StringBuilder();
              Arrays.stream(cookies).forEach(cookie -> {
                  sb.append(cookie.getName()).append("=").append(cookie.getValue()).append("\n");
              });
              return sb.toString();
          }
          return "No cookies found.";
      }
  }
  ```

  



### 3. 修改和删除 Cookie

- **修改 Cookie:** 本质上是创建一个同名的新 Cookie 并覆盖旧的。
- **删除 Cookie:** 将 Cookie 的 `Max-Age` 设置为 `0`。

```JAVA
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class CookieController {

    @GetMapping("/delete-cookie")
    public String deleteCookie(HttpServletResponse response) {
        // 创建一个同名的 Cookie
        Cookie cookie = new Cookie("username", null);
        
        // 将 Max-Age 设置为 0，通知浏览器立即删除该 Cookie
        cookie.setMaxAge(0);
        
        // 确保 Path 与要删除的 Cookie 一致
        cookie.setPath("/"); 
        
        // 将其添加到响应中
        response.addCookie(cookie);
        
        return "Cookie has been deleted.";
    }
}
```



## Cookie 的优缺点

### 优点

- **简单性和高兼容性:** 作为 HTTP 标准的一部分，所有现代浏览器都支持 Cookie。其键值对的存储方式简单直观，易于理解和使用。
- **可配置性强:** 开发者可以通过设置 `Expires`, `Path`, `Domain`, `Secure`, `HttpOnly`, `SameSite` 等属性，精细地控制 Cookie 的生命周期、作用域和安全策略。
- **无状态协议的补充:** HTTP 协议本身是无状态的，Cookie 提供了一种轻量级的机制来维持客户端和服务器之间的会话状态。

### 缺点

- **大小和数量限制:** 大多数浏览器对单个 Cookie 的大小限制在 4KB 左右，对每个域名的 Cookie 数量也有限制(通常是 50 个左右).
  这使得它不适合存储复杂的数据
- **安全风险:** Cookie 数据存储在客户端，容易被篡改或窃取
- **增加网络流量:** Cookie 在每次请求时都会被自动添加到请求头中发送到服务器，即使是请求图片、CSS 等静态资源。这会造成不必要的带宽浪费
- **隐私问题:** 第三方 Cookie 长期以来被用于跨站追踪用户行为，引发了严重的隐私担忧，目前正逐渐被主流浏览器禁用或限制
- **Cookie 不能跨域**，**哪个网站设置的 Cookie**，**就只有哪个网站（以及其指定的路径）能读取**



# Session技术

## 什么是 Session？

- **Session（会话）** 是一种在服务器端存储用户信息的机制。
  - 当一个用户首次访问应用时，服务器会为该用户创建一个唯一的 Session 对象，并生成一个与之对应的 **Session ID**。
    这个 Session ID 会通过 Cookie 发送给用户的浏览器。
    之后，浏览器在每次向该服务器发送请求时，都会自动带上这个 Session ID。
    服务器根据接收到的 Session ID，就能找到对应的 Session 对象，从而识别出用户身份和状态
- **Session 机制通常依赖于 Cookie 来实现**



## Session 的工作流程

1. **用户登录**：用户在登录页面输入用户名和密码，并提交给服务器。
2. **服务器验证**：服务器接收到请求后，验证用户名和密码是否正确。
3. **创建 Session**：验证通过后，服务器为该用户创建一个 Session 对象，并将用户的关键信息（如用户ID、角色、权限等）存入这个 Session 对象中。
4. **生成并发送 Session ID**：服务器为这个 Session 生成一个全局唯一的 Session ID（例如，通过 `JSESSIONID` 这个名字）。然后，服务器将这个 Session ID 放在 HTTP 响应的 `Set-Cookie` 头部，发送给浏览器。
5. **浏览器存储 Cookie**：浏览器收到响应后，会自动将这个包含 Session ID 的 Cookie 保存起来。
6. **后续请求**：当用户访问该网站的其他受保护页面时，浏览器会自动在 HTTP 请求的 `Cookie` 头部中带上之前保存的 Session ID。
7. **服务器验证 Session**：服务器收到后续请求后，会解析 Cookie，获取 Session ID。然后根据这个 ID 在内存（或其他存储位置）中查找对应的 Session 对象。
   - 如果找到了，说明用户已登录且会话有效，服务器就处理该请求。
   - 如果找不到（或 Session 已过期），说明用户未登录或登录超时，服务器会拒绝访问，并通常会重定向到登录页面。
8. **用户登出**：用户点击“登出”按钮时，服务器会找到对应的 Session 对象，并将其**销毁（invalidate）**。这样，即使用户的浏览器还存有旧的 Session ID，也无法再访问受保护的资源了。



## Spring Boot 中使用 Session

- 在 Spring Boot 中使用 Session 非常简单，因为它底层集成了 Servlet API，并对 `HttpSession` 提供了良好的支持

### 获取 HttpSession 对象

- 在 Controller 的方法中，你只需要将 `HttpSession` 作为参数，Spring MVC 会自动将其注入

  ```java
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RestController;
  import javax.servlet.http.HttpSession;
  
  @RestController
  public class SessionController {
  
      @GetMapping("/set")
      public String setSession(HttpSession session) {
          // 使用 session.setAttribute() 方法将数据存入 Session
          session.setAttribute("username", "admin");
          return "Session data has been set.";
      }
  
      @GetMapping("/get")
      public String getSession(HttpSession session) {
          // 使用 session.getAttribute() 方法从 Session 中获取数据
          String username = (String) session.getAttribute("username");
          if (username != null) {
              return "Username from session: " + username;
          }
          return "No session data found.";
      }
  }
  ```

  

### `HttpSession` 常用方法

```java
void 	setAttribute(String name, Object value)		//将一个对象绑定到此会话，使用指定的名称。
Object 	getAttribute(String name)					//返回与此会话中指定名称绑定的对象，如果不存在则返回 null。
void 	removeAttribute(String name)					//从此会话中移除与指定名称绑定的对象。
void 	invalidate()								//使此会话无效，然后解除绑定到它的任何对象。这是实现登出功能的关键
String 	getId()										//返回包含分配给此会话的唯一标识符的字符串
void 	setMaxInactiveInterval(int interval)		//以秒为单位指定此会话在被服务器失效之前可以保持不活动状态的时间
int 	getMaxInactiveInterval()					//返回会话在被失效之前可以保持不活动状态的最大时间间隔(以秒为单位)
```



### 配置 Session

- 你可以在 `application.properties` 或 `application.yml` 中对 Session 进行配置

- **`application.properties` 示例:**

  ```properties
  # 设置 Session 的超时时间，单位为秒。这里设置为 30 分钟。
  server.servlet.session.timeout=1800s
  
  # 配置 Cookie 的名称，默认为 JSESSIONID
  server.servlet.session.cookie.name=MYSESSIONID
  
  # 配置 Cookie 的有效期，-1 表示关闭浏览器后失效
  server.servlet.session.cookie.max-age=-1
  ```

  

- **`application.yml` 示例:**

  ```yaml
  server:
    servlet:
      session:
        timeout: 1800s # 30 分钟
        cookie:
          name: MYSESSIONID
          max-age: -1
  ```

  

## Session 认证的优缺点

### 优点

1. **简单易用**：原理和实现都相对简单，尤其是在 Spring Boot 框架下，开发者可以快速上手。
2. **安全性较高**：敏感的用户信息（如权限、用户ID）存储在服务器端，只把一个无意义的 Session ID 传给客户端，减少了敏感信息在网络中传输的风险。
3. **管理灵活**：服务器可以随时主动销毁某个用户的 Session，强制其下线，便于管理。



### 缺点

1. **占用服务器资源**
   - 每个用户的 Session 都会在服务器内存中占据一定的空间。当在线用户数量巨大时，会对服务器造成较大的内存压力
2. **不适用于分布式/集群环境**：
   - 默认情况下，Session 存储在单个服务器的内存中。如果你的应用部署在多台服务器上 ( 集群 ) ，一个用户的请求可能会被负载均衡器分发到不同的服务器
   - 如果用户的第一次请求在服务器A上创建了 Session，第二次请求被分发到了服务器B，那么服务器B的内存中没有这个用户的 Session 信息，就会导致认证失败
   - **解决方案**
     - 虽然有解决方案，比如
       - **Session 复制**（在集群中的所有服务器之间同步 Session 数据）
       - **Session 粘滞**（Sticky Session，确保同一用户的请求总是被分发到同一台服务器）
       - **Session 共享**（使用 Redis、Memcached 等外部存储来集中管理 Session），
     - 但这些都增加了系统的复杂性
3. **对移动端和前后端分离不友好**：
   - 基于 Cookie 的 Session 机制在浏览器环境中工作得很好，但对于原生移动 App 来说，它们没有内置的 Cookie 管理机制，处理起来比较麻烦
   - 在前后端分离的架构中，如果前端和后端部署在不同的域下，会遇到**跨域**问题，Cookie 的发送和接收会受到限制



# Cookie 和 Session 区别图

| 特性           | Cookie                                         | Session                                                      |
| :------------- | :--------------------------------------------- | :----------------------------------------------------------- |
| **存储位置**   | **客户端（浏览器）**                           | **服务器端**                                                 |
| **数据大小**   | 有限，通常为 **4KB** 左右                      | 理论上无限制（受服务器内存影响）                             |
| **安全性**     | **较低**。数据明文存储在客户端，易被窃取或篡改 | **较高**。数据存储在服务器，只在网络中传输无意义的 Session ID |
| **生命周期**   | 可设置较长有效期，即使用户关闭浏览器也可保留   | 通常较短，用户关闭浏览器或会话超时后即失效                   |
| **服务器压力** | **无**，不占用服务器资源                       | **有**，每个用户的 Session 都会占用服务器内存                |
| **跨域支持**   | 默认不支持跨域，但可通过配置实现               | 与 Cookie 绑定时，同样受跨域限制                             |