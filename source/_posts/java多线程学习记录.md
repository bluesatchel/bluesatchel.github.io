---
title: java多线程学习记录
date: 2022-08-04 22:46:29
tags:
      - java
categories: java
description: java多线程学习记录
---

# 多线程学习记录

## 1.线程概述

### 线程的常用方法

##### currentThread方法

java中的任何一段代码都是执行在某个线程当中的,执行当前代码的线程就是 当前线程

同一段代码可能被不用的线程执行,因此当前线程是相对的,Thread.currentThread()方法的返回值是在代码实际运行时候的

大概来说就是调用某个方法的线程就是其currentThread()

##### isAlive()

判断当前线程是否处于活动状态(线程已启动且尚未终止)

##### sleep()

让当前线程休眠指定的毫秒数

##### getId()

返回线程的唯一编号

##### yield()

作用是放弃当前的cpu资源

##### setPriority(num)

1<=num<=10

本质上是给线程调度器一个提示信息,以便于调度器决定先调度哪些线程,不能完全保证优先级高的先运行

##### interrupt()

仅仅是跟线程打一个中断标志

##### setDaemon()

java中的线程分为用户线程与守护线程

守护线程是为其他线程提供服务的线程,如GC就是一个典型的守护线程

守护线程不能单独运行,当JVM中没有其他用户线程,只有守护线程时,守护线程会自动销毁,jvm会退出

**启动**之前就设置为守护线程

### 线程的生命周期

可以通过`getState()`方法来获取是Thread.State枚举类型定义的,有以下几种:

| Thread.State的值 | 具体含义                                                     |
| ---------------- | ------------------------------------------------------------ |
| NEW              | 创建了线程对象但还没有启动                                   |
| RUNNABLE         | 它是一个复合状态,包含READY和RUNNING两个状态,READY表示可以被调度器调度从而使它处于RUNNING状态 |
| BLOCKED          | 阻塞状态,线程发起阻塞的I/O操作,或者申请由其他线程占用的独占资源,线程会转换为BLOCKED阻塞状态,处于阻塞状态的线程不会占用CPU资源,当阻塞I/O操作执行完,或者线程获得了由其申请的资源,线程就可以转换为RUNNABLE状态 |
| WAITING          | 线程执行了Object.wait方法,或者Thread.join方法,执行Object.notify方法或者加入的线程执行完毕,就会转换为RUNNABLE状态 |
| TIMED_WAITING    | 一般调用Thread.sleep或者Object.wait()方法进入,与WAITING有点像,但是这个状态如果超过指定的时间范围,就会自动转换为RUNNABLE |
| TERMINATED       | 线程结束,处于终止状态                                        |

![image-20220805135545694](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220805135545694.png)

### 多线程存在的问题与风险

#### 线程安全

多线程共享数据时,如果没有采取正确的并发访问控制措施,就可能会产生数据一致性问题,如读脏数据(过期的数据),如丢失数据更新

#### 线程活性

由于程序自身的缺陷或者由资源稀缺性导致线程一直处于非RUNNABLE状态,这就是线程活性问题,常见的活性故障有以下几种

##### 死锁(Deadlock)

最简单的例子是：线程 A 占有一号锁，正在请求二号锁，且线程 B 占有二号锁，正在请求一号锁，A、B 线程互相等待对方释放锁，进入了无线等待的状态。

##### 锁死(Lockout)

等待线程由于唤醒其所需的条件永远无法成立，或者其他线程无法唤醒这个线程而一直处于非运行状态（线程并未终止）导致其任务 一直无法进展

**类似于睡美人故事中王子挂了**

##### 活锁(Livelock)

活锁是指正在执行的线程或进程没有发生阻塞，但由于某些条件没有满足，导致反复重试-失败-重试-失败的过程。与死锁最大的区别在于：活锁状态的线程或进程是一直处于运行状态的，在失败中不断重试，重试中不断失败，一直处于所谓的“活”态，不会停止。

##### 饥饿(starvation)

资源总是被别的线程抢走了

#### 上下文切换

#### 可靠性

可能会由一个线程导致JVM意外终止,其他的线程也无法执行



## 2.线程安全问题

线程安全主要表现为三个方面:原子性,可见性和有序性

#### 原子性(Atomic)

就是不可分割的意思,原子操作的不可分割有两层含义:

- 访问(读,写)某个共享变量的操作从其他线程来看,这个操作要么已经执行完毕,要么尚未发生,其他线程看不到当前这个操作的中间结果
- 访问同一组共享变量的原子操作时不能够交叉的

java有两种方式实现原子性:一种是使用锁,另一种是利用处理器的CAS指令

- 锁具有排他性,保证共享变量在某一时刻只能被一个线程访问

- CAS指令直接在硬件(处理器和内存)层次上实现,看做是硬件锁

#### 可见性(visibility)

在多线程环境中,一个线程对某个共享变量进行更新之后,后续其他的线程可能无法立即读取到这个更新的结果,这就是线程安全问题的另外一种形式

如果一个线程对共享变量更新后,后序访问该变量的其他线程可以读取到更新的结果,称这个线程对共享变量的更新对其他线程可见,否则称这个线程对共享变量的更新对其他线程不可见

#### 有序性(Ordering)

有序性是指在什么情况下一个处理器上运行的一个线程所执行的,内存访问操作在另外一个处理器运行的其他线程看起来是乱序的

乱序是指内存访问操作的顺序看起来发生了变化

##### 重排序

在多核处理器的环境下,编写的顺序结构,这种操作执行的顺序可能是没有保障的:

编译器可能会改变两个操作的先后顺序

处理器也可能不会按照目标代码的顺序执行

这种一个处理器上执行的多个操作,在其他处理器看来它的顺序与目标代码指定的顺序可能不一样,这种现象称为重排序

重排序是对内存访问有序操作的一种优化,可以在不影响单线程顺序正确的情况下提升程序的性能,但是可能对多线程程序的正确性产生影响,即可能导致线程安全问题

##### 与内存操作顺序有关的几个概念

源代码顺序:源码中指定的内存访问顺序

程序顺序:处理器上运行的目标代码所指定的内存访问顺序

执行顺序:内存访问操作在处理器上的实际执行顺序

感知顺序:给定处理器所感知到的该处理器及其他处理器的内存访问操作的顺序



可以把重排序分为指令重排序与存储子系统重排序两种

- 指令重排序

在源码顺序与程序顺序不一致,或者程序顺序与执行顺序不一致的情况下,我们就说发生了指令重排序

指令重拍是一种动作,确实对指令的顺序做了调整,重排序的对象是指令

javac编译器一般不会执行指令重排序,而jit编译器可能执行指令重排序

处理器也可能执行指令重排序,使得执行顺序与程序顺序不一致

指令重排序不会对单线程程序的结果产生影响,可能导致多线程程序出现非预期的结果

- 存储子系统重排序

存储子系统是指写缓冲器与高速缓存

高速缓存是CPU中为了匹配与主内存处理速度不匹配而设计的一个高速缓存,写缓冲器用来提高写告诉缓存操作的效率

即使处理器严格按照程序顺序执行两个内存访问操作,在存储子系统的作用下,其他处理器对这两个操作的感知顺序也可能与程序顺序不一致,即这两个操作的顺序看起来像是发生了变化,这种现象称为存储子系统重排序

存储子系统重排序对象是内存操作的结果

内存重排序分类

从处理器角度来看,读内存就是从指定的RAM地址中加载数据到寄存器,称为Load操作;写内存就是把数据存储到指定的地址表示的RAM存储单元中,称为Store操作,内存重排序有以下四种可能

LoadLoad重排序,一个处理器先后执行两个读操作L1和L2,其他处理器对两个内存操作的感知顺序可能是L2,L1

StoreStore重排序,一个处理器先后执行两个读操作W1和W2,其他处理器对两个内存操作的感知顺序可能是W2,W1

同理还有两种

LoadStore和StoreLoad重排序

##### 貌似串行语义

JIT编译器,处理器,存储子系统是按照一定的规则对指令内存操作的结果进行重排序,给单线程程序造成一种假象---指令是按照源码的顺序执行的,这种假象称为貌似串行语义

并不能保证多线程环境下程序的正确性

为了保证貌似串行语义,有数据依赖关系的语句不会被重排序,只有不存在数据依赖关系的语句才会被重排序

存在控制依赖关系的语句允许重拍,一条语句(指令)的执行结果会决定另一条语句(指令)能否被执行,这两条语句(指令)存在控制依赖关系,如在if语句中允许重排,可能存在处理器先执行if代码块,再判断If条件是否成立

##### 保证内存访问的顺序性

可以使用volatile关键字,synchronized关键字实现有序性

### 内存

1.每个线程都有独立的栈空间

2.每个线程都可以访问堆内存

3.计算机的cpu不直接从内存读取数据,读的是从主存映射到cache的数据

4.jvm中的共享的数据可能会被分配到Register寄存器中,每个cpu都有自己的register寄存器,一个cpu不能读取其他cpu寄存器中的内容,如果两个线程分别运行在不同的处理器上

5.即使jvm中的共享数据分配到主内存中,也不能保证数据的可见性,cpu不直接对主内存访问,而是通过cache高速缓存进行的,一个处理器上运行的线程对数据的更新可能只是更新到处理器的写缓冲器中,还没有到达cache缓存,更不用说主内存了,另外一个处理器不能读取到该处理器写缓冲器上的内容,会产生运行在另外一个处理器上的线程无法看到该处理器对共享数据的更新

6.一个处理器的cache不能直接读取另外一个处理器的cache,但是一个处理器可以通过缓存一致性协议(Cache Coherence Protocol)来读取其他处理器缓存中的数据,并将读取到的数据更新到该处理器的cache中,这个过程称为缓存同步,缓存同步使得一个处理器上运行的线程可以读取到另外一个处理器上运行的线程对共享数据的所做的更新,即保障了可见性

**为了保障可见性,必须使一个处理器对共享数据的更新最终被写入该处理器的cache中,这个过程称为冲刷处理器缓存**





![image-20220805225223525](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220805225223525.png)

## 3.线程同步

### 锁概述

将多个线程对共享数据的并发访问转换为串行访问,即一个共享数据一次只能被一个线程访问

