---
title: javaWeb项目搭建(subms)
date: 2022-01-17 17:06:18
tags:
      - javaWeb
      - java
categories: java
---

# 项目搭建--javaWeb

#### 1.配置一个maven webapp项目

#### 2.配置tomcat

#### 3.测试项目是否能够跑起来

#### 4.导入项目中可能会遇到的jar包

servlet,jsp,mysql-connector,standard....,记得maven同步一下

#### 5.创建项目包结构

![image-20220117181550313](https://gitee.com/blue_satchel/images/raw/master/image-20220117181550313.png)

#### 6.编写实体类

ORM映射: 表---类映射

#### 7.编写基础公共类

##### 数据库配置文件

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/subms?useUnicode&characterEncoding=utf-8
username=root
password=root
```

##### 编写数据库的公共类   BaseDao.java

```java
package com.blue.dao;

import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

public class BaseDao {
    private static String driver;
    private static String url;
    private static String username;
    private static String password;

    //静态代码块,类加载的时候就初始化了
    static {
        Properties properties =new Properties();
        //通过类加载器读取对应的资源
        InputStream is=BaseDao.class.getClassLoader().getResourceAsStream("db.properties");
        try{
            properties.load(is);
        }catch (IOException e){
            e.printStackTrace();
        }
        driver=properties.getProperty("driver");
        driver=properties.getProperty("username");
        driver=properties.getProperty("password");
    }
    public static Connection getConnection(){
        Connection connection = null;
        try{
            Class.forName(driver);
            connection= DriverManager.getConnection(url,username,password);
        }catch (Exception e){
            e.printStackTrace();
        }
        return connection;
    }
    public static ResultSet execute(Connection connection,String sql,Object[] params,ResultSet resultSet) throws SQLException {
        PreparedStatement preparedStatement=connection.prepareStatement(sql);
        for(int i=1;i<params.length;i++){
            //setObject方法不能从1开始,但是数组是从0开始
            preparedStatement.setObject(i+1,params[i]);
        }
        resultSet = preparedStatement.executeQuery(sql);
        return resultSet;
    }
    public static int execute(Connection connection,String sql,Object[] params,PreparedStatement preparedStatement) throws SQLException{
        preparedStatement=connection.prepareStatement(sql);
        for(int i=0;i<params.length;i++){
            preparedStatement.setObject(i+1,params[i]);
        }
        int updateRows=preparedStatement.executeUpdate();
        return updateRows;
    }
    //释放资源
    public static boolean closeResource(Connection connection,PreparedStatement preparedStatement,ResultSet resultSet){
        boolean flag=true;
        if(resultSet!=null){
            try{
                resultSet.close();
                resultSet=null;
            }catch (SQLException e){
                e.printStackTrace();
                flag=false;
            }
        }
        if(preparedStatement!=null){
            try{
                preparedStatement.close();
                preparedStatement=null;
            }catch (SQLException e){
                e.printStackTrace();
                flag=false;
            }
        }
        if(connection!=null){
            try{
                connection.close();
                connection=null;
            }catch (SQLException e){
                e.printStackTrace();
                flag=false;
            }
        }
        return flag;
    }

}

```

##### 配置字符编码过滤器

```xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>com.blue.filter.CharactorEncodingFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <!--过滤所有请求-->
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

```java
package com.blue.filter;
import javax.servlet.*;
import java.io.IOException;

public class CharactorEncodingFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }
    @Override
    //filterChain:链的意思,一边出一边进,这样就可以使用多个过滤器串联起来
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setCharacterEncoding("utf-8");
        servletResponse.setContentType("text/html;charset=UTF-8");
        filterChain.doFilter(servletRequest, servletResponse);//让请求继续走,不写程序就会在这里停止
    }
    @Override
    public void destroy() {

    }
}
```

