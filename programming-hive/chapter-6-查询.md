### 查询

#### SELECT ... FROM 语句
SELECT 是 SQL 中的射影算子, FROM 子句标识了从哪个表, 视图或嵌套查询中选择记录
...

##### 使用正则表达式来指定列
TODO

##### 使用列值进行计算
TODO

##### 算术运算符
Hive 支持所有典型的算术运算符

|运算符|类型|描述|
|A + B|数值| A 和 B 相加|
|A - B|数值| A 减去 B|
|A * B|数值| A 和 B 相乘|
|A / B|数值| A 除以 B|
|A % B|数值| A 除以 B 余数|
|A & B|数值| A 和 B 按位取与|
|A | B|数值| A 和 B 按位取或|
|A ^ B|数值| A 和 B 按位取异或|
|~A|数值| A 按位取反|

算术运算符接受任意的数值类型, 不过如果数据类型不同, 那么两种类型中值范围较小的那个数据类型将转换为其他范围更广的数据类型; z在进行算术运算时, 用户需要注意数据移除或者数据下溢的问题

##### 使用函数
1. 数学函数

|返回值类型|样式|描述|
|---|---|---|
|BIGINT|round(DOUBLE d)|返回 DOUBLE 类型 d 的 BIGINT 类型的近似值|
|DOUBLE|round(DOUBLE d, INT n)|返回 DOUBLE 类型 d 的保留 n 位小数的 DOUBLE 类型的近似值|
|BIGINT|floor(OUBBLE d)|d 是 DOUBLE 类型的, 返回 <= d 的最大的 BIGINT 类型值|
|BIGINT|ceil(DOUBLE d), ceiling(DOUBLE d)|d 是 DOUBLE 类型的, 返回 >= d 的最大的 BIGINT 类型值|
|DOUBLE|rand(), rand(INT seed)|返回一个 DOUBLE 类型的随机数, 整数 seed 是随机因子|
|DOUBLE|exp(DOUBLE d)|返回 e 的 d 幂次方, 返回的是个 DOUBLE 类型值|
|DOUBLE|ln(DOUBLE d)|以 e 为底的 d 的对数, 返回 DOUBLE 类型值|
|DOUBLE|log10(DOUBLE d)|以 10 为底的 d 的对数, 返回 DOUBLE 类型值|
|DOUBLE|log2(DOUBLE d)|以 2 为底的 d 的对数, 返回 DOUBLE 类型值|
|DOUBLE|log(DOUBLE base, DOUBLE d)|以 base 为底的 d 的对数, 返回 DOUBLE 类型值|
|DOUBLE|pow(DOUBLE d, DOUBLE p), power(DOUBLE d, DOUBLE p)|计算 d 的 p 次幂, 返回 DOUBLE 类型值|
|DOUBLE|sqrt(DOUBLE d)|d 是 DOUBLE 类型的, 计算 d 的平方根|
|STRING|bin(BIGINT i)|计算二进制 i 的 STRING 类型值, 其中 i 是 BIGINT 类型|
|STRING|hex(STRING str)|计算十六进制 i 的 STRING 类型值, 其中 i 是 BIGINT 类型|
|STRING|hex(STRING str)|计算十六进制表达的值 str 的 STRING 类型值|
|STRING|hex(BINARY b)|计算二进制表达的值 i 的 STRING 类型值|
|STRING|unhex(STRING i)|hex(STRING str) 的逆方法|
|STRING|conv(BIGINT num, INT from_base, INT to_base)|将 BIGINT 类型的 num 从 from_base 进制转换成 to_base 进制, 并返回 STRING 类型的结果|
|STRING|conv(STRING num, INT from_base, INT to_base)|将 STRING 类型的 num 从 from_base 进制转换成 to_base 进制, 并返回 STRING 类型的结果|
|DOUBLE|abs(DOUBLE d)|计算 DOUBLE 类型值 d 的绝对值, 返回结果也是 DOUBLE 类型的|
|INT|pmod(INT i1, INT i2)|INT 值 i1 对 INT 值 i2 取模, 结果也是 INT 类型的|
|DOUBLE|pmod(DOUBLE i1, DOUBLE i2)|DOUBLE 值 i1 对 DOUBLE 值 i2 取模, 结果也是 DOUBLE 类型的|
|DOUBLE|sin(DOUBLE d)|在弧度度量中, 返回 DOUBLE 类型值 d 的正弦值, 结果也是 DOUBLE 类型的|
|DOUBLE|asin(DOUBLE d)|在弧度度量中, 返回 DOUBLE 类型值 d 的反正弦值, 结果也是 DOUBLE 类型的|
|DOUBLE|cos(DOUBLE d)|在弧度度量中, 返回 DOUBLE 类型值 d 的余弦值, 结果也是 DOUBLE 类型的|
|DOUBLE|acos(DOUBLE d)|在弧度度量中, 返回 DOUBLE 类型值 d 的反余弦值, 结果也是 DOUBLE 类型的|
|DOUBLE|tan(DOUBLE d)|在弧度度量中, 返回 DOUBLE 类型值 d 的正切值, 结果也是 DOUBLE 类型的|
|DOUBLE|atan(DOUBLE d)|在弧度度量中, 返回 DOUBLE 类型值 d 的反正切值, 结果也是 DOUBLE 类型的|
|DOUBLE|degrees(DOUBLE d)|将 DOUBLE 类型的弧度值 d 转换为角度值, 结果是 DOUBLE 类型的|
|DOUBLE|redians(DOUBLE d)|将 DOUBLE 类型的角度值 d 转换为弧度值, 结果是 DOUBLE 类型的|
|INT|positive(INT i)|返回 INT 类型值 i|
|DOUBLE|positive(DOUBLE d)|返回 DOUBLE 类型值 d|
|INT|negative(INT i)|返回 INT 类型值 i|
|DOUBLE|negative(DOUBLE d)|返回 DOUBLE 类型值 d|
|FLOAT|sign(DOUBLE d)|如果 DOUBLE 类型值 d 是正数的话, 则返回 float 值 1.0, 如果 d 是负数的话, 则返回 float 值 -1.0, 否则返回 0.0|
|DOUBLE|e()|数学常数 e, 返回是 DOUBLE 类型的|
|DOUBLE|pi()|数学常数 pi, 返回是 DOUBLE 类型的|

