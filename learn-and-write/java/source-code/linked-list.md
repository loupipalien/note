### LinkedList (JDK1.8.0_151)
LinkedList 是基于双向链表实现的列表, 链表中的每个节点都有指向前驱和后继的指针

#### 类图
`LinkedList` 实现的接口和继承的抽象类如下图  
![LinkedList](http://static.iocoder.cn/images/JDK/2019_12_04/01.jpg)  
LinkedList 也实现了四个接口并继承了一个抽象类
- `java.util.List` 接口
- `java.lang.Cloneable` 接口
- `java.io.Serializable` 接口
- `java.util.Deque` 接口, 提供双端队列的功能, LinkedList 支持快速的在头尾添加元素和读取元素
- `java.util.AbstractSequentialList` 抽象类, 是 `AbstractList` 的子类, 实现了只能连续访问 `数据存储` 的 `get(int index), add(int index, E element)` 等随机操作的方法

#### 属性
在 `LinkedList` 中通过 `Node` 节点指向前驱和后继从而形成双向链表
- `first` 和 `last` 属性
  - 在初始的时候, `first` 和 `last` 指向 `null`, 因为暂时没有节点
  - 当添加节点后, `first` 执行头节点, `last` 指向尾节点
- `size` 属性
链表的节点数量, 通过它进行计数, 避免每次需要 `List` 大小时, 需要从头到尾遍历链表

对应代码如下
```Java
transient int size = 0;

/**
 * Pointer to first node.
 * Invariant: (first == null && last == null) ||
 *            (first.prev == null && first.item != null)
 */
transient Node<E> first;

/**
 * Pointer to last node.
 * Invariant: (first == null && last == null) ||
 *            (last.next == null && last.item != null)
 */
transient Node<E> last;

private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

#### 构造方法
`LinkedList` 一共有两个构造方法
```Java
/**
 * Constructs an empty list.
 */
public LinkedList() {
}

/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param  c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

#### 添加单个元素
##### 添加到列表末尾
```Java
/**
 * Appends the specified element to the end of this list.
 *
 * <p>This method is equivalent to {@link #addLast}.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}

/**
 * Links e as last element.
 */
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```
`add(E e)` 的方法比较简单, 就是调用 `linkLast(E e)` 方法将元素添加到链表的末尾
- 将 `last` 节点的值保存到一个临时节点
- 将添加元素包装为 `Node` 节点, 并赋值了前驱
- 将新节点赋值给 `last` 节点
- 如果之前的 `last` 的值为 `null`, 表示原来没有节点, 否则将新节点赋值为原 `last` 的后继
- `size` 自增, `modCount` 自增
##### 添加到列表的指定位置
```Java
/**
 * Inserts the specified element at the specified position in this list.
 * Shifts the element currently at that position (if any) and any
 * subsequent elements to the right (adds one to their indices).
 *
 * @param index index at which the specified element is to be inserted
 * @param element element to be inserted
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```
- 如果 `index` 刚好等于链表的大小, 直接调用 `linkLast(E element)` 方法添加到尾部即可
- 如果不等于则先调用 `node(int index)` 的方法获得第 `index` 位置的节点, 然后调用 `linkBefore(E element, Node node)` 方法, 将新节点添加到 `node` 的前面, 即 `node` 的前一个节点的 `next` 指向新节点, `node` 的 `prev` 指向新节点
###### node(int index)
```Java
/**
 * Returns the (non-null) Node at the specified element index.
 */
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```
这里有一个类似二分的操作, 如果 `index` 小于 `size` 的一半则从前向后遍历寻找, 反之则从后向前寻找
###### linkBefore(E e, Node<E> succ)
```Java
/**
 * Inserts element e before non-null Node succ.
 */
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
}
```
这里操作比较简单, 就是将 `e` 节点作为一个非空节点的前驱插入

##### 添加头节点和尾节点
因为 `LinkedList` 实现了 `Deque` 接口, 所以它实现了 `addFisrt(E e)` 和 `addLast(E e)` 方法以及 `offerFirst(E e)` 和 `offerLast(E e)`, 代码如下
```Java
/**
 * Inserts the specified element at the beginning of this list.
 *
 * @param e the element to add
 */
public void addFirst(E e) {
    linkFirst(e);
}

/**
 * Appends the specified element to the end of this list.
 *
 * <p>This method is equivalent to {@link #add}.
 *
 * @param e the element to add
 */
public void addLast(E e) {
    linkLast(e);
}

/**
 * Inserts the specified element at the front of this list.
 *
 * @param e the element to insert
 * @return {@code true} (as specified by {@link Deque#offerFirst})
 * @since 1.6
 */
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

/**
 * Inserts the specified element at the end of this list.
 *
 * @param e the element to insert
 * @return {@code true} (as specified by {@link Deque#offerLast})
 * @since 1.6
 */
public boolean offerLast(E e) {
    addLast(e);
    return true;
}
```
另外, 由于 `Deque` 接口继承了 `Queue` 的接口, 所以 `LinkedList` 还实现了 `offer(E e)` 和 `push(E e)` 方法
```Java
/**
 * Adds the specified element as the tail (last element) of this list.
 *
 * @param e the element to add
 * @return {@code true} (as specified by {@link Queue#offer})
 * @since 1.5
 */
public boolean offer(E e) {
    return add(e);
}

/**
 * Pushes an element onto the stack represented by this list.  In other
 * words, inserts the element at the front of this list.
 *
 * <p>This method is equivalent to {@link #addFirst}.
 *
 * @param e the element to push
 * @since 1.6
 */
public void push(E e) {
    addFirst(e);
}
```

#### 添加多个元素
```Java
/**
 * Appends all of the elements in the specified collection to the end of
 * this list, in the order that they are returned by the specified
 * collection's iterator.  The behavior of this operation is undefined if
 * the specified collection is modified while the operation is in
 * progress.  (Note that this will occur if the specified collection is
 * this list, and it's nonempty.)
 *
 * @param c collection containing elements to be added to this list
 * @return {@code true} if this list changed as a result of the call
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}

/**
 * Inserts all of the elements in the specified collection into this
 * list, starting at the specified position.  Shifts the element
 * currently at that position (if any) and any subsequent elements to
 * the right (increases their indices).  The new elements will appear
 * in the list in the order that they are returned by the
 * specified collection's iterator.
 *
 * @param index index at which to insert the first element
 *              from the specified collection
 * @param c collection containing elements to be added to this list
 * @return {@code true} if this list changed as a result of the call
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    if (index == size) {
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }

    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```
- 检查插入点 `index` 的有效性
- 将 `c` 转换为数组, 如果数组长度为 `0` 则返回 `false`
- 如果 `index` 等于 `size`, 那么插入点的后继为 `null`, 前驱为 `last`; 否则后继为 `index` 位置的节点, 前驱为后继的前驱
- 将 `a` 中的值包装为新节点连接到 `pred` 的节点之后
- 处理后继节点
- 修改 `size` 的值, 自增 `modCount` 的值; 返回 `true`

#### 移除单个元素
##### 移除指定位置上的元素
```Java
/**
 * Removes the element at the specified position in this list.  Shifts any
 * subsequent elements to the left (subtracts one from their indices).
 * Returns the element that was removed from the list.
 *
 * @param index the index of the element to be removed
 * @return the element previously at the specified position
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```
- 方法检查 `index` 的有效性
- 调用 `node(int index)` 获取节点, 再调用 `unlink(Node<E> x)` 移除该节点
###### unlink(Node<E> x)
```Java
/**
 * Unlinks non-null node x.
 */
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```
- 获取节点 `x` 的 `element`, 前驱 `prev` 和后继 `next`,
- 如果 `prev` 等于 `null` 则直接将 `next` 赋值给 `first`, 否则将 `x` 的后继作为和 `x` 的前驱的后继, 并将 `x.pred` 置为 `null`
- 如果 `next` 等于 `null` 则直接当 `prev` 赋值给 `last`, 否则将 `x` 的前驱作为 `x` 后继的前驱, 并将 `x.next` 置为 `null`
- 将 `x.item` 置为 null
- `size` 自减, `modCount` 自增, 返回移除节点的值

##### 移除指定值的元素
```Java
/**
 * Removes the first occurrence of the specified element from this list,
 * if it is present.  If this list does not contain the element, it is
 * unchanged.  More formally, removes the element with the lowest index
 * {@code i} such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
 * (if such an element exists).  Returns {@code true} if this list
 * contained the specified element (or equivalently, if this list
 * changed as a result of the call).
 *
 * @param o element to be removed from this list, if present
 * @return {@code true} if this list contained the specified element
 */
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```
- 在列表中饭寻找首个等于 `o` 的节点
- 调用 `unlink(Node<E> x)` 方法移除该节点

##### 移除首次出现的节点和最后一次出现的节点
因为 `LinkedList` 实现了 `Deque` 接口, 所以它实现了 `removeFirstOccurrence(Object o)` 和 `removeLastOccurrence(Object e)`
```Java
/**
 * Removes the first occurrence of the specified element in this
 * list (when traversing the list from head to tail).  If the list
 * does not contain the element, it is unchanged.
 *
 * @param o element to be removed from this list, if present
 * @return {@code true} if the list contained the specified element
 * @since 1.6
 */
public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}

/**
 * Removes the last occurrence of the specified element in this
 * list (when traversing the list from head to tail).  If the list
 * does not contain the element, it is unchanged.
 *
 * @param o element to be removed from this list, if present
 * @return {@code true} if the list contained the specified element
 * @since 1.6
 */
public boolean removeLastOccurrence(Object o) {
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

##### 移除头结点和尾节点
因为 `LinkedList` 实现了 `Deque` 接口, 所以它实现了 `remove()`,  `removeFirst()`, `removeLast()` 方法
```Java
/**
 * Retrieves and removes the head (first element) of this list.
 *
 * @return the head of this list
 * @throws NoSuchElementException if this list is empty
 * @since 1.5
 */
public E remove() {
    return removeFirst();
}

/**
 * Removes and returns the first element from this list.
 *
 * @return the first element from this list
 * @throws NoSuchElementException if this list is empty
 */
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
```
此方法较为简单, 当 `first` 不为 `null` 时, 调用 `unlinkFirst(Node<E> f)` 移除头节点
```Java
/**
 * Unlinks non-null first node f.
 */
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    final E element = f.item;
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```
- 将 `first` 节点赋值为 `first` 的 `next` 节点
- 如果 `next` 为 `null`, 那么将 `last` 也赋值为 `null`, 否则将 `next` 的前驱置为 `null`
- `size` 自减, `modCount` 自增, 返回被移除的元素

```Java
/**
 * Removes and returns the last element from this list.
 *
 * @return the last element from this list
 * @throws NoSuchElementException if this list is empty
 */
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}

