# Apache POI 完全学习指南

## 1. Apache POI 概述

### 1.1 什么是Apache POI

Apache POI是Apache软件基金会的开源项目，提供Java API用于读写Microsoft Office格式的文档。POI是"Poor Obfuscation Implementation"的缩写，意为"简陋的模糊实现"。

### 1.2 POI支持的文件格式

| 组件  | 应用类型   | 支持格式                 |
| ----- | ---------- | ------------------------ |
| HSSF  | Excel      | XLS (Excel 97-2003)      |
| XSSF  | Excel      | XLSX (Excel 2007+)       |
| SXSSF | Excel      | XLSX (大数据量流式写入)  |
| HWPF  | Word       | DOC (Word 97-2003)       |
| XWPF  | Word       | DOCX (Word 2007+)        |
| HSLF  | PowerPoint | PPT (PowerPoint 97-2003) |
| XSLF  | PowerPoint | PPTX (PowerPoint 2007+)  |
| HDGF  | Visio      | VSD (Visio 97-2003)      |
| XDGF  | Visio      | VSDX (Visio 2007+)       |

### 1.3 核心组件说明

- **POIFS**: POI File System - 读写OLE2格式文档的基础
- **HSSF**: Horrible Spreadsheet Format - 读写Excel 97-2003格式
- **XSSF**: XML Spreadsheet Format - 读写Excel 2007+格式
- **SXSSF**: Streaming XML Spreadsheet Format - 大数据量Excel的流式处理
- **Common SS**: HSSF和XSSF的通用接口

## 2. 环境配置

### 2.1 Maven依赖

```xml
<!-- Apache POI 核心依赖 -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>5.2.5</version>
</dependency>

<!-- Excel OOXML 格式支持 (.xlsx) -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>

<!-- Excel OOXML schemas -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml-schemas</artifactId>
    <version>4.1.2</version>
</dependency>

<!-- Word 文档处理 -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-scratchpad</artifactId>
    <version>5.2.5</version>
</dependency>

<!-- 日期处理工具（可选） -->
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.12.5</version>
</dependency>
```

### 2.2 Gradle依赖

```gradle
implementation 'org.apache.poi:poi:5.2.5'
implementation 'org.apache.poi:poi-ooxml:5.2.5'
implementation 'org.apache.poi:poi-ooxml-schemas:4.1.2'
implementation 'org.apache.poi:poi-scratchpad:5.2.5'
```

## 3. Excel操作详解

### 3.1 Excel基本概念

```java
// Excel文档结构层次
Workbook (工作簿) 
  └── Sheet (工作表)
        └── Row (行)
              └── Cell (单元格)
```

### 3.2 创建Excel文件

#### 3.2.1 创建简单的Excel

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileOutputStream;
import java.io.IOException;

public class ExcelWriter {
    
    public void createSimpleExcel() throws IOException {
        // 创建工作簿
        Workbook workbook = new XSSFWorkbook();
        
        // 创建工作表
        Sheet sheet = workbook.createSheet("员工信息");
        
        // 创建表头行
        Row headerRow = sheet.createRow(0);
        String[] headers = {"ID", "姓名", "年龄", "部门", "薪资"};
        
        for (int i = 0; i < headers.length; i++) {
            Cell cell = headerRow.createCell(i);
            cell.setCellValue(headers[i]);
        }
        
        // 创建数据行
        Object[][] data = {
            {1, "张三", 28, "技术部", 15000},
            {2, "李四", 32, "市场部", 12000},
            {3, "王五", 25, "人事部", 10000}
        };
        
        for (int i = 0; i < data.length; i++) {
            Row row = sheet.createRow(i + 1);
            Object[] rowData = data[i];
            
            for (int j = 0; j < rowData.length; j++) {
                Cell cell = row.createCell(j);
                
                if (rowData[j] instanceof Integer) {
                    cell.setCellValue((Integer) rowData[j]);
                } else if (rowData[j] instanceof String) {
                    cell.setCellValue((String) rowData[j]);
                }
            }
        }
        
        // 自动调整列宽
        for (int i = 0; i < headers.length; i++) {
            sheet.autoSizeColumn(i);
        }
        
        // 写入文件
        try (FileOutputStream fos = new FileOutputStream("employee.xlsx")) {
            workbook.write(fos);
        }
        
        workbook.close();
    }
}
```

#### 3.2.2 设置单元格样式

```java
public void createStyledExcel() throws IOException {
    Workbook workbook = new XSSFWorkbook();
    Sheet sheet = workbook.createSheet("样式示例");
    
    // 创建字体
    Font headerFont = workbook.createFont();
    headerFont.setBold(true);
    headerFont.setFontHeightInPoints((short) 14);
    headerFont.setColor(IndexedColors.BLUE.getIndex());
    
    // 创建单元格样式
    CellStyle headerStyle = workbook.createCellStyle();
    headerStyle.setFont(headerFont);
    headerStyle.setFillForegroundColor(IndexedColors.LIGHT_YELLOW.getIndex());
    headerStyle.setFillPattern(FillPatternType.SOLID_FOREGROUND);
    headerStyle.setBorderBottom(BorderStyle.THIN);
    headerStyle.setBorderTop(BorderStyle.THIN);
    headerStyle.setBorderLeft(BorderStyle.THIN);
    headerStyle.setBorderRight(BorderStyle.THIN);
    headerStyle.setAlignment(HorizontalAlignment.CENTER);
    headerStyle.setVerticalAlignment(VerticalAlignment.CENTER);
    
    // 创建日期格式样式
    CellStyle dateStyle = workbook.createCellStyle();
    CreationHelper createHelper = workbook.getCreationHelper();
    dateStyle.setDataFormat(createHelper.createDataFormat().getFormat("yyyy-MM-dd"));
    
    // 创建货币格式样式
    CellStyle currencyStyle = workbook.createCellStyle();
    currencyStyle.setDataFormat(createHelper.createDataFormat().getFormat("¥#,##0.00"));
    
    // 创建表头
    Row headerRow = sheet.createRow(0);
    String[] headers = {"姓名", "入职日期", "基本工资", "奖金", "总收入"};
    
    for (int i = 0; i < headers.length; i++) {
        Cell cell = headerRow.createCell(i);
        cell.setCellValue(headers[i]);
        cell.setCellStyle(headerStyle);
    }
    
    // 创建数据行
    Row dataRow = sheet.createRow(1);
    
    Cell nameCell = dataRow.createCell(0);
    nameCell.setCellValue("张三");
    
    Cell dateCell = dataRow.createCell(1);
    dateCell.setCellValue(new Date());
    dateCell.setCellStyle(dateStyle);
    
    Cell salaryCell = dataRow.createCell(2);
    salaryCell.setCellValue(15000);
    salaryCell.setCellStyle(currencyStyle);
    
    Cell bonusCell = dataRow.createCell(3);
    bonusCell.setCellValue(3000);
    bonusCell.setCellStyle(currencyStyle);
    
    // 添加公式
    Cell totalCell = dataRow.createCell(4);
    totalCell.setCellFormula("C2+D2");
    totalCell.setCellStyle(currencyStyle);
    
    // 设置列宽
    for (int i = 0; i < headers.length; i++) {
        sheet.setColumnWidth(i, 15 * 256);
    }
    
    // 设置行高
    headerRow.setHeightInPoints(30);
    
    try (FileOutputStream fos = new FileOutputStream("styled_excel.xlsx")) {
        workbook.write(fos);
    }
    
    workbook.close();
}
```

### 3.3 读取Excel文件

#### 3.3.1 基本读取操作

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import java.io.FileInputStream;
import java.io.IOException;

public class ExcelReader {
    
    public void readExcel(String filePath) throws IOException {
        try (FileInputStream fis = new FileInputStream(filePath);
             Workbook workbook = new XSSFWorkbook(fis)) {
            
            // 获取第一个工作表
            Sheet sheet = workbook.getSheetAt(0);
            
            // 遍历所有行
            for (Row row : sheet) {
                // 跳过空行
                if (row == null) continue;
                
                // 遍历所有单元格
                for (Cell cell : row) {
                    String value = getCellValue(cell);
                    System.out.print(value + "\t");
                }
                System.out.println();
            }
        }
    }
    
    private String getCellValue(Cell cell) {
        if (cell == null) return "";
        
        switch (cell.getCellType()) {
            case STRING:
                return cell.getStringCellValue();
            case NUMERIC:
                if (DateUtil.isCellDateFormatted(cell)) {
                    return cell.getDateCellValue().toString();
                } else {
                    return String.valueOf(cell.getNumericCellValue());
                }
            case BOOLEAN:
                return String.valueOf(cell.getBooleanCellValue());
            case FORMULA:
                return cell.getCellFormula();
            case BLANK:
                return "";
            default:
                return "";
        }
    }
}
```

