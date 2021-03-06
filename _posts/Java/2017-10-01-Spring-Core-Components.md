---
layout: post
title:  Spring源码解析（1）核心组件
date:   2017-10-1 19:15:10
categories:  Java
tags:  Spring
keywords: String
description:         
---

Spring的核心组件只有3个
```
Context
Core
Bean
```

Spring是面向Bean的编程，所以Bean在Spring是主要角色。Spring把Object之间的依赖关系转换成了用配置文件或者注解来管理，也就是依赖注入机制，而这个依赖注入关系在一个觉IOC（控制反转）的容器中管理。

IOC容器中存放着被Bean包裹的对象，Spring就是把对象包裹在Bean中从而达到管理这些对象以及一系列额外的目的。

## 核心组件协调工作
Bean是Spring中的关键因素，那么Context和Core呢？如果把Bean比作一场演出中的演员，Context就是这场演出的舞台背景，Core就是演出的道具。

我们知道Bean包装的是Object，而Object必然有数据，如何给这些数据提供生存环境就是Context解决的问题，Context就是要发现每个Bean之间的关系，为它们建立这种关系并且维护好这种关系。所以Context就是一个Bean关系的集合，这个关系集合也叫IOC容器。一旦建立好这个IOC容器，Spring就可以为你工作了。Core就是发现、建立和维护每个Bean之间的关系所需要的一系列工具，也许把Bean叫做Util更加贴切。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/Spring%281%29.jpg)
