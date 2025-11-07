# 数据库基础与MySQL简介

## 1.什么是数据库？

> 本质就是磁盘的一个文件夹

- **数据库 (Database, DB)** 是一个长期存储于计算机内、有组织的、可共享的数据集合。它的核心是**数据**，但并非简单堆砌，而是以特定结构进行**组织**，以便于高效管理和访问
- **数据库管理系统 (Database Management System, DBMS)** 是位于用户与数据库之间的一层管理软件。所有对数据库的实际操作，如定义数据结构、添加、修改、删除、查询数据，以及控制数据访问权限等，都由 DBMS 统一管理和执行。它确保了数据的**一致性**和**安全性**
  - 我们通常所说的“学习 MySQL”，其本质是学习如何使用 **MySQL 这个 DBMS** 来操作和管理数据



## 2.什么是关系型数据库？

- **关系型数据库 (Relational Database, RDBMS)** 是建立在**关系模型**基础上的数据库。在关系模型中，数据被组织在称为**表**的二维结构中

  - **表 (Table)**：数据存储的基本单位，由行和列组成。每个表在数据库中都有一个唯一的名称，例如 `students` 表
  - **列 (Column / Field)**：表中的一个垂直列，代表一个**字段**。每个列都有明确的数据类型（如 `VARCHAR` 用于字符串，`INT` 用于整数）和名称

  - **行 (Row / Record)**：表中的一个水平行，代表一条**记录**，即一个实体的一组相关数据值

- **核心在于“关系”**：

  - 关系型数据库的强大之处在于能够在不同的表之间建立逻辑关联。这种关联是通过共享的字段（通常是**主键和外键**）来实现的。

- **示例**：

  - **`students` 表** (存储学生基本信息)

    | id (主键) | name |
    | --------- | ---- |
    | 1         | 张三 |
    | 2         | 李四 |

  - **`scores` 表** (存储学生成绩)

    | id   | student_id (外键) | subject | score |
    | ---- | ----------------- | ------- | ----- |
    | 1    | 1                 | 数学    | 95    |
    | 2    | 2                 | 数学    | 88    |
    | 3    | 1                 | 英语    | 92    |

    - 通过 `scores` 表中的 `student_id` 字段，可以关联到 `students` 表中的 `id` 字段。这样，我们就可以将两张表的数据连接起来，查询出“张三的各科成绩”，这就是“关系”的体现



## 3.什么是 SQL？

- **SQL (Structured Query Language, 结构化查询语言)** 是用于与关系型数据库进行通信的**标准编程语言**。它是一种声明式语言，用户只需说明“做什么”，而无需关心“怎么做”

- SQL 的主要功能范畴包括：

  - **数据查询语言 (DQL)**: `SELECT` - 用于从数据库中检索数据。

  - **数据操作语言 (DML)**: `INSERT`, `UPDATE`, `DELETE` - 用于操作表中的数据。

  - **数据定义语言 (DDL)**: `CREATE`, `ALTER`, `DROP` - 用于定义和管理数据库对象，如表、索引等。

  - **数据控制语言 (DCL)**: `GRANT`, `REVOKE` - 用于管理用户权限。



## 4.什么是 MySQL？

- **MySQL** 是一个具体的、开源的关系型数据库管理系统(RDBMS) . 它是目前世界上最受欢迎的数据库之一，尤其在 Web 开发领域

- **核心技术特点**：

  - **RDBMS 实现**：MySQL 是关系模型的一个成熟实现，使用 SQL 作为其标准交互语言。

  - **开源与社区驱动**：其社区版免费，拥有庞大的用户和开发者社区，提供了丰富的文档和技术支持。

  - **高性能**：以其出色的读写性能和响应速度而闻名，并支持通过索引等方式进行性能优化。

  - **跨平台**：可以在多种操作系统上运行，包括 Linux、Windows 和 macOS。

  - **技术栈中的关键角色**：
    - 在经典的 Web 技术栈 **LAMP** (Linux + Apache + MySQL + PHP/Python/Perl) 和 **LNMP** (Linux + Nginx + MySQL + PHP/Python/Perl) 中，MySQL 承担着核心的数据存储任务

- MySQL 是一个你可以实际安装、部署并用于应用程序开发的、功能强大的开源 RDBMS 软件



# SQL核心语法与基础操作

## DDL - 数据定义语言

- **DDL (Data Definition Language)** 用于创建、修改和删除数据库的结构性对象，如数据库、表、索引等。它的操作对象是**结构**，而不是数据本身

### 数据库级操作

#### **1. 创建数据库**

- **作用**：在 MySQL 服务器上创建一个新的数据库容器

- **语法**：

  ```mysql
  CREATE DATABASE [IF NOT EXISTS] 数据库名 [CHARACTER SET 字符集];
  ```

  - `IF NOT EXISTS` 是一个安全选项，表示只有当该数据库不存在时才执行创建操作，避免因数据库已存在而报错

  - `CHARACTER SET` 用于指定数据库的默认字符集，如 `utf8mb4`，以支持包括 Emoji 在内的多种字符

    > MySQL8版本，创建数据库，如果不设置字符集，默认也是utf8mb4

- **示例**：创建一个名为 `my_school` 的数据库，并指定字符集为 `utf8mb4`

  ```mysql
  CREATE DATABASE IF NOT EXISTS my_school CHARACTER SET utf8mb4;
  ```



#### **2. 查看所有数据库**

- **作用**：列出当前 MySQL 服务器上所有可访问的数据库

  >`SELECT DATABASE();`  ：是**查询当前选中的数据库**，告诉当前的工作位置

- **语法**：

  ```mysql
  SHOW DATABASES;
  ```



#### **3. 选择一个数据库**

- **作用**：指定后续的 SQL 命令将在哪个数据库的上下文中执行。在操作表之前，必须先选择数据库

  > 有些语句不选择数据库没办法执行

- **语法**：

  ```mysql
  USE 数据库名;
  ```

- **示例**：

  ```mysql
  USE my_school;
  ```



#### **4. 删除某个数据库**

- **作用**：永久性地删除一个数据库及其包含的所有表和数据。**此操作不可逆，请极度谨慎使用！**

- **语法**：

  ```mysql
  DROP DATABASE [IF EXISTS] 数据库名;
  ```

- **示例**：

  ```mysql
  DROP DATABASE IF EXISTS my_school;
  ```



### **表级操作**

#### **1. 创建一个表**

- **作用**：在当前选定的数据库中，根据指定的列名、数据类型和约束来创建一个新表。

- **语法**：

  ```mysql
  CREATE TABLE 表名 (
      列名1 数据类型 [约束条件] [comment 字段1注释],
      列名2 数据类型 [约束条件] [comment 字段2注释],
      ...
      [表级约束]
  )[comment 表注释];
  ```

- **示例**：在 `my_school` 数据库中创建一个 `students` 表。

  ```mysql
  CREATE TABLE students (
      id INT PRIMARY KEY AUTO_INCREMENT,
      student_no VARCHAR(20) NOT NULL UNIQUE,
      name VARCHAR(50) NOT NULL,
      age INT,
      gender CHAR(1) DEFAULT '男',
      enrollment_date DATE
  );
  ```



#### **2. 查看所有表**

- **作用**：列出当前数据库中所有的表。

- **语法**：

  ```mysql
  SHOW TABLES;
  ```



#### **3. 查看某个表的结构**

- **作用**：显示一个表的详细结构，包括所有列的名称、数据类型、是否允许为空（NULL）、键信息（KEY）、默认值（Default）等。

- **语法**：

  ```mysql
  DESCRIBE 表名; -- 或者缩写为 DESC 表名;
  ```

- **示例**：

  ```mysql
  DESC students;
  ```



#### 4. 查看建表语句

- 作用：查看某个表的建表语句

- 语法：

  ```mysql
  show create table 表名;
  ```

- 示例：

  ```mysql
  show create table emp;
  ```

  





#### **5. 修改某个表**

- **作用**：用于修改已存在表的结构。这是一个功能非常强大的命令。

- **语法与示例**：

  - **添加列**

    - **语法**: 

      ```mysql
      ALTER TABLE 表名 ADD COLUMN 新列名 数据类型 [约束];
      ```

    - **示例**:

      ```mysql
      ALTER TABLE students ADD COLUMN email VARCHAR(100) UNIQUE;
      ```

    

  - **修改列定义**

    - **语法**: 

      ```mysql
      ALTER TABLE 表名 MODIFY COLUMN 列名 新数据类型 [新约束];
      ```

    - **示例**:

      ```mysql
      ALTER TABLE students MODIFY COLUMN name VARCHAR(100) NOT NULL;
      ```

    

  - **删除列**

    - **语法**: 

      ```mysql
      ALTER TABLE 表名 DROP COLUMN 列名;
      ```

    - **示例**:

      ```mysql
      ALTER TABLE students DROP COLUMN enrollment_date;
      ```

    

  - **重命名表**

    - **语法**: 

      ```mysql
      ALTER TABLE 旧表名 RENAME TO 新表名;
      ```

    - **示例**:

      ```mysql
      ALTER TABLE students RENAME TO student_info;
      ```

  

  - **重命名并修改列定义	**	

    - **作用**: 用于**重命名一个列**，并且可以**同时修改该列的数据类型和约束**。这是它与 `MODIFY COLUMN` 的最大区别。

    - **语法**:

      ```SQL
      ALTER TABLE 表名 CHANGE COLUMN 旧列名 新列名 新数据类型 [新约束];
      ```

    - **示例**:

      ```SQL
      -- 将 emp 表中的 qq 列重命名为 qq_number，并将其类型改为 VARCHAR(15)，同时添加注释
      ALTER TABLE emp CHANGE COLUMN qq qq_number VARCHAR(15) COMMENT 'QQ号码';
      ```



#### **6. 删除表**

- **作用**：永久性地删除一个表及其所有数据。**此操作同样不可逆，需谨慎！**

- **语法**：

  ```mysql
  DROP TABLE [IF EXISTS] 表名;
  ```



#### **7. 清空表**

- **作用**：快速删除一个表中的所有行，但保留表结构。

- **语法**：

  ```mysql
  TRUNCATE TABLE 表名;
  ```

- **与 `DELETE` 的区别**：`TRUNCATE` 速度更快，不写日志，不可回滚，且会重置自增列（`AUTO_INCREMENT`）的计数器。而 `DELETE` 是一行一行地删，可以回滚



## DML - 数据操作语言

- **DML (Data Manipulation Language)** 用于操作表中的**具体数据记录**，即对表中的行进行增、删、改

### **1. 插入数据**

- **作用**：向表中添加一行或多行新数据。

- **语法**：

  ```MySQL
  -- 插入单行，并指定列
  INSERT INTO 表名 (列1, 列2, ...) VALUES (值1, 值2, ...);
  
  -- 单行试图全部字段都加入值
  insert into 表名 values (值1, 值2, ...);
  
  -- 插入多行
  INSERT INTO 表名 (列1, 列2, ...) VALUES (值1, 值2, ...), (值A, 值B, ...);
  
  -- 多行试图全部字段都加入值
  insert into 表名 values (值1, 值2, ...), (值1, 值2, ...);
  ```

- **示例**：

  ```mysql
  INSERT INTO students (student_no, name, age, gender) 
  VALUES ('2025001', '王明', 18, '男'), ('2025002', '李莉', 17, '女');
  ```



### **2. 更新数据**

- **作用**：修改表中已存在记录的某些列的值。

- **语法**：

  ```mysql
  UPDATE 表名 SET 列1 = 新值1, 列2 = 新值2, ... WHERE 条件;
  ```

  > **[极其重要]** `WHERE` 子句用于定位要更新的记录。如果省略 `WHERE` 子句，**将会更新表中的所有行**，这通常是灾难性的。
  >
  > 所以一定不要遗忘了`WHERE`子句

- **示例**：将学号为 `2025001` 的学生的年龄更新为 19。

  ```mysql
  UPDATE students SET age = 19 WHERE student_no = '2025001';
  ```



### **3. 删除数据**

- **作用**：从表中删除满足条件的记录

- **语法**：

  ```mysql
  DELETE FROM 表名 WHERE 条件;
  ```

  > **[极其重要]** `WHERE` 子句用于定位要删除的记录。如果省略 `WHERE` 子句，**将会删除表中的所有行**

- **示例**：删除学号为 `2025003` 的学生记录

  ```mysql
  DELETE FROM students WHERE student_no = '2025003';
  ```



## DQL - 数据查询语言

- **DQL (Data Query Language)** 用于从表中检索数据，是 SQL 中使用最频繁、功能最丰富的部分

  ![image-20250814214709964](./assets/image-20250814214709964.png)



### **1. 基本查询 (SELECT)**

- **作用**：从一个或多个表中查询数据

- **语法**：

  ```mysql
  SELECT 列名1, 列名2, ... FROM 表名;
  ```

  - 使用 `*` 可以代表查询所有列

- **示例**：

  ```mysql
  -- 查询所有学生的所有信息
  SELECT * FROM students;
  
  -- 只查询学生的学号和姓名
  SELECT student_no, name FROM students;
  ```



### **2. 列别名 (AS)**

- **作用**：在查询结果中，为**列**提供一个临时的、可读性更好的名称。`AS` 关键字可以省略

- **语法**：

  ```mysql
  SELECT 列名 AS 别名 FROM 表名;
  ```

- **示例**：

  ```mysql
  SELECT student_no AS '学号', name AS '姓名' FROM students;
  ```



### **3. 条件查询 (WHERE)**

#### 基本概念

