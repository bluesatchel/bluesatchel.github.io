---
title: SpringMVC入门
date: 2022-01-28 00:14:20
tags:
      - java
      - Spring
      - web
categories: java
---

# SpringMVC入门学习<!--more-->

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

- 导入SpringMVC相关坐标

```xml
<dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.3.15</version>
    </dependency>
```

- web.xml配置SpringMVC核心控制器DispathcerServlet

```xml
<servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-mvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup><!--代表服务器启动时候就加载调度员对象-->
  </servlet>
  <servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

- 创建Controller类和视图页面
- 使用注解配置Controller类中业务方法的映射地址

```java
package com.manage.Controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller

public class UserController {
    //请求地址为/user/quick
    @RequestMapping("/quick")
    public String save(){
        System.out.println("controller save running...");
        return "success.jsp";
    }

}
```

- 配置SpringMVC核心文件spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd">
<!--Controller的组件扫描-->
<context:component-scan base-package="com.manage.Controller"/>
    
</beans>
```

- 客户端发起请求测试

### SpringMVC的执行流程

![image-20220129160149808](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220129160149808.png)

1.用户发请求至前端控制器DispatcherServlet

2.DispatcherServlet收到请求调用HanderMapping处理器映射器

3.处理器引射器找到具体的处理器(可以根据xml配置,注解进行查找),生成处理器对象及处理器拦截器(如果有则生成)并一并返回给DispatcherServlet

4.DispatcherServlet调用HandlerAdapter处理器适配器

5.HandlerAdapter经过适配调用具体的处理器(Controller,也叫后端控制器)

6.Controller执行完成返回ModelAndView

7.HandlerAdapter将controller执行结果ModelAndView返回给DisatcherServlet

8.DispatcherServlet将ModelAndView传给ViewReslover视图解析器

9.ViewReslover解析后返回具体View

10.DispatcherServlet根据View进行渲染视图(即将模型数据填充至视图中),DispatcherServlet响应用户



SpringMVC注解

- @RequestMapping

建立请求URL和处理请求方法之间的对应关系,可以写在类上面,也可以写在方法上面,类的请求路径对于方法属于包含关系

value:用于指定请求的url

method:指定请求的方式

params:用于指定限制请求参数的条件,它支持简单的表达式,要求请求参数的key和value必须和配置的一模一样

```
params={"accountName"},表示请求参数必须有accountName
params={"money!=100"},表示请求参数中money不能是100
```

```java
@Controller
public class UserController {
    @RequestMapping("/quick")
    public String save(){
        System.out.println("controller save running...");
        return "/success.jsp";
        //可以指定转发或者重定向,默认是重定向
        return "forward:/success.jsp"//转发
        return "redirect:/success.jsp"//重定向
    }
}
```

**可以配置视图解析器,让返回的页面可以简写**

#### SpringMVC的数据响应方式

##### 页面跳转

- 直接返回字符串
- 通过ModelAndView对象返回

```java
@RequestMapping("/quick2")
    public ModelAndView save2(){
        /*
        Model:模型 作用封装数据
        View:视图 作用展示数据
        */
        ModelAndView modelAndView=new ModelAndView();
        //设置模型数据
        modelAndView.addObject("username","aaahhahahaha")
        //设置视图名称,这是类似于重定向的
        modelAndView.setViewName("index.jsp");
        return modelAndView;
    }
---------------------------------------------------------
也可以单独返回model或者view
    SpringMVC可以将形参当成实参用,不需要传入参数
     public ModelAndView save2(ModelAndView modelAndView)和上面作用相等,SpringMVC会帮我们创建modelAndView对象,可以直接在方法里面用modelAndView
    
```

##### 回写数据

- 直接返回字符串
  - 使用response中的write方法	

  ```java
  @RequestMapping("/quick3")
      public void save3(HttpServletResponse response) throws IOException {
          response.getWriter().print("hellodsjfdsjfsdfd");
      }
  ```

  - 可以添加@ResponseBody注解告诉SpringMVC框架,方法返回的字符串不是跳转,是直接在http响应体中返回

  ```java
  @RequestMapping("/quick4")
      @ResponseBody
      public String save3() throws IOException {
          return "djfjdsfdsjf";
      }
  ```

- 直接返回json字符串

不管它是json字符串还是啥,本质都是字符串

将对象转换为json字符串需要导入jackson依赖

```xml
	<dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.13.1</version>
    </dependency>
	<dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.13.1</version>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.13.1</version>
    </dependency>
```



```java
@RequestMapping("/quick5")
    @ResponseBody
    public String save4() throws IOException {
        User user=new User();
        user.setName("llllll");
        user.setAge(18);
        //使用json的转换工具将对象转换成json格式
        ObjectMapper objectMapper=new ObjectMapper();
        String json=objectMapper.writeValueAsString(user);
        return json;
    }
```

期间出现了报错

![image-20220129222455211](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220129222455211.png)

原因是需要将原来类中的属性设置为public或者提供get方法

![image-20220129222727835](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220129222727835.png)

- 返回对象或者集合

上面返回json的方法比较麻烦,可以通过配置处理器映射器的方法,自动将返回对象转换为json字符串

spring-mvc.xml

```xml
<!--配置处理器映射器,方便返回json字符串-->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="messageConverters">
            <list>
                <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"></bean>
            </list>
        </property>
    </bean>

```

这样一配置,现在上面的代码就可以简化为,同时还帮忙解决了中文乱码的问题

