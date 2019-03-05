### 运行 ZooKeeper

#### 配置 ZooKeeper 服务器
ZooKeeper 服务器启动时从名为 zoo.cfg 的配置文件中读取配置; data 目录下的 myid 文件用于区分各个服务器

##### 基本配置
- clientPort
客户端所连接的服务器所监听的 TCP 端口, 默认情况下, 服务端会监听在所有的网络连接接口上的这个端口, 除非设置了 clientPortAddress 参数; 默认端口是 2818
- dataDir 和 dataLogDir
dataDir 用于配置内存数据库保存的模糊快照的目录, 如果某个服务器为集群中的一台, id 文件也保存在该目录下; dataDir 不需要配置到专用的存储设备上, 快照会以后台线程的方式写入, 且并不会锁定数据库, 因为其写入方式并不是同步方式; dataLogDir 用于存储日志文件, 事务日志对该目录所处的存储设备上的其他活动更加敏感, 服务端会尝试进行顺序写入事务日志, 因为服务端在确认一个事务前必须将数据同步到存储中, 该存储设备的其他活动可能会导致同步操作缓慢, 从而影响写入的吞吐能力
- tickTime
tick 的时长单位为毫秒, tick 为 ZooKeeper 使用的基本时间度量单位; ZooKeeper 中使用的超时时间单位使用 tickTime 指定的, 所以最小的超时时间为一个 tickTime, 客户端最小的超时时间为两个 tickTime; tickTime 的默认值为 3000 毫秒, 更低的值可以更快的发现超时问题, 但也会导致更高的网络流量 (心跳消息) 和更高的 CPU 使用率 (会话存储器的处理)

##### 存储配置
- preAllocSize
用于设置预分配的事务日志文件 (ZooKeeper.preAllocSize) 的大小值, 以 KB 为单位; 当写入事务日志文件时, 服务端每次会分配 preAllocSize 的存储大小, 通过这种方式可以分摊文件系统将磁盘分配存储空间和更新元数据的开销, 更重要的是减少了文件寻址操作的次数; 其默认大小是 64 MB, 缩小该值的一个原因是事务日志永远不会达到这么大, 因为每次快照后都会重新启动新的事务日志; 默认值适用于默认的 snapCount 值和平均超过 512 字节的情况
- snapCount
指定每次快照之间的事务数 (ZooKeeper.snapCount); 当 ZooKeeper 服务器重启后需要恢复其状态, 恢复时两大时间因素是读取快照时间以及快照启动后的执行时间; snapCount 的默认值是 100000, 因为进行快照时会影响性能, 所有集群中所有服务器最好不要在同一时间进行快照操作, 只要仲裁服务器不会一同进行快照, 处理时间就不会受到影响, 因此每次快照中实际的事务数为一个接近 snapCount 值的随机数; 如果 snapCount 数已经达到, 但前一个快照正在进行中, 新的快照将并不会开始, 服务器也将等到下一个 snapCount 数量的事务后再开启一个新的快照
- autopurge.snapRetainCount
当进行清理数据操作时, 需要保留在快照数量和对应的事务日志文件数量; ZooKeeper 会定期对快照和事务日志进行垃圾回收, autopurge.snapRetainCount 的最小值是 3, 也是默认值的大小
- autopurge.purgeInterval
对快照和日志进行垃圾回收操作的时间间隔的小时数; 如果设置为零, 默认情况下将不会进行垃圾回收, 而需要通过脚本 zkCleanup.sh 手动进行
- fsync.warningthresholdms
触发警告的存储同步时间阈值, 以毫秒为单位; ZooKeeper 服务器在应答变化消息前会同步变化情况到存储中, 如果同步系统调用消耗了太长时间, 系统性能就会受到严重影响, 服务器会跟踪同步调用的持续时间, 如果超过 fsync.warningthresholdms 就会产生一个警告消息, 默认值为 1000 毫秒
- weight.x = n
该选项常以一组参数进行配置, 该选项指定组成一个仲裁机构的某个服务器的权重为 n, 其权重值指示了该服务器在进行投票时的权重值; 默认值为 1
- traceFile
持续跟踪 ZooKeeper 的操作, 并将操作记录到跟踪文件中, 跟踪文件名为 traceFile.year.month.day; 除非设置了该选项 (requestTraceFile), 否则跟踪功能将不会被启用; 确保不要将跟踪日志文件写到日志文件的存储设备中, 因为会争用磁盘的 IO

##### 网络配置
- globalOutstandingLimit
ZooKeeper 中待处理请求的最大值 (zookeeper.globalOutstandingLimit); ZooKeeper 客户端提交的请求比 ZooKeeper 服务端处理请求要快很多, 服务端将会对请求进行队列化, 最终可能导致服务端的内存溢出; 
