### Java 并发编程基础

#### 线程简介

##### 什么是线程
现代操作系统调度的最小单元是线程, 也叫轻量级进程; 在一个进程里可以创建多个线程, 这些线程都有自己的计数器, 堆栈和局部变量等, 并且能够访问共享的内存变量

##### 为什么要使用多线程
- 更多的处理器核心
- 更快的响应时间
- 更好的编程模型

##### 线程优先级
Java 线程中通过一个整型成员变量 priority 来控制优先级, 优先级的范围从 1 - 10, 默认优先级是 5; 优先级高的线程分配的时间片的数量要多于优先级低的线程; 在设置线程优先级时, 针对频繁阻塞 (休眠或 I/O 操作) 的线程需要设置较高优先级, 而偏重计算的线程则设置较低的优先级, 确保处理器不会被独占; 在不同的 JVM 以及操作系统上, 线程规划会有差异, 有些操作系统甚至会忽略对线程优先级的设定, 所以线程优先级不能作为程序正确性的依赖

##### 线程的状态
Java 线程在运行的生命周期中可能处于 6 种不同的状态, 在给定的一个时刻, 线程只能处于其中的一个状态

| 状态名称 | 说明 |
| :------------- | :------------- |
| NEW | 初始状态, 线程被构建, 但是还没调用 start() 方法 |
| RUNNABLE | 运行状态, Java 线程将操作系统中的就绪和运行两种状态统称为 "运行中" |
| BLOCKED | 阻塞状态, 表示线程被阻塞 |
| WAITING | 等待状态, 表示线程进入等待状态, 进入该状态表示当前线程需要等待其他线程做出一些特定动作 (通知或中断) |
| TIME_WAITING | 超时等待状态, 不同于 WAITING, 它是可以在指定的时间自行返回的 |
| TERMINATED | 终止状态, 表示当前线程已经执行完毕 |

##### Daemon 线程
Daemon 线程是一种支持型线程, 主要被用作于程序后台调度以及支持性工作, 这意味着当 Java 虚拟机中不存在非 Daemon 线程的时候, Java 虚拟机将会退出, 可以使用 Thread.setDeamon(true) 将线程设置为 Daemon 线程

#### 启动和终止线程

##### 构造线程
在运行线程之前首先要构造一个线程对象, 线程对象在构造的时候需要提供线程所需要的一些属性; 以下代码是 Thread 类中对线程进行初始化的部分
```
private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;

        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            /* Determine if it's an applet or not */

            /* If there is a security manager, ask the security manager
               what to do. */
            if (security != null) {
                g = security.getThreadGroup();
            }

            /* If the security doesn't have a strong opinion of the matter
               use the parent thread group. */
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }

        /* checkAccess regardless of whether or not threadgroup is
           explicitly passed in. */
        g.checkAccess();

        /*
         * Do we have the required permissions?
         */
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }
        // 线程组未启动的线程加一
        g.addUnstarted();
        // 继承父线程的 group, daemon, priority
        this.group = g;
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        this.inheritedAccessControlContext =
                acc != null ? acc : AccessController.getContext();
        this.target = target;
        setPriority(priority);
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals = ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        /* Stash the specified stack size in case the VM cares */
        this.stackSize = stackSize;

        /* Set thread ID */
        tid = nextThreadID();
    }
```
由此可见一个线程对象是由其父线程来进行空间分配的, 子线程继承了父线程的一些域, 并且分到一个 tid 来标识此线程; 至此一个能够运行的线程初始化完成, 在堆内存中等待着运行

##### 启动线程
线程对象初始化完成后, 调用 start() 方法就可以启动这个线程, start() 方法的含义是: 当前线程 (即父线程) 同步告知 Java 虚拟机, 只要线程规划器空闲, 应立即启动此线程; 启动一个线程前最好设置线程名称, 在使用 jstack 分析程序时可以获得一些有用的提示

##### 理解中断
线程通过自身检查是否被中断来进行响应, 线程通过方法 isInterrupted() 来进行判断是否被中断, 也可以调用静态方法 Thread.interrupted() 对当前线程的中断标识进行复位; 如果该线程已经处于终结状态, 级该线程被中断过, 在调用该线程对象的 isInterrupted() 时依然会返回 false; 其他许多抛出 InterruptedException 的方法在抛出此异常前, JVM 会先将该线程的中断标识位清除然后抛出异常, 此时调用 isInterrupted() 方法将会返回 false

##### 过期的 suspend(), resume(), stop()
这些方法过期的原因是会带来副作用: suspend() 方法在 调用后不会释放已经占有的资源 (比如锁), 而是占着资源进入睡眠状态, 这样容易引发死锁问题; stop() 方法在终结线程时不会保证线程的资源正确释放, 通常是没有给予线程完成资源释放工作的机会, 会导致程序可能工作在不确定的状态下

##### 安全地终止线程
通过中断或者一个 boolean 变量来控制线程是否终止

