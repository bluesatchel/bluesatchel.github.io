---
title: java注解
date: 2022-01-11 13:12:36
tags:
      - java
categories: java
---

## 注解(Annotation)

<!--more-->

### 注解简介

##### 注解的作用:

- 不是程序本身,可以对程序做出解释,也能对程序的正确性进行矫正(比如写了@Override就要重写)
- 可以被其他程序(比如编译器等)读取

##### 注解的格式:

注解是以"@注解名"在代码中存在的,还可以添加一些参数值,例如:@SuppressWarnings(Value="unchecked")

##### 注解在哪里使用

可以附加在package,class,method,field等上面,相当于给他们添加了额外的辅助信息,我们可以通过反射机制编程实现对这些元数据的访问

### 内置注解

##### @Override

定义在java.long.Override中,用于修饰方法,表示一个方法声明打算重写

##### @Deprecated

定义在java.long.Deprecated中,此注释可以用于修饰方法,属性类,表示不鼓励程序员使用这一的元素,通过是一万年它很危险或者存在更好的选择

##### @SuppressWarnings

定义在java.long.SuppressWarnings中,用来抑制编译时的警告信息,它需要添加一个参数才能使用

例如:

- @SuppressWarnings("all")

- @SuppressWarnings("unchecked")

- @SuppressWarnings(value={"unchecked","deprecation"})

能传递多个参数源自于其源码中的最后一行

<img src="https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220111133538313.png" alt="image-20220111133538313" style="zoom:80%;" />

- .......

### 元注解

元注解的作用就是解释其他注解,java定义了四个标准的元注解(meta_annotation)

- @Target 

用于表示注解的使用范围

​							  @Target(ElementType.TYPE)   //接口、类、枚举

　　　　　　　　@Target(ElementType.FIELD) //字段、枚举的常量

　　　　　　　　@Target(ElementType.METHOD) //方法

　　　　　　　　@Target(ElementType.PARAMETER) //方法参数

　　　　　　　　@Target(ElementType.CONSTRUCTOR)  //构造函数

　　　　　　　　@Target(ElementType.LOCAL_VARIABLE)//局部变量

　　　　　　　　@Target(ElementType.ANNOTATION_TYPE)//注解

　　　　　　　　@Target(ElementType.PACKAGE) ///包   


- @Retention

表示需要在什么级别保存该注释信息,用于描述注解的声明周期

Runtime>class>source

- @Documented

说明该注解将被包含在javadoc中

- @Inherited

说明子类可以继承父类中的该注解

### 自定义注解

使用@interface自定义注解,自动继承java.lang.annotation.Annotation接口

- 格式

public @ interface 注解名 {定义内容}

- 其中的每一个方法实际上是声明了一个配置参数

- 方法的名称就是参数的名称
- 返回值类型就是参数的类型(返回值只能是Class,String,enum)
- 可以通过default来声明参数的默认值
- 如果只有一个参数成员,一般参数名为value<label style="color:red">(value参数可以省略传参)</label>
- 注解元素必须要有值,我们定义注解元素时,经常使用空字符串,0作为默认值

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

public class test1 {
    @myAnnotation(Class = "不写默认就是一班二班")
    public void method(){
    }
}
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@interface myAnnotation{
    //注解的参数:参数类型+参数名();
    String name() default "lol";//定义了注解的参数就要传参,也可以通过default定义不传参时默认情况下的参数
    String [] Class() default {"一班","二班"};
}
```

