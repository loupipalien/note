### Java 中的锁

#### Lock 接口
锁是用来控制多个线程访问共享资源的方式, 一般来说一个锁能够防止多个线程同时访问共享资源 (有些锁可以允许多个线程并发的访问共享资源, 比如读写锁); 在 Lock 接口出现之前, Java 程序都是靠 synchronized 关键字实现锁的功能, JDK 5 之后, 并发包中新增了 Lock 接口 (以及相关实现类) 用来实现锁功能, 它提供了与 synchronized 关键字相同的同步功能, 只是在使用时需要显示的获取和释放锁; 虽然 Lock 接口缺少了隐式获取和释放锁的便捷性, 但是拥有了锁获取和释放的可操作性, 可中断的获取锁以及超时获取锁等多种 synchronized 关键字所不具备的同步特性; Lock 锁的使用也很简单
```
Lock lock = new ReentrantLock();
lock.lock();
try {
    ...
} finally {
    lock.unlock();
}
```
在 finally 块中释放锁, 目的是保证在获取到锁之后, 最终能够被释放; 不要将获取锁的过程写在 try 块中, 因为如果在获取锁 (定义锁的实现) 时发生了异常, 异常抛出的同时, 也会导致锁无故释放; Lock 接口提供 synchronized 关键字不具备的主要特征

| 特性 | 描述 |
| :------------- | :------------- |
| 尝试非阻塞地获取锁 | 当前线程尝试获取锁, 如果这一时刻没有被其他线程获取到, 则成功获取并持有锁 |
| 能被中断地获取锁 | 与 synchronized 不同, 获取到锁的线程能够响应中断, 当获取到锁的线程被中断时, 中断异常将会被抛出, 同时锁会被释放 |
| 超时获取锁 | 在指定的截止时间之前获取锁, 如果截止时间到了仍未获取锁, 则返回 |

Lock 是一个接口, 定义了锁获取和释放的基本操作

| 方法名称 | 描述 |
| :------------- | :------------- |
| void lock() | 获取锁, 调用该方法当前线程将会获取锁, 当锁获得后, 从该方法返回 |
| void lockInterruptibly() throws InterruptedException | 可中断地获取锁, 和 lock() 方法的不同之处在于, 该方法会响应中断, 即在锁获取中可以中断该线程 |
| boolean tryLock() | 尝试非阻塞的获取锁, 调用该方法后立刻返回, 如果能够获取则返回 true, 否则则返回 false |
| boolean tryLock(long time, TimeUnit unit) throws InterruptedException | 超时的获取锁, 当前线程在以下三种情况下会返回: 当前线程在超时时间内获得了锁; 当前线程在超时时间内被中断; 超时时间结束, 返回 false |
| void unlock() | 释放锁 |
| Condition newCondition() | 获取等待通知组件, 该组件和当前的锁绑定, 当前线程只有获得了锁, 才能调用该组件的 wait() 方法, 而调用后, 当前线程将释放锁 |

Lock 接口的实现基本上都是通过聚合了一个同步器的子类来完成线程访问控制的

#### 队列同步器
队列同步器 AbastactQueuedSynchronizer 是用来构建锁或者其他同步组件的基础框架, 它使用了一个 int 成员变量表示同步状态, 通过内置的 FIFO 队列来完成资源获取线程的排队工作  
同步器的主要使用方式是继承, 子类通过继承同步器并实现它的抽象方法来管理同步状态, 同步器提供了 3 个方法 (`getState(), setState(int newState), compareAndSetState(int expect, int update)`) 进行操作; 子类推荐被定义为同步组件的静态内部类, 同步器本身没有实现任何同步接口, 它仅仅是定义了若干同步状态获取和释放的方法来供自定义同步组件使用, 同步器既可以支持独占式地获取同步状态, 也可以共享式地获取同步状态, 可以方便实现不同类型的同步组件 (`ReentrantLock, ReentrantReadWriteLock, CountDownLatch`)  
同步器是实现锁 (任意同步组件) 的关键, 在锁的实现中聚合同步器, 利用同步器实现锁的语义; 可以这样理解二者之间的关系: 锁是面向使用者的, 它定义了使用者与锁交互的接口, 隐藏了实现细节; 同步器面向的是锁的实现者, 它简化了锁的实现方式, 屏蔽了同步状态管理, 线程排队, 等待和唤醒等底层操作; 锁和同步器很好的隔离了使用者和实现者所需关注的领域  

##### 队列同步器的接口与示例
同步器的设计是基于模板方法模式的; 重写同步器指定的方法, 需要使用同步器提供的如下三个方法
- getState(): 获取当前同步状态
- setState(int newState): 设置当前同步状态
- compareAndSetState(int expect, int update): 使用 CAS 设置当前状态, 该方法能够保证状态设置的原子性

同步器可重写的方法如下

| 方法名称 | 描述 |
| :------------- | :------------- |
| protected boolean tryAcquire(int arg) | 独占式获取同步状态, 实现该方法需要查询当前状态并判断同步状态是否符合预期, 然后再进行 CAS 设置同步状态|
| protected boolean tryRelease(int arg) | 独占式释放同步状态, 等待获取同步状态的线程有机会获取同步状态 |
| protected int tryAcquireShared(int arg) | 共享式获取同步状态, 返回大于等于 0 的值, 表示成功, 反之表示失败 |
| protected boolean tryReleaseShared(int arg)| 共享式释放同步状态 |
| protected boolean isHeldExclusively() | 当前同步器是否在独占模式下被线程占用, 一般该方法表示是否被当前线程所占用 |

实现自定义同步组件时, 会调用同步器提供的模板方法

