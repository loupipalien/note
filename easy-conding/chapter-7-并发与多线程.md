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
属性定义部分
```
public class ThreadPoolExecutor extends AbstractExecutorService {
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
    private static final int COUNT_BITS = Integer.SIZE - 3;  // MARK: Integer 共 32 位, 最右边 29 位表示工作线程数, 最左边 3 位表示线程池状态
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;  // MARK: 线程池容量, 000-11111111111111111111111111111

    // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;  // MARK: 此状态表示线程池可以接受新任务, 111-0000000000000000000000000000
    private static final int SHUTDOWN   =  0 << COUNT_BITS;  // MARK: 此状态表示不再接受新任务, 但是可以继续执行队列中的任务, 000-0000000000000000000000000000
    private static final int STOP       =  1 << COUNT_BITS;  // MARK: 此状态表示不再接受新任务, 并中断正在处理的任务, 001-0000000000000000000000000000
    private static final int TIDYING    =  2 << COUNT_BITS;  // MARK: 此状态表示所有任务已经被终止, 010-0000000000000000000000000000
    private static final int TERMINATED =  3 << COUNT_BITS;  // MARK: 此状态表示已清理完现场, 011-0000000000000000000000000000

    // Packing and unpacking ctl
    private static int runStateOf(int c)     { return c & ~CAPACITY; }  // MARK: CAPACITY 取反, 做与操作后只保留 c 的前三位, 即获取状态
    private static int workerCountOf(int c)  { return c & CAPACITY; }  // MARK: 获取工作线程数
    private static int ctlOf(int rs, int wc) { return rs | wc; }  // MARK: runState 和 workerCount 做或运算, 合成一个值
    ...
}
```
execute() 方法的实现
```
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();  // MARK: 获取包含线程数以及线程池状态的 Integer 类型数值
    if (workerCountOf(c) < corePoolSize) {  // MARK: 当工作线程数小于 corePoolSize, 增加 worker
        if (addWorker(command, true))
            return;
        c = ctl.get();  // MARK: 如果新增 worker 失败, 重新获取值, 防止外部已在线程池中加入了新任务
    }
    if (isRunning(c) && workQueue.offer(command)) {  // MARK: 如果线程池还在运行中, 则将任务放入队列
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))  // MARK: 重新检查, 如果线程池不在运行中, 也成功将任务从队列中移出, 并拒绝该任务
            reject(command);
        else if (workerCountOf(recheck) == 0)  // MARK: 如果线程池中无 worker, 则新增 worker
            addWorker(null, false);
    }
    else if (!addWorker(command, false))  // MARK: 如果核心线程和队列都已满, 尝试创建一个线程, 失败则拒绝该任务
        reject(command);
}
```
execute() 方法中有三次 addWorker, 此外发生拒绝的原因有两个: 线程池状态为非 RUNNING 状态, 或者核心线程和队列都已满, 尝试创建一个线程又失败  
以下是 addWorker() 方法源码
```
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);  // MARK: 获取线程池状态

        // Check if queue empty only if necessary.
        if (rs >= SHUTDOWN &&  // MARK: 线程池状态 > SHUTDOWN, 或者线程池状态 >= SHUTDOWN 并且 firstTask != null, 或者线程池状态 >= SHUTDOWN 并且队列为空
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))  // MARK: worker 数大于 CAPACITY, 或者大于 corePoolSize 或 maximumPoolSize
                return false;
            if (compareAndIncrementWorkerCount(c))  // MARK: 使用 CAS 自增 1, 如果成功则跳出循环
                break retry;
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)  // MARK: 线程池状态如果发生改变, 则重试循环
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }
    // MARK: 开始创建线程
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        w = new Worker(firstTask);  // MARK: 封装成 worker
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();  // MARK: 获取锁
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {  // MARK: 当线程池在运行中, 或者为 SHUTDOWN 且 firstTask 为 null
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                t.start();  // MARK: 启动线程, 注意这里并非是 execute() 方法中的 command 参数
                workerStarted = true;
            }
        }
    } finally {
        if (! workerStarted)  // MARK: 如果 worker 启动
            addWorkerFailed(w);
    }
    return workerStarted;
}
```
addWorker 时先利用 compareAndIncrementWorkerCount() 加 1, 再去创建线程, 线程创建失败再减 1 的方式, 比先创建线程, 在加 1 时检查超过了线程数限制再销毁线程的方式代价小的多  
以下是 Worker 类的实现
```
private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
    /**
     * This class will never be serialized, but we provide a
     * serialVersionUID to suppress a javac warning.
     */
    private static final long serialVersionUID = 6138294804551838833L;

    /** Thread this worker is running in.  Null if factory fails. */
    final Thread thread;
    /** Initial task to run.  Possibly null. */
    Runnable firstTask;
    /** Per-thread task counter */
    volatile long completedTasks;

    /**
     * Creates with given first task and thread from ThreadFactory.
     * @param firstTask the first task (null if none)
     */
    Worker(Runnable firstTask) {
        setState(-1); // inhibit interrupts until runWorker
        this.firstTask = firstTask;
        this.thread = getThreadFactory().newThread(this);
    }

    /** Delegates main run loop to outer runWorker  */  // MARK: thread 执行 start() 后, 执行 runWorker() 方法
    public void run() {
        runWorker(this);
    }
    ...
}
```
使用线程时需要注意以下几点
- 合理设置各类参数, 应根据实际业务场景来设置合理的工作线程数
- 线程资源必须通过线程池创建, 不允许在应用中自行显式创建线程
- 创建线程或线程池时请指定有意义的线程名称, 方便出错时回溯

