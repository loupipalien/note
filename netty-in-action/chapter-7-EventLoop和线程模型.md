### EventLoop 和线程模型
#### 线程模型概述
基本的线程池模式可以描述为
- 从池的空闲线程列表中选择一个 Thread, 并且指派它去运行一个已提交的任务
- 当任务完成时, 将该 Thread 返回给该列表, 使其可被重用

虽然池化和重用线程对于简单地为每个任务都创建和销毁线程是一种进步, 但是它并没有消除上下文切换带来的开销, 其将随着线程数量的增加很快变得明显, 并且在高负载下愈演愈烈

##### EventLoop 接口
运行任务来处理在连接的生命周期内发生的事件是任何网络框架的基本功能, 与之相对应的编程上的构造通常称之为事件循环 (EventLoop)  
以下代码说明了事件循环的基本思想, 其中每个任务都是一个 Runable 的实例
```
while (!terminated) {
    // 阻塞, 直到有事件就绪可被运行
    List<Runable> readyEvents = blockUntilEventReady();
    // 遍历循环, 处理所有事件
    for (Runable ev : readyEvents) {
        ev.run()
    }
}
```
在这个模型中, 一个 EventLoop 将由一个永远都不会改变的 Thread 驱动, 同时任务 (Runable 或 Callable) 可以直接提交给 EventLoop 实现, 以立即执行或调度执行; 根据配置和可用核心的不同, 可能会创建多个 EventLoop 实例以优化资源的使用, 并且单个 EventLoop 可能会被指派用于服务多个 Channel
>**事件/任务的执行顺序**
事件和任务是以先进先出的顺序执行的, 这样可以通过保证字节内容总是按正确的顺序被处理, 消除潜在的数据损坏的可能

##### Netty 4 中的 I/O 和事件处理
在 Netty 4 中, 所有的 I/O 操作和事件都由已经被分配给了 EventLoop 的那个 Thread 来处理

##### Netty 3 中的 I/O 操作
在 Netty 3 中使用的线程模型, 只保证了入站 (之前称之为上游) 事件会在所谓的 I/O 线程中执行, 所有出站 (下游) 事件都由调用线程处理, 其可能是 I/O 线程也可能是别的线程

#### 任务调度
##### JDK 的任务调度 API
以下展示了 java.util.concurrent.Executors 的相关工厂方法

| 方法 | 描述 |
| :--- | :--- |
| newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)  | 创建一个 ScheduledThreadExecutorService, 用于调度命令在指定延迟之后运行或者周期性执行, 它使用 corePoolSize 参数来计算线程数 |
| newSingleThreadScheduledExecutor(ThreadFactory threadFactory) | 创建一个 SingleThreadScheduledExecutor, 用于调度命令在指定延迟之后运行或者周期性执行, 它使用一个线程来执行被调度的任务 |

使用 ScheduledThreadExecutorService 调度任务
```
// 创建具有 10 线程的线程池
ScheduledExecutorService executor = Executors.newScheduledThreadPool(10);
// 创建一个 Runable 在 60 秒后执行
ScheduledFuture<?> future = executor.schedule(() -> System.out.println("60 seconds later"), 60, TimeUnit.SECONDS);
...
// 一旦调度任务执行完成, 就关闭 ScheduledExecutorService 以释放资源
executor.shutdown();
```

##### 使用 EventLoop 调度任务
ScheduledExecutorService 的实现具有局限性, 即作为线程池管理的一部分, 将会有额外的线程创建, 如果有大量任务被紧凑地调度, 那么这将成为一个瓶颈; Netty 通过 Channel 的 EventLoop 实现的任务调度解决了这一问题  
使用 EventLoop 调度任务
```
Channel ch = ...;
// 创建一个 Runable 在 60 秒后执行
ScheduledFuture<?> future = ch.eventLoop.schedule(() -> System.out.println("60 seconds later"), 60, TimeUnit.SECONDS);
```

