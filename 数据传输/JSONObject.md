# Fastjson JSONObject 全面学习笔记

## 一、Fastjson 概述

### 1.1 什么是 Fastjson

Fastjson 是阿里巴巴开源的高性能 JSON 解析库，用于 Java 对象与 JSON 字符串之间的相互转换。它以速度快、使用简单著称，是国内使用最广泛的 JSON 处理库之一。

### 1.2 核心特点

- **速度快**：采用独创的算法，解析速度极快
- **功能完善**：支持泛型、复杂类型的序列化与反序列化
- **使用简单**：API 设计简洁直观
- **无依赖**：不需要额外的 jar 包

### 1.3 版本选择

Fastjson 目前有两个主要版本：

| 版本         | 包名                    | 说明                               |
| ------------ | ----------------------- | ---------------------------------- |
| Fastjson 1.x | `com.alibaba.fastjson`  | 经典版本，使用广泛但有安全漏洞历史 |
| Fastjson 2.x | `com.alibaba.fastjson2` | 全新重写版本，更安全、性能更好     |

> ⚠️ **建议**：新项目推荐使用 Fastjson 2.x 版本，旧项目建议升级。

------

## 二、Maven 依赖配置

### 2.1 Fastjson 1.x（经典版本）

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.83</version>
</dependency>
```

### 2.2 Fastjson 2.x（推荐版本）

```xml
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2</artifactId>
    <version>2.0.43</version>
</dependency>
```

### 2.3 Fastjson 2.x 兼容包

如果你的项目使用 Fastjson 1.x API，可以使用兼容包平滑迁移：

```xml
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2-extension</artifactId>
    <version>2.0.43</version>
</dependency>
```

------

## 三、核心类介绍

### 3.1 类层次结构

```
java.util.Map
    └── java.util.HashMap
            └── com.alibaba.fastjson.JSONObject
                    
java.util.List
    └── java.util.ArrayList
            └── com.alibaba.fastjson.JSONArray
```

### 3.2 三个核心类

| 类名         | 说明                                    | 对应 JSON 结构 |
| ------------ | --------------------------------------- | -------------- |
| `JSON`       | 工具类，提供静态方法进行序列化/反序列化 | -              |
| `JSONObject` | 表示 JSON 对象，继承自 HashMap          | `{ }`          |
| `JSONArray`  | 表示 JSON 数组，继承自 ArrayList        | `[ ]`          |

------

## 四、JSONObject 基础操作

### 4.1 创建 JSONObject

#### 4.1.1 使用无参构造器创建空对象

```java
import com.alibaba.fastjson.JSONObject;

// 创建空的 JSONObject
JSONObject jsonObject = new JSONObject();
```

#### 4.1.2 使用 Map 创建

```java
Map<String, Object> map = new HashMap<>();
map.put("name", "张三");
map.put("age", 25);
map.put("city", "北京");

JSONObject jsonObject = new JSONObject(map);
System.out.println(jsonObject);
// 输出: {"name":"张三","age":25,"city":"北京"}
```

#### 4.1.3 使用有序 Map 创建（保持插入顺序）

```java
// 构造器参数 true 表示使用 LinkedHashMap，保持插入顺序
JSONObject jsonObject = new JSONObject(true);
jsonObject.put("id", 1);
jsonObject.put("name", "张三");
jsonObject.put("age", 25);
// 输出时会保持 id -> name -> age 的顺序
```

#### 4.1.4 从 JSON 字符串解析创建

```java
String jsonStr = "{\"name\":\"张三\",\"age\":25,\"city\":\"北京\"}";

// 方式一：使用 JSON.parseObject()
JSONObject jsonObject1 = JSON.parseObject(jsonStr);

// 方式二：使用 JSONObject.parseObject()
JSONObject jsonObject2 = JSONObject.parseObject(jsonStr);
```

#### 4.1.5 从 Java 对象转换创建

```java
User user = new User("张三", 25, "北京");

// 将 Java 对象转换为 JSONObject
JSONObject jsonObject = (JSONObject) JSON.toJSON(user);
```

### 4.2 添加数据 - put() 方法

```java
JSONObject jsonObject = new JSONObject();

