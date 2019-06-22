---
title: java多线程之ThreadPoolExcutor
date: 2019-06-18 16:53:26
categories: java多线程
tags: java
---

## 一、为什么需要使用线程池
- **线程创建和销毁的开销非常高**：频繁的创建和销毁线程需要消耗时间，会使响应变慢；同时消耗计算资源。
- **资源耗尽**：空闲的线程会占用内存，会给垃圾回收带来压力，线程竞争CPU也会产生性能开销，线程池可合理管理空闲线程
- **稳定性**：在一定范围内，增加线程可以提升系统的处理能力，如果超过这个范围，继续创建线程只会降低执行速度，甚至导致系统OOM,线程池即可以通过程序控制线程数。

## 二、几个常用线程池
首先看下线程池的构造函数：
``` 
/**
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }

```

1. **newFixedThreadPool**:将创建一个**长度固定的线程池**，每当新提交一个任务时就创建一个新线程，直到达到线程池的最大数量，这时线程池的大小不在变化（如果某个线程发生未预期的Exception而结束，那么线程池会补充一个新线程）。
```
 public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>(),
                                      threadFactory);
    }
```
2. **newCachedThreadPool**:将创建一个**可缓存的线程池**，如果有线程空闲，会回收空闲线程；如果任务增加时，可添加新线程，线程池的规模不存在任何界限。
```
 public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
3. **newSingleThreadPool**:将创建一个**单线程的线程池**，如果线程异常结束，将创建另一个线程来替代；阻塞队列为LinkedBlockingQueue，提交的任务可按照顺序执行
```
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```
4. **newScheduledThreadPool**： 创建了一个**固定长度的线程池,而且以延迟或定时的方式来执行任务**,类似于Timer。
```
    public ScheduledThreadPoolExecutor(int corePoolSize,
                                       ThreadFactory threadFactory) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue(), threadFactory);
    }
```

## 三、线程池的六个重要参数：
1. **corePoolSize**：核心线程数
    - 核心线程会一直存活，及时没有任务需要执行
    - 线程数 < corePoolSize时，即使有线程空闲，线程池也会优先创建新线程处理
    - 设置allowCoreThreadTimeout=true（默认false）时，核心线程会超时关闭
2. **queueCapacity**：任务队列容量（阻塞队列）
    当核心线程数达到最大时，新任务会放在队列中排队等待执行
3. **maxPoolSize**：最大线程数
    - 线程数>=corePoolSize，且任务队列已满时。线程池会创建新线程来处理任务
    - 线程数=maxPoolSize，且任务队列已满时，线程池会拒绝处理任务而抛出异常
4. **keepAliveTime**：线程空闲时间
    - 当线程空闲时间达到keepAliveTime时，线程会退出，直到线程数量=corePoolSize
    - 如果allowCoreThreadTimeout=true，则会直到线程数量=0
5. **allowCoreThreadTimeout**：允许核心线程超时
6. **rejectedExecutionHandler**：任务拒绝处理器,两种情况会拒绝处理任务：
    - 当线程数已经达到maxPoolSize，切队列已满，会拒绝新任务
    - 当线程池被调用shutdown()后，会等待线程池里的任务执行完毕，再shutdown。如果在调用shutdown()和线程池真正shutdown之间提交任务，会拒绝新任务
    - 线程池会调用rejectedExecutionHandler来处理这个任务。如果没有设置默认是AbortPolicy，会抛出异常
7. 拒绝策略：
    - AbortPolicy 直接抛出一个RejectedExecutionException，这也是JDK默认的拒绝策略
    - CallerRunsPolicy 尝试直接运行被拒绝的任务，如果线程池已经被关闭了，任务就被丢弃了
    - DiscardPolicy 不能执行的任务将被删除
    - DiscardOldestPolicy 移除最晚的那个没有被处理的任务，然后执行被拒绝的任务。同样，如果线程池已经被关闭了，任务就被丢弃了
    - 自定义：实现RejectedExecutionHandler接口，可自定义处理器
 
## 四、ThreadPoolExecutor执行顺序：
     线程池按以下行为执行任务
 
1. currentThreadNum < corePoolSize->创建线程。
2. corePoolSize =< currentThreadNum ，对列未满，加入队列。
3. corePoolSize =< currentThreadNum <maxPoolSize ，且任务队列已满,创建线程
4. corePoolSize =< currentThreadNum &&currentThreadNum>=maxPoolSize ,且任务队列已满,创建线程，抛出异常，拒绝任务
 
## 五、如何设置参数
 
- 默认值
corePoolSize=1
queueCapacity=Integer.MAX_VALUE
maxPoolSize=Integer.MAX_VALUE
keepAliveTime=60s
allowCoreThreadTimeout=false
rejectedExecutionHandler=AbortPolicy()
- 如何来设置
    1. 需要根据几个值来决定
    - tasks ：每秒的任务数，假设为500~1000
    - taskcost：每个任务花费时间，假设为0.1s
    - responsetime：系统允许容忍的最大响应时间，假设为1s
    2. 做几个计算
    - corePoolSize = 每秒需要多少个线程处理？ 
threadcount = tasks/(1/taskcost) =tasks*taskcout =  (500~1000)*0.1 = 50~100 个线程。corePoolSize设置应该大于50
根据8020原则，如果80%的每秒任务数小于800，那么corePoolSize设置为80即可
    - queueCapacity = (coreSizePool/taskcost)*responsetime
计算可得 queueCapacity = 80/0.1*1 = 80。意思是队列里的线程可以等待1s，超过了的需要新开线程来执行
切记不能设置为Integer.MAX_VALUE，这样队列会很大，线程数只会保持在corePoolSize大小，当任务陡增时，不能新开线程来执行，响应时间会随之陡增。
    - maxPoolSize = (max(tasks)- queueCapacity)/(1/taskcost)
计算可得 maxPoolSize = (1000-80)/10 = 92
（最大任务数-队列容量）/每个线程每秒处理能力 = 最大线程数
    - rejectedExecutionHandler：根据具体情况来决定，任务不重要可丢弃，任务重要则要利用一些缓冲机制来处理
    - keepAliveTime和allowCoreThreadTimeout采用默认通常能满足

 