线程池不允许使用 Executors, 而是通过 ThreadPoolExecutor 的方式创建, 这样的创建方式更加明确线程池的运行规则

#### ThreadLocal
ThreadLocal 初衷是在线程并发时, 解决变量共享问题, 但由于过度设计, 例如弱引用和哈希碰撞, 导致理解难度大, 使用成本高, 反而成为故障高发点, 容易出现内存泄漏, 脏数据, 共享对象更新等问题

##### 引用类型
JVM 会自动管理内存的分配与使用, 但在某些场景下, 即使引用可达, 也希望能够根据语义的强弱进行有选择的回收, 以保证系统的正常运行; 根据引用类型语义的强弱来决定垃圾回收的阶段, 可以把引用分为强引用, 软引用, 弱引用和虚引用四类; 后三类引用本质上是可以让开发工程师通过代码来决定对象的垃圾回收时机
- 强引用, 即 Strong Reference
即类似 `Object obj = new Object();`, 这样的变量声明和定义就会产生对该对象的强引用, 只要对象有强引用, 并且 GC Roots 可达, 那么在回收内存时, 即使内存耗尽也不会回收该对象
- 软引用, 即 Soft Reference
引用力度弱于强引用, 是用于非必须对象的场景, 在即将 OOM 之前, 垃圾回收器会把这些软引用指向的对象加入回收范围, 以便获得更多的内存空间
- 弱引用, 即 Weak Reference
引用比前两者更弱, 也是用来描述非必须对象的; 如果弱引用执行的对象值存在弱引用这一条线路, 则下次 YGC 时会被回收, 由于 YGC 的不确定性, 所以在 WeakReference.get() 时需要注意 NPE
- 虚引用, Phanton Reference
是一种极弱的引用关系, 定义完成后就无法通过该引用获得指向的对象; 为一个对象设置虚引用的唯一目的就是希望能在这个对象被回收时收到一个系统通知; 虚引用必须与引用队列联合使用, 当垃圾回收时, 如果发现存在虚引用, 就会在回收对象内存前, 把这个虚引用加入与之关联的引用队列中   

