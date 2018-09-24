### 锁
尽管 HiveQL 是一种 SQL 方言, 但是 Hive 缺少通常在 update 和 insert 类型的查询中使用到的对于列, 行或者查询级别的锁支持; Hadoop 中的文件通常是一次写入的 (尽管 Hadoop 支持追加的功能), 因为这个一次写入的天性和 MapReduce 的 streaming 类型, 访问细粒度锁是不必要的  
Hive 是多用户的, 那么在一些情况下, 锁和协调会是非常有用的, 例如多个用户同时查询修改同一张表时; Hive 可以被认为是一个胖客户端, 在某种意义上每个 Hive CLI, Thrift Server 或者 Web 接口实例都不是完全独立于其他实例的; 因为这个独立性, 所有锁必须由单独的系统进行协调

#### Hive 结合 Zookeeper 支持锁功能
Hive 中包含了一个使用 Zookeeper 进行锁定的锁功能, Zookeeper 实现了高度可靠的分布式协调功能, 处理需要增加一些额外的配置 (hive.zookeeper.quirum 和 hive.support.concurrency) 即可, Zookeeper 对于 Hive 用户来说是透明的; 以下是一些查看锁的语句
```
# 查看所有锁
show locks;
# 查看表上的锁
show locks table_name [extended];
# 查看分区上的锁
show locks table_name partition(...) [extended];
```
Hive 中提供了共享和独占两种锁; 当表是分区表时, 对一个分区获取独占锁时会导致需要对表本身获取共享锁来防止发生不相容的变更, 例如当表的一个分区正在被修改时尝试删除这个表, 当然对表使用独占锁会全局影响所有的分区

#### 显示锁和独占锁
用户同样可以显示的管理锁, 如
```
# 锁表
lock table table_name exclusive;
# 解锁表
unlock table table_name;
```
