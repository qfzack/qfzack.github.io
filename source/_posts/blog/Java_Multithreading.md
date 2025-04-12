---
title: "学习笔记：Java并发、多线程"

date: 2020-04-28T05:00:00Z
# image: "/images/placeholder.png"
categories: ["Java"]
author: "Qingfeng Zhang"

---

# 1.基本概念

## 1.1 线程与进程

操作系统是包含多个进程的容器，而每个进程又都是容纳多个线程的容器；  
**进程** 是程序的一次运行，在用户下达运行程序的命令后，就会产生进程；可以把进程理解为程序的运行实例，是资源分配的基本单位；  
**线程** 是CPU的基本调度单位，每个线程执行的都是进程代码的某个片段；

**进程与线程的不同：**  
1.处理器的速度远大于外设，线程是为了提高CPU的利用率；  
2.进程是具有独立功能的程序运行的活动，是系统分配资源和调度的基本单位，线程是CPU的基本调度单位；  
3.内存共享方式不同，进程之间的内存一般不共享，线程服务于同一个进程，之间可以通信；  
4.拥有的资源不同，不同的线程会共享进程的代码段等信息，但每个线程都会有自己的堆栈；  
5.一个进程至少包含一个线程；  
6.线程开销更小，线程的创建、终止时间比进程短；  
**进程与线程的相似点：** 生命周期（都包含创建、就绪、等待、运行、结束等状态）

Java在设计之初就支持多线程，并且线程是一对一映射到操作系统的内核线程；  
JVM会自动启动线程：  
Signal Dispatcher：把操作系统发来的信号分发给适当的处理程序，连接操作系统和应用程序；  
Finalizer：负责对象的finalize()方法；  
Reference Handler：和垃圾回收GC、引用相关的线程；  
main：主线程，用户程序的入口；

## 1.2 多线程

**多线程的概念** ：如果一个程序允许两个或两个以上的线程，那么它就是一个多线程程序。多线程是指在单个进程中运行多个线程；  
**多线程的作用** ：  
1.提高CPU的利用率（提高处理速度，避免无效等待，提高用户体验）；  
2.便于编程建模（拆解任务，每个线程负责一个任务）；  
3.摩尔定律失效，CPU性能很难提高，要求程序的并行化以提高性能；  
**多线程的局限** ：  
1.性能问题：上下文切换带来的消耗；  
2.异构化任务很难高效并行；  
3.线程安全问题：包括数据安全问题以及线程带来的活跃性问题；

## 1.3 串行、并行、并发

**并发** ：  
1.形容多个任务的执行状态：两个或多个任务可以在重叠的时间段内启动、运行和完成，即在一段时间内交替运行；  
2.并发性的简称：**并发性是程序并发和并行执行的前提条件**
，指程序不同的部分可以无序或同时执行，且不影响最终结果，在不同核心的计算机上表现不同（此时并行和并发不是同一维度上的）；  
**并行** ：在同一时刻，有多个任务同时执行；  
![](images/Java_Multithreading/1.png)  
由上图可以看出**并发是单处理器，逻辑上的同时运行** ，**并行是多处理器，物理上的同时运行**
，因此单处理器无法实现并行，且并行（多个线程同时执行）一定是并发

## 1.4 高并发

**高并发** 指有大量的请求同时到达服务端，而**多线程** 是为了防止高并发带来的一些线程安全和性能问题，即多线程是应对高并发场景的一种重要的解决方案；  
高并发并不意味着是多线程，比如数据库中的高并发可以用Redis应对，而Redis的底层是单线程处理的；  
高并发的指标：

  * QPS(Queries Per Second)每秒查询数
  * 带宽
  * PV(Page View)
  * UV(Unique Visitor)
  * IP
  * 并发连接数
  * 服务器平均请求等待时间

## 1.5 同步与异步、阻塞与非阻塞

