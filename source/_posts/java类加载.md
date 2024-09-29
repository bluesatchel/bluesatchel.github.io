---
title: java类加载
date: 2022-03-23 21:31:52
tags:
      - java
categories: java
---

### 什么是java的"字节码"

<!--more-->

严格来说，Java字节码（ByteCode）其实仅仅指的是Java虚拟机执行使用的一类指令，通常被存储 在.class文件中。 

众所周知，不同平台、不同CPU的计算机指令有差异，但因为Java是一门跨平台的编译型语言，所以这 些差异对于上层开发者来说是透明的，上层开发者只需要将自己的代码编译一次，即可运行在不同平台 的JVM虚拟机中。

 甚至，开发者可以用类似Scala、Kotlin这样的语言编写代码，只要你的编译器能够将代码编译成.class文 件，都可以在JVM虚拟机中运行：

![image-20220323213706173](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220323213706173.png)

通常所说的“字节码”，可以理解的更广义一些——所有能够恢复成一个类并在JVM虚拟机里加 载的字节序列

### 利用URLClassLoader加载远程class文件

`URLClassLoader `实际上是我们平时默认使用的 `AppClassLoader `的父类，所以，我们解释 `URLClassLoader` 的工作过程实际上就是在解释默认的Java类加载器的工作流程。

正常情况下，Java会根据配置项 sun.boot.class.path 和 java.class.path 中列举到的基础路径（这 些路径是经过处理后的 java.net.URL 类）来寻找.class文件来加载，而这个基础路径有分为三种情况： 

- URL未以斜杠 / 结尾，则认为是一个JAR文件，使用 JarLoader 来寻找类，即为在Jar包中寻 找.class文件 

- URL以斜杠 / 结尾，且协议名是 file ，则使用 FileLoader 来寻找类，即为在本地文件系统中寻 找.class文件 

- URL以斜杠 / 结尾，且协议名不是 file ，则使用最基础的 Loader 来寻找类 

  

我们正常开发的时候通常遇到的是前两者，那什么时候才会出现使用 Loader 寻找类的情况呢？当然是 非 file 协议的情况下，最常见的就是 http 协议。

使用python搭建一个简易Http服务器,将class文件放在服务器上面,尝试加载

`python3 -m http.server 5555`

```java
public static void main(String[] args) throws Exception{

        URLClassLoader urlClassLoader=new URLClassLoader(new URL[]{new URL("http://xxxxxxxxxxx:5555/")});
        Class<?> test = urlClassLoader.loadClass("test");
        test.newInstance();

    }
```

所以，作为攻击者，如果我们能够控制目标Java ClassLoader的基础路径为一个http服务器，则可以利 用远程加载的方式执行任意代码了。



### 利用ClassLoader#defineClass直接加载字节码

其实,不管是远程加载class文件还是本地的class或者Jar文件,经历的欧式下面这三个方法的调用

`loadClass`--->`findClass`--->`defineClass`

- 其中： loadClass 的作用是从已加载的类缓存、父加载器等位置寻找类（这里实际上是双亲委派机 制），在前面没有找到的情况下，执行 

- findClass findClass 的作用是根据基础URL指定的方式来加载类的字节码，就像上一节中说到的，可能会在 本地文件系统、jar包或远程http服务器上读取字节码，然后交给 defineClass 

- defineClass 的作用是处理前面传入的字节码，将其处理成真正的Java类

所以可见，真正核心的部分其实是 defineClass ，他决定了如何将一段字节流转变成一个Java类，Java 默认的 `ClassLoader#defineClass` 是一个native方法，逻辑在JVM的C语言代码中。\

注意一点，在 defineClass 被调用的时候，类对象是不会被初始化的，只有这个对象显式地调用其构造 函数，初始化代码才能被执行。

在实际场景中，因为defineClass方法作用域是不开放的，所以攻击者很少能直接利用到它，但它却是我 们常用的一个攻击链 `TemplatesImpl` 的基石。

##### 自定义ClassLoader

