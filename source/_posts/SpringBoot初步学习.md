---
title: SpringBoot初步学习
date: 2022-02-01 18:54:54
tags:
      - SpringBoot
      - java
categories: java
---

## 1. SpringBoot简介<!--more-->

### 1.1 为什么要学习SpringBoot

​	SSM还是使用起来不够方便。

- 还需要写很多的配置才能进行正常的使用。
- 实现一个功能需要引入很多的依赖，尤其是要自己去维护依赖的版本，特别容易出现依赖冲突等问题。

​	SpringBoot就能很好的解决上述问题。



### 1.2 SpringBoot是什么

​	Spring Boot是基于Spring开发的全新框架，相当于对Spring做了又一层封装。

​	其设计目的是用来简化Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。（自动配置）

​	并且对第三方依赖的添加也进行了封装简化。（起步依赖）



​	所以Spring能做的他都能做，并且简化了配置。

​	并且还提供了一些Spring所没有的比如：

- 内嵌web容器，不再需要部署到web容器中

  提供准备好的特性，如指标、健康检查和外部化配置 



​	最大特点：**自动配置**、**起步依赖**



​	官网：https://spring.io/projects/spring-boot

## 2.快速入门



#### 2.1清理Maven仓库脚本

~~~~
@echo off
rem create by NettQun
  
rem 这里写你的仓库路径
set REPOSITORY_PATH=E:\Develop\maven_rep
rem 正在搜索...
for /f "delims=" %%i in ('dir /b /s "%REPOSITORY_PATH%\*lastUpdated*"') do (
    echo %%i
    del /s /q "%%i"
)
rem 搜索完毕
pause
~~~~

创建一个bat文件，然后复制上述脚本进去，修改其中maven本地仓库的地址，保存后双击执行即可。



#### 2.2HelloWorld程序

##### 继承父工程,添加依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>springBoot</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.3</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.6.3</version>
        </dependency>
    </dependencies>

</project>
```

##### 创建启动类

创建一个类并加上@SpringBootApplication注解标识为启动类

```java
package com.blue;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class,args);
    }
}

```

创建controller包,新建helloController.java,和SpringMVC类似

```java
package com.blue.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
@Controller
public class HelloController {
    @RequestMapping("/hello")
    @ResponseBody
    public String hello(){
        return "HelloSpringBoot";
    }
}
```

启动的时候不需要去配置Tomcat,直接启动刚才配置的启动类的main方法即可

![image-20220201190334027](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220201190334027.png)

### 2.3 SpringBoot打包部署

1.添加maven插件,pom.xml

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.6.3</version>
            </plugin>

        </plugins>
    </build>
```

2.maven打包,运行package

![image-20220201191519221](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220201191519221.png)

找到对应target目录下生成的jar包

3.运行jar包

```shell
java -jar jar包路径
```

### 2.4 快速部署

​	https://start.spring.io/

## 3.起步依赖

​	SpringBoot依靠父项目中的版本锁定和starter机制让我们能更轻松的实现对依赖的管理。



### 3.0 依赖冲突及其解决方案 

#### 3.0.1 依赖冲突

​	一般程序在运行时发生类似于 java.lang.ClassNotFoundException，Method not found: '……'，或者莫名其妙的异常信息，这种情况一般很大可能就是 jar包依赖冲突的问题引起的了。

​	一般在是A依赖C(低版本)，B也依赖C(高版本)。 都是他们依赖的又是不同版本的C的时候会出现。

​	

#### 3.0.2 解决方案

​	如果出现了类似于 java.lang.ClassNotFoundException，Method not found: 这些异常检查相关的依赖冲突问题，排除掉低版本的依赖，留下高版本的依赖。

​	

​	

### 3.1 版本锁定

​	我们的SpringBoot模块都需要继承一个父工程：**spring-boot-starter-parent**。在spring-boot-starter-parent的父工程**spring-boot-dependencies**中对常用的依赖进行了版本锁定。这样我们在添加依赖时，很多时候都不需要添加依赖的版本号了。

​	

​	我们也可以采用覆盖properties配置或者直接指定版本号的方式修改依赖的版本。

例如：

直接指定版本号

~~~~xml
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.7.2</version>
        </dependency>
~~~~

覆盖properties配置

~~~~xml
    <properties>
        <aspectj.version>1.7.2</aspectj.version>
    </properties>
~~~~



### 3.2 starter机制

​	当我们需要使用某种功能时只需要引入对应的starter即可。一个starter针对一种特定的场景，其内部引入了该场景所需的依赖。这样我们就不需要单独引入多个依赖了。

​	命名规律

- 官方starter都是以  `spring-boot-starter`开头后面跟上场景名称。例如：spring-boot-starter-data-jpa
- 非官方starter则是以 `场景名-spring-boot-starter`的格式，例如：mybatis-spring-boot-starter

## 4.自动配置

​	SpringBoot中最重要的特性就是自动配置。

​	Springboot遵循“**约定优于配置**”的原则，自动进行了默认配置。这样我们就不需要做大量的配置。

​	当我们需要使用什么场景时，就会自动配置这个场景相关的配置。

