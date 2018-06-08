---
layout: post
title:   Spring AOP 注解 
date:   2017-10-07 19:15:10
categories:  工具
tags: Spring
keywords: 
description: 
---

## AOP hello world

先抛开那些复杂的的名词，我们来看个简单的例子。假如我们在编写一个网站

网站主页的controller
```
@Controller
public class IndexController{
	@RequestMapping(path = {"/","/index"} ,method ={RequestMethod.GET})
	@ResponseBody
    public String index() {
        System.out.println("hello");
        return "hello world";
    }
}
```
AOP切面
```
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Aspect           //用@Aspect注解声明整个类是一个切面
@Component        //一定要将切面注入
public class AOP {

	//用@Before注解声明一个切点,在目标方法调用之前执行
    @Before("execution(* com.myzhihu.controller.IndexController.index(..))")
    public void before(){
        System.out.println("before");
    }

	//用@After注解声明一个切点,在目标方法执行完后执行
    @After("execution(* com.myzhihu.controller.IndexController.index(..))")
    public void After(){
        System.out.println("After");
    }
}
```
那么我们每次访问localhost:8080/index 时 控制台都会输出
```
before
hello world
after
```
这种在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。


## 切点的声明
``
AspectJ注解声明切点有如下通知方法

> @Before 在目标方法调用之前执行
> @Around 将目标方法封装起来
> @After 在目标方法返回或者抛出异常后调用
> @AfterThrowing 在目标方法抛出异常后调用
> @AfterReturning 在目标方法返回后调用


## 切点的匹配

切点就是我们想把切面程序切入到哪里去.

我们看到，相同的切点表达式我们重复了两遍，好在我们可以使用@Pointcut来定义可重用的切点

```
@Aspect
@Component
public class AOP {

    @Pointcut("execution(* com.myzhihu.controller.IndexController.index(..))")
    public void index(){}

    @Before("index()")
    public void before(){
        System.out.println("before");
    }

    @After("index()")
    public void After(){
        System.out.println("After");
    }
}
```

index()方法并不重要，他是个空的，只是供给@Pointcut依附。

**1.execution匹配具体方法**

![enter image description here](http://p7lixluhf.bkt.clouddn.com/AOP.png)
execution也支持通配符,你也可以用execution(* com.myzhihu.controller.IndexController.*(..))来表示IndexController类中的任何方法都做切点。也可以用execution(* com.myzhihu.controller.*Controller.*(..))来表示用controller包下所有以Controller结尾的类中的所有方法做切点。

**2.within来匹配包和类**

```
//&& 代表and  within里代表myzhihu包下所有子包的方法被调用时
@Pointcut("execution(* com.myzhihu.controller.IndexController.index(..)) && within(com.myzhihu.*)")
public void test(){}
```

**3.参数匹配**

```
//匹配任何以index开头并且只有一个int型参数的方法
@Pointcut("execution(* *..index*(int))")
public void argsDemo1(){}

//匹配任何只有一个int型参数的方法
@Pointcut("args(int)")
public void argsDemo2(){}

//匹配任何以index开头并且第一个参数为int型的方法
@Pointcut("execution(* *..index*(int,..))")
public void argsDemo3(){}


//匹配任何第一个参数为int型的方法
@Pointcut("args(int,..)")
public void argsDemo2(){}

```

**4.匹配注解**

```
//匹配方法标注有@RequestMapping的方法（注意，注解只能是方法级别的）
 @Pointcut("@annotation(org.springframework.web.bind.annotation.RequestMapping)")
public void annotationDemo(){}

//匹配标注有@Controller的类底下的方法（注意，注解必须是类级别）
@Pointcut("@within(org.springframework.stereotype.Controller)")
public void withinDemo(){}

//匹配标注有Controller的类底下的方法，注解的RetentionPolicy必须为RUNTIME
@Pointcut("@target(org.springframework.stereotype.Controller)")
public void targetDemo(){}

//匹配传入的参数类标注有@Component的方法
@Pointcut("@args(org.springframework.stereotype.Component)")
public void argsDemo(){}

```

## 小结：

AOP是对面向对象编程的一个强大的补充。通过AspectJ，我们可以把之前分散在应用各处的行为放入可重用的模块中，这有效的减少了代码冗余，让我们的类关注自身的主要功能。
而且可以无侵入的为代码添加功能。 