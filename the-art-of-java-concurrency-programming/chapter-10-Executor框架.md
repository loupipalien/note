### Executor 框架
Java 线程既是工作单元也是执行机制, 在 JDK 5 开始把工作单元与执行机制分离开来; 工作单元包括 Runnable 和 Callable, 执行机制由 Excutor 框架提供

#### Executor 框架简介

##### Executor 框架的两级调度模型
在 Hotspot VM 的线程模型中, Java 线程被一对一映射为本地操作系统线程, Java 线程启动时会创建一个本地操作系统线程; 当 Java 线程终止时, 这个操作系统线程也会被回收, 操作系统会调度所有线程并将它们分配给可用的 CPU; 应用层通过 Executor 框架控制上层的调度, 下层的调度由操作系统内核控制, 下层的调度不受应用程序的控制

##### Executor 框架的结构与成员

###### Executor 框架的结构
Executor 框架主要由三大部分组成
- 任务: 包括被执行任务需要实现的接口, Runnable 和 Callable 接口
- 任务的执行: 包括任务执行机制的核心接口 Executor, 以及继承自 Executor 的 ExecutorService 接口; Executor 框架有两个关键类实现了 ExecutorService 接口, ThreadPoolExecutor 和 ScheduledThreadPoolExecutor
- 异步计算的结果: 包括接口 Future 接口和 FutureTask 类

###### Executor 框架的成员
ThreadPoolExecutor, ScheduledThreadPoolExecutor, Future 接口, Runnable 接口, Callable 接口, Executors
- ThreadPoolExecutor
ThreadPoolExecutor 通常使用工厂类 Executors 来创建, 有三种类型的 ThreadPoolExecutor: SingleThreadExecutor, FixedThreadPool, CachedThreadPool
  - SingleThreadExecutor: 适用于需要保证顺序执行各个任务, 并且在任意时间地点不会有多个线程是活动的场景
  - FixedThreadPool: 适用于为了满足资源管理的需求, 而需要限制当前线程数量的应用场景, 适用于负载较重的服务器
  - CachedThreadPool: 是大小无界的线程池, 适用于执行很多的短期异步任务的小程序, 或者是负载较轻的服务器
- ScheduledThreadPoolExecutor
  - ScheduledThreadPoolExecutor: 适用于需要多个后台线程执行周期任务, 同时满足管理的需求而需要限制后台线程数量的应用场景
  - SingleThreadScheduledExecutor: 适用于需要单个后台线程执行周期任务, 同时需要保证顺序执行各个任务的场景
- Future 接口
Future 接口和实现 Future 接口的 FutureTask 类用来表示异步计算的结果
- Runnable 接口和 Callable 接口
这两个接口的实现类都可以被 ThreadPoolExecutor 或 ScheduledThreadPoolExecutor 执行, 它们的区别仅在于 Runnable 不会返回结果, Callable 可以返回结果

#### ThreadPoolExecutor 详解
TODO

##### FixedThreadPool 详解
```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```
当线程数达到 corePoolSize 后, 新任务将在无界队列中等待, 线程池中线程数不会超过 corePoolSize; maximumPoolSize 和 keepAliveTime 是两个无效参数, 由于使用无界队列, 运行中的 FixedThreadPool 不会拒绝任务

##### SingleThreadExecutor 详解
```
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```
corePoolSize 和 maximumPoolSize 都设置为 1, 其余与 FixedThreadPool 一样

##### CachedThreadPool 详解
```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```
corePoolSize 设置为 0, maximumPoolSize 设置为 Integer.MAX_VALUE, 意味着创建的线程数可能很多, 极端情况下会因为创建过多线程而耗尽 CPU; keepAliveTime 为 60 即空闲线程闲置 60 秒就会被销毁, 而队列使用阻塞的无容量队列 SynchronousQueue, 当提交的任务没有被空闲的线程或者新创建的线程获取, 否则队列一直阻塞

#### ScheduledThreadPoolExecutor 详解
ScheduledThreadPoolExecutor 继承与 ThreadPoolExecutor, 主要用于来在给定的延迟之后运行任务, 或定期执行任务; ScheduledThreadPoolExecutor 的功能与 Timer 类似, 但功能更强大, 更灵活; Timer 对应的是单个后台线程, ScheduledThreadPoolExecutor 可以在构造函数中指定多个对应的后台线程

##### ScheduledThreadPoolExecutor 的运行机制
TODO 