// 添加基本类型
jsonObject.put("name", "张三");           // String
jsonObject.put("age", 25);                // Integer
jsonObject.put("salary", 15000.50);       // Double
jsonObject.put("isActive", true);         // Boolean
jsonObject.put("birthday", new Date());   // Date

// 添加 null 值
jsonObject.put("nickname", null);

// 添加嵌套对象
JSONObject address = new JSONObject();
address.put("city", "北京");
address.put("district", "海淀区");
jsonObject.put("address", address);

// 添加数组
JSONArray hobbies = new JSONArray();
hobbies.add("读书");
hobbies.add("游泳");
hobbies.add("编程");
jsonObject.put("hobbies", hobbies);

System.out.println(jsonObject.toJSONString());
```

输出结果：

```json
{
  "name": "张三",
  "age": 25,
  "salary": 15000.50,
  "isActive": true,
  "birthday": "2024-01-15 10:30:00",
  "nickname": null,
  "address": {
    "city": "北京",
    "district": "海淀区"
  },
  "hobbies": ["读书", "游泳", "编程"]
}
```

### 4.3 链式调用 - fluentPut()

```java
JSONObject jsonObject = new JSONObject()
    .fluentPut("name", "张三")
    .fluentPut("age", 25)
    .fluentPut("city", "北京")
    .fluentPut("active", true);

System.out.println(jsonObject);
// 输出: {"name":"张三","age":25,"city":"北京","active":true}
```

------

## 五、获取数据方法详解

### 5.1 通用获取方法

#### 5.1.1 get() - 返回 Object 类型

```java
JSONObject json = JSONObject.parseObject("{\"name\":\"张三\",\"age\":25}");

Object name = json.get("name");        // 返回 Object
Object notExist = json.get("xxx");     // 返回 null
```

#### 5.1.2 containsKey() - 判断 key 是否存在

```java
boolean hasName = json.containsKey("name");  // true
boolean hasXxx = json.containsKey("xxx");    // false
```

### 5.2 类型安全的获取方法

JSONObject 提供了一系列类型安全的 getXxx() 方法：

#### 5.2.1 getString() - 获取字符串

```java
JSONObject json = JSONObject.parseObject("{\"name\":\"张三\",\"age\":25,\"score\":98.5}");

String name = json.getString("name");      // "张三"
String age = json.getString("age");        // "25" (数字会转为字符串)
String score = json.getString("score");    // "98.5"
String notExist = json.getString("xxx");   // null
```

#### 5.2.2 getInteger() / getIntValue() - 获取整数

```java
JSONObject json = JSONObject.parseObject("{\"age\":25,\"score\":\"100\"}");

// getInteger() - 返回 Integer 对象，可能为 null
Integer age1 = json.getInteger("age");        // 25
Integer notExist1 = json.getInteger("xxx");   // null

// getIntValue() - 返回 int 基本类型，不存在时返回 0
int age2 = json.getIntValue("age");           // 25
int notExist2 = json.getIntValue("xxx");      // 0

// 字符串数字会自动转换
Integer score = json.getInteger("score");     // 100
```

#### 5.2.3 getLong() / getLongValue() - 获取长整数

```java
JSONObject json = JSONObject.parseObject("{\"id\":123456789012345}");

Long id1 = json.getLong("id");           // 123456789012345L
long id2 = json.getLongValue("id");      // 123456789012345L
long notExist = json.getLongValue("xxx"); // 0L
```

#### 5.2.4 getDouble() / getDoubleValue() - 获取浮点数

```java
JSONObject json = JSONObject.parseObject("{\"price\":99.99}");

Double price1 = json.getDouble("price");        // 99.99
double price2 = json.getDoubleValue("price");   // 99.99
double notExist = json.getDoubleValue("xxx");   // 0.0
```

#### 5.2.5 getBigDecimal() - 获取精确小数（推荐用于金额）

```java
JSONObject json = JSONObject.parseObject("{\"amount\":12345.67}");

BigDecimal amount = json.getBigDecimal("amount");  // 12345.67
// 适用于金融计算，避免浮点数精度问题
```

#### 5.2.6 getBoolean() / getBooleanValue() - 获取布尔值

```java
JSONObject json = JSONObject.parseObject("{\"active\":true,\"deleted\":\"false\",\"flag\":1}");

Boolean active1 = json.getBoolean("active");       // true
boolean active2 = json.getBooleanValue("active");  // true

