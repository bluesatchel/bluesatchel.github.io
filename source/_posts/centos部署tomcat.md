---
title: centos部署tomcat
date: 2022-01-18 19:15:09
tags:
      - tomcat部署
      - linux
      - centos
categories: linux
---

查看系统的位数,通过该命令就能查看

`getconf LONG_BIT`

![image-20220119161624050](https://gitee.com/blue_satchel/images/raw/master/image-20220119161624050.png)

#### 安装JDK

下载jdk对应安装包linuxx64.tar.gz的

通过ssh工具上传到/usr/local下面

解压

`tar -zxvf jdk-linux-x64.tar.gz `

重命名,方便等会环境变量的配置

`mv jdk1.8.0_131/ jdk1.8`

#### 配置环境变量

`vim /etc/profile`

##### 编辑profile文件

在文件末尾添加

```properties
export JAVA_HOME=/usr/local/jdk1.8/

export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export PATH=$PATH:$JAVA_HOME/bin

```

##### 通过source命令重新加载profile文件

`source /etc/profile`

##### 检查是否安装成功

`java -version`

出现如下显示,表示安装成功

![image-20220119173528732](https://gitee.com/blue_satchel/images/raw/master/image-20220119173528732.png)

#### 安装tomcat

在usr/local目录下

`mkdir myDir`

进入myDir,上传tomcat包

解压

`tar -zxvf apache-tomcat-9.0.56.tar.gz`

重命名

`mv apache-tomcat-9.0.56 tomcat9`

开放端口,可以参考[这里](https://www.bluesatchel.space/2021/01/13/centos7%E9%98%B2%E7%81%AB%E5%A2%99%E9%85%8D%E7%BD%AE/)

`firewall-cmd --zone=public --add-port=3306/tcp --permanent`

重启防火墙,这点很重要!!!!

还要在对应云服务提供商的控制台去修改防火墙策略添加对应端口

修改./conf/server.xml配置站点

修改端口

![image-20220119175609053](https://gitee.com/blue_satchel/images/raw/master/image-20220119175609053.png)

进入tomcat/bin目录下启动tomcat服务

`./startup.sh `

但是出现了无法访问的情况

查看端口开放情况

`netstat -tlun`,3306也是开放的

![image-20220119180956606](https://gitee.com/blue_satchel/images/raw/master/image-20220119180956606.png)

查看防火墙端口开放情况

`/etc/firewalld/zones/public.xml`

也是开放的

![image-20220119181250440](https://gitee.com/blue_satchel/images/raw/master/image-20220119181250440.png)

使用`lsof -i:3306`,查看3306占用情况,才发现自己忘记了Mysql在3306运行,把端口改成3333就行了,确实蠢驴一个呜呜呜

<img src="https://gitee.com/blue_satchel/images/raw/master/tomcat_emoji.jpg" alt="tomcat_emoji" style="zoom: 33%;" />

##### 访问管理页面managerApp

根据提示,默认情况下管理页面只能允许同一台主机本地访问

配置conf/tomcat-users.xml

在tomcat-users下添加

```xml
<role rolename="manager-gui"/>
<user password="admin" roles="manager-gui" username="tomcat"/>
```

打开**/webapps/manager/META-INF/目录下context.xml文件**

若不加Value元素默认不限制ip

若不加allow默认限制所有ip访问

这里为了方便,把value注释掉

![image-20220119184223964](https://gitee.com/blue_satchel/images/raw/master/image-20220119184223964.png)

现在访问管理页面,按照上面role的账号和密码输入就能访问了
