# Apache POI 

## 1. 核心概念：Excel 拆解成 Java 对象

在写代码之前，必须先建立一个观念：**Excel 文件本质上就是一种层级结构的数据**

Apache POI 的设计思想非常直观，它把 Excel 的物理结构一一映射成了 Java 类

请看下面的对应关系，这是所有操作的基础：



### 1.1 层级结构图

```
物理 Excel 文件 (.xlsx)   ---->   Java 类: Workbook (工作簿)
       │
       ├── 下面的标签页    ---->   Java 类: Sheet (工作表)
       │      │
       │      ├── 每一行   ---->   Java 类: Row (行)
       │      │     │
       │      │     └── 格子 ---->   Java 类: Cell (单元格)
```



### 1.2 关键类解释

1. **`Workbook` (工作簿)**

   - 这就是你电脑上那个 `.xlsx` 文件本身⭐
   - 在代码里，它是所有操作的入口。没有 Workbook，就没有 Sheet
   - *注意*：你需要区分 `HSSFWorkbook` (老版 .xls) 和 `XSSFWorkbook` (新版 .xlsx)。现在开发通常只用 `XSSFWorkbook`

   

2. **`Sheet` (工作表)**

   - 就是 Excel 左下角的那个 Sheet1, Sheet2 标签

   - 一个 Workbook 里可以有很多 Sheet


   

3. **`Row` (行)**

   - Sheet 里的横向行

   - **重点**：计算机是从 0 开始计数的。Excel 里的“第1行”，在 POI 里索引是 `0`


   

4. **`Cell` (单元格)**

   - Row 里的每一个格子
   - 它也是从 0 开始计数的



### 1.3 常用API

`Workbook`, `Sheet`, `Row`, `Cell` 的一些常用方法

#### 1. Workbook (工作簿) 常用方法

`Workbook` 是 Excel 文件的顶层容器

| 方法名              | 参数             | 返回值      | 说明                         | 避坑指南                                                    |
| ------------------- | ---------------- | ----------- | ---------------------------- | ----------------------------------------------------------- |
| `createSheet`       | `(String name)`  | `Sheet`     | 创建一个新的 Sheet 页        | 名字不能包含 `/ \ ? * [ ]` 等特殊字符，且不能超过 31 个字符 |
| `getSheetAt`        | `(int index)`    | `Sheet`     | 获取指定索引的 Sheet         | `0` 代表第一个 Sheet。如果索引越界会抛异常                  |
| `getSheet`          | `(String name)`  | `Sheet`     | 根据名字获取 Sheet           | 如果找不到，返回 `null`                                     |
| `getNumberOfSheets` | 无               | `int`       | 获取 Sheet 总数              | 遍历所有 Sheet 时用                                         |
| `createCellStyle`   | 无               | `CellStyle` | **创建**一个新的样式对象     | **严禁在循环内调用**，必须全局复用                          |
| `createFont`        | 无               | `Font`      | **创建**一个新的字体对象     | 同样严禁在循环内调用                                        |
| `write`             | `(OutputStream)` | `void`      | 将数据写入输出流             | 写入后 Workbook 不会自动关闭，需手动关闭                    |
| `dispose`           | 无               | `boolean`   | **SXSSF 专用**：清理临时文件 | 使用 `SXSSFWorkbook` 时必须调用，否则磁盘会被撑爆           |
| `close`             | 无               | `void`      | 关闭工作簿                   | 释放内存资源，建议写在 `try-with-resources` 中              |



**代码示例**：

```java
Workbook wb = new XSSFWorkbook();
Sheet sheet1 = wb.createSheet("销售数据"); // 创建
Sheet sheet2 = wb.getSheetAt(0);         // 获取
CellStyle style = wb.createCellStyle();  // 准备样式
```



#### 2. Sheet (工作表) 常用方法

`Sheet` 是数据的承载页面

| 方法名                  | 参数           | 返回值 | 说明                  | 避坑指南                                                     |
| ----------------------- | -------------- | ------ | --------------------- | ------------------------------------------------------------ |
| `createRow`             | `(int rowNum)` | `Row`  | **创建** 新的一行     | 如果该行已存在，旧数据会被**覆盖/清空**                      |
| `getRow`                | `(int rowNum)` | `Row`  | **获取** 已存在的一行 | 如果该行从未被创建过，返回 `null`。**SXSSF 写过的行无法获取** |
| `getLastRowNum`         | 无             | `int`  | 获取最后一行的索引    | **注意**：返回的是 `index` (从0开始)，如果有5行数据，返回的是 `4` |
| `setColumnWidth`        | `(col, width)` | `void` | 设置列宽              | 宽度单位很奇怪，是 `1/256` 个字符宽。通常写 `20 * 256` 代表 20 个字符宽 |
| `autoSizeColumn`        | `(int col)`    | `void` | 自动调整列宽          | **性能杀手**。SXSSF 中不可用。数据量大时慎用                 |
| `addMergedRegion`       | `(Region)`     | `int`  | 合并单元格            | 传入 `CellRangeAddress(起行, 止行, 起列, 止列)`              |
| `setDefaultColumnWidth` | `(int)`        | `void` | 设置全局默认列宽      | 懒人做法，给所有列设置一个默认宽度                           |



