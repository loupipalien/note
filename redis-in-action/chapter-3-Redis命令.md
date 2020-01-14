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
| SETBIT | SETBIT key-name offset value , 将字符串看作是二进制位串, 并将未串中偏移量为 offset 的二进制为位的值设置为 value |
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
| BLPOP | BLPOP key-name [key-name ...] timeout, 从第一个非空列表中弹出位于最左端的元素, 或者在 timeout 秒之内阻塞并等待可弹出的元素出现 |
| BRPOP | BRPOP key-name [key-name ...] timeout, 从第一个非空列表中弹出位于最右端的元素, 或者在 timeout 秒之内阻塞并等待可弹出的元素出现 |
| RPOPLPUSH | RPOPLPUSH source-key dest-key, 从 source-key 列表中弹出位于最右端的元素, 然后将这个元素推入 dest-key 列表的最左端, 并向用户返回这个元素 |
| BRPOPLPUSH | BRPOPLPUSH source-key dest-key timeout, 从 source-key 列表中弹出位于最右端的元素, 然后将这个元素推入 dest-key 列表的最左端, 并向用户返回这个元素, 如果 source-key 为空, 那么在 timeout 秒之内阻塞并等待可弹出的元素出现 |

#### 集合
Redis 的集合以无序的方式来存储多个各不相同的元素, 用户可以快速地对集合执行添加元素操作, 移除元素操作以及检查一个元素是否存在集合里

| 命令 | 用例和描述 |
| :--- | :--- |
| SADD | SADD key-name item [item ...], 将一个或多个元素添加到集合里, 并返回被添加元素当中原本并不存在于集合里面的元素数量 |
| SREM | SREM key-name item [item ...], 从集合里面移除一个或多个元素, 并返回被移除元素的数量 |
| SISMEMBER | SISMEMBER key-name item, 检查元素 item 是否存在于集合 key-name 里 |
| SCARD | SCARD key-name, 返回集合包含的元素的数量 |
| SMEMBERS | SMEMBERS key-name, 返回集合包含的所有元素 |
| SRANMEMBER | SRANMEMBER key-name [count], 从集合里面随机返回一个或多个元素, 当 count 为正数时, 命令返回的随机元素不会重复, count 为负数时, 命令返回的随机元素可能会出现重复 |
| SPOP | SPOP key-name dest-key item, 如果集合 source-key 包含元素 item, 那么从集合 source-key 里移除元素 item, 并将元素 item 添加到集合 dest-key 中, 如果 item 被成功移除命令返回 1, 否则返回 0 |
| SDIFFSTORE | - |
| SINTER | - |
| SINTERSTORE | - |
| SUNION | - |
| SUNIONSTORE | - |

#### 散列
Redis 的散列可以让用户将多个键值对存储到一个 Redis 键里

| 命令 | 用例和描述 |
| :--- | :--- |
| HMGET | HMGET key-name key [key ...], 从散列里获取一个或多个键的值 |
| HMSET | HMSET key-name key value [key value ...], 为散列里面的一个或多个键设置值 |
| HDEL | HDEL key-name key [key ...], 删除散列里面的一个或多个键值对, 返回成功找到并删除的键值对数量 |
| HLEN | HLEN key-name, 返回散列包含的键值对的数量 |
| HEXISTS | - |
| HKEY | - |
| HVALS | - |
| HGETALL | - |
| HINCRBY | - |
| HINCRBYFLOAT | - |


#### 有序集合
和散列存储着键与值之间的映射类似, 有序集合也存储着成员与分值之间的映射, 并且提供了分值处理命令, 以及根据分值大小有序的获取或扫描成员和分值的命令

| 命令 | 用例和描述 |
| :--- | :--- |
| ZADD |  ZADD key-name score member [score member], 将带有给定分值的成员添加到有序集合中 |
| ZREM | ZREM key-name member [member ...], 从有序集合里面移除给定的成员, 并返回被移除成员的数量 |
| ZCARD | ZCARD key-name, 返回有序集合包含的成员数量 |
| ZINCRBY | ZINCRBY key-name increment member, 将 member 成员分支上加上 increment |
| ZCOUNT | ZCOUNT key-name min max, 返回介于 min 和 max 之间的成员数量 |
| ZRANK | - |
| ZSCORE | - |
| ZRANGE | - |
| ZREVRANK | - |
| ZREVRANGE | - |
| ZRANGEBYSCORE | - |
| ZREVRNGEBYSCORE | - |
| ZREVRNGEBYRANK | - |
| ZREMRANGEBYSCORE | - |
| ZINTERSTORE | - |
| ZUNIONSTORE | - |

#### 发布与订阅
发送者 (publisher) 负责项频道发送二进制字符串消息 (binary string message), 每当有消息被发送到给定频道时, 频道的所有订阅者都会收到消息

| 命令 | 用例和描述 |
| :--- | :--- |
| SUBSCRIBE | SUBSCRIBE channel [channel ...], 订阅给定的一个或多个频道 |
| UNSUBSCRIBE | UNSUBSCRIBE [channel [channel ...]], 退订给定的一个或多个频道, 如果执行时没有给定任何频道, 那么退订所有频道 |
| PUBLISH | PUBLISH channel message, 向指定频道发送消息 |
| PSUBSCRIBE | PSUBSCRIBE pattern [pattern ...], 订阅与给定模式相匹配的所有频道 |
| PUNSUBSCRIBE | PUNSUBSCRIBE [pattern [pattern ...]], 退订给定的模式, 如果执行时没有给定任何模式, 那么退订所有模式 |

#### 其他命令

##### 排序
TODO

##### 基本的 Redis 事务
Redis 的基本事务 (basic transaction) 需要用到 MULTI 命令和 EXEC 命令, 这种事务可以让一个客户端在不被其他客户端打断的情况下执行多个命令

TODO

##### 键的过期时间
Redis 可以让一个键在给定的时限之后自动删除

| 命令 | 用例和描述 |
| :--- | :--- |
| PERSIST | PERSIST key-name, 移除键的过期时间 |
| TTL | TTL key-name, 查看给定键距离时间还有多少秒 |
| EXPIRE | EXPIRE key-name seconds, 让给定键在指定的秒数之后过期 |
| EXPIREAT | EXPIREAT key-name timestamp, 将给定键的过期时间设置为给定时间 |
| PTTL | - |
| PEXPIRE | - |
| PEXPIREAT | - |
