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

#### 队列同步器
队列同步器 AbastactQueuedSynchronizer 是用来构建锁或者其他同步组件的基础框架, 它使用了一个 int 成员变量表示同步状态, 通过内置的 FIFO 队列来完成资源获取线程的排队工作  
同步器的主要使用方式是继承, 子类通过继承同步器并实现它的抽象方法来管理同步状态, 同步器提供了 3 个方法 (getState(), setState(int newState), compareAndSetState(int expect, int update)) 进行操作; 子类推荐被定义为同步组件的静态内部类, 同步器本身没有实现任何同步接口, 它仅仅是定义了若干同步状态获取和释放的方法来供自定义同步组件使用, 同步器既可以支持独占式地获取同步状态, 也可以共享式地获取同步状态, 可以方便实现不同类型的同步组件 (ReentrantLock, ReentrantReadWriteLock, CountDownLatch)  
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
```
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

    // 仅需要将操作代理到 sync 上即可
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

##### 队列同步器的实现分析

###### 同步队列
同步器依赖内部的同步队列 (一个 FIFO 双向队列) 来完成同步状态的管理, 当前线程获取同步状态失败时, 同步器会将当前线程以及等待状态等信息构造成一个节点 (Node) 并将其加入同步队列, 同时会阻塞当前线程, 当同步状态释放时, 会把首节点中的线程唤醒, 使其再次尝试获取同步状态; 同步队列中的节点 (Node) 用来保存获取同步状态失败的线程引用, 等待状态以及前驱和后继节点, 节点属性类型以及名称等

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
    |                                                       ^
    --------------------------------------------------------|
```
同步队列遵顼 FIFO, 头节点是获取同步状态成功的节点, 头节点的线程在释放同步状态时, 将会唤醒后继节点, 而后继节点将会在获取同步状态成功时将自己设置为首节点
```
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
通过调用同步器的 acquire(int arg) 方法可以获取同步状态, 该方法对中断不敏感, 线程获取同步方法失败后进入同步队列中
```

```
