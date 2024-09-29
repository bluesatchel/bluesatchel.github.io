---
title: java调试环境配置
date: 2022-03-09 22:54:01
tags:
categories: java
---

java代码调试环境配置<!--more-->

首先需要有对应版本的jdk文件,由于windows上的jdk都是exe,所以可以让它安装到虚拟机里面,再把安装的jdk文件夹拷贝出来

然后去下载对应版本的openjdk

参照这篇文章进行操作https://www.cnblogs.com/haimishasha/p/9909055.html

解压jdk中的src压缩包

![image-20220311225711001](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220311225711001.png)

将`openjdk`中`src\share\classes`目录下的sun文件夹复制到`jdk`src目录中

![image-20220311225744162](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220311225744162.png)

接着在idea中将`src`目录添加到

![image-20220311230642856](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220311230642856.png)
