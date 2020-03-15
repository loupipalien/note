### TreeMap
按照 key 排序的 Map 实现类

#### 类图
`TreeMap` 实现的接口和继承的抽象类如下图  
![TreeMap](http://static.iocoder.cn/images/JDK/2019_12_13_02/01.png)  
- 继承了 `java.util.AbstractMap` 抽象类
- 实现了 `java.util.NavigableMap` 接口
- 实现了 `java.io.Serializable` 接口
- 实现了 `java.lang.Cloneable` 接口

#### 属性
在开始了解 `TreeMap` 的具体属性之前, 先简单说说 `TreeMap` 的实现原理, `TreeMap` 采用红黑树实现
```
红黑树是一种自平衡二叉查找树, 是在计算机科学中用到的一种数据结构, 典型的用途是实现关联数组; 它在 1972 年由鲁道夫·贝尔发明, 被称为 `对称二叉B树`, 红黑树的结构复杂, 但它的操作有着良好的最坏情况运行时间, 并且在实践中高效: 它可以在 `logN` 时间内完成查找, 插入和删除, 这里的 `N` 是树中元素的数目
```
- 有序性: 红黑树是一种二叉查找树, 父节点的 key 小于左子节点的 key, 大于右子节点的 key; 这样就完成了 `TreeMap` 的有序的特性
- 高性能: 红黑树会进行自平衡, 避免树的高度过高, 导致查找性能下滑; 这样红黑树能够提供 `logN` 的时间复杂度

```Java
/**
 * The comparator used to maintain order in this tree map, or
 * null if it uses the natural ordering of its keys.
 *
 * @serial
 */
private final Comparator<? super K> comparator;

private transient Entry<K,V> root;

/**
 * The number of entries in the tree
 */
private transient int size = 0;

/**
 * The number of structural modifications to the tree.
 */
private transient int modCount = 0;
```
- comparator: key 的排序器; 通过该属性可以自定义 key 的排序规则, 如果未设置则使用 key 类型自己的排序
- root: 红黑树的根节点; 其中 `Entry` 是 `TreeMap` 的内部静态类
```Java
/**
 * Node in the Tree.  Doubles as a means to pass key-value pairs back to
 * user (see Map.Entry).
 */

static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;

    /**
     * Make a new cell with given key, value, and parent, and with
     * {@code null} child links, and BLACK color.
     */
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }

    /**
     * Returns the key.
     *
     * @return the key
     */
    public K getKey() {
        return key;
    }

    /**
     * Returns the value associated with the key.
     *
     * @return the value associated with the key
     */
    public V getValue() {
        return value;
    }

    /**
     * Replaces the value currently associated with the key with the given
     * value.
     *
     * @return the value associated with the key before this method was
     *         called
     */
    public V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;

        return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
    }

    public int hashCode() {
        int keyHash = (key==null ? 0 : key.hashCode());
        int valueHash = (value==null ? 0 : value.hashCode());
        return keyHash ^ valueHash;
    }

    public String toString() {
        return key + "=" + value;
    }
}
```
- size: key-value 的键值对数量
- modCount: 修改次数

#### 构造方法
`TreeMap` 一共有四个构造方法
##### TreeMap()
```Java
/**
 * Constructs a new, empty tree map, using the natural ordering of its
 * keys.  All keys inserted into the map must implement the {@link
 * Comparable} interface.  Furthermore, all such keys must be
 * <em>mutually comparable</em>: {@code k1.compareTo(k2)} must not throw
 * a {@code ClassCastException} for any keys {@code k1} and
 * {@code k2} in the map.  If the user attempts to put a key into the
 * map that violates this constraint (for example, the user attempts to
 * put a string key into a map whose keys are integers), the
 * {@code put(Object key, Object value)} call will throw a
 * {@code ClassCastException}.
 */
public TreeMap() {
    comparator = null;
}
```
默认构造方法, 不使用自定义排序其, 此时 `comparator` 为空

##### TreeMap(Comparator<? super K> comparator)
```Java
/**
 * Constructs a new, empty tree map, ordered according to the given
 * comparator.  All keys inserted into the map must be <em>mutually
 * comparable</em> by the given comparator: {@code comparator.compare(k1,
 * k2)} must not throw a {@code ClassCastException} for any keys
 * {@code k1} and {@code k2} in the map.  If the user attempts to put
 * a key into the map that violates this constraint, the {@code put(Object
 * key, Object value)} call will throw a
 * {@code ClassCastException}.
 *
 * @param comparator the comparator that will be used to order this map.
 *        If {@code null}, the {@linkplain Comparable natural
 *        ordering} of the keys will be used.
 */
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}
```
可传入 `comparator` 参数，自定义 key 的排序规则

##### TreeMap(Map<? extends K, ? extends V> m)
```Java
/**
 * Constructs a new tree map containing the same mappings as the given
 * map, ordered according to the <em>natural ordering</em> of its keys.
 * All keys inserted into the new map must implement the {@link
 * Comparable} interface.  Furthermore, all such keys must be
 * <em>mutually comparable</em>: {@code k1.compareTo(k2)} must not throw
 * a {@code ClassCastException} for any keys {@code k1} and
 * {@code k2} in the map.  This method runs in n*log(n) time.
 *
 * @param  m the map whose mappings are to be placed in this map
 * @throws ClassCastException if the keys in m are not {@link Comparable},
 *         or are not mutually comparable
 * @throws NullPointerException if the specified map is null
 */
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}
```
传入 `Map` 类型参数, 调用 `putAll(Map<? extends K, ? extends V> map)` 方法添加元素; `putAll(Map<? extends K, ? extends V> map)` 方法代码如下
```Java
/**
 * Copies all of the mappings from the specified map to this map.
 * These mappings replace any mappings that this map had for any
 * of the keys currently in the specified map.
 *
 * @param  map mappings to be stored in this map
 * @throws ClassCastException if the class of a key or value in
 *         the specified map prevents it from being stored in this map
 * @throws NullPointerException if the specified map is null or
 *         the specified map contains a null key and this map does not
 *         permit null keys
 */
public void putAll(Map<? extends K, ? extends V> map) {
    int mapSize = map.size();
    if (size==0 && mapSize!=0 && map instanceof SortedMap) {
        Comparator<?> c = ((SortedMap<?,?>)map).comparator();
        if (c == comparator || (c != null && c.equals(comparator))) {
            ++modCount;
            try {
                buildFromSorted(mapSize, map.entrySet().iterator(),
                                null, null);
            } catch (java.io.IOException cannotHappen) {
            } catch (ClassNotFoundException cannotHappen) {
            }
            return;
        }
    }
    super.putAll(map);
}
```
- 获取 `map` 的大小
- 如果 `map` 是非空的 `SortedMap`, 并且 `size` 大小为 `0`, 则调用 `buildFromSorted` 方法来处理
- 否则调用超类方法遍历添加所有元素

##### TreeMap(SortedMap<K, ? extends V> m)
```Java
/**
 * Constructs a new tree map containing the same mappings and
 * using the same ordering as the specified sorted map.  This
 * method runs in linear time.
 *
 * @param  m the sorted map whose mappings are to be placed in this map,
 *         and whose comparator is to be used to sort this map
 * @throws NullPointerException if the specified map is null
 */
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```
- 设置 `comparator` 为传入的 `SortedMap` 的排序器
- 调用 `buildFromSorted(int size, Iterator<?> it, ObjectInputStream str, V defaultVal)` 方法, 使用传入的 `SortedMap` 构造出红黑树; 因为 SortedMap 类型天然有序, 所以可基于 `m` 的中间节点为红黑树的根节点, `m` 的左边为左子树, `m` 的右边为右子树

`buildFromSorted(int size, Iterator<?> it, ObjectInputStream str, V defaultVal)` 方法代码如下
```Java
/**
 * Linear time tree building algorithm from sorted data.  Can accept keys
 * and/or values from iterator or stream. This leads to too many
 * parameters, but seems better than alternatives.  The four formats
 * that this method accepts are:
 *
 *    1) An iterator of Map.Entries.  (it != null, defaultVal == null).
 *    2) An iterator of keys.         (it != null, defaultVal != null).
 *    3) A stream of alternating serialized keys and values.
 *                                   (it == null, defaultVal == null).
 *    4) A stream of serialized keys. (it == null, defaultVal != null).
 *
 * It is assumed that the comparator of the TreeMap is already set prior
 * to calling this method.
 *
 * @param size the number of keys (or key-value pairs) to be read from
 *        the iterator or stream
 * @param it If non-null, new entries are created from entries
 *        or keys read from this iterator.
 * @param str If non-null, new entries are created from keys and
 *        possibly values read from this stream in serialized form.
 *        Exactly one of it and str should be non-null.
 * @param defaultVal if non-null, this default value is used for
 *        each value in the map.  If null, each value is read from
 *        iterator or stream, as described above.
 * @throws java.io.IOException propagated from stream reads. This cannot
 *         occur if str is null.
 * @throws ClassNotFoundException propagated from readObject.
 *         This cannot occur if str is null.
 */
private void buildFromSorted(int size, Iterator<?> it,
                             java.io.ObjectInputStream str,
                             V defaultVal)
    throws  java.io.IOException, ClassNotFoundException {
    this.size = size;
    root = buildFromSorted(0, 0, size-1, computeRedLevel(size),
                           it, str, defaultVal);
}
```
- 设置键值对的数量 `size`
- 调用 `computeRedLevel(int size)` 方法计算红黑树的高度
- 调用 `buildFromSorted(int level, int lo, int hi, int redLevel, Iterator<?> it, java.io.ObjectInputStream str, V defaultVal)` 方法构造红黑树并返回根节点

`buildFromSorted(int level, int lo, int hi, int redLevel, Iterator<?> it, java.io.ObjectInputStream str, V defaultVal)` 方法代码如下
```Java
/**
 * Recursive "helper method" that does the real work of the
 * previous method.  Identically named parameters have
 * identical definitions.  Additional parameters are documented below.
 * It is assumed that the comparator and size fields of the TreeMap are
 * already set prior to calling this method.  (It ignores both fields.)
 *
 * @param level the current level of tree. Initial call should be 0.
 * @param lo the first element index of this subtree. Initial should be 0.
 * @param hi the last element index of this subtree.  Initial should be
 *        size-1.
 * @param redLevel the level at which nodes should be red.
 *        Must be equal to computeRedLevel for tree of this size.
 */
@SuppressWarnings("unchecked")
private final Entry<K,V> buildFromSorted(int level, int lo, int hi,
                                         int redLevel,
                                         Iterator<?> it,
                                         java.io.ObjectInputStream str,
                                         V defaultVal)
    throws  java.io.IOException, ClassNotFoundException {
    /*
     * Strategy: The root is the middlemost element. To get to it, we
     * have to first recursively construct the entire left subtree,
     * so as to grab all of its elements. We can then proceed with right
     * subtree.
     *
     * The lo and hi arguments are the minimum and maximum
     * indices to pull out of the iterator or stream for current subtree.
     * They are not actually indexed, we just proceed sequentially,
     * ensuring that items are extracted in corresponding order.
     */

    if (hi < lo) return null;

    int mid = (lo + hi) >>> 1;

    Entry<K,V> left  = null;
    if (lo < mid)
        left = buildFromSorted(level+1, lo, mid - 1, redLevel,
                               it, str, defaultVal);

    // extract key and/or value from iterator or stream
    K key;
    V value;
    if (it != null) {
        if (defaultVal==null) {
            Map.Entry<?,?> entry = (Map.Entry<?,?>)it.next();
            key = (K)entry.getKey();
            value = (V)entry.getValue();
        } else {
            key = (K)it.next();
            value = defaultVal;
        }
    } else { // use stream
        key = (K) str.readObject();
        value = (defaultVal != null ? defaultVal : (V) str.readObject());
    }

    Entry<K,V> middle =  new Entry<>(key, value, null);

    // color nodes in non-full bottommost level red
    if (level == redLevel)
        middle.color = RED;

    if (left != null) {
        middle.left = left;
        left.parent = middle;
    }

    if (mid < hi) {
        Entry<K,V> right = buildFromSorted(level+1, mid+1, hi, redLevel,
                                           it, str, defaultVal);
        middle.right = right;
        right.parent = middle;
    }

    return middle;
}
```
- 当 `hi < lo` 时递归结束
- 计算中间值 `mid`
- 当 `lo < mid` 时递归构建左子树
- 使用 `it` 或从 `stream` 流中获取中间节点的 `key` 和 `value` 的值
- 使用 `key` 和 `value` 创建中间节点 `middle`
- 如果到树的最大高度, 则设置为红节点
- 如果左子数不等于 `null` 则进行设置
- 当 `lo < mid` 时递归构建右子树并进行设置
- 返回节点 `middle`

#### 添加单个元素
```Java
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 *
 * @return the previous value associated with {@code key}, or
 *         {@code null} if there was no mapping for {@code key}.
 *         (A {@code null} return can also indicate that the map
 *         previously associated {@code null} with {@code key}.)
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        compare(key, key); // type (and possibly null) check

        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
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
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```
- 保存 `root` 节点给 `t`
- 如果无根节点, 直接使用键值对创建根节点
  - 校验 key 的类型
  - 使用 `key` 和 `value` 创建 `Entry` 节点
  - 设置 `size` 的数量
  - `modCount` 自增一
  - 返回 `null`
- 遍历红黑树, 保存 `comparator` 比较器
- 如果 `comparator` 不为 `null` 时则使用它来比较
  - 设置节点 `t` 为父节点 `parent`
  - 比较 `key` 与 `t` 节点的 `key` 的大小保存为 `cmp`
  - 如果 `cmp` 小于零则 `t = t.left`; 大于零则 `t = t.right`; 等于零则将 `t` 节点的值设置为 `value` 并返回旧值
  - 当 `t != null` 时循环
- 如果 `comparator` 为 `null` 时则使用 `key` 自身的比较器
  - 设置节点 `t` 为父节点 `parent`
  - 比较 `key` 与 `t` 节点的 `key` 的大小保存为 `cmp`
  - 如果 `cmp` 小于零则 `t = t.left`; 大于零则 `t = t.right`; 等于零则将 `t` 节点的值设置为 `value` 并返回旧值
  - 当 `t != null` 时循环
- 当没有找到 `key` 的位置则创建 `Entry` 节点
- 并根据 `cmp` 作为 `parent` 的左孩子或右孩子
- 插入后调用 `fixAfterInsertion(Entry<K,V> x)` 进行自平衡
- `size` 和 `modCount` 都自增一
- 由于没有旧值, 返回 `null`

#### 获取单个元素
```Java
/**
 * Returns the value to which the specified key is mapped,
 * or {@code null} if this map contains no mapping for the key.
 *
 * <p>More formally, if this map contains a mapping from a key
 * {@code k} to a value {@code v} such that {@code key} compares
 * equal to {@code k} according to the map's ordering, then this
 * method returns {@code v}; otherwise it returns {@code null}.
 * (There can be at most one such mapping.)
 *
 * <p>A return value of {@code null} does not <em>necessarily</em>
 * indicate that the map contains no mapping for the key; it's also
 * possible that the map explicitly maps the key to {@code null}.
 * The {@link #containsKey containsKey} operation may be used to
 * distinguish these two cases.
 *
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}
```
调用 `getEntry(Object key)` 方法找到对应的 `Entry`; `getEntry(Object key)` 方法代码如下
```Java
/**
 * Returns this map's entry for the given key, or {@code null} if the map
 * does not contain an entry for the key.
 *
 * @return this map's entry for the given key, or {@code null} if the map
 *         does not contain an entry for the key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
final Entry<K,V> getEntry(Object key) {
    // Offload comparator-based version for sake of performance
    if (comparator != null)
        return getEntryUsingComparator(key);
    if (key == null)
        throw new NullPointerException();
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}

/**
 * Version of getEntry using comparator. Split off from getEntry
 * for performance. (This is not worth doing for most methods,
 * that are less dependent on comparator performance, but is
 * worthwhile here.)
 */
final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
        K k = (K) key;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
    }
    return null;
}
```
- 如果 `comparator` 为 `null`, 则使用 `key` 自身的比较器进行比较查找
- 如果 `comparator` 不为 `null`, 则使用 `key` 自身的比较器进行比较二分查找红黑树的节点后返回

#### 删除单个元素
```Java
/**
 * Removes the mapping for this key from this TreeMap if present.
 *
 * @param  key key for which mapping should be removed
 * @return the previous value associated with {@code key}, or
 *         {@code null} if there was no mapping for {@code key}.
 *         (A {@code null} return can also indicate that the map
 *         previously associated {@code null} with {@code key}.)
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}
```
- 调用 `getEntry(Object key)` 方法获取对应的 `Entry` 节点
- 如果不存在 `Entry` 节点则直接返回
- 如果存在 `Entry` 节点则保存旧值
- 调用 `deleteEntry(Object key)` 方法删除该节点

由于红黑树也是一颗二叉查找树, 中序遍历是有序的, 这里删除节点有四种场景
- 无子节点
直接删除父节点对其的指向即可
- 只有左子节点
将删除节点的父节点指向删除节点的左子节点
- 只有右子节点
与情况二相同, 将删除节点的父节点指向删除节点的右子节点
- 有左子节点也有右子节点
这种情况相对复杂, 由于无法使用子节点替换掉删除的节点, 所以此时有一种巧妙的思路
  - 查找删除节点的右子树的最小值节点
  - 将右子树的最小值节点的键值对设置到删除节点上, 这样再删除右子树的最小值节点即可
  - 因为右子树的最小值节点必然是无左子树, 则跳转为场景三或场景一

`deleteEntry(Object key)` 方法代码如下
```Java
/**
 * Delete node p, and then rebalance the tree.
 */
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    // If strictly internal, copy successor's element to p and then make p
    // point to successor.
    if (p.left != null && p.right != null) {
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    } // p has 2 children

    // Start fixup at replacement node, if it exists.
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    if (replacement != null) {
        // Link replacement to parent
        replacement.parent = p.parent;
        if (p.parent == null)
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;

        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;

        // Fix replacement
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) { // return if we are the only node.
        root = null;
    } else { //  No children. Use self as phantom replacement and unlink.
        if (p.color == BLACK)
            fixAfterDeletion(p);

        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```
- `modCount` 自增一, `size` 自减一
- 如果删除的节点 p 有左子节点也有右子节点
  - 获取删除节点的后继, 也就是右子树的最小值节点 s
  - 修改 p 节点的 `key-value` 为 s 节点的 `key-value`
  - 将 p 指向右子树最小值节点 s
- 如果 p 节点的左子节点不为 `null` 将其设置为替换节点 r, 否则为 p 节点的右子节点设置为替换节点
- 如果替换节点 r 不为 `null`
  - 将替换节点 r 的父节点设置为 p 的父节点
  - 如果 p 的父节点为 `null`, 将 r 设置为 `root` 节点
  - 如果 p 是父节点的左子节点, 则将 p 的父节点的孩子替换为 r, 否则则将 p 的父节点的右孩子替换为 r
  - 置空 p 节点的所有指向
  - 如果 p 节点的颜色为黑色, 则执行 r 节点的自平衡
- 如果 p 没有父节点为 `null`, 说明删除的是根节点, 并且红黑树只有根节点, 将 `root` 置为 `null` 即可
- 如果删除节点 p 也没有孩子节点
  - 如果 p 节点的颜色为黑色, 则执行 p 节点的自平衡
  - 删除与父节点的相互指向

这里有个查找节点后继方法 `successor(Entry<K,V> t)`, 其代码如下
```Java
/**
 * Returns the successor of the specified Entry, or null if no such.
 */
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```
- 如果 t 节点的右子节点不为 `null`
  - 查找其右子子树的最左节点 p (即为 t 节点右子树的最小值)
- 如果 t 节点的右子节点为 `null`
  - 向上获取 t 节点的父节点 p, 并将 t 赋值给 c
  - 然后不断向上变量父节点, 直到 c 不为 p 的右子节点
- 返回 p

#### 查找接近的元素
在 `NavigableMap` 接口中, 定义了四个查找接近的元素的方法
- lowerEntry(K key): 返回小于 key 的最大的 key 值节点或 `null`
- floorEntry(K key): 小于等于 key 的最大的 key 值节点或 `null`
- higherEntry(K key): 大于 key 的最小的 key 值节点或 `null`
- ceilingEntry(K key) 方法，大于等于 key 的最小的 key 值节点或 `null`

这里只看 `lowerEntry(K key)` 方法
```Java
/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @since 1.6
 */
public Map.Entry<K,V> lowerEntry(K key) {
    return exportEntry(getLowerEntry(key));
}
```
调用 `getLowerEntry(K key)` 然后使用 `exportEntry(TreeMap.Entry<K,V> e)` 包装为 `Entry` 返回
```Java
/**
 * Return SimpleImmutableEntry for entry, or null if null
 */
static <K,V> Map.Entry<K,V> exportEntry(TreeMap.Entry<K,V> e) {
    return (e == null) ? null :
        new AbstractMap.SimpleImmutableEntry<>(e);
}

/**
 * Returns the entry for the greatest key less than the specified key; if
 * no such entry exists (i.e., the least key in the Tree is greater than
 * the specified key), returns {@code null}.
 */
final Entry<K,V> getLowerEntry(K key) {
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = compare(key, p.key);
        if (cmp > 0) {
            if (p.right != null)
                p = p.right;
            else
                return p;
        } else {
            if (p.left != null) {
                p = p.left;
            } else {
                Entry<K,V> parent = p.parent;
                Entry<K,V> ch = p;
                while (parent != null && ch == parent.left) {
                    ch = parent;
                    parent = parent.parent;
                }
                return parent;
            }
        }
    }
    return null;
}
```
- 复制根节点 `root` 为 p
- 如果 p 不为 `null` 则进入循环, 否则返回 `null`
- 比较 `key` 与 p 节点的 key 值
- 如果 cmp 大于 0
 - 当 p 的右子树不为 `null` 时将 p 赋值为 p 的右子树
 - 当 p 的右子树为 `null` 时直接返回 p
- 如果 cmp 不大于 0
 - 当 p 的左子树不为 `null` 时将 p 赋值为 p 的左子树
 - 当 p 的左子树为 `null` 是找到其前驱节点返回

 #### 获得首尾节点
 ```Java
 /**
 * Returns the first Entry in the TreeMap (according to the TreeMap's
 * key-sort function).  Returns null if the TreeMap is empty.
 */
final Entry<K,V> getFirstEntry() {
    Entry<K,V> p = root;
    if (p != null)
        while (p.left != null)
            p = p.left;
    return p;
}

/**
 * Returns the last Entry in the TreeMap (according to the TreeMap's
 * key-sort function).  Returns null if the TreeMap is empty.
 */
final Entry<K,V> getLastEntry() {
    Entry<K,V> p = root;
    if (p != null)
        while (p.right != null)
            p = p.right;
    return p;
}
```
首节点即为红黑树最左节点, 尾节点即为红黑树最右节点

#### 清空
```Java
/**
 * Removes all of the mappings from this map.
 * The map will be empty after this call returns.
 */
public void clear() {
    modCount++;
    size = 0;
    root = null;
}
```
- `modCount` 自增一
- `size` 置为 0
- `root` 置为 `null`

#### 克隆
```Java
/**
 * Returns a shallow copy of this {@code TreeMap} instance. (The keys and
 * values themselves are not cloned.)
 *
 * @return a shallow copy of this map
 */
public Object clone() {
    TreeMap<?,?> clone;
    try {
        clone = (TreeMap<?,?>) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }

    // Put clone into "virgin" state (except for comparator)
    clone.root = null;
    clone.size = 0;
    clone.modCount = 0;
    clone.entrySet = null;
    clone.navigableKeySet = null;
    clone.descendingMap = null;

    // Initialize clone with our mappings
    try {
        clone.buildFromSorted(size, entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }

    return clone;
}
```
- 调用父类的 `clone()` 方法
- 初始化值
- 调用 `buildFromSorted(int size, Iterator<?> it, java.io.ObjectInputStream str, V defaultVal)` 方法构建红黑树

#### 序列化
```Java
/**
 * Save the state of the {@code TreeMap} instance to a stream (i.e.,
 * serialize it).
 *
 * @serialData The <em>size</em> of the TreeMap (the number of key-value
 *             mappings) is emitted (int), followed by the key (Object)
 *             and value (Object) for each key-value mapping represented
 *             by the TreeMap. The key-value mappings are emitted in
 *             key-order (as determined by the TreeMap's Comparator,
 *             or by the keys' natural ordering if the TreeMap has no
 *             Comparator).
 */
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out the Comparator and any hidden stuff
    s.defaultWriteObject();

    // Write out size (number of Mappings)
    s.writeInt(size);

    // Write out keys and values (alternating)
    for (Iterator<Map.Entry<K,V>> i = entrySet().iterator(); i.hasNext(); ) {
        Map.Entry<K,V> e = i.next();
        s.writeObject(e.getKey());
        s.writeObject(e.getValue());
    }
}
```
- 写入非静态属性, 非 `transient` 属性
- 写入 `size`
- 使用迭代器遍历写入具体的键值对

#### 反序列化
```Java
/**
 * Reconstitute the {@code TreeMap} instance from a stream (i.e.,
 * deserialize it).
 */
private void readObject(final java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in the Comparator and any hidden stuff
    s.defaultReadObject();

    // Read in size
    int size = s.readInt();

    buildFromSorted(size, null, s, null);
}
```
- 读出非静态属性, 非 `transient` 属性
- 读出 `size`
- 调用 `buildFromSorted(int size, Iterator<?> it, java.io.ObjectInputStream str, V defaultVal)` 方法构建红黑树

#### 迭代器
TODO
#### 查找范围的元素
TODO

#### 小结
- `TreeMap` 按照 key 的顺序的 Map 实现类, 底层采用红黑树来实现存储
- `TreeMap` 因为采用树结构, 所以无需初始考虑像 `HashMap` 考虑容量问题, 也不存在扩容问题
- `TreeMap` 的 key 不允许为 `null`, 可能是因为红黑树是一颗二叉查找树，需要对 key 进行排序
- `TreeMap` 的查找, 添加, 删除 `key-value` 键值对的平均时间复杂度为 `O(logN)`; 因为 `TreeMap` 采用红黑树, 操作都需要经过二分查找, 而二分查找的时间复杂度是 `O(logN)`
- 相比 `HashMap` 来说, `TreeMap` 不仅仅支持指定 key 的查找, 也支持 key 范围的查找; 这得益于 `TreeMap` 数据结构能够提供的有序特性

>**参考:**
- [集合（六）TreeMap](http://svip.iocoder.cn/JDK/Collection-TreeMap/)
- [教你初步了解红黑树](https://blog.csdn.net/v_july_v/article/details/6105630)
- [史上最清晰的红黑树讲解（上）》](https://www.cnblogs.com/CarpenterLee/p/5503882.html)
- [史上最清晰的红黑树讲解（下）](https://www.cnblogs.com/CarpenterLee/p/5525688.html)
