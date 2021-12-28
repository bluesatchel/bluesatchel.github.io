---
title: java多线程
date: 2021-12-23 23:24:47
tags: 
     - java
     - 多线程
categories: java
description: java多线程
typora-root-url: ..
---

#### 多线程相关概念

并发:同一个时刻,多个任务交替执行,造成一种貌似同时的错觉,简单的说,单核CPU实现的多任务就是并发

并行:同一个时刻,多个任务同时执行,多核CPU可以实现并行

#### 两种实现多线程方式的对比分析

直接继承Thread类和实现Runnable接口都能实现多线程,具体有什么区别呢

1.从java的设计来看,通过继承Thread或者实现Runnable接口来创建线程本质上都没有区别,Thread类本身就实现了Runnable接口

2.实现Runnable接口方式更加适合多个线程共享一个资源的情况,并且避免了单继承的限制

#### 直接继承Thread

通过两段代码来对比一下

```java
public class S8_1 {
    public static void main(String[] args) {
        new TicketWindow().start();
        new TicketWindow().start();
        new TicketWindow().start();
        new TicketWindow().start();
    }
}
class TicketWindow extends Thread{
    private int tickies=100;
    @Override
    //重写run方法
    public void run(){
        while(true){
            if(tickies>0){
                Thread th= Thread.currentThread();//获取当前线程
                String th_name=th.getName();//获取当前线程名称
                System.out.println(th_name+"正在发售"+tickies--+"张票");
            }
        }
    }
}

```

![image-20211223233326400](/images/java%E5%A4%9A%E7%BA%BF%E7%A8%8B/image-20211223233326400.png)

由运行结果可知,当前每个线程处于"各卖各的票"的情况,对tickies没有实现共享

##### 为什么不调用run方法而调用start方法?

因为run方法就只是一个普通的方法,没有真正的启动一个线程

```java
(1)
public synchronized void start(){
    start0();
}

(2)
private native void start0();
//start0()是本地方法(native),是JVM调用,底层是c/c++实现
//真正实现多线程的效果,是start0()方法
```

**start()方法调用start0()方法后,该线程并不一定会立刻执行,只是将线程变成了可运行状态,具体什么时候执行,取决于cpu,由cpu统一调度**

#### 实现Runnable接口

##### 代理模式

<span id="example">例子</span>

该方法底层使用了一个设计模式**线程代理模式**,接下来用代码进行模拟,代码和真正有差别,但是原理一样

```java
public class proxy {
    public static void main(String[] args) {
        Tiger tiger=new Tiger();//Tiger类实现了Runnable接口
        ThreadProxy threadProxy = new ThreadProxy(tiger);//为什么能把tiger放进去,因为tiger实现了Runnable
        threadProxy.start();
    }
}
class Animal{

}
class Tiger extends Animal implements Runnable{

    @Override
    public void run() {
        System.out.println("老虎嗷嗷叫");
    }
}
class ThreadProxy implements Runnable{
    private Runnable target =null;//类型是Runnable的属性
    @Override
    public void run() {
        if(target!=null){
            target.run();//动态绑定,最后调用的还是tiger的run
        }
    }
    //构造方法
    public ThreadProxy(Runnable target){
        this.target=target;
    }
    public void start(){
        start0();
    }
    public void start0(){
        run();
    }
}

总的来说就是,Tiger没有start方法,但是通过实现Runnable接口,就可以通过Runnable做中间代理从而使用Runnable中的start方法来调用Tiger的run方法实现线程的启动
```

在通常的开发当中，一般会选择实现Runnable接口，原因有二：
1、避免单继承的局限，在Java当中一个类可以实现多个接口，但只能继承一个类
2、适合资源的共享

缺点:编程多一层对象包装,如果线程有执行结果是不可以直接返回的

<span id="sell">售票程序</span>

```java
public class S8_1 {
    public static void main(String[] args) {
         TicketWindow tw=new TicketWindow();//创建TicketWindow的实例对象tw
        //这几个线程其实都在执行tw这个对象
         new Thread(tw,"窗口1").start();
         new Thread(tw,"窗口2").start();
         new Thread(tw,"窗口3").start();
         new Thread(tw,"窗口4").start();
    }
}
class TicketWindow implements Runnable{
    private int tickies=100;
    public void run(){
        while(true){
            if(tickies>0){
                Thread th= Thread.currentThread();//获取当前线程
                String th_name=th.getName();//获取线程名称
                System.out.println(th_name+"正在发售"+tickies--+"张票");
            }
        }
    }
}
//可能会出现超卖的现象
```

<img src="/images/java%E5%A4%9A%E7%BA%BF%E7%A8%8B/image-20211223234915498.png" alt="image-20211223234915498" style="zoom:67%;" />

这样就不会出现重复卖票的情况,实现了tickies的共享

Runnable接口还可以通过匿名对象来实现

