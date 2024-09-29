---
title: java_web
date: 2022-01-11 20:38:27
tags:
      - java_web
categories: java
---

http1.0:客户端与web服务器连接后,只能获得一个web资源,断开连接

http2.0: 客户端与web服务器连接后,可以获得多个web资源

<!--more-->

##### Http响应

响应状态码

200: 请求响应成功

3**: 请求重定向

4**: 找不到资源

5**: 服务器代码错误500 502(网关错误)

##### MAVEN环境配置

M2_HOME   maven目录下的bin目录

MAVEN_HOME  maven目录

path中配置  %MAVEN_HOME%\bin

#### Servlet

##### Servlet简介

要在web.xml中导包

- Servlet就是sun公司开发动态web的一门技术
- Sun在这些API中提供一个接口叫做:Servlet,如果你想开发一个Servlet程序,只需要完成两个小步骤:
  - 编写一个类,实现Servlet接口
  - 把开发好的java类部署到web服务器中

HelloServlet

```java
public class helloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        PrintWriter writer=resp.getWriter();//响应流
        writer.println("hello,Servlet");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

    }
}
```

目录结构

![image-20220113162720716](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113162720716.png)

##### 编写Servlet映射

写的是java程序,但是要通过浏览器访问,而浏览器需要连接web服务器,所以我们需要在web服务中注册我们写的servlet,还需要给它一个浏览器能够访问的路径

![image-20220113162502747](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113162502747.png)

![image-20220113162617543](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113162617543.png)

请求/hello后,会在web.xml中找/hello对应的servlet-name  hello,然后再向上找hello对应的servlet-class

##### servlet原理

本质上就是requset调用servlet中的service抽象类中我们自己实现的方法,再返回给servlet

![image-20220113163642745](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113163642745.png)

##### Mapping问题

- 一个servlet可以指定一个或者多个路径,(本质上都是请求同一个class)

- 一个servlet可以指定通用(默认)映射路径

  - ```xml
    <servlet-mapping>
    	<servlet-name>hello</servlet-name>
        <url-pattern>/hello/*</url-pattern>  <!--请求hello/下的都会请求到hello-->
    </servlet-mapping>
    ```

- 默认请求路径(等于替代掉了index吧....)

  - ```xml
    <servlet-mapping>
    	<servlet-name>hello</servlet-name>
        <url-pattern>/*</url-pattern>  <!--默认请求到hello-->
    </servlet-mapping>
    ```

    

- 可以自定义后缀实现请求映射(类似于正则)

  - ```xml
    <servlet-mapping>
    	<servlet-name>hello</servlet-name>
        <url-pattern>*.abcd</url-pattern>  <!--只要请求后缀有abcd都会映射到hello-->
    </servlet-mapping>
    ```

    

#### ServletContext

web容器在启动的时候,它会为每个web程序都创建一个对应的ServletContext对象,它代表了当前的web应用

##### 共享数据

很像android中页面传参的intent.putExtra和intent.getExtra

- 在一个servlet中保存的数据,可以在另外一个servlet中拿到

有一说一,java万物皆对象的思想确实牛逼,继承了直接就可以用父类方法,不同子类还能互动

![image-20220113173732922](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113173732922.png)

![image-20220113173739121](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113173739121.png)

![image-20220113173749274](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113173749274.png)

![image-20220113173756430](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113173756430.png)

注意点:因为setAttribute写在了helloServlet中的doGet里面,所以要先请求一下/hello,再去请求/getname

记得类型转换为String

##### 获取初始化参数

web.xml中

```xml
<context-param>
    <param-name>url</param-name>
    <param-value>jdbc:mysql//localhost:3306/mybatis</param-value>
</context-param>
```

java类里面

```java
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");
        ServletContext context=this.getServletContext();
        String url = context.getInitParameter("url");
        resp.getWriter().print(url)        
    }
```

请求对应页面会打印出jdbc:mysql//localhost:3306/mybatis

##### 请求转发

```java
public class dispatcher extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context= this.getServletContext();
        context.getRequestDispatcher("/hello").forward(req,resp);
    }
}
```

请求sd就等于请求了/hello,并且浏览器状态码为200,不是3**的重定向

我觉得对于安全性来说很好,A请求B,B可以去请求C的内容,虽然A得到了C的内容,但是A永远只和B交流



![image-20220113222128264](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113222128264.png)

##### 读取资源(配置)文件

Properties

- 在java目录下新建properties
- 在resources目录下新建properties

发现:都被打包到了同一个路径下:classes,俗称这个路径为classpath

需要一个文件流

### 

### HttpServletResponse

