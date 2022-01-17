---
title: IDEA插件下载慢怎么解决
date: 2022-01-1 15:33:59
tags:
      - 插件下载
      - 小技巧
categories: 小技巧
---

想学习Javaweb,但是用的是社区版的idea,并不支持web开发,需要下载一个名叫Smart Tomcat的插件,下载要么很慢,要么失败,在网络上寻到一个很好用的方法,也可以适用于其他集成开发环境的插件下载

首先用站长工具测试访问插件域名最快的节点http://tool.chinaz.com/speedtest

idea下载插件请求的域名是   plugins.jetbrains.com

测试完成后找到一个延迟最低的最好和自己运营商一致的ip

![image-20220113154149803](https://gitee.com/blue_satchel/images/raw/master/image-20220113154149803.png)

然后打开hosts文件,修改域名和ip的对应关系

`C:\Windows\System32\drivers\etc\hosts`

![image-20220113154237161](https://gitee.com/blue_satchel/images/raw/master/image-20220113154237161.png)