```java
public class test {
    public static void main(String[] args) {
        Runnable target=new Runnable() {
            @Override
            public void run() {
                System.out.println("子线程");
            }
        };
        Thread t=new Thread(target);
        t.start();
    }
}

```

还可以进一步简化上面的代码

```java
public class test {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("子线程");
            }
        }).start();
    }
}

//jdk8后更简化
new Thread(()->{
            System.out.println("子线程2");
        }).start();


```

#### 第三种多线程实现方案

##### 利用Callable,FutureTask接口实现





![image-20211224011547555](/images/java%E5%A4%9A%E7%BA%BF%E7%A8%8B/image-20211224011547555.png)



##### 线程的常用方法

setName //设置线程名称,使之与参数name相同

getName  //返回该线程的名称

常用`Thread.currentThread().getName()`

start //使该线程开始执行,java虚拟机底层调用该线程的start0方法

run //调用线程对象run方法

setPriority //更改线程优先级

getPriority //获取线程的优先级

sleep //在指定的毫秒数内让当前正在执行的线程休眠

interrupt //中断线程,但是并没有真正结束线程,往往是中断休眠sleep,转而执行catch中的语句



yield //是Thread的静态方法,直接`Thread.yield()`调用即可线程的礼让,让出cpu,让其他线程执行,但让出的时间不确定,所以也不一定礼让成功

join //线程的插队.插队的线程一旦插队成功,则肯定先执行完插入的线程所有的任务

案例:创建一个子线程,每隔1s输出hello,输出20次,主线程每隔一秒,输出hi,输出20次,要求:两个完成同时执行,当主线程输出5次后,就让子线程运行完毕,主线程再继续





##### 用户线程和守护线程

1.用户线程:也叫工作线程,当线程的任务执行完或通知方式结束

2.守护线程:一般是为工作线程服务的,当所有的用户线程结束,守护线程自动结束

3.常见的守护线程:垃圾回收机制

##### 如何把一个子线程设置为守护线程

`myDaemonThread.setDaemon(true)`就可以把一个子线程设置为守护线程,主线程结束 它也结束

```java
public class S8_5 {
    public static void main(String[] args) throws Exception{
        MyDaemonThread myDaemonThread=new MyDaemonThread();
        myDaemonThread.setDaemon(true);
        myDaemonThread.start();
        for(int i=0;i<3;i++){
            System.out.println("主线程运行中");

        }
    }
}
class MyDaemonThread extends Thread{
    @Override
    public void run(){
        while(true) {
            System.out.println("子线程运行中");
        }
    }
}
```

主线程结束后,子线程也会结束



##### 线程的生命周期

<img src="/images/java%E5%A4%9A%E7%BA%BF%E7%A8%8B/image-20211225234147135.png" style="zoom: 80%;" />

#### 线程同步机制

在多线程编程中,一些敏感数据不允许被多个线程同时访问,此时就使用同步访问技术,保证数据在任何统一时刻,最多有一个线程访问,以保证数据的完整性

也可以这样理解:线程同步,即当有一个线程在对内存进行操作时,其他线程都不可以对这个内存地址进行操作,知道该线程完成操作,其他线程才能对该内存地址进行操作

##### 同步具体方法

1.同步代码块

```java
synchronized(对象){//得到对象的锁,才能操作同步代码
	//需要被同步的代码
}
```

2.synchronized还可以放在方法声明中,表示整个方法为同步方法

```java
public synchronized void m(String name){
	//需要被同步的代码
}
```

3.如何理解

就好像上厕所,A上厕所关上门(上锁),完事出来后(解锁),那么其他人就可以再使用厕所了

例子

之前的售票窗口程序,通过继承Runnable接口,虽然实现了票数的共享,但是售票顺序是乱的

[售票窗口程序](#sell)

通过线程同步,就可以实现每次只卖一张票,也就是每次只允许一个线程访问tickies变量并执行,这样还可以防止超卖现象

```java
public class S8_1 {
    public static void main(String[] args) {
         TicketWindow tw=new TicketWindow();
         new Thread(tw,"窗口1").start();
         new Thread(tw,"窗口2").start();
         new Thread(tw,"窗口3").start();
         new Thread(tw,"窗口4").start();
    }
}
class TicketWindow implements Runnable{
    private int tickies=100;
    private boolean loop=true;
    public synchronized void sell(){
            if(tickies>0){
                Thread th= Thread.currentThread();//获取当前线程
                String th_name=th.getName();//获取线程名称
                System.out.println(th_name+"正在发售"+tickies--+"张票");
            }else{
                loop=false;
            }
    }
    public void run(){
        while(loop){
            sell();//sell是一种同步方法
        }
    }
}
```

由运行结果可知,票按照tickies大小逐次递减卖出,就是不知道为啥,每次只有一到两个线程能抢到cpu资源并且执行

![image-20211226000540093](/images/java%E5%A4%9A%E7%BA%BF%E7%A8%8B/image-20211226000540093.png)
