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
hashCode 和 equals 用来标识对象, 两个方法协同工作可用来判断两个对象是否相等; 根据生成的哈希将数据离散开来, 可以使存取元素更快, 对象通过调用 Object.hashCode() 生成哈希值, 由于不可避免会存在哈希值冲突的情况, 因此当 hashCode 相同时在调用 equals() 进行比较; 但是当 hashCode 不同时可以直接判定 Objects 不同, 从而跳过 equals, 这加快了处理效率; Object 类定义中对 hashCode 和 equals 要求如下
- 如果两个对象的 equals 的结果是相等的, 则两个对象的 hasdCode 的返回结果也必须相等
- 任何时候覆写 equals, 都必须同时覆写 hasdCode

在 Map 和 Set 类集合中的 get() 方法实现, 首先判断 hasdCode 方法再判断 equals() 的结果
```
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node                // MARK: 先比较 hash 值
            ((k = first.key) == key || (key != null && key.equals(k))))     // MARK: 再比较地址或者 equals 方法
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```
如果自定义对象作为 Map 的键, 那么必须同时覆写 hasdCode 和 equals 方法, 此外 Set 也是如此; 如果不覆写 hasdCode 但是覆写了 equals 方法, 当两个对象的 equals 方法返回 true 时, 在集合中仍然是两个不同的元素, 因为默认的 hasdCode 码是根据地址生成的  
另外, equals() 方法的实现方式与类的具体处理逻辑有关, 但又各不相同, 应该尽量分析源码确定结果, 例如以下
```
public static void main(String[] args) {
    LinkedList<Integer> linkedList = new LinkedList<>();
    linkedList.add(1);
    ArrayList<Integer> arrayList = new ArrayList<>();
    arrayList.add(1);
    System.out.println(arrayList.equals(linkedList));
}
```
以上代码会输出  true, 这是由于 ArrayList 和 LinkedList 都是继承了 AbstractList 的 equals() 方法, 只判断了接口类型和其中元素
```
public boolean equals(Object o) {
    if (o == this)
        return true;
    if (!(o instanceof List))
        return false;

    ListIterator<E> e1 = listIterator();
    ListIterator<?> e2 = ((List<?>) o).listIterator();
    while (e1.hasNext() && e2.hasNext()) {
        E o1 = e1.next();
        Object o2 = e2.next();
        if (!(o1==null ? o2==null : o1.equals(o2)))
            return false;
    }
    return !(e1.hasNext() || e2.hasNext());
}
```