/**
 * Unlinks non-null last node l.
 */
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}
```
除此之外还有 `poll(), pop(), pollFirst(), pollLast()` 等方法
```Java
/**
 * Retrieves and removes the head (first element) of this list.
 *
 * @return the head of this list, or {@code null} if this list is empty
 * @since 1.5
 */
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

/**
 * Pops an element from the stack represented by this list.  In other
 * words, removes and returns the first element of this list.
 *
 * <p>This method is equivalent to {@link #removeFirst()}.
 *
 * @return the element at the front of this list (which is the top
 *         of the stack represented by this list)
 * @throws NoSuchElementException if this list is empty
 * @since 1.6
 */
public E pop() {
    return removeFirst();
}

/**
 * Retrieves and removes the first element of this list,
 * or returns {@code null} if this list is empty.
 *
 * @return the first element of this list, or {@code null} if
 *     this list is empty
 * @since 1.6
 */
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}

/**
 * Retrieves and removes the last element of this list,
 * or returns {@code null} if this list is empty.
 *
 * @return the last element of this list, or {@code null} if
 *     this list is empty
 * @since 1.6
 */
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
```

#### 移除多个元素
```Java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over this collection, checking each
 * element returned by the iterator in turn to see if it's contained
 * in the specified collection.  If it's so contained, it's removed from
 * this collection with the iterator's <tt>remove</tt> method.
 *
 * <p>Note that this implementation will throw an
 * <tt>UnsupportedOperationException</tt> if the iterator returned by the
 * <tt>iterator</tt> method does not implement the <tt>remove</tt> method
 * and this collection contains one or more elements in common with the
 * specified collection.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 *
 * @see #remove(Object)
 * @see #contains(Object)
 */
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    boolean modified = false;
    Iterator<?> it = iterator();
    while (it.hasNext()) {
        if (c.contains(it.next())) {
            it.remove();
            modified = true;
        }
    }
    return modified;
}
```
该方法是通过父类 `AbstractCollection` 来实现的, 通过迭代器来遍历 `LinkedList`, 然后判断 `c` 中如果包含则进行移除
```Java
/**
 * {@inheritDoc}
 *
 * <p>This implementation iterates over this collection, checking each
 * element returned by the iterator in turn to see if it's contained
 * in the specified collection.  If it's not so contained, it's removed
 * from this collection with the iterator's <tt>remove</tt> method.
 *
 * <p>Note that this implementation will throw an
 * <tt>UnsupportedOperationException</tt> if the iterator returned by the
 * <tt>iterator</tt> method does not implement the <tt>remove</tt> method
 * and this collection contains one or more elements not present in the
 * specified collection.
 *
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 *
 * @see #remove(Object)
 * @see #contains(Object)
 */
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    boolean modified = false;
    Iterator<E> it = iterator();
    while (it.hasNext()) {
        if (!c.contains(it.next())) {
            it.remove();
            modified = true;
        }
    }
    return modified;
}
```
此方法正好和 `removeAll(Collection<?> c)` 方法相反, 返回 `LinkedList` 和多个元素的交集

##### 查找单个元素
```Java
/**
 * Returns the index of the first occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the lowest index {@code i} such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * or -1 if there is no such index.
 *
 * @param o element to search for
 * @return the index of the first occurrence of the specified element in
 *         this list, or -1 if this list does not contain the element
 */
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```
遍历查找指定元素的下标, `contains(Object o)` 就是基于 `indexOf(Object o)` 实现的, `lastIndexOf(Object o)` 与 `indexOf(Obejct o)` 逻辑类似

#### 获得指定位置的元素
```Java
/**
 * Returns the element at the specified position in this list.
 *
 * @param index index of the element to return
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```
随机访问 `index` 位置的元素, 时间复杂度为 `O(n)`

#### 设置指定位置的元素
```Java
/**
 * Replaces the element at the specified position in this list with the
 * specified element.
 *
 * @param index index of the element to replace
 * @param element element to be stored at the specified position
 * @return the element previously at the specified position
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```

#### 转换为数组
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
 * @return an array containing all of the elements in this list
 *         in proper sequence
 */
