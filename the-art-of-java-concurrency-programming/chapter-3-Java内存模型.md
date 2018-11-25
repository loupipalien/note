### Java 内存模型

#### Java 内存模型的基础
##### 并发编程模型的两个关键问题
在并发编程中需要处理两个关键问题: 线程之间如何通信以及线程之间如何同步; 通信是指线程之间以何种机制来交换信息, 在命令式编程中, 线程之间的通信机制有两种: 共享内存和消息传递  
在共享内存的并发模型里, 线程之间共享程序的公共状态, 通过读写内存中的公共状态进行隐式通信; 在消息传递的并发模型里, 线程之间没有公共状态, 线程之间必须通过发送消息来显示进行通信  
同步是值程序中用于控制不同线程间操作发生相对顺序的机制, 在共享内存并发模型里, 同步是显式进行的; 在消息传递的模型里, 由于消息的发送必须在消息接收之前, 因此同步是隐式进行的  
Java 的并发采用的是共享内存模型, Java 线程之间的通信是隐式进行的

##### Java 内存模型的抽象结构
在 Java 中所有的实例域, 静态域, 数组元素都存储在堆内存中, 堆内存在线程之间共享; 局部变量, 方法定义参数, 异常处理器参数不会在线程之间共享, 因此不会有内存可见性问题, 也不受内存模型的影响  
Java 线程之间的通信由 Java 内存模型 (JMM) 控制, JMM 决定了一个线程对共享变量的写入何时对另一个线程可见; JMM 定义了线程和主内存之间的抽象关系: 线程之间的共享变量存储在主内存 (Main Memory) 中, 每个线程都有一个私有的本地内存 (Local Memory), 本地内存中存储了该线程以读/写共享变量的副本 (本地内存只是一个抽象, 并不存在)
```
-----------                   -----------
|  线程 A  |                  |  线程 B  |
-----------                   -----------
    |                              |
    V                              V
-------------                 -------------
|本地内存 A  |                 |本地内存 B  |
|共享变量副本|                 |共享变量副本 |  
-------------                 -------------    
    |                              |
    V                              V
--------------------------------------------
|                 主内存                    |
|                共享变量                   |
--------------------------------------------
```
如果线程 A 和线程 B 要通信的话, 必须通过以下两个步骤
- 线程 A 把本地内存 A 更新过的共享变量刷新到主内存中去
- 线程 B 到主内存中读取线程 A 之前更新过的变量
从整体上来看, 以上两个步骤实质上是线程 A 向线程 B 发消息, 而且这个通信必须经过主内存; JMM 通过控制主内存与每个线程的本地内存之间的交互, 来为 Java 程序提供内存可见性

##### 从源代码到指令序列的重排序
在执行程序时, 为了提高性能, 编译器和处理器常会对指令做重新排序, 分以下三种
- 编译器优化的重排序
- 指令级并行的重排序
- 内存系统的重排序
对于编译器, JMM 的编译器重排序规则会禁止特定类型的编译器重排序; 对于处理器, JMM 的处理器重排序规则会要求 Java 编译器在生成指令序列时, 插入特定类型的内存屏障指令, 同步内存屏障指令来禁止特定类型的处理器重排序; JMM 属于语言级的内存模型, 确保在不同的编译器和不同的处理器平台上, 通过禁止特定类型的编译器和处理器重排序, 提供一致的内存可见性保证

##### 并发编程模型的分类
常见处理器的重排序规则

| 处理器\规则 | Load-Load | Load-Store | Store-Store | Store-Load | 数据依赖 |
| :------------- | :------------- | :------------- | :------------- | :------------- | :------------- |
| SPARC-TSO | N | N | N | Y | N |
| x86 | N | N | N | Y | N |
| IA64 | Y | Y | Y | Y | N |
| PowerPC | Y | Y | Y | Y | N |

常见的处理器都允许 Store-Load 重排序, 都不允许存在数据依赖的操作做重排序, SPARC-TSO 和 x86 拥有相对较强的处理器内存模型, 仅允许对写 - 读操作做重排序 (因为都用了写缓冲区); 为了保证内存可见性, Java 编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序; JMM 把内存屏障指令分为 4 类

| 屏障类型 | 指令示例 | 说明 |
| :------------- | :------------- | :------------- |
| LoadLoad Barriers | Load1;LoadLoad;Load2 |确保 Load1 数据的装载先于 Load2 及后续装载指令的装载|
| StoreStore Barriers | Store1;StoreStore;Store2 |确保 Store1 数据对其他处理器可见 (刷新到内存) 先于 Store2 及后续装载指令的装载|
| LoadStore Barriers | Load1;LoadStore;Store2 |确保 Load1 数据的装载先于 Store2 及后续装载指令的装载|
| StoreLoad Barriers | Store1;StoreLoad;Load2 |确保 Store1 数据对其他处理器可见 (刷新到内存) 先于 Load2 及后续装载指令的装载; StoreLoad Barriers 会使该屏障之前的所有内存访问指令 (存储和装载指令) 完成之后, 才执行该屏障之后的内存访问指令|

StoreLoad Barriers 是一个 "全能型" 的屏障, 它同时具有其他 3 个屏障的效果, 现代的多处理器大多支持此屏障; 执行该屏障开销会很昂贵, 因为当前处理器通常要把写缓冲区的数据全部刷新到内存中