// 字符串 "true"/"false" 会自动转换
Boolean deleted = json.getBoolean("deleted");      // false

// 数字 1/0 也会转换（1=true, 0=false）
Boolean flag = json.getBoolean("flag");            // true

// 不存在时
Boolean notExist1 = json.getBoolean("xxx");        // null
boolean notExist2 = json.getBooleanValue("xxx");   // false
```

#### 5.2.7 getDate() - 获取日期

```java
JSONObject json = JSONObject.parseObject("{\"createTime\":\"2024-01-15 10:30:00\",\"timestamp\":1705289400000}");

// 支持字符串格式的日期
Date createTime = json.getDate("createTime");

// 支持时间戳（毫秒）
Date timestamp = json.getDate("timestamp");
```

### 5.3 获取嵌套对象和数组

#### 5.3.1 getJSONObject() - 获取嵌套对象

```java
String jsonStr = "{\"user\":{\"name\":\"张三\",\"age\":25}}";
JSONObject json = JSONObject.parseObject(jsonStr);

JSONObject user = json.getJSONObject("user");
String name = user.getString("name");  // "张三"
int age = user.getIntValue("age");     // 25
```

#### 5.3.2 getJSONArray() - 获取数组

```java
String jsonStr = "{\"scores\":[85,90,95],\"users\":[{\"name\":\"张三\"},{\"name\":\"李四\"}]}";
JSONObject json = JSONObject.parseObject(jsonStr);

// 获取简单数组
JSONArray scores = json.getJSONArray("scores");
for (int i = 0; i < scores.size(); i++) {
    System.out.println(scores.getInteger(i));
}

// 获取对象数组
JSONArray users = json.getJSONArray("users");
for (int i = 0; i < users.size(); i++) {
    JSONObject user = users.getJSONObject(i);
    System.out.println(user.getString("name"));
}
```

### 5.4 获取方法对比总结

| 方法                | 返回类型   | key 不存在时 | 值为 null 时 |
| ------------------- | ---------- | ------------ | ------------ |
| `getString()`       | String     | null         | null         |
| `getInteger()`      | Integer    | null         | null         |
| `getIntValue()`     | int        | 0            | 0            |
| `getLong()`         | Long       | null         | null         |
| `getLongValue()`    | long       | 0L           | 0L           |
| `getDouble()`       | Double     | null         | null         |
| `getDoubleValue()`  | double     | 0.0          | 0.0          |
| `getBoolean()`      | Boolean    | null         | null         |
| `getBooleanValue()` | boolean    | false        | false        |
| `getBigDecimal()`   | BigDecimal | null         | null         |
| `getDate()`         | Date       | null         | null         |
| `getJSONObject()`   | JSONObject | null         | null         |
| `getJSONArray()`    | JSONArray  | null         | null         |

------

## 六、JSONObject 与 Java 对象互转

### 6.1 准备实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
    private Date createTime;
    private List<String> hobbies;
    private Address address;
}

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Address {
    private String city;
    private String district;
    private String street;
}
```

### 6.2 Java 对象转 JSON 字符串

```java
User user = new User();
user.setId(1L);
user.setName("张三");
user.setAge(25);
user.setEmail("zhangsan@example.com");
user.setCreateTime(new Date());
user.setHobbies(Arrays.asList("读书", "游泳"));
user.setAddress(new Address("北京", "海淀区", "中关村大街"));

// 方式一：转为 JSON 字符串
String jsonStr = JSON.toJSONString(user);
System.out.println(jsonStr);

// 方式二：转为格式化的 JSON 字符串（便于阅读）
String prettyJson = JSON.toJSONString(user, SerializerFeature.PrettyFormat);
System.out.println(prettyJson);

// 方式三：转为 JSONObject
JSONObject jsonObject = (JSONObject) JSON.toJSON(user);
```

### 6.3 JSON 字符串转 Java 对象

```java
String jsonStr = "{\"id\":1,\"name\":\"张三\",\"age\":25,\"email\":\"zhangsan@example.com\"}";

// 转为 Java 对象
User user = JSON.parseObject(jsonStr, User.class);
System.out.println(user.getName());  // 张三
```

### 6.4 JSONObject 转 Java 对象