| 方法名称 | 描述 |
| :------------- | :------------- |
| void acquire(int arg) | 独占式获取同步状态, 如果当前线程获取同步状态成功, 则由该方法返回; 否则将会进入等待队列, 该方法将会调用 tryAcquire(int arg) 方法|
| void acquireInterruptibly(int arg) | 与 acquire(int arg) 相同, 但是该方法响应中断, 当前线程未获取到同步状态而进入同步队列中等待, 如果当前线程被中断, 则该方法会抛出 InterruptedException 并返回 |
| boolean tryAcquireNanos(int arg, long nanos) | 在 acquireInterruptibly(int arg) 基础上增加超时限制, 如果当前线程在超时时间内没有获取到同步状态, 那么将会返回 false, 如果获取到返回 true |
| void acquireShared(int arg) | 共享式的获取同步状态, 如果当前线程未获取到同步状态, 将会进入同步队列等待, 与独占式获取的主要区别是在同一时刻可以有多个状态获取到同步状态 |
| void acquireSharedInterruptibly(int arg) | 与 acquireShared(int arg) 相同, 该方法响应中断 |
| boolean  tryAcquireSharedNanos(int arg, long nanos) | 在 acquireInterruptibly(int arg) 基础之上增加了超时限制 |
| boolean release(int arg) | 独占式的释放同步状态, 该方法会在释放同步状态之后, 将同步队列中第一个节点包含的线程唤醒 |
| boolean releaseShared(int arg) | 共享式的释放同步状态 |
| Collection<Thread> getQueuedThreads | 获取等待在同步队列上的线程集合 |

同步器提供的模板方法基本分三种: 独占式获取和释放同步状态, 共享式获取和释放同步状态, 查询同步队列中的等待线程情况  
独占锁就是在同一时刻只能有一个线程获取到锁, 其他获取锁的线程只能处于同步队列中等待, 只有获取锁的线程释放了锁, 后继线程才能获取锁; 以下 Mutex 是一个独占式的自定义同步组件, 同一时刻只允许一个线程占有锁
```Java
public class Mutex implements Lock {

    private static class Sync extends AbstractQueuedSynchronizer {
        // 是否处于占用状态
        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        // 当状态为 0 的时候获取锁
        @Override
        protected boolean tryAcquire(int arg) {
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        // 释放锁, 将状态设置为 0
        @Override
        protected boolean tryRelease(int arg) {
            if (getState() == 0) {
                throw new IllegalMonitorStateException();
            }
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        // 返回一个 Condition, 每个 Condition 都包含了一个 condition 队列
        Condition newCondition() {
            return new ConditionObject();
        }
    }

    // 其他则仅需要将操作代理到 sync 上即可
    private final Sync sync = new Sync();

    @Override
    public void lock() {
        sync.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.tryRelease(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```
上述示例中, 独占锁 Mutex 是一个自定义同步组件, 在同一时刻只允许一个线程占有锁

##### 队列同步器的实现分析

###### 同步队列
同步器依赖内部的同步队列 (一个 FIFO 双向队列) 来完成同步状态的管理, 当前线程获取同步状态失败时, 同步器会将当前线程以及等待状态等信息构造成一个节点 (Node) 并将其加入同步队列, 同时会阻塞当前线程, 当同步状态释放时, 会把首节点中的线程唤醒, 使其再次尝试获取同步状态  
同步队列中的节点 (Node) 用来保存获取同步状态失败的线程引用, 等待状态以及前驱和后继节点, 节点属性类型以及名称等

| 属性类型与名称 | 描述 |
| :------------- | :------------- |
| int waitStatus | 等待状态, 包括如下状态: CANCLELED, 值为 1, 由于在同步队列中等待的线程等待超时或者被中断, 需要从同步队列中取消等待, 节点进入该状态将不会变化; SIGNAL, 值为 -1, 后继节点的线程处于等待状态, 而当前节点的线程如果释放了同步状态或者被取消, 将会通知后继节点, 使后继节点的线程得以运行; CONDITION, 值为 -2, 节点在等待队列中, 节点线程等待在 Condition 上, 当其他线程对 Condition 调用 signal() 方法后, 该节点会从等待队列中转移到同步队列中, 加入到对同步状态的获取中; PROPAGATE, 值为 -3, 表示下一次共享式同步状态获取将会无条件的被传播下去; INITIAL, 值为 0, 初始状态 |
| Node pre | 前驱节点, 当节点加入同步队列时被设置 (尾部添加) |
| Node next | 后继节点 |
| Node nextWaiter | 等待队列中的后继节点; 如果当前节点是共享的, 那么这个字段将是一个 SHARED 常量, 就是说节点类型 (独占和共享) 和等待队列中的后继节点共用同一个字段 |
| Thread thread | 获取同步状态的线程 |

同步器拥有头节点 (head) 和尾节点 (tail), 没有成功获取同步状态的线程将会成为接待你加入该队列的尾部
```
---------  setHead(Node update)
| 同步器 |      -------       -------       -------       -------
|       |      | 节点 |      | 节点 |       | 节点 |      | 节点 |
| head  | ---> | prev | <--- | prev | <--- | prev | <--- | prev |   
| tail  |      | next | ---> | next | ---> | next | ---> | next |
---------      -------       -------       -------       --------
    |     compareAndSetTail(Node expect, Node update)       ^
    --------------------------------------------------------|
```
当一个线程成功获取了同步状态, 其他线程将无法获取到同步状态, 因此被构造成节点并加入到同步队列, 加入队列的过程要保证线程安全, 同步器提供了一个基于 CAS 的设置尾节点的方法: compareAndSetTail(Node expect, Node update)
```
---------
| 同步器 |      -------       -------       -------      ========
|       |      | 节点 |      | 节点 |       | 节点 |      | 节点 |
| head  | ---> | prev | <--- | prev | <--- | prev | <--- | prev |   
| tail  |      | next | ---> | next | ---> | next | ---> | next |
---------      -------       -------       -------       ========
    |                   CAS 设置尾节点                       ^
    --------------------------------------------------------|
```
同步队列遵顼 FIFO, 头节点是获取同步状态成功的节点, 头节点的线程在释放同步状态时, 将会唤醒后继节点, 而后继节点将会在获取同步状态成功时将自己设置为首节点
```
              设置首节点
---------   --------------
| 同步器 |  |   =======   |   -------       -------      --------
|       |  |   | 节点 |   |  | 节点 |       | 节点 |      | 节点 |
| head  | --   | prev |   -> | prev | <--- | prev | <--- | prev |   
| tail  |      | next |      | next | ---> | next | ---> | next |
---------      ========      -------       -------       --------
    |                                                       ^
    --------------------------------------------------------|
```
设置头节点是通过获取同步状态成功的线程来完成的, 由于只有一个线程能够成功获取到同步的状态, 所以设置头节点不需要使用 CAS 来保证, 只需要将头节点设置为原头节点的后继节点并断开原头节点的 next 引用即可

