# JSON

## 什么是JSON

### 基本概念

**JSON** (JavaScript Object Notation) 是一种轻量级的数据交换格式

它是一种独立于语言的文本格式，专为高效的数据交换而设计。"轻量级"指的是它相比其他格式（如XML）占用的字符更少，冗余信息更少，因此在网络传输时更快、更节省带宽



### 核心特点

**易于阅读和编写**

- JSON使用人类可读的纯文本格式

  其结构（如`"key": "value"`）和符号（如 `{}` 表示对象，`[]` 表示数组）对于开发者来说非常直观，易于手动创建和审查

**易于解析和生成**

- JSON的语法规则非常简单且明确

  这使得计算机程序（无论使用何种语言）都能快速、高效地将其解析为对应语言的数据结构（如对象、字典、列表等），或反过来将程序中的数据结构生成为JSON字符串

**语言无关**

- JSON的数据类型（对象、数组、字符串、数字、布尔值、null）是C家族语言（包括C++, Java, JavaScript, Python等）的通用子集

  这意味着任何支持这些基本数据类型的现代编程语言都可以轻松地处理JSON数据，促进了不同技术栈系统之间的互操作性

**文本格式**

- JSON是100%的纯文本。这使得它可以在任何支持文本传输的协议（如HTTP）上轻松传递，并且可以存储在任何支持文本的文件系统或数据库中，无需担心二进制兼容性问题（通常使用UTF-8编码）

**自描述性**

- JSON的结构通过键值对来组织数据

  键（key）本身就是一个字符串，用于描述其对应的值（value）的含义

  这使得数据在一定程度上具有“自我描述”的能力，读取者不需要外部的文档或模式（Schema）也能大致理解数据的含义

------



## JSON的历史和用途

### 历史

- **2001年**

  - 由Douglas Crockford首次提出并推广，旨在为Web应用提供一种比XML更轻便的数据交换方式，尤其适用于早期的AJAX技术

    它最初是基于JavaScript的对象字面量语法（Object Literal Syntax）的一个严格子集

- **2013年**

  - 正式成为ECMA-404标准

- **2017年**

  - 成为国际标准ISO/IEC 21778:2017



### 主要用途

**Web API** 

- 这是JSON最广泛的用途

  在现代Web开发中，服务器和客户端（如浏览器或移动App）之间的数据通信几乎完全依赖JSON

  RESTful API 和 GraphQL API 都以JSON作为首选的数据响应格式

**配置文件**

- 由于其清晰的层级结构和易读性，JSON非常适合用作软件应用的配置文件

  例如：Node.js的 `package.json`（项目元数据和依赖管理）、`tsconfig.json`（TypeScript编译器配置）、VS Code的设置文件等

**数据存储**

- 许多NoSQL数据库（如MongoDB, CouchDB, RethinkDB）使用JSON（或其二进制变体，如BSON）作为其核心的文档存储格式

  这允许开发者存储结构灵活、模式动态的数据

**数据传输**

- 泛指所有需要交换数据的场景，不仅限于Web API

  例如，在微服务架构中，不同服务之间可能通过消息队列（如Kafka, RabbitMQ）传递JSON格式的消息

**日志记录**

- "结构化日志"正变得越来越流行

  开发者不再记录纯文本日志，而是将日志信息（如时间戳、级别、消息、上下文）组织成JSON对象

  这极大地简化了日志的后期处理、查询和分析（例如在ELK Stack或Splunk中）

------



## JSON数据类型

⭐**对于JSON对象而言，键一定是字符串，值可以是JSON的六种数据类型 (字符串、数字、布尔值、null、对象 或 数组) 之一**

> 这里提醒一下，请侧重注意 “JSON对象” 这个术语，避免与 JSON Text 混淆
>
> JSON 对象，就是下面的 5 



### 1. 字符串 (String)

用于存储文本数据。必须用双引号包围，并支持一系列转义字符来表示特殊字符

