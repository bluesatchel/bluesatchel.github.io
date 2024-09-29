---
title: java反序列化学习
date: 2022-03-06 12:34:42
tags:
      - java反序列化
categories: java
---

### java反序列化

##### 反序列化特征<!--more-->

下方的特征可以作为序列化的标志参考:

- 一段数据以`rO0AB`开头，你基本可以确定这串就是Java序列化base64加密的数据。
- 或者如果以`aced`开头，那么他就是这一段Java序列化的16进制。

Java序列化是指把java对象转换为字节序列的过程,而java反序列化是指把字节序列恢复为java对象的过程

序列化分将数据分解成字节流,以便存储在文件中或在网络上传输,反序列化就是打开字节流并重构对象,对象序列化不仅要将基本数据类型转换成字节表示,有时还要恢复数据,恢复数据要求有恢复数据的对象实例

应用场景:

1.想把内存中的对象保存到一个文件中或者数据库中的时候

2.想用套接字在网络上传送对象的时候

3.想通过RMI传输对象的时候



java现在有很多种不同的序列化方式,先从原生的开始

#### java原生序列化

先通过简单的代码来了解java反序列化的流程

```java
public class Person implements Serializable {
    public String name;
    public int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

序列化代码

```java
public class serializeTest {
    public static void serialize(Object obj)throws IOException{
        ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream("C:\\Temp\\test.bin"));
        oos.writeObject(obj);
    }

    public static void main(String[] args) throws IOException {
        Person person = new Person("张三", 29);
        serialize(person);
    }
}

```

反序列化代码

```java
public class unserializeTest {
    public static Object unserialize(String filename) throws IOException, ClassNotFoundException {
        ObjectInputStream ois=new ObjectInputStream(new FileInputStream(filename));
        Object o = ois.readObject();
        return o;
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Person person=(Person)unserialize("C:\\Temp\\test.bin");
        System.out.println(person);
    }
}
```

**transient标记的变量不参与反序列化**





Java反序列化的很多操作,是需要开发者深入参与的,所以你会发现大量的库会实现`readObject`,`writeObject`方法

##### objectAnnotation是什么?



在执行完默认的 `s.defaultWriteObject()` 后，向stream里写入了一个字符串 This is a object 。

然后在radObject中调用readObject()方法取出该字符串并打印

其实`writeObject`和`readObject`方法默认都支持重写,重写之后就不会执行到try catch了

![image-20220323004013073](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220323004013073.png)

它调用的`writeObjectOverride`方法这样描述

![image-20220323004127574](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220323004127574.png)

该方法就是给子类重写默认writeObject方法准备的

```java
public class Person implements java.io.Serializable {
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public String name;
    public int age;
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    private void writeObject(java.io.ObjectOutputStream s) throws
            IOException {
        s.defaultWriteObject();
        s.writeObject("This is a object");
    }
    private void readObject(java.io.ObjectInputStream s)
            throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        String message = (String) s.readObject();
        System.out.println(message);
    }

    public static void main(String[] args) throws Exception{
        Person person = new Person("张三", 19);
        ByteArrayOutputStream buf=new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream= new ObjectOutputStream(buf);
        objectOutputStream.writeObject(person);

        ObjectInputStream objectInputStream=new ObjectInputStream(new ByteArrayInputStream(buf.toByteArray()));
        person=(Person) objectInputStream.readObject();
        System.out.println(person);
    }
}

```

![serializeDu](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/serializeDu.png)

可以在objectAnnotation中看到 This is a object

##### classAnnotations是什么？

`ObjectOutputStream`类中有一个方法`annotateClass`方法,`ObjectOutputStream`的子类需要向序列化后的数据里放任何内容,都可以重写这个方法,写入自己想要写入的数据,然后反序列化的时候就可以读取到这个信息并使用

比如，RMI的 类 `MarshalOutputStream` 就将当前的 `codebase` 写入

所以在分析序列化数据的时候看到`classAnnotations`,实际上就是`annotateClass`方法写入的内容



有些快递打包和拆包有特殊需求,比如易碎朝上,类比重写writeObject和readObject

使用`SerializationDumper`需要的是16进制字符串

##### 16进制字符串和byte相互转换

```java
/* Convert byte[] to hex string.这里我们可以将byte转换成int，然后利用Integer.toHexString(int)来转换成16进制字符串。
 * @param src byte[] data
 * @return hex string
 */
public static String bytesToHexString(byte[] src){
    StringBuilder stringBuilder = new StringBuilder("");
    if (src == null || src.length <= 0) {
        return null;
    }
    for (int i = 0; i < src.length; i++) {
        int v = src[i] & 0xFF;
        String hv = Integer.toHexString(v);
        if (hv.length() < 2) {
            stringBuilder.append(0);
        }
        stringBuilder.append(hv);
    }
    return stringBuilder.toString();
}
/**
 * Convert hex string to byte[]
 * @param hexString the hex string
 * @return byte[]
 */
