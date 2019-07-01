---
title: 'java多线程之FutureTask'
date: 2019-07-01 21:14:54
categories: java多线程
tags: java
---

## FutureTask和Future

### Future
#### Future接口主要方法：
```
    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
```


### FutureTask
FutureTask是Future的实现类，它提供了对Future的基本实现。可使用FutureTask包装Callable或Runnable对象，因为FutureTask实现了Runnable，所以也可以将FutureTask提交给Executor

#### FutureTask的几种状态
FutureTask实现了Runnable和Future接口。因此，FutureTask可以交给
Executor执行，也可以由调用线程直接执行（FutureTask.run()）。根据FutureTask.run()方法被执行
的时机，FutureTask可以处于下面3种状态。
1. 未启动:FutureTask.run()方法还没有被执行之前，FutureTask处于未启动状态。当创建一
个FutureTask，且没有执行FutureTask.run()方法之前，这个FutureTask处于未启动状态。
2. 已启动:FutureTask.run()方法被执行的过程中，FutureTask处于已启动状态。
3. 已完成:此状态有三种情况
    - 正常结束：FutureTask.run()方法执行完后。
    - 被取消：FutureTask.cancel（…）
    - 异常结束： 执行FutureTask.run()方法时抛出异常而异常结束，FutureTask处于已完成状态

#### FutureTask中get和cancel
1. get方法：`V get() throws InterruptedException, ExecutionException;`
    - 当FutureTask处于未启动或已启动状态时，执行FutureTask.get()方法将导致调用线程阻塞；
    -  当FutureTask处于已完成状态，调用FutureTask.get()方法将导致调用线程立即返回结果或者抛出异常。
 2. cancel方法: `boolean cancel(boolean mayInterruptIfRunning);`
    - 当FutureTask处于未启动状态时，执行FutureTask.cancel()方法将导致此任务永远不会被执行；
    - 当FutureTask处于已启动状态时，执行FutureTask.cancel（true）方法将以中断执行此任务线程的方式来试图停止任务；
    - 当FutureTask处于已启动状态时，执行FutureTask.cancel（false）方法将不会对正在执行此任务的线程产生影响（让正在执行的任务运行完成）；
    - 当FutureTask处于已完成状态时，执行FutureTask.cancel（…）方法将返回false。
3. get和cancel执行示例图
![FutureTask的get和cancel的执行示意图.jpg](/source/images/jus/FutureTask1.png)

#### FutureTask和Future的使用
Callable、Future、FutureTask一般都是和线程池配合使用的,以下是常用方法：
```
 <T> Future<T> submit(Callable<T> task);
 <T> Future<T> submit(Runnable task, T result);
 <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)throws InterruptedException;
 <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,long timeout, TimeUnit unit)throws  InterruptedException;
```


 1. Callable+Future使用示例
 ```
 public class FutureTaskTest {
    static class CallableTask implements Callable<String> {
        private Integer sleepTime;

        public CallableTask(Integer sleepTime) {
            this.sleepTime = sleepTime;
        }

        @Override
        public String call() throws Exception {
            System.out.println(Thread.currentThread().getName()+"--begin--"+System.currentTimeMillis());
            Thread.sleep(sleepTime);
            System.out.println(Thread.currentThread().getName()+"-------end--"+System.currentTimeMillis());
            return Thread.currentThread().getName();
        }
    }

    public static void main(String[] args) {

        ExecutorService executorService = Executors.newFixedThreadPool(3);

        List<Callable<String>> callableList = new ArrayList<>();
        callableList.add(new CallableTask(2000));
        callableList.add(new CallableTask(5000));
        callableList.add(new CallableTask(2000));

        try {
            List<Future<String>> futures = executorService.invokeAll(callableList);
            System.out.println("return future--"+System.currentTimeMillis());
            for (Future future : futures) {
                System.out.println(future.get() + "--get---"+System.currentTimeMillis());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        System.out.println("main thread compelete");
        executorService.shutdown();
    }
}
console result:
pool-1-thread-1--begin--1561992318002
pool-1-thread-3--begin--1561992318002
pool-1-thread-2--begin--1561992318002
pool-1-thread-3-------end--1561992320003
pool-1-thread-1-------end--1561992320003
pool-1-thread-2-------end--1561992323003
return future--1561992323003
pool-1-thread-1--get---1561992323003
pool-1-thread-2--get---1561992323004
pool-1-thread-3--get---1561992323004
 ```

 2. Callable+FutureTask使用示例
 ```
public static class CallableThread implements Callable<String>
{
    public String call() throws Exception
    {
        System.out.println("进入CallableThread的call()方法, 开始睡觉, 睡觉时间为" + System.currentTimeMillis());
        Thread.sleep(10000);
        return "123";
    }
}
    
public static void main(String[] args) throws Exception
{
    ExecutorService es = Executors.newCachedThreadPool();
    CallableThread ct = new CallableThread();
    FutureTask<String> f = new FutureTask<String>(ct);
    es.submit(f);
    es.shutdown();
        
    Thread.sleep(5000);
    System.out.println("主线程等待5秒, 当前时间为" + System.currentTimeMillis());
        
    String str = f.get();
    System.out.println("Future已拿到数据, str = " + str + ", 当前时间为" + System.currentTimeMillis());
}
 ```
 Callable+Future的方式，es.submit(ct)方法返回的Future，底层实现new出来的是一个FutureTask


 

