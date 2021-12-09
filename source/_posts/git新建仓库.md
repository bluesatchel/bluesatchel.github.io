---
title: git新建仓库
date: 2021-12-04 16:49:31
tags: 
     - git
     - github
categories: git
typora-root-url: ..
---

git新建仓库上传代码

<!--more-->

<img src="/images/git%E6%96%B0%E5%BB%BA%E4%BB%93%E5%BA%93/image-20211204165449892.png" alt="image-20211204165449892" style="zoom: 67%;" />

首先create a new repository新建一个仓库,接下来找到本地要上传代码的文件夹

在该文件夹下打开git bash

输入下列命令

```
git config --global user.name "用户名"
git config --global user.email "邮箱"
git init  会生成.git文件夹
git add .
git commit -m "first commit"
git branch -M master
//使用ssh之前要先配置公钥
git remote add origin git@github.com:用户名/仓库名.git
git push -u origin main
```