###### 独占式同步状态获取与释放
通过调用同步器的 acquire(int arg) 方法可以获取同步状态, 该方法对中断不敏感, 线程获取同步方法失败后进入同步队列中, 后继对线程进行中断操作时, 线程不会从同步队列中移出
```
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
以上代码主要完成了同步状态获取, 节点构造, 加入同步队列以及在同步队列中自旋等待等工作; 其主要逻辑是: 首先调用自定义同步器实现 `tryAcquire(int arg)` 方法, 该方法保证线程安全的获取同步状态, 如果同步状态获取失败, 则构造同步节点 (独占式 `Node.EXCLUSIVE`, 同一时刻只能有一个线程获取同步状态) 并通过 `addWaiter(Node node)` 方法将该节点加入到同步队列的尾部, 最后调用 `acquireQueued(Node node, int arg)` 方法, 使得该节点以 "死循环" 的方式获取同步状态; 如果获取步到则阻塞节点中的线程, 而被阻塞线程的唤醒主要依靠前驱节点的出队或阻塞线程被中断来实现  

**首先是节点构造以及加入同步队列**
```Java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {                     
            pred.next = node;
            return node;
        }
    }
    enq(node);                                      
    return node;
}
...
private Node enq(final Node node) {
    for (;;) {                                                              
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```
在 enq(final Node node) 方法中, 同步器通过 "死循环" 来保证节点的正确添加, 在 "死循环" 中只有通过 CAS 将节点设置为尾节点之后, 当前线程才能从该方法返回, 否则将不断的尝试设置; 可以看到, enq(final Node node) 方法将并发添加节点的请求通过 CAS 变得 "串行化" 了   

**接下来是自旋过程**
节点进入同步队列之后, 就进入到了一个自旋的过程, 每个节点 (或者说每个线程) 都在自省观察, 当条件满足, 获取到同步状态, 就可以从这个自旋过程中退出, 否则依旧留在这个自旋过程中 (并且会阻塞节点的线程)
```
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();                          
            if (p == head && tryAcquire(arg)) {                        
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
acquireQueued(final Node node, int arg) 方法中, 当前线程 "死循环" 中尝试获取同步的状态, 只有前驱节点是头节点时才能够尝试获取同步状态, 原因如下
- 头节点是成功获取到同步状态的节点, 而头节点的线程释放了同步状态之后, 将会唤醒其后继节点, 后继节点的线程被唤醒后需要检查自己的前驱节点是否是头节点
- 维护同步队列中的 FIFO 原则
```
---------   头节点拥有同步状态  |---v         |---v         |---v         |---v (自旋)
| 同步器 |      --------      -------       -------       -------       -------
|       |      |头节点 |      | 节点 |      | 节点 |      |  节点 |      |尾节点|
| head  | ---> | prev | <--- | prev | <--- | prev | <--- | prev | <--- | prev |   
| tail  |      | next | ---> | next | ---> | next | ---> | next | ---> | next |
---------      --------      -------       -------       -------       --------
    |                                                                      ^
    -----------------------------------------------------------------------|
```
非头节点线程前驱节点出队或者被中断而从等待状态返回, 随后检查自己的前驱是否是头节点, 如果是则尝试获取同步状态; 可以看到节点之间在循环检查的过程中基本不相互通信, 而是简单的判断自己的前驱是否为头节点, 这样就使得节点释放规则符合 FIFO, 并且也便于过早通知的处理(指前驱节点不是头节点的线程由于中断被唤醒)

**独占式同步状态获取流程 (acquire(int arg))**
![]()  
前驱节点为头节点并且能够获取同步状态的判断条件, 线程进入等待状态是获取同步状态的自旋过程; 当同步状态获取成功之后, 当前线程从 `acquire(int arg)` 方法返回, 表示着当前线程获取了锁  

**释放同步状态**
通过调用同步器的 `release(int arg)` 方法可以释放同步状态, 该方法在释放了同步状态之后, 会唤醒其后继节点 (进而使后继节点尝试获取同步状态)
```Java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```
该方法执行会唤醒头节点的后继节点线程, `unparkSuccessor(Node node)` 使用 LockSupport 来唤醒处于等待状态的线程  
独占式同步状态获取和释放过程总结: 在获取同步状态时, 同步其维护一个同步队列, 获取状态失败的线程都会被加入到队列中进行自旋; 移出队列 (或停止自旋) 的前提是前驱节点为头节点且成功获取了同步状态; 在释放同步队列时, 同步器调用 `tryRelease(int arg)` 方法释放同步状态, 然后唤醒头节点的后继节点

###### 共享式同步状态获取与释放
共享式获取与独占式获取最主要的区别在于同一时刻能否有多个线程同时获取到同步状态; 下图为共享式与独占式访问资源的对比
```
      ------                   -------
 共享 |     |              独占 |     |
------>     |             ------>    |
 共享 |     |              共享 |     |
------>     |             ---->|     |
 共享 | 资源|              共享 | 资源 |
------>     |             ---=>|     |
 独占 |     |              共享 |     |
---->|     |              ---->|     |
     -------                   ------
```
左半部分, 共享式访问资源时, 其他共享式的访问均被允许; 而独占式访问被阻塞, 右半部分是独占式访问资源时, 同一时刻其他访问被阻塞   
通过调用同步器的 `acquireShared(int arg)` 方法而可以共享式地获取同步状态
```Java
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
在 `acquireShared(int arg)` 方法中, 同步器调用 `tryAcquireShared(int arg)` 方法尝试获取同步状态, `tryAcquireShared(int arg)` 方法返回值为 int 类型, 当返回值大于等于 0 时, 表示能够获取到同步状态; 因此在共享式获取的自旋过程中, 成功获取到同步状态并退出自旋的条件就是 `tryAcquireShared(int arg)` 方法返回值大于等于 0; 可以看到, 在 `doAcquireShared(int arg)` 方法的自旋过程中, 如果当前节点的前驱头节点时, 尝试获取同步状态, 如果返回值大于等于 0, 表示该次获取同步状态成功并从自旋过程中退出  
与独占式一样, 共享式获取也需要释放同步状态, 通过调用 `releaseShared(int arg)` 方法可以释放同步状态
```Java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```
该方法在释放同步状态之后, 将会唤醒后续处于等待状态的节点; 对于能够支持多个线程同时访问的并发组件 (如 Semaphore), 它和独占式主要区别在于 `tryReleaseShared(int arg)` 方法确保同步状态 (或资源数) 线程安全释放一般是通过循环和 CAS 来保证的, 因为释放同步状态的操作会同时来自于多个线程

###### 独占式超时获取同步状态
通过调用同步器的 `doAcquireNanos(int arg, long nanosTimeout)` 方法可以超时获取同步状态, 即在指定时间段内获取同步状态, 如果获取到同步状态则返回 true, 否则返回 false; 该方法提供了 synchronized 关键字不具备的特性  
在 Java 5 之前, 在响应中断的同步状态获取过程中, 当一个线程获取不到锁而被阻塞在 synchronized 之外时, 对该线程进行中断操作, 此时该线程的中断标志会被修改, 但线程依旧会阻塞在 synchronized 上, 等待着获取锁; 在 Java 5 之后, 同步器提供了 `acquireInterruptibly(int arg)` 方法, 这个方法在等待获取同步状态时, 如果当前线程被中断, 会立刻返回, 并抛出 `InterruptedException`  
超时获取同步状态过程可以被视作响应中断获取同步状态过程的 "增强版", `doAcquireNanos(int arg, long nanosTimeout)` 方法在支持响应中断的基础上, 增加了超时获取的特性; 该方法代码如下
```Java
private boolean doAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;   
                return true;
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
该方法在自旋过程中, 当节点的前驱节点为头节点时尝试获取同步状态, 如果获取成功则从该方法返回, 这个过程和独占式同步获取的过程类似, 只是在同步状态获取失败的处理上有所不同; 如果当前线程获取同步状态失败, 则判断是否超时, 如果没有超时则重新计算超时间隔, 直到 nanosTimeout 小于等于零时返回; 独占式超时获取同步状态的流程图如下所示  
![独占式超时获取同步状态的流程.png](http://ww1.sinaimg.cn/large/d8f31fa4gy1g6pb4fl3ttj20i00mjdh3.jpg)

##### 自定义同步组件 --- TwinsLock
设计一个同步工具: 该工具在同一时刻, 只允许至多两个线程同时访问, 超过两个线程的访问将被阻塞, 将其命名为 TwinsLock  
分析可知此工具支持共享式访问, 所以必须重写 `acquireShared(int arg)` 等与 Shared 相关的方法, 其次至多两个线程同时访问, 表明同步资源数为 2, 即设置初始状态 status 为 2; 当资源都被获取, 此时再有新的线程获取同步状态则只能被阻塞, 在同步状态变更时, 需要使用 `compareAndSet(int expect, int update)` 方法做原子性保障
```Java
class TwinsLock implements Lock {

    public static void main(String[] args) throws InterruptedException {
        final Lock lock = new TwinsLock();
        class Worker extends Thread {
            @Override
            public void run() {
                while (true) {
                    lock.lock();
                    try {
                        TimeUnit.SECONDS.sleep(1);
                        System.out.println(Thread.currentThread().getName());
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            }
        }

        // 启动 10 个线程
        for (int i = 0; i < 10; i++) {
            Worker worker = new Worker();
            worker.setDaemon(true);
            worker.start();
        }

        // 每隔 1 秒换行
        for (int i = 0; i < 100; i++) {
            TimeUnit.SECONDS.sleep(1);
            System.out.println();
        }
    }

    private final Sync sync = new Sync(2);

    @Override
    public void lock() {
        sync.acquireShared(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.tryAcquireShared(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquireShared(1) >= 0;
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.releaseShared(1);
    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }

    private static final class Sync extends AbstractQueuedSynchronizer {

        Sync(int count) {
            if (count <= 0) {
                throw new IllegalArgumentException("The count must large than zero.");
            }
            setState(count);
        }

        @Override
        protected int tryAcquireShared(int reduceCount) {
            for (;;) {
                int current = getState();
                int newCount = current - reduceCount;
                if (newCount < 0 || compareAndSetState(current, newCount)) {
                    return newCount;
                }
            }
        }

        @Override
        protected boolean tryReleaseShared(int returnCount) {
            for (;;) {
                int current = getState();
                int newCount = current + returnCount;
                if (compareAndSetState(current, newCount)) {
                    return true;
                }
            }
        }

        Condition newCondition() {
            return new ConditionObject();
        }
    }
}
```
可以看到, 同步器作为一个桥梁, 连接线程访问以及同步状态控制等底层技术与不同并发组件 (如 Lock, CountDownLatch 等)

#### 重入锁
ReentrantLock 就是支持重进入的锁, 它表示该锁能够支持一个线程对资源的重复加锁; 除此之外, 该锁还支持获取锁时的公平和非公平性的选择  
之前的 Mutex 的 lock() 方法获取锁之后, 如果再次调用 lock() 方法, 则该线程将会被自己所阻塞, 原因是 Mutex 在实现 `tryAcquire(int arg)` 方法时没有考虑占有锁 的线程再次获取锁的场景; 简单的说就是 Mutex 是一个不支持重进入的锁; 而 synchronized 关键字隐式的支持重进入, 比如一个 synchronized 修饰的递归方法, 在方法执行时执行线程在获取了锁之后仍能连续多次的获得该锁, 而不像 Mutex 由于获得了锁, 而在下一次获取锁时阻塞自己的情况  
ReentrantLock 虽然没能像 synchronized 关键字一样支持隐式的重进入, 但是在调用 lock() 方法时, 已经获取的锁的线程, 能够再次调用 lock() 方法获取锁而不被阻塞  
对于公平性, 先对锁进行获取的请求一定先被满足, 那么这个锁是公平的, 反之是不公平的; 然而公平锁机制的效率往往没有非公平锁的效率高, 但公平锁能够减少 "饥饿" 发生的概率; ReentrantLock 提供了一个构造函数, 能够控制锁是否公平
##### 实现重进入
重进入是值任意线程在获取到锁之后能够再次获取该锁而不会被锁所阻塞, 该特性的实现需要解决以下两个问题
- 线程再次获取锁: 所需要去识别获取锁的线程是否为当前占据锁的线程, 如果是则再次成功获取
- 锁的最终释放: 线程重复 n 次获取了锁, 随后在第 n 次释放锁之后, 其他线程应该能够获取到该锁; 锁的最终释放要求锁对于获取进行计数自增, 对于释放进行计数自减, 当计数为 0 时表示锁已经成功释放  

ReentrantLock 默认的 (非公平) 实现如下
```Java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
该方法增加了再次获取同步状态的处理逻辑: 通过判断当前线程是否为获取锁的线程来决定获取操作是否成功, 如果是获取锁的线程再次请求, 则将同步状态值进行增加并返回 true; 成功获取锁的线程再次获取锁时只是增加了同步状态值, 这也要求 ReentrantLock 在释放同步状态时减少同步状态值
```Java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```
如果该锁被获取了 n 次, 那么前 n - 1 次 `tryRelease(int releases)` 方法必须返回 false

##### 公平锁与非公平锁的区别
公平性与否是针对获取锁而言的, 如果一个锁是公平的, 那么锁的获取顺序就应该符合 FIFO; 对于非公平锁, 只要 CAS 设置同步状态成功, 则表示当前线程获取了锁, 而公平锁不同
```Java
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
与非公平锁唯一的不同是判断条件多了 `hasQueuedPredecessors()` 方法, 即使用了队列来维护线程获取锁顺序的 FIFO

#### 读写锁
之前提到的锁 (Mutex 和 ReentrantLock) 基本都是排他锁, 这些锁在同一时刻只允许一个线程进行访问; 读写锁维护了一对锁, 一个读锁和一个写锁, 通过分离读锁和写锁, 使得并发性相比一般的排他锁有了很大提升  
一般情况下, 读写锁的性能都比排他锁好, 因为大多数场景读是多于写的; 在读多于写的情况下, 读写锁能提供比排他锁更好的并发性和吞吐量; ReentrantReadWriteLock 提供的特性如下

| 特性 | 说明 |
| :--- | :--- |
| 公平性选择 | 支持非公平 (默认) 和公平的锁的提取方式, 吞吐量是非公平优于公平 |
| 重进入 | 该锁支持重进入, 以读写线程为例: 读线程在获取了读锁之后, 能够再次获取读锁; 而写线程在获取了写锁之后能够再次获取写锁, 同时也可以获取读锁 |
| 锁降级 | 遵循获取写锁, 获取读锁再释放写锁的次序, 写锁能够降级成为读锁 |

##### 读写锁的接口与示例
ReadWriteLock 仅定义了获取读锁和写锁的两个方法, 而其实现 ReentrantReadWriteLock 除了接口方法外, 还提供了一些便于外界监控其内部工作状态的方法

| 方法名称 | 描述 |
| :--- | :--- |
| int getReadLockCount() | 返回当前读锁被获取的次数, 该次数不等于获取读锁的线程数, 例如仅一个线程, 它连续重入了 n 次读锁, 那么占据读锁的线程数是 1, 但该方法返回 n |
| int getReadHoldCount() | 返回当前线程获取读锁的次数 |
| boolean isWriteLocked() | 判断写锁是否被获取 |
| int getWriteHoldCount() | 返回当前写锁被获取的次数 |

以下是一个缓存示例, 展示了读写锁的使用方式
```Java
public class Cache {
    static Map<String,Object> map = new HashMap<>();
    static ReadWriteLock rwl = new ReentrantReadWriteLock();
    static Lock rl = rwl.readLock();
    static Lock wl = rwl.writeLock();

    // 获取一个 key 对应的 value
    public static final Object get(String key) {
        rl.lock();
        try {
            return map.get(key);
        } finally {
             rl.unlock();
        }
    }

    // 设置 key 对应的 value, 并返回旧的 value
    public static final Object put(String key, Object value) {
        wl.lock();
        try {
            return map.put(key, value);
        } finally {
            wl.unlock();
        }
    }

    public static final void clear() {
        wl.lock();
        try {
            map.clear();
        } finally {
            wl.unlock();
        }
    }
}
```

##### 读写锁的实现分析
ReentrantReadWriteLock 的实现主要包括: 读写状态的设计, 写锁的获取与释放, 读锁的获取与释放以及锁降级

###### 读写状态的设计
读写锁同样依赖自定义同步器来实现同步功能, 而读写状态就是其同步器的同步状态, 在 ReentrantLock 中自定义同步器的实现, 同步状态表示锁被一个线程重复获取的次数, e而读写锁的自定义同步器需要在同步状态 (一个整型变量) 上维护多个读线程和一个写线程的状态, 使得该状态的设计成为读写锁的实现关键  
如果在一个整型变量上维护多种状态, 就一定需要 "按位切割使用" 这个变量, 读写锁将变量切分成了两个部分, 高 16 位表示读, 低 16 位表示写, 划分方式如下图所示
```Java
32 位
[0][0][0][0][0][0][0][0][0][0][0][0][0][0][1][0][0][0][0][0][0][0][0][0][0][0][0][0][0][0][1][1]
                                                16 位
                                                [0][0][0][0][0][0][0][0][0][0][0][0][0][0][1][1] 写状态 = 3
16 位                                
[0][0][0][0][0][0][0][0][0][0][0][0][0][0][1][0] 读状态 = 2
```
当前同步状态表示了一个线程已经获取了写锁, 并且重入了两次, 同时也连续获取了两次写锁; 当 S 不等于 0 时, 当写状态 (S & 0x0000FFFF) 等于 0 时, 则读状态 (S >>> 16) 大于 0, 即读锁已被获取

###### 写锁的获取与释放
写锁是一个支持重进入的排它锁; 如果当前线程已经获取了写锁, 则增加写状态, 如果当前线程在获取写锁时, 读锁已经被获取 (读状态不为 0) 或者该线程不是已经获取写锁的线程, 则当前线程进入等待状态; 获取**写锁**的代码如下
```Java
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```
该方法除了重入条件 (当前线程为获取了写锁的线程) 之外, 增加了一个读锁是否存在的判断, 如果存在读锁, 则写锁不能被获取, 原因在于: 读写锁要确保写锁的操作对读锁可见, 如果允许读锁在已被获取的情况下获取写锁, 那么正在运行的其他读线程就无法感知到当前写线程的操作; 写锁的释放与 ReentrantLock 的释放过程基本类似

###### 读锁的获取与释放
读锁是一个支持重进入的共享锁, 它能够被多个线程同时获取, 在没有其他写线程访问 (或者写状态为 0) 时, 读锁总会被成功地获取, 而所做的也只是 (线程安全的) 增加读状态; 如果当前线程已经获取了读锁, 则增加读状态; 如果当前线程在获取读锁时, 写锁已经被其他线程获取, 则进入等待状态; 获取读锁的实现版本升级中变得复杂, 主要是因为新增了一些功能; 例如 `getReadHoldCount()` 方法, 作用是返回当前线程获取读锁的次数; 读状态是所有线程获取读锁次数总和, 而每个线程各自后去读锁的次数只能选择保存在 ThreadLocal 中, 由线程自身维护, 这使得读取锁的实现变得复杂; 获取**读锁**的代码如下
```Java
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    int c = getState();
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```
如果其他线程获取了写锁, 则当前线程获取读锁失败, 进入等待状态; 如果当前线程获取了写锁或者写锁未被获取, 则当前线程增加读状态, 成功获取读锁

###### 锁降级
锁降级指的是写锁降级成为读锁, 如果当前线程拥有写锁, 然后将其释放, 最后再获取读锁, 这种分段完成的过程不能称之为锁降级; 锁降级是指把持住 (当前拥有的) 写锁, 再获取到读锁, 随后释放 (先前拥有的) 写锁的过程  
以下是一个锁降级的示例; 因为数据不常变化, 所以多个线程可以并发的进行数据处理, 当数据变更后, 如果当前线程感知到数据变化, 则进行数据的装备工作, 同时其他处理线程被阻塞, 直到当前线程完成数据的准备工作
```Java
public void processData() {
    readLock.lock();
    if (!update) {
        // 必须先释放锁
        readLock.unlock();
        // 锁降级从写锁获取到开始
        writeLock.lock();
        try {
            if (!update) {
                // 准备数据的流程
                update = true;
            }
            readLock.lock();
        } finally {
            // 锁降级完成, 写锁降级为读锁
            writeLock.unlock();
        }
    }
    try {
        // 使用数据的流程
    } finally {
        readLock.unlock();
    }
}
```
当数据发生变更后, update 变量 (volatile 修饰的 boolean 变量) 被设置为 false, 此时所有访问 `processData()` 的线程都能感知到变化; 在这里锁降级是否必须? 答案是必须的; 主要是为了保证数据的可见性, 如果当前线程不获取读锁而是直接释放写锁, 假设此时另一个线程 (假设为 T)获取了写锁并修改了数据, 那么当前线程无法感知线程 T 的数据更新; 如果当前线程获取读锁, 即遵循锁降级的步骤, 线程 T 会被阻塞, 知道当前线程使用数据并释放读锁之后, 线程 T 才能获取写锁进行数据更新  
ReentrantReadWriteLock 不支持锁升级 (把持读锁, 获取写锁, 最后释放读锁的过程)

#### LockSupport 工具
当需要阻塞或唤醒一个线程的时候, 都会使用 LockSupport 工具类来完成相应工作, LockSupport 定义了一组公共静态方法, 这些方法提供了最基本的线程阻塞和唤醒功能, 而 LockSupport 也成为了构建同步组件的基础工具; LockSupport 定义了一组以 park 开头的方法用来阻塞当前线程, 以及 unpark(Thread thread) 的方法来唤醒一个被阻塞的线程

| 方法名称 | 描述 |
| :--- | :--- |
| void park() | 阻塞当前线程, 如果调用 unpark(Thread thread) 方法或者当前线程被中断, 才能从 park() 方法返回 |
| void parkNanos(long nanos) | 阻塞当前线程, 最长不超过 nanos 纳秒, 返回条件在 park() 的基础上增加了超时返回 |
| void parkUntil(long deadline) | 阻塞当前线程, 直到 deadline 时间 |
| void unpark(Thread thread) | 唤醒处于阻塞状态的线程 thread |

#### Condition 接口
任意一个 Java 对象, 都拥有一组监视器方法 (定义在 java.lang.Object 上), 主要包括 `wait(), wait(long timeout), notify(), notifyAll()` 方法, 这些方法与 synchronized 同步关键字配合, 可以实现等待/通知模式; Condition 接口也提供了类似 Object 的监视器方法, 与 Lock 配合可以实现等待/通知模式, 但是这两者在使用方式上以及功能特性上还是有差别的

| 对比项 | Object Monitor Methods | Condition |
| :--- | :--- | :--- |
| 前置条件 | 获取对象的锁 | 调用 Lock.lock() 获取锁, 调用 Lock.newCondition() 获取 Condition 对象 |
| 调用方式 | 直接调用 object.wait() | 直接调用 condition.await() |
| 等待队列个数 | 一个 | 多个 |
| 当前线程释放锁并进入等待状态 | 支持 | 支持 |
| 当前线程释放锁并进入等待状态, 在等待状态中不响应中断 | 不支持 | 支持 |
| 当前线程释放锁并进入超时等待状态 | 支持 | 支持 |
| 当前线程释放锁并进入等待状态到将来的某个时间 | 不支持 | 支持 |
| 唤醒等待队列中的一个线程 | 支持 | 支持 |
| 唤醒等待队列中的多个线程 | 支持 | 支持 |

##### Condition 接口与示例
Condition 定义了等待/通知两种类型的方法, 当前线程调用这些方法时, 需要提前获取到 Condition 对象关联的锁; Condition 对象是由 Lock 对象 (调用 Lock 对象的 newCondition() 方法) 创建出来的, 即 Condition 是依赖 Lock 对象的; Condition 的使用方式比较简单, 需要注意在调用方法前获取所, 使用示例如下
```Java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();

public void conditionAwait() throws InterruptedException {
    lock.lock();
    try {
        condition.await();
    } finally {
        lock.unlock();
    }
}

public void conditionSignal() {
    lock.lock();
    try {
        condition.signal();
    } finally {
        lock.unlock();
    }
}
```
如示例所示, 一般都会将 Condition 对象作为成员变量; 当调用了 await() 方法后, 当前线程会释放锁并在此等待, 而其他线程调用 Condition 对象的 signal 方法, 通知当前线程后, 当前线程才从 await() 方法中返回, 并且在返回前已经获得了锁  
Condition 定义的部分方法以及描述如下

| 方法名称 | 描述 |
| :--- | :--- |
| void await() | 当前线程进入等待状态直到被通知 (signal) 或中断, 当前线程进入运行状态且从 await() 方法返回的情况包括: 其他线程调用该 condition 的 signal() 或 signalAll() 方法, 而当前线程被选中唤醒 |
| void awaitUninterruptibly() | 当前线程进入等待状态直到被通知, 从方法名称上可以看出该方法对中断不敏感 |
| long awaitNanos(long nanosTimeout) | 当前线程进入等待状态知道被通知, 中断或者超时; 返回值表示剩余是时间 |
| boolean awaitUntil(Date deadline) | 当前线程进入等待状态直到被通知, 中断或者到某个时间; 如果没有到指定时间就被通知, 方法返回 true, 否则表示到了指定时间, 方法返回 false |
| void signal() | 唤醒一个等待在 Condition 上的线程, 该线程从等待方法返回前必须获取的与 Condition 相关联的锁 |
| void signalAll() | 唤醒所有等待在 Condition 上的线程, 能够从等待方法返回的线程必须获得与 Condition 相关的锁 |

获取一个 Condition 必须通过 Lock.newCondition() 方法; 下面展示一个有界队列的示例展示 Condition 的使用方式
```Java
public class BoundedQueue<T> {
    private Object[] items;
    // 添加的下标, 删除的下标和数组当前数量
    private int addIndex, removeIndex, count;
    private Lock lock = new ReentrantLock();
    private Condition notEmpty = lock.newCondition();
    private Condition notFull = lock.newCondition();

    public BoundedQueue(int size) {
        items = new Object[size];
    }

    // 添加一个元素, 如果数组满了, 则添加线程进入等待状态, 直到有 "空位"
    public void add(T t) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) {
                notFull.await();
            }
            items[addIndex] = t;
            if (++addIndex == items.length) {
                addIndex = 0;
            }
            count++;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    // 由头部删除一个元素, 如果数组空, 则删除线程进入等待状态, 直到有新添加元素
    public T remove() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                notEmpty.wait();
            }
            Object obj = items[removeIndex];
            if (++removeIndex == items.length) {
                removeIndex = 0;
            }
            count--;
            notFull.signal();
            return (T) obj;
        } finally {
            lock.unlock();
        }
    }
}
```
首先需要获得锁, 目的是确保数组修改的可见性和排他性; 在添加和删除方法中使用 while 循环而非 if 判断, 目的是防止过早或意外的通知; 这与经典范式的等待/通知是非常类似的

##### Condition 的实现分析
ConditionObject 是同步器 AbstractQueuedSynchronizer 的内部类, 因为 Condition 的操作需要获取相关联的锁, 所以作为同步器的内部类也是合理的; 每个 Condition 对象都包含着一个队列, 该队列是 Condition 对象实现等待/通知功能的关键; 下面将分析 Condition 的实现, 主要包括等待队列, 等待和通知
###### 等待队列
等待队列是一个 FIFO 的队列, 在队列中的每个节点都包含了一个线程引用, 该线程就是在 Condition 对象上等待的线程, 如果一个线程调用了 Condition.await() 方法, 那么该线程将会释放锁, 构造成节点加入等待队列并进入等待状态; 这里的节点定义复用了同步器中节点的含义, 也就是说, 同步队列和等待队列中的节点类型都是同步器的静态内部类 AbstractQueuedSynchronizer.Node
```
-----------------
|    Condtion   |          --------------          --------------          --------------
| ------------- |          |    节点    |          |     节点    |          |    节点    |
| |firstWaiter| | ------>  | nextWaiter | -------> | nextWaiter | -------> | nextWaiter |
| ------------- |          --------------          --------------          --------------
| |lastWaiter | | ----------------------------------------------------------------^
| ------------- |
-----------------
```
Condtion 拥有首尾节点的引用, 新增节点只需要将原有的尾节点 nextWaiter 指向它, 并且更新尾节点即可; 上述节点引用更新的过程并没有使用 CAS 保证, 原因在于调用 await() 方法的线程必定是获取了锁的线程, 也就是说该过程是由锁来保证线程安全的  
在 Object 的监视模型上, 一个对象拥有一个同步队列和等待队列, 而并发包 Lock (更确切的说是同步器) 拥有一个同步队列和多个等待队列
```
---------
| 同步器 |      -------       -------       -------       -------
|       |      | 节点 |      | 节点 |       | 节点 |      | 节点 |
| head  | ---> | prev | <--- | prev | <--- | prev | <--- | prev |   
| tail  |      | next | ---> | next | ---> | next | ---> | next |
---------      -------       -------       -------       --------
 ^  |                                                       ^
 |  --------------------------------------------------------|
 |
 |  -----------------
 |  |    Condtion   |          --------------          --------------          --------------
 |  | ------------- |          |    节点    |          |     节点    |          |    节点    |
 |  | |firstWaiter| | ------>  | nextWaiter | -------> | nextWaiter | -------> | nextWaiter |
 ---| ------------- |          --------------          --------------          --------------
 |  | |lastWaiter | | ----------------------------------------------------------------^
 |  | ------------- |
 |  -----------------
 |
 |  -----------------
 |  |    Condtion   |          --------------          --------------          --------------
 |  | ------------- |          |    节点    |          |     节点    |          |    节点    |
 |  | |firstWaiter| | ------>  | nextWaiter | -------> | nextWaiter | -------> | nextWaiter |
 ---| ------------- |          --------------          --------------          --------------
    | |lastWaiter | | ----------------------------------------------------------------^
    | ------------- |
    -----------------
```
Condtion 的实现是同步器的内部类, 因此每个 Condition 实例都能够访问同步器提供的方法

###### 等待
调用 Condition 的 await() 方法 (或者以 await 开头的方法), 会使当前线程进入等待队列并释放锁, 同时线程状态变为等待状态; 当从 await() 方法返回时, 当前线程一定获取了 Condition 相关联的锁; 从队列的角度来看, 当调用 await() 方法时, 相当于同步队列的首节点 (获取了锁的节点) 移动到 Condition 的等待队列中
```Java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    long savedState = fullyRelease(node);
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
```
调用该方法的线程成功获取了锁的线程, 也就是同步队列中的首节点, 该方法会将当前线程构造成节点并加入等待队列中, 然后释放同步状态, 唤醒同步队列中的后继节点, 然后当前线程会进入等待状态; 当等待队列中的节点被唤醒, 则唤醒节点的线程开始尝试获取同步状态, 如果不是通过其他线程调用 Condition.signal() 方法唤醒, 而是对等待线程进行中断, 则会抛出 InterruptedException
```
---------   --------------------v
| 同步器 |  |   =======       -------       -------       -------
|       |   |  | 节点 |      | 节点 |       | 节点 |      | 节点 |
| head  | ---  | prev | <--- | prev | <--- | prev | <--- | prev |   
| tail  |      | next | ---> | next | ---> | next | ---> | next |
---------      =======       -------       -------       --------
 ^  |             |                                         ^
 |  --------------|-----------------------------------------|
 |                --------------------------------------------------------------------
 |  -----------------            以节点的线程构造新的节点并加入等待队列                  v
 |  |    Condtion   |          --------------          --------------          ==============
 |  | ------------- |          |    节点    |          |     节点    |          |    节点    |
 |  | |firstWaiter| | ------>  | nextWaiter | -------> | nextWaiter | -------> | nextWaiter |
 ---| ------------- |          --------------          --------------          ==============
    | |lastWaiter | | ----------------------------------------------------------------^
    | ------------- |
    -----------------
```

###### 通知
调用 Condition 的 signal() 方法, 将会唤醒在等待队列中等待时间最长的节点 (首节点), 在唤醒节点之前, 会将节点移到同步队列中
```Java
public final void signal() {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```
调用该方法的前置条件是当前线程必须获取了锁, 可以看到 signal() 方法进行了 isHeldExclusively() 检查, 也就是当前线程必须是获取了锁的线程; 接着获取等待队列的首节点, 将其移动到同步队列并使用 LockSupport 唤醒节点中的线程
```
---------   
| 同步器 |      -------       -------       -------       -------
|       |      | 节点 |      | 节点 |       | 节点 |      | 节点 |
| head  | ---> | prev | <--- | prev | <--- | prev | <--- | prev |   
| tail  |      | next | ---> | next | ---> | next | ---> | next |
---------      =======       -------       -------       --------
 ^  |                                                       ^
 |  ------------------------------        -------------------
 |                               |        |
 |  -----------------            v        |                                               
 |  |    Condtion   |          ==============          --------------          --------------
 |  | ------------- |          |    节点    |          |     节点    |          |    节点    |
 |  | |firstWaiter| | ----     | nextWaiter |          | nextWaiter | -------> | nextWaiter |
 ---| ------------- |    |     =============          --------------           --------------
    | |lastWaiter | |    ------------------------------------^                      ^
    | ------------- |----------------------------------------------------------------
    -----------------
```
通过调用同步器的 `enq(Node node)` 方法, 等待队列中的头节点线程安全的移动到同步队列; 当节点移动到同步队列后, 当前线程再使用 LockSupport 唤醒该节点的线程; 被唤醒的线程将从 await() 方法的 while 循环中退出 (isOnSyncQueue(Node node) 方法返回 true), 进而调用同步器的 acquireQueued() 方法加入到获取同步状态的竞争中; 成功获取同步状态后, 被唤醒的线程将从先前调用的 await() 方法返回, 此时该线程已经成功获取了锁; Condition 的 signalAll() 方法相当于等待队列中的每个节点执行一次 signal() 方法