public Object[] toArray() {
    Object[] result = new Object[size];
    int i = 0;
    for (Node<E> x = first; x != null; x = x.next)
        result[i++] = x.item;
    return result;
}

/**
 * Returns an array containing all of the elements in this list in
 * proper sequence (from first to last element); the runtime type of
 * the returned array is that of the specified array.  If the list fits
 * in the specified array, it is returned therein.  Otherwise, a new
 * array is allocated with the runtime type of the specified array and
 * the size of this list.
 *
 * <p>If the list fits in the specified array with room to spare (i.e.,
 * the array has more elements than the list), the element in the array
 * immediately following the end of the list is set to {@code null}.
 * (This is useful in determining the length of the list <i>only</i> if
 * the caller knows that the list does not contain any null elements.)
 *
 * <p>Like the {@link #toArray()} method, this method acts as bridge between
 * array-based and collection-based APIs.  Further, this method allows
 * precise control over the runtime type of the output array, and may,
 * under certain circumstances, be used to save allocation costs.
 *
 * <p>Suppose {@code x} is a list known to contain only strings.
 * The following code can be used to dump the list into a newly
 * allocated array of {@code String}:
 *
 * <pre>
 *     String[] y = x.toArray(new String[0]);</pre>
 *
 * Note that {@code toArray(new Object[0])} is identical in function to
 * {@code toArray()}.
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
        a = (T[])java.lang.reflect.Array.newInstance(
                            a.getClass().getComponentType(), size);
    int i = 0;
    Object[] result = a;
    for (Node<E> x = first; x != null; x = x.next)
        result[i++] = x.item;

    if (a.length > size)
        a[size] = null;

    return a;
}
```
这里可以看到 `LinkedList` 转换为数组的逻辑与 `ArrayList` 类似

#### 求哈希值 (AbstractList)
与 `ArrayList` 类似, 都是使用父类的方法

#### 判断相等 (AbstractList)
与 `ArrayList` 类似, 都是使用父类的方法

#### 清空列表
```Java
/**
 * Removes all of the elements from this list.
 * The list will be empty after this call returns.
 */