#### 3.3.2 高级读取 - 使用迭代器

```java
public void readExcelWithIterator(String filePath) throws IOException {
    try (FileInputStream fis = new FileInputStream(filePath);
         Workbook workbook = new XSSFWorkbook(fis)) {
        
        Sheet sheet = workbook.getSheetAt(0);
        
        // 使用行迭代器
        Iterator<Row> rowIterator = sheet.iterator();
        while (rowIterator.hasNext()) {
            Row row = rowIterator.next();
            
            // 使用单元格迭代器
            Iterator<Cell> cellIterator = row.cellIterator();
            while (cellIterator.hasNext()) {
                Cell cell = cellIterator.next();
                System.out.print(getCellValue(cell) + "\t");
            }
            System.out.println();
        }
    }
}
```

### 3.4 大数据量处理 - SXSSF

```java
import org.apache.poi.xssf.streaming.SXSSFWorkbook;
import org.apache.poi.xssf.streaming.SXSSFSheet;

public class LargeExcelWriter {
    
    public void writeLargeExcel() throws IOException {
        // 内存中保留100行，超过部分写入磁盘
        SXSSFWorkbook workbook = new SXSSFWorkbook(100);
        workbook.setCompressTempFiles(true); // 压缩临时文件
        
        SXSSFSheet sheet = workbook.createSheet("大数据");
        
        // 写入100万行数据
        for (int rowNum = 0; rowNum < 1000000; rowNum++) {
            Row row = sheet.createRow(rowNum);
            
            for (int cellNum = 0; cellNum < 10; cellNum++) {
                Cell cell = row.createCell(cellNum);
                cell.setCellValue("行" + rowNum + "列" + cellNum);
            }
            
            // 每1000行刷新一次
            if (rowNum % 1000 == 0) {
                sheet.flushRows(100); // 保留最后100行在内存中
            }
        }
        
        try (FileOutputStream fos = new FileOutputStream("large_data.xlsx")) {
            workbook.write(fos);
        }
        
        // 清理临时文件
        workbook.dispose();
        workbook.close();
    }
}
```

### 3.5 Excel高级操作

#### 3.5.1 合并单元格

```java
import org.apache.poi.ss.util.CellRangeAddress;

public void mergeCells() throws IOException {
    Workbook workbook = new XSSFWorkbook();
    Sheet sheet = workbook.createSheet("合并示例");
    
    // 创建行
    Row row = sheet.createRow(0);
    Cell cell = row.createCell(0);
    cell.setCellValue("合并的单元格");
    
    // 合并单元格 (起始行, 结束行, 起始列, 结束列)
    CellRangeAddress region = new CellRangeAddress(0, 2, 0, 3);
    sheet.addMergedRegion(region);
    
    // 设置合并单元格的样式
    CellStyle style = workbook.createCellStyle();
    style.setAlignment(HorizontalAlignment.CENTER);
    style.setVerticalAlignment(VerticalAlignment.CENTER);
    cell.setCellStyle(style);
    
    try (FileOutputStream fos = new FileOutputStream("merged_cells.xlsx")) {
        workbook.write(fos);
    }
    
    workbook.close();
}
```

