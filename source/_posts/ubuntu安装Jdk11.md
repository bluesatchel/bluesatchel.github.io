---
title: ubuntu安装Jdk11
date: 2021-09-01 18:10:47
tags:
      - java
---

##### 1.首先去官网下载jdk11

![image-20220901181241699](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220901181241699.png)

上传到linux服务器

##### 2.在/usr/local/创建jdk目录，并解压到jdk目录，并改名为jdk11

```bash
mkdir -p /usr/local/jdk
#解压jdk11到jdk目录
tar zxvf jdk-11.0.16_linux-x64_bin.tar.gz -C /usr/local/jdk/
#改名进行区分
mv /usr/local/jdk/jdk-11.0.16 /usr/local/jdk/jdk11
```

##### 3、使用update-alternatives对java版本进行管理，方便后续使用时需要切换

**这个工具的使用知识需要补充一下,解决了传统的手工配置环境变量的麻烦**

```bash
#添加java到update-alternatives进行管理(100这个值是可以调整的，越小越优先！！！)
update-alternatives --install /usr/bin/java java /usr/local/jdk/jdk11/bin/java 100
##这里是指导你怎么切换java版本，如果只有一个默认就是jdk11，就不用执行这条命令了
update-alternatives --config java
```

