### 基础知识
Hive 提供了一个被称为 Hive 的查询语言 (简称为 HiveQL 或 HQL) 的 SQL 方言, 来查询存储在 Hadoop 集群中的数据; Hive 可将大多数的查询转换为 MapReduce 任务 (job), 进而在介绍一个令人熟悉的 SQL 抽象的同时, 拓宽 Hadoop 的可扩展性  
Hive 不是一个完整的数据库, Hadoop 以及 HDFS 的设计本身约束和局限性限制了 Hive 所能胜任的操作, 其中最大的限制就是 Hive 不支持记录级别的更新, 插入或删除操作; 但是用户可以通过查询生成新表或者将查询结果导入到文件中; 同时, 因为 Hadoop 是一个面向批处理的系统, 而 MapReduce 任务 (job) 的启动过程需要消耗较长时间, 所以 Hive 查询掩饰严重; 在传统数据库中在秒级别可以完成的查询, 在 Hive 中, 即使数据集相对较小, 往往也需要执行更长的时间; 最后需要说明的是 Hive 不支持事务, 因此 Hive 不支持 OLTP (联机事务查询) 所需的关键功能, 而更接近成为一个 OLAP (联机分析技术) 工具  
HQL 和大多数 SQL 方言一样并不符合 ANSI SQL 标准, 其和 Oracle, MySQL, SQL server 支持的常规 SQL 方言在很多方面有差异 (不过, HQL 和 MySQL 提供的方言最接近)

#### Hadoop 和 MapReduce 综述
TODO

#### Java 和 Hive: 词频统计算法
TODO

#### 后续事情
我们描述了 Hive 在 Hadoop 生态系统中所扮演的重要角色, 现在我们开始!
