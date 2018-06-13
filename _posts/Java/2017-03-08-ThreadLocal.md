---
layout: post
title:   深入ThreadLocal
date:   2017-03-08 19:15:10
categories:  Java
tags:  Java并发
keywords: 
description: 
---

## ThreadLocal的作用和目的

用于实现线程内的数据共享，即对于相同的程序代码，多个模块在同一个线程中运行时要共享一份数据，而在另外线程中运行时又共享另外一份数据。

ThreadLocal为变量在每个线程中都创建了一个实例，所以每个线程可以访问自己内部的副本变量，不同线程之间不会互相干扰。

ThreadLocal的作用是提供线程内的局部变量，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。

## ThreadLocal使用

如下所示，创建一个ThreadLocal变量：
```
private ThreadLocal myThreadLocal = new ThreadLocal();
```

你实例化了一个ThreadLocal对象。每个方法仅需要实例化一次即可，虽然不同的线程执行同一段代码时，访问同一个ThreadLocal变量，但是每个线程只能看到私有的ThreadLocal实例。所以不同的线程在给ThreadLocal对象设置不同的值时，他们也不能看到彼此的修改。

一旦创建了一个ThreadLocal对象，你就可以通过以下方式来存储此对象的值：
```
myThreadLocal.set("A thread local value");
```

也可以直接读取一个ThreadLocal对象的值：
```
String threadLocalValue = (String) myThreadLocal.get();
```
get()方法会返回一个Object对象，而set()方法则依赖一个Object对象参数。

## ThreadLocal实现原理

从线程Thread的角度来看，每个线程内部都会持有一个对ThreadLocalMap实例的引用，ThreadLocalMap实例相当于线程的局部变量空间，存储着线程各自的数据，具体如下： 

![enter image description here](http://p7lixluhf.bkt.clouddn.com/ThreadLocal.png)

### 1.Entry

Entry继承自WeakReference类，是存储线程私有变量的数据结构。ThreadLocal实例作为引用，意味着如果ThreadLocal实例为null，就可以从table中删除对应的Entry。
```
class Entry extends WeakReference<ThreadLocal<?>> {
      Object value;
      Entry(ThreadLocal<?> k, Object v) {
           super(k);
           value = v;
      }
}
```


### 2.ThreadLocalMap

内部使用table数组存储Entry，默认大小INITIAL_CAPACITY(16)，先介绍几个参数：

size：table中元素的数量。

threshold：table大小的2/3，当size >= threshold时，遍历table并删除key为null的元素，如果删除后size >= threshold*3/4时，需要对table进行扩容。

## set和get的实现

### 1.ThreadLocal.set() 实现
```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
``` 

从上面代码中看出来：

* 1.从当前线程Thread中获取ThreadLocalMap实例，
* 2.ThreadLocal实例和value封装成Entry。

接下去看看Entry存入table数组如何实现的：
``` 
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        if (k == key) {
            e.value = value;
            return;
        }
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }

    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
``` 
* 1.通过ThreadLocal的nextHashCode方法生成hash值。
``` 
private static AtomicInteger nextHashCode = new AtomicInteger();
private static int nextHashCode() {    
 return nextHashCode.getAndAdd(HASH_INCREMENT);
}
``` 
从nextHashCode方法可以看出，ThreadLocal每实例化一次，其hash值就原子增加HASH_INCREMENT。
* 2.通过 hash & (len -1) 定位到table的位置i，假设table中i位置的元素为f。
* 3.如果f != null，假设f中的引用为k：
　如果k和当前ThreadLocal实例一致，则修改value值，返回。
　如果k为null，说明这个f已经是stale(陈旧的)的元素。调用replaceStaleEntry方法删除table中所有陈旧的元素（即entry的引用为null）并插入新元素，返回。
　否则通过nextIndex方法找到下一个元素f，继续进行步骤3。
* 4.如果f == null，则把Entry加入到table的i位置中。
* 5.通过cleanSomeSlots删除陈旧的元素，如果table中没有元素删除，需判断当前情况下是否要进行扩容。

## 2.ThreadLocal.get() 实现
``` 
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}

private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
``` 
获取当前的线程的threadLocals。

* １.如果threadLocals不为null，则通过ThreadLocalMap.getEntry方法找到对应的entry，如果其引用和当前key一致，则直接返回，否则在table剩下的元素中继续匹配。
* ２.如果threadLocals为null，则通过setInitialValue方法初始化，并返回。
``` 
private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
	Entry[] tab = table;
	int len = tab.length;
	while (e != null) {
		ThreadLocal<?> k = e.get();
		if (k == key)
		        return e;
		if (k == null)
		         expungeStaleEntry(i);
	        else
		         i = nextIndex(i, len);
         e = tab[i];
      }
      return null;
}
``` 