```json
{
  "name": "张三",
  "address": "北京市朝阳区",
  "emoji": "😊",
  "escaped": "第一行\n第二行",
  "quote": "他说:\"你好\""
}
```

**常用转义字符:**

- `\"` - 双引号
- `\\` - 反斜杠
- `\/` - 斜杠 (技术上可选，但常用于转义HTML中的`</script>`)
- `\b` - 退格
- `\f` - 换页
- `\n` - 换行
- `\r` - 回车
- `\t` - 制表符
- `\uXXXX` - 四位十六进制的Unicode字符 (例如 `\u4F60` 代表 "你")



### 2. 数字 (Number)

用于存储数值。JSON不区分整数和浮点数（小数）。所有数字均基于IEEE 754双精度浮点格式

```json
{
  "integer": 42,
  "negative": -17,
  "float": 3.14159,
  "scientific": 1.23e-4,
  "zero": 0
}
```

**规范:**

- 不支持八进制 (如 `012`) 和十六进制 (如 `0xFF`) 格式
- 不支持非数字值，如 `NaN` (Not a Number) 和 `Infinity`
- 数字不能以点开头 (如 `.5` 是错误的, 必须写成 `0.5`)
- 数字不能有前导零 (如 `01` 是错误的, 必须写成 `1`)，除非它就是 `0` 或者是 `0` 后跟小数



### 3. 布尔值 (Boolean)

代表逻辑上的“真”或“假”

```json
{
  "isActive": true,
  "isDeleted": false
}
```

**规范:**

- 必须是小写的 `true` 或 `false`
- `True`, `FALSE`, `"true"`, `1`, `0` 都不是有效的JSON布尔值





### 4. 空值 (Null)

用于表示“空”或“无值”的特定值

```json
{
  "middleName": null,
  "spouse": null
}
```

**规范:**

- 必须是小写的 `null`
- `"null"` 或 `Null` 都不是有效的JSON空值



### 5. 对象 (Object)

一个无序的键值对集合，用大括号 (`{}`) 包围。这允许创建嵌套的、复杂的层级结构

```json
{
  "person": {
    "name": "李四",
    "age": 30,
    "address": {
      "city": "上海",
      "district": "浦东新区"
    },
    "isMarried": true
  }
}
```





### 6. 数组 (Array)

一个有序的值的列表，用方括号 (`[]`) 包围。数组中的值可以是任意JSON数据类型，包括其他数组或对象

```json
{
  "numbers": [1, 2, 3, 4, 5],
  "mixed": [1, "text", true, null, {"key": "value"}],
  "nested": [
    [1, 2],
    [3, 4],
    [5, 6]
  ],
  "empty": []
}
```



## JSON对象语法规则

### 基本规则

1. ⭐**对于JSON对象而言，键一定是字符串，值可以是JSON的六种数据类型 (字符串、数字、布尔值、null、对象 或 数组) 之一**

   > 这里提醒一下，请侧重注意 “JSON对象” 这个术语，避免与 JSON Text 混淆

   

2. **数据以键值对形式存储**：这是JSON对象的基本单元。一个“键” (key) 对应一个“值” (value)

   ```json
   {
     "name": "张三"
   }
   ```

   

3. **数据由逗号分隔`,`**：在对象或数组中，多个键值对或多个值之间必须用逗号隔开

   ```json
   {
     "name": "张三",
     "age": 25
   }
   ```

   

4. **对象用大括号 `{}` 包围**：对象是一个无序的键值对集合

   ```json
   {
     "person": {
       "name": "张三"
     }
   }
   ```

   

5. **数组用方括号 `[]` 包围**：数组是一个有序的值的列表

   ```json
   {
     "fruits": ["苹果", "香蕉", "橙子"]
   }
   ```

   

6. **键必须是字符串,且必须用双引号包围**：这是JSON与JavaScript对象字面量的关键区别之一

   ```json
   {
     "name": "正确",
     'name': "错误",
     name: "错误"
   }
   ```

   以下是错误示例：

   ```json
   {
     'name': "错误 (使用了单引号)",
     name: "错误 (没有引号)"
   }
   ```



