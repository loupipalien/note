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
当使用 ArrayList 使用无参的构造函数时, 在第一次 add 时分配默认 10 的容量, 后续每次扩容都会调用 Arrays.copyOf() 方法; 如果需要放置 1000 个元素则需要扩容 13 次; 如果知道容量需求可以使用有参构造函数, 从未避免扩容和数组复制的额外开销

##### HashMap
HashMap 使用无参构造函数, 默认的 capacity 是 16, loadFactor 是 0.75, 基于这两个参数的乘积, 内部使用 threshold 变量表示 HashMap 中能放入的元素个数; HashMap 容量不会在 new 的时候分配, 而是在第一次 put 时创建
```
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
为提高运算速度, 设置 HashMap 的容量大小为 $ 2^n $; 如果初始化时指定了 initialCapacity, 则会先计算出不小于 initialCapacity 的 2 次幂存入 threshold, 第一次 put 时会按照这个值初始化数组大小, 后续每次扩容都增加 2 倍; 如果知道容量需求可以使用有参构造函数, 从未避免扩容带来的额外开销  

#### 数组与集合
声明数组和赋值的方式如下
```
String[] array1 = {"a", "b"};
String[] array2 = new String[2];
array2[0] = "a";
array2[1] = "b";
```
数组是的大小是固定的, 动态大小的数组集合提供了 Vector 和 ArrayList 两个类, 前者线程安全但性能较差, 后者线程不安全但性能好  
数组和集合都是用来存储对象的容器, 两者之间可以互相转换, 但在数组转集合的过程中需要注意是否使用了视图的方式直接返回数组中的元素, Arrays.asList() 返回的 ArrayList 是 Arrays 的内部类, 此类仅仅是数组的视图, 实际操作还是对应到数组
```
private static class ArrayList<E> extends AbstractList<E>
    implements RandomAccess, java.io.Serializable
{
    private static final long serialVersionUID = -2764017481108945198L;
    private final E[] a;

    ArrayList(E[] array) {
        a = Objects.requireNonNull(array);
    }

    @Override
    public int size() {
        return a.length;
    }

    @Override
    public Object[] toArray() {
        return a.clone();
    }

    @Override
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        int size = size();
        if (a.length < size)
            return Arrays.copyOf(this.a, size,
                                 (Class<? extends T[]>) a.getClass());
        System.arraycopy(this.a, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    @Override
    public E get(int index) {
        return a[index];
    }

    @Override
    public E set(int index, E element) {
        E oldValue = a[index];
        a[index] = element;
        return oldValue;
    }

    @Override
    public int indexOf(Object o) {
        E[] a = this.a;
        if (o == null) {
            for (int i = 0; i < a.length; i++)
                if (a[i] == null)
                    return i;
        } else {
            for (int i = 0; i < a.length; i++)
                if (o.equals(a[i]))
                    return i;
        }
        return -1;
    }

    @Override
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }

    @Override
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(a, Spliterator.ORDERED);
    }

    @Override
    public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        for (E e : a) {
            action.accept(e);
        }
    }

    @Override
    public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        E[] a = this.a;
        for (int i = 0; i < a.length; i++) {
            a[i] = operator.apply(a[i]);
        }
    }

    @Override
    public void sort(Comparator<? super E> c) {
        Arrays.sort(a, c);
    }
}
```
集合转数组时需要注意一点, 给定的数组长度要大于等于集合的大小, 返回会返回一个元素全为 null 的数组
```
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}