##### happens-before 简介
JDK 5 开始 Java 使用 JSR-133 内存模型, JSR-133 中使用 happens-before 的概念來阐述操作之间的可见性; 在 JMM 中如果一个操作执行的结果需要对另一个操作可见, 那么这两个操作之间必须要存在 happens-before 关系; 这里的两个操作可以在一个线程中也可以不在一个线程中
- 程序顺序规则: 一个线程中的每个操作, happens-before 于该线程中的任意后续操作
- 监视器锁规则: 对一个锁解锁, happens-before 于随后对这个锁的加锁
- volatile 变量规则: 对一个 volatile 域的写, happens-before 于任意后续对这个 volatile 域的读
- 传递性: 如果 A happens-before B, B happens-before C, 那么 A happens-before C

#### 重排序
重排序是编译器和处理器优化程序性能而对指令序列进行重排序的一种手段

##### 数据依赖性
如果两个操作访问同一个变量, 且这两个操作中有一个为写操作, 此时这两个操作之间就存在数据依赖性

| 名称 | 代码示例 | 说明 |
| :------------- | :------------- | :------------- |
| 写后读 | a = 1; b = a; | 写一个变量后再读这个变量 |
| 写后写 | a = 1; a = 2; | 写一个变量后再写这个变量 |
| 读后写 | a = b; b = 1; | 读一个变量后再写这个变量 |

编译器和处理器不会对存在数据依赖的两个操作执行顺序, 这里仅说的是对单个处理器中执行的指令序列和单线程中执行的操作; 不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑

##### as-if-serial 语义
as-if-serial 语义的意思是: 不管怎么重排序, (单线程) 程序的执行结果不能被改变
```
double pi = 3.14; // A
double r = 1.0;  // B
double area = pi * r * r; // C
```
以上代码重排序可能的执行顺序是 A -> B -> C 或者 B -> A -> C; 遵守 as-if-serial 的编译器, runtime, 处理器共同为编写程序的人制造了一个幻觉: 单线程程序是程序的顺序来执行的; as-if-serial 语义使单线程程序员无需担心重排序会干扰, 也无须担心重排序的可见性问题

##### 程序顺序规则
按上面计算圆的面积代码来看
- A happens-before B
- B happens-before C
- A happens-before C

A happens-before B, JMM 并不要求 A 一定要在 B 之前执行, JMM 仅要求前一个操作 (执行结果) 对后一个操作可见, 且前一个操作排在第二个操作之前; 这里操作 A 的执行结果不需要对操作 B 可见, 重排序后的结果与 happens-before 顺序执行的一致; JMM 认为这种重排序并不违法, 允许这种重排序

##### 重排序对多线程的影响
```
class ReorderExample {
    int a = 0;
    boolean flag = false;

    public void writer() {
        a = 1; // 1
        flag = true; // 2
        ...
    }

    public void reader() {
        if (flag) {  // 3
            int i = a;  // 4
            ...
        }
    }
}
```
TODO  

在单线程中, 对存在控制依赖的操作重排序, 不会改变执行结果; 但在多线程中, 对存在控制依赖的操作重排序, 可能会改变程序的执行jie'guo

#### 顺序一致性
顺序一致性内存模型是一个理论参考模型

##### 数据竞争与顺序一致性
当程序未正确同步时, 就可能存在数据竞争; 如果程序是正确同步的, 程序的执行将具有顺序一致性 (Sequentially Consistent), 即程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同

##### 顺序一致性内存模型
顺序一致性内存模型有两大特性
- 一个线程中的所有操作必须按照程序的顺序来执行
- (不管程序是否同步) 所有的线程都只能看到一个单一的操作执行顺序, 在顺序一致性内存模型中, 每个操作都必须原子执行且立刻对所有线程可见

假设有 A, B 两个线程, 每个线程有三步执行, 且使用了监视器锁来做同步, 那么两个线程的执行顺序可能是: A1 -> A2 -> A3 -> B1 -> B2 -> B3; 如果没有使用同步, 执行顺序可能是: B1 -> A1 -> A2 -> B2 -> B3 -> A3; 但是在 JMM 中就没有这种保证, 未同步的程序在 JMM 中不但整体的执行顺序是无序的, 而且所有线程看到的操作执行顺序也可能是不一致的

##### 同步程序的顺序一致性结果
```
class ReorderExample {
    int a = 0;
    boolean flag = false;

    public synchronized void writer() {  // 获得锁
        a = 1;
        flag = true;
        ...
    }  // 释放锁

    public synchronized void reader() {  // 获得锁
        if (flag) {
            int i = a;
            ...
        }
    }   // 释放锁
}
```
顺序一致性模型中, 所有操作完成按照程序的顺序串行执行, 而在 JMM 中临界区的代码可以重新排序, JMM 会在进入临界区和退出临界区的两个关键时间点做一些特别的处理, 使得线程在这两个时间点具有与顺序一致性模型相同的内存视图; JMM 的在事项上的基本方针是: 在不改变 (正确同步的) 程序的执行结果前提下, 尽可能地为编译器和处理器的优化打开方便之门