### 重要规范

- **只能使用双引号** - 单引号不被允许

  JSON标准规定，所有的键以及所有为字符串类型的值都**必须**使用双引号。单引号 (`'`) 或不使用引号（用于键）都是无效的JSON

  

- **不允许注释** - 标准JSON不支持注释

  ECMA-404标准JSON规范中没有注释的语法。任何`//`或`/* */`都会导致解析失败。此设计的目的是保持格式的纯粹性和解析器的简单性

  

- **不允许尾随逗号** - 在对象或数组的最后一个元素后面不能有逗号

  错误示例：

  ```json
  {
    "name": "张三",
    "age": 25,  <-- 这里的逗号是无效的
  }
  ```

  错误示例：

  ```json
  {
    "fruits": ["苹果", "香蕉",]  <-- 这里的逗号是无效的
  }
  ```



- **严格的数据类型**

  字符串（如`"123"`）和数字（如`123`）是完全不同的类型。布尔值（`true`）和null（`null`）必须小写，且不能用引号包围



## JSON根元素

### 基本概念

⭐**一个 JSON 文件只能有一个根元素**

> 你不能在根级别（最外层）并列放置多个值，即使是用逗号分隔也不行。逗号 (`,`) 只能用于分隔**对象内部**的键值对或**数组内部**的值
>
> **无效的 JSON 示例 (多个根)：**
>
> ```json
> [ "苹果", "香蕉" ],  <-- 错误：不能在根级别使用逗号
> "Hello, World!"
> ```



根据JSON的官方标准 (ECMA-404)，一个**合法的、完整的JSON文档的根元素，可以是六种JSON数据类型中的任意一种**

>它**不一定**非得是JSON对象 (`{...}`)



### **核心概念辨析**

**JSON 对象**

- 指的是以 `{}` 包裹的数据结构
- **规则：** 在 `{}` 内部，数据**必须**以“键: 值”对的形式出现。键必须是字符串，值可以是六种类型中的任意一种

**JSON 文本 (官方术语:JSON Text)**:

- 指的是**整个文件(`.json`)或 整个API响应体(例如后端返回给前端)**

  > 我明白了!
  >
  > 这里在后端返回给前端的东西，其实不管怎么样都是一个Json Text，
  >
  > 我之前脑海中没这个概念，以为返回东西就单纯的返回一个普通的符合前端的东西
  >
  > 现在什么都明白了!

- **规则：** Json Text 的 根 可以 是六种类型中的任一 一种



## JSON对象结构

JSON通过对象（`{}`）和数组（`[]`）的组合与嵌套，可以灵活地表示从简单到极其复杂的对象数据结构

### 简单对象

这是最基础的JSON对象结构，也称为“扁平对象”。它只包含一层键值对，没有嵌套的对象或数组

**用途：** 非常适合表示简单的数据记录，如单个用户信息、配置项或简单的API响应

```json
{
  "id": 1,
  "username": "zhangsan",
  "email": "zhangsan@example.com",
  "isActive": true,
  "balance": 1000.50
}
```



### 嵌套对象

JSON允许一个对象的值是另一个JSON对象。这体现了JSON的层级结构能力

**用途：** 

- 用于组织和归类相关数据

  例如，将用户的个人资料（`profile`）或联系方式（`contact`）组织在一个内嵌的对象中，使主对象的结构更清晰

```json
{
  "user": {
    "id": 1,
    "profile": {
      "firstName": "三",
      "lastName": "张",
      "age": 25,
      "contact": {
        "email": "zhangsan@example.com",
        "phone": "13800138000"
      }
    }
  }
}
```



### 对象数组

这是JSON中最常见和最有用的对象结构之一。它是一个数组（`[]`），数组中的每个元素都是一个对象（`{}`），并且这些对象通常（但不必须）具有相同的结构

