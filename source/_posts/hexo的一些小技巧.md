---
title: hexo的一些小技巧
date: 2021-12-03 13:48:36
tags:
    - hexo
categories: hexo
typora-root-url: ..
---

#### 设置文章摘要

打开next配置文件

将该项设置为true

![image-20211205134957202](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20211205134957202.png)

显示文章摘要有两种方法

1.在需要显示截断的地方添加`<!--more-->`标签

2.在`front-matter`中添加`description`,`description`中的内容就会添加到首页上面,并且它的优先级高于`<!--more-->`标签

#### 设置字体高亮

`<label style="background:yellow">字体背景高亮</lable>`

<label style="background:yellow">字体背景高亮</lable>

`<label style="color:red">修改字体颜色</lable>`

<label style="color:red">修改字体颜色</lable>