#### fail-fast 机制
这时一种对集合遍历操作时的错误检测机制, 在遍历中途出现意料之外的修改时, 通过 unchecked 的异常暴力的反馈处来; 这种机制长出现在多线程的环境下, 当前线程会维护一个技术比较器, 即 expectedModCount, 如果这两个数据不相等则抛出异常; java.util 包下的所有集合类都是 fail-fast, 而 java.util.concurrent 包中的集合类都是 fail-safe 的, 与 fail-fast 不同的是, 遍历时先保存一个快照在遍历, 不会被中途意外的修改打断
以下 ArrayList.subList() 示例进一步阐述 fail-fast 机制
```
public static void main(String[] args) {
    List<String> masterList = new ArrayList<>();
    masterList.add("one");
    masterList.add("two");
    masterList.add("three");
    masterList.add("four");
    masterList.add("five");

    List<String> branchList = masterList.subList(0, 3);

    // 如果注释以下三行代码, 会导致 branchList 操作抛错
    masterList.remove(0);
    masterList.add("ten");
    masterList.clear();

    // 如果下方四行代码正确执行
    branchList.clear();
    branchList.add("six");
    branchList.add("seven");
    branchList.remove(0);

    // 正常遍历只有一个 seven
    for (String str : branchList) {
        System.out.println(str);
    }

    // 子列表修改会导致主列表改动, 期望输出 ["seven", "four", "five"]
    System.out.println(masterList);
}
```
但实际上, masterList 的操作导致了 branchList 操作的 fail-fast, 抛出 ConcurrentModificationException 异常; 此外 ArrayList$SubList 类没有实现序列化接口, 不可以序列化  
在使用迭代器或者使用增强 for (实际也是使用迭代器实现, 可使用 jad 等工具反编译查看) 遍历 List 时, 操作移除非倒数第二个元素时会抛出 ConcurrentModificationException 异常, 而移除倒数第二个元素时, 则能正确执行
```
public static void main(String[] args) {
    List<String> list = new ArrayList<>();
    list.add("one");
    list.add("two");
    list.add("three");

    // 移除倒数第二个元素可以正确执行
    for (String str : list) {
        if (str.equals("two")) {
            list.remove(str);
        }
    }
    // 以上增强 for 反编译结果
    // do {
    //     if(!iterator.hasNext())
    //         break;
    //     String str = (String)iterator.next();
    //     if(str.equals("two"))
    //         list.remove(str);
    // } while(true);

    System.out.println(list);
}
```
移除倒数第二个元素能正确执行是一个凑巧的结果, iterator.hasNext() 中使用一个 cursor != size 判定是否有下一个元素; 以上代码执行流程是 iterator.hasNext() => iterator.next() => list.remove(), 在 remove() 方法中 size 会自减一, 当循环第三次执行 iterator.hasNext() 时, cursor = 2 并且 size = size - 1 = 2, 所以 break 出循环, 避开了 iterator.next() 中的 checkForComodification(), 此方法用来判断 expectedModCount 和 modCount 是否相等, 如果不相等则抛出 ConcurrentModificationException 异常; 所以在使用迭代器遍历集合时, 请使用迭代器的删除方法, 如果是多线程还需要在 Iterator 遍历时加锁
```
Iterator<String> iterator = list.iterator();
while(iterator.hasNext()){
    synchronized (Object.class) {
        String str = iterator.next();
        if (str.equals("three")) {
            iterator.remove(str);
        }
    }
}
```
或者使用并发容器 CopyOnWriteArrayList 代替 ArrayList, 该容器内部会对 Iterator 加锁; COW 家族 (Copy-On-Write) 实行读写分离, 如果是写操作则复制一个集合, 在新介个内添加和删除数据, 待一切修改完成后再将原集合的引用指向新集合; 这样的好处是可以高并发的对 COW 进行读和遍历操作, 而且不需要加锁, 因为当前集合没有添加任何元素; 使用 COW 时要注意两点: 第一尽量设置合理的容量值, 因为扩容的代价较大, 第二使用批量添加和删除的方法, 避免增加一个元素而复制整个集合的操作; 由于 COW, 会导致内存使用增加和 GC 频繁, 所以 COW 适用于读多写少的场景  
COW 是 fail-safe 机制的, fail-safe 是在安全的副本上进行遍历的, 集合的修改和遍历是没有任何关系的, 但缺点是读不到最新的数据; 这也是 CAP 理论中的 C(Consistent) 与 A(Avaliability) 的矛盾

#### Map 类集合
Map 和 Collection 类是平级的接口, 在集合框架图上, 它与 Collection 有这微弱的依赖, 即部分方法返回 Collection 视图, 例如 keySet() 和 values() 方法; Map 类集合中存储的单位是 KV 键值对, Map 类就是使用一定哈希算法形成的一组比较均匀的哈希值作为 Key, Value 值挂在 Key 上; Map 类特点如下
- Map 取代了旧的抽象类 Dictionary, 有更好的性能
- 没有重复的 Key, 可以有多个重复的 Value
- Value 可以是 List, Set, Map 类对象
- KV 是否允许为 null, 以实现类约束为准

Map 除了传统的增删查改外, 还提供了返回所有的 Key, 所有的 Value, 所有的 KV 键值对, 通常这些视图支持 clear(), remove() 操作, 但是没有实现 add 操作, 细节可查看源码; 以下是各个 Map 类对 KV 是否可以为 null 的约束

| Map 集合类 | Key | Value | Super | JDK | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| HashTable | 不允许为 null | 不允许为 null | Dictionary | 1.0 | 线程安全 (过时) |
| ConcurrentHashMap | 不允许为 null | 不允许为 null | AbstractMap | 1.5 | 锁分段技术或 CAS |
| TreeMap | 不允许为 null | 允许为 null | AbstractMap | 1.2 | 线程不安全 (有序) |
| HashMap | 允许为 null | 允许为 null | AbstractMap | 1.2 | 线程不安全 (resize 死链问题) |

在大多数情况下, 使用 ConcurrentHashMap 替换 HashMap 没有任何问题, 在性能上区别也不大, 而且更加安全, 但是需要注意 ConcurrentHashMap 的 KV 不能为 null; 建议在任何 Map 类集合中都避免 KV 为 null