#### 线程间通信
Java 支持多个线程同时访问一个对象或者对象的成员变量, 由于每个线程可以拥有这个变量的拷贝 (每个线程有一份拷贝是为了加速程序的运行), 所以在程序执行过程中, 一个线程看到的变量并不一定是最新的; 可以使用 volatile 或 synchronized 关键字来保证共享变量对其他线程的可见性  
以下代码使用了同步代码块和同步方法, 使用 javap 分析同步的实现细节
```
public class Synchronized {
    public static void main(String[] args) {
        // 同步代码块, 对 Synchronized Class 对象进行加锁
        synchronized (Synchronized.class) {}
        // 静态同步方法, 对 Synchronized Class 对象进行加锁
        m();
    }

    public static synchronized void m() {}
}
```
javap -v Synchronized.class 后如下 (省略部分信息)
```
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #2                  // class com/ltchen/java/demo/lang/Synchronized
         2: dup
         3: astore_1
         4: monitorenter
         5: aload_1
         6: monitorexit
         7: goto          15
        10: astore_2
        11: aload_1
        12: monitorexit
        13: aload_2
        14: athrow
        15: invokestatic  #3                  // Method m:()V
        18: return
      ...

  public static synchronized void m();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=0, locals=0, args_size=0
         0: return
      ...
```
可以看到同步代码块使用 monitorenter 和 monitorexit 实现, 而同步方法则使用 ACC_SYNCHRONIZED 来完成; 无论采用哪种方式, 本质上都是对一个对象的监视器 (monitor) 进行获取; 任意一个对象都有自己的监视器, 当这个对象由同步代码块或者同步方法调用时, 执行方法的线程必须先获取到该对象的监视器才能进入同步代码块或者同步方法, 而没有获得监视器的线程将会被阻塞在入口处, 进入 BLOCKED 状态; 对象, 对象的监视器, 同步队列, 执行线程关系如下
```
      Mointor.Enter   |---------|    Mointor.Enter 成功    |---------|    Mointor.Exit
----------------------| Monitor |--------------------------| Object  |------------------>
                      |---------|                          |---------|
   Mointor.Exit 后       ^    |    Mointor.Enter 失败后
  通知其余线程出队列      |    V    线程入队列
                 ---------------------
                 | SynchronizedQueue |
                 ---------------------
```

##### 等待 / 通知机制
一个线程修改了一个对象的值, 而另一个线程感知到了变化, 然后进行相应的操作, 这个过程开始于一个线程, 而最终执行又是另一线程; 前者是生产者, 后者是消费者; 简单的做到此的办法是让消费者不断的检查变量是否符合预期, 但这难以保证及时性以及低开销; Java 内置的等待/通知机制能很好的解决这个矛盾, 以下是等待 / 通知的相关方法

| 方法名称 | 描述 |
| :------------- | :------------- |
| notify() | 通知一个先对象上等待的线程, 使其从 wait() 方法返回, 而返回的前提是该线程获取到了对象的锁 |
| notifyAll() | 通知所有等待在该对象上的线程 |
| wait() | 调用该方法的线程进入 WAITING 状态, 只有等待另外线程的通知或被中断才会返回, 在调用 wait() 方法后会释放对象的锁 |
| wait(long) | 超时等待一段时间, 这里参数是毫秒, 如果没有通知就超时返回 |
| wait(long, int) | 对于超时时间更细粒度的控制, 可以达到纳秒 |

等待 / 通知机制, 是指一个线程 A 调用了对象 O 上的 wati() 方法进入等待状态, 而另一个线程调用了对象 O 上的 notify 或 notifyAll() 方法, 线程 A 收到通知后从对象 O 的 wait() 方法返回, 进而执行后续操作; 两个线程通过对象 O 来完成交互, 而对象上的 wait() 和 notify/notifyAll() 的关系就像开关信号, 用来完成等待方和通知方之间的交互工作; 在调用 wait() 和 notify/notifyAll() 需要注意以下细节
- 使用 wait(), notify(), notifyAll() 时需要先对调用对象加锁
- 调用 wait() 方法, 线程由 RUNNING 变为 WAITING, 并将当前线程放置到对象的等待队列
- notify() 会 notifyAll() 方法调用后, 等待线程依旧不会从 wait() 放回, 需要 notif() 或 notifyAll() 的线程释放锁之后, 等待线程才有机会从 wait() 返回
- notify() 方法将等待队列中的一个等待线程从等待队列中移到同步队列中, 而 notifyAll() 方法则是将等待队列中的所有线程全部移到同步队列中, 被移动的线程状态由 WAITING 变为 BOLOCKED
- 从 wait() 方法返回的前提是获得了调用对象的锁

等待 / 通知机制依托于同步机制, 其目的就是确保等待线程从 wait() 方法返回时能够感知通知线程对变量做出的修改; 以下是等待 / 通知机制的工作示例图
```
             Mointor.Enter           Mointor.Enter 成功                  Object.notify()
NotifyThread --------------  ---------------------------------    ------ Object.notifyAll()
                          |  |                               |    |    |
                          V  |                               V    |    |
      Mointor.Enter   |---------|    Mointor.Enter 成功    |---------| V  Mointor.Exit
WaitThread -----------| Monitor |--------------------------| Object  |------------------>
                      |---------|                          |---------|
   Mointor.Exit 后       ^    |    Mointor.Enter 失败后          |
  通知其余线程出队列      |    V    线程入队列                     |
                ---------------------                           |
                | SynchronizedQueue |                           | Object.wait()
                ---------------------                           |
                          ^                                     |
                          | 迁移到同步队列                       |
                ------------------------------------------      |
                |               WaitQueue                |<------
                ------------------------------------------
```
WaitThread 首先获取了对象的锁, 然后调用对象的 wait() 方法, 从而放弃了锁进入了对象的等待队列 WaitQueue 中, 进入等待状态; 由于 WaitThread 释放了对象的锁, NotifyThread 随后获取了对象的锁, 并调用对象的 notify() 方法, 将 WaitThread 从 WaitQueue 移到了 SynchronizedQueue 中, 此时 WaitThread 的状态变为阻塞状态; NotifyThread 释放了锁之后, WaitThread 再次获取到锁并从 wait() 方法返回继续执行

##### 等待 / 通知的经典范式
等待方 (消费者)
- 获取对象的锁
- 如果条件不满足, 那么调用对象的 wait() 方法, 被通知后仍要检查条件
- 条件满足则执行对应的逻辑

通知方 (生产者)
- 获取对象的锁
- 改变条件
- 通知所有等待在对象上的线程

##### 管道输入 / 输出流 
