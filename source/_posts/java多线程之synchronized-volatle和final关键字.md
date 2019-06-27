---
title: 'java多线程之synchronized,volatle和final关键字'
date: 2019-06-26 22:53:03
categories: java多线程
tags: java
---

## synchronized简介
一个共享变量同时被多个线程读写时，会出现不一致问题。我们来看下面的代码：
```

/**
 * @Description: 同时1000个线程给count加一
 * @Auther: ThomasWu
 * @Date: 2019/6/27 21:43
 * @Email:1414924381@qq.com
 */
public class SynchronizedTest {
    private static Integer count = 0;

    public static class MyThread extends Thread {
        @Override
        public void run() {
            addNum();
        }
    }

    public static void addNum() {
        count++;
    }

    public static void main(String[] args) {
        for (int i = 0; i < 1000; i++) {
            MyThread thread = new MyThread();
            thread.start();
        }

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("result:" + count);
    }
}
第一次运行：result:660
第二次运行：result:660
第三次运行：result:660
每一次运行结果都不等于1000

```
## synchronized使用的各种场景
|位置|被锁住的对象|实例|
|---|----|----|
| 实例方法|类的实例对象|`public synchronized void method{}`|
| 静态方法|类对象|`public static synchronized void method{}`|
| 实例对象|类的实例对象|` synchronized (this){}`|
| class对象|类对象|` synchronized (Demo.class){}`|
| 任意实例对象|实例对象|` String loclk="";   synchronized (lock){}`|


### synchronized用于实例方法上
示例代码
```
 package syn;

/**
 * @Description: TODO
 * @Auther: ThomasWu
 * @Date: 2019/6/27 22:25
 * @Email:1414924381@qq.com
 */
public class LockObject {

    static class ThreadB extends Thread {
        private LockObject lockObject;

        public ThreadB(LockObject lockObject) {
            super();
            this.lockObject = lockObject;
        }

        @Override
        public void run() {
            super.run();
            lockObject.methodB();

        }
    }

    static class ThreadA extends Thread {
        private LockObject lockObject;

        public ThreadA(LockObject lockObject) {
            super();
            this.lockObject = lockObject;
        }

        @Override
        public void run() {
            super.run();
            lockObject.methodA();
        }
    }

    public synchronized void methodA() {
        try {
            System.out.println(Thread.currentThread().getName() + "--begin--" + System.currentTimeMillis());
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + "--end----" + System.currentTimeMillis());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void methodB() {
        try {
            System.out.println(Thread.currentThread().getName() + "--begin--" + System.currentTimeMillis());
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + "--end----" + System.currentTimeMillis());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        LockObject lockObject = new LockObject();

        ThreadA threadA = new ThreadA(lockObject);
        threadA.setName("A");

        ThreadB threadB = new ThreadB(lockObject);
        threadB.setName("A");

        threadA.start();
        threadB.start();
    }
}

result1 :methodA为同步方法，methodB为同步方法执行结果
A--begin--1561647391094
A--begin--1561647391094
A--end----1561647392094
A--end----1561647392094

result2:methodB,methodA都为同步方法执行结果
A--begin--1561647419478
A--end----1561647420480
A--begin--1561647420480
A--end----1561647421480

```
由此上面的实验可以得出：
- 1、A线程持有Object对象的Lock锁，B线程可以以异步方式调用Object对象中的非synchronized类型的方法
- 2、A线程持有Object对象的Lock锁，B线程如果在这时调用Object对象中的synchronized类型的方法则需要等待，也就是同步