##### 红黑树

###### 树
树结构的特点如下
- 一个节点, 即只有根节点, 也可以是一棵树
- 其中任何一个节点与下面所有节点构成的树称为子树
- 根节点没有父节点, 而叶子节点没有子节点
- 除根节点外, 任何节点有且仅有一个父节点
- 任何节点可以有 0 ~ n 个子节点

至多有两个子节点的树称为二叉树

###### 平衡二叉树
平衡二叉树的性质如下
- 树的左右高度差不能超过 1
- 任何往下递归的左子树和右子树, 必须符合第一条性质
- 没有任何节点的空树或只有根节点的树也是平衡二叉树

###### 二叉查找树
二叉查找树又称二叉搜索树, 二叉查找树非常擅长数据查找; 二叉查找树额外增加了如下要求: 对于任何节点来说, 它的左子树上所有节点的值都小于, 而它右子树上所有节点的值都大于它; 查找过程从树的根节点开始, 沿着简单的判断往下走, 小于节点值的往左走, 大于节点值的往右走, 直到找到目标数据或者到达叶子节点还未找到  
遍历所有节点的常用方式有三种: 前序遍历, 中序遍历, 后序遍历; 它们三者规律如下
- 在任何递归子树中, 左节点一定在右节点前先遍历
- 前序遍历的顺序是根节点, 左节点, 右节点; 中序遍历的顺序是左节点, 根节点, 右节点; 后序遍历的顺序是左节点, 右节点, 根节点

二叉查找树容易构造, 但是如果缺少约束条件, 很可能往一个方向野蛮生长, 成为查找复杂度为 O(n) 的树; 所以二叉查找树需要引入一种检测机制, 随着新值的插入动态的调整树结构; 二叉查找树由于随着数据不断地增加或删除容易失衡, 为了保持二叉树重要的平衡性, 有很多算法实现, 如 AVL 树, 红黑树, SBT(Size Balanced Tree), Treap(树堆) 等; Java 底层框架很多算法实现是以红黑树为基础的

###### AVL 树
AVL 树算法是以苏联数学家 Adelson-Velsky 和 Landis 名字命名, 可以使二叉树的使用效率最大化; AVL 树是一种平衡二叉查找树, 增加和删除节点后通过树旋转重新达到平衡; 右旋是以某个节点为中心, 将它沉入当前右子节点的位置, 而让当前的左子节点作为新树的根节点, 也称顺时针旋转; 同理, 左旋是以某个节点为中心, 将它沉入当前左子节点的位置, 而让当前右子节点作为新树的根节点, 也称为逆时针旋转; AVL 树就是通过不断旋转来达到树平衡的

###### 红黑树
红黑树的主要特征是在每个节点上增加一个属性来表示节点的颜色, 可以是红色也可以是黑色  
红黑树与 AVL 树类似, 都是在进行插入和删除元素时, 通过特定的旋转来保持平衡的, 从而获得较高的查找性能; 与 AVL 树相比, 红黑树并不追求所有递归子树的高度差不超过 1, 而是保证从根节点到叶子节点的最长路径不超过最短路径的 2 倍, 所以它的最坏运行时间也是 $ O(log_n) $; 红黑树通过重新着色和旋转, 更加高效的完成了插入和删除操作后的自平衡调整; 当然红黑树本质还是二叉树, 它额外引入了 5 个约束条件
- 节点只能是红色或者黑色
- 根节点必须是黑色
- 所有 NIL 节点都是黑色的; NIL 即叶子节点下挂的两个虚节点
- 一条路径上不能出现响铃的两个红色节点
- 在任何递归子树内, 根节点到叶子节点的所有路径上包含相同数目的黑色节点

上述 5 个约束条件保证了红黑树的新增, 查找, 删除的最坏时间复杂度均为 $ O(log_n) $; 如果一个树的左子节点或右子节点不存在, 则均认定为黑色; 红黑树的任何旋转在 3 次之内均可完成

###### 红黑树和 AVL 数的比较
TODO

