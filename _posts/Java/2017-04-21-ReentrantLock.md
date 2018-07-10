---
layout: post
title:  ReentrantLock
date:   2017-04-21 19:15:10
categories:  Java
tags: Java并发
keywords: 
description: 
---

## 显式锁 Lock接口
而Java SE 5之后，并发包中新增了Lock接口（以及相关实现类）用来实现锁功能，它提供了与synchronized关键字类似的同步功能，只是在使用时需要显式地获取和释放锁。虽然它缺少了（通过synchronized块或者方法所提供的）隐式获取释放锁的便捷性，但是却拥有了锁获取与释放的可操作性、可中断的获取锁以及超时获取锁等多种synchronized关键字所不具备的同步特性。

Java 5 提供了Lock与ReadWriteLock两个接口，ReadWriteLock可以实现对共同资源的并发访问 
为Lock接口提供了ReentrantLock（可重入锁）实现类； 
为ReadWriteLock接口提供了ReentrantReadWriteLock实现类；

Lock接口提供的synchronized关键字所不具备的主要特性如下
* 与synchronized不同，获取到Lock的线程能够响应中断,当获取到锁的线程被中断时能够自动释放锁
* Lock超时的获取锁，在一个timeout内，如果没获取到锁，则返回

API

![](http://p7lixluhf.bkt.clouddn.com/ReentrantLock2.png)

使用ReentrantLock的代码格式
```
public class x{
    //定义锁对象
    private final Lock lock = new ReentrantLock();
    public void method(){
        //加锁
        try{
        //需要保证线程安全的代码
        } finally{
            lock.unlock();  //使用finally保证锁的释放
        }
    }
}
```
要将获取锁的过程写在try块中，因为如果在获取锁（自定义锁的实现）时发生了异常，异常抛出的同时，也会导致锁无故释放。

ReentrantLock具有可重入性，也就是说，一个线程可以对已被加锁的ReentrantLock锁再次加锁。ReentrantLock对象会维持一个计数器来追踪lock()方法的嵌套调用，线程每次调用lock()方法加锁后，必须显示的调用unlock()方法解锁。

## ReentrantLock中的公平锁和非公平锁

首先我们先看看ReentrantLock的构造方法， 分为两种，公平锁和非公平锁。
```
//默认非公平锁
public ReentrantLock() {
    sync = new NonfairSync();
}

//fair为true时，采用公平锁策略
public ReentrantLock(boolean fair) {    
    sync = fair ? new FairSync() : new NonfairSync();
}
```

* 公平锁：公平锁遵循先来后到的原则，对于先对锁进行获取的请求一定先被满足 
* 非公平锁：反之，不遵循先来后到的原则。直接抢资源

一般来说，非公平锁的效率比较高，但是公平锁能减少”饥饿“的发生概率。

## ReentrantLock的实现原理
首先看看ReentrantLock的源码
```
public class ReentrantLock implements Lock, java.io.Serializable {
    private final Sync sync;
    abstract static class Sync extends AbstractQueuedSynchronizer {

        //抽象的Lock方法，交给NonfairSync和FairSync去实现 
        abstract void lock();

        /**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

    }
    //默认非公平锁
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    //fair为false时，采用公平锁策略
    public ReentrantLock(boolean fair) {    
        sync = fair ? new FairSync() : new NonfairSync();
    }
    public void lock() { sync.lock(); }
    public void unlock() { sync.release(1);}
    public Condition newCondition() {    return sync.newCondition();}
    ...
}
```
如将ReentrantLock的lock函数转化为对Sync的lock函数的调用，而具体会根据采用的策略（如公平策略或者非公平策略）的不同而调用到Sync的不同子类。

![](http://p7lixluhf.bkt.clouddn.com/ReentrantLock1.png)

### 公平锁Lock方法源码
我们先看看Lock方法的源码
```
public void lock() { sync.lock(); }
```
```
public void lock() { acquire(1); }
```
```
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
通过分析ReentrantLock的源码，可知对其操作都转化为对Sync对象的操作，由于Sync继承了AQS，所以基本上都可以转化为对AQS的操作。

上述代码主要完成了同步状态获取、节点构造、加入同步队列以及在同步队列中自旋等待的相关工作。

其主要逻辑： 
首先调用自定义同步器实现的tryAcquire(int arg)方法，该方法保证线程安全的获取同步状态，如果同步状态获取失败，则构造同步节点（独占式Node.EXCLUSIVE，同一时刻只能有一个线程成功获取同步状态）并通过addWaiter(Node node)方法将该节点加入到同步队列的尾部，最后调用acquireQueued(Node node,int arg)方法，使得该节点以“死循环”的方式获取同步状态。

其函数逻辑： 
* 1.tryAcquire()尝试直接去获取资源，如果成功则直接返回； 
* 2.addWaiter()将该线程加入等待队列的尾部，并标记为独占模式； 
* 3.acquireQueued()使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。 
* 4.如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。

![](http://p7lixluhf.bkt.clouddn.com/ReentrantLock.png)

整个acquire()方法的调用流程，我们用一个流程图整理一下

![](http://p7lixluhf.bkt.clouddn.com/ReentrantLock3.png)

### 重入的实现
重入的实现
重进入是指任意线程在获取到锁之后能够再次获取该锁而不会被锁所阻塞。 
1）线程再次获取锁。锁需要去识别获取锁的线程是否为当前占据锁的线程，如果是，则再 
次成功获取。 
2）锁的最终释放。线程重复n次获取了锁，随后在第n次释放该锁后，其他线程能够获取到 
该锁。锁的最终释放要求锁对于获取进行计数自增，计数表示当前锁被重复获取的次数，而锁 
被释放时，计数自减，当计数等于0时表示锁已经成功释放。

这里我们以非公平锁为例。
```
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();                                  //AQS提供的方法，获取当前同步状态,c=0代表锁被释放
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);            //设置成当前占有锁的线程
            return true;
        }
    } else if (current == getExclusiveOwnerThread()) {   //判断时候为当前占据锁的线程，则C继续增加1
        int nextc = c + acquires;
        if (nextc < 0) throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
通过判断当前线程是否为获取锁的线程来决定获取操作是否成功，如果是获取锁的线程再次请求，则将同步状态值进行增加并返回true，表示获取同步状态成功。

成功获取锁的线程再次获取锁，只是增加了同步状态值，这也就要求ReentrantLock在释放同步状态时减少同步状态值。
```
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;            
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```
如果该锁被获取了n次，那么前(n-1)次tryRelease(int releases)方法必须返回false，而只有同步状态完全释放了，才能返回true。可以看到，该方法将同步状态是否为0作为最终释放的条件，当同步状态为0时，将占有线程设置为null，并返回true，表示释放成功。

### 非公平锁实现
在非公平锁中，每当线程执行lock方法时，都尝试利用CAS把state从0设置为1。

那么Doug lea是如何实现锁的非公平性呢？ 
我们假设这样一个场景：

1.持有锁的线程A正在running，队列中有线程BCDEF被挂起并等待被唤醒；

2.在某一个时间点，线程A执行unlock，唤醒线程B； 
同时线程G执行lock，这个时候会发生什么？线程B和G拥有相同的优先级，这里讲的优先级是指获取锁的优先级，同时执行CAS指令竞争锁。如果恰好线程G成功了，线程B就得重新挂起等待被唤醒。

NonfairSync类继承了Sync类，表示采用非公平策略获取锁，其实现了Sync类中抽象的lock方法，源码如下。
```
    // 非公平锁
    static final class NonfairSync extends Sync {
        // 版本号
        private static final long serialVersionUID = 7316153563782823691L;

        // 获得锁
        final void lock() {
            if (compareAndSetState(0, 1)) // 比较并设置状态成功，状态0表示锁没有被占用
                // 把当前线程设置独占了锁
                setExclusiveOwnerThread(Thread.currentThread());
            else // 锁已经被占用，或者set失败
                acquire(1);   //该线程加入AQS的尾节点
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```
### 公平锁的实现
公平锁实现
在公平锁中，每当线程执行lock方法时，如果同步器的队列中有线程在等待，则直接加入到队列中。 
场景分析：

1.持有锁的线程A正在running，对列中有线程BCDEF被挂起并等待被唤醒；

2.线程G执行lock，队列中有线程BCDEF在等待，线程G直接加入到队列的对尾。 
所以每个线程获取锁的过程是公平的，等待时间最长的会最先被唤醒获取锁。

FairSync类也继承了Sync类，表示采用公平策略获取锁，其实现了Sync类中的抽象lock方法，源码如下。
```
    // 公平锁
    static final class FairSync extends Sync {
        // 版本序列化
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            // 以独占模式获取对象，忽略中断
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        // 尝试公平获取锁
        protected final boolean tryAcquire(int acquires) {
            // 获取当前线程
            final Thread current = Thread.currentThread();
            // 获取状态
            int c = getState();
            if (c == 0) { // 状态为0
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {   // 不存在已经等待更久的线程并且比较并且设置状态成功
                    setExclusiveOwnerThread(current);　  // 设置当前线程独占锁
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) { // 状态不为0，即资源已经被线程占据，而当前线程为占有锁的线程
                int nextc = c + acquires;
                if (nextc < 0) // 超过了int的表示范围
                    throw new Error("Maximum lock count exceeded");
                // 设置状态
                setState(nextc);                             //State加一
                return true;
            }
            return false;
        }
    }
```
该方法与nonfairTryAcquire(int acquires)比较，唯一不同的位置为判断条件多了hasQueuedPredecessors()方法，即加入了同步队列中当前节点是否有前驱节点的判断，如果该方法返回true，则表示有线程比当前线程更早地请求获取锁，因此需要等待前驱线程获取并释放锁之后才能继续获取锁。

## 总结

* 1.ReentrantLock提供了内置锁类似的功能和内存语义。
* 2.此外，ReetrantLock还提供了其它功能，包括定时的锁等待、可中断的锁等待、公平性、以及实现非块结构的加锁、Condition，对线程的等待和唤醒等操作更加灵活，一个ReentrantLock可以有多个Condition实例，所以更有扩展性，不过ReetrantLock需要显示的获取锁，并在finally中释放锁，否则后果很严重。
* 3.ReentrantLock在性能上似乎优于Synchronized，其中在jdk1.6中略有胜出，在1.5中是远远胜出。那么为什么不放弃内置锁，并在新代码中都使用ReentrantLock？
* 4.在java1.5中， 内置锁与ReentrantLock相比有例外一个优点：在线程转储中能给出在哪些调用帧中获得了哪些锁，并能够检测和识别发生死锁的线程。ReentrantLock的非块状特性任然意味着，获取锁的操作不能与特定的栈帧关联起来，而内置锁却可以。
