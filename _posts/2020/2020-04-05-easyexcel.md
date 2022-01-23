---
layout: post
categories: Java
title: 我去，还在这样读写 excel 这也太低效了吧！
tagline: by 小黑
published: false
tags: 
  - 小黑
---

## 前言

最近读者小 H 在知识星球中给阿粉发来私信：

> 阿粉，最近我在负责公司报表平台开发，需要导出报表到 excel 中。每次使用 POI 开发，都要写长长的一坨代码，好几次因为没加入判空判断，导致生成失败。想跟你请教下有没有更加高效一点读写 excel 方法？

> ps：知识星球汇集一片大神，感兴趣的同学可以加入知识星球，有任何问题都会有大神及时解答。

<!--more-->

使用过 poi 的开发同学可能都有此体会，每次都要写一坨代码，最后的代码如下面一样:

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200405/00831rSTly1gdioo76dncj30u00u314d.jpg)

这样的代码是不是又臭又长？当字段数量多的时候，一不小心还容易写错。阿粉还记得当初使用 poi 导出一个二十多字段的 excel，不断复制粘贴，行号一不小心就写错了，那叫个一个心酸。

今天阿粉就来推荐一个阿里开源的项目『**EasyExcel**』，带大家彻底告别上面又长又臭的代码，彻底解决这个问题。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200405/00831rSTly1gdiwtehnvbj308c081q34.jpg)

## EasyExcel

**EasyExcel** 是一个阿里出品的开源项目 ，看名字就能看出这个项目是为了让你更加简单的操作 Excel。另外 **EasyExcel** 还解决了poi 内存溢出问题，修复了一些并发情况下一些 bug。

github 地址：https://github.com/alibaba/easyexcel

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200405/00831rSTly1gdiotluunjj31o20bgq5c.jpg)

截止阿粉写文章时，已有 **13.6k** star 数据，可见这个项目还是深受大家欢迎。

废话不多说，我们直接进入源码实战环节。

首先我们需要引入 **EasyExcel** pom 依赖：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>2.1.6</version>
</dependency>
```

这里建议大家使用 2.0 以上的正式版本，不要再使用 1.0 的老版本，两者使用 API 差别很大。另外 **beta** 版本可能会存在某些 bug，大家谨慎使用。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200405/00831rSTly1gdip3rw6y9j322o082769.jpg)

### 普通方式

**一行代码生成 Excel**

```java
// 写法1
String fileName = "temp/" + "test" + System.currentTimeMillis() + ".xlsx";
EasyExcel.write(fileName)
        .head(head())// 设置表头
        .sheet("模板")// 设置 sheet 的名字
        // 自适应列宽
        .registerWriteHandler(new LongestMatchColumnWidthStyleStrategy())
        .doWrite(dataList());// 写入数据

```

生成 excel 代码特别简单，这里使用链式语句，一行代码直接搞定生成代码。代码中再也不用我们指定行号，列号了。

> 上面代码中使用自适应列宽的策略。

下面我们来看下表头与标题如何生成。

**创建表头**

```java
/**
 * 创建表头，可以创建复杂的表头
 *
 * @return
 */