#### 3.5.2 添加图片

```java
import org.apache.poi.util.IOUtils;
import org.apache.poi.ss.usermodel.ClientAnchor;
import org.apache.poi.ss.usermodel.Drawing;
import org.apache.poi.ss.usermodel.Picture;

public void addImage() throws IOException {
    Workbook workbook = new XSSFWorkbook();
    Sheet sheet = workbook.createSheet("图片示例");
    
    // 读取图片
    FileInputStream imageStream = new FileInputStream("logo.png");
    byte[] imageBytes = IOUtils.toByteArray(imageStream);
    imageStream.close();
    
    // 添加图片到工作簿
    int pictureIdx = workbook.addPicture(imageBytes, Workbook.PICTURE_TYPE_PNG);
    
    // 创建绘图对象
    Drawing<?> drawing = sheet.createDrawingPatriarch();
    
    // 创建锚点（定位图片位置）
    CreationHelper helper = workbook.getCreationHelper();
    ClientAnchor anchor = helper.createClientAnchor();
    
    // 设置图片位置（左上角在第2行第2列）
    anchor.setCol1(1); // 列索引
    anchor.setRow1(1); // 行索引
    anchor.setCol2(4); // 结束列
    anchor.setRow2(10); // 结束行
    
    // 创建图片
    Picture picture = drawing.createPicture(anchor, pictureIdx);
    
    // 可以调整图片大小
    // picture.resize(); // 原始大小
    // picture.resize(0.5); // 缩放50%
    
    try (FileOutputStream fos = new FileOutputStream("excel_with_image.xlsx")) {
        workbook.write(fos);
    }
    
    workbook.close();
}
```

#### 3.5.3 添加图表

```java
import org.apache.poi.xddf.usermodel.chart.*;
import org.apache.poi.xssf.usermodel.*;

public void createChart() throws IOException {
    XSSFWorkbook workbook = new XSSFWorkbook();
    XSSFSheet sheet = workbook.createSheet("图表示例");
    
    // 创建数据
    Row row0 = sheet.createRow(0);
    row0.createCell(0).setCellValue("月份");
    row0.createCell(1).setCellValue("销售额");
    row0.createCell(2).setCellValue("利润");
    
    String[] months = {"1月", "2月", "3月", "4月", "5月", "6月"};
    double[] sales = {10000, 15000, 12000, 18000, 20000, 25000};
    double[] profits = {2000, 3500, 2800, 4200, 5000, 6500};
    
    for (int i = 0; i < months.length; i++) {
        Row row = sheet.createRow(i + 1);
        row.createCell(0).setCellValue(months[i]);
        row.createCell(1).setCellValue(sales[i]);
        row.createCell(2).setCellValue(profits[i]);
    }
    
    // 创建绘图区域
    XSSFDrawing drawing = sheet.createDrawingPatriarch();
    XSSFClientAnchor anchor = drawing.createAnchor(0, 0, 0, 0, 4, 1, 12, 15);
    
    // 创建图表
    XSSFChart chart = drawing.createChart(anchor);
    chart.setTitleText("月度销售报表");
    chart.setTitleOverlay(false);
    
    // 创建图例
    XDDFChartLegend legend = chart.getOrAddLegend();
    legend.setPosition(LegendPosition.TOP_RIGHT);
    
    // 创建类别轴和值轴
    XDDFCategoryAxis bottomAxis = chart.createCategoryAxis(AxisPosition.BOTTOM);
    bottomAxis.setTitle("月份");
    XDDFValueAxis leftAxis = chart.createValueAxis(AxisPosition.LEFT);
    leftAxis.setTitle("金额（元）");
    leftAxis.setCrosses(AxisCrosses.AUTO_ZERO);
    
    // 创建数据源
    XDDFDataSource<String> months_ds = XDDFDataSourcesFactory.fromStringCellRange(sheet,
            new CellRangeAddress(1, 6, 0, 0));
    XDDFNumericalDataSource<Double> sales_ds = XDDFDataSourcesFactory.fromNumericCellRange(sheet,
            new CellRangeAddress(1, 6, 1, 1));
    XDDFNumericalDataSource<Double> profit_ds = XDDFDataSourcesFactory.fromNumericCellRange(sheet,
            new CellRangeAddress(1, 6, 2, 2));
    
    // 创建线图
    XDDFLineChartData lineChart = (XDDFLineChartData) chart.createData(ChartTypes.LINE, bottomAxis, leftAxis);
    XDDFLineChartData.Series series1 = (XDDFLineChartData.Series) lineChart.addSeries(months_ds, sales_ds);
    series1.setTitle("销售额", null);
    series1.setSmooth(false);
    series1.setMarkerStyle(MarkerStyle.STAR);
    
    XDDFLineChartData.Series series2 = (XDDFLineChartData.Series) lineChart.addSeries(months_ds, profit_ds);
    series2.setTitle("利润", null);
    series2.setSmooth(false);
    series2.setMarkerStyle(MarkerStyle.CIRCLE);
    
    chart.plot(lineChart);
    
    try (FileOutputStream fos = new FileOutputStream("chart_example.xlsx")) {
        workbook.write(fos);
    }
    
    workbook.close();
}
```

## 4. Word文档操作

### 4.1 创建Word文档

