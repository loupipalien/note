### 并发编程的挑战
并发编程的目的是为了让程序运行得更快, 但并不是启动更多的线程就能让程序最大限度地并发执行

#### 上下文切换
即使是单核处理器也支持多线程执行代码, CPU 通过给每个线程分配 CPU 时间片来实现这个机制; CPU 通过时间片分配算法来循环执行任务, 当前任务执行一个时间片后会切换到下一个任务, 但是在切换前会保存上一个任务的状态, 以便下次切换会这个任务时, 可以加载这个任务的状态; 任务从保存到再加载的过程就是一次上下文切换, 而遮掩的上下文切换是有代价的

##### 多线程一定快吗
```
public class ConcurrencyTest {
    private static final long count = 100001L;

    public static void main(String[] args) throws InterruptedException {
        concurrency();
        serial();
    }

    private static void concurrency() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                int a = 0;
                for (long i = 0; i < count; i++) {
                    a += 5;
                }
            }
        });
        thread.start();

        int b = 0;
        for (long i = 0; i < count; i++) {
            b--;
        }
        thread.join();
        long time = System.currentTimeMillis() - start;
        System.out.println("concurrency: " + time + "ms, b = " + b);
    }

    private static void serial() {
        long start = System.currentTimeMillis();
        int a = 0;
        for (long i = 0; i < count; i++) {
            a += 5;
        }

        int b = 0;
        for (long i = 0; i < count; i++) {
            b--;
        }
        long time = System.currentTimeMillis() - start;
        System.out.println("serial: " + time + "ms, b = " + b);
    }
}
```
以上代码在并发执行累加操作不超过百万次是, 串行执行比并行快, 这时因为线程有创建和上下文切换开销

##### 测试上下文切换次数和时长
- 使用 Lmbench3 可以测试上下文切换的时长
- 使用 vmstat 可以测试上下文切换的次数

##### 如何减少上下文切换
- 无锁并发编程: 多线程竞争锁时, 会引起上下文的切换, 所以多线程处理数据时可以用一些办法来避免使用锁; 例如将数据 id 按照 hash 分段, 不同的线程处理不同段的数据
- CAS 算法: Java 的 Atomic 包使用 CAS 算法来更新数据, 而不需要加锁
- 使用最少线程: 避免创建不需要的线程; 例如任务很少时但创建了大量的线程来处理, 这样会造成大量线程都处于等待状态
- 协程: 在单线程里实现多任务的调度, 并在单线程里维持多个任务间的切换

##### 减少上下文切换实战
TODO

#### 死锁
```
public class DeadLockDemo {
    private static String A = "A";
    private static String B = "B";

    public static void main(String[] args) {
        new DeadLockDemo().deadLock();
    }

    private void deadLock() {
        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (A) {
                    try {
                        Thread.currentThread().sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (B) {
                        System.out.println("t1");
                    }
                }
            }
        });

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (B) {
                    synchronized (A) {
                        System.out.println("t2");
                    }
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```
以上代码演示了什么是死锁, 以下是几个避免死锁的常见方法
- 避免一个线程同时获取多个锁
- 避免一个线程在锁内同时占用多个资源, 尽量保证每个锁只占用一个资源
- 尝试使用定时锁, 使用 lock.tryLock(timeout) 来替代使用内部锁机制
- 对于数据库, 加锁和解锁必须在一个数据库连接里, 否则会出现解锁失败的情况

#### 资源限制的挑战
TODO
