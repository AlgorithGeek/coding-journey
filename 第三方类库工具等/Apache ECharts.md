# Apache ECharts 完全指南

## 目录

1. [ECharts 概述](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#1-echarts-概述)
2. [环境搭建与引入方式](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#2-环境搭建与引入方式)
3. [Spring Boot 集成方案](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#3-spring-boot-集成方案)
4. [核心概念与架构](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#4-核心概念与架构)
5. [基础配置详解](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#5-基础配置详解)
6. [图表类型详解](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#6-图表类型详解)
7. [数据处理与格式化](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#7-数据处理与格式化)
8. [交互功能实现](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#8-交互功能实现)
9. [主题与样式定制](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#9-主题与样式定制)
10. [性能优化策略](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#10-性能优化策略)
11. [实战案例](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#11-实战案例)
12. [常见问题与解决方案](https://claude.ai/chat/22a302b2-2379-4073-b6b4-60f882afe29e#12-常见问题与解决方案)

------

## 1. ECharts 概述

### 1.1 什么是 Apache ECharts

Apache ECharts（原百度 ECharts）是一个基于 JavaScript 的开源可视化图表库，提供直观、交互丰富、高度可定制的数据可视化图表。2021年1月正式成为Apache软件基金会的顶级项目。

### 1.2 核心特性

- **丰富的图表类型**：支持折线图、柱状图、饼图、散点图、K线图、地图、热力图、关系图等20多种图表类型
- **强大的渲染能力**：支持Canvas和SVG两种渲染方式
- **移动端优化**：良好的移动端适配和触摸事件支持
- **跨平台支持**：可在PC、移动设备、Node.js、微信小程序等多平台运行
- **高度可定制**：几乎所有组件和图表元素都可以自定义
- **数据驱动**：数据与视图分离，便于动态更新
- **无障碍访问**：支持ARIA标准，提供无障碍访问支持

### 1.3 版本选择

```markdown
- ECharts 5.x（推荐）：最新版本，性能优化，新增多种图表类型
- ECharts 4.x：稳定版本，广泛使用，文档完善
- ECharts 3.x：旧版本，不推荐新项目使用
```

------

## 2. 环境搭建与引入方式

### 2.1 CDN 引入

```html
<!-- 引入 ECharts 5 -->
<script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>

<!-- 或使用具体版本 -->
<script src="https://cdn.jsdelivr.net/npm/echarts@5.4.3/dist/echarts.min.js"></script>

<!-- 引入精简版（体积更小，但功能有限） -->
<script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.simple.min.js"></script>
```

### 2.2 NPM 安装

```bash
# 安装最新版本
npm install echarts

# 安装指定版本
npm install echarts@5.4.3

# 如果使用 TypeScript
npm install @types/echarts
```

### 2.3 模块化引入

```javascript
// 引入 echarts 核心模块
import * as echarts from 'echarts/core';

// 引入柱状图图表
import { BarChart } from 'echarts/charts';

// 引入提示框、标题、直角坐标系组件
import {
    TitleComponent,
    TooltipComponent,
    GridComponent
} from 'echarts/components';

// 引入 Canvas 渲染器
import { CanvasRenderer } from 'echarts/renderers';

// 注册必须的组件
echarts.use([
    TitleComponent,
    TooltipComponent,
    GridComponent,
    BarChart,
    CanvasRenderer
]);
```

### 2.4 在线定制

访问 [ECharts 在线定制](https://echarts.apache.org/zh/builder.html) 可以按需选择组件，生成最小化的ECharts库。

------

## 3. Spring Boot 集成方案

### 3.1 项目结构

```
spring-boot-echarts/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/
│   │   │       ├── controller/
│   │   │       │   ├── ChartController.java
│   │   │       │   └── DataController.java
│   │   │       ├── service/
│   │   │       │   └── ChartDataService.java
│   │   │       ├── model/
│   │   │       │   └── ChartData.java
│   │   │       └── Application.java
│   │   └── resources/
│   │       ├── static/
│   │       │   ├── js/
│   │       │   │   └── echarts.min.js
│   │       │   └── css/
│   │       ├── templates/
│   │       │   ├── dashboard.html
│   │       │   └── chart.html
│   │       └── application.yml
└── pom.xml
```

### 3.2 Maven 依赖配置

```xml
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Thymeleaf 模板引擎 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    
    <!-- JSON 处理 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
    
    <!-- WebJars 方式引入 ECharts（可选） -->
    <dependency>
        <groupId>org.webjars</groupId>
        <artifactId>echarts</artifactId>
        <version>5.4.3</version>
    </dependency>
</dependencies>
```

### 3.3 后端数据接口

```java
// ChartData.java - 数据模型
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ChartData {
    private String name;
    private Object value;
    private String type;
    private Map<String, Object> extra;
}

// ChartDataService.java - 业务逻辑
@Service
public class ChartDataService {
    
    public Map<String, Object> getLineChartData() {
        Map<String, Object> result = new HashMap<>();
        
        // X轴数据
        List<String> xAxis = Arrays.asList("周一", "周二", "周三", "周四", "周五", "周六", "周日");
        
        // 系列数据
        List<Integer> series1 = Arrays.asList(120, 132, 101, 134, 90, 230, 210);
        List<Integer> series2 = Arrays.asList(220, 182, 191, 234, 290, 330, 310);
        
        result.put("xAxis", xAxis);
        result.put("series", Arrays.asList(
            Map.of("name", "邮件营销", "data", series1),
            Map.of("name", "联盟广告", "data", series2)
        ));
        
        return result;
    }
    
    public List<Map<String, Object>> getPieChartData() {
        List<Map<String, Object>> data = new ArrayList<>();
        data.add(Map.of("value", 1048, "name", "搜索引擎"));
        data.add(Map.of("value", 735, "name", "直接访问"));
        data.add(Map.of("value", 580, "name", "邮件营销"));
        data.add(Map.of("value", 484, "name", "联盟广告"));
        data.add(Map.of("value", 300, "name", "视频广告"));
        return data;
    }
    
    // 实时数据推送
    public Map<String, Object> getRealTimeData() {
        Map<String, Object> data = new HashMap<>();
        data.put("time", new Date());
        data.put("value", Math.random() * 100);
        data.put("category", "实时数据");
        return data;
    }
}

// DataController.java - REST控制器
@RestController
@RequestMapping("/api/chart")
public class DataController {
    
    @Autowired
    private ChartDataService chartDataService;
    
    @GetMapping("/line")
    public ResponseEntity<Map<String, Object>> getLineData() {
        return ResponseEntity.ok(chartDataService.getLineChartData());
    }
    
    @GetMapping("/pie")
    public ResponseEntity<List<Map<String, Object>>> getPieData() {
        return ResponseEntity.ok(chartDataService.getPieChartData());
    }
    
    @GetMapping("/realtime")
    public ResponseEntity<Map<String, Object>> getRealTimeData() {
        return ResponseEntity.ok(chartDataService.getRealTimeData());
    }
    
    // 支持 Server-Sent Events 实时推送
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<Map<String, Object>>> streamData() {
        return Flux.interval(Duration.ofSeconds(1))
            .map(sequence -> ServerSentEvent.<Map<String, Object>>builder()
                .id(String.valueOf(sequence))
                .event("chart-data")
                .data(chartDataService.getRealTimeData())
                .build());
    }
}
```

### 3.4 前端页面模板

```html
<!-- dashboard.html - Thymeleaf 模板 -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>ECharts Dashboard</title>
    <script src="/webjars/echarts/5.4.3/dist/echarts.min.js"></script>
    <style>
        .chart-container {
            width: 100%;
            height: 400px;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <h1>数据可视化仪表板</h1>
    
    <!-- 折线图容器 -->
    <div id="lineChart" class="chart-container"></div>
    
    <!-- 饼图容器 -->
    <div id="pieChart" class="chart-container"></div>
    
    <!-- 实时数据图表 -->
    <div id="realtimeChart" class="chart-container"></div>
    
    <script>
        // 初始化折线图
        var lineChart = echarts.init(document.getElementById('lineChart'));
        
        // 从后端获取数据并渲染
        fetch('/api/chart/line')
            .then(response => response.json())
            .then(data => {
                var option = {
                    title: {
                        text: '一周销售趋势'
                    },
                    tooltip: {
                        trigger: 'axis'
                    },
                    legend: {
                        data: data.series.map(s => s.name)
                    },
                    xAxis: {
                        type: 'category',
                        data: data.xAxis
                    },
                    yAxis: {
                        type: 'value'
                    },
                    series: data.series.map(s => ({
                        name: s.name,
                        type: 'line',
                        data: s.data
                    }))
                };
                lineChart.setOption(option);
            });
        
        // 初始化饼图
        var pieChart = echarts.init(document.getElementById('pieChart'));
        
        fetch('/api/chart/pie')
            .then(response => response.json())
            .then(data => {
                var option = {
                    title: {
                        text: '访问来源',
                        left: 'center'
                    },
                    tooltip: {
                        trigger: 'item'
                    },
                    legend: {
                        orient: 'vertical',
                        left: 'left'
                    },
                    series: [{
                        name: '访问来源',
                        type: 'pie',
                        radius: '50%',
                        data: data,
                        emphasis: {
                            itemStyle: {
                                shadowBlur: 10,
                                shadowOffsetX: 0,
                                shadowColor: 'rgba(0, 0, 0, 0.5)'
                            }
                        }
                    }]
                };
                pieChart.setOption(option);
            });
        
        // 实时数据图表
        var realtimeChart = echarts.init(document.getElementById('realtimeChart'));
        var realtimeData = [];
        var timeData = [];
        
        var realtimeOption = {
            title: {
                text: '实时数据监控'
            },
            tooltip: {
                trigger: 'axis'
            },
            xAxis: {
                type: 'category',
                data: timeData
            },
            yAxis: {
                type: 'value',
                boundaryGap: [0, '100%']
            },
            series: [{
                name: '实时数据',
                type: 'line',
                smooth: true,
                data: realtimeData
            }]
        };
        
        realtimeChart.setOption(realtimeOption);
        
        // 使用 EventSource 接收实时数据
        var eventSource = new EventSource('/api/chart/stream');
        eventSource.addEventListener('chart-data', function(e) {
            var data = JSON.parse(e.data);
            
            // 保持最近20个数据点
            if (realtimeData.length > 20) {
                realtimeData.shift();
                timeData.shift();
            }
            
            realtimeData.push(data.value);
            timeData.push(new Date(data.time).toLocaleTimeString());
            
            realtimeChart.setOption({
                xAxis: {
                    data: timeData
                },
                series: [{
                    data: realtimeData
                }]
            });
        });
        
        // 窗口大小改变时重新调整图表
        window.addEventListener('resize', function() {
            lineChart.resize();
            pieChart.resize();
            realtimeChart.resize();
        });
    </script>
</body>
</html>
```

------

## 4. 核心概念与架构

### 4.1 ECharts 实例

```javascript
// 创建 ECharts 实例
var myChart = echarts.init(dom, theme, opts);

// 参数说明：
// dom: HTMLDivElement | HTMLCanvasElement - 容器元素
// theme: Object | string - 主题配置
// opts: {
//     devicePixelRatio: number,  // 设备像素比
//     renderer: 'canvas' | 'svg', // 渲染器
//     width: number | string,     // 宽度
//     height: number | string,    // 高度
//     locale: string              // 语言
// }
```

### 4.2 配置项架构

```javascript
option = {
    // 全局配置
    title: {},        // 标题组件
    legend: {},       // 图例组件
    grid: {},         // 直角坐标系网格
    xAxis: {},        // X轴
    yAxis: {},        // Y轴
    polar: {},        // 极坐标系
    radiusAxis: {},   // 极坐标径向轴
    angleAxis: {},    // 极坐标角度轴
    radar: {},        // 雷达图坐标系
    dataZoom: [],     // 数据区域缩放
    visualMap: [],    // 视觉映射
    tooltip: {},      // 提示框
    axisPointer: {},  // 坐标轴指示器
    toolbox: {},      // 工具栏
    brush: {},        // 区域选择
    geo: {},          // 地理坐标系
    parallel: {},     // 平行坐标系
    parallelAxis: [], // 平行坐标轴
    singleAxis: [],   // 单轴
    timeline: {},     // 时间线
    graphic: [],      // 图形元素
    calendar: [],     // 日历坐标系
    dataset: {},      // 数据集
    aria: {},         // 无障碍
    series: [],       // 系列列表
    color: [],        // 调色盘
    backgroundColor: string, // 背景色
    textStyle: {},    // 全局文字样式
    animation: true,  // 是否开启动画
    animationThreshold: 2000,     // 动画阈值
    animationDuration: 1000,      // 动画时长
    animationEasing: 'cubicOut',  // 动画缓动
    animationDelay: 0,            // 动画延迟
    animationDurationUpdate: 300, // 更新动画时长
    animationEasingUpdate: 'cubicOut', // 更新动画缓动
    animationDelayUpdate: 0,      // 更新动画延迟
    stateAnimation: {},            // 状态切换动画
    blendMode: 'source-over',     // 图形混合模式
    hoverLayerThreshold: 3000,    // hover层阈值
    useUTC: false                  // 是否使用UTC时间
};
```

### 4.3 坐标系

ECharts 支持多种坐标系：

1. **直角坐标系（Cartesian）**
   - grid + xAxis + yAxis
   - 适用于：折线图、柱状图、散点图
2. **极坐标系（Polar）**
   - polar + radiusAxis + angleAxis
   - 适用于：极坐标柱状图、角度堆叠图
3. **地理坐标系（Geo）**
   - geo + 地图数据
   - 适用于：地图、热力图
4. **单轴坐标系（Single）**
   - singleAxis
   - 适用于：单轴散点图
5. **平行坐标系（Parallel）**
   - parallel + parallelAxis
   - 适用于：平行坐标图
6. **雷达坐标系（Radar）**
   - radar
   - 适用于：雷达图
7. **日历坐标系（Calendar）**
   - calendar
   - 适用于：日历图

------

## 5. 基础配置详解

### 5.1 标题配置（Title）

```javascript
title: {
    show: true,                 // 是否显示
    text: '主标题',             // 主标题文本
    link: '',                   // 主标题链接
    target: 'blank',            // 链接打开方式
    subtext: '副标题',          // 副标题文本
    sublink: '',                // 副标题链接
    subtarget: 'blank',         // 副标题链接打开方式
    
    // 位置配置
    left: 'center',             // 左侧位置：'left' | 'center' | 'right' | number | percentage
    top: 'auto',                // 顶部位置
    right: 'auto',              // 右侧位置
    bottom: 'auto',             // 底部位置
    
    // 样式配置
    textStyle: {
        color: '#333',
        fontStyle: 'normal',
        fontWeight: 'bold',
        fontFamily: 'sans-serif',
        fontSize: 18,
        lineHeight: 20,
        width: 200,
        height: 50,
        textBorderColor: 'transparent',
        textBorderWidth: 0,
        textBorderType: 'solid',
        textShadowColor: 'transparent',
        textShadowBlur: 0,
        textShadowOffsetX: 0,
        textShadowOffsetY: 0,
        overflow: 'truncate',   // 'truncate' | 'break' | 'breakAll'
        ellipsis: '...'
    },
    
    subtextStyle: {
        // 副标题样式，同 textStyle
    },
    
    // 其他配置
    textAlign: 'auto',          // 水平对齐
    textVerticalAlign: 'auto',  // 垂直对齐
    triggerEvent: false,        // 是否触发事件
    padding: 5,                 // 内边距
    itemGap: 10,               // 主副标题间距
    zlevel: 0,                 // 层级
    z: 2,                      // 组件层级
    backgroundColor: 'transparent',
    borderColor: '#ccc',
    borderWidth: 0,
    borderRadius: 0,
    shadowBlur: 0,
    shadowColor: 'transparent',
    shadowOffsetX: 0,
    shadowOffsetY: 0
}
```

### 5.2 图例配置（Legend）

```javascript
legend: {
    type: 'plain',              // 'plain' | 'scroll'
    show: true,
    zlevel: 0,
    z: 2,
    left: 'center',
    top: 'auto',
    right: 'auto',
    bottom: 'auto',
    width: 'auto',
    height: 'auto',
    orient: 'horizontal',       // 'horizontal' | 'vertical'
    align: 'auto',              // 'auto' | 'left' | 'right'
    padding: 5,
    itemGap: 10,               // 图例间距
    itemWidth: 25,             // 图例标记宽度
    itemHeight: 14,            // 图例标记高度
    itemStyle: {               // 图例图形样式
        color: 'inherit',
        borderColor: '#000',
        borderWidth: 0,
        borderType: 'solid',
        opacity: 1
    },
    lineStyle: {               // 图例线条样式
        color: 'inherit',
        width: 1,
        type: 'solid',
        opacity: 1
    },
    symbolRotate: 'inherit',    // 图例标记旋转角度
    
    formatter: function (name) {
        return 'Legend ' + name;
    },
    
    selectedMode: true,         // true | false | 'single' | 'multiple'
    inactiveColor: '#ccc',      // 图例关闭时颜色
    inactiveBorderColor: '#ccc',
    
    selected: {                 // 图例选中状态
        '系列1': true,
        '系列2': false
    },
    
    textStyle: {
        color: '#333',
        fontSize: 12
    },
    
    tooltip: {                  // 图例提示框
        show: false
    },
    
    icon: 'circle',             // 图例图标类型
    // 'circle', 'rect', 'roundRect', 'triangle', 'diamond', 'pin', 'arrow', 'none'
    // 或自定义路径 'path://M30.9,53.2...'
    
    data: [{                    // 图例数据
        name: '系列1',
        icon: 'circle',
        itemStyle: {},
        lineStyle: {},
        textStyle: {}
    }],
    
    backgroundColor: 'transparent',
    borderColor: '#ccc',
    borderWidth: 1,
    borderRadius: 0,
    shadowBlur: 0,
    shadowColor: '#000',
    shadowOffsetX: 0,
    shadowOffsetY: 0,
    
    // scroll 类型特有
    scrollDataIndex: 0,
    pageButtonItemGap: 5,
    pageButtonGap: null,
    pageButtonPosition: 'end',  // 'start' | 'end'
    pageFormatter: '{current}/{total}',
    pageIcons: {
        horizontal: ['M0,0L12,-10L12,10z', 'M0,0L-12,-10L-12,10z'],
        vertical: ['M0,0L20,0L10,-20z', 'M0,0L20,0L10,20z']
    },
    pageIconColor: '#2f4554',
    pageIconInactiveColor: '#aaa',
    pageIconSize: 15,
    pageTextStyle: {
        color: '#333'
    },
    
    animation: true,
    animationDurationUpdate: 800
}
```

### 5.3 提示框配置（Tooltip）

```javascript
tooltip: {
    show: true,
    trigger: 'item',            // 'item' | 'axis' | 'none'
    axisPointer: {              // 坐标轴指示器
        type: 'line',           // 'line' | 'shadow' | 'none' | 'cross'
        axis: 'auto',           // 'x' | 'y' | 'radius' | 'angle'
        snap: false,
        z: 0,
        label: {
            show: false,
            precision: 'auto',
            formatter: null,
            margin: 3,
            color: '#fff',
            fontStyle: 'normal',
            fontWeight: 'normal',
            fontFamily: 'sans-serif',
            fontSize: 12,
            lineHeight: 12,
            padding: [5, 7, 5, 7],
            backgroundColor: 'auto',
            borderColor: null,
            borderWidth: 0,
            shadowBlur: 3,
            shadowColor: '#aaa',
            shadowOffsetX: 0,
            shadowOffsetY: 0
        },
        lineStyle: {
            color: '#555',
            width: 1,
            type: 'solid',
            opacity: 0.7
        },
        shadowStyle: {
            color: 'rgba(150,150,150,0.3)',
            opacity: 1
        },
        crossStyle: {
            color: '#555',
            width: 1,
            type: 'dashed',
            opacity: 0.7
        }
    },
    
    showContent: true,          // 是否显示提示框内容
    alwaysShowContent: false,   // 是否一直显示
    triggerOn: 'mousemove|click', // 触发条件
    showDelay: 0,               // 显示延迟
    hideDelay: 100,             // 隐藏延迟
    enterable: true,            // 鼠标是否可进入提示框
    renderMode: 'html',         // 'html' | 'richText'
    confine: false,             // 是否将tooltip限制在图表区域内
    appendToBody: false,        // 是否将tooltip添加到body
    className: '',              // 额外CSS类名
    transitionDuration: 0.4,    // 动画时长
    
    position: function (point, params, dom, rect, size) {
        // point: 鼠标位置
        // params: 数据信息
        // dom: tooltip的dom对象
        // rect: 只有鼠标在图形上时有效
        // size: {contentSize: [width, height], viewSize: [width, height]}
        return [point[0], point[1]];
    },
    
    formatter: function (params, ticket, callback) {
        // params: 数据信息
        // ticket: 异步回调标识
        // callback: 异步回调函数
        return params.seriesName + '<br/>' +
               params.name + ': ' + params.value;
    },
    
    valueFormatter: function (value) {
        return value.toFixed(2);
    },
    
    backgroundColor: 'rgba(50,50,50,0.7)',
    borderColor: '#333',
    borderWidth: 0,
    padding: 5,
    textStyle: {
        color: '#fff',
        fontStyle: 'normal',
        fontWeight: 'normal',
        fontFamily: 'sans-serif',
        fontSize: 14,
        lineHeight: 20
    },
    
    extraCssText: '',           // 额外CSS样式
    
    order: 'seriesAsc'          // 排序方式
    // 'seriesAsc' | 'seriesDesc' | 'valueAsc' | 'valueDesc'
}
```

### 5.4 工具栏配置（Toolbox）

```javascript
toolbox: {
    show: true,
    orient: 'horizontal',       // 'horizontal' | 'vertical'
    itemSize: 15,               // 工具栏图标大小
    itemGap: 10,                // 图标间隔
    showTitle: true,            // 是否显示工具提示
    
    feature: {
        saveAsImage: {          // 保存为图片
            show: true,
            type: 'png',        // 'png' | 'jpg'
            name: 'chart',      // 文件名
            backgroundColor: 'auto',
            connectedBackgroundColor: '#fff',
            excludeComponents: ['toolbox'],
            pixelRatio: 1,
            title: '保存为图片'
        },
        
        restore: {              // 配置项还原
            show: true,
            title: '还原'
        },
        
        dataView: {             // 数据视图
            show: true,
            title: '数据视图',
            readOnly: false,
            lang: ['数据视图', '关闭', '刷新'],
            backgroundColor: '#fff',
            textareaColor: '#fff',
            textareaBorderColor: '#333',
            textColor: '#000',
            buttonColor: '#c23531',
            buttonTextColor: '#fff'
        },
        
        dataZoom: {             // 数据区域缩放
            show: true,
            title: {
                zoom: '区域缩放',
                back: '区域缩放还原'
            },
            filterMode: 'filter', // 'filter' | 'weakFilter' | 'empty' | 'none'
            xAxisIndex: false,
            yAxisIndex: false
        },
        
        magicType: {            // 动态类型切换
            show: true,
            type: ['line', 'bar', 'stack'],
            title: {
                line: '切换为折线图',
                bar: '切换为柱状图',
                stack: '切换为堆叠'
            }
        },
        
        brush: {                // 选框组件
            type: ['rect', 'polygon', 'lineX', 'lineY', 'keep', 'clear'],
            title: {
                rect: '矩形选择',
                polygon: '圈选',
                lineX: '横向选择',
                lineY: '纵向选择',
                keep: '保持选择',
                clear: '清除选择'
            }
        }
    },
    
    iconStyle: {
        borderColor: '#666',
        borderWidth: 1
    },
    
    emphasis: {
        iconStyle: {
            borderColor: '#000',
            shadowBlur: 10,
            shadowColor: 'rgba(0,0,0,0.5)',
            shadowOffsetX: 0,
            shadowOffsetY: 0,
            borderWidth: 0
        }
    },
    
    zlevel: 0,
    z: 2,
    left: 'auto',
    top: 'auto',
    right: 'auto',
    bottom: 'auto',
    width: 'auto',
    height: 'auto'
}
```

------

## 6. 图表类型详解

### 6.1 折线图（Line）

```javascript
series: [{
    name: '折线图',
    type: 'line',
    
    // 坐标系
    coordinateSystem: 'cartesian2d',
    xAxisIndex: 0,
    yAxisIndex: 0,
    polarIndex: 0,
    
    // 数据
    data: [
        120, 132, 101, 134, 90, 230, 210
        // 或对象格式
        // {value: 120, name: '周一'}
    ],
    
    // 折线样式
    lineStyle: {
        color: '#5470c6',
        width: 2,
        type: 'solid',       // 'solid' | 'dashed' | 'dotted'
        opacity: 1,
        shadowBlur: 0,
        shadowColor: 'rgba(0, 0, 0, 0.5)',
        shadowOffsetX: 0,
        shadowOffsetY: 0
    },
    
    // 区域填充样式
    areaStyle: {
        color: 'rgba(84, 112, 198, 0.5)',
        origin: 'auto',      // 'auto' | 'start' | 'end'
        shadowBlur: 0,
        shadowColor: 'rgba(0, 0, 0, 0.5)',
        shadowOffsetX: 0,
        shadowOffsetY: 0,
        opacity: 1
    },
    
    // 标记点
    symbol: 'circle',        // 'circle' | 'rect' | 'roundRect' | 'triangle' | 'diamond' | 'pin' | 'arrow' | 'none'
    symbolSize: 4,
    symbolRotate: 0,
    symbolKeepAspect: false,
    symbolOffset: [0, 0],
    
    showSymbol: true,
    showAllSymbol: 'auto',
    
    // 标记点样式
    itemStyle: {
        color: '#5470c6',
        borderColor: '#000',
        borderWidth: 0,
        borderType: 'solid',
        opacity: 1
    },
    
    // 高亮样式
    emphasis: {
        scale: true,
        focus: 'series',     // 'none' | 'self' | 'series'
        blurScope: 'coordinateSystem',
        lineStyle: {},
        areaStyle: {},
        itemStyle: {}
    },
    
    // 淡出样式
    blur: {
        lineStyle: {},
        areaStyle: {},
        itemStyle: {}
    },
    
    // 选中样式
    select: {
        disabled: false,
        lineStyle: {},
        areaStyle: {},
        itemStyle: {}
    },
    
    // 标签
    label: {
        show: false,
        position: 'top',     // 位置
        distance: 5,
        rotate: 0,
        offset: [0, 0],
        formatter: '{c}',
        color: 'auto',
        fontStyle: 'normal',
        fontWeight: 'normal',
        fontFamily: 'sans-serif',
        fontSize: 12,
        align: 'center',
        verticalAlign: 'middle',
        lineHeight: 12,
        backgroundColor: 'transparent',
        borderColor: 'transparent',
        borderWidth: 0,
        borderRadius: 0,
        padding: 0,
        shadowColor: 'transparent',
        shadowBlur: 0,
        shadowOffsetX: 0,
        shadowOffsetY: 0
    },
    
    // 标注线
    labelLine: {
        show: true,
        length: 10,
        length2: 5,
        smooth: false,
        minTurnAngle: 90,
        lineStyle: {}
    },
    
    // 标线
    markPoint: {
        symbol: 'pin',
        symbolSize: 50,
        symbolRotate: 0,
        symbolKeepAspect: false,
        symbolOffset: [0, 0],
        silent: false,
        label: {},
        itemStyle: {},
        emphasis: {},
        blur: {},
        data: [
            {type: 'max', name: '最大值'},
            {type: 'min', name: '最小值'}
        ]
    },
    
    // 标线
    markLine: {
        silent: false,
        symbol: ['circle', 'arrow'],
        symbolSize: [8, 16],
        precision: 2,
        label: {},
        lineStyle: {},
        emphasis: {},
        blur: {},
        data: [
            {type: 'average', name: '平均值'}
        ]
    },
    
    // 标域
    markArea: {
        silent: false,
        label: {},
        itemStyle: {},
        emphasis: {},
        blur: {},
        data: [
            [{xAxis: 'min', yAxis: 'min'}, {xAxis: 'max', yAxis: 'max'}]
        ]
    },
    
    // 其他配置
    smooth: false,           // 平滑曲线
    smoothMonotone: null,    // 平滑约束
    sampling: 'none',        // 采样策略 'lttb' | 'average' | 'min' | 'max' | 'sum'
    stack: null,            // 堆叠
    cursor: 'pointer',
    connectNulls: false,    // 连接空数据
    step: false,            // 阶梯线 true | false | 'start' | 'middle' | 'end'
    clip: true,             // 裁剪超出坐标系的部分
    
    zlevel: 0,
    z: 2,
    silent: false,
    
    animation: true,
    animationThreshold: 2000,
    animationDuration: 1000,
    animationEasing: 'linear',
    animationDelay: 0,
    animationDurationUpdate: 300,
    animationEasingUpdate: 'cubicInOut',
    animationDelayUpdate: 0
}]
```

### 6.2 柱状图（Bar）

```javascript
series: [{
    name: '柱状图',
    type: 'bar',
    
    // 数据
    data: [120, 200, 150, 80, 70, 110, 130],
    
    // 柱条样式
    itemStyle: {
        color: 'auto',
        borderColor: '#000',
        borderWidth: 0,
        borderType: 'solid',
        barBorderRadius: 0,
        shadowBlur: 0,
        shadowColor: 'rgba(0, 0, 0, 0.5)',
        shadowOffsetX: 0,
        shadowOffsetY: 0,
        opacity: 1
    },
    
    // 柱条的背景
    showBackground: false,
    backgroundStyle: {
        color: 'rgba(180, 180, 180, 0.2)',
        borderColor: '#000',
        borderWidth: 0,
        borderType: 'solid',
        barBorderRadius: 0,
        shadowBlur: 0,
        shadowColor: 'rgba(0, 0, 0, 0.5)',
        shadowOffsetX: 0,
        shadowOffsetY: 0,
        opacity: 1
    },
    
    // 柱条宽度
    barWidth: 'auto',        // 自适应 | 具体像素值 | 百分比
    barMaxWidth: null,       // 最大宽度
    barMinWidth: null,       // 最小宽度
    barMinHeight: 0,         // 最小高度
    barGap: '30%',           // 不同系列柱间距离
    barCategoryGap: '20%',   // 同系列柱间距离
    
    // 堆叠
    stack: null,             // 堆叠组名
    
    // 标签
    label: {
        show: false,
        position: 'top',     // 'top' | 'left' | 'right' | 'bottom' | 'inside' | 'insideLeft' | ...
        distance: 5,
        rotate: 0,
        formatter: '{c}'
    },
    
    // 高亮状态
    emphasis: {
        disabled: false,
        focus: 'series',
        blurScope: 'coordinateSystem',
        itemStyle: {},
        label: {}
    },
    
    // 淡出状态
    blur: {
        itemStyle: {},
        label: {}
    },
    
    // 选中状态
    select: {
        disabled: false,
        itemStyle: {},
        label: {}
    },
    
    // 大数据优化
    large: false,            // 是否开启大数据优化
    largeThreshold: 400,     // 大数据优化阈值
    progressive: 5000,       // 渐进式渲染数据量
    progressiveThreshold: 10000,
    progressiveChunkMode: 'sequential', // 'sequential' | 'mod'
    
    // 其他配置
    coordinateSystem: 'cartesian2d',
    xAxisIndex: 0,
    yAxisIndex: 0,
    cursor: 'pointer',
    zlevel: 0,
    z: 2,
    silent: false
}]
```

### 6.3 饼图（Pie）

```javascript
series: [{
    name: '饼图',
    type: 'pie',
    
    // 数据
    data: [
        {value: 335, name: '直接访问'},
        {value: 310, name: '邮件营销'},
        {value: 234, name: '联盟广告'},
        {value: 135, name: '视频广告'},
        {value: 1548, name: '搜索引擎'}
    ],
    
    // 位置和大小
    center: ['50%', '50%'],  // 中心位置
    radius: ['0%', '75%'],   // 半径 [内半径, 外半径]
    
    // 扇区样式
    itemStyle: {
        borderColor: '#fff',
        borderWidth: 1,
        borderRadius: 10
    },
    
    // 标签
    label: {
        show: true,
        position: 'outside', // 'outside' | 'inside' | 'center'
        formatter: '{b}: {c} ({d}%)',
        color: null,
        distanceToLabelLine: 5,
        alignTo: 'none',     // 'none' | 'labelLine' | 'edge'
        bleedMargin: 10,
        edgeDistance: '25%'
    },
    
    // 标签线
    labelLine: {
        show: true,
        length: 15,
        length2: 15,
        smooth: false,
        lineStyle: {
            color: '#000',
            width: 1,
            type: 'solid'
        }
    },
    
    // 南丁格尔图（玫瑰图）
    roseType: false,         // false | 'radius' | 'area'
    
    // 选中模式
    selectedMode: false,     // false | 'single' | 'multiple'
    selectedOffset: 10,      // 选中扇区偏移量
    
    // 顺时针排布
    clockwise: true,
    
    // 起始角度
    startAngle: 90,
    
    // 最小扇区角度
    minAngle: 0,
    
    // 最小显示标签角度
    minShowLabelAngle: 0,
    
    // 避免标签重叠
    avoidLabelOverlap: true,
    
    // 扇区是否忽略
    stillShowZeroSum: true,
    
    // 百分比精度
    percentPrecision: 2,
    
    // 高亮状态
    emphasis: {
        scale: true,
        scaleSize: 10,
        focus: 'self',
        blurScope: 'coordinateSystem',
        itemStyle: {},
        label: {},
        labelLine: {}
    },
    
    // 动画
    animationType: 'expansion', // 'expansion' | 'scale'
    animationUpdate: true,
    animationTypeUpdate: 'transition'
}]
```

### 6.4 散点图（Scatter）

```javascript
series: [{
    name: '散点图',
    type: 'scatter',
    
    // 数据格式：[x, y, size, ...]
    data: [
        [10.0, 8.04],
        [8.0, 6.95],
        [13.0, 7.58],
        [9.0, 8.81],
        [11.0, 8.33]
    ],
    
    // 坐标系
    coordinateSystem: 'cartesian2d',
    xAxisIndex: 0,
    yAxisIndex: 0,
    
    // 标记
    symbol: 'circle',
    symbolSize: function (data) {
        return Math.sqrt(data[2]) * 5;
    },
    symbolRotate: 0,
    symbolKeepAspect: false,
    symbolOffset: [0, 0],
    
    // 大数据优化
    large: false,
    largeThreshold: 2000,
    
    // 标签
    label: {
        show: false,
        position: 'top',
        formatter: function (params) {
            return params.data[2];
        }
    },
    
    // 标记样式
    itemStyle: {
        color: function (params) {
            // 根据数据动态设置颜色
            var colorList = ['#c23531', '#2f4554', '#61a0a8'];
            return colorList[params.dataIndex % colorList.length];
        },
        opacity: 0.8
    },
    
    // 高亮状态
    emphasis: {
        scale: 1.5,
        focus: 'self',
        itemStyle: {},
        label: {}
    }
}]
```

### 6.5 雷达图（Radar）

```javascript
// 雷达图坐标系配置
radar: {
    indicator: [
        {name: '销售', max: 6500},
        {name: '管理', max: 16000},
        {name: '信息技术', max: 30000},
        {name: '客服', max: 38000},
        {name: '研发', max: 52000},
        {name: '市场', max: 25000}
    ],
    center: ['50%', '50%'],
    radius: '75%',
    startAngle: 90,
    splitNumber: 5,
    shape: 'polygon',        // 'polygon' | 'circle'
    
    // 坐标轴名称
    axisName: {
        show: true,
        formatter: '{value}',
        color: '#333',
        fontSize: 12
    },
    
    // 坐标轴
    axisLine: {
        show: true,
        lineStyle: {
            color: '#333',
            width: 1,
            type: 'solid'
        }
    },
    
    // 分割线
    splitLine: {
        show: true,
        lineStyle: {
            color: '#ccc',
            width: 1,
            type: 'solid'
        }
    },
    
    // 分割区域
    splitArea: {
        show: true,
        areaStyle: {
            color: ['rgba(250,250,250,0.3)', 'rgba(200,200,200,0.3)']
        }
    }
},

// 雷达图系列配置
series: [{
    name: '雷达图',
    type: 'radar',
    
    data: [
        {
            value: [4200, 3000, 20000, 35000, 50000, 18000],
            name: '预算分配'
        },
        {
            value: [5000, 14000, 28000, 31000, 42000, 21000],
            name: '实际开销'
        }
    ],
    
    // 拐点标记
    symbol: 'circle',
    symbolSize: 4,
    
    // 线条样式
    lineStyle: {
        width: 2,
        type: 'solid'
    },
    
    // 区域填充
    areaStyle: {
        opacity: 0.5
    },
    
    // 高亮状态
    emphasis: {
        lineStyle: {
            width: 4
        },
        areaStyle: {
            opacity: 0.8
        }
    }
}]
```

------

## 7. 数据处理与格式化

### 7.1 数据集（Dataset）

```javascript
option = {
    dataset: {
        // 提供数据
        source: [
            ['product', '2015', '2016', '2017'],
            ['Matcha Latte', 43.3, 85.8, 93.7],
            ['Milk Tea', 83.1, 73.4, 55.1],
            ['Cheese Cocoa', 86.4, 65.2, 82.5],
            ['Walnut Brownie', 72.4, 53.9, 39.1]
        ]
        
        // 或使用对象数组
        // source: [
        //     {product: 'Matcha Latte', '2015': 43.3, '2016': 85.8, '2017': 93.7},
        //     {product: 'Milk Tea', '2015': 83.1, '2016': 73.4, '2017': 55.1}
        // ]
    },
    
    // 声明X轴对应到dataset的第一列
    xAxis: {type: 'category'},
    
    // 声明多个Y轴对应到dataset的其他列
    yAxis: {},
    
    series: [
        {type: 'bar'},  // 自动对应到 dataset.source 第二列
        {type: 'bar'},  // 自动对应到 dataset.source 第三列
        {type: 'bar'}   // 自动对应到 dataset.source 第四列
    ]
};

// 使用 encode 映射
option = {
    dataset: {
        dimensions: ['product', '2015', '2016', '2017'],
        source: [
            {product: 'Matcha Latte', '2015': 43.3, '2016': 85.8, '2017': 93.7},
            {product: 'Milk Tea', '2015': 83.1, '2016': 73.4, '2017': 55.1}
        ]
    },
    xAxis: {type: 'category'},
    yAxis: {},
    series: [{
        type: 'bar',
        encode: {
            x: 'product',    // 将 'product' 维度编码到 X 轴
            y: '2015'        // 将 '2015' 维度编码到 Y 轴
        }
    }]
};
```

### 7.2 数据转换（Transform）

```javascript
option = {
    dataset: [{
        source: [
            ['product', 'sales', 'price', 'cost'],
            ['Milk', 100, 5.5, 3.5],
            ['Bread', 80, 3.0, 1.5],
            ['Egg', 200, 1.8, 0.8],
            ['Apple', 60, 4.0, 2.0]
        ]
    }, {
        // 数据过滤
        transform: {
            type: 'filter',
            config: { dimension: 'sales', gte: 80 }
        }
    }, {
        // 数据排序
        transform: {
            type: 'sort',
            config: { dimension: 'price', order: 'asc' }
        }
    }],
    
    series: [{
        type: 'bar',
        datasetIndex: 1  // 使用过滤后的数据
    }, {
        type: 'bar',
        datasetIndex: 2  // 使用排序后的数据
    }]
};

// 聚合转换
option = {
    dataset: [{
        source: [/* 原始数据 */]
    }, {
        transform: {
            type: 'ecStat:regression',
            config: {
                method: 'linear',  // 线性回归
                dimension: 0,
                expression: 'y = a * x + b'
            }
        }
    }]
};
```

### 7.3 动态数据更新

```javascript
// 初始化图表
var myChart = echarts.init(document.getElementById('main'));

// 初始配置
var option = {
    xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        type: 'line',
        data: [820, 932, 901, 934, 1290]
    }]
};

myChart.setOption(option);

// 动态更新数据
setInterval(function () {
    // 生成新数据
    var newData = [];
    for (var i = 0; i < 5; i++) {
        newData.push(Math.round(Math.random() * 1000));
    }
    
    // 更新配置
    myChart.setOption({
        series: [{
            data: newData
        }]
    });
}, 2000);

// 增量更新
function appendData(chart, newData) {
    var option = chart.getOption();
    option.xAxis[0].data.push(newData.name);
    option.series[0].data.push(newData.value);
    
    // 保持最多显示10个数据点
    if (option.xAxis[0].data.length > 10) {
        option.xAxis[0].data.shift();
        option.series[0].data.shift();
    }
    
    chart.setOption(option);
}

// 批量更新
function batchUpdate(chart, dataList) {
    chart.setOption({
        series: [{
            data: dataList
        }]
    }, {
        notMerge: false,     // 是否不与之前配置合并
        lazyUpdate: false,   // 是否延迟更新
        silent: false        // 是否阻止事件触发
    });
}
```

### 7.4 数据加载状态

```javascript
// 显示加载动画
myChart.showLoading({
    text: '数据加载中...',
    color: '#c23531',
    textColor: '#000',
    maskColor: 'rgba(255, 255, 255, 0.8)',
    zlevel: 0,
    fontSize: 12,
    showSpinner: true,
    spinnerRadius: 10,
    lineWidth: 5,
    fontWeight: 'normal',
    fontStyle: 'normal',
    fontFamily: 'sans-serif'
});

// 模拟异步数据加载
setTimeout(function () {
    // 隐藏加载动画
    myChart.hideLoading();
    
    // 设置数据
    myChart.setOption({
        series: [{
            data: [/* 加载的数据 */]
        }]
    });
}, 3000);

// 空数据处理
if (data.length === 0) {
    myChart.setOption({
        title: {
            text: '暂无数据',
            left: 'center',
            top: 'center',
            textStyle: {
                color: '#ccc',
                fontSize: 20
            }
        },
        xAxis: {show: false},
        yAxis: {show: false},
        series: []
    });
}
```

------

## 8. 交互功能实现

### 8.1 事件处理

```javascript
// 支持的事件类型
var eventTypes = [
    'click',              // 鼠标点击
    'dblclick',          // 鼠标双击
    'mousedown',         // 鼠标按下
    'mousemove',         // 鼠标移动
    'mouseup',           // 鼠标抬起
    'mouseover',         // 鼠标悬停
    'mouseout',          // 鼠标移出
    'globalout',         // 鼠标移出图表区域
    'contextmenu',       // 右键菜单
    
    // 图例事件
    'legendselectchanged',  // 图例选择改变
    'legendselected',       // 图例选中
    'legendunselected',     // 图例取消选中
    'legendscroll',         // 图例滚动
    
    // 数据区域缩放事件
    'datazoom',          // 数据区域缩放
    'datarangeselected', // 数据范围选择
    
    // 工具栏事件
    'restore',           // 重置
    'dataviewchanged',   // 数据视图修改
    
    // 其他事件
    'magictypechanged',  // 动态类型切换
    'georoam',          // 地图漫游
    'treeroam',         // 树图漫游
    'brushselected',    // 刷选
    'brush',            // 刷选中
    'brushEnd',         // 刷选结束
    'rendered',         // 渲染完成
    'finished'          // 渲染动画完成
];

// 绑定事件
myChart.on('click', function (params) {
    console.log('点击了：', params);
    console.log('组件类型：', params.componentType);
    console.log('系列索引：', params.seriesIndex);
    console.log('数据索引：', params.dataIndex);
    console.log('数据：', params.data);
    console.log('数据名：', params.name);
    console.log('数据值：', params.value);
    console.log('颜色：', params.color);
});

// 条件触发
myChart.on('click', 'series.line', function (params) {
    // 只响应折线图的点击
});

myChart.on('mouseover', {seriesIndex: 0}, function (params) {
    // 只响应第一个系列的鼠标悬停
});

// 解绑事件
myChart.off('click');

// 触发行为
myChart.dispatchAction({
    type: 'highlight',      // 高亮
    seriesIndex: 0,
    dataIndex: 3
});

myChart.dispatchAction({
    type: 'showTip',        // 显示提示框
    seriesIndex: 0,
    dataIndex: 3
});

// 支持的行为类型
var actionTypes = [
    'highlight',         // 高亮
    'downplay',         // 取消高亮
    'select',           // 选中
    'unselect',         // 取消选中
    'toggleSelected',   // 切换选中
    'showTip',          // 显示提示框
    'hideTip',          // 隐藏提示框
    'dataZoom',         // 数据缩放
    'takeGlobalCursor', // 取得全局光标
    'legendSelect',     // 图例选中
    'legendUnSelect',   // 图例取消选中
    'legendToggleSelect', // 图例切换选中
    'legendScroll',     // 图例滚动
    'restore'           // 重置
];
```

### 8.2 数据区域缩放（DataZoom）

```javascript
dataZoom: [{
    // 内置型数据区域缩放组件
    type: 'inside',
    xAxisIndex: [0],        // 控制X轴
    start: 0,               // 数据窗口起始百分比
    end: 100,               // 数据窗口结束百分比
    startValue: null,       // 数据窗口起始值
    endValue: null,         // 数据窗口结束值
    minSpan: null,          // 最小窗口宽度
    maxSpan: null,          // 最大窗口宽度
    minValueSpan: null,     // 窗口最小值跨度
    maxValueSpan: null,     // 窗口最大值跨度
    orient: 'horizontal',   // 布局方向
    zoomLock: false,        // 锁定大小
    throttle: 100,          // 触发频率
    rangeMode: null,        // 范围模式
    zoomOnMouseWheel: true, // 鼠标滚轮缩放
    moveOnMouseMove: true,  // 鼠标移动平移
    moveOnMouseWheel: false,// 鼠标滚轮平移
    preventDefaultMouseMove: true
}, {
    // 滑动条型数据区域缩放组件
    type: 'slider',
    show: true,
    xAxisIndex: [0],
    start: 0,
    end: 100,
    height: 20,             // 组件高度
    bottom: 0,              // 组件位置
    
    // 手柄样式
    handleIcon: 'path://M10,10 L0,0 L10,-10',
    handleSize: '80%',
    handleStyle: {
        color: '#fff',
        borderColor: '#ACB8D1'
    },
    
    // 文本标签
    labelFormatter: function (value, valueStr) {
        return Math.round(value) + '%';
    },
    
    // 选中区域样式
    dataBackground: {
        lineStyle: {
            color: '#8392A5'
        },
        areaStyle: {
            color: '#8392A5',
            opacity: 0.2
        }
    },
    
    // 选中区域外样式
    selectedDataBackground: {
        lineStyle: {
            color: '#8392A5'
        },
        areaStyle: {
            color: '#8392A5',
            opacity: 0.5
        }
    },
    
    // 边框
    borderColor: '#ccc',
    
    // 是否实时更新
    realtime: true,
    
    // 文本样式
    textStyle: {
        color: '#333'
    }
}]
```

### 8.3 视觉映射（VisualMap）

```javascript
visualMap: [{
    // 连续型视觉映射
    type: 'continuous',
    min: 0,
    max: 100,
    text: ['高', '低'],      // 两端文字
    dimension: 0,           // 映射维度
    
    // 定义映射
    inRange: {
        color: ['#50a3ba', '#eac736', '#d94e5d'],
        symbolSize: [10, 50]
    },
    
    // 超出范围
    outOfRange: {
        color: '#ccc'
    },
    
    // 位置和大小
    left: 'center',
    bottom: '10%',
    orient: 'horizontal',   // 'horizontal' | 'vertical'
    
    // 是否显示
    show: true,
    
    // 是否可调节
    calculable: true,
    
    // 是否反向
    inverse: false,
    
    // 精度
    precision: 0,
    
    // 分段数
    splitNumber: 5,
    
    // 选择范围
    range: null,
    
    // 文本样式
    textStyle: {
        color: '#333'
    }
}, {
    // 分段型视觉映射
    type: 'piecewise',
    pieces: [
        {min: 0, max: 20, label: '0-20', color: '#50a3ba'},
        {min: 20, max: 40, label: '20-40', color: '#eac736'},
        {min: 40, max: 60, label: '40-60', color: '#d94e5d'},
        {min: 60, max: 80, label: '60-80', color: '#d94e5d'},
        {min: 80, max: 100, label: '80-100', color: '#d94e5d'}
    ],
    
    // 或使用分段数自动分段
    // splitNumber: 5,
    
    // 定义类别
    // categories: ['低', '中', '高'],
    
    // 选择模式
    selectedMode: 'multiple', // 'multiple' | 'single'
    
    // 项的间隔
    itemGap: 10,
    itemWidth: 20,
    itemHeight: 14,
    
    // 分段文本
    formatter: function (value, value2) {
        return value + ' ~ ' + value2;
    }
}]
```

### 8.4 图形元素（Graphic）

```javascript
graphic: [{
    type: 'group',          // 组
    left: 'center',
    top: 'center',
    children: [{
        type: 'rect',       // 矩形
        z: 100,
        left: 'center',
        top: 'middle',
        shape: {
            width: 190,
            height: 90
        },
        style: {
            fill: '#fff',
            stroke: '#555',
            lineWidth: 2,
            shadowBlur: 8,
            shadowOffsetX: 3,
            shadowOffsetY: 3,
            shadowColor: 'rgba(0,0,0,0.3)'
        }
    }, {
        type: 'text',       // 文本
        z: 100,
        left: 'center',
        top: 'middle',
        style: {
            fill: '#333',
            text: '自定义文本',
            font: 'bold 26px sans-serif'
        }
    }]
}, {
    type: 'image',          // 图片
    style: {
        image: 'http://example.com/image.png',
        width: 100,
        height: 100
    },
    left: 'center',
    top: 'center'
}, {
    type: 'circle',         // 圆形
    shape: {
        cx: 0,
        cy: 0,
        r: 50
    },
    style: {
        fill: '#c23531'
    }
}, {
    type: 'line',           // 直线
    shape: {
        x1: 0,
        y1: 0,
        x2: 100,
        y2: 100
    },
    style: {
        stroke: '#999',
        lineWidth: 2
    }
}, {
    type: 'polygon',        // 多边形
    shape: {
        points: [[0, 0], [100, 0], [100, 100], [0, 100]]
    },
    style: {
        fill: 'rgba(0, 0, 255, 0.5)'
    }
}]
```

------

## 9. 主题与样式定制

### 9.1 内置主题

```javascript
// 使用内置主题
var chart1 = echarts.init(dom, 'light');    // 明亮主题
var chart2 = echarts.init(dom, 'dark');     // 暗黑主题

// 动态切换主题
chart.dispose();  // 先销毁
chart = echarts.init(dom, 'dark');  // 重新初始化
chart.setOption(option);
```

### 9.2 自定义主题

```javascript
// 注册自定义主题
echarts.registerTheme('myTheme', {
    color: ['#c23531', '#2f4554', '#61a0a8', '#d48265', '#91c7ae'],
    backgroundColor: '#f7f7f7',
    
    textStyle: {
        color: '#333',
        fontFamily: 'Arial, sans-serif'
    },
    
    title: {
        textStyle: {
            color: '#333',
            fontSize: 18,
            fontWeight: 'bold'
        },
        subtextStyle: {
            color: '#aaa'
        }
    },
    
    line: {
        itemStyle: {
            borderWidth: 1
        },
        lineStyle: {
            width: 2
        },
        symbolSize: 4,
        smooth: false
    },
    
    bar: {
        itemStyle: {
            barBorderWidth: 0,
            barBorderColor: '#ccc'
        }
    },
    
    pie: {
        itemStyle: {
            borderWidth: 0,
            borderColor: '#ccc'
        }
    },
    
    categoryAxis: {
        axisLine: {
            show: true,
            lineStyle: {
                color: '#333'
            }
        },
        axisTick: {
            show: true,
            lineStyle: {
                color: '#333'
            }
        },
        axisLabel: {
            show: true,
            color: '#333'
        },
        splitLine: {
            show: false,
            lineStyle: {
                color: ['#ccc']
            }
        },
        splitArea: {
            show: false,
            areaStyle: {
                color: ['rgba(250,250,250,0.3)', 'rgba(200,200,200,0.3)']
            }
        }
    },
    
    valueAxis: {
        axisLine: {
            show: false,
            lineStyle: {
                color: '#333'
            }
        },
        axisTick: {
            show: false,
            lineStyle: {
                color: '#333'
            }
        },
        axisLabel: {
            show: true,
            color: '#333'
        },
        splitLine: {
            show: true,
            lineStyle: {
                color: ['#ccc']
            }
        },
        splitArea: {
            show: false,
            areaStyle: {
                color: ['rgba(250,250,250,0.3)', 'rgba(200,200,200,0.3)']
            }
        }
    },
    
    toolbox: {
        iconStyle: {
            borderColor: '#999'
        },
        emphasis: {
            iconStyle: {
                borderColor: '#666'
            }
        }
    },
    
    legend: {
        textStyle: {
            color: '#333'
        }
    },
    
    tooltip: {
        axisPointer: {
            lineStyle: {
                color: '#ccc',
                width: 1
            },
            crossStyle: {
                color: '#ccc',
                width: 1
            }
        }
    },
    
    dataZoom: {
        backgroundColor: 'rgba(47,69,84,0)',
        dataBackgroundColor: 'rgba(47,69,84,0.3)',
        fillerColor: 'rgba(167,183,204,0.4)',
        handleColor: '#a7b7cc',
        handleSize: '100%',
        textStyle: {
            color: '#333'
        }
    },
    
    markPoint: {
        label: {
            color: '#eee'
        },
        emphasis: {
            label: {
                color: '#eee'
            }
        }
    }
});

// 使用自定义主题
var myChart = echarts.init(dom, 'myTheme');
```

### 9.3 调色板

```javascript
option = {
    // 全局调色板
    color: ['#5470c6', '#91cc75', '#fac858', '#ee6666', '#73c0de', '#3ba272'],
    
    series: [{
        type: 'pie',
        // 局部调色板（会覆盖全局调色板）
        color: ['#dd6b66', '#759aa0', '#e69d87', '#8dc1a9', '#ea7e53'],
        
        // 或使用回调函数
        itemStyle: {
            color: function(params) {
                // 自定义颜色逻辑
                var colorList = ['#c23531', '#2f4554', '#61a0a8'];
                return colorList[params.dataIndex % colorList.length];
            }
        }
    }]
};

// 线性渐变
color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
    {offset: 0, color: '#83bff6'},
    {offset: 0.5, color: '#188df0'},
    {offset: 1, color: '#188df0'}
]);

// 径向渐变
color: new echarts.graphic.RadialGradient(0.5, 0.5, 0.5, [
    {offset: 0, color: '#ffffff'},
    {offset: 1, color: '#000000'}
]);

// 纹理
color: {
    image: imageDom,  // 支持 Canvas/HTMLImageElement/HTMLCanvasElement
    repeat: 'repeat'  // 'repeat' | 'repeat-x' | 'repeat-y' | 'no-repeat'
}
```

------

## 10. 性能优化策略

### 10.1 大数据量优化

```javascript
// 1. 开启大数据优化模式
series: [{
    type: 'scatter',
    large: true,             // 开启大数据优化
    largeThreshold: 2000,    // 大数据阈值
    data: largeData
}]

// 2. 使用采样
series: [{
    type: 'line',
    sampling: 'lttb',        // 采样算法
    data: data
}]

// 3. 渐进式渲染
series: [{
    type: 'bar',
    progressive: 5000,       // 渐进式渲染阈值
    progressiveThreshold: 10000,
    progressiveChunkMode: 'sequential'
}]

// 4. 使用 SVG 渲染器（适合数据量较小但图形复杂的场景）
var myChart = echarts.init(dom, null, {
    renderer: 'svg'
});

// 5. 数据分片加载
function loadDataInChunks(chart, allData, chunkSize) {
    var chunks = [];
    for (var i = 0; i < allData.length; i += chunkSize) {
        chunks.push(allData.slice(i, i + chunkSize));
    }
    
    var currentChunk = 0;
    function loadNextChunk() {
        if (currentChunk < chunks.length) {
            chart.appendData({
                seriesIndex: 0,
                data: chunks[currentChunk]
            });
            currentChunk++;
            setTimeout(loadNextChunk, 100);
        }
    }
    
    loadNextChunk();
}
```

### 10.2 动画优化

```javascript
option = {
    // 关闭动画
    animation: false,
    
    // 或调整动画配置
    animation: true,
    animationThreshold: 2000,     // 动画阈值
    animationDuration: function (idx) {
        // 根据数据点索引返回不同的动画时长
        return idx * 50;
    },
    animationEasing: 'cubicOut',
    animationDelay: function (idx) {
        // 根据数据点索引返回不同的延迟时间
        return idx * 10;
    },
    
    // 更新动画
    animationDurationUpdate: 300,
    animationEasingUpdate: 'cubicOut',
    
    // 状态切换动画
    stateAnimation: {
        duration: 300,
        easing: 'cubicOut'
    }
};
```

### 10.3 内存优化

```javascript
// 1. 及时销毁实例
if (myChart) {
    myChart.dispose();
    myChart = null;
}

// 2. 清理不必要的数据
myChart.clear();

// 3. 使用 notMerge 选项避免数据合并
myChart.setOption(option, {
    notMerge: true  // 不与之前的option合并
});

// 4. 移除事件监听
myChart.off('click');
myChart.off('mouseover');

// 5. 使用 WeakMap 存储图表实例
const chartMap = new WeakMap();
function getChart(dom) {
    if (!chartMap.has(dom)) {
        chartMap.set(dom, echarts.init(dom));
    }
    return chartMap.get(dom);
}
```

### 10.4 渲染优化

```javascript
// 1. 使用 silent 选项禁用交互
series: [{
    silent: true,  // 图形不响应鼠标事件
    data: data
}]

// 2. 减少不必要的组件
option = {
    tooltip: { show: false },
    legend: { show: false },
    toolbox: { show: false }
};

// 3. 合理使用 zlevel 和 z
series: [{
    zlevel: 0,  // Canvas 分层
    z: 2,       // 组件层级
    data: data
}]

// 4. 延迟渲染
var timer = null;
function delayRender() {
    clearTimeout(timer);
    timer = setTimeout(function() {
        myChart.setOption(option);
    }, 200);
}

// 5. 使用虚拟滚动
function virtualScroll(chart, allData, viewSize) {
    var startIndex = 0;
    
    function updateView() {
        var viewData = allData.slice(startIndex, startIndex + viewSize);
        chart.setOption({
            xAxis: {
                data: viewData.map(d => d.name)
            },
            series: [{
                data: viewData.map(d => d.value)
            }]
        });
    }
    
    // 监听滚动事件
    container.addEventListener('scroll', function() {
        startIndex = Math.floor(container.scrollTop / itemHeight);
        updateView();
    });
    
    updateView();
}
```

------

## 11. 实战案例

### 11.1 实时监控仪表板

```javascript
// Spring Boot Controller
@RestController
@RequestMapping("/api/monitor")
public class MonitorController {
    
    @GetMapping(value = "/metrics", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<ServerSentEvent<MetricsData>> streamMetrics() {
        return Flux.interval(Duration.ofSeconds(1))
            .map(sequence -> {
                MetricsData metrics = new MetricsData();
                metrics.setCpu(Math.random() * 100);
                metrics.setMemory(Math.random() * 100);
                metrics.setNetwork(Math.random() * 1000);
                metrics.setTimestamp(new Date());
                
                return ServerSentEvent.<MetricsData>builder()
                    .id(String.valueOf(sequence))
                    .event("metrics")
                    .data(metrics)
                    .build();
            });
    }
}

// 前端实时图表
function createRealtimeMonitor() {
    var chart = echarts.init(document.getElementById('monitor'));
    
    var data = {
        cpu: [],
        memory: [],
        network: [],
        time: []
    };
    
    var option = {
        title: { text: '系统监控' },
        tooltip: {
            trigger: 'axis',
            axisPointer: {
                type: 'cross'
            }
        },
        legend: {
            data: ['CPU', '内存', '网络']
        },
        xAxis: {
            type: 'category',
            data: data.time
        },
        yAxis: [{
            type: 'value',
            name: '使用率(%)',
            position: 'left'
        }, {
            type: 'value',
            name: '流量(KB/s)',
            position: 'right'
        }],
        series: [{
            name: 'CPU',
            type: 'line',
            smooth: true,
            data: data.cpu
        }, {
            name: '内存',
            type: 'line',
            smooth: true,
            data: data.memory
        }, {
            name: '网络',
            type: 'line',
            smooth: true,
            yAxisIndex: 1,
            data: data.network
        }]
    };
    
    chart.setOption(option);
    
    // 连接SSE
    var eventSource = new EventSource('/api/monitor/metrics');
    
    eventSource.addEventListener('metrics', function(e) {
        var metrics = JSON.parse(e.data);
        
        // 保持最近60个数据点
        if (data.time.length >= 60) {
            data.cpu.shift();
            data.memory.shift();
            data.network.shift();
            data.time.shift();
        }
        
        data.cpu.push(metrics.cpu.toFixed(2));
        data.memory.push(metrics.memory.toFixed(2));
        data.network.push(metrics.network.toFixed(2));
        data.time.push(new Date(metrics.timestamp).toLocaleTimeString());
        
        chart.setOption({
            xAxis: { data: data.time },
            series: [
                { data: data.cpu },
                { data: data.memory },
                { data: data.network }
            ]
        });
    });
    
    return { chart, eventSource };
}
```

### 11.2 数据钻取功能

```javascript
function createDrillDownChart() {
    var chart = echarts.init(document.getElementById('drilldown'));
    
    // 层级数据
    var levelData = {
        country: [
            {name: '中国', value: 1000},
            {name: '美国', value: 800},
            {name: '日本', value: 600}
        ],
        province: {
            '中国': [
                {name: '广东', value: 400},
                {name: '北京', value: 300},
                {name: '上海', value: 300}
            ],
            '美国': [
                {name: '加州', value: 300},
                {name: '纽约', value: 250},
                {name: '德州', value: 250}
            ],
            '日本': [
                {name: '东京', value: 300},
                {name: '大阪', value: 200},
                {name: '京都', value: 100}
            ]
        },
        city: {
            '广东': [
                {name: '广州', value: 200},
                {name: '深圳', value: 150},
                {name: '东莞', value: 50}
            ]
            // ... 更多城市数据
        }
    };
    
    var currentLevel = 'country';
    var breadcrumb = [];
    
    function renderChart(data, title) {
        chart.setOption({
            title: {
                text: title,
                left: 'center'
            },
            tooltip: {},
            series: [{
                type: 'pie',
                radius: ['40%', '70%'],
                center: ['50%', '60%'],
                data: data,
                label: {
                    formatter: '{b}: {c} ({d}%)'
                }
            }],
            graphic: [{
                type: 'text',
                left: 'center',
                top: '45%',
                style: {
                    text: breadcrumb.length > 0 ? '点击返回' : '',
                    fill: '#999',
                    fontSize: 14
                },
                onclick: function() {
                    if (breadcrumb.length > 0) {
                        drillUp();
                    }
                }
            }]
        }, true);
    }
    
    function drillDown(name) {
        if (currentLevel === 'country' && levelData.province[name]) {
            breadcrumb.push({level: 'country', name: name});
            currentLevel = 'province';
            renderChart(levelData.province[name], name);
        } else if (currentLevel === 'province' && levelData.city[name]) {
            breadcrumb.push({level: 'province', name: name});
            currentLevel = 'city';
            renderChart(levelData.city[name], name);
        }
    }
    
    function drillUp() {
        if (breadcrumb.length > 0) {
            var last = breadcrumb.pop();
            if (breadcrumb.length === 0) {
                currentLevel = 'country';
                renderChart(levelData.country, '国家销售数据');
            } else {
                var parent = breadcrumb[breadcrumb.length - 1];
                currentLevel = parent.level === 'country' ? 'province' : 'city';
                renderChart(
                    levelData[currentLevel][parent.name],
                    parent.name
                );
            }
        }
    }
    
    // 初始渲染
    renderChart(levelData.country, '国家销售数据');
    
    // 绑定点击事件
    chart.on('click', function(params) {
        drillDown(params.name);
    });
    
    return chart;
}
```

### 11.3 联动图表

```javascript
function createLinkedCharts() {
    var chart1 = echarts.init(document.getElementById('chart1'));
    var chart2 = echarts.init(document.getElementById('chart2'));
    
    // 共享数据
    var sharedData = [
        {month: '1月', sales: 100, profit: 20},
        {month: '2月', sales: 120, profit: 30},
        {month: '3月', sales: 90, profit: 15},
        {month: '4月', sales: 150, profit: 40},
        {month: '5月', sales: 130, profit: 35},
        {month: '6月', sales: 160, profit: 45}
    ];
    
    // 图表1 - 柱状图
    chart1.setOption({
        title: { text: '销售额' },
        xAxis: {
            type: 'category',
            data: sharedData.map(d => d.month)
        },
        yAxis: { type: 'value' },
        series: [{
            type: 'bar',
            data: sharedData.map(d => d.sales),
            itemStyle: {
                color: function(params) {
                    return params.dataIndex === highlightIndex ? 
                           '#ff7875' : '#5470c6';
                }
            }
        }]
    });
    
    // 图表2 - 折线图
    chart2.setOption({
        title: { text: '利润率' },
        xAxis: {
            type: 'category',
            data: sharedData.map(d => d.month)
        },
        yAxis: { 
            type: 'value',
            axisLabel: {
                formatter: '{value}%'
            }
        },
        series: [{
            type: 'line',
            data: sharedData.map(d => (d.profit / d.sales * 100).toFixed(2)),
            smooth: true,
            itemStyle: {
                color: function(params) {
                    return params.dataIndex === highlightIndex ? 
                           '#ff7875' : '#91cc75';
                }
            }
        }]
    });
    
    var highlightIndex = -1;
    
    // 联动高亮
    function highlight(index) {
        highlightIndex = index;
        
        // 更新两个图表
        chart1.setOption({
            series: [{
                itemStyle: {
                    color: function(params) {
                        return params.dataIndex === highlightIndex ? 
                               '#ff7875' : '#5470c6';
                    }
                }
            }]
        });
        
        chart2.setOption({
            series: [{
                itemStyle: {
                    color: function(params) {
                        return params.dataIndex === highlightIndex ? 
                               '#ff7875' : '#91cc75';
                    }
                }
            }]
        });
    }
    
    // 绑定事件
    chart1.on('mouseover', function(params) {
        if (params.componentType === 'series') {
            highlight(params.dataIndex);
        }
    });
    
    chart2.on('mouseover', function(params) {
        if (params.componentType === 'series') {
            highlight(params.dataIndex);
        }
    });
    
    // 鼠标移出时取消高亮
    chart1.on('mouseout', function() {
        highlight(-1);
    });
    
    chart2.on('mouseout', function() {
        highlight(-1);
    });
    
    // 使用 connect 实现更高级的联动
    echarts.connect([chart1, chart2]);
    
    return { chart1, chart2 };
}
```

------

## 12. 常见问题与解决方案

### 12.1 图表不显示问题

```javascript
// 问题1：容器没有宽高
// 解决方案：确保容器有明确的宽高
<div id="chart" style="width: 600px; height: 400px;"></div>

// 问题2：在隐藏的容器中初始化
// 解决方案：显示后重新调整大小
$('#myTab').on('shown.bs.tab', function() {
    myChart.resize();
});

// 问题3：动态容器大小
// 解决方案：监听窗口变化
window.addEventListener('resize', function() {
    myChart.resize();
});

// 问题4：Vue/React中的生命周期问题
// 解决方案：在正确的生命周期初始化
// Vue
mounted() {
    this.$nextTick(() => {
        this.initChart();
    });
}

// React
useEffect(() => {
    const chart = echarts.init(chartRef.current);
    return () => {
        chart.dispose();
    };
}, []);
```

### 12.2 数据更新问题

```javascript
// 问题：数据更新后图表没有变化
// 解决方案1：使用 setOption 更新
myChart.setOption({
    series: [{
        data: newData
    }]
});

// 解决方案2：完全替换配置
myChart.setOption(newOption, true);

// 解决方案3：清空后重新设置
myChart.clear();
myChart.setOption(newOption);

// 问题：异步数据加载
// 解决方案：使用 Promise 或 async/await
async function loadChartData() {
    myChart.showLoading();
    try {
        const response = await fetch('/api/data');
        const data = await response.json();
        myChart.setOption({
            series: [{
                data: data
            }]
        });
    } finally {
        myChart.hideLoading();
    }
}
```

### 12.3 性能问题

```javascript
// 问题：大数据量渲染慢
// 解决方案1：使用采样
series: [{
    type: 'line',
    sampling: 'lttb',
    data: largeData
}]

// 解决方案2：使用大数据模式
series: [{
    type: 'scatter',
    large: true,
    data: largeData
}]

// 解决方案3：分批加载
function loadDataInBatches(data, batchSize = 1000) {
    let index = 0;
    
    function loadNextBatch() {
        const batch = data.slice(index, index + batchSize);
        myChart.appendData({
            seriesIndex: 0,
            data: batch
        });
        
        index += batchSize;
        if (index < data.length) {
            requestAnimationFrame(loadNextBatch);
        }
    }
    
    loadNextBatch();
}
```

### 12.4 样式问题

```javascript
// 问题：文字重叠
// 解决方案1：调整标签位置
label: {
    position: 'top',
    rotate: 45
}

// 解决方案2：使用 formatter 简化显示
label: {
    formatter: function(params) {
        return params.value > 100 ? params.name : '';
    }
}

// 问题：图例过多
// 解决方案：使用滚动图例
legend: {
    type: 'scroll',
    orient: 'vertical',
    right: 10,
    top: 20,
    bottom: 20
}
```

### 12.5 兼容性问题

```javascript
// 问题：IE浏览器兼容
// 解决方案：使用polyfill
<script src="https://cdn.polyfill.io/v3/polyfill.min.js"></script>

// 问题：移动端适配
// 解决方案：响应式配置
var option = {
    // 使用相对单位
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true
    },
    
    // 移动端优化
    ...(isMobile ? {
        legend: {
            orient: 'horizontal',
            bottom: 0
        },
        toolbox: {
            show: false
        }
    } : {})
};
```

### 12.6 导出问题

```javascript
// 问题：导出图片质量低
// 解决方案：设置像素比
var url = myChart.getDataURL({
    pixelRatio: 2,  // 提高分辨率
    backgroundColor: '#fff'
});

// 问题：导出包含动态数据
// 解决方案：导出前确保数据加载完成
myChart.on('finished', function() {
    var url = myChart.getDataURL();
    // 处理导出
});

// 导出为Canvas
var canvas = myChart.getDom().querySelector('canvas');
var context = canvas.getContext('2d');
// 使用 canvas 进行进一步处理

// 导出SVG（需要使用SVG渲染器）
var svgChart = echarts.init(dom, null, {
    renderer: 'svg'
});
```

------

## 总结

Apache ECharts 是一个功能强大、高度可定制的数据可视化库。本笔记涵盖了：

1. **基础知识**：环境搭建、核心概念、配置架构
2. **Spring Boot 集成**：前后端完整实现方案
3. **图表类型**：各种图表的详细配置和使用场景
4. **高级特性**：数据处理、交互功能、主题定制
5. **性能优化**：大数据处理、渲染优化、内存管理
6. **实战案例**：实时监控、数据钻取、图表联动
7. **问题解决**：常见问题及解决方案

### 最佳实践建议

1. **按需引入**：使用模块化引入减小打包体积
2. **数据分离**：将数据与配置分离，便于维护
3. **响应式设计**：监听容器变化，适配不同设备
4. **性能优先**：合理使用采样、大数据模式等优化手段
5. **用户体验**：添加加载动画、错误处理、交互提示
6. **代码复用**：封装通用配置和工具函数
7. **版本管理**：锁定版本，避免升级带来的兼容问题

### 学习资源

- [官方文档](https://echarts.apache.org/zh/index.html)
- [官方示例](https://echarts.apache.org/examples/zh/index.html)
- [API 文档](https://echarts.apache.org/zh/api.html)
- [GitHub 仓库](https://github.com/apache/echarts)
- [在线编辑器](https://echarts.apache.org/examples/zh/editor.html)
- [社区讨论](https://github.com/apache/echarts/discussions)

继续深入学习 ECharts，你可以：

- 探索更多图表类型和组合
- 研究源码了解实现原理
- 开发自定义系列和组件
- 结合其他库（如 D3.js）扩展功能
- 参与开源社区贡献代码