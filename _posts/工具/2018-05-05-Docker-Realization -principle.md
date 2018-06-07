---
layout: post
title:   Docker实现原理
date:   2018-05-05 19:15:10
categories:  工具
tags: Docker
keywords: 
description: 
---

## Docker实现原理
linux是基于linux核心的namespaces和cgroup等来隔离资源,还有libvirt这样的接口实口实现，Dcoker基本上能当个虚拟机来用，又很轻量级。
Docker底层依赖Linux操作系统的命名空间（namespaces）、控制组（control Groups）、联合文件系统（Union File System）和Linux虚拟网络支持。

### 1.客户端和服务端
Docker采用的C/S构架，客户端和服务端可以运行在一台机器上，也可以用socket或者RESTful API来进行通信
 
 ![enter image description here](http://p7lixluhf.bkt.clouddn.com/docker.jpg)

Docker daemon一般在宿主机后台运行，作为服务端接收客户的请求，并处理这些请求（创建、运行、分发容器）。

Docker客户端则为用户提供一系列可执行命令，用户用这些命令实现与Docker daemon的交互。

### 2.NameSpace
命名空间是Linux内核针对实现容器虚拟化而引入的一个强大特性，
每个容器都可以拥有自己单独的命名空间，运行在其中的应用都像是在独立的操作系统中运行一样，保证了容器之间彼此不想和影响。

Linux通过NameSpace将内核、文件系统、网络、PID、UID、IPC、内存、硬盘、CPU等资源相互隔离，虽然这些进程都共用一个内核，但是他们彼此是不可见的。

### 3.CGroups
控制组是Linux内核的一个特性，主要用来对共享资源进行隔离、限制、审计。只有能控制分配到容器的资源，Docker才能避免多个容器同时运行时的系统资源竞争。

Cgroups是Linux内核功能，它能提供两种基础功能：
* 限制Linux进程组的资源占用（内存、CPU）
* 为进程组制作 PID、UTS、IPC、网络、用户及装载命名空间

Cgroups让Docker可以为进程提供一个隔离“系统”、可控的资源、运行环境。


