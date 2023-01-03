---
layout: post
categories: Java
title: 你知道什么是 @Component 注解的派生性吗？
tagline: by 子悠
tags: 
  - 子悠

---

对于 `@Component` 注解在日常的工作中相信很多小伙伴都会使用到，作为一种 `Spring` 容器托管的通用模式组件，任何被 `@Component` 注解标注的组件都会被 `Spring` 容器扫描。那么有的小伙伴就要问了，很多时候我们并没有直接写 `@Component` 注解呀，写的是类似于 `@Service`，`@RestController`，`@Configuration` 等注解，不也是一样可以被扫描到吗？那这个 `@Component` 有什么特别的吗？

<!--more-->

## 元注解

在回答上面的问题之前，我们先来了解一下什么叫元注解，所谓元注解就是指一个能声明在其他注解上的注解，换句话说就是如果一个注解被标注在其他注解上，那么它就是元注解。要说明的是这个元注解并不是 `Spring` 领域的东西， 而是 `Java` 领域的，像 `Java` 中的很多注解比如 `@Document`，`@Repeatable` ，`@Target` 等都属于元注解。

根据上面的解释我们可以发现在 `Spring` 容器里 `@Component`  就是以元注解的形式存在，因为我们可以在很多其他注解里面找到它的身影，如下所示

![Configuration](https://tva1.sinaimg.cn/large/008vxvgGgy1h96nxg715jj31140pyq5r.jpg)

![controller](https://tva1.sinaimg.cn/large/008vxvgGgy1h96nzac4bhj30z40oq0vf.jpg)

## @Component 的派生性

通过上面的内容我们是不是可以猜测一下那就是 `@Component` 注解的特性被"继承"下来了？这就可以解释为什么我们可以直接写`@Service`，`@RestController` 注解也是可以被扫描到的。但是由于 `Java` 的注解是不支持继承的，比如你想通过下面的方式来实现注解的继承是不合法的。

![@interface](https://tva1.sinaimg.cn/large/008vxvgGgy1h96oag3okpj31dq0mu0we.jpg)

为了验证我们的猜想，可以通过跟踪源代码来验证一下，我们的目的是研究为什么不直接使用 `@Component` 注解也能被 `Spring` 扫描到，换句话说就是使用 `@Service` 和 `@RestController` 的注解也能成为 `Spring Bean`。那我们很自然的就可以想到，在扫描的时候一定是根据注解来进行了判断是否要初始化成 `Spring Bean` 的。我们只要找到了判断条件就可以解决我们的疑惑了。

由于 `SpringBoot` 项目是通过 `main` 方法进行启动的，调试起来还是很方便的，阿粉这边准备了一个简单的 `SpringBoot` 工程，里面除了启动类之外只有一个`DemoController.java` 代码如下

```java
package com.example.demojar.controller;

import org.springframework.web.bind.annotation.RestController;

@RestController
public class DemoController {
}

```

启动类如下

```java
package com.example.demojar;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.WebApplicationType;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication(scanBasePackages = {"com.example.demojar"})
public class DemoJarApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoJarApplication.class, args);
	}
}

```

`Debug run` 方法，我们可以定位到 `org.springframework.boot.SpringApplication#run(java.lang.String...) `方法，该方法里面会初始化 `SpringBoot` 上下文 `context`。

```java
context = createApplicationContext();
```

默认情况下会进到下面的方法，并创建 `AnnotationConfigServletWebServerApplicationContext` 并且其构造函数中构造了 `ClassPathBeanDefinitionScanner` 类路径 Bean 扫描器。此处已经越来越接近扫描相关的内容了。

`org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext.Factory#create`

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h978l4wmkmj328y0agaeb.jpg)

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h978nblgpsj31y608w76a.jpg)

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h978nwskqlj31aa09i0uu.jpg)

`context` 上下文创建完成过后，接下来我们我们会接入到 `org.springframework.context.support.AbstractApplicationContext#refresh`，再到 `org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors` 

`org.springframework.context.annotation.ConfigurationClassPostProcessor#postProcessBeanDefinitionRegistry`

`org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions`

经过上面的步骤，最终可以可以定位到扫描的代码在下面的方法 `org.springframework.context.annotation.ComponentScanAnnotationParser#parse` 里面，调用前面上下文初始化的扫描器的 `org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan` 方法，

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h97900so9tj326o0oigss.jpg)

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h979193ojlj31kv0u0ais.jpg)

到这里我们已经定位到了扫描具体包路径的方法，这个方法里面主要看 `findCandidateComponents(basePackage);` 方法的内容，这个方法就是返回合法的候选组件。说明这个方法会最终返回需要被注册成 Spring Bean 的候选组件，那我们重点就要看这个方法的实现。

跟踪这个方法 `org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#findCandidateComponents` 进去我们可以看到通过加进类路径里面的资源文件，然后再根据资源文件生成 `MetadataReader` 对象，最后判断这个 `MetadataReader` 对象是否满足候选组件的条件，如果满足就添加到 `Set` 集合中进行返回。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h97u0f4719j31h20u0agt.jpg)

