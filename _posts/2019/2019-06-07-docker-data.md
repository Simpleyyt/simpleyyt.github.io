---
layout: post
title:  Docker 数据卷操作[Docker 系列-6]
tagline: by 江南一点雨
categories: Docker系列
tags: 
    - 江南一点雨
---

# 数据卷入门

在前面的案例中，如果我们需要将数据从宿主机拷贝到容器中，一般都是使用 Docker 的拷贝命令，这样性能还是稍微有点差，没有办法能够达到让这种拷贝达到本地磁盘 I/O 性能呢？有！

<!--more-->

数据卷可以绕过拷贝系统，在多个容器之间、容器和宿主机之间共享目录或者文件，数据卷绕过了拷贝系统，可以达到本地磁盘 I/O 性能。  

本文先通过一个简单的案例向读者展示数据卷的基本用法。

以前面使用的 nginx 镜像为例，在运行该容器时，可以指定一个数据卷，命令如下：

```
docker run -itd --name nginx -v /usr/share/nginx/html/ -p 80:80 bc26f1ed35cf
```

运行效果如下：  

![15-1](/assets/images/2019/java/image_javaboy/0530/15-1.png)  

此时，我们创建了一个数据卷并且挂载到容器的 `/usr/share/nginx/html/` 目录下，小伙伴们知道，该目录实际上是 nginx 保存 html 目录，在这里挂载数据卷，一会我们只需要修改本地的映射位置，就能实现页面的修改了。  

接下来使用 docker inspect 命令查看刚刚创建的容器的具体情况，找到数据卷映射目录，如下：  

![15-2](/assets/images/2019/java/image_javaboy/0530/15-2.png)  

可以看到，Docker默认将宿主机的 `/var/lib/docker/volumes/0746bdcfc045b237a6fe2288a3af9d7b80136cacb3e965db65a212627e217d75/_data` 目录作为source目录，接下来，进入到该目录中，如下：  

![15-3](/assets/images/2019/java/image_javaboy/0530/15-3.png)

此时发现该目录下的文件内容与容器中 `/usr/share/nginx/html/` 目录下的文件内容一致，这是因为挂载一个空的数据卷到容器中的一个非空目录中，那么这个目录下的文件会被复制到数据卷中（如果挂载一个非空的数据卷到容器中的一个目录中，那么容器中的目录中会显示数据卷中的数据。如果原来容器中的目录中有数据，那么这些原始数据会被隐藏掉）。  

小贴士：

> 由于 Mac 中的 Docker 有点特殊，上文提到的 /var/lib/xxxx 目录，如果是在 linux 环境下，则直接进入即可，如果是在 mac 中，需要首先执行如下命令，在新进入的命令行中进入到 /var/lib/xxx 目录下：  
> screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty  


接下来修改改文件中的index.html文件内容，如下：  

```
echo "hello volumes">index.html
```
 
修改完成后，再回到浏览器中，输入 http://localhost查看nginx中index.html 页面中的数据，发现已经发生改变。说明宿主机中的文件共享到容器中去了。

# 结合宿主机目录

上文中对于数据卷的用法还不是最佳方案，一般来说，我们可能需要明确指定将宿主机中的一个目录挂载到容器中，这种指定方式如下：  

```
docker run -itd --name nginx -v /Users/sang/blog/docker/docker/:/usr/share/nginx/html/ -p 80:80 bc26f1ed35cf
```  
这样便是将宿主机中的 `/Users/sang/blog/docker/docker/` 目录挂载到容器的 `/usr/share/nginx/html/` 目录下。接下来读者只需要在 `/Users/sang/blog/docker/docker/` 目录下添加 html 文件，或者修改 html 文件，都能在 nginx 访问中立马看到效果。  

这种用法对于开发测试非常方便，不用重新部署，重启容器等。 

**注意：宿主机地址是一个绝对路径**

# 更多操作

## Dockerfile中的数据卷

如果开发者使用了 Dockerfile 去构建镜像，也可以在构建镜像时声明数据卷，例如下面这样：  

```
FROM nginx
ADD https://www.baidu.com/img/bd_logo1.png /usr/share/nginx/html/
RUN echo "hello docker volume!">/usr/share/nginx/html/index.html
VOLUME /usr/share/nginx/html/
``` 

这样就配置了一个匿名数据卷，运行过程中，将数据写入到 `/usr/share/nginx/html/` 目录中，就可以实现容器存储层的无状态变化。  

## 查看所有数据卷

使用如下命令可以查看所有数据卷：  

```
docker volume ls
```  

如图：  

![17-1](/assets/images/2019/java/image_javaboy/0530/17-1.png)  

## 查看数据卷详情

根据 volume name 可以查看数据详情，如下：  

```
docker volume inspect 
```  

执行结果如下图：  

![17-2](/assets/images/2019/java/image_javaboy/0530/17-2.png)  

## 删除数据卷

可以使用 `docker volume rm` 命令删除一个数据卷，也可以使用 `docker volume prune` 批量删除数据卷，如下：  

![17-3](/assets/images/2019/java/image_javaboy/0530/17-3.png)  

![17-4](/assets/images/2019/java/image_javaboy/0530/17-4.png)  

批量删除时，未能删除掉所有的数据卷，还剩一个，这是因为该数据卷还在使用中，将相关的容器停止并移除，再次删除数据卷就可以成功删除了，如图：  

![17-5](/assets/images/2019/java/image_javaboy/0530/17-5.png)  


# 数据卷容器

数据卷容器是一个专门用来挂载数据卷的容器，该容器主要是供其他容器引用和使用。所谓的数据卷容器，实际上就是一个普通的容器，举例如下：  

- 创建数据卷容器

使用如下方式创建数据卷容器：  

```
docker run -itd -v /usr/share/nginx/html/ --name mydata ubuntu
```  

命令执行效果如下图：  

![18-1](/assets/images/2019/java/image_javaboy/0530/18-1.png)  

- 引用容器

使用如下命令引用数据卷容器：  

```
docker run -itd --volumes-from mydata -p 80:80 --name nginx1 nginx
docker run -itd --volumes-from mydata -p 81:80 --name nginx2 nginx
```  

此时， nginx1 和 nginx2 都挂载了同一个数据卷到 `/usr/share/nginx/html/` 目录下，三个容器中，任意一个修改了该目录下的文件，其他两个都能看到变化。  

此时，使用 `docker inspect` 命令查看容器的详情，发现三个容器关于数据卷的描述都是一致的，如下图：  

![18-2](/assets/images/2019/java/image_javaboy/0530/18-2.png)  

# 总结

本文主要向大家介绍了数据卷中的容器操作，整体来说还是非常简单的，小伙伴们，你学会了吗？

参考资料：

[1] 曾金龙，肖新华，刘清.Docker开发实践[M].北京：人民邮电出版社，2015.