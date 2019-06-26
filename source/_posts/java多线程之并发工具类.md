---
title: java多线程之并发工具类
date: 2019-06-22 20:41:00
categories: java多线程
tags: java
---

## CountDownLatch

- CountDownLatch的作用：允许一个或多个线程，等待其他线程达到触发条件时（触发条件即CountDownLatch对象计数器减为0时）才开始执行
- 主要方法
    1. CountDownLatch构造函数：CountDownLatch的构造函数接收一个int类型的参数作为计数器，如果你想等待N个点完
成，这里就传入N
    2. countDown：当我们调用CountDownLatch的countDown方法时，N就会减1
    3. await:CountDownLatch的await方法
会阻塞当前线程，直到N变成零,当前线程才会执行；await（long time，TimeUnit unit）这个方法等待特定时
间后，就会不再阻塞当前线程
- 示例代码：
```
**
 * @Description: TODO
 * @Auther: ThomasWu
 * @Date: 2019/6/22 20:53
 * @Email:1414924381@qq.com
 */
public class CountDownLatchTest {

    private static class WorkTask extends Thread {
        private CountDownLatch countDownLatch;

        public WorkTask(String name, CountDownLatch countDownLatch) {
            super(name);
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            try {
                System.out.println(this.getName() + "启动了"+System.currentTimeMillis());
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                System.out.println(this.getName() + "执行完了"+System.currentTimeMillis());
                countDownLatch.countDown();
            }
        }
    }

    private static class DoneTask extends Thread {
        private CountDownLatch countDownLatch;

        public DoneTask(String name, CountDownLatch countDownLatch) {
            super(name);
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            try {
                System.out.println(this.getName() + "开始等待"+System.currentTimeMillis());
                countDownLatch.await();
                System.out.println(this.getName() + "执行完了"+System.currentTimeMillis());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }


    public static void main(String[] args) {
        CountDownLatch countDownLatch=new CountDownLatch(3);
        ExecutorService service=Executors.newFixedThreadPool(5);
        service.submit(new DoneTask("DonwTask1",countDownLatch));
        service.submit(new DoneTask("DoneTask2",countDownLatch));
        service.submit(new WorkTask("workTask1",countDownLatch));
        service.submit(new WorkTask("workTask2",countDownLatch));
        service.submit(new WorkTask("workTask3",countDownLatch));
    }
}

console result
DonwTask1开始等待1561210606078
DoneTask2开始等待1561210606078
workTask2启动了1561210606079
workTask1启动了1561210606079
workTask3启动了1561210606079
workTask3执行完了1561210607079
workTask2执行完了1561210607079
workTask1执行完了1561210607079
DonwTask1执行完了1561210607079
DoneTask2执行完了1561210607079
```

## CyclicBarrier(同步屏障或栅栏)

- CyclicBarrier作用:让一
组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会
开门，所有被屏障拦截的线程才会继续运行
- 主要方法
    1. 构造函数CyclicBarrier（int parties，Runnable barrierAction）:用于在线程到达屏障时，优先执行barrierAction
    2. 默认构造函数CyclicBarrier（int parties）：屏障拦截的线程数
量，每个线程调用await方法告诉CyclicBarrier我已经到达了屏障，然后当前线程被阻塞
    3. await:调用该方法的线程会被阻塞
- 注意：线程池中使用CyclicBarrier ，要考虑线程被阻塞问题，

- 示例代码
```
/**
 * @Description: 多任务结算银行流水
 * @Auther: ThomasWu
 * @Date: 2019/6/22 22:32
 * @Email:1414924381@qq.com
 */
public class CyclicBarrierTest {
    static class BankWaterTask implements Runnable {
        private ConcurrentHashMap<String, Integer> sheetBankWaterCount = new ConcurrentHashMap<String, Integer>();
        private CyclicBarrier c = new CyclicBarrier(4, this);
        private Executor executor = Executors.newFixedThreadPool(4);

        private void count() {
            System.out.println("count 开始" + System.currentTimeMillis());
            for (int i = 0; i < 4; i++) {
                executor.execute(() -> {
                    sheetBankWaterCount.put(Thread.currentThread().getName(), 1);
                    try {
                        c.await();
                    } catch (InterruptedException | BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                });
            }
            System.out.println("count 结束" + System.currentTimeMillis());
        }

        @Override
        public void run() {
            System.out.println("run" + System.currentTimeMillis());
            int result = 0;
            for (Map.Entry<String, Integer> sheet : sheetBankWaterCount.entrySet()) {
                result += sheet.getValue();
            }
            sheetBankWaterCount.put("result", result);
            System.out.println(result);
        }

    }

   public static void main(String[] args) {
        BankWaterTask task = new BankWaterTask();
        task.count();
    }
```

### CyclicBarrier和CountDownLatch的区别
- CountDownLatch：调用countDown的线程（A线程），调用了await的线程（B线程），A线程先执行，B线程阻塞，满足条件（计数器为0时）B线程开始执行，没有被阻塞；
- CyclicBarrier：构造函数传入barrierAction线程（A线程） ，调用了await的线程（B线程），A线程先阻塞，B线程也阻塞，满足条件（B线程都调用都调用await后），A线程开始执行完后，B线程开始执行
- CyclicBarrier：barrierAction只能指定一个线程
- CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset()方法重置
- CyclicBarrier还提供其他有用的方法
    1. getNumberWaiting方法可以获得Cyclic-Barrier阻塞的线程数量
    2. isBroken()方法用来了解阻塞的线程是否被中断

 ## Semaphore 