##### TreeMap
TreeMap 是按照 Key 的排序结果来组织内部结构的 Map 类集合, 它改变了 Map 散乱无序的形象; 在 TreeMap 的接口继承树中, 有两个狱中不同的接口: SortedMap 和 NavigableMap; SortedMap 接口表示它的 Key 是有序不可重复的, 支持获取头尾 Key-Value 元素, 或者根据 Key 指定范围获取子集合等; 插入的 Key 必须实现 Comparable 或提供额外的比较器 Comparator, 所以 Key 不可为 null; NavigableMap 接口继承了 SortedMap 接口, 根据指定的搜索条件返回最匹配的 Key-Value 元素; 与 HashMap 不同的是, TreeMap 是依靠 Comparable 或 Comparator 来实现 Key 去重的; 如果要用 TreeMap 对 Key 进行排序, 调用如下方法
```
final int compare(Object k1, Object k2) {
    return comparator==null ? ((Comparable<? super K>)k1).compareTo((K)k2)
        : comparator.compare((K)k1, (K)k2);
}
```
如果 comparator 不为 null, 优先使用比较器 comparator 的 compare() 方法; 如果为 null, 则使用 Key 实现的自然排序 Comparable 接口的 compareTo 方法; 如果两者都无法满足抛出异常  
基于红黑树实现的 TreeMap 提供了平均和最坏复杂度 $ O(log_n) $ 的增删查改操作, 并实现了 NavigableMap 接口; TreeMap 通过 put() 和 deleteEntry() 实现红黑树的增加和删除节点操作, 有以下三个条件需要明确
- 需要调整的新节点总是红色的
- 如果插入新节点的父节点是黑色的, 则无须调整; 因为依然能符合红黑树的 5 个约束条件
- 如果插入新节点的父节点是红色的, 因为红黑树规定不能出现相邻的两个红色节点, 所以进入循环判断, 或重新着色, 或左右旋转, 最终达到红黑树的五个约束条件, 退出条件如下
```
while (x != null && x != root && x.parent.color == RED) {
    ...
}
```
TreeMap 的插入操作就是按照 Key 的对比往下遍历, 大于比较节点只 向右走, 小于比较节点值向左走, 先按照二叉查找树的特性操作, 无须关心节点颜色与树平衡, 后续会重新着色和旋转, 保持红黑树的特性, 以下是 put() 方法源码
```
public V put(K key, V value) {
    // t 表示当前节; 先把 TreeMap 的根节点 root 引用赋值给当前节点
    Entry<K,V> t = root;
    // 如果当前节点为 null, 即是空树, 新增的 KV 形成的节点就是根节点
    if (t == null) {
        // 检查参数 key 是否可比较
        compare(key, key); // type (and possibly null) check
        // 使用 KV 构造出 Entry 对象, 第三个参数是 parent, 根节点没有父节点
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    // 用来保存比较结果
    int cmp;
    Entry<K,V> parent;
    // 构造方法中传入的比较器
    Comparator<? super K> cpr = comparator;  // split comparator and comparable paths
    /*
     * 根据二叉查找树的特性, 找到新节点插入的合适位置
     */
    if (cpr != null) {
        // 根据参数 key 与当前节点的 key 不断进行比较
        do {
            // 当前节点赋值给父节点, 即从根节点开始比较
            parent = t;
            // 与输入的参数 key 比较
            cmp = cpr.compare(key, t.key);
            // 参数 key 较小则走左子树, 即把当前节点引用移动至它的左子节点上
            if (cmp < 0)
                t = t.left;
            // 参数 key 较大则走右子树, 即把当前节点引用移动至它的右子节点上
            else if (cmp > 0)
                t = t.right;
            // 如果相等, 则覆盖当前节点的值, 并返回更新前的值
            else
                return t.setValue(value);
        // 如果没有相等的 key, 一直会遍历到 NIL 节点为止        
        } while (t != null);
    }
    // 在没有指定比较器的情况下, 调用自然排序的 Comparable 比较
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // 创建  Entry 对象, 并把 parent 置入参数
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        // 如果比较结果小于 0, 则成为 parent 的左孩子
        parent.left = e;
    else
        // 如果比较结果大于 0, 则成为 parent 的右孩子
        parent.right = e;
    // 还需要对这个新节点进行重新着色和旋转操作, 以达到平衡
    fixAfterInsertion(e);
    size++;
    modCount++;
    // 成功插入新节点后, 返回为 null
    return null;
}
```
如果一个新节点插入时能够运行到 fixAfterInsertion() 进行着色和旋转, 说明新节点加入之前是非空树, 以及新节点的 key 与任何节点都不相同
```
private void fixAfterInsertion(Entry<K,V> x) {
    // 虽然内部类 Entry 的属性 color 默认为黑色, 但新节点一律先赋值为红色
    x.color = RED;

    // 新节点是根节点或者其父节点为黑色, 插入红色节点并不会破坏红黑树的特性, 无须调整
    // x 值的改变的过程是不断的向上游遍历 (内部调整), 直到父亲为黑色, 或者到达根节点
    while (x != null && x != root && x.parent.color == RED) {
        //  如果父亲是其父节点 (爷爷) 的左子节点
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            // 这时得看爷爷的右子节点 (右叔) 的脸色
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            // 如果右叔是红色, 此时通过局部颜色调整, 就可以使子树继续满足红黑树的性质
            if (colorOf(y) == RED) {
                // 父亲置为黑色
                setColor(parentOf(x), BLACK);
                // 右叔置为黑色
                setColor(y, BLACK);
                // 爷爷置为红色
                setColor(parentOf(parentOf(x)), RED);
                // 爷爷成为新的节点, 进入到下一轮循环
                x = parentOf(parentOf(x));
            }
            // 如果右叔是黑色, 则需要加入旋转
            else {
                // 如果 x 是父亲的右子节点, 先对父亲做一次左旋操作; 转化为 x 为父亲的左子节点的情形
                if (x == rightOf(parentOf(x))) {
                    // 对父亲做一次左旋操作, 红色的父亲会沉入其左侧位置, 将父亲赋值给 x
                    x = parentOf(x);
                    rotateLeft(x);
                }
                // 重新着色并对爷爷进行右旋操作
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        }
        // 如果父亲是其父节点 (爷爷) 的右子节点
        else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    // 根节点置为黑色
    root.color = BLACK;
}
```
左旋与右旋代码类似, 以下是左旋代码, 输入参数为失去平衡的子树的根节点
```
private void rotateLeft(Entry<K,V> p) {
    // 如果参数节点不为 NIL 节点
    if (p != null) {
        //  获取 p 的右子节点 r
        Entry<K,V> r = p.right;
        // 将 r 的左子树设置为 p 的右子树
        p.right = r.left;
        // 若 r 的左子树不为空, 则将 p 设置为 r 左子树的父亲
        if (r.left != null)
            r.left.parent = p;
        // 将 p 的父亲设置为 r 的父亲
        r.parent = p.parent;
        // 无论如何, r 都要替代父亲心中 p 的位置
        if (p.parent == null)
            root = r;
        else if (p.parent.left == p)
            p.parent.left = r;
        else
            p.parent.right = r;
        // 将 p 设置为 r 的左子树, 将 r 设置为 p 的父亲
        r.left = p;
        p.parent = r;
    }
}
```
在树的演化过程中, 插入节点的过程中, 如果需要重新着色或旋转, 存在三种情形
- 节点的父亲是红色的, 叔叔是红色的, 则重新着色
- 节点的父亲是红色的, 叔叔是黑色的, 而新节点是父亲的左节点, 则进行右旋
- 节点的父亲是红色的, 叔叔是黑色的, 而新节点是父亲的右节点, 则进行左旋

红黑树相比 AVL 树, 任何不平衡都能在 3 次旋转之内调整完成; 每次向上回溯的步长是 2, 对于频繁插入和删除的场景, 红黑树的优势是明显的; TreeMap 是线程不安全的集合, 在多线程进行写操作时, 需要增加互斥机制, 或者把对象放在 Collections.synchronizedMap(map) 中同步  
如果只是对单个元素进行排序, 使用 TreeSet 即可, TreeSet 的底层则是 TreeMap, Value 共享使用一个静态变量
```
// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();

public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
```

##### HashMap
除局部方法或者绝对线程安全的情形外, 优先推荐使用 ConcurrentHashMa, 二者虽然性能相差无几, 但后者解决了高并发下的线程安全问; HashMap 的死链问题以及扩容数据丢失问题是慎用 HashMap 的两个主要原因; 以下是 HashMap 中三个基本存储概念

| 名称 | 说明  |
| :--- | :---- |
| table | 存储所有节点的数组 |
| slot | 哈希槽, 即 table[i] 的位置 |
| bucket | 哈希桶, table[i] 上所有元素形成的表或数的集合 |

TODO

##### ConcurrentHashMap
TODO