##### 未同步程序的执行特性
对于未同步或未正确同步的多线程程序, JMM 只提供最小安全性: 线程执行时读取到的值, 要么是之前某个线程写入的值, 要么是默认的值 (0, Null, False), JMM 保证线程读操作读取到的值不会无中生有 (Out of Thin Air) 的冒出来的; 为了实现最小安全性, JVM 在堆上分配对象时, 首先会对内存空间进行清零, 然后才会在上面分配对象; 因此在已清零的内存空间中分配对象时, 域的默认初始化已经完成了  
JVM 不保证未同步的程序的执行结果与该程序在顺序一致性模型中的执行结果一致, 因为想要保证一致, 就要禁止大量编译器和处理器的优化, 这非常影响程序执行的性能; 未同步的程序在两个模型中的执行特性有如下差异
- 顺序一致性模型保证单线程内的操作会按程序的顺序执行, 而 JMM 不保证单线程内操作会按程序的顺序执行
- 顺序一致性模型保证所有线程只能看到一致性的操作执行顺序, 而 JMM 不保证所有线程看到一致的操作执行顺序
- JMM 不保证对 64 位的 long 型和 double 型变量写操作具有原子性, 而顺序一致性模型保证对所有的内存读/写操作都具有原子性

#### volatile 的内存语义
理解 volatile 特性一个好方法是把 volatile 变量的单个读/写, 看成是使用同一个锁对这些单个读/写操作做了同步
```
class VolatileFeaturesExample {
    volatile long vl = 0L;

    public void set(long l) {
        vl = l;
    }

    public void getAndIncrement() {
        vl++;
    }

    public long get() {
        return vl;
    }
}
```
以上程序等价于
```
class VolatileFeaturesExample {
    long vl = 0L;

    public synchronized void set(long l) {
        vl = l;
    }

    public void getAndIncrement() {
        long tmp = get();
        tmp += 1L;
        set(tmp);
    }

    public synchronized long get() {
        return vl;
    }
}
```
简而言之, volatile 变量自身具有以下特性
- 可见性: 对一个 volatile 变量的读, 总是能看到 (任意线程) 对这个 volatile 变量最后的写入
- 原子性: 对任意单个 volatile 变量的读/写具有原子性, 但类似于 volatile++ 这种复合操作不具有原子性

##### volatile 写 - 读建立的 happens-before 关系
从 JSR-133 开始 (即从 JDK5 开始), volatile 变量的写读可以实现线程之间的通信; 从内存语义的角度来说, volatile 的读写与锁的获取和释放有相同的内存语义
```
class VolatileExample {
    int a = 0;
    volatile boolean flag = false;

    public void writer() {
        a = 1;  // 1
        flag = true;  // 2
        ...
    }

    public void reader() {
        if (flag) {  // 3
            int i = a;  // 4
            ...
        }
    }
}
```
假设线程 A 执行 writer() 之后, 线程 B 执行 reader() 方法; 根据 happens-before 规则, 可以建立以下关系
- 根据程序次序规则: 1 happens-before 2, 3 happens-before 4
- 根据 volatile 规则: 2 happens-before 3;
- 根据 happens-before 的传递性规则, 1 happens-before 4

##### volatile 写 - 读的内存语义
- volatile 写的内存语义: 当写一个 volatile 变量时, JMM 会把该线程对应的本地内存中的共享变量值刷新到主内存中
即当线程 A 在写 flag 变量后, 本地内存 A 中被线程 A 更新过的两个共享变量值被刷新到主内存中
- volatile 读的内存语义: 当读一个 volatile 变量时, JMM 会把该线程对应的本地内存置为无效, 线程接下来将从主内存中读取共享变量

如果把 volatile 的写读两个步骤综合起来看, 在读线程 B 读一个 volatile 变量后, 在写线程 A 写一个 volatile 变量之前, 所有可见的共享变量的值都将立即变得对读线程 B 可见

##### volatile 内存语义的实现
为了实现 volatile 内存语义, JMM 分别会限制编译器和处理器的重排序, 下表是 JMM 对编译器执行的 volatile 重排序规则表

| 第一操作\第二操作 | 普通读/写 | volatile 读 | volatile 写 |
| :------------- | :------------- | :------------- | :------------- |
| 普通读/写 | - | - | NO |
| volatile 读 | NO | NO | NO |
| volatile 写 | - | NO | NO |

- 当第二个操作是 volatile 写时, 无论第一个操作是什么都不能重排序; 这个规则确保 volatile 写之前的操作不会被编译器重排序到 volatile 写之后
- 当第一个操作是 volatile 读时, 无论第二个操作是什么都不能重排序; 这个规则确保 volatile 读之后的操作不会被编译器重排序到 volatile 读之前
- 当第一个操作是 volatile 写, 第二个操作是 volatile 读时, 不能重排序

为了实现 volatile 的内存语义, 编译器在生成字节码时, 会在指令序列中插入内存屏障来禁止特定类型的处理器重排序; 对于编译器来说最小化插入内存屏障总是不可能的, 所以 JMM 采用保守策略保证在任意处理器平台都能得到正确的 volatile 内存语义
- 在每个 volatile 写操作的前面插入一个 StoreStore 屏障
- 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障
- 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障
- 在每个 volatile 读操作的后面插入一个 LoadStore 屏障