​	如果他的默认配置不符合我们的需求时修改这部分配置即可。

## 5.YML配置

### 	5.1.简介	

#### 5.1.1YML是什么

​		YAML (YAML Ain't a Markup Language)YAML不是一种标记语言，通常以.yml为后缀的文件，是一种直观的能够被电脑识别的数据序列化格式，并且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，一种专门用来写配置文件的语言。

​		 YAML试图用一种比XML更敏捷的方式，来完成XML所完成的任务。

​		 例如： 

~~~~yml
student:
    name: sangeng
    age: 15
~~~~



~~~~xml
<student>
    <name>sangeng</name>
    <age>15</age>
</student>
~~~~

#### 5.1.2YML优点

1. YAML易于人们阅读。

2. 更加简洁明了

### 5.2.语法

#### 5.2.1约定

- k: v 表示键值对关系，**冒号后面必须有一个空格**

- 使用空格的缩进表示层级关系，空格数目不重要，**只要是左对齐的一列数据，都是同一个层级的**

- 大小写敏感

- 缩进时**不允许使用Tab键，只允许使用空格**。(IDEA会帮忙替换tab为空格)

- <label style="background:yellow">java中对于驼峰命名法，可用原名或使用-代替驼峰，如java中的lastName属性,在yml中使用lastName或 last-name都可正确映射。</label>

- yml中注释前面要加#

  

#### 5.2.2键值关系

##### 普通值(字面量)

k: v：字面量直接写；

   字符串默认不用加上单引号或者双绰号；

   "": 双引号；转义字符能够起作用

​       name:   "sangeng \n caotang"：输出；sangeng 换行  caotang

   ''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据

~~~~yml
name1: sangeng 
name2: 'sangeng  \n caotang'
name3: "sangeng  \n caotang"
age: 15
flag: true
~~~~

##### 日期

~~~~yml
date: 2019/01/01
~~~~



##### 对象(属性和值)、Map(键值对)

多行写法：

在下一行来写对象的属性和值的关系，注意缩进 

~~~~yml
student:
  name: zhangsan
  age: 20
~~~~

行内写法：

~~~~yml
student: {name: zhangsan,age: 20}
~~~~

##### 数组、list、set

 用- 值表示数组中的一个元素 

多行写法：

~~~~yml
pets:
  - dog
  - pig
  - cat
~~~~



行内写法：

~~~~yml
pets: [dog,pig,cat]
~~~~



##### 对象数组、对象list、对象set

~~~~yml
students:
 - name: zhangsan
   age: 22
 - name: lisi
   age: 20
 - {name: wangwu,age: 18}
~~~~



#### 5.2.3 占位符赋值

可以使用 **${key:defaultValue}** 的方式来赋值，若key不存在，则会使用defaultValue来赋值。

例如

~~~~yml
server:
  port: ${myPort:8080}

myPort: 80   
~~~~

### 5.3.SpringBoot读取YML

#### 5.3.1 @Value注解

​	注意使用此注解只能获取简单类型的值（8种基本数据类型及其包装类，String,Date）

~~~~yml
student:
  lastName: sangeng
~~~~

~~~~java
@RestController
public class TestController {
    @Value("${student.lastName}")
    private String lastName;
    @RequestMapping("/test")
    public String test(){
        System.out.println(lastName);
        return "hi";
    }
    
}
~~~~

**注意：加了@Value的类必须是交由Spring容器管理的**

#### 5.3.2 @ConfigurationProperties

​	yml配置

~~~~yml
student:
  lastName: sangeng
  age: 17
student2:
  lastName: sangeng2
  age: 15
~~~~

​    在类上添加注解@Component  和@ConfigurationProperties(prefix = "配置前缀")

~~~~java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Component
@ConfigurationProperties(prefix = "student")
public class Student {
    private String lastName;
    private Integer age;
}
~~~~

​	从spring容器中获取Student对象

~~~~java
@RestController
public class TestController {

    @Autowired
    private Student student;
    @RequestMapping("/test")
    public String test(){
        System.out.println(student);
        return "hi";
    }
}

~~~~

​	**注意事项：要求对应的属性要有set/get方法，并且key要和成员变量名一致才可以对应的上。**

##### 小问题

今天SpringBoot配置文件部署时,发出警告`Spring Boot Configuration Annotation Processor not configured`但是不影响运行

![image-20220201205503274](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220201205503274.png)

解决方法

在maven中引入依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

配置注解执行器配置完成后，当执行类中已经定义了对象和该对象的字段后，在配置文件中对该类赋值时，便会非常方便的弹出提示信息。	**注意：添加完依赖加完注解后要运行一次程序才会有相应的提示。**

### 5.4.练习

要求把下面实体类中的各个属性在yml文件中进行赋值。然后想办法读取yml配置的属性值，进行输出测试。

~~~~java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Student {
    private String lastName;
    private Integer age;
    private Boolean boss;

    private Date birthday;
    private Map<String,String> maps;
    private Map<String,String> maps2;
    private List<Dog> list;

    private Dog dog;
    private String[] arr;
    private String[] arr2;

    private Map<String,Dog> dogMap;
}
@Data
@AllArgsConstructor
@NoArgsConstructor
class Dog {
    private String name;
    private Integer age;
}
~~~~







#### 答案


~~~~~yml
# 练习
student:
  lastName: sangeng
  age: 15
  boss: true
  birthday: 2006/2/3
  maps:
    name: sangeng
    age: 11
  maps2: {name: caotang,age: 199}
  list:
    - name: 小白
      age: 3
    - name: 小黄
      age: 4
    - {name: 小黑,age: 1}
  dog:
    name: 小红
    age: 5
  arr:
    - sangeng
    - caotang

  arr2: [sangeng,caotang]
  dogMap:
    xb: {name: 小白,age: 9}
    xh:
      name: 小红
      age: 6

~~~~~

~~~~java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Component
@ConfigurationProperties(prefix = "student")
public class Student {
    private String lastName;
    private Integer age;
    private Boolean boss;

    private Date birthday;
    private Map<String,String> maps;
    private Map<String,String> maps2;
    private List<Dog> list;

    private Dog dog;
    private String[] arr;
    private String[] arr2;

    private Map<String,Dog> dogMap;
}
@Data
@AllArgsConstructor
@NoArgsConstructor
class Dog {
    private String name;
    private Integer age;
}
~~~~

### 5.5 YML和properties配置的相互转换

​	我们可以使用一些网站非常方便的实现YML和properties格式配置的相互转换。

转换网站：https://www.toyaml.com/index.html

# SpringBoot-常见场景

## 1.热部署

​	SpringBoot为我们提供了一个方便我们开发测试的工具dev-tools。使用后可以实现热部署的效果。当我们运行了程序后对程序进行了修改，程序会自动重启。

​	 原理是使用了两个ClassLoder,一个ClassLoader加载哪些不会改变的类(第三方jar包),另一个ClassLoader加载会更改的类.称之为Restart ClassLoader,这样在有代码更改的时候,原来的Restart Classloader被丢弃,重新创建一个Restart ClassLoader,由于需要加载的类相比较少,所以实现了较快的重启。

​	

### 1.1 准备工作

①设置IDEA自动编译

​	 在idea中的setting做下面配置 

![自动编译配置](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/%E8%87%AA%E5%8A%A8%E7%BC%96%E8%AF%91%E9%85%8D%E7%BD%AE.png)



②设置允许程序运行时自动启动

​	 ctrl + shift + alt + / 这组快捷键后会有一个小弹窗，点击Registry 就会进入下面的界面，找到下面的配置项并勾选，勾选后直接点close 

![允许运行时自动启动](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/%E5%85%81%E8%AE%B8%E8%BF%90%E8%A1%8C%E6%97%B6%E8%87%AA%E5%8A%A8%E5%90%AF%E5%8A%A8.png)



### 1.2使用

①添加依赖

~~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
~~~~

②触发热部署

​	当我们在修改完代码或者静态资源后可以切换到其它软件，让IDEA自动进行编译，自动编译后就会触发热部署。

​	或者使用Ctrl+F9手动触发重新编译。

## 2.单元测试

​	我们可以使用SpringBoot整合Junit进行单元测试。

​	**Spring Boot 2.2.0 版本开始引入 JUnit 5 作为单元测试默认库**。

​	Junit5功能相比于Junit4也会更强大。但是本课程是SpringBoot的课程，所以主要针对SpringBoot如何整合Junit进行单元测试做讲解。暂不针对Junit5的新功能做介绍。如有需要会针对Junit5录制专门的课程进行讲解。

​	

### 2.1 使用

#### ①添加依赖

~~~~xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
~~~~

#### ②编写测试类

~~~~java
import com.sangeng.controller.HelloController;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class ApplicationTest {

    @Autowired
    private HelloController helloController;

    @Test
    public void testJunit(){
        System.out.println(1);
        System.out.println(helloController);
    }
}
~~~~



**注意：测试类所在的包需要和启动类是在同一个包下。否则就要使用如下写法指定启动类。**

~~~~java
import com.sangeng.controller.HelloController;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

//classes属性来指定启动类
@SpringBootTest(classes = HelloApplication.class)
public class ApplicationTest {

    @Autowired
    private HelloController helloController;

    @Test
    public void testJunit(){
        System.out.println(1);
        System.out.println(helloController);
    }
}

~~~~



### 2.2 兼容老版本

​	如果是对老项目中的SpringBoot进行了版本升级会发现之前的单元测试代码出现了一些问题。

​	因为Junit5和之前的Junit4有比较大的不同。

​	先看一张图：

 ![img](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/junit5.jpeg) 

​	从上图可以看出  **JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage**



- **JUnit Platform**： 这是Junit提供的平台功能模块，通过它，其它的测试引擎也可以接入
- **JUnit JUpiter**：这是JUnit5的核心，是一个基于JUnit Platform的引擎实现，它包含许多丰富的新特性来使得自动化测试更加方便和强大。
- **JUnit Vintage**：这个模块是兼容JUnit3、JUnit4版本的测试引擎，使得旧版本的自动化测试也可以在JUnit5下正常运行。



​	虽然Junit5包含了**JUnit Vintage**来兼容JUnit3和Junit4，但是**SpringBoot 2.4 以上版本对应的spring-boot-starter-test移除了默认对** **Vintage 的依赖。**所以当我们仅仅依赖spring-boot-starter-test时会发现之前我们使用的@Test注解和@RunWith注解都不能使用了。

​	我们可以单独在依赖vintage来进行兼容。

~~~~xml
        <dependency>
            <groupId>org.junit.vintage</groupId>
            <artifactId>junit-vintage-engine</artifactId>
            <scope>test</scope>
        </dependency>
~~~~



**注意：**

​		**org.junit.Test对应的是Junit4的版本，就搭配@RunWith注解来使用。**

SpringBoot2.2.0之前版本的写法

~~~~java
import com.sangeng.controller.HelloController;
//import org.junit.jupiter.api.Test;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

//classes属性来指定启动类
@SpringBootTest
@RunWith(SpringRunner.class)
public class ApplicationTest {

    @Autowired
    private HelloController helloController;

    @Test
    public void testJunit(){
        System.out.println(1);
        System.out.println(helloController);
    }
}
~~~~



## 4.Web开发

### 4.1 静态资源访问

​	由于SpringBoot的项目是打成jar包的所以没有之前web项目的那些web资源目录(webapps)。

​	那么我们的静态资源要放到哪里呢？

​	从SpringBoot官方文档中我们可以知道，我们可以把静态资源放到 `resources/static`   (或者 `resources/public` 或者`resources/resources` 或者 `resources/META-INF/resources`) 中即可。

​	静态资源放完后，

​	例如我们想访问文件：resources/static/index.html  只需要在访问时资源路径写成/index.html即可。  

​	例如我们想访问文件：resources/static/pages/login.html  访问的资源路径写成： /pages/login.html



#### 4.1.1 修改静态资源访问路径

​	SpringBoot默认的静态资源路径匹配为/** 。如果想要修改可以通过 `spring.mvc.static-path-pattern` 这个配置进行修改。

​	例如想让访问静态资源的url必须前缀有/res。例如/res/index.html 才能访问到static目录中的。我们可以修改如下：

在application.yml中

~~~~yml
spring:
  mvc:
    static-path-pattern: /res/** #修改静态资源访问路径
~~~~



#### 4.1.2 修改静态资源存放目录

​	我们可以修改 spring.web.resources.static-locations 这个配置来修改静态资源的存放目录。

​	例如:

~~~~yml
spring:
  web:
    resources:
      static-locations:
        - classpath:/sgstatic/ 
        - classpath:/static/
~~~~



### 4.2 设置请求映射规则@RequestMapping

​	详细讲解：https://www.bilibili.com/video/BV1AK4y1o74Y  P5-P12

​	该注解可以加到方法上或者是类上。（查看其源码可知）

​	我们可以用其来设定所能匹配请求的要求。只有符合了设置的要求，请求才能被加了该注解的方法或类处理。

#### 4.2.1 指定请求路径

​	path或者value属性都可以用来指定请求路径。

例如：

​	我们期望让请求的资源路径为**/test/testPath**的请求能够被**testPath**方法处理则可以写如下代码

~~~~java
@RestController
@RequestMapping("/test")
public class HelloController {
    @RequestMapping("/testPath")
    public String testPath(){
        return "testPath";
    }
}
~~~~

~~~~java
@RestController
public class HelloController {

    @RequestMapping("/test/testPath")
    public String testPath(){
        return "testPath";
    }
}
~~~~

#### 4.2.2 指定请求方式

​	method属性可以用来指定可处理的请求方式。

例如：

​	我们期望让请求的资源路径为**/test/testMethod**的**POST**请求能够被**testMethod**方法处理。则可以写如下代码

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {

    @RequestMapping(value = "/testMethod",method = RequestMethod.POST)
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "testMethod";
    }
}

