### 并发与多线程
在并发环境下, 由于程序性的封闭性打破, 出现了以下特点
- 并发程序之间有互相制约的关系
直接制约体现为一个程序需要另一个程序的计算结果; 间接制约体现为多个程序竞争共享资源
- 并发程序的执行过程是断断续续的
程序需要记忆现场指令以及执行点
- 当并发数设置合理并且 CPU 拥有足够的处理能力时, 并发会提高程序的运行效率

#### 线程安全
线程可以拥有自己的操作栈, 程序计数器, 局部变量表等资源; 与同一进程内的其他线程共享该进程的所有资源; 线程在生命周期内存在多种状态
- NEW, 即新建状态, 是线程被创建且未启动的状态
创建线程的方式有三种: 继承 Thread 类; 实现 Runnable 接口; 实现 Callable 接; 相比第一种更推荐使用第二种方式, 因为继承不符合里氏替换原则; Runnable 和 Callable 有两点不同: Callable 可以通过 call() 获得返回值, 其次可以抛出异常, 而 Runnable 只有通过 setDefaultUncaughtExceptionHandler() 的方式才能在主线程中捕捉到子线程的异常
- RUNNABLE, 即就绪状态, 是调用 start() 之后运行之前的状态
线程不能 start() 多次, 否则会抛出 IllegalStateException 异常
- RUNNING, 即运行状态, 是 run() 正在执行线程的状态
线程可能会由于某些因素而退出 RUNNING, 如时间, 调度, 锁等
- BLOCKED, 即阻塞状态, 进入此状态, 有以下几种情况
  - 同步阻塞: 锁被其他线程占用
  - 主动阻塞: 调用 Thread 的某些方法, 主动让出 CPU 执行权, 如 sleep(), join() 等
  - 等待阻塞: 执行了 wait() 等
- DEAD, 即终止状态, 是 run() 执行结束, 或因异常退出后的状态, 此状态不可逆转

保证高并发场景下的线程安全, 可以从以下四个维度考量
- 数据单线程内可见
单线总是安全的, 通过限制数据仅在单线程内可见, 可以避免数据被其他线程篡改
- 只读对象
只读对象总是安全的, 它的特性是允许复制, 拒绝写入
- 线程安全类
某些线程安全类的内部都有非常明确的线程安全机制
- 同步与锁机制
线程安全的核心理念就是 “要么只读, 要么加锁”; 合理利用 java.uitl.concurrent 包, 该包中主要有几个家族
  - 线程同步类: 这些类使线程之间协调更加容易, 支持了更加丰富的线程协调场景
  - 并发集合类: 集合并发操作的要求是执行速度快, 提取数据准确
  - 线程管理类: 各种线程池
  - 锁相关类: 锁以 Lock 接口为核心, 派生出在一些实际场景中进行互斥操作的锁相关类

#### 什么是锁
计算机中的锁开始是从悲观锁, 发展到后来的乐观锁, 偏向锁, 分段锁等; 锁主要提供了两种特性: 互斥性和不可见性; 因为锁的存在, 某些操作对外界来说是黑箱操作的, 只有锁的持有者才知道对变量进行了什么修改; Java 中常用锁实现的方式有两种

##### 用并发包中的锁类
并发包的类族中, Lock 是 JUC 包的顶层接口, 它的实现逻辑并未用到 synchronized, 而是利用了 volatile 的可见性; ReentrantLock 对于 Lock 接口的实现主要依赖了 Sync, 而 Sync 继承了 AbstractQueuedSynchronizer (AQS), 它是 JUC 包实现同步的基础工具; 在哦 AQS 中, 定义了一个 volatile int state 变量作为共享资源, 如果线程获取资源失败, 则进入同步 FIFO 队列中等待, 如果成功获取资源就执行临界区代码; 执行完释放资源时, 会通知同步队列中的等待线程来获取资源后出队并执行  
AQS 是抽象类, 内置自旋锁实现的同步队列, 封装入队和出队的操作, 提供了独占, 共享, 中断等特性的方法; AQS 的子类可以定义不同的资源实现不同性质的方法; 如可重入锁 ReentrantLock, 定义 state 为 0 时可以获取资源并置为 1, 若已获得资源 state 不断加 1, 在释放资源时 state 减 1, 直至为 0; CountDownLatch 初始化时定义了资源总量 state = count, countDown() 不断减 1, 当 state = 0 时才能获得锁, 释放后 state 就一直为 0, 所有线程调用 await() 都不会等待, 所以 CountDownLatch 是一次性的, 用完后如果再想用就只能重建一个; 如果希望循环利用, 推荐使用基于 ReentrantLock 实现的 CyclicBarrier; Semaphore 与 CountDownLatch 略有不同, 同样一是定义了资源总量 state = permits, 当 state > 0 时就能获得锁, 并将 state 减 1, 当 state - 0 时只能等待其他线程释放锁, 当释放锁时 state 加 1, 其他等待线程又能获得这个锁; 当 Semaphore 的 permits 定义为 1 时, 就是互斥锁, 当 permits > 1 就是共享锁; JDK8 新出了 StampedLock, 改进了读写锁 ReentrantReadWriteLock; 这些锁降低了并发编程的难度