**用途：** Web API返回数据列表的标准方式。例如，获取所有用户、产品列表、文章列表或搜索结果时，服务器会返回一个对象数组

```json
{
  "users": [
    {
      "id": 1,
      "name": "张三",
      "age": 25
    },
    {
      "id": 2,
      "name": "李四",
      "age": 30
    },
    {
      "id": 3,
      "name": "王五",
      "age": 28
    }
  ]
}
```



### 复杂对象结构

通过结合使用嵌套对象、对象数组、以及包含不同数据类型（如字符串、数字、数组）的值，可以构建出任意复杂的结构来模拟现实世界的数据模型

**用途：** 用于表示复杂的数据实体，如大型配置文件、组织架构、复杂的API请求体（Payload）或完整的文档数据

```json
{
  "company": {
    "name": "科技有限公司",
    "founded": 2010,
    "departments": [
      {
        "name": "技术部",
        "employees": [
          {
            "id": 1,
            "name": "张三",
            "position": "高级工程师",
            "skills": ["Python", "JavaScript", "Docker"],
            "projects": [
              {
                "name": "项目A",
                "status": "进行中",
                "deadline": "2025-12-31"
              }
            ]
          }
        ]
      },
      {
        "name": "市场部",
        "employees": [
          {
            "id": 2,
            "name": "李四",
            "position": "市场经理",
            "skills": ["营销", "策划"],
            "projects": []
          }
        ]
      }
    ]
  }
}
```



## JSON vs XML

在JSON出现之前，XML是数据交换的主导格式。JSON的许多设计决策都是为了解决XML在某些场景（尤其是Web API）中的复杂性

### 核心差异对比

| 特性                      | JSON (JavaScript Object Notation)                            | XML (eXtensible Markup Language)                             |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **设计目标**              | 专为**数据交换**设计，追求简洁和高效。                       | 设计为**标记语言**，用于结构化文档，功能更全面。             |
| **可读性与简洁性**        | 语法简洁，使用键值对，冗余信息少，人类阅读负担小。           | 语法较冗长，使用闭合标签（`<tag>...</tag>`），可读性好但信息密度低。 |
| **数据大小**              | 通常更小。由于冗余少，网络传输时占用的带宽更少。             | 通常更大。标签的开销导致相同数据量的文件体积更大。           |
| **解析速度**              | 解析速度快。语法简单，解析器实现更轻量、更高效。             | 解析速度较慢。XML解析器需要处理更复杂的语法（如属性、命名空间）。 |
| **数据类型支持**          | **原生支持数组 (Array)**。这对于表示列表数据非常自然。       | **没有原生数组类型**。通常使用多个相同名称的标签来模拟列表（如此页示例中的`<tag>...`）。 |
| **注释**                  | **不支持**。JSON标准将其视为纯粹的数据，不应包含元信息。     | **支持**。可以使用 `<!-- ... -->` 语法添加注释。             |
| **命名空间 (Namespaces)** | **不支持**。结构简单，不需要解决复杂文档中的命名冲突。       | **支持**。允许在复杂文档中区分同名但不同含义的元素。         |
| **属性 (Attributes)**     | **不支持**。所有数据都必须是键值对中的“值”。                 | **支持**。数据可以存储在标签的属性中（如`<book id="1">`）或标签内容中。 |
| **模式验证**              | **JSON Schema**。一个独立的规范，用于定义和验证JSON数据的结构。 | **XSD, DTD**。内置于XML生态系统中的强大验证机制。            |



### 示例对比

为了直观展示差异，这里使用相同的数据结构分别用JSON和XML表示



#### JSON 示例

JSON使用键值对来描述数据，并使用数组（`[]`）来自然地表示标签列表

```json
{
  "book": {
    "title": "JSON教程",
    "author": "张三",
    "price": 49.99,
    "tags": ["编程", "Web开发"]
  }
}
```



