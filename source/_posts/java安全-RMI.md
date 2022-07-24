---
title: java安全-RMI
date: 2022-03-04 16:40:39
tags:
      - java安全
categories: java安全

---

RMI全称是Remote Method Invocation，远程⽅法调⽤。

从这个名字就可以看出，他的⽬标和RPC其实 是类似的，是让某个Java虚拟机上的对象调⽤另⼀个Java虚拟机中对象上的⽅法，只不过RMI是Java独 有的⼀种机制

要调用远程方法肯定要先注册一个服务器

P牛这里的server类使用了内部类,对于内部类不是很熟悉,先去学习一下内部类再回来看

Server

```java
import java.rmi.Naming;
import java.rmi.Remote;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.server.UnicastRemoteObject;

public class Server {
    public interface IRemoteHello extends Remote{
        public String hello() throws RemoteException;
    }
    public class RemoteHello extends UnicastRemoteObject implements IRemoteHello{

        protected RemoteHello() throws RemoteException {
            super();
        }

        @Override
        public String hello() throws RemoteException {
            System.out.println("call from");
            return "hello world";
        }

    }
    private void start() throws Exception{
        RemoteHello remoteHello = new RemoteHello();
        LocateRegistry.createRegistry(9999);
        Naming.rebind("rmi://127.0.0.1:9999/Hello",remoteHello);
    }
    public static void main(String[] args) throws Exception {
        new Server().start();
    }


}
```

⼀个RMI Server分为三部分： 

1. ⼀个继承了 java.rmi.Remote 的接⼝，其中定义我们要远程调⽤的函数，⽐如这⾥的 hello() 
2. ⼀个实现了此接⼝的类 
3. ⼀个主类，⽤来创建Registry，并将上⾯的类实例化后绑定到⼀个地址。这就是我们所谓的Server 了。 

Client

```java
import java.rmi.Naming;

public class Client {
    public static void main(String[] args) throws Exception {
        Server.IRemoteHello hello= (Server.IRemoteHello) Naming.lookup("rmi://127.0.0.1:9999/Hello");
        String res=hello.hello();
        System.out.println(res);
    }
}

```

首先使用`Naming.lookup`在Registry中寻找名字是Hello的对象,根据`Naming.rebind`的结果是remoteHello对象

虽说执⾏远程⽅法的时候代码是在远程服务器上执⾏的，但实际上我们还是需要知道有哪些⽅法，这时 候接⼝的重要性就体现了，这也是为什么我们前⾯要继承 `Remote` 并将我们需要调⽤的⽅法写在接⼝ `IRemoteHello`⾥，因为客户端也需要⽤到这个接⼝。

> 在这里有一个疑问,为什么在创建hello对象的时候可以直接加上`Server.IRemoteHello`





##### Naming.bind和registy.bind的区别

查看源码后

其实Naming.bind方法是对registy.bind的一种封装,具体到对应的格式和端口

```java
LocateRegistry.getRegistry("127.0.0.1", 8494).bind("R1", 
            UnicastRemoteObject.exportObject(new RemoteObject(), 0));
    
    Naming.bind("rmi://127.0.0.1:8494/R1", 
            UnicastRemoteObject.exportObject(new RemoteObject (), 0));
```





通常在创建RMI Registry的时候,都会直接绑定一个对象在上面

```java
LocateRegistry.createRegistry(9999);
Naming.rebind("rmi://127.0.0.1:9999/Hello",new RemoteHello());
```

第一行创建RMI Registry,第二行将RemoteHello对象绑定到Hello这个路径上面

`Naming.bind `的第一个参数是一个URL，形如：` rmi://host:port/name `。其中，`host`和`port`就是 RMI Registry的地址和端口，`name`是远程对象的名字。

#### 两个关于攻击的问题

1. 如果我们能访问RMI Registry服务，如何对其攻击？ 

我的理解是如果能访问RMI Regittry服务,但也仅仅是能访问,缺少对应的攻击方法,如何在Server中创建恶意对象和攻击方法....

JAVA中对于远程访问RMI Regittry服务做了限制,只有来源是本地的时候才能调用`rebind`,`bind`,`unbind`方法来修改绑定的对象和方法

但是可以通过list方法和lookup可以远程调用,使用Naming.list

```java
String[] s = Naming.list("rmi://127.0.0.1:9999");
//使用该方法可以获取绑定到该端口的远程对象名字数组
Naming.lookup可以调用该方法
```

> 总结
>
> 能访问到RMI Registry服务,需要目标Server上面有比较危险的方法供我们调用才能造成攻击

2. 如果我们控制了目标RMI客户端中 Naming.lookup 的第一个参数（也就是RMI Registry的地 址），能不能进行攻击？

如果能够控制`RMI客户端`中RMI Registry的地址,那么自己就可以在攻击机上面创建对应的恶意方法让其远程调用



### RMI利用codebase执行任意代码

codebase是一个地址，告诉Java虚拟机我们应该从哪个地方去搜索类，有点像我们日常用的 CLASSPATH，但CLASSPATH是本地路径，而codebase通常是远程URL，比如http、ftp等。



如果我们指定 `codebase=http://example.com/` ，然后加载 `org.vulhub.example.Example` 类，则 Java虚拟机会下载这个文件` http://example.com/org/vulhub/example/Example.class `，并作为 Example类的字节码。

>RMI的流程中，客户端和服务端之间传递的是一些序列化后的对象，这些对象在反序列化时，就会去寻 找类。如果某一端反序列化时发现一个对象，那么就会去自己的CLASSPATH下寻找想对应的类；如果在 本地没有找到这个类，就会去远程加载codebase中的类。

在RMI中，我们是可以将codebase随着序列化数据一起传输的，服务器在接收到这个数据后就会去 CLASSPATH和指定的codebase寻找类，由于codebase被控制导致任意命令执行漏洞。



不过显然官方也注意到了这一个安全隐患，所以只有满足如下条件的RMI服务器才能被攻击： 

- 安装并配置了SecurityManager 
- Java版本低于7u21、6u45，或者设置了 java.rmi.server.useCodebaseOnly=false 
- 其中 java.rmi.server.useCodebaseOnly 是在Java 7u21、6u45的时候修改的一个默认设置： 
  - https://docs.oracle.com/javase/7/docs/technotes/guides/rmi/enhancements-7.html 
  - https://www.oracle.com/technetwork/java/javase/7u21-relnotes-1932873.html 

官方将 java.rmi.server.useCodebaseOnly 的默认值由 false 改为了 true 。在java.rmi.server.useCodebaseOnly 配置为` true` 的情况下，**Java虚拟机将只信任预先配置好的 codebase ，不再支持从RMI请求中获取。**

java -Djava.rmi.server.hostname=192.168.135.142 - Djava.rmi.server.useCodebaseOnly=false -Djava.security.policy=client.policy RemoteRMIServer

这个复现出现了问题,自己百度没有解决,等解决了再继续















客户端和服务端都需要定义一个接口继承Remote接口，但是服务端需要实现这个接口，毕竟调用的是服务端的方法，它是实打实存在的

服务端必须继承`UnicastRemoteObject`类
