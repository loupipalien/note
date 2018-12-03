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