~~~~



注意：我们可以也可以运用如下注解来进行替换

- ​    @PostMapping    等价于   @RequestMapping(method = RequestMethod.POST) 

- ​	@GetMapping    等价于   @RequestMapping(method = RequestMethod.GET) 
- ​	@PutMapping    等价于   @RequestMapping(method = RequestMethod.PUT) 
- ​	@DeleteMapping    等价于   @RequestMapping(method = RequestMethod.DELETE) 

例如：

​	上面的需求我们可以使用下面的写法实现

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {

    @PostMapping(value = "/testMethod")
    public String testMethod(){
        System.out.println("testMethod处理了请求");
        return "testMethod";
    }
}
~~~~



#### 4.2.3 指定请求参数

​	我们可以使用**params**属性来对请求参数进行一些限制。可以要求必须具有某些参数，或者是某些参数必须是某个值，或者是某些参数必须不是某个值。



例如：

​	我们期望让请求的资源路径为**/test/testParams**的**GET**请求,并且请求参数中**具有code参数**的请求能够被testParams方法处理。则可以写如下代码

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {
    @RequestMapping(value = "/testParams",method = RequestMethod.GET,params = "code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "testParams";
    }
}
~~~~

​	

​	如果是要求**不能有code**这个参数可以把改成如下形式

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {
    @RequestMapping(value = "/testParams",method = RequestMethod.GET,params = "!code")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "testParams";
    }
}
~~~~

