### 集合
![Collection](https://www.runoob.com/wp-content/uploads/2014/01/2243690-9cd9c896e0d512ed.gif)

#### Java 集合框架的基础接口有哪些
- Collection: 为集合层级的根接口, 一个集合代表组对象, 这些对象即为它的元素
  - Set: 是一个不能包含重复元素的集合, 这个接口对数学集合抽象进行建模
  - List: 是一个有序集合, 可以包含重复元素, 可以通过索引来访问元素
- Map: 是一个将 key 映射到 value 的对象, 一个 Map 不能包含重复的 key, 每个 key 最多映射一个 value
- 其他接口: Queue, Dequeue, SortedSet, SortedMap, ListIterator

##### 集合框架底层数据结构总结
- List
  - ArrayList: Object 数组
  - Vector: Object 数组
  - LinkedList: 双向链表 (JDK6 之前未循环链表, JDK7 取消了循环)  
- Set
  - HashSet: 无序唯一的元素集合
  - TreeSet: 有序唯一的元素集合
- Map
  - HashMap
    - JDK8 之前, HashMap 由数组 + 链表组成, 数组是 HashMap 的主体, 链表则是主要为了解决哈希冲突而存在的 (链式地址法解决冲突)  
    - JDK8 之后, 在结局哈希冲突时有了较大变化, 当链表长度大于阈值 (默认为 8) 时, 将链表转化为红黑树以减少搜索时间
  - LinkedHashMap: LinkedHashMap 继承自 HashMap, 所以它的底层仍然是基于链式地址法的散列结构, 即数组和链表或红黑树组成; 另外 LinkedHashMap 在此基础之上增加了一条双向链表, 使得可以保持键值对的插入顺序, 详细可见 [LinkedHashMap 源码详细分析 (JDK8)](https://www.imooc.com/article/22931)
  - HashTable: 数组和链表组成, 数组是 HashMap 的主体, 链表则是主要为了解决哈希冲突而存在的
  - TreeMap: 红黑树 (自平衡的排序二叉树)

#### 什么是迭代器
迭代器可以在迭代的过程中删除底层集合的元素, 但是不可以直接调用集合的 `remove(Object obj)` 方法删除, 可以通过迭代器的 `remove()` 方法删除
##### 快速失败 (fial-fast) 和安全失败 (fail-safe) 的区别是什么
差别在于 ConcurrentModification 异常
- 快速失败: 当在迭代一个集合的时候, 如果有另一个线程正在修改你正在访问的那个集合, 就会抛出一个 ConcurrentModification 异常, 在 `java.util` 包下都是快速失败
- 安全失败: 在迭代的时候回去底层集合做一个拷贝, 所以在修改上层集合的时候是不受影响的, 不会抛出 ConcurrentModification 异常, 在 java.util.concurrent 包下都是安全失败的
##### 为何 Iterator 接口没有具体的实现
Iterator 接口定义了遍历集合的方法, 但它的实现则是集合实现类的责任, 每个能够返回用于遍历的 Iterator 的集合都有它自己的 Iterator 实现内部类; 这就运行集合类去选择迭代器是 fail-fast 还是 fail-safe 的, 例如 ArrayList 迭代器是 fail-fast 的, 而 CopyOnWriteArrayList 迭代器是 fail-safe 的
##### Iterator 和 Enumeration 有什么不同
- Enumeration 比 Iterator 相比较更快, 而且占用更少内存
- 但是 Iterator 比 Enumeration 更安全, 因为其他线程不能修改当前迭代器遍历的集合对象, 同时 Iterators 允许调用者从底层集合中移除元素, 这些 Enumeration 是无法完成的; Enumeration 的使用介绍见 [Java Enumeration 接口](http://www.runoob.com/java/java-enumeration-interface.html)

#### Comparable 和 Comparator 的区别
Comparable 接口是能够比较的类需要实现的接口, 在 `java.lang` 包下, 用于当前对象与其他对象的比较, 需要重写 `compareTo(Object obj)` 方法来排序, 该方法只有一个参数  
Comparator 接口在 `java.util` 包下, 用于传入两个对象进行比较, 有 `compare(Object obj1, Object obj2)` 方法用来排序, 该方法有两个参数

#### 关于 Java 集合框架的最佳实现
- 基于应用的需求来选择使用正确类型的集合, 这对性能来说是非常重要的
- 一些集合类允许指定初始容量, 因此知道了存储数据的大概数量,就可以避免重散列或调整大小
- 总是使用泛型来保证类型安全, 可靠性和健壮性, 同时使用泛型还可以避免运行时的 ClassCastException 异常
- 返回零长度的集合或数组, 而不是返回一个 null, 避免 NPE

#### ArrayList 和 LinkedList 区别
##### ArrayList
- 优点: ArrayList 是实现了基于数组的动态数据结构, 因为地址连续一旦数据存储好了, 查询效率会比较高
- 缺点: 因为地址连续, ArrayList 要移动数据, 所以插入和删除操作效率较低
- 适用: 需要对数据进行多次增加删除修改的场景下
##### LinkedList
- 优点: LinkedList 基于链表的数据结构, 地址是任意的, 所以开辟内存空间的时候不需要等一个连续的地址, 对于新增和删除操作性能较好
- 缺点: 因为 LinkedList 要移动指针, 所以查询性能比较低
- 适用: 需要对数据进行随机访问的场景下

#### ArrayList 是如何扩容的
TODO [ArrayList 动态扩容详解](https://www.cnblogs.com/kuoAT/p/6771653.html)

#### HashMap 和 HashTable 的区别
- HashTable 继承 Dictionary, HashMap 实现是 JDK2 出现的 Map 接口
- HashMap 去掉了 HashTable 中的 contains 方法, 但是加上了 containsKey 和 containsValue 方法
- HashMap 允许 null 键, 而 HashTable 不允许
- HashTable 是同步的, 而 HashMap 是非同步的, 效率上比 HashTable 高很多, 因此 HashMap 也只能在单线程中使用, 而 HashTable 适用于多线程
- HashMap 的迭代器是 fail-fast 迭代器, HashTable 的 Enumerator 迭代器不是 fail-fast 的
- HashTable 中数组默认大小是 11, 扩容方法是 `old * 2 + 1`, HashMap 默认大小是 16, 扩容每次为 2 的指数大小

现在不推荐使用 HashTable, 一是 HashTable 是遗留类, 内部实现很多没有优化和冗余, 二是即使在多线程环境下, ConcurrentHashMap 比 HashTable 有更好的性能

#### HashMap 和 ConcurrentHashMap 的区别
ConcurrentHashMap 是线程安全的 HashMap 的实现
- ConcurrentHashMap 对整个数组进行了分段 (Segment), 然后在每个分段上都用 lock 锁进行保护, 相对于 HashTa 的 synchronized 关键字锁的粒度更精细了一些, 并发性能更好, 而 HashMap 没有锁机制, 不是线程安全的; JDK8 之后 ConcurrentHashMap 启用了 CAS 算法实现锁机制
- HashMap 的键值对允许有 null, 但是 ConcurrentHashMap 都不允许

#### 能否使用任何类作为 Map 的 key
原则上可以使用任何类作为 Map 的 key, 但是在使用之前需要考虑以下几点
- 类是否重写了 equals 和 hashCode 方法
- 类的所有实例需要遵循与 equals 和 hashCode 方法的相关规则
- 如果没有一个类没有使用 equals, 则不应该在 hashCode 中使用它
- 用户自定义的 key 的类最佳实践是使之不可变, 这样 hashCode 值可以被缓存起来, 拥有更好的性能, 不可变的类也可以确保 hashCode 和 equals 在未来不会改变, 这样就会解决与可变相关的问题了

#### HashMap 的长度为什么是 2 的幂次方
为了能让 HashMap 存取高效, 尽量减少碰撞, 也就是要尽量吧数据分配均匀, 每个链表伙伴红黑树长度大致相同, 均匀分配可以采用 `%` 取余的方式, 在取余操作中如果除数是 3 的幂次等价于与其除数减一的与 `&`, 也就是说 `hash % length == hash & (length - 1)`, 前提是 length 是 2 的 n 次方; 这样做的原因是相比于 `%` 运算, `&` 运算高效的多

#### EnumSet 是什么
TODO [EnumSet 源码解析](https://blog.csdn.net/u010887744/article/details/50834738)

>**参考:**
- [Java 集合框架](https://www.runoob.com/java/java-collections.html)
