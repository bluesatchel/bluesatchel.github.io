---
title: hexo上传图片
date: 2021-12-02 17:32:02
tags: hexo上传图片
categories: hexo
typora-root-url: ..
---

参考文章:

https://yinyoupoet.github.io/2019/09/03/hexo%E5%8D%9A%E5%AE%A2%E4%B8%AD%E6%8F%92%E5%85%A5%E5%9B%BE%E7%89%87/

所有的博客文件保存在_post目录下

<!--more-->

在source下新建一个images文件夹

打开Typora的 文件->偏好设置

![](/images/pic-hexo/image-20211203001406102.png)

这样该文章中的图片都会保存在/source/images/文章名/图片名称

现在的相对路径是基于当前md文件的,在服务器上可没有md文件,需要再typora中进一步设置

在typora菜单栏点击 格式->图像->设置图片根目录，将hexo/source作为其根目录。

会在md文件中会自动添加这一段

![](/images/pic-hexo/image-20211202174427901.png)

然后在每次写文章时加上上面这句话,就能保证根目录正确了



#### 后记

这种方式还是很不稳定,最后采用了腾讯云COS+picgo+typora的方式
