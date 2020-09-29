---
layout: post
categories: SpringBoot
title: SpringBoot2.x 整合 thumbnailator 图片处理
tagline: by 淼淼之森
tags: 
    - 淼淼之森
---

## 1、序
在实际项目中，有时为了响应速度，难免会对一些高清图片进行一些处理，比如图片压缩之类的，而其中压缩可能就是最为常见的。最近，阿粉就被要求实现这个功能，原因是客户那边嫌速度过慢。借此机会，阿粉今儿就给大家介绍一些一下我做这个功能时使用的 `Thumbnailator` 库。

<!--more-->

 `Thumbnailator` 是一个优秀的图片处理的 Google 开源 Java 类库，专门用来生成图像缩略图的，通过很简单的 API 调用即可生成图片缩略图，也可直接对一整个目录的图片生成缩略图。两三行代码就能够从现有图片生成处理后的图片，且允许微调图片的生成方式，同时保持了需要写入的最低限度的代码量。可毫不夸张的说，它是一个处理图片十分棒的开源框架。
 
**支持**：图片缩放，区域裁剪，水印，旋转，保持比例。

`Thumbnailator` 官网：[https://code.google.com/p/thumbnailator/](https://code.google.com/p/thumbnailator/)

有了这玩意，就不用在费心思使用 Image I/O API,Java 2D API 等等来生成缩略图了。

废话少说，直接上代码，先来看一个最简单的例子：

## 2、代码示例
#### 2.1. 新建一个springboot项目
#### 2.2. 引入依赖 thumbnailator
```xml
<dependency>
	<groupId>net.coobird</groupId>
	<artifactId>thumbnailator</artifactId>
	<version>0.4.8</version>
</dependency>
```
#### 2.3. controller
主要实现了如下几个接口作为测试：
```
@RestController
public class ThumbnailsController {
	@Resource
	private IThumbnailsService thumbnailsService;

	/**
	 * 指定大小缩放
	 * @param resource
	 * @param width
	 * @param height
	 * @return
	 */
	@GetMapping("/changeSize")
	public String changeSize(MultipartFile resource, int width, int height) {
		return thumbnailsService.changeSize(resource, width, height);
	}

	/**
	 * 指定比例缩放
	 * @param resource
	 * @param scale
	 * @return
	 */
	@GetMapping("/changeScale")
	public String changeScale(MultipartFile resource, double scale) {
		return thumbnailsService.changeScale(resource, scale);
	}

	/**
	 * 添加水印 watermark(位置,水印,透明度)
	 * @param resource
	 * @param p
	 * @param shuiyin
	 * @param opacity
	 * @return
	 */
	@GetMapping("/watermark")
	public String watermark(MultipartFile resource, Positions p, MultipartFile shuiyin, float opacity) {
		return thumbnailsService.watermark(resource, Positions.CENTER, shuiyin, opacity);
	}

	/**
	 * 图片旋转 rotate(度数),顺时针旋转
	 * @param resource
	 * @param rotate
	 * @return
	 */
	@GetMapping("/rotate")
	public String rotate(MultipartFile resource, double rotate) {
		return thumbnailsService.rotate(resource, rotate);
	}

	/**
	 * 图片裁剪
	 * @param resource
	 * @param p
	 * @param width
	 * @param height
	 * @return
	 */
	@GetMapping("/region")
	public String region(MultipartFile resource, Positions p, int width, int height) {
		return thumbnailsService.region(resource, Positions.CENTER, width, height);
	}
}
```
## 3、功能实现
其实引入了这个 `Thumbnailator` 类库后，代码其实很少，因为我们只需要按照规则调用其 API 来实现即可。就个人而言，挺喜欢这种 API 的方式，简洁，易懂，明了。
### 3.1 指定大小缩放
```
/**
 * 指定大小缩放 若图片横比width小，高比height小，放大 
 * 若图片横比width小，高比height大，高缩小到height，图片比例不变
 * 若图片横比width大，高比height小，横缩小到width，图片比例不变 
 * 若图片横比width大，高比height大，图片按比例缩小，横为width或高为height
 * 
 * @param resource  源文件路径
 * @param width     宽
 * @param height    高
 * @param tofile    生成文件路径
 */
public static void changeSize(String resource, int width, int height, String tofile) {
	try {
		Thumbnails.of(resource).size(width, height).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```
测试：
![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/09-01/01.png)

### 3.2 指定比例缩放
```
/**
 * 指定比例缩放 scale(),参数小于1,缩小;大于1,放大
 * 
 * @param resource   源文件路径
 * @param scale      指定比例
 * @param tofile     生成文件路径
 */
public static void changeScale(String resource, double scale, String tofile) {
	try {
		Thumbnails.of(resource).scale(scale).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```
测试：
![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/09-01/02.png)

### 3.3 添加水印
```
/**
 * 添加水印 watermark(位置,水印,透明度)
 * 
 * @param resource  源文件路径
 * @param p         水印位置
 * @param shuiyin   水印文件路径
 * @param opacity   水印透明度
 * @param tofile    生成文件路径
 */
public static void watermark(String resource, Positions p, String shuiyin, float opacity, String tofile) {
	try {
		Thumbnails.of(resource).scale(1).watermark(p, ImageIO.read(new File(shuiyin)), opacity).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```
测试：
![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/09-01/03.png)

### 3.4 图片旋转
```
/**
 * 图片旋转 rotate(度数),顺时针旋转
 * 
 * @param resource  源文件路径
 * @param rotate    旋转度数
 * @param tofile    生成文件路径
 */
public static void rotate(String resource, double rotate, String tofile) {
	try {
		Thumbnails.of(resource).scale(1).rotate(rotate).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```
测试：
![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/09-01/04.png)

### 3.5 图片裁剪
```
/**
 * 图片裁剪 sourceRegion()有多种构造方法，示例使用的是sourceRegion(裁剪位置,宽,高)
 * 
 * @param resource  源文件路径
 * @param p         裁剪位置
 * @param width     裁剪区域宽
 * @param height    裁剪区域高
 * @param tofile    生成文件路径
 */
public static void region(String resource, Positions p, int width, int height, String tofile) {
	try {
		Thumbnails.of(resource).scale(1).sourceRegion(p, width, height).toFile(tofile);
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```
测试：
![](http://www.justdojava.com/assets/images/2019/java/image-mmzsblog/2020/09-01/05.png)


说明：
- 1.`keepAspectRatio(boolean arg0)`  图片是否按比例缩放（宽高比保持不变）默认 `true`
- 2.`outputQuality(float arg0)` 图片质量
- 3.`outputFormat(String arg0)` 格式转换

### 小结
值得注意的是，若 png、gif 格式图片中含有透明背景，使用该工具压缩处理后背景会变成黑色，这是 `Thumbnailator` 的一个 bug，预计后期版本会解决。

**代码地址** ：[https://github.com/justdojava/java-samples/tree/master/springboot_thumbnails](https://github.com/justdojava/java-samples/tree/master/springboot_thumbnails)