public static byte[] hexStringToBytes(String hexString) {
    if (hexString == null || hexString.equals("")) {
        return null;
    }
    hexString = hexString.toUpperCase();
    int length = hexString.length() / 2;
    char[] hexChars = hexString.toCharArray();
    byte[] d = new byte[length];
    for (int i = 0; i < length; i++) {
        int pos = i * 2;
        d[i] = (byte) (charToByte(hexChars[pos]) << 4 | charToByte(hexChars[pos + 1]));
    }
    return d;
}
```







##### 为什么会产生安全问题?

只要服务器端反序列化数据,客户端传递类的readObject中代码会自动执行,给予攻击者在服务器上运行代码的能力



可能的形式

1.入口类的readObject直接调用危险方法

2.入口类参数中包含可控类,该类有危险方法,readObject时调用

入口A HashMap 接收参数O

3.入口类参数中包含可控类,该类又调用其他有危险方法的类,readObject时调用

4.构造函数/静态代码块等类加载时隐式执行

直接在readObject进行攻击是不现实的,最好包一个类



共同条件    继承Serializeable

入口类 source (重写readObject   参数类型宽泛   最好jdk自带)   

调用链  





反射在序列化中的应用

定制需要的对象

通过invoke调用除了同名函数以外的函数

通过Class类创建对象,引入不能序列化的类







### JDK动态代理

#### 代理模式

![image-20220307121128637](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220307121128637.png)

##### 代理模式优点

- 职责清晰
- 高扩展，只要实现了接口，都可以用代理。
- 智能化，动态代理。

#### 静态代理

个人理解:**静态代理就是两个实现了同一个接口的类,其中一个类当被代理类,另一个类中实现实例化被代理的类,并在其同名方法中增加自己的个性化内容,必须"收中介费"**,代理类也可以有多个,必须二手房产可以有不同的品牌

以知乎上租房子的例子为例

```java
public interface IRentHouse {
    void rentHouse();
}

--------------------------
public class RentHouse implements IRentHouse{
    @Override
    public void rentHouse() {

        System.out.println("租房子");
    }
}
---------------------------
public class RentHouseProxy implements IRentHouse{

    private IRentHouse rentHouse;

    public RentHouseProxy(IRentHouse rentHouse) {
        this.rentHouse = rentHouse;
    }

    @Override
    public void rentHouse() {
        System.out.println("交中介费");
        rentHouse.rentHouse();
    }
}
--------------------------------
public class test {
    public static void main(String[] args) {
        RentHouse rentHouse=new RentHouse();
        RentHouseProxy rentHouseProxy=new RentHouseProxy(rentHouse);

        rentHouseProxy.rentHouse();


    }
}
最后结果是:
交中介费
租房子
```

##### 优缺点

方便实现

但是当目标类增加了,代理类可能也需要成倍的增加,代理类数量过多,接口增加一个方法,所有实现类都需要增加重写方法

#### 动态代理

**在程序执行过程中,使用jdk的反射机制,创建代理类对象**,并动态的指定要代理目标类,换句话说,动态类是一种创建java对象的能力,帮调用者自动创建对象

动态代理的实现方式常用的有两种:

- 使用jdk代理代理

使用java反射包中的类和接口实现动态代理的功能,反射包 java.lang.reflect ,里面有三个类: InvocationHandler,Method,Proxy

- 通过CGLIB动态代理

cglib是第三方的工具库,创建代理对象,  cglib的原理是继承,     cglib通过继承目标类,创建它的子类,在子类中重写父类中同名的方法,实现功能的修改

要求目标类和方法不能是final的

动态代理则可以根据传递的方法而自动调用对应方法,类似于`反射`中的getMethod和invoke结合起来使用了



首先要定义一个InvocationHandler对象并实现InvocationHandler接口



InvocationHandler(调用处理器)接口,就一个方法invoke(),

​		invoke():表示代理对象要执行的功能代码,代理类要完成的功能就写在invoke()方法中



怎么使用?

1. 创建类实现接口InvocationHandler
2. 重写invoke()方法,把原来静态代理中代理类要完成的功能,写在这里

Method类:表示方法的,确切的说就是目标类中的方法

作用:通过Method可以执行某个目标类的方法 method.invoke(目标对象,方法的参数),这个Invoke和InvocationHandler中的Invoke没有关系



Proxy类 :核心的对象,创建代理对象,之前创建对象都是new 类的构造方法,现在我们是使用Proxy类的方法,代理new的使用

方法:静态方法 newProxyInstance()

作用是:创建代理对象,等同于静态代理中的new RentHouseProxy

```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
```

参数:

1. ClassLoader  loader类加载器,负责向内存中加载对象的,使用反射获取对象的ClassLoader类
2. Class<?>[] interfaces: 接口,目标对象实现的接口,也是反射获取的
3. InvocationHandler h :自己写的代理类要完成的功能

返回值:            就是代理对象



实现动态代理的步骤:

1. 创建接口,定义目标类要完成的功能
2. 创建目标类实现接口
3. 创建InvocationHandler接口的实现类,在invoke方法中完成代理类的功能
   1. 调用目标方法
   2. 增强功能
4. 使用Proxy类的静态方法    创建代理对象,并把返回值转为接口类型

为什么能转换成接口类型,因为它代理的类实现了该接口





```java
public class RentHouseInvocationHandler implements InvocationHandler {
    IRentHouse rentHouse;