2. 聚合函数
聚合函数是一类比较特殊的函数, 其可以对多行进行一些计算, f然后得到一个结果值

|返回值类型|样式|描述|
|---|---|---|
|BIGINT|count(*)|计算总行数, 包括含有 NULL 值的行|
|BIGINT|count(expr)|计算提供的 expr 表达式的值非 NULL 的行数|
|BIGINT|count(DISTINCT expr [, expr_.])|计算提供的 expr 表达式的值排重后非 NULL 的行数|
|DOUBLE|sum(col)|计算指定行的值的和|
|DOUBLE|sum(DISTINCT col)|计算排重后值的和|
|DOUBLE|avg(col)|计算指定行的值的平均值|
|DOUBLE|avg(DISTINCT col)|计算排重后的平均值|
|DOUBLE|min(col)|计算指定行的最小值|
|DOUBLE|max(col)|计算指定行的最大值|
|DOUBLE|variance(col), var_pop(col)|返回集合 col 中的一组数值的方差|
|DOUBLE|var_samp(col)|返回集合 col 中一组数值的样本方差|
|DOUBLE|stddev_pop(col)|返回一组数值的标准偏差|
|DOUBLE|stddev_samp(col)|返回一组数值的标准样本偏差|
|DOUBLE|covar_pop(col1, col2)|返回一组数值的协方差|
|DOUBLE|covar_samp(col1, col2)|返回一组数值的样本协方差|
|DOUBLE|corr(col1, col2)|返回两组数值的相关系数|
|DOUBLE|percentile(BIGINT int_expr, p)|int_expr 在 p (范围是: [0,1]) 处的对应百分比, 其中 p 是一个 DOUBLE 型数值|
|ARRAY|percentile(BIGINT int_expr, ARRAY(p1[, p2] ...))|int_expr 在 p (范围是: [0,1]) 处的对应百分比, 其中 p 是一个 DOUBLE 型数组|
|DOUBLE|percentile_approx(DOUBLE col, p[, NB])|col 是 p (范围是: [0,1]) 处的对应百分比, 其中 p 是一个 DOUBLE 型数值, NB 是用于估计的直方图中的仓库数量 (默认是 10000)|
|ARRAY|percentile_approx(DOUBLE col, ARRAY(p1[, p2] ...)[, NB])|col 是 p (范围是: [0,1]) 处的对应百分比, 其中 p 是一个 DOUBLE 型数组, NB 是用于估计的直方图中的仓库数量 (默认是 10000)|
|ARRAY<STRUCT{'x',''y}>|histogram_numeric(col, NB)|返回 NB 数量的直方图仓库数组, 返回结果中的值 x 是中心, 值 y 是仓库的高|
|ARRAY|collect_set(col)|返回集合 col 元素排重后的数组|

