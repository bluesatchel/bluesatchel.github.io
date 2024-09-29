---
title: codeql安装
date: 2022-05-06 12:14:36
tags:
      - codeql
categories: codeql
---

本文章为多次尝试之后,自己成功安装并运行成功codeql的记录,没啥技术含量...<!--more-->

### 下载

首先安装最新版的vsCode

#### 安装扩展

在其中搜索并安装CodeQl拓展

![image-20220506121927675](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506121927675.png)

#### 下载codeql引擎

接着下载codeql引擎,选择对应版本即可

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506121845766.png" alt="image-20220506121845766" style="zoom:80%;" />

点击小齿轮,Extension Settings 打开拓展设置

![image-20220506121953523](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506121953523.png)

把刚才下载并解压的ql引擎文件中的`codeql.exe`的路径填写上去

![image-20220506122124707](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506122124707.png)

同时把codeql.exe的路径配置到环境变量`path`中去,这一步是为了后面使用命令行构建数据库做准备

![image-20220506122629448](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506122629448.png)

#### 设置workspace

接下来设置workspace

直接去clone这个项目https://github.com/github/vscode-codeql-starter,这个项目中有大佬设置好的workspace,可以直接拿来用

在clone的时候因为其包含子项目,不能直接git clone,需要加一个`--recursive`参数

```bash
git clone --recursive https://github.com/github/vscode-codeql-starter.git
```

如果https的报ssl错误,记得换成ssh的连接clone

```bash
git clone --recursive git@github.com:github/vscode-codeql-starter.git
```

### 构建数据库

现在需要下载的东西都下载完成了,接下来是很重要的一步,也就是构建数据库

这里以java为例,建议第一次先新建一个maven项目,随便写点东西进去

给一个简单的maven项目建立数据库

```
codeql database create D:\TOOLS\codeql\database\jndi --language="java" --command="mvn clean install --file pom.xml" --source-root=E:\java\jndi --overwrite
```

第一部分是给建立数据库指定的文件夹,

`--source-root`顾名思义就是源码文件夹

`--command`则是指定使用maven来帮助进行编译打包,这里是最容易出问题的地方,各种依赖呀啥的都需要编译打包之后才能添加进数据库里面进行查询

也就是说codeql需要mvn帮助打包才能构建数据库

`--overwrite`参数表示可以覆盖旧的数据库

对于java来说,如果在构建数据库的过程中出错了,那么一般都是因为mvn打包出了问题,可以单独在idea终端中尝试`mvn clean install --file pom.xml`根据报错信息来推断哪里出了问题

### 执行查询语句

首先打开vscode,通过`file-> Open Workspace from file`打开刚才clone好的那个workspace项目中的`.code-workspace`文件

这样就成功引入了workspace

![image-20220506124421164](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506124421164.png)

![image-20220506124308766](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506124308766.png)

接着在vsCode的侧边栏

![image-20220506124042825](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506124042825.png)

点击这个QL,进去之后,就可以添加数据库了

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506124111095.png" alt="image-20220506124111095" style="zoom:67%;" />

通过目录引入刚才新建好的数据库

![image-20220506124535532](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506124535532.png)

接着在custom这个目录下新建test.ql文件,编写ql语句,进行第一次查询

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506124610207.png" alt="image-20220506124610207" style="zoom:80%;" />

![image-20220506124647783](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506124647783.png)

```sql
from int x, int y
where x = 3 and y in [0 .. 2]
select x, y, x * y as product, "product: " + product
as res order by  res desc
```

右键在当前数据库Run query

![image-20220506124801886](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506124801886.png)

![image-20220506124822269](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220506124822269.png)

成功执行了
