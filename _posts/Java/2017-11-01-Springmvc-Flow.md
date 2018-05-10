---
layout: post
title: SpringMVC原理
date:   2017-11-1 19:15:10
categories:  Java
tags:  SpringMVC
keywords: SpringMVC
description: 
---


## Springmvc执行流程

![enter image description here](http://p7lixluhf.bkt.clouddn.com/SpringMVC-Flow.jpg)

*  首先用户发送请求——>DispatcherServlet，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；

* DispatcherServlet——>HandlerMapping,HandlerMapping中存储这URL和处理类的映射关系，在HandlerMapping中定义了根据一个URL必须返回一个由HandlerExecutionChain代表的处理链，我们可以在处理链中添加任意HandlerAdapters实例来处理URL请求。HandlerMapping将请求映射为HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象。

*  DispatcherServlet——>HandlerAdapter，DispatchServlet根据Handler对象在其handlerAdapters集合中匹配哪个HandlerAdapter实例来执行Handler对象的相应方法，返回一个ModelAndView对象（包含模型数据、逻辑视图名），接下来就通过ModelAndView对象来渲染页面

*  ModelAndView的逻辑视图名——> ViewResolver，ModelAndView持有一个ModelMap对象和View对象，ModelAndView中不包含真正的视图，只返回一个逻辑视图名称的时候，ViewResolver就会根据ViewName创建一个View对象。

*  View——>渲染，View会根据传进来的Model模型数据进行渲染，此处的Model实际是一个Map数据结构，View对象会调用render(Map model,HttpServletRequest request,HttpServletResponse response)方法对视图渲染，把结果返回给浏览器的

## Control设计
SpringMVC中，Control主要由HandlerMapping和HandlerAdapters两个组件提供。

HandlerMapping主要做两个工作，
* 将URL和处理类的映射关系保存到handlerMap集合中。
* 将所有的interceptors对象保存在adapterInterceptors数组中。

HandlerMapping会返回一个HandlerExecutionChain的处理链。

HandlerAdapter初始化的时候会创建一个handlerAdapter对象，将这个对象存在DispatcherServlet的handlerAdapters集合中。当SpringMVC将某个URL对应到这个Handler时，在handlerAdapters集合中查询哪个handlerAdapter对象supports这个Handler，handlerAdapter对象会被返回，并调用这个handlerAdapter接口对应的方法。

## Model设计
ModelAndView对象是连接业务逻辑和View展现层的桥梁，ModelAndView持有一个ModelMap对象和View对象,ModelMap其实也是个Map，在Handler中将模板中需要的对象存到这个Map中，然后传递到View对应的ViewResolvers中，不同的ViewResolvers会对这个Map中的对象有不同的处理方式：比如Velocity将这个Map保存到org.apache.velocity.VelocityContext中，对FreeMarker模板引起来说将ModelMap包装成freemarker.temolate.TemolateHashModel，对JSP来说，将每个ModelMap中的元素分别设置到request.setAttribute(modelName,modelValue)中

## View设计
ViewResolvers根据请求的ViewName创建一个View对象，View对象调用render(Map model,HttpServletRequest request,HttpServletResponse response)方法对视图渲染。