**同步** ：同步异步指的是被调用者(服务器)的行为，而不是请求方的行为，同步是指在得到结果之前，服务端不返回任何结果；  
**异步** ：调用在发出之后，服务端会立刻返回，告诉调用方已收到请求；  
![](images/Java_Multithreading/2.png)  
**阻塞与非阻塞** ：调用者在调用一个东西之后，结果返回前是否可以做其他事；

  * 同步阻塞
  * 同步非阻塞
  * 异步阻塞
  * 异步非阻塞

# 2.线程的创建

有三种方式创建线程：  
**1.继承Thread类：**  
继承Thread类并重写run()方法，调用start开启线程，thread类就是实现了Runnable接口；

```java
public class TestThread extends Thread{  
    @Override  
    public void run(){  
        for(int i=0;i<20;i++){  
            System.out.println("Learning."+i);  
        }  
    }  
    
    public static void main(String[] args){  
        //main线程，主线程  
    
        //创建线程对象  
        TestThread test = new TestThread();  
        //调用start()方法开启线程  
        test.start();  //调用run只有主线程一条路径，调用start才是多线程同时执行，由CPU调度  
    
        for(int i=0;i<20;i++){  
            System.out.prinltn("Playing."+i);  
        }  
    }  
}  
``` 
  
**2.实现Runnable接口：**  
实现Runnable接口并重写run方法，执行线程需要放入Runnable接口的实现类，调用start方法；

```java    
public class TestThread implements Runnable{  
    @Override  
    public void run(){  
        for(int i=0;i<20;i++){  
            System.out.println("Learning."+i);  
        }  
    }  
    
    public static void main(String[] args){  
        //main线程，主线程  
    
        //创建Runnable接口的实现类对象  
        TestThread test = new TestThread();  
        //创建线程对象，通过线程对象开启线程，代理  
        Thread thread = new Thread(test);  
        //调用start()方法开启线程  
        thread.start();  
    
        for(int i=0;i<20;i++){  
            System.out.prinltn("Playing."+i);  
        }  
    }  
}  
```
  
**因为Java是单继承的，所以推荐使用Runnable接口** ；

**3.实现Callable接口：**  
可以定义返回值并抛出异常；

```java
public class TestThread implements Callable<Boolean>{  
    @Override  
    public boolean run(){  
        for(int i=0;i<20;i++){  
            System.out.println("Learning."+i);  
        }  
        return true;  
    }  
    
    public static void main(String[] args){  
        //main线程，主线程  
    
        //创建callable接口的实现类对象  
        TestThread test = new TestThread();  
        //创建执行服务  
        ExecutorService ser = Executors.newFixedThreadPool(1);  
        //提交任务  
        Future<boolean> r = ser.submit(test);  
        //获取执行结果  
        boolean rs = r.get();  
        //关闭服务  
        ser.shutdownNow();   
    
        for(int i=0;i<20;i++){  
            System.out.prinltn("Playing."+i);  
        }  
    }  
}  
```
  
# 3.线程的状态与方法

**线程的5个状态：**  
创建状态、就绪状态、阻塞状态、运行状态、死亡状态；  
**状态标记：**  
NEW(线程尚未启动)  
RUNNABLE(在JVM中执行的线程)  
BLOCKED(被阻塞等待监视器锁定)  
WAITING(正在等待另一个线程执行特定动作)  
TIMED_WAITING(正在等待另一个线程执行动作到达指定等待时间)  
TERMINATED(线程已退出)

方法 | 说明  
---|---  
setPriority(int newPriority) | 更改线程优先级  
sleep(long millis) | 指定时间长度让线程休眠(毫秒)  
join() | 合并线程，待此线程执行完再执行其他线程(插队)，其他线程阻塞  
yield() | 暂停当前正在执行的线程(礼让)，CPU重新调度(可能还是原线程执行)  
interrupt() | 中断线程(不建议使用)  
isAlive() | 测试线程是否处于活动状态  
  
# 4.线程优先级

Java提供了一个线程调度器来监控程序中启动后进入就绪状态的所有线程，线程调度器按照优先级决定应该调度哪个线程来执行(优先级高调度的概率则高)；  
线程优先级的范围是1~10：

  * Thread.MIN_PRIORITY = 1;
  * Thread.MAX_PRIORITY = 10;
  * Thread.NORM_PRIORITY = 5;