通常, 可以通过设置属性 hive.map.aggr = true 来提高聚合性能,这个设置会触发在 map 阶段进行的 "顶级" 聚合过程 (非顶级的聚合过程会在执行一个 group by 后进行), 不过这个设置将需要更多的内存

3. 表生成函数
与聚合函数 "相反的" 一类函数就是所谓的表生成函数, 其可以将单列扩展成多列或者多行, 在使用表生成函数时, Hive 要求使用别名

|返回值类型|样式|描述|
|---|---|---|
|N 行结果|explode(ARRAY array)|返回 0 到多行结果, 每行都对应输入 array 数组中的一个元素|
|N 行结果|explode(MAP map)|返回 0 到多行结果, 每行对应每个 map 键值对, 其中一个字段是 map 的键, 另一个字段对应着 map 的值|
|数组的类型|explode(ARRAY<TYPE> a)|对于 a 中的每个元素, explode() 会生成一行记录包含这个元素|
|结果插入表中|inline(ARRAY<STRUCT[, STRUCT]>)|将结构体数组提取出来并插入到表中|
|TUPLE|json_tuple(STRING jsonstr, p1, p2, ..., pn)|本函数可以接受多个标签名称, 对输入的 JSON 字符串进行处理, 与 get_json_object 这个 UDF 类似, 但是此函数更高效, 可以通过一次调用能获得多个键值|
|TUPLE|parse_url_tuple(url, partname1, partname2, ..., partnameN), 其中 N > 1|从 URL 中解析出 N 个部分信息, 其输入参数是 URL, 以及多个要抽取的部分的名称; 所有输入的参数类型都是 STRING; 部分名称是大小写敏感的, 而且不应该包含有空格: HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, USERINFO, QUERY<KEY NAME>|
|N 行结果|stack(INT n, col1, ..., colM)|把 M 列转换为 N 行, 每行有 M / N 个字段, 其中 n 为常数|

4. 其他内置函数
TODO

##### LIMIT 语句
LIMIT 子句用于限制返回的行数

##### 列别名

##### 嵌套 SELECT 语句

##### CASE ... WHEN ... THEN 句式
case ... when ... then 语句和 if 语句类似, 用于处理单个列的查询结果

##### 什么情况下 Hive 可以避免进行 MapReduce
大多数情况下 Hive 查询都会触发一个 Mapreduce 任务, Hive 中对某些情况的查询可以不必使用 Mapreduce, 也就是所谓的本地模式， 如
```
select * from employees;
```
这种情况下, Hive 可以简单的读取 employees 对应的存储目录下的文件; 对于 where 语句中过滤条件只是分区字段的这种情况也是无需 MapReduce 过程的; 此外, 如果设置 hive.exec.mode.local.auto = true, Hive 还会尝试使用本地模式执行其他操作

#### WHERE 语句
select 语句用于选取字段, where 语句用于过滤条件, 两者结合使用可以查找到符合过滤条件的记录   
,,,
where 语句中不能使用列别名  
...

##### 谓词操作符

