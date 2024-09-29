---
title: java多线程下载器
date: 2022-08-26 19:09:49
tags:
      - 多线程
      - java
description: 简单的java多线程下载器的学习记录
---

记录一些[java多线程下载器](https://github.com/bluesatchel/MultiThreadDownloader)学习过程中遇到的问题和解决方法

### 断点续传实现

#### RandomAccessFile类

RandomAccessFile支持"随机访问"的方式，程序可以直接跳转到文件的任意地方来**读写**数据。

1. RandomAccessFile**可以自由访问文件的任意位置**。
2. RandomAccessFile**允许自由定位文件记录指针**。
3. RandomAccessFile**只能读写文件**而不是流。

使用`accessFile.seek(accessFile.length());`方法将指针移动到文件的末尾

同时再结合HTTP协议中关于文件的一个请求头`Content-Range `来实现从某个字节到某个字节之间的文件数据的获取即可实现简单的http断点续传功能

```java
public static HttpURLConnection getHttpURLConnection(String url,long startPos,long endPos) throws Exception{
        HttpURLConnection httpURLConnection = getHttpURLConnection(url);
        log.info("下载的区间是:{}--{}",startPos,endPos);
      if(endPos!=0){
          httpURLConnection.setRequestProperty("RANGE","bytes="+startPos+"-"+endPos);

      }else{
          //最后一块只需要传入开始的字节位置即可
          httpURLConnection.setRequestProperty("RANGE","bytes="+startPos+"-");
      }
      return httpURLConnection;
    }
```

![image-20220901175702361](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220901175702361.png)

### 下载信息的获取

在多个线程同时进行下载时,需要实时获取下载的进度

例如:已经下载的文件大小数据,它需要被各个线程按照自己已经下载的数据大小实时更新

有几种选择:

- 原子类
- LOCK锁
- Synchronized内部锁

在这里综合考量下,使用原子类更为方便

```java
//本地已下载文件的大小，
    public static LongAdder finishedSize=new LongAdder();
```

原因如下:

​	如果使用synchronized内部锁,一般直接锁的是对象,并且还需要向外暴露一个同步方法或者包含同步代码块的方法给各个线程使用

​	如果使用LOCK锁也有同样的麻烦,这里只是简单的一个对象中属性的操作,使用原子类无疑是粒度最小的

接着单独开启一个线程,让其按照固定的时间去检测已经下载的文件的大小`finishedSize`属性,并在控制台做出输出即可

这里使用`Executors.newScheduledThreadPool(1);`线程池最佳,使用其中的`scheduledExecutorService.scheduleAtFixedRate`方法添加任务进去即可