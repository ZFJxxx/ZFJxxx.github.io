---
layout: post
title:   线程状态
date:   2017-03-02 20:15:10
categories:  Java
tags:  Java并发
keywords: 
description: 
---

## Java线程的状态

Java线程在某个时刻只能处于以下七个状态中的一个。
* 1.New（新创建），一个线程刚刚被创建出来，还没有调用start()方法；

* 2.Runnable（可运行状态），当线程对象调用了start()方法的时候，线程启动进入可运行的状态。

* 3.Running（运行状态），获取了时间片，运行run()方法中的代码

* 4.Blocked（阻塞状态），表示线程阻塞于锁

* 5.Waiting（等待状态），表示线程进入等待状态，需要其他线程做出一些特定动作

* 6.Timed_Waiting（超时等待）不同于Waiting,他是在指定时间内自行返回

* 7.Terminated（被终止），因为run方法正常退出而死亡，或者因为没有捕获的异常终止了run方法而死亡。 

![enter image description here](http://p7lixluhf.bkt.clouddn.com/threadstatus.jpg)

## wait()和sleep()的区别

sleep()来自Thread类，和wait()来自Object类

调用sleep()方法的过程中，线程不会释放对象锁。而 调用 wait()方法线程会释放对象锁。sleep()睡的时候只是抱着锁睡觉，会释放cpu时间片。wait()把时间片和资源的锁就全都释放出去了。

sleep(milliseconds)需要指定一个睡眠时间，时间一到会自动唤醒。wait()需要notify唤醒.

![enter image description here](http://p7lixluhf.bkt.clouddn.com/waitsleep.png)

## 进程和线程的区别
* 进程：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含1–n个线程。

* 线程：同一类线程共享代码和数据空间，每个线程有独立的工作内存和程序计数器(PC)，线程切换开销小。线程间还可以通过共享的主内存来进行值的传递。

* 多进程是指操作系统能同时运行多个任务（程序）。
* 多线程是指在同一程序中有多个顺序流在执行。
