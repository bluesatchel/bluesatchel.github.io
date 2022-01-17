---
title: java反射
date: 2022-01-11 13:02:55
tags:
      - java
      - reflection
categories: java
---

先认识一下动态和静态语言<!--more-->

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

- ...............

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

![image-20220111164508670](https://gitee.com/blue_satchel/images/raw/master/image-20220111164508670.png)

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

<img src="https://gitee.com/blue_satchel/images/raw/master/image-20220111170812325.png" alt="image-20220111170812325" style="zoom:80%;" />

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
