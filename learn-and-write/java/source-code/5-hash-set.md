### HashSet
`HashSet` 是基于 `HashMap` 的 `Set` 实现类; 有排重的需求可以考虑使用 `HashSet`

#### 类图
`HashSet` 实现的接口和继承的抽象类如下图  
![HashSet](http://static.iocoder.cn/images/JDK/2019_12_13/01.png)  
- `java.lang.Cloneable` 接口
- `java.io.Serializable` 接口
- `java.util.Set` 接口
- `java.util.AbstractSet` 抽象类

#### 属性
`HashSet` 只有一个属性就是 `map`
```Java
private transient HashMap<E,Object> map;

// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();
```
`map` 的 key 存储 `HashSet` 的每个 key, `map` 的 value 为统一的 `PRESENT`

#### 构造方法
`HashSet` 一共有五个构造方法
```Java
/**
 * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
 * default initial capacity (16) and load factor (0.75).
 */
public HashSet() {
    map = new HashMap<>();
}

/**
 * Constructs a new set containing the elements in the specified
 * collection.  The <tt>HashMap</tt> is created with default load factor
 * (0.75) and an initial capacity sufficient to contain the elements in
 * the specified collection.
 *
 * @param c the collection whose elements are to be placed into this set
 * @throws NullPointerException if the specified collection is null
 */
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}

/**
 * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
 * the specified initial capacity and the specified load factor.
 *
 * @param      initialCapacity   the initial capacity of the hash map
 * @param      loadFactor        the load factor of the hash map
 * @throws     IllegalArgumentException if the initial capacity is less
 *             than zero, or if the load factor is nonpositive
 */
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

/**
 * Constructs a new, empty set; the backing <tt>HashMap</tt> instance has
 * the specified initial capacity and default load factor (0.75).
 *
 * @param      initialCapacity   the initial capacity of the hash table
 * @throws     IllegalArgumentException if the initial capacity is less
 *             than zero
 */
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

/**
 * Constructs a new, empty linked hash set.  (This package private
 * constructor is only used by LinkedHashSet.) The backing
 * HashMap instance is a LinkedHashMap with the specified initial
 * capacity and the specified load factor.
 *
 * @param      initialCapacity   the initial capacity of the hash map
 * @param      loadFactor        the load factor of the hash map
 * @param      dummy             ignored (distinguishes this
 *             constructor from other int, float constructor.)
 * @throws     IllegalArgumentException if the initial capacity is less
 *             than zero, or if the load factor is nonpositive
 */
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```
五个构造方法都比较简单, 需要注意的是最后一个构造方法, 是包私有的构造方法, 使用 `LinkedHashMap` 构造, `dummy` 参数只是为了与其他构造方法做区分

#### 添加元素
##### 添加单个元素
```Java
/**
 * Adds the specified element to this set if it is not already present.
 * More formally, adds the specified element <tt>e</tt> to this set if
 * this set contains no element <tt>e2</tt> such that
 * <tt>(e==null&nbsp;?&nbsp;e2==null&nbsp;:&nbsp;e.equals(e2))</tt>.
 * If this set already contains the element, the call leaves the set
 * unchanged and returns <tt>false</tt>.
 *
 * @param e element to be added to this set
 * @return <tt>true</tt> if this set did not already contain the specified
 * element
 */
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```
`map` 的 value 值为 `PRESENT`
##### 添加多个元素 (java.util.AbstractCollection)
```Java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over the specified collection, and adds
 * each object returned by the iterator to this collection, in turn.
 *
 * <p>Note that this implementation will throw an
 * <tt>UnsupportedOperationException</tt> unless <tt>add</tt> is
 * overridden (assuming the specified collection is non-empty).
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 * @throws IllegalArgumentException      {@inheritDoc}
 * @throws IllegalStateException         {@inheritDoc}
 *
 * @see #add(Object)
 */
public boolean addAll(Collection<? extends E> c) {
    boolean modified = false;
    for (E e : c)
        if (add(e))
            modified = true;
    return modified;
}
```
添加多个方法继承自 `AbstractCollection` 抽象类

#### 移除元素
```Java
/**
 * Removes the specified element from this set if it is present.
 * More formally, removes an element <tt>e</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>,
 * if this set contains such an element.  Returns <tt>true</tt> if
 * this set contained the element (or equivalently, if this set
 * changed as a result of the call).  (This set will not contain the
 * element once the call returns.)
 *
 * @param o object to be removed from this set, if present
 * @return <tt>true</tt> if the set contained the specified element
 */
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```

#### 查找元素
```Java
/**
 * Returns <tt>true</tt> if this set contains the specified element.
 * More formally, returns <tt>true</tt> if and only if this set
 * contains an element <tt>e</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>.
 *
 * @param o element whose presence in this set is to be tested
 * @return <tt>true</tt> if this set contains the specified element
 */
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

#### 序列化
```Java
/**
 * Save the state of this <tt>HashSet</tt> instance to a stream (that is,
 * serialize it).
 *
 * @serialData The capacity of the backing <tt>HashMap</tt> instance
 *             (int), and its load factor (float) are emitted, followed by
 *             the size of the set (the number of elements it contains)
 *             (int), followed by all of its elements (each an Object) in
 *             no particular order.
 */
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out any hidden serialization magic
    s.defaultWriteObject();

    // Write out HashMap capacity and load factor
    s.writeInt(map.capacity());
    s.writeFloat(map.loadFactor());

    // Write out size
    s.writeInt(map.size());

    // Write out all elements in the proper order.
    for (E e : map.keySet())
        s.writeObject(e);
}
```
- 写入非静态属性, 非 `transient` 属性
- 写入 `map` 的 table 数组大小
- 写入 `map` 的加载因子
- 写入 `map` 的大小
- 遍历 `map` 将 `key` 逐个序列化

#### 反序列化
```Java
/**
 * Reconstitute the <tt>HashSet</tt> instance from a stream (that is,
 * deserialize it).
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in any hidden serialization magic
    s.defaultReadObject();

    // Read capacity and verify non-negative.
    int capacity = s.readInt();
    if (capacity < 0) {
        throw new InvalidObjectException("Illegal capacity: " +
                                         capacity);
    }

    // Read load factor and verify positive and non NaN.
    float loadFactor = s.readFloat();
    if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
        throw new InvalidObjectException("Illegal load factor: " +
                                         loadFactor);
    }

    // Read size and verify non-negative.
    int size = s.readInt();
    if (size < 0) {
        throw new InvalidObjectException("Illegal size: " +
                                         size);
    }
    // Set the capacity according to the size and load factor ensuring that
    // the HashMap is at least 25% full but clamping to maximum capacity.
    capacity = (int) Math.min(size * Math.min(1 / loadFactor, 4.0f),
            HashMap.MAXIMUM_CAPACITY);

    // Constructing the backing map will lazily create an array when the first element is
    // added, so check it before construction. Call HashMap.tableSizeFor to compute the
    // actual allocation size. Check Map.Entry[].class since it's the nearest public type to
    // what is actually created.

    SharedSecrets.getJavaOISAccess()
                 .checkArray(s, Map.Entry[].class, HashMap.tableSizeFor(capacity));

    // Create backing HashMap
    map = (((HashSet<?>)this) instanceof LinkedHashSet ?
           new LinkedHashMap<E,Object>(capacity, loadFactor) :
           new HashMap<E,Object>(capacity, loadFactor));

    // Read in all elements in the proper order.
    for (int i=0; i<size; i++) {
        @SuppressWarnings("unchecked")
            E e = (E) s.readObject();
        map.put(e, PRESENT);
    }
}
```
- 读取非静态属性, 非 `transient` 属性
- 读取 `map` 的 table 数组大小并校验
- 读取 `map` 的加载因子并校验
- 读取 `map` 的大小并校验
- 根据是 `LinkedHashSet` 还是 `HashSet` 构造 `map`
- 读取 `key` 依次添加到 `map` 中

#### 克隆
```Java
/**
 * Returns a shallow copy of this <tt>HashSet</tt> instance: the elements
 * themselves are not cloned.
 *
 * @return a shallow copy of this set
 */
@SuppressWarnings("unchecked")
public Object clone() {
    try {
        HashSet<E> newSet = (HashSet<E>) super.clone();
        newSet.map = (HashMap<E, Object>) map.clone();
        return newSet;
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }
}
```
- 调用父方法，克隆创建 `HashSet` 对象
- 克隆 `map` 属性赋值给克隆的 `HashSet`
- 返回克隆对象

#### 小结
`HashSet` 是基于 `HashMap` 的 `Set` 实现类

>**参考:**
- [集合（五）哈希集合 HashSet](http://svip.iocoder.cn/JDK/Collection-HashSet/)
