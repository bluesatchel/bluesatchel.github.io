---
title: centos编译openjdk7成功---但是创建codeql数据库失败的记录
date: 2022-05-16 12:15:58
tags:

---

## centos编译openjdk7成功---但是创建codeql数据库失败的记录

>实验环境
>
>centos7
>
>jdk1.6.0_45 (这点很重要,起初由于使用了jdk8作为bootjdk导致莫名其妙的报错,查询了很久才解决)

本篇文章可以作为编译openjdk7的参考,但是由于不熟悉编译原理,到底为什么可以编译openjdk7却不能像openjdk8一样生成对应的codeql数据库就不知道为啥了



安装所需库和工具,依次执行按y即可

```bash
yum install cups-devel.x86_64
yum install freetype.x86_64 freetype-devel.x86_64
yum install alsa*
yum install ant
yum install cups-devel
yum install glibc-static libstdc++-static
yum install libX*
yum install libXt-devel
yum install libXrender-devel
yum install libXext-devel
yum install libXtst-devel
yum install gcc gcc-c++
yum install freetype-devel
yum install libstdc++-static
```

### 安装BootJdk

去官网下载安装jdk1.6

>链接https://www.oracle.com/java/technologies/javase-java-archive-javase6-downloads.html

![image-20220516125517310](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220516125517310.png)

下载下来通过ssh工具传到usr/local目录下

`chmod 775 jdk-6u45-linux-x64.bin `

使用改命令修改其为可执行

`./jdk-6u45-linux-x64.bin`

会自动在当前目录解压出来一个`jdk1.6.0_45`

接着,修改系统变量

`vim /etc/profile`

在最下面插入

```bash
#set Java enviroment

export JAVA_HOME=/usr/local/jdk1.6.0_45

export JRE_HOME=${JAVA_HOME}/jre

export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:/usr/local/src/apache-ant-1.8.2/lib/ant-launcher.jar

#set Ant enviroment

ANT_HOME=/usr/local/src/apache-ant-1.8.2

export PATH=$JAVA_HOME/bin:$ANT_HOME/bin:$CLASSPATH:$JRE_HOME/bin:$PATH
```

然后`!wq`退出

`source /etc/profile`更新变量

`java-version`

显示为jdk1,6即配置成功

### 下载openjdk

直接去官网下载很慢,推荐一个技巧,参考链接https://blog.csdn.net/wbo112/article/details/121584122

文章中添加的是openjdk的仓库,我们在openjdk的github仓库那里搜索jdk7u按照一样的方式即可生成到码云

我们需要编译的是jdk7u21,在生成好的jdk7u仓库里面通过标签找到对应版本下载即可

![image-20220516132045787](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220516132045787.png)

通过ssh上传到centos的任意位置

然后通过`unzip`进行解压

解压完成后进入解压出来的目录

在里面创建bash脚本

`touch jdk.sh`

`chmod jdk.sh`

`vim jdk.sh`

往里面添加

```bash
export LANG=C

export ALT_BOOTDIR=/usr/local/jdk1.6.0_45

export ALT_JDK_IMPORT_PATH=/usr/local/jdk1.6.0_45

export ALLOW_DOWNLOADS=true

export HOTSPOT_BUILD_JOBS=4

export ALT_PARALLEL_COMPILE_JOBS=4

export SKIP_COMPARE_IMAGES=true

export USE_PRECOMPILED_HEADER=true

export BUILD_LANGTOOLS=true

#export BUILD_JAXP=false

export BUILD_JAXWS=false

#export BUILD_CORBA=false

export BUILD_HOTSPOT=true

export BUILD_JDK=true

export DISABLE_HOTSPOT_OS_VERSION_CHECK=ok

export SKIP_DEBUG_BUILD=false

export SKIP_FASTDEBUG_BUILD=true

export DEBUG_NAME=debug

BUILD_DEPLOY=false

BUILD_INSTALL=false

export ALT_OUTPUTDIR=/usr/local/src/openjdk/build

unset JAVA_HOME

unset CLASSPATH

make sanity
```

然后`!wq`退出并保存

接着`source ./jdk.sh`

如果输出结果为Sanity check passed表示编译检查通过(但是不代表编译一定不会出错)

如果检查通过了,不要切换终端,(因为设定的变量在其他终端中无法读取)

就在当前终端中输入`make`进行编译

![image-20220516124540997](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220516124540997.png)

最终在经过大概15分钟的等待之后,出现这段字符就表示编译成功了,我这里两分钟是因为已经编译过一次了



### 生成codeql数据库失败

我编译jdk7u21的数据库的初衷是想使用codeql编写7u21反序列化漏洞的查询语句试试水

对编译原理啥的都不咋了解,甚至连jdk为啥要编译都不知道...

编译数据库的过程是参考的这篇大佬编译jdk8数据库的[文章](https://blog.csdn.net/mole_exp/article/details/122330521)

按照本篇文章的操作,只要通过make编译jdk的准备都做好,并且make编译还成功了的话,使用这段语句便可成功编译出jdk的codeql数据库

`codeql database create openjdk7u21-db --language="java" --command="make" --overwrite`

但是不知道为什么在jdk7上面行不通,如果后面编译成功了的话,再来更新这篇文章





>参考文章
>
>https://blog.csdn.net/m0_37726449/article/details/81226606
>
>https://blog.csdn.net/mole_exp/article/details/122330521

