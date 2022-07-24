---
title: codeql编写cc链查询
date: 2022-05-14 15:58:37
tags:
      - codeql
      - java
      - cc
categories: codeql
---

有了之前的cc3的数据库,就可以尝试编写cc的反序列化调用链了,在编写ql语句之前,首先需要明确反序列化利用链的几个基础条件

[参考这篇文章](https://tttang.com/archive/1511/)

```
序列化利用链的条件:
1. source，入口点，一般就是一个readObject方法
2. sink，执行点，一般是动态方法执行、Jndi注入、写文件之类的   在本次的链中就是invokerTransformer的transform方法
3. gadget，连接source和sink的多个类。有几个条件：
	1). 类之间的方法调用是链式的。
	2). 类实例之间的关系是嵌套的，调用链上后一个类实例是前一个类实例的属性。
	3). 调用链上的类都需要是可以序列化的。
```

由于codeql无法在多个数据库之间查询流,并且cc的source全是jdk自带类的readObject,万恶之源还得是LazyMap和ChainedTransformer,就暂且以他两作为source,执行点呢就更简单了,就是InvokerTransformer和InstantiareTransformer