- Semaphore的作用：用来控制同时访问特定资源的线程数量，它通过协调各个线程，以
保证合理的使用公共资源
- 使用场景： Semaphore可以用于做流量控制，特别是公用资源有限的应用场景，比如数据库连接。假
如有一个需求，要读取几万个文件的数据，因为都是IO密集型任务，我们可以启动几十个线程
并发地读取，但是如果读到内存后，还需要存储到数据库中，而数据库的连接数只有10个，这
时我们必须控制只有10个线程同时获取数据库连接保存数据，否则会报错无法获取数据库连
接。这个时候，就可以使用Semaphore来做流量控制
- 主要方法：
    1. Semaphore（int permits）：接受一个整型的数字，表示可用的许可证数量
    2. void acquire()：获取一个许可证，如果当前信号量大于0，则当前信号量的计数减1，该方法直接返回；如果当前信号量等于0，则当前线程会被放入AQS阻塞队列
    3. void release():将当前信号量加1

示例代码：
```

/**
 * @Description: TODO
 * @Auther: ThomasWu
 * @Date: 2019/6/23 15:19
 * @Email:1414924381@qq.com
 */
public class SemaphoreTest {
    private static final int THREAD_COUNT = 10;
    private static ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_COUNT);
    private static Semaphore s = new Semaphore(5);

    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
            threadPool.execute(() -> {
                try {
                    s.acquire();
                    System.out.println(Thread.currentThread().getName()+"获得了信号量"+"---当前信号量:"+s.availablePermits()+"---等待信号量的线程数:"+s.getQueueLength()+"---"+System.currentTimeMillis());
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    s.release();
                    System.out.println(Thread.currentThread().getName()+"释放了信号量------"+"---当前信号量:"+s.availablePermits()+"---等待信号量的线程数:"+s.getQueueLength()+System.currentTimeMillis());
                }
            });
        }
        threadPool.shutdown();
    }
}

console result
pool-1-thread-2获得了信号量---当前信号量:4---等待信号量的线程数:0---1561275554694
pool-1-thread-1获得了信号量---当前信号量:3---等待信号量的线程数:0---1561275554694
pool-1-thread-4获得了信号量---当前信号量:2---等待信号量的线程数:0---1561275554694
pool-1-thread-3获得了信号量---当前信号量:1---等待信号量的线程数:0---1561275554694
pool-1-thread-5获得了信号量---当前信号量:0---等待信号量的线程数:0---1561275554694
pool-1-thread-6获得了信号量---当前信号量:0---等待信号量的线程数:4---1561275556695
pool-1-thread-7获得了信号量---当前信号量:0---等待信号量的线程数:3---1561275556695
pool-1-thread-4释放了信号量---------当前信号量:0---等待信号量的线程数:31561275556695
pool-1-thread-1释放了信号量---------当前信号量:0---等待信号量的线程数:31561275556695
pool-1-thread-8获得了信号量---当前信号量:0---等待信号量的线程数:2---1561275556695
pool-1-thread-2释放了信号量---------当前信号量:0---等待信号量的线程数:21561275556695
pool-1-thread-9获得了信号量---当前信号量:1---等待信号量的线程数:0---1561275556696
pool-1-thread-3释放了信号量---------当前信号量:0---等待信号量的线程数:01561275556696
pool-1-thread-5释放了信号量---------当前信号量:1---等待信号量的线程数:01561275556696
pool-1-thread-10获得了信号量---当前信号量:0---等待信号量的线程数:0---1561275556696
pool-1-thread-6释放了信号量---------当前信号量:1---等待信号量的线程数:01561275558695
pool-1-thread-8释放了信号量---------当前信号量:3---等待信号量的线程数:01561275558695
pool-1-thread-7释放了信号量---------当前信号量:2---等待信号量的线程数:01561275558695
pool-1-thread-10释放了信号量---------当前信号量:5---等待信号量的线程数:01561275558696
pool-1-thread-9释放了信号量---------当前信号量:5---等待信号量的线程数:01561275558696
```



 ## Exchanger

 - Exchanger作用：用于进行线程间的数据交
换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过
exchange方法交换数据，如果第一个线程先执行exchange()方法，它会一直等待第二个线程也
执行exchange方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产
出来的数据传递给对方。
- 注意：如果两个线程有一个没有执行exchange()方法，则会一直等待，如果担心有特殊情况发
生，避免一直等待，可以使用exchange（V x，longtimeout，TimeUnit unit）设置最大等待时长



 示例代码：
 ```
/**
 * @Description: TODO
 * @Auther: ThomasWu
 * @Date: 2019/6/23 15:53
 * @Email:1414924381@qq.com
 */
public class ExchangerTest {

    private static final Exchanger<String> exchanger = new Exchanger<String>();
    private static ExecutorService threadPool = Executors.newFixedThreadPool(2);

    public static void main(String[] args) {
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    // A录入银行流水数据
                    String str = "银行流水A";
                    System.out.println(Thread.currentThread().getName() + str);
                    str = exchanger.exchange(str);
                    System.out.println(Thread.currentThread().getName() + str);
                } catch (InterruptedException e) {
                }
            }
        });
        threadPool.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    String str = "银行流水B";
                    System.out.println(Thread.currentThread().getName() + str);
                    str = exchanger.exchange(str);
                    System.out.println(Thread.currentThread().getName() + str);

                } catch (InterruptedException e) {
                }
            }
        });
        threadPool.shutdown();
    }
}

 ```