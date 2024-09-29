---
title: java安全-反射-学习
date: 2022-03-01 22:03:55
tags:
      - java
      - java安全
categories: java安全
---

## java安全,反射学习

加入了p神的知识星球之后,通过p神发的java安全漫谈学习Java安全,p神讲解很好,恶补了java反射有关知识后,将知识点和代码自己边写边理解一遍

---

java中反射赋予了java动态的特性

Class.forName()可以通过类的路径获取到一个类

```java
Class.forName(className)  //这个可以理解为下一个的封装
//等于
Class.forName(className,true,currentLoader)
    //第一个参数是类名,第二个参数表示是否初始化,第三个参数是ClassLoader
```

Java默认的 ClassLoader 就是根据类名来加载类， 这个类名是类完整路径，如 java.lang.Runtime 



#### 类中"初始化方法"的调用顺序与区别

- `static{}`静态代码块在"类初始化"的时候调用

- `{}`代码块的内容会在父类构造函数`super()`之后调用

- 最后调用构造函数

还记得学java的时候说super必须写在构造函数的第一行

---

**Person p = new Person("zhangsan",20); 该句话都做了什么事情？ **

1，因为new用到了Person.class.所以会先找到Person.class文件并加载到内存中。 

2，执行该类中的static代码块，如果有的话，给Person.class类进行初始化。

3，在堆内存中开辟空间，分配内存地址。 

4，在堆内存中建立对象的特有属性。并进行默认初始化。 

5，对属性进行显示初始化。 

6，对对象进行构造代码块初始化。 

7，对对象进行对应的构造函数初始化。 

8，将内存地址付给栈内存中的p变量。

---

```java
public class TrainPrint {
    {
        System.out.println("类内代码块");
    }
    static{
        System.out.println("静态代码块");
    }
    public TrainPrint(){
        System.out.println("无参构造方法");
    }
}
```

在获取类对象的时候静态代码块就会执行

```java
@Test
    public void init(){

        try {
            //需要抛出一个类找不到的异常
            Class<?> aClass = Class.forName("com.reflect.test.TrainPrint");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```

![image-20220301221808194](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220301221808194.png)



**注:如果在forName获取Class对象的时候将是否初始化关闭(也就是 第二个参数 为 `false`),则不会执行静态代码块**



![image-20220307130344019](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220307130344019.png)

剩下两个"初始化方法"必须在实例化的时候才会调用

```java
@Test
    public void init(){

        try {
            //需要抛出一个类找不到的异常
            Class<?> aClass = Class.forName("com.reflect.test.TrainPrint");
            Object o = aClass.newInstance();

        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }
    }
```

运行后的顺序

![image-20220301222222222](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220301222222222.png)

#### 实践

综上所述,其中静态代码块可以写入恶意代码从而在"类初始化"的时候调用

