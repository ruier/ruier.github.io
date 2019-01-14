---
title: SylixOS 系统初探
categories: [blog]
description: 记录的是随心所欲
keywords:  work
---

# 国产嵌入式硬实时操作系统 SylixOS 初体验

## 关于 SylixOS 

详细了解请见：http://wiki.sylixos.com/index.php/%E7%B3%BB%E7%BB%9F%E7%AE%80%E4%BB%8B




SylixOS是一款嵌入式硬实时操作系统，同其类似的操作系统，全球比较知名的还有VxWorks(主要应用于航空航天、军事与工业自动化领域)、RTEMS(起源于美国国防部导弹与火箭控制实时系统)、ThreadX(主要应用于航空航天与数码通讯)等。 

从全球范围上看，SylixOS作为实时操作系统的后来者，在设计思路上借鉴了众多实时操作系统的设计思想，其中就包括RTEMS、VxWorks、ThreadX等，使得具体性能参数上达到或超过了众多实时操作系统的水平，成为国内实时操作系统的最优秀代表之一。


## 入门指南

http://wiki.sylixos.com/index.php/SylixOS%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97

这篇文档太老太旧，很多下载链接失效，包括新的 win10 更新造成驱动不可用等等。导致我根本没法进行下去。

另一篇：http://wiki.sylixos.com/index.php/Linux%E7%8E%AF%E5%A2%83%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97

是 Linux 下的指导文档，仍然太老太旧，主要是新的代码已经不适用这文档了。

## 编译运行 

最新的代码编译是基于 RealEvo v3.0 的。有兴趣的可以申请官方的体验版。地址：

http://www.acoinfo.com/html/experience.php

但是我们这搞嵌入式的，还是需要自己编译自己拿到的源码。还在 git 信息记录了整个系统的提交历史，于是我们回退到之前的版本就行了，大概是文档的时间 2016 年 5 月的版本都是可以用的，这里：

强烈建议 Sylix 的开发者添加 tag 和 branch 信息，不然文档维护实在是太麻烦了。国内开源的环境还是需要提高。

回退后按照文档就可以编译了。但是 linux 的 qemu src 库没有了。所以只能在 Windows 上体验运行。

## 驱动安装

win10 系统安装驱动，需要超级用户权限。之后就可以运行了：

![启动界面](/images/blog/sylix_start.png)