​	

​	如果要求有code这参数，并且这参数值必须**是某个值**可以改成如下形式

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {
    @RequestMapping(value = "/testParams",method = RequestMethod.GET,params = "code=sgct")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "testParams";
    }
}
~~~~



​	如果要求有code这参数，并且这参数值必须**不是某个值**可以改成如下形式	

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {
    @RequestMapping(value = "/testParams",method = RequestMethod.GET,params = "code!=sgct")
    public String testParams(){
        System.out.println("testParams处理了请求");
        return "testParams";
    }
}
~~~~



#### 4.2.4 指定请求头

​	我们可以使用**headers**属性来对请求头进行一些限制。



例如：

​	我们期望让请求的资源路径为**/test/testHeaders的**GET**请求,并且请求头中**具有**deviceType**的请求能够被testHeaders方法处理。则可以写如下代码

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {
    
    @RequestMapping(value = "/testHeaders",method = RequestMethod.GET,headers = "deviceType")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "testHeaders";
    }
}
~~~~



​	如果是要求不能有**deviceType**这个请求头可以把改成如下形式

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {
    
    @RequestMapping(value = "/testHeaders",method = RequestMethod.GET,headers = "!deviceType")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "testHeaders";
    }
}
~~~~



​	如果要求有deviceType这个请求头，并且其值必须**是某个值**可以改成如下形式

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {
    
    @RequestMapping(value = "/testHeaders",method = RequestMethod.GET,headers = "deviceType=ios")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "testHeaders";
    }
}
~~~~