- **作用**：在 `SELECT`, `UPDATE`, `DELETE` 语句中，通过指定一个或多个条件来过滤出需要操作的记录。只有满足 `WHERE` 子句中所有条件的行才会被返回或影响。

- **语法**：

  ```mysql
  SELECT 列名 FROM 表名 WHERE 条件表达式;
  ```



#### 条件表达式

- **条件表达式**: 条件表达式由**列名、运算符和值组成**。**多个表达式可以通过逻辑运算符组合**



##### **a. 比较运算符**

- 用于比较两个表达式的值。

  - `=`: 等于
  - `!=` 或 `<>`: 不等于
  - `>`: 大于
  - `<`: 小于
  - `>=`: 大于等于
  - `<=`: 小于等于

- **示例**: 查询年龄大于 18 岁的学生

  ```mysql
  SELECT name, age FROM students WHERE age > 18;
  ```



##### **b. 逻辑运算符**

- 用于连接或反转多个条件表达式，以构建更复杂的过滤逻辑。

  - `AND(&&)`: 逻辑与。当所有由 `AND` 连接的条件都为真时，整个条件才为真。
  - `OR(||)`: 逻辑或。当由 `OR` 连接的条件中至少有一个为真时，整个条件就为真。
  - `NOT(!)`: 逻辑非。反转一个条件的布尔值。

- **优先级**: `NOT` > `AND` > `OR`。可以使用括号 `()` 来改变运算优先级。

- **示例**: 查询年龄大于 18 岁 **并且** 性别为 '女' 的学生

  ```mysql
  SELECT * FROM students WHERE age > 18 AND gender = '女';
  ```

- **示例**: 查询年龄大于 20 岁 **或者** 性别为 '男' 的学生

  ```mysql
  SELECT * FROM students WHERE age > 20 OR gender = '男';
  ```



##### **c. 范围运算符**

- `BETWEEN ... AND ...`: 用于检查一个值是否在指定的范围内（包含边界值）。

- **语法**: 

  ```mysql
  字段名(列名) BETWEEN 值1 AND 值2
  ```

  > 注意：这里的值1不能大于值2，等价关系是下面那样

  - **等价于**: 

    ```mysql
    字段名(列名) >= 值1 AND 列名 <= 值2
    ```



- **示例**: 查询分数在 80 到 90 分之间的学生。

  ```mysql
  SELECT name, score FROM students WHERE score BETWEEN 80 AND 90;
  ```



##### **d. 集合运算符**

- `IN (...)`: 用于检查一个值是否匹配列表中的任意一个值。

- `NOT IN (...)`: 用于检查一个值是否**不**匹配列表中的任意一个值。

- **语法**: 

  ```mysql
  字段名(列名) IN (值1, 值2, ...);
  ```

  

- **优点**: 比多个 `OR` 条件更简洁、可读性更高。

- **示例**: 查询学号是 '2025001' 或 '2025002' 的同学。

  ```mysql
  SELECT * FROM students WHERE student_no IN ('2025001', '2025002');
  ```



##### **e. 模糊匹配**

- `LIKE`: 用于在字符串中搜索指定的模式

- **通配符**:

  - `%` (百分号): 代表零个、一个或多个任意字符
  - `_` (下划线): 代表一个任意字符

- **示例**: 查询所有姓“王”的同学。

  ```mysql
  SELECT * FROM students WHERE name LIKE '王%';
  ```

- **示例**: 查询名字是两个字，且第二个字是“明”的同学。

  ```mysql
  SELECT * FROM students WHERE name LIKE '_明';
  ```



##### **f. 空值判断**

- `IS NULL`: 用于检查一个值是否为 `NULL`。

- `IS NOT NULL`: 用于检查一个值是否不为 `NULL`。

- **[极其重要]**: 不能使用比较运算符如 `= NULL` 或 `!= NULL` 来判断空值，因为 `NULL` 是一个特殊的值，它不等于任何值，包括它自己。必须使用 `IS` 或 `IS NOT`

- **示例**: 查询还没有分配邮箱地址的学生。

  ```mysql
  SELECT name FROM students WHERE email IS NULL;
  ```



## DCL - 数据控制语言

- DCL (Data Control Language) 用于定义数据库的访问权限和安全级别，通常由数据库管理员（DBA）使用

  - **授予权限 (GRANT)**：赋予用户或角色对数据库对象的操作权限。

  - **撤销权限 (REVOKE)**：收回已经赋予的权限。





# 常用数据类型

- 在数据库中，为表的每一列选择一个最合适的数据类型至关重要。一个好的选择可以节省存储空间、提升查询效率并保证数据的有效性。

## 数值类型

- 用于存储各种数字，如年龄、价格、数量等

  ![image-20250814185715350](./assets/image-20250814185715350.png)



### **整数类型**

- 用于存储没有小数部分的数字。不同类型的区别在于它们的存储空间和取值范围

  | 类型          | 存储空间 | 有符号范围 (Signed)            | 无符号范围(Unsigned) | 常用场景                                         |
  | ------------- | -------- | ------------------------------ | -------------------- | ------------------------------------------------ |
  | **`TINYINT`** | 1 字节   | -128 到 127                    | 0 到 255             | 存储状态（如 0/1）、年龄、很小的枚举值           |
  | **`INT`**     | 4 字节   | -2147483648 到 2147483647      | 0 到 4294967295      | **最常用的整数类型**，用于ID、计数等             |
  | **`BIGINT`**  | 8 字节   | 约 -9.22 x 10¹⁸ 到 9.22 x 10¹⁸ | 约 0 到 1.84 x 10¹⁹  | 数据量极大的表的主键ID、需要存储非常大整数的场景 |

- **`UNSIGNED` 关键字**：如果确定一列的数值永远不会是负数（如自增ID、年龄），可以将其声明为 `UNSIGNED`（无符号），使其正数存储范围扩大一倍。

  ```mysql
  CREATE TABLE example (
      user_id INT UNSIGNED,
      age TINYINT UNSIGNED
  );
  ```



### 定点数类型

- 用于需要**精确**小数计算的场景，不会出现浮点数那样的精度误差

- **`DECIMAL(M, D)`**
  - **作用**：以字符串形式存储的精确小数。非常适合用于存储货币、汇率等对精度要求极高的数据。
  - **`M` (Precision)**：总位数（整数部分+小数部分），最大为 65。
  - **`D` (Scale)**：小数部分的位数，最大为 30。
  - **示例**：`price DECIMAL(10, 2)` 可以存储最多 10 位数字，其中 2 位是小数，范围从 `-99999999.99` 到 `99999999.99`。



### 浮点数类型

- 用于存储近似的小数值，存储空间较小，但可能存在精度损失

  - **`FLOAT`**: 单精度浮点数，占用 4 字节。

  - **`DOUBLE`**: 双精度浮点数，占用 8 字节，精度比 `FLOAT` 更高。

> **何时使用?** 当你不需要精确值，只需要一个大概的数值时（例如科学计算中的平均值），可以使用浮点数。**在涉及金融计算时，应坚决避免使用 `FLOAT` 和 `DOUBLE`，而使用 `DECIMAL`。**



## 字符串类型

- 用于存储文本数据，如姓名、地址、文章内容等

  ![image-20250814185840272](./assets/image-20250814185840272.png)



### `CHAR(N)`

- **作用**：用于存储**固定长度**的字符串。`N` 的范围是 0 到 255

- **`N` 的含义**：`N` 定义了该列的**固定字符长度**。

- **特点**：

  1. **固定空间**：无论实际存储的字符串有多长，它都会占用 `N` 个字符的存储空间。
  2. **自动补全**：如果存入的字符串长度小于 `N`，MySQL 会在右侧用**空格**补全至 `N` 的长度。
  3. **长度约束**：如果试图存入的字符串长度**大于 `N`**，MySQL 会报错并拒绝操作（在严格模式下）。

- **适用场景**：存储长度非常固定的数据，如性别（'男', '女'）、MD5哈希值（32位）、邮政编码等。在这些场景下，它的查询效率通常比 `VARCHAR` 略高

  

### `VARCHAR(N)`

- **作用**：用于存储**可变长度**的字符串。`N` 的范围是 0 到 65535（理论值，实际受字符集和行大小限制）。
- **`N` 的含义**：`N` 定义了该列所能存储的**最大字符数**，是一个上限约束。
- **特点**：
  1. **可变空间**：存储空间根据字符串的实际长度变化，还会额外使用 1-2 个字节来记录字符串的实际长度。
  2. **长度约束**：如果试图存入的字符串长度**大于 `N`**，MySQL 同样会报错并拒绝操作。
  3. **最常用类型**：这是**最常用**的字符串类型，能有效节省存储空间。
- **适用场景**：存储长度不固定的数据，如用户名、地址、标题等。

- **`TEXT` 类型**
  - **作用**：用于存储非常长的文本数据。
  - **类型系列**：
    - `TINYTEXT`: 最多 255 字符。
    - `TEXT`: 最多 65,535 字符。
    - `MEDIUMTEXT`: 最多 约1600万 字符。
    - `LONGTEXT`: 最多 约42亿 字符。
  - **适用场景**：存储文章内容、用户评论、日志信息等。



### `CHAR`vs`VARCHAR`

| 特性         | `CHAR(N)`                               | `VARCHAR(N)`                     |
| ------------ | --------------------------------------- | -------------------------------- |
| **长度**     | 固定长度                                | 可变长度                         |
| **空间占用** | 始终为 N 个字符的空间                   | 实际字符长度 + 1/2字节长度开销   |
| **处理方式** | 不足N则用空格填充                       | 按实际长度存储                   |
| **最大长度** | 255                                     | 65,535 (理论值)                  |
| **效率**     | 长度固定，处理速度略快                  | 长度可变，需要计算长度           |
| **选择时机** | **数据长度固定**，如MD5值、手机号、邮编 | **数据长度不固定**，绝大多数场景 |



## 日期和时间类型

- 用于存储时间相关的信息

  ![image-20250814185924352](./assets/image-20250814185924352.png)

  - **`DATE`**

    - **作用**：只存储日期。

    - **格式**：`'YYYY-MM-DD'`

    - **范围**：`'1000-01-01'` 到 `'9999-12-31'`

    - **适用场景**：生日、注册日期、事件发生日期等。

      

  - **`TIME`**

    - **作用**：只存储时间或时间间隔。

    - **格式**：`'HH:MM:SS'`

    - **范围**：`-838:59:59` 到 `838:59:59`

    - **适用场景**：记录一天的某个时刻，或计算持续时间。

      

  - **`DATETIME`**

    - **作用**：存储日期和时间的组合。

    - **格式**：`'YYYY-MM-DD HH:MM:SS'`

    - **范围**：`'1000-01-01 00:00:00'` 到 `'9999-12-31 23:59:59'`

    - **特点**：存储的值与时区**无关**。数据库里存的是什么，取出来就是什么。

    - **适用场景**：需要精确记录发生时间，且应用不涉及跨时区转换。

      

  - **`TIMESTAMP`**

    - **作用**：也存储日期和时间的组合，但功能更特殊。

    - **范围**：`'1970-01-01 00:00:01'` UTC 到 `'2038-01-19 03:14:07'` UTC。

    - **特点**：

      1. **与时区相关**：存储时，MySQL会将其从当前连接的时区转换为 UTC（世界标准时间）进行存储；检索时，又会从 UTC 转换回当前连接的时区。
      2. **自动更新**：可以将 `TIMESTAMP` 列设置为在行创建或更新时自动记录当前时间。

    - **适用场景**：记录数据的创建时间（`create_time`）和最后修改时间（`update_time`），或需要处理多时区业务的应用。

      

  - **`YEAR`**

    - **作用**：存储年份。
    - **格式**：`YYYY`
    - **范围**：`1901` 到 `2155`。



# 数据完整性:约束与键

- 约束是施加在表列上的规则，用于强制保证数据的完整性。当对表中的数据进行增、删、改操作时，如果违反了约束，数据库将拒绝执行该操作

## 1.`NOT NULL`(非空约束)

- **作用**：确保该列的值**不能为 `NULL`**。

- **目的**：强制字段必须包含一个有效值。`NULL` 表示“未知”或“无值”，在很多业务场景下，关键字段（如姓名、订单号）是不允许为空的

- **语法**：

  ```mysql
  -- 在创建表时定义
  CREATE TABLE 表名 (
      列名 数据类型 NOT NULL
  );
  
  -- 在已存在的表上添加
  ALTER TABLE 表名 MODIFY COLUMN 列名 数据类型 NOT NULL;
  ```

- **示例**：

  ```mysql
  CREATE TABLE employees (
      id INT PRIMARY KEY,
      name VARCHAR(100) NOT NULL, -- 姓名不能为空
      department VARCHAR(50)
  );
  
  -- 下面的插入会成功
  INSERT INTO employees (id, name, department) VALUES (1, '张三', '技术部');
  
  -- 下面的插入会失败，因为 name 违反了 NOT NULL 约束
  INSERT INTO employees (id, department) VALUES (2, '市场部');
  ```



## 2.`UNIQUE`(唯一约束)

- **作用**：确保该列中的所有值都是**唯一的**，没有重复值。

- **目的**：防止业务上需要唯一的数据出现重复，如身份证号、邮箱、用户名等。

- **注意**：一个表中可以有多个 `UNIQUE` 约束的列。`UNIQUE` 约束的列可以包含 `NULL` 值，并且可以包含多个 `NULL` 值（因为 `NULL` 不等于任何值，包括它自己）。