```java
JSONObject jsonObject = new JSONObject();
jsonObject.put("id", 1L);
jsonObject.put("name", "张三");
jsonObject.put("age", 25);

// 方式一：使用 toJavaObject()
User user1 = jsonObject.toJavaObject(User.class);

// 方式二：使用 JSON.toJavaObject()
User user2 = JSON.toJavaObject(jsonObject, User.class);

// 方式三：先转字符串再解析
User user3 = JSON.parseObject(jsonObject.toJSONString(), User.class);
```

### 6.5 List 与 JSONArray 互转

```java
// List 转 JSON 字符串
List<User> userList = Arrays.asList(
    new User(1L, "张三", 25, null, null, null, null),
    new User(2L, "李四", 30, null, null, null, null)
);
String jsonArrayStr = JSON.toJSONString(userList);

// JSON 字符串转 List
List<User> users = JSON.parseArray(jsonArrayStr, User.class);

// JSONArray 转 List
JSONArray jsonArray = JSON.parseArray(jsonArrayStr);
List<User> users2 = jsonArray.toJavaList(User.class);
```

------

## 七、JSONArray 详解

### 7.1 创建 JSONArray

```java
// 方式一：创建空数组
JSONArray array1 = new JSONArray();

// 方式二：从 List 创建
List<String> list = Arrays.asList("Java", "Python", "Go");
JSONArray array2 = new JSONArray(list);

// 方式三：从 JSON 字符串解析
String jsonStr = "[1, 2, 3, 4, 5]";
JSONArray array3 = JSON.parseArray(jsonStr);

// 方式四：解析对象数组
String userArrayStr = "[{\"name\":\"张三\"},{\"name\":\"李四\"}]";
JSONArray array4 = JSON.parseArray(userArrayStr);
```

### 7.2 JSONArray 常用方法

```java
JSONArray array = new JSONArray();

// 添加元素
array.add("元素1");
array.add(100);
array.add(new JSONObject().fluentPut("key", "value"));

// 获取元素
Object obj = array.get(0);
String str = array.getString(0);
Integer num = array.getInteger(1);
JSONObject jsonObj = array.getJSONObject(2);

// 获取大小
int size = array.size();

// 判断是否为空
boolean empty = array.isEmpty();

// 遍历
for (int i = 0; i < array.size(); i++) {
    System.out.println(array.get(i));
}

// 使用 forEach
array.forEach(item -> System.out.println(item));

// 转换为 List
List<String> stringList = array.toJavaList(String.class);
```

### 7.3 JSONArray 与 Stream API 结合

```java
String jsonStr = "[{\"name\":\"张三\",\"age\":25},{\"name\":\"李四\",\"age\":30},{\"name\":\"王五\",\"age\":28}]";
JSONArray users = JSON.parseArray(jsonStr);

// 筛选年龄大于 25 的用户
List<JSONObject> filtered = users.stream()
    .map(obj -> (JSONObject) obj)
    .filter(user -> user.getIntValue("age") > 25)
    .collect(Collectors.toList());

// 提取所有用户名
List<String> names = users.stream()
    .map(obj -> ((JSONObject) obj).getString("name"))
    .collect(Collectors.toList());

// 计算平均年龄
double avgAge = users.stream()
    .map(obj -> (JSONObject) obj)
    .mapToInt(user -> user.getIntValue("age"))
    .average()
    .orElse(0);
```

------

## 八、序列化特性（SerializerFeature）

### 8.1 常用序列化特性

```java
User user = new User();
user.setName("张三");
// age、email 等字段为 null

// 默认情况下，null 值字段不会输出
String json1 = JSON.toJSONString(user);
// {"name":"张三"}

// 输出 null 值字段
String json2 = JSON.toJSONString(user, SerializerFeature.WriteMapNullValue);
// {"name":"张三","age":null,"email":null,...}
```

### 8.2 SerializerFeature 枚举详解

| 特性                             | 说明                            |
| -------------------------------- | ------------------------------- |
| `WriteMapNullValue`              | 输出值为 null 的字段            |
| `WriteNullStringAsEmpty`         | 将 null 的 String 输出为 ""     |
| `WriteNullListAsEmpty`           | 将 null 的 List 输出为 []       |
| `WriteNullNumberAsZero`          | 将 null 的 Number 输出为 0      |
| `WriteNullBooleanAsFalse`        | 将 null 的 Boolean 输出为 false |
| `PrettyFormat`                   | 格式化输出（带缩进）            |
| `WriteDateUseDateFormat`         | 使用默认日期格式输出 Date       |
| `WriteClassName`                 | 输出类名（不推荐，有安全风险）  |
| `DisableCircularReferenceDetect` | 禁用循环引用检测                |
| `WriteEnumUsingName`             | 枚举使用 name() 输出            |
| `WriteEnumUsingToString`         | 枚举使用 toString() 输出        |

