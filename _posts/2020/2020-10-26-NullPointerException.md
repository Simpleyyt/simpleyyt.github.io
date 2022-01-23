---
layout: post
categories: Java
title: 被同事的空指针硬生生的坑了好几个小时
tagline: by 懿
tags: 
  - 懿
---

阿粉入职这么久了，无论如何也不会想到会被自己同事写的一个接口返回的空指针异常折磨致死，折磨的死去活来，却完全不知道是什么原因，你有没有过这种经历呢？

<!--more-->

### NullPointerException

标题醒目，是为了给大家说，这个空指针异常，说实话，在项目里面很多都是很容易能够解决的，但是有时候发生问题的原因却是你无论如何想不到的，事情是这个样子的。

前端代码如下：

```

var setting = {
                url:"findFileById",
                data:{
                    id:id
                },
                success:function (data) {
                    console.log(data);
                },
                error:function (data) {
                    console.log("查询文件异常")
                }
            }
ajax(setting);

ajax在这里只是进行了一个封装

function ajax(setting) {
    $.ajax({
        type:"post",
        url:setting.url+".do",
        dataType:setting.dataType||"json",
        contentType:"application/json;utf-8",
        data:JSON.stringify(setting.data)||{},
        async:setting.async,
        success:function (data) {
            setting.success(data);
        },
        error:function (data) {
            setting.error("接口出错，请重试");
        }
    })

```

后台业务处理如下：

```
@PostMapping("findFileById")
@ResponseBody
public File findFileById(HttpServletRequest request, HttpServletResponse response, @RequestBody Map<String,Object> map){
    return deliverFileService.findFileById(request,map);
}

```

大家肯定会说，这么简单的事情你都不会，阿粉你干啥吃的，一个查询文件都有问题，而事实上，在代码里面我的同事也没有完全去处理这个空值的问题，结果导致一直都出在ajax里面出现“接口出错，请重试”的错误。

而问题就在于他没有处理空的数据，而直接就返给我了，这种问题也是非常的奇怪，很多时候不都是应该处理一下空的数据为防止NULL的异常么？而阿粉也第一时间找到了他，他说没问题，在他那里正常调用，我当时就尴尬了，我给你传递的参数是没问题的，查询数据如果为空，应该会有提示的才对。于是阿粉只能是简单的修改了一下他的代码，变成了

```
@PostMapping("findFileById")
@ResponseBody
public File findFileById(HttpServletRequest request, HttpServletResponse response, @RequestBody Map<String,Object> map){
    return deliverFileService.findFileById(request,map)!= null ?deliverFileService.findFileById(request,map) : new ; new DeliverFile();
}

```

阿粉不能给他改动太大，只能改成我这里调用如果是 null的时候，返还给我一个空对象就好了，如果不是的话，就把查询回来的数据完整的返还给我。

那么阿粉现在就来说说这个如何处理我们的空值的问题，不然以后你如果写好的数据接口，给别人调用，调用出来如果是个空的字符串也就罢了，但如果像是null这种不处理的东西，那么一定会被别人鄙视死。

### 如何处理空指针异常的问题

**什么时候出现NullPointerException？**

我们都知道 NullPointerException 是继承 RuntimeException 的，也就是运行的时候会出的异常信息，当我们写代码的时候，如果代码在运行的时候，我们使用的对象没有初始化的时候，或者是为空的时候，就会出现空指针的异常，而这个异常也是我们感觉最 Low 的，最不可能出现的异常，但是往往因为自己的不注意，就出现了。

![](http://www.justdojava.com/assets/images/2019/java/image_yi/2020/10-20/3.jpg)

其实这个办法可就太多了，而很多时候我们也是不去注意这个事情的，就比如说对象，判空操作，但是你如果在每个对象使用的时候都判空，那么你的代码真的就会出现：

```
if(a!=null){
    if(b!=null){
        if(c!=null){
            ....
        }
    }
}

```

当你看到这种代码的时候，第一感觉有没有直接想把这个朋友拉过来捶一顿，这种要是写多了，人都快疯了，尤其是二次维护的人员。

其实这种方法虽然笨，但是也算是一个习惯，判空，对功能上来说，肯定是不会出现很多麻烦，但是这么个空值也是很折磨人的，那么我们就来处理一下他把。

1.这个我们就不说了直接判断对象是不是为空就行了。

第二个，就是比对equals方法的时候，我们很多时候的写作习惯就是这种

```
if(text.equals("xxxxx")){
    
}

```

其实这么比对没有问题，但是你有没有想过，如果说你的text是个空呢？你比对的时候不就出错了？而曾经也有一个面试官问我，为什么在笔试题里面去吧已知的字符串写在前面，当时可能只是一种习惯，而后来却发现这是真的有用滴。

你改成：

```
if("xxxxx".equals(text)){
    
}
```
就会避免了出现空指针的错误了，多好的习惯不是么？

第三个，也是我们在Java8里面提供的特性Optional

ofNullable，就是Optional中提供的，将我们需要的参数传递过去，就可以判断是否为空了。

而对于集合来说，大家就可以使用之前修改的那个方法，判断是否为null，如果是null，那么我们一定要返回一个哪怕是空对象，或者是一个空的集合，这样对于之后调用你接口的人来说，也是非常友善的。

我知道很多人会说，那我在接口上面写上个注释，查询返回的值会有可能是个空，大家小心调用，虽然你提示了问题，但是你这是没有解决问题的体现呀，就相当于，你把所有的异常全部都抛出去了，而没有去处理他。

我们这时候还可以使用 Java8 里面提供的 Optional ，比如这个样子

```
Optional<Product>getProductOptional(String id)
```

这个时候，当我们的调用者知道有 Optional 的存在的时候，自然而然的明白了。

关于处理null，你还有其他的好的方式么？