```java
import org.apache.poi.xwpf.usermodel.*;
import java.io.FileOutputStream;
import java.io.IOException;

public class WordWriter {
    
    public void createSimpleDocument() throws IOException {
        XWPFDocument document = new XWPFDocument();
        
        // 创建段落
        XWPFParagraph titleParagraph = document.createParagraph();
        titleParagraph.setAlignment(ParagraphAlignment.CENTER);
        
        XWPFRun titleRun = titleParagraph.createRun();
        titleRun.setText("项目报告");
        titleRun.setBold(true);
        titleRun.setFontSize(20);
        titleRun.setFontFamily("宋体");
        
        // 创建正文段落
        XWPFParagraph paragraph = document.createParagraph();
        XWPFRun run = paragraph.createRun();
        run.setText("这是项目报告的第一段内容。");
        run.setFontSize(12);
        
        // 添加换行
        run.addBreak();
        run.setText("这是第二行内容。");
        
        // 创建带格式的段落
        XWPFParagraph formattedPara = document.createParagraph();
        
        XWPFRun boldRun = formattedPara.createRun();
        boldRun.setText("粗体文本 ");
        boldRun.setBold(true);
        
        XWPFRun italicRun = formattedPara.createRun();
        italicRun.setText("斜体文本 ");
        italicRun.setItalic(true);
        
        XWPFRun underlineRun = formattedPara.createRun();
        underlineRun.setText("下划线文本");
        underlineRun.setUnderline(UnderlinePatterns.SINGLE);
        
        try (FileOutputStream fos = new FileOutputStream("document.docx")) {
            document.write(fos);
        }
        
        document.close();
    }
}
```

### 4.2 创建表格

```java
public void createTable() throws IOException {
    XWPFDocument document = new XWPFDocument();
    
    // 创建表格（3行4列）
    XWPFTable table = document.createTable(3, 4);
    
    // 设置表格宽度
    table.setWidth("100%");
    
    // 获取第一行（表头）
    XWPFTableRow headerRow = table.getRow(0);
    headerRow.getCell(0).setText("姓名");
    headerRow.getCell(1).setText("年龄");
    headerRow.getCell(2).setText("部门");
    headerRow.getCell(3).setText("职位");
    
    // 设置表头样式
    for (XWPFTableCell cell : headerRow.getTableCells()) {
        cell.setColor("D3D3D3");
        XWPFParagraph para = cell.getParagraphs().get(0);
        para.setAlignment(ParagraphAlignment.CENTER);
        XWPFRun run = para.getRuns().get(0);
        run.setBold(true);
    }
    
    // 填充数据
    String[][] data = {
        {"张三", "28", "技术部", "工程师"},
        {"李四", "32", "市场部", "经理"}
    };
    
    for (int i = 0; i < data.length; i++) {
        XWPFTableRow row = table.getRow(i + 1);
        for (int j = 0; j < data[i].length; j++) {
            row.getCell(j).setText(data[i][j]);
        }
    }
    
    // 添加新行
    XWPFTableRow newRow = table.createRow();
    newRow.getCell(0).setText("王五");
    newRow.getCell(1).setText("25");
    newRow.getCell(2).setText("人事部");
    newRow.getCell(3).setText("专员");
    
    try (FileOutputStream fos = new FileOutputStream("table_document.docx")) {
        document.write(fos);
    }
    
    document.close();
}
```

### 4.3 添加图片到Word

```java
public void addImageToWord() throws IOException, InvalidFormatException {
    XWPFDocument document = new XWPFDocument();
    
    XWPFParagraph paragraph = document.createParagraph();
    XWPFRun run = paragraph.createRun();
    run.setText("下面是一张图片：");
    run.addBreak();
    
    // 添加图片
    try (FileInputStream is = new FileInputStream("image.jpg")) {
        run.addPicture(is, 
                      XWPFDocument.PICTURE_TYPE_JPEG, 
                      "image.jpg", 
                      Units.toEMU(300), 
                      Units.toEMU(200));
    }
    
    try (FileOutputStream fos = new FileOutputStream("document_with_image.docx")) {
        document.write(fos);
    }
    
    document.close();
}
```

### 4.4 读取Word文档

```java
public void readWordDocument(String filePath) throws IOException {
    try (FileInputStream fis = new FileInputStream(filePath);
         XWPFDocument document = new XWPFDocument(fis)) {
        
        // 读取段落
        List<XWPFParagraph> paragraphs = document.getParagraphs();
        for (XWPFParagraph paragraph : paragraphs) {
            System.out.println("段落: " + paragraph.getText());
        }
        
        // 读取表格
        List<XWPFTable> tables = document.getTables();
        for (XWPFTable table : tables) {
            for (XWPFTableRow row : table.getRows()) {
                for (XWPFTableCell cell : row.getTableCells()) {
                    System.out.print(cell.getText() + "\t");
                }
                System.out.println();
            }
        }
        
        // 读取图片
        List<XWPFPictureData> pictures = document.getAllPictures();
        for (XWPFPictureData picture : pictures) {
            System.out.println("图片名称: " + picture.getFileName());
            System.out.println("图片类型: " + picture.getPackagePart().getContentType());
            
            // 保存图片
            byte[] pictureData = picture.getData();
            try (FileOutputStream fos = new FileOutputStream(picture.getFileName())) {
                fos.write(pictureData);
            }
        }
    }
}
```

## 5. SpringBoot集成POI

### 5.1 配置类

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Bean;

@Configuration
public class PoiConfig {
    
    @Bean
    public ExcelService excelService() {
        return new ExcelService();
    }
}
```

### 5.2 Excel服务类

```java
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.http.HttpServletResponse;

@Service
public class ExcelService {
    
    /**
     * 导入Excel
     */
    public List<Map<String, Object>> importExcel(MultipartFile file) throws IOException {
        List<Map<String, Object>> result = new ArrayList<>();
        
        try (InputStream is = file.getInputStream();
             Workbook workbook = WorkbookFactory.create(is)) {
            
            Sheet sheet = workbook.getSheetAt(0);
            
            // 获取表头
            Row headerRow = sheet.getRow(0);
            List<String> headers = new ArrayList<>();
            for (Cell cell : headerRow) {
                headers.add(cell.getStringCellValue());
            }
            
            // 读取数据
            for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                Row row = sheet.getRow(i);
                if (row == null) continue;
                
                Map<String, Object> rowData = new HashMap<>();
                for (int j = 0; j < headers.size(); j++) {
                    Cell cell = row.getCell(j);
                    rowData.put(headers.get(j), getCellValue(cell));
                }
                result.add(rowData);
            }
        }
        