```
普通读/写 -> StoreStore 屏障 -> volatile 写 -> StoreLoad 屏障
```
StoreStore 屏障可以保证在 volatile 写之前, 其他前面的所有普通写操作刷新到主内存, 对任意处理器可见; volatile 写后面的 StoreLoad 屏障的作用是避免 volatile 写与后面可能有的  volatile 读/写操作重排序, 但编译器常无法判断在一个 volatile 写后是否需要插入 StoreLoad 屏障 (例如一个 volatile 写之后方法立即 return); 为保证正确实现 volatile 的内存语义, JMM 采用保守策略: 在每个 volatile 写后面或每个 volatile 读前插入一个 StoreLoad 屏障, 为了执行效率 JMM 选择在 volatile 写后插入屏障
```
volatile 读 -> LoadLoad 屏障 -> LoadStore 屏障 -> 普通读/写
```
LoadLoad 屏障用来禁止处理器把前面的 volatile 读与后面普通读重排序, LoadStore 屏障用来禁止处理器把前面的 volatile 读与后面的普通写重排序

##### JSR-133 为什么要增强 volatile 的内存语义
TODO

#### 锁的内存语义

##### 锁的释放 - 获取建立的 happens-before 关系
锁除了让临界区互斥执行外, 还可以让释放锁 的线程向同一个锁的线程发送消息
```
class MonitorExample {
    int a = 0;

    public synchronized void writer() {  // 1
        a++;                             // 2
    }                                    // 3

    public synchronized void reader() {  // 4
        int i = a;                       // 5
    }                                    // 6
}
```
假设线程 A 执行 writer() 之后, 线程 B 执行 reader() 方法; 根据 happens-before 规则, 可以建立以下关系
- 根据程序次序规则: 1 happens-before 2, 2 happens-before 3, 4 happens-before 5, 5 happens-before 6
- 根据监视器锁规则: 3 happens-before 4;
- 根据 happens-before 的传递性规则, 2 happens-before 5

##### 锁的释放和获取的内存语义
当线程释放锁时, JMM 会把该线程对应的本地内存中的共享变量刷新到主内存中; 当线程获取锁时, JMM 会把该线程对应的本地内存置为无效, 从而使得被监视器保护的临界区代码必须从主内存中读取共享变量  
对比 volatile 内存语义, 锁释放与 volatile 写有相同的内存语义, 锁获取与 volatile 读有相同的内存语义; 以下是所获取释放的实质
- 线程 A 释放一个锁, 实质上是线程 A 向接下来将要获取这个锁的某个线程发出了 (线程 A 对共享变量所做修改的) 消息
- 线程 B 获取一个锁, 实质上是线程 B 接受了之前某个线程发出的 (在释放这个锁之前对共享变量所做修改的) 消息
- 线程 A 释放锁, 随后线程 B 获取锁, 这个过程的实质是线程 A 通过主内存向线程 B 发送消息

##### 锁内存语义的实现
以下借助 ReentrantLock 分析锁内存语义的具体实现
```
class ReentrantLockExample {
    int a = 0;
    ReentrantLock lock = new ReentrantLock()

    public void writer() {  
        lock.lock();
        try {
            a++;                             
        } finally {
            lock.unlock();
        }
    }                                    

    public  void reader() {  
        lock.lock();
        try {
            int i = a;                             
        } finally {
            lock.unlock();
        }                   
    }                                   
}
```
ReentrantLock 的实现依赖于 Java 同步框架 AbastactQueuedSynchronizer (简称 AQS), AQS 使用一个整形的 volatile 变量 state 来维护同步状态  

TODO

##### concurrent 包的实现
由于 Java 的 CAS 同时具有 volatile 读和 volatile 写的内存语义, 因此 Java 线程之间通信现在有了以下 4 种方式
- A 线程写 volatile 变量, 随后 B 线程读这个 volatile 变量
- A 线程写 volatile 变量, 随后 B 线程用 CAS 更新这个 volatile 变量
- A 线程用 CAS 更新一个 volatile 变量, 随后 B 线程用 CAS 更新这个 volatile 变量
- A 线程用 CAS 更新一个 volatile 变量, 随后 B 线程读这个 volatile 变量

volatile 变量的读/写和 CAS 可以实现线程之间的通信, 把这些整合在一起形成了 concurrent 包的基石; concurrent 包通用化的实现模式是: 首先声明共享变量为 volatile, 然后使用 CAS 的原子条件更新来实现线程之间的异步, 同时配合 volatile 读/写和 CAS 所具有的 volatile 读写的内存语义来实现线程之间的通信
```
---------------------------------------------------------------------
| --------   ----------   -----------   -----------    -----------  |   
| | Lock |   | 同步器 |   | 阻塞队列 |   | Executor |   | 并发容器 |  |                                      
| --------   ----------   -----------   -----------    -----------  |                                  
---------------------------------------------------------------------                                                  ^                     ^                      ^
          |                     |                      |
----------------------------------------------------------------------
|        -------      ----------------    ------------               |   
|        | AQS |      | 非阻塞数据结构 |   | 原子变量类 |              |                                      
|        -------      ----------------    ------------               |                                  
----------------------------------------------------------------------  
         ^                     ^                      ^
         |                     |                      |
----------------------------------------------------------------------
|        ----------------------             -------                  |   
|        | volatile 变量的读写 |             | CAS |                  |                                      
|        ----------------------             -------                  |                                  
----------------------------------------------------------------------  
```

