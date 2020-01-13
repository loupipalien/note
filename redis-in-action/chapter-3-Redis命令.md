### Redis 命令

#### 字符串
在 Redis 中, 字符串可以存储以下三种类型的值
- 字符串
- 整数
- 浮点数

Redis 中的自增命令和自减命令

| 命令 | 用例和描述 |
| :--- | :--- |
| INCR | INCR key-name, 将键存储的值加上 1 |
| DECR | DECR key-name, 将键存储的值减去 1 |
| INCRBY | INCRBY key-name amout, 将键存储的值加上整数 amount |
| DECRBY | DECRBY key-name amout, 将键存储的值减去整数 amount |
| INCRBYFLOAT | INCRBYFLOAT key-name amout, 将键存储的值加上浮点数 amount, 2.6+ 版本可用 |
| APPEND | APPEND key-name value, 将值 value 追加到给定键 key-name 当前存储的值的末尾 |
| GETRANGE | GETRANGE key-name start end, 获取一个由偏移量 start 至偏移量 end 范围内所有字符组成的子串, 包括 start 和 end |
| SETRANGE | SETRANGE key-name offset value, 将从 start 偏移量开始的子串设置为给定值 |
| GETBIT | GETBIT key-name offset, 将字符串看作是二进制位串 (bit string), 并返回位串中偏移量为 offset 的二进制位的值 |
| SETBIT | SETBIT key-name offset value, 将字符串看作是二进制位串, 并将未串中偏移量为 offset 的二进制为位的值设置为 value |
| BITCOUNT | BITCOUNT key-name [start end], 统计二进制位串值为 1 的二进制位的数量, 可以指定 start, end 范围统计 |
| BITOP | BITOP operation dest-key key-name [key-name ...], 对一个或多个二进制位串执行包括并 (AND), 或(OR), 异或(XOR), 非(NOT) 在内的任意的一种按位运算操作并将计算得出结果保存在 dest-key 中 |

#### 列表
Redis 的列表允许用户从序列的两端推入或弹出元素, 获取列表元素以及执行各种常见的列表操作等

| 命令 | 用例和描述 |
| :--- | :--- |
| RPUSH | RPUSH key-name value [value ...], 将一个或多个值推入列表的右端 |
| LPUSH | LPUSH key-name value [value ...], 将一个或多个值推入列表的左端 |
| RPOP | RPOP key-name, 移除并返回列表最右端的元素 |
| LPOP | LPOP key-name, 移除并返回列表最左端的元素 |
| LRANGE | LRANGE key-name start end, 返回列表从 start 偏移量到 end 偏移量范围内的所有元素, 包含 start 和 end |
| LTRIM | LTRIM key-name start end, 对列表进行修剪, 只保留从 start 偏移量到 end 偏移量范围内的元素, 包含 start 和 end |
| BLPOP | BLPOP key-name [key-name ...], 从第一个非空列表中弹出位于最左端的元素, 或者在 timeout 秒之内阻塞并等待可弹出的元素出现 |
