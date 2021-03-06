### MySQL 架构与历史

#### MySQL 逻辑架构
![MySQL服务器逻辑架构图.png](http://ww1.sinaimg.cn/large/d8f31fa4gy1g7x5yi65i6j20d30bvgln.jpg)  
最上层的服务并不是 MySQL 所独有的, 大多数基于网络的客户端/服务器的工具或者服务都有类似的架构; 比如连接处理, 授权认证, 安全等等  
第二层架构是 MySQL 的核心服务功能, 包括查询解析, 分析, 优化, 缓存以及所有的内置函数 (例如, 日期, 时间, 数学函数等), 所有跨存储引擎的功能都在这一层实现: 存储过程, 触发器, 视图等  
第三层包含了存储引擎, 存储引擎负责 MySQL 中的数据的存储和提取, 服务器通过 API 与存储引擎进行通信

##### 连接管理的安全性
TODO

##### 优化与执行
MySQL 会解析查询, 并创建内部数据结构, 然后对其进行各种优化, 包括重写查询, 决定表的读取顺序, 以及选择合适的索引等; 用户可以通过特殊关键字提示优化器, 影响它的决策过程

TODO 优化解释, 存储引擎的影响, 缓存

#### 并发控制
无论何时, 只要有多个查询需要在同一时刻修改数据, 都会产生并发控制问题; MySQL 在两个层面控制并发: 服务器层和存储引擎层

##### 读写锁
并发问题可以通过实现一个由两种类型的锁组成的所系统来解决; 这两种类型的锁通常被称为共享锁 (shared lock) 和排他锁 (exclusive lock), 也叫读锁 (read lock) 和写锁 (write lock)  
读锁是共享的, 或者说是相互不阻塞的, 多个客户端在同一时刻可以读取同一个资源, 而互不干扰; 写锁是排他的, 也就是说一个写锁会阻塞其他的写锁和读锁, 这是出于安全的考虑, 只有这样才能确保在给定的时间里, 只有一个用户能执行写入, 并防止其他用户读取正在写入的同一资源

##### 锁粒度
一种提高共享资源并发性的方式就是让锁定对象更有选择性, 尽量只锁定需要修改的部分数据, 而不是所有的资源; 给理想的方式是, 只会对修改的数据片进行精确的锁定, 任何时候在给定的资源上, 锁定的数据量越少, 则系统的并发度越高, 只要相互之间不发生冲突即可  
问题是加锁也需要消耗资源, 锁的各种操作, 包括获得锁, 检查锁是否已经解除, 释放锁等, 都会增加系统的开销; 如果系统花费大量的时间来管理锁, 而不是存取数据, 那么系统的性能可能会因此受到影响  
所谓的锁策略, 就是在锁的开销和数据的安全性之间寻求平衡, 这种平衡当然也会影响到性能, 大多数商业数据库没有提供更多的选择, 一般都是在表上施加行级锁 (row-level lock), 并以各种复杂的方式来实现, 以便在锁比较多的情况下尽可能提供更好的性能  
MySQL 提供了多种选择, 每种 MySQL 存储引擎都可以实现自己的锁策略和锁粒度

###### 表锁 (table lock)
表锁是 MySQL 中最基本的锁策略, 并且是开销最小的策略, 表锁会锁定整张表  
在特定的场景中, 表锁也可能有良好的性能; 另外写锁也比读锁有更高的优先级, 因此一个写锁请求可能被插入到读锁队列的前面  
尽管存储引擎可以管理自己的锁, MySQL 本身还是会使用各种有效的表锁来实现不同的目的; 例如, 服务器会为诸如 ALTER TABLE 之类的语句使用表锁, 而忽略存储引擎的锁机制

###### 行级锁 (row lock)
行级锁可以最大程度支持并发处理 (同时也带来了最大的锁开销); 在 InnoDB 和 XtraDB, 以及其他一些存储引擎中实现了行级锁, 行级锁只在存储引擎层实现, 而 MySQL 服务器层没有实现, 服务器层完全不了解引擎中的锁实现

#### 事务
事务就是一组原子性的 SQL 查询, 或者说一个独立的工作单元; 也就是说, 事务内的语句, 要么全部执行成功, 要么全部执行失败  
ACID 表示原子性 (atomicity), 一致性(consistency), 隔离性(isolation), 持久性(durability), 一个运行良好的事务处理系统, 必须具备这些标准特征  
- 原子性 (atomicity)
一个事务必须被视为一个不可分割的最小工作单元, 整个事务中的所有操作要么全部提交成功, 要么全部失败回滚, 对于一个事务来说, 不可能只执行其中的一部分操作, 这就是事务的原子性
- 一致性 (consistency)
数据库总是从一个一致性的状态转换到另一个一致性的状态
- 隔离性 (isolation)
通常来说, 一个事务所做的修改在最终提交前, 对其他事务是不可见的
- 持久性 (durability)
一旦事务提交, 则其所做的修改就会永久保存到数据库中, 即使此时系统崩溃, 修改的数据也不会丢失

就像锁粒度的升级会增加系统开销一样, 这种事务处理过程中额外的安全性, 也会需要数据库系统做更多的额外工作; 一个实现了 ACID 的数据库, 相比没有实现 ACID 的数据库, 通常会需要更强的 CPU 处理能力, 更大的内存和更多的磁盘空间; 所以用户可以根据业务是否需要事务处理, 来选择合适的存储引擎, 对于一些不需要事务的查询类应用, 也可以通过 LOCK TABLE 语句为应用提供一定程度的保护

##### 隔离级别
SQL 标准中定义了四种隔离级别, 每一种级别都规定了一个事务中所做的修改, 哪些在事务内和事务间是可见的, 哪些是不可见的; 较低级别的隔离通常可以执行更高的并发, 系统的开销也更低  
- READ UNCOMMITTED (未提交读)
在 READ UNCOMMITTED 级别, 事务中的修改即使没有提交, 对其他事物也都是可见的; 事务可以读取未提交的数据, 这也被称为脏读 (Drity Read)l; 这个级别会导致很多问题, 从性能上来说, READ UNCOMMITTED 不会比其他的级别好太多, 但缺乏其他级别的很多好处; 除非真的有必要, 在实际应用中很少使用
- READ COMMITTED (提交读)
大多数数据库系统默认隔离级别都是 READ COMMITTED (但 MySQL 不是); READ COMMITTED 满足前面提到的隔离性的简单定义: 一个事务开始时, 只能看见已经提交的事务所做的修改, 即一个事务从开始直到提交之前, 所做的任何修改对其他事务都是不可见的, 换句话说, 一个事务从开始直到提交前, 所做的任何修改对其他事务都是不可见的; 这个级别有时候也叫做不可重复读 (norepeatable read), 因为两次执行同样的查询, 可能会得到不一样的结果
- REPEATABLE READ (可重复读)
REPEATABLE READ 解决了脏读的问题, 该级别保证了在同一个事务中多次读取同样记录的结果是一致的; 但是在理论上, 可重复读级别还是无法解决另外一个幻读 (Phantom Read) 的问题; 所谓幻读, 指的是当某个事务在读取某个范围内的记录时, 另外一个事务又在该范围内插入了新的记录, 当之前的事务再次读取该范围的记录时, 会产生幻行 (Phantom Row); InnoDB 和 XtraDB 存储引擎通过多版本并发控制 (MVCC, Multiversion Concurrency Control) 解决了幻读的问题  
可重复读是 MySQL 的默认事务隔离级别
- SERIALIZABLE (可串行化)
SERIALIZABLE 是最高的隔离级别, 它通过强制事务串行执行, 避免了之前说的幻读问题; 简单来说 SERIALIZABLE 会在读取的每一行数据上都加锁, 所以可能导致大量的超时和锁争用问题, 实际应用中也很少用到这个隔离级别, 只有在非常需要确保一致性而且可以接受没有并发的情况下, 才考虑采用该级别

| 隔离级别 | 脏读可能性 | 不可重复读可能性 | 幻读可能性 | 加锁读 |
| :--- | :--- | :--- | :--- | :--- |
| READ UNCOMMITTED | Yes | Yes | Yes | No |
| READ COMMITTED | No | Yes | Yes | No |
| REPEATABLE READ | No | No | Yes | No |
| SERIALIZABLE | No | No | No | Yes |

##### 死锁
死锁是指两个或多个事务在同一资源上相互占用, 并请求锁定对方占用资源, 从而导致恶性循环的现象; 当多个事务试图以不同的顺序锁定资源时就可能会产生死锁, 多个事务同时锁定同一个资源时也会产生死锁  
为了解决这个问题, 数据库系统实现了各种死锁检测和超时机制; InnoDB 存储引擎目前处理死锁的方法是将持有最少行级排它锁的事务进行回滚; 还有一种解决方法是当查询的时间达到锁等待超时的设定后放弃锁请求, 但这种方法相对来说不太好

##### 事务日志
事务日志可以帮助提高事务的效率, 使用事务日志, 存储引擎在修改表的数据时只需要修改器内存拷贝, 再把该修改行为记录到持久在硬盘上的事务日志中, 而不用每次都将修改的数据本身持久到硬盘; 事务日志采用的是追加的方式, 因此写日志的操作是硬盘上一小块区域内的顺序 I/O, 而不想随机 I/O 需要在硬盘的多个地方移动磁头, 所以采用事务日志的方式相对来说要快得多; 事务日志持久以后, 内存中被修改的数据在后台可以慢慢的刷回磁盘中, 目前大多数存储引擎都是这样实现的, 通常称之为预写式日志 (Write-Ahead Logging), 修改数据需要写两次硬盘  
如果数据的修改已经记录到事务日志并持久化, 但数据本身还没有写回磁盘, 此时系统崩溃, 存储引擎在重启时能够自动恢复这部分修改的数据

##### MySQL 中的事务
MySQL 提供了两种事务型的存储引擎: InnoDB 和 NDB Cluster; 另外还有一些第三方存储引擎也支持事务, 如 XtraDB 和 PBXT

###### 自动提交 (AUTOCOMMIT)
MySQL 默认采用自动提交模式; 也就是说, 如果不是显式的开始一个事务, 则每个查询都会被当作一个事务执行提交操作; MySQL 可以通过语句来修改 AUTOCOMMIT, 但要注意的是修改 AUTOCOMMIT 对非事务型的表, 如 MyISAM 或者内存表, 不会有任何影响; 另外还有一些如 DDL 的命令即使修改了 AUTOCOMMIT, 也会强制提交当前的活动事务  
MySQL 也支持使用语句修改隔离级别, MySQL 能够识别所有的 4 个 ANSI 隔离级别, InnoDB 引擎也支持所有的隔离级别

###### 在事务中混合使用存储引擎
MySQL 服务器层是不管理事务的, 事务是由下层的存储引擎实现的, 所以在同一个事务中, 使用多种存储引擎是不可靠的; 因为回滚一个事务时, 非事务型表的操作无法撤销

###### 隐式和显式锁定
InnoDB 采用的是两阶段锁定协议 (two-phase locking protocol); 在事务执行过程中, 随时都可以被执行锁定, 锁只有在执行 COMMIT 或者 ROOLBACK 的时候才会释放, 并且所有的锁是在同一时刻被释放; 前面叙述的锁都是隐式锁定的, InnoDB 会根据隔离级别在需要的时候自动加锁  
MySQL 也支持 LOCK 和 UNLOCK 语法, 这主要在服务层实现的, 和存储引擎无关; 无论在使用何种存储引擎的情况下, 都不建议使用这两个语法

#### 多版本并发控制
MySQL 的大多数事务型存储引擎实现的都不是简单的行级锁, 基于提升并发性能的考虑, 它们一般都实现了多版本并发控制 (MVCC); 可以任务 MVCC 是行级锁的一个变种, 但是它在很多情况下避免了加锁操作, 因此开销更低, 虽然实现机制有所不同, 但大都实现了非阻塞的读操作, 写操作也只锁定必要的行  
MVCC 的实现, 是通过板寸数据在某个时间点的快照来实现的; 也就是说, 不管需要执行多长时间, 每个事务看到的数据都是一致的; 根据事务开始的时间不同, 每个事务对同一张表, 同一时刻看到的数据可能是不一样的  
不同存储引擎的 MVCC 实现是不同的, 典型的有乐观并发控制, 悲观并发控制; 以下是 InnoDB 的 MVCC 简化版叙述  
InnoDB 的 MVCC 是通过在每行记录后面保存两个隐藏的列来实现的, 这两个列一个保存了行的创建时间, 一个保存了行的过期时间 (或删除时间), 当然这存储的不是实际的时间戳, 而是系统版本号 (system version number); 每开始一个新的事务, 系统版本号都会自动递增, 事务开始时刻的系统版本号会作为事务的版本号, 用来和查询到的每行记录的版本号进行比较; 以下是在 REPEATABLE READ 隔离级别下, MVCC 具体的操作
- SELECT
InnoDB 会根据以下两个条件检查每行记录
  - InnoDB 值查找版本早于当前事务版本的数据行 (也就是, 行的系统版本号小于或等于事务的系统版本号), 这样可以确保事务读取的行, 要么是在事务开始前已经存在, 要么是事务自身插入或修改的
  - 行的删除版本要么是未定义的, 要么大于当前事务版本号; 这样可以确保事务读取到的行, 在事务开始之前未被删除
- INSERT
InnoDB 为新插入的每一行保存当前系统版本号作为行版本号
- DELETE
InnoDB 为删除的每一行保存当前系统版本号作为行删除标示
- UPDATE
InnoDB 为插入一行新记录, 保存当前系统版本号作为行版本号, 同时保存当前系统版本号到原来的行作为行删除标识

保存这两个额外的系统版本号, 使得大多数读不需要加锁, 性能提升; 但是也需要更多的存储空间和维护工作  
MVCC 只在 REPEATABLE READ 和 READ COMMITTED 两个隔离级别下工作, 其他两个隔离级别都和 MVCC 不兼容; 因为 READ UNCOMMITTED 总是读取最新的数据行不符合当前事务版本的数据行, 而 SERIALIZABLE 则会对所有读取的行都加锁

#### MySQL 的存储引擎
TODO

##### InnoDB 存储引擎
InnoDB 是 MySQL 的默认事务型引擎, 也是最重要的, 使用最广泛的存储引擎, 它被设计用来处理大量的短期事务
###### InnoDB 的历史
TODO
###### InnoDB 概率
InnoDB 的数据存储在表空间中, 表空间是由 InnoDB 管理的一个黑盒子, 由一系列的数据文件组成  
InnoDB 采用 MVCC 来支持高并发, 并且实现了四个标准的隔离级别, 并默认级别是可重复读, 并通过间隙锁 (next-key locking) 策略防止幻读的出现, 间隙锁使得 InnoDB 不仅仅锁定查询涉及的行, 还会对索引中的间隙进行锁定, 以防止幻影行的插入  
InnoDB 表是基于聚簇索引建立的, 与其他存储引擎不同, 聚簇索引对主键查询有很高的性能

##### MyISAM 存储引擎
MySQL 5.1 及之前的版本, MyISAM 是默认的存储引擎; 但 MyISAM 引擎不支持事务和行级锁, 并且有一个毫无疑问的缺陷就是崩溃后无法安全恢复  
TODO

##### MySQL 内建的其他存储引擎
TODO

##### 第三方存储引擎
TODO

##### 选择合适的引擎
除非需要用到某些 InnoDB 不具备的特性, 并且没有其他办法可以替代, 否则都应该优先选择 InnoDB 引擎  
TODO

##### 转换表的引擎
有很多中方法可以将表的存储引擎转换成另外一种; 这里讲述三种方法
- ALTER TABLE
- 导出与导入
- 创建与查询

#### MySQL 时间线
TODO

#### MySQL 的开发模式
TODO

#### 总结
TODO
