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

注意：Entry中没有next字段，所以就不存在链表的情况了。这点和HashMap不同


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

每个ThreadLocal对象都有一个hash值threadLocalHashCode，每初始化一个ThreadLocal对象，hash值就增加一个固定的大小0x61c88647。

在插入过程中，根据ThreadLocal对象的hash值，定位到table中的位置i，过程如下：
* 1.如果当前位置是空的，那么正好，就初始化一个Entry对象放在位置i上；
* 2.不巧，位置i已经有Entry对象了，如果这个Entry对象的key正好是即将设置的key，那么重新设置Entry中的value；
* 3.很不巧，位置i的Entry对象，和即将设置的key没关系，那么只能找下一个空位置；

这样的话，在get的时候，也会根据ThreadLocal对象的hash值，定位到table中的位置，然后判断该位置Entry对象中的key是否和get的key一致，如果不一致，就判断下一个位置

可以发现，set和get如果冲突严重的话，效率很低，因为ThreadLoalMap是Thread的一个属性，所以即使在自己的代码中控制了设置的元素个数，但还是不能控制其它代码的行为。


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


## 内存泄露

ThreadLocal可能导致内存泄漏，先看看Entry的实现：
``` 
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
``` 

通过之前的分析已经知道，当使用ThreadLocal保存一个value时，会在ThreadLocalMap中的数组插入一个Entry对象，按理说key-value都应该以强引用保存在Entry对象中，但在ThreadLocalMap的实现中，key被保存到了WeakReference对象中。

这就导致了一个问题，ThreadLocal在没有外部强引用时，发生GC时会被回收，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

### 如何避免内存泄露
既然已经发现有内存泄露的隐患，自然有应对的策略，在调用ThreadLocal的get()、set()可能会清除ThreadLocalMap中key为null的Entry对象，这样对应的value就没有GC Roots可达了，下次GC的时候就可以被回收，当然如果调用remove方法，肯定会删除对应的Entry对象。

如果使用ThreadLocal的set方法之后，没有显示的调用remove方法，就有可能发生内存泄露，所以养成良好的编程习惯十分重要，使用完ThreadLocal之后，记得调用remove方法。
``` 
ThreadLocal<String> localName = new ThreadLocal();
try {
    localName.set("Freddie");
    // 其它业务逻辑
} finally {
    localName.remove();
}
``` 
