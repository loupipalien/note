### Java 中的 13 个原子操作类
JDK 5 开始提供了 java.uitl.concurrent.atomic 包, 该包中包含的类属于 4 种类型的原子更新方式, 分别是原子更新基本类型, 原子更新数组, 原子更新引用, 原子更新属性; atomic 包中的基本都是使用 Unsafe 实现的包装类

#### 原子更新基本类型类
- AtomicBoolean: 原子更新布尔类型
- AtomicInteger: 原子更新整型
- AtomicLong: 原子更新长整型

AtomicInteger 类中的常用方法

| 方法签名 | 描述 |
| :------------- | :------------- |
| int addAndSet(int delta) | 以原子方式将输入的值和实例中的值相加后返回 |
| boolean compareAndSet(int expect, int update) | 如果输入的数值等于预期值, 则以原子的方式将该值设置为输入的值 |
| int getAndIncrement() | 以原子的方式加一 |
| void lazySet(int newValue) | 最终会设置为 newValue, 使用 lazySet 设置值后, 可能导致其他线程在之后的一小段时间内还是可以读取到旧值 |
| int getAndSet(int newValue) | 以原子的方式设置为 newValue 的值, 并且返回旧值 |

atomic 包提供了三种基本类型的原子更新, 是使用 Unsafe 类实现的, Unsafe 类只提供了三种 CAS 的方法
```
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```
其他基本类型的原子更新实现可以参考 AtomicBoolean 的实现, 将整型先转换为整型, 再使用 compareAndSwapInt 进行 CAS

#### 原子更新数组
- AtomicIntegerArray: 原子更新整型数组里的元素
- AtomicLongArray: 原子更新长整型数组里的元素
- AtomicReferenceArray: 原子更新引用类型数组里的元素

AtomicIntegerArray 类中的常用方法

| 方法签名 | 描述 |
| :------------- | :------------- |
| int addAndGet(int i, int delta) | 以原子方式将输入的值和数组中索引 i 的值相加后返回 |
| boolean compareAndSet(int i, int expect, int update) | 如果输入的数值等于预期值, 则以原子的方式将数组位置 i 的值设置为输入的值 |

需要注意的是, AtomicIntegerArray 构造方法会将参数数组复制一份, 而不会影响传入的数组

#### 原子更新引用类型
- AtomicReference: 原子更新引用类型
- AtomicReferenceFieldUpdater: 原子更新引用类型里的字段
- AtomicMarkableReference: 原子更新带有标记位的引用类型

#### 原子更新字段类
- AtomicIntegerFieldUpdater: 原子更新整型的字段更新器
- AtomicLongFieldUpdater: 原子更新长整型的字段更新器
- AtomicStampedFieldUpdater: 原子更新带有版本号的引用类型; 该类型将整数值与引用关联, 可用于原子的更新数据和数据的版本号, 可以解决使用 CAS 进行原子更新时出现的 ABA 问题
