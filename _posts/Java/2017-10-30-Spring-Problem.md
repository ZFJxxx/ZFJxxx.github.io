---
layout: post
title: Spring中的一些问题
date:   2016-09-30 19:15:10
categories:  Java
tags:  Spring
keywords: Spring
description: 
---


## Spring的bean在什么时候实例化? 

第一：如果你使用BeanFactory，如XmlBeanFactory作为Spring Bean的工厂类，则所有的bean都是在第一次使用该bean的时候实例化 。

第二：如果你使用ApplicationContext作为Spring Bean的工厂类，则又分为以下几种情况：      
*  如果bean的scope是singleton的，并且lazy-init为false（默认是false，所以可以不用设置），则ApplicationContext启动的时候就实例化该bean，并且将实例化的bean放在一个线程安全的 ConcurrentHashMap 结构的缓存中，下次再使用该Bean的时候，直接从这个缓存中取 。       
*  如果bean的scope是singleton的，并且lazy-init为true，则该bean的实例化是在第一次使用该bean的时候进行实例化 。      
*  如果bean的scope是prototype的，则该bean的实例化是在第一次使用该Bean的时候进行实例化 。

