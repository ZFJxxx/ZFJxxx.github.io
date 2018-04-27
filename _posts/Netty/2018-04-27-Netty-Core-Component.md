---
layout: post
title:  Netty核心组件
date:   2018-04-27 19:15:10
categories:  Netty
tags:  netty
keywords: Netty
description: 
---
----------------------------------

## Netty核心组件
Nettty 有如下几个核心组件：
```
Channel
EventLoop
ChannelFuture
ChannelHandler
ChannelPipeline
```

## Channel
Channel 是 Netty 网络操作抽象类，它除了包括基本的 I/O 操作，如 bind、connect、read、write 之外，还包括了 Netty 框架相关的一些功能，如获取该 Channel的 EventLoop。

在传统的网络编程中，作为核心类的 Socket ，它对程序员来说并不是那么友好，直接使用其成本还是稍微高了点。而Netty 的 Channel 则提供的一系列的 API ，它大大降低了直接与 Socket 进行操作的复杂性。而相对于原生 NIO 的 Channel，Netty 的 Channel 具有如下优势

* 在 Channel 接口层，采用 Facade 模式进行统一封装，将网络 I/O 操作、网络 I/O 相关联的其他操作封装起来，统一对外提供。
* Channel 接口的定义尽量大而全，为 SocketChannel 和 ServerSocketChannel 提供统一的视图，由不同子类实现不同的功能，公共功能在抽象父类中实现，最大程度地实现功能和接口的重用。
* 具体实现采用聚合而非包含的方式，将相关的功能类聚合在 Channel 中，有 Channel 统一负责和调度，功能实现更加灵活。


## EventLoop

Netty 基于事件驱动模型，使用不同的事件来通知我们状态的改变或者操作状态的改变。它定义了在整个连接的生命周期里当有事件发生的时候处理的核心抽象。

Channel 为Netty 网络操作抽象类，EventLoop 主要是为Channel 处理 I/O 操作，两者配合参与 I/O 操作。

下图是Channel、EventLoop、Thread、EventLoopGroup之间的关系

![enter image description here](http://p7lixluhf.bkt.clouddn.com/616953-20170509162546738-1955835364.png)
   
当一个连接到达时，Netty 就会注册一个 Channel，然后从 EventLoopGroup 中分配一个 EventLoop 绑定到这个Channel上，在该Channel的整个生命周期中都是有这个绑定的 EventLoop 来服务的。

## ChannelFuture
Future 提供了另一种在操作完成时通知应用程序的方式。每个 Netty 的出站 I/O 操作都将返回一个 ChannelFuture；也就是说，它们都不会阻塞。


Netty 为异步非阻塞，即所有的 I/O 操作都为异步的，因此，我们不能立刻得知消息是否已经被处理了。Netty 提供了 ChannelFuture 接口，通过该接口的 addListener() 方法注册一个 ChannelFutureListener，当操作执行成功或者失败时，监听就会自动触发返回结果。

每个 Netty 的出站 I/O 操作都将返回一个 ChannelFuture；也就是说，它们都不会阻塞。Netty 完全是异步和事件驱动的

## ChannelHandler

ChannelHandler 为 Netty 中最核心的组件，它充当了所有处理入站和出站数据的应用程序逻辑的容器。ChannelHandler 主要用来处理各种事件，这里的事件很广泛，比如可以是连接、数据接收、异常、数据转换等。

ChannelHandler 有两个核心子类 ChannelInboundHandler 和 ChannelOutboundHandler，其中 ChannelInboundHandler 用于接收、处理入站数据和事件，而 ChannelOutboundHandler 则相反。


## ChannelPipeline

ChannelPipeline 为 ChannelHandler 链提供了一个容器并定义了用于沿着链传播入站和出站事件流的 API。一个数据或者事件可能会被多个 Handler 处理，在这个过程中，数据或者事件经流 ChannelPipeline，由 ChannelHandler 处理。在这个处理过程中，一个 ChannelHandler 接收数据后处理完成后交给下一个 ChannelHandler，或者什么都不做直接交给下一个 ChannelHandler。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/Channelpipe.png)

当一个数据流进入 ChannlePipeline 时，它会从 ChannelPipeline 头部开始传给第一个 ChannelInboundHandler ，当第一个处理完后再传给下一个，一直传递到管道的尾部。与之相对应的是，当数据被写出时，它会从管道的尾部开始，先经过管道尾部的 “最后” 一个ChannelOutboundHandler，当它处理完成后会传递给前一个 ChannelOutboundHandler 。