|操作符|支持的数据类型|描述|
|---|---|---|
|A = B|基本数据类型|如果 A 等于 B 则返回 TRUE, 反之返回 FALSE|
|A <=> B|基本数据类型|如果 A 和 B 都为 NULL 则返回 TRUE, 其他的和符号 (=) 操作符的结果一致, 如果任一为 NULL, 则结果为 BULL|
|A == B|没有|错误的语法, SQL 使用 = 而不是 ==|
|A <> B, A != B|基本数据类型|A 或者 B 为 NULL 则返回 NULL; 如果 A 不等于 B 则返回 TRUE, 否则返回 FALSE|
|A < B|基本数据类型|A 或者 B 为 NULL 则返回 NULL; 如果 A 小于 B 则返回 TRUE, 否则返回 FALSE|
|A <= B|基本数据类型|A 或者 B 为 NULL 则返回 NULL; 如果 A 小于或等于 B 则返回 TRUE, 否则返回 FALSE|
|A > B|基本数据类型|A 或者 B 为 NULL 则返回 NULL; 如果 A 大于 B 则返回 TRUE, 否则返回 FALSE|
|A >= B|基本数据类型|A 或者 B 为 NULL 则返回 NULL; 如果 A 大于或等于 B 则返回 TRUE, 否则返回 FALSE|
|A [NOT] BETWEEN B AND C|基本数据类型|如果 A, B, C 任一为 NULL 则返回 NULL; 如果 A 的值大于或等于 B 而且小于或等于 C, 则返回 TRUE, 否则返回 TRUE; 使用关键字 NOT 则可达到相反的效果|
|A IS NULL|所有数据类型|如果 A 等于 NULL 则返回 TRUE, 否则返回 FALSE|
|A IS NOT NULL|所有数据类型|如果 A 不等于 NULL 则返回 TRUE, 否则返回 FALSE|
|A [NOT] LIKE B|STRING 类型|B 是一个 SQL 下的简单正则表达式, 如果 A 与其匹配的话, 则返回 TRUE, 否则返回 FALSE; B 的表达式说明如下: 'x%' 表示 A 必须是字母 x 开头, '%x' 表示 A 必须是字母 x 结尾, '%x%' 表示 A 包含有字母 x, 可以位于开头结尾或中间; 类似的下划线 '_' 匹配单个字符; B 必须要和整个字符串 A 匹配才行, 如果使用 NOT 关键字则可以达到相反的效果|
|A RLIKE B, A REGEXP B|STRING 类型|B 是一个正则表达式, 如果 A 与其相匹配, 则返回 TRUE， 否则返回 FALSE; 匹配使用的是 JDK 中的正则表达式接口实现的, 因为正则规则也依据其中的规则; 正则表达式必须和整个字符串 A 相匹配|

##### 关于浮点数比较
浮点数比较的一个常见的陷阱出现在不同类型间作比较的时候 (也就是 FLOAT 和 DOUBLE 比较); 用户在写一个浮点数的时 (比如 0.2), Hive 会将改值保存为 DOUBLE 类型的, 当与 FLOAT 类型的的值比较时, 这意味着 Hive 将隐式的将其转换为 DOUBLE 类型后再比较; 但事实上这样是行不通的, 因为数字 (0.2) 不能够使用 FLOAT 或者 DOUBLE 进行准确比较, 0.2 的最近似的精确值应略大于 0.2, 也就是 0.2 后面的若干个 0 后存在的非零数值, 简化来说就是 0.2 对于 FLOAT 类型来说是 0.2000001, 而对于 DOUBLE 类型是 0.200000000001, 这是因为一个 8 字节的 DOUBLE z值具有更多的xiaoshuwe; 当 FLOAT 类型隐式转换为 DOUBLE 时变为 0.200000100000, 这个值实际要比 0.200000000001 大

##### LIKE 和 RLIKE
RLIKE 是支持正则表达式版的 LIKE  
...

##### GROUP BY 语句
group by 语句通常会和聚合函数一起使用, 按照一个或多个列对结果进行分组, 然后对每个组执行聚合操作
....

##### HAVING 语句
HAVING 子句允许用户通过一个简单的语法完成原本需要通过子查询才能对 group by 语句产生的分组进行条件过滤的任务  
...

#### JOIN 语句
Hve 通常支持的是 SQL JOIN 语句, 但只支持等值连接

