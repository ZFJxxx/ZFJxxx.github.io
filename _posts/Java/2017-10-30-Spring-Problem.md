---
layout: post
title: Spring中的一些问题
date:   2017-10-30 19:15:10
categories:  Java
tags:  Spring
keywords: Spring
description: 
---


## Spring的bean在什么时候实例化? 
IOC容器有两种：基础版的BeanFactory和升级版的ApplicationContext

第一：如果你使用BeanFactory，如XmlBeanFactory作为Spring Bean的工厂类，则所有的bean都是在第一次使用该bean的时候实例化 。

第二：如果你使用ApplicationContext作为Spring Bean的工厂类，则又分为以下几种情况：      
*  如果bean的scope是singleton的，并且lazy-init为false（默认是false，所以可以不用设置），则ApplicationContext启动的时候就实例化该bean，并且将实例化的bean放在一个线程安全的 ConcurrentHashMap 结构的缓存中，下次再使用该Bean的时候，直接从这个缓存中取 。       
*  如果bean的scope是singleton的，并且lazy-init为true，则该bean的实例化是在第一次使用该bean的时候进行实例化 。      
*  如果bean的scope是prototype的，则该bean的实例化是在第一次使用该Bean的时候进行实例化 。

## 元数据信息

元数据信息主要是指BeanDefinition，BeanDefinition是对Bean定义的描述数据结构，是IOC容器的关键数据结构。元数据信息的准备过程主要有三步：

* 定位：Resource资源定位（文件、路径、网络等，是BeanDefinition的资源定位）
* 读取与转化：通过各类Reader和Parser将Resource解析为BeanDefinition数据结构
* 注册：将BeanDefinition注册到IOC容器中的HashMap中去

## 对象依赖关系图的建立
关系图的建立大体有三步：

* 实例化Bean
* 注入设置Bean的属性
* Bean的初始化

IOC容器持有了BeanDefinition后，就可以根据BeanDefinition中的信息在实例化Bean时逐步建立起该Bean的依赖关系图了。这一步与JVM中的类加载中解析阶段有些神似，将BeanDefinition中的“符号引用”解析为实例引用。

建立工作通过getBean()方法触发，会首先实例化该Bean(createBean()方法)，然后会对该Bean的属性进行设置（populateBean()方法），此时会向IOC容器再次getBean以获取属性值，依次类推，递归调用，最终建立对象依赖关系图。

## Bean的生命周期

* Bean实例的创建（getBean->createBean）
* 为Bean实例设置属性(populateBean)
* 初始化Bean(initializeBean)
* Bean 被应用使用
* 容器关闭时，Bean销毁(destoryMethod)

其中Bean的初始化主要依次做了4件事：

* 设定Bean对容器的感知（3个Aware接口）
* BeanPostProcessorsBeforeInitialization()方法
* 如果Bean实现了InitializingBean接口，调用afterPropertiesSet方法，如果指定了init-method，调用init-method
* BeanPostProcessorsAfterInitialization()方法