- **语法**：

  ```mysql
  -- 列级约束
  CREATE TABLE 表名 (
      列名 数据类型 UNIQUE
  );
  
  -- 表级约束（可用于定义多列组合的唯一约束）
  CREATE TABLE 表名 (
      列1 数据类型,
      列2 数据类型,
      UNIQUE (列1, 列2)
  );
  ```

- **示例**：

  ```mysql
  CREATE TABLE users (
      id INT PRIMARY KEY,
      username VARCHAR(50) NOT NULL,
      email VARCHAR(100) UNIQUE, -- 邮箱必须唯一
      UNIQUE (username) -- 用户名也必须唯一
  );
  ```



## 3.`PRIMARY KEY`(主键约束)

- **作用**：主键是表中一条记录的**唯一标识符**。

- **特性**：主键约束本质上是 `NOT NULL` 和 `UNIQUE` 的组合。

  1. **唯一性**：主键值必须唯一。
  2. **非空性**：主键值不能为空。

- **重要性**：

  - 每个表**最多只能有一个**主键。
  - 主键是表的核心，是行数据的“身份证号”。
  - 在主键列上，数据库会自动创建一个高效的索引，极大地提升查询速度。

- **语法**：

  ```mysql
  -- 列级约束（单列主键）
  CREATE TABLE 表名 (
      列名 数据类型 PRIMARY KEY
  );
  
  -- 表级约束（用于单列或复合主键）
  CREATE TABLE 表名 (
      列1 数据类型,
      列2 数据类型,
      PRIMARY KEY (列1, 列2) -- 复合主键
  );
  ```

- **示例**：

  ```mysql
  -- 单列主键
  CREATE TABLE products (
      product_id INT PRIMARY KEY,
      product_name VARCHAR(100)
  );
  
  -- 复合主键：由多个列共同组成一个唯一标识
  CREATE TABLE order_items (
      order_id INT,
      product_id INT,
      quantity INT,
      PRIMARY KEY (order_id, product_id) -- 同一个订单中的同一个产品只能有一条记录
  );
  ```



## 4.`FOREIGN KEY`(外键约束)

- **作用**：外键用于在一个表中建立与另一个表主键的关联，以强制实现**参照完整性 (Referential Integrity)**

- **目的**：确保外键列中的值必须在被参照表（父表）的主键列中存在。这可以防止产生无效的关联数据

- **语法**：

  ```mysql
  -- 在创建表时定义
  CREATE TABLE 表名 (
      ...
      [CONSTRAINT 约束名] FOREIGN KEY (本表列名) REFERENCES 父表名(父表主键列名)
  );
  ```

- **示例场景**：一个 `orders`（订单）表，其中有一个 `customer_id` 列。为了确保每个订单都对应一个真实存在的客户，就可以将 `orders.customer_id` 设置为外键，参照 `customers` 表的主键 `id`



- **级联操作 (Cascading Actions)**：可以**在定义外键时**，指定**当父表中的记录被删除或更新时**，**子表中关联的记录应该如何处理**。

  - `ON DELETE ...`

  - `ON UPDATE ...`

  - **可选动作**：

    - `RESTRICT` (默认): 如果存在关联记录，则禁止删除或更新父表记录
    - `CASCADE`: 当父表记录被删除或更新时，自动删除或更新子表中的关联记录
    - `SET NULL`: 当父表记录被删除或更新时，将子表中的外键列设置为 `NULL` (前提是该列允许为 `NULL`)

  - **示例 (带级联删除)**：

    ```MYSQL
    CREATE TABLE order_items (
        item_id INT PRIMARY KEY,
        order_id INT,
        product_info VARCHAR(255),
        FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
    );
    -- 这意味着如果一个订单被删除了，该订单下的所有订单项也会被自动删除。
    ```

    



5.`DEFAULT`(默认值约束)

- **作用**：为列提供一个默认值。如果在**插入新记录时没有为该列指定值，数据库会自动使用这个默认值**。

- **语法**：

  ```MYSQL
  CREATE TABLE 表名 (
      列名 数据类型 DEFAULT 默认值
  );
  ```

- **示例**：

  ```MYSQL
  CREATE TABLE employees (
      id INT PRIMARY KEY,
      name VARCHAR(100) NOT NULL,
      status VARCHAR(20) DEFAULT 'active' -- 员工状态默认为 'active'
  );
  
  -- 下面的插入语句中没有提供 status，它将自动设为 'active'
  INSERT INTO employees (id, name) VALUES (3, '李四');
  ```



## 6.`AUTO_INCREMENT` (自增属性)

- **作用**：**这是一个列属性，而不是严格意义上的约束**。它允许在插入新记录时，由数据库自动为该列生成一个唯一的、递增的数值。

- **使用规则**：

  - 通常用于 `INT` 或 `BIGINT` 类型的**主键列**。
  - 一个表只能有一个 `AUTO_INCREMENT` 列。
  - **该列必须被索引（通常是主键）**

- **语法**：

  ```MYSQL
  CREATE TABLE 表名 (
      列名 INT PRIMARY KEY AUTO_INCREMENT
  );
  ```

- **示例**：

  ```MYSQL
  CREATE TABLE categories (
      category_id INT PRIMARY KEY AUTO_INCREMENT,
      category_name VARCHAR(50) NOT NULL
  );
  
  -- 插入时无需为 category_id 指定值
  INSERT INTO categories (category_name) VALUES ('电子产品'); 	-- id 会自动成为 1
  INSERT INTO categories (category_name) VALUES ('图书');     	  -- id 会自动成为 2
  ```



## 7.`CHECK` 约束

- **作用**：为列中的值设置一个必须满足的布尔表达式。如果表达式的结果为 `FALSE`，则数据库会拒绝该值的插入或更新

- **目的**：实现比基础约束更复杂的业务规则校验

- **版本注意**：`CHECK` 约束在 **MySQL 8.0.16** 版本之后才被真正支持和强制执行。在之前的版本中，虽然可以写 `CHECK` 子句，但它会被忽略，不起作用

- **语法**：

  ```MYSQL
  -- 列级约束
  CREATE TABLE 表名 (
      列名 数据类型 CHECK (表达式)
  );
  
  -- 表级约束
  CREATE TABLE 表名 (
      ...
      CONSTRAINT 约束名 CHECK (表达式)
  );
  ```

- **示例**：

  ```MYSQL
  CREATE TABLE products (
      id INT PRIMARY KEY,
      name VARCHAR(100),
      price DECIMAL(10, 2) CHECK (price >= 0), -- 价格必须大于等于0
      gender CHAR(1) CHECK (gender IN ('M', 'F')) -- 性别只能是'M'或'F'
  );
  
  -- 下面的插入会成功
  INSERT INTO products VALUES (1, '手机', 2999.00, 'M');
  -- 下面的插入会失败，因为 price 违反了 CHECK 约束
  INSERT INTO products VALUES (2, '次品', -100.00, 'F');
  -- 下面的插入会失败，因为 gender 违反了 CHECK 约束
  INSERT INTO products VALUES (3, '电脑', 5000.00, 'X');
  ```



# MySQL 常用内置函数

- MySQL 提供了大量功能强大的内置函数，用于处理字符串、数值、日期等各种数据类型。熟练掌握这些函数，可以极大地简化 SQL 查询和数据处理的复杂度。



## 字符串函数

- 这些函数用于操作和处理字符串（`CHAR`, `VARCHAR`, `TEXT` 等类型）数据

| 函数                                        | 说明                                                         | 示例                                                         |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **`CONCAT(str1, str2, ...)`**               | 将多个字符串连接成一个字符串。                               | `SELECT CONCAT('你好', ', ', '世界');` <br>→ '你好, 世界'    |
| **`CONCAT_WS(separator, str1, str2, ...)`** | 使用指定的分隔符将多个字符串连接起来。                       | `SELECT CONCAT_WS('-', '2025', '08', '14');` <br>→ '2025-08-14' |
| **`LENGTH(str)`**                           | 返回字符串占用的**字节数**。                                 | `SELECT LENGTH('你好abc');` <br>→ 9 (在UTF8中)               |
| **`CHAR_LENGTH(str)`**                      | 返回字符串包含的**字符数**。                                 | `SELECT CHAR_LENGTH('你好abc');` <br>→ 5                     |
| **`UPPER(str)` / `UCASE(str)`**             | 将字符串转换为大写。                                         | `SELECT UPPER('Hello');` <br>→ 'HELLO'                       |
| **`LOWER(str)` / `LCASE(str)`**             | 将字符串转换为小写。                                         | `SELECT LOWER('Hello');` <br>→ 'hello'                       |
| **`SUBSTRING(str, pos, len)`**              | 从字符串中截取子字符串。`pos` 从1开始。                      | `SELECT SUBSTRING('你好世界', 3, 2);` <br>→ '世界'           |
| **`REPLACE(str, from_str, to_str)`**        | 在字符串中查找所有 `from_str`，并替换为 `to_str`。           | `SELECT REPLACE('我爱MySQL', '爱', '喜欢');` <br>→ '我喜欢MySQL' |
| **`TRIM(str)`**                             | 去除字符串两端的空格。                                       | `SELECT TRIM('  hello  ');` <br>→ 'hello'                    |
| **`INSTR(str, substr)`**                    | 返回子字符串 `substr` 在字符串 `str` 中首次出现的位置。找不到则返回0。 | `SELECT INSTR('你好世界', '好世');` <br>→ 2                  |



## 数值函数

- 这些函数用于执行各种数学计算

| 函数                         | 说明                                             | 示例                                                 |
| ---------------------------- | ------------------------------------------------ | ---------------------------------------------------- |
| **`ABS(x)`**                 | 返回 x 的绝对值。                                | `SELECT ABS(-10);` <br>→ 10                          |
| **`ROUND(x, d)`**            | 将数值 x 四舍五入到 d 位小数。                   | `SELECT ROUND(3.14159, 2);` <br>→ 3.14               |
| **`CEIL(x)` / `CEILING(x)`** | 向上取整，返回大于或等于 x 的最小整数。          | `SELECT CEIL(3.14);` <br>→ 4                         |
| **`FLOOR(x)`**               | 向下取整，返回小于或等于 x 的最大整数。          | `SELECT FLOOR(3.99);` <br>→ 3                        |
| **`MOD(n, m)`**              | 取模运算，返回 n 除以 m 的余数。                 | `SELECT MOD(10, 3);` <br>→ 1                         |
| **`RAND()`**                 | 生成一个 0 到 1 之间的随机浮点数。               | `SELECT RAND();`                                     |
| **`FORMAT(x, d)`**           | 将数值 x 格式化，保留 d 位小数，并带千位分隔符。 | `SELECT FORMAT(1234567.89, 2);` <br>→ '1,234,567.89' |



## 日期和时间函数

- 这些函数用于处理日期和时间（`DATE`, `DATETIME`, `TIMESTAMP` 等类型）数据

| 函数                                     | 说明                           | 示例                                                         |
| ---------------------------------------- | ------------------------------ | ------------------------------------------------------------ |
| **`NOW()`**                              | 返回当前的日期和时间。         | `SELECT NOW();` <br>→ '2025-08-14 19:43:00'                  |
| **`CURDATE()`**                          | 返回当前的日期。               | `SELECT CURDATE();` <br>→ '2025-08-14'                       |
| **`CURTIME()`**                          | 返回当前的时间。               | `SELECT CURTIME();` <br>→ '19:43:00'                         |
| **`YEAR(date)`**                         | 从日期中提取年份。             | `SELECT YEAR('2025-08-14');` <br>→ 2025                      |
| **`MONTH(date)`**                        | 从日期中提取月份。             | `SELECT MONTH('2025-08-14');` <br>→ 8                        |
| **`DAY(date)`**                          | 从日期中提取天。               | `SELECT DAY('2025-08-14');` <br>→ 14                         |
| **`DATE_FORMAT(date, format)`**          | 将日期按指定格式进行格式化。   | `SELECT DATE_FORMAT(NOW(), '%Y年%m月%d日');` <br>→ '2025年08月14日' |
| **`DATEDIFF(date1, date2)`**             | 计算两个日期之间的天数差。     | `SELECT DATEDIFF('2025-08-20', '2025-08-14');` <br>→ 6       |
| **`DATE_ADD(date, INTERVAL expr unit)`** | 在一个日期上增加一个时间间隔。 | `SELECT DATE_ADD('2025-08-14', INTERVAL 1 MONTH);` <br>→ '2025-09-14' |
| **`DATE_SUB(date, INTERVAL expr unit)`** | 在一个日期上减去一个时间间隔。 | `SELECT DATE_SUB('2025-08-14', INTERVAL 7 DAY);` <br>→ '2025-08-07' |



## 流程控制函数

- 在 SQL 查询中，流程控制函数允许我们根据特定条件执行不同的逻辑，从而实现更灵活和强大的数据处理。这类似于在编程语言中使用 `if-else` 或 `switch` 语句。MySQL 提供了几个非常有用的流程控制函数，下面我们来详细探讨。

### IF(expr, v1, v2)

- `IF()` 函数是一个基础的条件判断函数，可以看作是三元运算符的 SQL 版本。

#### **语法解析：**

- `expr`: 一个表达式，其结果将被判断为 `TRUE` 或 `FALSE`。
- `v1`: 如果 `expr` 的结果为 `TRUE`，则函数返回该值。
- `v2`: 如果 `expr` 的结果为 `FALSE`，则函数返回该值。

`v1` 和 `v2` 的数据类型需要兼容。

#### **使用场景：** 

- 适用于简单的二选一逻辑判断，例如“是/否”、“通过/不通过”、“男/女”等。

#### **示例：**

