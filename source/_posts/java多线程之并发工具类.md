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

## CyclicBarrier和CountDownLatch的区别
- CountDownLatch：调用countDown的线程（A线程），调用了await的线程（B线程），A线程先执行，B线程阻塞，满足条件（计数器为0时）B线程开始执行，没有被阻塞；
- CyclicBarrier：构造函数传入barrierAction线程（A线程） ，调用了await的线程（B线程），A线程先阻塞，B线程也阻塞，满足条件（B线程都调用都调用await后），A线程开始执行完后，B线程开始执行

- CyclicBarrier：barrierAction只能指定一个线程

- CountDownLatch的计数器只能使用一次，而CyclicBarrier的计数器可以使用reset()方法重
置
- CyclicBarrier还提供其他有用的方法
    1. getNumberWaiting方法可以获得Cyclic-Barrier阻塞的线程数量
    2. isBroken()方法用来了解阻塞的线程是否被中断