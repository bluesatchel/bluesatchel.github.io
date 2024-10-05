---
title: upload-labs
date: 2021-12-05 13:30:37
tags:
    - upload
    - upload-labs
categories: upload

---

### upload-labs笔记

##### pass-01

查看网页源代码发现是js本地判断文件格式,禁用js即可上传成功

##### pass-02

抓包分析,更改Content-Type为image/jpeg即可

##### pass-03

查看代码后发现是黑名单禁止,并且大小写,空格,加点,换行均不能绕过,所以考虑到使用php3,phtml绕过,但是执行需要服务器开启响应扩展才行..局限性:默认是无法解析这种后缀的,而且只在php5.2.17上可以开启并实现,其余版本均失败

![image-20241004211929155](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/202410042119188.png)

##### pass-04

利用apache解析漏洞,上传后缀为php.xxx,当其无法解析时,会向前寻找到正确的后缀并进行解析或者使用.htaccess文件让apache以php方式进行解析图片格式的文件

使用这种写法可以精准解析该文件名的文件

![image-20241004212504493](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/202410042125519.png)

##### pass-05

通过对源码的阅读,发现少了字母一律转换成小写的代码,所以可以使用phP进行绕过同理使用.htaccess文件也可进行大写绕过

##### pass-06

通过尝试发现空格即可绕过,通过对源码的阅读印证了想法,未在源码中添加首尾去空的功能

##### pass-07

本题禁止上传所有带有可以解析的后缀的文件利用在文件名末尾添加点绕过没有对后缀名末尾的点进行处理，利用windows特性，会自动去掉后缀名中最后的”.”，可在后缀名中加”.”绕过：pass-08::$DATA绕过**没有对后缀名中的’::$DATA’进行过滤。在php+windows的情况下：如果文件名+"::$DATA"会把::$DATA之后的数据当成文件流处理,不会检测后缀名.且保持"::$DATA"之前的文件名。利用windows特性，可在后缀名中加” ::$DATA”绕过:

![image-20241004211945904](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/202410042119930.png)

##### pass-09

利用.加空格绕过代码先是去除文件名前后的空格，再去除文件名最后所有的.，再通过strrchar函数来寻找.来确认文件名的后缀，但是最后保存文件的时候没有重命名而使用的原始的文件名，导致可以利用1.php. .（点+空格+点）来绕过

![image-20241004212153923](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/202410042121971.png)

##### pass-10

分析代码可知代码使用了黑名单设置,对文件名中包含的黑名单中的后缀使用trim函数进行移除,所以思路就是写一个被移除后正好能变成可解析后缀的文件名即可,如,pphphp

![image-20241004212055562](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/202410042120596.png)

##### pass-11

使用白名单只允许特定后缀 白名单判断，使用jpg格式后缀即可,但$img_path是直接拼接，因此可以利用%00截断,利用创建路径的函数修改文件名和后缀%00在解码后是空字符

![image-20241004212114248](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/202410042121277.png)

使用00%截断绕过(截断条件：php版本小于5.3.4，php的magic_quotes_gpc为OFF状态)

![image-20241004212129298](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/202410042121323.png)

##### pass-12

和上个pass思路一样,因为存储路径可控,所以一样使用00截断,只不过 这次的路劲通过POST方式传递,不会对%00进行url解码,所以需要在burp suite的hex功能点处,找到对应位置,将其修改为00

##### pass-13

该pass检查文件开头的标识,所以用图片马过需要配合文件包含漏洞或者解析漏洞

##### pass-14

该pass通过getimagesize($filename);函数判断是否为图片需要配合文件包含漏洞或者解析漏洞

##### pass-17

因为是先上传再判断,所以存在条件竞争重复发上传的数据包,在存在的一瞬间访问到就行

##### pass-18

条件竞争,并且先判断了后缀再移动,上传图片马,利用文件包含漏洞

##### pass-19

1.上传名称可控,但是是黑名单判断,可以写xx.php%00.jpg用%00截断

2.将保存名称写为xxx.php/.绕过黑名单,并且还会保存为xxx.php
