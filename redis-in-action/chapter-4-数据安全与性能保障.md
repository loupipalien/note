### 数据安全与性能保障

#### 持久化选项
Redis 提供了两种不同的持久化方法来将数据存储到硬盘中
- 快照(snapshotting): 可以将存在于某一时刻的所有数据都写入硬盘里面
- 追加文件 (append-only file, AOF): 会在执行写命令时, 将被执行的写命令复制到硬盘里面

这两种持久化方法既可以同时使用, 又可以单独使用, 在某些情况下甚至可以两种方法都不使用, 具体选择哪种持久化方法需要使用用户的数据以及应用来决定

##### 快照持久化
Redis 可以通过创建快照来获得存储在内存里的数据在某个时间点上的副本, 可以复制到其他服务器也可以留在本地以便重启服务器使用  
根据配置, 快照将被写入 dbfilename 选项指定的文件里面, 并存储在 dir 选项指定的路径上, 如果新的快照文件创建完毕之前, Redis, 系统, 硬盘三者中任一一个崩溃都将导致 Redis 丢失最后一次创建快照之后写入的数据; 创建快照的方式有以下几种
- 客户端可以通过项 Redis 发送 BGSAVE 的命令来创建一个快照, Redis 会调用 fork 来创建一个子进程来复制写入快照
- 客户端还可以通过向 Redis 发送 SAVE 命令来创建一个快照, 接到 SAVE 命令的 Redis 服务器在快照创建完毕之前将不再响应其他命令
- 用户设置了 save 配置项, 例如 `save 60 10000`,  自最后一次快照之后开始算起 60 秒内有 10000 写入则自动触发 BGSAVE 命令; 支持多个 save 配置项
- 当 Redis 通过 SHUTDOWN 命令接收到关闭服务器的请求时, 或者接收到标准 TERM 信号时, 会执行一个 SAVE 命令, 会阻塞所有客户端的命令, 并在 SAVE 命令完成之后关闭服务器
- 当一个 Redis 服务器连接另一个 Redis 服务器, 并向对方发送 SYNC 命令来开始复制操作的时候, 如果主服务器当前没有执行 BGSAVE 操作则会触发 BGSAVE 命令

##### AOF 持久化
AOF 持久化会将被执行的写命令写到 AOF 文件的末尾, 以此来记录数据发生的变化, 因此 Redis 只要从头到尾重新执行一次 AOF 文件包含的所有写命令, 就可以恢复 AOF 文件所记录的数据集; AOF 持久化可以通过 `appendonly yes ` 配置项来打开

| 选项 | 同步频率 |
| :--- | :--- |
| always | 每个写命令都要同步到硬盘, 这样做会严重降低 Redis 的速度 (推荐) |
| everysec | 每秒执行一次同步, 显式的将多个命令同步到硬盘 |
| no | 让操作系统决定应该何时同步 |


#### 复制
关系数据库通常会使用一个主服务器向多个从服务器发送更新, 并使用从服务器来处理所有读请求, Redis 也采用了同样的方法来实现自己的复制特性, 并将其作为扩展的一种手段  
在接收到主服务器发送的数据初始副之后, 客户端每次向主服务器进行写入时, 从服务器都会实时得到更新, 在部署好主从服务器之后, 客户端就可以向任意一个从服务器发送读请求, 从而做到读写分离和负载均衡

##### 对 Redis 的复制相关选项进行配置
可以通过发送 `SLAVEOF host port` 命令来让服务器开始复制一个新的主服务器, 通过发送 `SLAVEOF no one` 命令来让服务器终止复制操作

##### Redis 复制的启动过程
从服务器在连接一个主服务器的时候, 主服务器会创建一个快照文件并发送到从服务器, 但这只是主从复制过程的第一步, 以下列出了当从服务器连接主服务器时, 主从服务器执行的所有操作

| 步骤 | 主服务器 | 从服务器 |
| :--- | :--- | :--- |
| 1 | (等待命令进入) | 连接 (或重连) 主服务器, 发送 SYNC 命令 |
| 2 | 开始执行 BGSAVE, 并使用缓存区记录 BGSAVE 之后执行的所有写命令 | 根据配置选项来决定是继续使用现有的数据 (如果有的话), 来处理客户端的命令请求, 还是向发送请求的客户端返回错误 |
| 3 | BGSAVE 执行完毕, 向从服务器发送快照文件, 并在发送期间继续使用缓冲区记录被执行的命令 | 丢弃所有旧数据 (如果有的话), 开始载入主服务器发送的快照 |
| 4 | 快照文件发送完毕, 开始向从服务器发送存储在缓冲区里的写命令 | 完成对快照文件的解释操作, 开始接受命令请求 |
| 5 | 缓冲区存储的写命令发送完毕, 开始每执行一个写命令就向从服务器发送相同的写命令 | 执行主服务器发送的所有存储在缓冲区里的写命令, 并从现在开始, 接收并执行主服务器传来的每个写命令 |

当一个从服务器连接到一个已有的主服务器时, 有时可以重用已有的快照文件

| 当有新的从服务器连接主服务器时 | 主服务的操作 |
| :--- | :--- |
| 上表中步骤 3 尚未执行时 | 所有从服务器会接收到相同的快照文件和相同的缓冲区写命令 |
| 上表中步骤 3 正在执行或已经执行了 | 当主服务器与较早进行连接的从服务器执行完复制所需的 5 个步骤之后, 主服务器会与新连接的服务器执行一次新的步骤 1 至 5 |

##### 主从链
从服务器对从服务器进行复制在操作上和从服务器对主服务器进行复制的唯一区别在于, 如果从服务器 X 拥有从服务器 Y, 那么当从服务器 X 执行上表步骤 4 时, 它将断开与从服务器 Y 的连接, 导致从服务器 Y 需要重新连接并重新同步

##### 检验硬盘写入
检查 INFO 命令输出结果中 aof_pending_bio_fsync 属性的值是否为 0, 如果是的话那么就表示服务器已经将已知额所有数据保存到硬盘里了

#### 处理系统故障
TODO

#### Redis 事务
TODO

#### 非事务型流水线
TODO

#### 关于性能方面的注意事项
TODO