        return result;
    }
    
    /**
     * 导出Excel
     */
    public void exportExcel(List<Map<String, Object>> data, 
                           String fileName, 
                           HttpServletResponse response) throws IOException {
        
        // 设置响应头
        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setHeader("Content-Disposition", 
                          "attachment; filename=" + URLEncoder.encode(fileName, "UTF-8"));
        
        Workbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet("数据");
        
        if (!data.isEmpty()) {
            // 创建表头
            Row headerRow = sheet.createRow(0);
            List<String> headers = new ArrayList<>(data.get(0).keySet());
            for (int i = 0; i < headers.size(); i++) {
                headerRow.createCell(i).setCellValue(headers.get(i));
            }
            
            // 填充数据
            for (int i = 0; i < data.size(); i++) {
                Row row = sheet.createRow(i + 1);
                Map<String, Object> rowData = data.get(i);
                
                for (int j = 0; j < headers.size(); j++) {
                    Object value = rowData.get(headers.get(j));
                    Cell cell = row.createCell(j);
                    
                    if (value instanceof Number) {
                        cell.setCellValue(((Number) value).doubleValue());
                    } else if (value instanceof Date) {
                        cell.setCellValue((Date) value);
                    } else {
                        cell.setCellValue(String.valueOf(value));
                    }
                }
            }
            
            // 自动调整列宽
            for (int i = 0; i < headers.size(); i++) {
                sheet.autoSizeColumn(i);
            }
        }
        
        workbook.write(response.getOutputStream());
        workbook.close();
    }
}
```

### 5.3 控制器实现

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import javax.servlet.http.HttpServletResponse;

@RestController
@RequestMapping("/api/excel")
public class ExcelController {
    
    @Autowired
    private ExcelService excelService;
    
    /**
     * 上传Excel文件
     */
    @PostMapping("/upload")
    public ResponseEntity<?> uploadExcel(@RequestParam("file") MultipartFile file) {
        try {
            if (!file.getOriginalFilename().endsWith(".xlsx") && 
                !file.getOriginalFilename().endsWith(".xls")) {
                return ResponseEntity.badRequest().body("请上传Excel文件");
            }
            
            List<Map<String, Object>> data = excelService.importExcel(file);
            return ResponseEntity.ok(data);
            
        } catch (Exception e) {
            return ResponseEntity.status(500).body("文件处理失败: " + e.getMessage());
        }
    }
    
    /**
     * 下载Excel文件
     */
    @GetMapping("/download")
    public void downloadExcel(HttpServletResponse response) {
        try {
            // 模拟数据
            List<Map<String, Object>> data = new ArrayList<>();
            for (int i = 1; i <= 10; i++) {
                Map<String, Object> row = new HashMap<>();
                row.put("ID", i);
                row.put("姓名", "用户" + i);
                row.put("年龄", 20 + i);
                row.put("创建时间", new Date());
                data.add(row);
            }
            
            excelService.exportExcel(data, "用户数据.xlsx", response);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 导出大量数据
     */
    @GetMapping("/export-large")
    public void exportLargeData(HttpServletResponse response) throws IOException {
        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setHeader("Content-Disposition", "attachment; filename=large_data.xlsx");
        
        // 使用SXSSF处理大数据量
        SXSSFWorkbook workbook = new SXSSFWorkbook(100);
        Sheet sheet = workbook.createSheet("大数据");
        
        // 创建表头
        Row headerRow = sheet.createRow(0);
        String[] headers = {"ID", "姓名", "邮箱", "电话", "地址", "创建时间"};
        for (int i = 0; i < headers.length; i++) {
            headerRow.createCell(i).setCellValue(headers[i]);
        }
        
        // 生成10万条数据
        for (int i = 1; i <= 100000; i++) {
            Row row = sheet.createRow(i);
            row.createCell(0).setCellValue(i);
            row.createCell(1).setCellValue("用户" + i);
            row.createCell(2).setCellValue("user" + i + "@example.com");
            row.createCell(3).setCellValue("1380000" + String.format("%04d", i % 10000));
            row.createCell(4).setCellValue("地址" + i);
            row.createCell(5).setCellValue(new Date().toString());
            
            // 每1000条刷新一次
            if (i % 1000 == 0) {
                ((SXSSFSheet) sheet).flushRows(100);
            }
        }
        
        workbook.write(response.getOutputStream());
        workbook.dispose();
        workbook.close();
    }
}
```

## 6. 实用工具类

### 6.1 Excel工具类

```java
public class ExcelUtil {
    
    /**
     * 判断是否为Excel文件
     */
    public static boolean isExcel(String fileName) {
        return fileName.endsWith(".xls") || fileName.endsWith(".xlsx");
    }
    
    /**
     * 获取Workbook对象
     */
    public static Workbook getWorkbook(InputStream is, String fileName) throws IOException {
        if (fileName.endsWith(".xls")) {
            return new HSSFWorkbook(is);
        } else if (fileName.endsWith(".xlsx")) {
            return new XSSFWorkbook(is);
        } else {
            throw new IllegalArgumentException("文件格式不正确");
        }
    }
    
    /**
     * 获取单元格值
     */
    public static String getCellValue(Cell cell) {
        if (cell == null) return "";
        
        // 强制设置为字符串类型
        cell.setCellType(CellType.STRING);
        return cell.getStringCellValue().trim();
    }
    
    /**
     * 获取单元格数值
     */
    public static Double getCellNumericValue(Cell cell) {
        if (cell == null) return null;
        
        switch (cell.getCellType()) {
            case NUMERIC:
                return cell.getNumericCellValue();
            case STRING:
                try {
                    return Double.parseDouble(cell.getStringCellValue());
                } catch (NumberFormatException e) {
                    return null;
                }
            default:
                return null;
        }
    }
    
    /**
     * 获取日期值
     */
    public static Date getCellDateValue(Cell cell) {
        if (cell == null) return null;
        
        if (cell.getCellType() == CellType.NUMERIC && DateUtil.isCellDateFormatted(cell)) {
            return cell.getDateCellValue();
        }
        
        return null;
    }
    
    /**
     * 设置单元格值（自动判断类型）
     */
    public static void setCellValue(Cell cell, Object value) {
        if (value == null) {
            cell.setCellValue("");
        } else if (value instanceof Number) {
            cell.setCellValue(((Number) value).doubleValue());
        } else if (value instanceof Date) {
            cell.setCellValue((Date) value);
        } else if (value instanceof Boolean) {
            cell.setCellValue((Boolean) value);
        } else {
            cell.setCellValue(String.valueOf(value));
        }
    }
}
```

