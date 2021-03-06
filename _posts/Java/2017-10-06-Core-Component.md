---
layout: post
title: Spring源码分析（6）Core组件
date:   2017-10-06 19:15:10
categories:  Java
tags:  Spring
keywords: Spring
description: 
---
----------------------------------

## Core组件
Core 组件作为 Spring 的核心组件，他其中包含了很多的关键类，其中一个重要组成部分就是定义了资源的访问方式。这种把所有资源都抽象成一个接口的方式很值得在以后的设计中拿来学习。下面就重要看一下这个部分在 Spring 的作用。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/core.png)

从上图可以看出 Resource 接口封装了各种可能的资源类型，也就是对使用者来说屏蔽了文件类型的不同。对资源的提供者来说，如何把资源包装起来交给其他人用这也是一个问题，我们看到 Resource 接口继承了 InputStreamSource 接口，这个接口中有个 getInputStream 方法，返回的是 InputStream 类。这样所有的资源都被可以通过 InputStream 这个类来获取，所以也屏蔽了资源的提供者。

```
public interface InputStreamSource {  
    //获取资源
    InputStream getInputStream() throws IOException;  
}  
```


另外还有一个问题就是加载资源的问题，也就是资源的加载者要统一，从上图中可以看出这个任务是由 ResourceLoader 接口完成，他屏蔽了所有的资源加载者的差异，只需要实现这个接口就可以加载所有的资源，他的默认实现是 DefaultResourceLoader。

## Core组件和Context组件建立联系

![enter image description here](http://p7lixluhf.bkt.clouddn.com/core2.png)

Context 是把资源的加载、解析和描述工作委托给了 ResourcePatternResolver 类来完成，他相当于一个接头人，他把资源的加载、解析和资源的定义整合在一起便于其他组件使用。

通过 InputStreamSource获取资源，我们的Resource经过ResourceLoader的调和，用ClassLoader加载，最后会被XmlBeanDefinitionReader解析成BeanDefinition。

```
public interface Resource extends InputStreamSource {  
	//返回当前Resource代表的底层资源是否存在，true表示存在
        boolean exists(); 
        
        //返回当前Resource代表的底层资源是否可读 
        boolean isReadable();  
        
        //返回当前Resource代表的底层资源是否已经打开，如果返回true，则只能被读取一次然后关闭以避免资源泄露
        boolean isOpen();   
         
        //如果当前Resource代表的底层资源能由java.util.URL代表，则返回该URL       
        URL getURL() throws IOException;  

	//如果当前Resource代表的底层资源能由java.util.URI代表，则返回该URI
        URI getURI() throws IOException;  

	//如果当前Resource代表的底层资源能由java.io.File代表，则返回该File
        File getFile() throws IOException;  

	//返回当前Resource代表的底层文件资源的长度
        long contentLength() throws IOException;  

	//返回当前Resource代表的底层资源的最后修改时间。
        long lastModified() throws IOException;  

	//用于创建相对于当前Resource代表的底层资源的资源，比如当前Resource代表文件资源“d:/test/”则createRelative（“test.txt”）将返回表文件资源“d:/test/test.txt”Resource资源。
        Resource createRelative(String relativePath) throws IOException;  

	//返回当前Resource代表的底层文件资源的文件路径，比如File资源“file://d:/test.txt”将返回“d:/test.txt”
        String getFilename();  
	
	//返回当前Resource代表的底层资源的描述符
        String getDescription();  
}  
```
Resource接口提供了足够的抽象，足够满足我们日常使用。而且提供了很多内置Resource实现：ByteArrayResource、InputStreamResource 、FileSystemResource 、UrlResource 、ClassPathResource、ServletContextResource、VfsResource等