##### INNER JOIN
内连接中只有进行连接的两个表中都存在与连接标准相匹配的数据才会被保留下来; 标准额 SQL 是支持对连接关键词进行非等值连接的
```
select a.ymd, a.price_close, b.price_close
from stocks a join stocks b on a.ymd <= b.ymd
where a.symbol = 'AAPL' and b.symbol = 'IBM';
```
这个语句在 Hive 中是非法的, 主要原因是同通过 MapReduce 很难实现这种类型的连接, 同时 Hive 目前还不支持在 on 子句中的谓词间使用 or  
...
用户可以对多余 2 张表的多张表进行连接操作
```
select a.ymd, a.price_close, b.price_close, c.price_close
from stocks a join stocks b on a.ymd <= b.ymd join stocks c on a.ymd = c.ymd
where a.symbol = 'AAPL' and b.symbol = 'IBM' and c.symbol = 'GE';
```
大多数情况下, Hive 会对每对 join 连接对象启动一个 MapReduce 任务; 在本例中首先启动一个 MapReduce job 对表 a 和表 b 进行连接操作, 然后会再启动一个 MapReduce job 将第一个 MapReduce job 的输出表和表 c 进行连接操作

##### JOIN 优化
当对 3 个或者更多个表进行 JOIN 连接时, 如果每个 on 子句都使用相同的连接键的话, 那么只会产生一个 MapReduce job; Hive 同时假定查询中最后一个表是最大的那个表, 在对每行记录进行连接操作时, 它会尝试将其他表缓存起来, 然后扫描最后那个表进行计算, 因此用户需要保证连接查询中的表的大小从左到右是依次增加的  
...  
幸运的是, 用户并非总是要将最大的表放置在查询语句的最后面的, 这是因为 Hive 还提供了一个 "标记" 机制来显式的告诉查询优化器那张表是大表
```
select /*+ STREAMTABLE(s) */ s.ymd, s.price_close, s.price_close, d.dividend
from stocks s join dividends d on s.ymd = d.ymd and s.symbol = d.symbol
where s.symbol = 'AAPL';
```
Hive 将会尝试将表 stocks 作为驱动表, 即使其在查询中不是位于最后面的; 还有另外一个类似的非常重要的优化叫做 map-side JOIN

##### LEFT OUTER JOIN
左外连接通过关键字 LEFT OUTER 进行标识, 在这种 JOIN 连接操作中, JOIN 操作符左边表中符合 where 子句的所有记录将会被返回, JOIN 操作符右边表中如果没有符合 on 后面连接条件的记录时, 那么从右边表指定选择的列的值将会是 NULL

##### OUTER JOIN
在 where 子句中增加分区过滤器可以加快查询速度, 对以下两张表的 exchange (分区) 字段增加谓词限定
```
select s.ymd, s.symbol, s.price_close, d.dividend
from stocks s left outer join dividends d on s.ymd = d.ymd and s.symbol = d.symbol
where s.symbol = 'AAPL' and s.exchange = 'NASDAQ' and d.exchange = 'NASDAQ';
```
以上语句的效果会发现和内连接是一样的, 这是因为先执行了 join 语句, 然后再将结果通过 where 语句进行过滤; 这样的结果并不想要的, 一个直接有效的解决方法是: 移除掉 where 语句中对 dividends 表的过滤条件, 也就是去掉 d.exchange = 'NASDAQ' 的这个限制条件; 但这样的运行效率不是令人满意的, Hive Wiki 中宣称 where 语句的 (分区) 过滤条件可以放在 on 语句中
```
select s.ymd, s.symbol, s.price_close, d.dividend
from stocks s left outer join dividends d on s.ymd = d.ymd and s.symbol = d.symbol
and s.symbol = 'AAPL' and s.exchange = 'NASDAQ' and d.exchange = 'NASDAQ';
```
但事实上, 对于外连接会忽略掉分区过滤条件, 对于内连接来讲这样确实是有效的  
幸运的是, 有一个适用于所有种类连接的解决方案, 那就是使用嵌套 select 语句
```
select s.ymd, s.symbol, s.price_close, d.dividend from
(select * from stocks where symbol = 'AAPL' and exchange = 'NASDAQ') s
left outer join
(select * from dividends where symbol = 'AAPL' and exchange = 'NASDAQ') d
on s.ymd = d.ymd;
```
嵌套 select 语句会按照要求执行 "下推" 过程, 即在数据进行连接之前会先进行分区过滤

##### RIGHT OUTER JOIN
右连接会返回右边表所有符合 where 语句的记录, 左表匹配不上的字段值使用 NULL 代替

##### FULL OUTER JOIN
完全外连接将会返回所有表中符合 where 语句条件的所有记录, 如果任一表的指定字段没有符合条件的值的话, 那么就使用 NULL 值替代