public void clear() {
    // Clearing all of the links between nodes is "unnecessary", but:
    // - helps a generational GC if the discarded nodes inhabit
    //   more than one generation
    // - is sure to free memory even if there is a reachable Iterator
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}
```
- 遍历链表将节点置 `null`
- 将 `first` 和 `last` 节点置 `null`
- 将 `size` 置为 0, 自增 `modCount`

#### 序列化链表
与 `ArrayList` 实现类似

#### 反序列化链表
与 `ArrayList` 实现类似

#### 克隆
与 `ArrayList` 实现类似

#### 创建子数组 (AbstractList)
```Java
/**
 * {@inheritDoc}
 *
 * <p>This implementation returns a list that subclasses
 * {@code AbstractList}.  The subclass stores, in private fields, the
 * offset of the subList within the backing list, the size of the subList
 * (which can change over its lifetime), and the expected
 * {@code modCount} value of the backing list.  There are two variants
 * of the subclass, one of which implements {@code RandomAccess}.
 * If this list implements {@code RandomAccess} the returned list will
 * be an instance of the subclass that implements {@code RandomAccess}.
 *
 * <p>The subclass's {@code set(int, E)}, {@code get(int)},
 * {@code add(int, E)}, {@code remove(int)}, {@code addAll(int,
 * Collection)} and {@code removeRange(int, int)} methods all
 * delegate to the corresponding methods on the backing abstract list,
 * after bounds-checking the index and adjusting for the offset.  The
 * {@code addAll(Collection c)} method merely returns {@code addAll(size,
 * c)}.
 *
 * <p>The {@code listIterator(int)} method returns a "wrapper object"
 * over a list iterator on the backing list, which is created with the
 * corresponding method on the backing list.  The {@code iterator} method
 * merely returns {@code listIterator()}, and the {@code size} method
 * merely returns the subclass's {@code size} field.
 *
 * <p>All methods first check to see if the actual {@code modCount} of
 * the backing list is equal to its expected value, and throw a
 * {@code ConcurrentModificationException} if it is not.
 *
 * @throws IndexOutOfBoundsException if an endpoint index value is out of range
 *         {@code (fromIndex < 0 || toIndex > size)}
 * @throws IllegalArgumentException if the endpoint indices are out of order
 *         {@code (fromIndex > toIndex)}
 */