#### 相同数据的 XML 示例

XML使用标签来描述数据。请注意 `tags` 是如何通过嵌套多个 `<tag>` 元素来模拟一个列表的

```json
<?xml version="1.0" encoding="UTF-8"?>
<book>
  <title>JSON教程</title>
  <author>张三</author>
  <price>49.99</price>
  <tags>
    <tag>编程</tag>
    <tag>Web开发</tag>
  </tags>
</book>
```



### 适用场景

**JSON** 更适合作为现代Web API和应用程序（尤其是移动端）的数据交换格式，因为它更轻量、更快、且与JavaScript等语言的数据结构（对象和数组）完美契合

**XML** 仍然在需要强大文档结构、模式验证、命名空间和注释的企业级系统、SOAP Web服务和配置文件（如大型框架配置、SVG）中占有一席之地



------

## 各编程语言中的JSON

JSON 作为一种数据格式，其价值体现在被各种编程语言解析（反序列化）为该语言的原生数据结构，或将该语言的数据结构（如对象、字典、列表）生成（序列化）为 JSON 字符串

### JavaScript (原生支持)

JSON 源于 JavaScript，因此 JavaScript 提供了原生的 `JSON` 全局对象来处理 JSON，无需任何外部库

#### JSON 解析 (字符串 → 对象)

使用 `JSON.parse()` 方法将 JSON 字符串转换为 JavaScript 对象

```javascript
// 这是一个JSON格式的字符串
const jsonString = '{"name": "张三", "age": 25, "isActive": true}';

// 将字符串解析为 JavaScript 对象
const obj = JSON.parse(jsonString);

// 现在可以像操作普通对象一样操作它
console.log(obj.name); // 输出: "张三"
console.log(obj.age);  // 输出: 25
```



#### JSON 序列化 (对象 → 字符串)

使用 `JSON.stringify()` 方法将 JavaScript 对象或值转换为 JSON 字符串

```javascript
// 这是一个 JavaScript 对象
const person = {
  name: "李四",
  age: 30,
  skills: ["Java", "SQL"],
  company: null
};

// 1. 转换为紧凑的 JSON 字符串
const json = JSON.stringify(person);
console.log(json);
// 输出: {"name":"李四","age":30,"skills":["Java","SQL"],"company":null}

// 2. 美化输出 (格式化)
// 第三个参数指定缩进的空格数 (这里是2个空格)
const prettyJson = JSON.stringify(person, null, 2);
console.log(prettyJson);
/*
输出:
{
  "name": "李四",
  "age": 30,
  "skills": [
    "Java",
    "SQL"
  ],
  "company": null
}
*/
```



### Java

Java 是强类型语言，处理JSON通常需要一个外部库

这里的示例中使用 `Jackson` ，我喜欢`Jackson`

#### 步骤 1: 定义 POJO 

首先，你需要创建一个与 JSON 结构匹配的 Java 类

```java
// Person.java
// (这个类用于映射 JSON 数据)
public class Person {
    // 字段名应与 JSON 的键匹配
    private String name;
    private int age;
    private boolean isActive;

    // Jackson 库需要一个无参构造函数
    public Person() {
    }

    // (可选) 一个用于创建对象的构造函数
    public Person(String name, int age, boolean isActive) {
        this.name = name;
        this.age = age;
        this.isActive = isActive;
    }

    // Jackson 通过 getter 和 setter 来访问私有字段
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }

    public boolean isActive() {
        return isActive;
    }
    public void setActive(boolean active) {
        isActive = active;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", isActive=" + isActive +
                '}';
    }
}
```



#### 步骤 2: 使用 ObjectMapper

`ObjectMapper` 是 Jackson 库的核心类，用于执行序列化和反序列化

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.core.JsonProcessingException;

