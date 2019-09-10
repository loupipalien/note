### Java 并发容器和框架

#### ConcurrentHashMap 的实现原理与使用
ConcurrentHashMap 是线程安全且高效的 HashMap

##### 为什么要使用 ConcurrentHashMap
- 线程不安全的 HashMap
在多线程环境下, 使用 HashMap 进行 put 操作会引起死循环, 是因为多线程会导致 HashMap 的 Entry 链表形成环形数据结构, 一旦形成环形数据结构, Entry 的 next 节点永远不为空, 就会产生死循环获取 Entry, 导致 CPU 利用率接近 100%
- 效率低下的 HashTable
HashTable 容器使用 synchronized 来保证线程安全, 但在线程竞争激烈的情况下效率十分低下,
- ConcurrentHashMap 的锁分段技术可有效提升并发访问率
HashTable 容器在竞争激烈的并发环境下表现效率低下的原因是所有操作都要竞争同一把锁; 如果容器里有多把锁, 每一把锁用于锁容器的一部分数据, 多个线程访问不同数据段时就不存在竞争, 从而提高并发访问率, 这就是 ConcurrentHashMap 的锁分段技术

##### ConcurrentHashMap 的结构
ConcurrentHashMap 是由 Segment 数组结构和 HashEntry 数组结构组成; Segment 是一种可重入锁 (ReentrantLock), 在 ConcurrentHashMap 中扮演锁的角色; HashEntry 则用于存储键值对数据, 一个 ConcurrentHashMap 里包含一个 Segment 数组, Segment 的结构和 HashMap 类似, 是一种数组和链表结构, 一个 Segment 里包含一个 HashEntry 数组, 每个 HashEntry 是一个链表结构的元素; 每个 Segment 守护着一个 HashEntry 数组里的元素, 当 HashEntry 数组的元素进行修改时, 必须首先获得对应的 Segment 锁

##### ConcurrentHashMap 的初始化
ConcurrentHashMap 初始化方法是通过 initialCapacity, loadFactor, concurrencyLevel 等几个参数来初始化 segment 数组, 段偏移量 segmentShift, 段掩码 segmentMask 和每个 segment 里的 HashEntry 来实现的

TODO

#### ConcurrentLinkedQueue
如果要实现一个线程安全的队列有两种方式: 一种是使用阻塞算法, 另一种是使用非阻塞算法; 使用阻塞算法的队列可以用一个锁 (入队和出队使用同一把锁) 或两个锁 (入队和出队使用不同的锁) 的方式来实现; 非阻塞的实现方式则可以使用循环 CAS 的方式来实现; ConcurrentLinkedQueue 是一个基于来链接节点的无界线程安全队列, 它采用先进先出的规则对节点进行排序

TODO

#### Java 中的阻塞队列

##### 什么是阻塞队列
阻塞队列 (BlockingQueue) 是一个支持两个附加操作的队列; 这两个附加的操作支持阻塞的插入和移除方法
- 支持阻塞的插入方法: 当队列满时, 队列会阻塞插入元素的线程, 直到队列不满
- 支持阻塞的移除方法: 当队列空时, 获取元素的线程会等待队列变为非空

阻塞队列常用于生产者和消费者的场景, 生产者向队列里添加元素的线程, 消费者是从队列里取元素的线程; 阻塞队列就是生产者用来存放元素, 消费者用来获取元素的容器  
在阻塞队列不可用时, 这两个附加操作提供了四种处理方式

| 方法 / 处理方式 | 抛出异常 | 返回特殊值 | 一直阻塞 | 超时退出 |
| :--- | :--- | :--- | :--- | :--- |
| 插入方法 | add(e) | offer(e) | put(e) | offer(e, time, unit) |
| 移除方法 | remove() | poll() | take() | poll(time, unit) |
| 检查方法 | element() | peek() | 不可用 | 不可用 |

- 抛出异常: 当队列满时, 如果再往队列里插入元素, 会抛出 IllegalStateException 异常; 当队列空时, 从队列里获取元素会抛出 NoSuchElementException 异常
- 返回特殊值: 当队列插入元素时, 会返回元素是否插入成功, 成功返回 true; 如果是移除方法, 则是从队列里取出一个元素, 如果没有则返回 null
- 一直阻塞: 当阻塞队列满时, 如果生产者线程往队列里 put 元素, 队列会一直阻塞生产者线程, 直到队列可用或者响应中断退出; 当队列空时, 如果消费者线程从队列里 take 元素, 队列会阻塞住消费者线程, 直到队列不为空
- 超时退出: 当阻塞队列满时, 如果生产者线程往队列里插入元素, 队列会阻塞生产者线程一段时间, 如果超过了指定时间, 生产者线程就会退出

##### Java 里的阻塞队列
- ArrayBlockingQueue: 一个由数组结构组成的有界阻塞队列
- LinkedBlockingQueue: 一个由链表结构组成的有界阻塞队列
- PriorityBlockingQueue: 一个支持优先级排序的无界阻塞队列
- DelayQueue: 一个使用优先级队列实现的无界阻塞队列
- SynchronousQueue: 一个不存储元素的阻塞队列
- LinkedTransferQueue: 一个由链表结构组成的无界阻塞队列
- LinkedBlockingDueue: 一个由链表结构组成的双向有界阻塞队列