web服务器接收到客户端的http请求,针对这个请求,分别创建一个代表请求的HttpServletRequest对象,代表响应的一个HttpServletResponse对象

获取客户端请求过来的参数,找HttpServletRequest

如果要给客户端响应信息,找HttpServletResponse

#### 响应状态码

```java
int SC_CONTINUE = 100;
    int SC_SWITCHING_PROTOCOLS = 101;
    int SC_OK = 200;
    int SC_CREATED = 201;
    int SC_ACCEPTED = 202;
    int SC_NON_AUTHORITATIVE_INFORMATION = 203;
    int SC_NO_CONTENT = 204;
    int SC_RESET_CONTENT = 205;
    int SC_PARTIAL_CONTENT = 206;
    int SC_MULTIPLE_CHOICES = 300;
    int SC_MOVED_PERMANENTLY = 301;
    int SC_MOVED_TEMPORARILY = 302;
    int SC_FOUND = 302;
    int SC_SEE_OTHER = 303;
    int SC_NOT_MODIFIED = 304;
    int SC_USE_PROXY = 305;
    int SC_TEMPORARY_REDIRECT = 307;
    int SC_BAD_REQUEST = 400;
    int SC_UNAUTHORIZED = 401;
    int SC_PAYMENT_REQUIRED = 402;
    int SC_FORBIDDEN = 403;
    int SC_NOT_FOUND = 404;
    int SC_METHOD_NOT_ALLOWED = 405;
    int SC_NOT_ACCEPTABLE = 406;
    int SC_PROXY_AUTHENTICATION_REQUIRED = 407;
    int SC_REQUEST_TIMEOUT = 408;
    int SC_CONFLICT = 409;
    int SC_GONE = 410;
    int SC_LENGTH_REQUIRED = 411;
    int SC_PRECONDITION_FAILED = 412;
    int SC_REQUEST_ENTITY_TOO_LARGE = 413;
    int SC_REQUEST_URI_TOO_LONG = 414;
    int SC_UNSUPPORTED_MEDIA_TYPE = 415;
    int SC_REQUESTED_RANGE_NOT_SATISFIABLE = 416;
    int SC_EXPECTATION_FAILED = 417;
    int SC_INTERNAL_SERVER_ERROR = 500;
    int SC_NOT_IMPLEMENTED = 501;
    int SC_BAD_GATEWAY = 502;
    int SC_SERVICE_UNAVAILABLE = 503;
    int SC_GATEWAY_TIMEOUT = 504;
    int SC_HTTP_VERSION_NOT_SUPPORTED = 505;
```

#### 常见应用

##### 1.向浏览器输出消息

getWriter(),getOutputStream()

![image-20220113224928187](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220113224928187.png)

##### 2.下载文件

1. 要获取下载文件路径
2. 下载文件名是啥
3. 设置想办法让浏览器能够支持下载我们需要的东西
4. 获取下载文件的输入流
5. 创建缓冲区
6. 获取OutputStream对象
7. 将FileOutputStream流写入到buffer缓冲区
8. 使用OutputStream将缓冲区中的数据输出到客户端

```java
package com.blue.servlet;

import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.FileInputStream;
import java.io.IOException;

public class down extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String realPath="F:\\Project\\maven\\maven_web_1\\src\\main\\resources\\img.png";
        //2.获取下载文件名
        String filename=realPath.substring(realPath.lastIndexOf("\\")+1);
        //3.设置头
        /中文文件名的话还要设置编码
        resp.setHeader("Content-Disposition","attachment;filename="+filename);
        //4.获取下载文件的输入流
        FileInputStream in=new FileInputStream(realPath);
        //5.创建缓冲区
        int len=0;
        byte[] buffer=new byte[1024];
        //6.获取OutputStream对象
        ServletOutputStream out=resp.getOutputStream();
        //7.将fileOutputStream流写入到buffer缓冲区,使用OutputStream将缓冲区中的数据输出到客户端
        while (in.read(buffer)>0){
            out.write(buffer,0,len);
        }
        in.close();
        out.close();
    }
}

```

#### 实现网页重定向

不同于servlet的资源请求重定向,response是通过浏览器实现的重定向

`resp.sendRedirect("路径");`

`&{pageContext.request.contextPath}`代表当前项目路径

重定向和转发的区别:

编码:转发307,重定向302

相同点:页面都会跳转

不同点:url变与不变







### HttpServletRequest

通过HttpServletRequest获取客户端的信息

获取前端提交的参数

`req.getParameter("username");`

`getParamaterValues();`返回一个字符数组

#### Cookie

- 一个Cookie只能保存一个信息
- 一个web站点可以给浏览器发送多个cookie,最多存放20个cookie
- cookie大小有限制4kb
- 300个cookie浏览器上限