#### final 域的内存语义

###### final 域的重排序规则
对于 final 域, 编译器和处理器遵循以下两个重排序规则
- 在构造函数内对一个 final 域的写入, 与随后把这个被构造对象的引用赋给一个引用变量, 这两个操作之间不能被重排序
- 初次读一个包含 final 域的对象的引用, 与随后初次读这个 final 域, 这两个操作之间不能被重排序

```
public class FianlExample {
    int i;
    int final j;
    static FianlExample obj;

    public FianlExample() {
        i = 1;  // 写普通域
        j = 2;  // 写 final 域
    }

    public static void writer() {  // 写线程 A 执行
        obj = new FianlExample();
    }

    public static void reader() {  // 读线程 B 执行
        FianlExample object = obj;  // 读对象引用
        int a = object.a;           // 读普通域
        int b = object.b;           // 读 final 域
    }
}
```

##### 写 final 域的重排序规则
写 final 域的重排序规则禁止把 final 域的写重排序到构造函数外, 实际包含以下两方面
- JMM 禁止编译器把 final 域的写重排序到构造函数外
- 编译器会在 final 域的写之后, 构造函数 return 之前, 插入一个 StoreStore 屏障, 这个屏障禁止处理器把 final 域的写重排序到构造函数外

假设线程 A 先执行 writer() 方法, 随后线程 B 执行 reader() 方法, 可能出现这样的执行顺序
```
A: 构造函数开始执行 -> A: 写 final 域 (j = 2) -> A: StoreStore 屏障 -> A: 构造函数执行结束 -> A: 把构造对象的引用赋值给引用变量 obj -> B: 读引用对象 obj -> B: 读对象的普通域 i -> B: 读对象的 final 域 -> A: 写普通域 (i = 1)
```

##### 读 final 的重排序规则
读 final 域的重排序规则是: 在一个线程中, 初次读对象引用与初次读该对象包含的 final 域, JMM 禁止处理器重排序这两个操作, 因为编译器会在读 final 域操作的前面插入一个 LoadLoad 屏障  
假设线程 A 没有发生任何重排序, 可能出现这样的执行顺序
```
A: 构造函数开始执行 -> B: 读对象的普通域 i -> A: 写 final 域 (j = 2) -> A: StoreStore 屏障 -> A: 构造函数执行结束 -> A: 把构造对象的引用赋值给引用变量 obj -> B: 读引用对象 obj ->  B: 读对象的 final 域 -> A: 写普通域 (i = 1)
```
TODO

##### final 为引用类型
```
public class FinalReferenceExample {
    final int[] intArray;  //  final 是引用类型
    static FinalReferenceExample obj;

    puublic FinalReferenceExample() {
        intArray = new int[1];  // 1
        intArray[0] = 1;        // 2
    }

    public static void writerOne() {        // 写线程 A 执行
        obj = new FinalReferenceExample();  // 3
    }

    public static void writerTne() {  // 写线程 B 执行
        obj.intArray[0] = 2;          // 4
    }

    public static void reader() {      // 读线程 C 执行
        if (obj != null) {             // 5
            int tmp = obj.intArray[0]; // 6
        }
    }
}
```
对于引用类型, 写 final 域的重排序规则对编译器和处理器增加了如下约束: 在构造函数内对一个 final 引用的对象的成员域的写入, 与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量, 这两个操作之间不能被重排序

##### 为什么 final 引用不能从构造函数内 "溢出"
写 final 域的重排序规则可以确保: 在引用变量为任意线程可见之前, 该引用变量指向的对象的 final 域已经在构造函数中被正确初始化过了; 但其实要得到这个保证, 还需要在构造函数内部不能让这个被构造对象的引用为其他线程所见, 也就是说对象引用不能在构造函数总 "逸出"
```
public class FinalReferenceEscaoeExample {
    final int i;
    static FinalReferenceEscaoeExample obj;

    puublic FinalReferenceEscaoeExample() {
        i = 1;                  // 1 写 final 域
        obj = this;             // 2 this 引用在此处 "逸出"
    }

    public static void writer() {   
        new FinalReferenceEscaoeExample();
    }

    public static void reader() {      
        if (obj != null) {             // 3
            int tmp = obj.i;           // 4
        }
    }
}
```
假设线程 A 先执行 writer() 方法, 随后线程 B 执行 reader() 方法, 可能出现这样的执行顺序
```
A: 构造函数开始执行 -> A: obj = this, 被构造对象的引用在此 "逸出" -> B: if (obj != null), 读取不为 null 的对象引用 -> B: int tmp = obj.i, 将读到 final 域初始化之前的值 -> A: i = 1, 对 final 域初始化 -> A: 构造函数结束
```
因此, 被构造对象的引用不能为其他线程所见, 因为此时的 final 域可能还没有初始化

##### final 语义在处理器中的实现
TODO

