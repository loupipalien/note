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