##### LEFT SEMI-JOIN
左半开连接会返回左边表的记录, 前提是其记录对于右边表满足 on 语句中的判定条件; 以下语句将试图返回限定的股息支付日内的股票交易记录, 不过这个查询 Hive 是不支持的
```
select s.ymd, s.symbol, s.price_close from stocks s
where s.ymd, s.symbol in (select d.ymd, d.symbol from dividends d);
```
不过可以使用如下的 left semi join 语句达到同样的目的
```
select s.ymd, s.symbol, s.price_close from stocks s
left semi join dividends d on s.ymd = d.ymd and s.symbol = d.symbol;
```
但需要注意的是, select 和 where 语句中不能引用到右边表中的字段 (另外 Hive 不支持 RIGHT SEMI-JOIN); semi join 比通常的 inner join 要更高效, 原因是对于左边表中的一条指定记录, 在右边表中一旦找到匹配的记录, Hive 就会立即停止扫描, 从这点来看左边表选择的列是可预测的

##### 笛卡儿积 JOIN
笛卡儿积是一种连接, 表示左边表的行数乘以右边表的行数等于笛卡尔积结果集额大小
```
select * from stocks join dividends
```
笛卡尔积会产生大量的数据, 和其他连接类型不同, 笛卡儿积不是并行执行的, 而且使用 MapReduce 计算架构的话, 任何方式都无法进行优化; 如果使用了错误的连接语法可能会导致产生一个执行时间长, 运行缓慢的笛卡儿积查询, 例如以下查询
```
select * from stocks join dividends where s.symbol = 'AAPL' and s.symbol = d.symbol;
```
这个优化在很多数据库中会被优化成内连接 (inner join), 但是在 Hive 中没有这个优化, 因此会在 where 语句的过滤条件前先进行笛卡儿积计算, 这个过程会很消耗时间; 如果设置属性 hive.mapred.mode 的值为 strict 的话, Hive 会阻止用户执行笛卡儿积查询

#####  MAP-SIDE JOIN
如果所有表中只有一张表是小表, 那么可以在最大的表通过 mapper 的时候将小表完全放在内存中, Hive 可以在 map 端执行连接过程 (称为 map-side join), 这是因为 Hive 可以和内存中的小表进行逐一匹配, 从而省略掉常规连接操作所需要的 reduce 过程, 即使对于很小的数据集, 这个优化也明显地要快于常规的连接操作, 其不仅减少了 reduce 过程, 而且有时还可以同时减少 map 过程的执行步骤  
在 Hive v0.7 的之前版本中如果想使用这个优化, 需要在查询语句中增加一个标记进行触发
```
select /*+ MAPJOIN(d) */ s.ymd, s.price_close, s.price_close, d.dividend
from stocks s join dividends d on s.ymd = d.ymd and s.symbol = d.symbol
where s.symbol = 'AAPL';
```
从 Hive v0.7 版本开始, 废弃了这种标记的方式, 不过如果增加了这个标记同样是有效的; 如果不加上这个标记, 那么这时用户需要设置属性 hive.auto.convert.join = true, 这样 Hive 才会在必要的时候启动这个优化, 默认情况下这个值是 false; 需要注意的是, 也可以配置能够使用这个优化的小表的大小, 如下是这个属性的默认值 (单位是字节) , hive.mapjoin.smalltable.filesize = 25000000; Hive 对于右外连接和全外连接不支持这个优化的  
如果所有表中的数据是分桶的, 那么对于大表在特定的情况下同样可以使用这个优化; 即表中的数据必须是按照 on 语句中的键进行分桶的, 而且其中一张表的分桶的个数必须是另一张表分桶个数的若干倍; 当满足这些条件时, 那么 Hive 可以在 map 阶段按照分桶数据进行连接; 在这种情况下, 不需要先获取到表中所有的内容, 之后才去和另一张表中每个分桶进行匹配连接; 同样这个优化默认是没有开启, 需要设置参数 hive.optimize.bucketmapjoin = true; 如果涉及的分桶表都具有相同的分桶数, 而且数据是按照连接键或桶的键进行排序的, 那么这时 Hive 可以设置一个更快的分类-合并连接, 同样需要设置以下参数才可开启:
```
set hive.input.format = org.apache.hadoop.hive.ql.io.BucketizedHiveInputFormat;
set hive.optimize.bucketmapjoin = true;
set hive.optimize.bucketmapjoin.sortedmerge = true;
```

