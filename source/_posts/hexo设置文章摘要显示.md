---
title: hexo的next主题设置文章摘要显示
date: 2021-12-03 13:48:36
tags:
    - hexo
categories: hexo
typora-root-url: ..
---

打开next配置文件

将该项设置为true

![image-20211205134957202](/images/hexo%E8%AE%BE%E7%BD%AE%E6%96%87%E7%AB%A0%E6%98%BE%E7%A4%BA%E9%83%A8%E5%88%86%E5%86%85%E5%AE%B9/image-20211205134957202.png)

显示文章摘要有两种方法

1.在需要显示截断的地方添加`<!--more-->`标签

2.在`front-matter`中添加`description`,`description`中的内容就会添加到首页上面,并且它的优先级高于`<!--more-->`标签
