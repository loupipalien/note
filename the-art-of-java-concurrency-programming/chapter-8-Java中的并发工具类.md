### Java 中的并发工具类

#### 等待多线程完成的 CountDownLatch
CountDownLatch 允许一个或多个线程等待其他线程完成操作
```
public class CountDownLatchTest {
    static CountDownLatch c = new CountDownLatch(2);

    public static void main(String[] args) throws InterruptedException {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(1);
                c.countDown();
                System.out.println(2);
                c.countDown();
            }
        }).start();

        c.await();
        System.out.println(3);
    }
}
```
CountDownLatch 的构造函数接受一个 int 参数作为计数器, 调用 countDown 方法时, N 就会减 1, CountDownLatch 的 await() 方法会阻塞当前线程, 直到 N 变为零 (一个线程调用 countDown() 方法 happens-before 于另一个线程调用 await() 方法)

#### 同步屏障 CyclicBarrier
CyclicBarrier 的字面意思是可循环使用的屏障, 它可以让一组线程到达一个屏障 (也可以叫同步点) 时被阻塞, 直到最后一个线程到达屏障, 屏障才会开门, 所有被拦截的线程才会继续进行

##### CyclicBarrier 简介
CyclicBarrier 默认的构造方法是 CyclicBarrier(int parties), 其参数表示屏障拦截的线程数量, 每个线程调用 await 方法告诉 CyclicBarrier 表示到达屏障

#### CyclicBarrier 的应用场景
CyclicBarrier 可以用于多线程计算数据, 最后合并计算结果的场景
```
import java.util.Map;
import java.util.concurrent.*;

public class BankWaterService implements Runnable {

    /**
     * 创建 4 个屏障, 处理完之后执行当前类的 run 方法
     */
    private CyclicBarrier c = new CyclicBarrier(4, this);

    /**
     * 假设只有 4 个 sheet, 所以启动 4 个线程
     */
    private Executor executor = Executors.newFixedThreadPool(4);

    /**
     * 保存每个 sheet 计算出的银流结果
     */
    private ConcurrentHashMap<String, Integer> sheetBankWaterCount = new ConcurrentHashMap<>();

    private void count() {
        for (int i = 0; i < 4; i++) {
            executor.execute( () -> {
                // 计算当前 sheet 的银流结果,计算代码省略
                sheetBankWaterCount.put(Thread.currentThread().getName(), 1);
                // 插入屏障
                try {
                    c.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
        }
    }

    @Override
    public void run() {
        int result = 0;
        // 汇总结果
        for (Map.Entry<String,Integer> entry : sheetBankWaterCount.entrySet()) {
            result += entry.getValue();
        }
        // 输出结果
        sheetBankWaterCount.put("result", result);
        System.out.println(result);
    }

    public static void main(String[] args) {
        new BankWaterService().count();
    }
}
```

##### CyclicBarrier 和 CountDownLatch 的区别
CountDownLatch 的计数器只能使用一次, 而 CyclicBarrier 的计算器可以使用 reset() 方法重置; CyclicBarrier 还提供其他有用的方法, 例如 getNumberWaiting 方法可以获得 CyclicBarrier 阻塞的线程数量, isBroken 方法用来了解阻塞的线程是否被中断

#### 控制并发线程数的 Semaphore
Semaphore (信号量) 是用来控制同时访问特定资源的线程数量, 它通过协调各个线程, 以保证合理的使用公共资源

##### 应用场景
Semaphore 可以用于做流量控制, 特别是公用资源有限的应用场景, 例如数据库连接
```
public class SemaphoreTest {

    private static final int THREAD_COUNT = 20;

    private static ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_COUNT);

    private static Semaphore s = new Semaphore(10);

    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
            threadPool.execute(() -> {
                try {
                    s.acquire();
                    System.out.println("save data.");
                    s.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        threadPool.shutdown();
    }
}
```

##### 其他方法
- int availablePermits(): 返回此信号量中当前可用的许可证数
- int getQueueLenght(): 返回正在等待获取许可证的线程数
- boolean hasQueuedThreads(): 是否有线程正在等待获取许可证
- void reducePermits(int reduction): 减少 reduction 个许可证
- Collection getQueuedThreads(): 返回所有等待获取许可证的线程集合

#### 线程间交换数据的 Exchanger
Exchanger (交换者) 是一个用于线程间协作的工具类; Exchanger 用于进行线程间的数据交换; 它提供一个同步点, 在这个同步点, 两个线程可以交换彼此的数据, 这两个线程通过 exchange() 方法交换数据, 如果第一个线程先执行 exchange() 方法, 当两个线程都到达同步点时, 这两个线程就可以交换数据, 将本线程生产出来的数据传递给对方
```
public class ExchangerTest {

    private static final Exchanger<String> exchanger = new Exchanger<>();

    private static ExecutorService threadPool = Executors.newFixedThreadPool(2);

    public static void main(String[] args) {
        threadPool.execute(() -> {
            try {
                String A = "线程 A";
                String B = exchanger.exchange(A);
                System.out.println(A + " => " + B);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        threadPool.execute(() -> {
            try {
                String B = "线程 B";
                String A = exchanger.exchange(B);
                System.out.println(B + " => " + A);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        threadPool.shutdown();
    }
}
```
