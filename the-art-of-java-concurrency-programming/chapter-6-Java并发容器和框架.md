### Java 并发容器和框架

#### ConcurrentHashMap 的实现原理与使用

##### 为什么要使用 ConcurrentHashMap
- 线程不安全的 HashMap
在多线程环境下, 使用 HashMap 进行 put 操作会引起死循环, 是因为多线程会导致 HashMap 的 Entry 链表形成环形数据结构, 一旦形成环形数据结构, Entry 的 next 节点永远不为空, 就会产生死循环获取 Entry, 导致 CPU 利用率接近 100%
- 效率低下的 HashTable
HashTable 容器使用 synchronized 来保证线程安全, 但在线程竞争激烈的情况下效率十分低下,
- ConcurrentHashMap 的锁分段技术可有效提升并发访问率
HashTable 容器在竞争激烈的并发环境下表现效率低下的原因是所有操作都要竞争同一把锁; 如果容器里有多把锁, 每一把锁用于锁容器的一部分数据, 多个线程访问不同数据段时就不存在竞争, 从而提高并发访问率, 这就是 ConcurrentHashMap 的锁分段技术

##### ConcurrentHashMap 的结构
ConcurrentHashMap 是由 Segment 数组结构和 HashEntry 数组结构组成; Segment 是一种可重入锁 (ReentrantLock), 在 ConcurrentHashMap 中扮演锁的角色; HashEntry 则用于存储键值对数据, 一个 ConcurrentHashMap 里包含一个 Segment 数组, Segment 的结构和 HashMap 类似, 是一种数组和链表结构, 一个 Segment 里包含一个 HashEntry 数组, 每个 HashEntry 是一个链表结构的元素; 每个 Segment 守护着一个 HashEntry 数组里的元素, 当 HashEntry 数组的元素进行修改时, 必须首先获得对应的 Segment 锁

##### ConcurrentHashMap 的初始化
ConcurrentHashMap 初始化方法是通过 initialCapacity, loadFactor, concurrencyLevel 等几个参数来初始化 segment 数组, 段偏移量 segmentShift, 段掩码 segmentMask 和每个 segment 里的 HashEntry 来实现的

TODO

#### ConcurrentLinkedQueue
