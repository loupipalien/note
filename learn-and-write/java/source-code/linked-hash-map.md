### LinkedHashMap
在一些业务场景下如果希望能够提供有序访问的 `HashMap`, 那么有两种选择
- TreeMap: 按照 key 的顺序
- LinkedHashMap: 按照 key 的插入和访问的顺序

`LinkedHashMap` 在 `HashMap` 的基础之上提供了顺序访问的特性, 而这里的顺序包括两种
- 按照 `key-value` 的插入顺序进行访问
- 按照 `key-value` 的访问顺序进行访问 (基于这个特性可以实现 LRU 算法的缓存)

#### 类图
`LinkedHashMap` 实现的接口和继承的抽象类如下图  
[LinkedHashMap](http://static.iocoder.cn/images/JDK/2019_12_10/01.png)  

- 实现了 `Map` 接口
- 继承了 `HashMap` 类

#### 属性
以下是 `HashMap.Node, LinkedHashMap.Entry, HashMap.TreeNode` 类的继承结构图  
[image](http://static.iocoder.cn/images/JDK/2019_12_07/04.jpg)  
可以看到 `LinkedHashMap` 实现了自定义的节点 `Entry`, 一个支持指向前后节点的 `Node` 子类
```Java
/**
 * HashMap.Node subclass for normal LinkedHashMap entries.
 */
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```
`LinkedHashMap` 是 `LinkedList` + `HashMap` 的组合, 那么必然就会有头尾节点, 所以属性如下
```Java
/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;

/**
 * The iteration ordering method for this linked hash map: <tt>true</tt>
 * for access-order, <tt>false</tt> for insertion-order.
 *
 * @serial
 */
final boolean accessOrder;
```
- head: 头节点
- tail: 尾节点
- accessOrder: 决定了 `LinkedHashMap` 的是否为访问顺序
  - true: 当 `Entry` 节点被访问时, 放置到链表的结尾, 被 `tail` 指向
  - false: 当 `Entry` 节点被插入时, 放置到链表的结尾, 被 `tail` 指向; 如果插入的 `key` 对应的  `Entry` 节点已经存在, 也会被放到结尾

#### 构造方法
`LinkedHashMap` 一共有 5 个构造方法, 其中前四个与 `HashMap` 相同, 只是多了初始化 `accessOrder = false`, 所以默认使用插入顺序进行访问; 另外一个构造方法允许自定义 `accessOrder` 属性
```Java
/**
 * Constructs an empty <tt>LinkedHashMap</tt> instance with the
 * specified initial capacity, load factor and ordering mode.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @param  accessOrder     the ordering mode - <tt>true</tt> for
 *         access-order, <tt>false</tt> for insertion-order
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```  

#### 创建节点
在插入 `key-value` 键值对时, `put(K key, V value)` 在放入不存在的节点时, 会调用 `newNode(int hash, K key, V value, Node<K,V> e)` 方法创建节点
```Java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

// link at the end of list
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
        head = p;
    else {
        p.before = last;
        last.after = p;
    }
}
```
- 创建 `Entry` 节点
- 将其连接在 `tail` 之后, 并重置 `tail`
- 返回新创建的节点

#### 节点回调
`HashMap` 的读取, 添加, 删除分别提供了 `#afterNodeAccess(Node<K,V> e), afterNodeInsertion(boolean evict), afterNodeRemoval(Node<K,V> e)` 回调方法, 这样 `LinkedHashMap` 可以通过它们实现自定义扩展逻辑

##### afterNodeAccess
在 `accessOrder == true` 时, 当 `Entry` 节点被访问时, 放置到链表的结尾, 被 `tail` 指向; 所以其方法体如下
```Java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```
- 将 `e` 从链表中移除, 并保存 `e` 的 `before` 和 `after` 节点
- 处理 `before, after, head, tail` 节点之间的关系
- 将 `e` 放置链表尾部, `modCount` 自增

在 `HashMap` 中的 `get(Object key)` 和 `getOrDefault(Object key, V defaultValue)` 方法都没有调用 `afterNodeAccess(Node<K,V> e)` 方法, 所以 `LinkedHashMap` 中重写了其逻辑
```Java
/**
 * Returns the value to which the specified key is mapped,
 * or {@code null} if this map contains no mapping for the key.
 *
 * <p>More formally, if this map contains a mapping from a key
 * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
 * key.equals(k))}, then this method returns {@code v}; otherwise
 * it returns {@code null}.  (There can be at most one such mapping.)
 *
 * <p>A return value of {@code null} does not <i>necessarily</i>
 * indicate that the map contains no mapping for the key; it's also
 * possible that the map explicitly maps the key to {@code null}.
 * The {@link #containsKey containsKey} operation may be used to
 * distinguish these two cases.
 */
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}

/**
 * {@inheritDoc}
 */
public V getOrDefault(Object key, V defaultValue) {
   Node<K,V> e;
   if ((e = getNode(hash(key), key)) == null)
       return defaultValue;
   if (accessOrder)
       afterNodeAccess(e);
   return e.value;
}
```

##### afterNodeInsertion
先来看如何基于 `LinkedHashMap` 实现 LRU 算法的缓存
```Java
class LRUCache<K, V> extends LinkedHashMap<K, V> {

    private final int CACHE_SIZE;

    /**
     * 传递进来最多能缓存多少数据
     * @param cacheSize 缓存大小
     */
    public LRUCache(int cacheSize) {
        // true 表示让 LinkedHashMap 按照访问顺序来进行排序，最近访问的放在头部，最老访问的放在尾部。
        super((int) Math.ceil(cacheSize / 0.75) + 1, 0.75f, true);
        CACHE_SIZE = cacheSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 当 map 中的数据量大于指定的缓存个数的时候，就自动删除最老的数据。
        return size() > CACHE_SIZE;
    }
}
```
为何这样就可以实现了 LRU 算法的缓存, 来看 `LinkedHashMap` 的
```Java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```
因为在 `HashMap` 中的插入节点的代码都有调用 `afterNodeInsertion(boolean evict)` 方法, 当实现了此方法移除 `head` 指向的节点 (即最老的节点), 即实现了 LRU 算法

##### afterNodeRemoval
在节点被移除时, `LinkedHashMap` 也需要将节点从链表中移除, 所以重写 `afterNodeRemoval(Node<K,V> e)` 方法
```Java
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```
- 将 `e` 从链表中移除, 并保存 `e` 的 `before` 和 `after` 节点后置为 `null`
- 处理 `before, after, head, tail` 节点之间的关系

#### 清空
```Java
/**
 * {@inheritDoc}
 */
public void clear() {
    super.clear();
    head = tail = null;
}
```
除了调用父类的方法, 还需要将 `head` 和 `tail` 置为 `null`

#### 小结
- `LinkedHashMap` 是 `HashMap` 的子类, 增加了顺序访问的特性
  - `accessOrder == fasle` 时, 按照 `key-value` 的插入顺序进行访问
  - `accessOrder == true` 时, 按照 `key-value` 的读取顺序进行访问
- `LinkedHashMap` 的顺序特性是通过内部的双向链表实现的, 所以可以看成 `LinkedList + HashMap` 的组合
- `LinkedHashMap` 是通过重写 `HashMap` 提供的回调方法, 从而实现了对顺序的特性处理
- `LinkedHashMap` 可以方便的实现 LRU 算法的缓存

>**参考:**
- [集合（四）哈希表 LinkedHashMap](http://svip.iocoder.cn/JDK/Collection-LinkedHashMap)
