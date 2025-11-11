# @ConditionalOnMissingBean 注解详解

## 📋 基本信息

- **所属包**: `org.springframework.boot.autoconfigure.condition`
- **所属框架**: Spring Boot
- **注解类型**: 条件注解（Conditional Annotation）
- **作用范围**: 类、方法

## 🎯 核心作用

`@ConditionalOnMissingBean` 是 Spring Boot 提供的条件注解，用于在 **Spring 容器中不存在指定 Bean 时**才创建新的 Bean。它是实现"约定优于配置"和提供默认配置的关键注解。

## 💡 使用场景

### 1. 提供默认配置

最常见的使用场景是在自动配置类中提供默认 Bean，允许用户自定义覆盖：

```java
@Configuration
public class DataSourceAutoConfiguration {
    
    // 只有当容器中没有 DataSource 类型的 Bean 时，才创建默认的 DataSource
    @Bean
    @ConditionalOnMissingBean(DataSource.class)
    public DataSource defaultDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:h2:mem:testdb");
        dataSource.setUsername("sa");
        dataSource.setPassword("");
        return dataSource;
    }
}
```

### 2. 自定义覆盖默认配置

用户可以通过定义自己的 Bean 来覆盖默认配置：

```java
@Configuration
public class CustomDataSourceConfig {
    
    // 用户自定义的 DataSource 会优先于默认配置
    @Bean
    public DataSource customDataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        return dataSource;
    }
}
```

## 📖 注解属性详解

### 1. value / type

指定 Bean 的类型：

```java
@Bean
@ConditionalOnMissingBean(value = RestTemplate.class)
public RestTemplate restTemplate() {
    return new RestTemplate();
}

// 或者使用 type 属性（效果相同）
@Bean
@ConditionalOnMissingBean(type = "com.example.MyService")
public MyService myService() {
    return new MyServiceImpl();
}
```

### 2. name

通过 Bean 的名称进行判断：

```java
@Bean
@ConditionalOnMissingBean(name = "myCustomBean")
public MyBean myBean() {
    return new MyBean();
}
```

### 3. annotation

检查是否存在带有特定注解的 Bean：

```java
@Bean
@ConditionalOnMissingBean(annotation = CustomAnnotation.class)
public MyService myService() {
    return new MyServiceImpl();
}
```

### 4. ignored

指定要忽略的 Bean 类型：

```java
@Bean
@ConditionalOnMissingBean(
    value = DataSource.class,
    ignored = EmbeddedDataSource.class
)
public DataSource dataSource() {
    return new HikariDataSource();
}
```

### 5. search

指定 Bean 的搜索策略：

```java
@Bean
@ConditionalOnMissingBean(
    value = ObjectMapper.class,
    search = SearchStrategy.CURRENT  // 只在当前容器中搜索
)
public ObjectMapper objectMapper() {
    return new ObjectMapper();
}
```

**SearchStrategy 枚举值：**

- `CURRENT`: 只在当前容器中搜索
- `ANCESTORS`: 在当前容器和父容器中搜索
- `ALL`: 在所有容器中搜索（默认）

## 🔍 实战示例

### 示例 1：缓存管理器配置

```java
@Configuration
@ConditionalOnClass(CacheManager.class)
public class CacheAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean(CacheManager.class)
    public CacheManager cacheManager() {
        // 提供默认的简单缓存管理器
        return new ConcurrentMapCacheManager();
    }
}

// 用户可以自定义 Redis 缓存管理器
@Configuration
public class RedisCacheConfig {
    
    @Bean
    public CacheManager redisCacheManager(RedisConnectionFactory factory) {
        return RedisCacheManager.builder(factory).build();
    }
}
```

### 示例 2：JSON 序列化配置

```java
@Configuration
public class JacksonAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        // 配置默认的序列化设置
        mapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        return mapper;
    }
}
```

### 示例 3：线程池配置

```java
@Configuration
public class ExecutorAutoConfiguration {
    
    @Bean
    @ConditionalOnMissingBean(name = "taskExecutor")
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

## ⚠️ 注意事项

### 1. Bean 创建顺序问题

```java
@Configuration
public class Config {
    
    // ❌ 错误：同一个配置类中的 Bean 创建顺序可能导致问题
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
    
    @Bean
    @ConditionalOnMissingBean(MyService.class)
    public MyService defaultMyService() {
        // 这个方法可能不会按预期工作
        return new DefaultMyServiceImpl();
    }
}

// ✅ 正确：将默认配置和用户配置分离
@Configuration
@AutoConfigureBefore(UserConfiguration.class)
public class DefaultConfiguration {
    
    @Bean
    @ConditionalOnMissingBean(MyService.class)
    public MyService defaultMyService() {
        return new DefaultMyServiceImpl();
    }
}

@Configuration
public class UserConfiguration {
    
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

### 2. 泛型 Bean 的判断

```java
@Bean
@ConditionalOnMissingBean  // 不指定类型时，使用方法返回类型
public List<String> myStringList() {
    return Arrays.asList("a", "b", "c");
}
```

### 3. 与其他条件注解组合使用

```java
@Bean
@ConditionalOnClass(RedisTemplate.class)
@ConditionalOnMissingBean(RedisTemplate.class)
@ConditionalOnProperty(name = "spring.redis.enabled", havingValue = "true")
public RedisTemplate<String, Object> redisTemplate(
        RedisConnectionFactory connectionFactory) {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(connectionFactory);
    return template;
}
```

## 🔧 底层实现原理

`@ConditionalOnMissingBean` 的核心实现类是 `OnBeanCondition`，它实现了 Spring 的 `Condition` 接口：

```java
// 简化的实现逻辑
public class OnBeanCondition implements Condition {
    
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 1. 获取注解属性
        MultiValueMap<String, Object> attributes = metadata.getAllAnnotationAttributes(
            ConditionalOnMissingBean.class.getName());
        
        // 2. 获取 BeanFactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        
        // 3. 检查容器中是否存在指定的 Bean
        // 如果不存在，返回 true，创建该 Bean
        // 如果存在，返回 false，不创建该 Bean
        return !beanExists(beanFactory, attributes);
    }
}
```

## 📚 相关注解对比

| 注解                        | 作用               | 使用场景      |
| --------------------------- | ------------------ | ------------- |
| `@ConditionalOnMissingBean` | Bean 不存在时生效  | 提供默认配置  |
| `@ConditionalOnBean`        | Bean 存在时生效    | 依赖其他 Bean |
| `@ConditionalOnClass`       | 类存在时生效       | 依赖特定库    |
| `@ConditionalOnProperty`    | 配置属性满足时生效 | 基于配置开关  |

## 💼 最佳实践

1. **优先使用类型判断**：尽量使用 `value` 或 `type` 属性，而不是 `name`
2. **注意加载顺序**：使用 `@AutoConfigureBefore` 或 `@AutoConfigureAfter` 控制配置类的加载顺序
3. **明确搜索范围**：根据需要设置合适的 `search` 策略
4. **文档说明**：在提供默认配置时，添加清晰的注释说明如何覆盖
5. **避免循环依赖**：注意配置类之间的依赖关系

## 🎓 总结

`@ConditionalOnMissingBean` 是 Spring Boot 自动配置的核心注解之一，它让框架能够提供开箱即用的默认配置，同时又保持高度的可定制性。理解并正确使用这个注解，是掌握 Spring Boot 自动配置机制的关键。