    public RentHouseInvocationHandler(IRentHouse rentHouse) {
        this.rentHouse = rentHouse;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        method.invoke(rentHouse,args);
        return null;
    }
}
```

```java
public static void main(String[] args) {
        RentHouse rentHouse=new RentHouse();
        
        RentHouseInvocationHandler rentHouseInvocationHandler = new RentHouseInvocationHandler(rentHouse);
        IRentHouse o = (IRentHouse)Proxy.newProxyInstance(rentHouse.getClass().getClassLoader(), rentHouse.getClass().getInterfaces(), rentHouseInvocationHandler);
        o.rentHouse();

    }
```



动态代理在漏洞利用的用法,

如果某个类是动态代理类,里面调用传入类的f方法,那么就可以传入B从而调用B的f方法

readObject->反序列化自动执行

invoke->有函数调用





#### 动态类加载方法

Class.forName    可以设置是否初始化

ClassLoader.loadClass不进行初始化





ClassLoader->SecureClassLoader->URLClassLoader->AppClassLoader

loadClass->findClass(重写的方法)->defineClass(从字节码加载类)

### 任意类加载

URLClassLoader 

ClassLoader.defineClass 字节码加载类

Unsafe.defineClass 字节码加载    :Spring中有一个可以直接生成的方法



#### URLClassLoader : file http  jar

Test.java

```java
public class Test {
    static {

        try {
            Runtime.getRuntime().exec("calc");
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

##### file协议

```java
public class LoadClassTest {
    public static void main(String[] args) throws Exception {
        URLClassLoader urlClassLoader=new URLClassLoader(new URL[]{new URL("file:///C:\\Temp\\classes\\")});
        Class<?> test = urlClassLoader.loadClass("Test");
        Object o = test.newInstance();
    }
}
```

##### jar协议

jar协议中可以使用http,file

```java
jar -cvf Test.jar .\Test.class   //先编译成class文件再打包
```

```java
public class LoadClassTest {
    public static void main(String[] args) throws Exception {
        URLClassLoader urlClassLoader=new URLClassLoader(new URL[]{new URL("jar:file:///C:\\Temp\\classes\\Test.jar!/")});
        Class<?> test = urlClassLoader.loadClass("Test");
        Object o = test.newInstance();
    }
}
```

#### DefineClass通过字节码加载类

**defineClass只是将字节码转换为Class,而不会主动加载类,需要再次实例化,才能调用到静态代码块**

```java
Method defineClass = ClassLoader.class.getDeclaredMethod("defineClass", String.class, byte[].class, int.class, int.class);
        defineClass.setAccessible(true);
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        byte[] code= Files.readAllBytes(Paths.get("C:\\Temp\\classes\\Test.class"));
        Class o= (Class)defineClass.invoke(systemClassLoader, "Test", code, 0, code.length);
        o.newInstance();
```

#### loadClass,findClass,defineClass区别

##### loadClass

- findLoadedClass(String) 调用这个方法，查看这个Class是否已经被加载

- 如果没有被加载，继续往下走，查看父类加载器，递归调用loadClass()

- 如果父类加载器是null，说明是启动类加载器，查找对应的Class

- 如果都没有找到，就调用findClass(String)

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

##### findClass()

- 根据名称或位置加载.class字节码,然后使用defineClass
- 通常由子类去实现

##### defineClass()

- 把字节码转化为Class



#### Unsafe

Unsafe类采用了单例模式设计,但是它其中也有defineClass方法

```java
Class<Unsafe> unsafeClass = Unsafe.class;
        Field theUnsafe = unsafeClass.getDeclaredField("theUnsafe");
        byte[] code= Files.readAllBytes(Paths.get("C:\\Temp\\classes\\Test.class"));
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        theUnsafe.setAccessible(true);
        Unsafe unsafe=(Unsafe) theUnsafe.get(null);
        Class test=(Class) unsafe.defineClass("Test",code,0,code.length,systemClassLoader,null);
        test.newInstance();
```

