private static List<List<String>> head() {
    List<List<String>> list = new ArrayList<List<String>>();
    // 第一列表头
    List<String> head0 = new ArrayList<String>();
    head0.add("第一列");
    head0.add("第一列第二行");
    // 第二列表头
    List<String> head1 = new ArrayList<String>();
    head1.add("第一列");
    head1.add("第二列第二行");
    // 第三列
    List<String> head2 = new ArrayList<String>();
    head2.add("第一列");
    head2.add("第三列第二行");
    list.add(head0);
    list.add(head1);
    list.add(head2);
    return list;
}
```

上面每个 `List<String>` 代表**一列**的数据，集合内每个数据将会顺序写入这列每一行。如果每一列的相同行数的内容相同，将会自动合并单元格。通过这个规则，我们创建复杂的表头。

最终创建表头如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200405/00831rSTly1gdiq5yk82cj30my04g0sx.jpg)

**写入表体数据**

```java
private static List dataList() {
    List<List<Object>> list = new ArrayList<List<Object>>();
    for (int i = 0; i < 10; i++) {
        List<Object> data = new ArrayList<Object>();
        data.add("点赞+" + i);
        // date 将会安装 yyyy-MM-dd HH:mm:ss 格式化
        data.add(new Date());
        data.add(0.56);
        list.add(data);
    }
    return list;
}
```

表体数据然后也是使用 `List<List<Object>>`，但是与表头规则不一样。

每个 `List<Object>` 代表**一行**的数据，数据将会按照顺序写入每一列中。

集合中数据 **EasyExcel** 将会按照默认的格式化转换输出，比如 `date` 类型数据就将会按照  `yyyy-MM-dd HH:mm:ss` 格式化。

如果需要转化成其他格式，建议直接将数据格式化成字符串加入 List，不要通过 **EasyExcel** 转换。

最终效果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200405/00831rSTly1gdiq6aggk9j30mu0gagno.jpg)

看完这个是不是想立刻体验一下？等等，上面使用方式还是有点繁琐，使用 **EasyExcel** 还可以更快。我们可以使用注解方式，无需手动设置表头与表体。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200405/00831rSTly1gdixgwna2vj308c07ht8o.jpg)

### 注解方式

注解方式生成 Excel 代码如下：

```java
String fileName = "temp/annotateWrite" + System.currentTimeMillis() + ".xlsx";
// 这里 需要指定写用哪个class去写，然后写到第一个sheet，名字为模板 然后文件流会自动关闭
// 如果这里想使用03 则 传入excelType参数即可
EasyExcel
        .write(fileName, DemoData.class)
        .sheet("注解方式")
        .registerWriteHandler(createTableStyle())// Excel 表格样式
        .doWrite(data());
```

这里代码与上面大体一致，只不过这里需要在 `write` 方法传入 `DemoData` 数据类型。**EasyExcel** 会根据 `DemoData` 类型自动生成表头。

下面我们来看下 `DemoData`这个类到底内部到底是啥样？

```java
@ContentRowHeight(30)// 表体行高
@HeadRowHeight(20)// 表头行高
@ColumnWidth(35)// 列宽
@Data
public class DemoData {
    /**
     * 单独设置该列宽度
     */
    @ColumnWidth(50)
    @ExcelProperty("字符串标题")
    private String string;
    /**
     * 年月日时分秒格式
     */
    @DateTimeFormat("yyyy年MM月dd日HH时mm分ss秒")
    @ExcelProperty(value = "日期标题")
    private Date date;
    /**
     * 格式化百分比
     */
    @NumberFormat("#.##%")
    @ExcelProperty("数字标题")
    private Double doubleData;
    @ExcelProperty(value = "枚举类",converter = DemoEnumConvert.class)
    private DemoEnum demoEnum;
    /**
     * 忽略这个字段
     */
    @ExcelIgnore
    private String ignore;
}
```

`DemoData` 就是一个普通的 `POJO` 类，上面使用 **ExayExcel** 相关注解，**ExayExcel** 将会通过反射读取字段类型以及相关注解，然后直接生成 Excel 。

ExayExcel 提供相关注解类，直接定义 Excel 的数据模型：

- `@ExcelProperty` 指定当前字段对应excel中的那一列，内部 value 属性指定表头列的名称
- `@ExcelIgnore` 默认所有字段都会和excel去匹配，加了这个注解会忽略该字段
- `@ContentRowHeight` 指定表体行高
- `@HeadRowHeight` 指定表头行高
- `@ColumnWidth` 指定列的宽度

另外 ExayExcel 还提供几个注解，自定义日期以及数字的格式化转化。

- `@DateTimeFormat`
- `@NumberFormat`

另外我们可以自定义格式化转换方案，需要实现 `Converter` 类相关方法即可。

```java
public class DemoEnumConvert implements Converter<DemoEnum> {
    @Override
    public Class supportJavaTypeKey() {
        return DemoEnum.class;
    }

    @Override
    public CellDataTypeEnum supportExcelTypeKey() {
        return CellDataTypeEnum.STRING;
    }

    /**
     * excel 转化为 java 类型，excel 读时将会被调用
     * @param cellData
     * @param contentProperty
     * @param globalConfiguration
     * @return
     * @throws Exception
     */
    @Override
    public DemoEnum convertToJavaData(CellData cellData, ExcelContentProperty contentProperty, GlobalConfiguration globalConfiguration) throws Exception {
        return null;
    }

