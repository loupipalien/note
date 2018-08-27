### 数据定义
HQL 是 Hive 查询语言, 和普遍使用的 SQL 方言一样, 未完全遵守 ANSI SQL 标准; HQL 可能和 MySQL 方言最为接近, 但是两者还是存在显著的差异, Hive 不支持行级插入操作, 更新操作和删除操作, 也不支持事务; Hive 增加了 Hadoop 背景下的可以提供高性能的扩展, 以及一些个性化的扩展, 甚至还增加了一些外部程序

#### Hive 中的数据库
...
如果数据库特别多的话, 那么可以使用正则表达式来筛选出需要的数据库名 (但并不是支持所有的正则表达式)
```
hive> show databases like "h.*"
```
...
需要明确的是, hdfs:///user/hive/warehouse/financials.db 和 hdfs://master-server/user/hive/warehouse/financials.db 是等价的, 其中 master-server 是主节点的 DNS 名和可选的端口号; 为了保证完整性, 当用户指定一个相对路径 (例如: some/relative/path) 时, 对于 HDFS 和 Hive, 都会将这个相对路径放到分布式文件系统指定的根目录下 (例如: hdfs:///user/<user-name>), 然而如果用户是在本地模式下执行的话, 那么当前的本地巩固走目录将时 some/relative/path 的父目录
...
`describe database extended <database>` 可以显示出数据库的一些键值对属性信息
...
默认情况下, Hive 是不允许用户删除一个包含有表的数据库的, 用户要么先删除数据库中的表, 然后再删除数据库, 要么在删除命令的最后面加上关键字 CASECADE, 这样可以使 Hive 自行先删除数据库中的表, 如果使用的是 RESTRICT 这个关键字而不是 CASECADE 这个关键字的话, 那么就和默认情况一样

#### 修改数据库
用户可以使用 ALTER DATABASE 命令为某个数据库的 DBPROPERTIES 设置键值对属性来描述这个数据库的属性信息, 数据库的其他元数据信息都是不可更改的, 包括数据库名和数据库所在的目录位置:
```
hive> alter database financials set dbproperties('edited-by' = 'Joe Dba');
```
没有办法可以删除或者 “重置” 数据库属性

#### 创建表
...
Hive 会自动增加两个表属性: 一个是 last_modified_by, 其保存着最后修改这个表的用户的用户名, 另外一个是 last_modified_time, 其保存着最后一次修改的新纪元的时间秒 (但是如果没有定义任何的用户自定义表属性的话, 那么它们也不会显示在表的详细信息中)
...
默认情况下, Hive 总是将创建表的目录放置在这个表所属的数据库目录之后, 不过 default 库是个例外, 其在 /user/hive/warehouse 下并灭有对应一个数据库目录, 因此 default 数据库中的表目录会直接位于 /user/hive/warehouse 目录之后 (除非用户明确指定了其他路径)
...
用户还可以拷贝一张已经存在的表的模式 (无需拷贝数据)
```
hive> create table if not exists mydb.employees2 like mydb.employees;
```
这个版本还可以接受可选的 location 语句, 但是其他的属性, 包括模式都是不可重新定义的, 这些信息直接从原始表获得
...
用户可以使用 describe extended mydb.employees 命令来查看这个表的详细表结构信息, 使用 formatted 代替 extended 关键字的话, 可以提供更多可读的冗长的输出信息; 如果用户只想查看某一个列的信息, 那么只要在表名后增加这个字段的名称即可, 这种情况下 extended 关键字也不会增加更多的输出信息

##### 管理表
管理表又叫做内部表, Hive (或多或少) 会控制着这种表数据的生命周期, Hive 默认情况下会将这些表的数据存储在由配置项 hive.metastore.warehouse.dir 所定义的子目录下, 当我们删除一个管理表时 Hive 也会删除这个表中的数据  
...

##### 外部表
关键字 external 告诉 Hive 这个表是外部的, 后面的 location 子句则是告诉 Hive 数据位于那个路径下; 因为表是外部的, 所以 Hive 并非认为其完全拥有这份数据, 因此删除表并不会删除这份数据, 不过描述表的元数据信息会被删除掉  
...