##### JSR-133 为什么要增强 final 的语义
在旧的 Java 内存模型中, 一个最严重的缺陷就是线程可能看到 final 域的值会改变; 为了修复这个漏洞, JSR-133 增强后保证: 只要对象正确构造了 (被构造对象的引用在构造函数中没有 "逸出"), 那么不需要使用同步 (指 lock 和 volatile 的使用) 就可以保证任意线程都能看到这个 final 域在构造函数中初始化之后的值

#### happens-before
happens-before 是 JMM 最核心的概念

##### JMM 的设计
在设计 JMM 时需要考虑的两个关键因素
- 程序员对内存模型的使用: 程序员希望内存模型易于理解, 易于编程, 程序员希望基于一个强内存模型来编写代码
- 编译器和处理器对内存模型的实现: 编译器和处理器希望内存模型对它们的束缚越少越好, 这样就可以做尽可能多的优化来提高性能, 编译器和处理器希望实现一个弱内存模型

由于以上两个因素互相矛盾, JMM 要保证为程序员提供足够强的内存可见性保证和对编译器和处理器的限制要尽可能的放松
```
double pi = 3.14;            // A
double r = 1.0;              // B
double area = pi * r * r;    // C
```
以上代码存在 3 个 happens-before 关系
- A happens-before B
- B happens-before C
- A happens-before C

以上三个关系中, 1 是非必须的, 2 和 3 是必需的, 因此 JMM 把 happens-before 要求禁止的重排序分为了以下两类
- 会改变程序执行结果的重排序
- 不会改变程序执行结果的重排序

JMM 对这两种不同性质的重排序, 采取了不同的策略
- 对于会改变程序执行结果的重排序, JMM 要求编译器和处理器必须禁止这种重排序
- 对于不会改变程序执行结果的重排序, JMM 要求编译器和处理器不做要求

JMM 向程序员提供的 happens-before 规则能满足程序员的要求, 也提供了足够强的内存可见性保证; JMM 对编译器和处理器的要求是: 只要不改变程序的执行结果 (单线程程序和正确同步的多线程程序), 编译器和处理器怎么优化都行

##### happens-before 的定义
 JSR-133 使用 happens-before 的概念来指定两个操作之间的执行顺序, 由于两个操作可以在一个线程之内, 也可以是在不同的线程之间, 因此 JMM 可以通过 happens-before 关系向程序员提供跨线程的内存可见性保证; JSR-133 对 happens-before 关系的定义如下
 -  如果一个操作 happens-before 另一个操作, 那么第一个操作的执行结果将对第二个操作可见, 而且第一个操作的执行顺序排在第二个操作之前 (JMM 对程序员的承诺)
 - 两个操作之间存在 happens-before 关系, 并不意味着 Java 平台的具体实现必须要按照 happens-before 关系指定的顺序来执行, 如果重排序之后的操作与按 happens-before 关系来执行的结果是一致的, 那么这种重排序并不非法 (JMM 对编译器和处理器重排序的约束原则)

happens-before 关系本质上和 as-if-serial 语义是一回事; as-if-serial 语义保证单线程内程序的执行的执行结果不变, happens-before 关系保证正确同步的多线程程序的执行结果不被改变; as-if-serial 语义给编程单线程的程序员提供一个幻境: 单线程程序的是按照程序的顺序来执行的, happens-before 关系给编写正确同步的多线程程序的程序员提供一个幻境: 正确同步的多线程程序是按 happens-before 指定的顺序来执行的; as-if-serial 和 happens-before 都是为了在不改变程序执行的结果的前提下尽可能地提高执行的并行度

##### happens-before 规则
JSR-133 定义了如下 happens-before 规则
- 程序顺序规则: 一个线程中的每个操作, happens-before 于该线程的任意后续操作
- 监视器索规则: 对一个锁的解锁, happens-before 于随后对这个锁的加锁
- volatile 变量规则: 对一个 volatile 域的写, happens-before 与任意后续对这个 volatile 的读
- 传递性: 如果 A happens-before B, 且 B happens-before C, 那么 A  happens-before C
- start() 规则: 如果线程 A 执行操作 ThreadB.start(), 那么 A 线程的 ThreadB.start() 操作 happens-before 于线程 B 中的任意操作
- join() 规则: 如果线程 A 执行操作 ThreadB.join() 并成功返回, 那么线程 B 中的任意操作 happens-before 与线程 A 从 ThreadB.join() 操作成功返回

#### 双重检查锁定与延迟初始化

