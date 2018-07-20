---
layout: post
title:   深入Synchronized
date:   2017-03-06 19:15:10
categories:  Java
tags:  Java并发
keywords: 
description: 
---

## Synchronized的基本使用

Synchronized是Java中解决并发问题的一种最常用的方法，也是最简单的一种方法。Synchronized的作用主要是确保线程互斥的访问同步代码

Synchronized总共有三种用法：

* 1.对于修饰普通方法，锁是当前实例对象。
* 2.对于修饰静态方法，锁是当前类的Class对象。
* 3.对于修饰代码块，锁是Synchonized括号里配置的对象。

```
//对方法同步
public synchronized void method(){
    ..............................
}
```
```
//对代码块同步
public void method(){
    synchronized(obj){
        ..........................
    }
}
```

## Synchronized 原理

如果对上面的执行结果还有疑问，也先不用急，我们先来了解Synchronized的原理，再回头上面的问题就一目了然了。我们先通过反编译下面的代码来看看Synchronized是如何实现对代码块进行同步的：

```
public class SynchronizedDemo {
     public void method() {
         synchronized (this) {
            System.out.println("Method 1 start");
         }
     }
}
```
反编译结果：
![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/synchronized1.jpg)

于这两条指令的作用，我们直接参考JVM规范中描述：

monitorenter ：
> Each object is associated with a monitor. A monitor is locked if and only if it has an owner. The thread that executes monitorenter attempts to gain ownership of the monitor associated with objectref, as follows:
• If the entry count of the monitor associated with objectref is zero, the thread enters the monitor and sets its entry count to one. The thread is then the owner of the monitor.
• If the thread already owns the monitor associated with objectref, it reenters the monitor, incrementing its entry count.
• If another thread already owns the monitor associated with objectref, the thread blocks until the monitor's entry count is zero, then tries again to gain ownership.

这段话的大概意思为：

每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程如下：

1、如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。

2、如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.

3.如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

monitorexit：　

> The thread that executes monitorexit must be the owner of the monitor associated with the instance referenced by objectref.
The thread decrements the entry count of the monitor associated with objectref. If as a result the value of the entry count is zero, the thread exits the monitor and is no longer its owner. Other threads that are blocking to enter the monitor are allowed to attempt to do so.

这段话的大概意思为：

执行monitorexit的线程必须是objectref所对应的monitor的所有者。

指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。 

通过这两段描述，我们应该能很清楚的看出Synchronized的实现原理，Synchronized的语义底层是通过一个monitor的对象来完成，其实wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因。

**我们再来看一下同步方法的反编译结果：**

源代码：

```
package com.paddx.test.concurrent;
	public class SynchronizedMethod {
	     public synchronized void method() {
	         System.out.println("Hello World!");
     }
}
```

反编译结果：
![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/synchronized2.jpg)


　　从反编译的结果来看，方法的同步并没有通过指令monitorenter和monitorexit来完成（理论上其实也可以通过这两条指令来实现），不过相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符。JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。
  
## Synchronize关键字为什么jdk1.5后效率提高了
Java SE 1.6为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁”，级别从低到高依次是：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态。偏向所锁，轻量级锁都是乐观锁，重量级锁是悲观锁。

### 偏向锁
大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。偏向锁默认开启。

引入偏向锁的目的：在没有多线程竞争的情况下，尽量减少不必要的轻量级锁执行路径，轻量级锁的获取及释放依赖多次CAS原子指令，而偏向锁只依赖一次CAS原子指令置换ThreadID，不过一旦出现多个线程竞争时必须撤销偏向锁，所以撤销偏向锁消耗的性能必须小于之前节省下来的CAS原子操作的性能消耗，不然就得不偿失了。

当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头里是否存储着指向当前线程的偏向锁。如果测试成功，表示线程已经获得了锁。

偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。如果你确定应用程序里所有的锁通常情况下处于竞争状态，可以通过JVM参数关闭偏向锁，那么程序默认会进入轻量级锁状态。

### 轻量级锁
轻量级锁适用的场景是在线程交替获取某个锁执行同步代码块的场景，如果出现多个进程同时竞争同一个锁时，轻量级锁会膨胀成重量级锁。

轻量级锁认为竞争存在，但是竞争的程度很轻，一般两个线程对于同一个锁的操作都会错开，或者说稍微等待一下（自旋），另一个线程就会释放锁。 但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁，重量级锁使除了拥有锁的线程以外的线程都阻塞，防止CPU空转。


### 自旋锁
这也是对锁的一种优化，线程的挂起和恢复会给系统的并发性带来压力；

共享数据的锁定状态一般只持续很短一段时间，为了这段时间区挂起恢复线程不值得；如果物理机由多核处理器，就能让多线程并行执行，我们就能让后面请求锁的线程“稍等”，但是并不挂起线程去等待，看持有锁的线程会不会很快的释放锁。

我们只需让线程执行一个忙循环（自旋），这就是所谓的自旋锁。

自旋锁避免了线程切换的开销，如果锁占用的时间短，就很好，反正锁被占用的时间长，自旋的线程只会白白消耗处理器资源。

![](http://p7lixluhf.bkt.clouddn.com/sychronized.PNG)