```java
public class exeCommand {
    static{
        Runtime runtime = Runtime.getRuntime();
        try {
            runtime.exec("cmd /C calc");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

通过加载该类就可以执行`static{}`中的代码

```java
@Test
    public void exe(){
        try {
            Class<?> aClass = Class.forName("com.reflect.test.exeCommand");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```

### 调用类中的方法执行恶意代码

在正常情况下，除了系统类，如果我们想拿到一个类，需要先 import 才能使用。而使用forName就不 需要，这样对于我们的攻击者来说就十分有利，我们可以加载任意类。

ava的普通类 C1 中支持编写内部类 C2 ，而在编译的时候，会生成两个文件： C1.class 和 C1$C2.class ，我们可以把他们看作两个无关的类，通过 Class.forName("C1$C2") 即可加载这个内 部类。 

获得类以后，我们可以继续使用反射来获取这个类中的属性、方法，也可以实例化这个类，并调用方 法。

`class.newInstance()` 的作用就是调用这个类的无参构造函数，这个比较好理解。不过，我们有时候 在写漏洞利用方法的时候，会发现使用 `newInstance` 总是不成功，这时候原因可能是：

- 类没有无参构造函数
- 类的构造函数是私有的

```java
@Test
    public void runtime(){
        try {
            Class<?> aClass = Class.forName("java.lang.Runtime");
            aClass.getMethod("exec",String.class).invoke(aClass.newInstance(),"cmd /C whoami");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
    }
```

会报一个这样的错

![image-20220302002903587](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220302002903587.png)

为什么会将构造方法设置为private呢

很常见的一个设计模式就是"单例模式"

比如对于web应用来说,数据库只需要建立一次连接,而不是每次用到数据库的时候再建立一个连接,此时就可以将数据库连接使用的类的构造方法设置为私有,再通过一个static方法来获取

```java
public class connection {
    private static connection instance=new connection();
    public static connection getInstance(){
        return instance;
    }
    private connection(){
        System.out.println("数据库连接建立");
    }
}
```

这样只有类初始化的时候会执行一个构造函数,后面只能通过`getInstance`获取这个对象,避免建立多个数据库连接



Runtime类就是"单例模式",只能通过`Runtime.getRuntime()`来获取到`Runtime`对象

对代码做出修改之后成功执行命令

其中`aClass.getMethod("getRuntime").invoke(aClass)`,用来获取Runtime对象

```java
try {
            Class<?> aClass = Class.forName("java.lang.Runtime");
            aClass.getMethod("exec",String.class).invoke(aClass.getMethod("getRuntime").invoke(aClass),"calc");
        } catch (ClassNotFoundException | NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
```

### 两个问题

#### 如果一个类没有无参构造方法,也没有类似单例模式里的静态方法，我们怎样通过反射实例化该类呢？

需要使用到getConstructor()方法获取无参构造函数,因为构造函数也支持重载,所以必须用参数列表类型才能唯一确定一个构造函数

比如经常使用的另一种执行命令的类ProcessBuilder,使用反射来获取其有参的构造函数,然后调用`start()`来执行命令

```java
@Test
    public void processBuilder() throws Exception{
        Class<?> aClass = Class.forName("java.lang.ProcessBuilder");
        ((ProcessBuilder)aClass.getConstructor(List.class).newInstance(Arrays.asList("calc"))).start();
    }
```

如果不能强制类型转换的时候,利用反射的getMethod和invoke来执行

通过 getMethod("start") 获取到start方法，然后 invoke 执行， invoke 的第一个参数就是 ProcessBuilder Object了。

```java
aClass.getMethod("start").invoke(aClass.getConstructor(List.class).newInstance(Arrays.asList("calc")));
```

那么，如果我们要使用 `public ProcessBuilder(String... command) `这个构造函数，需要怎样用反 射执行呢？

这个`String... command`参数表示可变长参数,在Java中编译的时候会编译成一个数组

```java
Class clazz = Class.forName("java.lang.ProcessBuilder");
((ProcessBuilder)clazz.getConstructor(String[].class).newInstance(new String[][]{{"calc.exe"}})).start();

```



```java
aClass.getMethod("start").invoke(aClass.getConstructor(String[].class).newInstance(new String[][]{{"calc"}}));
```



#### 如果一个方法或构造方法是私有方法，我们是否能执行它呢？

此时就需要getDeclared系列的反射了

- getMethod 系列方法获取的是当前类中所有公共方法，包括从父类继承的方法 

- getDeclaredMethod 系列方法获取的是当前类中“声明”的方法，是实在写在这个类里的，包括私 有的方法，但从父类里继承来的就不包含了

之前提到过,Runtime的构造方法时私有的,现在可以通过getDeclaredConstructor来获取这个私有的构造方法,使用setAccessible(true)来设置私有方法的作用域来调用它

```java
@Test
    public void declar() throws Exception{
        Class<?> aClass = Class.forName("java.lang.Runtime");
        Constructor<?> declaredConstructor = aClass.getDeclaredConstructor();
        declaredConstructor.setAccessible(true);
        aClass.getMethod("exec",String.class).invoke(declaredConstructor.newInstance(),"calc");

    }
```





