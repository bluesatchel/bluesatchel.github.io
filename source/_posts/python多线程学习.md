---
title: python多线程学习
date: 2022-01-26 21:03:28
tags:
      - python
      - 多线程
      - web
categories: python
---

# python并发简单学习

<!--more-->

### 什么是cpu密集型计算,IO密集型计算

#### cpu密集型

cpu密集型特点是cpu占用高,运行效率受到cpu能力限制,比如解压缩,加密解密等

#### io密集型

大部分都是读写操作,比如:爬虫,读写数据库等

### 多线程,多进程,多协程的对比

前两种,一个进程中,可以启动N个进程

一个线程中,可以启动N个协程

#### 多进程

##### 优点

可以利用多核CPU并行运算

##### 缺点

占用资源最多,可启动数据比线程少

##### 适用于

CPU密集型计算

#### 多线程

##### 优点

相比进程,更轻量级,占用资源少

##### 缺点

相比进程,多线程只能并发执行,不能利用多CPU(GIL)

相比协程,启动数目有限制,占用内存资源,有线程切换开销

##### 适用于

IO密集型计算,同时运行的人物数目要求不多

#### 多协程

##### 优点

内存开销最少,启动协程数量最多

##### 缺点

支持的库有限制(aiohttp(√),requests(×)),代码实现复杂

##### 适用于

IO密集型计算,需要超多任务运行,但有现成协程库支持的场景

### GIL

全局解释器锁(英语:Global Interpreter Lock,缩写GIL)

是计算机程序设计语言解释器用于同步编程的一种机制,它使得任何时刻仅有一个线程在执行

即便在多核心处理器上,使用GIL的解释器也只允许同一时间执行一个线程,遇到IO操作解锁GIL

Python设计初期,为了规避并发问题引入了GIL,现在想去除却去不掉了

为了解决多线程之间数据完整性和状态同步问题

##### 如何规避GIL限制

![image-20220126221005962](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220126221005962.png)

![image-20220126221037204](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220126221037204.png)

## python多线程实操

```python
1.准备一个函数
def my_func(a,b):
    do_craw(a,b)
2.怎样创建一个线程
import threading
t=threading.Thread(target=my_func,args=(100,200))
3.启动线程
t.start()
4.等待结束
t.join()
```

技巧

可以通过循环和数组来添加线程

```python
urls为一个url
threads = []
for url in urls:
    threads.append(threading.Thread(target=my_func,args=(url,)))#切记args不能丢了,
for thread in threads:
    thread.start()
for thread in threads:
    thread.join()
```

### 待续......................