public class JsonDemo {
    public static void main(String[] args) {
        // 1. 初始化 ObjectMapper
        ObjectMapper mapper = new ObjectMapper();

        // ------------------------------------------
        // JSON 解析 (字符串 -> Java 对象)
        // ------------------------------------------
        String jsonString = "{\"name\":\"张三\",\"age\":25,\"isActive\":true}";

        try {
            Person person1 = mapper.readValue(jsonString, Person.class);
            System.out.println("解析成功: " + person1);
            // 输出: 解析成功: Person{name='张三', age=25, isActive=true}

        } catch (JsonProcessingException e) {
            System.err.println("JSON 解析失败: " + e.getMessage());
        }

        // ------------------------------------------
        // JSON 序列化 (Java 对象 -> 字符串)
        // ------------------------------------------
        Person person2 = new Person("李四", 30, false);

        try {
            // 2. 转换为紧凑的 JSON 字符串
            String json = mapper.writeValueAsString(person2);
            System.out.println("序列化成功: " + json);
            // 输出: 序列化成功: {"name":"李四","age":30,"active":false}

            // 3. 转换为美化输出 (格式化)
            String prettyJson = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(person2);
            System.out.println("美化输出:\n" + prettyJson);
            /*
            输出:
            美化输出:
            {
              "name" : "李四",
              "age" : 30,
              "active" : false
            }
            */

        } catch (JsonProcessingException e) {
            System.err.println("JSON 序列化失败: " + e.getMessage());
        }
    }
}
```



## JSON Schema

> 暂略

JSON Schema 用于验证JSON数据的结构和内容





## 最佳实践

### 1. 命名规范

JSON的键应该遵循一致的命名约定。最流行的两种风格是：

- **camelCase (驼峰命名法)**: `firstName`, `orderNumber`。这是JavaScript生态系统的标准，因此在Web API中最为常见
- **snake_case (下划线命名法)**: `first_name`, `order_number`。这在Python, Ruby, PHP等后端语言中很常

**核心思想：** 选择一种风格，并在你的整个项目或API中**始终坚持使用它**。

```json
{
  "goodNaming": {
    "firstName": "驼峰命名法 (推荐用于JS生态)",
    "last_name": "下划线命名法 (在Python/Ruby等生态中常见)",
    "useConsistentStyle": "在整个API中保持一致的风格"
  },
  "badNaming": {
    "FirstName": "避免PascalCase (大写开头, 通常用于类名)",
    "LAST_NAME": "避免ALL_CAPS (全大写, 通常用于常量)",
    "mixed_Style": "不要混合不同风格 (最糟糕的做法)"
  }
}
```



### 2. 数据组织:使用对象包装根元素

当API返回数据时，**始终**使用JSON对象 (`{}`) 作为根元素，而不是数组 (`[]`)

**不推荐：** 直接使用数组作为根元素

```js
[
  {"id": 1, "name": "张三"},
  {"id": 2, "name": "李四"}
]
```

**推荐：** 使用对象包装数据，并将列表放在一个键下（如`"data"`）

```js
{
  "status": "success",
  "data": [
    {"id": 1, "name": "张三"},
    {"id": 2, "name": "李四"}
  ],
  "meta": {
    "total": 2,
    "page": 1,
    "limit": 10
  }
}
```

**为什么？**

1. **可扩展性：** 

   - 对象根元素允许你轻松添加元数据，如分页信息 (`meta`)、API状态 (`status`)、消息 (`message`) 等，而不会破坏数据结构

2. **一致性：** 

   - 无论你返回的是列表还是单个对象，响应的顶层结构始终是一个对象，使客户端处理更简单

3. **安全性：** 

   - 在非常古老的浏览器中，存在一种“JSON劫持”漏洞，可以覆盖JavaScript的`Array`构造函数来窃取顶层为数组的JSON数据

     使用对象作为根可以完全避免此问题



### 3. 错误处理:提供结构化错误

当API请求失败时，返回一个结构化的JSON错误对象，而不是纯文本或HTML

```json
{
  "status": "error",
  "error": {
    "code": "INVALID_INPUT",
    "message": "用户名不能为空",
    "details": {
      "field": "username",
      "constraint": "required"
    }
  },
  "timestamp": "2025-11-10T10:30:00Z"
}
```

**为什么？**

- `status: "error"`: 客户端可以简单地检查 `if (response.status === 'error')` 来启动错误处理逻辑
- `error.code`: (机器可读) 一个唯一的错误代码，供客户端程序识别错误类型（例如用于翻译错误消息）
- `error.message`: (人类可读) 一个清晰的错误描述，可以直接显示给用户
- `error.details`: (可选) 提供上下文，帮助开发者调试或在UI上高亮显示特定字段



### 4. 日期和时间格式:使用 ISO 8601

在JSON中表示日期和时间时，推荐**始终**使用 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 格式的字符串

```json
{
  "createdAt": "2025-11-10T10:30:00Z",
  "updatedAt": "2025-11-10T15:45:30+08:00",
  "birthDate": "1990-05-15"
}
```

**为什么？**

1. **消除歧义：** `10/11/2025` 到底是10月11日还是11月10日？ISO 8601 (`YYYY-MM-DD`) 格式是唯一的，没有歧义
2. **包含时区：** `Z` (Zulu) 表示UTC时间（世界标准时间）。`+08:00` 表示东八区时间。这对于全球化的应用至关重要
3. **机器友好：** 几乎所有编程语言的日期库都原生支持解析ISO 8601
4. **可排序：** 作为字符串，它可以按字典序正确排序



### 5. 布尔值命名:使用前缀

对于布尔值，使用 `is...`, `has...`, `can...`, `should...` 等前缀，使其读起来像一个问题

```json
{
  "good": {
    "isActive": true,
    "hasPermission": false,
    "canEdit": true,
    "shouldNotify": false
  },
  "avoid": {
    "active": true,
    "permission": false
  }
}
```

**为什么？** `if (user.active)` 并不如 `if (user.isActive)` 清晰。前者可能意味着“用户的活跃等级”，而后者明确是一个“是/否”的问题



### 6. 使用null表示缺失值

如果一个字段有意义，但它“没有值”或“不适用”，请明确使用 `null`

```json
{
  "user": {
    "firstName": "张",
    "middleName": null,
    "lastName": "三"
  }
}
```

**为什么？**

-  这比直接省略 `middleName` 键要好

  `"middleName": null` 明确地传达了“我们知道中间名这个属性，但该用户没有中间名”这一信息

  而省略该键可能意味着“我们不知道他是否有中间名”或“这个字段在此次请求中未被查询”

  `null` 是最明确的“无值”表达



### 7. 保持数据扁平化

在设计JSON结构时，应在“扁平化”和“组织性”之间找到平衡

**过度嵌套 (不推荐)**

```json
{
  "user": {
    "info": {
      "personal": {
        "id": 1,
        "name": "张三"
      }
    }
  }
}
```

- **问题：** 数据访问变得非常繁琐 (例如 `user.info.personal.name`)，增加了不必要的层级



**扁平化 (有时推荐):**

```json
{
  "userId": 1,
  "userName": "张三",
  "userEmail": "zhangsan@example.com"
}
```

- **优点：** 访问简单。=
- **缺点：** 如果字段过多（例如有50个`user...`字段），会失去组织性



**合理的嵌套 (推荐):** 

```json
{
  "user": {
    "id": 1,
    "name": "张三",
    "contact": {
      "email": "zhangsan@example.com",
      "phone": "13800138000"
    }
  }
}
```

- **平衡：** 逻辑上相关的数据（如`contact`）被组合在一起，使结构清晰，同时又避免了无意义的过度嵌套



## 常见错误与陷阱

JSON 的语法规则非常严格，远超 JavaScript 对象的字面量。大多数错误都源于将 JavaScript 的灵活写法错误地应用到了 JSON 上

### 1. 使用单引号

JSON 标准 (ECMA-404) 严格规定，所有的**键和字符串值都必须**使用双引号 (`"`)