#### 实现细节
##### 线程管理
Netty 线程模型的卓越性能取决于当前执行的 Thread 的身份的确定, 也就是说确定它是否是分配给当前 Channel 以及它的 EventLoop 的那一个线程; 如果当前调用线程正是支撑 EventLoop 的线程, 那么所提交的代码块将会被 (直接) 执行, 否则 EventLoop 将调度该任务以便稍后执行, 并将它放入内部队列中; 当 EventLoop 下次处理它的事件时, 它会执行队列中的那些任务/事件; 这也就解释了任何的 Thread 是如何与 Channel 直接交互而无需在 ChannelHandler 中进行额外同步的
```

                                             --------      -----------
                               ------------> | 任务 | ---> | 执行 (3) |
                               |             --------      -----------
-----------           ----------------
| 任务 (1) |  ------> | EventLoop (2)|
-----------           ----------------
                               |            --------      -----------
                               -----------> | 任务 | ---> | 入队 (4) |
                                            --------      -----------
(1): 将要在 EventLoop 中执行的任务
(2): 在把任务传递给 execute 方法之后, 执行检查以确定当前调用线程是否就是分配给 EventLoop 的那个线程
(3): 如果就是相同的线程, 则在 EventLoop 中可以直接执行任务
(4): 如果线程不是 EventLoop 的那个线程, 则将任务放入队列以便 EventLoop 下一次处理它的事件时执行
```
注意, 不要将一个长时间运行的任务放入执行队列中, 因为它将阻塞需要在同一线程上执行的其他任务; 如果必须要进行阻塞调用或者执行长时间运行的任务, 建议使用一个专门的 EventExecutor

##### EventLoop/线程的分配
服务于 Channel 的 I/O 和事件的 EventLoop 包含在 EventLoopGroup 中, 根据不同的传输实现, EventLoop 的创建和分配方式也不同

###### 异步传输
异步传输实现只使用了少量的 EventLoop (以及和它们相关联的 Thread), 而且在当前的线程模型中, 它们可能会被多个 Channel 所共享; 这使得可以通过尽可能少量的 Thread 来支撑大量的 Channel, 而不是一个 Channel 分配一个 Thread
```
                                       (2)                     (3)
                                 -------------      -------------------------
            -------------------> | EventLoop | ---> | Channel, Channel, ... |
            |                    -------------      -------------------------
     (1)    |
-------------------              -------------      -------------------------
| EventLoopGroup  | -----------> | EventLoop | ---> | Channel, Channel, ... |
-------------------              -------------      -------------------------
            |
            |                    -------------      -------------------------
            -------------------> | EventLoop | ---> | Channel, Channel, ... |
                                 -------------      -------------------------
(1): 所有的 EventLoop 都由这个 EventLoopGroup 分配
(2): 每个 EventLoop 将处理分配给它的所有 Channel 的所有事件和任务, 每个 EventLoop 都和一个 Thread 相关联
(3): EventLoopGroup 将为每个新创建的 Channel 分配一个 EventLoop; 在每个 Channel 的整个生命周期内, 所有的操作都将由相同的 Thread 执行
```
一旦一个 Channel 被分配给一个 EventLoop, 它将在它的整个生命周期中都使用这个 EventLoop (以及相关联的 Thread); 另外, 因为一个 EventLoop 通常会被用于支撑多个 Channel, 所以对于所有相关联的 CHannel 来说, ThreadLocal 都将是一样的

###### 阻塞传输
这里的每一个 CHannel 都将被分配给一个 EventLoop (以及它的 Thread)
```
                                 -------------      -----------
            -------------------> | EventLoop | ---> | Channel |
            |                    -------------      -----------
            |
-------------------              -------------      -----------
| EventLoopGroup  | -----------> | EventLoop | ---> | Channel |
-------------------              -------------      -----------
            |
            |                    -------------      -----------
            -------------------> | EventLoop | ---> | Channel |
                                 -------------      -----------
```
这如同异步传输一样, 可以保证每个 Channel 的 I/O 事件都将只会被一个 Thread 处理

#### 小结
TODO