通过`getPriority()`获取线程优先级，`setPriority(int n)`改变线程优先级；

# 5.守护线程

线程分为**用户线程** 和**守护(daemon)线程**
，虚拟机需要保证用户线程执行完毕，而不用等守护线程执行完毕，如后台操作日志，监控内存，垃圾回收等；

# 6.线程的同步

处理多线程问题时，会出现多个线程访问同一个对象的情况，此时会出现线程不安全，数据紊乱，如下：

```java
    public class test implements Runnable{  
        private int ticket = 100;  
      
        @Override  
        public void run(){  
            while(true){  
                if(ticket<=0) break;  
                try{  
                    Thread.sleep(200);  //延时  
                }catch(InterruptedException e){  
                    e.printStackTrace();  
                }  
                System.out.println(Thread.currentThread().getName()+ticket--);  
            }  
        }  
      
        public static void main(String[] args){  
            Test test = new Test();  
      
            new Thread(test,"A").start();  
            new Thread(test,"B").start();  
            new Thread(test,"C").start();  
        }  
    }  
```
  
线程同步是一种等待机制，多个需要同时访问此对象的线程进入这个**对象的等待池**
中形成队列，等前面的线程使用完毕，下一个线程再使用，为了保证数据在方法中被访问时的正确性，在访问时加入**锁机制synchronized**
，当一个线程获得对象的排它锁，独占资源，其他线程必须等待，使用后释放锁，锁机制存在以下问题：

  * 一个线程持有锁会导致其他需要此锁的线程挂起；
  * 多线程竞争下，加锁、释放锁会导致较多的上下文切换和调度延时，引起性能问题；
  * 如果一个优先级高的线程等待一个优先级低的线程释放锁，会导致优先级导致，引起性能问题；

实现锁机制的synchronize关键字有两个用法：**synchronize方法** 和**synchronize块** ；  
synchronized方法控制对对象的访问，每个对象对应一把锁，每个synchronized方法都必须获得调用该方法的对象的锁才能执行，否则线程阻塞，方法一旦执行，就独占该锁，直到该方法返回才释放锁；  
除了synchronized外还有lock也可以实现锁机制；

关于**ArrayList线程不安全** 的一个例子：

```java
public class unsafe{  
    List<String> l = new ArrayList<>();  
    for(int i=0;i<10000;i++){  
        new Thread(() -> {  
            l.add(Thread.currentThread().getName());  //放入当前线程的名字  
        }).start();  
    }  
    System.out.println(l.size());  //小于10000  
    //因为有多个线程同时操作l  
}  
```
  
可以使用synchronize块使得上面的ArrayList线程同步：

```java    
    public class unsafe{  
        List<String> l = new ArrayList<>();  
        for(int i=0;i<10000;i++){  
            new Thread(() -> {  
                synchronized(l){  
                    l.add(Thread.currentThread().getName());   
                }  
            }).start();  
        }  
        System.out.println(l.size());  //等于10000  
    }  
```

`java.util.concurrent`中的CopyOnWriteArrayList就是一个比较线程安全的容器；

# 7.死锁

多个线程各自占有一些共享资源，并且相互等待其他线程占有的资源才能运行，导致两个或多个线程都在等待对方释放资源；某一个同步块同时拥有**两个以上对象的锁**
时，就可能会发生死锁；

# 8.线程协作

Java中解决线程通信的几个方法(Object类的方法)：

方法 | 说明  
---|---  
wait() | 表示线程会一直等待，直到其他线程通知，与sleep不同，会释放锁  
wait(long timeout) | 指定等待时间  
notify() | 唤醒一个处于等待的线程  
notifyAll() | 唤醒同一个对象上所有调用wait()的线程，按优先级调度  
  
# 9.线程池

经常创建和销毁、使用量特别大的资源，比如并发情况下的线程，对性能影象很大，可以提前创建多个线程，放入**线程池**
中，使用时直接获取，使用完放回，可以避免频繁创建和销毁，实现重复利用，好处是提高响应速度、降低资源消耗、便于线程管理；