### 8.3 组合使用多个特性

```java
String json = JSON.toJSONString(user, 
    SerializerFeature.WriteMapNullValue,
    SerializerFeature.WriteNullStringAsEmpty,
    SerializerFeature.WriteNullListAsEmpty,
    SerializerFeature.PrettyFormat
);
```

### 8.4 日期格式化

```java
User user = new User();
user.setCreateTime(new Date());

// 方式一：使用 SerializerFeature
String json1 = JSON.toJSONString(user, SerializerFeature.WriteDateUseDateFormat);
// 输出: "createTime":"2024-01-15 10:30:00"

// 方式二：指定格式
String json2 = JSON.toJSONStringWithDateFormat(user, "yyyy-MM-dd HH:mm:ss");

// 方式三：使用注解（推荐）
public class User {
    @JSONField(format = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;
}
```

------

## 九、@JSONField 注解详解

### 9.1 注解属性

```java
public @interface JSONField {
    String name() default "";           // 指定 JSON 中的字段名
    String format() default "";         // 日期格式
    boolean serialize() default true;   // 是否参与序列化
    boolean deserialize() default true; // 是否参与反序列化
    int ordinal() default 0;            // 字段排序
    String defaultValue() default "";   // 默认值
}
```

### 9.2 使用示例

```java
@Data
public class User {
    
    // 指定 JSON 字段名
    @JSONField(name = "user_name")
    private String name;
    
    // 日期格式化
    @JSONField(format = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;
    
    // 不参与序列化（不输出到 JSON）
    @JSONField(serialize = false)
    private String password;
    
    // 不参与反序列化（不从 JSON 读取）
    @JSONField(deserialize = false)
    private String internalCode;
    
    // 指定序列化顺序
    @JSONField(ordinal = 1)
    private Long id;
    
    @JSONField(ordinal = 2)
    private String email;
}
```

### 9.3 字段名映射示例

```java
@Data
public class ApiResponse {
    
    @JSONField(name = "error_code")
    private Integer errorCode;
    
    @JSONField(name = "error_msg")
    private String errorMsg;
    
    @JSONField(name = "data")
    private Object data;
}

// 使用
String json = "{\"error_code\":0,\"error_msg\":\"success\",\"data\":{}}";
ApiResponse response = JSON.parseObject(json, ApiResponse.class);
System.out.println(response.getErrorCode());  // 0
```

------

## 十、高级特性

### 10.1 泛型处理 - TypeReference

当需要反序列化带泛型的类型时，使用 `TypeReference`：

```java
// 解析 List<User>
String jsonStr = "[{\"name\":\"张三\"},{\"name\":\"李四\"}]";
List<User> userList = JSON.parseObject(jsonStr, new TypeReference<List<User>>() {});

// 解析 Map<String, User>
String mapJson = "{\"user1\":{\"name\":\"张三\"},\"user2\":{\"name\":\"李四\"}}";
Map<String, User> userMap = JSON.parseObject(mapJson, new TypeReference<Map<String, User>>() {});

// 解析复杂嵌套泛型
String complexJson = "{\"code\":200,\"data\":[{\"name\":\"张三\"}]}";
Result<List<User>> result = JSON.parseObject(complexJson, new TypeReference<Result<List<User>>>() {});
```

### 10.2 处理复杂嵌套 JSON

```java
String complexJson = """
    {
        "code": 200,
        "message": "success",
        "data": {
            "users": [
                {
                    "id": 1,
                    "name": "张三",
                    "department": {
                        "id": 100,
                        "name": "技术部"
                    }
                }
            ],
            "total": 1
        }
    }
    """;

JSONObject json = JSON.parseObject(complexJson);

// 层层获取
int code = json.getIntValue("code");
JSONObject data = json.getJSONObject("data");
JSONArray users = data.getJSONArray("users");
JSONObject firstUser = users.getJSONObject(0);
String userName = firstUser.getString("name");
JSONObject dept = firstUser.getJSONObject("department");
String deptName = dept.getString("name");

System.out.println("用户: " + userName + ", 部门: " + deptName);
// 输出: 用户: 张三, 部门: 技术部
```

