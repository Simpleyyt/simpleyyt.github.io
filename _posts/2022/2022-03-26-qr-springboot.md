---
layout: post
categories: Java
title: Java 代码基于开源组件生成带头像的二维码，推荐收藏！
tagline: by 子悠
tags: 
  - 子悠
---

二维码在我们目前的生活工作中，随处可见，日常开发中难免会遇到需要生成二维码的场景，网上也有很多开源的平台可以使用，不过这里我们可以通过几个开源组件，自己来实现一下。

<!--more-->

在动手之前我们先思考一下需要进行的操作，首先我们需要生成一个二维码，其次我们需要在这里二维码中间添加一个头像。

这里我们生成二维码使用工具 `zxing`，合成图片我们采用 `thumbnailator`，接下来我们实操一下吧。

### 生成二维码

首先我们先根据目标地址，生成一个二维码，这里我们使用的是组件 `zxing`，在 `SpringBoot` 的`pom`依赖中，我们加入下面的依赖。

```xml
<dependency>
  <groupId>com.google.zxing</groupId>
  <artifactId>core</artifactId>
  <version>3.4.0</version>
</dependency>
<dependency>
  <groupId>com.google.zxing</groupId>
  <artifactId>javase</artifactId>
  <version>3.4.0</version>
</dependency>
```

然后编写工具类 `QRCodeGenerator.java`

```java
import com.google.zxing.BarcodeFormat;
import com.google.zxing.EncodeHintType;
import com.google.zxing.MultiFormatWriter;
import com.google.zxing.WriterException;
import com.google.zxing.client.j2se.MatrixToImageWriter;
import com.google.zxing.common.BitMatrix;

import java.io.IOException;
import java.nio.file.FileSystems;
import java.nio.file.Path;
import java.util.HashMap;


public class QRCodeGenerator {

  /**
   * 根据内容，大小生成二维码到指定路径
   *
   * @param contents 跳转的链接
   * @param width    宽度
   * @param height   高度
   * @param filePath 路径
   * @throws WriterException
   * @throws IOException
   */
  public static void generateQrWithImage(String contents, int width, int height, String filePath) throws WriterException, IOException {
    //构造二维码写码器
    MultiFormatWriter mutiWriter = new com.google.zxing.MultiFormatWriter();
    HashMap<EncodeHintType, Object> hint = new HashMap<>(16);
    hint.put(EncodeHintType.CHARACTER_SET, "UTF-8");
    hint.put(EncodeHintType.ERROR_CORRECTION, com.google.zxing.qrcode.decoder.ErrorCorrectionLevel.H);
    hint.put(EncodeHintType.MARGIN, 1);
    //生成二维码
    BitMatrix bitMatrix = mutiWriter.encode(contents, BarcodeFormat.QR_CODE, width, height, hint);
    Path path = FileSystems.getDefault().getPath(filePath);
    MatrixToImageWriter.writeToPath(bitMatrix, "jpeg", path);
  }
}

```

这个静态方法有四个参数，分别是：

1. 跳转的链接内容；
2. 二维码的宽度；
3. 二维码的高度；
4. 生成的二维码后的存放路径；

代码中还有几个常量，`EncodeHintType.CHARACTER_SET`：表示编码；`EncodeHintType.ERROR_CORRECTION` 表示二维码的容错率；`EncodeHintType.MARGIN` 表示二维码的边框。

解释一下什么是二维码的容错率，大家在日常生活或者工作中应该会发现，有些二维码轻轻一扫就扫成功了，有的二维码却很难扫成功，这背后就是二维码的容错率的原因（对，有时候并不是你的网络问题！）。

> 不同密度的二维码所包含的信息其编码的字符、容错率均不同。密度越低，编码的字符个数越少、容错率越低，二维码容错率表示二维码图标被遮挡多少后，仍可以被扫描出来的能力。目前，典型的二维码的容错率分为 7%、15%、25%、30% 四个等级，容错率越高，越容易被快速扫描。“但是，容错率越高，二维码里面的黑白格子也就越多。因此，对于目前主流手机，在绝大多数扫描场景下，仍普遍应用 7% 容错率的二维码就能满足需求。

感兴趣的小伙伴也可以自己尝试几个不同的容错率，看看扫码的难度有没有变化。

接下来，我们编写一个 main 方法来生成一个二维码

```java
public static void main(String[] args) {
    try {
      QRCodeGenerator.generateQrWithImage("https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=MzkzODE3OTI0Ng==&scene=124#wechat_redirect", 500, 500, "./QRCode.jpeg");
    } catch (Exception e) {
    }
  }
```

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h1bkkm875pj20dw0dwdhj.jpg)

运行完上面的 `main` 方法，我们可以得到一个二维码，到这里我们第一步已经完成了，接下来就是给这个二维码加上我们的头像。

### 添加头像