锁可以理解为对共享数据进行保护的一个许可证,对于同一个许可证保护的共享数据来说,任何相乘想要访问这些共享数据必须先持有该许可证,一个线程只有在持有许可证的情况下才能对这些共享数据进行访问,并且一个许可证一次只能被衣蛾线程持有,许可证线程在结束对共享数据的访问后必须释放其持有的许可证

一个线程在访问共享数据前必须先获得锁,获得锁的线程称为锁的持有线程,一个锁一次只能被一个线程持有.锁的持有线程在获得锁之后和释放锁之前这段时间锁执行的代码称为临界区(Critical Section)



锁具有排他性,即一个锁一次只能被一个线程持有,这种锁称为排它锁或互斥锁



jvm把锁分为内部锁和显示锁两种,内部锁通过synchronized关键字实现;显示锁通过Lock接口的实现类来实现



锁能够保障有序性,写线程在临界区所执行的在读线程所执行的临界区看来像是完全按照源码顺序执行的

注意:

使用锁保障线程的安全性,必须满足以下条件:

**这些线程在访问共享数据时必须使用同一个锁**

**即使是读取共享数据的线程也需要使用同步锁**

**锁的获得隐含着刷新处理器缓存的动作,释放锁隐含着处理器缓存冲刷的动作**(拿进来改完再扔回去)

- **冲刷处理器缓存**:当一个处理器对共享变量进行更新后,必须让它的更新最终被写入到高速缓冲或者主内存中.
- **刷新处理器缓存**:当处理器操作一个共享变量的时候,其它处理器在此之前已经对这个共享变量进行了更新,那么必须要对高速缓存或者主内存进行缓存同步.

#### 锁相关的概念

##### 可重入性

描述这样一个问题:一个线程持有该锁的时候能再次申请该锁

如果一个线程持有一个锁的时候还能够继续成功申请该锁,称该锁是可重入的,否则就称该锁为不可重入的

##### 锁的争用与调度

java平台中内部锁属于非公平锁,显示Lock锁既支持公平锁又支持非公平锁

##### 锁的粒度

一个锁可以保护的共享数据的数量大小称为锁的粒度

锁保护共享数据量大,称该锁的粒度粗,否则就称该锁的粒度细

锁的粒度过粗会导致线程在申请锁时会进行不必要的等待,锁的粒度过细会增加锁调度的开销

##### 内部锁synchronized关键字

java中的每个对象都有一个与之关联的内部锁,这种锁也称为监视器,这种内部锁是一种排他锁,可以保障原子性,可见性与有序性

内部锁是通过synchronized关键字实现的,synchronized关键字可以修饰代码块,修饰方法

修饰代码块的语法:

synchronized(锁对象,一般用this){

同步代码块,可以在同步代码块中访问共享数据

}

synchronized修饰实例对象,默认**this**作为锁对象

synchronized修饰静态方法,默认**运行时类**作为锁对象



**同步方法锁的粒度粗,同步代码块锁的粒度细**



#### 脏读

不仅对修改数据的代码块进行同步,还要对读取数据的代码块进行同步

如果不这样操作,可能会出现脏读的问题



#### 线程出现异常

线程出现异常会自动释放锁

#### 死锁

如何避免死锁?

所有线程获得锁的顺序保持一致即可



### 轻量级同步机制:volatile关键字

volatile关键字可以强制线程从公共内存中读取变量的值,而不是从工作内存中读取

##### volatile与synchronized关键字比较

1.volatile关键字是线程同步的轻量级实现,所以volatile性能肯定比synchronized要好,volatile只能修饰变量,而synchronized可以修饰方法,代码块,随着jdk新版本的发布,synchronized的执行效率也就较大的提升,在开发中使用synchronized的比率还是很大的

2.多线程访问volatile变量不会发生阻塞,而synchronized可能会阻塞

3.volatile能保证数据的可见性,但是不能保证原子性,而synchronized既可以保证原子性,也可以保证可见性

4.关键字volatile解决的是变量在多个线程之间的可见性,  synchronized解决的是多个线程之间访问公共资源的同步性

##### volatile不能保证原子性的例子

是直接公共内存拿过来的,说不定其他线程对其的操作还未结束,拿到的是中间值

```java
public class test1 {
    public static void main(String[] args) {
        for(int i=0;i<10;i++){
            new MyThread().start();
        }
    }
    static class MyThread extends Thread{
        volatile  public static int num;
        public static void add(){
            for(int i=0;i<1000;i++){
                num++;
            }
            System.out.println("num==="+num);
        }
        @Override
        public void run() {
            add();
        }
    }
}
```

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220812155935083.png" alt="image-20220812155935083" style="zoom:67%;" />

```java
///改为synchronized即可保证原子性
public class test1 {
    public static void main(String[] args) {
        for(int i=0;i<10;i++){
            new MyThread().start();
        }
    }
    static class MyThread extends Thread{
        public static int num;
        public synchronized static void add(){
            for(int i=0;i<1000;i++){
                num++;
            }
            System.out.println("num==="+num);
        }
        @Override
        public void run() {
            add();
        }
    }
}
```

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220812160208655.png" alt="image-20220812160208655" style="zoom:67%;" />

### CAS(Compare And Swap)原理

是由硬件实现的

CAS可以将read-modify-write这类的操作转换为原子操作

i++自增包括三个子操作

- 从主内存读取i变量值
- 对i的值加1
- 再把加1后的值保存到主内存

CAS原理:在把数据更新到主内存时,再次读取主内存变量的值,如果现在变量的值与期望的值(操作起始时读取的值)一样就更新

![image-20220812164219675](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220812164219675.png)

使用代码模拟CAS算法

```java
public class casTest {

    public static void main(String[] args) {
        final CASCounter casCounter=new CASCounter();
        for(int i=0;i<100;i++){
            new Thread(() -> System.out.println(casCounter.incrementAndGet())).start();
        }
    }
}
class CASCounter{
    volatile private long value;
    public long getValue() {
        return value;
    }
    //定义一个Compare and swap方法
    private boolean cas(long expectedValue,long newValue){
        synchronized (this){
            if(value==expectedValue){
                value=newValue;
                return true;
            }else{
                return false;
            }
        }
    }
    public long incrementAndGet(){
        long oldValue;
        long newValue;
        do{
            oldValue=value;
            newValue=oldValue+1;
            //cas() return false之后撤销本次递增,再增加一次
        }while (!cas(oldValue,newValue));
        return value;
    }
}
```

CAS实现原子操作背后有一个假设:共享变量的当前值与当前线程提供的期望值相同,就认为这个变量没有被其他线程修改过

实际上这种假设不一定总是成立,如有共享变量count=0

A线程将count修改为10,B线程将count修改为20,C线程将count值修改为0

当前线程看到count的值为0,**不能简单的认为count的值没有被其他线程更新**

这就是CAS中的ABA问题,即共享变量经历了A->B->A,是否能够接受ABA问题跟实现的算法有关,如果想要规避ABA问题,可以为共享变量引入一个修订号(时间戳),每次修改共享变量时,相应的修订号发生变化AtmicStampedReference类就是基于这种思想产生的

 

#### 原子变量类概述

原子变量类基于CAS实现,其内部就是借助一个volatile变量

| 分组       | 原子变量类                                                   |
| ---------- | ------------------------------------------------------------ |
| 基础数据型 | AtomicInteger,AtomicLong,AtomicBoolean                       |
| 数组型     | AtomicArray,AtomicLongArray,AtomicReferenceArray             |
| 字段更新器 | AtomicIntegerFieldUpdater,AtomicLongFieldUpdater,AtomicReferenceFieldUpdater |
| 引用型     | AtomicReference,AtomicStampedReference,AtomicMarkableReference |

##### AtomicIntegerFieldUpdater

AtomicIntegerFieldUpdater可以对原子整数字段进行更新,要求:

1.字符必须使用volatile修饰,使线程之间可见

2.只能是实例变量,不能是静态变量,也不能使用final修饰

例子

```java
public class test {
    public static void main(String[] args) {
        User user=new User(1,5);
        for (int i = 0; i < 5; i++) {
            new SubThread(user).start();

        }
        System.out.println(user);
    }
}
class SubThread extends Thread{
    private User user;
    private AtomicIntegerFieldUpdater<User> updater=AtomicIntegerFieldUpdater.newUpdater(User.class,"age");
    public SubThread(User user){
        this.user=user;
    }
    @Override
    public void run() {
        for (int i = 0; i <10; i++) {
            System.out.println(updater.getAndIncrement(user));
        }
    }
}
class User{
    int id;
    volatile int age;
    @Override
    public String toString() {
        return "test{" +
                "id=" + id +
                ", age=" + age +
                '}';
    }
    public User(int id, int age) {
        this.id = id;
        this.age = age;
    }
}
```

类似的几种也是这样的使用方式

##### AtomicReference可能会出现CAS的ABA问题

```java
public class test {

    private static AtomicReference<String> atomicReference=new AtomicReference<>("abc");

    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(new Runnable() {
            @Override
            public void run() {
                atomicReference.compareAndSet("abc","def");
                System.out.println(Thread.currentThread().getName()+"---"+atomicReference.get());
                atomicReference.compareAndSet("def","abc");
            }
        });
        Thread t2=new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(atomicReference.compareAndSet("abc","hello"));
            }
        });
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(atomicReference.get());
    }
}
```

![image-20220812224758071](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220812224758071.png)

##### 使用AtomicStampReference中的版本号来解决ABA问题

```java

public class test {

    private static AtomicStampedReference<String> stampedReference=new AtomicStampedReference<>("abc",0);

    public static void main(String[] args) throws InterruptedException {

        Thread t1=new Thread(new Runnable() {
            @Override
            public void run() {
                stampedReference.compareAndSet("abc","def",stampedReference.getStamp(),stampedReference.getStamp()+1);
                System.out.println(Thread.currentThread().getName()+"----"+stampedReference.getReference());
                stampedReference.compareAndSet("def","abc",stampedReference.getStamp(),stampedReference.getStamp()+1);
            }
        });

        Thread t2=new Thread(new Runnable() {
            @Override
            public void run() {
                int stamp=stampedReference.getStamp();
                //先获得了版本号,但是sleep期间上面的线程导致版本号+1了
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                //此处版本号不一致,所以并没有做出修改
                System.out.println(stampedReference.compareAndSet("abc","hello",stamp,stamp+1));
            }
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(stampedReference.getReference());
    }

}
```

