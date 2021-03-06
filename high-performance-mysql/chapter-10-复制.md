### 复制

#### 复制概述
复制解决的基本问题是让一台服务器的数据与其他服务器保持同步; 一台主库的数据可以同步到多台备库上, 备库本身也可以被配置成另外一台服务器的主库; 主库和备库之间可以有多种不同的组合方式  
MySQL 支持两种复制方式: 基于行的复制和基于语句的复制; 基于语句的复制 (也称之为逻辑复制) 早在 MySQL 3.23 版本中就存在, 而基于行的复制方式在 5.1 版本中才被加进来; 这两种方式都是通过在主库上记录二进制日志, 在备库重放日志的方式来实现异步的数据复制; 这意味着, 在同一时间节点上备库的数据可能与主库存在不一致, 并且无法保证主备之间的延迟

#### 复制解决的问题
- 数据分布
MySQL 复制通常不会对带宽造成很大的压力; 但在 5.1 版本引入的基于行的复制会比基于传统的基于语句的复制模式的带宽压力更大
- 负载均衡
通过 MySQL 复制可以将读操作分不到多个服务器上, 实现对读密集型应用的优化, 并且实现很方便, 通过简单的代码修改就能实现基本的负载均衡
- 备份
对于备份来说, 复制是一项很有意义的技术补充, 但复制既不是备份也不能够取代备份
- 高可用性和故障切换
复制能够帮助应用程序避免 MySQL 单点失败, 一个包含复制的设计良好的故障切换系统能够显著的缩短宕机时间
- MySQL 升级测试
使用一个更高版本的 MySQL 作为备库, 保证升级全部实例前, 查询能够在备库按照预期执行

##### 复制如何工作
- 在主库上把数据更改记录到二进制日志 (Binary Log) 中 (这些记录被称为二进制日志事件)
这一步是在主库上记录二进制日志, 在每次准备提交事务完成数据更新前, 主库将数据更新的事件记录到二进制日志中, MySQL 会按事务提交的顺序而非每条语句的执行顺序来记录二进制日志; 在记录二进制日志后, 主库会告诉存储引擎可以提交事务了
- 备库将主库上的日志复制到自己的中继日志 (Relay Log) 中
备库会启动一个工作线程 (I/O 线程), 此线程跟主库建立一个普通的客户端连接, 然后在主库上启动一个特殊的二进制转储 (binlog dump) 线程, 这个二进制转储线程会读取主库上二进制日志的事件; 该线程如果追赶上了主库, 它将进入睡眠状态, 直到主库发送信号量通知其有新的事件产生时才会被唤醒, 备库 I/O 线程会将接收到的事件记录到中继日中
- 备库读取中继日志中的事件, 将其重放到备库数据之上
备库的 SQL 线程从中继日志中读取事件并在备库上执行, 从而实现备库数据的更新

![MySQL复制.png](http://ww1.sinaimg.cn/large/d8f31fa4gy1g8ehp9tpvvj20i30anjrs.jpg)  
这种复制架构实现了获取事件和重放事件的解耦, 允许这两个过程异步进行; 也就是说 I/O 线程能够独立于 SQL 线程之外工作; 但这种架构也限制了复制的过程, 其中最重要的一点是在主库上并发运行的查询在备库只能串行化执行, 因为只有一个 SQL 线程来重放中继日志的事件

#### 配置复制
TODO

#### 复制的原理

##### 基于语句的复制
在 MySQL 5.0 及之前版本中只支持基于语句的复制 (也称为逻辑复制), 这在数据库于是甚少见的; 基于语句的复制模式下, 主库会记录那些造成数据更改的查询, 当备库读取并重放这些事件时, 实际上只是把主库上执行过的 SQL 再执行一遍, 这种方式既有好处也有缺点  
好处是实现相当简单, 二进制日志里的事件也更加紧凑; 缺点是主库上的数据更新除了执行的语句之外, 可能还依赖了其他因素, 如时间参数等, 还有就是更新必须是串行的, 这需要更多的锁

##### 基于行的复制
MySQL 5.1 开始支持基于行的复制, 这种方式会将实际数据记录在二进制日志中, 跟其他数据库的实现比较相像, 它有其自身的一些优点和缺点, 最大的好处是可以正确的复制每一行, 一些语句可以被更加有效的复制, 而另一些语句会有更大的开销  
由于没有哪种模式是完美的, MySQL 能够在这两种复制模式间动态切换; 默认情况下使用的是基于语句的复制方式, 但是如果发现语句无法被正确地复制, 就切换为基于行的复制模式

##### 基于语句或基于行哪种更优
TODO

##### 复制文件
在复制时会使用到一些文件
- mysql-bin.index
当在服务器上开启二进制日志时, 同时会生成一个和二进制同名但以 `.index` 作为后缀的文件, 该文件用于记录磁盘上的二进制日志文件
- mysql-relay-bin.index
这个文件是中继日志的索引文件, 和 mysql-bin.index 的作用类似
- master.info
这个文件用于保存备库连接到主库所需要的信息, 格式为纯文本 (每行一个值), 不同的 MySQL 版本, 其记录的信息可能不同
- relay-log.info
这个文件包含了当前备库复制的二进制日志和中继日志坐标; 不能删除此文件, 否则备库重启后将无法获知从哪个位置开始复制


##### 发送复制事件到其他备库
log_slave_updates 选项可以让备库变成其他服务器的主库; 在设置该选项后, MySQL 会将其执行过的事件记录到它自己的二进制日志中, 这样它的备库就可以从其日志中检索并执行事件

##### 复制过滤器
复制过滤选项允许仅复制服务器上一部分数据, 有两种复制过滤方式: 在主库上过滤记录到二进制日志中的事件, 以及在备库上过滤记录到中继日志的事件; 使用选项 binlog_do_db 和 binlog_ignore_db 来控制过滤, 但并不推荐使用

#### 复制拓扑
可以在任意个主库和备库之间建立复制, 只有一个限制: 每个备库只能有一个主库; 无论何种拓扑, 请记住以下的基本原则
- 一个 MySQL 备库实例只能有一个主库
- 每个备库必须有一个唯一的服务器 ID
- 一个主库可以有多个备库 (一个备库可以有多个兄弟备库)
- 如果打开了 log_slave_updates 选项, 一个备库可以把主库上的数据变化传播到其他备库

TODO

#### 复制和容量规划
TODO

#### 复制的管理和维护
TODO

#### 复制的问题和解决方案
TODO

#### 复制有多快
TODO

#### MySQL 复制的高级特性
TODO

#### 其他复制技术
TODO

#### 总结
TODO