##### 双重检查锁的由来
在 Java 程序中, 有时需要推迟一些高开销的对象初始化操作, 并且只有在使用这些对象时才进行初始化
```
public class UnsafeLazyInitialization {
    private static Instance instance;

    public static Instance getInstance() {
        if (instance == null) {             // 1: 线程 A 执行
            instance = new Instance();      // 2: 线程 B 执行
        }
        return instance;
    }
}
```
在 UnsafeLazyInitialization 类中, 假设 A 线程执行代码 1 的同时, B 线程执行代码 2, 此时线程 A 可能会看到 instance 引用的对象还没有初始化; 对于 UnsafeLazyInitialization 类可以对 getInstance() 方法来同步处理来实现线程安全的延迟初始化
```
public class SafeLazyInitialization {
    private static Instance instance;

    public synchronized static Instance getInstance() {
        if (instance == null) {            
            instance = new Instance();     
        }
        return instance;
    }
}
```
对 getInstance() 方法做了同步处理, synchronized 将导致性能开销; 如果 getInstance() 方法被多个线程频繁调用则会导致执行性能下降, 如果 getInstance() 方法不会被多个线程频繁调用, 那么这个延迟初始化方案将能提供令人满意的性能; 在早期的 JVM 中 synchronized 会有巨大的性能开销, 人们使用使用双重检查锁定 (Double-Checked Locking) 来解决这个问题
```
public class DoubleCheckedLocking {                          // 1
    private static Instance instance;                        // 2

    public static Instance getInstance() {                   // 3
        if (instance == null) {                              // 4: 第一次检查
            synchronized (DoubleCheckedLocking.class) {      // 5：加锁
                if (instance == null) {                      // 6: 第二次检查
                    instance = new Instance();               // 7：问题的根源出在这里
                }                                            
            }                                                // 8:
        }                                                    // 9:
        return instance;                                     // 10:   
    }                                                        // 11:
}
```
如果第一次检查 instance 不为 null, 那么就不需要执行下面的加锁和初始化操作, 就可以大幅度降低 synchronized 带来的性能开销; 以上代码看起来两全其美, 但是线程执行到第 4 行, 代码读取到 instance 不为 null 时, instance 引用的对象有可能还没有完成初始化

##### 问题的根源
双重检查锁定示例代码的第 7 行 (instance = new Singleton();) 创建了一个对象, 这一行代码可以分解为如下三行
```
memory = allocate();    // 1: 分配对象的内存空间
ctorInstance(memory);   // 2: 初始化对象
instance = memory;      // 3: 设置 instance 指向刚分配的内存地址
```
上面 3 行伪代码的 2 和 3 之间可能会被重排序, 2 和 3 重排序之后的执行时序如下
```
memory = allocate();    // 1: 分配对象的内存空间
instance = memory;      // 3: 设置 instance 指向刚分配的内存地址 (此时对象还未初始化)
ctorInstance(memory);   // 2: 初始化对象
```
根据 Java 语言规范, 所有线程在执行 Java 程序时必须要遵守 intra-thread semantics, intra-thread semantics 保证重排序不会改变单线程内的程序执行结果, 即 2 和 3 重排序并不违反 intra-thread semantics; 这个重排序在没有改变单线程程序执行结果的前提下, 可以提高程序的执行性能; 但在多线程当中就可能引发问题, 以下是引发问题的线程执行顺序图
```
A: 1: 分配对象的内存空间 -> A: 3: 设置 instance 指向内存空间 -> B: 1: 判断 instance 是否为空 -> B: 2: B 线程初次访问对象 -> A: 2: 初始化对象 -> A: 4: A 线程初次访问对象
```
可以看出, 单线程遵守 intra-thread semantics 能保证 A 线程的执行结果不会被改变, 但 B 线程却可能看到一个还没有被初始化的对象; 即 DoubleCheckedLocking 代码第 7 行 (instance = new Singleton();) 如果发生重排序, 另一个并发执行的线程 B 就有可能在第 4 行判断 instance 不为 null, 线程 B 接下来访问 instance 所引用的对象, 但此时这个对象可能还没有被 A 线程初始化; 有两种方法可以实现线程安全的延迟初始化
- 不允许 2 和 3 重排序
- 允许 2 和 3 重排序, 但不允许其他线程 "看到" 这个重排序

##### 基于 volatile 的解决方案
```
// 需要 JDK 5 或更高版本的支持, 因为 JDK 5 使用 JSR-133, 增强了 volatile 的语义
public class SafeDoubleCheckedLocking {                          
    private volatile static Instance instance;                       

    public static Instance getInstance() {                  
        if (instance == null) {                              
            synchronized (DoubleCheckedLocking.class) {      
                if (instance == null) {                      
                    instance = new Instance();               
                }                                            
            }                                               
        }                                                   
        return instance;                                    
    }                                                      
}
```
即把 instance 变量声明为 volatile, 来禁止 2 和 3 的重排序到达目的; 以上代码在多线程中执行的时序
```
A: 1: 分配对象的内存空间 -> A: 2: 初始化对象 -> A: 3: 设置 instance 指向内存空间 -> B: 1: 判断 instance 是否为空 -> B: 2: B 线程初次访问对象 -> A: 4: A 线程初次访问对象
```

