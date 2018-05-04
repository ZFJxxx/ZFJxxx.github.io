---
layout: post
title: Spring源码分析（5）Context组件
date:   2017-10-04 19:15:10
categories:  Java
tags:  Spring
keywords: Spring
description: 
---
----------------------------------

## Context组件

我们再回到[Spring源码分析（3）](https://zfjxxx.github.io/2017-10-03-IOC/)中提到的场景。

```
public void test() {
	ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
	SomeBean someBean= (SomeBean) context.getBean("someBean");
	someBean.doSomething();
}
```

这里的ClassPathXmlApplicationContext，是我们用来读文件的，可以读配置的文件的类不计其数，现在只取一个分析。

ClassPathXmlApplicationContext是AbstractXmlApplicationContext的子类，其顶级父类是ResourceLoader。我们先看看ResourceLoader的作用。
```
public interface ResourceLoader {
    //定义了加载地址
    String CLASSPATH_URL_PREFIX = "classpath:";
    //定义了资源加载的方法	
    Resource getResource(String var1);
    //定义了类加载器
    ClassLoader getClassLoader();
}
```
ResourceLoader就是IoC的灵魂嘛，负责就是找到资源和加载资源。Resource就是定义资源，ClassLoader定义类加载器。

我们再来看ApplicationContext,

![enter image description here](http://p7lixluhf.bkt.clouddn.com/applicationContext.png)

ApplicationContext继承BeanFactory，说明ApplicationContext的主体是对Bean进行操作，ApplicationContext继承ResourceLoader接口使得ApplicationContext可以访问任何的外部资源。

```
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver {

	//返回这个context的唯一ID
	@Nullable
	String getId();

	//返回此上下文所属的已部署应用程序的名称
	String getApplicationName();

	//返回这个Context的friendly name 
	String getDisplayName();

	//返回这个Context初次被加载的时间戳
	long getStartupDate();

	//返回父Context，如果没有父项，则返回null。
	@Nullable
	ApplicationContext getParent();

	//get一个AutowireCapableBeanFactory 
	AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;

}
```
Context作为Spring的IOC容器，基本整合了Spring的大部分功能的基础。

总体来说 ApplicationContext 必须要完成以下几件事：

* 标识一个应用环境
* 利用 BeanFactory 创建 Bean 对象
* 保存对象关系表
* 能够捕获各种事件