public List<E> subList(int fromIndex, int toIndex) {
    return (this instanceof RandomAccess ?
            new RandomAccessSubList<>(this, fromIndex, toIndex) :
            new SubList<>(this, fromIndex, toIndex));
}
```
该方法是通过 `AbstractList` 来实现的, 根据是否实现了 `RandomAccess` 接口, 返回 `RandomAccessSubList` 类对象或 `SubList` 类对象

#### 创建 Iterator 迭代器 (AbstractSequentialList)
```Java
/**
 * Returns an iterator over the elements in this list (in proper
 * sequence).<p>
 *
 * This implementation merely returns a list iterator over the list.
 *
 * @return an iterator over the elements in this list (in proper sequence)
 */
public Iterator<E> iterator() {
    return listIterator();
}
```
该方法是通过父类的 `AbstractSequentialList` 来实现的, 整个的调用过程是 `AbstractSequentialList#iterator()` => `AbstractList#listIterator()` => `AbstractList#listIterator(int index)`, `LinkedList` 重写了 `AbstractList#listIterator(int index)` 方法

#### 创建 ListIterator 迭代器
```Java
/**
 * Returns a list-iterator of the elements in this list (in proper
 * sequence), starting at the specified position in the list.
 * Obeys the general contract of {@code List.listIterator(int)}.<p>
 *
 * The list-iterator is <i>fail-fast</i>: if the list is structurally
 * modified at any time after the Iterator is created, in any way except
 * through the list-iterator's own {@code remove} or {@code add}
 * methods, the list-iterator will throw a
 * {@code ConcurrentModificationException}.  Thus, in the face of
 * concurrent modification, the iterator fails quickly and cleanly, rather
 * than risking arbitrary, non-deterministic behavior at an undetermined
 * time in the future.
 *
 * @param index index of the first element to be returned from the
 *              list-iterator (by a call to {@code next})
 * @return a ListIterator of the elements in this list (in proper
 *         sequence), starting at the specified position in the list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @see List#listIterator(int)
 */
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}
```
##### 创建 ListItr 迭代器
`ListItr` 实现代码较为简单, 这里不再赘述

#### 小结
- `LinkedList` 基于节点实现的双向链表的 `List`, 每个节点都指向前一个和后一个节点从而形成链表
- `LinkedList` 可以提供栈, 队列, 双端队列的功能
- `LinkedList` 随机访问平均时间复杂度为 O(n), 查找指定元素的平均时间复杂度是 O(n)
- `LinkedList` 移除指定位置的元素的最好时间复杂度是 O(1), 最坏时间复杂度是 O(1), 平均时间复杂度是 O(n)
