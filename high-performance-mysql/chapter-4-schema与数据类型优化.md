### Schema 与数据类型优化

#### 选择优化的数据类型
MySQL 支持的数据类型非常多, 选择正确的数据类型对于获得高性能至关重要, 以下几个原则有助于做出更好的选择
- 更好的通常更好
一般情况下, 应该尽量使用可以正确存储数据的最小数据类型; 更小的数据类型通常更快, 因为它们占用更少的磁盘, 内存和 CPU 缓存, 并且处理时需要的 CPU 周期也更少
- 简单就好
简单的数据类型的操作通常需要更少的 CPU 周期
- 尽量避免 NULL
很多表通常包含可为 NULL 的列, 通常情况下最好指定列为 NOT NULL, 除非需要存储 NULL 值; 如果查询中包含可为 NULL 的列, 对 MySQL 来说更难优化, 因为可为 NULL 的列使得索引, 索引统计, 值比较都更复杂, 对 MySQL 的列会使用更多的空间, 在 MySQL 里也需要特殊处理

下一步是选择具体类型, 第一步需要确定合适的大类型, 当相同大类型的不同子类型有时也有一些特殊的行为和属性; 例如, DATETIME 和 TIMESTAMP 列都可以存储相同类型的数据: 日期和时间, 精确到秒; 然而 TIMESTAMP 只使用 DATETIME 一半的存储空间, 并且会根据时区变化, 具有特殊的自动更新功能; 另一方面, TIMESTAMP 允许的时间范围要小很多

##### 整数类型
有两种类型的数字: 整数 (whole number) 和实数 (real number); 如果存储整数, 可以使用这几种整数类型: TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT, 分别使用 8, 16, 24, 32, 64 位存储空间; 整数类型可选 UNSIGNED 属性, 表示不允许负值; MySQL 可以为整数类型指定宽度, 例如 INT(11), 对大多数应用这是没有意义的, 它不会限制值的合法范围, 只是规定 MySQL 的一些交互工具用来显示字符的个数, 对于存储和计算来说, INT(1) 和 INT(20) 是相同的

##### 实数类型
实数是带有小数部分的数字, 然而它们不只是为了存储小数部分, 也可以使用 DECIMAL 存储比 BIGINT 还大的整数; MySQL 既支持精确类型 (DECIMAL), 也支持不精确类型 (FALOT, DOUBLE)  
浮点类型在存储同样范围的值时, 通常比 DECIMAL 使用更少的空间; FLOAT 使用 4 个字节, DOUBLE 使用 8 个字节, 相比 FLOAT 有更高的精度和更大的范围; 和整数类型一样, 能选择的只是存储类型, MySQL 使用 DOUBLE 作为内部浮点计算的类型

##### 字符串类型
###### VARCHAR 和 CHAR 类型
VARCHAR 和 CHAR 是两种最主要的字符串类型, 这些值如何存储在内存和磁盘中, 这和存储引擎的具体实现有关; 以下假定使用的是 InnoDB 和 MyISAM 存储引擎
- VARCHAR
VARCHAR 用于存储可变长字符串, 是最常见的字符串数据类型, 它比定长类型更节省空间, 因为它仅使用必要的空间; VARCHAR 需要使用 1 到 2 个额外字节记录字符串的长度
但是由于行是变长的, 在 UPDATE 时可能使行变得比原来更长, 这就导致需要做额外的工作, 如果一个行占用的空间增长, 并且在页内没有更多的空间可以存储, 在这种情况下, 不同存储引擎的处理方式是不一样的; MyISAM 会将行拆成不同的片段存储, InnoDB 则需要分裂页来使行放进页内
所以在字符串列的最大长度比平均长度大很多时, 并且更新很少时, 使用 VARCHAR 类型是合适的
- CHAR
CHAR 类型是定长的, MySQL 总是根据定义的字符串长度分配足够的空间, 多余的空间使用空格填充; CHAR 类型适合存储定长且长更新的值

与 VARCHAR 和 CHAR 类似的类型还有 BINARY 和 VARBINARY, 它们存储的是二进制字符串, 二进制字符串跟常规字符串相似, 但是二进制字符串存储的是字节码而不是字符, 填充的是 `\0` (零字节) 而不是空格

###### BLOB 和 TEXT 类型
BLOB 和 TEXT 都是为存储很大的数据而设计的字符串数据类型, 分别采用二进制何人字符方式存储; 实际上它们分属两组不同的数据类型家族: 字符类型是 TINYTEXT, SMALLTEXT, TEXT, MEDIUMTEXT, LONGTEXT; 对应的二进制类型是 TINYBLOB, SMALLBLOB, BLOB, MEDIUMBLOB, LONGBLOB  
与其他类型不同, MySQL 把每个 BLOB 和 TEXT 值当作一个独立的对象处理, 存储引擎在存储时通常会做特殊处理; 当 BLOB 和 TEXT 值太大时, InnoDB 会使用专门的 "外部" 存储区域来进行存储, 此时每个值在行内需要 1 - 4 个字节存储一个指针, 然后在外部存储区域存储实际的值  
MySQL 对 BLOB 和 TEXT 列进行排序与其他类型不同, 它值对每个列的最前 max_sort_length 字节而不是整个字符串做排序; 如果只需要对前面小部分字符排序, 则可以减小 max_sort_length 的值, 或者使用 ORDER BY SUSTRING(culomn, length)

