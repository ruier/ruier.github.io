---
title: 突破百度云盘的大文件下载限制
categories: [blog]
description: 记录的是随心所欲
keywords:  work
---

# 突破百度云盘的大文件下载限制

## 百度的下载限制 

百度云盘有个很抽风的限制，文件过大，就会要求下载他的客户端，并且下载速度只有 100k，还要购买会员加速，恶心到不行。实在是不忍直视。

然而偶尔又有客户或者其他的人用百度云盘共享文件，这时候还是要从上面下载。

## 浏览器安装突破工具 

比如 Firefox，安装 https://addons.mozilla.org/zh-CN/firefox/addon/%E5%93%94%E5%BA%A6%E5%A8%98%E7%BD%91%E7%9B%98/?src=api

## 下载文件使用批量下载 

下载的时候选择要下载的大文件，和一个小文件，一起下载就会生成一个下载链接，复制链接就可以通过迅雷和 wget 下载了。