1. **基础示例：** 判断 1 是否大于 0，如果是，返回 '是'，否则返回 '否'。

   ```sql
   SELECT IF(1 > 0, '是', '否');
   -- 结果: '是'
   ```

2. **实际应用：** 假设我们有一个 `students` 表，包含 `student_name` 和 `score` 字段。我们想根据学生的分数是否及格（大于等于60分）来显示“及格”或“不及格”。

   ```sql
   -- 假设有如下数据：
   -- student_name | score
   -- '张三'         | 85
   -- '李四'         | 52
   
   SELECT
       student_name,
       score,
       IF(score >= 60, '及格', '不及格') AS status
   FROM
       students;
   
   -- 查询结果:
   -- student_name | score | status
   -- '张三'         | 85    | '及格'
   -- '李四'         | 52    | '不及格'
   ```



### IFNULL(v1, v2)

- `IFNULL()` 函数专门用于处理 `NULL` 值，当一个字段或表达式可能为 `NULL` 时，可以用它来提供一个默认值。

#### **语法解析：**

- `v1`: 需要检查是否为 `NULL` 的表达式。
- `v2`: 如果 `v1` 的值为 `NULL`，则函数返回该值。如果 `v1` 不为 `NULL`，则函数返回 `v1` 自身。

#### **使用场景：**

- 在查询结果中，`NULL` 值通常不直观，使用 `IFNULL()` 可以将其替换为更有意义的文本，如“未提供”、“默认值”或 `0`，从而提高报表的可读性。

#### **示例**

1. **基础示例：** 检查一个值是否为 `NULL`，如果是，则返回 '默认值'。

   ```sql
   SELECT IFNULL(NULL, '默认值');
   -- 结果: '默认值'
   
   SELECT IFNULL('Hello', '默认值');
   -- 结果: 'Hello'
   ```

2. **实际应用：** 假设我们有一个 `products` 表，其中 `description` 字段允许为 `NULL`。当查询产品信息时，我们不希望描述显示为 `NULL`。

   ```sql
   -- 假设有如下数据：
   -- product_name | description
   -- '笔记本电脑'   | '高性能游戏本'
   -- '鼠标'         | NULL
   
   SELECT
       product_name,
       IFNULL(description, '暂无描述') AS product_description
   FROM
       products;
   
   -- 查询结果:
   -- product_name | product_description
   -- '笔记本电脑'   | '高性能游戏本'
   -- '鼠标'         | '暂无描述'
   ```



### CASE ... END

- `CASE` 表达式是 SQL 中实现复杂条件逻辑的标准方式，比 `IF()` 函数更强大，可以处理多个条件分支。它有两种主要形式

#### 1. 简单 CASE 表达式

- 这种形式将一个表达式的值与一系列 `WHEN` 子句中的值进行比较

##### **语法解析**

```sql
CASE case_value
    WHEN when_value1 THEN result1
    WHEN when_value2 THEN result2
    ...
    ELSE else_result
END
```

- 它会计算 `case_value` 的值，然后依次与 `when_value1`, `when_value2` 等进行比较
- 如果匹配成功，则返回对应的 `result`
- 如果没有匹配成功，则返回 `ELSE` 子句中的 `else_result`。`ELSE` 是可选的，如果省略且没有匹配项，则返回 `NULL`



##### **示例**

- 根据用户的等级（level）显示不同的中文名称。

```sql
-- 假设有 users 表，包含 user_name 和 level (1, 2, 3)
SELECT
    user_name,
    level,
    CASE level
        WHEN 1 THEN '初级会员'
        WHEN 2 THEN '中级会员'
        WHEN 3 THEN '高级会员'
        ELSE '未知等级'
    END AS level_name
FROM
    users;
```



#### 2. 搜索 CASE 表达式

- 这种形式更灵活，它不依赖于单个值的比较，而是允许在每个 `WHEN` 子句中定义独立的布尔表达式

##### **语法解析**

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE else_result
END
```

- 它会依次评估 `condition1`, `condition2` 等条件。
- 返回第一个为 `TRUE` 的条件所对应的 `result`。
- 如果所有条件都不为 `TRUE`，则返回 `ELSE` 子句中的 `else_result`。

##### **示例**

- 根据学生的分数范围来评定等级：优秀、良好、及格、不及格。

```sql
SELECT
    student_name,
    score,
    CASE
        WHEN score >= 90 THEN '优秀'
        WHEN score >= 80 THEN '良好'
        WHEN score >= 60 THEN '及格'
        ELSE '不及格'
    END AS grade
FROM
    students;