### 10.3 JSONPath 表达式

JSONPath 提供了类似 XPath 的方式来访问 JSON 数据：

```java
String json = """
    {
        "store": {
            "book": [
                {"title": "Java入门", "price": 49.9},
                {"title": "Spring实战", "price": 79.9},
                {"title": "MySQL高性能", "price": 89.9}
            ],
            "bicycle": {
                "color": "red",
                "price": 199.9
            }
        }
    }
    """;

Object obj = JSON.parse(json);

// 获取所有书籍的标题
List<String> titles = (List<String>) JSONPath.eval(obj, "$.store.book[*].title");
// [Java入门, Spring实战, MySQL高性能]

// 获取第一本书
Object firstBook = JSONPath.eval(obj, "$.store.book[0]");

// 获取价格大于 50 的书
List<Object> expensiveBooks = (List<Object>) JSONPath.eval(obj, "$.store.book[?(@.price > 50)]");

// 获取所有价格
List<Double> allPrices = (List<Double>) JSONPath.eval(obj, "$..price");
```

### 10.4 循环引用处理

```java
// 当对象存在循环引用时
public class Department {
    private String name;
    private List<Employee> employees;
}

public class Employee {
    private String name;
    private Department department;  // 循环引用
}

// 默认会使用 $ref 表示循环引用
// {"name":"技术部","employees":[{"name":"张三","department":{"$ref":".."}}]}

// 禁用循环引用检测
String json = JSON.toJSONString(dept, SerializerFeature.DisableCircularReferenceDetect);
// 注意：如果真的存在循环引用，会导致 StackOverflowError
```

------

## 十一、在 Spring Boot 中的应用

### 11.1 配置 Fastjson 作为默认 JSON 处理器

```java
@Configuration
public class FastjsonConfig {
    
    @Bean
    public HttpMessageConverters fastjsonHttpMessageConverters() {
        // 创建 FastJson 消息转换器
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        
        // 配置
        FastJsonConfig config = new FastJsonConfig();
        config.setSerializerFeatures(
            SerializerFeature.WriteMapNullValue,
            SerializerFeature.WriteDateUseDateFormat,
            SerializerFeature.PrettyFormat
        );
        config.setDateFormat("yyyy-MM-dd HH:mm:ss");
        
        converter.setFastJsonConfig(config);
        
        // 设置支持的媒体类型
        List<MediaType> mediaTypes = new ArrayList<>();
        mediaTypes.add(MediaType.APPLICATION_JSON);
        converter.setSupportedMediaTypes(mediaTypes);
        
        return new HttpMessageConverters(converter);
    }
}
```

### 11.2 Controller 中的使用

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public Result<User> createUser(@RequestBody JSONObject requestBody) {
        // 直接接收 JSONObject
        String name = requestBody.getString("name");
        Integer age = requestBody.getIntValue("age");
        
        // 也可以转换为实体类
        User user = requestBody.toJavaObject(User.class);
        
        // 处理业务逻辑...
        
        return Result.success(user);
    }
    
    @GetMapping("/{id}")
    public JSONObject getUser(@PathVariable Long id) {
        // 直接返回 JSONObject
        JSONObject result = new JSONObject();
        result.put("id", id);
        result.put("name", "张三");
        result.put("createTime", new Date());
        return result;
    }
}
```

### 11.3 处理第三方 API 响应

```java
@Service
public class ThirdPartyApiService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    public User getUserFromApi(Long userId) {
        String url = "https://api.example.com/users/" + userId;
        String response = restTemplate.getForObject(url, String.class);
        
        JSONObject json = JSON.parseObject(response);
        
        // 检查响应状态
        if (json.getIntValue("code") != 200) {
            throw new RuntimeException(json.getString("message"));
        }
        
        // 提取数据
        JSONObject data = json.getJSONObject("data");
        return data.toJavaObject(User.class);
    }
}
```

------

## 十二、常见问题与解决方案

### 12.1 日期格式问题

**问题**：日期字段序列化后格式不对

```java
// 解决方案一：全局配置
JSON.DEFFAULT_DATE_FORMAT = "yyyy-MM-dd HH:mm:ss";
String json = JSON.toJSONString(user, SerializerFeature.WriteDateUseDateFormat);