#### 分区表, 管理表
...
如果表中的数据以及分区个数都非常大的话, 执行这样一个包含所有分区的查询可能会触发一个巨大的 MapReduce 任务; 一个高度建议的安全措施就是将 Hive 设置为 "strict" 模式, 这样如果对分区表进行查询而 where 子句没有加分区过滤的话, 将会禁止提交这个任务  
...

##### 外部分区表
...
创建非分区外部表时要求使用一个 location 子句, 对于外部分区表则没有这样的要求, alter table 语句可以单独进行增加分区
...
Hive 不关心一个分区对应的分区目录是否存在或者分区目录下是否有文件, 如果分区目录不存在或分区目录下没有文件, 则对于这个过滤分区的查询将没有返回结果
...
alter table ... add partition 语句并非只有对外部表才能够使用; 对于管理表, 当有分区数据不是由 load 或者 insert 语句产生时, 用户同样可以使用这个命令指定分区路径; 用户需要记住并非所有的表数据都是放在通常的 Hive 的 "warehouse" 目录下的, 同时当删除管理表时, 这些数据不会连带被删除掉! 因此, 从 "理智" 的角度来看, 是否敢于对管理表使用这个功能是一个问题

##### 自定义表的存储格式
Hive 的默认存储格式是文本文件格式, 这个也可以通过可选的子句 stored as textfile 显示指定
...
用户可以将 textfile 替换为其他 Hive 所支持的内置文件格式, 包括 sequencefile 和 rcfile, 这两种文件格式都是使用二进制编码和压缩 (可选) 来优化磁盘空间使用以及 I/O 带宽性能的
...
用户还可以指定第三方的输入和输出格式以及 serde, 这个功能允许用户自定义 Hive 本身不支持的其他广泛的文件格式
...

#### 删除表
对于管理表, 表的元数据和表内的数据都会被删除; 对于外部表, 表的元数据信息会被删除, 但是表中的数据不会被删除

#### 修改表
大多数的表属性可以通过 alter table 语句来修改, 这种操作会修改元数据但不会修改数据本身; 这些语句可以修改表模式中出现的错误, 改变分区路径以及其他操作

##### 表重命名
```
alter table log_message rename to logmsgs;
```

##### 增加, 修改和删除表分区
TODO

##### 修改列信息
TODO

##### 增加列
TODO

##### 删除或者替换列
以下示例移除了之前所有的字段并重新制定了新的字段
```
alter table log_message replace columns (
hours_mins_secs INT COMMENT 'hour, minute, seconds, from timestamp',
severity STRING COMMENT 'the message serverity',
message STRING COMMENT 'the rest of the message'
);
```
这个语句实际上重命名了之前 hms 字段并且从之前表定义的模式中移除了字段 server 和 process_id, 因为是 alter 语句, 所以只有表的元数据变了  
replace 语句只能用于使用了如下 2 种内置 serde 模块的表: DynamicSerDe 或者 MetadataTypedColumnsetSerDe

##### 修改表属性
TODO

##### 修改存储属性
TODO

##### 众多的修改表语句
- alter table ... touch
```
alter table log_message touch partition(yaer=2012, month=1, day=1);
```
partition 子句用于分区表, 这种语句的一个典型应用场景是, 当表中存储的文件在 Hive 之外被修改了, 就会触发钩子执行
- alter table ... archive partition
```
alter table log_message archive partition(yaer=2012, month=1, day=1);
```
这个语句会将分区内的文件打成一个 Hadoop 压缩包 (HAR) 文件, 但是这样仅仅可以降低文件系统中的文件数以及减轻 namenode 的压力, 而不会减少任何的存储空间, 使用 unarchive 替换 archive 就可以反向操作, 这个功能只能用于分区表中独立的分区
- alter table ... partition ... enable no_drop/offline
```
alter table log_message partition(yaer=2012, month=1, day=1) enable no_drop;
alter table log_message partition(yaer=2012, month=1, day=1) enable offline;
```
这些语句可以防止分区被删除和被查询, 使用 disable 代替 enable 可以达到反向的目的, 这些操作也都不可用于非分区表