![image-20220812224743210](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220812224743210.png)

## 4.线程间的通信

#### 等待/通知机制

##### 概念

在多线程编程中,可能线程A的条件没有满足只是暂时的,稍后其他的线程B可能会更新条件使得A线程的条件得到满足,可以将A线程暂停,直到它的条件得到满足后,再将A线程唤醒

#### 实现

##### Object#wait()

`Object#wait()`方法可以是执行当前代码的线程等待,暂停执行直到接到通知或被中断为止

**注意:**

1. **wait()方法只能在同步代码块中由锁对象调用**
2. **调用wait()方法,当前线程会释放锁**

伪代码如下

```java
//在调用wait()方法前获得对象的内部锁
synchronized(锁对象){
    while(条件不成立){
        //通过锁对象调用wait()方法暂停线程,会释放锁对象
        锁对象.wait();
    }
    //线程的条件满足了继续向下执行
}
```

wait(long)也可以提供一个毫秒让它到时间自动唤醒

##### Object#notify()

Object类的notify()可以唤醒线程,**该方法也必须在同步代码块中由锁对象调用**,没有使用锁对象调用wait()/notify()会抛出IllegalMonitorStateException异常,如果有多个等待的线程,notify()方法只能唤醒其中的一个,**在同步代码块中调用nitify()方法后,并不会立即释放锁对象,需要等当前同步代码看执行完后才会释放锁对象,一般将notify方法放在同步代码块的最后**

伪代码如下

```java

synchronized(锁对象){
    //执行修改保护条件的代码
    //唤醒其他线程
    锁对象.notify();   
}
```

简单的例子

```java
public class test1 {

    public static void main(String[] args) throws InterruptedException {
        String lock="hello!!!";//定义一个String类作为锁对象
        Thread t1=new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock){
                    System.out.println("线程1开始等待"+System.currentTimeMillis());
                    try {
                        lock.wait();//线程等待,会释放锁对象,线程进入blocked阻塞状态
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("线程1等待结束"+System.currentTimeMillis());

                }
            }
        });
        Thread t2=new Thread(new Runnable() {
            @Override
            public void run() {
                //notify方法也需要在同步代码块中由锁对象调用
                synchronized (lock){
                    System.out.println("线程2开始唤醒"+System.currentTimeMillis());

                    lock.notify();//唤醒在lock锁对象上等待的某一个线程
                    System.out.println("线程2唤醒结束"+System.currentTimeMillis());
                }
            }
        });
        t1.start();
        Thread.sleep(3000);
        t2.start();
    }
```

![image-20220815175756390](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220815175756390.png)

##### notify不会立即释放锁对象

**唤醒线程不会立即释放锁对象,需要等到当前同步代码块都执行完后才能释放锁对象**

```java
public class test2 {
    public static void main(String[] args) throws InterruptedException {
        List<String> list=new ArrayList<>();
        Thread t1=new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (list){
                    if(list.size()!=5){
                        System.out.println("线程1开始等待"+System.currentTimeMillis());
                        try {
                            list.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        System.out.println("线程1被唤醒"+System.currentTimeMillis());
                    }
                }
            }
        });
        Thread t2=new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (list){
                    for (int i=0;i<10;i++){
                        list.add("string"+i);
                        System.out.println("t2添加了第"+i+"个数据");
                        if(list.size()==5){
                            list.notify();
                            System.out.println("线程2已结发起唤醒通知");
                        }
                    }
                }
            }
        });
        t1.start();
        Thread.sleep(500);//保证线程1先执行
        t2.start();
    }
}
```

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220816003428991.png" alt="image-20220816003428991" style="zoom:80%;" />

##### interrupt()方法会中断wait()

当线程处于wait()等待状态时, 调佣线程对象的interrupt()方法会中断线程的等待状态,会产生Interrupted

中断线程会唤醒线程等待

```java
public class test3 {
    private static final Object LOCK=new Object();
    public static void main(String[] args) {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (LOCK){
                    try {
                        System.out.println("等待开始");
                        LOCK.wait();
                        System.out.println("等待结束");
                    } catch (InterruptedException e) {
                        System.out.println("wait被中断了");
                        e.printStackTrace();
                    }
                }
            }
        });
        t1.start();
        t1.interrupt();
    }
}
```

![image-20220816140527344](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220816140527344.png)

##### notify()与notifyAll()

锁对象.notify()一次只能唤醒一个线程,如果有多个等待的线程,只能随机唤醒其中的某一个,想要唤醒所有等待的线程,需要调用锁对象.notifyAll()

##### 通知过早

如果先开启通知线程,再开启等待线程,可能会出现t1线程等待没有收到通知的情况

比较好的解决办法就是设置一个isFirst的静态布尔变量,当线程是第一个开启的就等待

```java
Thread1{
    while(isFirst){
    	锁对象.lock()
	}
}
Thread2{
    锁对象.notify()
    isFirst=false;
}

```

##### wait条件发生了变化

在使用wait/notify模式时,注意wait条件发生了变化,也可能会造成程序逻辑的混乱





先开启添加数据的线程,再开启一个取数据的线程,大多数情况下会正常取数据

先开启取数据的线程,再开启添加数据的线程,取数据的线程会先等待,等到添加数据之后nofity后再取数据

```java
public class test4 {
    static List list=new ArrayList<>();
    public static void subtract(){
        synchronized (list){
            if(list.size()==0){
                try {
                    System.out.println(Thread.currentThread().getName()+"开始等待");
                    list.wait();
                    System.out.println(Thread.currentThread().getName()+"结束等待");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
              }
            Object o = list.remove(0);
            System.out.println(Thread.currentThread().getName()+"从集合中取了"+o+",,,剩余数量为"+list.size());

        }
    }
    public static void add(){
        synchronized (list){

            list.add("datum");
            System.out.println(Thread.currentThread().getName()+"添加了一条数据");
            list.notifyAll();
        }
    }
    static class SubtractThread extends Thread{
        @Override
        public void run() {
            subtract();
        }
    }
    static class AddThread extends Thread{

        @Override
        public void run() {
            add();
        }
    }
    public static void main(String[] args) throws InterruptedException {


        SubtractThread subtractThread1=new SubtractThread();
        SubtractThread subtractThread2=new SubtractThread();
        subtractThread1.setName("subtractThread1");
        subtractThread2.setName("subtractThread2");
        AddThread addThread1=new AddThread();
        addThread1.setName("addThread1");

        //开启两个取数据的线程,再开启添加数据的线程
        subtractThread1.start();
        subtractThread2.start();
        addThread1.start();
    }
}
```

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220817201024794.png" alt="image-20220817201024794" style="zoom: 80%;" />

当等待的线程被唤醒后,需要再判断一次集合中是否有数据可取,即需要把subtract()方法中的if判断改为while,改完后上述代码执行结果发生如下转变

![image-20220817201159396](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220817201159396.png)

#### 生产者消费者设计模式

有多个线程的时候为了防止取不到值的问题,把if改为while

ValueOP

```java
package demo.producerTest;
public class ValueOP {
    private String value="";
    //定义方法修改value的值
    public void setValue(){
        synchronized (this){
            //如果不是空串就等待
            while (!value.equalsIgnoreCase("")){
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //如果value值是空串,就设置value字段的值
            String value=System.currentTimeMillis()+"--"+System.nanoTime();
            System.out.println("set设置的值是:"+value);
            this.value=value;
            this.notify();
        }
    }
    public void getValue(){
        synchronized (this){
            //是空串就等待
            while (value.equalsIgnoreCase("")){
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //不是空串,读取字段值
            System.out.println("get的值是:"+value);
            this.value="";
            this.notify();
        }
    }
}
```

ProducerThread

```java
package demo.producerTest;
public class ProducerThread extends Thread{
    private ValueOP obj;
    public ProducerThread(ValueOP obj) {
        this.obj = obj;
    }
    @Override
    public void run() {
        while (true){
            obj.setValue();
        }
    }
}
```

ConsumerThread

```java
package demo.producerTest;
public class ConsumerThread extends Thread {
    private ValueOP obj;
    public ConsumerThread(ValueOP obj) {
        this.obj = obj;
    }
    @Override
    public void run() {
        while (true){
            obj.getValue();
        }
    }
}
```

测试类

```java
public static void main(String[] args) {
        ValueOP valueOP=new ValueOP();
        ProducerThread p1=new ProducerThread(valueOP);
        ProducerThread p2=new ProducerThread(valueOP);
        ProducerThread p3=new ProducerThread(valueOP);
        ConsumerThread c1=new ConsumerThread(valueOP);
        ConsumerThread c2=new ConsumerThread(valueOP);
        ConsumerThread c3=new ConsumerThread(valueOP);

        p1.start();
        c1.start();
        p2.start();
        c2.start();
        p3.start();
        c3.start();
    }
```

![image-20220818165125092](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220818165125092.png)

但是上述代码可能会出现线程假死的状态,所有线程一直处于等待状态

大概原因是锁对象.notify()是随机唤醒一个线程,有很大可能不是按照预期的,消费结束就去唤醒生产者线程

所以需要将上述ValueOP类代码中的notify()改为notifyAll()

```java
public class ValueOP {
    private String value="";
    //定义方法修改value的值
    public void setValue(){
        synchronized (this){
            //如果不是空串就等待
            while (!value.equalsIgnoreCase("")){
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //如果value值是空串,就设置value字段的值
            String value=System.currentTimeMillis()+"--"+System.nanoTime();
            System.out.println("set设置的值是:"+value);
            this.value=value;
            this.notifyAll();
        }
    }
    public void getValue(){
        synchronized (this){
            //是空串就等待
            while (value.equalsIgnoreCase("")){
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //不是空串,读取字段值
            System.out.println("get的值是:"+value);
            this.value="";
            this.notifyAll();
        }
    }
}
```

##### 模拟栈

相比上面的操作方式,还可以通过模拟栈的方式,来安排消费者和生产者

MyStack.java

