---
title: 构建Connons-Colletions3.2_x 的Codeql数据库
date: 2022-05-06 13:41:35
tags:
      - codeql
      - java
categories: codeql
---


# 构建Connons-Colletions3.2_x 的Codeql数据库

安装配置好codeql也有一段时间了,期间看了一些简单的语法和例子,最近学校期末了,老师安排的屁事有点多,导致好几天没学习了

之前就听说codeql分析利用链特别厉害,java中反序列化利用链最出名的就是cc链了吧(个人观点),我学习codeql的目标就是自己能仿照着编写cc链的ql语句

但是codeql现在的java模块有个问题,就是生成数据库的时候,只能生成一个项目的代码数据库,它的依赖只有在代码中引入并加载的类才会被添加到数据库当中去

对此,我去github提问了一下(这也是我第一次在github提问...)

![image-20220512152353868](https://img-blog.csdnimg.cn/img_convert/4e3482e87f6c59f7fb66e9d7f4c6645b.png)

这是维护者大佬做出的回复

![image-20220512152328284](https://img-blog.csdnimg.cn/img_convert/05cf854bb259eefe20bbb18ed9314c4d.png)

总而言之,就是想分析哪个依赖,就得单独为它创建数据库,我点了大佬的链接进去,下载下来的数据库是cc4的最新版本的数据库,可能是因为master分支放的是cc4吧.....



那没办法只能自己建cc3的库了.....

于是参照这篇文章https://blog.csdn.net/mole_exp/article/details/122330521

这篇文章中提到两种数据库获取方式,第一种就是去lgtm直接下载,第二种是自己clone一份代码下来,然后上传到github仓库并配置lgtm让lgtm扫描后再去下载数据库,但是转念一想,既然它能生成,那我自己用clone的代码来搞不也一样吗



于是开始了痛苦之路

## 建立CC3数据库

接下来的东西需要对git的基础命令和原理有一定的了解,可以先去b站看[狂神的git视频](https://www.bilibili.com/video/BV1FE411P7B3?spm_id_from=333.337.search-card.all.click)学习一下

git clone并切换源码到指定分支

先打开git bash  然后 clone cc项目

```
git clone https://github.com/apache/commons-collections.git
```

接下来cd进入项目目录,切换分支为cc3

```
git checkout COLLECTIONS_3_2_X
```

> 错误方式

一看里面有pom.xml文件,大喜过望,开始mvn clean install生成数据库,然后一个大大的fatal甩在脸上......

```bash
codeql database create D:\TOOLS\codeql\database\commons-collection3.2 --language="java" --command="mvn clean install --file pom.xml" --source-root=D:\TOOLS\codeql\soutce-project\commons-collections --overwrite 
```

之前的文章中说过,生成数据库出错了不能着急,要一步步排查,首先codeql命令行是不会给出具体的错误提示的,只会说是mvn这条命令报错了

总的来说有两个错误,我们一一来分析解决

#### 错误一

然后我们打开idea,在idea命令行中使用mvn clean install --file pom.xml进行编译,出现了这样返回类型不一致的报错,打开报错的类的源码,发现这几个名字牛逼轰轰的Map类中remove方法返回的是Object类型的,这和其间接继承的Map接口中定义的boolean截然不同,怪不得报错

![image-20220512153751430](https://img-blog.csdnimg.cn/img_convert/8d6d61815dec7b21aa543fcb46fa45c0.png)

![image-20220512154328441](https://img-blog.csdnimg.cn/img_convert/bc39db9b1d776707a9b8c386825b546c.png)

###### 解决方法

当我在idea的Project Structure中把jdk版本从平时用的1.8下调到1.7的时候,这个报错就消失了,欧耶!!!

#### 错误二

但是与此同时,还有个问题也发生了,使用idea右侧边连的maven 依次 clean和install的时候,报了

```
错误: 不再支持目标选项 1.2。请使用 7 或更高版本。
```

的错误

![image-20220512154036543](https://img-blog.csdnimg.cn/img_convert/5c58d7e1dc433f315606b54789280a6c.png)

去百度之后发现大概意思就是说,jdk版本太低了,咋了咋了的

在cc3的pom.xml最下面发现它自己指定了编译的jdk版本是1.2

![image-20220512154601318](https://img-blog.csdnimg.cn/img_convert/b7c35e183f5f612933fc48656b934440.png)
###### 解决方法
为了一次性到位,首先把pom.xml中的版本改成1.7

![image-20220512154736745](https://img-blog.csdnimg.cn/img_convert/fdf861fb56261b5ff30edf2d38ee09a3.png)

接着在最下面的plugins中添加这两个

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220512154819016.png" alt="image-20220512154819016" style="zoom:80%;" />

```xml
<plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <source>1.7</source>
          <target>1.7</target>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.22.2</version>
        <configuration>
          <testFailureIgnore>true</testFailureIgnore>
        </configuration>
      </plugin>
```

并且,打开本地maven的setting.xml文件

找到对应的位置,就是`<profiles>`标签下面就行,添加如下语句,指定默认jdk版本

```xml
  <profile> 
<id>jdk-1.7</id> 
<activation> 
<activeByDefault>true</activeByDefault> 
<jdk>1.7</jdk> 
</activation> 
<properties> 
<maven.compiler.source>1.7</maven.compiler.source> 
<maven.compiler.target>1.7</maven.compiler.target> 
<maven.compiler.compilerVersion>1.7</maven.compiler.compilerVersion> 
</properties> 
</profile>
```

但是还是报错了,最后去环境变量里面把JAVA_HOME改成1.7的就行了(到这里终于目标为啥配环境变量的时候非得搞个叫JAVA_HOME的,而不是直接往path里面添加,改起来是真滴快啊)

![image-20220512155048464](https://img-blog.csdnimg.cn/img_convert/3ec4b8d3b4b6e92bacb55f064798a99f.png)

#### 成功

最后再次打开命令行输入

```bash
codeql database create D:\TOOLS\codeql\database\commons-collection3.2 --language="java" --command="mvn clean install --file pom.xml" --source-root=D:\TOOLS\codeql\soutce-project\commons-collections --overwrite 
```

成功构建cc3的数据库

![image-20220512155241642](https://img-blog.csdnimg.cn/img_convert/25f2a1ac18122cb378b43056fe34a42f.png)