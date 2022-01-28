---
title: SpringMVC入门
date: 2022-01-28 00:14:20
tags:
      - java
      - Spring
      - web
categories: java
---

# SpringMVC入门学习

### SpringMVC开发步骤

在maven中导入需要的依赖(环境是tomcat9)

```xml
<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.3.15</version>
    </dependency>
<!--springMvc依赖,和spring-web版本号需要保持一致-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.3.15</version>
    </dependency>
<!--spring-context依赖,和上面版本保持一致-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.3.15</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    
    <!--servlet依赖-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
    </dependency>
    <!--mysql依赖-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.25</version>
    </dependency>
```

##### 页面跳转

需求:客户端发起请求,服务端接收请求,执行逻辑并进行视图跳转

springMVC开发步骤:

1. 导入SpringMVC相关坐标
2. web.xml配置SpringMVC核心控制器DispathcerServlet
3. 创建Controller类和视图页面
4. 使用注解配置Controller类中业务方法的映射地址
5. 配置SpringMVC核心文件spring-mvc.xml
6. 客户端发起请求测试