##### ThreadLocal 的价值
ThreadLocal 在定义时可以覆写 initialValue() 方法, 这个方法会在 ThreadLocal.get() 时执行到
```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
每个线程都有自己的 ThreadPoolMap, 如果 map == null 或者 map.get(this) == null 时会执行 setInitialValue() 方法
```
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);  // MARK: 就是获取 t.threadLocals
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```
ThreadLocal 有个静态内部类 ThreadLocalMap, ThreadLocalMap 有个静态内部类 Entry, 在 Thread 中的 ThreadLocalMap 属性的赋值是在 ThreadLocal 类中的 createMap() 中进行的; ThreadLocal 与 ThreadLocalMap 有着三组对应的方法: get(), set(), remove(), 在 ThreadLocal 中只做判断和校验, 最终的实现会落在 ThreadLocalMap 上; Entry 继承自 WeakReference, 且没有方法, 只有一个 value 成员变量, 它的 key 是 ThreadLocal 对象; 以下是从栈与堆的内存角度看两者关系
```
---------------   ---------------------------------------------------------
|             |   |  --------------------------    ----------------------  |
|    Thread   |   |  | ----------------------- |  |   (线程内成员变量)    | |
|   对象引用 =======> | | ThreadLocalMap 引用 | |  | ThreadLocalMap 对象  | |
|             |   |  | ----------------------- |  | -------------------  | |
| ThreadLocal |   |  |       当前线程对象       |  | |   Entry 对象-1   | | |
|  对象引用-1  |   |  --------------------------   | |-----------------| | |
|             |   |                               | |    弱引用Key-1   | | |
| ThreadLocal |   |  ---------------------------  | |                 |  | |
|  对象引用-1 ======> |   ThreadLocal 对象-1    | <= |     Value-1     |  | |
|             |   |  ---------------------------  | -------------------  | |
|             |   |                               |                      | |
| ThreadLocal |   |  ---------------------------  | -------------------  | |
|  对象引用-2 ======> |   ThreadLocal 对象-1    | <= |   Entry 对象-1  |  | |
|             |   |  ---------------------------  | |-----------------|  | |
| ThreadLocal |   |                               | |   弱引用Key-1   |  | |
|  对象引用-n  |   |                               | |                 |  | |
|             |   |                               | |      Value-1    |  | |
|             |   |                               | -------------------  | |
|             |   |                                                        |
---------------   ----------------------------------------------------------
```
- 1 个 Thread 有且仅有一个 ThreadLocalMap 对象
- 1 个 Entry 对象的 Key 弱引用指向一个v ThreadLocal 对象
- 1 个 ThreadLocalMap 对象存储多个 Entry 对象
- 1 个 ThreadLocal 对象可以被多个线程所共享
- ThreadLocal 不持有 Value, Value 由线程的 Entry 对象持有

所有的 Entry 对象都被 ThreadLocalMap 类实例化对象 threadLocals 持有; 当线程对象执行完毕时, 线程对象内的实例属性均会被垃圾回收; 即使线程在执行中, 只要 ThreadLocal 对象引用被置为 null, Entry 的 Key 就会自动在下一次 YGC 时被垃圾回收; 而在 ThreadLocal 使用 set() 和 get() 时, 会将 key == null 的 value 置为 null, 使得 value 能够被垃圾回收; 但是 ThreadLocal 对象通常作为私有静态变量使用, 那么生命周期不会随着线程的结束而结束, 线程使用 ThreadLocal 有三个重要方法
- set(): 如果没有 set 操作的 ThreadLocal 容易引起脏数据
- get(): 始终没有 get 操作的 ThreadLocal 对象是没有意义的
- remove(): 如果没有 remove 操作, 容易引起内存泄漏

SimpleDateFormat 是线程不安全的类, 定义为 static 对象时, 会有数据同步风险, SimpleDateFormat 内部有一个 Calender 对象, 多线程共享时有非常高的概率产生错误, 推荐解决方法就是使用 ThreadLocal

##### ThreadLocal 的副作用
- 脏数据
线程池会重用 Thread 对象, 那么与 Thread 绑定的静态属性 ThreadLocal 变量也会被重用, 如果在实现的 run() 方法中不显示的 remove() 清理与线程相关的 ThreadLocal 信息, 那么下一个线程不调用 set() 设置初始值, 则 get() 就重用了线程信息
- 内存泄漏
使用 static 修饰了 ThreadLocal, 寄希望于 ThreadLocal 对象失去引用后, 触发弱引用机制来回收 Entry 的 Value 就不现实了, 建议用完 ThreadLocal 后调用 remove() 