​	如果要求有deviceType这个请求头，并且其值必须**不是某个值**可以改成如下形式

~~~~java
@RestController
@RequestMapping("/test")
public class TestController {
    
    @RequestMapping(value = "/testHeaders",method = RequestMethod.GET,headers = "deviceType!=ios")
    public String testHeaders(){
        System.out.println("testHeaders处理了请求");
        return "testHeaders";
    }
}
~~~~



#### 4.2.5 指定请求头Content-Type

​	我们可以使用**consumes**属性来对**Content-Type**这个请求头进行一些限制。



##### 范例一

​	我们期望让请求的资源路径为**/test/testConsumes**的POST请求,并且请求头中的Content-Type头必须为 **multipart/from-data** 的请求能够被testConsumes方法处理。则可以写如下代码

~~~~java
    @RequestMapping(value = "/testConsumes",method = RequestMethod.POST,consumes = "multipart/from-data")
    public String testConsumes(){
        System.out.println("testConsumes处理了请求");
        return "testConsumes";
    }
~~~~

##### 范例二

​	如果我们要求请求头Content-Type的值必须**不能为某个multipart/from-data**则可以改成如下形式：

~~~~java
    @RequestMapping(value = "/testConsumes",method = RequestMethod.POST,consumes = "!multipart/from-data")
    public String testConsumes(){
        System.out.println("testConsumes处理了请求");
        return "testConsumes";
    }
~~~~

### 4.3 获取请求参数

#### 4.3.1 获取路径参数

##### RestFul风格

​	RestFul风格的接口一些参数是在请求路径上的。类似： /user/1  这里的1就是id。

​	如果我们想获取这种格式的数据可以使用**@PathVariable**来实现。



##### 范例一

​	要求定义个RestFul风格的接口，该接口可以用来根据id查询用户。请求路径要求为  /user  ，请求方式要求为GET。

​	而请求参数id要写在请求路径上，例如  /user/1   这里的1就是id。

​	我们可以定义如下方法，通过如下方式来获取路径参数：

~~~~java
@RestController
public class UserController {

    @RequestMapping(value = "/user/{id}",method = RequestMethod.GET)
    public String findUserById( @PathVariable("id")Integer id){
        System.out.println("findUserById");
        System.out.println(id);
        return "findUserById";
    }
}
~~~~

##### 范例二

​	如果这个接口，想根据id和username查询用户。请求路径要求为  /user  ，请求方式要求为GET。

​	而请求参数id和name要写在请求路径上，例如  /user/1/zs   这里的1就是id，zs是name

​	我们可以定义如下方法，通过如下方式来获取路径参数：

~~~~java
@RestController
public class UserController {
    @RequestMapping(value = "/user/{id}/{name}",method = RequestMethod.GET)
    public String findUser(@PathVariable("id") Integer id,@PathVariable("name") String name){
        System.out.println("findUser");
        System.out.println(id);
        System.out.println(name);
        return "findUser";
    }
}

~~~~

#### 4.3.2 获取请求体中的Json格式参数

​	RestFul风格的接口一些比较复杂的参数会转换成Json通过请求体传递过来。这种时候我们可以使用**@RequestBody**注解获取请求体中的数据。

##### 4.3.2.1 配置

​	SpringBoot的web启动器已经默认导入了jackson的依赖，不需要再额外导入依赖了。



##### 4.3.2.2 使用

###### 范例一

​	要求定义个RestFul风格的接口，该接口可以用来新建用户。请求路径要求为  /user  ，请求方式要求为POST。

