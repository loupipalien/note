### TreeSet
`TreeSet` 基于 `TreeMap` 的 `Set` 实现类

#### 类图
`TreeSet` 实现的接口和继承的抽象类如下图   
![TreeSet](http://static.iocoder.cn/images/JDK/2019_12_16/01.png)  
- 继承 `java.util.AbstractSet` 抽像类
- 实现 `java.util.NavigableSet` 接口
- 实现 `java.io.Serializable` 接口
- 实现 `java.lang.Cloneable` 接口

#### 属性
`TreeSet` 只有一个属性 `m`
```Java
/**
 * The backing map.
 */
private transient NavigableMap<E,Object> m;

// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
```
`m` 的 key 存储 `TreeSet ` 的每个 key, `m` 的 value 为统一的 `PRESENT`

#### 构造方法
`TreeSet` 一共有五个构造方法
```Java
/**
 * Constructs a set backed by the specified navigable map.
 */
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}

/**
 * Constructs a new, empty tree set, sorted according to the
 * natural ordering of its elements.  All elements inserted into
 * the set must implement the {@link Comparable} interface.
 * Furthermore, all such elements must be <i>mutually
 * comparable</i>: {@code e1.compareTo(e2)} must not throw a
 * {@code ClassCastException} for any elements {@code e1} and
 * {@code e2} in the set.  If the user attempts to add an element
 * to the set that violates this constraint (for example, the user
 * attempts to add a string element to a set whose elements are
 * integers), the {@code add} call will throw a
 * {@code ClassCastException}.
 */
public TreeSet() {
    this(new TreeMap<E,Object>());
}

/**
 * Constructs a new, empty tree set, sorted according to the specified
 * comparator.  All elements inserted into the set must be <i>mutually
 * comparable</i> by the specified comparator: {@code comparator.compare(e1,
 * e2)} must not throw a {@code ClassCastException} for any elements
 * {@code e1} and {@code e2} in the set.  If the user attempts to add
 * an element to the set that violates this constraint, the
 * {@code add} call will throw a {@code ClassCastException}.
 *
 * @param comparator the comparator that will be used to order this set.
 *        If {@code null}, the {@linkplain Comparable natural
 *        ordering} of the elements will be used.
 */
public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}

/**
 * Constructs a new tree set containing the elements in the specified
 * collection, sorted according to the <i>natural ordering</i> of its
 * elements.  All elements inserted into the set must implement the
 * {@link Comparable} interface.  Furthermore, all such elements must be
 * <i>mutually comparable</i>: {@code e1.compareTo(e2)} must not throw a
 * {@code ClassCastException} for any elements {@code e1} and
 * {@code e2} in the set.
 *
 * @param c collection whose elements will comprise the new set
 * @throws ClassCastException if the elements in {@code c} are
 *         not {@link Comparable}, or are not mutually comparable
 * @throws NullPointerException if the specified collection is null
 */
public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}

/**
 * Constructs a new tree set containing the same elements and
 * using the same ordering as the specified sorted set.
 *
 * @param s sorted set whose elements will comprise the new set
 * @throws NullPointerException if the specified sorted set is null
 */
public TreeSet(SortedSet<E> s) {
    this(s.comparator());
    addAll(s);
}
```
在构造方法中, 会创建 `TreeMap` 对象, 赋予到 m 属性

#### 添加元素
##### 添加单个元素
```Java
/**
 * Adds the specified element to this set if it is not already present.
 * More formally, adds the specified element {@code e} to this set if
 * the set contains no element {@code e2} such that
 * <tt>(e==null&nbsp;?&nbsp;e2==null&nbsp;:&nbsp;e.equals(e2))</tt>.
 * If this set already contains the element, the call leaves the set
 * unchanged and returns {@code false}.
 *
 * @param e element to be added to this set
 * @return {@code true} if this set did not already contain the specified
 *         element
 * @throws ClassCastException if the specified object cannot be compared
 *         with the elements currently in this set
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 */
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
```
`m` 的 value 值就是 `PRESENT`
##### 添加多个元素
```Java
/**
 * Adds all of the elements in the specified collection to this set.
 *
 * @param c collection containing elements to be added to this set
 * @return {@code true} if this set changed as a result of the call
 * @throws ClassCastException if the elements provided cannot be compared
 *         with the elements currently in the set
 * @throws NullPointerException if the specified collection is null or
 *         if any element is null and this set uses natural ordering, or
 *         its comparator does not permit null elements
 */
public  boolean addAll(Collection<? extends E> c) {
    // Use linear-time version if applicable
    if (m.size()==0 && c.size() > 0 &&
        c instanceof SortedSet &&
        m instanceof TreeMap) {
        SortedSet<? extends E> set = (SortedSet<? extends E>) c;
        TreeMap<E,Object> map = (TreeMap<E, Object>) m;
        Comparator<?> cc = set.comparator();
        Comparator<? super E> mc = map.comparator();
        if (cc==mc || (cc != null && cc.equals(mc))) {
            map.addAllForTreeSet(set, PRESENT);
            return true;
        }
    }
    return super.addAll(c);
}
```
在实现上和 `TreeMap` 的批量添加是一样的, 对于前一种情况会进行优化

#### 移除元素
```Java
/**
 * Removes the specified element from this set if it is present.
 * More formally, removes an element {@code e} such that
 * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>,
 * if this set contains such an element.  Returns {@code true} if
 * this set contained the element (or equivalently, if this set
 * changed as a result of the call).  (This set will not contain the
 * element once the call returns.)
 *
 * @param o object to be removed from this set, if present
 * @return {@code true} if this set contained the specified element
 * @throws ClassCastException if the specified object cannot be compared
 *         with the elements currently in this set
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 */
public boolean remove(Object o) {
    return m.remove(o)==PRESENT;
}
```

#### 查找接近元素
在 `NavigableSet` 中, 定义了四个查找接近的元素
- lower(E e): 返回小于 key 的最大的 key 值元素或 `null`
- floor(E e): 小于等于 key 的最大的 key 值元素或 `null`
- higher(E e): 大于 key 的最小的 key 值元素或 `null`
- ceiling(E e) 方法，大于等于 key 的最小的 key 值元素或 `null`

```Java
/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 * @since 1.6
 */
public E lower(E e) {
    return m.lowerKey(e);
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 * @since 1.6
 */
public E floor(E e) {
    return m.floorKey(e);
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 * @since 1.6
 */
public E higher(E e) {
    return m.higherKey(e);
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified element is null
 *         and this set uses natural ordering, or its comparator
 *         does not permit null elements
 * @since 1.6
 */
public E ceiling(E e) {
    return m.ceilingKey(e);
}
```

#### 清空
```Java
/**
 * Removes all of the elements from this set.
 * The set will be empty after this call returns.
 */
public void clear() {
    m.clear();
}
```

#### 克隆
```Java
/**
 * Returns a shallow copy of this {@code TreeSet} instance. (The elements
 * themselves are not cloned.)
 *
 * @return a shallow copy of this set
 */
@SuppressWarnings("unchecked")
public Object clone() {
    TreeSet<E> clone;
    try {
        clone = (TreeSet<E>) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }

    clone.m = new TreeMap<>(m);
    return clone;
}
```
- 克隆创建 `TreeSet` 对象 `clone`
- 创建 `TreeMap` 对象, 赋值给 `clone` 的 `m` 属性

#### 序列化
```Java
/**
 * Save the state of the {@code TreeSet} instance to a stream (that is,
 * serialize it).
 *
 * @serialData Emits the comparator used to order this set, or
 *             {@code null} if it obeys its elements' natural ordering
 *             (Object), followed by the size of the set (the number of
 *             elements it contains) (int), followed by all of its
 *             elements (each an Object) in order (as determined by the
 *             set's Comparator, or by the elements' natural ordering if
 *             the set has no Comparator).
 */
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out any hidden stuff
    s.defaultWriteObject();

    // Write out Comparator
    s.writeObject(m.comparator());

    // Write out size
    s.writeInt(m.size());

    // Write out all elements in the proper order.
    for (E e : m.keySet())
        s.writeObject(e);
}
```
- 写入非静态属性和非 `transient` 属性
- 写入比较器
- 写入 `key-value` 键值对数量
- 写入具体的 `key-value` 键值对

#### 反序列化
```Java
/**
 * Reconstitute the {@code TreeSet} instance from a stream (that is,
 * deserialize it).
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in any hidden stuff
    s.defaultReadObject();

    // Read in Comparator
    @SuppressWarnings("unchecked")
        Comparator<? super E> c = (Comparator<? super E>) s.readObject();

    // Create backing TreeMap
    TreeMap<E,Object> tm = new TreeMap<>(c);
    m = tm;

    // Read in size
    int size = s.readInt();

    tm.readTreeSet(size, s, PRESENT);
}
```
- 读取非静态属性和非 `transient` 属性
- 读取比较器
- 读取 `key-value` 键值对数量
- 读取具体的 `key-value` 键值对

#### 小结
`TreeSet` 基于 `TreeMap` 的 `Set` 实现类

>**参考:**
- [集合（七）TreeSet](http://svip.iocoder.cn/JDK/Collection-TreeSet/)