JavaScript 允许使用单引号 (`'`)，但 JSON 不允许

```json
// ❌ 错误：JSON 不允许使用单引号
{'name': '张三'}

// ✅ 正确：必须使用双引号
{"name": "张三"}
```



### 2. 尾随逗号

在对象或数组的最后一个元素后面添加逗号，在 JSON 中是严格禁止的

这在现代 JavaScript 中是允许的（为了方便增删条目），但在 JSON 解析器中会导致语法错误

```json
// ❌ 错误：最后一个元素 "age": 25 后面不能有逗号
{
  "name": "张三",
  "age": 25,
}

// ✅ 正确：最后一个元素后没有逗号
{
  "name": "张三",
  "age": 25
}
```



### 3. 注释

标准 JSON 格式**不支持**任何形式的注释（无论是 `//` 还是 `/* */`）

JSON 被设计为一种纯粹的*数据交换格式*，而不是配置文件或程序

如果确实需要添加描述性信息，推荐的做法是将其作为数据的一部分，使用一个特殊的键（如下划线开头的 `_comment`）

```json
// ❌ 错误 (标准JSON不支持注释)
{
  // 这是注释
  "name": "张三"
}

// ✅ 正确 (如果需要注释,使用特殊字段作为元数据)
{
  "_comment": "这是用户信息",
  "name": "张三"
}
```



