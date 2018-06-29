---
layout: post
title:  深入ConcurrentHashMap JDK 1.7
date:   2017-05-05 19:15:10
categories:  Java
tags:  Java容器
keywords: 
description:         
---

## 线程不安全的HashMap 
在多线程环境下，使用HashMap进行put操作会引起死循环，导致CPU利用率接近100%，所 以在并发情况下不能使用HashMap。例如，执行以下代码会引起死循环。

HashMap在并发执行put操作时会引起死循环，是因为多线程会导致HashMap的Entry链表 形成环形数据结构，一旦形成环形数据结构，Entry的next节点永远不为空，就会产生死循环获取Entry.

## 效率低下的HashTable 
HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable 的效率非常低下。因为当一个线程访问HashTable的同步方法，其他线程也访问HashTable的同 步方法时，会进入阻塞或轮询状态。如线程1使用put进行元素添加，线程2不但不能使用put方法添加元素，也不能使用get方法来获取元素，所以竞争越激烈效率越低。

## 锁分离的ConcurrentHashMap
通过分析Hashtable就知道，synchronized是针对整张Hash表的，即每次锁住整张表让线程独占，ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术。它使用了多个锁来控制对hash表的不同部分进行的修改。ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的hashtable，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。 
ConcurrentHashMap和Hashtable主要区别就是围绕着锁的粒度以及如何锁,可以简单理解成把一个大的HashTable分解成多个，形成了锁分离。

### ConcurrentHashMap 结构

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁（ReentrantLock），在ConcurrentHashMap里扮演锁的角色；HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组。Segment的结构和HashMap类似，是一种数组和链表结构。一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元 素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时， 必须首先获得与它对应的Segment锁。