**代码示例**：

```java
Sheet sheet = wb.createSheet();
// 设置第0列宽度为 20个字符
sheet.setColumnWidth(0, 20 * 256); 

// 合并第1行到第2行，第0列到第5列
sheet.addMergedRegion(new CellRangeAddress(0, 1, 0, 5)); 

// 获取最后一行索引用于追加数据
int lastRowIndex = sheet.getLastRowNum(); 
```



#### 3. Row (行) 常用方法

`Row` 是单元格的容器

| 方法名              | 参数        | 返回值 | 说明            | 避坑指南                                             |
| ------------------- | ----------- | ------ | --------------- | ---------------------------------------------------- |
| `createCell`        | `(int col)` | `Cell` | **创建** 单元格 | 如果该位置已有单元格，会被覆盖初始化                 |
| `getCell`           | `(int col)` | `Cell` | **获取** 单元格 | 如果没数据，返回 `null`。读取时必须判空              |
| `setHeight`         | `(short)`   | `void` | 设置行高        | 单位是 `1/20` 点 (Points)。写 `30 * 20` 代表 30px 高 |
| `setHeightInPoints` | `(float)`   | `void` | 设置行高(点)    | 这个更直观，直接传 `30` 就是 30px                    |
| `getRowNum`         | 无          | `int`  | 获取当前行号    | 偶尔有用                                             |



**代码示例**：

```java
Row row = sheet.createRow(5);
row.setHeightInPoints(30); // 设置高一点，适合做表头
Cell cell = row.createCell(0);
```



#### 4. Cell (单元格) 常用方法

`Cell` 是数据的**最小单位**，操作最频繁

##### 4.1 写入类 (Setters)

| 方法名           | 参数      | 说明                                                         |
| ---------------- | --------- | ------------------------------------------------------------ |
| `setCellValue`   | `String`  | 写入文本                                                     |
| `setCellValue`   | `double`  | 写入数字。所有整数、小数都算 double                          |
| `setCellValue`   | `boolean` | 写入布尔值 (TRUE/FALSE)                                      |
| `setCellValue`   | `Date`    | **写入日期**。必须配合 `setCellStyle` 设置日期格式，否则显示为数字 |
| `setCellStyle`   | `Style`   | 应用样式（边框、颜色、日期格式等）                           |
| `setCellFormula` | `String`  | 设置公式，例如 `"SUM(A1:B1)"`                                |



##### 4.2 读取类(Getters) - 必须先判断类型

| 方法名                | 返回值     | 说明                                                         |
| --------------------- | ---------- | ------------------------------------------------------------ |
| `getCellType`         | `CellType` | **最重要**。枚举值：`STRING`, `NUMERIC`, `BOOLEAN`, `BLANK`, `FORMULA`, `ERROR` |
| `getStringCellValue`  | `String`   | 获取文本。如果格子里是数字，调用这个会**报错**               |
| `getNumericCellValue` | `double`   | 获取数字（包含日期）                                         |
| `getBooleanCellValue` | `boolean`  | 获取布尔值                                                   |
| `getDateCellValue`    | `Date`     | 获取日期。前提是 `getCellType` 是 NUMERIC 且 `DateUtil.isCellDateFormatted` 为真 |
| `toString`            | `String`   | 快速获取内容的字符串表示（调试用，不建议生产环境依赖其格式） |



#### 5. CellStyle (样式) 常用方法

样式稍微复杂一点，通常需要链式调用

| 方法名                   | 参数                  | 说明                                                         |
| ------------------------ | --------------------- | ------------------------------------------------------------ |
| `setDataFormat`          | `short`               | 设置数据格式（如日期 `yyyy-MM-dd`，货币 `¥#,##0.00`）        |
| `setAlignment`           | `HorizontalAlignment` | 水平对齐。常用：`CENTER` (居中), `LEFT` (左对齐)             |
| `setVerticalAlignment`   | `VerticalAlignment`   | 垂直对齐。常用：`CENTER` (垂直居中)                          |
| `setFillForegroundColor` | `short`               | 设置背景色。颜色常量在 `IndexedColors` 类中                  |
| `setFillPattern`         | `FillPatternType`     | **必须设置**，通常用 `SOLID_FOREGROUND` (纯色填充)。只设颜色不设Pattern，背景不会显示 |
| `setFont`                | `Font`                | 应用字体对象                                                 |
| `setWrapText`            | `boolean`             | 是否自动换行                                                 |
| `setBorderBottom`        | `BorderStyle`         | 设置底边框。常用：`THIN` (细线), `THICK` (粗线)              |



