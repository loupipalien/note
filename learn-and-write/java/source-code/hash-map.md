### HashMap
`HashMap` 是一种散列表, 用于存储 key-value 键值对的数据结构, 一般翻译为哈希表, 提供平均时间复杂度为 O(1) 的, 基于 key 级别的 get/put 操作

#### 类图
![HashMap](http://static.iocoder.cn/images/JDK/2019_12_07/03.jpg)  
- 实现了 `java.util.Map` 接口, 并继承了 `java.util.AbstractMap` 抽象类
- 实现了 `java.io.Serializable` 接口
- 实现了 `java.lang.Cloneable` 接口

#### 原理
`HashMap` 所提供的 O(1) 是平均时间复杂度, 大多数情况下保证 O(1), 在极端的情况下, 有可能退化为 O(N) 的时间复杂度  
`HashMap` 其实是在数组的基础上实现的, 下标的计算是基于 key 的, 通过 `hash(key)` 的过程可以将 key 成功的转为一个整数, 但是 `hash(key)` 可能会超过数组的容量, 所以需要 `hash(key) % size` 作为下标, 放入数组的对应位置; 至此, 基本上可以在 O(1) 时间复杂度上快速的从 `HashMap` 中进行 `get` 操作了; 但这里还有其他的问题
- `hash(key)` 计算出来的哈希值, 并不保证唯一
- `hash(key) % size` 的操作, 即使不同的哈希值, 也可能变成相同的结果

这就是通常所说的 `哈希冲突`, 解决的方法通常有两种
- 开放地址法, 可参考 [散列表的开放地址法](https://blog.csdn.net/zuoyigexingfude/article/details/40394859)
- 链表法, 在 Java 的 `HashMap` 中是采用了此方法, 此方法通过将数组的每个元素对应一个链表, 将相同的 `hash(key) % size` 放到对应下标的链表中

到了这里, 采用了链地址法也并没有就能到达了平均复杂度 O(1) 的目标; 可以试想放入 N 个 key-value 键值对到 `HashMap` 中的情况
- 其中 k 个 key 的 `hash(key) % size` 值是一样的, 那么在 `hash(key) % size` 下标的链表就有 k 的节点, 那么过去这 k 个 key 的时间复杂度是 O(k)
- 再更极端的情况下, k 恰好等于 N, 那么时间复杂度降到 O(N)

所以为了解决 O(N) 的时间复杂度的情况, 可以将数组的每个下标对应的链表构造为其他数据结构, 例如红黑树和跳表, 它们两者的时间复杂度是 $O(logN)$, 这样 `O(N)` 就可以缓解成 `O(logN)` 的时间复杂度  
- 在 JDK7 的版本中, `HashMap` 采用 `数组 + 链表` 的形式实现
- 在 JDK8 的版本中, `HashMap` 采用 `数组 + 链表 + 红黑树` 的形式实现, 在空间和时间复杂度中做取舍

除此之外, 为了是在链表上查找的时间更少, 那么最好的办法是没有链表, 使每个下标只有一个元素, 从而到达 O(1) 的时间复杂度, 为了尽可能的达到这一点当键值对数量达到阈值后就会进行扩容; 这里的阈值是根据 `负载因子` 计算的, 当 `size / capacity > loadFactor` 时就需要进行扩容, 其中 `size` 为键值对的个数, `capacity` 为数组容量  

#### 属性
```Java
/**
 * The table, initialized on first use, and resized as
 * necessary. When allocated, length is always a power of two.
 * (We also tolerate length zero in some operations to allow
 * bootstrapping mechanics that are currently not needed.)
 */
transient Node<K,V>[] table;

/**
 * Holds cached entrySet(). Note that AbstractMap fields are used
 * for keySet() and values().
 */
transient Set<Map.Entry<K,V>> entrySet;

/**
 * The number of key-value mappings contained in this map.
 */
transient int size;

/**
 * The number of times this HashMap has been structurally modified
 * Structural modifications are those that change the number of mappings in
 * the HashMap or otherwise modify its internal structure (e.g.,
 * rehash).  This field is used to make iterators on Collection-views of
 * the HashMap fail-fast.  (See ConcurrentModificationException).
 */
transient int modCount;

/**
 * The next size value at which to resize (capacity * load factor).
 *
 * @serial
 */
// (The javadoc description is true upon serialization.
// Additionally, if the table array has not been allocated, this
// field holds the initial array capacity, or zero signifying
// DEFAULT_INITIAL_CAPACITY.)
int threshold;

/**
 * The load factor for the hash table.
 *
 * @serial
 */
final float loadFactor;
```
- table: 在首次使用时初始化, 必要时会重置大小, 长度总是 2 的次方
- entrySet: 调用 `entrySet` 方法后的缓存
- size: map 中包含的键值对数量
- modCount: 修改次数
- threshold: 阈值, 当超过此值时 (capacity * loadFactor) 时会重置大小
- loadFactor: 哈希表的负载因子

#### 内部类
##### Node
```Java
/**
 * Basic hash bin node, used for most entries.  (See below for
 * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
 */
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```
实现了定义在 `java.util.Map` 接口中的 `Map.Entry` 接口; 其中 `hash, key, value` 定义了节点的 3 个重要属性, `next` 属性指向下一个节点, 通过它可以实现 `table` 数组的每一位可以形成链表
##### TreeNode
![HashMap.TreeNode](http://static.iocoder.cn/images/JDK/2019_12_07/04.jpg)  
`TreeNode` 定义在 `HashMap` 中, 红黑树节点, 通过它可以实现 `table` 数组的每一个位置形成红黑树

#### 构造方法
`HashMap` 一共有四个构造方法
##### HashMap()
```Java
/**
 * Constructs an empty <tt>HashMap</tt> with the default initial capacity
 * (16) and the default load factor (0.75).
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```
构造一个空的 `HashMap`, 初始容量 16, 负载因子 0.75; 这里 `table` 数组还没有被初始化, 它是延迟初始化的
##### HashMap(int initialCapacity)
```Java
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and the default load factor (0.75).
 *
 * @param  initialCapacity the initial capacity.
 * @throws IllegalArgumentException if the initial capacity is negative.
 */
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```
自定义设置初始容量, 负载因子仍为默认值 0.75
##### HashMap(initialCapacity, float loadFactor)
```Java
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and load factor.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
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
```
- 检查 `initialCapacity` 和 `loadFactor` 的有效性
- 使用 `tableSizeFor(int initialCapacity)` 方法返回 `initialCapacity` 的最小的二次方作为 `threshold`

###### tableSizeFor
```Java
/**
 * Returns a power of two size for the given target capacity.
 */
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
此方法是将 `cap` 的最高位的 `1` 赋值到后续的低位, 得到最小的大于 `cap` 的 2 的次方的数, 这里需要注意的是允许的最大值为 $2^30$, 因为 int 的最大值为 $2^31$

##### HashMap(Map<? extends K, ? extends V> m)
```Java
/**
 * Constructs a new <tt>HashMap</tt> with the same mappings as the
 * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
 * default load factor (0.75) and an initial capacity sufficient to
 * hold the mappings in the specified <tt>Map</tt>.
 *
 * @param   m the map whose mappings are to be placed in this map
 * @throws  NullPointerException if the specified map is null
 */
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```
此方法主要调用 `putMapEntries(Map<? extends K, ? extends V> m, boolean evict)` 批量将元素添加到 `table` 中
###### putMapEntries(Map<? extends K, ? extends V> m, boolean evict)
```Java
/**
 * Implements Map.putAll and Map constructor
 *
 * @param m the map
 * @param evict false when initially constructing this map, else
 * true (relayed to method afterNodeInsertion).
 */
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```
- 当 `m` 的 `size` 大于零时, 根据 `table` 是否为 `null` 调整 `threshold` 的大小 (注意当 `table` 为 `null` 时, 这里值调整了大小但 `table` 尚未初始化)
- 遍历  `m` 集合, 调用 `putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict)` 方法添加到 `HashMap` 中

#### 哈希函数
对于哈希函数来说, 有两个方面特别重要
- 性能足够高, 因为基本 `HashMap` 所有的操作都需要用到哈希函数
- 对于计算出来的哈希值足够离散, 保证哈希冲突的概率更小

```Java
/**
 * Computes key.hashCode() and spreads (XORs) higher bits of hash
 * to lower.  Because the table uses power-of-two masking, sets of
 * hashes that vary only in bits above the current mask will
 * always collide. (Among known examples are sets of Float keys
 * holding consecutive whole numbers in small tables.)  So we
 * apply a transform that spreads the impact of higher bits
 * downward. There is a tradeoff between speed, utility, and
 * quality of bit-spreading. Because many common sets of hashes
 * are already reasonably distributed (so don't benefit from
 * spreading), and because we use trees to handle large sets of
 * collisions in bins, we just XOR some shifted bits in the
 * cheapest possible way to reduce systematic lossage, as well as
 * to incorporate impact of the highest bits that would otherwise
 * never be used in index calculations because of table bounds.
 */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
- 高效性: 这里的计算量仅仅是 `key.hashCode()` 和与 `key.hashCode() >>> 6` 的异或操作, 可以说是相当高效的
- 离散型: `key.hashCode() ^ (key.hashCode() >>> 16)` 为什么有更好的离散性见[这里](https://www.zhihu.com/question/20733617/answer/111577937)

#### 添加单个元素  
```Java
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>.)
 */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
- `tab` 表示 `Node` 数组, `p` 表示目标索引的 `Node`, `n` 为数组的大小, `i` 为目标索引
- 当 `table` 数组为 `null` 或者 `length` 为 `0` 时调用 `resize()` 方法
- 使用 `(n - 1) & hash` 计算目标索引
  - 如果目标位置为 `null`, 则使用 `value` 生成新节点赋值此目标索引
  - 如果目标位置不为 `null` 时, 找到对应节点并赋值给节点 `e`
    - 如果要插入元素的 hash 和 key 与 `p` 节点的 hash 和 key 相同
    - 如果 `p` 为  `TreeNode` 类型 (红黑树), 则调用 `TreeNode.putTreeVal` 方法
    - 如果 `p` 不为 `TreeNode` 类型 (链表), 遍历链表找到对应节点或在尾部添加新节点, 当节点数大于等于 `TREEIFY_THRESHOLD - 1 = 7` 时链表树化
  - 如果节点 `e` 不为 空, 根据参数修改原有节点值, 并执行节点访问的回调, 最后返回被覆盖的值
- `modCount` 自增
- 检查阈值, 如有必要则进行扩容
- 执行节点插入的回调
- 返回 `null`

#### 扩容
`resize()` 方法两倍扩容 `HashMap`, 实际上在构造方法中看到 `table` 数组并未初始化, 它是在 `resize()` 方法中进行初始化的, 所以该方法的另一个作用是: `初始化数组`
```Java
/**
 * Initializes or doubles table size.  If null, allocates in
 * accord with initial capacity target held in field threshold.
 * Otherwise, because we are using power-of-two expansion, the
 * elements from each bin must either stay at same index, or move
 * with a power of two offset in the new table.
 *
 * @return the table
 */
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```