![enter image description here](http://p7lixluhf.bkt.clouddn.com/concurrenthashmap2.jpg)

**单个Segment结构**

Segment是什么呢？Segment本身就相当于一个HashMap对象。 
同HashMap一样，Segment包含一个HashEntry数组，像这样的Segment对象，在ConcurrentHashMap集合中有多少个呢？有2的N次方个，共同保存在一个名为segments的数组当中。
![enter image description here](http://p7lixluhf.bkt.clouddn.com/concurrhashmap1.jpg)


## 并发中的ConcurrentHashMap

ConcurrentHashMap当中每个Segment各自持有一把锁。在保证线程安全的同时降低了锁的粒度，让并发操作效率更高。

Case1:可以并发对不同的Segment进行写入

![enter image description here](http://p7lixluhf.bkt.clouddn.com/CHM1.jpg)

Case2:可以并发对同一Segment执行读 和 写操作

![enter image description here](http://p7lixluhf.bkt.clouddn.com/CHM2.jpg)

Case: 不能对同意Segment进行写操作

![enter image description here](http://p7lixluhf.bkt.clouddn.com/CHM3.jpg)



## Get操作
Segment的get操作实现非常简单和高效。 
1.为输入的Key做Hash运算，得到hash值。 
2.通过hash值，定位到对应的Segment对象 
3.再次通过hash值，定位到Segment当中HashEntry数组的具体位置。
```
public V get(Object key) {
    int hash = hash(key.hashCode());
    return segmentFor(hash).get(key, hash);
}
```
get操作的高效之处在于整个get过程不需要加锁，除非读到的值是空才会加锁重读。我们知道HashTable容器的get方法是需要加锁的，那么ConcurrentHashMap的get操作是如何做到不加锁的呢？

原因是它的get方法里将要使用的共享变量都定义成volatile类型，如用于统计当前 Segement大小的count字段和用于存储值的HashEntry的value。定义成volatile的变量，能够在线程之间保持可见性，能够被多线程同时读，并且保证不会读到过期的值，但是只能被单线程写 （有一种情况可以被多线程写，就是写入的值不依赖于原值），在get操作里只需要读不需要写,共享变量count和value，所以可以不用加锁。之所以不会读到过期的值，是因为根据Java内存模 型的happen before原则，对volatile字段的写入操作先于读操作，即使两个线程同时修改和获取 volatile变量，get操作也能拿到最新的值，这是用volatile替换锁的经典应用场景。

## Put操作
1.为输入的Key做Hash运算，得到hash值。 
2.通过hash值，定位到对应的Segment对象 
3.获取可重入锁 
4.再次通过hash值，定位到Segment当中数组的具体位置。 
5.插入或覆盖HashEntry对象。 
6.释放锁。

插入操作需要经历两个 步骤，第一步判断是否需要对Segment里的HashEntry数组进行扩容，第二步定位添加元素的位置，然后将其放在HashEntry数组里。

（1）是否需要扩容 
在插入元素前会先判断Segment里的HashEntry数组是否超过容量（threshold），如果超过阈 值，则对数组进行扩容。

（2）如何扩容 
在扩容的时候，首先会创建一个容量是原来容量两倍的数组，然后将原数组里的元素进 行再散列后插入到新的数组里。为了高效，ConcurrentHashMap不会对整个容器进行扩容，而只 对某个segment进行扩容。

场景：线程A和线程B同时执行相同Segment对象的put方法

1、线程A执行tryLock()方法成功获取锁，则把HashEntry对象插入到相应的位置； 
2、线程B获取锁失败，则执行scanAndLockForPut()方法，在scanAndLockForPut方法中，会通过重复执行tryLock()方法尝试获取锁，在多处理器环境下，重复次数为64，单处理器重复次数为1，当执行tryLock()方法的次数超过上限时，则执行lock()方法挂起线程B； 
3、当线程A执行完插入操作时，会通过unlock()方法释放锁，接着唤醒线程B继续执行；

## Size操作
如果要统计整个ConcurrentHashMap里元素的大小，就必须统计所有Segment里元素的大小后求和。Segment里的全局变量count是一个volatile变量，那么在多线程场景下，是不是直接把所有Segment的count相加就可以得到整个ConcurrentHashMap大小了呢？不是的，虽然相加时可以获取每个Segment的count的最新值，但是可能累加前使用的count发生了变化，那么统计结 果就不准了，比如我统计玩Segment1之后Segment1马上写入一个值。所以，最安全的做法是在统计size的时候把所有Segment的put、remove和clean方法全部锁住，但是这种做法显然非常低效。 
　 
因为在累加count操作过程中，之前累加过的count发生变化的几率非常小，所以ConcurrentHashMap的做法是先尝试2次通过不锁住Segment的方式来统计各个Segment大小，如果统计的过程中，容器的count发生了变化，则再采用加锁的方式来统计所有Segment的大小。 
　 
那么ConcurrentHashMap是如何判断在统计的时候容器是否发生了变化呢？使用modCount变量，在put、remove和clean方法里操作元素前都会将变量modCount进行加1，那么在统计size前后比较modCount是否发生变化，从而得知容器的大小是否发生变化。 
　 
代码实现
```
try {
    for (;;) {
        if (retries++ == RETRIES_BEFORE_LOCK) {
            for (int j = 0; j < segments.length; ++j)
                ensureSegment(j).lock(); // force creation
        }
        sum = 0L;
        size = 0;
        overflow = false;
        for (int j = 0; j < segments.length; ++j) {
            Segment<K,V> seg = segmentAt(segments, j);
            if (seg != null) {
                sum += seg.modCount;
                int c = seg.count;
                if (c < 0 || (size += c) < 0)
                    overflow = true;
            }
        }
        if (sum == last)
            break;
        last = sum;
    }
} finally {
    if (retries > RETRIES_BEFORE_LOCK) {
        for (int j = 0; j < segments.length; ++j)
            segmentAt(segments, j).unlock();
    }
}
```
为什么这样设计呢？这种思想和乐观锁悲观锁的思想如出一辙。

为了尽量不锁住所有Segment，首先乐观地假设Size过程中不会有修改。当尝试一定次数，才无奈转为悲观锁，锁住所有Segment保证强一致性。

tips:
以上是JDK 1.7版本的ConcurrentHashMap，在JDK 1.8版本中的ConcurrentHashMap做出了很多优化和改变。