**综合样式示例**：

```java
// 1. 创建样式对象
CellStyle style = workbook.createCellStyle();

// 2. 设置居中
style.setAlignment(HorizontalAlignment.CENTER);
style.setVerticalAlignment(VerticalAlignment.CENTER);

// 3. 设置黄色背景
style.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
style.setFillPattern(FillPatternType.SOLID_FOREGROUND);

// 4. 设置边框
style.setBorderBottom(BorderStyle.THIN);
style.setBorderLeft(BorderStyle.THIN);

// 5. 设置日期格式
style.setDataFormat(workbook.getCreationHelper().createDataFormat().getFormat("yyyy-MM-dd"));

// 6. 应用给单元格
cell.setCellStyle(style);
```



#### 6. 实用工具类 (Utils)

POI 官方提供了一些静态工具类，不用实例化直接用

- **`CellRangeAddress`**: 用于定义区域（合并单元格时用）
  - `new CellRangeAddress(firstRow, lastRow, firstCol, lastCol)`
- **`DateUtil`**: 用于判断数字是不是日期
  - `DateUtil.isCellDateFormatted(cell)`: 返回 true/false
- **`IOUtils`**: 用于读取图片流放入 Excel
  - `IOUtils.toByteArray(inputStream)`



## 2. 准备工作：引入依赖

作为 Spring Boot 开发者，我们通常只需要处理 `.xlsx` (2007+版本) 格式的 Excel

在 `pom.xml` 中加入这两个核心包。不需要引入太多乱七八糟的，这两个就够最基础的使用了：

```xml
<!-- 1. POI 核心包 (处理基础架构) -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.3</version>
</dependency>
```

```xml
<!-- 2. POI OOXML 包 (专门处理 .xlsx 格式) -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.3</version>
</dependency>
```





## 3. 第一行 POI 代码：Hello World

### 3.0 第一个代码

我们不要写复杂的循环，也不搞样式。现在的任务是：**创建一个 Excel 文件，在左上角第一个格子里写上 "Hello POI"**

请仔细看注释，每一步都对应上面的“层级结构”

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileOutputStream;
import java.io.IOException;

public class HelloPoi {