    /**
     * java 类型转 excel 类型，excel 写时将会被调用
     * @param value
     * @param contentProperty
     * @param globalConfiguration
     * @return
     * @throws Exception
     */
    @Override
    public CellData convertToExcelData(DemoEnum value, ExcelContentProperty contentProperty, GlobalConfiguration globalConfiguration) throws Exception {
        return new CellData(value.getDesc());
    }
}
```

最后我们还需要在 `@ExcelProperty` 注解上使用 `converter` 指定自定义格式转换方案。

使用方式如下：

```java
@ExcelProperty(value = "枚举类",converter = DemoEnumConvert.class)
private DemoEnum demoEnum;
```

最后我们运行一下，看下 Excel 实际效果如何：

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200405/00831rSTly1gdiv6h69k9j31la0o07b8.jpg)

怎么样，效果还是可以吧。

![](http://www.justdojava.com/assets/images/2019/java/image_andyxh/20200405/00831rSTly1gdixmfngpnj3073073jri.jpg)

对了,默认的样式表格样式可不是这样，这个效果是因为我们在 `registerWriteHandler` 方法中设置自定义的样式，具体代码如下：

```java
/***
 * 设置 excel 的样式
 * @return
 */
private static WriteHandler createTableStyle() {
    // 头的策略
    WriteCellStyle headWriteCellStyle = new WriteCellStyle();
    // 背景设置为红色
    headWriteCellStyle.setFillForegroundColor(IndexedColors.PINK.getIndex());
    // 设置字体
    WriteFont headWriteFont = new WriteFont();
    headWriteFont.setFontHeightInPoints((short) 20);
    headWriteCellStyle.setWriteFont(headWriteFont);
    // 内容的策略
    WriteCellStyle contentWriteCellStyle = new WriteCellStyle();
    // 这里需要指定 FillPatternType 为FillPatternType.SOLID_FOREGROUND 不然无法显示背景颜色.头默认了 FillPatternType所以可以不指定
    contentWriteCellStyle.setFillPatternType(FillPatternType.SOLID_FOREGROUND);
    // 背景绿色
    contentWriteCellStyle.setFillForegroundColor(IndexedColors.LEMON_CHIFFON.getIndex());

    WriteFont contentWriteFont = new WriteFont();
    // 字体大小
    contentWriteFont.setFontHeightInPoints((short) 20);
    contentWriteCellStyle.setWriteFont(contentWriteFont);
    // 设置边框的样式
    contentWriteCellStyle.setBorderBottom(BorderStyle.DASHED);
    contentWriteCellStyle.setBorderLeft(BorderStyle.DASHED);
    contentWriteCellStyle.setBorderRight(BorderStyle.DASHED);
    contentWriteCellStyle.setBorderTop(BorderStyle.DASHED);

    // 这个策略是 头是头的样式 内容是内容的样式 其他的策略可以自己实现
    HorizontalCellStyleStrategy horizontalCellStyleStrategy =
            new HorizontalCellStyleStrategy(headWriteCellStyle, contentWriteCellStyle);
    return horizontalCellStyleStrategy;
}
```

## 使用注意点

### poi 冲突问题

理论上当前 `easyexcel`兼容支持 poi 的`3.17`,`4.0.1`,`4.1.0`所有较新版本，但是如果项目之前使用较老版本的 poi，由于 poi 内部代码调整，某些类已被删除，这样直接运行时很大可能会抛出以下异常：

- `NoSuchMethodException`
- `ClassNotFoundException`
- `NoClassDefFoundError`

所以使用过程中一定要**注意统一项目中的 poi 的版本**。

### 非注解方式自定义行高列宽

非注解方式自定义行高以及列宽比较麻烦，暂时没有找到直接设置的入口。查了一遍 github 相关 issue，开发人员回复需要实现 `WriteHandler` 接口，自定义表格样式。

## 总结

本文主要给各位小伙伴们安利 EasyExcel 强大的功能，介绍 EasyExcel 两种生成 excel 方式，以及演示相关的示例代码。 **EasyExcel** 除了写之外，当然还支持快读读取 Excel 的功能，这里就不再详细介绍。Github 上相关文档例子非常丰富，大家可以自行参考。

Github 文档地址：https://alibaba-easyexcel.github.io/index.html

## Reference

1. https://github.com/alibaba/easyexcel
2. https://alibaba-easyexcel.github.io/index.html
3. https://cloud.tencent.com/developer/article/1431888

