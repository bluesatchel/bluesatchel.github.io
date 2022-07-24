---
title: 使用codeql的cc链分析
date: 2022-05-12 16:38:44
tags:
      - codeql
      - java
      - cc
categories: codeql
---

使用codeql来分析cc链

由于codeql无法多个数据库之间查询流,所以只能采用半条链拼接半条链的方式来进行操作

[参考文章](https://www.freebuf.com/articles/web/283795.html)

#### 基础概念

```
什么是source和sink

在代码自动化安全审计的理论当中，有一个最核心的三元组概念，就是(source，sink和sanitizer)。

source是指漏洞污染链条的输入点。比如获取http请求的参数部分，就是非常明显的Source。

sink是指漏洞污染链条的执行点，比如SQL注入漏洞，最终执行SQL语句的函数就是sink(这个函数可能叫query或者exeSql，或者其它)。

sanitizer又叫净化函数，是指在整个的漏洞链条当中，如果存在一个方法阻断了整个传递链，那么这个方法就叫sanitizer。
```

分析cc1进行一次尝试