继续追踪源码我们可以找到具体的判断方法在 `org.springframework.core.type.filter.AnnotationTypeFilter#matchSelf` 方法中，如下所示，可以看到这里对 `MetadataReader` 对象进行了判断是否有元注解 `@Component`。在调试的时候我们会发现 `DemoController` 在此处会返回 `true`，并且该 `MetadataReader` 对象里面还有多个 `mappings` ，其实这些 `mappings` 对应的就是 `Spring` 的注解。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h97uhqa48vj321e0c60xc.jpg)

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h97v4pgkljj31t10u0n7h.jpg)

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h97v96qeb5j31jc0oe0xd.jpg)

这个 `mappings` 里面的注解确实包含了 `@Component` 注解，因此会返回 `true`。那么接下来问题就转换成，我们的 `DemoController` 对应的 `MetadataReader` 对象是如何创建的。

我们看回到 `org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#scanCandidateComponents` 方法，看看具体 MetadataReader 对象是如何创建的，`MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);`

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h97uwrfuh4j31l20bsgox.jpg)

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h97uxcr5u3j31w00bkn0c.jpg)

通过构造方法 `org.springframework.core.type.classreading.SimpleMetadataReader#SimpleMetadataReader` 进行 `MetadataReader`  对象的创建，`org.springframework.core.type.classreading.SimpleAnnotationMetadataReadingVisitor#visitEnd`，最终定位到 `org.springframework.core.annotation.MergedAnnotationsCollection#MergedAnnotationsCollection` 这里进行 `mappings` 赋值。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h975k232fhj323i0u0tj9.jpg)

继续定位到 `org.springframework.core.annotation.AnnotationTypeMappings.Cache#createMappings`，`org.springframework.core.annotation.AnnotationTypeMappings#addAllMappings`，`addAllmappings` 方法，内部使用了一个 `while` 循环和 `Deque` 来循环查询元注解进行赋值，代码如下所示，重点是这一行 `Annotation[] metaAnnotations = AnnotationsScanner.getDeclaredAnnotations(source.getAnnotationType(), false);`

```java
private void addAllMappings(Class<? extends Annotation> annotationType,
			Set<Class<? extends Annotation>> visitedAnnotationTypes) {

		Deque<AnnotationTypeMapping> queue = new ArrayDeque<>();
		addIfPossible(queue, null, annotationType, null, visitedAnnotationTypes);
		while (!queue.isEmpty()) {
			AnnotationTypeMapping mapping = queue.removeFirst();
			this.mappings.add(mapping);
			addMetaAnnotationsToQueue(queue, mapping);
		}
	}

	private void addMetaAnnotationsToQueue(Deque<AnnotationTypeMapping> queue, AnnotationTypeMapping source) {
		Annotation[] metaAnnotations = AnnotationsScanner.getDeclaredAnnotations(source.getAnnotationType(), false);
		for (Annotation metaAnnotation : metaAnnotations) {
			if (!isMappable(source, metaAnnotation)) {
				continue;
			}
			Annotation[] repeatedAnnotations = this.repeatableContainers.findRepeatedAnnotations(metaAnnotation);
			if (repeatedAnnotations != null) {
				for (Annotation repeatedAnnotation : repeatedAnnotations) {
					if (!isMappable(source, repeatedAnnotation)) {
						continue;
					}
					addIfPossible(queue, source, repeatedAnnotation);
				}
			}
			else {
				addIfPossible(queue, source, metaAnnotation);
			}
		}
	}
```

综上所述我们可以发现尽管我们没有直接写 `@Component` 注解，只要我们加了类似于 `@Service`，`@RestController` 等注解也是可以成功被 `Spring` 扫描到注册成 `Spring Bean` 的，本质的原因是因为这些注解底层都使用了 `@Component` 作为元注解，经过源码分析我们发现了只要有 `@Component` 元注解标注的注解类也是同样会被进行扫描的。

## 总结

上面的源码追踪过程可能会比较枯燥和繁琐，最后我们来简单总结一下上面的内容：

1. 方法 `org.springframework.boot.SpringApplication#run(java.lang.String...) `中进行 `Spring` 上下文的创建；
2. 在初始化上下文的时候会创建扫描器 `ClassPathBeanDefinitionScanner`；
3. 在 `org.springframework.context.support.AbstractApplicationContext#refresh` 进行 `beanFactory` 准备；
4. `org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan` 进行资源扫描
5. 在 `org.springframework.core.annotation.MergedAnnotationsCollection#MergedAnnotationsCollection` 进行注解 `mappings` 的赋值；
6. `org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#scanCandidateComponents` 方法中进行候选组件的判断；

上面追踪的过程可能会比较复杂，但是只要我们理解了原理还是可以慢慢跟上的，因为我们只要把握好了方向，知道首先肯定会进行资源扫描，扫描完了肯定是根据注解之间的关系进行判断，最终得到我们需要的候选组件集合。至于如何创建 MetadataReader 和如何获取元注解，只要我们一步步看下去就是可以找到的。

最后说明一下，`Spring Framework` 每个版本的具体实现会有差异，阿粉使用的版本是 `5.3.24 `，所以如果小伙伴看到自己的代码追踪的效果跟阿粉的不一样也不会奇怪，可能是因为版本不一样而已，不过本质上都是一样的。

![](https://yuandifly.com/wp-content/uploads/2022/07/1639927740-3dd04cdc7b7e92c-1.jpg)

更多优质内容欢迎关注公众号【Java 极客技术】，我准备了一份面试资料，回复【bbbb07】免费领取。希望能在这寒冷的日子里，帮助到大家。
