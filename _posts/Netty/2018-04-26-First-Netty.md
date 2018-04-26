---
layout: post
title:  Netty入门
date:   2018-04-26 19:15:10
categories:  Netty
tags: Netty
keywords: Netty
description: 
---
----------------------------------

## Netty是什么？
在知乎上看到一个非常NICE的描述，为什么需要使用Netty。

有了Netty，你可以实现自己的HTTP服务器，FTP服务器，UDP服务器，RPC服务器，WebSocket服务器，Redis的Proxy服务器，MySQL的Proxy服务器等等。

如果你想知道Nginx是怎么写出来的，如果你想知道Tomcat和Jetty是如何实现的，如果你也想实现一个简单的Redis服务器，那都应该好好理解一下Netty，它们高性能的原理都是类似的。

我们回顾一下传统的HTTP服务器的原理
* 1.创建一个ServerSocket，监听并绑定一个端口
* 2.一系列客户端来请求这个端口
* 3.服务器使用Accept，获得一个来自客户端的Socket连接对象
* 4.启动一个新线程处理连接
  *  （1）读Socket，得到字节流
  *  （2）解码协议，得到Http请求
  *  （3）对象处理Http请求，得到一个结果，封装成一个HttpResponse对象
  *  （4）编码协议，将结果序列化字节流
  *  （5）写Socket，将字节流发给客户端
*  5.继续循环步骤3

HTTP服务器之所以称为HTTP服务器，是因为编码解码协议是HTTP协议，如果协议是Redis协议，那它就成了Redis服务器，如果协议是WebSocket，那它就成了WebSocket服务器，等等。

使用Netty你就可以定制编解码协议，实现自己的特定协议的服务器。

上面我们说的是一个传统的多线程服务器，这个也是Apache处理请求的模式。在高并发环境下，线程数量可能会创建太多，操作系统的任务调度压力大，系统负载也会比较高。那怎么办呢？

于是NIO诞生了，NIO并不是Java独有的概念，NIO代表的一个词汇叫着IO多路复用。它是由操作系统提供的系统调用，早期这个操作系统调用的名字是select，但是性能低下，后来渐渐演化成了Linux下的epoll和Mac里的kqueue。我们一般就说是epoll，因为没有人拿苹果电脑作为服务器使用对外提供服务。而Netty就是基于Java NIO技术封装的一套框架。为什么要封装，因为原生的Java NIO使用起来没那么方便，而且还有臭名昭著的bug，Netty把它封装之后，提供了一个易于操作的使用模式和接口，用户使用起来也就便捷多了。

那NIO究竟是什么东西呢？NIO的全称是NoneBlocking IO，非阻塞IO，区别与BIO，BIO的全称是Blocking IO，阻塞IO。传统的多线程服务器是BlockingIO模式的，从头到尾所有的线程都是阻塞的。这些线程就干等在哪里，占用了操作系统的调度资源，什么事也不干，是浪费。

那么NIO是怎么做到非阻塞的呢。它用的是事件机制。它可以用一个线程把Accept，读写操作，请求处理的逻辑全干了。如果什么事都没得做，它也不会死循环，它会将线程休眠起来，直到下一个事件来了再继续干活，这样的一个线程称之为NIO线程。

Netty是建立在NIO基础之上，Netty在NIO之上又提供了更高层次的抽象。

在Netty里面，Accept连接可以使用单独的线程池去处理，读写操作又是另外的线程池来处理。

Accept连接和读写操作也可以使用同一个线程池来进行处理。而请求处理逻辑既可以使用单独的线程池进行处理，也可以跟放在读写线程一块处理。线程池中的每一个线程都是NIO线程。用户可以根据实际情况进行组装，构造出满足系统需求的并发模型。

Netty提供了内置的常用编解码器，包括行编解码器［一行一个请求］，前缀长度编解码器［前N个字节定义请求的字节长度］，可重放解码器［记录半包消息的状态］，HTTP编解码器，WebSocket消息编解码器等等

Netty提供了一些列生命周期回调接口，当一个完整的请求到达时，当一个连接关闭时，当一个连接建立时，用户都会收到回调事件，然后进行逻辑处理。Netty可以同时管理多个端口，可以使用NIO客户端模型，这些对于RPC服务是很有必要的。

Netty除了可以处理TCP Socket之外，还可以处理UDP Socket。

在消息读写过程中，需要大量使用ByteBuffer，Netty对ByteBuffer在性能和使用的便捷性上都进行了优化和抽象。

## Netty优点
最早的Java是所谓的阻塞IO(Blocking IO)， 想处理多个socket的连接的话需要创建多个线程， 一个线程对应一个。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/Netty.png)

 这种方式写起来倒是挺简单的，但是连接（socket）多了就受不了了，如果真的有成千上万个线程同时处理成千上万个socket，占用大量的空间不说，光是线程之间的切换就是一个巨大的开销。更重要的是，虽然有大量的socket，但是真正需要处理的（可以读写数据的socket）却不多，大量的线程处于等待数据状态

后来Java为了解决这个问题，又搞了一个非阻塞IO(NIO：Non-Blocking IO，有人也叫做New IO)， 改变了一下思路：通过多路复用的方式让一个线程去处理多个Socket。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/Netty2.png)

这样一来，只需要使用少量的线程就可以搞定多个socket了，线程只需要通过Selector去查一下它所管理的socket集合，哪个Socket的数据准备好了，就去处理哪个Socket，一点儿都不浪费。

Netty就是一个基于NIO的网络框架， 封装了Java NIO那些复杂的底层细节，给你提供简单好用的抽象概念来编程。