### 6.2 注解驱动的Excel处理

```java
// 定义注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ExcelColumn {
    String value() default "";
    int order() default 0;
    String format() default "";
    boolean ignore() default false;
}

// 实体类
public class User {
    @ExcelColumn(value = "用户ID", order = 1)
    private Long id;
    
    @ExcelColumn(value = "用户名", order = 2)
    private String username;
    
    @ExcelColumn(value = "邮箱", order = 3)
    private String email;
    
    @ExcelColumn(value = "创建时间", order = 4, format = "yyyy-MM-dd HH:mm:ss")
    private Date createTime;
    
    @ExcelColumn(ignore = true)
    private String password;
    
    // getters and setters
}

// Excel注解处理器
public class ExcelAnnotationProcessor {
    
    /**
     * 导出对象列表到Excel
     */
    public static <T> void exportToExcel(List<T> dataList, Class<T> clazz, OutputStream os) 
            throws IOException, IllegalAccessException {
        
        Workbook workbook = new XSSFWorkbook();
        Sheet sheet = workbook.createSheet();
        
        // 获取标注了ExcelColumn的字段
        List<Field> fields = getExcelFields(clazz);
        
        // 创建表头
        Row headerRow = sheet.createRow(0);
        for (int i = 0; i < fields.size(); i++) {
            Field field = fields.get(i);
            ExcelColumn annotation = field.getAnnotation(ExcelColumn.class);
            Cell cell = headerRow.createCell(i);
            cell.setCellValue(annotation.value());
        }
        
        // 创建数据行
        for (int i = 0; i < dataList.size(); i++) {
            Row row = sheet.createRow(i + 1);
            T obj = dataList.get(i);
            
            for (int j = 0; j < fields.size(); j++) {
                Field field = fields.get(j);
                field.setAccessible(true);
                Object value = field.get(obj);
                
                Cell cell = row.createCell(j);
                if (value != null) {
                    if (field.getType() == Date.class) {
                        ExcelColumn annotation = field.getAnnotation(ExcelColumn.class);
                        String format = annotation.format();
                        if (!format.isEmpty()) {
                            SimpleDateFormat sdf = new SimpleDateFormat(format);
                            cell.setCellValue(sdf.format(value));
                        } else {
                            cell.setCellValue((Date) value);
                        }
                    } else {
                        cell.setCellValue(String.valueOf(value));
                    }
                }
            }
        }
        
        // 自动调整列宽
        for (int i = 0; i < fields.size(); i++) {
            sheet.autoSizeColumn(i);
        }
        
        workbook.write(os);
        workbook.close();
    }
    
    /**
     * 从Excel导入数据
     */
    public static <T> List<T> importFromExcel(InputStream is, Class<T> clazz) 
            throws IOException, InstantiationException, IllegalAccessException {
        
        List<T> result = new ArrayList<>();
        Workbook workbook = WorkbookFactory.create(is);
        Sheet sheet = workbook.getSheetAt(0);
        
        // 获取标注了ExcelColumn的字段
        List<Field> fields = getExcelFields(clazz);
        
        // 读取数据（跳过表头）
        for (int i = 1; i <= sheet.getLastRowNum(); i++) {
            Row row = sheet.getRow(i);
            if (row == null) continue;
            
            T obj = clazz.newInstance();
            
            for (int j = 0; j < fields.size(); j++) {
                Field field = fields.get(j);
                field.setAccessible(true);
                
                Cell cell = row.getCell(j);
                if (cell != null) {
                    Object value = getCellValue(cell, field.getType());
                    field.set(obj, value);
                }
            }
            
            result.add(obj);
        }
        
        workbook.close();
        return result;
    }
    
    private static List<Field> getExcelFields(Class<?> clazz) {
        List<Field> fields = new ArrayList<>();
        
        for (Field field : clazz.getDeclaredFields()) {
            ExcelColumn annotation = field.getAnnotation(ExcelColumn.class);
            if (annotation != null && !annotation.ignore()) {
                fields.add(field);
            }
        }
        
        // 按order排序
        fields.sort(Comparator.comparingInt(f -> 
            f.getAnnotation(ExcelColumn.class).order()));
        
        return fields;
    }
    
    private static Object getCellValue(Cell cell, Class<?> fieldType) {
        String value = cell.getStringCellValue();
        
        if (fieldType == Long.class) {
            return Long.parseLong(value);
        } else if (fieldType == Integer.class) {
            return Integer.parseInt(value);
        } else if (fieldType == Double.class) {
            return Double.parseDouble(value);
        } else if (fieldType == Date.class) {
            return cell.getDateCellValue();
        } else {
            return value;
        }
    }
}
```

## 7. 性能优化技巧

### 7.1 内存优化