```java
@RequestMapping("/quick6")
    @ResponseBody
    public User save6() throws IOException {
        User user=new User();
        user.setName("llllll");
        user.setAge(18);
        return user;
    }
```

**经典上面白学**

只需要在spring-mvc.xml配置一个mvc的注解驱动就可以自动完成上面的所有跟jackson有关的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
                          http://www.springframework.org/schema/context  http://www.springframework.org/schema/context/spring-context.xsd">
<!--Controller的组件扫描-->
<context:component-scan base-package="com.manage.Controller"/>
   <!--mvc的注解驱动-->
    <mvc:annotation-driven/>

</beans>
```

### SpringMVC的获得请求参数

#### 获得POJO参数



#### 获得数组类型参数

Controller中的业务方法数组名称与请求参数的name一致,参数会自动映射匹配

```java
请求地址http://localhost:8080/quick7?strings=111&strings=222&strings=333
@RequestMapping("/quick7")
    @ResponseBody
    public void save7(String[] strings) throws IOException {
        System.out.println(Arrays.asList(strings));
    }
```

#### 获得集合类型参数

获得集合参数时,要将集合参数包装到一个POJO中才可以

使用ajax提交时,可以指定contentType为json形式,那么在方法参数位置使用@RequestBody可以直接接收集合数据而无需使用POJO进行包装

### 配置静态资源访问

可以配置静态资源的对应路径和请求路径

```xml
<mvc:resources mapping="/img/**" location="/img/"/>
```

或者交给Tomcat找

```xml
<mvc:default-servlet-handler/>
```

### 配置全局编码过滤器

和之前学习servlet的过滤器一个作用

web.xml

```xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

#### 参数绑定注解@RequestParam

当请求的参数名称与Controller的业务方法参数名称不一致时,就需要通过@RequestParam注解显示的绑定

```
@RequestMapping("/quick8")
    @ResponseBody
    public void save8(@RequestParam("name") String username) throws IOException {
        System.out.println(username);
    }
    http://localhost:8080/quick8?name=张三
```

这样在前端传递参数的时候就可以写name=....,等于同义替换了一下

@RequestParam注解的参数简介

- value:请求参数名称(默认的参数)
- **required**:可以选择该参数在请求中是否必须包含,默认是true,如果此时没有包含则会报错,false则可以不包含该参数发起请求
- defaultValue:当没有指定请求参数时,则使用指定的默认值赋值,就是给前面的name一个默认值

#### 获取Servlet相关API

直接在函数形参中定义形参使用即可,剩下的交给框架

如

```java
@RequestMapping("/quick3")
    public void save3(HttpServletResponse response) throws IOException {
        response.getWriter().print("hellodsjfdsjfsdfd");
    }
```

#### 获取请求头

##### @RequestHeader

使用@RequestHeader注解可以获得请求头信息,相当于web的时候学习的request.getHeader(name)

@RequestHeader属性

- value:请求头名称
- required:是否必须携带此请求头

```java
@RequestMapping("/quick9")
    @ResponseBody
    public void save9(@RequestHeader(value = "User-Agent") String userAgent) throws IOException {
        System.out.println(userAgent);
    }
```

##### @CookieValue

使用@CookieValue可以获得指定Cookie的值

@CookieValue属性

- value:指定Cookie的名称
- required:是否必须携带此Cookie

```java
@RequestMapping("/quick10")
    @ResponseBody
    public void save10(@CookieValue(value="JSESSIONID") String jsessionId) throws IOException {
        System.out.println(jsessionId);
    }
```

#### 文件上传

##### 文件上传客户端三要素

- 表单项type="file"
- 表单的提交方式是post
- 表单的enctype属性是多部分表单形式,即enctype="multipart/from-data"

##### 文件上传原理

- 当from表单修改为多部分表单时,request.getParameter()将失效
- enctype="application/x-www-from-urlencoded"时,from表单的正文内容格式是:key=value&key2=value2
- 当from表单的enctype取值为mutilpart/from-data时,正文内容就变成多部分形式,(数据包中通过横杠来分割)

##### 单文件上传步骤

- 导入fileupload和io依赖

```xml
<dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.4</version>
    </dependency>
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.6</version>
    </dependency>
```



- 配置文件上传解析器

spring-mvc.xml

```xml
<!--配置文件上传解析器-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="defaultEncoding" value="UTF-8"/>
        <property name="maxUploadSize" value="50000"/>
    </bean>
```



- 编写文件上传代码

```java
@RequestMapping("/quick11")
    @ResponseBody
    public void save11(String username, MultipartFile uploadFile) throws IOException{
        String fileName=uploadFile.getOriginalFilename();
        uploadFile.transferTo(new File("C:\\upload\\"+fileName));
    }
```

##### 多文件上传

只需要多添加几个type="file"的表单项,MultipartFile 类型变量名和表单中的name要保持一致

也可以让name都一样,通过同名数组来接收

```html
<form action="http://localhost:8080/quick11" method="post" enctype="multipart/form-data">
    名称<input type="text" name="username">
    文件<input type="file" name="uploadFiles">
    <input type="file" name="uploadFiles">
    <input type="file" name="uploadFiles">
    <input type="submit" value="上传">

</form>
```

```java
@RequestMapping("/quick11")
    @ResponseBody
    public void save11(String username, MultipartFile[] uploadFiles) throws IOException{
        for (MultipartFile uploadFile:uploadFiles
             ) {

            String fileName=uploadFile.getOriginalFilename();
            uploadFile.transferTo(new File("C:\\upload\\"+fileName));
        }
    }
```