#### ORDER BY 和 SORT BY
Hive 中 order by 语句和其他 SQL 方言一致, 会对查询结果集执行一个全局排序, 这也就是说会有一个所有的数据都通过一个 reducer 进行处理的过程, 对于大数据集这个过程可能很漫长  
Hive 增加了一种可供选择的方式, 也就是 sort by, 其只会在每个 reducer 中对数据进行排序, 也就是执行一个局部排序的过程, 这样可以保证每个 reducer 的输出数据都是有序的 (但并非全局有序)  
因为 order by 操作可能会导致运行时间很长, 如果属性 hive.mapred.mode = strict, 那么 Hive 要求这样的语句必须加油 limit 语句进行限制

#### 含有 SORT BY 的 DISTRIBUTE BY
distirbute by 控制 map 的输出在 reducer 中是如何划分的; 默认情况下, MapReduce 计算框架会依据 map 输入的键计算相应的哈希值, 然后按照得到的哈希值将键值对均匀分发到多个 reducer 中去; 但是不幸的是这意味着当使用 sort by 时, 不同 reducer 的输出内容会有明显的重叠, 至少对于排列顺序是这样的, 即使每个 reducer 的输出的数据都是有序的  
以下语句使用的 distribute by 将相同的股票交易码的记录会分发到同一个 reducer 中进行处理, 然后再用 sort by 进行排序
```
select s.ymd, s.symbol, s.price_close from stocks s
distribute by s.symbol sort by s.symbol asc, s.ymd asc
```
distribute by 和 group by 在其控制着 reducer 是如何接受一行行数据进行处理的方面是类似的; 需要注意的是, Hive 要求 distribute by 语句要在 sort by 语句之前

#### CLUSTER BY
在前面的例子中, 如果 distribute by 和 sort by 语句中涉及的列完全相同, 而且采用的是升序方式; 那么在这种情况下 CLUSTER BY 就等价于前面的两个语句, 相当于前面 2 个句子的一个简写方式
```
select s.ymd, s.symbol, s.price_close from stocks s cluster by s.symbol
```
使用 distribute by ... sort by 语句或简化版 cluster by 语句会剥夺 sort by 语句的并行性, 然而这样可以实现输出文件的数据是全局排序的

#### 类型转换
类型转换的语法是 cast(value as type), 当 value 是不合法的 type 值, Hive 将返回 NULL; 需要注意的是, 将浮点数转换成整数的推荐方式是 round(), floor()函数, 而不是使用类型转换操作符 cast

###### 类型转换 BINARY 值
Hive v0.8.0 版本中引入的 binary 类型的值只支持与 string 类型互转

#### 抽样查询
对于非常大的数据集, 有时候需要使用的是一个具有代表性的查询结果而不是全部结果; Hive 可以通过对表进行分桶抽样来满足这一需求  
假设 numbers 表中只有 number 字段, 其值是 1 到 10
```
select * from numbers tablesample(bucket 1 out of 2 on number) s;
```
分桶语句中的分母表示的是数据将会被散列的桶的个数, 而分子表示将会选择的桶的个数

##### 数据块抽样
Hive 提供了另外一种按照抽样百分比进行抽样的方式, 这种是基于行数的, 按照输入路径下的数据块百分比进行的抽样
```
select * from numbersflat tablesample(0.1 percent) s;
```
基于百分比的抽样方式提供了一个 hive.sample.seednumber 的变量, 用于控制基于数据块的调优的种子信息

##### 分桶表的输入裁剪
如果 tablesample 语句中指定的列和 cluster by 语句中指定的列相同, 那么 tablesample 查询就只会扫描涉及到表的哈希分区下的数据
```
create table numbers_bucketed(number int) cluster by(number) into 3 buckets;
set hive.enforce.bucketing = true;
insert overwrite table numbers_bucketed select number from numbers;
select * from numbers_bucketed tablesample(bucket 2 of 3 on number) s;
```

#### UNION ALL
union all 可以将 2 个或多个表进行合并, 每一个 union 子查询都必需具有相同的列, 而且对应的每个字段的字段类型必须是一致的
