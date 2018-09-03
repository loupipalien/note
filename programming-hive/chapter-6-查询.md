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