```java
public class MemoryOptimization {
    
    /**
     * 使用事件模式读取大文件（SAX方式）
     */
    public void readLargeExcelWithSAX(String filePath) throws Exception {
        OPCPackage pkg = OPCPackage.open(filePath);
        XSSFReader reader = new XSSFReader(pkg);
        
        SharedStringsTable sst = reader.getSharedStringsTable();
        XMLReader parser = XMLReaderFactory.createXMLReader();
        
        // 自定义处理器
        ContentHandler handler = new SheetHandler(sst);
        parser.setContentHandler(handler);
        
        Iterator<InputStream> sheets = reader.getSheetsData();
        while (sheets.hasNext()) {
            InputStream sheet = sheets.next();
            InputSource sheetSource = new InputSource(sheet);
            parser.parse(sheetSource);
            sheet.close();
        }
        
        pkg.close();
    }
    
    /**
     * 自定义Sheet处理器
     */
    private static class SheetHandler extends DefaultHandler {
        private SharedStringsTable sst;
        private String lastContents;
        private boolean nextIsString;
        
        private SheetHandler(SharedStringsTable sst) {
            this.sst = sst;
        }
        
        @Override
        public void startElement(String uri, String localName, String name,
                                Attributes attributes) throws SAXException {
            // c => 单元格
            if(name.equals("c")) {
                String cellType = attributes.getValue("t");
                if(cellType != null && cellType.equals("s")) {
                    nextIsString = true;
                } else {
                    nextIsString = false;
                }
            }
            lastContents = "";
        }
        
        @Override
        public void endElement(String uri, String localName, String name)
                throws SAXException {
            if(nextIsString) {
                int idx = Integer.parseInt(lastContents);
                lastContents = new XSSFRichTextString(sst.getEntryAt(idx)).toString();
                nextIsString = false;
            }
            
            // v => 单元格的值
            if(name.equals("v")) {
                System.out.println(lastContents);
            }
        }
        
        @Override
        public void characters(char[] ch, int start, int length)
                throws SAXException {
            lastContents += new String(ch, start, length);
        }
    }
}
```

### 7.2 批处理优化

```java
@Service
@Transactional
public class BatchExcelService {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    /**
     * 批量导入数据到数据库
     */
    public void batchImportToDatabase(MultipartFile file) throws IOException {
        try (InputStream is = file.getInputStream();
             Workbook workbook = WorkbookFactory.create(is)) {
            
            Sheet sheet = workbook.getSheetAt(0);
            
            List<Object[]> batchArgs = new ArrayList<>();
            
            for (int i = 1; i <= sheet.getLastRowNum(); i++) {
                Row row = sheet.getRow(i);
                if (row == null) continue;
                
                Object[] args = new Object[]{
                    getCellValue(row.getCell(0)),
                    getCellValue(row.getCell(1)),
                    getCellValue(row.getCell(2)),
                    new Date()
                };
                
                batchArgs.add(args);
                
                // 每1000条执行一次批量插入
                if (batchArgs.size() >= 1000) {
                    executeBatchInsert(batchArgs);
                    batchArgs.clear();
                }
            }
            
            // 插入剩余数据
            if (!batchArgs.isEmpty()) {
                executeBatchInsert(batchArgs);
            }
        }
    }
    
    private void executeBatchInsert(List<Object[]> batchArgs) {
        String sql = "INSERT INTO users (name, email, phone, create_time) VALUES (?, ?, ?, ?)";
        jdbcTemplate.batchUpdate(sql, batchArgs);
    }
    
    /**
     * 分页导出数据
     */
    public void exportWithPagination(HttpServletResponse response) throws IOException {
        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setHeader("Content-Disposition", "attachment; filename=data.xlsx");
        
        SXSSFWorkbook workbook = new SXSSFWorkbook(100);
        Sheet sheet = workbook.createSheet();
        
        // 创建表头
        Row headerRow = sheet.createRow(0);
        headerRow.createCell(0).setCellValue("ID");
        headerRow.createCell(1).setCellValue("Name");
        headerRow.createCell(2).setCellValue("Email");
        
        int pageSize = 1000;
        int pageNum = 0;
        int rowNum = 1;
        
        while (true) {
            // 分页查询数据
            String sql = "SELECT id, name, email FROM users LIMIT ? OFFSET ?";
            List<Map<String, Object>> data = jdbcTemplate.queryForList(sql, 
                                                                      pageSize, 
                                                                      pageNum * pageSize);
            
            if (data.isEmpty()) {
                break;
            }
            
            // 写入数据
            for (Map<String, Object> record : data) {
                Row row = sheet.createRow(rowNum++);
                row.createCell(0).setCellValue(String.valueOf(record.get("id")));
                row.createCell(1).setCellValue(String.valueOf(record.get("name")));
                row.createCell(2).setCellValue(String.valueOf(record.get("email")));
            }
            
            // 定期刷新
            if (rowNum % 1000 == 0) {
                ((SXSSFSheet) sheet).flushRows(100);
            }
            
            pageNum++;
        }
        
        workbook.write(response.getOutputStream());
        workbook.dispose();
        workbook.close();
    }
}
```

## 8. 常见问题与解决方案

### 8.1 内存溢出问题

```java
// 问题：处理大文件时OutOfMemoryError
// 解决方案：使用SXSSF或事件模式

// 方案1：使用SXSSF
public void handleLargeFile() {
    SXSSFWorkbook workbook = new SXSSFWorkbook(100);
    // 处理逻辑
    workbook.dispose(); // 清理临时文件
}

// 方案2：增加JVM内存
// -Xmx2g -Xms1g
```

### 8.2 日期格式问题

```java
// 问题：日期显示为数字
// 解决方案：正确设置日期格式

CellStyle dateStyle = workbook.createCellStyle();
CreationHelper createHelper = workbook.getCreationHelper();
dateStyle.setDataFormat(createHelper.createDataFormat().getFormat("yyyy-MM-dd HH:mm:ss"));
cell.setCellStyle(dateStyle);
cell.setCellValue(new Date());
```

### 8.3 公式计算问题

```java
// 问题：公式不自动计算
// 解决方案：强制重新计算

// 创建公式评估器
FormulaEvaluator evaluator = workbook.getCreationHelper().createFormulaEvaluator();

// 方式1：评估单个单元格
Cell cell = row.getCell(0);
if (cell.getCellType() == CellType.FORMULA) {
    evaluator.evaluateInCell(cell);
}

// 方式2：评估所有公式
evaluator.evaluateAll();

// 方式3：设置自动重算
workbook.setForceFormulaRecalculation(true);
```

