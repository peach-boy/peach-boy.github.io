---
title: interceptor 配合Threadlocal全局处理请求基本信息
date: 2019-06-18 16:43:32
categories: java基础
tags: java
---

### 业务场景

> 最近接手一个供app端调用的网关项目，app端迭代更新，接口就要相应的区分版本并做相应的逻辑区分，所以每一次请求必须要获取版本号。

### 实现逻辑

> 期初的做法是在controller中直接定义 @RequestParam("name") String name，但发现调用的方法层数增加时，参数需要逐层传递，调用链上每个方法都需要加参数，代码及不优雅。后改为添加一个拦截器VersionInfoInterceptor，在拦截器中操作Threadlocal中的变量，代码如下：

    /**
     * @author:xiantao.wu
     * @createDate:2018/7/1011:25
     * 先执行preHandle方法，将需要的变量放入ThreadLocal,方法执行完执行afterCompletion，将变量remove掉，拦截器原理后续文章将会介绍
     * 
     **/
    public class VersionInfoInterceptor extends HandlerInterceptorAdapter {
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            String clientVersion = request.getHeader("x-client-version");
            if (clientVersion != null) {
                RequestHeaderBaseInfo requestHeaderBaseInfo=new RequestHeaderBaseInfo();
                requestHeaderBaseInfo.setVersion(clientVersion);
                ThreadLocalUtil.set(requestHeaderBaseInfo);
                return true;
            } else {
                return false;
            }
        }
    
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            ThreadLocalUtil.remove();
        }
    

> 直接TreadlocalUtil.get()即可获取version放入的变量

    /**
     * @author:xiantao.wu
     * @createDate:2018/7/1011:25
     * 提供工具类你，操作Threadlocal
     **/
    public class ThreadLocalUtil {
        private ThreadLocalUtil() {
        }
    
        private static ThreadLocal<RequestHeaderBaseInfo> LOCAL = new  TransmittableThreadLocal<>();
    
        public static void set(RequestHeaderBaseInfo requestHeaderBaseInfo) {
            LOCAL.set(requestHeaderBaseInfo);
        }
    
        public static RequestHeaderBaseInfo get() {
            return LOCAL.get();
        }
    
        public static void remove() {
            LOCAL.remove();
        }
    
    }
    

### 敲黑板（需要避免的坑） 现在ThreadLocal除原生实现外，还有InheritableThreadLocal，TransmittableThreadLocal（眼尖的同学可能已经注意到了）这两种实现，上面选用的就是TransmittableThreadLocal，以下是三者的比较：

|  实现 | 能否在传递到子线程  | 使用注意点  |
| :------------ | :------------ | :------------ |
| ThreadLocal  | 否  | 只能在当前线程中获取本地变量  |
|InheritableThreadLocal|是|可获取父线程的中的本地变量，线程池场景不适用，存在线程复用，会导致混|
|TransmittableThreadLocal|是|可获取父线程中的本地变量，适用线程池场景|

    

### 测试结果

/** * Created by wuxiantao on 2018/7/14. */ public class TransmittableThreadlocalTest {

    private static ThreadLocal<String> transmittableThreadLocal=new TransmittableThreadLocal<>();
    
    public static void main(String[] args) throws InterruptedException {
        ThreadPoolExecutor threadPoolExecutor=new ThreadPoolExecutor(3,3,10, TimeUnit.MINUTES, new LinkedBlockingQueue<Runnable>());
        transmittableThreadLocal.set("父线程设定的值");
        System.out.println(Thread.currentThread().getName()+"===="+transmittableThreadLocal.get());
    
        for (int i = 0; i < 5; i++) {
            threadPoolExecutor.submit(TtlCallable.get(new Callable<String>() {
                public String call() throws Exception {
                    System.out.println(Thread.currentThread().getName()+"------"+transmittableThreadLocal.get());
                    transmittableThreadLocal.set("我被修改了");
                    return transmittableThreadLocal.get();
                }
            }));
        }
    }
    

}

    <br />使用transmimttable-threadlocal中TtlCallable.get()提交任务，在子线程中修改Threadlocal,子线程还是会取主线程中的threadlocal
    
```java
main====父线程设定的值
pool-1-thread-2------父线程设定的值
pool-1-thread-1------父线程设定的值
pool-1-thread-1------父线程设定的值
pool-1-thread-1------父线程设定的值
pool-1-thread-3------父线程设定的值
```
\``\`