###### ArrayBlockingQueue
ArrayBlockingQueue 是一个用数组实现的有界阻塞队列; 此队列按照先进先出 (FIFO) 的原则对元素进行排序; 默认情况下不保证线程公平的访问队列, 所谓公平访问队列是指阻塞的线程, 可以按照阻塞的先后顺序访问队列, 即先阻塞线程访问队列; 非公平性是对先等待的线程是非公平的, 当队列可用时, 阻塞的线程都可以争夺访问队列的资格, 有可能先阻塞的线程最后才访问到队列; 为了保证公平性, 通常会降低吞吐量
###### LinkedBlockingQueue
LinkedBlockingQueue 是一个用链表实现的有界阻塞队列, 此队列的默认和最大长度为 Integer.MAX_VALUE, 按照先进先出的原则对元素进行排序

###### PriorityBlockingQueue
PriorityBlockingQueue 是一个支持优先级的无界阻塞队列; 默认情况下元素采取自然升序排列, 可以指定 Comparator 来对元素进行排序; 需要注意的是不能保证同优先级的顺序

###### DelayQueue
DelayQueue 是一个支持延时获取元素的无界阻塞队列; 队列使用 priorityQueue 来实现, 队列中的元素必须实现 Delayed 接口, 在创建元素时可以指定多久才能从队列中获取当前元素; 只有在延迟期满时才能从队列中提取元素; DelayQueue 非常用用, 可以考虑运用在以下场景
- 缓存系统的设计: 可以用 DelayQueue 保存缓存元素的有效期, 使用一个线程循环查询 DelayQueue, 一旦能从 DelayQueue 中获取到元素时, 表示缓存有效期到了
- 定时任务调度: 使用 DelayQueue 保存当天将会执行的任务和执行时间, 一旦从 DelayQueue 中获取到任务就开始执行, TimerQueue 就是使用 DelayQueue 实现的

###### SynchronousQueue
SynchronousQueue 是一个不存储元素的阻塞队列, 每个 put 操作必须等待一个 take 操作, 否则不能继续添加元素  
它支持公平访问队列, 默认情况下线程采用非公平性策略访问队列; 队列本身并不存储任何元素, 非常适合传递性场景, SynchronousQueue 的吞吐量高于 ArrayBlockingQueue 和 LinkedBlockingQueue

###### LinkedTransferQueue
LinkedTransferQueue 是一个由链表结构组成的无界阻塞 TransferQueue 队列; 相对于其他阻塞队列多了 tryTransfer 和 transfer 方法
- transfer 方法
如果当前有消费者正在等待接收元素, transfer 方法可以把生产者传入的元素立刻 transfer 给消费者; 如果没有消费者在等待接收元素, transfer 方法会将元素放在队列的 tail 节点, 等到该元素被消费者消费了才返回
- tryTransfer 方法
tryTransfer 方法是用来试探生产者传入的元素是否能直接传给消费者, 如果没有消费者等待接收元素则返回 false; 和 transfer 方法的区别是 tryTransfer 方法无论消费者是否接收都立即返回, 而 transfer 方法是必须等到消费者消费了才返回

###### LinkedBlockingDueue
LinkedBlockingDueue 是一个由链表结构组成的双向阻塞队列, 所谓双向队列指的是可以从队列的两端插入和移除元素; 双向队列因为多了一个操作队列的入口, 在多线程同时入队时, 也就减少了一半的竞争

##### 阻塞队列的实现原理
使用通知模式实现阻塞队列, 所谓通知模式就是当生产者往满的队列里添加元素时会阻塞住生产者, 当消费者消费了一个队列中的元素后, 会通知生产者当前队列可用; ArrayBlockingQueue 使用 Condition 来实现
```Java
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}

public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        enqueue(e);
    } finally {
        lock.unlock();
    }
}

private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    notEmpty.signal();
}

public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}

private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal();
    return x;
}
```
当往队列里压入一个元素时, 如果队列不可用, 那么阻塞生产这主要通过 LockSupport.park(this) 来实现
```Java
// AbstractQueuedSynchronizer
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}

// LockSupport
public static void park(Object blocker) {
    Thread t = Thread.currentThread();
    setBlocker(t, blocker);
    UNSAFE.park(false, 0L);
    setBlocker(t, null);
}
```
LockSupport.park(Object blocker) 方法中会调用 setBlocker 先保存将要阻塞的线程, 然后调用 unsafe.park 阻塞当前线程, unsafe.park 视个 native 方法
```Java
public native void park(boolean var1, long var2);
```
这个方法会阻塞当前线程, 只有以下四种情况中的一种发生时, 该方法才会返回
-  与 park 对应的 unpark 执行或已经执行时; "已经执行" 是指 unpark 先执行, 然后再执行 park 的情况
- 线程被中断时
- 等待万 time 参数指定毫秒数时
- 异常现象发生时, 这个异常现象没有任何原因

当线程被阻塞队列阻塞时, 线程会进入 WAITING(parking) 状态

#### Fork/Join 框架

##### 什么是 Fork/Join 框架
Fork/Join 框架是 Java 7 提供的一个用于并行执行任务的框架, 是把一个大任务分割成若干个小任务, 最终汇总每个小任务结果后得到大任务的框架; 其运行流程图如下
```
                             [大任务]
                                | Fork
                   ----------------------------
                   | Fork       | Fork        | Fork
               [子任务1]     [子任务2]      [子任务3]
                   | Fork       |             | Fork
              ----------        |         ----------
              |        |        |         |        |
        [子任务1.1][子任务1.2]   |    [子任务3.1][子任务3.2]
              |        |        |         |        |
              ----------        |         ----------
                  | Join        |              | Join
              [任务1结果]   [任务2结果]    [任务3结果]
                  |             |              |
                  ------------------------------
                                | Join
                           [大任务结果]
```

###### 工作窃取法算法