### 8.4 中文乱码问题

```java
// 问题：中文显示乱码
// 解决方案：设置正确的字符编码

// 文件名编码
String fileName = URLEncoder.encode("中文名称.xlsx", "UTF-8");
response.setHeader("Content-Disposition", "attachment; filename=" + fileName);

// 内容编码
response.setCharacterEncoding("UTF-8");
```

### 8.5 样式丢失问题

```java
// 问题：复制单元格时样式丢失
// 解决方案：正确复制样式

public void copyCellWithStyle(Cell sourceCell, Cell targetCell) {
    // 复制值
    switch (sourceCell.getCellType()) {
        case STRING:
            targetCell.setCellValue(sourceCell.getStringCellValue());
            break;
        case NUMERIC:
            targetCell.setCellValue(sourceCell.getNumericCellValue());
            break;
        case BOOLEAN:
            targetCell.setCellValue(sourceCell.getBooleanCellValue());
            break;
        case FORMULA:
            targetCell.setCellFormula(sourceCell.getCellFormula());
            break;
    }
    
    // 复制样式
    CellStyle newStyle = targetCell.getSheet().getWorkbook().createCellStyle();
    newStyle.cloneStyleFrom(sourceCell.getCellStyle());
    targetCell.setCellStyle(newStyle);
}
```

## 9. 最佳实践

### 9.1 资源管理

```java
// 使用try-with-resources自动关闭资源
try (FileInputStream fis = new FileInputStream("file.xlsx");
     Workbook workbook = WorkbookFactory.create(fis);
     FileOutputStream fos = new FileOutputStream("output.xlsx")) {
    
    // 处理逻辑
    workbook.write(fos);
    
} catch (IOException e) {
    log.error("处理Excel失败", e);
}
```

### 9.2 异常处理

```java
@Service
public class ExcelServiceImpl {
    
    public Result<List<User>> importUsers(MultipartFile file) {
        try {
            // 验证文件
            if (file.isEmpty()) {
                return Result.error("文件不能为空");
            }
            
            String fileName = file.getOriginalFilename();
            if (!ExcelUtil.isExcel(fileName)) {
                return Result.error("请上传Excel文件");
            }
            
            // 处理文件
            List<User> users = parseExcel(file.getInputStream());
            return Result.success(users);
            
        } catch (IOException e) {
            log.error("读取文件失败", e);
            return Result.error("文件读取失败");
        } catch (Exception e) {
            log.error("处理Excel失败", e);
            return Result.error("数据处理失败: " + e.getMessage());
        }
    }
}
```

### 9.3 模板方式导出

```java
public class TemplateExporter {
    
    /**
     * 基于模板导出
     */
    public void exportWithTemplate(String templatePath, 
                                  Map<String, Object> data,
                                  OutputStream os) throws IOException {
        
        try (InputStream is = new FileInputStream(templatePath);
             Workbook workbook = WorkbookFactory.create(is)) {
            
            Sheet sheet = workbook.getSheetAt(0);
            
            // 替换占位符
            for (Row row : sheet) {
                for (Cell cell : row) {
                    if (cell.getCellType() == CellType.STRING) {
                        String value = cell.getStringCellValue();
                        
                        // 替换变量 ${variable}
                        Pattern pattern = Pattern.compile("\\$\\{(.+?)\\}");
                        Matcher matcher = pattern.matcher(value);
                        
                        StringBuffer sb = new StringBuffer();
                        while (matcher.find()) {
                            String key = matcher.group(1);
                            Object replacement = data.get(key);
                            matcher.appendReplacement(sb, 
                                replacement != null ? replacement.toString() : "");
                        }
                        matcher.appendTail(sb);
                        
                        cell.setCellValue(sb.toString());
                    }
                }
            }
            
            workbook.write(os);
        }
    }
}
```

### 9.4 数据验证

```java
public class DataValidator {
    
    /**
     * 添加数据验证
     */
    public void addDataValidation(Sheet sheet) {
        DataValidationHelper helper = sheet.getDataValidationHelper();
        
        // 下拉列表验证
        DataValidationConstraint constraint = helper.createExplicitListConstraint(
            new String[]{"选项1", "选项2", "选项3"}
        );
        
        CellRangeAddressList range = new CellRangeAddressList(1, 100, 0, 0);
        DataValidation validation = helper.createValidation(constraint, range);
        validation.setShowErrorBox(true);
        validation.createErrorBox("错误", "请从下拉列表中选择");
        
        sheet.addValidationData(validation);
        
        // 数字范围验证
        DataValidationConstraint numConstraint = helper.createNumericConstraint(
            DataValidationConstraint.ValidationType.INTEGER,
            DataValidationConstraint.OperatorType.BETWEEN,
            "1", "100"
        );
        
        CellRangeAddressList numRange = new CellRangeAddressList(1, 100, 1, 1);
        DataValidation numValidation = helper.createValidation(numConstraint, numRange);
        numValidation.createErrorBox("错误", "请输入1-100之间的数字");
        
        sheet.addValidationData(numValidation);
    }
}
```

## 10. 总结

### Apache POI的优势

- **功能全面**：支持Excel、Word、PowerPoint等多种Office格式
- **兼容性好**：同时支持新旧版本的Office格式
- **社区活跃**：有大量的文档和示例代码
- **与Spring集成方便**：可轻松集成到SpringBoot项目中

### 使用建议

1. **选择合适的API**：
   - 小文件：使用XSSF/HSSF
   - 大文件：使用SXSSF或事件模式
   - 简单读取：使用WorkbookFactory
2. **注意资源管理**：
   - 及时关闭流和工作簿
   - 使用try-with-resources
   - SXSSF使用后调用dispose()
3. **性能优化**：
   - 批量操作使用批处理
   - 大数据量使用流式处理
   - 合理设置缓存大小
4. **错误处理**：
   - 做好文件验证
   - 捕获特定异常
   - 提供友好的错误信息