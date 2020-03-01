### ArrayList (JDK1.8.0_151)
ArrayList 是基于数组实现的, 支持自动扩容的列表

#### 类图
`ArrayList` 实现的接口和继承的抽象类如下图
![ArrayList](http://static.iocoder.cn/images/JDK/2019_12_01/01.png)  
ArrayList 实现了四个接口并继承了一个抽象类
- `java.lang.Cloneable` 接口, 表示支持克隆
- `java.io.Serializable` 接口, 表示支持序列化的功能
- `java.util.RandomAccess` 接口, 表示支持快速的随机访问
- `java.util.List` 接口, 提供数组的添加, 删除, 修改, 迭代遍历等操作
- `java.util.AbstractList` 抽象类, AbstractList 提供了 List 接口的骨架实现, 大幅度减少了实现迭代遍历相关操作的代码

#### 属性
ArrayList 的属性很少, 仅仅只有 2 个: `elementData` 和 `size`
![ArrayList](http://static.iocoder.cn/images/JDK/2019_12_01/02.png)
- `elementData`: 元素数组
- `size`: 表示 ArrayList 已使用的数组大小

```Java
/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
 */
transient Object[] elementData; // non-private to simplify nested class access

/**
 * The size of the ArrayList (the number of elements it contains).
 *
 * @serial
 */
private int size;
```

#### 构造方法
ArrayList 一共三个构造方法
##### ArrayList(int initialCapacity)
根据传入的初始化容量, 创建 elementData 数组; 如果在使用时预先知道数组大小, 一定要使用此构造方法, 可以避免数组扩容提升性能, 同时也是合理使用内存
```Java
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```
##### ArrayList(Collection<? extends E> c)
传入的参数是一个集合, 将其转化为数组作为 elementData
```Java
/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```
##### ArrayList()
无参构造方法, 也是使用的最多的方法
```Java
/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```
这里可以看到无参构造方法是设置了一个空数组, 在添加第一个元素的时候才真正初始化为容量 10 的数组; 除此之外还需要注意的是, 有参构造的空 ArrayList 和无参构造的空 ArrayList 分别使用的是 `EMPTY_ELEMENTDATA` 和 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`

#### 扩容
```Java
/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
- 先按照当前的数组大小扩容 1.5 倍
- 如果扩容后的大小仍然小于需求的大小, 则扩容至需求的大小
- 扩容后的大小如果超过 MAX_ARRAY_SIZE, 则进行进一步确认
- 进行扩容数组的拷贝

#### 缩容
```Java
/**
 * Trims the capacity of this <tt>ArrayList</tt> instance to be the
 * list's current size.  An application can use this operation to minimize
 * the storage of an <tt>ArrayList</tt> instance.
 */
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```
- 自增修改次数
- 如果有多余空间则进行缩容

#### 添加元素
##### 添加单个元素
```Java
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```
- 第一行确保容量足够, 如果不足则需调用 `#grow()` 方法来扩容; 并自增 `modCount` 变量, 用于记录数组修改的次数  
- 第二行将新增元素添加到末尾, 并将 `size` 自增
- 发挥添加结果

##### 添加多个元素
```Java
/**
 * Appends all of the elements in the specified collection to the end of
 * this list, in the order that they are returned by the
 * specified collection's Iterator.  The behavior of this operation is
 * undefined if the specified collection is modified while the operation
 * is in progress.  (This implies that the behavior of this call is
 * undefined if the specified collection is this list, and this
 * list is nonempty.)
 *
 * @param c collection containing elements to be added to this list
 * @return <tt>true</tt> if this list changed as a result of the call
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}
```
- 第一行将集合转为数组
- 确认容量大小, 并自增 `modCount`
- 将新增元素复制到 `elementData` 中
- 设置 `size` 的大小
- 返回列表是否被改变的结果

#### 移除元素
#### 移除指定位置的元素
`remove(int index)` 方法, 移除指定位置的元素, 并返回该位置的原元素
```Java
/**
 * Removes the element at the specified position in this list.
 * Shifts any subsequent elements to the left (subtracts one from their
 * indices).
 *
 * @param index the index of the element to be removed
 * @return the element that was removed from the list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```
- 检查移除元素的下标是否有效
- 自增 `modCount` 值并获取要移除的元素
- 将此元素后续的值向前赋值
- 将原有的最后一位置为 null
- 返回被移除的元素

##### 移除指定元素
```Java
/**
 * Removes the first occurrence of the specified element from this list,
 * if it is present.  If the list does not contain the element, it is
 * unchanged.  More formally, removes the element with the lowest index
 * <tt>i</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
 * (if such an element exists).  Returns <tt>true</tt> if this list
 * contained the specified element (or equivalently, if this list
 * changed as a result of the call).
 *
 * @param o element to be removed from this list, if present
 * @return <tt>true</tt> if this list contained the specified element
 */
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}
```
- 遍历 `elementData` 找到相等的元素并移除

##### 移除多个元素
`#removeRange(int fromIndex, int toIndex)` 方法, 批量移除 `[fromIndex, toIndex)` 的多个元素
```Java
/**
 * Removes from this list all of its elements that are contained in the
 * specified collection.
 *
 * @param c collection containing elements to be removed from this list
 * @return {@code true} if this list changed as a result of the call
 * @throws ClassCastException if the class of an element of this list
 *         is incompatible with the specified collection
 * (<a href="Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if this list contains a null element and the
 *         specified collection does not permit null elements
 * (<a href="Collection.html#optional-restrictions">optional</a>),
 *         or if the specified collection is null
 * @see Collection#contains(Object)
 */
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}

private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```
- 将集合 `c` 中不包含的元素从头依次覆盖到 `elementData`
- 将下标 `w` 之后的元素置为 `null`, 并修改 `modCount` 的值
- 返回是否被修改的结果

#### 查找单个元素
`indexOf(Object o)` 方法, 查找首个为指定元素的位置
```Java
/**
 * Returns the index of the first occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the lowest index <tt>i</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * or -1 if there is no such index.
 */
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```
- 遍历寻找在 `elementData` 中首次出现的要查找元素的位置
- 如果找到返回位置下标, 否则返回 -1

`contains(Object o)` 方法是基于此方法实现的, 类似相同的实现还有 `lastIndexOf(Object o)` 方法

#### 获得指定位置的元素
```Java
/**
 * Returns the element at the specified position in this list.
 *
 * @param  index index of the element to return
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```
- 检查下标是否在有效的范围内
- 返回 `elementData` 数组中对应下标的元素

#### 设置指定位置的元素
```Java
/**
 * Replaces the element at the specified position in this list with
 * the specified element.
 *
 * @param index index of the element to replace
 * @param element element to be stored at the specified position
 * @return the element previously at the specified position
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E set(int index, E element) {
    rangeCheck(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```
- 检查下标是否在有效的范围内
- 获取下标位置原有值
- 将下标位置设置新的值
- 返回原有值

#### 转换为数组
##### 转换为 Object[] 数组
```Java
/**
 * Returns an array containing all of the elements in this list
 * in proper sequence (from first to last element).
 *
 * <p>The returned array will be "safe" in that no references to it are
 * maintained by this list.  (In other words, this method must allocate
 * a new array).  The caller is thus free to modify the returned array.
 *
 * <p>This method acts as bridge between array-based and collection-based
 * APIs.
 *
 * @return an array containing all of the elements in this list in
 *         proper sequence
 */
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
```
- 利用 `Arrays` 工具类完成复制后返回

##### 转换为 T[] 数组
```Java
/**
 * Returns an array containing all of the elements in this list in proper
 * sequence (from first to last element); the runtime type of the returned
 * array is that of the specified array.  If the list fits in the
 * specified array, it is returned therein.  Otherwise, a new array is
 * allocated with the runtime type of the specified array and the size of
 * this list.
 *
 * <p>If the list fits in the specified array with room to spare
 * (i.e., the array has more elements than the list), the element in
 * the array immediately following the end of the collection is set to
 * <tt>null</tt>.  (This is useful in determining the length of the
 * list <i>only</i> if the caller knows that the list does not contain
 * any null elements.)
 *
 * @param a the array into which the elements of the list are to
 *          be stored, if it is big enough; otherwise, a new array of the
 *          same runtime type is allocated for this purpose.
 * @return an array containing the elements of the list
 * @throws ArrayStoreException if the runtime type of the specified array
 *         is not a supertype of the runtime type of every element in
 *         this list
 * @throws NullPointerException if the specified array is null
 */
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
```
- 如果参数数组长度小于 `size`, 则返回一个新数组 (所以最好使用 `a = list.toArray(1)` 的形式)
- 如果参数数组长度小于 `size`, 则将 `elementData` 中的元素复制到数组 `a` 中, 并将 size 下标置为 `null` (此操作可在元素中没有 `null` 的情况下判定列表的长度), 最后返回数组 `a`

#### 求哈希值 (AbstractList)
```Java
/**
 * Returns the hash code value for this list.
 *
 * <p>This implementation uses exactly the code that is used to define the
 * list hash function in the documentation for the {@link List#hashCode}
 * method.
 *
 * @return the hash code value for this list
 */
public int hashCode() {
    int hashCode = 1;
    for (E e : this)
        hashCode = 31*hashCode + (e==null ? 0 : e.hashCode());
    return hashCode;
}
```
- 使用 31 作为乘子计算 (PS: [为什么 hashCode 方法选择数字 31 作为乘子](http://www.iocoder.cn/Fight/Why-did-the-String-hashCode-method-select-the-number-31-as-a-multiplier/?vip&self))

#### 判断相等 (AbstractList)
```Java
/**
 * Compares the specified object with this list for equality.  Returns
 * {@code true} if and only if the specified object is also a list, both
 * lists have the same size, and all corresponding pairs of elements in
 * the two lists are <i>equal</i>.  (Two elements {@code e1} and
 * {@code e2} are <i>equal</i> if {@code (e1==null ? e2==null :
 * e1.equals(e2))}.)  In other words, two lists are defined to be
 * equal if they contain the same elements in the same order.<p>
 *
 * This implementation first checks if the specified object is this
 * list. If so, it returns {@code true}; if not, it checks if the
 * specified object is a list. If not, it returns {@code false}; if so,
 * it iterates over both lists, comparing corresponding pairs of elements.
 * If any comparison returns {@code false}, this method returns
 * {@code false}.  If either iterator runs out of elements before the
 * other it returns {@code false} (as the lists are of unequal length);
 * otherwise it returns {@code true} when the iterations complete.
 *
 * @param o the object to be compared for equality with this list
 * @return {@code true} if the specified object is equal to this list
 */
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
- 检查对象 `o` 是否为本身或 `List` 对象
- 遍历比较列表中的元素, 如有不等则返回 `false`
- 如列表元素个数不等则返回 `false`

#### 清空数组
```Java
/**
 * Removes all of the elements from this list.  The list will
 * be empty after this call returns.
 */
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```
- 自增 `modCount` 的值
- 将 `elementData` 数组中的前 `size` 个元素置为 `null`
- 重置 `size` 为 0

#### 序列化
```Java
/**
 * Save the state of the <tt>ArrayList</tt> instance to a stream (that
 * is, serialize it).
 *
 * @serialData The length of the array backing the <tt>ArrayList</tt>
 *             instance is emitted (int), followed by all of its elements
 *             (each an <tt>Object</tt>) in the proper order.
 */
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```
- 调用 `ObjectOutputStream.defaultWriteObject()` 方法, 写入非静态属性和非 `transient` 属性; 相关序列化知识可见 [Serializable 原理](https://juejin.im/entry/5bf622436fb9a04a0b21cbe7)
- 写入 `size`, 主要为了和 `clone` 方法兼容; 第一步在 `ObjectOutputStream.defaultWriteObject()` 中已经写入了 `size`, 至于为什么再写一遍, 目前只有看到这个讨论 [源码分析：ArrayList 的 writeobject 方法中的实现是否多此一举](https://www.zhihu.com/question/41512382)
- 逐个写入 `elementData` 的元素, 此字段使用了 `transient` 关键字修饰, 这是因为 `elementData` 数组不一定是全满的, 如果全部序列化会有很多空间浪费, 所以只序列化 `[0, size)` 的元素以减少空间占用

#### 反序列化
```Java
/**
 * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,
 * deserialize it).
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```
- 读取非静态和非 `transient` 的字段
- 读取序列化时多写入的 `size` 忽略
- 确认 `elementData` 容量后逐个读取赋值

#### 克隆
```Java
/**
 * Returns a shallow copy of this <tt>ArrayList</tt> instance.  (The
 * elements themselves are not copied.)
 *
 * @return a clone of this <tt>ArrayList</tt> instance
 */
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```
- 这里没看到 size 的拷贝, 应该是在 `super.clone()` 的方法里处理的, 具体代码待探究

#### 创建子数组
```Java
/**
 * Returns a view of the portion of this list between the specified
 * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.  (If
 * {@code fromIndex} and {@code toIndex} are equal, the returned list is
 * empty.)  The returned list is backed by this list, so non-structural
 * changes in the returned list are reflected in this list, and vice-versa.
 * The returned list supports all of the optional list operations.
 *
 * <p>This method eliminates the need for explicit range operations (of
 * the sort that commonly exist for arrays).  Any operation that expects
 * a list can be used as a range operation by passing a subList view
 * instead of a whole list.  For example, the following idiom
 * removes a range of elements from a list:
 * <pre>
 *      list.subList(from, to).clear();
 * </pre>
 * Similar idioms may be constructed for {@link #indexOf(Object)} and
 * {@link #lastIndexOf(Object)}, and all of the algorithms in the
 * {@link Collections} class can be applied to a subList.
 *
 * <p>The semantics of the list returned by this method become undefined if
 * the backing list (i.e., this list) is <i>structurally modified</i> in
 * any way other than via the returned list.  (Structural modifications are
 * those that change the size of this list, or otherwise perturb it in such
 * a fashion that iterations in progress may yield incorrect results.)
 *
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws IllegalArgumentException {@inheritDoc}
 */
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}
```
- 检查下标范围的有效性
- 返回 `SubList` 类实例, 需要注意的是子列表和父列表共享 `elementData` 数组

#### 创建 Iterator 迭代器
```Java
/**
 * Returns an iterator over the elements in this list in proper sequence.
 *
 * <p>The returned iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
 *
 * @return an iterator over the elements in this list in proper sequence
 */
public Iterator<E> iterator() {
    return new Itr();
}
```
`Itr` 迭代器实现了 `java.util.Iterator` 接口, 是 `ArrayList` 的内部类, 虽说 `AbstractList` 也实现了一个 `Itr` 的实现, 但是 `ArrayList` 为了更好的性能所以在内部实现了, 其类上也注释了 `An optimized version of AbstractList.Itr`

##### ArrayList.Itr
`Itr` 类一共三个属性
- cursor: 表示下一个访问元素的下标
- lastRet: 表示上一个我访问元素的位置; 初始化时为 -1, 指向的元素被移除也置为 -1
- expectedModCount: 期望的修改次数; 如果在迭代的过程中列表发生了变化, 会抛出 `ConcurrentModificationException`

###### 判断是否有下一个元素
```Java
public boolean hasNext() {
    return cursor != size;
}
```
- 非常简单, 判断 `cursor` 是否到数组末尾即可
###### 获取下一个元素
```Java
public E next() {
    checkForComodification();
    int i = cursor;
    if (i >= size)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i + 1;
    return (E) elementData[lastRet = i];
}
```
- 检查是否有修改
- 检查 `cursor` 是否大于等于 `size`
- 检查 `cursor` 是否大于等于 `elementData.length`
- `cursor` 自增加一
- 将 `lastRet` 置为当前下标, 并返回下标上的值

###### 移除当前元素
```Java
public void remove() {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();

    try {
        ArrayList.this.remove(lastRet);
        cursor = lastRet;
        lastRet = -1;
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```
- 判断 `lastRet` 是否为有效下标
- 检查是够有修改
- 调用 `ArrayList.remove` 方法移除当前元素
- `cursor` 重置为当前下标, `lastRet` 置为 `-1`
- 重置 `expectedModCount` 的值 (因为 `ArrayList.remove` 会修改 `modCount` 的值)

###### 遍历消费剩余元素
```Java
public void forEachRemaining(Consumer<? super E> consumer) {
    Objects.requireNonNull(consumer);
    final int size = ArrayList.this.size;
    int i = cursor;
    if (i >= size) {
        return;
    }
    final Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length) {
        throw new ConcurrentModificationException();
    }
    while (i != size && modCount == expectedModCount) {
        consumer.accept((E) elementData[i++]);
    }
    // update once at end of iteration to reduce heap write traffic
    cursor = i;
    lastRet = i - 1;
    checkForComodification();
}
```
- 检查是否还有剩余元素
- 检查 `cursor` 的值是否大于等于 `elementData.length`
- 消费剩余元素
- 重置 `cursor` 和 `lastRet` 的值
- 检查数组是否有变化

#### 创建 ListIterator 迭代器
```Java
/**
 * Returns a list iterator over the elements in this list (in proper
 * sequence), starting at the specified position in the list.
 * The specified index indicates the first element that would be
 * returned by an initial call to {@link ListIterator#next next}.
 * An initial call to {@link ListIterator#previous previous} would
 * return the element with the specified index minus one.
 *
 * <p>The returned list iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
 *
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public ListIterator<E> listIterator(int index) {
    if (index < 0 || index > size)
        throw new IndexOutOfBoundsException("Index: "+index);
    return new ListItr(index);
}
```
`ListItr` 迭代器实现了 `java.util.ListIterator` 接口, 是 `ArrayList` 的内部类, 虽说 `AbstractList` 也实现了一个 `ListItr` 的实现, 但是 `ArrayList` 为了更好的性能所以在内部实现了, 其类上也注释了 `An optimized version of AbstractList.Itr`

##### ArrayList.ListItr
`ListItr` 直接继承 `Itr` 类, 无自定义的属性; 构造方法如下
```Java
ListItr(int index) {
    super();
    cursor = index;
}
```
可以手动设置指定的位置开始迭代

###### 获取前一个元素
```Java
public E previous() {
    checkForComodification();
    int i = cursor - 1;
    if (i < 0)
        throw new NoSuchElementException();
    Object[] elementData = ArrayList.this.elementData;
    if (i >= elementData.length)
        throw new ConcurrentModificationException();
    cursor = i;
    return (E) elementData[lastRet = i];
}
```
- 检查是否有修改
- 获取前一个元素的下标
- 检查前一个元素下标的有效性
- 返回前一个元素的值并将 `lastRet` 设置为前一个元素的下标

###### 设置元素
```Java
public void set(E e) {
    if (lastRet < 0)
        throw new IllegalStateException();
    checkForComodification();

    try {
        ArrayList.this.set(lastRet, e);
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```
- 检查 `lastRet` 的有效性
- 调用 `ArrayList.set` 方法设置 `lastRet` 的值

###### 添加元素
```Java
public void add(E e) {
    checkForComodification();

    try {
        int i = cursor;
        ArrayList.this.add(i, e);
        cursor = i + 1;
        lastRet = -1;
        expectedModCount = modCount;
    } catch (IndexOutOfBoundsException ex) {
        throw new ConcurrentModificationException();
    }
}
```
- 检查是否有修改
- 调用 `ArrayList.add` 方法设置 `cursor` 的值
- 修改 `cursor` 和 `lastRet` 的值
- 重置 `expectedModCount` 的值为 `modCount`


#### 其他方法
- spliterator()
- removeIf(Predicate<? super E> filter)
- replaceAll(UnaryOperator<E> operator)
- sort(Comparator<? super E> c)
- forEach(Consumer<? super E> action)

#### 小结
- `ArrayList` 是基于数组的 `List` 实现类, 支持在数组容量不够时, 一般按照 1.5 倍自动扩容; 同时它支持手动扩容和手动缩容。
- `ArrayList` 随机访问时间复杂度是 `O(1)`, 查找指定元素的平均时间复杂度是 `O(n)`
