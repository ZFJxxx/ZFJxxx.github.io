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