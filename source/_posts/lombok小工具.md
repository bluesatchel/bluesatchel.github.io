---
title: lombok小工具
date: 2022-02-01 20:21:01
tags:
      - java
      - lombok
categories: java
---

使用这款小工具可以通过添加注解来实现类的构造方法,set,get方法,让代码更简洁

### 导入maven依赖

```xml
<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.16</version>
        </dependency>
```

```java
@Data     //添加get,set方法
@NoArgsConstructor//无参构造
@AllArgsConstructor//全参构造
```

### 安装lombok插件

IDEAsetting里面一直无法下载插件

IDEA官网下载链接,下载后直接安装就行

https://plugins.jetbrains.com/plugin/6317-lombok

idea2020.1.1版本的

https://plugins.jetbrains.com/files/6317/100775/lombok-plugin-0.33-2020.1.zip?updateId=100775&pluginId=6317&family=INTELLIJ

### 开启AnnocationProcessors

点击File-- Settings设置界面，开启 AnnocationProcessors：

按下alt+7即可看到已经生成了对应的构造器和get,set方法

![image-20220201204237651](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220201204237651.png)