###### 使用枚举 (ENUM) 代理字符串类型
有时候可以使用枚举列代替常用的字符串类型, 枚举列可以把一些不重复的字符串存储成一个预定义的集合; MySQL 在存储枚举时非常紧凑, 会根据列表值的数据压缩到一个或两个字节中,MySQL 内部会将每个值在列表中的位置保存为整数, 并且在表的 `.frm` 文件中保存 "数字 - 字符串" 映射关系的 "查找表"  
如果使用数字作为 ENUM 枚举常量, 这种双重性很容易导致混乱, 建议必满这样做; 另外一个不同的地方是, 枚举字段是按照内部存储的整数而不是定义的字符串进行排序的, 一种绕过这种限制的方式按照需要来定义枚举列, 或者在查询中使用 FIELD() 函数显示指定排序顺序

##### 日期和时间类型
MySQL 提供两种相似的日期类型: DATETIME 和 TIMESTAMP; 对于很多场景它们都能工作, 但是在某些场景, 一个比另一个工作得好
###### DATETIME
这个类型能保存大范围的值, 从 1001 到 9999 年, 精度为秒; 它把日期和时间封装到格式为 YYYYMMDDHHMMSS 的整数中, 与时区无关, 使用 8 个字节的存储空间
###### TIMESTAMP
这个类型保存了从 1970 年 1 月 1 日午夜以来的秒数, 和 UNIX 时间戳相同, 它值使用 4 个字节的存储空间, 因此它的范围比 DATETIME 小得多: 只能从 1970 年到 2038 年; TIMESTAMP 显示的值依赖于时区, MySQL 服务器, 操作系统, 以及客户端连接都有时区设置  
除特殊情况以外, 一般应该尽量使用 TIMESTAMP, 因为它比 DATETIME 空间效率更高; 有时候有人会将 UNIX 时间存储为时间戳, 但这不会带来任何收益, 用整数保存时间戳的格式通常不方便处理, 所以不推荐这样做

##### 位数据类型
MySQLd 有少数几种存储类型使用紧凑的位存储数据, 所有这些位类型, 不管是底层存储格式和处理方式如何, 从技术上来说都是字符串类型
###### BIT
在 MySQL 5.0 之前, BIT 是 TINYINT 的同义词, 但是在此之后, 这是一个特性完全不同的数据类型  
可以在 BIT 列在一列中存储一个或多个 true/false 值; BIT(1) 定义一个包含单个位的字段, BIT(2) 存储 2 个位, 依次类推; BIT 列的最大长度是 64 个位  
MySQL 把 BIT 当作字符串类型, 而不是数字类型; 当检索 BIT(1) 的值时, 结果是一个包含二进制 0 和 1 的字符串, 而不是 ASCII 码的 "0" 或 "1"; 然而在数字上下文的场景中检索时, 结果将是为字符串转换成的数字; 例如, 如果存储一个值 `b'00111001'` (二进制等于 57) 到 BIT(8) 的列表并检索它, 得到的内容发是字符码为 57 的字符串; 也就是说得到 ASCII 码为 57 的字符 "9", 但是在数字上下文场景中得到的是数字 57
###### SET
如果需要保存很多 true/false 值, 可以考虑合并这些列得到一个 SET 数据类型, 它在 MySQL 内部是以一系列打包的位的集合来表示的; 这样就有效的利用了存储空间, 但是主要的缺点是改变列的定义的代价高; 一种替代 SET 的方式是使用一个整数包装一系列的位, 例如可以把 8 个位包装成一个 TINYINT 中, 并且按位操作来使用

##### 选择标识符 (identifier)
为标识列选择合适的数据类型非常重要, 一般来说更有可能用标识列与其他值进行比较, 或通过表示列寻找其他列, 标识列也可能在另外的表中作为外键使用等等
###### 整数类型
整数通常是标识列最好的选择, 因为它们很快并且可以使用 AUTO_INCREMENT
###### ENUM 和 SET 类型
对于标识列来说, ENUM 和 SET 类型通常是一个糟糕的选择
###### 字符串类型
如果可能, 应该避免使用字符串作为标识列, 因为它们很消耗空间, 并且通常比数字慢  
对于完全 "随机" 的字符串也需要多加注意, 例如 MD5, SHA1(), UUID() 产生的字符串, 这些函数生成的新值会任意分布在很大的空间内, 这会导致 INSERT 以及一些 SELECT 语句变慢
- 因为插入值会随机写道索引的不同位置, 所以使得 INSERT 语句更慢, 这会导致页分裂, 磁盘随机访问, 以及对于聚簇存储引擎产生聚簇索引碎片
- SELECT 语句会变得更慢, 因为逻辑上相邻的行会分布在磁盘和内存的不同地方
- 随机值导致缓存对所有类型的查询语句效果都很差, 会使得缓存赖以工作的访问局部性原理失效

##### 特殊类型数据
某些类型的数据并不直接与内置类型一致, 例如低于秒级精度的时间戳, IPv4 的地址

#### MySQL Schema 设计中的陷阱
- 太多的列
- 过度使用枚举
- 错误使用 SET

#### 范式和反范式
##### 范式的优点和缺点
范式可以带来以下好处
- 范式化的更新奥做通常比反范式化要快
- 当数据较好地范式化时, 就只有很少或者没有重复数据, 所以只需要修改更少的数据
- 范式化表通常更小, 可以更好的放在内存里, 所以会执行的更快
- 很少冗余的数据则表示更少需要 DISTICT 或 GROUP BY 语句

缺点是通常需要关联, 稍微复杂一些的查询语句在符合范式的 Schema 上至少需要一次关联, 也许更多, 这个代价昂贵

##### 反范式的优点和缺点
TODO

##### 混用范式化和反范式化
TODO

#### 缓存表和汇总表
TODO

#### 加快 ALTER TABLE 操作的速度
TODO

#### 总结
TODO
