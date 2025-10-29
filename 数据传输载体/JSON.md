# JSON

- **JSON** (JavaScript Object Notation) 是一种轻量级的数据交换格式。它易于人阅读和编写，同时也易于机器解析和生成。尽管它的名字源于 JavaScript，但 JSON 是一种完全独立于语言的文本格式，许多编程语言都有解析和生成 JSON 数据的库。



## 核心概念

- **数据交换格式**：JSON 的主要用途是在客户端和服务器之间传输数据。例如，当你在网页上填写表单并提交时，这些信息可以被打包成 JSON 格式发送到服务器。
- **轻量级**：与 XML 等其他数据格式相比，JSON 的语法更简洁，占用的带宽更少，这使得网络传输更快。
- **易于阅读**：它的结构清晰，基于键值对，非常接近人类的自然阅读习惯。



## 基本语法规则

- **数据由键值对 (key/value pairs) 组成**：`"key": value`。
- **键 (key) 必须是字符串**：并且必须用双引号 `"` 包裹。单引号是不被允许的。
- **数据之间用逗号 `,` 分隔**。
- **花括号 `{}` 用于保存对象 (Object)**：对象是无序的键值对集合。
- **方括号 `[]` 用于保存数组 (Array)**：数组是有序的值的集合。
- **不允许有注释**：标准 JSON 格式不支持注释。
- **不允许尾随逗号**：在最后一个键值对或数组元素后面不能有逗号。



## 数据类型

JSON 支持以下几种数据类型：

| 类型                 | 描述                                       | 示例                            |
| -------------------- | ------------------------------------------ | ------------------------------- |
| **字符串 (String)**  | 必须用双引号 `"` 包裹的文本。              | `"Hello, World!"`, `"name"`     |
| **数字 (Number)**    | 整数或浮点数。不支持 `NaN` 或 `Infinity`。 | `101`, `3.14`, `-25`            |
| **对象 (Object)**    | 无序的键值对集合，包含在花括号 `{}` 中。   | `{"name": "Alice", "age": 30}`  |
| **数组 (Array)**     | 有序的值的集合，包含在方括号 `[]` 中。     | `["apple", "banana", "cherry"]` |
| **布尔值 (Boolean)** | `true` 或 `false`。必须是小写。            | `true`                          |
| **null**             | 表示空值或无值。必须是小写。               | `null`                          |



## 结构示例

### 简单对象

- 这是一个描述用户的简单 JSON 对象

```js
{
  "id": 12345,
  "name": "张三",
  "email": "zhangsan@example.com",
  "isVerified": true,
  "registrationDate": "2023-10-27"
}
```

### 复杂对象 (嵌套)

- JSON 对象和数组可以相互嵌套，形成复杂的数据结构

```js
{
  "id": 67890,
  "name": "李四",
  "email": null,
  "isStudent": true,
  "address": {
    "street": "人民路100号",
    "city": "上海",
    "postalCode": "200000"
  },
  "courses": [
    {
      "title": "计算机科学导论",
      "credits": 3,
      "teacher": "王教授"
    },
    {
      "title": "数据结构",
      "credits": 4,
      "teacher": "刘教授"
    }
  ]
}
```

### 对象数组

- 一个由多个相似对象组成的数组，常用于表示列表数据。

```js
[
  {
    "productName": "笔记本电脑",
    "price": 7999,
    "inStock": true
  },
  {
    "productName": "智能手机",
    "price": 4999,
    "inStock": false
  },
  {
    "productName": "无线耳机",
    "price": 899,
    "inStock": true
  }
]
```



## 常见应用场景

1. **Web APIs**：绝大多数现代的 RESTful API 都使用 JSON 作为请求和响应的数据格式。
2. **配置文件**：许多应用程序（如 VS Code、npm）使用 `.json` 文件来存储配置信息。
3. **前后端数据交换**：在现代 Web 开发（如 React, Vue, Angular）中，前端框架通过 API 从后端获取 JSON 数据，然后在页面上渲染。
4. **NoSQL 数据库**：像 MongoDB 这样的文档数据库直接以类似 JSON 的格式（BSON）存储数据。



## JS中关于JSON操作的常见方法

- `JSON.stringify(object)`: 将 JavaScript 对象转换为 JSON 字符串
- `JSON.parse(string)`: 将 JSON 字符串解析为 JavaScript 对象

**示例**

```JS
const user = {
  id: 1,
  name: "Leanne Graham",
  email: "Sincere@april.biz"
};

// 转换为 JSON 字符串
const jsonString = JSON.stringify(user);
console.log(jsonString);
// '{"id":1,"name":"Leanne Graham","email":"Sincere@april.biz"}'

// 从 JSON 字符串解析回对象
const parsedUser = JSON.parse(jsonString);
console.log(parsedUser.name); // Leanne Graham
```