用户数据会转换成json通过请求体传递。
​	请求体数据

~~~~json
{"name":"三更","age":15}
~~~~

​	

1.获取参数封装成实体对象

​	如果我们想把Json数据获取出来封装User对象,我们可以这样定义方法：

~~~~~java
@RestController
public class UserController {
    @RequestMapping(value = "/user",method = RequestMethod.POST)
    public String insertUser(@RequestBody User user){
        System.out.println("insertUser");
        System.out.println(user);
        return "insertUser";
    }
}
~~~~~

​	User实体类如下：

~~~~java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Integer id;
    private String name;
    private Integer age;
}

~~~~

​	

2.获取参数封装成Map集合

​	也可以把该数据获取出来封装成Map集合：

~~~~java
    @RequestMapping(value = "/user",method = RequestMethod.POST)
    public String insertUser(@RequestBody Map map){
        System.out.println("insertUser");
        System.out.println(map);
        return "insertUser";
    }
~~~~



###### 范例二

​	如果请求体传递过来的数据是一个User集合转换成的json，Json数据可以这样定义：

~~~~java
[{"name":"三更1","age":14},{"name":"三更2","age":15},{"name":"三更3","age":16}]
~~~~

​	方法定义：

~~~~java
    @RequestMapping(value = "/users",method = RequestMethod.POST)
    public String insertUsers(@RequestBody List<User> users){
        System.out.println("insertUsers");
        System.out.println(users);
        return "insertUser";
    }
~~~~



##### 4.3.2.3 注意事项

​	<label style="background:yellow">如果需要使用@RequestBody来获取请求体中Json并且进行转换，要求请求头 Content-Type 的值要为： application/json 。</label>

#### 4.3.3 获取QueryString格式参数 

​	如果接口的参数是使用QueryString的格式的话，我们也可以使用SpringMVC快速获取参数。

​	我们可以使用**@RequestParam**来获取QueryString格式的参数。



##### 4.3.3.1 使用

###### 范例一

​	要求定义个接口，该接口请求路径要求为  /testRequestParam，请求方式无要求。参数为id和name和likes。使用QueryString的格式传递。



1.参数单独的获取

​	如果我们想把id，name，likes单独获取出来可以使用如下写法：

​	在方法中定义方法参数，方法参数名要和请求参数名一致，这种情况下我们可以省略**@RequestParam**注解。

~~~~java
    @RequestMapping("/testRquestParam")
    public String testRquestParam(Integer id, String name, String[] likes){
        System.out.println("testRquestParam");
        System.out.println(id);
        System.out.println(name);
        System.out.println(Arrays.toString(likes));
        return "testRquestParam";
    }

~~~~

​	如果方法参数名和请求参数名不一致，我们可以加上**@RequestParam**注解例如：

~~~~java
    @RequestMapping("/testRquestParam")
    public String testRquestParam(@RequestParam("id") Integer uid,@RequestParam("name") String name, @RequestParam("likes")String[] likes){
        System.out.println("testRquestParam");
        System.out.println(uid);
        System.out.println(name);
        System.out.println(Arrays.toString(likes));
        return "testRquestParam";
    }
~~~~



2.获取参数封装成实体对象

​	如果我们想把这些参数封装到一个User对象中可以使用如下写法：

~~~~java
    @RequestMapping("/testRquestParam")
    public String testRquestParam(User user){
        System.out.println("testRquestParam");
        System.out.println(user);
        return "testRquestParam";
    }
~~~~

​	User类定义如下：

~~~~java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Integer id;
    private String name;
    private Integer age;
    private String[] likes;
}
~~~~

​	测试时请求url如下：

~~~~java
http://localhost:8080/testRquestParam?id=1&name=三更草堂&likes=编程&likes=录课&likes=烫头
~~~~



​	**注意：实体类中的成员变量要和请求参数名对应上。并且要提供对应的set/get方法。**



#### 4.3.4 相关注解其他属性

##### 4.3.4.1 required

​	代表是否必须，默认值为true也就是必须要有对应的参数。如果没有就会报错。

​	如果对应的参数可传可不传则可以把其设置为fasle

例如：

~~~~java
    @RequestMapping("/testRquestParam")
    public String testRquestParam(@RequestParam(value = "id",required = false) Integer uid,@RequestParam("name") String name, @RequestParam("likes")String[] likes){
        System.out.println("testRquestParam");
        System.out.println(uid);
        System.out.println(name);
        System.out.println(Arrays.toString(likes));
        return "testRquestParam";
    }
~~~~



##### 4.3.4.2 defaultValue

​	如果对应的参数没有，我们可以用defaultValue属性设置默认值。

例如：

~~~~java
    @RequestMapping("/testRquestParam")
    public String testRquestParam(@RequestParam(value = "id",required = false,defaultValue = "777") Integer uid,@RequestParam("name") String name, @RequestParam("likes")String[] likes){
        System.out.println("testRquestParam");
        System.out.println(uid);
        System.out.println(name);
        System.out.println(Arrays.toString(likes));
        return "testRquestParam";
    }
~~~~

### 4.4 响应体响应数据

