---
layout: post
title: Dockerfile学习笔记
date: 2018-07-10 19:50:24.000000000 +08:00
---

上一篇文章我们介绍了docker常用命令及其原理，但docker还可以通过dockerfile的方式来实现自动创建镜像，提高效率。本篇文章将介绍：什么是dockerfile，以及它的一些基本语法。

**Dockerfile的概念**

熟悉Linux环境下程序编译的读者应该都知道Makefile，Makefile的作用及就是把程序之间的依赖关系告诉make，使make可以依照Makefile指定的规则自动实现编译。

同理，Dockerfile也起到了同样作用，它是一个文本文件，里面包含创建镜像所需的一条条指令（instruction），每条指令构建一层。

**Dockerfile的及语法**

Dockerfile的每一条指令格式可以描述为：

{% highlight ruby %}
INSTRUCTION argument
{% endhighlight %}

所有的Dockerfile必须以`FROM`命令开始，该命令指定镜像基于哪个基础镜像创建。接下来的命令也都会基于这个基础镜像进行操作。

`FROM`可以多次使用，表示创建多个镜像，具体语法为:

{% highlight ruby %}
FROM < image name >
{% endhighlight %}

其他常用的实现自动化的命令还包括：

1.`MAINTAINER`，用于设置镜像的作者

{% highlight ruby %}
MAINTAINER < author name >
{% endhighlight %}

2.`RUN`，用于执行shell或exec环境下的命令。

RUN会在新创建的镜像上添加新的只读层。

{% highlight ruby %}
RUN < command >
{% endhighlight %}

3.`ADD`，用于复制文件。

{% highlight ruby %}
ADD < src > < dest >
{% endhighlight %}

src可以是URL或者启动配置上下文中的一个文件，dest是容器内的路径。

4.`CMD`，容器默认会执行的命令。

Dockerfile只允许使用一次CMD指令，如果有多个CMD指令存在，则只有最后一个是有效的。CMD有三种形式：

{% highlight ruby %}
CMD ["executable","para1","para2"]
CMD ["para1","para2"]
CMD command para1 para2
{% endhighlight %}

5.`EXPOSE`，指定容器运行时监听的端口。

{% highlight ruby %}
EXPOSE < port >
{% endhighlight %}

6.`ENTRYPOINT`，给容器配置一个可执行命令。这样一来，每次使用镜像创建容器时，可以设置一个特定的应用程序为默认程序。同时，该镜像每次被调用时也只能运行这个指定的应用程序。

Docker只允许有一个ENTRYPOINT。如果有多个存在，只有最后一个有效。

{% highlight ruby %}
ENTRYPOINT ["executable","para1","para2"]
ENTRYPOINT command para1 para2
{% endhighlight %}

7.`WORKDIR`，指定`RUN`、`CMD`与`ENTRYPOINT`命令的工作目录。

{% highlight ruby %}
WORKDIR /path/to/workdir
{% endhighlight %}

8.`ENV`，用于设置环境变量。

{% highlight ruby %}
ENV < key > < value >
{% endhighlight %}

9.`USER`，设置容器运行时的UID。

{% highlight ruby %}
USER < uid >
{% endhighlight %}

10.`VOLUME`，授权容器访问主机上的目录。

{% highlight ruby %}
VOLUME ["/data"]
{% endhighlight %}

**注意事项**

**上述的这些指令，每运行一次，都是在已有只读层的基础上新建一个层**。所以，若非必要，尽量将可合并的命令合并到一个指令中，可采用`&&`符号进行串联。

此外，为保证可读性，docker支持shell命令的行尾添加`\`表示换行。行首可采用`#`表示注释。

实际上，在Docker Store(https://store.docker.com/) 上有很多已经做好的官方镜像，可以直接拿来使用，如`nginx`、`redis`、`mongo`、`mysql`、`httpd`、`php`、`tomcat`等。也有一些方便开发的语言镜像，如`node`、`openjdk`、`python`、`ruby`、`golang`等。

当然，如果想自己动手创建镜像，那么采用dockerfile无疑是最佳的选择。