```

- 这个例子就无法用简单的 `IF()` 函数直接实现，充分体现了 `CASE` 表达式的强大之处。



## 聚合函数

- 这些函数对一组行的某个列进行计算，并返回一个单一的值，通常与 `GROUP BY` 配合使用

| 函数            | 说明                     |
| --------------- | ------------------------ |
| **`COUNT()`**   | 用于统计行数。           |
| **`SUM(列名)`** | 计算指定列的数值总和。   |
| **`AVG(列名)`** | 计算指定列的数值平均值。 |
| **`MAX(列名)`** | 找出指定列中的最大值。   |
| **`MIN(列名)`** | 找出指定列中的最小值。   |



## 信息函数

- 这些函数用于获取 MySQL 服务器的元数据或状态信息

| 函数                            | 说明                            | 示例                 |
| ------------------------------- | ------------------------------- | -------------------- |
| **`DATABASE()`**                | 返回当前默认数据库的名称。      | `SELECT DATABASE();` |
| **`USER()` / `CURRENT_USER()`** | 返回当前登录用户的账户信息。    | `SELECT USER();`     |
| **`VERSION()`**                 | 返回当前 MySQL 服务器的版本号。 | `SELECT VERSION();`  |



# 高级查询(DQL)

- DQL 的高级功能使得我们能够处理更复杂的查询需求，从数据中提取出更有价值的信息

  ![image-20250814214911783](./assets/image-20250814214911783.png)

## 1.去除重复行 (`DISTINCT`)

- **作用**：用于从 `SELECT` 查询结果中移除重复的行。它作用于所有指定的列，只有当所有选择的列的值都相同时，才被认为是重复行。

- **语法**：

  ```mysql
  SELECT DISTINCT 列1, 列2, ... FROM 表名;
  ```

- **示例**：查询所有学生所在的班级ID（不重复）。

  ```mysql
  -- 假设 students 表中 class_id 有多个 '1' 和 '2'
  SELECT DISTINCT class_id FROM students;
  -- 结果只会返回 1, 2, NULL 各一次
  ```



## 2.排序与分页 (`ORDER BY`, `LIMIT`)

### **`ORDER BY` (排序)**

#### 基础概念

- **作用**：用于对查询结果集进行排序

- **语法**：

  ```mysql
  SELECT 列列表 FROM 表名
  [WHERE 条件]
  ORDER BY 排序列1 [ASC|DESC], 排序列2 [ASC|DESC], ...;		-- 这里支持多列排序
  ```

- **关键字**：

  - `ASC` (Ascending): 升序排序，这是**默认值**
  - `DESC` (Descending): 降序排序

- **多列排序**：可以按多个列进行排序。**排序时，会首先按照第一个列排序，当第一个列的值相同时，再按照第二个列排序**，以此类推

- **示例**：

  - 为了演示，我们假设有如下 `students` 表：

    | id   | name | age  | score |
    | :--- | :--- | :--- | :---- |
    | 1    | 张三 | 20   | 85    |
    | 2    | 李四 | 19   | 92    |
    | 3    | 王五 | 20   | 92    |

  ```mysql
  -- 按分数降序排序，如果分数相同，则按年龄升序排序
  SELECT name, score, age FROM students ORDER BY score DESC, age ASC;
  -- 结果: 李四(92, 19), 王五(92, 20), 张三(85, 20)
  ```



#### 排序规则

- `ORDER BY` 子句用于对 `SELECT` 查询返回的结果集进行排序。其排序行为遵循以下核心规则：

##### 1. 按数据类型排序

- **数值类型**
  - 严格按照数值大小进行排序
- **字符串类型**
  - 根据当前数据库或列指定的**字符集排序规则 (Collation)** 进行排序
  - 对于英文字符，通常按字典顺序（A-Z）排序
  - 对于中文字符，通常按拼音或笔画顺序排序，具体取决于排序规则的设置
- **日期和时间类型**
  - 严格按照时间上的先后顺序进行排序（从最早到最晚）

##### 2. 多列排序规则

- 当 `ORDER BY` 后跟有多个列时，排序具有明确的**优先级**。
- 首先完全按照第一个列进行排序。
- 仅当第一个列出现相同值时，才会在这些具有相同值的行内部，应用第二个列的排序规则。
- 以此类推，后续的排序列只在前一排序列的值相等时才生效。

##### 3. `NULL` 值的处理规则

- 在 MySQL 中，`NULL` 被认为是“最小值”。
- 因此，在**升序 (`ASC`)** 排序中，`NULL` 值的行会出现在**最前面**。
- 在**降序 (`DESC`)** 排序中，`NULL` 值的行会出现在**最后面**。

##### 4. 自定义排序规则

- 可以通过流程控制函数（如 `CASE ... END`）或特定函数（如 `FIELD()`）来定义不依赖于数据类型本身的、完全自定义的排序逻辑。



### **`LIMIT` (限制结果数量/分页)**

- **作用**：用于限制 `SELECT` 语句返回的记录数量。它有两个核心应用场景：获取最上面的N个记录(一般获取前几名?)和实现分页

- **语法**：`LIMIT` 子句接受一个或两个**非负整数参数**

  - **语法一：`LIMIT 数量`**

    - **含义**：从结果集的第 1 行开始，返回最多 `数量` 条记录。

    - **示例**：查询分数最高的 2 位学生。

      ```mysql
      SELECT name, score FROM students ORDER BY score DESC LIMIT 2;
      ```

  - **语法二：`LIMIT 偏移量, 数量`**

    > 我理解的偏移量是离某个地方的距离，比如从3开始，跳过1和2，偏移量就是2

    - **含义**：从结果集的第 `偏移量 + 1` 行开始（**注意：偏移量从 0 开始计数**），返回最多 **`数量`** 条记录

    - **示例**：从第 3 位学生开始，查询 2 位学生的信息（即第3和第4位）

      ```mysql
      SELECT name, score FROM students ORDER BY id LIMIT 2, 2;
      ```

      

- **分页逻辑详解**：这是 `LIMIT` 最常见的用途

  - **场景**：假设一个网站的文章列表，每页需要显示 5 篇文章

  - **已知变量**：

    - `pageSize` (每页数量) = 5
    - `pageNow` (当前页码)

  - **计算偏移量**：

    ```java
    offset = (pageNow - 1) * pageSize
    ```

  - **查询第 1 页 (pageNo = 1)**：

    - `offset = (1 - 1) * 5 = 0`
    - SQL: `SELECT * FROM articles ORDER BY publish_date DESC LIMIT 0, 5;`

  - **查询第 2 页 (pageNo = 2)**：

    - `offset = (2 - 1) * 5 = 5`
    - SQL: `SELECT * FROM articles ORDER BY publish_date DESC LIMIT 5, 5;`

  - **查询第 3 页 (pageNo = 3)**：

    - `offset = (3 - 1) * 5 = 10`
    - SQL: `SELECT * FROM articles ORDER BY publish_date DESC LIMIT 10, 5;`



- **重要提示：`LIMIT` 必须与 `ORDER BY` 配合使用**

  - 如果不使用 `ORDER BY` 对结果进行排序，数据库返回行的顺序是**不确定**的
  - 在一个不确定的顺序上使用 `LIMIT` 进行分页，会导致每次查询时，同一页的数据都可能不同，造成数据错乱
  - **规则**：为了保证分页结果的稳定性和可预测性，**务必**在使用 `LIMIT` 前，先用 `ORDER BY` 对结果进行一个确定的排序（通常是按主键 `id` 或创建时间 `create_time`）

  

## 3.聚合函数与分组

- 聚合函数与分组是 SQL 数据分析与报表生成的核心。它们允许我们将多行数据汇总成单一的统计值，从而揭示数据的整体趋势和特征

### 聚合函数

- 聚合函数对一组行的某个列进行计算，并返回一个单一的值

- **示例表 (`products` 表)**

  | id   | name | category | price | stock |
  | :--- | :--- | :------- | :---- | :---- |
  | 1    | 手机 | 电子产品 | 5200  | 100   |
  | 2    | 耳机 | 电子产品 | 200   | 300   |
  | 3    | 键盘 | 电子产品 | NULL  | 50    |
  | 4    | 牛奶 | 食品     | 60    | 200   |
  | 5    | 面包 | 食品     | 10    | 500   |



#### **常用聚合函数表**

| 函数 (Function) | 作用                   | 参数类型         | 对NULL的处理                      | 示例                                                         |
| --------------- | ---------------------- | ---------------- | --------------------------------- | ------------------------------------------------------------ |
| **`COUNT()`**   | 用于统计行数           | 任何类型         | `COUNT(*)`不忽略，`COUNT(列)`忽略 | 见下方详解                                                   |
| **`SUM(列名)`** | 计算指定列的数值总和   | 数值类型         | 忽略 `NULL`                       | `SELECT SUM(stock) FROM products;` (结果: 1150)              |
| **`AVG(列名)`** | 计算指定列的数值平均值 | 数值类型         | 忽略 `NULL`                       | `SELECT AVG(price) FROM products;` (结果: (5200+200+60+10)/4 = 1367.5) |
| **`MAX(列名)`** | 找出指定列中的最大值   | 数值/字符串/日期 | 忽略 `NULL`                       | `SELECT MAX(price) FROM products;` (结果: 5200)              |
| **`MIN(列名)`** | 找出指定列中的最小值   | 数值/字符串/日期 | 忽略 `NULL`                       | `SELECT MIN(price) FROM products;` (结果: 10)                |



#### **`COUNT()` 的三种用法详解**

- `COUNT()` 是**最常用但也最容易混淆的聚合函数**，其行为因参数不同而异

  1. **`COUNT(*)`**

     - **作用**：返回表中的**总行数**，无论行中是否包含 `NULL` 值。

     - **示例**：`SELECT COUNT(*) FROM products;` (结果: 5)

     - **最佳实践**：当你只想知道表的总记录数时，这是最推荐、**效率最高**的方式

       - 这个`COUNT(*)`效率高，不像`SELECT *`效率低

       

  2. **`COUNT(列名)`**

     - **作用**：返回指定列中**非 `NULL` 值**的行数。它会忽略所有 `NULL` 值。

     - **示例**：`SELECT COUNT(price) FROM products;` (结果: 4, 因为价格为NULL的键盘被忽略了)

       

  3. **`COUNT(DISTINCT 列名)`**

     - **作用**：返回指定列中**不重复的、非 `NULL` 值**的行数。
     - **示例**：`SELECT COUNT(DISTINCT category) FROM products;` (结果: 2, 因为只有'电子产品'和'食品'两种)



### 分组

#### `GROUP BY`(分组)

- **作用**：`GROUP BY` 子句通常与聚合函数一起使用，它会根据一个或多个列的值，将结果集中的行分成不同的组，然后对**每个组**分别执行聚合函数

  - **`GROUP BY` 在它的工作过程中，其效果就包含了对分组列的“去重”**

- **语法**:

  ```mysql
  SELECT 列1, 聚合函数(列2)
  FROM 表名
  [WHERE 条件]
  GROUP BY 列1, 列3, ...;
  ```

- **核心规则**：所有**在 `SELECT` 子句中出现的、未被聚合函数包裹的列，都必须出现在 `GROUP BY` 子句中**

  - **原因**：因为 `GROUP BY` 已经**将多行合并为一行(”去重“)**，对于非聚合列，数据库不知道应该显示哪一行的值。通过将其放入 `GROUP BY`，你告诉数据库这些列的值在组内是相同的

- **示例**：

  1. **按类别统计商品数量**

     ```mysql
     SELECT category, COUNT(*) AS product_count
     FROM products
     GROUP BY category;
     ```

     - **结果**

       | category | product_count |
       | :------- | :------------ |
       | 电子产品 | 3             |
       | 食品     | 2             |

  2. **按类别计算平均价格**

     ```mysql
     SELECT category, AVG(price) AS avg_price
     FROM products
     GROUP BY category;
     ```

     - **结果**

       | category | avg_price |
       | :------- | :-------- |
       | 电子产品 | 2700.00   |
       | 食品     | 35.00     |



#### `HAVING` (分组后过滤)

- **作用**：**`WHERE` 子句在分组前过滤原始数据行**，而 **`HAVING` 子句在 `GROUP BY` 之后**，用于**过滤分组后的结果**

- **关键区别**：`HAVING` 的条件中**可以**使用聚合函数，而 `WHERE` **不能**

- **语法**:

  ```mysql
  SELECT 列1, 聚合函数(列2)
  FROM 表名
  GROUP BY 列1
  HAVING 分组条件;
  ```

- **示例**：找出商品数量超过2个的类别。

  ```mysql
  SELECT category, COUNT(*) AS product_count
  FROM products
  GROUP BY category
  HAVING COUNT(*) > 2; -- 或者 HAVING product_count > 2
  ```

  - **结果**

    | category | product_count |
    | :------- | :------------ |
    | 电子产品 | 3             |



#### `WHERE` vs `HAVING` 深度对比

| 对比项             | `WHERE` 子句                       | `HAVING` 子句                    |
| ------------------ | ---------------------------------- | -------------------------------- |
| **执行时机**       | 在 `GROUP BY` 分组**之前**执行     | 在 `GROUP BY` 分组**之后**执行   |
| **作用对象**       | 作用于表中的**原始数据行**         | 作用于 `GROUP BY` 生成的**分组** |
| **能否用聚合函数** | **不能**，因为此时还未进行聚合计算 | **可以**，因为此时聚合计算已完成 |

- **综合示例**：查询 `products` 表中，价格不为空的商品里，平均价格高于1000元的商品类别。

  ```mysql
  SELECT 
      category, 
      AVG(price) AS avg_price
  FROM 
      products
  WHERE 
      price IS NOT NULL   -- 步骤1: 先用 WHERE 过滤掉原始数据中价格为空的行
  GROUP BY 
      category            -- 步骤2: 对过滤后的结果按类别分组
  HAVING 
      AVG(price) > 1000;  -- 步骤3: 对分组后的结果，用 HAVING 筛选出平均价大于1000的组
  ```

  - **结果**:

    | category | avg_price |
    | :------- | :-------- |
    | 电子产品 | 2700.00   |



## 4.多表连接查询 (`JOIN`)

- 连接查询是关系型数据库的核心，它允许我们根据某些关联条件，从两个或多个表中检索数据。连接的本质是根据表与表之间的关系，将它们虚拟地组合成一个更大的数据表，以便进行查询。

  - **示例表**：

    - **students 表**

      | id   | name | class_id |
      | ---- | ---- | -------- |
      | 1    | 张三 | 1        |
      | 2    | 李四 | 1        |
      | 3    | 王五 | 2        |
      | 4    | 赵六 | NULL     |

    - **classes 表**

      | id   | class_name |
      | ---- | ---------- |
      | 1    | 一班       |
      | 2    | 二班       |
      | 3    | 三班       |



### **`INNER JOIN` (内连接)**

- **作用**：只返回两个表中连接字段相匹配的行。可以理解为取两个表的“交集”。如果一个表中的某行在另一个表中没有匹配的行，那么这行数据就不会出现在结果中

  - 这个`INNER JOIN`可以缩写成`JOIN`
  - 不像`LEFT JOIN`和`RIGHT JOIN`保留左表或右表的所有数据，这个`INNER JOIN`只保留匹配上的数据

- **语法**：

  ```mysql
  SELECT 列列表
  FROM 表1
  INNER JOIN 表2 ON 连接条件;
  	-- "连接条件"是一个返回 TRUE 或 FALSE 的逻辑表达式，例如 s.class_id = c.id
  	-- "INNER" 关键字是可选的，可以直接写成 "JOIN"
  ```

- **示例**：查询所有有班级的学生及其班级名称。

  ```mysql
  SELECT s.name, c.class_name
  FROM students s
  INNER JOIN classes c ON s.class_id = c.id;
  	-- 结果: 张三(一班), 李四(一班), 王五(二班)
  	-- (学生"赵六"因为 class_id 为 NULL，没有匹配到任何班级，所以不会出现在结果中)
  	-- ("三班"因为在 students 表中没有对应的学生，所以也不会出现在结果中)
  ```



### **`LEFT JOIN` (左连接)**

#### 基本

- **作用**：返回左表（FROM 子句后的第一个表）的**所有行**，以及右表中与连接字段相匹配的行。如果右表中没有匹配项，则结果中右表的列为 `NULL`。它常用于“查找一个表中存在，而另一个表中不存在”的场景

- **语法**：

  ```mysql
  SELECT 列列表
  FROM 左表
  LEFT JOIN 右表 ON 连接条件;
  	-- "连接条件"是一个返回 TRUE 或 FALSE 的逻辑表达式。
  	-- LEFT JOIN 也可以写成 LEFT OUTER JOIN, "OUTER" 关键字是可选的
  ```

  

- **示例 1**：查询所有学生，以及他们对应的班级名称（即使学生没有班级）。

  ```mysql
  SELECT s.name, c.class_name
  FROM students s
  LEFT JOIN classes c ON s.class_id = c.id;
  	-- 结果: 张三(一班), 李四(一班), 王五(二班), 赵六(NULL)
  	-- (左表 students 的所有记录都被返回了，包括没有班级的赵六)
  ```

- **示例 2**：查询哪些学生没有分配班级。

  ```mysql
  SELECT s.name
  FROM students s
  LEFT JOIN classes c ON s.class_id = c.id
  WHERE c.id IS NULL; -- 关键的过滤条件
  	-- 结果: 赵六
  	-- (通过判断右表匹配的列是否为 NULL，可以筛选出左表中独有的记录)
  ```

  

#### **深入理解 LEFT JOIN 的本质**

- 底层逻辑上理解 `LEFT JOIN` ，可以将其看作一个“两步走”的过程，这个过程始于两个表的笛卡尔积（所有可能的行组合）
  - **第一步：执行一次 `INNER JOIN`**
    - 首先，数据库会找出笛卡尔积中所有满足 **`ON` 连接条件**的行。这会得到所有左右表能够成功匹配上的数据。这部分的结果与 `INNER JOIN` 完全相同
  - **第二步：补回左表中“掉队”的行**
    - 接下来，数据库会回头检查**左表**，找出在第一步中**没有**找到任何匹配项的行（未匹配行）
    - 对于每一个未匹配的行，数据库会将其“捞起”，保留左表的数据，并手动将所有来自**右表**的列填充为 `NULL`
    - 最后，将这些补回的行添加到第一步的结果集中，形成最终的 `LEFT JOIN` 结果
- **结论总结**:
  - **`INNER JOIN` 的本质** = **笛卡尔积** + **`ON` 条件过滤**
  - **`LEFT JOIN` 的本质** = **`INNER JOIN` 的结果** + **补回左表未匹配的行并填充 `NULL`**
  - 这个“补回”操作，正是 `LEFT JOIN` 能够保留左表所有行的核心所在

- **关于左表行重复的说明**
  - 左表中的一行**不一定**在结果中只出现一次
  - 如果左表的一行在右表中找到了 **N** 个匹配项 (N > 1)，那么该行在结果中将会被**复制 N 次**，分别与右表的 N 个匹配行组合。



### **`RIGHT JOIN` (右连接)**

- **作用**：返回右表（`JOIN` 子句后的表）的**所有行**，以及左表中与连接字段相匹配的行。如果左表中没有匹配项，则结果中左表的列为 `NULL`。`RIGHT JOIN` 的逻辑与`LEFT JOIN` 相反。

- **语法**：

  ```mysql
  SELECT 列列表
  FROM 左表
  RIGHT JOIN 右表 ON 连接条件;
  -- "连接条件"是一个返回 TRUE 或 FALSE 的逻辑表达式。
  -- RIGHT JOIN 也可以写成 RIGHT OUTER JOIN, "OUTER" 关键字是可选的
  ```

- **示例 1**：查询所有班级，以及班级里的学生（即使班级里没有学生）。

  ```mysql
  SELECT s.name, c.class_name
  FROM students s
  RIGHT JOIN classes c ON s.class_id = c.id;
  	-- 结果: 张三(一班), 李四(一班), 王五(二班), NULL(三班)
  	-- (右表 classes 的所有记录都被返回了，包括没有学生的三班)
  ```

- **示例 2**：查询哪些班级是“空班”，即没有任何学生。

  ```mysql
  SELECT c.class_name
  FROM students s
  RIGHT JOIN classes c ON s.class_id = c.id
  WHERE s.id IS NULL; -- 关键的过滤条件
    -- 结果: 三班
  ```

  - **事实上**，`RIGHT JOIN` 并不常用，因为任何 `RIGHT JOIN` 都可以通过交换表的位置改写成一个等价的 `LEFT JOIN`
    - 例如，上面的示例 1 等价于`SELECT s.name, c.class_name FROM classes c LEFT JOIN students s ON c.id = s.class_id;`
    - 使用 `LEFT JOIN` 通常更符合阅读习惯



### **`CROSS JOIN` (交叉连接 / 笛卡尔积)**

- **作用**：返回左表中的每一行与右表中的每一行的所有可能组合。结果集的行数等于左表行数乘以右表行数。它在缺少连接条件时会产生笛卡尔积。

- **语法**：

  ```mysql
  -- 显式语法
  SELECT * FROM 表1 CROSS JOIN 表2;
  
  -- 隐式语法 (旧式写法，不推荐)
  SELECT * FROM 表1, 表2;
  ```

- **注意**：`CROSS JOIN` 一般不带 `ON` 子句。如果带了 `ON` 子句，其效果就等同于 `INNER JOIN`。它在某些特定场景下有用（如生成测试数据、数据初始化等），但要小心使用，因为它可能在表行数较多时产生非常庞大的结果集，消耗大量服务器资源。

- **示例**：将每个学生与每个班级都组合一次。

  ```mysql
  SELECT s.name, c.class_name
  FROM students s
  CROSS JOIN classes c;
    -- 结果会是 4 * 3 = 12 条记录
    -- 张三(一班), 张三(二班), 张三(三班)
    -- 李四(一班), 李四(二班), 李四(三班)
    -- ... 等等
  ```



### SELF JOIN (自连接)

- **作用**：自连接不是一种新的 `JOIN` 类型，而是一种特殊的用法：**将一个表与它自身进行连接**。这通常用于处理表内具有层级或递归关系的数据。

- **实现**：在 FROM 子句中，必须为同一个表使用两个不同的别名，从而在逻辑上将其视为两个独立的表进行操作。

- **场景**：常用于处理表内具有层级关系的数据，如员工与其经理、商品分类的父子关系、区域的省市关系等。

- **示例**：假设 employees 表有 id, name, manager_id (经理的id，该id也存在于本表的id列中)，查询每个员工及其直属经理的姓名。

  - **employees 示例表**

    | id   | name  | manager_id |
    | ---- | ----- | ---------- |
    | 1    | 老板A | NULL       |
    | 2    | 经理B | 1          |
    | 3    | 员工C | 2          |
    | 4    | 员工D | 2          |

  ```mysql
  SELECT
      e.name AS 员工姓名,
      m.name AS 经理姓名
  FROM
      employees e -- 将表作为"员工表"
  LEFT JOIN -- 使用 LEFT JOIN 以确保没有经理的员工（如老板）也会被列出
      employees m ON e.manager_id = m.id; -- 将同一个表作为"经理表"
  
    -- 结果:
    -- 老板A, NULL
    -- 经理B, 老板A
    -- 员工C, 经理B
    -- 员工D, 经理B
  ```



### **深入理解 `ON` 连接条件**

- **核心概念**：ON 子句中的条件**并不只局限于等号 =**。它是一个通用的条件表达式，可以使用几乎所有标准的SQL运算符，从而实现更复杂的连接逻辑。

- **可用运算符**：

  - **比较运算符**: `>` (大于), `<` (小于), `>=` (大于等于), `<=` (小于等于), `!=` 或 `<>` (不等于)。
  - **范围运算符**: `BETWEEN ... AND ...`
  - **列表运算符**: `IN`
  - **模式匹配**: `LIKE`
  - **逻辑运算符**: `AND`, `OR`, `NOT` (用于组合多个条件)。

- **连接分类**:

  - **等值连接 (Equi-join)**: 使用 = 作为连接条件，这是最常见的情况，通常用于主外键关联。
  - **非等值连接 (Non-equi-join)**: 使用除 = 之外的运算符进行连接。在处理**范围匹配**、**时间段重叠**等场景时非常强大。

- **非等值连接的实用示例**：

  - **场景1：薪资与等级匹配**
    假设需要根据员工的薪资，为其匹配对应的薪资等级。

    - **employees 表**

      | id   | name | salary |
      | :--- | :--- | :----- |
      | 1    | 张三 | 8500   |
      | 2    | 李四 | 12000  |
      | 3    | 王五 | 4800   |

    - **salary_grades 表**

      | grade | min_salary | max_salary |
      | :---- | :--------- | :--------- |
      | B     | 10000      | 19999      |
      | C     | 5000       | 9999       |
      | D     | 0          | 4999       |

    - **查询**：

      ```mysql
      SELECT
          e.name,
          e.salary,
          g.grade
      FROM
          employees e
      JOIN
          salary_grades g ON e.salary BETWEEN g.min_salary AND g.max_salary;
          -- 或者写成 ON e.salary >= g.min_salary AND e.salary <= g.max_salary
      ```

    - **结果**：

      | name | salary | grade |
      | :--- | :----- | :---- |
      | 张三 | 8500   | C     |
      | 李四 | 12000  | B     |
      | 王五 | 4800   | D     |

    

  - **场景2：使用自连接查找重复项**
    在一个表中查找 id 不同但内容相似的记录。

    - **products 表**

      | id   | product_name |
      | :--- | :----------- |
      | 101  | 苹果手机 12  |
      | 102  | 苹果手机12   |
      | 205  | 三星电视     |

    - **查询**：

      ```mysql
      SELECT
          p1.id AS id_1,
          p1.product_name AS name_1,
          p2.id AS id_2,
          p2.product_name AS name_2
      FROM
          products p1
      JOIN
          products p2 ON p1.product_name = p2.product_name -- 条件1: 名称相同
                      AND p1.id < p2.id; -- 条件2: ID不同，且避免重复匹配
      ```

    - **解释**：`p1.id < p2.id` 这个非等值条件非常关键，它同时实现了两个目的：

      1. **避免自我比较** (`p1.id` 不会等于 `p2.id`)。
      2. **避免结果重复** (如果已有 (101, 102)，就不会再出现 (102, 101))。



## 5.子查询

- **作用**：子查询是嵌套在另一个 SQL 查询（主查询）中的查询。它可以出现在 SELECT、FROM、WHERE 或 HAVING 子句中，为主查询提供数据或作为过滤条件。

- **分类与位置**：

  - **标量子查询**: 返回单个值（一行一列）
    - 常用于 WHERE 和 SELECT 子句中
  - **列子查询**: 返回单列多行
    - 常与 IN、ANY、ALL 操作符一起在 WHERE 子句中使用
  - **行子查询**: 返回单行多列
    - 常用于 WHERE 子句中进行多列值的比较
  - **表子查询**: 返回一个虚拟表（多行多列）
    - 常用于 FROM 子句中，必须给子查询结果指定一个别名

- **示例 1 (标量子查询)**：查询分数比所有学生的平均分要高的学生

  ```mysql
  -- 子查询 (SELECT AVG(score) FROM students) 返回一个单独的平均分数值
  SELECT name, score
  FROM students
  WHERE score > (SELECT AVG(score) FROM students);
  ```

- **示例 2 (列子查询)**：查询与“张三”在同一个班级的所有其他学生。

  ```mysql
  -- 子查询 (SELECT class_id FROM students WHERE name = '张三') 返回 '张三' 所在的班级ID列表
  SELECT name
  FROM students
  WHERE class_id IN (SELECT class_id FROM students WHERE name = '张三')
    AND name <> '张三'; -- 排除张三本人
  ```

- **示例 3 (表子查询)**：查询每个班级的平均分，并将其作为一张虚拟表与学生表连接。

  ```mysql
  -- 子查询部分先计算出每个班级的平均分，形成一个临时表 avg_scores
  SELECT
      s.name,
      s.score,
      avg_s.class_avg_score
  FROM
      students s
  JOIN
      (SELECT class_id, AVG(score) AS class_avg_score FROM students GROUP BY class_id) AS avg_s
  ON
      s.class_id = avg_s.class_id;
  ```

- **相关子查询 (Correlated Subquery)**：

  - 这是一种特殊的子查询，它的执行依赖于主查询。主查询的每一行都会触发子查询执行一次。

  - **示例**：查询每个班级中分数最高的学生。

    ```mysql
    SELECT name, score, class_id
    FROM students s1
    WHERE score = (
        SELECT MAX(score)
        FROM students s2
        WHERE s2.class_id = s1.class_id -- 子查询依赖于主查询的 s1.class_id
    );
    ```

  - **注意**：相关子查询通常效率较低，因为其执行频率很高。在很多情况下，可以通过 `JOIN` 或窗口函数等方式进行优化，以获得更好的性能。



## 6.集合操作 (`UNION`, `UNION ALL`)

- **作用**：用于合并两个或多个 `SELECT` 语句的结果集。当你需要将来自不同表、结构相似的数据纵向拼接在一起时，这个操作非常有用。

- **使用规则**：

  1. 所有 `SELECT` 语句必须拥有**相同数量的列**。
  2. 所有 `SELECT` 语句中对应列的**数据类型必须相似**或可以相互转换。
  3. 每个 `SELECT` 语句中的列的**顺序必须相同**。

- **示例准备**： 为了演示，我们假设有两张表：`students` 表和 `teachers` 表，它们都有一个 `name` 列。

  - **`students` 表** 

    | name |
    | :--- |
    | 张三 |
    | 李四 |
    | 王五 |

  - **`teachers` 表** 

    | name   |
    | :----- |
    | 张三   |
    | 赵老师 |



### **`UNION` (合并并去重)**

- **作用**：合并结果集，并**自动去除重复的行**。它会进行一次排序和去重操作，因此性能相对较低。

- **语法**：

  ```mysql
  SELECT 列1, 列2 FROM 表1
  UNION
  SELECT 列1, 列2 FROM 表2;
  ```

- **示例**：合并学生和老师的姓名，并去除重复的名字。

  ```mysql
  SELECT name FROM students
  UNION
  SELECT name FROM teachers;
  ```

- **结果分析**： `students` 表有【张三, 李四, 王五】，`teachers` 表有【张三, 赵老师】。`UNION` 会将它们合并为【张三, 李四, 王五, 张三, 赵老师】，然后发现“张三”是重复的，于是将其去除一个。 

  - **最终结果 (4行)**

    | name   |
    | :----- |
    | 张三   |
    | 李四   |
    | 王五   |
    | 赵老师 |



### **`UNION ALL` (全部合并)**

- **作用**：合并结果集，但**保留所有行，包括重复的行**。它只是简单地将结果拼接在一起，不进行去重，因此效率比 `UNION` 高。

- **语法**：

  ```mysql
  SELECT 列1, 列2 FROM 表1
  UNION ALL
  SELECT 列1, 列2 FROM 表2;
  ```

- **示例**：合并学生和老师的姓名，保留所有记录。

  ```mysql
  SELECT name FROM students
  UNION ALL
  SELECT name FROM teachers;
  ```

- **结果分析**： `UNION ALL` 直接将两个表的结果拼接在一起，不做任何处理。

  - **最终结果 (5行)**

    | name   |
    | :----- |
    | 张三   |
    | 李四   |
    | 王五   |
    | 张三   |
    | 赵老师 |



### **总结与选择**

- 当你确定合并的结果中没有重复数据，或者你希望保留所有重复数据时，**优先使用 `UNION ALL`**，因为它的性能更好。
- 当你需要一个干净、无重复的合并结果时，使用 `UNION`。



## SQL 逻辑查询执行顺序

- 这是SQL执行顺序，不是代码的书写顺序
  1. **`FROM` / `JOIN`s**: 确定要操作的表，执行连接生成一个虚拟的中间表。
  2. **`WHERE`**: 对上述中间表中的行进行过滤。
  3. **`GROUP BY`**: 将过滤后的行进行分组。
  4. **`HAVING`**: 对分组后的结果进行过滤。
  5. **`SELECT`**: 选择要显示的列，此时可以计算表达式和使用别名。
  6. **`DISTINCT`**: 对 `SELECT` 出来的结果进行去重。
  7. **`ORDER BY`**: 对最终的结果集进行排序。
  8. **`LIMIT`**: 从最终结果集中选取指定的行。

- **关键推论**：

  - `WHERE` 子句在 `SELECT` 之前执行，所以 `WHERE` 中不能使用 `SELECT` 中定义的列别名

  - `ORDER BY` 在 `SELECT` 之后执行，所以 `ORDER BY` 中**可以**使用列别名



# SQL别名(Alias)

## 什么是别名 (Alias)？

- 别名是为数据库中的**表（Table）**或**字段（Column）**提供的一个临时的、可读性更强的名称。它在查询执行期间生效，并不会改变数据库中原始的表名或字段名

## 为什么要使用别名？

1. **提高可读性**：为复杂的表名或计算字段赋予简洁、有意义的名称。
2. **简化查询**：在多表连接（JOIN）时，使用短别名可以大大缩短SQL语句的长度。
3. **区分同名字段**：当连接的多个表中包含相同名称的字段时，必须使用别名来区分。
4. **子查询要求**：在 `FROM` 子句中使用子查询（派生表）时，大多数数据库系统强制要求为其指定别名。



## 表的别名

### 1. 定义位置

- **只能**在 `FROM` 子句或 `JOIN` 子句中定义

  ```sql
  -- 方式一：使用 AS 关键字 (推荐，更清晰)
  SELECT * FROM employees AS e;
  
  -- 方式二：省略 AS 关键字 (更常见)
  SELECT * FROM employees e JOIN departments d ON e.dept_id = d.id;
  ```

  

### 2. 核心规则：一旦设定,原名失效

- **一旦在查询中为表指定了别名，那么在该查询的所有地方（`SELECT`, `WHERE`, `GROUP BY` 等），都必须使用这个别名来引用该表。原始表名在该查询的作用域内将无法使用。**

- ##### ✅ 正确示例：

  ```sql
  SELECT
      e.first_name,
      d.department_name
  FROM
      employees AS e
  JOIN
      departments AS d ON e.department_id = d.id
  WHERE
      e.salary > 50000; -- 必须使用别名 e
  ```

- ##### ❌ 错误示例：

  ```sql
  -- 错误！employees 已被别名 e 替代
  SELECT employees.first_name FROM employees AS e;
  ```

  

## 字段/列的别名

### 1. 定义位置

- **只能**在 `SELECT` 子句中定义

  ```sql
  SELECT
      first_name,
      last_name,
      salary * 12 AS annual_salary, -- 为计算表达式起别名
      CONCAT(first_name, ' ', last_name) AS "Full Name" -- 可使用引号处理含空格的别名
  FROM
      employees;
  ```

  

### 2. 核心规则：受SQL逻辑执行顺序影响

- 字段别名的可用性取决于它在哪个子句中被**使用**。你**不能**在逻辑上先于 `SELECT` 执行的子句使用字段别名

- **SQL逻辑执行顺序（简化版）：**

  1. `FROM` / `JOIN`
  2. `WHERE`
  3. `GROUP BY`
  4. `HAVING`
  5. **`SELECT`** <-- **字段别名在此处生效**
  6. `ORDER BY`

- ##### ✅ 正确使用 (`ORDER BY`)

  - `ORDER BY` 在 `SELECT` 之后执行，所以它**认识**字段别名

    ```sql
    SELECT
        salary * 12 AS annual_salary
    FROM
        employees
    ORDER BY
        annual_salary DESC; -- 正确！
    ```

    

- ##### ❌ 错误使用 (`WHERE`, `GROUP BY`)

  - `WHERE` 和 `GROUP BY` 在 `SELECT` 之前执行，所以它们**不认识**字段别名

    ```sql
    -- 错误！WHERE 执行时，annual_salary 还不存在
    SELECT
        salary * 12 AS annual_salary
    FROM
        employees
    WHERE
        annual_salary > 60000;
    
    -- 正确的写法是使用原始表达式
    WHERE
        salary * 12 > 60000;
    ```



## 快速总结

| 类别         | 定义位置        | 核心规则                                                  |
| ------------ | --------------- | --------------------------------------------------------- |
| **表别名**   | `FROM` / `JOIN` | 一旦设定，查询中**必须且只能**使用别名，原名失效。        |
| **字段别名** | `SELECT`        | 只能在逻辑执行顺序**靠后**的子句（如 `ORDER BY`）中使用。 |



# 行构造函数

## 1. 是什么？

- 这种 `WHERE (col1, col2) = (val1, val2)` 的写法，被称为 **行构造函数 (Row Constructor)**，有时也叫 **行值表达式**

  - 可以把它理解为一种 **“元组比较”**。它的核心思想是**将多个列的值作为一个不可分割的整体（一行或一个元组）来进行比较**

- **基础示例：** 下面这两种写法在功能上是**完全等价**的

  ```sql
  -- 使用行构造函数
  SELECT * FROM users WHERE (id, name) = (1, 'admin');
  
  -- 使用传统的 AND 连接
  SELECT * FROM users WHERE id = 1 AND name = 'admin';
  ```

  

## 2. 何时使用？

- 虽然两种写法等价，但在不同场景下的使用频率和推荐度是不同的。

### 场景一：简单的静态值比较 (不常用)

- 对于上面最基础的例子，绝大多数开发者会优先使用 `AND` 的写法。因为它更传统、更直白，符合大多数人的编码习惯。在这种简单场景下，行构造函数的优势并不明显。

### 场景二：多列 `IN` 查询 (非常推荐)

- 当需要根据多个字段的组合来筛选数据时，行构造函数的优势就体现出来了，代码会变得异常简洁和清晰。

- **推荐写法：**

  ```sql
  -- 查询国家和城市匹配 ('中国', '上海') 或 ('美国', '纽约') 的用户
  SELECT * FROM users
  WHERE (country, city) IN (
      ('中国', '上海'),
      ('美国', '纽约')
  );
  ```

- **替代的复杂写法：**

  ```sql
  SELECT * FROM users
  WHERE (country = '中国' AND city = '上海')
     OR (country = '美国' AND city = '纽约');
  ```

  

- 当组合条件超过两个时，第一种写法的可读性优势会急剧增加



### 场景三：与返回单行的子查询结合 (非常推荐)

- 这是行构造函数最强大、最优雅的用途。当需要与一个子查询返回的多个列进行匹配时，它是最佳选择。

- **推荐写法：**

  ```sql
  -- 查找和 "张三" 在同一个部门、同一个职位的其他所有员工
  SELECT * FROM employees
  WHERE (department_id, job_id) = (
      SELECT department_id, job_id FROM employees WHERE name = '张三'
  ) AND name != '张三';
  ```

- 这个查询逻辑非常直观，如果不用行构造函数，实现起来会复杂很多



# 索引 (Index)

## 1. 索引的原理与优势

### **什么是索引？**

- 在数据库中，**索引是一种特殊的数据结构，它存储了表中特定列的值以及指向包含这些值的行的物理地址的指针**。其核心目的就是为了**大幅提高数据查询（SELECT）的速度**。

> 可以把数据库索引想象成一本书的**目录**。如果没有目录，当你想查找某个特定的章节时，你必须从第一页开始逐页翻阅，直到找到为止。这个过程称为**全表扫描 (Full Table Scan)**，在数据量大时，效率极低。
>
> 而目录（索引）则提供了一个快速通道。它将章节标题（索引键）与对应的页码（数据行的物理地址）关联起来。你只需在目录中快速找到目标，然后直接翻到指定页即可。



### **索引的优势与劣势**

- **优势**:
  1. **极大地加快数据检索速度**：这是索引最主要的好处。
  2. **保证数据唯一性**：通过创建唯一索引，可以保证数据表中每一行数据的唯一性。
  3. **加速表连接**：对用于连接（JOIN）的列创建索引，可以显著提高连接查询的效率。
  4. **加速排序和分组**：对用于 `ORDER BY` 和 `GROUP BY` 的列创建索引，可以减少查询中排序和分组的时间。
- **劣势**:
  1. **占用存储空间**：索引本身也是一个文件，需要占用物理存储空间。
  2. **降低写操作的速度**：当对表中的数据进行增加（`INSERT`）、删除（`DELETE`）和修改（`UPDATE`）时，数据库不仅要保存数据，还需要同步更新索引文件，这会带来额外的性能开销。



## 2. 索引的底层数据结构

- MySQL 索引的实现方式有多种，但最常用、最重要的就是 **B+树 (B+ Tree)**。
- 你不需要深入理解 B+树的每一个实现细节，但需要知道它的核心特性：
  1. **有序性**：B+树中的所有节点都是按索引列的值排序的。
  2. **平衡性**：它是一种平衡树，无论数据量多大，从根节点到任意叶子节点的路径长度都基本相同，这保证了查询性能的稳定性。
  3. **查询效率高**：B+树的结构使得查找、范围查找等操作的效率非常高，时间复杂度为 O(log N)。
- 正是因为 B+树的这些特性，使得数据库可以快速定位到目标数据，避免了全表扫描。



## 3. 索引的类型与创建

### **3.1 主键索引**

- 一种特殊的唯一索引，不允许有空值。

- 每个表只能有一个主键索引。

- **创建方式**：在创建表时，通过 `PRIMARY KEY` 约束自动创建。

  ```mysql
  -- 在创建表时定义
  CREATE TABLE 表名 (
      id INT PRIMARY KEY,
      ...
  );
  
  -- 在已存在的表上添加
  ALTER TABLE 表名 ADD PRIMARY KEY (id);
  ```



### **3.2 唯一索引**

- 索引列的值必须唯一，但允许有空值 (`NULL`)。

- **创建方式**：

  ```mysql
  -- 方式一：在创建表时，通过 UNIQUE 约束定义
  CREATE TABLE 表名 (
      列名 数据类型 UNIQUE,
      ...
  );
  
  -- 方式二：在创建表时，通过 UNIQUE INDEX 定义
  CREATE TABLE 表名 (
      ...
      UNIQUE INDEX 索引名 (列名)
  );
  
  -- 方式三：在已存在的表上添加
  CREATE UNIQUE INDEX 索引名 ON 表名 (列名);
  ```



### **3.3 普通索引**

- 最基本的索引类型，没有任何限制，仅仅是为了加速查询。

- 允许索引列中包含重复值和空值。

- **创建方式**：

  ```mysql
  -- 方式一：在创建表时定义（INDEX 和 KEY 是同义词）
  CREATE TABLE 表名 (
      ...
      INDEX 索引名 (列名),
      KEY 另一个索引名 (另一列)
  );
  
  -- 方式二：在已存在的表上添加
  CREATE INDEX 索引名 ON 表名 (列名);
  ```



### **3.4 复合索引**

- **定义**：在一个索引中包含多个列。

- **最左前缀原则 (Leftmost Prefix Principle)**：这是复合索引最重要的规则。当创建一个 `(列1, 列2, 列3)` 的复合索引时，它实际上相当于创建了 `(列1)`、`(列1, 列2)`、`(列1, 列2, 列3)` 三个索引。

  - 查询条件使用 `列1`，可以用到索引。
  - 查询条件使用 `列1` 和 `列2`，可以用到索引。
  - 查询条件使用 `列1`、`列2` 和 `列3`，可以用到索引。
  - **但是**，如果查询条件**没有包含最左边的 `列1`**，例如只使用 `列2` 或 `列3`，则无法使用该复合索引。

- **创建方式**：与普通/唯一索引类似，只是在括号内指定多个列。

  ```mysql
  -- 在创建表时定义
  CREATE TABLE 表名 (
      ...
      INDEX 索引名 (列1, 列2)
  );
  
  -- 在已存在的表上添加
  CREATE INDEX 索引名 ON 表名 (列1, 列2);
  ```



### **3.5. 全文索引**用于在 `CHAR`, `VARCHAR`, `TEXT` 类型的列上进行全文搜索

- 它使用不同于 B+树的算法，可以高效地搜索文本内容中的关键词，而不是简单的“等于”或“大于”比较

- **创建方式**：

  ```
  -- 在创建表时定义
  CREATE TABLE 表名 (
      ...
      FULLTEXT INDEX 索引名 (列名)
  );
  
  -- 在已存在的表上添加
  CREATE FULLTEXT INDEX 索引名 ON 表名 (列名);
  ```



## 4. 管理索引

### **查看索引**

- **语法**：

  ```mysql
  SHOW INDEX FROM 表名;
  -- 或者
  SHOW KEYS FROM 表名;
  ```

- 这会列出表上所有索引的详细信息，包括索引名、所在的列、是否唯一等。



### **删除索引**

- **语法**：

  ```mysql
  DROP INDEX 索引名 ON 表名;
  ```

- **注意**：主键索引不能用 `DROP INDEX` 删除，必须通过 `ALTER TABLE 表名 DROP PRIMARY KEY;` 来删除。



## 5. 索引的最佳实践

### **何时应该创建索引？**

1. **主键**：必须有索引（自动创建）。
2. **频繁作为 `WHERE` 查询条件的列**：这是最常见的创建索引的理由。
3. **作为 `JOIN` 连接条件的列**：通常是外键列，为其创建索引能极大提升连接效率。
4. **频繁需要 `ORDER BY` 排序的列**：索引本身是有序的，可以利用索引的有序性来避免额外的排序操作。
5. **频繁需要 `GROUP BY` 分组的列**。



### **何时不应该创建索引？**

1. **表记录太少**：如果一个表只有几百条记录，创建索引的意义不大，全表扫描可能更快。
2. **频繁进行写操作（`INSERT`/`UPDATE`/`DELETE`）的表**：过多的索引会严重降低写操作的性能。
3. **列的“区分度”不高**：例如“性别”列，只有男、女、未知等几个值，大量重复。为这样的列创建索引，效果微乎其微。
4. **很少在查询中使用的列**。





# 事务控制

## 1.什么是事务？

- 在数据库中，**事务 (Transaction)** 是一个**不可分割的、最小的逻辑工作单元**。它由一个或多个 SQL 语句组成，这些语句在逻辑上共同完成一个完整的业务操作
  - 这个工作单元的特点是：**要么所有操作全部成功执行，要么全部失败回滚**，数据库绝对不会停留在中间某个不完整的状态

- **经典示例：银行转账**

  - 假设 A 用户要向 B 用户转账 1000 元。这个业务操作包含两个核心的 SQL 语句：
    1. `UPDATE accounts SET balance = balance - 1000 WHERE user_id = 'A';` (A账户扣款)
    2. `UPDATE accounts SET balance = balance + 1000 WHERE user_id = 'B';` (B账户收款)

  - 事务的作用就是将这两个操作“捆绑”在一起。

    - **如果两个操作都成功**：事务提交 (Commit)，转账完成。

    - **如果在操作1成功后，操作2执行前，系统突然断电或崩溃**：事务会自动回滚 (Rollback)，数据库会撤销操作1（将A的1000元退回），保证A的钱没有平白无故地消失。
  
  >数据库本身并不知道您定义的“一个完整的业务操作”应该包含多少条SQL语句的。在上面的转账例子中，对于MySQL来说，`BEGIN` 和 `COMMIT` 之间可以是一条、两条、甚至一百条`UPDATE`语句。**在它看来，只要收到了 `COMMIT`，就意味着您这位“指挥官”下达了“任务完成，存档！”的命令**。它不会去猜测：“指挥官是不是忘了给我第二条指令？”



## 2.事务的ACID特性

- ACID 是衡量事务可靠性的四个核心特性，是所有关系型数据库都必须遵循的黄金准则。

### **2.1 原子性**

- **定义**：一个事务中的所有操作，要么**全部完成**，要么**全部不完成**，不会结束在中间某个环节。事务是数据库操作的最小单位，不可再分
- **体现**：银行转账中，扣款和收款这两个操作就是一个原子操作。



### **2.2 一致性**

- **定义**：事务执行前后，数据库必须从一个**一致性状态**转移到另一个**一致性状态**。
- **体现**：在转账前，A和B账户的总金额为X。转账事务成功后，A和B账户的总金额仍然为X。事务不会凭空创造或销毁数据，保证了业务规则的正确性。



### **2.3 隔离性**

- **定义**：当多个事务并发执行时，一个事务的执行不应被其他事务干扰。即一个事务内部的操作及使用的数据对**并发的其他事务是隔离的**，并发执行的各个事务之间不能互相干扰。
- **体现**：当A向B转账时，另一个事务（如查询系统总资产）不应该看到A账户已扣款但B账户还未收款的“中间状态”。它看到的要么是转账前的状态，要么是转账后的状态。



### **2.4 持久性**

- **定义**：一旦事务被成功提交，它对数据库中数据的改变就是**永久性的**。即使接下来系统发生故障，提交的事务结果也不会丢失。
- **体现**：只要系统提示转账成功，那么这个转账记录就一定被永久地保存下来了，即使服务器立即断电重启，数据也不会丢失。



## 3 事务控制语句 (TCL)

- 在 MySQL 中，默认情况下，每一条 DML 语句（INSERT, UPDATE, DELETE）都是一个独立的事务，执行完后会自动提交。
  要手动控制一个事务的范围，将多个操作捆绑成一个原子单元，我们就需要使用事务控制语言 (TCL)

  - **`START TRANSACTION`** 或 **`BEGIN`**

    - **作用**：显式地开启一个新事务。执行此命令后，所有后续的数据库操作都将成为该事务的一部分，直到遇到 COMMIT 或 ROLLBACK

      

  - **`COMMIT`**

    - **作用**：提交事务。将事务中所有已执行但未保存的操作（如UPDATE的修改）永久地写入数据库的物理存储中，使这些变更对其他会话（或事务）可见。事务成功结束

      

  - **`ROLLBACK`**

    - **作用**：回滚事务。撤销当前事务中所有未提交的操作，将数据库恢复到执行 START TRANSACTION 命令之前的状态。事务失败结束

      

  - **`SAVEPOINT 保存点名`**

    - **作用**：在事务中创建一个“保存点”或“检查点”。这在复杂的事务中非常有用，因为它允许你回滚到事务的某个特定点，而不是撤销整个事务的所有操作

      

  - **`ROLLBACK TO SAVEPOINT 保存点名`**

    - **作用**：可以将事务回滚到指定的保存点。这会撤销该保存点之后的所有操作，但保留保存点之前的操作。注意，执行此操作后，事务并未结束，你仍可以继续执行其他操作，并最终通过 COMMIT 或 ROLLBACK 来结束整个事务

      

- **使用流程示例**：

  ```mysql
  -- 场景：银行转账
  
  -- 1. 开启事务
  START TRANSACTION;
  
  -- 2. 执行一系列不可分割的操作
  -- A账户扣款1000
  UPDATE accounts SET balance = balance - 1000 WHERE user_id = 'A';
  
  -- 在某些复杂场景下，可以设置一个保存点
  SAVEPOINT sp1; -- 创建一个保存点 sp1
  
  -- B账户加款1000
  UPDATE accounts SET balance = balance + 1000 WHERE user_id = 'B';
  
  -- 假设此时发现B账户是无效或被冻结的账户，需要撤销给B加款的操作，
  -- 但A账户的扣款操作需要暂时保留（可能转入一个待处理账户或稍后重试）。
  -- 此时，我们可以回滚到保存点 sp1，而不是整个事务。
  ROLLBACK TO SAVEPOINT sp1; 
  -- 此命令执行后，给B账户加款的操作被撤销了，但A账户扣款的操作依然存在于当前事务中。
  
  -- 3. 最终决定提交或回滚
  -- 如果后续问题解决，可以提交事务中剩余的操作（即A的扣款）
  COMMIT; 
  -- 或者，如果决定整个转账操作完全取消
  -- ROLLBACK;
  ```

  

## 4 并发问题与隔离级别

- 在多用户环境中，为了提升系统性能，数据库允许多个事务同时访问和操作同一份数据，这就是并发。然而，并发执行如果缺乏有效的控制，就可能会引发一些问题，破坏事务的隔离性。

### **并发事务带来的问题**

1. **脏读 (Dirty Read)**：一个事务（T1）读取到了另一个事务（T2）**修改过但尚未提交**的数据。如果 T2 最终执行了 ROLLBACK，那么 T1 读取到的数据就是临时的、无效的“脏”数据，基于脏数据所做的任何操作都将是错误的。
2. **不可重复读 (Non-Repeatable Read)**：在一个事务（T1）内，两次读取**同一行数据**，但得到的结果不同。这是因为在 T1 两次读取之间，有另一个事务（T2）对这行数据执行了 UPDATE 或 DELETE 操作并**已提交**。重点在于**数据值的变更或行的删除**。
3. **幻读 (Phantom Read)**：在一个事务（T1）内，两次执行**相同的范围查询**（例如 WHERE age > 25），但第二次查询返回了第一次查询没有出现的“幻影”行。这是因为在 T1 两次查询之间，有另一个事务（T2）提交了 INSERT 操作，插入了新的、符合 T1 查询条件的数据行。重点在于**新增了符合条件的行**。



### **SQL 标准的四种隔离级别**

- 为了解决上述问题，SQL 标准定义了四种隔离级别。这些级别是并发性能与数据一致性之间的权衡，级别越高，数据一致性越好，但并发性能通常越差。

  | 隔离级别                        | 脏读 | 不可重复读 | 幻读 |
  | ------------------------------- | ---- | ---------- | ---- |
  | **Read Uncommitted (读未提交)** | 可能 | 可能       | 可能 |
  | **Read Committed (读已提交)**   | 不会 | 可能       | 可能 |
  | **Repeatable Read (可重复读)**  | 不会 | 不会       | 可能 |
  | **Serializable (串行化)**       | 不会 | 不会       | 不会 |

  - **Read Uncommitted (读未提交)**：最低的隔离级别，允许一个事务读取另一个事务未提交的更改。性能最好，但数据一致性最差

  - **Read Committed (读已提交)**：一个事务只能读取到其他事务已经提交的数据。这是大多数数据库（如 Oracle, SQL Server）的默认隔离级别。它能避免脏读，但不能避免不可重复读和幻读

  - **Repeatable Read (可重复读)**：确保在同一个事务中，多次读取同一行数据的结果是一致的。它能避免脏读和不可重复读，但仍可能出现幻读

  - **Serializable (串行化)**：最高的隔离级别。它通过强制事务串行执行（一个接一个）来避免所有并发问题。性能最差，但能提供最强的数据一致性保证

  - **MySQL (InnoDB) 默认隔离级别**：**Repeatable Read (可重复读)**。值得注意的是，MySQL 的 InnoDB 存储引擎在 Repeatable Read 级别下，通过其独特的多版本并发控制机制和一种称为“间隙锁”的技术，在很大程度上解决了幻读问题。MVCC确保事务看到的是一个一致性的数据快照，而间隙锁会锁定一个查询范围，防止其他事务在这个范围内插入新行。因此，标准的SQL规范认为此级别有幻读问题，但在MySQL InnoDB中，幻读问题得到了有效的抑制。



# 用户与权限管理

- 在生产环境中，数据库通常由多个用户和应用程序共同访问。为了保证数据的安全性和隔离性，必须对每个访问者进行精细的权限控制

## 1.MySQL 用户账户

- 在 MySQL 中，一个用户账户并不仅仅是一个用户名，它是由**用户名 (username)** 和**主机名 (hostname)** 共同组成的，格式为：`'username'@'hostname'`

  - **`username`**：用户的登录名。

  - **`hostname`**：指定了该用户可以从哪个主机或 IP 地址连接到 MySQL 服务器。
    - `localhost`：只允许从服务器本机连接。
    - `192.168.1.100`：只允许从 IP 地址为 `192.168.1.100` 的客户端连接。
    - `%`：一个通配符，表示允许从**任何**主机或 IP 地址连接。

  - 例如，`'webapp'@'192.168.1.100'` 和 `'webapp'@'localhost'` 是两个完全不同的账户

## 2.用户管理

### **2.1 创建用户**

- **作用**：创建一个新的 MySQL 用户账户。

- **语法**：

  ```mysql
  CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';
  ```

- **示例**：

  ```mysql
  -- 创建一个只能从本机登录的 app_user 用户
  CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'a_strong_password';
  
  -- 创建一个可以从任何地方登录的 remote_user 用户
  CREATE USER 'remote_user'@'%' IDENTIFIED BY 'another_password';
  ```



### **2.2 删除用户**

- **作用**：永久删除一个用户账户。

- **语法**：

  ```mysql
  DROP USER '用户名'@'主机名';
  ```



### **2.3 修改用户**

- **重命名用户 (RENAME USER)**

  ```mysql
  RENAME USER '旧用户名'@'旧主机名' TO '新用户名'@'新主机名';
  ```

- **修改密码 (ALTER USER)** (推荐方式)

  ```mysql
  ALTER USER '用户名'@'主机名' IDENTIFIED BY '新密码';
  ```



## 3. 高级账户安全与资源控制

### **3.1 认证插件**

- **背景**：MySQL 使用认证插件来验证用户密码。这是一个非常关键的配置，尤其对于应用开发者。

- **常见插件**：

  - `caching_sha2_password`：MySQL 8.0 及以上版本的**默认**认证方式，更安全
  - `mysql_native_password`：MySQL 5.7 及之前版本的默认认证方式

- **常见问题**：很多旧的客户端、驱动程序（如旧版的 JDBC 驱动）可能不支持新的 `caching_sha2_password`，导致连接时报认证错误

- **解决方案**：在创建或修改用户时，明确指定其使用的认证插件

  ```mysql
  -- 创建一个使用旧版认证插件的用户，以兼容老客户端
  CREATE USER 'legacy_user'@'localhost' 
  IDENTIFIED WITH mysql_native_password BY 'password123';
  
  -- 修改已有用户的认证插件和密码
  ALTER USER 'app_user'@'localhost' 
  IDENTIFIED WITH mysql_native_password BY 'new_password';
  ```



### **3.2 密码安全策略**

- **密码过期策略**：可以强制用户定期更换密码。

  ```mysql
  -- 创建用户时设置密码90天后过期
  CREATE USER 'sec_user'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE INTERVAL 90 DAY;
  
  -- 修改已有用户的密码为永不过期
  ALTER USER 'sec_user'@'localhost' PASSWORD EXPIRE NEVER;
  ```

- **密码强度校验**：MySQL 提供了密码验证组件（`validate_password_component`），可以安装并配置，以强制要求密码必须满足一定的长度、大小写、数字和特殊字符的组合。



### **3.3 资源限制**

- **作用**：为了防止某个用户滥用数据库资源（例如发起过多查询导致服务器过载），可以对其进行资源使用限制。

- **语法**：在 `CREATE USER` 或 `ALTER USER` 语句后添加 `WITH` 子句。

  ```
  ALTER USER '用户名'@'主机名'
  WITH MAX_QUERIES_PER_HOUR 1000  -- 每小时最多执行1000次查询
       MAX_UPDATES_PER_HOUR 100   -- 每小时最多执行100次更新
       MAX_CONNECTIONS_PER_HOUR 60; -- 每小时最多建立60个连接
  ```



## 4 权限管理

新创建的用户没有任何权限，甚至无法登录。我们需要使用 `GRANT` 和 `REVOKE` 命令来管理其权限。

### **4.1 授予权限**

- **作用**：赋予用户对数据库对象的操作权限。

- **语法**：

  ```mysql
  GRANT 权限类型 [(列列表)]
  ON 数据库名.表名
  TO '用户名'@'主机名'
  [WITH GRANT OPTION];
  ```

- **常用权限类型**：`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `ALTER`, `DROP`, `ALL PRIVILEGES`。

