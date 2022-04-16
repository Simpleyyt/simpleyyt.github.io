---
layout: post
categories: Java
title: 在 Java 日常开发中，排名前五的 Exception，保证你全部遇到过！
tagline: by 子悠
tags: 
  - 子悠
---

说到 Java 中的 `Exception` 可以说是谁见谁恨，一旦遇见 `Exceptio` 说明我们的程序出了异常，我们都知道 `Java` 的异常都是 `Throwable` 对象，`Throwable` 有两个子类，分别是 `Error` 和 `Exception`，对于 `Error` 中我们常见的无非就是 `OutOfMemoryError` 和 `StackOverflowError`，而对于 `Exception` 我们常见的会稍微多几个。这篇文章给大家介绍在开发中 Top 5 的异常，相信每一个你都遇到过！

<!--more-->

首先 `Exception` 又分为 `RuntimeException` (运行时异常)和 `CheckedException` (检查时异常)，两者区别如下：

- `RuntimeException`：顾名思义，在程序运行的时候触发的异常，这种异常往往是没办法提前知道的，只有程序在运行的时候才能触发出来，通常情况下出现这种 `Exception` 基本上都是代码的逻辑错误。

- `CheckedException`：编译期间可以检查到的异常，必须显式的进行处理（捕获或者抛出到上一层）。

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h1bkl4yf3jj20rw0d0ab1.jpg)

这里只列表最常见的五个 `Exception`，包含运行时和编译期间可检查的异常，下面我们一起看下吧。

### 5. NumberFormatException

字符串类型的数字在日常开发中经常会遇到，通常会使用类似于`int n = Integer.parseInt(num);` 的代码，如果传进来的 `num` 是数字类型的字符串，那么没问题，但是当传进来的 num 是字母的时候，就会出现 `NumberFormatException`，例如下面的代码：

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h1bkl8t16oj21ay0mkdjx.jpg)

这种异常通常会发生在前端传递的一些数值类型的表单参数，但是前后端可能没有进行验证，直接将不正常的数值字符串传到后端，从而导致。所以我们在做这种转换的时候，一定要先验证字符串是否是数值。可是使用工具类`NumberUtil.isNumber(str)` 来进行验证，这里推荐使用`hutool`，很强大的工具包。 

### 4. IllegalArgumentException

`IllegalArgumentException` 这个异常相信大家也经常会遇到，当调用一些方法或者一些接口的时候，经常会出现这样的异常，本质的原因是因为传递的参数非法，下游的方法抛出了这个异常。这个异常跟上面的 `NumberFormatException` 异常有点类似的味道，不过 `NumberFormatException` 这个异常更具体说明是字符串的类型。

解决这个异常的方法就是把参数类型匹配上就好了，通常在开发和调试的时候，就可以解决，线上很少的情况才会出现，除非有版本升级不兼容。

### 3. ClassCastException

强制类型转换异常 `ClassCastException` 也是一个很常见的异常，当我们试图将一个类强制转换为另一个实例的类时，就会发生 `ClassCastException`。如下所示

![](https://tva1.sinaimg.cn/large/e6c9d24egy1h1bklcaicrj21ge0nqaev.jpg)

处理此异常的方法是使用 `instanceof `运算符，我们可以通过 `instanceof` 来判断是什么对象，然后进行对应的处理，这一点在反射的时候，有时候会很有用。

```java
/**
     * 通过反射，获取实体所有非null的属性，并增加到map中
     *
     * @param modelObj
     * @return
     */
public static Map<String, Object> getClassFieldsValues(Object modelObj) {
        Map<String, Object> map = new HashMap<>();
        Field[] fields = modelObj.getClass().getDeclaredFields();
        for (Field field : fields) {
            Object obj = ReflectionUtils.getFieldValue(modelObj, field.getName());
            if (obj != null && !obj.equals("")) {
                String fieldName = field.getName();
                if (isJavaClass(field.getType())) {
                    if (obj instanceof Date) {
                        map.put(fieldName, DateUtils.formatDate((Date) obj, "yyyy-MM-dd HH:mm:ss"));
                    } else {
                        map.put(fieldName, obj);
                    }
                } else {
                    for (Field subField : obj.getClass().getDeclaredFields()) {
                        Object subObj = ReflectionUtils.getFieldValue(obj, subField.getName());
                        if (subObj != null && !subObj.equals("")) {
                            if (subObj instanceof Date) {
                                map.put(fieldName + "." + subField.getName(), DateUtils.formatDate((Date) subObj, "yyyy-MM-dd HH:mm:ss"));
                            } else {
                                map.put(fieldName + "." + subField.getName(), subObj);
                            }
                        }
                    }
                }
            }

        }
        return map;
    }
```

### 2. ClassNotFoundException

`ClassNotFoundException` 是一个可以检查的异常，主要在使用当应用程序尝试通过其完全限定名称加载一个类并且无法在类路径上找到它的定义时发生。这主要发生在尝试使用 `Class.forName()`、`ClassLoader.loadClass()` 或 `ClassLoader.findSystemClass()` 加载类时。因此，我们需要格外小心 `java.lang.ClassNotFoundException`。为避免此异常，我们需要确保将类正确添加到类路径中。

同样的还有一个 `NoSuchMethodException`， 这个异常的发生主要在前端后调用，或者服务之间调用的时候版本不一致发生。

处理这两种异常，我们要保证访问的类和调用的方法都存在，对应的版本要正确，基本上不会有什么问题。这里再强调下，遇到这两种异常的时候，一定要定位好运行时的环境，依赖和版本；出现这种异常肯定是没找到，不要因为本地存在或者测试环境能找到就觉得怀疑报错了异常，要知道代码是骗不了人的。

### 1. NullPointerException

排名第一的绝对是臭名昭著的 `NullPointerException`。对于我们 `Java` 开发人员来说，不用再细说 `NPE`，当我们尝试访问指向空引用的变量时就会出现空指针异常。

所以再使用一些传入的或者调用的获得的对象的时候，我们要做的就是先判断是否为 `null`，只有在非 `null` 的时候才能正确使用，不然就会报空指针。

空指针的优雅处理相关的文章网上已经很多了，阿粉这里就不过多说明了，只能说空指针的发明真的是一个鸡肋。

### 总结

今天给大家介绍了 `Java` 开发人员常见的 Top5 的异常，每一个都那么令人讨厌。