##### 利用同步代码块
同步代码块一般使用 synchronized 关键字来实现, 有两种方式对方法进行加锁操作: 第一在方法签名处加 synchronized 关键字, 第二使用 synchronized(对象或类) 进行同步; 这里的原则是锁的范围尽可能的小, 锁的时间尽可能的短  
synchronized 锁特性由 JVM 负责实现, JVM 底层是通过监视锁来实现 synchronized 同步的, 监视锁即 monitor, 是每个对象与生俱来的一个隐藏字段; 线程在进入同步方法或代码块时, 会获取该方法或代码块所属对象的 monitor, 进行加锁判断; 如果成功加锁就成为该 monitor 的唯一持有者, monitor 在被释放前, 不能再被其他线程获取; JDK6 后不断优化使得 synchronized 提供三种锁的实现, 包括偏向锁, 轻量级锁, 重量级锁, 还提供了自动的升级和降级机制; JVM 就是在对象头上设置线程 ID, 表示这个对象偏向于当前线程, 这就是偏向锁  
偏向锁是为了在资源没有被多线程竞争的情况下尽量减少锁带来的性能开销; 在锁对象的对象头中有一个 ThreadId 字段, 当第一个线程访问锁时, 如果该锁没有被其他线程访问过, 即 ThreadId 字段为空, 那么 JVM 让其持有偏向锁, 并将 ThreadId 字段的只设置为该线程的 ID; 当下一次获取锁时, 会判断当前线程 ID 是否与锁对象的 ThreadId 一致; 如果一致, 那么该线程不会再重复获取锁, 从而提高了程序的运行效率; 如果出现锁的竞争情况, 那么偏向锁会被撤销并升级为轻量级锁, 如果资源的竞争非常激烈, 会升级为重量级锁; 偏向锁可以降低无竞争开销, 它不是互斥锁, 不存在线程竞争的情况, 省去再次同步判断的步骤, 提升了性能

#### 线程同步
资源共享的两个原因是资源紧缺和共建需求; 线程共享 CPU 是从资源紧缺的维度来考虑的, 而多线程共享同一变量, 通常是从共建需求的维度来考虑的;

##### volatile
TODO

##### 信号量同步
TODO

#### 线程池

##### 线程池的好处
线程池的作用包括
- 利用线程池管理并复用线程, 控制最大并发数
- 实现任务线程队列缓存策略和拒绝机制
- 实现某些与时间相关的功能, 如定时执行, 周期执行等
- 隔离线程环境

以下是 ThreadPoolExecutor 的构造方法
```
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
- corePoolSize
表示常驻核心线程数, 如果等于 0, 则是任务执行完之后, 没有任何请求进入时销毁线程池的线程, 如果大于 0, 即使任务执行完成线程也不会销毁
- maximumPoolSize
表示线程池能够容纳同时执行的最大线程数
- keepAliveTime
表示线程池中线程空闲时间, 当空闲时间达到 keepAliveTime 值时, 线程会被销毁, 直到只剩下 corePoolSize 个线程为止; 在默认情况下, 当线程池的线程数大于 corePoolSize 时, keepAliveTime 才会起作用, 但是当 ThreadPoolExecutor 的 allowCoreThreadTimeOut 变量设置为 true 时, 核心线程超时后也会被收回
- TimeUnit
表示 keepAliveTime 的时间单位
- workQueue
表示缓存队列
- threadFactory
表示线程工厂; 用来生产一组相同任务的线程, 线程池创建的线程命名是通过这个 factory 增加组名前缀实现的
- handler
表示执行拒绝策略的对象

以下是线程池相关类图
```
                        Executor
                           ^
                           |
                     ExecutorService
                           ^
                           |
                  ------------===============
                  |                         |
       AbstractExecutorService   ScheduledExecutorService
                  ^                         ^
                  |                         |
       ====================                 |
       ^                  ^                 |
       |                  |                 |
 ForkJoinPool    ThreadPoolExecutor         |
                          ^                 |
                          |                 |
                          ==========---------
                                    ^
                                    |
                      ScheduledThreadPoolExecutor
```
ExecutorService 接口继承了 Executor 接口, 定义了管理线程任务的方法; ExecutorService 的抽象类提供了 submit() 和 invokeAll() 等部分方法的实现; 核心方法 Executor.execute() 在不同的实现类中实现; Executors 的核心方法有五个
- Executors.newWorkStealingPool()
JDK8 引入, 创建持有足够线程的线程池支持给定的并行度, 并通过使用多个队列减少竞争
  ```
  public static ExecutorService newWorkStealingPool() {
      return new ForkJoinPool
          (Runtime.getRuntime().availableProcessors(),
           ForkJoinPool.defaultForkJoinWorkerThreadFactory,
           null, true);
  }
  ```
- Executors.newCachedThreadPool()
maximumPoolSize 最大可至 Integer.MAX_VALUE, 是高度可伸缩的线程池
- Executors.newScheduledThreadPool()
线程树最大至 Integer.MAX_VALUE, 是 ScheduledExecutorService 接口家族的实现类, 支持定时及周期性任务执行; 与 newCachedThreadPool 的区别是不回收线程
- Executors.newSingleThreadExecutor()
创建一个单线程的线程池
- Executors.newFixedThreadPool()
创建一个固定线程数的线程池

##### 线程池源码详解
