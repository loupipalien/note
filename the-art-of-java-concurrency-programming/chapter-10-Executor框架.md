### Executor 框架
Java 线程既是工作单元也是执行机制, 在 JDK 5 开始把工作单元与执行机制分离开来; 工作单元包括 Runnable 和 Callable, 执行机制由 Excutor 框架提供

#### Executor 框架简介

##### Executor 框架的两级调度模型
在 Hotspot VM 的线程模型中, Java 线程被一对一映射为本地操作系统线程, Java 线程启动时会创建一个本地操作系统线程; 当 Java 线程终止时, 这个操作系统线程也会被回收, 操作系统会调度所有线程并将它们分配给可用的 CPU  
在上层, Java 多线程程序通常把应用分解为若干个任务, 然后使用用户级的调度器 (Executor 框架) 将这些任务映射为固定数量的线程; 在底层, 操作系统内核将这些线程映射到硬件处理器上; 这两级调度模型如下图所示
![任务的两级调度模型.png](http://ww1.sinaimg.cn/large/d8f31fa4gy1g6w6ncfcxoj20h00dhwf4.jpg)  
应用层通过 Executor 框架控制上层的调度, 下层的调度由操作系统内核控制, 下层的调度不受应用程序的控制

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
Executor 框架最核心的类是 ThreadPoolExecutor, 它是线程池的实现类, 主要由以下四个构建组成
- corePool: 核心线程池大小
- maximumPool:最大线程池的大小
- BlockingQueue: 用来暂时保存任务的工作队列
- RejectedeExecutionHandler: ThreadPoolExecutor 关闭或饱和时, 用于拒绝 execute() 方法执行线程的策略

通过 Executor 框架的工具类 Executors 可以创建三种 ThreadPoolExecutor: FixedThreadPool, SingleThreadExecutor, CachedThreadPool
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
SingleThreadScheduledExecutor 的执行任意图如下  
![SingleThreadScheduledExecutor执行任意图.png](http://ww1.sinaimg.cn/large/d8f31fa4gy1g6w78980qij20gr0aa0t1.jpg)  
DelayQueue 是一个无界队列, 所以 ThreadPoolExecutor 的 maximumPoolSize 在 ScheduledThreadPoolExecutor 中是无意义的; 其执行主要分为两大部分
- 调用 scheduleAtFixedRate() 或 scheduleWithFixedDelay() 时会向 DelayQueue 中添加一个实现了 RunnableScheduledFuture 接口的 ScheduledFutureTask
- 线程池中的线程从 DelayQueue 中获取 ScheduledFutureTask, 然后执行任务

##### ScheduledThreadPoolExecutor 的实现
ScheduledFutureTask 主要包含三个成员变量
- long 型的 time, 表示这个任务将要被执行的具体时间
- long 型的 sequenceNumber, 表示这个任务被添加到 ScheduledThreadPoolExecutor 中的序号
- long 型的成员变量 period, 表示任务执行的间隔周期

DelayQueue 封装了一个 PriorityQueue, 这个 PriorityQueue 会对队列中的 ScheduledFutureTask 进行排序; 排序时 time 小的排在前面 (时间早的任务将被先执行); 如果两个 ScheduledFutureTask 的 time 相同, 就比较 sequenceNumber, sequenceNumber 小的排在前面; 下图为 ScheduledThreadPoolExecutor 中线程执行某个周期任务的四个步骤
![ScheduledThreadPoolExecutor中线程执行某个周期任务的四个步骤.png](http://ww1.sinaimg.cn/large/d8f31fa4gy1g6w7se3xijj20fm09bgm5.jpg)
- 线程 1 从 DelayQueue 中获取已到期的 ScheduledFutureTask (DelayQueue.take())
- 线程 1 执行这个 ScheduledFutureTask
- 线程 1 将这个 ScheduledFutureTask 的 time 修改为下次要被执行的时间
- 线程 1 把这个修改 time 之后的 ScheduledFutureTask 放回 DelayQueue 中 (DelayQueue.add())

DelayQueue.take() 方法实现如下
```Java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();  // 1
    try {
        for (;;) {
            E first = q.peek();
            if (first == null)
                available.await();  // 2.1
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    return q.poll();  // 2.2
                first = null; // don't retain ref while waiting
                if (leader != null)  
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);  // 2.3
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();  // 2.3
        lock.unlock();
    }
}
```
- 获取 lock
- 获取周期任务
  - fisrt 元素为空则等待
  - fisrt 元素的不为空, 且 time 小于等于当前时间则返回 fisrt 元素
  - fisrt 元素的不为空, 且 time 大于前时间则将 fisrt 置 null; 根据 leader 元素等待
- 释放 Lock

DelayQueue.add() 方法如下
```Java
public boolean add(E e) {
    return offer(e);
}

public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        q.offer(e);
        if (q.peek() == e) {
            leader = null;
            available.signal();
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```
- 获取 lock
- 添加任务
  - 向队列中添加任务
  - 如果添加的任务是头元素, 则唤醒在 Condition 中等待的线程
- 释放 Lock

#### FutureTask 详解
Future 接口和实现 Future 接口的实现 FutureTask 类代表异步计算的结果
##### FutureTask 简介
FutureTask 除了实现 Future 接口外, 还实现了 Runnable 接口; 因此, FutureTask 可以交给 Executor 执行, 也可以由调用线程直接执行 (FutureTask.run()); 根据 FutureTask.run() 方法被执行的时机, FutureTask 可以处于以下三种状态
- 未启动: FutureTask.run() 方法还没有被执行之前, FutureTask 处于未启动状态
- 已启动: FutureTask.run() 方法被执行的过程中, FutureTask 处于已启动状态
- 已完成: FutureTask.run() 方法执行完成后正常结束, 或被取消, 或异常结束

当 FutureTask 处于未启动或已启动状态时, 执行 FutureTask.get() 方法将导致调用线程阻塞; 当 FutureTask 处于已完成状态时, 执行 FutureTask.get() 方法将导致调用线程立即返回结果或抛出异常  
当 FutureTask 处于未启动状态时, 执行 FutureTask.cancel() 方法将导致此任务永远不会被执行; 当 FutureTask 处于已启动状态, 执行 FutureTask.cancel(true) 方法可以中断执行, 而执行 FutureTask.cancel(false) 方法将不会对正在执行任务的线程产生影响; 当 FutureTask 处于已完成状态, 执行 FutureTask.cancel(..) 方法将返回 false

##### FutureTask 的使用
当多个线程视图同时执行同一个任务时, 只允许一个线程执行任务, 其他线程需要等待这个任务执行完后才能继续执行
```Java
private final ConcurrentHashMap<Object,Future<String>> taskCache = new ConcurrentHashMap<>();

private String executionTask(final String taskName) throws ExecutionException, InterruptedException {
    while (true) {
        Future<String> future = taskCache.get(taskName);
        if (future == null) {
            Callable<String> task = new Callable<String>() {
                @Override
                public String call() throws Exception {
                    return taskName;
                }
            };
            // 创建任务
            FutureTask<String> futureTask = new FutureTask<>(task);
            future =taskCache.put(taskName, futureTask);
            if (future == null) {
                future = futureTask;
                // 执行任务
                futureTask.run();
            }
        }
        try {
            // 等待任务完成
            return future.get();
        } catch (CancellationException e) {
            taskCache.remove(taskName, future);
        }
    }
}
```
##### FutureTask 的实现
FutureTask 的实现基于 AbstractQueuedSynchronizer (AQS); 基于 AQS 实现的同步器包括: ReentrantLock, Semaphore, ReentrantReadWriteLock, CountDownLatch 和 FutureTask  
基于 "复合优先于继承" 的原则, FutureTask 声明了一个内部私有的继承于 AQS 的子类 Sync, 对 FutureTask 所有共有方法的调用都会委托给这个内部子类; AQS 被作为 "模板方法模式" 的基础类提供给 FutureTask 的内部子类 Sync, 这个内部子类只需要实现状态检查和状态更新的方法即可, 这些方法将控制 FutureTask 的获取和释放操作; 具体来说, Sync 实现了 AQS 的 tryAcquireShared(int) 方法和 tryReleaseShared(int) 方法, Sync 通过这两个方法来检查和更新同步状态  
![FutureTask设计示意图.png](http://ww1.sinaimg.cn/large/d8f31fa4ly1g70h9lnsgnj20l50epgmk.jpg)  
FutureTask.get() 方法会调用 acquireSharedInterruptibly(int arg) 方法, 这个方法执行过程如下
- 这个方法首先会回调子类 Sync 中实现的 tryAcquireShared() 方法来判断 acquire 操作是否成功; 此操作可以成功的条件为: state 为执行完成状态 RAN 或已取消状态 CANCELLED, 且 runner 不为 null
- 如果成功则 get() 方法立即放回, 如果失败则到线程等待队列中去等待其他线程执行 release 操作
- 当其他线程执行 release 操作唤醒当前线程后, 当前线程再次执行 tryAcquireShared() 将返回 1, 当前线程将离开线程等待队列并唤醒它的后继线程
- 最后返回计算的结果或抛出异常

FutureTask.run() 的执行过程如下
- 执行在构造函数中执行的任务 (Callable.call())
- 以原子方式来更新同步状态; 如果这个原子操作成功, 就设置代表计算结果的变量 result 值为 Callable.call() 的返回值, 然后调用 AQS.releaseShared(int arg)
- AQS.releaseShared(int arg) 首先会在子类 Sync 中实现的 tryReleaseShared(arg) 来执行 release 擦做; AQS.releaseShared(int arg), 然后唤醒线程等待队列中的第一个线程
- 调用 FutureTask.done()
