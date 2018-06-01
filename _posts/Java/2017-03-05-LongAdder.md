---
layout: post
title:    JDK8新增 LongAdder
date:   2017-03-05 19:15:10
categories:  Java
tags:  Java基础
keywords: 
description: 
---

LongAdder是JDK8新增的用于并发环境的计数器，目的是为了在高并发情况下，代替AtomicLong/AtomicInteger，成为一个用于高并发情况下的高效的通用计数器。

高并发下计数，一般最先想到的应该是AtomicLong/AtomicInteger，AtmoicXXX使用硬件级别的指令 CAS 来更新计数器的值，这样可以避免加锁，机器直接支持的指令，效率也很高。但是AtomicXXX中的 CAS 操作在出现线程竞争时，失败的线程会白白地循环一次，在并发很大的情况下，因为每次CAS都只有一个线程能成功，竞争失败的线程会非常多。失败次数越多，循环次数就越多，很多线程的CAS操作越来越接近 自旋锁（spin lock）。计数操作本来是一个很简单的操作，实际需要耗费的cpu时间应该是越少越好，AtomicXXX在高并发计数时，大量的cpu时间都浪费会在 自旋 上了，这很浪费，也降低了实际的计数效率。

LongAdder比在高并发时比AtomicLong更高效，这么说有什么依据呢？LongAdder是根据ConcurrentHashMap这类为并发设计的类的基本原理——锁分段，来实现的，它里面维护一组按需分配的计数单元，并发计数时，不同的线程可以在不同的计数单元上进行计数，这样减少了线程竞争，提高了并发效率。本质上是用空间换时间的思想，不过在实际高并发情况中消耗的空间可以忽略不计。
现在，在处理高并发计数时，应该优先使用LongAdder，而不是继续使用AtomicLong。当然，线程竞争很低的情况下进行计数，使用Atomic还是更简单更直接，并且效率稍微高一些。
其他情况，比如序号生成，这种情况下需要准确的数值，全局唯一的AtomicLong才是正确的选择，此时不应该使用LongAdder。
```
public class LongAdderTest {
    private static final Logger logger = LoggerFactory.getLogger(LongAdderTest.class);
    //请求总数
    private static int clientTotal = 5000;
    //同时并发执行数
    private static int threadTotal = 200;
    //使用LongAdder可以更高效的达到预期效果
    private static LongAdder count = new LongAdder();

    public static void main (String[] args)throws Exception{
        ExecutorService executorService = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for(int i = 0; i< clientTotal; i++){
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                }catch (Exception e){
                    logger.error("exception",e);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        logger.info("count:{}",count);
    }
    private static void add(){
        count.increment();
    }
}
```

## 优缺点

* LongAdder在AtomicLong的基础上将单点的更新压力分散到各个节点，在低并发的时候通过对base的直接更新可以很好的保障和AtomicLong的性能基本保持一致，而在高并发的时候通过分散提高了性能。

* 缺点是LongAdder在统计的时候如果有并发更新，可能导致统计的数据有误差。