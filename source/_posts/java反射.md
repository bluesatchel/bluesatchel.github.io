---
title: java反射
date: 2022-01-11 13:02:55
tags:
      - java
      - reflection
categories: java
---

先认识一下动态和静态语言<!--more-->

**反射的一个特点:破坏封装**

#### 动态语言

是一类在运行时可以改变其结构的语言:例如新的函数,对象,甚至代码可以被引进,已有的函数可以被删除或是其他结构上的变化,通俗点说就是在运行时代码可以 根据某些条件改变自身结构

主要动态语言:c#,javaScript,PHP,Python等

#### 静态语言

与动态语言相对应的,运行时结构不可改变的语言就是静态语言,不如java,c,c++

java不是动态语言,但是java可以称之为"准动态语言",即java有一定的动态性,我们可以<label style="background:yellow">利用反射机制获得类似动态语言的特性</label>,java的动态性让编程的时候更加灵活

## java反射

**通俗讲就是可以通过对象找到它的类**

reflection是java被视为动态语言的关键,反射机制允许程序在执行期间借助于Reflection API取得任何类的内部信息,并能直接操作任意对象的内部属性及方法

`Class c=Class.forName("java.lang.String")`

加载完类之后,在堆内存的方法区中就产生了一个Class类型的对象(一个类只有一个Class对象),这个对象就<label style="background:yellow">包含了完整的类的结构信息</label>,我们可以通过这个对象看到类的结构,这个对象就像一面镜子,透过这个镜子看到类的结构,所以,它被形象的称之为反射

正常方式:    `引入需要的"包类"名称`--->`通过new实例化`--->`取得实例化对象`

反射方式:    `实例化对象`----->`getClass()方法`----->`得到完整的"包类"名称`

#### 反射相关的主要API

- java.lang.Class  代表一个类
- java.lang.reflect.Method  代表类的方法
- java.lang.reflect.Field  代表类的成员变量
- java.lang.reflect.Constructor  代表类的构造器
- java.lang.reflect.Annotation 类表示注解
- ...............

![image-20220228144001904](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220228144001904.png)

#### Class类

对于每个类而言,JRE都为其保留了一个不变的Class类型的对象,一个Class对象包含了特定某个结构的所有信息

- Class本身也是一个类
- Class对象只能由系统建立对象
- 一个加载的类在JVM中只会有一个Class实例
- 一个Class对象对应的是一个加载到JVM中的一个class文件
- 每个类的实例都会记得自己是由哪个Class实例所生成
- 通过Class可以完整地得到一个类中的所有被夹在的结构
- Class类是Reflection的根源,针对任何你想动态加载,运行的类,唯有先获得相应的Class对象

#### 获取Class类的方法

![image-20220111164508670](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220111164508670.png)

此时hashCode相等,说明三种方法获得的都是同一个class

还有一个方式四,但是只能获得基本内置类型

```java
Class c4 = Integer.TYPE;
```

还可以通过getSuperclass()获得父类的Class对象

```java
Class c5 = c1.getSuperclass;
```

方法还有..........

##### 所有类型的Class对象

```java
package com;

import java.lang.annotation.ElementType;

public class test2 {
    public static void main(String[] args) {
        Class c1=Object.class;//类
        Class c2=Comparable.class;//接口
        Class c3=String[].class;//一维数组
        Class c4=int[][].class;//二维数组
        Class c5=Override.class;//注解
        Class c6= ElementType.class;//枚举
        Class c7=Integer.class;//基本数据类型
        Class c8=void.class;//void
        Class c9=Class.class;//class
        System.out.println(c1);
        System.out.println(c2);
        System.out.println(c3);
        System.out.println(c4);
        System.out.println(c5);
        System.out.println(c6);
        System.out.println(c7);
        System.out.println(c8);
        System.out.println(c9);
        int[] a=new int[10];
        int[] b=new int[100];
        int[][] c=new int [10][10];
        System.out.println(a.getClass().hashCode());
        System.out.println(b.getClass().hashCode());
        System.out.println(c.getClass().hashCode());

    }
}

```

<label style="background:yellow">只要元素类型与维度一样,就是同一个Class</label>

<img src="https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220111170812325.png" alt="image-20220111170812325" style="zoom:80%;" />

#### 类加载内存分析

##### java内存

1. 堆

- 存放new的对象和数组

- 可以被所有线程共享,不会存放别的对象引用

2. 栈

- 存放基本变量类型(会包含这个基本类型的具体数值)

- 引用对象的变量(会存放这个引用在堆里面的具体地址)

3. 方法区

- 可以被所有的线程共享
- 包含了所有的class和static变量

### 反射能做什么

- 在运行时判断任意一个对象所属的类
- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时调用任意一个对象的方法
- 生成动态代理

### 获取类对象的对象有三种

- 类.class
- obj.getClass()     //通过对象的方式获取类对象