// 解决方案二：注解配置（推荐）
@JSONField(format = "yyyy-MM-dd HH:mm:ss")
private Date createTime;

// 解决方案三：手动转换
String json = JSON.toJSONStringWithDateFormat(user, "yyyy-MM-dd HH:mm:ss");
```

### 12.2 字段顺序问题

**问题**：JSON 输出的字段顺序不固定

```java
// 解决方案一：使用有序 JSONObject
JSONObject json = new JSONObject(true);  // 使用 LinkedHashMap

// 解决方案二：使用 @JSONField ordinal 属性
@JSONField(ordinal = 1)
private Long id;

@JSONField(ordinal = 2)
private String name;
```

### 12.3 特殊字符转义问题

```java
String text = "Hello \"World\"";
JSONObject json = new JSONObject();
json.put("text", text);

// 默认会正确转义
System.out.println(json.toJSONString());
// {"text":"Hello \"World\""}
```

### 12.4 Long 类型精度丢失问题

**问题**：前端 JavaScript 处理大数字 Long 时精度丢失

```java
// 解决方案：将 Long 序列化为 String
public class User {
    @JSONField(serializeUsing = ToStringSerializer.class)
    private Long id;
}

// 或者使用全局配置
SerializeConfig config = new SerializeConfig();
config.put(Long.class, ToStringSerializer.instance);
config.put(Long.TYPE, ToStringSerializer.instance);
String json = JSON.toJSONString(user, config);
```

### 12.5 null 值处理

```java
JSONObject json = new JSONObject();
json.put("name", null);

// 获取时判断
String name = json.getString("name");  // 返回 null

// 提供默认值
String nameWithDefault = json.getString("name") != null ? json.getString("name") : "默认值";

// 或使用 Optional
Optional.ofNullable(json.getString("name")).orElse("默认值");
```

### 12.6 枚举类型处理

```java
public enum Status {
    ACTIVE(1, "激活"),
    INACTIVE(0, "未激活");
    
    private int code;
    private String desc;
    // getter...
}

// 默认序列化为枚举名称
// {"status":"ACTIVE"}

// 序列化为 ordinal
@JSONField(serializeUsing = WriteEnumUsingToString.class)
private Status status;

// 自定义序列化
public enum Status {
    ACTIVE(1, "激活"),
    INACTIVE(0, "未激活");
    
    @JSONField
    private int code;
    
    private String desc;
}
// {"status":{"code":1}}
```

------

## 十三、性能优化建议

### 13.1 避免重复解析

```java
// 不推荐：多次解析同一个 JSON 字符串
String jsonStr = "...";
String name = JSON.parseObject(jsonStr).getString("name");
Integer age = JSON.parseObject(jsonStr).getInteger("age");

// 推荐：解析一次，多次使用
JSONObject json = JSON.parseObject(jsonStr);
String name = json.getString("name");
Integer age = json.getInteger("age");
```

### 13.2 使用合适的数据类型

```java
// 如果只需要读取 JSON 数据
JSONObject json = JSON.parseObject(jsonStr);

// 如果需要操作实体对象
User user = JSON.parseObject(jsonStr, User.class);

// 如果只需要特定字段，使用 JSONPath
String name = (String) JSONPath.eval(JSON.parse(jsonStr), "$.name");
```

### 13.3 大数据量处理

```java
// 对于大型 JSON 数组，使用 Stream 处理
String largeJsonArray = "[...]";  // 大量数据
JSONArray array = JSON.parseArray(largeJsonArray);

array.stream()
    .parallel()  // 并行处理
    .map(obj -> (JSONObject) obj)
    .filter(json -> json.getIntValue("age") > 18)
    .forEach(json -> processUser(json));
```

------

## 十四、Fastjson 2.x 新特性

### 14.1 主要改进

- **更安全**：默认关闭 autoType，避免反序列化漏洞
- **性能提升**：重新设计的解析器，性能更优
- **更好的标准兼容性**：更好地支持 JSON 标准
- **新的 API 设计**：更简洁的 API

### 14.2 主要 API 变化

```java
// Fastjson 2.x 推荐用法
import com.alibaba.fastjson2.JSON;
import com.alibaba.fastjson2.JSONObject;
import com.alibaba.fastjson2.JSONArray;

