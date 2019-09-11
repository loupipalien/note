### Java 中的线程池
在 Java 中几乎异步和并发执行任务的程序都可以使用线程池, 合理使用线程池可以有以下好处
- 降低资源消耗: 通过重复利用已创建的线程降低线程创建和销毁造成的消耗
- 提高响应速度: 当任务到达时, 任务可以不需要等到线程创建就能立即执行
- 提高线程的可管理性: 线程是稀缺资源, 如果无限制地创建, 不仅会消耗资源还会降低资源的稳定性, 使用线程池可以进行统一分配, 调优, 监控

#### 线程池的实现原理
ThreadPoolExecutor 执行 execute() 方法的示意图
```
         --------
使用者:  |提交任务|
         --------
            |
            v
         -----------------  是  ------------  是 --------------  是 -------------------------
线程者:  |核心线程池是否已满| ->  |队列是否已满| -> |线程池是否已满| -> |按照策略处理无法执行的任务|
         -----------------      ------------     --------------     -------------------------
                | 否                  | 否
                v                    v
         -----------------      -----------------
        | 创建线程执行任务 |     |将任务存储在队列里|
         -----------------      -----------------
```
从图中可以看出, 当提交一个新任务到线程池时, 线程池的处理流程如下
- 线程池判断核心线程池里的线程是否都在执行任务, 如果不是则创建一个新的工作线程来执行任务; 如果核心线程池里的线程都在执行任务则进入下一个流程
- 线程池判断工作队列是否已经满了, 如果工作队列没有满, 则将新提交的任务存储在这个工作队列里; 如果工作队列满了则进入下个流程
- 线程池判断线程池的线程是否都处于工作状态, 如果没有则创建一个新的工作线程来执行任务; 如果已经满了则交给饱和策略来处理这个任务

ThreadPoolExecutor 执行 execute() 方法示意图如下
![ThreadPoolExecuto执行示意图.png](http://ww1.sinaimg.cn/large/d8f31fa4gy1g6w5y6nreaj20hw0dq0th.jpg)  
- 如果当运行的线程少于 corePoolSize, 则创建新线程来执行任务 (执行这一步需要获取全局锁)
- 如果运行的线程等于或多于 corePoolSize, 则将任务加入 BlockingQueue
- 如果无法将任务加入 BlockingQueue (队列已满), 则创建新的线程来处理任务 (执行这一步需要获取全局锁)
- 如果创建新线程将使当前运行的线程超过 maximumPoolSize, 任务将被拒接, 并调用 RejectedeExecutionHandler.rejectedExecution() 方法

ThreadPoolExecutor 采取上述步骤, 是为了在执行 execute() 方法时, 尽可能地避免获取全局锁 (是一个严重的伸缩瓶颈); 在 ThreadPoolExecutor 在完成预热之后 (当前运行线程树大于等于 corePoolSize), 几乎所有的 execute() 方法调用都是执行步骤 2, 因为不用获取全局锁

#### 线程池的使用

##### 线程池的创建
```
new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, threadFactory, handler);
```
- corePoolSize (线程池的基本大小) : 当提交一个任务到线程池时, 线程池会创建一个线程来执行任务, 即使其他空闲的基本线程能够执行新任务也会创建线程, 等到需要执行的任务数大于线程池基本大小时就不再创建; 如果调用了线程池的 prestartAllCoreThreads() 方法, 线程池就会提前创建并启动所有基本线程
- maximumPoolSize (线程池最大数量) : 线程池允许创建的最大线程数; 如果队列满了, 并且已创建的线程数小于最大线程数, 则线程池会再创建新的线程执行任务; **如果使用了无界的任务队列这个参数就没什么效果**
- keepAliveTime (线程活动保持时间) : 线程池的工作线程空闲后, 保持存活的时间; 如果任务很多并且执行时间很短, 可以考虑调大时间来提高线程的复用率
- unit (线程活动保持时间的单位) : DAYS, HOURS, MINUTES, MILLISECONDS, MICROSECONDS, NANOSECONDS
- workQueue (工作队列) : 用于保存等待执行工作的阻塞队列, 可以选择以下队列
  - ArrayBlockingQueue: 是一个基于数组结构的有界阻塞队列, 此队列按 FIFO 原则对元素排序
  - LinkedBlockingQueue: 是一个基于链表结构的阻塞队列, 此队列按 FIFO 原则对元素排序, 吞吐率通常高于 ArrayBlockingQueue; Executors.newFixedThreadPool() 使用这个队列
  - SynchronousQueue: 一个不存储元素的阻塞队列; 每个插入操作必须等到另一个线程调用移除操作, 否则插入一直处于阻塞状态, 吞吐率通常高于 LinkedBlockingQueue; Executors.newCachedThreadPool() 使用这个队列
  - PriorityBlockingQueue: 一个具有优先级的无限阻塞队列
- maximumPoolSize (线程池最大数量): 线程池允许创建的最大线程数, 如果队列满了, 并且已创建的线程数小于最大线程数, 则线程池会再创建先的线程执行任务; 值得注意的是, 如果使用了无界的任务队列则这个参数无效果  
- threadFactory: 用于设置创建线程的工厂, 可以通过线程工厂给每个创建出的线程设置有意义的名字
- handler (饱和策略) : 当队列和线程池都满了, 说明线程池处于饱和状态, 那么必须采用一种策略处理提交的新任务, 默认是 AbortPolicy
  - AbortPolicy: 直接抛出异常
  - CallerRunsPolicy: 只用调用者所在线程来运行
  - DiscardOldestPolicy: 丢弃队列里最近的一个任务, 并执行当前任务
  - DiscardPolicy: 不处理, 丢弃掉
- keepAliveTime (线程活动保持时间): 线程池的工作空闲后, 保持存活的时间 
- TimeUnit (线程获得保持时间单位): 天, 小时, 分钟, 毫秒, 微秒, 纳秒  
##### 向线程池提交任务
execute() 方法用于提交不需要返回值的任务; submit() 方法用于提交需要返回值的任务

##### 关闭线程池
可以通过调用线程池的 shutdown() 和 shutdownNow() 方法来关闭线程池; 它们的原理是遍历线程池中的线程, 然后逐个调用线程的 interrupt() 方法, 所以无法响应中断的任务可能永远都无法停止; 但这两个方法也是有区别的: shutdownNow() 首先将线程池的状态设置为 STOP, 然后尝试停止所有的正在执行或暂停任务的线程, 并返回等待执行的任务列表; shutdown() 只是将线程池的状态设置为 SHUTDOWN 状态, 然后中断没有执行任务的线程; 只要调用了这两个方法中的任意一个, isShutdown() 方法就会返回 true; 当所有的任务都关闭后, 才表示线程池关闭成功, 这时调用 isTerminated() 方法会返回 true

##### 合理的配置线程池
TODO

##### 线程池的监控
TODO