​	无论是RestFul风格还是我们之前web阶段接触过的异步请求，都需要把数据转换成Json放入响应体中。



#### 4.4.1 数据放到响应体

​	我们的SpringMVC为我们提供了**@ResponseBody**来非常方便的把Json放到响应体中。

​	**@ResponseBody**可以加在哪些东西上面？类上和方法上

​	具体代码请参考范例。



#### 4.4.2 数据转换成Json

##### 4.4.2.1 配置

​	SpringBoot项目中使用了web的start后，不需要进行额外的依赖和配置



##### 4.4.2.2 使用

​	只要把要转换的数据直接作为方法的返回值返回即可。SpringMVC会帮我们把返回值转换成json。具体代码请参考范例。



#### 4.4.3 范例

##### 范例一

​	要求定义个RestFul风格的接口，该接口可以用来根据id查询用户。请求路径要求为  /response/user  ，请求方式要求为GET。

​	而请求参数id要写在请求路径上，例如   /response/user/1   这里的1就是id。

​	要求获取参数id,去查询对应id的用户信息（模拟查询即可，可以选择直接new一个User对象），并且转换成json响应到响应体中。

~~~~java
@Controller
@RequestMapping("/response")
public class ResponseController {

    @RequestMapping("/user/{id}")
    @ResponseBody
    public User findById(@PathVariable("id") Integer id){
        User user = new User(id, "三更草堂", 15, null);
        return user;
    }
}
~~~~

### 4.5 跨域请求

#### 4.5.1 什么是跨域

​	浏览器出于安全的考虑，使用 XMLHttpRequest对象发起 HTTP请求时必须遵守同源策略，否则就是跨域的HTTP请求，默认情况下是被禁止的。 同源策略要求源相同才能正常进行通信，即协议、域名、端口号都完全一致。 



#### 4.5.2 CORS解决跨域

​	CORS是一个W3C标准，全称是”跨域资源共享”（Cross-origin resource sharing），允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。

​	它通过服务器增加一个特殊的Header[Access-Control-Allow-Origin]来告诉客户端跨域的限制，如果浏览器支持CORS、并且判断Origin通过的话，就会允许XMLHttpRequest发起跨域请求。

​	

#### 4.5.3 SpringBoot使用CORS解决跨域

##### 1.使用@CrossOrigin

可以在支持跨域的方法上或者是Controller上加上@CrossOrigin注解

~~~~java
@RestController
@RequestMapping("/user")
@CrossOrigin
public class UserController {

    @Autowired
    private UserServcie userServcie;

    @RequestMapping("/findAll")
    public ResponseResult findAll(){
        //调用service查询数据 ，进行返回
        List<User> users = userServcie.findAll();

        return new ResponseResult(200,users);
    }
}

~~~~



##### 2.使用 WebMvcConfigurer 的 addCorsMappings 方法配置CorsInterceptor

~~~~java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
      // 设置允许跨域的路径
        registry.addMapping("/**")
                // 设置允许跨域请求的域名
                .allowedOriginPatterns("*")
                // 是否允许cookie
                .allowCredentials(true)
                // 设置允许的请求方式
                .allowedMethods("GET", "POST", "DELETE", "PUT")
                // 设置允许的header属性
                .allowedHeaders("*")
                // 跨域允许时间
                .maxAge(3600);
    }
}
~~~~



### 4.6 拦截器

#### 4.6.0 登录案例



##### 4.6.0.1 思路分析

​		在前后端分离的场景中，很多时候会采用token的方案进行登录校验。

​		登录成功时，后端会根据一些用户信息生成一个token字符串返回给前端。

​		前端会存储这个token。以后前端发起请求时如果有token就会把token放在请求头中发送给后端。

​		后端接口就可以获取请求头中的token信息进行解析，如果解析不成功说明token超时了或者不是正确的token，相当于是未登录状态。

​		如果解析成功，说明前端是已经登录过的。

##### 4.6.0.2 Token生成方案-JWT

​		本案例采用目前企业中运用比较多的JWT来生成token。

​		使用时先引入相关依赖

~~~~xml
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.0</version>
        </dependency>
~~~~

新建utils包,下面新建JwtUtil类

![image-20220206224426968](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220206224426968.png)		

然后可以使用下面的工具类来生成和解析token,因为是静态方法,直接调用即可

~~~~java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtBuilder;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;
import java.util.Date;
import java.util.UUID;

/**
 * JWT工具类
 */
public class JwtUtil {

    //有效期为
    public static final Long JWT_TTL = 60 * 60 *1000L;// 60 * 60 *1000  一个小时
    //设置秘钥明文
    public static final String JWT_KEY = "sangeng";

    /**
     * 创建token
     * @param id
     * @param subject
     * @param ttlMillis
     * @return
     */
    public static String createJWT(String id, String subject, Long ttlMillis) {

        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        if(ttlMillis==null){
            ttlMillis=JwtUtil.JWT_TTL;
        }
        long expMillis = nowMillis + ttlMillis;
        Date expDate = new Date(expMillis);
        SecretKey secretKey = generalKey();

        JwtBuilder builder = Jwts.builder()
                .setId(id)              //唯一的ID
                .setSubject(subject)   // 主题  可以是JSON数据
                .setIssuer("sg")     // 签发者
                .setIssuedAt(now)      // 签发时间
                .signWith(signatureAlgorithm, secretKey) //使用HS256对称加密算法签名, 第二个参数为秘钥
                .setExpiration(expDate);// 设置过期时间
        return builder.compact();
    }