- **`WITH GRANT OPTION`**：允许该用户将他自己拥有的权限再授予给其他用户。**请谨慎使用此选项！**



### **4.2 查看权限**

- **作用**：查看指定用户当前拥有的权限。

- **语法**：

  ```mysql
  SHOW GRANTS FOR '用户名'@'主机名';
  ```



### **4.3 撤销权限**

- **作用**：收回已经赋予用户的权限。

- **语法**：

  ```mysql
  REVOKE 权限类型
  ON 数据库名.表名
  FROM '用户名'@'主机名';
  ```

- **刷新权限**：在执行 `GRANT` 或 `REVOKE` 后，通常权限会立即生效。如果未生效，可执行 `FLUSH PRIVILEGES;` 来手动重载授权表。



## 5 角色管理(MySQL 8.0+)

- 当用户数量很多时，为每个用户单独授权会变得非常繁琐。MySQL 8.0 引入了**角色**的概念，角色是**权限的集合**
  - **逻辑**：将权限授予角色，再将角色授予用户。这样，修改角色的权限，所有被授予该角色的用户的权限都会自动更新

### **5.1 创建与删除角色**

- 语法

  ```sql
  CREATE ROLE '角色名1', '角色名2';
  DROP ROLE '角色名';
  ```

  



### **5.2 给角色授权**

