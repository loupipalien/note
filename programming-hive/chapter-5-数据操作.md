### 数据操作

#### 向管理表中转载数据
Hive 没有行级别的数据插入, 数据更新和删除操作, 那么往表中装载数据的唯一途径就是使用一种 "大量" 的数据装载操作, 或者通过其他方式仅仅将文件写入到正确的目录下
```
load data local inpath '${env:HOME}/california-employees'
overwrite into table employees
partition (country = 'US', state = 'CA');
```
通常情况下指定的路径应该是一个目录, 而不是一个单独的文件, Hive 会将所有文件都拷贝到这个目录中, 文件名会保持不变  
...
Hive 不会验证用户装载的数据和表的模式是否匹配, 但是 Hive 会验证文件格式是否和表结构定义的一致; 例如, 如果表在创建时定义的存储格式是 sequencefile, 那么装载进去的文件也应该是 sequencefile 格式才行

#### 通过查询语句向表中插入数据
insert 语句允许用户通过查询语句向目标表中插入数据
```
insert overwrite table employees partition (country = 'US', state = 'OR')
select * from staged_employees se where se.cnty = 'US' and se.st = 'OR'
```

##### 动态插入
如果需要创建非常多的分区, Hive 提供了动态分区的功能, 可以基于查询参数推断出需要创建的分区名称
```
insert overwrite table employees partition (country, state)
select ... , se.cnty, se.st from staged_employees se
```
以上 select 语句的最后 2 列确定分区字段 country 和 state 的值, 源表的字段值和输出分区值之间的关系是根据位置而不是根据命名匹配的
...
用户也可以混合使用动态和静态分区, 如以下这个例子中制定了 country 字段的值为静态的 US, 而分区字段 state 是动态值
```
insert overwrite table employees partition (country = 'US', state)
select ... , se.cnty, se.st from staged_employees se where se.cnty = 'US'
```
静态分区键必须出现在动态分区键之前; 动态分区的功能默认情况下是没有开启的, 开启后默认是以 "严格" 模式执行的, 在这种模式下要求至少一列分区字段是静态的, 这有助于阻止因设计错误导致查询产生大量的分区; 以下是与动态分区相关的属性

|属性名称|缺省值|描述|
|---|---|---|
|hive.exec.dynamic.partition|false|设置成 true, 表示开启动态分区功能|
|hive.exec.dynamic.partition.mode|strict|设置成 nonstrict, 表示允许所有分区都是动态的|
|hive.exec.max.dynamic.partitions.pernode|100|每个 mapper 或 reducer 可以创建的最大动态分区个数; 如果某个 mapper 或 reducer 尝试创建大于这个值的分区的话, 则会抛出一个致命的错误信息|
|hive.exec.max.dynamic.partitions|1009|一个动态分区创建语句可以创建的最大动态分区数, 如果超过这个值则会抛出一个致命的错误信息|
|hive.exec.max.created.files|100000|全局可以创建的最大文件个数, 有一个 Hadoop 计数器会跟踪记录创建了多少个文件, 如果超过这个值则会抛出一个致命的错误信息|

#### 单个查询语句中创建表并加载数据
```
create table ca_employees
as select name, salary, address from employees where employees.state = 'CA';
```
这个功能不能用于外部表

#### 导出数据
TODO
