---
layout: post
title:    Cookie和Session
date:   2017-07-10 19:15:10
categories:  基础
tags:  NetWork
keywords: CookieSession
description: 
---


1.由于HTTP协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户，这个机制就是Session.典型的场景比如购物车，当你点击下单按钮时，由于HTTP协议无状态，所以并不知道是哪个用户操作的，所以服务端要为特定的用户创建了特定的Session，用用于标识这个用户，并且跟踪用户，这样才知道购物车里面有几本书。这个Session是保存在服务端的，有一个唯一标识。在服务端保存Session的方法很多，内存、数据库、文件都有。集群的时候也要考虑Session的转移，在大型的网站，一般会有专门的Session服务器集群，用来保存用户会话，这个时候 Session 信息都是放在内存的，使用一些缓存服务比如redis之类的来放 Session。

2.思考一下服务端如何识别特定的客户？这个时候Cookie就登场了。每次HTTP请求的时候，客户端都会发送相应的Cookie信息到服务端。实际上大多数的应用都是用 Cookie 来实现Session跟踪的，第一次创建Session的时候，服务端会在HTTP协议中告诉客户端，需要在 Cookie 里面记录一个Session ID，以后每次请求把这个会话ID发送到服务器，我就知道你是谁了。有人问，如果客户端的浏览器禁用了 Cookie 怎么办？一般这种情况下，会使用一种叫做URL重写的技术来进行会话跟踪，即每次HTTP交互，URL后面都会被附加上一个诸如 sid=xxxxx 这样的参数，服务端据此来识别用户。

3.Cookie其实还可以用在一些方便用户的场景下，设想你某次登陆过一个网站，下次登录的时候不想再次输入账号了，怎么办？这个信息可以写到Cookie里面，访问网站的时候，网站页面的脚本可以读取这个信息，就自动帮你把用户名给填了，能够方便一下用户。

所以，总结一下：
Session是在服务端保存的一个数据结构，用来跟踪用户的状态，这个数据可以保存在集群、数据库、文件中；
Cookie是客户端保存用户信息的一种机制，用来记录用户的一些信息，也是实现Session的一种方式。

## Cookie
Cookie，指某些网站为了辨别用户身份、进行 session 跟踪而储存在用户本地终端上的数据（通常经过加密）。
Cookie 一般有两个作用。

第一个作用是识别用户身份。

比如用户 A 用浏览器访问了 http://a.com，那么 http://a.com 的服务器就会立刻给 A 返回一段数据「ticket1」（这就是 Cookie）。当 A 再次访问 http://a.com 的其他页面时，就会附带上「ticket1」这段数据。

同理，用户 B 用浏览器访问 http://a.com 时，http://a.com 发现 B 没有附带 uid 数据，就给 B 分配了一个新的 ticket，为2，然后返回给 B 一段数据「ticket2」。B 之后访问 http://a.com 的时候，就会一直带上「ticket2」这段数据。

将ticket和user_id绑定在一起，就可以鉴别用户了。
借此，http://a.com 的服务器就能区分 A 和 B 两个用户了。

第二个作用是记录历史。

假设 http://a.com 是一个购物网站，当 A 在上面将商品 A1 、A2 加入购物车时，JS 可以改写 Cookie，改为「uid=1; cart=A1,A2」，表示购物车里有 A1 和 A2 两样商品了。
这样一来，当用户关闭网页，过三天再打开网页的时候，依然可以看到 A1、A2 躺在购物车里，因为浏览器并不会无缘无故地删除这个 Cookie。

### Cookie如何工作
（1）服务端创建Cookie下发客户端
```
if (map.containsKey("ticket")) {
	//注册的时候生成一个UUID，存到map中，然后new cookie取出UUID下发
	Cookie cookie = new Cookie("ticket", map.get("ticket"));
        cookie.setPath("/");
        //记住我，就是加长Cookie的过期时限
        if(rememberme){
	        cookie.setMaxAge(3600*24*5);
        }
        //通过HttpServletresponse下发到客户端
        response.addCookie(cookie);               
        if(StringUtils.isNotBlank(next)){
                return "redirect:"+next;
        }
        return "redirect:/index";
}
```
（2）从客户端获取Cookie
浏览器会根据URL路径将符合条件的Cookie放在Request请求头中传给服务端，服务端通过request.getCookies()取得所有Cookie。

### Cookie压缩
2KB大小的Cookie在压缩前后大小差距20%，如果一个网站Cookie在2~3KB之间，一天一亿PV，那么一天就能残生4TB的带宽流量，Cookie压缩还是有必要的。

压缩可用gzip和deflate算法。

## Session

Session代表服务器与浏览器的一次会话过程，这个过程是连续的，也可以时断时续的。在Servlet中，session指的是HttpSession类的对象。当JSP页面没有显式禁止session的时候，在打开浏览器第一次请求该jsp的时候，服务器会自动为其创建一个session，并赋予其一个sessionID，发送给客户端的浏览器。以后客户端接着请求本应用中其他资源的时候，会自动在请求头上添加：Cookie:JSESSIONID= 客户端第一次拿到的session ID （也就是Cookie）之后，服务器就能通过SessionId识别访问者是谁了。所以Session是基于Cookie的。

## Cookie和Session的区别
Cookie和Session都可以追踪客户端的访问记录，但是其工作方式不同
* Cookie将所有数据通过HTTP的头部从客户端传到服务端，又从服务端传到客户端，所有的数据都存在客户端浏览器里，不够安全。
* Session安全性高，Session将数据存在服务器中，只是通过Cookie传递一个SessionID而已，Session更适合存储用户隐私和重要数据。

## 分布式Session

### Cookie的缺点
* Cookie占用传输字节，如果Cookie个数增多，也会占用带宽资源
* 客户端浏览器有Cookie存储限制，超过限制会出现Cookie丢失现象
* Cookie存在安全问题

### Session的缺点
* 不容易在多台服务器之间共享

基于以上，分布式Session就自然而然被提出来了。分布式Session可实现Session配置的统一管理和不同集群之间的Session共享。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/session.jpg)

分布式Session架构图上图，将Session统一存储到分布式缓存服务器中，如Redis。每当应用启动的时候，先去订阅服务器订阅这个应用需要的Seesion和Cookie，这样可以限制这个应用能够使用哪些Session和Cookie,甚至可以控制Session的可读或者可写。

统一通过订阅服务器推送配置可以有效集中管理资源，这里的订阅服务器可以用Zookeeper实现，可以统一管理所有服务器的配置文件。省去了每个应用都来配置Cookie。比如一个应用要用一个新增的Cookie，可以通过一个统一的平台来申请，申请通过才将这个配置增加到订阅服务器。如果是一个所有应用要需要的全局Cookie，将这个Cookie通过订阅服务器统一推送过去就行了，不需要在每个应用中手动增加Cookie。

然后向Session都统一存储到分布式缓存Redis集群中，从分布式集群中存储或获取Session。

### 跨域名同步Session
我们知道Cookie具有域名限制，也就是一个域名下的Cookie不能被另一个域名访问。那么我想登录一个域名后，登录另外一个域名任然保证登录状态有效，如何解决这个问题呢？

![enter image description here](http://p7lixluhf.bkt.clouddn.com/%E5%88%86%E5%B8%83%E5%BC%8FSession.jpg)

SessionID的同步，需要一个跳转应用。