删除Cookie

- 不设置有效期,关闭浏览器,自动失效
- 设置有效期为0

传递中文记得使用URLEncoder.encode("中文","utf-8");

#### Session

- 服务器会给每一个用户(浏览器)创建一个Session对象
- 一个Session独占一个浏览器,只要浏览器没有关闭,这个Session就存在
- 用户登录之后,整个网站它都可以访问

### JSP

JAVA Server Pages

最大特点:

- 写JSP就像在写HTML
- 区别:
  - HTML静态数据
  - JSP页面可以嵌入JAVA代码,为用户提供动态数据

JSP本质上就是Servlet

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220114211237653.png" alt="image-20220114211237653"  />

##### jsp与对应jsp.java转换关系

只要是java代码就原封不动的输出

如果是HTML代码,就会被转换为:

```
out.write("<xxxxxxx>")
```



#### JSP基础语法

开始前要先导入依赖,[maven依赖在线查询](https://mvnrepository.com/)

```xml
<!--Servlet依赖-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/javax.servlet.jsp/javax.servlet.jsp-api -->
    <!--JSP依赖-->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.3.3</version>
      <scope>provided</scope>
    </dependency>
    <!--JSTL表达式的依赖-->

    <dependency>
      <groupId>org.glassfish.web</groupId>
      <artifactId>jakarta.servlet.jsp.jstl</artifactId>
      <version>2.0.0</version>
    </dependency>
    <!--standard标签库-->
    <!-- https://mvnrepository.com/artifact/taglibs/standard -->
    <dependency>
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>1.1.2</version>
    </dependency>

```

#### 语法

##### 表达式

```jsp
<%=变量或者表达式%>
<%=new java.util.Date()%>

```

可以用${变量名}(EL表达式)代替,更高级,并且会将一些错误默认不显示,比如不存在的变量

##### 脚本片段

```jsp
<%
	java脚本片段
%>
```

##### JSP声明

```jsp
<%!
private int globalVar = 0;    
%>
```

会被编译到jsp生成的java类中,其他的会被生成到对应的java文件中的jspService方法中

##### 在代码中嵌入HTML元素

```jsp
<html>
<body>
<%for (int i=0;i<5;i++){
    %>
    <h1>hello,world <%=i%> </h1>
<%
    }
%>
</body>
</html>
```

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220115211903441.png" alt="image-20220115211903441" style="zoom:80%;" />

### JSP指令

##### 配置错误页面<%@page  args...%>

##### 方法1

![image-20220115215532167](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220115215532167.png)

```jsp
<%@page errorPage="error/500.jsp" %>
```

##### 方法2

修改web.xml

![image-20220115215913478](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220115215913478.png)

##### <%@include file=""%>

#### 九大内置对象

- pageContext     存东西    还可以forward()转发请求
- request    存东西
- Response
- Session    存东西
- Application      (ServletContext)存东西
- config            (ServletConfig)
- out
- page    用来导包
- exception

对象设置的值的作用域和时间比较

```jsp
<%
    pageContext.setAttribute("name1","名字1");//保存的数据只在一个页面中有效(它有个scope参数,可以修改作用域)
    request.setAttribute("name1","名字2");//保存的数据只在一次请求中有效,请求转发会携带这个数据
    session.setAttribute("name1","名字3");//保存的数据只在一次会话中有效,从打开浏览器到关闭浏览器
    application.setAttribute("name1","名字4");//保存的数据只在服务器中有效,从打开服务器到关闭服务器
%>
```

一些应用场景

request:客户端向服务器发送请求,产生的数据,用户看完就没用了,比如,新闻

session:客户端向服务器发送请求,产生的数据,用户看完还有用

application:客户端向服务器发送请求,产生的数据,一个用户用完了,其他用户还可能用,比如:聊天数据

### JSP标签,JSTL标签,EL表达式

需要导入两个包

```xml
<dependency>
      <groupId>org.glassfish.web</groupId>
      <artifactId>jakarta.servlet.jsp.jstl</artifactId>
      <version>2.0.0</version>
    </dependency>
    <!--standard标签库-->
    <!-- https://mvnrepository.com/artifact/taglibs/standard -->
    <dependency>
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>1.1.2</version>
    </dependency>
```

#### EL表达式

```jsp
<%@ page  isELIgnored="false"%>
```

- 获取数据
- 执行运算
- 获取web开发的常用对象

比如获取表单中的数据${param.参数名}

![image-20220116002754237](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220116002754237.png)

- ~~调用java方法~~

#### JSP标签

可以通过标签转发请求并携带参数

```jsp
<%--<jsp:include page="page1.jsp"></jsp:include>--%>
<jsp:forward page="page1.jsp">
    <jsp:param name="user" value="wuhu"/>
    <jsp:param name="pwd" value="***"/>
</jsp:forward>
```

#### JSTL表达式

<label style="background:yellow">JSTL标签库的使用就是为了弥补HTML标签的不足</label>

东西太多了,边用边查就行,使用前记得导包

##### 核心标签(重点掌握一下)

要先在jsp页面中引入JSTL核心标签库

```jsp
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```

##### JSTL标签库使用步骤

- 引入对应的taglib
- 使用其中的方法
- 在Tomcat  lib中也需要引入JSTL的包,否则会报JSTL解析错误(其他包的解析错误也可用同样的方法手动导入到lib目录)

```jsp
<%
    ArrayList<String> students =new ArrayList<>();
    students.add("张三");
    students.add("李四");
    students.add("王五");
    students.add("赵六");
    request.setAttribute("list",students);
%>
<c:forEach var="student" items="${list}">
    <c:out value="${student}"/><br>

</c:forEach>
```

![image-20220116130300211](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220116130300211.png)

在使用jstl的过程中出现了诸多问题,对于notFoundclass这一类的问题的同意解决办法就是手动导入jstl1.2和standard1.1.2包到WEB-INF目录下的lib目录和tomcat的lib目录下,切记,不能2.0的包和1.2的包混着用

对于使用maven阿里云镜像无法导入jstl1.2我表示真坑

#### javaBean

实体类

javaBean有特定的写法

- 必须要有一个无参构造
- 属性必须私有化
- 必须有对应的get/set方法

一般用来和数据库的字段做映射 ORM:

##### ORM: 对象关系映射

- 表---->类
- 字段----->属性
- 行记录----->对象

### MVC三层架构

Model  View  Controller 模型,视图,控制器

![image-20220116213212850](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220116213212850.png)

Model

- 业务处理:业务逻辑(service)
- 数据持久层: CRUD (Dao)

View

- 展示数据
- 提供链接发起Servlet请求

Controller

- 接收用户的请求(req:请求参数,Session信息...)
- 交给业务层处理对应的代码
- 控制视图的跳转

```
登录--->接收用户的登录请求--->处理用户的请求(获取用户登录的参数)--->交给业务层处理登录业务(判断用户名密码是否正确)--->Dao层查询用户名和密码是否正确
```

### Filter

Filter过滤器,用来过滤网站的数据:

web服务有一些垃圾请求,后台不应该处理,或者应该直接报错

- 处理中文乱码
- 登录验证......

Filter开发步骤

1.导包

2.编写过滤器

- 导包别错,是servlet下面的filter
- 配置好web.xml,并且注意要过滤的路径



例子

web.xml中配置的内容,和servlet配置很像,`<url-pattern>/*</url-pattern>`表示要过滤的部分,和servlet的urlpattern相对应,属于包含关系了

```xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>CharacterEncodingFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <!--只要是经过/的请求,都会经过这个过滤器-->
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

CharacterEncodingFilter.java

```java
import javax.servlet.*;
import java.io.IOException;

public class CharacterEncodingFilter implements Filter {
    @Override
    //初始化
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    //filterChain:链的意思,一边出一边进,这样就可以使用多个过滤器串联起来
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setCharacterEncoding("utf-8");
        servletResponse.setContentType("text/html;charset=UTF-8");
        filterChain.doFilter(servletRequest,servletResponse);//让请求继续走,不写程序就会在这里停止
    }

    @Override
    public void destroy() {

    }
}

```

![image-20220116223839656](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220116223839656.png)

没过滤之前乱码

![image-20220116223824121](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220116223824121.png)

过滤后

![image-20220116223932653](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220116223932653.png)

### 监听器

实现一个监听器的接口,有很多种,具体具体实现,和filter设置

1.编写一个监听器

- 实现监听器接口

2.web.xml中配置监听器

3.看情况是否使用

### 过滤器应用

比如是否已经登录成功的校验,这样在访问网站需要登录才能访问的资源时,会经过一次loginFilter来校验,并决定是否允许其访问,这样在每次访问对应页面时候,就不用重复写校验是否已经登录的代码了

```java
public class loginFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //servletrequest拿不到session,做类型转换,转换成它爹httpservletrequest,response同理
        HttpServletRequest request=(HttpServletRequest) servletRequest;
        HttpServletResponse response=(HttpServletResponse) servletResponse;
        if(request.getSession().getAttribute("USER_SESSION")==null){
            response.sendRedirect("/error.jsp");

        }else{
            filterChain.doFilter(request,response);//已经登录则将请求传递下去给
        }
    }
    @Override
    public void destroy() {
    }
}
```



