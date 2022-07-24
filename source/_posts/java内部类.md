---
title: java内部类
date: 2022-03-04 16:47:37
tags:		
      - java
      - java内部类
categories: java
---

# 内部类

一般情况下,类和类之间是互相独立的,内部类的意思是是打破这种独立,让一个类称为另外一个类的内部成员,和成员变量,成员方法相同级别

> 为什么要使用内部类?
>
> 采用内部类这种技术,可以隐藏细节和内部结构,封装性更好,让程序的结构更加合理

### 非静态内部类

内部类在创建的时候不能直接new,需要加个前缀才能new出来,加的这个前缀是已经实例化的外部类对象

```java
OuterClass outerClass = new OuterClass();
        InnerClass innerClass = outerClass.new InnerClass();
        innerClass.display();
```

非静态内部类的使用就是将内部类当做外部类的一个成员变量去使用,所以必须依赖于外部类的对象才能调用



如果在类的内部去创建一个内部类对象,就不需要加前缀,和正常的创建方式一样

```java
public class outclass {
    private String outerName;

    public void display(){

        class innerClass{
            public void print(){

                System.out.println("内部类方法");
            }
        }
        innerClass innerClass = new innerClass();
        innerClass.print();

    }

    public static void main(String[] args) {
        outclass outerClass = new outclass();
        outerClass.display();

    }
}
```

### 静态内部类

静态内部类的构造不需要依赖于外部类对象,类中的所有静态组件都不需要依赖于任何对象,可以直接通过类本身进行构造

```java
public class OuterClass {

    private String outerName;

    public void display(){

        System.out.println("外部类显示");
        System.out.println(outerName);
    }
    //内部类
    public static class InnerClass{
        private String innerName;
        public void display(){

            System.out.println("内部类显示");
            System.out.println(innerName);
        }

        public InnerClass() {
            this.innerName = "inner Class";
        }
    }
    public static void main(String[] args) {
        OuterClass outerClass = new OuterClass();
        outerClass.display();
        OuterClass.InnerClass innerClass = new InnerClass();
        innerClass.display();
    }
}
```

### 匿名内部类

匿名内部类的本质是接口的匿名实现

```java
public interface Myinterface {
    public void test();
}
```

```java
public class Test {
    public static void main(String[] args) {
        //匿名内部类
        Myinterface myinterface=new Myinterface() {
            @Override
            public void test() {
                System.out.println("匿名内部类方法");
            }
        };
        myinterface.test();
    }
}
```



