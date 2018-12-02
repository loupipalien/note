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
