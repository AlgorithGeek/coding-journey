# 什么是 Filter (过滤器)？

> SpringBoot项目中更推荐使用Interceptor拦截器

- Filter 是 JavaWeb 三大组件(Servlet、Filter、Listener)之一

- 在 Java Web 应用中，Filter 是一个可以拦截和处理 **请求 (Request)** 和 **响应 (Response)** 的对象。

  - 它位于客户端和业务逻辑 (在 Spring Boot 中通常是 Controller) 之间

  - 可以把它想象成一个“检查站”。每一个进入系统的请求，都必须先经过这个检查站。程序可以在这里做一些“预处理”工作，比如：

    - **检查证件**：验证用户是否已登录 (在这里可以检验JWT)

    - **记录信息**：记录所有访问请求的日志，例如来源 IP、访问时间等

    - **统一编码**：确保所有请求都使用统一的字符编码 (如 UTF-8)

    - **数据转换**：对请求数据进行压缩或解密

- 当请求处理完毕，准备返回给客户端时，Filter 同样可以拦截到响应，进行一些“后处理”工作，比如对响应内容进行压缩、加密或者添加统一的响应头



# Filter 的生命周期

- 一个 Filter 在 Web 容器中从创建到销毁，会经历三个阶段，对应 `javax.servlet.Filter` 接口中的三个核心方法：

  1. **`init(FilterConfig filterConfig)`**:

     -  初始化方法。

     - 当 Web 容器 (如 Tomcat) 启动并加载 Filter 时，这个方法会被调用。

       - 它在 Filter 的整个生命周期中**只会被调用一次**。你可以在这里进行一些初始化的操作，比如读取配置文件、创建资源等

         

  2. **`doFilter(ServletRequest request, ServletResponse response, FilterChain chain)`**

     - 核心处理方法。

     - **每当有一个请求匹配到这个 Filter 的拦截规则时**，这个方法就会被调用。所有的过滤逻辑都在这里实现。这个方法有三个重要的参数：

       - `ServletRequest request`: 包含了客户端的请求信息

       - `ServletResponse response`: 用于向客户端发送响应

       - `FilterChain chain`: 过滤器链

         - 这是一个非常重要的对象，它的 `chain.doFilter(request, response)` 方法用于**将请求放行**，传递给下一个 Filter 或者最终的目标 (Controller)。**如果你忘记调用这个方法，请求将会被拦截在此，无法继续处理**

           

  3. **`destroy()`**: 销毁方法。

     - 当 Web 容器关闭或重新加载应用时，这个方法会被调用，用于释放 Filter 所占用的资源。它也**只会被调用一次**。



# 在 Spring Boot中创建和使用 Filter

- 在 Spring Boot 中，创建和注册一个 Filter 非常简单，只需要实现`Filter`接口

## 创建一个 Filter 类

- 首先，我们创建一个类，实现 `javax.servlet.Filter` 接口

  ```java
  import org.springframework.stereotype.Component;
  import javax.servlet.*;
  import javax.servlet.http.HttpServletRequest;
  import java.io.IOException;
  
  /**
   * 一个简单的日志过滤器，用于记录请求处理时间
   */
  @Component // 将这个 Filter 注册为 Spring Bean
  public class LoggingFilter implements Filter {
  
      @Override
      public void init(FilterConfig filterConfig) throws ServletException {
          // 可以在这里进行初始化
          System.out.println("LoggingFilter 初始化...");
      }
  
      @Override
      public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
              throws IOException, ServletException {
  
          // 预处理：记录请求开始时间
          long startTime = System.currentTimeMillis();
          HttpServletRequest httpRequest = (HttpServletRequest) request;
  
          System.out.println("请求进入: " + httpRequest.getRequestURI());
  
          // ★★★ 核心：放行请求，让它继续访问后续的 Filter 或 Controller
          chain.doFilter(request, response);
  
          // 后处理：记录请求结束时间并计算耗时
          long endTime = System.currentTimeMillis();
          System.out.println("请求 " + httpRequest.getRequestURI() + " 处理完毕，耗时: " + (endTime - startTime) + "ms");
      }
  
      @Override
      public void destroy() {
          // 可以在这里进行资源清理
          System.out.println("LoggingFilter 销毁...");
      }
  }
  ```

  

- **关键点**:

  - 我们使用了 `@Component` 注解，这是最简单的一种方式，Spring Boot 会自动扫描并注册这个 Filter。默认情况下，它会拦截所有的请求 (`/*`)

  

## (可选) 更灵活的注册方式 `FilterRegistrationBean`

- 有时候，你可能想更精确地控制 Filter 的行为，比如只拦截特定的 URL 模式，或者指定 Filter 的执行顺序。这时，使用 `FilterRegistrationBean` 会是更好的选择。

- 你需要创建一个配置类来完成注册：

  ```java
  import org.springframework.boot.web.servlet.FilterRegistrationBean;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  @Configuration
  public class FilterConfig {
  
      @Bean
      public FilterRegistrationBean<LoggingFilter> loggingFilterRegistrationBean() {
          // 1. 创建 FilterRegistrationBean
          FilterRegistrationBean<LoggingFilter> registrationBean = new FilterRegistrationBean<>();
  
          // 2. 设置自定义的 Filter
          registrationBean.setFilter(new LoggingFilter());
  
          // 3. 设置拦截的 URL 模式
          // 只拦截 /api/* 下的请求
          registrationBean.addUrlPatterns("/api/*");
  
          // 4. 设置 Filter 的执行顺序，数字越小，优先级越高
          registrationBean.setOrder(1);
  
          return registrationBean;
      }
  
      // 你可以在这里注册更多的 Filter...
  }
  ```

- 使用这种方式时，你的 `LoggingFilter` 类就不再需要 `@Component` 注解了

