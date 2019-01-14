---
layout: post
title: docker 基础用法
categories: [blog, docker]
description: 记录的是随心所欲
keywords: docker, work
---

# docker start guide

## 为什么要使用 docker

官方的用法：

在docker的网站上提到了docker的典型场景：

- Automating the packaging and deployment of applications（使应用的打包与部署自动化）
- Creation of lightweight, private PAAS environments（创建轻量、私密的PAAS环境）
- Automated testing and continuous integration/deployment（实现自动化测试和持续的集成/部署）
- Deploying and scaling web apps, databases and backend services（部署与扩展webapp、数据库和后台服务）

对于我们的 Android 开发环境，就遇到 java 版本切换， make 版本切换， gcc 版本切换等等，使得使用开发环境异常麻烦， docker 刚好可以解决这样的问题。

1. 在同一个系统中部署多种应用环境
2. 环境相互隔离，不影响其他应用
3. 统一存储，不浪费空间

## docker 安装

这里以阿里云 ECS 上安装为例，ECS 上有高速镜像下载，方便使用，自己在环境搭建上可能遇到下载缓慢问题。

<!--注意：docker 需要内核版本较高，而且只能运行在 64 bit 处理器--> 

```bash
$ sudo apt-get update

#如果内核版本较旧，则需要先更新内核，以下是更新xenial内核

$ sudo apt-get install linux-image-generic-lts-xenial
$ sudo reboot

#使用curl获取最新的Docker (使用阿里的镜像服务且使用云服务器外部网络下载)

$ curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -

```

安装完成后，测试 helloworld：

```bash

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash
```

现在可以使用 docker 来搭建环境了

## docker 使用

### 获取镜像

镜像获取使用 pull 命令

```bash
 docker pull ubuntu:16.04
```

这样就得到一个 ubuntu 16.04 的镜像

### 列出本地镜像

```bash
docker images
```

### 运行

```bash
docker run -t -i ubuntu:16.04 /bin/bash

    -t  分配一个伪终端并绑定到容器标准输入

    -i  让容器的标准输入保持打开
```

```
docker ps
```

### 结束

```bash
docker stop
```

### 后台运行

```bash
进入容器(-d 容器进入后台)

4.2.1 attach命令：

# docker run -idt ubuntu

# docker ps

# docker attach <docker-name>
```

### 使用docker exec进入Docker容器

　　docker在1.3.X版本之后还提供了一个新的命令exec用于进入容器，这种方式相对更简单一些，下面我们来看一下该命令的使用：

```
$ sudo docker exec --help   
```



接下来我们使用该命令进入一个已经在运行的容器

```
$ sudo docker ps  
$ sudo docker exec -it 775c7c9ee1e1 /bin/bash  
```



## 基本命令总结

```
基本命令总结

docker attach   登录一个已经在执行的容器

docker build    建立一个新的image

docker commit   提交一个新的image

docker cp   将容器中的文件拷贝到主机上

docker daemon    docker运行可指定项详解

docker diff     较一个容器不同版本提交的文件差异

docker events     获取sever中的实时事件

docker export    导出一个容器

docker history    显示一个image的历史

docker images     列出image

docker import     导入已有的image

docker info    展示docker的信息

docker inspect     显示更底层的容器或image信息

docker kill     杀死docker进程

docker load     加载image

docker login     登录docker注册服务器

docker logs     获取容器的日志

docker pause    暂停容器中的所有进程

docker port     端口转发

docker ps     列出所有容器

docker pull     从远端拉取一个image

docker push    推送image到注册服务器

docker restart     重启一个容器或多个容器

docker rmi    删除image

docker rm    删除一个或多个容器

docker run     运行一个新的容器

docker save    打包image

docker search     搜索images

docker start    启动一个容器

docker stop    停止一个容器

docker tag    为image打标签

docker top    显示容器中的进程

docker unpause    取消暂停所有的进程

docker version    显示版本信息

docker wait     阻塞容器运行

作者：MichaelChoo
链接：http://www.jianshu.com/p/7735a230841e
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