```java
//定义类模拟栈
public class MyStack {
    private List list= new ArrayList();
    private static final int MAX=1;
    public synchronized void push(){
        //当栈已满,就等待\
        while (list.size()>=MAX){
            System.out.println(Thread.currentThread().getName()+"开始等待");
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        String data="data---"+Math.random();
        System.out.println(Thread.currentThread().getName()+"添加了数据"+data);
        list.add(data);
        this.notifyAll();
    }
    public synchronized void pop(){
        while (list.size()==0){
            try {
                System.out.println(Thread.currentThread().getName()+"开始等待");
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(Thread.currentThread().getName()+"出栈数据:  "+list.remove(0));
        this.notifyAll();
    }
}
```

#### 通过管道实现线程间的通信

通过管道在两个线程之间传输字节流

```java
public class test {
    public static void main(String[] args) throws IOException {
        PipedInputStream inputStream=new PipedInputStream();
        PipedOutputStream outputStream=new PipedOutputStream();
        //在输入管道流与输出管道流之间建立连接
        inputStream.connect(outputStream);

        //创建新线程向管道流中写入数据
        new Thread(new Runnable() {
            @Override
            public void run() {
                writeData(outputStream);
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                readData(inputStream);
            }
        }).start();
    }
    public static void writeData(PipedOutputStream out){
            try {
                for(int i=0;i<100;i++){
                    String data=""+i;
                    out.write(data.getBytes());//把字节数组写入到输出管道流中
                }
                out.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
    }
    public static void readData(PipedInputStream in){
        byte[] bytes=new byte[1024];
        //从管道输入字节流中读取字节保存到字节数组中
        try {
            int len=in.read(bytes);
            while (len!=-1){
                //把bytes数组中从0开始读到的len个字节转换为字符串打印
                System.out.println(new String(bytes,0,len));
                len=in.read(bytes);//继续向下读
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### ThreadLocal的使用

除了控制资源的访问外,还可以通过增加资源来保证线程安全.

ThradLocal主要解决为每个线程绑定自己的值

给每一个线程都给一个`SimpleDateFormat`对象

```java
public class test1 {
    static ThreadLocal<SimpleDateFormat> threadLocal=new ThreadLocal<>();
    private static SimpleDateFormat sdf=new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
    //定义Runnable接口实现类
    static class ParseDate implements Runnable{
        private int i=0;
        public ParseDate(int i){
            this.i=i;
        }
        @Override
        public void run() {
            try {
                String text="2099年11月23日 08:39:"+i%60;
                //设定threadLocal中的对象
                if(threadLocal.get()==null){
                    threadLocal.set(new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss"));
                }
                Date date=threadLocal.get().parse(text);
                System.out.println(i+"--"+date);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        for(int i=0;i<100;i++){
            new Thread(new ParseDate(i)).start();
        }
    }
}
```

也可以通过重写`ThreadLocal#initialValue()`方法来设置初始值,这样get()得到的就不是null了

## 5.Lock显示锁

在JDK5中新增了LOCK锁接口,有ReentrantLock实现类,ReentranLock锁称为可重入锁,它的功能比synchronized多

#### 锁的可重入性

锁的可重入是指,当一个线程获得一个对象锁后,再次请求该对象锁时是可以获得该对象的锁的

一个简单的例子

![image-20220818221028397](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220818221028397.png)

#### ReentrantLock

##### ReentrantLock的基本使用

```java
public class test2 {
    //定义显示锁
    static Lock lock=new ReentrantLock();

    public static void sm1(){
        try{
            lock.lock();//获得锁
            System.out.println(Thread.currentThread().getName()+"---method 1---"+System.currentTimeMillis());
            Thread.sleep(new Random().nextInt(1000));
            System.out.println(Thread.currentThread().getName()+"---method 1---"+System.currentTimeMillis());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();//释放锁
        }
    }
    public static void sm2(){
        try{
            lock.lock();//获得锁
            System.out.println(Thread.currentThread().getName()+"---method 2---"+System.currentTimeMillis());
            Thread.sleep(new Random().nextInt(1000));
            System.out.println(Thread.currentThread().getName()+"---method 2---"+System.currentTimeMillis());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();//释放锁
        }
    }
    public static void main(String[] args) {
        Runnable r1=new Runnable() {
            @Override
            public void run() {
                sm1();
            }
        };
        Runnable r2=new Runnable() {
            @Override
            public void run() {
                sm2();
            }
        };
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r1).start();
        new Thread(r2).start();
        new Thread(r2).start();
        new Thread(r2).start();
    }
}
```

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220818222217922.png" alt="image-20220818222217922" style="zoom:67%;" />

只要锁对象是同一个,就可以实现同步

##### ReentrantLock的可重入性

```java
public class test3 extends Thread {
   static class SubThread extends Thread{
      //这里定义为static的Lock对象的原因就是为了保证每个实例对象拿到的都是同一把锁
      private static Lock lock=new ReentrantLock();
      public static int num=0;
      @Override
      public void run() {
         for(int i=0;i<10000;i++){
            try{
               lock.lock();
               lock.lock();
               num++;
            }finally {
               lock.unlock();
               lock.unlock();
            }
         }
      }
   }
   public static void main(String[] args) throws InterruptedException {
      SubThread t1=new SubThread();
      SubThread t2=new SubThread();
      t1.start();
      t2.start();
      //使用join让main线程等待其执行
      t1.join();
      t2.join();
      System.out.println(SubThread.num);
   }
}
```

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220818224401783.png" alt="image-20220818224401783" style="zoom:67%;" />

##### lockInterruptibly()

lockInterruptibly()方法的作用:如果当前线程未被中断则获得锁,如果当前线程被中断则抛出异常

**lock.lockInterruptibly()想获取某个锁时，假若此时线程A获取到了锁，而线程B只有等待，那么对线程B调用threadB.interrupt()方法能够中断线程B的等待过程。**

==注意是:等待的那个线程B可以被中断，不是正在执行的A线程被中断==

```java
public class test4 {
    static class Server{
        private Lock lock=new ReentrantLock();//定义锁对象
        public void serviceMethod(){
            try {
                //lock.lock();//获得锁,即使调用了线程的interrupt()方法,该线程也没有真正的中断
                lock.lockInterruptibly();
                System.out.println(Thread.currentThread().getName()+"--- begin lock");
                //执行一段耗时的操作
                for (int i=0;i<Integer.MAX_VALUE;i++){

                    new StringBuilder();
                }
                System.out.println(Thread.currentThread().getName()+"--- end lock");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();//释放锁
                System.out.println(Thread.currentThread().getName()+"*** 释放锁");
            }
        }
        public static void main(String[] args) throws InterruptedException {
            Server s =new Server();
            Runnable r=new Runnable() {
                @Override
                public void run() {
                    s.serviceMethod();
                }
            };
            Thread t1=new Thread(r);
            t1.start();
            Thread.sleep(200);
            Thread t2=new Thread(r);
            t2.start();
            Thread.sleep(200);
            t2.interrupt();
        }
    }
}
```

<img src="https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220819011214242.png" alt="image-20220819011214242" style="zoom:67%;" />

##### lockInterruptibly()可以解决死锁问题

对于synchronized内部锁来说,如果一个线程在等待锁,只有两个结果:要么该线程获得锁继续执行,要么就保持等待

==对于ReentrantedLock来说,可以在等待锁的过程中取消对锁的请求==

##### trylock()方法

###### trylock(long time,TimeUnit unit)

trylock(long time,TimeUnit unit)的作用在给定等待时长内锁没有被另外的线程持有,并且当前线程也没有被中断,则获得该锁,通过该方法可以实现锁对象的限时等待

```java
public class test5 {
    static class TimeLock implements Runnable{
        private static ReentrantLock lock=new ReentrantLock();
        @Override
        public void run() {
            try {
                if(lock.tryLock(3, TimeUnit.SECONDS)){
                    System.out.println(Thread.currentThread().getName()+"获得锁,执行耗时任务");
                    Thread.sleep(4000);

                }else{
                    System.out.println(Thread.currentThread().getName()+"没有获得锁");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }
    public static void main(String[] args) {
        TimeLock timeLock=new TimeLock();
        Thread t1=new Thread(timeLock);
        Thread t2=new Thread(timeLock);
        t1.start();
        t2.start();
    }
}
```

![image-20220819221602805](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220819221602805.png)

上述的执行结果中,Thread0先获得锁执行耗时4秒的任务,Thread1尝试获得锁,但是超过3秒之后Thread1就不在等待,直接放弃获得锁

###### trylock()

trylock()仅在调用时锁定未被其他线程持有的锁,如果调用方法时,锁对象被其他线程持有,则放弃

###### trylock()可以避免死锁

```java
public class test6 {

    static class IntLock implements Runnable {
        private static ReentrantLock lock1 = new ReentrantLock();
        private static ReentrantLock lock2 = new ReentrantLock();
        private int lockNum;

        public IntLock(int lockNum) {
            this.lockNum = lockNum;
        }

        @Override
        public void run() {
            if (lockNum % 2 == 0) {

                while (true) {
                    try {
                        if (lock1.tryLock()) {
                            System.out.println(Thread.currentThread().getName() + "获得锁1,还想获得锁2");
                            Thread.sleep(new Random().nextInt(100));
                            try {
                                if (lock2.tryLock()) {
                                    System.out.println(Thread.currentThread().getName() + "同时获得锁1与锁2");
                                    return;//结束run方法
                                }
                            } catch (Exception e) {
                                e.printStackTrace();
                            } finally {
                                if (lock2.isHeldByCurrentThread()) {
                                    lock2.unlock();
                                }
                            }
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        if (lock1.isHeldByCurrentThread()) {
                            lock1.unlock();
                        }
                    }
                }
            } else {
                while (true) {
                    try {
                        if (lock2.tryLock()) {
                            System.out.println(Thread.currentThread().getName() + "获得锁2,还想获得锁1");
                            Thread.sleep(new Random().nextInt(100));
                            try {
                                if (lock1.tryLock()) {
                                    System.out.println(Thread.currentThread().getName() + "同时获得锁2与锁1");
                                    return;//结束run方法
                                }
                            } catch (Exception e) {
                                e.printStackTrace();
                            } finally {
                                if (lock1.isHeldByCurrentThread()) {
                                    lock1.unlock();
                                }
                            }
                        }
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        if (lock2.isHeldByCurrentThread()) {
                            lock2.unlock();
                        }
                    }
                }

            }
        }
    }
    public static void main(String[] args) {
        IntLock intLock1 = new IntLock(11);
        IntLock intLock2 = new IntLock(10);
        Thread t1 = new Thread(intLock1);
        Thread t2 = new Thread(intLock2);
        t1.start();
        t2.start();
        //运行后,使用trylock()尝试获得锁,不会傻傻的等待,
        //通过循环不停的再次尝试,如果等待的时间足够长,线程总是或获得想要的资源
    }
}
```

![image-20220820000504397](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220820000504397.png)

实际上等了蛮长时间的,但是使用trylock()确实可以避免死锁

##### newCondition()方法

```java
Condition condition=lock.newCondition();
```



关键字synchronized与wait()/notify()这两个方法一起使用可以实现等待/通知模式,Lock锁的newCondition()方法返回Condition对象,Condition类也可以实现等待/通知模式

使用notify()通知时,JVM会随机唤醒某个等待的线程,使用Condition类可以进行选择性通知

await()会使当前线程等待,同时会释放锁,当其他线程调用signal()时,线程会重新获得锁并继续执行

signal()用于唤醒一个等待的线程

==不管是await()还是signal()都需要先持有锁==

注意:在调用Condition的await()/signal()方法前,也需要线程持有相关的Lock锁,调用await()后线程会释放这个锁,在singal()调用后会从当前Condition对象的等待队列中,唤醒一个线程,唤醒的线程会尝试获得锁,一旦获得锁成功就继续执行

###### 使用多个Condition对象实现通知部分线程

可以让多个condition对象管理多个等待

###### 使用Condition实现多线程交替打印

```java
public class test8 {
    static class MyService{
        private Lock lock=new ReentrantLock();
        private Condition condition=lock.newCondition();
        private boolean flag=true;
        private void printOne(){
            try {
                lock.lock();
                while (flag){
                    System.out.println(Thread.currentThread().getName()+"waiting...");
                    condition.await();
                }
                System.out.println(Thread.currentThread().getName()+"--------------");
                flag=true;
                condition.signalAll();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
        private void printTwo(){
            try {
                lock.lock();
                while (!flag){
                    System.out.println(Thread.currentThread().getName()+"waiting...");
                    condition.await();
                }
                System.out.println(Thread.currentThread().getName()+"===============");
                flag=false;
                condition.signalAll();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }
    public static void main(String[] args) {
        MyService myService=new MyService();
        for (int i = 0; i < 100; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i=0;i<100;i++){
                        myService.printOne();
                    }
                }
            }).start();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i=0;i<100;i++){
                        myService.printTwo();
                    }
                }
            }).start();
        }
    }
}
```

##### 公平锁与非公平锁

大多数情况下,锁的很强都是非公平的,如果线程1与线程2都在请求锁A,当锁A可用时,系统只是会从阻塞队列中随机的选择一个线程,不能保证其公平性

公平锁会按照时间先后顺序,保证先到先得,后到后得,公平锁的这一特点不会出现线程饥饿现象

synchronized是非公平锁,

ReentrantLock重入锁提供了一个构造方法:ReentrantLock(boolean fair),当在创建锁对象时实参传递true就可以把该锁设置为公平锁

如果是非公平锁,系统倾向于让一个线程再次获得已经持有的锁,这种分配策略是高效的,非公平的

如果是公平锁,多个线程不会发生同一个线程连续多次获得锁的可能,保证了公平性

```java
public class test9 {

    static Lock lock = new ReentrantLock(true);

    public static void main(String[] args) {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                while (true) {
                    try {
                        lock.lock();

                        System.out.println(Thread.currentThread().getName()+"获得了锁对象");
                    } finally {
                        lock.unlock();
                    }
                }
            }
        };
        for (int i = 0; i < 4; i++) {
            new Thread(runnable).start();
        }
    }
}
```

在这个例子中个让线程获得锁对象的顺序一开始是什么样,之后就是不断的轮回

![image-20220821012121602](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220821012121602.png)

>公平锁看起来很公平,但是要实现公平锁必须要求系统维护一个有序队列,公平锁的实现成本较高,性能也较低,因此默认情况下锁是非公平的,不是特别的需求,一般不适用公平锁

##### ReentrantLock几个常用的方法

###### int getHoldCount()

方法可以返回当前线程调用lock方法的次数

###### int getQueueLength()

返回正等待获得锁的线程**预估数**

###### int getWaitQueueLength(Condition condition)

返回与Condition相关的等待的线程预估数

###### hasQueuedThread(Thread thread)

查询参数执行的线程是否在等待获得锁

###### hasQueuedThreads()

查询是否还有线程在等待获得该锁

###### boolean hasWaiters(Condition condition)

查询是否有线程正在等待指定的condition条件

###### boolean isFair()

判断是否为公平锁

###### boolean isHeldByCurrentThread()

判断当前线程是否持有该锁

###### boolean islocked()

查询当前锁是否被线程持有,释放了锁对象后返回false

#### ReentrantReadWriteLock读写锁

synchronized内部锁与ReentrantLock锁都是独占锁(排它锁),同一时间只允许一个线程执行同步代码块,可以保证线程的安全性,但是执行效率低

ReentrantReadWriteLock读写锁是一种改进的排他锁,也可以称作共享/排他锁,允许多个线程同时读取共享数据,但是一次只允许一个线程对共享数据进行更新

读写锁通过读锁与写锁来完成读写操作,线程在读取共享数据前必须先持有读锁,该读锁可以被多个线程持有,即它是共享的,线程在修改共享数据前必须先持有写锁,写锁是排他的

读锁只是在读线程之间共享,任何一个线程持有读锁时,其他线程都无法获得写锁,保证了线程在读取数据期间没有其他线程对数据进行更新,使得读线程能够读到数据的最新值,保证在读数据期间共享变量不被修改

| 锁   | 获得条件                                                | 排他性                            | 作用                                                         |
| ---- | ------------------------------------------------------- | --------------------------------- | ------------------------------------------------------------ |
| 读锁 | 写锁未被任意线程持有                                    | 对读线程是共享的,对写线程是排他的 | 允许多个读线程可以同时读取共享数据,保证在读共享数据时,没有其他线程对共享数据进行修改 |
| 写锁 | 该写锁未被其他线程持有,并且响应的读锁也未被其他线程持有 | 对读线程和写线程都是排他的        | 保证写线程以独占的方式修改共享数据                           |

在java.util.concurrent.locks包中定义了ReadWriteLock接口,该接口中定义了readLock()返回读锁,定义writeLock()方法返回写锁,该接口的实现类是ReentrantReadWriteLock

**注意readLock()与writeLock()方法返回的锁对象是同一个锁的两个不同的角色,不是分别获得两个不同的锁**,ReadWriteLock接口实例可以充当两个角色

```java
//定义读写锁
ReadWriteLock rwLock=new ReentrantReadWriteLock();
//获得读锁
Lock readLock=rwLock.readLock();
//获得写锁
Lock writeLock=rwLock.writeLock();
```

##### 读读共享

ReadWriteLock读写锁可以实现多个线程同时读取数据,即读读共享,可以提高程序的读取数据的效率

小例子

```java
public class test1 {
    static class Service{
        //定义读写锁
        ReadWriteLock readWriteLock=new ReentrantReadWriteLock();
        //定义方法读取数据
        public void read(){
            try {
                readWriteLock.readLock().lock();
                System.out.println(Thread.currentThread().getName()+"获得读锁"+System.currentTimeMillis());

                TimeUnit.SECONDS.sleep(3);//模拟读取数据耗时
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                readWriteLock.readLock().unlock();
            }

        }
    }
    public static void main(String[] args) {
        Service service=new Service();
        for (int i=0;i<5;i++){
            new Thread(() -> service.read()).start();
        }
    }
}
```

![image-20220821143804836](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220821143804836.png)

##### 写写互斥

小例子

```java
public class test2 {
    static class Service{
        //定义读写锁
        ReadWriteLock readWriteLock=new ReentrantReadWriteLock();
        //定义方法模拟写数据
        public void write(){
            try {
                readWriteLock.writeLock().lock();
                System.out.println(Thread.currentThread().getName()+"获得写锁"+System.currentTimeMillis());
                TimeUnit.SECONDS.sleep(3);//模拟读取数据耗时
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                readWriteLock.writeLock().unlock();
            }
        }
    }
    public static void main(String[] args) {
        Service service=new Service();
        for (int i=0;i<6;i++){
            new Thread(() -> service.write()).start();
        }
    }
}
```

![image-20220821144443967](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220821144443967.png)

等待上一个线程释放写锁之后下一个线程才能得到写锁

##### 读写互斥

```java
public class test2 {
    static class Service{
        //定义读写锁
        ReadWriteLock readWriteLock=new ReentrantReadWriteLock();
        //定义方法读取数据
        public void read(){
            try {
                readWriteLock.readLock().lock();
                System.out.println(Thread.currentThread().getName()+"获得读锁"+System.currentTimeMillis());

                TimeUnit.SECONDS.sleep(3);//模拟读取数据耗时
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                readWriteLock.readLock().unlock();
            }

        }
        public void write(){
            try {
                readWriteLock.writeLock().lock();
                System.out.println(Thread.currentThread().getName()+"获得写锁"+System.currentTimeMillis());
                TimeUnit.SECONDS.sleep(3);//模拟读取数据耗时
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                readWriteLock.writeLock().unlock();
            }
        }
    }
    public static void main(String[] args) {
        Service service=new Service();
            new Thread(() -> service.write()).start();
            new Thread(() -> service.read()).start();
    }
}
```



![image-20220821144933171](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220821144933171.png)

等待上一个写线程执行完毕释放写锁之后才能获得读锁

## 6.线程管理

#### 线程组

在线程组中定义一组相关(相似)的线程,在线程组中也可以定义子线程组

Thread类有几个构造方法允许在创建线程时执行线程组,如果在创建线城时没有执行线程组,那么该线程就属于父线程所在的线程组

JVM在创建main线程时会为它指定一个线程组,因此每个Java线程组都有一个线程组与它关联,可以调用线程的getThreadGroup()方法返回线程组

线程组开始是出于安全的考虑设计用来区分不同的Applet,然而ThreadGroup并未实现这一目标,在新开发的系统中,已经不常用线程组了

##### 线程组创建

```java
public class test {
    public static void main(String[] args) {
        //返回当前main线程的线程组
        ThreadGroup mainGroup=Thread.currentThread().getThreadGroup();
        System.out.println(mainGroup);

        //定义线程组,如果不指定所属线程组,则自动归属到当前线程所属的线程组中
        ThreadGroup group1=new ThreadGroup("group1");
        System.out.println(group1);
        System.out.println(group1.getParent()==mainGroup);

        //定义线程组,同时指定父线程组
        ThreadGroup group2=new ThreadGroup(mainGroup,"group2");
        System.out.println(group2.getParent()==mainGroup);
        
        //创建线程时不指定线程组
        Runnable r=new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread());
            }
        };
        Thread t1=new Thread(r,"t1");
        System.out.println(t1);

        //创建线程时指定线程组
        Thread t2=new Thread(group1,r,"t2");
        System.out.println(t2);
    }
}
```

![image-20220821152550460](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220821152550460.png)

##### 线程组的基本操作

activeCount()返回当前线程组及子线程组中活动该线程的数量(近似)

activeGroupCount()返回当前线程组及子线程组中活动线程组的数量(近似值)

int enumerate(Thread[] list)将当前线程组中的活动线程复制到参数数组中

getMaxPriority()返回线程组的最大优先级,默认为10

getName()返回线程组名称

getParent()返回父线程组

interrupt()中断线程组中所有的线程

setDaemon(boolean daemon)设置线程组为守护线程组

isDaemon()判断当前线程组是否为守护线程组

list()将当前线程组中的活动线程打印出来

parentOf(ThreadGroup g)判断当前线程组是否为参数线程组的父线程组

##### 复制线程组中的线程及子线程组

enumerate(Thread[] list)把当前线程组合子线程组中所有的线程复制到参数数组中

enumerate(Thread[] list，boolean recurse)第二个参数设置为false,则只复制当前线程组的子线程组

##### 线程组的批量中断

线程组的interrupt()方法可以给该线程组中所有的活动线程添加中断标志

##### 设置守护线程组

守护线程是为其他线程提供服务的,当JVM中只有守护线程时,守护线程会自动销毁,JVM会退出

调用线程组的setDaemon(true)可以把线程组设置为守护线程组,**当守护线程组中没有任何活动线程时,守护线程组会自动销毁**

注意线程组的守护属性,不影响线程组中线程的守护属性,或者说守护线程组中的线程可以是非守护线程

```java
public class test2 {
    public static void main(String[] args) throws InterruptedException {
        //先定义线程组
        ThreadGroup group=new ThreadGroup("group");
        //设置线程组为守护线程组
        group.setDaemon(true);
        //向组中添加3个线程
        for (int i = 0; i < 3; i++) {
            new Thread(group, new Runnable() {
                @Override
                public void run() {
                    for (int j = 0; j < 20; j++) {
                        System.out.println(Thread.currentThread().getName()+"---"+j);

                        try {
                            Thread.sleep(500);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }).start();
        }
        Thread.sleep(5000);
        System.out.println("main线程结束");
    }
}
```

例子中,main线程5秒结束,守护线程组中的非守护线程需要10秒结束,在main线程结束后,他们还是继续运行,直到三个线程都结束,守护线程组才自动销毁

![image-20220821160517628](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220821160517628.png)

#### 捕获线程的执行异常

在线程的run()方法中,如果有受检异常必须进行捕获处理,如果想要获得run()方法中出现的运行时异常信息,可以通回调UncaughtExceptionHandler接口获得哪个线程出现了运行时异常,在Thread类中有关处理运行异常的方法有:

`getDefultUncaughtExceptionHandler()`获得全局的(默认的)UncaughtExceptionHandler

`getUncaughtExceptionHandler()`获得当前线程的UncaughtExceptionHandler

`setDefultUncaughtExceptionHandler()`设置全局的UncaughtExceptionHandler

`setUncaughtExceptionHandler()`设置当前线程的UncaughtExceptionHandler

##### 设置全局线程异常的回调接口

```java
public class test1 {
    public static void main(String[] args) {
        //设置线程异常全局的回调接口
        Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler(){

            @Override
            public void uncaughtException(Thread t, Throwable e) {
                System.out.println(t.getName()+"产生了异常:"+e.getMessage());
            }
        });
        Thread t1=new Thread(() -> System.out.println(10/0));
        t1.start();
    }
}
```

![image-20220822151634663](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220822151634663.png)

如果设置了自己线程的UncaughtExceptionHandler就就调用自己的UncaughtExceptionHandler

没有则调用看全局UncaughtExceptionHandler

全局也没有就直接把异常栈信息定向到System.err中

##### 注入Hook钩子线程

现在很多软件包括MySql,Zookeeper,kafka等都存在Hook线程的校验机制,目的是校验进程是否已启动,防止重复启动程序

Hook线程也称为钩子线程,当JVM退出的时候会执行Hook线程,经常在程序启动的时候创建一个.lock文件,用.lock文件校验程序是否启动,在程序退出(JVM退出)时删除该.lock文件,在Hook线程中除了防止重新启动进程外,还可以做资源释放

```java
public class test1 {
    public static void main(String[] args) {
        //1.注入Hook线程,在程序退出时删除.lock文件
        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("JVM退出,会启动柜当前hook线程,在hook线程中删除.lock文件");
                getLockFile().toFile().delete();
            }
        }));
        //2.程序运行时,检查.lock文件是否存在,如果存在,则抛出异常
        if(getLockFile().toFile().exists()){
            throw new RuntimeException("程序已启动");
        }else{
            try {
                getLockFile().toFile().createNewFile();
                System.out.println("程序启动并创建了.lock文件");
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        //模拟程序运行
        for (int i = 0; i < 10; i++) {
            try {
                System.out.println("程序运行中----");
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public static Path getLockFile(){
        return Paths.get("","tmp.lock");
    }
}
```



![image-20220822154013314](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220822154013314.png)

如果.lock文件没有删除运行则报错

![image-20220822154050264](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220822154050264.png)

#### 线程池

##### 什么是线程池

线程池内部可以预先创建一定数量的工作线程,客户端代码直接将任务作为一个对象提交给线程池,线程池将这些任务缓存在工作队列中,线程池中的工作线程不断地从队列中取出任务

##### JDK对线程池的支持

JDK提供了一套Executor框架,可以帮助开发人员有效的使用线程池

![image-20220822160953591](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220822160953591.png)

##### 线程池的基本使用

```java
public class test1 {
    public static void main(String[] args) {
        //创建有5个线程大小的线程池
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        //向线程池中提交18个任务
        for (int i = 0; i < 18; i++) {
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getId()+"号任务正在执行任务,开始时间:"+System.currentTimeMillis());

                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
    }
}
```

以5个为一轮,固定的5个线程执行任务

![image-20220822161546596](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220822161546596.png)

##### 线程池的关闭

两个方法

###### shutdown()

​	该方法执行后,线程池状态变为SHUTDOWN,不会接收新任务,但是会执行完已提交的任务,此方法不会阻塞调用线程的执行

shutdownNow()

​	该方法执行后,线程池状态变为STOP,不会接收新任务,会将队列中的任务返回,并用interrupt的方式中断正在执行的

##### Execurors提供的线程池和特点

1. `newSingleThreadExecutor()` 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
2. `newFixedThreadPool() `创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
3. `newScheduledThreadPool() `创建一个可定期或者延时执行任务的定长线程池，支持定时及周期性任务执行。 
4. `newCachedThreadPool()` 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。 

##### 核心线程池的底层实现

查看Executors工具类中`newFixedThreadPool`,`newFixedThreadPool`,`newSingleThreadExecutor`等方法,都调用了`ThreadPoolExecutor`这个构造方法

![image-20220822163534233](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220822163534233.png)

各个参数含义

| 参数            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| corePoolSize    | 指定线程池中核心线程的数量                                                                                       CPU 密集型： CPU 核数  + 1                                                               IO 密集型： 2 倍 CPU 核数 + 1 |
| maximumPoolSize | 指定线程池中最大线程数量                                     |
| keepAliveTime   | 当线程池中线程的数量超过corePoolSize时,多余的空闲线程的存活时长,即空闲线程在多长时长内销毁 |
| unit            | 是keepAliveTime时长单位                                      |
| wordQueue       | 任务队列,把任务提交到该任务队列中等待执行                    |
| handler         | 拒绝策略,当任务太多来不及处理时,如何拒绝                     |

###### wordQueue

workQueue工作队列是指提交未执行的任务队列,它是BlockingQueue接口的对象,仅用于存储Runnable任务,根据队列功能分类,在ThreadPoolExecutor构造方法中可以使用以下几种阻塞队列

1.直接提交队列

由`SynchronousQueue`对象提供,该队列没有容量,提交给线程池的任务不会被真实的保存,总是将新的任务提交给线程执行,如果没有空闲线程,则尝试创建新的线程,如果线程数量已经达到maximumPoolSize规定的最大值则执行拒绝策略

2.有界任务队列

由`ArrayBlockingQueue`实现,在创建`ArrayBlockingQueue`对象时,可以指定一个容量,当有任务需要执行时,如果线程池中线程数小于`corePoolSize`核心线程数则创建新的线程;如果大于`corePoolSize`核心线程数则加入等待队列;

如果队列已满则无法加入,在线程数小于`maximumPoolSize`指定的最大线程数前提下会创建新的线程来执行,如果线程数大于`maximumPoolSize`最大线程数则执行拒绝策略

![image-20220822172950757](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220822172950757.png)

3.无界任务队列

由`LinkedBlockingQueue`对象实现,与有界队列相比,除非资源耗尽,否则无界队列不存在任务入队失败的情况,当有新的任务时,在系统线程数小于`corePoolSize`核心线程数则创建新的线程来执行任务;当线程池中线程数量大于`corePoolSize`核心线程数则把任务加入阻塞队列

4.优先任务队列

通过`PriorityBlockingQueue`实现,是带有任务优先级的队列,是一个特殊的无界队列,不管是`ArrayBlockingQueue`队列还是`LinkedBlockingQueue`队列都是按照先进先出算法处理任务的,在`PriorityBlockingQueue`中可以根据任务优先级顺序先后执行

##### 拒绝策略

当线程池中的线程用完了,等待队列也满了,无法为新提交的任务服务,可以通过拒绝策略来处理这个问题,jdk提供了四种拒绝策略

###### AbortPolicy

抛出异常

###### CallerRunsPolicy

只要线程池没有关闭,它会在调用者线程中运行当前被丢弃的任务

###### DiscardOldestPolicy

将队列中最老的任务(即将要执行的任务)丢弃,尝试再次提交新任务

###### DiscardPolicy

直接丢弃这个无法处理的任务

##### ThreadFactory

ThreadFactory是一个接口,只有一个用来创建线程的方法:

Thread newThread(Runnable r);

当线程池中需要创建线程时就会调用该方法

##### 监控线程池

ThreadPoolExecutor提供了一组方法用于监控线程池

int getActiveCount()获得线程池中当前活动线程的数量

long getCompletedTaskCount()返回线程池完成任务的数量

int getCorePoolSize() 线程池中核心线程的数量

int getLargestPoolSize() 返回线程池曾经达到的线程的最大数

int getMaximumPoolSize()返回线程池的最大容量

int getPoolSize()返回当前线程池的大小

getQueue() 返回阻塞队列

long getTaskCount() 返回线程池收到的任务总数

```java
public class test3 {
    public static void main(String[] args) throws InterruptedException {
        //先定义任务
        Runnable r=new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getId()+"编号的线程开始执行: "+System.currentTimeMillis());
                try {
                    Thread.sleep(10000);//线程睡眠10秒,模拟任务执行时长
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };
        //定义线程池
        ThreadPoolExecutor poolExecutor=new ThreadPoolExecutor(2,5,0, TimeUnit.SECONDS,new ArrayBlockingQueue<>(5), Executors.defaultThreadFactory(),new ThreadPoolExecutor.DiscardPolicy());

        //向线程池提交30个任务

        for (int i = 0; i < 30; i++) {
            poolExecutor.submit(r);
            System.out.println("当前线程池核心线程数量"+poolExecutor.getCorePoolSize()+",最大线程数"+poolExecutor.getMaximumPoolSize()+",当前线程池大小"+poolExecutor.getPoolSize()+",收到任务数量:"+poolExecutor.getTaskCount()+",完成任务数:"+poolExecutor.getCompletedTaskCount()+",等待任务数"+poolExecutor.getQueue().size());
            TimeUnit.MILLISECONDS.sleep(500);
        }
        System.out.println("-----------------提交完成-------------------");
        TimeUnit.SECONDS.sleep(5);
        while (poolExecutor.getActiveCount()>0){
            System.out.println("当前线程池核心线程数量"+poolExecutor.getCorePoolSize()+",最大线程数"+poolExecutor.getMaximumPoolSize()+",当前线程池大小"+poolExecutor.getPoolSize()+",收到任务数量:"+poolExecutor.getTaskCount()+",完成任务数:"+poolExecutor.getCompletedTaskCount()+",等待任务数"+poolExecutor.getQueue().size());
        }
    }
}
```

![image-20220822191225957](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220822191225957.png)

在提交完成后,只收到了15个任务,因为创建的线程池最大线程数为5,最大容量为5,同时等待队列最大容量为5,每半秒提交一个任务,所以30个总共耗时15秒,理论上来说15秒内后面15个线程都没地方可去,执行拒绝策略`DiscardPolicy`都将其抛弃

##### 扩展线程池

有时需要对线程池进行扩展,如在监控每个任务的开始和结束时间,或者自定义一些其他增强的功能:

ThreadPoolExecutor线程池提供了两个方法:

afterExecute(),beforeExecute()

在线程池执行某个任务前会调用`beforeExecute()`,在任务结束后(任务异常退出)会调用`afterExecute()`

查看`ThreadPoolExecutor`源码,在该类中定义了一个内部类Worker,ThreadPoolExecutor线程池中的工作线程就是Worker类的实例,Worker实例在执行时会调用`beforeExecute()`与`afterExecute()`方法

这种重写类似于Spring中的Filter

```java
public class test4 {
    static class MyTask implements Runnable{
        private String name;
        public MyTask(String name) {
            this.name = name;
        }
        @Override
        public void run() {
            System.out.println(name+"任务正在被线程"+Thread.currentThread().getId()+" 执行");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args) {
        //定义扩展线程池,可以定义线程池类继承ThreadPoolExecutor方法,在子类中重写beforeExecute()/afterExecute()方法
        //也可以直接使用ThreadPoolExecutor的内部类
        ExecutorService executorService=new ThreadPoolExecutor(5,5,0, TimeUnit.SECONDS,new LinkedBlockingDeque<>()){
            @Override
            protected void beforeExecute(Thread t, Runnable r) {
                System.out.println(t.getId()+"线程准备执行任务"+((MyTask)r).name);
            }
            @Override
            protected void afterExecute(Runnable r, Throwable t) {
                System.out.println(((MyTask)r).name+"任务执行完毕");
            }
            @Override
            protected void terminated() {
                System.out.println("线程池退出");
            }
        };
        //创建5个任务添加到线程池中执行
        for (int i = 0; i < 5; i++) {
            executorService.execute(new MyTask("task--"+i));
        }
        //关闭线程池
        executorService.shutdown();//关闭线程池仅仅是说线程池不再接收新的任务,线程池中已接收的任务正常执行完毕
    }
}
```

![image-20220822232958662](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220822232958662.png)

##### 优化线程池数量

线程池大小对系统性能是有一定影响的,过大或过小都会无法发挥最优的系统性能,线程池大小不需要非常精确,只要避免极大或者极小的情况即可

**线程池大小=CPU核心数量×目标CPU的使用率×(1+平均等待时间/平均工作时间)**

##### 线程池死锁

如果在线程池中执行的任务A在执行过程中又向线程池提交了任务B,任务B添加到了线程池的等待队列中,如果任务A的结束需要等待任务B的执行结果,就有可能出现这种情况:**线程池中所有的工作线程都处于等待任务处理结果,而这些任务在阻塞队列中等待执行**,线程池中没有可以对阻塞队列中的任务进行处理的线程,这种等待会一直持续下去,从而造成死锁

   适合给线程池提交相互独立的任务,而不是批次依赖的任务,对于彼此依赖的任务,可以考虑分别提交给不同的线程池来执行

##### 线程池中的异常跟踪

直接使用原生的submit()方法往线程池添加任务,会将异常吞掉

1. 使用execute()添加任务
2. 自定义ThreadPoolExecutor类

```java
public class test5 {
    static class TraceThreadPoolExecutor extends ThreadPoolExecutor{
        public TraceThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
            super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
        }
        //定义方法,对执行的任务进行包装,接收两个参数,一个参数接收要执行的任务,另一个参数是Exception异常
        public Runnable wrap(Runnable task,Exception exception){
                return new Runnable() {
                    @Override
                    public void run() {
                        try {
                            task.run();
                        } catch (Exception e) {
                            exception.printStackTrace();
                            throw e;
                        }
                    }
                };
        }

        @Override
        public Future<?> submit(Runnable task) {
            return super.submit(wrap(task,new Exception("自定义跟踪异常")));
        }
        public static void main(String[] args) {
            TraceThreadPoolExecutor traceThreadPoolExecutor = new TraceThreadPoolExecutor(2,5,0, TimeUnit.SECONDS,new SynchronousQueue<>());
            traceThreadPoolExecutor.submit(new Runnable() {
                @Override
                public void run() {
                    System.out.println(0/0);
                }
            });
        }
    }
}
```

![image-20220823000927977](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220823000927977.png)

##### ForkJoinPool线程池

该线程池采用的是"分治"思想

系统对ForkJoinPool线程池进行了优化,提交的任务数量与线程的数量不一定是一对一关系,在多数情况下,一个物理线程实际上需要处理多个逻辑任务

![image-20220823105618999](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220823105618999.png)

###### 小例子

求两个正整数之间所有数的合

```java
public class test6 {
    static class CountTask extends RecursiveTask<Long>{
        public CountTask(long start, long end) {
            this.start = start;
            this.end = end;
        }
        private static final int THRESHOLD=10000;//阈值为10000
        private static final int TASKNUM=100;
        private long start;//计算数列的起始值
        private long end;//计算数列的结束值
        //重写RecursiveTask类的compute()方法
        @Override
        protected Long compute() {
            long sum=0;//保存计算的结果
            //判断任务是否需要继续分解,如果当前数列end与start范围的数超过阈值THRESHOLD,就需要继续分解
            if(end-start<THRESHOLD){
                //小于阈值就直接计算
                for (long i=start;i<=end;i++){
                    sum+=i;
                }
            }else{//数列范围超过阈值,需要继续分解
                //约定每次分解成100个小任务,计算每个任务的计算量
                long step=(end-start)/TASKNUM;
                ArrayList<CountTask> subTaskList=new ArrayList<>();
                long pos=start;//每个任务的起始位置
                for (int i = 0; i < TASKNUM; i++) {
                    long lastOne=pos+step;//每个任务的结束位置
                    if(lastOne>end){//调整最后一个任务的结束位置
                        lastOne=end;
                    }
                    //创建子任务
                    CountTask task=new CountTask(pos,lastOne);
                    //把任务添加到集合中
                    subTaskList.add(task);
                    //使用fork()提交子任务
                    task.fork();
                    //调整下个任务的起始位置
                    pos+=step+1;
                }
                //等待所有的子任务结束后,合并计算结果
                for (CountTask task:subTaskList){
                    sum+=task.join();
                }
            }
            return sum;
        }
    }

    public static void main(String[] args) {
        //创建ForkJoinPool线程池
        ForkJoinPool forkJoinPool=new ForkJoinPool();
        //创建一个大任务
        CountTask task=new CountTask(0,2000000);
        //把大任务提交给线程池
        ForkJoinTask<Long> result=forkJoinPool.submit(task);
        try {
            Long res=result.get();//调用任务的get()方法返回结果
            System.out.println(res);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

**总的来说,这个线程池的用法有点类似递归的写法,和递归一样,什么时候继续向下调用方法,是问题的关键**

## 7.保障线程安全的设计技术

#### java运行时存储空间

Java运行时空间可以分为栈区,堆区与方法区(非堆空间)

​	栈空间为线程的执行准备一段固定大小的存储空间,每个线程都有独立的线程栈空间,创建线城时就为线程分配栈空间,在线程栈中每调用一个方法就给方法分配一个**栈帧**,栈帧用于存储方法的局部变量,返回值等私有数据,即**局部变量存储在栈空间中**,基本类型变量也是存储在栈空间中,引用类型变量值也是存储在栈空间中,引用的对象存储在堆中,由于线程栈是相互独立的,一个线程不能访问另外一个线程的栈空间,因此线程对局部变量以及只能通过当前线程的局部变量才能访问的对象进行的操作具有固定的线程安全性.

​	栈空间用于存储对象,是在JVM启动时分配的一段可以动态扩容的内存空间,创建对象时,在堆空间中给对象分配存储空间,实例变量就是存储在堆空间中的,对空间是多个线程之间可以共享的空间,因此实例变量可以被多个线程共享,多个线程同时操作实例变量可以存在线程安全问题

​	非堆空间用于存储常量,类的元数据等,非堆空间也是在JVM启动时分配的一段可以动态扩容的存储空间,类的元数据包括静态变量,类有哪些方法以及这些方法的元数据(方法名,参数,返回值等),非堆空间也是多个线程可以共享的,因此访问非堆空间中的静态变量也可能存在线程安全问题.

​	==不被其他线程共享的空间具有固有的局部安全性==

#### 无状态对象

**通俗来讲,无状态对象就是没有变量的对象**

​	对象就是数据及对数据操作的封装,对象所包含的数据称为对象的状态(state),实例变量与静态变量称为状态变量

​	如果一个类的同一个实例被多个线程共享并不会使这些线程存储共享的状态,那么该类的实例就称为无状态对象(Stateless Object),反之,如果一个类的实例被多个线程共享会使这些线程存在共享状态,那么该类的实例称为有状态对象,**实际上无状态对象就是不包含任何实例变量的对象,也不包含任何静态变量**

​	线程安全问题的前提是多个线程存在共享的数据,实现线程安全的一种办法就是避免在多个线程之间共享数据,使用无状态对象就是这种方法

#### 不可变对象

​	不可变对象是指一经创建它的状态(数据)就保持不变的对象,不可变对象也具有固有的线程安全性,当不可变对象现实实体的状态发生变化时,系统会创建一个新的不可变对象,就如**String字符串对象**

##### 	**一个不可变对象需要满足以下条件:**

1. 类本身使用final修饰,防止通过创建子类来改变它的定义
2. 所有字段都是final修饰的,final在创建对象时必须显示初始化,不能被修改
3. 如果字段引用了其他状态可变的对象(集合,数组),则这些字段必须是private私有的

频繁创建不可变对象可能会提升GC的负担

##### 	不可变对象的主要应用场景

- 被创建的对象的状态变化不频繁
- 同时对一组相关数据进行写操作,可以应用不可变对象,既可以保障原子性也可以避免锁的使用
- 使用不可变对象作为安全可靠的Map键,HashMap键值对的存储位置与键的hashCode有关,如果键的内部状态发生了变化会导致键的hashCode不同,可能会影响键值对的存储位置,如果HashMap的键是一个不可变对象,则hashCode()方法的返回值恒定,存储位置是固定的

#### 线程特有对象

​	我们可以选择不共享非线程安全的对象,对于非线程安全的对象,每个线程都创建一个该对象的实例,各个线程访问各自创建的实例,一个线程不能访问另外一个线程创建的实例,这种各个线程创建各自的实例,一个实例只能被线程访问的对象就称为线程特有对象.

​	线程特有对象既保障了对线程安全对象的访问的线程安全,又避免了锁的开销,**线程特有对象也具有固有的线程安全性**

`ThreadLocal<T>`类就相当于线程访问其特有对象的代理,即各个线程通过ThreadLocal对象可以创建并访问各自的线程特有对象,泛型T指定了线程特有对象的类型,一个线程可以使用不同的ThreadLocal实例来创建并访问不同的线程特有对象

#### 装饰器模式

​	装饰器模式可以用来实现线程安全,基本思想是为非线程安全的对象创建一个相应的线程安全的外包装对象,客户端代码不直接访问非线程安全的对象而是访问它的外包装对象.外包装对象与非线程安全的对象具有相同的接口,即外包装对象的使用方式与非线程安全的对象的使用方式相同,而外包装对象内部通常会借助锁,以线程安全的方式调用相应的非线程安全对象的方法

​	在java.util.Collections工具类中提供了一组synchronizedXXX(xxx)方法可以把不是线程安全的xxx集合转换为线程安全的集合,它就是采用了这种装饰器模式,这个方法返回值就是指定集合的外包装对象,这类集合又称为同步集合.

![image-20220823123326082](https://blue-satchel.oss-cn-chengdu.aliyuncs.com/img/image-20220823123326082.png)

​	使用装饰器模式的一个好处就是实现关注点分离,在这种设计中,实现同一组功能的对象的两个版本:非线程安全的对象与线程安全的对象.

​	对于非线程安全的在设计时只关注要实现的功能,对于线程安全版本的只关注线程安全性

## 8.锁的优化及注意事项

### 有助于提高锁性能的几点建议

#### 1.减少锁持有时间

​	对于使用锁进行并发控制的应用程序来说,如果单个线程持有锁的时间过长,会导致锁的竞争更加激烈,会影响系统的性能,在程序中需要尽可能减少线程对锁的持有时间,如下面的代码:

```java
public synchronized void syncMethod(){
	otherCode1();
	mutexMethod();
	otherCode2();
}
```

在syncMethod同步方法中,假设只有mutexMethod()方法是需要同步的,otherCode1()与otherCode2()方法不需要进行同步,如果otherCode1()与otherCode2()这两个方法需要花费较长的cpu时间,在并发量较大的情况下,这种同步方案会导致等待线程的大量增加.一个较好的优化方案是,只在必要时进行同步,可以减少锁的持有时间,挺高系统的吞吐量,如把上面的代码改为:

```java
public void syncMethod(){
	otherCode1();
	synchronized(this){
        mutexMethod();
    }
	otherCode2();
}
```

只对`mutexMethod()`进行同步

#### 2.减少锁的粒度

​	一个锁保护的共享数据的数量大小称为锁的粒度,如果一个锁保护的共享数据的数据量大就称该锁的粒度粗

锁的粒度过粗会导致线程在申请锁时需要进行不必要的等待,减少锁粒度是一种削弱多线程竞争的一种手段

​	在JDK7前,`java.util.cocurrent.ConcurrentHashMap`类采用分段锁协议,可以提高程序的并发性

#### 3.使用读写分离锁代替独占锁

​	使用`ReadWriteLock`读写分离锁可以提高系统的性能,使用读写分离锁也是减小锁粒度的一种特殊情况.

​	读写锁是对系统功能点的分割,**尤其是在读多于锁的情况下,对性能提升明显**

#### 4.锁分离

​	将读写锁的思想进一步延伸就是锁分离.读写锁是根据读写操作功能上的不同进行了锁分离.根据应用程序功能的特点,也可以对独占锁进行分离,如`java.util.concurrent.LinkBlockingQueue`类中`take()`与`put()`方法分别是从队头取数据,把数据添加到队尾

​	虽然这两个方法都是对队列进行修改操作,由于操作的主体是链表,take()操作的是链表的头部,put()操作的是链表的尾部,两者并不冲突,如果采用独占锁的话,这两个操作不能同时并发,在该类中就采用锁分离,take()取数据时有取锁,put()添加数据时有自己的添加锁,这样take()与put()相互独立实现了并发

#### 5.锁粗化

​	为了保证多线程间的有效并发,会要求每个线程持有锁的时间尽量短,但是凡事都有一个度,如果对同一个锁不断的进行请求,同步和释放,也会消耗额外的资源,如:

```java
public void method1(){
    synchronized(lock){
		同步代码块1
    }
    synchronized(lock){
        同步代码块2
    }
}
```

JVM在遇到一连串不断对同一个锁进行请求和释放操作时,会把所有的锁整合成对锁的一次请求,从而减少对锁的请求次数,这个操作叫锁的粗化,如上一段代码会整合为:
```java
public void method1(){
    synchronized(lock){
		同步代码块1
        
        同步代码块2
    }  
}
```

**在开发过程中,也应该有意识的在合理的场合进行锁的粗化,尤其在循环体内请求锁时**

### JVM对锁的优化

#### 1.锁偏向

​	锁偏向是一种针对加锁操作的优化,如果一个线程获得了锁,那么锁就进入偏向模式,当这个线程再次请求锁时,无需再做任何同步操作,这样可以节省有关锁申请的时间,提高了程序的性能.

​	锁偏向在没有锁竞争的场合可以有较好的优化效果,对于锁竞争比较激烈的场景,效果不佳,锁竞争激烈的情况下可能每次都是不同的线程来请求锁,这时偏向模式失效.

#### 2.轻量级锁

​	如果锁偏向失败,JVM不会立即挂起线程,还会使用一种称为轻量级锁的优化手段,会将共享对象的头部作为指针,指向持有锁的线程堆栈内部,来判断一个线程是否持有对象锁,如果线程获得轻量级锁成功,就进入临界区,如果获得轻量级锁失败,表示其他线程抢到了锁,那么当前线程的锁的请求就膨胀为重量级锁,当前线程就转到阻塞队列中变为阻塞状态

​	偏向锁,轻量级锁都是乐观锁,重量级锁是悲观锁

​	一个对象刚开始实例化时,没有任何线程访问它,它是可偏向的,即它任务只可能有一个线程来访问它,所以当第一个线程来访问它的时候,它会偏向这个线程,偏向第一个线程,这个线程在修改对象搜称为偏向锁时使用CAS操作,将对象头中ThreadId改为自己的Id,之后再访问这个对象时,只需要对比id即可,一旦有第二个线程访问该对象,因为偏向锁不会主动释放,所以第二个线程可以查看对象的偏向状态,当第二个线程访问对象时,表示在这个对象上已经存在竞争了,检查原来持有对象锁的线程是否存活,如果挂了则将对象变为无锁状态,然后重新偏向新的线程,如果原来的线程依然存活,则马上执行原来线程的操作栈,检查该对象的使用情况,如果仍然需要偏向锁,则偏向锁升级为轻量级锁.

​	轻量级锁认为竞争存在,但是竞争的程度很轻,一般两个线程对同一个锁的操作会错开,或者稍微等待一下(自旋)另外一个线程就会释放锁.又来第三个线程访问时,轻量级锁会膨胀为重量级锁,重量级锁除了持有锁的线程外,其他的线程都阻塞.

[TOC]

