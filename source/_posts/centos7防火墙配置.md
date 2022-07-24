---
title: centos7防火墙配置
date: 2021-01-13 18:25:11
tags:
      - linux
      - centos
      - firwall
categories: linux
---

## CentOS7 防火墙设置

阿里云centos7中默认使用firewall,并且默认没有开启。

使用阿里云服务器,先要在阿里云后台开放端口,然后关闭centos防火墙或者开放centos的对应端口,只开放centos端口,不设置阿里云端口设置,仍然不能访问!firewall配置

#### 重启、关闭、开启firewalld服务

##### 启动

systemctl start firewalld

##### 开机启动

systemctl enable firewalld

##### 关闭

systemctl stop firewalld

##### 取消开机启动

systemctl disable firewalld

#####  重启

service firewalld restart

##### 查看firewall的状态

firewall-cmd --state

##### 查看防火墙规则

firewall-cmd --list-all

##### 用户配置目录

cd /etc/firewalld/命令行方式

##### 常用的几个端口和配置语句

```bash
**# web**

firewall-cmd --zone=public --permanent --add-port=80/tcp

**# mysql**

firewall-cmd --zone=public --permanent --add-port=3306/tcp

**# tomcat**

firewall-cmd --zone=public --permanent --add-port=8080/tcp

**# redis**

firewall-cmd --zone=public --permanent --add-port=6379/tcp



firewall-cmd：是Linux提供的操作firewall的一个工具； –permanent：表示设置为持久； –add-port：标识添加的端口； –zone=public：指定的zone为public；

```

##### 批量添加端口

firewall-cmd --zone=public --permanent --add-port=8081-8200/tcp

##### 生成的配置文件所在路径

vi /etc/firewalld/zones/public.xml

##### 配置完成后重启

service firewalld restart配置文件方式



```xml
<?xml version="1.0" encoding="utf-8"?><zone>

<short>Public</short>
<description>For use in public areas.</description>
<rule family="ipv4">
<source address="122.10.70.234"/>
<port protocol="udp" port="514"/>
<accept/>
</rule>
<rule family="ipv4">
<source address="123.60.255.14"/>
<port protocol="tcp" port="10050-10051"/>
<accept/>
</rule>
<rule family="ipv4">
<source address="192.249.87.114"/>
<port protocol="tcp" port="80"/>
<accept/>
</rule>
<rule family="ipv4">
<port protocol="tcp" port="9527"/>
<accept/>
</rule>
</zone>
```

`上面的对应的规则`

`固定IP 固定协议 的固定端口访问`

`固定IP 固定协议 的范围端口访问`

`固定IP 固定协议 的固定端口访问`

`任意IP 固定协议 的固定端口访问切换为iptables防火墙`

`关闭firewall：`



#### 安装iptables防火墙



##### 安装

`yum install iptables-services`

##### 开启iptables

`service iptables restart`

##### 设置防火墙开机启动

`systemctl enable iptables.service  或者chkconfig iptables on`

##### 

##### 查看已生效的规则

iptables -L -n命令行方式

`iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT`

##### 写入配置文件

`service iptables save`

这样下次重启依旧会生效配置文件方式

##### 编辑iptables防火墙配置

vi /etc/sysconfig/iptables

下边是一个完整的配置文件：



```bash
Firewall configuration written by system-config-firewall

Manual customization of this file is not recommended.


*filter


:INPUT ACCEPT [0:0]:FORWARD ACCEPT [0:0]:OUTPUT ACCEPT [0:0]


-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

-A INPUT -p icmp -j ACCEPT

-A INPUT -i lo -j ACCEPT


-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT

-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT

-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT


-A INPUT -j REJECT --reject-with icmp-host-prohibited

-A FORWARD -j REJECT --reject-with icmp-host-prohibited


COMMIT
```

保存退出

:wq!