// 基本使用方式相同
JSONObject json = JSON.parseObject(jsonStr);
String str = JSON.toJSONString(obj);

// 新增的便捷方法
JSONObject json = JSONObject.of("name", "张三", "age", 25);

// 更简洁的泛型支持
List<User> users = JSON.parseObject(jsonStr, new TypeReference<List<User>>() {});
```

------

## 十五、最佳实践总结

### 15.1 推荐做法

1. **新项目使用 Fastjson 2.x**，享受更好的性能和安全性
2. **使用 @JSONField 注解**进行字段映射和格式化
3. **使用 TypeReference**处理泛型类型
4. **使用 getXxxValue()**方法避免 NPE（返回默认值）
5. **使用链式调用 fluentPut()**构建 JSONObject
6. **大数字使用 BigDecimal**避免精度丢失
7. **Long 类型考虑转 String**传给前端

### 15.2 避免的做法

1. **避免使用 WriteClassName**特性（安全风险）
2. **避免在循环中重复解析**同一 JSON 字符串
3. **避免忽略 null 值检查**
4. **避免使用过时的 API**

### 15.3 安全建议

```java
// 禁用 autoType（Fastjson 2.x 默认已禁用）
ParserConfig.getGlobalInstance().setSafeMode(true);

// 或在解析时指定类型白名单
ParserConfig.getGlobalInstance().addAccept("com.mycompany.model.");
```

------

## 十六、与其他 JSON 库对比

| 特性        | Fastjson | Jackson  | Gson   |
| ----------- | -------- | -------- | ------ |
| 性能        | 极快     | 快       | 中等   |
| API 简洁性  | 优秀     | 一般     | 良好   |
| 功能完整性  | 完整     | 非常完整 | 完整   |
| Spring 支持 | 需配置   | 默认支持 | 需配置 |
| 社区活跃度  | 活跃     | 非常活跃 | 活跃   |
| 安全性      | 曾有漏洞 | 良好     | 良好   |

------

## 附录：常用方法速查表

### JSONObject 常用方法

| 方法                                       | 说明                  |
| ------------------------------------------ | --------------------- |
| `put(key, value)`                          | 添加键值对            |
| `fluentPut(key, value)`                    | 链式添加键值对        |
| `get(key)`                                 | 获取值（返回 Object） |
| `getString(key)`                           | 获取字符串            |
| `getInteger(key)` / `getIntValue(key)`     | 获取整数              |
| `getLong(key)` / `getLongValue(key)`       | 获取长整数            |
| `getDouble(key)` / `getDoubleValue(key)`   | 获取浮点数            |
| `getBoolean(key)` / `getBooleanValue(key)` | 获取布尔值            |
| `getBigDecimal(key)`                       | 获取 BigDecimal       |
| `getDate(key)`                             | 获取日期              |
| `getJSONObject(key)`                       | 获取嵌套对象          |
| `getJSONArray(key)`                        | 获取数组              |
| `containsKey(key)`                         | 判断 key 是否存在     |
| `remove(key)`                              | 移除键值对            |
| `size()`                                   | 获取键值对数量        |
| `isEmpty()`                                | 判断是否为空          |
| `toJSONString()`                           | 转为 JSON 字符串      |
| `toJavaObject(Class)`                      | 转为 Java 对象        |

### JSON 工具类常用方法

| 方法                             | 说明                    |
| -------------------------------- | ----------------------- |
| `JSON.parseObject(str)`          | 解析为 JSONObject       |
| `JSON.parseObject(str, Class)`   | 解析为 Java 对象        |
| `JSON.parseArray(str)`           | 解析为 JSONArray        |
| `JSON.parseArray(str, Class)`    | 解析为 List             |
| `JSON.toJSONString(obj)`         | 转为 JSON 字符串        |
| `JSON.toJSON(obj)`               | 转为 JSON 对象          |
| `JSON.toJavaObject(json, Class)` | JSONObject 转 Java 对象 |

------

> 📝 **文档信息**
>
> - 适用版本：Fastjson 1.2.x / Fastjson 2.x
> - 更新日期：2024年
> - 作者：Claude AI Assistant