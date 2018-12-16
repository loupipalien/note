### 数据结构与集合

#### 数据结构

##### 数据结构的定义
TODO

##### 数据结构分类
- 线性结构
0 至 1 个直接前驱和直接后继; 当线性结构非空时, 有唯一的首元素和尾元素, 除两者之外, 所有的元素都有唯一的直接前驱和后继, 线性结构包括顺序表, 链表, 栈, 队列等
- 树结构
0 至 1 个直接前驱和 0 至 n 个直接后继 (n 大于等于 2)
- 图结构
0 至 1 个直接前驱和 0 至 n 个直接后继 (n 大于等于 2); 包括简单图, 多重图, 有向图, 无向图等
- 哈希结构
没有直接前驱和直接后继; 哈希结构通过某种特定的哈希函数将索引与存储的值关联起来, 它是一种查找效率非常高的数据结构

从最好到最坏的常用时间复杂度是: 常数级 $ O(1) $, 对数级 $ O(log_n) $, 线性级 $ O(n) $, 线性对数级 $ O(nlog_n) $, 平方级 $ O(n^2) $, 立方级 $ O(n^3) $, 指数级 $ O(2^n) $

#### 集合框架图
集合框架主要分两类: 第一类是按照单个元素存储的 Collection, List 和 Set 都实现了 Collection 接口; 第二类是按照 Key-Value 存储的 Map

##### List 集合
List 集合是线性数据结构的主要实现, 该体系最常用的是 ArrayList 和 LinkedList  
ArrayList 是容量可以改变的非线程安全集合, 内部实现使用数组进行存储, 集合扩容时会创建更大的数组空间, 把原有数据复制到新数组中; ArrayList 支持对元素的随机访问, 但插入和删除数据较慢  
LinkedList 的本质是双向链表, 除了继承 AbstractList 之外, 还实现了 Deque 接口, 这个接口同时具有队列和栈的性质, LinkedList 中包含了三个重要成员: size, first, last; LinkedList 的插入和删除速度更快, 但随机访问速度较慢

##### Queue 集合
Queue 是一种先进先出的数据结构, 队列是一种特殊的线性表, 只允许在表的一段进行获取操作, 在表的另一端进行插入操作; 自从 BlockQueue 出现以来, 在各种高并发编程的场景中, 由于本身 FIFO 的特性和阻塞操作的特点, 经常被作为 Buffer (数据缓冲区) 使用

##### Map 集合
Map 集合是以 Key-Value 键值对作为存储元素实现的哈希结构; 最常使用的是 HashMap, TreeMap, ConcurrentHashMap

##### Set 集合
Set 是不允许出现重复元素的集合类型, 最常使用的是 HashSet, TreeSet, LinkedHashSet; HashSet 是使用 HashMap 来实现的, 只是 Value 固定为一个静态对象, 使用 Key 保证集合元素的唯一性, 但不保证集合元素的顺序; TreeSet 是使用 TreeMap 来实现的, 底层为树结构, 在添加新元素到集合中时, 按照某种比较规则将其插入到合适的位置, 保证插入后的集合仍然是有序的; LinkedHashSet 继承 HashSet, 具有 HashSet 的优点, 内部使用链表维护了元素插入的顺序

#### 集合初始化
集合初始化通常进行分配容量, 设置特定参数等相关工作

##### ArrayList
```
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

// MARK: 计算容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;    // MARK: 修改计数

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);    // MARK: 扩容 50%
    if (newCapacity - minCapacity < 0)    // MARK: 扩容 50% 后反而更小 (当 oldCapacity >> 1 溢出成负数), 则使用 minCapacity
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);  // MARK: 超大容量
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

// MARK: Integer.MAX_VALUE - 8 ~ Integer.MAX_VALUE
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```
当使用 ArrayList 使用无参的构造函数时, 在第一次 add 时分配默认 10 的容量, 后续每次扩容都会调用 Arrays.copyOf() 方法; 如果需要放置 1000 个元素则需要扩容 13 次, 如果知道容量需求可以使用有参构造函数, 从未避免扩容和数组复制的额外开销