### 4. 非标准的值

JSON 的数据类型中不包括 `undefined`, `NaN` (Not a Number) 和 `Infinity` (无穷大)。这些是 JavaScript 特有的值

在 JSON 中，如果你想表示一个“缺失的”、“空的”或“不适用”的值，**唯一**的标准方式是使用 `null`

```json
// ❌ 错误 (undefined, NaN, Infinity 都不是JSON标准类型)
{
  "name": undefined,
  "value": NaN,
  "infinity": Infinity
}

// ✅ 正确 (统一使用 null 来表示缺失或无效值)
{
  "name": null,
  "value": null,
  "infinity": null
}
```



### 5. 函数和日期对象

JSON 是一种 **纯数据** 格式，它不能序列化（存储）逻辑或复杂的原生对象

- **函数**：不能被序列化

- **日期**：

  - JavaScript 的 `new Date()` 是一个对象

    它在序列化时必须被转换为字符串，最佳实践是使用 ISO 8601 格式

```json
// ❌ 错误 (JSON不支持函数和 Date 对象类型)
{
  "callback": function() {},
  "date": new Date()
}

// ✅ 正确 (函数转为null，日期转为ISO字符串)
{
  "callback": null,
  "date": "2025-11-11T12:00:00Z"
}
```





## 实战示例

### REST API响应

```json
{
  "status": "success",
  "code": 200,
  "message": "数据获取成功",
  "data": {
    "posts": [
      {
        "id": 1,
        "title": "JSON学习指南",
        "slug": "json-learning-guide",
        "excerpt": "全面了解JSON数据格式",
        "content": "JSON是一种轻量级的数据交换格式...",
        "author": {
          "id": 101,
          "name": "张三",
          "avatar": "https://example.com/avatars/101.jpg"
        },
        "category": {
          "id": 5,
          "name": "编程教程"
        },
        "tags": ["JSON", "Web开发", "数据格式"],
        "publishedAt": "2025-11-01T10:00:00Z",
        "updatedAt": "2025-11-10T15:30:00Z",
        "stats": {
          "views": 5420,
          "likes": 234,
          "comments": 45
        },
        "featured": true
      }
    ]
  },
  "pagination": {
    "currentPage": 1,
    "totalPages": 10,
    "perPage": 20,
    "totalItems": 200,
    "hasNextPage": true,
    "hasPreviousPage": false
  },
  "meta": {
    "timestamp": "2025-11-10T10:30:00Z",
    "apiVersion": "v2",
    "requestId": "req-123456789"
  }
}
```