```java
package demo1;

import java.lang.reflect.Method;

public class TestClassLoader extends ClassLoader{
    private static String TestName="demo1.HelloWorld";
    private static byte[] testClassBytes = new byte[]{
            -54, -2, -70, -66, 0, 0, 0, 51, 0, 17, 10, 0, 4, 0, 13, 8, 0, 14, 7, 0, 15, 7, 0,
            16, 1, 0, 6, 60, 105, 110, 105, 116, 62, 1, 0, 3, 40, 41, 86, 1, 0, 4, 67, 111, 100,
            101, 1, 0, 15, 76, 105, 110, 101, 78, 117, 109, 98, 101, 114, 84, 97, 98, 108, 101,
            1, 0, 5, 104, 101, 108, 108, 111, 1, 0, 20, 40, 41, 76, 106, 97, 118, 97, 47, 108,
            97, 110, 103, 47, 83, 116, 114, 105, 110, 103, 59, 1, 0, 10, 83, 111, 117, 114, 99,
            101, 70, 105, 108, 101, 1, 0, 19, 84, 101, 115, 116, 72, 101, 108, 108, 111, 87, 111,
            114, 108, 100, 46, 106, 97, 118, 97, 12, 0, 5, 0, 6, 1, 0, 12, 72, 101, 108, 108, 111,
            32, 87, 111, 114, 108, 100, 126, 1, 0, 40, 99, 111, 109, 47, 97, 110, 98, 97, 105, 47,
            115, 101, 99, 47, 99, 108, 97, 115, 115, 108, 111, 97, 100, 101, 114, 47, 84, 101, 115,
            116, 72, 101, 108, 108, 111, 87, 111, 114, 108, 100, 1, 0, 16, 106, 97, 118, 97, 47, 108,
            97, 110, 103, 47, 79, 98, 106, 101, 99, 116, 0, 33, 0, 3, 0, 4, 0, 0, 0, 0, 0, 2, 0, 1,
            0, 5, 0, 6, 0, 1, 0, 7, 0, 0, 0, 29, 0, 1, 0, 1, 0, 0, 0, 5, 42, -73, 0, 1, -79, 0, 0, 0,
            1, 0, 8, 0, 0, 0, 6, 0, 1, 0, 0, 0, 7, 0, 1, 0, 9, 0, 10, 0, 1, 0, 7, 0, 0, 0, 27, 0, 1,
            0, 1, 0, 0, 0, 3, 18, 2, -80, 0, 0, 0, 1, 0, 8, 0, 0, 0, 6, 0, 1, 0, 0, 0, 10, 0, 1, 0, 11,
            0, 0, 0, 2, 0, 12
    };
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        if(name.equals(TestName))
            return defineClass(TestName,testClassBytes,0,testClassBytes.length);
        return super.findClass(name);
    }
    public static void main(String[] args) throws Exception{
        TestClassLoader loader=new TestClassLoader();
        Class<?> testClass = loader.loadClass(TestName);
        //虽然在自定义classLoader中加载的是testClassBytes中的数据,但是在这里获取到Class后,在newInstance的时候用的是当前包下面对应的java文件中的class
        Object newInstance = testClass.newInstance();
        Method hello = newInstance.getClass().getMethod("hello");
        String str=(String)hello.invoke(newInstance);
        System.out.println(str);
    }
}

```

字节数组中存的数据对应的类代码:

![image-20220423154839887](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220423154839887.png)



当前对应包名下面真正的类代码

![image-20220423154723513](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220423154723513.png)

newInstance出来对应的是后者,变量名也都存在,这里存在被修改的可能



同时加载的两个类是否相等,不仅仅由类的路径来决定,还需要是同一个类加载器加载的才能被判断为相等

比如有两个ClassLoader A和B

`ClassLoader A和ClassLoader B可以加载相同类名的类，但是ClassLoader A中的Class A和ClassLoader B中的Class A是完全不同的对象，两者之间调用只能通过反射`。







### 利用BCEL ClassLoader加载字节码

BCEL的全名应该是Apache Commons BCEL，属于Apache Commons项目下的一个子项目，但其因为 被Apache Xalan所使用，而Apache Xalan又是Java内部对于JAXP的实现，所以BCEL也被包含在了JDK的 原生库中。



我们可以通过BCEL提供的两个类` Repository` 和` Utility `来利用： `Repository `用于将一个Java Class 先转换成原生字节码，当然这里也可以直接使用javac命令来编译java文件生成字节码； Utility 用于将 原生的字节码转换成BCEL格式的字节码：

BCEL ClassLoader在Fastjson等漏洞的利用链构造时都有被用到，其实这个类和前面的` TemplatesImpl` 都出自于同一个第三方库，Apache Xalan。但是由于各种原因，在Java 8u251的更新中，这个ClassLoader被移除了.



待续.....
