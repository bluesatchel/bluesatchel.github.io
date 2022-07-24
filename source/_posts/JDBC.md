---
title: JDBC
date: 2022-01-17 15:13:34
tags:
      - java
      - 数据库
categories: java
---

## JDBC	

java database connectivity

JDBC保证了多种数据库不同操作的统一,(没有什么是加一层无法解决的哈哈哈)

<img src="https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220117151840559.png" alt="image-20220117151840559" style="zoom:80%;" />

需要jar包的支持:用maven导包失败了,最后手动导入了mysql的jar包

![image-20220117154728636](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220117154728636.png)



在IDEA中连接mysql数据库的时候记得设定时区timeZone为ShangHai

##### 简单尝试

![image-20220117155711276](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220117155711276.png)

```java
package com.blue.test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class testJdbc {
    public static void main(String[] args) throws Exception{
        //配置信息
        //useUnicode=true&characterEncoding=utf-8解决中文乱码
        String url="jdbc:mysql://localhost:3306/jdbc?useUnicode=true&characterEncoding=utf-8";
        String username="root";
        String password="root";
        //1.加载驱动,(通过反射)
        Class.forName("com.mysql.jdbc.Driver");
        //2.连接数据库,代表数据表
        Connection connection = DriverManager.getConnection(url,username,password);
        //3.向数据库发送SQL的对象Statement
        //此处可以使用PreparedStatement预编译,防止sql注入
        Statement statement= connection.createStatement();
        //4.编写sql
        String sql=("select * from users");
        //5.执行sql,execute的选择根据要执行的操作而异
        //增删改都使用executeUpdate(sql)
        ResultSet resultSet = statement.executeQuery(sql);//返回一个resultSet
        while (resultSet.next()){
            System.out.println("id="+resultSet.getObject("id"));
            System.out.println("name="+resultSet.getObject("name"));
            System.out.println("password="+resultSet.getObject("password"));
            System.out.println("email="+resultSet.getObject("email"));
            System.out.println("birthday="+resultSet.getObject("birthday"));
        }
        //6.关闭连接,释放资源(先开后关)
        resultSet.close();
        statement.close();
        connection.close();
    }

}

```

![image-20220117155700032](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220117155700032.png)

##### 预编译

可以防止sql注入

```java
String sql="insert into users(id,name,password,email,birthday) values (?,?,?,?,?)";
        PreparedStatement preparedStatement=connection.prepareStatement(sql);
        preparedStatement.setInt(1,1);
        preparedStatement.setString(2,"张三");
		.........................................
        preparedStatement.executeUpdate(sql);
```

#### 事务

要么都成功,要么都失败

ACID原则:保证数据的安全

开启事务

事务提交	commit()

事务回滚	rollback()

关闭事务

例子:

```
转账:A和B

如果A转账100给B,那么如果A-100和B+100就要绑定成一个事务,防止数据出现问题
start transaction ;#开启事务
update account set money=money-100 where name='A';
update account set money=money+100 where name='B';
commit ;
```

```java
package com.blue.test;
import org.junit.Test;

import java.sql.Connection;
import java.sql.DriverManager;

public class test {

    Connection connection=null;
    @Test
    public void test(){
        //配置信息
        //useUnicode=true&characterEncoding=utf-8解决中文乱码
        String url = "jdbc:mysql://localhost:3306/jdbc?useUnicode=true&characterEncoding=utf-8";
        String username = "root";
        String password = "root";
        try {
            //1.加载驱动,通过反射
            Class.forName("com.mysql.jdbc.Driver");
            //2.连接数据库,代表数据表
            connection = DriverManager.getConnection(url, username, password);
            //3.通知数据库开启事务,false 开启
            connection.setAutoCommit(false);
            String sql1 = "update account set money=money-100 where name='A'";
            connection.prepareStatement(sql1).executeUpdate();
            //制造错误
            int i = 1 / 0;
            String sql2 = "update account set money=money+100 where name='B'";
            connection.prepareStatement(sql2).executeUpdate();
            //上面两条都执行成功了,才能提交事务
            connection.commit();
            connection.close();
        }catch (Exception e){
            try{
                //如果出现异常,就通知数据库回滚事务
                connection.rollback();
            }catch (Exception ex){
                ex.printStackTrace();
            }
            e.printStackTrace();
        }
    }
}

```



#### Junit单元测试

依赖,pom.xml

```xml
<dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
</dependency>
```

##### 简单使用

@Test注解只在方法上有效,只要加了个这个注解的方法,就可以直接运行,不需要写main了

```java
package com.blue.test;
import org.junit.Test;
public class test {
    @Test
    public void test(){
        System.out.println(123);
    }
}
```