- 语法

  ```sql
  GRANT 权限类型 ON 数据库名.表名 TO '角色名';
  ```

- 示例

  ```sql
  GRANT SELECT ON my_app.* TO 'app_read_only';
  GRANT SELECT, INSERT, UPDATE, DELETE ON my_app.* TO 'app_read_write';
  ```

  



### **5.3 将角色授予用户**

- 语法

  ```sql
  GRANT '角色名' TO '用户名'@'主机名';
  ```

- 示例

  ```sql
  GRANT 'app_read_only' TO 'user1'@'localhost';
  GRANT 'app_read_write' TO 'user2'@'localhost';
  ```

  

### **5.4 激活角色与设置默认角色**

- 用户登录后，需要激活角色才能获得其权限。可以设置默认角色，使其在登录时自动激活。


- **语法**：

  ```sql
  SET DEFAULT ROLE '角色名' TO '用户名'@'主机名';
  ```

  

- **示例**：

  ```sql
  SET DEFAULT ROLE 'app_read_write' TO 'user2'@'localhost';
  ```

  

## 6 最佳实践

1. **遵循最小权限原则**：只授予用户完成其工作所必需的最小权限。
2. **为不同应用创建不同用户**：避免多个应用程序共享同一个高权限账户。
3. **限制主机来源**：尽量使用具体的主机名或IP地址，而不是通配符 `%`。
4. **强制密码策略**：使用密码过期和强度校验等策略，提升账户安全性。
5. **使用角色进行批量授权**：当用户职责类似时，优先使用角色来管理权限。
6. **定期审计权限**：定期使用 `SHOW GRANTS` 检查用户权限，移除不再需要的权限。





# 一些“约定大于配置”的设定

- 在约定中，我们通常会遵循 **“一个数据库表 对应 一个 Java 实体类”** 的原则
- 在约定中，我们通常会把 **”主键字段都设定为`id`“**
