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