// Arrays
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```
其中 elementData 是 Arraylist 集合对象中存储数据的数组, 这个数组由 transient 修饰, 表示此字段在类的序列化时被忽略; 因为集合序列化时系统会调用 writeObject 写入流中, 在网络客户端反序列化的 readObject 时, 会重新赋值到新对象的 elementData 中, 这样 "多此一举" 是因为 elementData 中的容量常大于实际存储的元素数量, 所以只需要发送真正有实际值的元素数据即可

#### 集合与泛型
List, List<Object>, List<Integer>, List<?> 之间的区别
```
public static void main(String[] args) {
    // 泛型出现之前的集合定义方式
    List a1 = new ArrayList();
    a1.add(new Object());
    a1.add(new Integer(1));
    a1.add(new String("Hello World 1"));

    // 增加了泛型 Object
    List<Object> a2 = a1;
    a2.add(new Object());
    a2.add(new Integer(2));
    a2.add(new String("Hello World 2"));

    // 增加了泛型 Integer
    List<Integer> a3 = a1;
    a3.add(new Integer(3));
    // 以下两行编译出错
    a3.add(new Object());
    a3.add(new String("Hello World 3"));

    // 增加了泛型通配符
    List<?> a4 = a1;
    a4.remove(0);
    a4.clear();
    // 编译出错
   a4.add(new Object());
}
```
List 不带泛型, 不检查添加的元素类型; 由于泛型不是协变的, 所以 List<Object> 和 List<Integer> 在编译时是完全两种不同的类型, 没有继承关系; List<?> 可以接受任何类型参数的 List 引用赋值, 可以 remove 和 clear, 但是不能 add, 因为不知道 ? 具体是什么类型, 所以 List<?> 一般作为参数来接受外部的集合, 或者返回一个不知道具体类型的集合  
<? extend T> 和 <? super T> 是 <?> 的约束表现形式, 同样一般作为参数来接受外部的集合, 或者返回一个不知道具体类型的集合; 表示的参数是生产者使用 <? extend T>, 参数表示是消费者则使用 <? super T>, 简称 PECS

#### 元素比较

##### Comparable 和 Comparator
Comparable 是自己和自己比较, 由类自己实现, 可以看作是自营性质的比较器; Comperator 是第三方实现的比较器, 可以看作平台性质的比较器; 不管是 Comparable 还是 Comperator 比较时小于情况返回 -1 或负数, 大于情况返回 1 或正数, 相等返回 0   
Comperator 最典型的应用是在 Arrays.sort 中作为比较器参数进行排序
```
public static <T> void sort(T[] a, Comparator<? super T> c) {
    if (c == null) {
        sort(a);
    } else {
        if (LegacyMergeSort.userRequested)
            legacyMergeSort(a, c);
        else
            TimSort.sort(a, 0, a.length, c, null, 0, 0);
    }
}
```
sort 方法中的 TimSort 算法是归并排序 (Merge Sort) 和插入排序 (Insertion Sort) 优化后的算法
- 归并排序
长度为 1 的数组是排序好的, 有 n 个元素的集合可以看作是 n 个长度为 1 的有序子集合; 对有序子集合进行两两归并, 并保证结果子集合有序, 最后得到 n/2 个长度为 2 的有序子集合; 重复上一步骤直到所有元素归并成一个长度为 n 的有序集合; 在此排序过程中, 主要工作都在归并处理, 如何使归并过程更快, 或者减少归并次数, 成为优化归并排序的重点
- 插入排序
长度为 1 的数组是有序的, 当有了 k 个已排序的元素, 将第 k + 1 个元素插入已有的 k 个元素中合适的位置, 就会得到一个长度为 k + 1 已排序的数组; 假设有 n 个元素且已经升序排列的数组, 并且在数组尾端有 n + 1 个元素的位置, 此时如果想要添加一个新的元素并保持数组有序, 根据插入排序, 可以将新元素放到第 n + 1 个位置上, 然后从后两两比较, 如果新值较小则交换位置, 直到新元素到达正确的位置

TimSort 算法相对于传统归并排序减少了归并次数, 相对于插入排序引入了二分排序, 提升了排序效率; TimSort 算法对于已经部分排序的数组, 时间复杂度最优可达 $ O(n) $, 对于随机数组, 时间复杂度最优可达 $ O(nlog_n) $, 平均时间复杂度为 $ O(nlog_n) $; JDK7 中使用 TimSort 取代归并排序, 主要有两个优化
- 归并排序的分段不再从单个元素开始, 而是每次先查找当前最大的排序号的数组片段 run, 然后对 run 进行扩展并利用二分排序, 之后将该 run 与其他已经排序好的 run 进行归并, 产生排序好大 run
- 引入二分排序 (binarySort), 二分排序是对插入排序的优化, 在插入排序中不再是从后往前逐个元素对比, 而是引入二分查找的思想, 将一次查找新元素合适位置的时间复杂度从  $ O(n) $ 降低到 $ O(log_n) $

##### hashCode 和 equals
hashCode 和 equals 用来标识对象, 两个方法协同工作可用来判断两个对象是否相等