添加头像我们需要准备一个头像的照片，阿粉这里就用阿粉的头像了，如果这里有现成大小的头像就直接拿来使用就行，如果没有也没有关系，我们可以自己裁剪，这里我们就需要用来图片处理工具 `thumbnailator` 了，

![阿粉](https://tva1.sinaimg.cn/large/e6c9d24egy1h1bkkq0x2wj20j60j60uj.jpg)

先将头像处理成 100 x 100 的大小，然后再合成上去就行，代码如下：

```java

  public static void main(String[] args) {
    try {
      // 将大图片缩小到指定大小
//    ThumbnailsImageUtils.size("./阿粉.jpeg", 100, 100, 1, "./阿粉100.png");
      // 通过水印的形式，将头像加到生成的二维码上面
      ThumbnailsImageUtils.watermark("./QRCode.jpeg",
      500, 500, Positions.CENTER, "./阿粉100.png",
        1f, 1f, "./result.png");
    } catch (Exception e) {
    }
  }
```

![result](https://tva1.sinaimg.cn/large/e6c9d24egy1h1bkkwhdbjj20dw0dwtap.jpg)

这个就是最终生成的带头像的二维码啦，友情提示，扫码可以看到更多优质文章。

通过上面的代码可以看到，`ThumbnailsImageUtils` 是真的很多强大，一行代码就能搞定图片缩小和图片合成，更多关于 `ThumbnailsImageUtils` 工具类的完整代码如下，赶紧收藏起来。

```java

import net.coobird.thumbnailator.Thumbnails;
import net.coobird.thumbnailator.geometry.Positions;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.OutputStream;

/**
 * <br>
 * <b>功能：</b><br>
 * <b>作者：</b>@author ziyou<br>
 * <b>日期：</b>2018-05-25 16:17<br>
 * <b>详细说明：</b>使用google开源工具Thumbnailator实现图片的一系列处理<br>
 */
public class ThumbnailsImageUtils {

    private final static Logger logger = LoggerFactory.getLogger(ThumbnailsImageUtils.class);

    /**
     * 将原图根据指定大小生产新图
     *
     * @param sourceFilePath 原始图片路径
     * @param width          指定图片宽度
     * @param height         指定图片高度
     * @param targetFilePath 目标图片路径
     * @return 目标图片路径
     * @throws IOException
     */
    public static String thumb(String sourceFilePath, Integer width, Integer height, String targetFilePath) throws IOException {
        Thumbnails.of(sourceFilePath)
                .forceSize(width, height)
                .toFile(targetFilePath);
        return targetFilePath;
    }


    /**
     * 按照比例进行缩放
     *
     * @param sourceFilePath 原始图片路径
     * @param scale          scale(比例)
     * @param targetFilePath 目标图片路径
     * @return 目标图片路径
     * @throws IOException
     */
    public static String scale(String sourceFilePath, Double scale, String targetFilePath) throws IOException {
        Thumbnails.of(sourceFilePath)
                .scale(scale)
                .toFile(targetFilePath);
        return targetFilePath;
    }


    /**
     * 不按照比例，指定大小进行缩放
     *
     * @param sourceFilePath 原始图片路径
     * @param width          指定图片宽度
     * @param height         指定图片高度
     * @param targetFilePath 目标图片路径
     *                       keepAspectRatio(false) 默认是按照比例缩放的
     * @return 目标图片路径
     * @throws IOException
     */
    public static String size(String sourceFilePath, Integer width, Integer height, float quality, String targetFilePath) throws IOException {
        Thumbnails.of(sourceFilePath)
                .size(width, height)
                .outputQuality(quality)
                .keepAspectRatio(false)
                .toFile(targetFilePath);
        return targetFilePath;
    }


    /**
     * 指定大小和角度旋转
     *
     * @param sourceFilePath 原始图片路径
     * @param width          指定图片宽度
     * @param height         指定图片高度
     * @param rotate         rotate(角度),正数：顺时针 负数：逆时针
     * @param targetFilePath 目标图片路径
     * @return 目标图片路径
     * @throws IOException
     */
    public static String rotate(String sourceFilePath, Integer width, Integer height, Double rotate, String targetFilePath) throws IOException {
        Thumbnails.of(sourceFilePath)
                .size(width, height)
                .rotate(rotate)
                .toFile(targetFilePath);
        return targetFilePath;
    }


    /**
     * 指定角度旋转
     *
     * @param sourceFilePath 原始图片路径
     * @param rotate         rotate(角度),正数：顺时针 负数：逆时针
     * @param targetFilePath 目标图片路径
     * @return 目标图片路径
     * @throws IOException
     */
    public static String rotate(String sourceFilePath, Double rotate, String targetFilePath) throws IOException {
        Thumbnails.of(sourceFilePath)
                .rotate(rotate)
                .toFile(targetFilePath);
        return targetFilePath;
    }


    /**
     * @param sourceFilePath 原始图片路径
     * @param width          指定图片宽度
     * @param height         指定图片高度
     * @param position       水印位置
     * @param waterFile      水印文件
     * @param opacity        水印透明度
     * @param quality        输出文件的质量
     * @param targetFilePath 目标图片路径
     * @return 目标图片路径
     * @throws IOException
     */
    public static String watermark(String sourceFilePath, Integer width, Integer height, Positions position, String waterFile, float opacity, float quality, String targetFilePath) throws IOException {
        Thumbnails.of(sourceFilePath)
                .size(width, height)
                .watermark(position, ImageIO.read(new File(waterFile)), opacity)
                .outputQuality(quality)
                .toFile(targetFilePath);
        return targetFilePath;
    }


    public static BufferedImage watermarkList(BufferedImage buffImg, int length, File[] waterFileArray) throws IOException {
        int x = 0;
        int y = 0;
        if (buffImg == null) {
            // 获取底图
            buffImg = new BufferedImage(1200, 1200, BufferedImage.SCALE_SMOOTH);
        } else {
            x = (length % 30) * 40;
            y = (length / 30) * 40;
        }
        // 创建Graphics2D对象，用在底图对象上绘图
        Graphics2D g2d = buffImg.createGraphics();
        // 将图像填充为白色
        g2d.setColor(Color.WHITE);
        g2d.fillRect(x, y, 1200, 40 * (waterFileArray.length + length));
        for (int i = 0; i < waterFileArray.length; i++) {
            // 获取层图
            BufferedImage waterImg = ImageIO.read(waterFileArray[i]);
            // 获取层图的宽度
            int waterImgWidth = waterImg.getWidth();
            // 获取层图的高度
            int waterImgHeight = waterImg.getHeight();
            // 在图形和图像中实现混合和透明效果
            g2d.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_ATOP, 1));
            // 绘制
            Integer j = i / 30;
            Integer index = i - j * 30;
            g2d.drawImage(waterImg, waterImgWidth * index + 1, waterImgHeight * j, waterImgWidth, waterImgHeight, null);

        }

        // 释放图形上下文使用的系统资源
        g2d.dispose();
        return buffImg;
    }

    /**
     * @param sourceFilePath 原始图片路径
     * @param waterFile      水印文件
     * @param targetFilePath 目标图片路径
     *                       透明度默认值0.5f，质量默认0.8f
     * @return 目标图片路径
     * @throws IOException
     */
    public static String watermark(String sourceFilePath, String waterFile, String targetFilePath) throws IOException {
        Image image = ImageIO.read(new File(waterFile));
        Integer width = image.getWidth(null);
        Integer height = image.getHeight(null);
        return watermark(sourceFilePath, width, height,
                Positions.BOTTOM_RIGHT, waterFile, 0.5f, 0.8f, targetFilePath);
    }


    /**
     * 将图片转化为指定大小和格式的图片
     *
     * @param sourceFilePath
     * @param width
     * @param height
     * @param format
     * @param targetFilePath
     * @return
     * @throws IOException
     */
    public static String changeFormat(String sourceFilePath, Integer width, Integer height, String format, String targetFilePath) throws IOException {
        Thumbnails.of(sourceFilePath)
                .size(width, height)
                .outputQuality(0.8f)
                .outputFormat(format)
                .toFile(targetFilePath);
        return targetFilePath;
    }


    /**
     * 根据原大小转化指定格式
     *
     * @param sourceFilePath
     * @param format
     * @param targetFilePath
     * @return
     * @throws IOException
     */
    public static String changeFormat(String sourceFilePath, String format, String targetFilePath) throws IOException {
        Image image = ImageIO.read(new File(sourceFilePath));
        Integer width = image.getWidth(null);
        Integer height = image.getHeight(null);
        Thumbnails.of(sourceFilePath)
                .size(width, height)
                .outputFormat(format)
                .toFile(targetFilePath);
        return targetFilePath;
    }

    /**
     * 输出到输出流
     *
     * @param sourceFilePath
     * @param targetFilePath
     * @return
     * @throws IOException
     */
    public static String toOutputStream(String sourceFilePath, String targetFilePath) throws IOException {
        OutputStream os = new FileOutputStream(targetFilePath);
        Thumbnails.of(sourceFilePath).toOutputStream(os);
        return targetFilePath;
    }


    /**
     * 输出到BufferedImage
     *
     * @param sourceFilePath
     * @param format
     * @param targetFilePath
     * @return
     * @throws IOException
     */
    public static String asBufferedImage(String sourceFilePath, String format, String targetFilePath) throws IOException {
        /**
         * asBufferedImage() 返回BufferedImage
         */
        BufferedImage thumbnail = Thumbnails.of(sourceFilePath).size(1280, 1024).asBufferedImage();
        ImageIO.write(thumbnail, format, new File(targetFilePath));
        return targetFilePath;
    }
}
```

### 总结

今天阿粉通过两个组件带大家实践了一个二维码的生成和图片的合成，这两个技巧在工作中难免会使用到，赶紧保存使用起来吧。