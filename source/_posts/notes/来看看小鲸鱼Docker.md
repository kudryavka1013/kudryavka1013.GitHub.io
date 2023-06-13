---
title: 看看Docker
tags:
  - Docker
categories:
  - 笔记
date: 2021-05-12 23:04:06
---
#### 写在前面

本文是我学习Docker的一些记录，介绍了一些相关的概念和使用方法，以及自己的一些理解。内容比较简单，也没有实际的例子。

#### Docker是个啥？

简单的说，Docker就是一种虚拟化技术，对应用进程进行了封装隔离，成为一个个容器。每个容器都有配置好的虚拟环境，容器里的进程都是和虚拟的环境进行交互。

我们可以想象对于一组应用进程，把它们放在一个盒子里，盒子里看不到外面，也不知道外面有什么，但是可以通过盒子预先开好的孔洞和外部交流，这就是容器的虚拟环境，孔洞则是Docker给我们提供的接口。而一台机器上，可以有很多个这样的盒子，每个盒子的孔洞可能不同，这就是每个容器的虚拟环境都可以定制。

其实这就对应了Docker的图标：小鲸鱼就是我们的机器，上面的方块就是我们的一个个容器。

#### 基本概念

Image：镜像，提供容器运行时所需的各种东西

Container：容器，运行中的镜像实体；容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间，有自己的一套环境

Repository：仓库，存放/分发镜像，包括私人仓库和公共仓库

Dockerfile：构建镜像的文件

Volume：数据卷，存放数据

有一个可能不太恰当的比喻：

- Docker = 灶台
- Image = 菜谱和原材料
- Container = 饭菜
- Dockerfile = 纸笔

在灶台（Docker）上，我们使用菜谱和原材料（image）来做饭菜（container），同时我们可以用纸笔（dockerfile）来写我们自己的菜谱

#### Docker安装

看这个不好吗？

[安装 Docker - Docker —— 从入门到实践 (gitbook.io)](https://yeasy.gitbook.io/docker_practice/install)

#### 镜像 Image

要真正用上Docker，我们无非是运行一些容器；要运行容器，我们首先要有镜像。镜像就相当于容器的模版，根据镜像生成容器，它们就像类和实例之间的关系。

查看本地镜像：`docker image ls`

获取仓库里的镜像：`docker pull imageName`

基于一个镜像生成一个容器：`docker run imageName`

删除本地镜像：`docker image rm imageName`

使用这些命令，一般都要附带一些选项参数，上面的只是最基本、最简单的用法

#### 容器 Container

有了镜像之后，我们就可以生成一个容器；而且，容器是可以停止的，我们也可以将终止的容器重新启动

新建容器并启动：`docker run imageName`
Docker会检查本地有没有该镜像，如果不存在对应镜像，就会从公共仓库（Docker Hub）下载

从终止的容器启动：`docker container start containerName`

停止一个容器的运行：`docker container stop containerName`

重启一个容器：`docker container restart containerName`

删除一个容器：`docker container rm containerName`

同样的，我们一般都需要一些额外的选项参数，比如给容器命名，映射端口等

#### 用 Dockerfile 定制镜像

我们当然不满足于使用别人的菜谱！我们需要构建自己的应用时，也得自己定制镜像，虽然这么话是这么说，其实我们自己定制，也基本是根据已有的镜像进行操作。就好比改进别人写好的食谱，来符合自己的口味。

##### 镜像的分层存储

在定制自己的镜像前，得先了解一些镜像结构方面的知识。

**Docker 镜像** 是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像 **不包含** 任何动态数据，其内容在构建之后也不会被改变。

严格来说，镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由**多层文件系统**联合组成。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。

##### 定制镜像

镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，这个脚本就是Dockerfile

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建，下面我们看最常见的两个指令，除了FROM和RUN，还有COPY，ADD，CMD等等

###### FROM指令

要定制一个新的镜像，必须以一个镜像为基础，那我们就使用 `FROM imageName` 来指定基础镜像，**这是必须的，而且是第一条指令**。

那有人就要问了，我有镜像，我就不用，我就要从头构建，诶，就是玩儿。从格式上来说，那不行，你得用 `FROM scratch` 表示，这个 scratch 表示是一个虚拟的空镜像，就相当于不以任何镜像为基础，从零开始构建。

###### RUN指令

`RUN` 指令是就是用来执行命令行命令的，它有两种格式：

- shell格式：`RUN <命令>`，就像直接在命令行中输入的命令一样
- exec格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式

前面说过了，每一条指令就构建一层，如果我们有很多条命令，一条命令就用一个RUN，就很多余。我们要从逻辑角度来思考，一个RUN对应就做一件事情。比如编译、安装就视为一层的事情。

```shell
FROM debian:stretch

RUN set -x; buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

我们不看中间的内容，注意到RUN命令的每行末尾使用了 `\` 来换行，一个RUN里使用 `&&` 将多条shell命令串联起来，它们共同组成了一层。这才是编写Dockerfile的正确打开方式，避免一条命令行语句一个RUN

此外，之前也说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。那么我们在构建每一层的时候，都要顺带的把这层的无关垃圾清理掉，不要带到下一层去，免得搞到最后整个镜像非常的臃肿。具体怎么清理，看你构建的时候用到了什么东西。

##### 构建镜像

Dockerfile写好了，接下来就要利用它，构建出一个我们自己的镜像

`docker build -t 镜像名称 <上下文路径>`

-t 选项后面，我们就可以指定输出的镜像名称

这里我们要讨论一下这个路径的问题，这里指明是 **上下文路径**。首先我们得知道 build 的原理，这命令并不是说在本地进行构建，而是把需要构建的文件发到远程服务端（Docker引擎），远端构建好了再给我们发回来。我们输入的这个路径下的所有内容会被打包上传，远端拿到这么一个包，再根据里面的Dockerfile来构建出我们的镜像文件。

所以，我们一般命令都会这么用，在上下文目录下开终端：

```shell
docker build -t 镜像名称 .
```

注意有个点，这个 `.` 就是指把当前目录作为上下文

和这相关的 `COPY` 类指令，里面的路径也只能用相对路径，因为是相对上下文目录而言的，这些文件不能超出上下文目录，如果需要，就该把这些文件放进上下文目录里。

#### 总结

到此为止，我们已经知道了Docker的使用流程，使用镜像，运行容器什么的基本没什么可说的，就是一些命令。关键在于Dockerfile的使用以及一些需要注意的问题。希望看了这个文章能帮助你简单了解和使用Docker。建议看看阮一峰的两篇文章，里面有详细的解说和教程。

#### 参考

[什么是 Docker - Docker —— 从入门到实践](https://yeasy.gitbook.io/docker_practice/introduction/what)

[Docker 入门教程 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

[Docker 微服务教程 - 阮一峰的网络日志 (ruanyifeng.com)](https://ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)
