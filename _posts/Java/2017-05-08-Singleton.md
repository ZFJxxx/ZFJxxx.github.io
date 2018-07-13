---
layout: post
title:  单例模式
date:   2017-05-08 19:15:10
categories:  Java
tags:  DesignPattern
keywords: 
description:         
---

## 1.饿汉式
```
public class Singleton{
	private static Singleton instance = new Singleton();
	private Singleton(){}
	public static Singleton getInstance(){
		return instance;
	}
}
```

##  2.懒汉式(Lazy Loading)
```
public class Singleton {
	private static Singleton instance = null;
	private Singleton(){}
	public static Singleton getInstance(){
		if(instance == null){
			instance = new Singleton();
		}
		return instance;
	} 
}
```

饿汉式和懒汉式很容易实现，但是并不是线程安全的，在懒汉式中，如果A、B两条线程同一时间通过判断instance == null，这样就建出两个实例了，显然不符合要求。

## 3.懒汉式 - 线程安全
```
public class Singleton {
	private static Singleton instance = null;
	private Singleton(){}
	public static synchronized Singleton getInstance(){
		if (instance == null){
			instance = new Singleton();
		}
		return instance;	
	}
}
```
这种方式具备很好的 lazy loading，能够在多线程中很好的工作，但是，效率很低，99% 情况下不需要同步。
* 优点：第一次调用才初始化，避免内存浪费。
* 缺点：必须加锁 synchronized 才能保证单例，但加锁会影响效率。

getInstance() 的性能对应用程序不是很关键的时候使用（该方法使用不太频繁）。


## 4.懒汉式 - 双重锁检测
```
public class Singleton{
	private volatile static Singleton instance = null;
	private Singleton(){}
	public static Singleton getInstance(){
		if (instance == null){
			synchronized(Singleton.class){
	            		if (instance == null)
                    			instance = new Singleton();
			}
		}
		return instance;
	}
}
```
这种方式采用双锁机制，安全且在多线程情况下能保持高性能。getInstance() 的性能对应用程序很关键。

**为什么要加volatile呢？**
如果不加volatile，这种情况表面看似没什么问题，要么Instance还没被线程A构建，线程B执行 if（instance== null）的时候得到true；要么Instance已经被线程A构建完成，线程B执行 if（instance == null）的时候得到false。


真的如此吗？答案是否定的。这里涉及到了JVM编译器的指令重排。


指令重排是什么意思呢？比如java中简单的一句 instance = new Singleton（），会被编译器编译成如下JVM指令：

```
memory =allocate();    //1：分配对象的内存空间 
ctorInstance(memory);  //2：初始化对象 
instance =memory;     //3：设置instance指向刚分配的内存地址 
```

但是这些指令顺序并非一成不变，有可能会经过JVM和CPU的优化，指令重排成下面的顺序：

```
memory =allocate();    //1：分配对象的内存空间 
instance =memory;     //3：设置instance指向刚分配的内存地址 
ctorInstance(memory);  //2：初始化对象 
```

当线程A执行完1,3,时，instance对象还未完成初始化，但已经不再指向null。此时如果线程B抢占到CPU资源，执行  if（instance == null）的结果会是false，**从而返回一个没有初始化完成的instance对象**。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/singleton.jpg)

如此在线程B看来，instance对象的引用要么指向null，要么指向一个初始化完毕的Instance，而不会出现某个中间态，保证了安全。

## 5. 静态内部类

```
public class Singleton {  
    private static class SingletonHolder {  
        private static final Singleton INSTANCE = new Singleton();  
    }  
    
    private Singleton (){}  
    public static final Singleton getInstance() {  
        return SingletonHolder.INSTANCE;  
    }  
}   
```

INSTANCE对象初始化的时机并不是在单例类Singleton被加载的时候，而是在调用getInstance方法，使得静态内部类LazyHolder被加载的时候。因此这种实现方式是利用classloader的加载机制来实现懒加载，并保证构建单例的线程安全。

这种方式能达到双检锁方式一样的功效，但实现更简单。

## 6.枚举

当然以上方法不是绝对安全的,还是可以利用反射打破单例：

```
//获得构造器
Constructor con = Singleton.class.getDeclaredConstructor();
//设置为可访问
con.setAccessible(true);
//构造两个不同的对象
Singleton singleton1 = (Singleton)con.newInstance();
Singleton singleton2 = (Singleton)con.newInstance();
//验证是否是不同对象
System.out.println(singleton1.equals(singleton2));
```

代码可以简单归纳为三个步骤：

* 1.获得单例类的构造器。
* 2.把构造器设置为可访问。
* 3.使用newInstance方法构造对象。

最后为了确认这两个对象是否真的是不同的对象，我们使用equals方法进行比较。毫无疑问，比较结果是false。

用枚举实现单例模式：

```
public enum SingletonEnum {
	INSTANCE;
}
```
这种方法线程安全，只不过不是懒加载。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/singleton1.jpg)

**经验之谈**：一般情况下，不建议使用第 2 种和第 3 种懒汉方式，建议使用第 1 种饿汉方式。只有在要明确实现 lazy loading 效果时，才会使用第 5 种静态内部类方式。如果涉及到反序列化创建对象时，可以尝试使用第 6 种枚举方式。如果有其他特殊的需求，可以考虑使用第 4 种双检锁方式。