##### 基于类初始化的解决方案
JVM 在类的初始化阶段 (即在 Class 被加载后, 且在线程使用之前), 会执行类的初始化, 在执行类的初始化期间 JVM 会去获取一个锁; 这个锁可以同步多个线程对同一个类的初始化; 基于此特性实现另一种线程安全的延迟初始化方案 (被称为 Instantiation On Demand Holder idiom)
```
public class InstanceFactory {
    /**
     * 这样虽然线程安全, 但是会被立即初始化 (符合情况 3)
     * private static Instance instance = new Instance();
     */

    private static class InstanceHolder {
        private static Instance instance = new Instance();
    }

    public static Instance getInstance() {
        return InstanceFactory.instance;   // 这里将导致 InstanceFactory 这个类被初始化
    }
}
```
这个方案的实质是: 允许 2 和 3 重排序, 但是不允许非构造线程 (这里指 B 线程) "看到" 这个重排序  
初始化一个类, 包括执行这个类的静态初始化和初始化在这个类中声明的静态字段; 根据 Java 语言规范, 在首次发生下列任意一种情况时, 一个类或接口类型 T 将立即被初始化
- T 是一个类, 而且一个 T 类型的实例被创建
- T 是一个类, 且 T 中声明的一个静态方法被创建
- T 中声明的一个静态字段被赋值
- T 中声明的一个静态字段被使用, 而且这个字段不是一个常量字段
- T 是一个顶级类, 而且一个断言语句嵌套在 T 内存被执行

在 InstanceFactory 示例中, 首次执行 getInstance() 方法的线程将导致 InstanceHolder 类被执行 (符合情况 4)  

对于类或接口的初始化, Java 语言制定了精巧而复杂的类初始化处理过程
TODO  

以上两种方案, 可以发现基于类初始化的方案的实现代码更加简洁, 但基于 volatile 的双重检查锁定的方案有一个额外的优势: 除了可以对静态字段实现延迟初始化, 还可以对实例字段实现延迟初始化; 如果确实需要对实例字段使用线程安全的延迟初始化, 使用基于 volatile 的延迟初始化, 如果确实需要对静态字段使用线程安全的延迟初始化, 使用基于类初始化的方法

#### Java 内存模型综述

##### 处理器的内存模型
根据对不同类型的读/写操作组合的执行顺序的放松, 可以把常见的处理器内存模型划分为如下几种类型
- 放松程序中写 - 读操作的顺序, 由此产生了 Total Store Ordering 内存模型 (简称为 TSO)
- 在上面的基础上, 继续放松程序中写 - 写操作的顺序, 由此产生了 Partial Store Order 内存模型 (简称 PSO)
- 在前面两条的基础上, 继续放松程序中读 - 写和读 - 读的操作顺序, 由此产生了 Relaxed Memory Order 内存模型 (简称 RMO) 和 PowerPC 内存模型

这里的读/写操作的放松, 是以两个操作之间不存在数据依赖性的前提的 (处理器要遵守 as-if-serial 语义, 不会对存在数据依赖性的两个内存操作做重排序的); 以下是常见处理器的细节特征

| 内存模型名称 | 对应的处理器 | Store-Load 重排序 | Store-Store 重排序 | Load-Load 和 Load-Store 重排序 | 可以更早读取到其他处理器的写 | 可以更早读取到当前处理器的写 |
| - | - | - | - | - | - | - |
| TSO | SPARC-TSO X64 | Y | | | | Y |
| PSO | SPARC-PSO | Y | Y | | | Y |
| RMO | IA64 | Y | Y | Y | | Y |
| PowerPC | PowerPC | Y | Y | Y | Y | Y |

所有处理器内存模型都允许写 - 读重排序, 因为它们都使用了写缓存区, 写缓存区可能导致写 - 读操作重排序; 所有处理器也都允许更早读到当前处理器的写, 同样也是因为写缓存; 表中各种处理器的内存模型从上到下模型变弱, 越是追求性能的处理器, 内存模型设计得就会越弱, 处理器的束缚就越少, 就可能做更多的优化; 由于常见的处理器内存模型比 JMM 要弱, Java 编译器在生成字节码时会在执行指令适当的位置插入内存屏障来限制处理器的重排序; JMM 屏蔽了不同处理器内存模型的差异, 在不同的处理器平台上给程序员呈现出一个一致的内存模型

##### 各种内存模型之间的关系
JMM 是一个语言级内存模型, 处理器内存模型是硬件级的内存模型, 顺序一致性模型是一个理论参考模型; 以下是各种内存模型强弱对比
```
>>> 最好 >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> 易编程性 >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> 最差 >>>
PowerPC - IA64 - SPARC-PSO - X86 - SPARC-TSO - C++ 11 MM - JMM - CLR 2.0 MM - Sequential Consistency
<<< 最差 <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 执行性能 <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<< 最好 <<<
```

##### JMM 的内存可见性保证
- 单线程程序: 不会出现内存可见性问题; 编译器, runtime, 处理器会共同确保单线程程序的执行结果和在顺序一致性模型中一致
- 正确同步的多线程程序: 正确同步的多线程程序的执行将具有顺序一致性, JMM 通过限制编译器和处理器的重排序来为提供内存可见性保证
- 未同步 / 未正确同步的多线程程序; JMM 提供了最小安全性保障: 线程执行时读取到的值, 要么是之前某个线程写入的值, 要么是默认值

##### JSR-133 对旧内存模型的修补
- 增强 volatile 的内存语义: 旧内存模型允许 volatile 变量与普通变量重排序, JSR-133 严格限制 volatile 变量与普通变量重排序, 使 volatile 的写 - 读和锁的释放 - 获取具有同样的内存语义
- 增强 final 的内存语义: 旧内存模型中多次读取同一个 final 变量的值可能会不同, JSR-133 为 final 增加了两个重排序规则, 保证在 final 引用不会从构造函数逸出的情况下, 具有初始化安全性
