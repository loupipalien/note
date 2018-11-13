### 并发
线程机制允许同时进行多个活动, 并发程序设计比单线程设计要困难得多, 因为有更多的东西可能出错, 也难以重现失败

#### 第 66 条: 同步访问共享的可变数据
关键字 synchronized 可以保证在同一时刻, 只有一个线程可以执行某一个方法, 或者某一代码块; 同步的意义在于: 不仅可以阻止一个线程看到对象处于不一致的状态中, 它还可以保证进入同步方法或者同步代码块的每个线程, 都看到由同一个锁保护的之前所有的修改效果  
Java 语言规范保证读写一个变量是原子性的, 除非这个变量的类型为 long 或者 double, 即读一个非 long 或 double 类型的变量, 可以保证返回的值是某个线程保存在该变量中的, 即使多个线程在没有同步的情况下并发的修改这个变量也是如此; 虽然语言规范保证了读取原子数据的时候, 不会看到任意的数值, 但它并不保证一个线程写入的值对另一个线程是可见的, 为了在线程之间进行可靠的通信, 也为了互斥访问, 同步是必要的; 这是由于 Java 语言规范中的内存模型 (memory model), 它规定了一个线程所做的变化可是以及如何变成对其他线程可见  
如果对共享的可变数据的访问不能同步, 其后果非常严重, 即使这个变量是原子可读写的
```
// Broken! - How long would you expect this program to run?
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throw InterruptedException {
        Thread backgroudThread = new Thread(new Runnable() {
            public void run() {
                int i = 0;
                while (!stopRequested) {
                    i++;
                }
            }
        });
        backgroudThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
看似主线程在 1 秒之后将 stopRequest 设置为 true, backgroudThread 线程应会停止, 但实际上 backgroudThread 线程并不会停下来; 这里的问题出在没有同步, 不能保证后台线程 "看到" 主线程对 stopRequest 的值所做的改变; 修正的方法如下
```
// Properly synchronized cooperative thread termination
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequest = true;
    }

    private start synchronized bpplean stopRequested() {
        return stopRequested
    }

    public static void main(String[] args) throw InterruptedException {
        Thread backgroudThread = new Thread(new Runnable() {
            public void run() {
                int i = 0;
                while (!stopRequested()) {
                    i++;
                }
            }
        });
        backgroudThread.start();
        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```
StopThread 中被同步方法的同步动作即使没有同步也是原子性的, 换句话说, 这些方法的同步只是为了它的通信效果, 而不是为了互斥访问; 虽然循环中每个迭代中的同步开销很小, 但还有其他开销更小性能更好的替代方法达到此目的; 即将 stopRequested 声明为 volatile, 第二种版本的 StopThread 中的锁就可以去掉了; 虽然 volatile 修饰符不执行互斥访问, 但它可以保证任何一个线程在读取该域的时候都将看到的是最近刚刚写入的值
```
// cooperative thread termination with a volatile field
public class StopThread {
    private static volatile boolean stopRequested;

    public static void main(String[] args) throw InterruptedException {
        Thread backgroudThread = new Thread(new Runnable() {
            public void run() {
                int i = 0;
                while (!stopRequested) {
                    i++;
                }
            }
        });
        backgroudThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
在使用 volatile 时候无必小心, 考虑以下获取下一个序列号的方法
```
// Broken - required synchronization
private static volatile long nextSerialNumber = 0;
public static long getnerateSerialNumber() {
    return nextSerialNumber++;
}
```
这段程序没有同步会失败的原因在于增量操作符 (++) 不是原子的, 它在 nextSerialNumber 域中执行两项操作: 首先读取它的值, 加一后再写回; 如果第二个线程在第一个线程读取旧值和写回新值之间读取了这个域, 那么第二个线程和第一个线程就会返回相同的序列号; 修正 getnerateSerialNumber 的方法一种是给方法增加 synchronized 修饰符加上锁, 这样也可以并应该去掉 volatile 修饰符; 另一种方法是使用原子类 java.util.concurrent.atomic.AtomicLong, 并且能够获得比加锁更好的性能
```
private static AtomicLong nextSerialNumber = 0;
public static long getnerateSerialNumber() {
    return nextSerialNumber.getAndIncrement();
}
```
让一个线程在短时间修改一个数据对象, 然后与其他线程共享是可以接受的; 只同步共享对象引用的动作, 然后其他线程没有进一步同步也可以读取对象, 只要它没有再被修改, 这种对象被称作事实上不可变的 (effectively immutable); 将这种对象引用从一个线程传递到其他线程被称作安全发布 (safe publication), 安全发布对象引用有许多种办法: 可以将其保存在静态域中, 作为类初始化的一部分; 可以保存到 volatile 域, final 域 或者通过正常锁定访问的域中; 或者可以将它放到并发的集合中 (见第 69 条)  
简而言之, 当多线程恭喜啊可变数据的时候, 每个读或者写数据的线程都必须执行同步, 如果没有同步, 就无法保证一个线程所做的修改可以被另一个线程获知; 如果只需要线程之间的交互通信, 而不需要互斥, volatile 修饰符就是一种可以接受的同步形式, 但是要正确的使用!

#### 第 67 条: 避免过度同步
为了避免火星失败和安全失败, 在一个被同步的方法或者代码块中, 永远不要放弃对客户端的放弃; 换句话说, 在一个被同步的区域内部, 不要调用设计成被要覆盖的方法, 或者是由客户端以函数对象的形式提供的方法 (见第 21 条); 从包含该同步区域的类的角度来看, 这样的方法是外来的, 该类不知道该方法会做什么事情, 也无法控制它; 根据外来方法的作用, 从同步区域中调用它会导致异常, 死锁或者数据损坏
```
// Broken - invokes alien method from synchronized block!
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) { super(set); }

    private final List<SetObserver<E>> observers = new ArrayList<SerObserver<E>>();

    public void addObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.add(observer);
        }
    }

    public void removeObserver(SetObserver<E> observer) {
        synchronized(observers) {
            observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized(observers) {
            for(SerObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            // call notifyElementAdded
            result != add(element);
        }
        return result;
    }
}

public interface SetObserver<E> {
    // Invoke when an element is added to the observerable set
    void added(ObservableSet<E>, E element)
}
```
Observer 通过调用 addObserver 方法预定通知, 通过调用 removeObserver 方法取消预定; 如果只是粗略的检验一下, ObservableSet 会显得很正常
```
public static void main(String[] args) {
    ObservableSet<Integer> set = new ObservableSet<Integer>(new HashSet<Integer>);

    set.addObserver(new SetObserver<Integer> {
        public void added(ObservableSet<Integer> s, Integer e) {
            System.out.println(e);
        }
    });

    for (int i = 0; i < 100; i++) {
        set.add(i);
    }
}
```
如果在尝试一些更复杂的例子
```
set.addObserver(new SetObserver<Integer> {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
            s.removeObserver(this);
        }
    }
});
```
以上会在打印 0-23 的数字之后, 抛出 ConcurrentModificationException; 问题在于当 notifyElementAdded 调用观察者的 added 方法时, 正处于遍历 observers 列表中, added 方法中调用可观察集合的 removeObserver 方法, 从而调用 observers.remove, 这时非法的; notifyElementAdded 方法迭代是在一个同步块中, 可以防止并发修改, 但是无法防止迭代线程本身回调到可观察的集合中, 也无法防止修改它的 observers 列表
```
set.addObserver(new SetObserver<Integer> {
    public void added(ObservableSet<Integer> s, Integer e) {
        System.out.println(e);
        if (e == 23) {
            ExecutorService executor = Executors.newSingleThreadExecutor();
            final SetObserver<Integer> observer = this;
            try {
                executor.submit(new Runnable() {
                      public void run() {
                          s.removeObserver(this);
                      }
                }).get();
            } catch (ExcutionException e) {
                throw new AssertionError(e.getCause());
            } catch (InterruptedException e) {
                hrow new AssertionError(e.getCause());
            } finally {
                executor.shutdown();
            }
        }
    }
});
```
使用后台线程调用造成了死锁, 它企图获得 observers 的锁, 但是以及被主线程锁住而无法获得, 这样造成了死锁  
幸运的是, 解决这个问题可将外来方法移出同步代码块并不是十分困难, 对于 notifyElementAdded 方法将 observers 拍张快照, 然后没有锁也可以安全的操作了
```
// Alien method moved outside of synchronized block - open calls
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot = null;
    synchronized(observers) {
        snapshot = new ArrayList<>(snapshot);
    }

    for(SerObserver<E> observer : snapshot) {
        observer.added(this, element);
    }
}
```
事实上要将外来方法的调用移出同步的代码块, 使用并发集合中的 CopyOnWriteArrayList 更好, 这是 ArrayList 的一种变体, 通过重新拷贝整个底层数组, 在这里实现所有的写操作
```
// Thread-safe observable set with CopyOnWriteArrayList
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<SerObserver<E>>();

public void addObserver(SetObserver<E> observer) {
      observers.add(observer);
}

public void removeObserver(SetObserver<E> observer) {
    observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for(SerObserver<E> observer : observers) {
        observer.added(this, element);
    }
}
```
如果一个可变的类要并发使用, 应该使这个类变成是线程安全的 (见第 70 条), 通过内部同步还可以获得明显比从外部锁定整个对象更高的并发性, 否则就不要在内部同步而是让客户必要的时候在外部同步  
简而言之, 为了避免死锁和数据破坏, 千万不要从同步区域内部调用外来方法, 更一般的讲是要尽量限制同步区域内部的工作量

#### 第 68 条: executor 和 task 优先于线程
Java 1.5 发行版本中增加了 java.util.concurrent 包, 这个包中包含了一个 Executor Framework, 这是一个很灵活的基于接口的任务执行工具; 在编程中尽量不要编写自己的工作队列, 而且还应该尽量不直接使用线程; 使用 Executor Framework 执行线程是更合理的

#### 第 69 条: 并发工具优先于 wait 和 notify
TODO

#### 第 70 条: 线程安全性的文档化
TODO

#### 第 71 条: 慎用延迟初始化
TODO

#### 第 72 条: 不要依赖于线程调度器
TODO

#### 第 73 条: 避免使用线程组
