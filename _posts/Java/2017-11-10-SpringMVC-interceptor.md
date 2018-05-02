---
layout: post
title: SpringMVC 拦截器Interceptor
date:   2017-11-10 19:15:10
categories:  Java
tags:  SpringMVC
keywords: SpringMVC
description: 
---
----------------------------------

SpringMVC 中的Interceptor 拦截器也是相当重要和相当有用的，它的主要作用是拦截用户的请求并进行相应的处理。比如通过它来进行权限验证，或者是来判断用户是否登陆。

## 常见应用场景
1、日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算PV（Page View）等。

2、权限检查：如登录检测，进入处理器检测检测是否登录，如果没有直接返回到登录页面；

3、性能监控：有时候系统在某段时间莫名其妙的慢，可以通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间（如果有反向代理，如apache可以自动记录）；

4、通用行为：读取cookie得到用户信息并将用户对象放入请求，从而方便后续流程使用，还有如提取Locale、Theme信息

## 拦截器具体实现


首先我们要实现HandlerInterceptor这个接口，然后去重写它里面preHandle,postHandle和afterCompletion这三个方法。

我们分别来看这三个方法

```
public class PassportInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}
```

![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/%E6%8B%A6%E6%88%AA%E5%99%A82.jpg)

**1.preHandle：**预处理回调方法，该方法将在请求处理之前进行调用，实现处理器的预处理（如登录检查），返回一个boolean类型的值。
　当返回值为true时就会继续调用下一个Interceptor 的preHandle 方法，如果已经是最后一个Interceptor 的时候就会是调用当前请求的Controller方法。
　当返回值是false表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器。
　
SpringMVC 中的Interceptor 是链式的调用的，在一个应用中或者说是在一个请求中可以同时存在多个Interceptor 。每个Interceptor 的调用会依据它的声明顺序依次执行，而且最先执行的都是Interceptor 中的preHandle 方法，所以可以在这个方法中进行一些前置初始化操作或者是对当前请求的一个预处理，也可以在这个方法中进行一些判断来决定请求是否要继续进行下去。

**2.postHandle：**后处理回调方法，（该方法要preHandle返回为true才会执行）
该方法将在请求处理之后，也就是在Controller 方法调用之后被调用，但是会在视图返回被渲染之前被调用，所以可以在这个方法里面通过改变数据模型Model 来改变数据的展示。Model 就是Controller 处理之后返回的Model 对象，我们可以通过改变它的属性来改变返回的Model 模型。

**3.afterCompletion：**整个请求处理完毕回调方法，（在视图渲染完毕时回调）（该方法要preHandle返回为true才会执行）
如性能监控中我们可以在此记录结束时间并输出消耗时间，还可以进行一些资源清理，类似于try-catch-finally中的finally。

## 运行流程图
我们假设有两个拦截器1和2.他们的执行过程如下。

![这里写图片描述](http://p7lixluhf.bkt.clouddn.com/%E6%8B%A6%E6%88%AA%E5%99%A81.jpg)

preHandle1返回true就执行preHandle2，preHandle2返回true就可以执行controller了.
如果任意一个preHandle返回false，立刻中断后面都不执行。


##  拦截器实现实例

我们就以用户登录为例子，用拦截器来判断用户访问页面时是否登陆过。
登录过，就找到其ticket来获取用户的信息。

1.首先在使用Interceptor之前，先要新建一个类webConfiguration，继承WebMvcConfigurerAdapter。

然后重写其中的addInterceptors方法，将我们新建的PassportInterceptor加载进去。

```
@Component
public class webConfiguration extends WebMvcConfigurerAdapter {

    @Autowired
    PassportInterceptor passportInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(passportInterceptor);
        super.addInterceptors(registry);
    }
}
```
2.配置好之后，我们就可以来写PassportInterceptor

```
//判断访问的用户是谁
@Component
public class PassportInterceptor implements HandlerInterceptor {
    @Autowired
    private LoginTicketDAO loginTicketDAO;

    @Autowired
    private UserDAO userDAO;

    @Autowired
    private HostHolder hostHolder;
    @Override
    //访问前通过ticket判断访问者的userId
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        String ticket = null;
        if (httpServletRequest.getCookies() != null) {
            for (Cookie cookie : httpServletRequest.getCookies()) {  //获取请求中名为ticket的cookie
                if (cookie.getName().equals("ticket")) {
                    ticket = cookie.getValue();
                    break;
                }
            }
        }

        //如果下发的ticket不为空，没过期，而且有效，就返回true
        if (ticket != null) {
            LoginTicket loginTicket = loginTicketDAO.selectLoginTicketByTicket(ticket);
            if (loginTicket == null || loginTicket.getExpired().before(new Date()) || loginTicket.getStatus() != 0) {
                return true;
            }
            User user = userDAO.selectById(loginTicket.getUserId());
            hostHolder.setUser(user);
        }
        return true;
    }

    @Override
    //将hostHolder中的user放进model中
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        if(modelAndView != null){
            modelAndView.addObject("user",hostHolder.getUser());
        }
    }

    @Override
    //结束访问后需要将用户清除
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        hostHolder.clear();
    }
}
```

首先，我们看preHandle
preHandle也就是在执行相应的controller之前，我们判断当前请求是否包含名为ticket的
cookie，如果有，就继续判断cookie是否过期，是否有效。 都通过之后就可以返回true，并且将cookie对应的用户信息封装到hostHolder中，也就是当前登陆者的信息。

然后我们看postHandle
postHandle是在controller执行之后，但是返回modelAndView之前执行，我们可以对返回的model进行修改，这里我们将hostHolder添加进modelAndView中，这样我们就可以在页面中调用当前登录者的信息，并进行页面渲染了

最后看afterComlption
afterComlption是在modelAndView渲染之后执行，一般干收尾清除的工作，这里我们将渲染之后的hostHolder信息清除掉，防止信息越来越多。

这样，我们就可以在访问每个页面的时候，都进行拦截器的一系列操作了。

那么如果我不想每一个页面都加拦截器，怎么办呢？

只要在后面加addPathPatterns方法就行了，这样只有 带/user/* 的页面会被拦截器拦截
```
@Component
public class webConfiguration extends WebMvcConfigurerAdapter {
    @Autowired
    PassportInterceptor passportInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(passportInterceptor).addPathPatterns("/user/*");
        super.addInterceptors(registry);
    }
}
```


----------
