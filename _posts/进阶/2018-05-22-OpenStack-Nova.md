---
layout: post
title:   OpenStack Nova
date:   2017-05-22 19:15:10
categories:  进阶
tags:  Openstack
keywords: 虚拟化
description: 
---

## OpenStack

首先我们要明白，OpenStack的作用。
OpenStack 为虚拟机提供并管理三大类资源：计算、网络和存储。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/openstack1.png)

所以我们的学习重点就是：
搞清楚 OpenStack 是如何对计算、网络和存储资源进行管理的。

## OpenStack架构

OpenStack众多组件中，最为重要的部分如下。 

![enter image description here](http://p7lixluhf.bkt.clouddn.com/openstack2.jpg)

其中Compute Service Nova 是 OpenStack 最核心的服务，负责维护和管理云环境的计算资源。 Neutron主要管理网络连接服务。Cinder主要管理存储。这三个部分是OpenStack中最为重要的部分。

## Nova

Nova控制着虚拟机的生老病死，管理他们的计算资源分配。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/nova.png)

### nova源码和执行流程

**1.发送http请求**

Client通过发送一个POST的http请求来实现请求虚拟机，需要注意的是，其http请求中的tokenid是来通过keystone的安全认证，content-type必须保证请求体发送类型为JSON，request body必须必须包含action状态、transaction_id，使用的zone区域、请求的server数量、flavor详细信息（包括disk大小、cpu核数、内存大小等）、镜像id和账号密码等。这里http符合RESTful风格。

**2.nova-api**
请求发送过来之后，第一个处理请求的接口时nova-api, 每个资源都被封装成为一个nova.api.openstack.wsgi.Resource 对象，且对应着一个 WSGI Application，http请求具体路由到哪个wsgi application，由/nova/etc/api_paste.ini来决定。Nova-api启动的时候，先根据enable_apis创建多个wsgi server,然后再运行过程中，根据paste deploy的配置文件路由到特定的wsgi application。路由到特定application之后如何执行函数呢？因为每个资源都被封装成一个resource对象，并对应着一个wsgi应用，APIRouterV21会建立路由规则，然后将自己完善的mapper对象交给基类router。监听完成后，RPC会调用/nova/conductor/manager/computeTaskMananger中的build_instances()方法.

![enter image description here](http://p7lixluhf.bkt.clouddn.com/nova-api.jpg)

**3.消息中间件**
/nova/openstack/common/rpc中封装了rpc的一系列实现，openstack系统解耦了各个组件之间的关系，都是基于AMQP实现了rpc进行通信，具体实现都已经被封装好，顾不深入研究。

**4.nova_conductor**
nova_couductor主要用来实现nova_compute和数据库之间的解耦，nova_api接收到http请求之后，会通过rpc远程调用nova_conductor,nova_conductor注册rpc server接收到rpc请求之后，由manager.py中的conductorManager完成访问数据库的操作。Nova_conductor会在build_instacces()方法中生成一个request_spec字典，里面包含了详细的虚机信息

**5.nova_scheduler**
nova_scheduler最主要的作用就是主要作用是裁决虚拟机的生存空间和资源分配，它通过考虑内存使用率与CPU负载等多种因素为虚拟机选择一个合适的主机。/nova/scheduler/filters中实现了很多种过滤方法，比如根据内存或者挑选出Active的主机等等，可以通过自己的需求来自行组合filters规则实现过滤，过滤玩之后就要进行weighting，权重分析，/nova/scheduler/weights中封装了RAMWeigher和Least Cost的计算权重的方法。计算之后就能根据权重系数选出合适的节点建立虚机。选出之后通过rpc调用nova_compute创建虚机。

**6.nova_compute**
nova-compute一般管理虚拟机的生命周期。但是，Nova并不提供虚拟化技术，只是借助主流虚拟化（比如：Xen KVM等）实现虚拟机的创建与管理，因此nova-compute需要和各种不同的hypervisor进行交互。目前Nova实现了Hyperv、Libvirt、VMware以及XenAPI四种。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/nova2.jpg)

Nova_compute首先会使用Resource Tracker的Claim机制检测主机的可用资源是否能够满足创建虚机的需要？然后通过指定的VirtDriver创建vm。Libvirt提供了api给nova_compute来实现vm的管理。
Nova_compute每创建迁移删除vm都要更新数据库中的内容，claim机制就是创建VM之前先测试主机的可用资源能否满足创建VM需要，满足就更新数据库，将主机上的可用资源删掉被创建的这部分，如果VM创建失败或者被删除，就将删掉的数据再回复回去。