    public static void main(String[] args) {
        
        // 第一步：创建一个空的工作簿对象 (Workbook)
        // 此时它只存在于内存中，还没写到硬盘上
        try (Workbook workbook = new XSSFWorkbook()) {

            // 第二步：在工作簿里创建一个工作表 (Sheet)
            // 名字叫 "我的测试表"
            Sheet sheet = workbook.createSheet("我的测试表");

            // 第三步：在 Sheet 里创建第一行 (Row)
            // 参数 0 代表 Excel 的第 1 行
            Row row = sheet.createRow(0);

            // 第四步：在 Row 里创建第一个单元格 (Cell)
            // 参数 0 代表第 1 列 (也就是 A 列)
            Cell cell = row.createCell(0);

            // 第五步：往这个格子里塞值
            cell.setCellValue("Hello POI");

            // 第六步：把内存里的 Workbook 写入硬盘
            // 这一步才是真正的"保存文件"
            try (FileOutputStream fos = new FileOutputStream("hello.xlsx")) {
                workbook.write(fos);
                System.out.println("文件生成成功！");
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```



### 3.1 运行结果

运行这段代码后，你的项目根目录下会出现一个 `hello.xlsx`。打开它，你会看到：

- 左下角有一个标签页叫“我的测试表”。
- A1 单元格（第1行第1列）里写着 "Hello POI"。



### 3.2 关键点

1. **层级不能乱**：必须先有 `Workbook`，再有 `Sheet`，再有 `Row`，最后才有 `Cell`。你不能直接创建一个 Cell 扔在空中

2. **必须手动创建**：

   - 不同于 Java 数组默认有默认值，POI 里的 Row 和 Cell 默认是不存在的

     如果你不 `createRow(5)`，直接去 `getRow(5)`，你会得到一个 `null`（空指针）



## 4. 场景模拟：导出员工列表

在实际开发中，我们通常是从数据库查出一个 `List<User>`，然后要把它变成 Excel

假设我们有这样一组模拟数据：

```java
// 这里的逻辑是：List<User> -> Excel Rows
List<String[]> dataList = new ArrayList<>();
dataList.add(new String[]{"1001", "张三", "2023-01-01", "15000.50"});
dataList.add(new String[]{"1002", "李四", "2023-02-15", "12000.00"});
```



### 批量写入的代码实现

请看下面的代码，它展示了如何利用循环结构来创建 Row 和 Cell

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileOutputStream;
import java.util.Date;

public class BatchWriteDemo {

    public static void main(String[] args) throws Exception {
        try (Workbook workbook = new XSSFWorkbook()) {
            Sheet sheet = workbook.createSheet("员工列表");

            // ==========================================
            // 第一步：创建表头 (Header)
            // ==========================================
            // 表头通常是第 0 行，且内容是固定的
            Row headerRow = sheet.createRow(0);
            String[] headers = {"工号", "姓名", "入职日期", "薪资"};
            
            for (int i = 0; i < headers.length; i++) {
                Cell cell = headerRow.createCell(i);
                cell.setCellValue(headers[i]);
            }

            // ==========================================
            // 第二步：准备样式 (关键步骤！)
            // ==========================================
            // 为什么要把样式创建放在循环外面？请看后文详解。
            
            // 1. 创建一个日期样式：将数字格式化为 "yyyy-MM-dd"
            CellStyle dateStyle = workbook.createCellStyle();
            CreationHelper createHelper = workbook.getCreationHelper();
            dateStyle.setDataFormat(createHelper.createDataFormat().getFormat("yyyy-MM-dd"));

            // 2. 创建一个金额样式：保留两位小数
            CellStyle moneyStyle = workbook.createCellStyle();
            moneyStyle.setDataFormat(createHelper.createDataFormat().getFormat("#,##0.00"));

            // ==========================================
            // 第三步：循环写入数据
            // ==========================================
            // 模拟插入 5 条数据
            for (int i = 1; i <= 5; i++) {
                // createRow(i) -> 创建第 1, 2, 3... 行 (注意不要覆盖了第0行表头)
                Row row = sheet.createRow(i);

                // 第 1 列：工号 (String)
                row.createCell(0).setCellValue("EMP_" + i);

                // 第 2 列：姓名 (String)
                row.createCell(1).setCellValue("员工" + i);

                // 第 3 列：日期 (Date) -> 这里有坑！
                Cell dateCell = row.createCell(2);
                dateCell.setCellValue(new Date()); 
                // 【必须应用样式】，否则 Excel 会显示为一串数字 (如 45123.5)
                dateCell.setCellStyle(dateStyle);

                // 第 4 列：薪资 (Double)
                Cell moneyCell = row.createCell(3);
                moneyCell.setCellValue(10000.50 + i * 100);
                moneyCell.setCellStyle(moneyStyle);
            }
            
            // (可选) 自动调整列宽，让内容不被遮挡
            // 性能提示：数据量大时禁止使用，非常耗时
            for (int i = 0; i < 4; i++) {
                sheet.autoSizeColumn(i);
            }

            // 第四步：写入文件
            try (FileOutputStream fos = new FileOutputStream("staff_list.xlsx")) {
                workbook.write(fos);
            }
        }
    }
}
```



## 5. 样式系统 (CellStyle)

在上面的代码中，如果你去掉了 `dateCell.setCellStyle(dateStyle)` 这一行，打开生成的 Excel，你会发现日期那一列变成了类似 `45231.54` 这样的数字



### 5.1 为什么日期变成了数字？

Excel 的底层并没有专门的“日期对象”存储格式。所有的日期在 Excel 内部本质上都是 **浮点数**（Double）

- 整数部分代表：距离 1900年1月1日 过了多少天
- 小数部分代表：当天的时间（比如 0.5 代表中午 12:00）

所以，**你必须显式地告诉 Excel："请把这个数字渲染成日期的样子"。** 这就是 `CellStyle` 的作用



### 5.2 黄金法则：样式对象必须复用

请注意看代码中，`CellStyle` 是在 `for` 循环**外面**创建的

**错误写法 (极度危险)**：

```javA
for (int i = 0; i < 10000; i++) {
    Row row = sheet.createRow(i);
    Cell cell = row.createCell(0);
    
    // 错误！在循环里疯狂创建样式对象
    CellStyle style = workbook.createCellStyle(); 
    style.setDataFormat(...);
    cell.setCellStyle(style);
}
```

**原因**：

1. Excel 文件规范对全文档的样式总数有限制（旧版本约 4000 个）
2. 如果在循环里 `createCellStyle()`，每行数据都会产生一个新的样式对象
3. 一旦超过限制，Excel 文件就会损坏，打不开；或者程序直接抛出异常

**正确做法**：

- **在循环外**定义好所有需要的样式（比如 `headerStyle`, `dateStyle`, `moneyStyle`）
- **在循环内**重复使用这些已经创建好的对象



## 6. 数据类型的处理

`setCellValue()` 方法有多种重载，POI 会根据你传入的 Java 类型自动处理：

1. **`String`**: 直接作为文本存储
2. **`double` / `int`**: 作为数字存储（可以进行求和计算）
3. **`boolean`**: 存储为 TRUE/FALSE
4. **`Date` / `LocalDateTime`**:
   - POI 会自动将其转换为 Excel 内部的双精度浮点数
   - **必须**配合 `CellStyle` 使用，否则用户看不懂





## 7. 为什么读取比写入更难？

写入时，数据类型是你决定的（你用 `setCellValue("abc")` 它就是文本）。 读取时，数据类型是用户决定的

**常见崩溃场景**：

1. **空指针**：用户看起来那是空的，你以为是空字符串 `""`，POI 告诉你那是 `null`
2. **类型异常**：用户在“电话号码”那一栏填了数字 `13800138000`，Excel 默认存成 `double`，你用 `getStringCellValue()` 读取，程序直接报错
3. **科学计数法**：手机号读出来变成了 `1.38E10`



## 8. 标准读取代码：健壮的写法

我们来写一个读取器，能够把 Excel 里任何类型的数据都读成 String（这是后端最常用的处理方式）

```java
import org.apache.poi.ss.usermodel.*;
import java.io.File;
import java.io.FileInputStream;
import java.text.DecimalFormat;
import java.text.SimpleDateFormat;
import java.util.Date;

public class RobustExcelReader {

    public static void main(String[] args) {
        // 假设我们读取上一节生成的 staff_list.xlsx
        File file = new File("staff_list.xlsx");

        try (FileInputStream fis = new FileInputStream(file);
             // WorkbookFactory 是个好东西，它能自动识别 .xls 和 .xlsx
             Workbook workbook = WorkbookFactory.create(fis)) {

            Sheet sheet = workbook.getSheetAt(0);

            // ==========================================
            // 遍历行 (Row)
            // ==========================================
            // 方式一：增强 for 循环 (推荐简单场景)
            // 这种方式会自动跳过那些完全没有数据的空行
            for (Row row : sheet) {
                
                // 跳过第一行表头
                if (row.getRowNum() == 0) continue;

                System.out.print("第 " + (row.getRowNum() + 1) + " 行: ");

                // ==========================================
                // 遍历列 (Cell)
                // ==========================================
                // 假设我们要读取前 4 列
                for (int i = 0; i < 4; i++) {
                    Cell cell = row.getCell(i);
                    
                    // 【核心】调用我们封装的工具方法
                    String value = getCellValueAsString(cell);
                    System.out.print("[" + value + "] ");
                }
                System.out.println();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 【万能转换器】
     * 不管 Excel 里是什么妖魔鬼怪，都变成 String 返回
     */
    private static String getCellValueAsString(Cell cell) {
        // 1. 处理 Null (最常见的坑)
        if (cell == null) {
            return "";
        }

        // 2. 根据类型分别处理
        switch (cell.getCellType()) {
            case STRING:
                return cell.getStringCellValue().trim();
            
            case BOOLEAN:
                return String.valueOf(cell.getBooleanCellValue());
            
            case NUMERIC:
                // 这里最麻烦：Excel 的日期也是 NUMERIC 类型
                if (DateUtil.isCellDateFormatted(cell)) {
                    // 如果是日期，转成标准格式字符串
                    Date date = cell.getDateCellValue();
                    return new SimpleDateFormat("yyyy-MM-dd").format(date);
                } else {
                    // 如果是普通数字 (比如工号、金额、手机号)
                    // 防止科学计数法 (1.38E10) 的最简单办法：使用 DecimalFormat
                    // #.########## 表示保留必要的小数，去掉末尾的0
                    DecimalFormat df = new DecimalFormat("#.##########");
                    return df.format(cell.getNumericCellValue());
                }
            
            case FORMULA:
                // 如果是公式，我们通常想要计算后的结果
                try {
                    return String.valueOf(cell.getNumericCellValue());
                } catch (IllegalStateException e) {
                    return cell.getStringCellValue();
                }
                
            case BLANK: // 空白单元格
            case ERROR: // 错误单元格 (#VALUE!)
            default:
                return "";
        }
    }
}
```



## 9. 代码建议

### 9.1 `WorkbookFactory.create()`

在之前的例子里，我们都是 `new XSSFWorkbook()`。但在读取时，强烈建议使用 `WorkbookFactory.create(inputStream)`

- **优点**：它会自动检测文件头，不管是老旧的 `.xls` 还是新的 `.xlsx`，它都能自动创建对应的 Workbook 对象，让你的代码通用性更强



### 9.2 空指针防御 (`cell == null`)

在 POI 中，**“没有内容的单元格”** 和 **“空白单元格”** 是两回事：

- **BLANK**: 有单元格对象，但是里面是空的
- **NULL**: 根本就没有创建过单元格对象

如果你直接调用 `row.getCell(i).getStringCellValue()`，如果那一格从未被编辑过，`getCell(i)` 会返回 `null`，然后你就崩了。所以 `if (cell == null)` 是必须要写的



### 9.3 数字 vs 日期

代码中的 `DateUtil.isCellDateFormatted(cell)` 是 POI 提供的一个静态工具方法。 Excel 内部把 `2023-11-20` 存储为数字 `45250.0`。如果你不检查这个，你读出来的入职日期就会变成一串莫名其妙的数字。



## 10. XSSF 的“内存溢出”

在前面的一些代码中，我们使用的是 `XSSFWorkbook`。它的工作原理是 **DOM (Document Object Model) 模式**

- 当你创建一个 `Row` 或 `Cell` 时，POI 会在 JVM 堆内存中创建一个对应的 Java 对象
- 如果你有 100 万行数据，每行 10 列，那就意味着有 **1000 万个 Cell 对象** 驻留在内存中
- 这会瞬间耗尽 Java 堆内存 (Heap Space)，导致程序崩溃

**结论**：`XSSFWorkbook` 只适合数据量小（建议 < 5万行）的场景



## 11. 解决方案：SXSSF (Streaming API)

Apache POI 针对大数据量提供了一个专门的扩展：**SXSSF (Streaming XSSF)**

Apache POI 使用了 **接口编程**（Interface-based programming）的思想，从 `XSSF` 切换到 `SXSSF` 几乎只需要改一行代码（构造函数）



### 工作原理：滑动窗口

SXSSF 不会将所有数据都放在内存里，它引入了一个 **“滑动窗口”** 的机制：

1. 设定一个窗口大小（比如 100 行）
2. 当你在内存中写入第 101 行时，**第 1 行** 的数据会自动被序列化（写入）到服务器磁盘的一个**临时文件**中，并从内存中移除
3. 这样，内存中永远只保留最新的 100 行对象



## 12. SXSSF 实战代码

这段代码演示了如何轻松导出 10 万行数据，而内存占用极低

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.streaming.SXSSFWorkbook;
import java.io.FileOutputStream;

public class BigDataWriter {

    public static void main(String[] args) {
        long startTime = System.currentTimeMillis();

        // ==========================================
        // 第一步：创建流式工作簿
        // ==========================================
        // 参数 100 表示：内存中只保留最近的 100 行
        // 之前的行会自动刷入磁盘临时文件
        SXSSFWorkbook workbook = new SXSSFWorkbook(100);

        // (可选) 开启临时文件压缩，以空间换时间，减少磁盘占用
        workbook.setCompressTempFiles(true);

        Sheet sheet = workbook.createSheet("百万数据测试");

        // ==========================================
        // 第二步：海量数据写入
        // ==========================================
        // 模拟写入 10 万行
        for (int i = 0; i < 100000; i++) {
            Row row = sheet.createRow(i);
            
            // 写入 10 列数据
            for (int j = 0; j < 10; j++) {
                Cell cell = row.createCell(j);
                cell.setCellValue("数据_" + i + "_" + j);
            }
            
            // 注意：在 SXSSF 中，你不能回头去访问已经刷入磁盘的行
            // 比如现在循环到了 i=200，你尝试去 getRow(0) 会返回 null
        }

        // ==========================================
        // 第三步：写入最终文件
        // ==========================================
        try (FileOutputStream fos = new FileOutputStream("big_data.xlsx")) {
            workbook.write(fos);
            System.out.println("导出完成！耗时：" + (System.currentTimeMillis() - startTime) + "ms");
        } catch (Exception e) {
            e.printStackTrace();
        }

        // ==========================================
        // 第四步：【极其重要】清理临时文件
        // ==========================================
        // SXSSF 会在系统临时目录下生成巨大的 XML 文件
        // 如果不手动 dispose，服务器磁盘很快就会被撑爆
        workbook.dispose(); 
    }
}
```



## 13. SXSSF 的限制与副作用

### 相关概念

虽然 SXSSF 解决了内存问题，但它也带来了限制。这是典型的 **“空间换功能”** 的妥协

| 功能         | XSSF (普通版)    | SXSSF (流式版)      | 原因                                                         |
| ------------ | ---------------- | ------------------- | ------------------------------------------------------------ |
| **随机访问** | 支持             | **不支持**          | 旧数据已经在磁盘里了，无法 `getRow()` 回来修改               |
| **读取文件** | 支持             | **不支持**          | SXSSF 只能用于“写”，不能用于“读”                             |
| **覆盖写入** | 支持             | **不支持**          | 比如想在最后一行计算完总和后，填回第一行，做不到             |
| **自动列宽** | `autoSizeColumn` | **不可用** (不准确) | 自动列宽需要扫描该列所有数据来计算最大宽度，但数据不在内存里，所以无法计算 |



### 常见坑点

如果你在 SXSSF 模式下强行调用 `sheet.autoSizeColumn(0)`，POI 会尝试跟踪所有列的宽度，这会破坏流式写入的初衷，重新导致内存飙升甚至报错。

**结论**：在大数据导出场景下，**请手动设置固定列宽** (`sheet.setColumnWidth`)，放弃自动列宽。



## 14. Web 导出的本质区别

在本地运行 Java 代码时，我们是把数据写入硬盘： `Workbook` -> `FileOutputStream` -> `D:/data.xlsx`

在 Web 环境下，我们是把数据写入网络的响应流： `Workbook` -> `response.getOutputStream()` -> `浏览器自动下载`

**核心原则**：Controller 层负责设置 HTTP 响应头（告诉浏览器这是个文件），Service 层负责把 Excel 写入流。



## 15. 标准 Spring Boot 架构写法

我们严格按照 Controller 和 Service 分离的规范来写

### 15.1 Service 层：只管写流，不管 HTTP

Service 层不需要知道什么是 HTTP，它只需要一个 `OutputStream` 把它填满就行了

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.streaming.SXSSFWorkbook;
import org.springframework.stereotype.Service;
import java.io.IOException;
import java.io.OutputStream;

@Service
public class ExcelExportService {

    /**
     * 生成报表并写入输出流
     * @param outputStream 来自 Controller 的响应流
     */
    public void exportUserReport(OutputStream outputStream) {
        // 1. 使用 SXSSF (流式处理)，防止 Web 高并发下内存溢出
        // 这里的 try-with-resources 只负责关闭 workbook，不负责关闭 outputStream
        // outputStream 由 Servlet 容器(Tomcat)管理，通常不需要我们手动关
        try (SXSSFWorkbook workbook = new SXSSFWorkbook(100)) {
            
            Sheet sheet = workbook.createSheet("用户数据");
            
            // 创建表头
            Row header = sheet.createRow(0);
            header.createCell(0).setCellValue("用户ID");
            header.createCell(1).setCellValue("用户名");
            
            // 模拟写入数据
            for (int i = 1; i <= 1000; i++) {
                Row row = sheet.createRow(i);
                row.createCell(0).setCellValue(i);
                row.createCell(1).setCellValue("User_" + i);
            }
            
            // 2. 核心步骤：将 Workbook 写入传入的流中
            workbook.write(outputStream);
            
            // 3. 清理临时文件
            workbook.dispose();
            
        } catch (IOException e) {
            throw new RuntimeException("Excel 生成失败", e);
        }
    }
}
```



### 15.2 Controller 层：处理协议与文件名

Controller 的核心任务是设置两个 HTTP Header：

1. `Content-Type`: 告诉浏览器返回的是 Excel，不是 HTML
2. `Content-Disposition`: 告诉浏览器弹窗下载，并指定文件名

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;

@RestController
@RequestMapping("/api/excel")
public class ExcelController {

    @Autowired
    private ExcelExportService excelExportService;

    @GetMapping("/download")
    public void download(HttpServletResponse response) {
        try {
            // ==========================================
            // 第一步：设置响应头 (Headers)
            // ==========================================
            
            // 1. 设置 MIME 类型为 Excel (.xlsx)
            response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
            
            // 2. 设置编码，防止中文乱码
            response.setCharacterEncoding("utf-8");
            
            // 3. 设置下载文件名 (这是最容易出乱码的地方)
            // 使用 URLEncoder 将中文转为 URL 编码 (比如 "用户" -> "%E7%94%A8%E6%88%B7")
            String fileName = URLEncoder.encode("月度用户报表.xlsx", StandardCharsets.UTF_8.name())
                                        .replaceAll("\\+", "%20"); // 修复空格变加号的小 Bug
            
            // attachment; filename=... 表示以附件形式下载
            response.setHeader("Content-Disposition", "attachment; filename*=UTF-8''" + fileName);

            // ==========================================
            // 第二步：调用 Service 写入数据
            // ==========================================
            // 直接获取 Servlet 的输出流传给 Service
            excelExportService.exportUserReport(response.getOutputStream());
            
            // 刷新缓冲区，确保数据发完
            response.flushBuffer();
            
        } catch (IOException e) {
            // 注意：如果流已经开始传输，这里再抛异常，前端可能收到一半损坏的文件
            // 通常这里记录日志即可
            e.printStackTrace();
        }
    }
}
```



## 16. 常见坑点解析

### 16.1 为什么 Controller 方法返回 `void`？

- 通常 Controller 方法会返回 `Result<T>` 或 `String`

  但文件下载比较特殊，因为我们直接接管了 `response` 的输出流 (`getOutputStream`)，手动写入了二进制数据

  在一次 HTTP 请求中，`HttpServletResponse` 对象内部只维护着 **唯一的一个输出流** (`OutputStream`)

  所有要发给浏览器的数据，都必须按顺序排队进入这个流

  如果你在方法签名里写了 `Result<String>` 并在最后 `return Result.success();`

  Spring MVC 框架会获取**同一个输出流**，将这串 JSON 字节**追加**到刚才的 Excel 数据后面，然后可能就会损坏数据或者文件



### 16.2 文件名乱码问题

这是后端开发最头疼的问题之一

- **错误做法**：直接 `filename="中文.xlsx"` -> 浏览器下载下来文件名可能是 `____.xlsx` 或者乱码
- **正确做法**：使用 `URLEncoder.encode` 并在 Header 中使用 `filename*=UTF-8''` 这种标准语法



### 16.3 异常处理

在 Web 下载中，一旦 `response.getOutputStream()` 开始写入数据，HTTP 响应的状态码（200 OK）就已经发出去了

如果在写入过程中（比如写到第 50 行）突然报错：

- **后果**：浏览器会收到一个“下载完成”的文件，但文件只有一半，打开会报错
- **处理**：这种情况后端很难优雅处理（因为头已经发出去了），通常只能在服务器端记录 Error 日志排查



## 17. 核心速查手册

### 1. 选型决策树

在写代码前，先问自己两个问题：**读还是写？数据量多大？**

| 场景     | 数据量               | 推荐组件  | 关键类名                         | 备注                                                        |
| -------- | -------------------- | --------- | -------------------------------- | ----------------------------------------------------------- |
| **读取** | 任意大小             | **XSSF**  | `WorkbookFactory.create(stream)` | 自动兼容 .xls/.xlsx，内存占用较高，超大文件建议用 EasyExcel |
| **写入** | 小数据 (< 5万行)     | **XSSF**  | `new XSSFWorkbook()`             | 支持随机读写，功能最全                                      |
| **写入** | **大数据 (> 5万行)** | **SXSSF** | `new SXSSFWorkbook(100)`         | **后端导出首选**，防止 OOM，只能写不能读                    |
| **维护** | 老旧系统 (.xls)      | HSSF      | `new HSSFWorkbook()`             | 除非客户死活不升级 Office，否则别用                         |



### 2. 开发“黄金法则”

#### ⚠️ 内存安全

- **规则**：大文件导出 **必须** 使用 `SXSSFWorkbook`
- **操作**：构造时传入内存行数 `new SXSSFWorkbook(100)`
- **收尾**：`workbook.write(out)` 后 **必须** 调用 `workbook.dispose()` 清理临时文件



#### ⚠️ 样式复用 (性能杀手)

- **规则**：`createCellStyle()` **严禁** 写在 `for` 循环内部

- **后果**：写在循环里会导致生成的 Excel 文件体积爆炸、损坏无法打开

- **正确姿势**：

  ```java
  // 1. 循环外创建样式
  CellStyle dateStyle = workbook.createCellStyle();
  dateStyle.setDataFormat(...);
  
  // 2. 循环内复用对象
  for (User u : users) {
      cell.setCellStyle(dateStyle);
  }
  ```



#### ⚠️ 日期处理

- **规则**：Excel 里的日期本质是**数字** (Double)
- **写入**：必须配合 `CellStyle` 设置格式 (`yyyy-MM-dd`)，否则显示为数字 (如 45213.5)
- **读取**：必须判断 `DateUtil.isCellDateFormatted(cell)`，否则读出来是数字



#### ⚠️ Web 下载

- **文件名**：必须使用 `URLEncoder.encode(name, "UTF-8")`，防止中文乱码
- **流处理**：直接对接 `response.getOutputStream()`，**不要**在服务器生成临时文件再读取



### 3. 常用代码模板 (Copy-Paste)

#### 3.1 Web 导出 Controller 模板

```java
// 设置头信息，告诉浏览器下载文件
response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
response.setCharacterEncoding("utf-8");
String fileName = URLEncoder.encode("报表.xlsx", "UTF-8").replaceAll("\\+", "%20");
response.setHeader("Content-Disposition", "attachment; filename*=UTF-8''" + fileName);

// 写入流
service.export(response.getOutputStream());
```



#### 3.2 万能读取器模板 (防空指针)

```java
private String getCellValue(Cell cell) {
    if (cell == null) return ""; // 防空指针
    switch (cell.getCellType()) {
        case STRING: return cell.getStringCellValue().trim();
        case NUMERIC: 
            if (DateUtil.isCellDateFormatted(cell)) {
                return new SimpleDateFormat("yyyy-MM-dd").format(cell.getDateCellValue());
            }
            // 防止手机号变成科学计数法
            return new DecimalFormat("#.########").format(cell.getNumericCellValue());
        case BOOLEAN: return String.valueOf(cell.getBooleanCellValue());
        default: return "";
    }
}
```



### 4. 常见异常速查

| 异常现象                                 | 可能原因                                             | 解决方案                                       |
| ---------------------------------------- | ---------------------------------------------------- | ---------------------------------------------- |
| `OutOfMemoryError: Java heap space`      | 用 `XSSF` 写了几十万行数据                           | 换成 `SXSSFWorkbook`                           |
| **文件打开报错** "We found a problem..." | 1. 循环创建样式2. SXSSF 写入后又尝试 `getRow` 旧数据 | 1. 提样式到循环外2. 检查逻辑，SXSSF 只能顺序写 |
| **文件名乱码** / `___.xlsx`              | HTTP Header 没设置编码                               | 使用 `URLEncoder` + `filename*=UTF-8''` 语法   |
| **服务器磁盘爆满**                       | `SXSSF` 用完没调 `dispose()`                         | `finally` 块中调用 `workbook.dispose()`        |
| **日期显示为 44562.5**                   | 没设置 `setDataFormat`                               | 给 Cell 应用日期格式的 CellStyle               |