    /**
     * 生成加密后的秘钥 secretKey
     * @return
     */
    public static SecretKey generalKey() {
        byte[] encodedKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }
    
    /**
     * 解析
     *
     * @param jwt
     * @return
     * @throws Exception
     */
    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(jwt)
                .getBody();
    }


}
~~~~

#### 编写拦截器

以token拦截器为例

```java
@Component
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //获取请求头中的token
        String token = request.getHeader("token");
        if(StringUtils.hasText(token)){

            response.sendError(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }
        try{
            Claims claims = JwtUtil.parseJWT(token);
            String subject=claims.getSubject();
            System.out.println(subject);
        }catch (Exception e){
            e.printStackTrace();
            response.sendError(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }
        //解析token,判断是否成功
        //如果解析过程没有异常,说明是已登录状态
        return true;
    }
}
```

#### 配置拦截器

在config包下新建拦截器配置类

![image-20220206235455580](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220206235455580.png)

```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Autowired
    private LoginInterceptor loginInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor)
                .addPathPatterns("/**");
                //.excludePathPatterns();//配置排除路径
    }
}

```

### 4.7 异常统一处理

#### ①创建类加上@ControllerAdvice注解进行标识

~~~~java
@ControllerAdvice
public class MyControllerAdvice {

}
~~~~

#### ②定义异常处理方法	

​	定义异常处理方法，使用**@ExceptionHandler**标识可以处理的异常。

~~~~java
@ControllerAdvice
public class MyControllerAdvice {

    @ExceptionHandler(RuntimeException.class)
    @ResponseBody
    public ResponseResult handlerException(Exception e){
        //获取异常信息，存放如ResponseResult的msg属性
        String message = e.getMessage();
        ResponseResult result = new ResponseResult(300,message);
        //把ResponseResult作为返回值返回，要求到时候转换成json存入响应体中
        return result;
    }
}
~~~~





### 4.8 获取web原生对象

​	我们之前在web阶段我们经常要使用到request对象，response，session对象等。我们也可以通过SpringMVC获取到这些对象。（不过在MVC中我们很少获取这些对象，因为有更简便的方式，避免了我们使用这些原生对象相对繁琐的API。）

​	我们只需要在方法上添加对应类型的参数即可，但是注意数据类型不要写错了，SpringMVC会把我们需要的对象传给我们的形参。

~~~~java
@RestController
public class TestController {

    @RequestMapping("/getRequestAndResponse")
    public ResponseResult getRequestAndResponse(HttpServletRequest request, HttpServletResponse response, HttpSession session){
        System.out.println(request);
        return new ResponseResult(200,"成功");
    }
}

~~~~

### 4.9 自定义参数解析

​	如果我们想实现像获取请求体中的数据那样，在Handler方法的参数上增加一个@RepuestBody注解就可以获取到对应的数据的话。

​	可以使用HandlerMethodArgumentResolver来实现自定义的参数解析。

①定义用来标识的注解

~~~~java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface CurrentUserId {

}

~~~~

②创建类实现HandlerMethodArgumentResolver接口并重写其中的方法

**注意加上@Component注解注入Spring容器**

~~~~java
public class UserIdArgumentResolver implements HandlerMethodArgumentResolver {

    //判断方法参数使用能使用当前的参数解析器进行解析
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        //如果方法参数有加上CurrentUserId注解，就能把被我们的解析器解析
        return parameter.hasParameterAnnotation(CurrentUserId.class);
    }
    //进行参数解析的方法，可以在方法中获取对应的数据，然后把数据作为返回值返回。方法的返回值就会赋值给对应的方法参数
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        //获取请求头中的token
        String token = webRequest.getHeader("token");
        if(StringUtils.hasText(token)){
            //解析token，获取userId
            Claims claims = JwtUtil.parseJWT(token);
            String userId = claims.getSubject();
            //返回结果
            return userId;
        }
        return null;
    }
}
~~~~

③配置参数解析器

~~~~java
@Configuration
public class ArgumentResolverConfig implements WebMvcConfigurer {

    @Autowired
    private UserIdArgumentResolver userIdArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(userIdArgumentResolver);
    }
}
~~~~



④测试

在需要获取UserId的方法中增加对应的方法参数然后使用@CurrentUserId进行标识即可获取到数据,这个非常好用,每次只需要添加注解就能获取token中的值并赋值给其后面的参数

~~~~java
@RestController
@RequestMapping("/user")
//@CrossOrigin
public class UserController {

    @Autowired
    private UserServcie userServcie;

    @RequestMapping("/findAll")
    public ResponseResult findAll(@CurrentUserId String userId) throws Exception {
        System.out.println(userId);
        //调用service查询数据 ，进行返回s
        List<User> users = userServcie.findAll();

        return new ResponseResult(200,users);
    }
}
~~~~