Cat cat=new Cat();

Class<? extends Cat> catClass = cat.getClass(); 

- Class.forName("类的名字")

读取properties配置文件工具类

```java
import java.io.IOException;
import java.util.Properties;

public class PropUtils {

    private static final Properties p= new Properties();
    static{
        try{
            p.load(PropUtils.class.getClassLoader().getResourceAsStream("object.properties"));
        }
        catch (IOException e){
            throw new ExceptionInInitializerError("加载配置文件失败");
        }
    }
    public static String getProperty(String key){
        return p.getProperty(key);
    }

}
```



```java
Class<?> clz=Class.forName(PropUtils.getProperty("animal"));
        Constructor<?> constructor = clz.getConstructor();
        Animal anl=(Animal) constructor.newInstance();
        anl.call();
```

类对象:是类加载的产物

类加载:JVM第一次读取一个类的时候,将类的字节码文件加载到内存的过程



#### 获取类中的属性

getFields()//获取公开的属性,包括父类中的公开属性

getDeclaredField()//获取声明的所有属性,不包含父类中定义的属性

getField()//获取指定的公开属性

getDeclaredField()//获取声明的所有属性,但是不包过父类中定义的属性



#### 修改对象中的值

通过属性去操作对象

```java
@Test
    public void test() throws Exception{

        Cat cat=new Cat();
        Class<? extends Cat> catClass = cat.getClass();
        Field f = catClass.getDeclaredField("name");
        f.set(cat,"小白");
        System.out.println(cat.name);
    }
```

并且也能修改私有属性,只需要添加一段代码  `f.setAccessible(true);`即可

```java
@Test
    public void test() throws Exception{

        Cat cat=new Cat();
        Class<? extends Cat> catClass = cat.getClass();
        Field f = catClass.getDeclaredField("name");
        f.setAccessible(true);
        f.set(cat,"小白");

        System.out.println(cat.getName());

    }
```



将给定的Map通过反射转换成一个对象

```java
@Test
    public void testMap() throws Exception{
        Map<String,Object> map=new HashMap<>();
        map.put("name","小白");
        map.put("age",2);
        map.put("type","加菲猫");

        Class<Cat> catClass = Cat.class;
        //获取所有属性
        Field[] declaredFields = catClass.getDeclaredFields();
        //获取父类中定义的属性
        Field[] declaredFields1 = catClass.getSuperclass().getDeclaredFields();
        //合并两个类的所有属性
        List<Field> collect = Arrays.stream(declaredFields).collect(Collectors.toList());
        List<Field> collect1 = Arrays.stream(declaredFields1).collect(Collectors.toList());
        collect.addAll(collect1);
        //遍历所有的field
        Cat cat=new Cat();

        for (Field f:collect
             ) {
            if(map.containsKey(f.getName())){
                f.setAccessible(true);
                f.set(cat,map.get(f.getName()));
            }

        }
        System.out.println(cat);
    }
```

##### 获取类的构造方法

```java
@Test
    public void getConstructor() throws Exception{
        Class<Dog> dogClass = Dog.class;
        //获取无参构造方法
        Constructor<Dog> declaredConstructor = dogClass.getDeclaredConstructor();
        //破坏封装
        declaredConstructor.setAccessible(true);
        //调用无参构造方法
        Dog dog = declaredConstructor.newInstance();
        System.out.println(dog);

        //调用指定的有参构造方法,根据参数类型去获取
        Constructor<Dog> declaredConstructor1 = dogClass.getDeclaredConstructor(String.class);
        Dog dog1 = declaredConstructor1.newInstance("小狗");
        System.out.println(dog1);

    }
```

##### 获取类的方法

```java
@Test
    public void getMethod() throws Exception{
        Class<Dog> dogClass = Dog.class;
        //获取该类中所有的公开的方法,包含继承的
        Method[] methods = dogClass.getMethods();

        //获取当前类自己声明的所有方法

        Method[] declaredMethods = dogClass.getDeclaredMethods();
        System.out.println(Arrays.toString(declaredMethods));

        //获取properties配置文件中指定的方法
        String methodName = PropUtils.getProperty("methodName");
        Method declaredMethod = dogClass.getDeclaredMethod(methodName);

        //获取方法修饰符
        int modifiers = declaredMethod.getModifiers();//什么都不加 是0 ， public  是1 ，private 是 2 ，protected 是 4，static 是 8 ，final 是 16。
        //获取方法返回值
        Class<?> returnType = declaredMethod.getReturnType();
        System.out.println(returnType);
        //还能获取异常类型...方法参数...
        
        //通过反射创建该类的对象
        Dog dog = dogClass.getDeclaredConstructor().newInstance();

        declaredMethod.invoke(dog);

        //执行带有参数的方法
        Method eat = dogClass.getDeclaredMethod("eat", String.class);
        eat.invoke(dog,"饼干");

    }
```

